---
layout: post
title: IP代理池的Python实现
date: 2015-11-17 17:41:46
tags: 
	- Python
	- 爬虫 
	- IP代理
category: Python
comments: true
---

爬虫采集数据时，如果频繁的访问某个网站，会被封IP，有些是禁止访问3小时，有些是直接拉黑名单。为了避免被禁，一般采取的措施有三种：
> 1. 放慢抓取的速度，设置一个时间间隔；
> 2. 模拟浏览器行为，如采用Selenium + PhantomJS；
> 3. 设置IP代理，定期更换代理IP，让网站不认为来自一个IP。
<!-- more -->
本文实现其中的第三种方法。
国内提供IP代理的网站有很多，我们以其中的一个为例：<http://www.haodailiip.com> 
分为三步来实现这个IP抓取类：
> 1. 解析网页中的IP和端口
> 2. Ping所有IP地址的连接速度 
> 3. 按速度从快到慢排序，保存到文件



## 一、解析网页中的IP和端口
抓取网页采用的是<font color=red> urlib + BeautifulSoup</font>。
解析网站：<http://www.haodailiip.com/guonei/page>，page=1,2...,10
    
	def parse(url):
	        try:
	            page = urllib.urlopen(url)
	            data =  page.read()
	            soup = BeautifulSoup(data, "html5lib")
	            print soup.get_text()
	            body_data = soup.find('table', attrs={'class':'content_table'})
	            res_list = body_data.find_all('tr')
	            for res in res_list:
	                each_data = res.find_all('td')
	                if len(each_data) > 3 and not 'IP' in each_data[0].get_text() and '.' in each_data[0].get_text():
	                    print each_data[0].get_text().strip(), each_data[1].get_text().strip()
	                    item = IPItem()
	                    item.ip = each_data[0].get_text().strip()
	                    item.port = each_data[1].get_text().strip()
	                    item.addr = each_data[2].get_text().strip()
	                    item.tpye = each_data[3].get_text().strip()
	                    self.ip_items.append(item)
	        except Exception,e:
	            print e

BeautifulSoup默认的解析器是lxml，但对于这个网址，发现网页内容解析的不完整，于是用了解析性最好的 html5lib，速度上会稍慢。
关于BeautifulSoup解析器的介绍见<http://www.crummy.com/software/BeautifulSoup/bs4/doc.zh/#id9>。
BS解析的过程是：
 * 先找到table class="content_table"的标签；
 * 在从上面的内容中找所有tr
 * 我们需要的信息在tr的td中
 * 结果存入IPItem类。



**IPItem的定义**
       
	class IPItem:
	    def __init__(self):
	        self.ip = ''    # IP
	        self.port = ''  # Port
	        self.addr = ''  # 位置
	        self.tpye = ''  #类型:http; https
	        self.speed = -1 #速度
        
## 二、Ping所有IP地址的连接速度 

    import pexpect
    def test_ip_speed(ip_items):
        tmp_items = []
        for item in ip_items:
	
            (command_output, exitstatus) = pexpect.run("ping -c1 %s" % item.ip, timeout=5, withexitstatus=1)
            if exitstatus == 0:
                print command_output
                m = re.search("time=([\d\.]+)", command_output)
                if m:
                    print 'time=', m.group(1)
                    item.speed = float(m.group(1))
                    tmp_items.append(item)
	
       ip_items = tmp_items
 
主要是利用pexpect模块调用系统的ping命令，上面代码在mac 10.11.1下测试通过。

## 三、按速度从快到慢排序，保存至文件
保存至文件利用pandas模块，只需一句代码即可搞定。
 1. 先把ip_items转换成pandas的DataFrame；
 2. 排序，df.sort_index()，按'Speed'列排序；
 3. 结果写入Excel文件，to_excel()

    
	def save_data(self):
	        df = DataFrame({'IP':[item.ip for item in ip_items],
	                        'Port':[item.port for item in self.ip_items],
	                        'Addr':[item.addr for item in self.ip_items],
	                        'Type':[item.tpye for item in self.ip_items],
	                        'Speed':[item.speed for item in self.ip_items]
	                        }, columns=['IP', 'Port', 'Addr', 'Type', 'Speed'])
	        print df[:10]
	        df['Time'] = GetNowTime()
	        df = df.sort_index(by='Speed')
	
	        now_data = GetNowDate()
	
	
	        file_name = self.dir_path +'ip_proxy_' + now_data + '.xlsx'
	
	        df.to_excel(file_name)

生成的excel文件如下：
![results](http://7xo67b.com1.z0.glb.clouddn.com/ip_results.png)
                                                                                                                                                         


