title: docker版Django
date: 2017-03-30 17:14:05
category: Docker
tags:
	- docker
	- Django
	- Dockerfile
---
Django的运行是基于python的环境，加上django包。在docker中运行django，实现方式是从docker下载python镜像，然后安装django运行所依赖的包。

在https://store.docker.com/images/python?tab=description  中介绍pull镜像方式有一种叫python:onbuild。
这种镜像创建方式根据项目中提供的requirements.txt文件自动pip安装依赖包。大多数情况，通过python:onbuild能创建一个满足工程所需的独立镜像。
<!-- more -->
# 一、编写requirements.txt 
下述的目录结构是一个Django Rest Framework例子，其中项目名称为restful，app名称为api。

![](http://7xo67b.com1.z0.glb.clouddn.com/2017-03-30/dd1.png)

首先我们需要把项目所依赖的包放到requirements.txt中：
```
Django==1.8
django-bootstrap-toolkit==2.15.0
django-filter==1.0.1
djangorestframework==3.5.4
djangorestframework-jwt==1.10.0
pandas==0.19.2
SQLAlchemy==1.1.4
MySQL-python==1.2.5
```
# 二、编写Dockerfile
本文是基于python2.7制作的，Dockerfile文件如下：
```
FROM python:2-onbuild
CMD [ "python", "./manage.py", "runserver", "0.0.0.0:8000"]
```
CMD命令执行Django启动程序，0.0.0.0是对所有IP开放，监听端口8000。
需要说明的是CMD中的每个参数得单独分开，像这样"runserver 0.0.0.0:8000"是运行不成功的。

# 三、构建镜像
- $ docker build -t my-python-app .
```
[cbb@number_api]$ docker build -t number_api_django:0.3 .
Sending build context to Docker daemon 655.9 kB
Step 1/2 : FROM python:2-onbuild
# Executing 3 build triggers...
Step 1/1 : COPY requirements.txt /usr/src/app/
Step 1/1 : RUN pip install --no-cache-dir -r requirements.txt
 ---> Running in 4711187b3011
Collecting Django==1.8 (from -r requirements.txt (line 2))
  Downloading Django-1.8-py2.py3-none-any.whl (6.2MB)
Collecting django-bootstrap-toolkit==2.15.0 (from -r requirements.txt (line 3))
  Downloading django-bootstrap-toolkit-2.15.0.tar.gz
Collecting django-filter==1.0.1 (from -r requirements.txt (line 4))
  Downloading django_filter-1.0.1-py2.py3-none-any.whl (54kB)
Collecting djangorestframework==3.5.4 (from -r requirements.txt (line 5))
  Downloading djangorestframework-3.5.4-py2.py3-none-any.whl (709kB)
Collecting djangorestframework-jwt==1.10.0 (from -r requirements.txt (line 6))
  Downloading djangorestframework_jwt-1.10.0-py2.py3-none-any.whl
Collecting pandas==0.19.2 (from -r requirements.txt (line 7))
  Downloading pandas-0.19.2-cp27-cp27mu-manylinux1_x86_64.whl (17.2MB)
Collecting SQLAlchemy==1.1.4 (from -r requirements.txt (line 8))
  Downloading SQLAlchemy-1.1.4.tar.gz (5.1MB)
Collecting MySQL-python==1.2.5 (from -r requirements.txt (line 9))
  Downloading MySQL-python-1.2.5.zip (108kB)
Collecting PyJWT<2.0.0,>=1.4.0 (from djangorestframework-jwt==1.10.0->-r requirements.txt (line 6))
  Downloading PyJWT-1.4.2-py2.py3-none-any.whl
Collecting pytz>=2011k (from pandas==0.19.2->-r requirements.txt (line 7))
  Downloading pytz-2016.10-py2.py3-none-any.whl (483kB)
Collecting numpy>=1.7.0 (from pandas==0.19.2->-r requirements.txt (line 7))
  Downloading numpy-1.12.1-cp27-cp27mu-manylinux1_x86_64.whl (16.5MB)
Collecting python-dateutil (from pandas==0.19.2->-r requirements.txt (line 7))
  Downloading python_dateutil-2.6.0-py2.py3-none-any.whl (194kB)
Requirement already satisfied: six>=1.5 in /usr/local/lib/python2.7/site-packages (from python-dateutil->pandas==0.19.2->-r requirements.txt (line 7))
Installing collected packages: Django, django-bootstrap-toolkit, django-filter, djangorestframework, PyJWT, djangorestframework-jwt, pytz, numpy, python-dateutil, pandas, SQLAlchemy, MySQL-python
  Running setup.py install for django-bootstrap-toolkit: started
    Running setup.py install for django-bootstrap-toolkit: finished with status 'done'
  Running setup.py install for SQLAlchemy: started
    Running setup.py install for SQLAlchemy: finished with status 'done'
  Running setup.py install for MySQL-python: started
    Running setup.py install for MySQL-python: finished with status 'done'
Successfully installed Django-1.8 MySQL-python-1.2.5 PyJWT-1.4.2 SQLAlchemy-1.1.4 django-bootstrap-toolkit-2.15.0 django-filter-1.0.1 djangorestframework-3.5.4 djangorestframework-jwt-1.10.0 numpy-1.12.1 pandas-0.19.2 python-dateutil-2.6.0 pytz-2016.10
Step 1/1 : COPY . /usr/src/app
 ---> 712a54b6b923
Removing intermediate container df33c056f7c0
Removing intermediate container 4711187b3011
Removing intermediate container 6220af43bf96
Step 2/2 : CMD python ./manage.py runserver 0.0.0.0:8000
 ---> Running in 53c0cf32d840
 ---> 17c97bc704d9
Removing intermediate container 53c0cf32d840
Successfully built 17c97bc704d9

[cbb@number_api]$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
number_api_django   0.3                 17c97bc704d9        23 seconds ago      868 MB
```
这样就成功创建了镜像number_api_django:0.3

# 四、运行容器
- docker run
```
[cbb@number_api]$ docker run -it --rm -p 8080:8000 --name api1 number_api_django:0.3
Performing system checks...

System check identified no issues (0 silenced).
March 30, 2017 - 07:34:03
Django version 1.8, using settings 'restful.settings'
Starting development server at http://0.0.0.0:8000/
Quit the server with CONTROL-C.
```
这样就启动了django程序。


