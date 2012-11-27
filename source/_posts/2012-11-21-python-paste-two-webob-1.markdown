---
layout: post
title: "Python.Paste指南之WebOb(1)-概念"
date: 2012-11-21 16:56
comments: true
categories: 
- Paste.webob
- Python
---

这次主要来学习Paste的核心内容，WebOb，内容依然来自官方翻译、网络参考以及自己的实践。

>
>

##What is WebOb?
WebOb是一个Python库，主要是用在WSGI中对请求环境变量request environment（也就是WSGI应用中的参数environ）进行包装（提供wrapper），并提供了一个对象来方便的处理返回response消息。WebOb提供的对象映射了大多数的HTTP方法，包括头解析，content协商等。这里的映射，就是说只需要对对象进行操作，就可以完成HHTP的方法，从而大大简化开发难度。引用官方文档，WebOb有以下特点：

>Maps most of HTTP spec to friendly data structures.
>Time-proven codebase that works around and hides all known WSGI quirks.
>Zero known issues (reported bugs are always fixed ASAP).
>100% test coverage.
>No external dependencies.
>Supports Python 3

WebOb为HTTP的请求和相应提供了相应的对象。通过对WSGI request environment进行wrap来简化操作。
##Request
`webob.Request`对象是对WSGI environ dictionary的一个包装。后者这个字典以键值的形式描述了一个HTTP请求包括path信息和query string，以及一个与文件对象类似的秒速请求的body，以及一些其他的自定义keys。如下可以创建一个简单的Request对象。

    >>>from webob import Request
    >>>environ = {'method': 'GET'}  
    >>>req = Request(environ) 

强调，`The request object wraps the environment`，仅仅是包装，所以你可以对这个environ进行读写的。可以通过req.environ可以访问到这个environ。但是一般都没必要这样做，直接用这个Request对象来操作就好了。如下：

    >>>req.environ
    {'method': 'GET'}
    >>>req.method 
    'GET'
    
此外，还有，req.accept_language, req.content_length, req.user_agent等。我们可以通过dir(req)来看看这个对象的方法咯。
---
{% codeblock - awesome.rb %}
>>>dir(req)
['GET', 'POST', 'ResponseClass', '__class__', '__delattr__', '__dict__', '__doc__', '__format__', '__getattr__', '__getattribute__', '__hash__', '__init__', '__module__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', '_body__del', '_body__get', '_body__set', '_body_file__del', '_body_file__get', '_body_file__set', '_cache_control__del', '_cache_control__get', '_cache_control__set', '_charset', '_check_charset', '_content_type__get', '_content_type__set', '_content_type_raw', '_copy_body_tempfile', '_headers', '_headers__get', '_headers__set', '_host__del', '_host__get', '_host__set', '_is_body_readable__get', '_is_body_readable__set', '_json_body__del', '_json_body__get', '_json_body__set', '_setattr_stacklevel', '_text__del', '_text__get', '_text__set', '_update_cache_control', '_urlargs__del', '_urlargs__get', '_urlargs__set', '_urlvars__del', '_urlvars__get', '_urlvars__set', 'accept', 'accept_charset', 'accept_encoding', 'accept_language', 'application_url', 'as_bytes', 'as_string', 'as_text', 'authorization', 'blank', 'body', 'body_file', 'body_file_raw', 'body_file_seekable', 'cache_control', 'call_application', 'charset', 'client_addr', 'content_length', 'content_type', 'cookies', 'copy', 'copy_body', 'copy_get', 'date', 'decode', 'encget', 'encset', 'environ', 'from_bytes', 'from_file', 'from_string', 'from_text', 'get_response', 'headers', 'host', 'host_port', 'host_url', 'http_version', 'if_match', 'if_modified_since', 'if_none_match', 'if_range', 'if_unmodified_since', 'is_body_readable', 'is_body_seekable', 'is_xhr', 'json', 'json_body', 'make_body_seekable', 'make_default_send_app', 'make_tempfile', 'max_forwards', 'method', 'params', 'path', 'path_info', 'path_info_peek', 'path_info_pop', 'path_qs', 'path_url', 'pragma', 'query_string', 'range', 'referer', 'referrer', 'relative_url', 'remote_addr', 'remote_user', 'remove_conditional_headers', 'request_body_tempfile_limit', 'scheme', 'script_name', 'send', 'server_name', 'server_port', 'str_GET', 'str_POST', 'str_cookies', 'str_params', 'text', 'upath_info', 'url', 'url_encoding', 'urlargs', 'urlvars', 'uscript_name', 'user_agent']
{% endcodeblock %}
---
一个WSGI的environ需要一些必须的参数，所以Request对象有一个构造器，在创建的时候就会自动添加最小的一些必要的项，如上，我们只指定了一个method，那么我们看看。

    >>> req
    <Request at 0x2bbb4d0 (invalid WSGI environ)>

