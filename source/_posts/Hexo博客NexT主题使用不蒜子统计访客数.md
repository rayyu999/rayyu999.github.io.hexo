---
title: Hexo博客NexT主题使用不蒜子统计访客数
date: 2020-05-11 18:06:33
categories: Env
tags: [Hexo, NexT, 不蒜子]
---



想为自己的博客添加访问统计，经过一番查阅，找到了好用又方便的不蒜子统计，不蒜子是一个极简的网页计数器。

<!--more-->



# 安装NexT

参考 http://theme-next.iissnan.com/getting-started.html



# 打开不蒜子统计开关

新版的next主题已经把不蒜子集成进去，只需要打开开关即可：

1. 编辑`\themes\next\_config.yml`，找到里面的`busuanzi_count`配置项，将`enable`设为`true`：

   ![](http://images.yingwai.top/picgo/busuanzif1.png)

   当`enable: true`时，代表开启全局开关。若`site_uv`、`site_pv`、`page_pv`的值均为`false`时，不蒜子仅作记录而不会在页面上显示。

2. 打开对应的站点配置：当`site_uv: true`时，代表在页面底部显示站点的UV值；当`site_pv: true`时，代表在页面底部显示站点的PV值。



# 不蒜子统计不显示的问题

完成上面的步骤后，进入博客，发现统计人数显示不出来：

![](http://images.yingwai.top/picgo/busuanzif2.png)

**原因：不蒜子的域名更换了，但是next主题里面写进去的域名还是以前的。**

[不蒜子官网](http://busuanzi.ibruce.info/)：

![](http://images.yingwai.top/picgo/busuanzif3.png)



**解决方法：**

打开`\themes\next\layout\_third-party\analytics`文件夹里面的`busuanzi-counter.swig`文件，将旧的域名更换为新的域名：

原来的域名：

![](http://images.yingwai.top/picgo/busuanzif4.png)

更换后：

![](http://images.yingwai.top/picgo/busuanzif5.png)

此时博客的访客数就可以正常显示了。