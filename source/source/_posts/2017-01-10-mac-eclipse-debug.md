title: mac版 eclipse 只能run不能debug的解决方案
date: 2017-01-10 11:07:36
category: Java
tags:
	- mac
	- eclipse
	- debug
---
# mac版 eclipse 只能run不能debug的解决方案
mac：macOS Sierra 10.12.2
eclipse：Version: Neon.1a Release (4.6.1)

debug时进度一直停在93%，然后超时报错：
<!-- more -->
![](http://7xo67b.com1.z0.glb.clouddn.com/20170110/17011001.png)


```
ERROR: transport error 202: gethostbyname: unknown host
ERROR: JDWP Transport dt_socket failed to initialize, TRANSPORT_INIT(510)
JDWP exit error AGENT_ERROR_TRANSPORT_INIT(197): No transports initialized [debugInit.c:750]
FATAL ERROR in native method: JDWP No transports initialized, jvmtiError=AGENT_ERROR_TRANSPORT_INIT(197)
```

分析错误提示，是找不到主机host，google一下，在stackoverflow找到了解决方案，在hosts中加入
```
127.0.0.1	localhost
```

-hosts修改方法：
hosts不能直接在/etc中修改；
在Finder中，点击shift+command+G，输入/etc，将hosts文件拷贝到桌面，修改后再拷贝回去（需要输入密码）。

再次debug，正常！
![](http://7xo67b.com1.z0.glb.clouddn.com/20170110/17011002.png)
- 参考:

> http://stackoverflow.com/questions/29188789/eclipse-mac-os-x-debug-error-fatal-error-in-native-method-jdwp-no-transports
