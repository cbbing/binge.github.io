---
layout: post
title: PyQt4 生成exe打包文件
date: 2015-11-26 17:00:51
tags: 
	- Python
	- PyQt 
	- exe
category: Python
comments: true
---

**PyQt4 生成exe打包文件**

pyqt是跨平台的GUI平台，本文的UI设计，代码编写在mac下进行，编译成exe，并打包在win7下做的。
python脚本语言，图形化平台不是其擅长的领域，一般都是直接运行脚本，这次因为客户需要一个“成型”的程序去外面给别人展示，故有了此文的背景。
QT作为一个跨平台的开发环境，编写出一个窗口程序，然后打包成python文件是比较迅速的。麻烦的是打包成windows的exe文件，试过py2exe,pyinstaller，都不是很好用，py2exe根本出不来图形界面，最后用到cxfreeze这个工具，才得以顺利打包。
<!-- more -->
## 前言
环境搭建参考：<http://www.cnblogs.com/zouzf/p/4308912.html>

## 一、Qt Designer设计界面
*安装Qt Designer，我的版本是5.2.1。*
设计出的界面如下，保存为.ui文件。
![Qt_UI](http://7xo67b.com1.z0.glb.clouddn.com/qt_main.png)



Qt的界面布局和MFC的比较类似，但它多了一层容器的概念。控件都放在容器Layout中，这点又和Android的手机布局比较相近。
> *UI布局教程参考：[PyQt4 精彩实例分析](http://www.linuxidc.com/Linux/2012-06/63652.htm)*
 


## 二、Qt布局文件.ui转换成.py文件
mac和win系统下先将pyuic4命令加入环境变量。mac中我是直接把/etc/paths拷贝到桌面，添加pyuic4路径后再拷贝覆盖回去。（ps: etc下不能直接修改）
在终端中执行：
 
    pyuic4 -x aaaaaaa.ui -o bbbbbb.py

即可将.ui文件转成py文件。

## 三、添加按钮动作
这里实现了三个功能：上传文件，运行py脚本，打开另一个Qt窗口
### 1，上传文件
#### a，post上传

    dlg = QFileDialog()
    filename = dlg.getOpenFileName()
    from os.path import isfile
    if isfile(filename):
        filename = str(filename)
        print type(filename)
        #dir_f = os.path.dirname(str(filename))

        # ------ web post -----
        # 在 urllib2 上注册 http 流处理句柄
        register_openers()
        
        # headers 包含必须的 Content-Type 和 Content-Length
        # datagen 是一个生成器对象，返回编码过后的参数
        datagen, headers = multipart_encode({"myfile": open(str(filename), "rb")})
        
        # 创建请求对象
        request = urllib2.Request("http://yourwebsite:8080/upload", datagen, headers)
        # 实际执行请求并取得返回
        print urllib2.urlopen(request).read()

#### b，通过 ftp 上传

 
    dlg = QFileDialog()
	filename = dlg.getOpenFileName()
	from os.path import isfile
	if isfile(filename):
	    filename = str(filename)
		
		# ------- ftp --------
		from ftplib import FTP
	    
		ftp=FTP()
		ftp.set_debuglevel(2)#打开调试级别2，显示详细信息;0为关闭调试信息
		ftp.connect('127.0.0.1','21')#连接
		ftp.login('Administrator','password')#登录，如果匿名登录则用空串代替即可
		print ftp.getwelcome()#显示ftp服务器欢迎信息
		#ftp.cwd(dir_f) #选择操作目录
		#filename='keys.xlsx'
		bufsize = 1024#设置缓冲块大小
		file_handler = open(filename,'rb')#以读模式在本地打开文件
		ftp.storbinary('STOR %s' % os.path.basename(filename),file_handler,bufsize)#上传文件
		ftp.set_debuglevel(0)
		file_handler.close()
		ftp.quit()
        print "ftp up OK"
      

### 2，运行py脚本
在服务器上用web.py搭建web服务器，通过网页请求运行py文件。

	#运行
	def on_run_clicked(self):
	    self.pushButton_run.setText(_translate("Dialog", "运行中", None))
	    self.pushButton_run.setEnabled(False)
	    response = urllib2.urlopen('http://%s:8080/run_video_search' % self.ip, timeout=3)
	    print response

这里采用的是同步的方式请求，服务器端的py脚本没执行完，则程序一直等待。不过设置了超时，过了3s返回超时错误，这种情况适合不需要得到服务器的反馈，只是执行远程py脚本而已。

### 3，打开另一个Qt窗口
pyqt 用起来的比较简单，直接run qt对应的py类

	#结果
	def on_show_result(self):
	
	    Dialog = QtGui.QDialog()
	    ui = Ui_Result_Dialog()
	    ui.setupUi(Dialog)
	    Dialog.show()
	    Dialog.exec_()


跳转过来的窗口如下，是一个数据库的查询界面。
![result ui](http://7xo67b.com1.z0.glb.clouddn.com/qt_result.png)                		
        
## 四、打包
cx_Freeze 支持跨平台，可在windows、linux，mac下使用。下载地址为(http://sourceforge.net/projects/cx-freeze/files/),,也可以直接通过pip安装
    
    pip install cx_freeze
    
安装成功后，在C:\Python27\Lib\site-packages\cx_Freeze\samples\PyQt4中找到pyqt4的使用例子。
查看setup.py

	import sys
	from cx_Freeze import setup, Executable
	
	base = None
	if sys.platform == 'win32':
	    base = 'Win32GUI'
	
	options = {
	    'build_exe': {
	        'includes': 'atexit'
	    }
	}
	
	executables = [
	    Executable('PyQt4app.py', base=base)
	]
	
	setup(name='simple_PyQt4',
	      version='0.1',
	      description='Sample cx_Freeze PyQt4 script',
	      options=options,
	      executables=executables
	      )


把这个setup.py文件拷贝到你要打包py文件的目录，然后将setup.py中的“PyQt4app.py”改成你要打包的py文件。
在cmd命令行，cd到当前目录，运行:

    python setup.py build

打包exe成功后，在当前目录下会生成build文件夹，在\build\exe.win32-2.7\中找到exe后缀的文件，执行。

## 五、制作安装包
采用的是Inno Setup 制作安装包，按照向导来生成.iss脚本，傻瓜化操作。