显示这个WSGI environ是无效的，因为缺少必要的信息。我们来调用构造器的方法：
{% codeblock %}
req = Request.blank('/article?id=1')
>>> from pprint import pprint
>>> pprint(req.environ)
{'HTTP_HOST': 'localhost:80',
 'PATH_INFO': '/article',
 'QUERY_STRING': 'id=1',
 'REQUEST_METHOD': 'GET',
 'SCRIPT_NAME': '',
 'SERVER_NAME': 'localhost',
 'SERVER_PORT': '80',
 'SERVER_PROTOCOL': 'HTTP/1.0',
 'wsgi.errors': <open file '<stderr>', mode 'w' at ...>,
 'wsgi.input': <...IO... object at ...>,
 'wsgi.multiprocess': False,
 'wsgi.multithread': False,
 'wsgi.run_once': False,
 'wsgi.url_scheme': 'http',
 'wsgi.version': (1, 0)}
{% endcodeblock %}
pprint用来打印字典等效果很好的。这里简单对url做一个说明，/article?id=1我们可以看到article是属于PATH中的，而?分割的是参数，也就是query_string。如果指定了多个参数，也是一起放在QUERY_STRING里面的，如下：
---
{}
>>> req = Request.blank('/article?id=1&id=2')
>>> pprint(req.environ)
{'HTTP_HOST': 'localhost:80',
 'PATH_INFO': '/article',
 'QUERY_STRING': 'id=1&id=2',
 'REQUEST_METHOD': 'GET',
 'SCRIPT_NAME': '',
 'SERVER_NAME': 'localhost',
 'SERVER_PORT': '80',
 'SERVER_PROTOCOL': 'HTTP/1.0',
 'wsgi.errors': <open file '<stderr>', mode 'w' at 0x7f33b863a270>,
 'wsgi.input': <_io.BytesIO object at 0x2b94e30>,
 'wsgi.multiprocess': False,
 'wsgi.multithread': False,
 'wsgi.run_once': False,
 'wsgi.url_scheme': 'http',
 'wsgi.version': (1, 0)}
{}
---
##Request body
req.body是一个描述了请求body（比如，POST一个表单，PUT的内容）的一个类文件对象（file-like object），我们先把body设置为一个字符串，会被自动转换为一个类文件对象，通过req.body可以访问到body的内容。

    >>> req.body="this is body"
    >>> req.body
    'this is body'
    >>> type(req.body)
    <type 'str'>

##Method & URL
一个请求的所有属性都可以通过Request对象的属性来访问，这些属性如下：
一个请求对象里面比较重要的属性有以下：
---
{% codeblock %}
>>> req.scheme
'http'
>>>req.method
'GET'
>>> req.GET
GET([(u'id', u'1'), (u'id', u'2')])
>>> req.params
NestedMultiDict([(u'id', u'1'), (u'id', u'2')])
>>> req.body
'this is body'
>>> req.headers
<webob.headers.EnvironHeaders object at 0x2bbbc10>
>>> req.script_name  # The base of the URL
''
>>> req.script_name = '/blog' # make it more interesting
>>> req.path_info    # The yet-to-be-consumed part of the URL
'/article'
>>> req.content_type # Content-Type of the request body
''
>>> req.host
'localhost:80'
>>> req.host_url
'http://localhost'
>>> req.application_url
'http://localhost/blog'
>>> req.path_url
'http://localhost/blog/article'
>>> req.url
'http://localhost/blog/article?id=1'
>>> req.path
'/blog/article'
>>> req.path_qs
'/blog/article?id=1'
>>> req.query_string
'id=1'

req.POST

req.cookies

req.urlvars或者req.urlargs

