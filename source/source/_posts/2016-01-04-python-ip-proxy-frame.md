---
layout: post
title: IP代理池的实现框架(安装包)
date: 2016-01-04 15:58:00
tags: 
	- Python
	- 爬虫 
	- IP代理
category: Python
comments: true
---

上一篇 [IP代理池的实现](http://kekefund.com/2015/11/17/pytho-ip-proxy/) 讲解了IP代理池的实现细节。

由于爬虫多个项目都需要用到IP代理，打造一个公用的IP代理库就很有必要。本文主要讲解公用的IP代理库的实现框架。

> 实现思路如下：
> 1，数据抓取：从各个IP代理网站抓取大量IP数据；
> 2，数据筛选：Ping每个IP，连接速度<1.5s的IP地址入库；
> 3，定时更新：设置定时任务，每日重新Ping数据库内的IP，更新连接速度；
<!-- more -->
> 4，定时新增：设置定时任务，每日定时从IP代理网站取新数据
> 5，提供获取接口



# 1，数据抓取

## 定义IPItem

```

class IPItem:
    def __init__(self):
        self.ip = ''    # IP
        self.port = ''  # Port
        self.addr = ''  # 位置
        self.type = ''  # 类型:http, https
        self.anonymous = '' # 匿名度
        self.speed = -1 #速度
        self.source = ''
        self.create_time = ''
        self.update_time = ''
```

## 解析多个代理网站的IP，返回IPItem列表

```

# parse ip web
def parse(self):

    ip_items_haodaili =self.parse_haodaili()
    ip_items_kuaidaili = self.parse_kuaidaili()
    ip_items_xici = self.parse_xici()
    ip_items_66 = self.parse_66ip()

    ip_items = []
    ip_items.extend(ip_items_haodaili)
    ip_items.extend(ip_items_kuaidaili)
    ip_items.extend(ip_items_xici)
    ip_items.extend(ip_items_66)

    return ip_items
```

# 2，数据筛选

多线程批量刷新ip_items，只保留ping速度在1.5s以内的ip_item。

```

def test_ip_speed(self, ip_items):

    #多线程
    pool = ThreadPool(processes=20)
    pool.map(self.ping_one_ip, ip_items)
    pool.close()
    pool.join()

    
    ip_items = [item for item in ip_items if item.speed >=0 and item.speed < 1500.0]    # 超时1.5s以内   
```

其中，ping_one_ip()函数在上一篇文章[IP代理池的实现](http://kekefund.com/2015/11/17/pytho-ip-proxy/)中有介绍。






# 3，定时更新
```
def check_useful_in_db(self):

    # 更新数据库中的IP Speed
    def update_ip_speed_to_db(ip_item):
        print ip_item.get_info()
        self.ping_one_ip(ip_item)
        print ip_item.get_info()
        sql = "update {table} set Speed={speed}, UpdateTime='{update_time}' " \
                "where IP='{ip}' and Port='{port}'".format(
                    table=mysql_table_ip,
                    speed=ip_item.speed,
                    update_time=GetNowTime(),
                    ip=ip_item.ip,
                    port=ip_item.port)
        print engine.execute(sql)


    ip_items = []

    df = pd.read_sql_table(mysql_table_ip, engine)
    for ix, row in df.iterrows():
        print type(ix), type(row)
        print ix, row
        ip_item = IPItem()
        ip_item.init_from_series(row)
        ip_items.append(ip_item)

    #多线程
    pool = ThreadPool(processes=10)
    pool.map(update_ip_speed_to_db, ip_items)
    pool.close()
    pool.join()

    print 'update speed success!'
```

# 4，定时新增
```
def run_add():

    ip_items = self.parse() # 获取ip代理网站数据
    print 'test speed begin...'
    test_ip_speed(ip_items)
    print 'test speed end'

    save_data(ip_items)
```

# 5，获取IP代理库的接口(对外)
```
# 获取IP代理地址
def get_ip_proxy(count=100, result_in_DataFrame = False):
    sql = 'select IP, Port, Type from {0} where Speed > 0 order by Speed limit {1}'.format(mysql_table_ip, count)
    df = pd.read_sql_query(sql, engine)
    if result_in_DataFrame:
        return df
    else:
        return df[['IP', 'Port', 'Type']].get_values()
```

下载：[安装包](http://7xo67b.com1.z0.glb.clouddn.com/IpProxy-1.0.tar.gz)

## 使用方法：

### 1，安装

```

$ tar -zxvf IpProxy-1.0.tar.gz

$ cd IpProxy-1.0

$ sudo python setup.py install

```

### 2，使用

```

In [6]: from ip_proxy import get_ip_proxy



In [7]: get_ip_proxy(count=10)

array([[u'123.56.177.156', u'8080', u'http'],

       [u'101.200.202.168', u'80', u'http'],

       [u'101.200.202.168', u'80', u'http'],

       [u'101.201.221.168', u'20151', u'http'],

       [u'101.201.221.168', u'20151', u'http'],

       [u'101.201.221.168', u'20151', u'http'],

       [u'114.112.103.21', u'3128', u'http'],

       [u'101.200.234.114', u'8080', u'http'],

       [u'114.112.103.21', u'3128', u'http'],

       [u'114.112.103.21', u'3128', u'http']], dtype=object)



n [8]: get_ip_proxy(count=10, result_in_DataFrame=True)

                IP   Port  Type

0   123.56.177.156   8080  http

1  101.200.202.168     80  http

2  101.200.202.168     80  http

3  101.201.221.168  20151  http

4  101.201.221.168  20151  http

5  101.201.221.168  20151  http

6   114.112.103.21   3128  http

7  101.200.234.114   8080  http

8   114.112.103.21   3128  http

9   114.112.103.21   3128  http

```