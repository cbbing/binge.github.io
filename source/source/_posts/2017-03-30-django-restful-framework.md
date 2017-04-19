title: Django Rest Framwork实现RESTful API
date: 2017-03-30 10:46:25
category: Python
tags:
	- Django
	- DjangoRestFramework
	- API
---
# 安装
```
pip install djangorestframework
pip install markdown # Markdown为可视化 API 提供了支持
pip install django-filter
```
<!-- more -->
# 创建工程
![](http://7xo67b.com1.z0.glb.clouddn.com/2017-03-30/drf1.png)

工程名：restful
app名：api
IDE：PyCharm


# 配置rest_framework

```

"setting.py"



...

# Application definition

INSTALLED_APPS = (
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    # 新增
    'api', 
    'rest_framework',

)

# 新增
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': ('rest_framework.permissions.IsAdminUser',),
    'PAGINATE_BY': 10
}
...
```


# 配置数据库

数据库采用mysql

```

"setting.py"



DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'restful',
        'USER': 'admin',
        'PASSWORD': '123',
        'HOST': '127.0.0.1',
        'PORT': 3306,
    }
}
```

# 建立模型
```
"models.py"

#coding:utf-8

from django.db import models

# Create your models here.

class User(models.Model):

    GENDER_CHOICES = (
        (1, "Male"),
        (2, "Female")
    )
    name = models.CharField(max_length=60, blank=False, verbose_name='姓名')
    birthday = models.DateField(blank=False)
    gender = models.IntegerField(choices=GENDER_CHOICES)

    def __unicode__(self):
        return self.name + " ( " + str(self.birthday) + ")"
```

# 同步数据库
```
python manage.py makemigrations
python manage.py migrate
```


# 序列化

在api下新建serializers.py

```

"serializers.py"



#coding:utf-8
from rest_framework import serializers
from models import User, NumberologyInfo, OtherInfo

class UserSerializer(serializers.ModelSerializer):
  
    class Meta:
        model = User

        fields = ('name', 'birthday', 'gender')
```

# 添加视图
```
"views.py"

#coding:utf-8

from django.shortcuts import render

# Create your views here.
from django.shortcuts import render

# Create your views here.
from rest_framework import viewsets
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.serializers import Serializer

from rest_framework import generics, permissions
from .models import User, NumberologyInfo
from .serializers import UserSerializer, NumberologyInfoSerializer

class UserViewSet(viewsets.ModelViewSet):
    """
    允许查看和编辑user 的 API endpoint
    """
    queryset = User.objects.all()
    serializer_class = UserSerializer
```

## 创建视图的三种方式
```
"views.py"

# 第一种方式：APIView
class TaskList(APIView):

    def get(self, request, format=None):
        users = User.objects.all()
        serializer = UserSerializer(users, many=True)
        return Response(serializer.data)

    def post(self, request, format=None):
        serializer = UserSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        else:
            return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

# 第二种方式：通用视图 ListCreateAPIView
class TaskListCreate(generics.ListCreateAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer


# 第三种方式：装饰器 api_view
@api_view(['GET', 'POST'])
def task_list(request):
    '''
    List all tasks, or create a new task.
    '''
    if request.method == 'GET':
        tasks = User.objects.all()
        serializer = UserSerializer(tasks, many=True)
        return Response(serializer.data)
    elif request.method == 'POST':
        serializer = UserSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        else:
            return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

# 设置url
```
"urls.py"

#coding:utf-8
from django.conf.urls import patterns, url, include
from rest_framework import routers
from api import views

router = routers.DefaultRouter()
router.register(r'users', views.UserViewSet)


# Wire up our API using automatic URL routing.
# Additionally, we include login URLs for the browseable API.

urlpatterns = [
    url(r'^', include(router.urls)),

    #验证登录使用
    url(r'auth',include('rest_framework.urls')),

    ]
```

# 启动服务

![](http://7xo67b.com1.z0.glb.clouddn.com/2017-03-30/drf2.png)

![](http://7xo67b.com1.z0.glb.clouddn.com/2017-03-30/drf3.png)








# 参考

> 1，[django-rest-framework 系列教程（一）- Start Your API](http://www.jianshu.com/p/653a0a5684eb)

> 2，[Django RESTful API 设计指南](http://www.cnblogs.com/Edifier-7/p/4994338.html)

> 3，[利用 Django REST framework 编写 RESTful API](https://blog.laisky.com/p/django-rest/)

> 4，[用Django Rest Framework和AngularJS开始你的项目](http://jingpin.jikexueyuan.com/article/56178.html)

> 5，[Django Rest Framework 入门指南](http://www.jianshu.com/p/943eae36f708)

> 6，[django-rest-framework里的api请求频率控制](http://me.iblogc.com/2016/12/17/django-rest-framework里的api请求频率控制/)

> 7，[ 验证和授权](https://github.com/thehackercat/django-rest-framework-tutorial/blob/master/4%20-%20%E9%AA%8C%E8%AF%81%E5%92%8C%E6%8E%88%E6%9D%83.md)
