---
layout: post
title: Scrapy结合Redis实现增量爬取
date: 2016-08-24 15:55:31
tags:
	- Scrapy
	- Redis

category: Python
comments: true	
---
Scrapy适合做全量爬取，但是，我们不是一次抓取完就完事了。很多情况，我们需要持续的跟进抓取的站点，增量抓取是最需要的。
Scrapy与Redis配合，在写入数据库之前，做唯一性过滤，实现增量爬取。
***
# 一、官方的去重Pipeline
官方文档中有一个去重的过滤器:
<!-- more -->

```
from scrapy.exceptions import DropItem

class DuplicatesPipeline(object):

    def __init__(self):
        self.ids_seen = set()

    def process_item(self, item, spider):
        if item['id'] in self.ids_seen:
            raise DropItem("Duplicate item found: %s" % item)
        else:
            self.ids_seen.add(item['id'])
            return item
```

官方的这个过滤器的缺陷是只能确保单次抓取不间断的情况下去重，因为其数据是保存在内存中的，当一个爬虫任务跑完后程序结束，内存就清理掉了。再次运行时就失效了。

# 二、基于Redis的去重Pipeline

为了能够多次爬取时去重，我们考虑用Redis，其快速的键值存取，对管道处理数据不会产生多少延时。

```
#pipelines.py

import pandas as pd
import redis
redis_db = redis.Redis(host=settings.REDIS_HOST, port=6379, db=4, password=settings.REDIS_PWD)
redis_data_dict = "f_uuids"

class DuplicatePipeline(object):
    """
    去重(redis)
    """

    def __init__(self):
        if redis_db.hlen(redis_data_dict) == 0:
            sql = "SELECT uuid FROM f_data"
            df = pd.read_sql(sql, engine)
            for uuid in df['uuid'].get_values():
                redis_db.hset(redis_data_dict, uuid, 0)

    def process_item(self, item, spider):

        if redis_db.hexists(redis_data_dict, item['uuid']):
             raise DropItem("Duplicate item found:%s" % item)

        return item
```

1.  首先，我们定义一个redis实例: redis_db和redis key：redis_data_dict。
2.  在DuplicatePipeline的初始化函数__init__()中，对redis的key值做了初始化。当然，这步不是必须的，你可以不用实现。
3. 在process_item函数中，判断redis的hash表中存在该值uuid，则为重复item。
至于redis中为什么没有用list而用hash？ 主要是因为速度，hash判断uuid是否存在比list快好几个数据级。
特别是uuid的数据达到100w+时，hash的hexists函数速度优势更明显。

最后别忘了在settings.py中加上：

```
# Configure item pipelines
# See http://scrapy.readthedocs.org/en/latest/topics/item-pipeline.html
ITEM_PIPELINES = {
    'fund_spider.pipelines.DuplicatePipeline': 200,
     #'fund_spider.pipelines.MySQLStorePipeline': 300,
}
```
# 三、总结
本文不是真正意义上的增量爬取，而只是在数据存储环节，对数据唯一性作了处理，当然，这样已经满足了大部分的需求。
后续我会实现不需要遍历所有的网页，判断抓取到所有最新的item，就停止抓取。敬请关注！