{% endcodeblock %}
---
对于一个URL，由于我们需要处理PATH_INFO，所以还有以下几种使用：

    >>> req.path_info_peek() # Doesn't change request
    'article'
    >>> req.path_info_pop()  # Does change request!
    'article'
    >>> req.script_name
    '/blog/article'
    >>> req.path_info #上面已经pop了，所以这里是空，否则应该是'/article'的。
    ''

##Headers
一个请求的header主要是content的定义以及host的定义，可能还会带有一些认证信息。注意，这里的操作大小写敏感。

    >>> req.headers['Content-Type'] = 'application/x-www-urlencoded'
    >>> req.headers.items()
    [('Content-Length', '4'), ('Content-Type', 'application/x-www-urlencoded'), ('Host', 'localhost:80')]
    >>> req.environ['CONTENT_TYPE']
    'application/x-www-urlencoded'

##Query & POST 
请求可以在两个位置拥有变量，一个是query string（类似上面的?id=1）,二是在body里面（常用于POST表单），但是即便是POST请求也可以拥有query string。所以这两个位置都可能存在变量。并且一个变量可能出现多次，比如?check=a&check=b。

对于上面的东西，WebOb使用了多字典（MultiDict）的形式。所谓MultiDict,就在把一个包含键值对的列表包装成一个字典，看起来就像一个单值字典（single valued dict），但是你可以获得某一个key的所有值，通过.getall(key)(始终返回一个列表)，也可以通过.items()来获得所有的键值对以及.values()获得所有的值。举个例子：

    >>> req = Request.blank('/test?check=a&check=b&name=Bob')
    >>> req.GET
    MultiDict([(u'check', u'a'), (u'check', u'b'), (u'name', u'Bob')])
    >>> req.GET['check']
    u'b'
    >>> req.GET.getall('check')
    [u'a', u'b']
    >>> req.GET.items()
    [(u'check', u'a'), (u'check', u'b'), (u'name', u'Bob')]
    >>> req.GET.values()
    [u'a', u'b', u'Bob']

接下来，我们把req变成一个POST请求。无非就是做两个事，一是更改method，一是给出body。

    >>> req.POST
    <NoVars: Not a form request>
    >>> req.POST.items()  # NoVars can be read like a dict, but not written
    []
    >>> req.method = 'POST'
    >>> req.body = 'name=Joe&email=joe@example.com'
    >>> req.POST
    MultiDict([(u'name', u'Joe'), (u'email', u'joe@example.com')])
    >>> req.POST['name']
    u'Joe'

前面我们提到，一个请求的变量（也可以较为参数）可以存在于URL中也可以在Body中，如果我们不关系，变量的位置，那么用req.params就可以看到所有位置的变量，同样，也是通过MultiDict这种形式保存的。

    >>> req.params
    NestedMultiDict([(u'check', u'a'), (u'check', u'b'), (u'name', u'Bob'), (u'name', u'Joe'), (u'email', u'joe@example.com')])
    >>> req.params['name']
    u'Bob'
    >>> req.params.getall('name')
    [u'Bob', u'Joe']
    >>> for name, value in req.params.items():
    ...     print '%s: %r' % (name, value)
    check: u'a'
    check: u'b'
    name: u'Bob'
    name: u'Joe'
    email: u'joe@example.com'

相对而言，这种方式还很简单一点。至于GET和POST，命名是历史遗留下的。req.GET也可以使用在非GET的请求中，从而来获得参数。如下：

    >>> req = Request.blank('/test?check=a&check=b&name=Bob')
    >>> req.method = 'PUT'
    >>> req.body = body = 'var1=value1&var2=value2&rep=1&rep=2'
    >>> req.environ['CONTENT_LENGTH'] = str(len(req.body))
    >>> req.environ['CONTENT_TYPE'] = 'application/x-www-form-urlencoded'
    >>> req.GET
    MultiDict([(u'check', u'a'), (u'check', u'b'), (u'name', u'Bob')])
    >>> req.POST
    MultiDict([(u'var1', u'value1'), (u'var2', u'value2'), (u'rep', u'1'), (u'rep', u'2')])

另外，应用中，应该强制使用UTF-8编码，也是通过制定字符集参数。

    >>> req.charset = 'utf8'
    >>> req.GET
    MultiDict([(u'check', u'a'), (u'check', u'b'), (u'name', u'Bob')])

<http://docs.webob.org/en/latest/reference.html#introduction>
<http://docs.webob.org/en/latest/index.html>
    














