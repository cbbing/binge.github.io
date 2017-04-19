title: docker部署mysql
date: 2017-03-19 23:54:44
category: Docker
tags:
	- docker
	- mysql
---
mysql在linux服务器上运行一直比较稳定，但是服务器迁移时mysql在新服务器上的配置是个比较头疼的问题，搞不好数据迁移过来了但是mysql启动不起来，坑比较多。特别是当新的服务器是离线时，安装mysql和数据同步软件更是困难重重。
用docker来运行mysql服务是一个比较好的解决方案，mysql的运行环境在容器内已经封装好了，而数据可以直接挂载在宿主主机上。

# 一、下载镜像
<!-- more -->
官网地址：https://hub.docker.com/_/mysql/

```
docker pull mysql
```
查看镜像
```
[root@VM_25_5_centos ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/mysql     latest              22be5748ecbe        13 days ago         405.6 MB
```
# 二、启动容器
```
$ docker run --name cbb-mysql1 -p 3307:3306 -v /home/mysql_data:/var/lib/mysql --restart=always -e MYSQL_ROOT_PASSWORD=123456 -d <IMAGE-ID>
b344e219ff03a92d65f75f74ab5b227838cce8619cbe695ccd1b6889f9a3d174
```

* -p：容器的3306映射到主机的3307端口
* -v：容器的/var/lib/mysql目录挂载在主机的/home/mysql_data目录
* -e 设置默认参数，支持参数：    
    * MYSQL_ROOT_PASSWORD
    * MYSQL_DATABASE
    * MYSQL_USER, MYSQL_PASSWORD
    * MYSQL_ALLOW_EMPTY_PASSWORD
    * MYSQL_RANDOM_ROOT_PASSWORD
    * MYSQL_ONETIME_PASSWORD
    
  
> 参考：https://hub.docker.com/_/mysql/ 的环境参数部分（Environment Variables)
 
返回一长串字符，则说明创建成功。
注：<IMAGE-ID>也可以是REPOSITORY+TAG，如docker.io/mysql: latest

# 三、进入容器
```
[root@VM_200_249_centos mysql-docker]# docker exec -it cbb-mysql1 mysql -uroot -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.17 MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

# 四、实践
离线数据库mysql目录下所有文件拷贝到离线服务器上
放到指定目录：/home/mysql_data
执行命令，建立mysql容器
```
[localhost mysql]# docker run --name cbb-mysql1 -v /home/mysql_data:/var/lib/mysql -p 3307:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.6
629a150d4a9cb87080df3d89a6b91e3d56ddd8699e2f6a2d3a908f39d6f87e4c
```
## 1, 进入容器，新建账户
```
mysql> use mysql;
mysql> create USER 'ts01'@'%' IDENTIFIED BY '123456';
```
说明：

- username：你将创建的用户名

- host：指定该用户在哪个主机上可以登陆，如果是本地用户可用localhost，如果想让该用户可以从任意远程主机登陆，可以使用通配符%

- password：该用户的登陆密码，密码可以为空，如果为空则该用户可以不需要密码登陆服务器

## 2, 授权:

命令:
```
GRANT privileges ON databasename.tablename TO 'username'@'host'
```
说明:

- privileges：用户的操作权限，如SELECT，INSERT，UPDATE等，如果要授予所的权限则使用ALL

- databasename：数据库名

- tablename：表名，如果要授予该用户对所有数据库和表的相应操作权限则可用*表示，如*.*

**例子:**

```
GRANT SELECT, INSERT ON test.user TO 'pig'@'%';
GRANT ALL ON *.* TO 'pig'@'%';
```

**注意:**

用以上命令授权的用户不能给其它用户授权，如果想让该用户可以授权，用以下命令:

```
GRANT privileges ON databasename.tablename TO 'username'@'host' WITH GRANT OPTION;
```

##  3，容器间连接
其它容器想访问mysql容器，可以使用link来连接。例如运行：
```
docker run -d -p 80:8096 -v /home/gtdata:/gtdata --restart=always --name web01 --link=cbb-mysql1:mydb java_web:1.0 /root/run.sh
```

注：
* -p 端口映射
* -v 宿主机目录挂载
* --name 容器名
* --link 连接的容器，另一个容器cbb-mysql在本容器中的名称为mydb，可以直接在本容器中使用mydb

java_web:1.0容器里运行的是tomcat。
进入jdbc.properties
修改
```
# mydb是mysql容器在web01容器中的别名，mydb等价于192.168.11.121:3306
jdbc.url=jdbc:mysql://mydb:3306/guotai?useUnicode=true&characterEncoding=utf-8
```

# 其它
> 1, [宿主服务器上安装VNC] (http://www.krizna.com/centos/install-vnc-server-centos-7/)
> 2, [centos版workbench](https://dev.mysql.com/downloads/file/?id=468286)


