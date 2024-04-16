## 大数据核心工作

- 数据存储，对应分布式存储
  - Hadoop中的HDFS
- 数据计算，对应分布式计算
  - Hadoop中的MapReduce
  - Hive
  - Spark
  - Flink
- 数据传输
  - Kafka

# Hadoop

## 构成

### HDFS组件

分布式存储组件

### MapReduce组件

分布式计算组件

### YARN组件

分布式资源调度组件，可供用户整体调整大规模集群的资源使用

## 为什么需要分布式存储

- 数据量太大，单机存储能力有上线，需要靠数量来解决
- 数量的提升带来网络传输，磁盘读写，cpu，内存等方面的综合提升

## 分布式的基础架构分析

### 去中心化模式

没有明确的中心，众多服务器之间基于特定规则进行同步协调

### 中心化模式

主从模式，由中心节点统筹其他服务器的工作

Hadoop就是这个模式

## HDFS的基础架构

主节点：NameNode

从节点：DataNode

辅助节点：SecondaryNameNode



### NameNode

- 是一个独立的进程
- 负责管理HDFS整个文件系统
- 负责管理DataNode

### DataNode

- 是独立进程
- 主要负责数据的的存储，读写

### SecondaryNameNode

- 是独立进程
- 帮助NameNode玩车个元数据的整理

## HDFS的shell操作

### 进程启停管理

#### 一键启停脚本

`start-dfs.sh`一键启动HDFS集群

原理：

1. 在执行此脚本的机器上，启动SecondaryNameNode
2. 读取core-site.xml内容，确认NameNode所在机器，启动NameNode
3. 读取workers内容，确定DataNode所在机器，启动全部DataNode



`stop-dfs.sh`一键关闭HDFS集群

原理：

1. 在执行此脚本的机器上，关闭SecondaryNameNode
2. 读取core-site.xml内容，确认NameNode所在机器，关闭NameNode
3. 读取workers内容，确定DataNode所在机器，关闭全部DataNode

#### 单进程启停

单独控制**所在机器**的进程启停

旧用法：

`hadoop-daemon.sh (start/status/stop) (NameNode/DataNode/SecondaryNameNode)`



新用法：

`hdfs --daemon (start/status/stop) (NameNode/DataNode/SecondaryNameNode)`

### 文件系统操作命令

HDFS和Linux系统的文件管理相似，区分用协议头：

HDFS：`hdfs://(主机名:通讯端口号)/`最后的/是根目录的意思

Linux：`file:///`第三个/是根目录的意思

协议头会自动识别，一般不需要加



旧用法：

`hadoop fs [可选项]`



新用法：

`hdfs dfs [可选项]`

#### 创建文件夹

`hadoop fs/hdfs dfs -mkdir [-p] path`

path表示待创建的目录

-p表示会沿着路径创建父目录



比如：

```
hdfs dfs -mkdir -p /xxx/yyy
```

#### 查看指定目录下内容

`hadoop fs/hdfs dfs -ls [-h] [-R] [path]`

path表示指定的目录

-h表示人性化显示文件大小

-R表示递归查看指定目录及其子目录



比如：

```
hdfs dfs -ls /
```

#### 上传文件到HDFS指定目录

`hadoop fs/hdfs dfs -put [-f] [-p] localpath dstpath`

-f表示覆盖已存在文件

-p表示保留访问和修改时间，所有权和权限

localpath是本地文件地址

dstpath是HDFS中目标地址



比如：

```
hdfs dfs -put -f file:///xxx/yyy hdfs://node1:8020/test/
```

#### 查看HDFS文件内容

`hadoop fs/hdfs dfs -cat path [| more]`

| more是使用管道符，读取大文件，按空格翻页



比如：

```
hdfs dfs -cat /xxx | more
```

#### 下载HDFS文件

`hadoop fs/hdfs dfs -get [-f] [-p] path localpath`

path是HDFS中路径

localpath是本地路径，且必须是文件目录



比如：

```
hdfs dfs -get /xxx .
```

#### 拷贝HDFS文件

`hadoop fs/hdfs dfs -cp [-f] path dstpath`

path和dstpath都是HDFS中的路径

也可以改名

比如：

```
hdfs dfs -cp /xxx /yyy/
```

