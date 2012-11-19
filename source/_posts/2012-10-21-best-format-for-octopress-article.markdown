---
layout: post
title: "Octopress中文章信息头的最好形式"
date: 2012-10-21 13:36
comments: true
categories: octopress
description: "在octopress中关于文章头信息的最佳格式"
---

主要记录一下octopres中编辑文章头信息最好的格式。
>OS:Ubuntu12.04 2012年10月21日 第一版

{% codeblock best format - awesome.rb %}

---
layout: post
title: "SEO for Octopress"  #文章标题
date: 2012-04-22 09:55  #写作时间
comments: true #允许评论
categories: [seo,octopress,...] #多个分类标记
tags: [key1,key2,...] #支持多个tag的功能
keywords: seo,octopress,... #关键字
description: “How to optimize Octopress for SEO,Heroku” #文章描述，用于搜索引擎显示的内容
---

{% endcodeblock %}

<!--more-->

其中，catagoies有两种方式：

{% codeblock %}
categories: [cate1,cate2,...]

categories:
- cate1
- cate2
- ...
{% endcodeblock %}

tags的也有如上两种方式，*至于keywords项的作用，还暂时不知道*,但是按照我目前的想法，categories支持多个以后，和tags的功能的功能重合了，所以tags可以不要，倒是这个description相比较重要，有利于SEO，关于SEO可以参考：

>[Github Pages + Jekyll搭建博客之SEO](http://zyzhang.github.com/blog/2012/09/03/blog-with-github-pages-and-jekyll-seo/)

>[SEO for Octopress,Heroku](http://www.yatishmehta.in/seo-for-octopress)
