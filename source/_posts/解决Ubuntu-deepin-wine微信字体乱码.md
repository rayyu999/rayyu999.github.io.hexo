---
title: 解决Ubuntu deepin-wine微信字体乱码
date: 2020-04-24 00:30:22
categories: Env
tags: [Ubuntu, Linux]
---

----



<!--more-->

# 解决乱码+修改字体(微软雅黑)

下载微软雅黑字体,msyh.ttc



## 添加字体

```shell
$ cp msyh.ttc ~/.deepinwine/Deepin-WeChat/drive_c/windows/Fonts
```



## 修改系统注册表

```shell
$ gedit ~/.deepinwine/Deepin-WeChat/system.reg
```

修改以下两行

```
"MS Shell Dlg"="msyh"
"MS Shell Dlg 2"="msyh"
```



## 字体注册

```shell
$ gedit msyh_config.reg
```

内容添加

```
REGEDIT4
[HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\FontLink\SystemLink]
"Lucida Sans Unicode"="msyh.ttc"
"Microsoft Sans Serif"="msyh.ttc"
"MS Sans Serif"="msyh.ttc"
"Tahoma"="msyh.ttc"
"Tahoma Bold"="msyhbd.ttc"
"msyh"="msyh.ttc"
"Arial"="msyh.ttc"
"Arial Black"="msyh.ttc"
#注册
WINEPREFIX=~/.deepinwine/Deepin-WeChat deepin-wine regedit msyh_config.reg
```



## Reboot