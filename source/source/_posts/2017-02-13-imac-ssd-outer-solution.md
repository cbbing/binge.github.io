title: 自己动手，升级iMac——USB3.0 SSD固态硬盘解决方案
date: 2017-02-13 23:23:59
category: 系统
tags:
	- iMac
	- SSD
	- USB3.0
	- 移动硬盘盒
---
家里的iMac是2013年买的，曾记得当年是自己开发的一款android程序在中国移动APP大赛上取得前十，这台imac就是用这笔奖金买的。小小的回忆下，虽然这款app已经没有接着开发了，想当初也是投入了满腔的热血加入移动开发的大营^_^。

如今iMac升级了好几代，但硬盘居然还是传统的机械硬盘，当然也有FusionDrive和SSD，但需要加钱。
最新的imac除了显示器升级到视网膜屏外，其它的外观基本没什么变化，自家的老款imac仍然可以来装装*，除了运行速度越来越慢~~~
<!-- more -->
![](http://7xo67b.com1.z0.glb.clouddn.com/2017-02-13/imac1.png)

开机启动要个3-5分钟，打开chrome，docker上的chrome图标要跳个30s左右才能完全把chrome打开，体验速度已经到了无法忍受的地步了！
用DiskSpeedTest测试硬盘的读写速度：

![](http://7xo67b.com1.z0.glb.clouddn.com/2017-02-13/imac2.png)

5400转的SATA理论读写速度是90M/s，由于已经使用了3年多，实际读写只有理论的一半了。
在网上看过不少将内置机械硬盘更换为SSD硬盘的帖子，感觉难度挺大，而且容易毁掉屏幕，还可能导致风扇异常，不愿冒这个险。
参考帖子:
> http://blog.devtang.com/2014/01/26/add-ssd-to-old-imac/

于是寻找外置硬盘的解决方案：
apple的雷电接口速度比较快，可惜市面上很少有雷电口的SSD硬盘，幸好imac的USB接口是3.0的，USB3.0的最大传输带宽高达5.0Gbps（500MB/s），与SSD硬盘相比慢了一点，但速度比机械硬盘那快了不止一点半点。

参考[iMac 2015 5K 版外接 USB3.0 硬盘盒+SSD 系统加速体验](https://luolei.org/imac-5k-external-usb-ssd-update/)，从马云家采购来硬盘盒和SSD硬盘:
- Toshiba/东芝 OCZ饥饿鲨TR150 240G SSD固态硬盘
- 世特力裸族CSS25U3BK6G移动硬盘盒SSD固态硬盘盒 USB3.0 SATA3 6G
 
![](http://7xo67b.com1.z0.glb.clouddn.com/2017-02-13/imac3.png)

519+135，654块搞定！

移动硬盘+硬盘盒接到iMac后面是这个样子的：
![](http://7xo67b.com1.z0.glb.clouddn.com/2017-02-13/imac4.png)

用DiskSpeedTest测试，读写速度分别能达到264M/s和128M/s，比iMac机械硬盘的速度提升了5倍！
![](http://7xo67b.com1.z0.glb.clouddn.com/2017-02-13/imac5.png)


当然，还是不能跟内置的SSD硬盘相比，下面是macbook pro的硬盘测试结果：
![](http://7xo67b.com1.z0.glb.clouddn.com/2017-02-13/imac6.png)


因为imac的速度太慢，在家闲置了好久，换了这个SSD外置硬盘，速度又飞起来了，现在又可以启用了！





