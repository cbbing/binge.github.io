---
layout: post
title: requests初步使用
date: 2016-01-07 16:52:33
tags: 
	- Python
	- requests 
category: Python
comments: true
---



# 基本用法

## 一、发送无参数的get请求

```

import requests

In [67]: r =requests.get('http://httpbin.org/get')

In [68]: print r.text
{
  "args": {}, 
  "headers": {
    "Accept": "*/*", 
    "Accept-Encoding": "gzip, deflate", 
    "Host": "httpbin.org", 
    "User-Agent": "python-requests/2.7.0 CPython/2.7.10 Darwin/14.5.0"
  }, 
  "origin": "220.231.47.169", 
  "url": "http://httpbin.org/get"
}
```
<!-- more -->
返回一个名为 r 的Response对象。可以从这个对象中获取所有我们想要的信息。


## 二、发送带参数的get请求

将key与value放入一个字典中，通过params来传递，其作用相当于urllib.urlencode

```

In [69]: pyqload = {'q':'cbb'}
In [70]: r = requests.get('http://www.so.com/s', params=pqyload, , timeout=10) # 10s
In [71]: r.url
Out[71]: u'http://www.haosou.com/s?q=%E7%AB%A0%E6%A5%A0%E6%A5%A0'
```

> 注意字典里值为 None 的键都不会被添加到 URL 的查询字符串里。


## 三、发送post请求，通过data参数来传递

```

In [72]: payload = {'a':'嘎子', 'b':'hello'}
In [73]: r = requests.post('http://httpbin.org/post', data=payload)
In [74]: print r.text
{
  "args": {}, 
  "data": "", 
  "files": {}, 
  "form": {
    "a": "\u560e\u5b50", 
    "b": "hello"
  }, 
  "headers": {
    "Accept": "*/*", 
    "Accept-Encoding": "gzip, deflate", 
    "Content-Length": "28", 
    "Content-Type": "application/x-www-form-urlencoded", 
    "Host": "httpbin.org", 
    "User-Agent": "python-requests/2.7.0 CPython/2.7.10 Darwin/14.5.0"
  }, 
  "json": null, 
  "origin": "220.231.47.169", 
  "url": "http://httpbin.org/post"
}
```
data不仅可以接受字典类型的数据，还可以接受json等格式：

```

In [75]: import json
In [76]: r = requests.post('http://httpbin.org/post', data=json.dumps(payload))
In [77]: print r.text
{
  "args": {}, 
  "data": "{\"a\": \"\\u560e\\u5b50\", \"b\": \"hello\"}", 
  "files": {}, 
  "form": {}, 
  "headers": {
    "Accept": "*/*", 
    "Accept-Encoding": "gzip, deflate", 
    "Content-Length": "35", 
    "Host": "httpbin.org", 
    "User-Agent": "python-requests/2.7.0 CPython/2.7.10 Darwin/14.5.0"
  }, 
  "json": {
    "a": "\u560e\u5b50", 
    "b": "hello"
  }, 
  "origin": "220.231.47.169", 
  "url": "http://httpbin.org/post"
}
```
可以看出，json参数时直接存为字符串保存为data字段，而字典类型参数放在form表单中。



## 四、发送文件的post类型， 向网站上传一张图片，文档等

```

In [78]: url = 'http://httpbin.org/post'
In [79]: files = {'file':open('touxiang.png','rb')}
In [80]: r = requests.post(url, files=files)
```

## 五、编码

Requests会自动解码来自服务器的内容。大多数unicode字符集都能被无缝地解码

请求发出后，Requests会基于HTTP头部对响应的编码作出有根据的推测。当你访问 r.text 之时，Requests会使用其推测的文本编码。你可以找出Requests使用了什么编码，并且能够使用r.encoding 属性来改变它:

```

In [11]: r = requests.get('http://www.baidu.com')



In [12]: r.encoding

Out[12]: 'utf-8'



In [18]: r.encoding = 'GBK'

```

## 六、保存图片：二进制响应内容

对于非文本请求，以字节的方式访问请求响应体。Requests会自动为你解码gzip和deflate传输编码的响应数据。

