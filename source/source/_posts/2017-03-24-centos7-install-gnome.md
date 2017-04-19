title: centos7安装VNC服务器
date: 2017-03-24 10:51:59
category: 系统
tags:
	- centos7
	- VNC
	- gnome
---
> centos7系统下的VNC服务器的中文安装教程多如牛毛，有些安装流程复杂但到最后却不成功，本人试验了不下10个教程，装的快要吐血😓。谷歌到这篇英文教程[How to install VNC server on Centos 7](http://www.krizna.com/centos/install-vnc-server-centos-7/)，发现是良心之作，操作简单可行。于是翻译之以饷读者。

VNC服务器用于从远程客户端连接到服务器的桌面环境。远程计算机上使用VNC客户端连接服务器。
在本文我们可以了解如何在centos 7上安装VNC服务器，将采用centos yum库中提供的默认包来安装。
<!-- more -->
# 安装 VNC服务器
如果你不曾安装过桌面环境（X windows），就按照以下命令来安装软件，重启后，你就会具有centos7的桌面。

```
[root@krizna ~]# yum check-update
[root@krizna ~]# yum groupinstall "X Window System"
[root@krizna ~]# yum install gnome-classic-session gnome-terminal nautilus-open-terminal control-center liberation-mono-fonts
[root@krizna ~]# unlink /etc/systemd/system/default.target
[root@krizna ~]# ln -sf /lib/systemd/system/graphical.target /etc/systemd/system/default.target
[root@krizna ~]# reboot
```

现在开始安装VCN包
**步骤1：**执行下面的命令安装VNC包
```
[root@krizna ~]# yum install tigervnc-server -y
```
**步骤2：**将/lib/systemd/system/vncserver@.service拷贝至/etc/systemd/system/，并重命名为vncserver@:1.service。
```
[root@krizna ~]# cp /lib/systemd/system/vncserver@.service /etc/systemd/system/vncserver@:1.service
```
**步骤3：**打开/etc/systemd/system/下的vncserver@:1.service，将<USER>替换为你的用户名。
找到这两行：
```
ExecStart=/sbin/runuser -l <USER> -c "/usr/bin/vncserver %i"
PIDFile=/home/<USER>/.vnc/%H%i.pid
```
替换为（假定用户名为john）：
```
ExecStart=/sbin/runuser -l john -c "/usr/bin/vncserver %i"
PIDFile=/home/john/.vnc/%H%i.pid
```

如果你是root用户，就这样替换：
```
ExecStart=/sbin/runuser -l root -c "/usr/bin/vncserver %i"
PIDFile=/root/.vnc/%H%i.pid
```

**步骤4：**重新加载systemd进行更改
```
[root@krizna ~]# systemctl daemon-reload
```

**步骤5：**创建VNC密码
```
[root@krizna ~]# vncpasswd
```

**步骤6：**启动服务，并设置开机自动运行
```
[root@krizna ~]# systemctl enable vncserver@:1.service
[root@krizna ~]# systemctl start vncserver@:1.service
```

**步骤7：**防火墙允许VNC访问

```
[root@krizna ~]# firewall-cmd --permanent --add-service vnc-server
[root@krizna ~]# systemctl restart firewalld.service
```

到这里，你就可以用VNC客户端连接服务器桌面了（192.168.11.165:1）。
![](http://7xo67b.com1.z0.glb.clouddn.com/2017-03-24/gnome.png)

> PS: VNC客户端各个系统版本都有，自行百度~。我这里用的是mac版的VNC Viewer。

对于其他用户，创建不同的端口文件vncserver@:2.service，参考步骤2，然后重复步骤3，4，5，6即可。

# 附加命令
- 停止VNC服务
```
[root@krizna ~]# systemctl stop vncserver@:1.service
```
- 取消开机自动运行
```
[root@krizna ~]# systemctl disable vncserver@:1.service
```
- 关闭防火墙（用于故障排除）
```
[root@krizna ~]# systemctl stop firewalld.service
```

好运~

翻译自: [How to install VNC server on Centos 7](http://www.krizna.com/centos/install-vnc-server-centos-7/)










