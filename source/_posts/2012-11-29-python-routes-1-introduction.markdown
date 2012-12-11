---
layout: post
title: "Python.Routes指南1 "
date: 2012-11-29 16:08
comments: true
categories: 
- Python
- Routes
- WSGI
- Web开发
---

##什么是Routes
用python重新实现的Rails Routes System，主要用来将URL映射到不同的应用动作（application actions）。特别适合于RESTFUL API。

Routes支持基于domain, cookies, HTTP method, 自定义function的映射。目前支持的功能：

Sophisticated route lookup and URL generation
Named routes
Redirect routes
Wildcard paths before and after static parts
Sub-domain support built-in
Conditional matching based on domain, cookies, HTTP method (RESTful), and more
Easily extensible utilizing custom condition functions and route generation functions
Extensive unit tests

安装：

    easy_install Routes
    pip install Routes

##1 基本概念
###component组件
一个URL中的一个个组成部分，例如/blog/2012就有两个component:blog,2012
###generation
基于route的名字或者变量的值创建一个URL，这个是match的反过程。
###mapper
routes的容器，一般一个应用一个mapper，一个mapper知道如何去匹配routes或者产生routes.
###minimization
允许短URL进行长路劲的匹配，已废弃。
###routes
一个规则，将一个URL映射成为一个routing variables的字典，比如，规则是“/{controller}/{action}”，请求的URL是“/help/about”，那么结果字典就是：

    {"controller": "help", "action": "about"}

routes并不知道这些变量的含义，它只是把这个结果返回给应用。一个route可以拥有一个名字。
###route paths
一条route中的URL模式（pattern）
###routing variables
匹配之后返回的键值对。route path中定义的变量叫做path变量，他们的值来自URL中。route path外定义的变量叫做默认变量，其值不受URL影响。

特别注意！在WSGI中，用于route的关键字是environment（environ）中的wsgiorg.routing_args.

##2 简单的例子：

    # Setup a mapper
    from routes import Mapper
    map = Mapper()
    map.connect(None, "/error/{action}/{id}, controller="error")
    map.connect("home", "/", controller="main", action="index")
    map.connect(None, "/{controller}/{action}")
    map.connect(None, "/{controller}/{action}/{id}")
    

    # Match a URL, returns a dict or None if no match
    result = map.match('/error/myapp/4')
    # result == {'controller': 'main', 'action': 'myapp', 'id': '4'}

对上面代码的一个简单解释：
1~2行创建了一个简单的mapper。

3行创建了一个以/error开始的任意三段的路由，并设置controller变量为一个常量！所以如果一个URL是`/error/images/arrow.jpg`，执行map.match（"/error/images/arrow.jpg"）得到的字典结果就会是：

`{"controller": "error", "action": "images", "id": "arrow.jpg"}`

4行创建了/的映射，并将controller和action设置为常量。而"home"表示此条route的名字。

6行匹配任务两端的URL。7行匹配任意三段的URL。所以前面的例子" “/error/images/arrow.jpg”"还能够匹配6、7行，在这种情况下，Routes采用的是以第一次匹配到的为准，顺序优先。

还有下面的例子：

    m.connect("/feeds/{category}/atom.xml", controller="feeds", action="atom")
    m.connect("history", "/archives/by_eon/{century}", controller="archives",
          action="aggregate")
    m.connect("article", "/article/{section}/{slug}/{page}.html",
          controller="article", action="view")

##3 初步深入
###3.1 Magic path_info
如果URL中使用了path_info，Routes会将先前的所有信息移动到“SCRIPT_NAME”这个环境变量中。当另外的WSGI应用有自己的routing的时候这个能起到很好的转移作用。看看如下的例子，用于WSGI环境。

    map.connect(None, "/cards/{path_info:.*}",
       controller="main", action="cards")
    # Incoming URL "/cards/diamonds/4.png"
    => {"controller": "main", action: "cards", "path_info": "/diamonds/4.png"}
    # Second WSGI application sees:
    # SCRIPT_NAME="/cards"   PATH_INFO="/diamonds/4.png"
###3.2 Condition条件匹配
在一条route中，可以加入相应的条件，以加强匹配效果。condition是一个字典，最多只有以下三个关键字：
* method
大写的HTTP方法。
* sub_domain
这个目前不是很熟悉。
* function
三种方式使用如下：用一个函数来对请求进行处理，`func(environ, match_dict) => bool`

{}
# Match only if the HTTP method is "GET" or "HEAD".
m.connect("/user/list", controller="user", action="list",
          conditions=dict(method=["GET", "HEAD"]))

# A sub-domain should be present.
m.connect("/", controller="user", action="home",
          conditions=dict(sub_domain=True))

# Sub-domain should be either "fred" or "george".
m.connect("/", controller="user", action="home",
          conditions=dict(sub_domain=["fred", "george"]))

# Put the referrer into the resulting match dictionary.
# This function always returns true, so it never prevents the match
# from succeeding.
def referals(environ, result):
    result["referer"] = environ.get("HTTP_REFERER")
    return True
m.connect("/{controller}/{action}/{id}",
    conditions=dict(function=referals))

{}
###3.3 在route中还可以使用通配符和正则表达式
###3.4 格式扩展
通过`{.format}`，可以匹配一个格式的扩展，比如.html或者.json。比如：

    map.connect('/entries/{id}{.format:json}')

就只会匹配到"/entries/1.json"。
###submapper
###从子程序添加routes
一个子程序可以传递给父程序一个routes的列表，通过mapper的`extend`方法，增加routes。如下：

    routes = [
       Route("index", "/index.html", controller="home", action="index"),
        ]

    map.extend(routes)
    # /index.html => {"controller": "home", "action": "index"}

    map.extend(routes, "/subapp")
    # /subapp/index.html => {"controller": "home", "action": "index"}


<http://routes.readthedocs.org/en/latest/setting_up.html>




