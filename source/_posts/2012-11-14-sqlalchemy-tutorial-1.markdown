---
layout: post
title: "Python Sqlalchemy指南(一)"
date: 2012-11-14 15:11
comments: true
categories: 
- sqlalchemy
- Pythonthon
---

最近项目需要学习sqlalchemy，对于这个神器，网上的教程实在太少了，而且版本太老。于是只好自己慢慢读，以此系列做一个记录的吧！

>Sqlalchemy version:0.8.b1
>使用sqlite或者mysql来练习
>主要是参照官方文档，以及实际的使用经验
>OS:Win7 or Ubuntu12.04

##Sqlalchemy的架构图

关于这个架构图，文档解释如下：
>InSQLAlchemy ORM, the Object Relational Mapper is introduced and fully described. New users should begin with theObject Relational Tutorial.

>InSQLAlchemy Core, the breadth of SQLAlchemy’s SQL and database integration and description services are doc-umented, the core of which is the SQL Expression language. The SQL Expression Language is a toolkit all its own, independent of the ORM package, which can be used to construct manipulable SQL expressions which can be programmatically constructed, modified, and executed, returning cursor-like result sets.

##环境准备
pip install 或者 'easy install'都可以

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






