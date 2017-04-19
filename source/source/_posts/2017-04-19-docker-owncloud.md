title: docker版私人网盘ownCloud
date: 2017-04-19 12:37:04
tags: 
	- Docker
	- ownCloud
	- 网盘
categories: Docker
---
ownCloud是一个自由且开源的个人云存储解决方案。ownCloud在客户端可通过网页界面，或者安装专用的客户端软件来使用。网页界面当然就是任何能开网页的平台都支持，而客户端软件也支持相当多平台，Windows、Linux、iOS、Android皆有。
除了云存储之外，ownCloud也可用于同步日历、电子邮件联系人、网页浏览器的书签；此外还有多人在线文件同步协作的功能（类似google documents或Duddle等等）。
ownCloud官方提供了Docker版的[ownCloud](https://store.docker.com/images/owncloud?tab=description)，部署安装能一步到位。
<!-- more -->

# 如何使用Docker
## 开始使用
直接运行：
```
$ docker run -d -p 80:80 owncloud:8.1
```
然后进入 [http://localhost/](http://localhost/)，根据向导配置。默认情况下使用SQLite作为数据储存。对于MySQL数据库，可以通过容器连接，例如:--link my-mysql:mysql。
## 数据持久化
所有的数据在数据库中管理，数据保存在/var/www/html。可以通过以下命令对容器的数据卷和宿主机的数据卷映射。
```
-v /<mydatalocation>:/var/www/html
```
对于更细粒度的数据持久，设置如下的命令：
```
-v /<mydatalocation>/apps:/var/www/html/apps installed / modified apps
-v /<mydatalocation>/config:/var/www/html/config local configuration
-v /<mydatalocation>/data:/var/www/html/data the actual data of your ownCloud
```
# 通过docker-compose
ownCloud的docker-compose.yml示例如下：
```
# ownCloud with MariaDB/MySQL
#
# Access via "http://localhost:8080" (or "http://$(docker-machine ip):8080" if using docker-machine)
#
# During initial ownCloud setup, select "Storage & database" --> "Configure the database" --> "MySQL/MariaDB"
# Database user: root
# Database password: example
# Database name: pick any name
# Database host: replace "localhost" with "mysql"

version: '2'

services:

  owncloud:
    image: owncloud
    volumes:
      - "/mydata/code/ownCloud/ownData:/var/www/html"
    ports:
      - 8021:80

  mysql:
    image: mysql:5.6
    volumes:
        - "/mydata/code/ownCloud/mysqldata:/var/lib/mysql"
    ports:
      - 3308:3306

    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: ownCloud
      MYSQL_USER: abc
      MYSQL_PASSWORD: 123456
```
## 创建
```
$ docker-compose up
```
## 查看状态
```
[root@VM_25_5_centos ownCloud]# docker-compose ps
      Name             Command             State              Ports
-------------------------------------------------------------------------
owncloud_mysql_1   docker-            Up                 3306/tcp
                   entrypoint.sh
                   mysqld
owncloud_ownclou   /entrypoint.sh     Up                 0.0.0.0:8021->80
d_1                apache2-for ...                       /tcp
```
## 删除
```
[root@VM_25_5_centos ownCloud]# docker-compose down
Stopping owncloud_owncloud_1 ... done
Stopping owncloud_mysql_1 ... done
Removing owncloud_owncloud_1 ... done
Removing owncloud_mysql_1 ... done
Removing network owncloud_default
```
# ownCloud配置
进入 http://localhost:8021/ , 出现页面：
![](http://7xo67b.com1.z0.glb.clouddn.com/2017-04-19/own1.png)
添加用户和数据库信息：
![](http://7xo67b.com1.z0.glb.clouddn.com/2017-04-19/own2.png)

注意：红框内的数据库地址为docker-compose.yml中mysql的名称。
点击“安装完成”！
网页版登录：
![](http://7xo67b.com1.z0.glb.clouddn.com/2017-04-19/own3.png)
ownCloud支持windows，mac桌面端，ios/android手机端。基本可以替代在线网盘如百度网盘等。

