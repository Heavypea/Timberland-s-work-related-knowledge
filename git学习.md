## Git配置文件名与邮箱

用于查看是谁提交的。



--global 全局配置，对所有仓库生效。它会影响用户主目录下的 `.gitconfig` 文件

--local 局部配置，对当前仓库生效，不显示指明的话默认就是local。它会影响当前仓库下的 `.git/config` 文件

--system 系统配置，对系统中所有用户和他们的仓库的配置（一般不使用它）。它会影响Git安装目录下的 `/etc/gitconfig` 文件

配置的优先级是：本地级别 (`--local`) > 全局级别 (`--global`) > 系统级别 (`--system`)

### Git配置文件名

 `git config --global user.name Timberland`

**注意，当用户名有空格时，需要双引号括起来 **

`git config --global user.name “Timber land”`



### Git配置邮箱

`git config --global user.email xxx@xx.com`

**注意，当邮箱有空格时，需要双引号括起来 **（一般没有）

### 保存你的输入

`git config --global credential.helper store`

配置好文件名和邮箱后输入这行命令用于保存，以后不需要每次输入了。

### 查看Git配置信息

`git config --global --list`



## Git创建仓库管理本地文件

### 创建新仓库

#### 一、git init

`git init`

将当前目录变为git可以管理的仓库

git init后可以指定目录的名称，若指定了，就在当前目录下面创建一个新的目录作为git仓库

`git init my_test`

#### 二、git clone

`git clone https://github.com/xxx`

将一个在远程服务器中的仓库复制到当前目录中



eg：

```
mkdir test
cd test
git init my_test
```

### Git本地区域

#### 一、工作区

也叫本地工作目录，就是我们自己电脑上的目录，在资源管理器中看到的文件夹就是它。所在的位置是`.git`所在的目录

当修改完工作区的文件后，使用`git add xxx.xx`将修改后的文件添加到暂存区

使用`ls -a`（Linux命令）查看工作区内容

#### 二、暂存区

也叫索引，用于保存即将提交到Git仓库的修改内容。所在的位置是`.git/index`

> [!IMPORTANT]
>
> 很重要的区域！ヾ(≧▽≦*)o   

使用`git ls-files`单独查看暂存区里的内容

使用`git commit`将暂存区的修改一并提交到仓库中。

#### 三、本地仓库

通过`git init`创建的本地仓库，包含完整的项目历史和元数据，是Git存储代码和版本信息的主要位置。所在的位置是`.git/objects`

### Git文件状态

#### 一、未跟踪

就是新创建的，还没有被Git管理起来的文件

#### 二、未修改

已经被Git管理起来，但文件内容未发生修改的文件

#### 三、已修改

已经被Git管理起来，修改了文件内容但未添加到暂存区里面的文件

#### 四、已暂存

修改后并添加到暂存区里面的文件

image-20240330152726195

### Git添加，提交，删除文件

`git status`用于查看仓库状态，如当前仓库在哪个分支，有哪些文件以及对应的状态



`git add xxx.xx`将文件添加到暂存区。

可以使用通配符。如`git add *.txt`就是把所有的以txt结尾的文件都加入到暂存区里。

可以以文件夹名字作为参数。如`git add .`就是把当前文件夹下所有文件添加到暂存区里。



`git rm xxx.xx`将文件从**工作区和暂存区删除**。

`git rm --cached xxx.xx`将文件从**暂存区删除，但保留在工作区**。

`git rm -r xx`从**工作区和暂存区**删除xx目录

`git rm -r *`从**工作区和暂存区**删除当前目录下的所有子目录和文件

> [!CAUTION]
>
> （；´д｀）ゞ
>
> - 在执行 `git rm` 命令后，文件或目录将从暂存区中移除。如果你还没有提交这个更改，那么你可以使用 `git reset` 命令来撤销这个操作。
> - 一旦你提交了 `git rm` 的更改，那么该文件或目录将从 Git 仓库的历史中删除。这意味着，除非你使用诸如 `git reflog` 或其他方法，否则你将无法恢复这个文件或目录。
> - 在执行 `git rm` 命令前，确保你真的想要删除这个文件或目录，并且已经备份了任何重要的内容。
> - 记得删除后`git commit`提交修改🌹



`git commit`只会提交在暂存区中的文件

```
git commit -m "Remove example.txt from the repository"
```

- `git commit`：这是 Git 的命令，用于将暂存区（index）中的变更提交到本地仓库中。
- `-m`：这是一个选项，用于后面紧接着提供一个字符串作为提交信息。如果不使用 `-m` 选项，Git 会打开一个文本编辑器，让你输入提交信息。
- `"Remove example.txt from the repository"`：这是你为这次提交提供的具体信息。这条信息会保存在提交历史中，用于描述这次提交做了什么。



`git log`用于查看提交记录，如id，作者，邮箱，注释

`git log --online`查看简洁版

### Git回退版本

