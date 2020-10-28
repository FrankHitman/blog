---
title: flask中自定义exception的使用
lang: zh-CN
date: 2018-05-28 16:45:01
categories:
    - python
tags: flask
---

- 参考资料[官方文档](http://flask.pocoo.org/docs/1.0/patterns/apierrors/)
- 需要解决的问题是：在业务逻辑复杂的情况下，简单的http response 的status_code无法满足api的需求时候，例如http code为400的时候需要包装“关联失败”，“删除需要解绑某些资源”等的code和msg到response中。为了避免每一个api都要重复写，我们把这部分代码提取出来做成Exception，并且定义handler注册到flask，让框架自动替我们做这件事。
<!--more-->
- 自定义exception
```python
class ApiException(Exception):
status_code = 400

def __init__(self, code=None, msg=None, data=None, code_context=None):
	self.code = code or ErrorCode.NETWORK_UNKNOWN_ERROR
	self.msg = msg or TRANSLATIONS.get(self.code, '')
	self.data = data or {}
	self.code_context = code_context or {}
```
code为用户自定义的code，用于返回自定义的错误码，msg传一些自定义的消息。自定义的异常返回的http的code统一为400。
- 为了避免返回给用户默认的服务器出错，需要自定义exception handler，用于处理自定义的异常。flask会自动捕捉exception，并把它转给handler处理，返回response。
```python
from flask import jsonify
from common.exc import ErrorCode

def handle_api_error(error):
	r_dict = {
		'code': error.code or ErrorCode.NETWORK_UNKNOWN_ERROR,
		'msg': error.msg or '',
		'data': error.data,
		'code_context': error.code_context,
	}
	response = jsonify(r_dict)
	response.status_code = error.status_code
	return response
```
- 自定义的exception handler需要注册到Flask的实例上，便于flask处理用户自定义的异常。这里有两种方法，第一种是在异常处理的方法前加上装饰器` @app.errorhandler(CustomException) `；第二种是调用app注册方法`app.register_error_handler(ApiException, handle_api_error)`
-  在API中需要的地方raise出自定义错误，框架会自动捕捉。
``` 
	raise CustomException('This view is gone', status_code=410)
```
或者定义个方法，在需要的地方引用。
```python
def raise_api_error(code=None, msg=None, data=None, code_context=None):
	raise ApiException(code, msg, data, code_context)
```


