---
title: Hexo博客搭建
date: 2020-04-11 18:05:12
categories: Env
tags: Hexo
---

----



<!-- more -->

## Hexo简介

​	Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 [Markdown](http://daringfireball.net/projects/markdown/)（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。



## 搭建流程（Windows）

### 准备工作

#### 安装Git

到 https://git-scm.com/download 选择对应的平台进行下载安装即可。



#### 安装Node.js

到 https://nodejs.org/en/ 下载安装即可。（Node.js 版本需不低于 8.10，建议使用 Node.js 10.0 及以上版本）



### 安装Hexo

安装需要借助npm包管理器，由于在国内这个镜像源很慢，因此可以利用npm安装cnpm淘宝镜像源，在命令行中输入：

```shell
$ npm install -g cnpm --registry=https://registry.npm.taobao.org
```

安装完后就可以使用cnpm安装hexo：

```shell
$ cnpm install -g hexo-cli
```

可以用`hexo -v`来验证是否安装成功。



### 正式搭建

#### 初始化

建立一个名为 *Blog* 的文件夹，在命令行中进入这个文件夹，在命令行中输入：

```shell
$ hexo init
```

初始化完成后，在命令行中输入`hexo s`，此时在浏览器中输入`localhost:4000`就可以看到博客已经创建好了，并且默认创建了一篇文章。

确认过后，键盘按 *Ctrl + C* 即可停止服务。



#### 部署到Github

首先登陆到[Github](https://github.com/)，新建一个仓库，命名为 "**你的Github昵称.github.io**"。

然后需要安装一个git部署插件，在命令行中打开 *Blog* 目录，输入：

```shell
$ cnpm install --save hexo-deployer-git
```

打开 *Blog* 目录下的站点配置文件，在文件最后的 ***repo*** 处输入刚刚创建的仓库的地址并在下方添加一行：

```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: https://github.com/xxx/xxx.github.io.git
  branch: master	# 指定分支，不填默认为master
```

保存退出后，在命令行中输入：

```shell
$ hexo d
```

部署到远端，此时刷新一下自己的Github仓库页面，会发现多了很多文件。在浏览器中访问 **xxx.github.io**，就可以看到博客已经成功部署。



## 使用Hexo

### 新建文章

在命令行中输入：

```shell
$ hexo n "文章标题"
```

会在博客目录的`/source/_posts`目录下生成一个markdown文件，使用编辑器打开即可编辑这篇文章。



### 发布文章

在命令行中依次输入：

```shell
$ hexo clean	# 清理缓存文件和已生成的静态文件
$ hexo g	# 生成静态文件
```

再使用`hexo s`命令就可以在`localhost:4000`页面看到新生成的文章已经发布到了博客上面。