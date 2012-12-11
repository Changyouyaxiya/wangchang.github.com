---
layout: post
title: "python routes 2 RESTful services"
date: 2012-11-29 18:37
comments: true
categories:
- Python
- Routes
- WSGI
- Web开发
---

这一节主要讲讲用Routes来配置RESTful web services。`map.resource`可以根据Atom publishing protocol协议来创建一个`创建/修改/删除`的routes。也就是说根据resource来配置routes。

<!--more-->
##Resource
一个resource route定义了一个collection中的成员member，以及一个collection自身。一般来说，一个collection就是一个复数单词，一个member就是相应的一个单数词，举例：

在restful服务中，我定义了关于虚拟连接VC的一些API。那么
* resoure:vc,vcs
* collection:vcs
* member:vc
再看看一个复杂点的例子：
{% codeblock lang:python %}
map = Mapper()
map.resource("message", "messages")

# The above command sets up several routes as if you had typed the
# following commands:
map.connect("messages", "/messages",
    controller="messages", action="create",
    conditions=dict(method=["POST"]))

map.connect("messages", "/messages",
    controller="messages", action="index",
    conditions=dict(method=["GET"]))

map.connect("formatted_messages", "/messages.{format}",
    controller="messages", action="index",
    conditions=dict(method=["GET"]))

map.connect("new_message", "/messages/new",
    controller="messages", action="new",
    conditions=dict(method=["GET"]))

map.connect("formatted_new_message", "/messages/new.{format}",
    controller="messages", action="new",
    conditions=dict(method=["GET"]))

map.connect("/messages/{id}",
    controller="messages", action="update",
    conditions=dict(method=["PUT"]))

map.connect("/messages/{id}",
    controller="messages", action="delete",
    conditions=dict(method=["DELETE"]))

map.connect("edit_message", "/messages/{id}/edit",
    controller="messages", action="edit",
    conditions=dict(method=["GET"]))

map.connect("formatted_edit_message", "/messages/{id}.{format}/edit",
    controller="messages", action="edit",
    conditions=dict(method=["GET"]))

map.connect("message", "/messages/{id}",
    controller="messages", action="show",
    conditions=dict(method=["GET"]))

map.connect("formatted_message", "/messages/{id}.{format}",
    controller="messages", action="show",
    conditions=dict(method=["GET"]))
{% endcodeblock %}

看到这里，我们可以推断出这么几个名词的定义：
* __controller:就是一个WSGI的应用
* __resource(s):就是REST API中的名词
* __method:就是HTTP的方法，GET POST等
* __action:Routes中与method对应，比如GET就对应"index"动作
* __collection:等同于resources。

上面的代码建立了如下的转换关系：
{% codeblock - awesome.rb %}
GET    /messages        => messages.index()    => url("messages")
POST   /messages        => messages.create()   => url("messages")
GET    /messages/new    => messages.new()      => url("new_message")
PUT    /messages/1      => messages.update(id) => url("message", id=1)
DELETE /messages/1      => messages.delete(id) => url("message", id=1)
GET    /messages/1      => messages.show(id)   => url("message", id=1)
GET    /messages/1/edit => messages.edit(id)   => url("edit_message", id=1)
{% endcodeblock %}
我们很明显的看到，这个`messages`就是一个WSGI应用，准确的来说，在代码中就是一个类的实例。

官网上有这么个解释，我就不翻译了：
>Thus, you GET the collection to see an index of links to members (“index” method). You GET a member to see it (“show”). You GET “COLLECTION/new” to obtain a new message form (“new”), which you POST to the collection (“create”). You GET “MEMBER/edit” to obtain an edit for (“edit”), which you PUT to the member (“update”). You DELETE the member to delete it. Note that there are only four route names because multiple actions are doubled up on the same URLs.

##Resource选项。
之前我们有`map.resource("message", "messages")`来增加了一个resource资源。可以增加一些参数。如下：
* __controller
使用指定的controller，否则routes根据collection的名字来决定controller的名字。
* __collection
为某以collection添加URL支持，如下：

    map.resource("message", "messages", collection={"rss": "GET"})
    # "GET /message/rss"  =>  ``Messages.rss()``.
    # Defines a named route "rss_messages".自动定义

* __member
为一个member添加URL支持，如下：

    map.resource('message', 'messages', member={'mark':'POST'})
    # "POST /message/1/mark"  =>  ``Messages.mark(1)``
    # also adds named route "mark_message"

* __new
为新成员加入URL支持，如下：

    map.resource("message", "messages", new={"preview": "POST"})
    # "POST /messages/new/preview"

* __path_prefix

* __name_prefix

    map.resource("message", "messages", controller="categories",
       path_prefix="/category/{category_id}",
       name_prefix="category_")
    # GET /category/7/message/1
    # Adds named route "category_message"

* __parenet_resource
一个包含父类资源的字典，用于创建嵌套（嵌入式）的资源。需要包含父类资源的member_name和collection_name，这个字典可以在请求的时候通过`request.environ["routes.route"]`访问。

后面不是很懂了。。。
If parent_resource is supplied and path_prefix isn’t, path_prefix will be generated from parent_resource as “<parent collection name>/:<parent member name>_id”.

If parent_resource is supplied and name_prefix isn’t, name_prefix will be generated from parent_resource as “<parent member name>_”.
{}
>>> m = Mapper()
>>> m.resource('location', 'locations',
...            parent_resource=dict(member_name='region',
...                                 collection_name='regions'))
>>> # path_prefix is "regions/:region_id"
>>> # name prefix is "region_"
>>> url('region_locations', region_id=13)
'/regions/13/locations'
>>> url('region_new_location', region_id=13)
'/regions/13/locations/new'
>>> url('region_location', region_id=13, id=60)
'/regions/13/locations/60'
>>> url('region_edit_location', region_id=13, id=60)
'/regions/13/locations/60/edit'

Overriding generated path_prefix:

>>> m = Mapper()
>>> m.resource('location', 'locations',
...            parent_resource=dict(member_name='region',
...                                 collection_name='regions'),
...            path_prefix='areas/:area_id')
>>> # name prefix is "region_"
>>> url('region_locations', area_id=51)
'/areas/51/locations'

Overriding generated name_prefix:

>>> m = Mapper()
>>> m.resource('location', 'locations',
...            parent_resource=dict(member_name='region',
...                                 collection_name='regions'),
...            name_prefix='')
>>> # path_prefix is "regions/:region_id"
>>> url('locations', region_id=51)
'/regions/51/locations'
{}