`git reset`用于回退版本，有三种模式

#### 一、soft

`git reset --soft xxx`表示回退到某一个版本，并保留工作区和暂存区的所有修改内容

#### 二、hard

`git reset --hard xxx`表示回退到某一个版本，并丢弃工作区和暂存区的所有修改内容

> [!TIP]
>
> 谨慎使用。( •̀ ω •́ )✧。
>
> 误操作后可以用`git reflog`查看操作的历史记录，找到误操作之前的版本号，使用`git reset`回退就行



#### 三、mixed

`git reset --mixed xxx`表示回退到某一个版本，并**保留工作区，丢弃暂存区**的所有修改内容（默认是mixed）

### Git查看差异

`git diff`用于查看文件在工作区，暂存区，仓库之间的差异。或者文件不同版本的差异，或者文件在两个分支之间的差异



`git diff`后不加参数，默认比较的是文件在工作区，暂存区之间的差异内容，会显示发生更改的文件以及更改的详细信息。

`git diff HEAD`加上HEAD，用于比较工作区与仓库之间的差异

`git diff --cached`加上--cached，用于比较暂存区与仓库之间的差异

`git diff xxx yyy`加上两个版本提交ID/分支名，比较两个版本/分支之间的差异内容

> [!IMPORTANT]
>
> (*￣3￣)╭
>
> - HEAD参数表示分支的最新提交节点。
>
>   ​	比如`git diff xxx HEAD`，使用版本提交ID和HEAD进行比较
>
> - HEAD~/HEAD^表示上一个提交节点。
>
>   ​	比如`git diff HEAD~ HEAD`，上一个版本和当前版本进行比较
>
> - HEAD~2/3/4……表示HEAD之前的第二/三/……个版本

`git diff HEAD~ HEAD xxx.xxx`加上了文件名，就只会查看xxx文件的差异内容

### `.gitignore`文件

可以让我们忽略掉一些不应该被加入到仓库中的文件

image-20240330165052406

将文件名或  文件夹名加上/ 或通配符写入`.gitignore`文件中就可以将其忽略(可以使用vim)

> [!CAUTION]
>
> w(ﾟДﾟ)w
>
> 无法作用于已经添加到仓库中的文件和空的文件夹(Git不会跟踪控制空的文件夹)

#### `.gitignore`文件匹配规则

image-20240330170048526

image-20240330170351812

image-20240330170237708
## Git远程仓库

### GitHub的使用与远程仓库操作

使用`git clone xxx.git`将远程仓库部署到本地

#### 配置SSH密钥

使用`ssh-keygen -t rsa -b 4096`生成SSH密钥(-t代表协议类型,-b代表大小)

> [!TIP]
>
> 若第一次生成,直接回车就行.若不是第一次,就要输入新的文件名,否则第一次生成的SSH密钥会被不可逆的覆盖

image-20240330181754497

image-20240330182043852
这就是生成的密钥,第一个文件没有拓展名,是私钥文件,不可泄漏

第二个文件有pub后缀,是公钥文件,打开文件(记事本)并复制其中内容到要使用的地方



不是第一次生成,还需要创建config文件并添加语句,让访问某个网站指定使用SSH下的某个密钥

image-20240330182508251

> [!NOTE]
>
> o(￣┰￣*)ゞ
>
> 注意本地仓库和远程仓库是独立的两个仓库，要同步需要使用pull和push，本地commit提交之后只会修改本地的

### 关联本地仓库和远程仓库

若本地已经有了仓库，要关联到远程仓库

先使用在本地仓库中使用`git remote add origin xxx.git`，其中origin是创建远程仓库的一个别名，一般都是这个

可以使用`git remote -v`查看我们当前本地仓库对应的远程仓库的别名和地址

再使用`git branch -M main`指定分支的名称为main

最后使用`git push -u origin main:main`将本地的main分支和远程origin仓库的main分支关联起来，因为名称相同，所以可以省略掉`:main`。

> [!IMPORTANT]
>
> (　o=^•ェ•)o　┏━┓
>
> `origin`是远程仓库的默认名称，第一个`main`代表本地分支名，第二个`main`代表远程分支名。`-u`参数会将本地分支设置为追踪远程分支，这意味着在未来的`git pull`或`git push`操作中，你可以不必指定远程仓库名或分支名



当远程仓库更新后，使用`git pull origin main`拉取更新后的文件到本地仓库。这里的`origin`是远程仓库的默认名称，第一个`main`代表远程分支名。在`git pull`命令中，通常只需要指定远程分支名，Git会自动将其合并到当前检出的本地分支。

> [!NOTE]
>
> (ノ｀Д)ノ
>
> 在执行git pull之后，Git会自动执行一次合并操作，若远程仓库重点修改内容和本地仓库中的修改内容没有冲突时，合并成功。反之失败
>
> 使用git fetch就不会自动执行合并，需要我们自己手动合并



