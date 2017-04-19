title: Django创建表单上传图片
date: 2016-08-31 09:38:40
tags:
	- Django
	- 表单
	- 图片上传
category: Python
comments: true
---


IOS开发中需要为创建的数据保存到网络后台长久存储，刚开始想到的是直接连接mysql，但要在ios中安装mysql的控件，实在是麻烦。于是定义一个restful接口，通过http请求的方式来上传和获取数据，是一种比较方便的方式。
本文是基于Django框架，实现以下几个功能：

- Model和ModelForm创建表单
- POST上传图片
 
 <!-- more -->
 ***
 
# 一、建立Model与mysql连接

## 1，定义model
```
# models.py

from django.db import models
from django.utils.timezone import now

# Create your models here.
class CureData(models.Model):
    STATUS_SIZES = (
        (0, '进行中'),
        (1, '已完成'),
    )

    name = models.CharField('名称', max_length=50)
    cureDuration = models.IntegerField('时长')
    create_at = models.DateTimeField("日期", default=now())
    note = models.CharField('备注', max_length=200, blank=True)
    image = models.ImageField('图片', upload_to='photos', blank=True)
    operator = models.CharField('操作者', max_length=50, blank=True)
    status = models.IntegerField('状态', default=0, choices=STATUS_SIZES) # 0,进行中; 1,已完成

    class Meta:
        ordering = ['create_at']

    def __unicode__(self):
        return self.name
```


## 2，配置数据库:
在工程的settings.py中设置database

```
# settings.py
# Database
# https://docs.djangoproject.com/en/1.9/ref/settings/#databases

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'mysite',
        'USER': 'user',
        'PASSWORD': 'yourpassword',
        'HOST': '127.0.0.1',
        'PORT': 3306,
    }
}
```
## 3，models.py同步到数据库
在shell中移到路径到当前工程根目录,执行命令:

```
python manage.py makemigrations mysite
python manage.py migrate
```

# 二、建立表单
## 1, forms.py

```
from django import forms
from models import CureData

class CureDataImageForm(forms.ModelForm):

    class Meta:
        model = CureData
        fields = '__all__'  # ['name', 'create_at',  ...] 
```

表单中有ImageField，需要在项目的settings.py中添加MEDIA_ROOT路径：

```
MEDIA_ROOT = './Data/media/'
```
model中定义的image的参数upload_to='photos'，上传的图片将保存至./Data/meida/photos/目录下。

## 2, views.py
```
from mysite.forms import CureDataImageForm

def update_data(request):
    if request.method == 'POST':

        form = CureDataImageForm(request.POST or None, request.FILES or None)
        if form.is_valid():
            image = form.save()
            print image.image.url
  
            return HttpResponseRedirect('/mysite/success/')
    else:
        form = CureDataImageForm()
    return render_to_response('mysite/data_form.html', {'form': form})
```

## 3, urls.py
```
# mysite/urls.py

from django.conf.urls import url, static
from . import views

urlpatterns = [

    url(r'^update_data/$', views.update_data),
    url(r'^success/$', views.success),
]
```

## 4, html文件
views.py中需要的两个html文件放在mysite/templates/mysite/目录下。
Form要支持上传图片，需要在form中设置 **enctype="multipart/form-data"**，不设置的话文件不支持上传。

**data_form.html**

```
<!-- data_form.html -->
<html>
<head>
    <title>data update</title>
</head>
<body>
    <h1>data update</h1>

    {% if form.errors %}
        <p style="color: red;">
            Please correct the error{{ form.errors|pluralize }} below.
        </p>
    {% endif %}

    <form action="" method="post", enctype="multipart/form-data">
        <table>
            {{ form.as_p }}
        </table>
        <input type="submit" value="Submit">
    </form>
</body>
</html>
```

**success.html**

```
<!-- success.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Success!</title>
</head>
<body>
<div>Success!</div>
</body>
</html>
```

# 三、另一种表单创建方式
上面创建表单是一种比较简洁的方式，如果不想所有字段保存到数据库，可以用另一种方式：
## 1，forms.py
```
from django import forms
from django.utils.timezone import now

class CureDataForm(forms.Form):
    name = forms.CharField(label="名称")
    cureDuration = forms.IntegerField(label="时长")
    create_at = forms.DateTimeField(label="创建时间", initial=now())
    note = forms.CharField(label="备注", required=False)
    image = forms.FileField(label="图片", required=False)
    operator = forms.CharField(label="操作者")
    status = forms.IntegerField(label="状态")
```

## 2, views.py
```
from mysite.forms import CureDataForm
def update_data(request):
    if request.method == 'POST':

        form = CureDataForm(request.POST or None, request.FILES or None)
        if form.is_valid():
            cd = form.cleaned_data
            print cd
            # img_url = form['image']
            # print img_url
            # 根据用户提交的注册信息在用户信息表中建立一个新的用户对象
            cureData = CureData.objects.create(
                name = form.cleaned_data['name'],
                cureDuration = form.cleaned_data['cureDuration'],
                create_at = form.cleaned_data['create_at'],
                note=form.cleaned_data['note'],
                image=form.cleaned_data['image'],
                operator=form.cleaned_data['operator'],
                status=form.cleaned_data['status'],
            )
            cureData.save()
            return HttpResponseRedirect('/bbcure/success/')
    else:
        form = CureDataForm()
    return render_to_response('bbcure/data_form.html', {'form': form})
```
# 四、效果

shell中执行:
```
manage.py runserver 0.0.0.0:8000
```
然后在浏览器中打开[http://0.0.0.0:8000/mysite/update_data/](http://0.0.0.0:8000/mysite/update_data/)

注：mysite为项目名称
![](http://7xo67b.com1.z0.glb.clouddn.com/2016-08-31/dj1.png)

提交后，数据库中如下：
![](http://7xo67b.com1.z0.glb.clouddn.com/2016-08-31/dj2.png)

# 五、总结
用Django创建站点很方便，只需很少的代码就能架构出一个功能很复杂的网页。本文只是冰山一角，从表单这一小块阐述Django的快速实现。







