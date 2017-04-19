title: docker版FTP服务器
date: 2017-03-20 21:52:43
category: Docker
tags:
	- docker
	- ftp
---
docker版ftp服务器，适用于部署离线局域网服务器
# 下载
来源：https://hub.docker.com/r/bogem/ftp/
```
[root@VM_25_5_centos mydata]# docker pull bogem/ftp

[root@VM_25_5_centos mydata]# docker images
REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
docker.io/bogem/ftp   latest              a40e9c43c530        4 weeks ago         174.7 MB
```
<!-- more -->

# 运行
```
docker run -d -v /mydata:/home/vsftpd -p 20:20 -p 21:21 
-p 47400-47470:47400-47470 \
-e FTP_USER=test 
-e FTP_PASS=test 
-e PASV_ADDRESS=0.0.0.0 
--name ftp1 \
--restart=always bogem/ftp
```

注：PASV_ADDRESS如果设置成127.0.0.1，则只能本地访问；设置成0.0.0.0 则可以外网访问。
详细参数参考:https://hub.docker.com/r/fauria/vsftpd/ 中的环境变量部分。


# FileZilla客户端连接
![](http://7xo67b.com1.z0.glb.clouddn.com/2017-03-20/dockerftp.png)
