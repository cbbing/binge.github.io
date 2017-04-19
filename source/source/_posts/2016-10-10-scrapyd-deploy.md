title: Scrapyd部署
date: 2016-10-10 11:10:30
category: Python
tags:
	- scrapy
	- scrapyd
	- 部署
---
scrapy爬虫写好后，需要用命令行运行，如果能在网页上操作就比较方便。scrapyd部署就是为了解决这个问题，能够在网页端查看正在执行的任务，也能新建爬虫任务，和终止爬虫任务，功能比较强大。

# 一、安装

## 1，安装scrapyd
```
pip install scrapyd
```

## 2， 安装 scrapyd-deploy
<!-- more -->
```
pip install scrapyd-client
```
windows系统，在c:\\python27\Scripts下生成的是**scrapyd-deploy**，无法直接在命令行里运行scrapd-deploy。
**解决办法：**
在c:\\python27\Scripts下新建一个scrapyd-deploy.bat，文件内容如下：
```
@echo off
C:\Python27\python C:\Python27\Scripts\scrapyd-deploy %*
```
添加环境变量：C:\Python27\Scripts;


# 二、使用

## 1，运行scrapyd
首先切换命令行路径到Scrapy项目的根目录下，
要执行以下的命令，需要先在命令行里执行scrapyd，将scrapyd运行起来

```
MacBook-Pro:~ usera$ scrapyd

/usr/local/bin/scrapyd:5: UserWarning: Module _markerlib was already imported from /Library/Python/2.7/site-packages/distribute-0.6.49-py2.7.egg/_markerlib/__init__.pyc, but /System/Library/Frameworks/Python.framework/Versions/2.7/Extras/lib/python is being added to sys.path
  from pkg_resources import load_entry_point
2016-09-24 16:00:21+0800 [-] Log opened.
2016-09-24 16:00:21+0800 [-] twistd 15.5.0 (/usr/bin/python 2.7.10) starting up.
2016-09-24 16:00:21+0800 [-] reactor class: twisted.internet.selectreactor.SelectReactor.
2016-09-24 16:00:21+0800 [-] Site starting on 6800
2016-09-24 16:00:21+0800 [-] Starting factory <twisted.web.server.Site instance at 0x102a21518>
2016-09-24 16:00:21+0800 [Launcher] Scrapyd 1.1.0 started: max_proc=16, runner='scrapyd.runner'
```
## 2，发布工程到scrapyd
### a，配置scrapy.cfg
在scrapy.cfg中，取消#url = http://localhost:6800/前面的“#”，具体如下：,
然后在命令行中切换命令至scrapy工程根目录，运行命令：
```
scrapyd-deploy <target> -p <project>
```
示例：
```
scrapd-deploy -p MySpider
```


* 验证是否发布成功

```
scrapyd-deploy -l

output:
TS                   http://localhost:6800/
```

# 一，开始使用
## 1，先启动 scrapyd，在命令行中执行：

```
MyMacBook-Pro:MySpiderProject user$ scrapyd
```
## 2，创建爬虫任务
```
curl http://localhost:6800/schedule.json -d project=myproject -d spider=spider2
```

* bug：
scrapyd deploy shows 0 spiders by scrapyd-client
scrapy中有的spider不出现，显示只有0个spiders。
* 解决
需要注释掉settings中的
```
# LOG_LEVEL = "ERROR"
# LOG_STDOUT = True
# LOG_FILE = "/tmp/spider.log"
# LOG_FORMAT = "%(asctime)s [%(name)s] %(levelname)s: %(message)s"
```
When setting LOG_STDOUT=True, scrapyd-deploy will return 'spiders: 0'. Because the output will be redirected to the file when execute 'scrapy list', like this: INFO:stdout:spider-name. Soget_spider_list can not parse it correctly.


## 3，查看爬虫任务
在网页中输入：http://localhost:6800/

下图为http://localhost:6800/jobs的内容：
![](http://7xo67b.com1.z0.glb.clouddn.com/2016-10-10/scrapyd.png)

## 4，运行配置
配置文件：C:\Python27\Lib\site-packages\scrapyd-1.1.0-py2.7.egg\scrapyd\default_scrapyd.conf

```
[scrapyd]
eggs_dir    = eggs
logs_dir    = logs
items_dir   = items
jobs_to_keep = 50
dbs_dir     = dbs
max_proc    = 0
max_proc_per_cpu = 4
finished_to_keep = 100
poll_interval = 5
http_port   = 6800
debug       = off
runner      = scrapyd.runner
application = scrapyd.app.application
launcher    = scrapyd.launcher.Launcher

[services]
schedule.json     = scrapyd.webservice.Schedule
cancel.json       = scrapyd.webservice.Cancel
addversion.json   = scrapyd.webservice.AddVersion
listprojects.json = scrapyd.webservice.ListProjects
listversions.json = scrapyd.webservice.ListVersions
listspiders.json  = scrapyd.webservice.ListSpiders
delproject.json   = scrapyd.webservice.DeleteProject
delversion.json   = scrapyd.webservice.DeleteVersion
listjobs.json     = scrapyd.webservice.ListJobs
```

参考
> http://www.cnblogs.com/jinhaolin/p/5033733.html
> https://scrapyd.readthedocs.io/en/latest/api.html#cancel-json

