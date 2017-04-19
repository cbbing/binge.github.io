title: docker 自动构建，基于Dockerfile文件
date: 2017-03-06 12:48:22
category: Docker
tags:
	- docker
	- dockerfile
---
# 1，Dockerfile的编写
在centos中创建一个目录:/mydata/data/dockertest/，新建Dockerfile文件
vim Dockerfile
```
# Verison 0.6:

# 基础镜像
FROM chenbb/fofeasy:0.6

# 维护者信息
MAINTAINER cbbing@163.com

# 镜像操作命令
RUN rm -rf /opt/tomcat/webapps/fofeasy
RUN rm -rf /opt/tomcat/webapps/fofeasy.war

ADD fofeasy.war /opt/tomcat/webapps/fofeasy.war

# 容器启动命令
#CMD ["/opt/tomcat/bin/catalina.sh", "run"]
```
编写完成后:wq保存。
<!-- more -->

# 2，构建
基于Dockerfile构建镜像，在Dockerfile文件所在目录下执行
```
[root@VM_200_249_centos dockertest]# docker build -t chenbb/fofeasy:0.7 .
Sending build context to Docker daemon 65.78 MB
Step 1 : FROM chenbb/fofeasy:0.6
 ---> c441af7f5405
Step 2 : MAINTAINER cbbing@163.com
 ---> Running in f7cbd5cd3199
 ---> cef4cee90997
Removing intermediate container f7cbd5cd3199
Step 3 : RUN rm -rf /opt/tomcat/webapps/fofeasy
 ---> Running in 79505ed64d7f
 ---> 4f85be099a20
Removing intermediate container 79505ed64d7f
Step 4 : RUN rm -rf /opt/tomcat/webapps/fofeasy.war
 ---> Running in be162f93530b
 ---> c5cc2ba60023
Removing intermediate container be162f93530b
Step 5 : ADD fofeasy.war /opt/tomcat/webapps/fofeasy.war
 ---> 8ede3a4f83e5
Removing intermediate container b9b557e26828
Successfully built 8ede3a4f83e5
[root@VM_200_249_centos dockertest]#
```
注：
> chenbb/fofeasy:0.7为新镜像的名字
> fofeasy.war文件放到同一目录
 
```
[root@VM_200_249_centos dockertest]# ll -lh
总用量 63M
-rw-r--r-- 1 root root 322 3月   3 17:00 Dockerfile
-rw-r--r-- 1 root root 63M 3月   3 16:46 fofeasy.war
```

# 3，启动
```
docker run -d -p 58080:8080 --name javaweb chenbb/javaweb:0.7 /root/run.sh
```

# 3，一些问题
* 容器启动不起来
考虑是容器里的命令执行报错引起的，重新从镜像创建容器，排除问题，或者通过
"docker logs <容器ID>" 查看错误日志

# 参考
> http://www.jianshu.com/p/690844302df5
