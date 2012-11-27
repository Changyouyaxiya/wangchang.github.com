---
layout: post
title: "Python Sqlalchemy指南(一)"
date: 2012-11-14 15:11
comments: true
categories: 
- sqlalchemy
- Python
---

最近项目需要学习sqlalchemy，对于这个神器，网上的教程实在太少了，而且版本太老。于是只好自己慢慢读，以此系列做一个记录的吧！

>Sqlalchemy version:0.8.b1
>使用sqlite或者mysql来练习
>主要是参照官方文档，以及实际的使用经验
>OS:Win7 or Ubuntu12.04

<!--more-->
##Sqlalchemy的架构图

关于这个架构图，文档解释如下：
>InSQLAlchemy ORM, the Object Relational Mapper is introduced and fully described. New users should begin with theObject Relational Tutorial.

>InSQLAlchemy Core, the breadth of SQLAlchemy’s SQL and database integration and description services are doc-umented, the core of which is the SQL Expression language. The SQL Expression Language is a toolkit all its own, independent of the ORM package, which can be used to construct manipulable SQL expressions which can be programmatically constructed, modified, and executed, returning cursor-like result sets.

##环境准备
pip install 或者 'easy install'都可以



##概念解释
###Engine Class
class engine主要作用是1、管理与数据库的连接（一个连接池） 2、一个包含多个策略的连接池，这些策略用来配置从连接池中获取连接的方式。
函数create_engine(url,args)用于创建一个和数据库的engine，用于连接的URL格式为：`driver://username:password@host:port/database`有以下几个方式:

    #Create a connection to a database
    engine = create_engine('sqlite://')
    engine = create_engine('sqlite:///data.sqlite')
    engine = create_engine('mysql://localhost/mysql_db')

其中，参数args是一个字典，有以下一些：

    echo=True or False
    strategy
    use_ansi
    encoding

一个engine有以下方法：connect

###MetaData Class
class metadata主要作用是，维护数据方法（mysql,sqlite等）以及表的定义。其中包含了收集管理描述表的类的功能。使用MetaData的构造器即可实例化，如果构造器中有engine或者url参数，那么这就是一个bound的MetaDate，如果没参数，就是一个unbound的MetaData。

    unbound_meta = MetaData()

    db = create_engine('sqlite://')
    bound_meta = MetaData(bind=db)

当MetaData被bind以后，就可以方便的创建表等操作了，如下：
类

{% codeblock lang:python %}
from sqlalchemy import *
from datetime import datetime
metadata=MetaData()
user_table = Table(
    'tf_user', metadata,
    Column('id', Integer, primary_key=True),
    Column('user_name', Unicode(16), unique=True, nullable=False),
    Column('email_address', Unicode(255), unique=True, nullable=False),
    Column('password', Unicode(40), nullable=False),
    Column('first_name', Unicode(255), default=''),
    Column('last_name', Unicode(255), default=''),
    Column('created', DateTime, default=datetime.now))

user_table.create()#在数据库中创建相应的Table
{% endcodeblock %}

###表、列、约束
Table.__init__(self, name, metadata,*args, **kwargs)
Column.__init__(self, name, type_, *args, **kwargs)

Primary keys

用primary_key=True来指定一个列的主键，例如

    Column('brand_id', Integer, ForeignKey('brand.id'),primary_key=True)

Foreign keys

Foreign keys是将一个表中的一行与另一个表中的一行进行联系，调用方式：
ForeignKey.__init__( self, col-umn, constraint=None, use_alter=False, name=None, onupdate=None, ondelete=None)
Column('brand_id', Integer, ForeignKey('brand.id'))




##
和数据库连接，需要一个engine！
{% codeblock %}
from sqlalchemy import create_engine
engine = create_engine(’sqlite:///:memory:’, echo=True)
{% endcodeblock %}
引擎的方法：
engine.execute("select 1").scalar() 直接执行sql语句。
在使用ORM中，应该有两步，一是描述我们要处理的数据库，而是将我们的类和数据库中的table关联起来。在sqlalchemy中，这两步合二为一，使用declarative机制，在这个机制中，所有需要映射的类都是根据一个维护分类和映射关系的基类来的，这个基类就是declarative base class。

from sqlalchemy.ext.declarative import declarative_base
Base = declarative_base() #创建这个base类