```

In[25]: from PIL import Image

In[26]: from StringIO import StringIO

In[27]: r = requests.get('http://7xo67b.com1.z0.glb.clouddn.com/1449027323885.jpg')

In[28]: i = Image.open(StringIO(r.content))

In[30]: i.show()

```

显示图片:

![刘楚恬](http://7xo67b.com1.z0.glb.clouddn.com/1449027323885.jpg)



## 七、JSON响应内容

Requests中也有一个内置的JSON解码器，助你处理JSON数据:

```


In[31]: r = requests.get('https://github.com/timeline.json')

In[32]: r.json()

Out[32]: 

{u'documentation_url': u'https://developer.github.com/v3/activity/events/#list-public-events',

 u'message': u'Hello there, wayfaring stranger. If you\u2019re reading this then you probably didn\u2019t see our blog post a couple of years back announcing that this API would go away: http://git.io/17AROg Fear not, you should be able to get what you need from the shiny new Events API instead.'}

```

如果JSON解码失败， r.json 就会抛出一个异常。例如，相应内容是 401 (Unauthorized) ，尝试访问 r.json 将会抛出 ValueError: No JSON object could be decoded 异常。



## 八、响应状态码

```

In[40]: r = requests.get('http://httpbin.org/get')

In[41]: r.status_code

Out[41]: 200

```

## 九、Cookies

获取cookies

```

In[49]: r.cookies

Out[49]: <RequestsCookieJar[]>

```

发送cookies到服务器

```

url = 'http://httpbin.org/cookies'

In[45]: cookies = dict(cookies_are='working')

In[47]:  r = requests.get(url, cookies=cookies)

In[48]: r.text

Out[48]: u'{\n  "cookies": {\n    "cookies_are": "working"\n  }\n}\n'

```

## 十、错误与异常

遇到网络问题（如：DNS查询失败、拒绝连接等）时，Requests会抛出一个 ConnectionError 异常。

遇到罕见的无效HTTP响应时，Requests则会抛出一个 HTTPError 异常。

若请求超时，则抛出一个 Timeout 异常。

若请求超过了设定的最大重定向次数，则会抛出一个 TooManyRedirects 异常。

所有Requests显式抛出的异常都继承自 requests.exceptions.RequestException 。



# 高级用法

## 一、会话对象

会话对象让你能够跨请求保持某些参数。它也会在同一个Session实例发出的所有请求之间保持cookies。

### 跨请求保留一些cookies

```


In[50]: s = requests.Session()

In[51]: s.get('http://httpbin.org/cookies/set/sessioncookie/123456789')

Out[51]: <Response [200]>

In[52]: r = s.get('http://httpbin.org/cookies')

In[53]: r.text

Out[53]: u'{\n  "cookies": {\n    "sessioncookie": "123456789"\n  }\n}\n'

```

### 会话也可用来为请求方法提供缺省数据。这是通过为会话对象的属性提供数据来实现的。

```


s = requests.Session()
s.auth = ('user', 'pass')
s.headers.update({'x-test': 'true'})

# both 'x-test' and 'x-test2' are sent
s.get('http://httpbin.org/headers', headers={'x-test2': 'true'})
```



## 二、代理

```

import requests

proxies = {
  "http": "http://10.10.1.10:3128",
  "https": "http://10.10.1.10:1080",
}

requests.get("http://example.org", proxies=proxies)
```

你也可以通过环境变量 HTTP_PROXY 和 HTTPS_PROXY 来配置代理。

```

$ export HTTP_PROXY="http://10.10.1.10:3128"
$ export HTTPS_PROXY="http://10.10.1.10:1080"
$ python
>>> import requests
>>> requests.get("http://example.org")
```

若你的代理需要使用HTTP Basic Auth，可以使用 http://user:password@host/ 语法:

```

proxies = {
    "http": "http://user:pass@10.10.1.10:3128/",
}
```








# 参考

> http://www.yangyanxing.com/?p=1079

> http://docs.python-requests.org/zh_CN/latest/user/advanced.html#advanced





















