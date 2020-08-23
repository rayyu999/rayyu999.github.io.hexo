---
title: Zotero的附件同步
date: 2020-07-10 23:19:21
categories: Env
tags: [Zotero, 坚果云, PaperShip]
---

-----

<!--more-->

Zotero的存储空间只有100M，不付费的话使用起来肯定不够。因此可以使用坚果云来同步附件，这样做的好处是ios上的PaperShip也可以通过WebDAV读取资料。

# 准备工作

注册一个坚果云账号，然后开启第三方应用管理：

1. 登陆坚果云网页端，点击右上角的账号名称——账户信息——安全选项

   ![](http://images.yingwai.top/picgo/zeterojgyf1.png)

   然后点击添加应用，应用名称填Zotero(也可以填其它，只要你记得对应的软件)，会自动生成一个应用密码。

2. 打开Zotero，点击菜单栏的编辑——首选项——同步，在文件同步中选择WebDav，复制坚果云内第三方应用管理的服务器地址、账户和密码，复制完成后点击 *验证服务器*。

   ![](http://images.yingwai.top/picgo/zeterojgyf2.png)

   这一步出现问题的话可以尝试在坚果云网页端手动建立一个zotero根目录，与zotero填的地址对应。



# ZotFile的安装与设置

进入[zotfile官网 ](http://zotfile.com/)下载。
然后打开Zotero，点击菜单栏工具——插件，有以下界面：

![](http://images.yingwai.top/picgo/zeterojgyf3.png)

点击右上角的齿轮，选择 *Install Add-on From File...*，在弹出的插件安装中选择刚才下载的xpi文件进行安装。

安装完成后点击菜单栏工具——ZotFile Preferences——General Settings：

1. 将Source Folder for Attaching new Files中的目录设置为浏览器默认的下载文件目录；
2. 将Location of Files中的目录设置为第一个Attach stored copy of files，要选择这个才能与前面的WebDAV设置配合使用。

![](http://images.yingwai.top/picgo/zeterojgyf4.png)

下面的 /%w/%y 是命名格式，也可以设置成别的。

完成以上操作后，当我们在浏览器中点击Zotero插件时，软件就会自动将下载下来的pdf文件拷贝到云盘的目录中，并将它的目录链接保存到对应的文献条目下。



# PaperShip使用

方法类似上面，只要在坚果云网页端添加应用，在PaperShip登陆时选择WebDAV，把账号密码输入即可。这样在PC端管理的文献就可以方便地在移动端进行访问。

效果：

![](http://images.yingwai.top/picgo/zoterojgyf5.jpg)