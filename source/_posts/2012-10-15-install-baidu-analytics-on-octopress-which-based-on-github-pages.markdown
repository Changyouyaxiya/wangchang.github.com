---
layout: post
title: "在用Github搭建的Octopress博客中安装百度统计"
date: 2012-10-15 20:38
comments: true
categories: 
- Octopress
- SEO
---

我发现用Github搭建的octopress有个缺点就是貌似不怎么容易被搜索引擎检索到,这个先不管了,以后再折腾.这里主要记录一下如何在octopress中安装百度统计.因为百度统计不是octopress默认支持的,不像Google分析那样直接改了配置就能用,所以需要手动安装.

>OS:Ubuntu 12.04 2012年10月15日 第一版

<!--more-->

##进入百度统计,添加站点,获取站点代码.
主要是一段javascript的代码,在有访问时通过这段代码将信息发送给百度统计的分析端.我的代码如下:
{% codeblock lang:js %}

<script type="text/javascript">
var _bdhmProtocol = (("https:" == document.location.protocol) ? " https://" : " http://");
document.write(unescape("%3Cscript src='" + _bdhmProtocol + "hm.baidu.com/h.js%3F3e617f2de1bb9051b3b53a2ac839280c' type='text/javascript'%3E%3C/script%3E"));
</script>

{% endcodeblock %}

##添加代码到Octopres
因为Octopress不是纯html,用了CSS,所以在`source/index.html`中没有Body段,整个界面大致分为header,navgiation,body,footer四个部分,所以我们把代码放在footber里面合适一点.进入`source/_includes`目录,新建文件`baidu_analytics.html`,在其中复制之前得到的统计代码.

随后更改`source/_includes`下的`after_footer.html`文件,增加一行:
{% codeblock %}

{\% include baidu_analytics.html %\} ##去掉反斜线\

{% endcodeblock %}

##测试
这里已经安装完成,进入[百度统计后台](http://tongji.baidu.com)进行代码安装检查,检测到代码安装成功就OK了.

