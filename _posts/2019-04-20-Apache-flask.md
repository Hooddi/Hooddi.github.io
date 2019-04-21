---
layout: post
title: Apache + mod_wsgi的方式部署flask
categories: Flask部署
description: 一在Windows环境，采用Apache + mod_wsgi的方式部署flask
keywords: Apache, mod_wsgi, Flask, Windows
---

在Windows环境，采用Apache + mod_wsgi的方式部署flask，不如Linux下方便。    
Apache、mod_wsgi、python三者必须是同位数，比如同为X64或者X86。另外，查看对应的VC版本。  
说明：在Windows环境中，Apache不支持flask中的多进程或多线程功能，具体见 [flask](http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/)  

****
## 安装Apache
[Apache下载地址](https://www.apachelounge.com/download/)  
本人用的是X64，python3.5，所以选择VC15编译的"Apache 2.4.39 Win64"  
下载完毕后解压得到Apache24,打开进入conf文件，用文本打开，我这里采用notepad++打开。  
现在说明一下，我们需要修改的地方：  

1. ServerRoot "C:/Apache24"：修改成自己的目录地址，Apache的默认配置是C:\Apache24。  
2. Listen 8080：监听端口配置，默认的80端口可能被IIS占用，可以将其改成"8080"或者其他，也可以添加多个监听端口。  
3. (备选)一般都是在虚拟环境中部署项目，因此在conf文件中修改两处，表示将启用虚拟主机:  
	* #LoadModule vhost_alias_module modules/mod_vhost_alias.so  前面的#去掉
	* #Include conf/extra/httpd-vhosts.conf 前面的#去掉  

第3点可以自行选择是否准备启用虚拟环境，本教程中使用虚拟主机。  
修改conf文件后，保存。   
打开cmd，cd到"C:\Apache24\bin"，运行:  
```httpd –k install```  
安装Apache服务，顺便启动它。  
(常用命令：运行服务httpd –k start，停止服务 httpd –k stop ，也可以直接从服务器管理器中操作)  
此时，打开服务器管理器可以看到**Apache2.4服务**，如下图所示：
 
![](https://ws1.sinaimg.cn/large/005v4RA1ly1g29grijjq5j30ma09x75d.jpg "安装后服务器管理器中看到Apache2.4服务")  

## 安装mod_wsgi
