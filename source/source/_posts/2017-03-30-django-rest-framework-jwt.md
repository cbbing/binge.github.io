title: Django Rest Framework 通过token访问
date: 2017-03-30 11:47:57
category: Python
tags:
	- Django
	- DjangoRestFramework
	- API
---


在web apps上实现身份验证时，首先考虑到的解决方案就是Cookie。基于Cookie的身份验证使用服务器端Cookie来对每个请求进行身份验证，这意味着您需要在数据库中（如Redis）保留一个会话存储。

基于token令牌的身份验证是一个最近比较流行的解决方案，它依赖于每个请求发送到服务器的签名令牌，对于移动端和网页端都比较适用。

# 一、安装

需先安装django rest framework
<!-- more -->

```
pip install djangorestframework-jwt

```



# 二、使用
setting.py
```
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': (
        'rest_framework.permissions.IsAuthenticated',
    ),
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_jwt.authentication.JSONWebTokenAuthentication',
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.BasicAuthentication',
    ),
}
```


工程的urls.py
```
from rest_framework_jwt.views import obtain_jwt_token
#...

urlpatterns = [
    '',
    # ...

    url(r'^api-token-auth/', obtain_jwt_token),
]
```


# 三、python 访问
## 1, 获得令牌
```
n [41]: import requests

In [42]: r = requests.post("http://localhost:8000/api-token-auth/", data={'username':'cbb','password':'cbb'})

In [43]: print r.text
{"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImNiYiIsInVzZXJfaWQiOjEsImVtYWlsIjoiY2JiaW5nQDE2My5jb20iLCJleHAiOjE0OTA4MjI5MDF9.k5fznq2RoEnsIIFYvc-afHLKXiEyfZyHjRwV8-db5FM"}
```
注意，post请求需要网址最后带“/”，django默认自动补全是关闭的。
> 网址不带/的出错提示： You called this URL via POST, but the URL doesn't end in a slash and you have APPEND_SLASH set. Django can't redirect to the slash URL while maintaining POST data. Change your form to point to 127.0.0.1:8000/api/users/ (note the trailing slash), or set APPEND_SLASH=False in your Django settings.

```
In [44]: import json

In [45]: jsonData = json.loads(r.text)

In [46]: jsonData[u'token']
Out[46]: u'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImNiYiIsInVzZXJfaWQiOjEsImVtYWlsIjoiY2JiaW5nQDE2My5jb20iLCJleHAiOjE0OTA4MjI5MDF9.k5fznq2RoEnsIIFYvc-afHLKXiEyfZyHjRwV8-db5FM'
```

## 2，通过令牌访问
- 构造headers
```
In [48]: headers = {'Authorization': 'JWT {}'.format(jsonData[u'token'])}

In [49]: headers
Out[49]: {'Authorization': 'JWT eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImNiYiIsInVzZXJfaWQiOjEsImVtYWlsIjoiY2JiaW5nQDE2My5jb20iLCJleHAiOjE0OTA4MjI5MDF9.k5fznq2RoEnsIIFYvc-afHLKXiEyfZyHjRwV8-db5FM'}
```
- get请求
```
In [50]: r = requests.get("http://127.0.0.1:8000/api/users?format=json", headers=headers)

In [51]: print r.text
[{"name":"cbb","birthday":"2017-03-07","gender":1},{"name":"xx","birthday":"2017-03-01","gender":2},{"name":"keke","birthday":"2016-03-01","gender":2},{"name":"小小","birthday":"2017-03-07","gender":2},{"name":"小小1","birthday":"2017-03-07","gender":2},{"name":"小小1","birthday":"2017-03-07","gender":2},{"name":"xx","birthday":"2017-03-07","gender":1},{"name":"ckk","birthday":"2017-03-22","gender":2},{"name":"ckk","birthday":"2017-03-22","gender":2},{"name":"ckkk","birthday":"2017-03-09","gender":2},{"name":"cbb","birthday":"2017-03-07","gender":1},{"name":"cbb","birthday":"2017-03-07","gender":1}]
```

- post请求
```
In [54]: data = {"name":"xx","birthday":"2017-03-01","gender":2}

In [55]: r = requests.post("http://127.0.0.1:8000/api/users/",data=data, headers=headers)

In [56]: print r.text
"{\"birthdayNumberCount\": {\"1\": 4, \"2\": 1, \"3\": 1, \"4\": 2, \"5\": 3, \"6\": 0, \"7\": 1, \"8\": 0, \"9\": 0}, \"suitableJob\": \"公众人物、开发商、投机者、设计师、新闻工作(媒体)、表演者、变革推动者、广告创意人才、探险家、心灵导师、作家、自由职业\", \"destinyNumber\": 5, \"destinyMean\": \"很注重感观享受，喜欢冒险、自由，个性开朗，人缘好；有口才，社交能力强，拥有演说和促销的天才。不容易离婚，爱美。\", \"destinyDetailMean\": \"因4的能力充斥内在，更需要职业的稳定来协助创造力与变动，不然就会形成外强中干，而无法让自己身心自由。\", \"birthdayNumbers\": [2, 0, 1, 7, 0, 3, 0, 1, 1, 4, 5], \"toLearn\": \"节制自由，学习承诺与勇气。\", \"userInfo\": {\"username\": \"xx\", \"gender\": 2, \"birthday\": \"2017-03-01\"}, \"talentNumbers\": [14]}"
```
# 四、进阶
自定义令牌有效期，如设置有效期为5小时，在setting.py中添加
```
JWT_AUTH = {
    'JWT_EXPIRATION_DELTA': datetime.timedelta(hours=5),  #seconds=300
}
```
官方文档中所有能自定义的参数如下：
```
JWT_AUTH = {
    'JWT_ENCODE_HANDLER':
    'rest_framework_jwt.utils.jwt_encode_handler',

    'JWT_DECODE_HANDLER':
    'rest_framework_jwt.utils.jwt_decode_handler',

    'JWT_PAYLOAD_HANDLER':
    'rest_framework_jwt.utils.jwt_payload_handler',

    'JWT_PAYLOAD_GET_USER_ID_HANDLER':
    'rest_framework_jwt.utils.jwt_get_user_id_from_payload_handler',

    'JWT_RESPONSE_PAYLOAD_HANDLER':
    'rest_framework_jwt.utils.jwt_response_payload_handler',

    'JWT_SECRET_KEY': settings.SECRET_KEY,
    'JWT_GET_USER_SECRET_KEY': None,
    'JWT_PUBLIC_KEY': None,
    'JWT_PRIVATE_KEY': None,
    'JWT_ALGORITHM': 'HS256',
    'JWT_VERIFY': True,
    'JWT_VERIFY_EXPIRATION': True,
    'JWT_LEEWAY': 0,
    'JWT_EXPIRATION_DELTA': datetime.timedelta(seconds=300),
    'JWT_AUDIENCE': None,
    'JWT_ISSUER': None,

    'JWT_ALLOW_REFRESH': False,
    'JWT_REFRESH_EXPIRATION_DELTA': datetime.timedelta(days=7),

    'JWT_AUTH_HEADER_PREFIX': 'JWT',
    'JWT_AUTH_COOKIE': None,

}
```
# 参考
> https://github.com/GetBlimp/django-rest-framework-jwt
> http://getblimp.github.io/django-rest-framework-jwt/#requirements
> http://stackoverflow.com/questions/21317899/how-do-i-create-a-login-api-using-django-rest-framework

