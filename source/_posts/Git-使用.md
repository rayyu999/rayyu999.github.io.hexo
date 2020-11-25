---
title: Git 使用
date: 2020-11-25 13:52:56
categories: Env
tags: Git
---



Git是目前世界上最先进的分布式版本控制系统。

<!--more-->



## 创建版本库

版本库又名仓库，英文名**repository**，你可以简单理解成一个目录，这个目录里面的所有文件都可以被Git管理起来，每个文件的修改、删除，Git都能跟踪，以便任何时刻都可以追踪历史，或者在将来某个时刻可以“还原”。

创建一个版本库非常简单，首先，选择一个合适的地方，创建一个空目录：

```shell
$ mkdir learngit
$ cd learngit
$ pwd
Path
----
D:\Coding\Java\crud-demo
```

第二步，通过 `git init` 命令把这个目录变成Git可以管理的仓库：

```shell
$ git init
Initialized empty Git repository in D:/Coding/Java/crud-demo/.git/
```



## 把文件添加到版本库

编写一个 `readme.md` 文件，放在版本库的目录下，然后通过两个步骤将文件放到仓库：

第一步，用 `git add` 告诉Git，把文件添加到仓库：

```shell
$ git add readme.md
```

第二步，用命令 `git commit` 告诉Git，把文件提交到仓库：

```shell
$ git commit -m "wrote a readme file"
[master (root-commit) 863006a] wrote a readme file
 1 file changed, 1 insertion(+)
 create mode 100644 readme.md
```

`-m` 后面输入的是本次提交的说明，可以输入任意内容，当然最好是有意义的，这样你就能从历史记录里方便地找到改动记录。



## 远程仓库

### GitHub设置添加SSH

由于你的本地Git仓库和GitHub仓库之间的传输是通过SSH加密的，所以，需要一点设置：

#### 创建SSH Key

在用户主目录下，看看有没有.ssh目录，如果有，再看看这个目录下有没有 `id_rsa` 和 `id_rsa.pub` 这两个文件，如果已经有了，可直接跳到下一步。如果没有，打开Shell（Windows下打开Git Bash），创建SSH Key：

```shell
$ ssh-keygen -t rsa -C "youremail@example.com"
```

把邮件地址换成自己的邮件地址，然后一路回车，使用默认值即可。

如果一切顺利的话，可以在用户主目录里找到 `.ssh` 目录，里面有 `id_rsa` 和 `id_rsa.pub` 两个文件，这两个就是SSH Key的秘钥对，`id_rsa` 是私钥，不能泄露出去，`id_rsa.pub` 是公钥，可以放心地告诉任何人。

#### 在GitHub设置中添加

登陆GitHub，打开"Account settings"，"SSH Keys"页面。然后，点"Add SSH Key"，填上任意Title，在Key文本框里粘贴 `id_rsa.pub` 文件的内容。最后点击 "Add Key"即可。



### 添加远程库

首先在GitHub创建一个Git仓库，然后将本地的仓库与远程仓库关联：

```shell
$ git remote add origin git@github.com:yourgithubname/yourreponame.git
```

把上面的 `yourgithubname` 替换成你自己的GitHub账户名，把 `yourreponame` 换成你的仓库名。

下一步，就可以把本地库的所有内容推送到远程库上：

```shell
$ git push -u origin master
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Writing objects: 100% (3/3), 224 bytes | 224.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:rayyu999/staff-crud-demo.git
 * [new branch]      master -> master
Branch 'master' set up to track remote branch 'master' from 'origin'.
```

把本地库的内容推送到远程，用 `git push` 命令，实际上是把当前分支 `master` 推送到远程。

由于远程库是空的，我们第一次推送 `master` 分支时，加上了 `-u` 参数，Git不但会把本地的 `master` 分支内容推送的远程新的 `master` 分支，还会把本地的 `master` 分支和远程的 `master` 分支关联起来，在以后的推送或者拉取时就可以简化命令。