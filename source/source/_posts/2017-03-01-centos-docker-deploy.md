title: centos + docker部署web java项目
date: 2017-03-01 00:01:59
category: Docker
tags:
	- centos
	- docker
	- tomcat
---
# 1，系统准备
CentOS 具体要求如下：
- 必须是 64 位操作系统
- 建议内核在 3.8 以上

通过以下命令查看您的 CentOS 内核：
```
[root@VM_200_249_centos ~]# uname -r
3.10.0-327.36.3.el7.x86_64
```
对于 CentOS 6.5 而言，内核版本默认是 2.6。首先，可通过以下命令安装最新内核：
<!-- more -->
```
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -ivh http://www.elrepo.org/elrepo-release-6-5.el6.elrepo.noarch.rpm
yum -y --enablerepo=elrepo-kernel install kernel-lt
```

随后，编辑以下配置文件：
```
vi /etc/grub.conf
```
将default=1修改为default=0。
最后，通过reboot或shutdown now命令重启操作系统。
重启后如果不出意外的话，再次查看内核，您的 CentOS 内核将会显示为 3.10。


# 2，安装Docker
只需通过以下命令即可安装 Docker 软件：
```
rpm -Uvh http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm
yum -y install docker-io
```

可通过以下命令启动 Docker 服务：

```
service docker start
chkconfig docker on # 设置开机启动

```

可使用以下命令，查看 Docker 是否安装成功：
```
[root@VM_200_249_centos ~]# docker version
Client:
 Version:      1.12.3
 API version:  1.24
 Go version:   go1.6.3
 Git commit:   6b644ec
 Built:
 OS/Arch:      linux/amd64

Server:
 Version:      1.12.3
 API version:  1.24
 Go version:   go1.6.3
 Git commit:   6b644ec
 Built:
 OS/Arch:      linux/amd64
```
若输出了 Docker 的版本号，则说明安装成功，我们下面就可以开始使用 Docker 了。





