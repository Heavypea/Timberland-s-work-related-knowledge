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

## Hive数据库操作语句

执行`bin/hive`进入Hive shell环境

### 创建数据库：

```hive
create database if not exits 数据库名 location '/myhive2';
```

location用于指定hdfs存储位置

### 选择数据库：

`use 数据库名;`

### 查看数据库详细信息：

`desc database 数据库名;`

### 删除数据库

`drop database 数据库名;`只能删除空数据库
`drop database 数据库名 cascade;`强制删除数据库和里面的表



### 创建表（简单）：

```hive
create table test(
	id int,
	name string,
	gender string
);
```

### 插入数据：

```hive
insert into test values(1,'xxx','male'),(...);
```

### 查询数据：

```hive
select gender,count(*) as cnt from test group by gender;
```

## Hive数据类型

- boolean
- tinyint：1字节的有符号整数，-128~127
- smallint：2字节的有符号整数，-32768~32767
- **int**：4字节的有符号整数
- bigint：8字节的有符号整数
- float：4字节单精度浮点数
- double：8字节双精度浮点数
- deicimal：任意精度的带符号小数
- **string**：字符串，不定长
- **varchar**：同string
- char：字符串，定长
- binary：字节数组
- **timestamp**：时间戳，毫秒精确度
- **date**：日期

复合类型

- array：有序的同类型的集合
- map：key-value。key必须为普通类型，value可以是任何类型
- struct：字段类型，类型可以不同
- union：在有限取值范围内的一个值

## Hive表操作语句

数据表名可以是`数据库名.表名`，指定在那个数据库中对表进行操作

### 创建表（完整）：

```hive
create [external] table [if not exits] 数据表名 [(col_name data_type [comment col_comment],...)] [comment tanle_comment] [partitioned by (col_name data_type [comment col_comment],...)] [clustered by (col_name,col_name,...) [sorted by (col_name [asc/desc],...)] into num_buckets buckets] [row format row_format] [stored as file_format] [location  hdfs_path];
```

### 删除表：

`drop table 数据表名;`

### 清空表：

`truncate table 数据表名;`

只能清空内部表

### 表重命名：

`alter table 旧数据表名 rename to 新数据表名;`

### 修改表属性值：

`alter table 数据表名 set tblproperties (表属性=表属性值);`

比如：

```hive
alter table table_name set tblproperties ('comment'=new_comment);//修改表注释

alter table 表名 set tblproperties ('EXTERNAL'='TRUE');
```

### 添加表的分区：

`alter table 数据表名 add partition (分区列=xxx);`

### 修改表的分区：

`alter table 数据表名 partition (分区列=xxx) rename to partition (列=yyy);`

实际上是修改元数据记录，HDFS的实体文件夹实不会改名的，但元数据记录中是改名了

### 删除表的分区：

`alter table 数据表名 drop partition (分区列=xxx);`

实际上知识删除元数据记录，数据本身还在

### 添加列：

`alter table 数据表名 add columns (列 列数据类型,...);`

### 修改列名：

`alter table 数据表名 change 旧列名 新列名 旧列数据类型;`

## Hive中表分类

### 内部表

未被关键字external关键字修饰的表，即普通表，也叫管理表。

内部表的数据存储位置由hive.metastore.warehouse.dir参数决定

删除内部表会直接删除元数据（表的信息）和存储数据，因此内部表不适合和其他工具共享数据

### 外部表

`create external table 数据表名 ... location ...;`

也叫关联表，表数据可以在任何位置，通过关键字location指定。

这个表在理念上并不是Hive内部管理的，可以随意临时连接到外部数据上，删除外部表时，仅仅删除元数据，不删除存储数据

- 可以先有表，然后把数据移动到表指定的location中
  1. 先检查：hadoop fs -ls /tmp，确认不存在/tmp/test_ext1目录
  2. 创建外部表并将location指定为/tmp/test_ext1
  3. select * from test_ext1，无数据
  4. 上传数据 hadoop fs -put test_external.txt /tmp/test_ext1/
  5. select * from test_ext1，可以看见数据
- 也可以先有数据，然后创建表通过location指向数据
  1. 先创建/tmp/test_ext2目录，hadoop fs -mkdir /tmp/test_ext2
  2. 上传数据hadoop fs -put test_external.txt /tmp/test_ext2/ 
  3. 创建外部库并将location指定为/tmp/test_ext2
  4. select * from test_ext2

### 分区表

将大文件分成一个个小文件。分区是将表拆分到不同的**子文件夹**中进行存储

`create table 表名(...) partitioned by (分区列 列类型,...) row format delimited fields terminated by '\t';`

partitioned by指的是通过哪些列进行分区

比如：

```hive
create table score(s_id string,c_id string s_score int) partitioned by (year string,month string,day string) row format delimited fields terminated by '\t';
```