#### 追加数据到HDFS文件中

`hadoop fs/hdfs dfs -appendToFile loaclpath dstpath`

- 将给定本地文件的内容追缴到给定dst文件中
- 若dst文件不存在就创建该文件
- 若loaclpath为`-`，则输入是从标准输入中读取

**HDFS中无法单独改变文件某一行的内容**



比如：

```
hdfs dfs -appendToFile /xxx /yyy/test
```

#### 移动HDFS文件

`hadoop fs/hdfs dfs -mv path dstpath`

path和dstpath都是HDFS中的路径

也可以改名

比如：

```
hadoop fs/hdfs dfs -mv /xxx /yyy/
```

#### 删除HDFS文件

`hadoop fs/hdfs dfs -rm [-r] [-skipTrash] path`

-r用来删文件夹

-skipTrash是跳过回收站，直接删除

## HDFS用户权限

HDFS文件系统的超级用户是启动namenode的用户

修改所属用户和组：

`hadoop fs/hdfs dfs -chown [-R] user[:group] /xxx`

修改权限：

`hadoop fs/hdfs dfs -chmod [-R] 权限 /xxx`

-R是对子目录生效

## HDFS存储原理

存储的基本单位是block块，每个256M，可修改

HDFS会存有块的副本，默认是3个



上传文件时，可以通过以下命令临时修改：

`hadoop fs/hdfs dfs -D dfs.replication=2 -put /xxx /yyy/`

对于已经存在的HDFS文件，可以通过以下命令临时修改：

`hadoop fs/hdfs dfs -setrep [-R] 2 path`



检查文件的副本数：

`hdfs fsck path [-files [-blocks [-locations]]]`

-files表示列出路径内的文件状态

-files -blocks表示输出文件有几个块，几个副本

-files -blocks -locations表示输出每一个块的详情



**NameNode基于一批edits文件和一个fsimage文件完成文件系统的管理和维护**，也叫做元数据：edits文件和fsimage文件

### edit文件

记录历hdfs中的每一次操作，以及本次操作影响的文件其对应的block

edits文件大小达到上限后，开启新的edits记录

缺点：

当用户查找某文件内容时，需要在全部的edits中按顺序从头到尾搜索，效率很低

### fsimage文件

全部edit文件合并后的最终结果

会定期进行edits的合并：

- 如当前没有fsimage文件，将全部edits合并为第一个fsimage文件
- 如当前已经有fsimage文件，将全部edits和已存在的fsimage文件进行合并，形成新的fsimage文件

定期条件：

**默认为1小时，或者100w次事务之后**

默认60秒检查一次条件是否满足



是SecondaryNameNode来进行合并，通过http从NameNode拉取数据（edits和fsimage），合并完成后提供给NameNode

## HDFS数据写入流程

1. 客户端向NameNode发起请求
2. NameNode审核权限和剩余空间后，若满足条件，允许写入并告知客户端写入的DataNode地址
3. 客户端向指定的DataNode发送数据包
4. 被写入数据的DataNode同时完成数据的副本复制工作，并分发给其他的DataNode
5. 如DataNode1首先复制给DataNode2,然后**DataNode2复制给**DataNode3,DataNode4...
6. 写入完成后客户端通知NameNode，NameNode左元数据记录工作

注意：

- NameNode不负责数据写入，只负责元数据记录和权限审批
- 客户端直接向1台DataNode写数据，这个DataNode一般是离客户端网络距离最近的那一个
- 数据块副本的复制，由DataNode之间完成，通过构建pipline，按顺序复制分发

## HDFS数据读取流程

1. 客户端向NameNode发起请求
2. NameNode审核权限，若满足条件，允许读取，并返回此文件的block列表
3. 客户端拿到block列表后自行寻找DataNode读取即可

注意：

- NameNode不提供数据本身
- NameNode提供的block列表会基于网络距离提供离客户端最近的，因为一个block会有3份，就找最近的



网络距离：

最近是在同一台机器

然后是同一个局域网

然后是跨越交换机

然后是跨越数据中心

## 分布式计算

### 分散-汇总模式：

1. 将数据分片，多台服务器各自负责一部分数据处理
2. 然后将各自的结果，进行汇总处理
3. 最终得到想要的计算结果