# 3， 安装镜像
[Docker 官网](https://www.docker.com/) 提供了所有的镜像[下载地址](https://registry.hub.docker.com/search?q=library)。（需要VPN翻墙）
直接pull下来(https://hub.docker.com/r/nimmis/java-centos/)
```
docker pull nimmis/java-centos
```
可以通过对应的标签选择不同的jdk版本，例如"docker pull nimmis/java-centos:openjdk-7-jdk"
- latest - currently Oracle Java version 8 JRE
- openjdk-7-jdk - OpenJDK Java version 7 JDK
- openjdk-7-jre - OpenJDK Java version 7 JRE
- openjdk-7-jre-headless - OpenJDK Java version 7 JRE headless
- openjdk-8-jdk - OpenJDK Java version 8 JDK
- openjdk-8-jre - OpenJDK Java version 8 JRE
- openjdk-8-jre-headless - OpenJDK Java version 8 JRE headless
- oracle-7-jre - Oracle Java version 7 JRE
- oracle-7-jdk - Oracle Java version 7 JDK
- oracle-8-jre - Oracle Java version 8 JRE
- oracle-8-jdk - Oracle Java version 8 JDK


## 镜像加速器
docker官网的镜像下载非常慢，国内提供了Docker镜像的下载点，如阿里、网易和DaoCloud。以阿里云为例：
需要先注册阿里云账号，进到
```
https://cr.console.aliyun.com/#/accelerator
```
选择左侧“加速器”，找到你的专属加速器地址
![](http://7xo67b.com1.z0.glb.clouddn.com/2017-03-01/dockermirror.png)
在centos下修改/etc/docker/daemon.json文件，添加：
```
{
    "registry-mirrors": ["https://yxz1pr3x.mirror.aliyuncs.com"]
}
```
设置后能获得每秒1兆的下载速度。


最后，使用以下命令查看本地所有的镜像：
```
[root@VM_200_249_centos ~]# docker images
REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
dockerxman/docker-centos     latest              4c89ecb22b17        2 minutes ago       395.6 MB
daocloud.io/library/centos   latest              67591570dd29        10 weeks ago        191.8 MB
```
可以看到，系统中有2个docker镜像，“dockerxman/docker-centos”，也可以
称其为仓库（Repository），镜像的标签（Tag）为lastest，此外还有镜像ID（IMAGE ID），大小有395兆。

# 4，启动容器
容器在镜像的基础上运行，一旦容器启动了，我们就可以登录到容器中，安装自己所需的软件或应用程序。
```
docker run -i -t -v /mydata/data:/mnt/software --restart=always 67591570dd29 /bin/bash
```

## 命令解释：
```
docker run <相关参数> <镜像 ID> <初始命令>
```
其中，相关参数包括：
* -i：表示以“交互模式”运行容器
* -t：表示容器启动后会进入其命令行
* -v：表示需要将本地哪个目录挂载到容器中，格式：-v <宿主机目录>:<容器目录>

初始命令表示一旦容器启动，需要运行的命令，此时使用“/bin/bash”，表示什么也不做，只需进入命令行即可。
需要说明的是，不一定要使用“镜像 ID”，也可以使用“仓库名:标签名”，例如：dockerxman/docker-centos:latest。

## 查看所有创建的容器
```
docker ps -a
```

## 查看正在运行的容器
```
[root@VM_200_249_centos ~]# docker ps
CONTAINER ID        IMAGE                COMMAND             CREATED             STATUS              PORTS                     NAMES
a197e23e0ddb        asy:0.1   "/root/run.sh"      24 hours ago        Up 24 hours         0.0.0.0:58080->8080/tcp   javaweb
```

## 启动容器
```
docker start <CONTAINER ID>
```

## 停止容器
```
docker stop <CONTAINER ID>
```

## 重新进入已创建的容器
```
docker attach <CONTAINER ID>
```
或
```
docker exec -it <CONTAINER ID> /bin/bash
```
> http://blog.csdn.net/u010397369/article/details/41045251


## 删除容器
```
docker rm <CONTAINER ID>
```
## 删除镜像
```
docker rmi <IMAGE ID>
```
注：需要先把镜像生成的容器全部删除才能删掉镜像。


# 5，安装软件
由于我们选择的镜像是包含JDK的，所以我们只需要安装tomcat。
tomcat我放在服务器上，用wget下载到容器中。
```
wget http://kekefund.com/soft/apache-tomcat-7.0.63.tar.gz
```

将tomcat放到/opt目录下，先移到到/opt目录，然后解压
```
cd /opt
tar -zxf /mnt/software/apache-tomcat-7.0.63.tar.gz -C .
```
重命名
```
mv apache-tomcat-7.0.63/ tomcat/
```
 
# 6，运行脚本
创建运行脚本：vi /root/run.sh
```
#!/bin/bash
source ~/.bashrc
sh /opt/tomcat/bin/catalina.sh run
```
为运行脚本添加执行权限
```
chmod u+x /root/run.sh
```

# 7, 创建 Java Web 镜像
使用以下命令，根据某个“容器 ID”来创建一个新的“镜像”：
```
docker commit d50f5048e212 chenbb/javaweb:0.1
```
该容器的 ID 是“ d50f5048e212”，所创建的镜像名是“ chenbb/javaweb:0.1”，随后可使用镜像来启动 Java Web 容器。

# 8, 启动 Java Web 容器
通过docker images查看所有镜像
```
[root@VM_200_249_centos ~]# docker images
REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
chenbb/javaweb               0.1                 99e35759d5ed        24 hours ago        700 MB
dockerxman/docker-centos     latest              4c89ecb22b17        26 hours ago        395.6 MB
daocloud.io/library/centos   latest              67591570dd29        10 weeks ago        191.8 MB
```

可见，此时已经看到了最新创建的镜像“ chenbb/javaweb:0.1”，其镜像 ID 是“ 99e35759d5ed”。正如上面所描述的那样，我们可以通过“镜像名”或“镜像 ID”来启动容器，与上次启动容器不同的是，我们现在不再进入容器的命令行，而是直接启动容器内部的 Tomcat 服务。此时，需要使用以下命令：
```
docker run -d -p 58080:8080 --name javaweb --restart=always chenbb/javaweb:0.1 /root/run.sh
```
参数介绍：
* -d：表示以“守护模式”执行/root/run.sh脚本，此时 Tomcat 控制台不会出现在输出终端上。
* -p：表示宿主机与容器的端口映射，此时将容器内部的 8080 端口映射为宿主机的 58080 端口，这样就向外界暴露了 58080 端口，可通过 Docker 网桥来访问容器内部的 8080 端口了。
* --name：表示容器名称，用一个有意义的名称命名即可。


# 9，浏览器查看
在浏览器中，输入以下地址，即可访问 Tomcat 首页：
http://192.168.1.124:58080/

# 10，镜像打包
打包后就可以移植到其他主机上运行了。
打包：
```
docker save chenbb/fofeasy:0.1 > /mydata/data/fofeasy0.1.tar
```
# 11，在另外的主机上导入镜像
```
docker load < fofeasy0.1.tar   #导入镜像
docker images   #查看存在的镜像
```



# 参考
> 1，[迈出使用Docker的第一步，学习第一个Docker容器](http://www.jianshu.com/p/26f15063de7d)
>  2，[使用 Docker 搭建 Java Web 运行环境](https://my.oschina.net/huangyong/blog/372491)
> 3，[使用 Docker 运行 Tomcat ＋ WAR 包 Java 应用](http://docs.daocloud.io/java-docker/docker-tomcat-war-java)
> 4，[docker的安装以及jdk和tomcat的环境配置](http://bbs.csdn.net/topics/390844669)
> 5，[运维人员的解放----Docker快速部署](http://yangrong.blog.51cto.com/6945369/1551327)




