title: ireport在centos7下生成pdf缺失字体问题解决
date: 2017-02-06 15:55:11
category: Java
tags:
	- ireport
	- centos7
	- 字体
---

在mac和window测试环境下调试均无问题，但部署到centos下生成报告时报如下错误：
```
11:07:19.093 ERROR c.d.f.report.controller.ProductDetailController - 生成报告失败，
失败原因:Font 'STZhongsong' is not available to the JVM. See the Javadoc for more details.
```
按照错误提示，把STZHONGS.ttf字体复制到centos的fonts中，按照[如何给CentOS安装字体库](http://www.cnblogs.com/xiaodiejinghong/p/4013454.html)中的方法，
<!-- more -->
```
mkfontscale

mkfontdir

fc-cache -fv

```

执行完成后，还是不行。


通过fc-list命令查看，发现

```

...

/usr/share/fonts/TTF/STZHONGS.ttf: 华文中宋,STZhongsong:style=Regular
...
```
系统中安装的是Regular样式的字体，而pdf用到的是STSong-Light字体。
***
来个暴力的方式，看看能不能解决。将window的fonts文件夹下的所有字体拷贝到centos的fonts下。
再执行
```
mkfontscale

mkfontdir

fc-cache -fv

```

重启tomcat，一键生成报告，pdf正常了！
pdf截图：
![](http://7xo67b.com1.z0.glb.clouddn.com/2017-02-06/ireport-pdf.png)


