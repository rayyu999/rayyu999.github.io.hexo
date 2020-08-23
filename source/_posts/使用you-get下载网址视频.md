---
title: 使用you-get下载网址视频
date: 2020-05-20 18:50:10
categories: Env
tags: Python
---

----



<!--more-->

you-get是基于Python开发的，实际它不只支持视频下载，还支持图片、音乐等。而且，只要视频的地址，一行代码即可。



# 安装you-get

安装you-get的方式有很多，下面三种择其一即可。



## 使用pip安装

```shell
$ pip3 install you-get
```

![](http://images.yingwai.top/picgo/yougetf1.png)



## Git克隆

```shell
$ git clone git://github.com/soimort/you-get.git
```

然后直接运行`./setup.py`即可

```shell
$ python3 setup.py install
```



## 通过HomeBrew安装（Mac）

```shell
$ brew install you-get
```



# 下载视频

命令行中输入以下代码下载视频：

```shell
$ you-get '视频地址URL'
```

这里以B站某视频为例：

![](http://images.yingwai.top/picgo/yougetf2.png)

下载好的视频存放在系统盘的用户目录下。



# 查看视频信息

命令行输入：

```shell
$ you-get -i '视频地址URL'
```

用刚刚下载的视频测试，结果如下：

![](http://images.yingwai.top/picgo/yougetf3.png)

可以看到它的默认设置不是MP4格式的视频，如果想要换成这种格式，可以在命令行输入：

```shell
$ you-get --itag=18 '视频地址URL'
```



# 支持的网站

除了B站，还可以用you-get下载国内外很多主流网站的视频、图片和音乐。

这里列出支持的国外网站：

![](http://images.yingwai.top/picgo/yougetf4.jpg)