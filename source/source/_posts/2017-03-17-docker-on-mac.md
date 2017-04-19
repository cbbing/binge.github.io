title: mac下安装docker
date: 2017-03-17 15:01:10
category: Docker
tags:
	- docker
	- mac
---
# 安装
系统要求：OS X EI Captian 10.11以上

docker默认是在linux下运行，要在mac下运行，需要安装linux的虚拟环境。好在docker官网提供了mac版的docker安装包。
在https://www.docker.com/docker-mac 下载Docker.img。
安装完成后，顶栏会出现
<!-- more -->
![](http://7xo67b.com1.z0.glb.clouddn.com/2017-03-17/dm1.png)

双击安装后，到终端查看：
docker version
```
[cbb@~]$ docker version
Client:
 Version:      17.03.0-ce
 API version:  1.26
 Go version:   go1.7.5
 Git commit:   60ccb22
 Built:        Thu Feb 23 10:40:59 2017
 OS/Arch:      darwin/amd64

Server:
 Version:      17.03.0-ce
 API version:  1.26 (minimum version 1.12)
 Go version:   go1.7.5
 Git commit:   3a232c8
 Built:        Tue Feb 28 07:52:04 2017
 OS/Arch:      linux/amd64
 Experimental: true
```
出现版本号即安装成功！

# 配置
可以设置开机启动
![](http://7xo67b.com1.z0.glb.clouddn.com/2017-03-17/dm2.png)

设置与宿主机的文件共享
![](http://7xo67b.com1.z0.glb.clouddn.com/2017-03-17/dm3.png)

设置CPU和内存大小，与VirtualBox一样。
![](http://7xo67b.com1.z0.glb.clouddn.com/2017-03-17/dm4.png)


