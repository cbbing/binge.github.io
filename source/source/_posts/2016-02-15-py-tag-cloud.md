---
layout: post
title: python 制作标签云
date: 2016-02-15 09:19:38
tags:
	- Python
	- pytagcloud
	- 标签云
category: Python
comments: true
---
标签云是比较直观的频率分布表现方式，很多网站和APP在年度盘点和总结时会使用。Python生成标签云有一个比较易用的库 pytagcloud。

![](http://7xo67b.com1.z0.glb.clouddn.com/2016-02-10-finance.png-540x360)
<!-- more -->


# 1，导入头文件

```
from pytagcloud import create_tag_image, make_tags
from pytagcloud.lang.counter import get_tag_counts
```



# 2，生成标签云

```
def finance_cloud():

      tag = 'cc xx xx china cc keke keke keke'
      tags = make_tags(get_tag_counts(tag),maxsize=100)
      # Set your output filename
      create_tag_image(tags,"cloud.png", size=(1280,800),background=(0, 0, 0, 255), fontname="SimHei")


finance_cloud()
```

生成的图片cloud.png可以指定尺寸size，设置背景background，指定字体fontname。

pytagcloud库默认的字体不支持中文，生成的图片中，中文是乱码。

解决办法是在py文件开始处指定图片输出的字体：

```
from pylab import mpl
mpl.rcParams['font.sans-serif'] = ['SimHei']#['FangSong'] # 指定默认字体
mpl.rcParams['axes.unicode_minus'] = False # 解决保存图像是负号'-'显示为方块的问题
```

# 3，字体名称

## Windows的字体对应名称
黑体 	SimHei 
微软雅黑 	Microsoft YaHei 
微软正黑体 	Microsoft JhengHei 
新宋体 	NSimSun 
新细明体 	PMingLiU 
细明体 	MingLiU 
标楷体 	DFKai-SB 
仿宋 	FangSong 
楷体 	KaiTi 
仿宋_GB2312 	FangSong_GB2312 
楷体_GB2312 	KaiTi_GB2312 

宋体：SimSuncss中中文字体（font-family）的英文名称 
新細明體：PMingLiU 
細明體：MingLiU 
標楷體：DFKai-SB 
黑体：SimHei 
新宋体：NSimSun 
仿宋：FangSong 
楷体：KaiTi 
仿宋_GB2312：FangSong_GB2312 
楷体_GB2312：KaiTi_GB2312 
微軟正黑體：Microsoft JhengHei 
微软雅黑体：Microsoft YaHei 
装Office会生出来的一些： 
隶书：LiSu 
幼圆：YouYuan 
华文细黑：STXihei 
华文楷体：STKaiti 
华文宋体：STSong 
华文中宋：STZhongsong 
华文仿宋：STFangsong 
方正舒体：FZShuTi 
方正姚体：FZYaoti 
华文彩云：STCaiyun 
华文琥珀：STHupo 
华文隶书：STLiti 
华文行楷：STXingkai 
华文新魏：STXinwei

## Mac OS的字体名称： 
华文细黑：STHeiti Light [STXihei] 
华文黑体：STHeiti 
华文楷体：STKaiti 
华文宋体：STSong 
华文仿宋：STFangsong 
儷黑 Pro：LiHei Pro Medium 
儷宋 Pro：LiSong Pro Light 
標楷體：BiauKai 
蘋果儷中黑：Apple LiGothic Medium 
蘋果儷細宋：Apple LiSung Light 


> 参考：http://www.it610.com/article/2569995.htm

