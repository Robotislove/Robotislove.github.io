---
layout: post
title: git使用总结笔记
date: 2022-08-21
author: lau
tags: [Giy, Blog]
comments: true
toc: true
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
### Git 常用命令
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


## 1. 本地分支与远程分支关联、推送内容

创建git仓库可以在远端创建一个仓库，然后check到本地，在本地的文件里创建工程文件，然后提交；也可以将本地现有的工程和远端的空仓库关联，本地创建了一个工程 iOSDemo
运行没有错误，就可以提交到远端了。

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