MapReduce就是这个



### 中心调度-步骤执行模式：

1. 由一个节点作为中心调度管理
2. 将任务划分为几个具体步骤
3. 管理者安排每个机器执行任务
4. 最终得到结果数据

Spark，Flink就是这个

## MapReduce程序接口

- Map：提供分散的功能，由服务器分布式对数据进行处理
- Reduce：提供汇总功能，将分布式的处理结果汇总统计

比如3台服务器进行Map，一台服务器进行Reduce

## YARN概叙

MapReduce是基于YARN运行的

YARN管控整个集群的资源进行调度，应用程序运行时，就是在YARN的监管下运行的



向YARN申请使用资源，YARN分配好资源后运行，空闲资源可供其他程序使用

## YARN核心架构

主从架构

### 主角色：ResourceManager

整个集群的资源调度者，负责协调调度各个程序所需的资源

### 从角色：NodeManager

单个服务器的资源调度者，负责调度单个服务器上的资源提供给应用程序使用

### 容器

NodeManager预先占用一部分资源，这部分资源叫做容器，然后将这部分资源提供给程序使用

程序运行在容器内无法突破容器的资源限制

## YARN辅助架构

### 代理服务器：ProxyServer

(web application proxy)web应用程序代理

为了减少通过YARN进行基于网络的攻击的可能性。

因为YARN在运行时会提供一个web ui站点，可供用户在浏览器内查看YARN的运行信息

对外提供web站点会由安全问题，使用代理服务器保证安全

如警告用户正在访问一个不受信任的站点，玻璃用户访问的cookie等

### 历史服务器：JobHistoryServer

应用程序历史信息记录服务

记录历史运行的程序的信息以及产生的日志，并提供web ui站点给用户使用浏览器查看

收集每个容器内的日志到HDFS，通过浏览器统一查看

## YARN启停管理

### 一键启停YARN集群

一键启动：`start-yarn.sh`

原理：

1. 从yarn-site.xml中读取配置，确定ResourceManager所在机器，并启动它
2. 读取workers文件，确定机器，启动全部的NodeManager
3. 在当前机器启动ProxyServer

一键停止：`stop-yarn.sh`

原理：

1. 从yarn-site.xml中读取配置，确定ResourceManager所在机器，并关闭它
2. 读取workers文件，确定机器，关闭全部的NodeManager
3. 在当前机器关闭ProxyServer

### 在当前机器单独启停

`yarn --daemon start/stop resourcemanager/nodemanager/proxyserver`

### 历史服务器启停

`mapred --daemon start/stop historyserver`

## 查看YARN运行网页

端口8088

## 运行MapReduce程序代码

`hadoop jar 程序文件路径 java类名 [程序参数]`

## Hive的功能

是一款分布式SQL计算的工具，将SQL语句翻译成MapReduce程序运行

## Hive架构

### 元数据管理

- 数据位置
- 数据结构
- 数据描述

通常存储在关系型数据库中，Hive中的元数据包括表的名字，列，分区和属性，数据所在目录等

Hive的数据存储在HDFS的/user/hive/warehouse中



### SQL解析器

- SQL分析
- SQL到MapReduce程序的转换
- 提交MapReduce程序运行并收集执行结果

Hive的Driver驱动程序，包括语法解释器，计划编译器，优化器，执行器。

### 用户接口

包括CLI,JDBC/ODBC,WebGUI等接口与Hive进行交互

## Hive的部署

Hive时单机工具，只需要部署在一台服务器

## Hive语句

执行`bin/hive`进入Hive shell环境

### 创建数据库：

```
create database if not exits 数据库名 location '/myhive2';
```

location用于指定hdfs存储位置

### 进入数据库：

```
use 数据库名;
```

### 查看数据库详细信息：

```
desc database 数据库名;
```

### 删除数据库

```
drop database 数据库名;//只能删除空数据库
drop database 数据库名 cascade;//强制删除数据库和里面的表
```



### 创建表：

```
create table test(id int,name string,gender string);
```

### 插入数据：

```
insert into test values(1,'xxx','male'),(...);
```

### 查询数据：

```
select gender,count(*) as cnt from test group by gender;
```

