---
title: Git的基本使用
date: 2021-05-15 13:52:21
tags:
- Git
- Shell
categories: Git
mathjax: true
---

Git 是一个开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目。Git 是 Linus Torvalds 为了帮助管理 Linux 内核开发而开发的一个开放源码的版本控制软件。Git 与常用的版本控制工具 CVS, Subversion 等不同，它采用了分布式版本库的方式，不必服务器端软件支持。

 <!-- more --> 

## 初始化git本地仓库

```shell
git init
```

## 查看工作目录和暂存区文件状态

```shell
git status
```

## 添加工作目录文件到暂存区

```shell
git add <file> #添加指定文件到暂存区
git add . #添加当前目录中的所有文件到暂存区
```

## 移除暂存区中的文件

```shell
git rm <file_path> #删除暂存区或分支上的文件，同时工作区也不需要这个文件
git rm --cache <file_path> # 删除暂存区或分支上的文件，但本地又需要使用，只是不希望这个文件被版本控制。
```



## 提交暂存区文件到本地仓库

```shell
git commit
git commit -m'提交的日志信息' #提交并记录对本地仓库的更改
```

## 查看提交的日志信息

```shell
git log
git log --pretty=oneline #在一行中显式日志
git log --graph --pretty=online #图形化日志
git log -3 --pretty=oneline #在一行中显式最近三次提交的日志
git reflog #查看所有日志
```

## 查看本地仓库和工作区之间的更改

```shell
git diff HEAD -- <file>
```

\-\-\- 表变动之前的文件；+++表变动后的文件。变动的位置用两个@作为起始和结束

## 从暂存区中撤销到工作区

```shell
git restore --staged <file>
```

## 撤销上一次操作

```shell
git reset HEAD <file> #撤销上一次的操作
```

## 本地仓库版本回退

```shell
git reset --hard HEAD^ #回退到上一个版本
git reset --hard HEAD^^ #回退两个版本
git reset --hard HEAD~2 #回退两个版本
git reset --hard <版本的唯一标识id> #回退或前进到指定id的版本
```

## 查看git本地仓库中的文件目录

```shell
git ls-files
```

## 从本地仓库中拉取文件到工作目录

```shell
git checkout
```

## 从本地仓库中删除指定文件

```shell
git rm <file>
```

## git查看远程仓库地址：

```shell
git remote -v
```

## 使用本地Git客户端生成SSH公钥与私钥

```shell
ssh-keygen -t rsa -C "your_email@example.com"
```

## 检查测试链接

```shell
ssh -T git@github.com
```

## 绑定远程仓库

```shell
git remote add origin <ssh>
```

## 将本地仓库推送到远程仓库

```shell
git push -u origin master #首次推送
git push #推送
git push origin <branch_name> #推送本地分支到指定远程分支
```

## 切换分支

```shell
git checkout <branch> #切换分支
git checkout -b <new_branch> #新建分支并切换到新建分支
```

## 删除分支

```shell
git branch -d <branch> #删除本地分支
git push origin :<remote_branch> #删除远程分支(本地分支还在)
```

## 查看所有分支

```shell
git branch #查看本地分支
git branch -a #查看本地和远程分支
```

## 合并分支

```shell
git merge <branch>
```

## 重命名分支

```shell
git branch -m|-M <oldbranch> <newbranch>
```

若newbranch名字分支已经存在，则需要使用-M为强制重命名，否则，使用-m进行重命名。

## 拉取远程仓库指定分支并在本地创建分支

```shell
git checkout -b <local_branch> origin/<remote_branch>
```

## 获取远程仓库中最新的状态

```shell
git fetch
```

## 新建标签

```shell
git tag <tag_name>
git tag <tag_name> <指定id版本>
```

## 添加标签并指定标签描述信息

```shell
git tag -a <tag_name> -m'XXX'
git tag -a <tag_name> -m'XXX' <指定id版本>
```

## 查看所有的标签

```shell
git tag
```

## 删除一个本地标签

```shell
git tag -d <tag_name>
```

## 推送本地标签到远程

```shell
git push origin <tag_name> #推送本地标签到远程
git push origin --tags #推送全部未推送过的本地标签到远程
```

## 删除一个远程标签

```shell
git push origin :refs/tags/<tag_name> #远程删除，但本地的还在
```

