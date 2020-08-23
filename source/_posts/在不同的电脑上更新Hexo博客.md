---
title: 在不同的电脑上更新Hexo博客
date: 2020-08-23 22:55:00
categories: Env
tags: Hexo
---

-----

<!--more-->

参考了https://helloliwen.github.io/870ed150.html以及https://www.zhihu.com/question/21193762中CrazyMilk的回答。

思路是使用git的分支来管理。

# 搭建流程

原本编辑博客的电脑：

1. 首先在github上新建一个hexo分支，并设置它为默认分支。

2. 在电脑上的其它目录中，克隆hexo仓库到本地，获得隐藏的.git文件夹。

3. 复制这个文件夹到原来的博客目录下，删除博客主题目录下的.git文件夹，然后前面克隆的文件夹就可以删除了。

4. 上传

   ```shell
   $ git add .
   $ git commit -m "备注"
   $ git push origin hexo
   ```



# 本机编辑博客

编辑完文章后，

1. 上传到hexo分支

   ```shell
   $ git add .
   $ git commit -m "备注"
   $ git push origin hexo
   ```

2. 部署

   ```shell
   $ hexo d -g
   ```



# 在别的电脑上编辑博客

## 准备工作

1. 安装git
2. 克隆github上的hexo分支到本地
3. 安装node.js
4. 安装hexo（不需要初始化）

后面两步可以参考[Hexo博客搭建](https://yuyingwai.cn/2020/04/11/Hexo博客搭建/)。



## 编辑博客

**每次换电脑进行博客更新时，不管上次在其他电脑有没有更新，最好先执行`git pull hexo` 命令**，即将远端最新的hexo分支拉到本地，使本地与远端达到同步。

然后与本机编辑一样，先用 `git add .`、`git commit -m "备注"` 和 `git push origin hexo` 上传，再进行部署。