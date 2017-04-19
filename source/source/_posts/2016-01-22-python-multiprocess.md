---
layout: post
title: python多线程与多进程 超简单使用
date: 2016-01-22 15:47:07
tags: 
	- Python
	- multiprocessing
	- 多进程
	- 多线程
category: Python
comments: true
---
Python 的GIL限制了多核CPU的性能，对于IO密集型的程序，采用多线程能显著提高运行速度；但对于计算密集型的程序，多线程就没多少用了，采用多进程编程，就能充分利用多核CPU的性能，CPU占用率能达到100%。

* 下面是在阿里云服务器上测试的数据：

> 配置：CPU：Xeon, E5-2680, 2.5GHz, 4核;  内存：16G, DDR4; 硬盘：100G, SSD
<!-- more -->

```
def run():
    pool = multiprocessing.Pool(processes = 8)
    result = []
    contents = []
    for ix, row in df.iterrows():
        content = row['title'] + " " + row['content']
        contents.append(content)
    result = pool.map(get_one_article_keys, contents)
    pool.close()
    pool.join()
    t1 = time.time()
    print 'time pass:{:.3f}'.format(t1-t0)

def get_one_article_keys(content):
    try:
        tags2 = jieba.analyse.textrank(content, topK=20)
        print multiprocessing.current_process()
        return ','.join(tags2)
    except Exception,e:
        print "get_one_article_keys():%s" % str(e)
        return ''
```

执行计算密集型任务的结果：


| multi_way | Processes | Time(s) |
|:-:|:-:|:-:|
|多进程 | 4 | 378 |
|多进程 | 8 | 381 |
|多进程 | 20 | 464 |
|<font color=red>多线程</font> | 8 | 3174|



对于一台四核的机器，设置进程数为对应的内核数，效率是最高的；当设置比内核多的进程时，在创建python进程时开销占时比较多，造成设置为20个进程数时，时间比4个进程多了1分钟多。
**因此，设置进程数与实际内核数相同，运行最快。**

> 获取内核数: multiprocessing.cpu_count() 

# 一、多进程编程
## 1，map方式
```
import multiprocessing
import time

def do_something(i):
    time.sleep(i)
    print 'good:%d' % i
    return 'good:%d' % i

print multiprocessing.cpu_count()
pool = multiprocessing.Pool(processes = 4)
result = pool.map(do_something, range(10)）
pool.close()
pool.join()
for res in result:
    print res
```
result 可以得到多进程执行do_something返回的结果，为list
> 1，map方式为阻塞模式，主进程必须等待所有子进程执行完毕了才能继续；
> 2，还有一种方式为非阻塞模式，<font color=red>map_async</font>，主进程不等待子进程是否完毕，接着向下执行。

## 2，apply 方式
```
...
result = []
for i in range(10):
    result.append(pool.apply(do_something, (i,)))
...

```

> 同map_async一样，还有非阻塞模式 <font color=red>apply_async</font>。


# 二、多线程编程
多线程实现也非常简单，跟多进程基本一样，只是创建pool时略有不同。
```
from multiprocessing.dummy import Pool as ThreadPool
import time

def do_something(i):
    time.sleep(i)
    print 'good:%d' % i
    return 'good:%d' % i

pool = ThreadPool(processes = 4)
result = pool.map(do_something, range(10)）
pool.close()
pool.join()
for res in result:
    print res
```















