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

![image](https://github.com/Heavypea/Timberland-s-work-related-knowledge/assets/90777267/1135d5b7-52b8-4573-8642-e5dce9b7150a)

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

```
git commit -am "Remove example.txt from the repository"
```

- 加入-a可以完成暂存和提交两步操作

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

![image](https://github.com/Heavypea/Timberland-s-work-related-knowledge/assets/90777267/40a77a45-c510-4ee3-9b74-f58f57fb99c7)

将文件名或  文件夹名加上/ 或通配符写入`.gitignore`文件中就可以将其忽略(可以使用vim)

> [!CAUTION]
>
> w(ﾟДﾟ)w
>
> 无法作用于已经添加到仓库中的文件和空的文件夹(Git不会跟踪控制空的文件夹)

#### `.gitignore`文件匹配规则

![image](https://github.com/Heavypea/Timberland-s-work-related-knowledge/assets/90777267/f448eca4-728c-42db-bd2b-de576272c67f)

![image](https://github.com/Heavypea/Timberland-s-work-related-knowledge/assets/90777267/6852607e-8ae5-48d6-be03-b84d16f619bf)

![image](https://github.com/Heavypea/Timberland-s-work-related-knowledge/assets/90777267/6bea9d93-c791-46de-82ce-1784d0557561)

## Git远程仓库

### GitHub的使用与远程仓库操作

使用`git clone xxx.git`将远程仓库部署到本地

#### 配置SSH密钥

使用`ssh-keygen -t rsa -b 4096`生成SSH密钥(-t代表协议类型,-b代表大小)

> [!TIP]
>
> 若第一次生成,直接回车就行.若不是第一次,就要输入新的文件名,否则第一次生成的SSH密钥会被不可逆的覆盖

![image](https://github.com/Heavypea/Timberland-s-work-related-knowledge/assets/90777267/712eff1d-a9dd-4324-a652-797071a00345)

![image](https://github.com/Heavypea/Timberland-s-work-related-knowledge/assets/90777267/e30ce24f-c3ae-4463-b7a2-9ab99b9d4ba1)
这就是生成的密钥,第一个文件没有拓展名,是私钥文件,不可泄漏

第二个文件有pub后缀,是公钥文件,打开文件(记事本)并复制其中内容到要使用的地方



不是第一次生成,还需要创建config文件并添加语句,让访问某个网站指定使用SSH下的某个密钥

![image](https://github.com/Heavypea/Timberland-s-work-related-knowledge/assets/90777267/59973b73-5e2a-4a82-892d-e9b463c5829e)

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

#### 本地复制的图片无法push到远程仓库中

需要在远程仓库中加入图片，真麻烦。

### 使用vscode

在命令行中进入目录，并输入`code .`就可以直接用vscode打开当前目录。但这个要进行配置，打开命令面板后输入path。然后点击图中命令即可

![image](https://github.com/Heavypea/Timberland-s-work-related-knowledge/assets/90777267/508f8a4b-b32f-4a9e-91b5-b8c0de9c36e2)

然后再源代码管理那里使用图形化操作，可参考【【GeekHour】一小时Git教程】 https://www.bilibili.com/video/BV1HM411377j/?p=15&share_source=copy_web&vd_source=ea2eaf8cb220ddebb93297f285dc38f6




## 分支

仓库中的文件名和提交记录要以最简单的方式命名

使用分支名加冒号加序号的方式编写提交记录

### 分支创建

使用`git branch`查看分支情况，命令行中前面带有*就是目前所处在的分支

使用`git branch xxx`创建一个名为xxx的分支

> [!CAUTION]
>
> ╰(￣ω￣ｏ)
> 使用`git branch xxx`只是创建分支，并没有切换到xxx分支哦





使用`git checkout xxx`切换到xxx分支。checkout还可以用来恢复文件或者目录到之前的某一个状态，若文件名/目录名和分支名相同时，默认执行切换xxx分支的效果

- `git checkout xxx` 用于切换到已存在的分支。如果分支已经存在，你可以使用这个命令来切换到那个分支。
- `git checkout -b xxx` 不仅会创建一个新的分支，还会立即切换到这个新创建的分支。这个命令相当于先执行 `git branch xxx` 创建新分支，然后执行 `git checkout xxx` 切换到新分支。





所以新的语句`git switch xxx`专门用来切换分支

**分支就是用来在不影响main分支和其他人的情况下进行开发和测试**

### 分支合并和删除

不同分支上的修改需要进行合并，使用`git merge xxx`将xxx分支合并到当前分支中。合并后会自动进行提交

> [!WARNING]
>
> o(><；)oo
> 千万别merge错了，xxx是要被合并的，不能写main！要先切换到main分支再合并

在命令行中使用`git log --graph --oneline --decorate --all`来查看分支图



合并后，被合并的分支还是存在的，可以使用`git branch -d xxx`来删除分支，-d表示删除已经完成合并的分支，未合并会报错，比较安全。要删除未合并的要改成-D

### 分支冲突解决

若两个分支修改的部分没有冲突的话，合并就没问题。若有冲突就需要手动解决冲突

冲突产生时，使用`git status`可以查看冲突文件的列表，也可以使用`git diff`查看冲突的具体内容

然后人工修改解决冲突，提交后会自动合并



若想要不合并这个冲突，可以使用`git merge --abort`终止合并



### 回退和rebase

`git rebase xxx`

在Git中，每个分支都有一个HEAD指针，指向当前分支的最新提交记录。在执行rebase操作时，Git会先找到当前分支和目标分支的共同祖先，再把**当前分支上**上从共同祖先到最新提交记录的所有提交全部都移动到xxx分支的最新提交后面

![image](https://github.com/Heavypea/Timberland-s-work-related-knowledge/assets/90777267/1b057abd-4c11-4ae5-a13c-af2a20de71a0)

如果在dev分支上执行`git rebase main`，就把dev分支和main分支共同祖先后的dev提交记录，移动到main分支的最新提交记录后面。反之同理。

> [!TIP]
>
> o(*￣▽￣*)o
> 可以使用alias命令给很长的命令起个短的别名
> 如`alias graph="git log --graph --oneline --decorate --all"`
>
> 然后直接输入graph就能执行这个长命令了

### merge和rebase的使用区别

merge的优点是不会破坏原分支的提交记录，方便回溯和查看

缺点是会产生额外的提交记录和两条分支线的合并，会使得提交记录变得复杂

大多数情况使用merge



rebase的有点是不需要新增额外的提交记录到目标分支，可以形成线性的提交历史记录，非常直观和干净

缺点是会改变提交历史，改变当前分支branchout的节点，要避免在一个共享分支上进行rebase操作



### Git分支管理和工作流模型

GitFlow模型

GitHub Flow模型

【【GeekHour】一小时Git教程】 https://www.bilibili.com/video/BV1HM411377j/?p=19&share_source=copy_web&vd_source=ea2eaf8cb220ddebb93297f285dc38f6

看视频更有效
