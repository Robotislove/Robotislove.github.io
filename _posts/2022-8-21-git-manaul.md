---
layout: post
title: git使用总结笔记
date: 2022-08-21
author: lau
tags: [Git, Blog]
comments: true
toc: false
pinned: false
---

git作为版本控制的常用软件，这里记录它的常用命令。

<!-- more -->
## 1. 概述
### 1.1 Git版本控制下的三种工程区域 & 文件状态
![](http://assets.processon.com/chart_image/6302dc735653bb478a4ee0a9.png)

1. 版本库(Repository)

在工作区中有一个隐藏目录.git，这个文件夹就是Git的版本库，里面存放了Git用来管理该工程的所有版本数据，也可以叫本地仓库。

2. 工作区（Working Directory）

日常工作的代码文件或者文档所在的文件夹。

3. 暂存区（stage）

一般存放在工程根目录 .git/index文件中，所以我们也可以把暂存区叫作索引（index）。

Git版本控制下的文件状态只有三种：
- 已提交（committed） :该文件已经被安全地保存在本地数据库中了；
- 已修改（modified） :修改了某个文件，但还没有提交保存；
- 已暂存（staged）:把已修改的文件放在下次提交时要保存的清单中。
### 1.2 Git 常用命令
**工程准备**

- 工程克隆—— git clone

**查看工作区**

- 查看工作区的修改内容—— git diff
- 查看工作区文件状态—— git status

**文件修改后提交推送**

- 新增/删除/移动文件到暂存区—— git add/ git rm/ git mv
- 提交更改的文件—— git commit
- 推送远端仓库—— git push

**查看日志**
- 查看当前分支上的提交日志—— git log

**分支管理**

- 列出本地分支—— git branch
- 新建分支—— git branch / git checkout –b
- 删除分支—— git branch –d
- 切换分支—— git checkout
- 更新分支—— git pull
- 合并分支—— git merge 

**撤销操作**

- 强制回退到历史节点—— git reset
- 回退本地所有修改而未提交的—— git checkout 

**分支合并**
- 合并目标分支内容到当前分支—— git merge/git rebase
#### 工程准备
git init用于在本地目录下新建git项目仓库。

执行git init后，当前目录下自动生成一个名为.git的目录，这代表当前项目所在目录已纳入Git管理。.git目录下存放着本项目的Git版本库，在此强烈不建议初学者改动.git目录下的文件内容。
Git仓库下的.git目录默认是不可见的，有一定程度上是出于防止用户误操作考虑。

#### git clone用于克隆远端工程到本地磁盘。

如果想从远端服务器获取某个工程，那么：

1.确定自己Git账号拥有访问、下载该工程的权限

2.获取该工程的Git仓库URL

3.本地命令行执行 git clone [URL]或 git lfs clone [URL]

注：如果你所在的项目git服务器已支持git-lfs，对二进制文件进行了区别管理，那么克隆工程的时候务必使用git lfs clone。否则克隆操作无法下载到工程中的二进制文件，工程内容不完整。

#### 新增/删除/移动文件到暂存区 

**在提交你修改的文件之前，需要git add把文件添加到暂存区。**

如果该文件是新创建，尚未被git跟踪的，需要先执行 git add 将该文件添加到暂存区，再执行提交。

如果文件已经被git追踪，即曾经提交过的 。在早期版本的git中，需要git add再提交；在较新版本的git中，不需要git add即可提交。

#### 新增/删除/移动文件到暂存区 

git rm 将指定文件彻底从当前分支的缓存区删除，因此它从当前分支的下一个提交快照中被删除。

如果一个文件被git rm后进行了提交，那么它将脱离git跟踪，这个文件在之后的节点中不再受git工程的管理。执行git rm后，该文件会在缓存区消失。

你也可以直接从硬盘上删除文件，然后对该文件执行 git commit，git会自动将删除的文件从索引中移除，效果一样。

git mv 命令用于移动文件，也可以用于重命名文件。

例1：需要将文件codehunter_nginx.conf从当前目录移动到config目录下，可执行：
```shell
git mv codehunter_nginx.conf config
```
#### 查看工作区

git diff用于比较项目中任意两个版本（分支）的差异，也可以用来比较当前的索引和上次提交间的差异。当然它还有其他用途，在后文会进一步描述。

**git status 命令用于显示工作目录和暂存区的状态。**

使用此命令能看到修改的git文件是否已被暂存, 新增的文件是否纳入了git版本库的管理。

下例中的信息表明：modeules/__init__.py已被修改并暂存，LICENSE已被修改但未暂存，README.md已被删除但未暂存，extend.txt已被新建但未跟踪。注意，请保证能理解git status回显的每一行文字含义。

#### 提交更改的文件
**git commit 主要是将暂存区里的文件改动提交到本地的版本库。**

在此强调，提交这个动作是本地动作，是往本地的版本库中记录改动，不影响远端服务器。git commit一般需要附带提交描述信息，所以常见用法是：`git commit file_name –m “commit message”`。

提交成功后，git日志可查到此次提交的id和提交描述信息。如果要一次性提交所有在暂存区改动的文件到版本库，可以执行:
```shell
git commit –am “commit message”
```

#### 查看日志
**git log用于查看提交历史。**
默认加其他参数的话，git log 会按提交时间由近到远列出所有的历史提交日志。每个日志基本包含提交节点、作者信息、提交时间、提交说明等。

常用的日志命令格式：git log

git log配合不同参数具有相当灵活强大的展示功能，常见的如—name-status/-p/--pretty/--graph等等，有兴趣的同学课后自行了解。
#### 推送至远端仓库

在使用git commit命令将自己的修改从暂存区提交到本地版本库后，可以使用git push将本地版本库的分支推送到远程服务器上对应的分支。成功推动远端仓库后，其他开发人员可以获取到你新提交的内容。常用的推送命令格式： `git push origin branch_name` 。branch_name决定了你的本地分支推送成功后，在远端服务器上的分支名，其他人据此可以获取该分支上的改动内容。
你的本地分支名可以与推送到远端的分支名不同 ： `git push origin branch_name:new_branch_name`。

#### 分支管理

git branch返回了当前本地工程所有的分支名称，其中master分支前面的“\*”表示——当前工作区所在的分支是master。

如果想查看远端服务器上拥有哪些分支，那么执行`git branch –r`即可，返回的分支名带origin前缀，表示在远端；

如果想查看远端服务器和本地工程所有的分支，那么执行`git branch –a`即可。

`git branch`和`git checkout –b` 的异同：

**相同点：**

`git branch`和`git checkout –b`都可以用于新建分支（默认基于当前分支节点创建）。

**区别点：**

`git branch`新建分支后并不会切换到新分支；

`git checkout –b`新建分支后会自动切换到新分支。

常用的新建分支命令格式：`git branch new_branch_name / git checkout –b branch_name`

`git branch –d`和`git branch –D`都可以用来删除本地分支，后者大写表示强制删除。

有时候当事分支上包含了未合并的改动，或者当事分支是当前所在分支，则-d无法删除，需要使用强制删除来达到目的。

常用的删除分支命令格式：`git branch –d branch_name/git branch –D branch_name`。

删除服务器上的远程分支可以使用`git branch -d -r branch_name`，其中branch_name为本地分支名。

删除后，还要推送到服务器上才行，即`git push origin : branch_name`。

`git checkout` 命令除了创建分支，还用来切换分支，当然比较官方的叫法是“检出”。

有时候，当前分支工作区存在修改而未提交的文件，与目的分支上的内容冲突，会导致checkout切换失败，这时候，可以使用git checkout –f进行强制切换。

常用的切换分支命令格式：`git checkout branch_name`。

`git checkout`对象可以是分支，也可以是某个提交节点或者节点下的某个文件。建议课后自行按需了解。

git pull的作用是，从远端服务器中获取某个分支的更新，再与本地指定的分支进行自动合并。

常用的更新分支命令格式：`git pull origin remote_branch:local_branch`

如果远程指定的分支与本地指定的分支相同，则可直接执行`git pull origin remote_branch`

git fetch的作用是，从远端服务器中获取某个分支的更新到本地仓库。

注意，与git pull不同，git fetch在获取到更新后，并不会进行合并（即下页中的git merge）操作，这样能留给用户一个操作空间，确认git fetch内容符合预期后，再决定是否手动合并节点。
常用的获取远端分支更新命令格式：`git fetch origin remote_branch:local_branch`

如果远程指定的分支与本地指定的分支相同，则可直接执行`git fetch origin remote_branch`。

**git merge命令是用于从指定的分支（节点）合并到当前分支的操作**。

git会将指定的分支与当前分支进行比较，找出二者最近的一个共同节点base，之后将指定分支在base之后分离的节点合并到当前分支上。分支合并，实际上是分支间差异提交节点的合并。
常用的切换分支命令格式：`git merge branch_name`。

git rebase用于合并目标分支内容到当前分支。
git rebase这条命令用于分支合并，git merge也是用于分支合并。如果你要将其他分支的提交节点合并到当前分支，那么git rebase和git merge都可以达到目的。

常用的合并命令格式：`git rebase branch_name`。

git rebase、git merge背后的实现机制和对合并后节点造成的影响有很大差异，有各自的风险存在，暂不在本课中展开。

#### 撤销操作

git reset通常用于撤销当前工作区中的某些git add/commit操作，可将工作区内容回退到历史提交节点。

常用的工作区回退命令格式：`git reset commit_id`。

**注**：git reset –-mixed/hard/soft的三种参数模式涉及概念较多，不在此区分讲解，请课后按实际需要去了解。

git checkout .用于回退本地所有修改而未提交的文件内容。

git checkout . 这是条有风险的命令，因为它会取消本地工作区的修改(相对于暂存区)，用暂存区的所有文件直接覆盖本地文件，达到回退内容的目的。但它不给用户任何确认机会，所以谨慎使用。

常用的回退命令格式：git checkout .

如果仅仅想回退某个文件的未提交改动，可以使用git checkout –filename来达到目的；如果想将工具区回退（检出）到某个提交版本，可以使用git checkout commit_id。


## 实例：本地分支与远程分支关联、推送内容

创建git仓库可以在远端创建一个仓库，然后check到本地，在本地的文件里创建工程文件，然后提交；也可以将本地现有的工程和远端的空仓库关联，本地创建了一个工程 iOSDemo
运行没有错误，就可以提交到远端了。如果要提供PR到开源库，可以先fork仓库，再本地修改后PR操作。

一般情况下，远端仓库创建成功之后会有以下提示

```shell
#Command line instructions

#Git global setup
git config --global user.name "wangjiangwei336"
git config --global user.email "222@smail.nju.edu.cn"

#Create a new repository
git clone http://gitlab.pab.com.cn/CARF/DUN-CLDS/dun-clds-open-front/ios/CarLoanAppR11.git
cd CarLoanAppR11
touch README.md
git add README.md
git commit -m "add README"
git push -u origin master

#Existing folder
cd existing_folder
git init
git remote add origin http://gitlab.pab.com.cn/CARF/DUN-CLDS/dun-clds-open-front/ios/CarLoanAppR11.git
git add .
git commit -m "Initial commit"
git push -u origin master

#Existing Git repository
cd existing_repo
git remote rename origin old-origin
git remote add origin http://gitlab.pab.com.cn/CARF/DUN-CLDS/dun-clds-open-front/ios/CarLoanAppR11.git
git push -u origin --all
git push -u origin --tags
```

1. 配置远程仓库 origin是远程仓库的别名 代替xxx.git的地址
```shell
git remote add origin https://gitee.com/kingCould/HelloWord.git
```

2. git创建分支并切换到当前新创建的分支上
```shell
git checkout -b dev
```

3. HelloWord工程结构的添加 命令
 ```shell
 git add -A 
 ```
4. 提交git到版本 -m是提交的注释
```shell
git commit -m "注释"
```

开发完成后
```shell
git push origin dev
```

此时就将本地分支推送到远程相应的分支上了，记得推到远端之前先拉取最新代码。
```shell
git pull
```

开始推送
 ```shell
git push <远程主机名> <本地分支名>:<远程分支名>
 
git push origin master:master
 ```

**小提示**

5. git push origin与git push -u origin master的区别

```shell
git push origin
```

上面命令表示，将当前分支推送到origin主机的对应分支。 

如果当前分支只有一个追踪分支，那么主机名都可以省略。 

```shell
git push
```

如果当前分支与多个主机存在追踪关系，那么这个时候-u选项会指定一个默认主机，这样后面就可以不加任何参数使用git push。

```shell
git push -u origin master
```

后面就可以不加任何参数使用git push了。 不带任何参数的git push，默认只推送当前分支，这叫做simple方式。此外，还有一种matching方式，会推送所有有对应的远程分支的本地分支。
Git 2.0版本之前，默认采用matching方法，现在改为默认采用simple方式。

还有代码检视Review、issue、冲突解决等问题，可以参考网上进行操作。



