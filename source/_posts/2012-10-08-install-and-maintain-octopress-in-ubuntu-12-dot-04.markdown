---
layout: post
title: "在Ubuntu 12.04下用Github安装和维护Octopress"
date: 2012-10-08 20:45
comments: true
categories:
- Life
- Octopress
---
##前言##
其实之前用Github+Octopress架设博客用的是Win7,但是始终解决不了中文问题,今天换回Ubuntu,顺利解决,这里就干脆用Ubtuntu来做一个记录吧.

##1 安装Ruby环境##
利用RVM安装ruby,RVM管理Ruby的话感觉更爽一点,自己也懒得编译了.

{% codeblock %}
root@Node1:apt-get install curl git
root@Node1:curl -L get.rvm.io | bash -s stable
{% endcodeblock %}

<!--more-->

根据RVM的要求,还需要将用户加入组以及更新配置.

{% codeblock %}
root@Node1:gpasswd -a root rvm
root@Node1:source /etc/profile.d/rvm.sh
{% endcodeblock %}

安装编译依赖以及Ruby

{% codeblock %}
root@Node1:apt-get install build-essential bison openssl libreadline6 libreadline6-dev curl git-core zlib1g
zlib1g-dev libssl-dev libyaml-dev libsqlite3-0 libsqlite3-dev sqlite3 libxml2-dev libxslt-dev autoconf
root@Node1:rvm install 1.9.2
{% endcodeblock %}

OK,环境安装完成.

##2 安装Octopress以及组件##
先通过git克隆octopress的代码.再安装相应的组件.

{% codeblock %}
git clone git://github.com/imathis/octopress.git octopress
cd octopress
gem install bundler
bundle install
{% endcodeblock %}

然后安装默认主题:

{% codeblock %}
rake install
{% endcodeblock %}

##3 设置Github##
这一步是以后维护的关键主要有以下步骤:

{% codeblock %}
rake setup_github_pages 设置于github的关联
rake generate 生成静态页面
rake deploy 向github发送页面
{% endcodeblock %}

注意,所有的操作都应该是在octopress的源码中进行,rake deploy只是将生成的静态页面发送到github的master分支.,所以最后我们需要将源码保存为source分支.

{% codeblock %}
git status
git add .
git commit -m 'commit message'
git push origin source
{% endcodeblock %}

以后维护的时候记得需要先checkout到source分支才行.