加载数据到分区表中

`LOAD DATA [LOCAL] INPATH 'filepath' [OVERWRITE] INTO TABLE score2 partition(year='2020',month='06',day='01');`

### 分桶表

将表拆分到**固定数量**的不同**文件**中进行存储

开启分桶的自动优化（自动匹配reduce task数量和桶数量一致）：

`set hive.enforce.bucketing=true;`

创建分桶表：

 `create table hive.course (c_id string,c_name string,t_id string) clustered by(c_id) into 3 buckets row format delimited fields terminated by '\t';`

clustered by表明按一个列进行分桶



加载数据到分桶表中：

1. 创建一个临时表，通过load的方法加载数据进入这个表
2. 然后通过insert select从临时表向桶表插入数据

因为需要将文件划分为桶的数量的份数，划分是基于分桶列的值进行hash取模来决定,基于结果放入对应序号的桶文件中。因为load data不会触发MapReduce，也就无法执行hash算法，所以无法使用load方法

比如：

```hive
create table course_common(c_id string,c_name string,t_id string) row format delimited fields terminated by '\t';

load data local inpath '/.../course.txt' into table course_common;

insert overwrite table course select * from course_common cluster by(c_id);
```



基于分桶列，分桶表可以过滤出单个值，可以优化双表join，可以自动分组

## 数据分隔符

Hive中的数据分隔符是`\001`，是ASCII码，键盘打不出来

有些文本编辑器显示是`SOH`

也可以在创建表的时候自己指定分隔符：

```hive
create table if not exists stu2(id int,name string) row format delimited fields terminated by '\t';
```

表示以\t分隔

`row format delimited fields terminated by '\t'`

## Hive查看表信息

`desc formatted 表名;`

## 内外表转换

### 内部表转外部表:

 `alter table 表名 set tblproperties('EXTERNAL'='TRUE');`

### 外部表转内部表:

 `alter table 表名 set tblproperties('EXTERNAL'='FALSE');`

> [!CAUTION]
>
> (●ˇ∀ˇ●)
>
> 'EXTERNAL'='FALSE'和'EXTERNAL'='TRUE'是固定写法，区分大小写

## Hive数据导入

### LOAD

使用LOAD语法，从外部将数据加载到Hive内，不会走MapReduce，小文件较快

`LOAD DATA [LOCAL] INPATH 'filepath' [OVERWRITE] INTO TABLE 表名;`

local表示数据是否在本地

- 当数据不在HDFS时，使用local，需使用file://协议头只当路径
- 当数据在HDFS的时候，不使用local，可以使用HDFS://协议头指定路径

filepath是数据路径

OVERWRITE是覆盖已存在数据，不写就不覆盖

> [!CAUTION]
>
> ˋ( ° ▽、° ) 
>
> 基于HDFS进行load加载数据，源数据文件会消失，本质是被移动到表所在的目录中

### INSERT SELECT

使用SQL语句，从其他表中加载数据

`INSERT [OVERWRITE/INTO] TABLE 表名 [PARTITION (partcol1=val1,partcol2=val2 ...) [IF NOT EXISTS]] select statement1 FROM from_statement;`

被SELECT查询的表可以是内部表或外部表

比如：

``` hive
INSERT INTO TABLE tbl1 SELECT * FROM tbl2;
INSERT OVERWRITE TABLE tbl1 SELECT * FROM tbl2;
```

## Hive数据导出

### INSERT

`insert overwrite [local] directory 'path' select_statement from from_statement;`

local将查询的结果导出到本地，不写local就是导出到HDFS上

比如：

```hive
insert overwrite local directory '/home/hadoop/ex1' select * from test_load;
```

可以加上`row format delimited fields terminated by '\t'`指定分隔符，加在'path'之后

### Hive shell

`bin/hive -e "select * from myhive.test_load;" > /home/hadoop/ex4/ex4.txt`

`bin/hive -f export.sql > /home/hadoop/ex4/ex4.txt`

-e是指定sql，直接执行sql语句，将结果通过Linux的重定向符号写入指定文件中

-f是指定脚本

\>是重定向符



export.sql是一个sql语句文件，里面存了select * from myhive.test_load;

## array类型的操作

一个列记录多个元素，用[]可以取任意位置的元素，所引从0开始

建表：

```hive
create table myhive.test_array(name string,work_locations array<string>) row format delimited fields terminated by '\t' collection items terminated by ',';
```

collection items terminated by ','表示array中的元素用逗号隔开



取：

```hive
select name,work_locations[0] from myhive.test_array;
```

查询array类型的元素个数：

size(数组)

```hive
select name,size(work_locations) from myhive.test_array;
```

通过array某个元素过滤：

array_contains(数组,数据)可以查看指定数据是否在数组中存在

```hive
select * from myhive.test_array where array_contains(work_locations,'xxx');
```

