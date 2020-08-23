---
title: Hexo博客NexT主题翻页按钮显示不正常的解决方法
date: 2020-05-11 18:23:29
categories: Env
tags: [Hexo, NexT]
---

搭好博客后，上去发现底下的翻页按钮显示有问题，显示为`<i class="fa fa-angle-right"></i>`，原因不明：

<!--more-->

![](http://images.yingwai.top/picgo/fanyef1.png)



**解决办法：**

最简单的办法就是将`<i class="fa fa-angle-right"></i>`这个不能正常显示的字体图标改成一般的字符，我就是按照网上的方法改成正常的一般左右键字符 “ > ”。

到`\themes\next\layout\_partials`目录下找到`pagination.swig`文件，将

```html
{% if page.prev or page.next %}
  <nav class="pagination">
    {{
      paginator({
        prev_text: '<i class="fa fa-angle-left"></i>',
        next_text: '<i class="fa fa-angle-right"></i>',
        mid_size: 1
      })
    }}
  </nav>
{% endif %}
```

改成

```html
{% if page.prev or page.next %}
  <nav class="pagination">
    {{
      paginator({
        prev_text: '<',
        next_text: '>',
        mid_size: 1
      })
    }}
  </nav>
{% endif %}
```

重新部署即可看到已经正常显示了：

![](http://images.yingwai.top/picgo/fanyef2.png)

