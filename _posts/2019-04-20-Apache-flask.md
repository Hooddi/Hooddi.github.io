---
layout: post
title: Apache + mod_wsgi的方式部署flask
categories: Flask部署
description: 一在Windows环境，采用Apache + mod_wsgi的方式部署flask
keywords: Apache, mod_wsgi, Flask, Windows
---

在Windows环境，采用Apache + mod_wsgi的方式部署flask，不如Linux下方便。    
Apache、mod_wsgi、python三者必须是**同位数**，比如同为X64或者X86。另外，查看对应的VC版本。  
说明：在Windows环境中，Apache不支持flask中的多进程或多线程功能，具体见 [flask](http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/)

****
## 安装Apache
[Apache下载地址](https://www.apachelounge.com/download/)  
本人用的是X64，python3.5，所以选择VC15编译的"Apache 2.4.39 Win64"  
下载完毕后解压得到Apache24,打开进入**conf文件**，用文本打开，我这里采用notepad++打开。  
现在说明一下，我们需要修改的地方：  
1. ServerRoot "C:/Apache24"：修改成自己的目录地址，Apache的默认配置是C:\Apache24。  
2. Listen 8080：监听端口配置，默认的80端口可能被IIS占用，可以将其改成"8080"或者其他，也可以添加多个监听端口。  
3. (备选)一般都是在虚拟环境中部署项目，因此在conf文件中修改两处，表示将启用虚拟主机:  
	* #LoadModule vhost_alias_module modules/mod_vhost_alias.so  前面的#去掉
	* #Include conf/extra/httpd-vhosts.conf 前面的#去掉  
第3点可以自行选择是否准备启用虚拟环境，本教程中使用虚拟主机。

修改conf文件后，保存。   
打开cmd，cd到"C:\Apache24\bin"，运行: ```httpd –k install```  
安装Apache服务，顺便启动它。  
(常用命令：运行服务```httpd –k start```，```停止服务 httpd –k stop```，也可以直接从服务器管理器中操作)  
打开服务器管理器可以看到**Apache2.4服务**，如下图所示：
 
![](https://ws1.sinaimg.cn/large/005v4RA1ly1g29grijjq5j30ma09x75d.jpg "服务器管理器中Apache2.4服务")

在浏览器中输入网址："localhost:8080"，看到下图说明Apache安装成功  
说明：8080端口与conf文件中一致 
![](https://ws1.sinaimg.cn/large/005v4RA1ly1g2a9upnwtsj30l5053mxg.jpg)

## 安装mod_wsgi
[mod_wsgi下载地址](https://www.lfd.uci.edu/~gohlke/pythonlibs/#mod_wsgi?tdsourcetag=s_pctim_aiomsg)

![](https://ws1.sinaimg.cn/large/005v4RA1ly1g2aa2t2y02j30q507rq3q.jpg)

"cp35"表示对应python3.5，选择与python版本对应的下载。  
本教程选择的是:
>mod_wsgi-4.5.24+ap24vc14-cp35-cp35m-win_amd64.whl  

mod_wsgi可以选择安装在本地系统或者部署项目时的虚拟环境中，本教程选择安装在本地。  
可以直接通过pip安装:
>pip install mod_wsgi-4.5.24+ap24vc14-cp35-cp35m-win_amd64.whl

使用pip安装后，使用cmd进入python目录下的Scripts文件夹，运行:
>mod_wsgi-express module-config

![](https://ws1.sinaimg.cn/large/005v4RA1ly1g2aau8twvej30r70630ss.jpg)

将上图中红色方框中的**三行信息复制**，打开conf文件  
找到：**# LoadModule foo_module modules/mod_foo.so**  
将复制的信息粘贴在下方，表示使用wsgi来作为Python Web服务网关接口
![](https://ws1.sinaimg.cn/large/005v4RA1ly1g2ab0l6156j30xz06zq36.jpg)

此时，重启Apache服务，可以看到Apache服务的名字变了，如下图所示：  
![](https://ws1.sinaimg.cn/large/005v4RA1ly1g2ab44jxqfj30d209bt8v.jpg)

到此，mod_wsgi安装完成并在Apache的conf文件中修改相关配置信息。 
## 测试
本教程是在虚拟环境中部署flask，python虚拟环境自行百度
>pip install virtualenv
 
切换到目标文件夹下，创建虚拟环境:
>virtualenv venv

此时，目标文件夹中生成"venv"文件, 切换到.\venv\Scripts目录下，运行；
>activate

当虚拟环境激活后，可以运用pip在虚拟环境中安装需要用到的工具包，不与本地工具包冲突（用虚拟环境的原因）。本教程中仅安装Flask：  
>pip inatll flask

当虚拟环境搭建好之后，运行：  
>deactivate

关闭虚拟环境

首先，目标文件夹中创建test.py文件
```python
from flask import Flask, request
app = Flask(__name__)

@app.route('/lulu')
def hello_1():
	return 'Hello lulu!'
	
@app.route('/hudi')
def hello_2():
	return 'Hello hudi!'
@app.route('/')
if __name__ == '__main__':
	app.run()
```  
然后，目标文件夹中创建test.wsgi（wsgi入口文件）  
```
activate_this = r'F:\Github_hudi\Public\Apache_Flask_windows\venv\Scripts\activate_this.py'
exec(compile(open(activate_this, "rb").read(), activate_this, 'exec'), dict(__file__=activate_this))
import sys
#Expand Python classes path with your app's path
sys.path.insert(0, r'F:\Github_hudi\Public\Apache_Flask_windows')
from test import app as application
```  
其中:
```
activate_this = r'F:\Github_hudi\Public\Apache_Flask_windows\venv\Scripts\activate_this.py'
exec(compile(open(activate_this, "rb").read(), activate_this, 'exec'), dict(__file__=activate_this))
```  
表示启用虚拟环境，即test.py是在虚拟环境中运行  
**F:\Github_hudi\Public\Apache_Flask_windows**表示项目路径（目标文件夹）  
在wsgi入口文件中，**路径一定要用r''包起来**，否则可能会出错!

配置Apache的**vhost.conf文件**  
打开httpd-vhost.conf,在文件最后加上test.wsgi的配置信息:
```
<VirtualHost *:8080 >
	ServerAdmin webmaster@dummy-host.example.com
	ServerName localhost
	WSGIScriptAlias / F:\Github_hudi\Public\Apache_Flask_windows\test.wsgi 	
	<Directory "F:\Github_hudi\Public\Apache_Flask_windows">
		AllowOverride none
		Require all granted
		Require host ip
	</Directory>
</VirtualHost>
```  
保存文件，重启Apache。

测试：在浏览器中输入网址："localhost:8080/hudi"，若看到下图所示信息，表示部署成功。
![](https://ws1.sinaimg.cn/large/005v4RA1ly1g2ad87erdtj30qm0693yt.jpg)
