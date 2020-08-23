---
title: Hexo博客NexT主题点击侧边栏日志出现问题的解决办法
date: 2020-05-11 18:08:08
categories: Env
tags: [Hexo, NexT]
---

部署好博客并安装了NexT主题后，发现一个问题：侧边栏头像下面的日志点击是404的页面。于是到网上搜索，发现是符号转码的问题。

<!--more-->

**解决方法：**

到`\themes\next\layout\_macro`目录下找到`sidebar.swig`文件，打开找到这一行：

![](http://images.yingwai.top/picgo/nextsidebarrizhif1.png)

原因是`url_for`函数将`||`转码了，

将`theme.menu.archives`后面的括号更换一下位置即可：

![](http://images.yingwai.top/picgo/nextsidebarrizhif2.png)

这时候点击日志就会自动跳转到归档页。