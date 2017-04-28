
---
title: 「Flask」0x03树莓派上使用Nginx和uWSGI部署Flask应用
date: 1970-01-01 00:00:00
updated: 2017-04-27 19:07:57
comments: true
tags:
    - Flask
categories:
    - 技术
toc: true
cover: cover.jpg 
---


## 0x00 部署：uWSGI 和 Nginx 介绍

Deploy Flask Application with Nginx and uWSGI

考虑到部署的问题，相信您已经完成了自己Flask应用的开发工作，或者跟我一样，一个demo 应用已经完成。好奇自己的应用如何发布出去。接下来，通过介绍一些常用的工具来完成我们应用的部署。先简单介绍下我的环境：

- 一台内网中的树莓派
- 一个花生棒(98RMB买的硬件)

使用花生棒做了内网的端口映射，对外暴露80端口提供http服务，至于为什么这么做，请参考[这里](http://www.jianshu.com/p/9855c15a1dd7)。

好了，接下来进入正题。

uWSGI作为Web服务器使用，nginx做反向代理。

### uWSGI

uWSGI做Web服务器

《Flask Web开发：基于Python的Web应用开发实战》有下面两段描述：

其一：

> Flask自带的开发Web服务器不够强健、安全和高效，无法在生产环境中使用。

其二：

> Flask自带的开发Web服务器表现很差，因为它不是为生产环境设计的服务器。有两个可以在生产环境中使用、性能良好且支持Flask程序的服务器，分别是Gunicorn(http://gunicorn.org/) 和 uWSGI(http://uwsgi-docs.readthedocs.org/en/latest/) 。

先介绍一下WSGI（from：[维基百科](https://zh.wikipedia.org/wiki/Web服务器网关接口)）：

> Web服务器网关接口（Python Web Server Gateway Interface，缩写为WSGI）是为Python语言定义的Web服务器和Web应用程序或框架之间的一种简单而通用的接口。自从WSGI被开发出来以后，许多其它语言中也出现了类似接口。
> 以前，如何选择合适的Web应用程序框架成为困扰Python初学者的一个问题，这是因为，一般而言，Web应用框架的选择将限制可用的Web服务器的选择，反之亦然。那时的Python应用程序通常是为CGI，FastCGI，mod_python中的一个而设计，甚至是为特定Web服务器的自定义的API接口而设计的。
>WSGI[1] （有时发音作'wiz-gee'）是作为Web服务器与Web应用程序或应用框架之间的一种低级别的接口，以提升可移植Web应用开发的共同点。WSGI是基于现存的CGI标准而设计的。

因此我们需要为我们的Flask应用配置一个 `Web服务器` 。《Flask Web开发》中介绍的部署到Heroku选择的是Gunicorn。Gunicorn  和 uWSGI 的比较网上很多，主要的问题可能就是谁的坑多坑少。经过比对，这里我选择uWSGI。

uWSGI 不仅仅是一个协议，同时也是一个应用服务器，可以服务于uWSGI、FastCGI和HTTP协议。uWSGI官方文档可参考[这里](http://uwsgi-docs.readthedocs.io/en/latest/) 。

支持WSGI的Web应用框架很多，比如：

- Django
- Flask
- web.py
- web2py
- Werkzeug
- Tornado
- and so on

### Nginx

nginx做反向代理 

> Nginx（发音同engine x）是一个网页服务器，它能反向代理HTTP, HTTPS, SMTP, POP3, IMAP的协议链接，以及一个负载均衡器和一个HTTP缓存。

> Nginx是一款面向性能设计的HTTP服务器，相较于Apache、lighttpd具有占有内存少，稳定性高等优势。与旧版本（<=2.2）的Apache不同，nginx不采用每客户机一线程的设计模型，而是充分使用异步逻辑，削减了上下文调度开销，所以并发服务能力更强。整体采用模块化设计，有丰富的模块库和第三方模块库，配置灵活。 在Linux操作系统下，nginx使用epoll事件模型，得益于此，nginx在Linux操作系统下效率相当高。同时Nginx在OpenBSD或FreeBSD操作系统上采用类似于epoll的高效事件模型kqueue。

> Nginx在官方测试的结果中，能够支持五万个平行连接，而在实际的运作中，是可以支持二万至四万个平行链接。

以上来信息自[维基百科](https://zh.wikipedia.org/wiki/Nginx)。

反向代理：

> 使用代理服务器可以将请求转发给内部的Web服务器，使用这种加速模式显然可以提升静态网页的访问速度。因此也可以考虑使用这种技术，让代理服务器将请求 均匀转发给多台内部Web服务器之一上，从而达到负载均衡的目的。这种代理方式与普通的代理方式有所不同，标准代理方式是客户使用代理访问多个外部Web 服务器，而这种代理方式是多个客户使用它访问内部Web服务器，因此也被称为反向代理模式。

参考：http://www.bing.com/knows/反向代理负载均衡

虽然你可能不知道Nginx是做什么的，但是你应该听过，再或者你至少听过Apache，他们都是HTTP服务器。恕我能力有限，目前为止并不能很好的解释为什么非得用Nginx，能告诉大家的是：Nginx帮我们处理很多静态资源，同时将动态请求转发给不同的Web服务器（负载均衡）。此处留坑，等以后补上。

接下来通过最简单的配置，将我们的Flask应用部署到我们的树莓派上，达到对部署的流程有个总体把握的目的。如果你需要更多的配置，可以参考各自部分的官方文档。


## 0x01 uWSGI

- 安装

	先从安装说起：`pip install uwsgi`

- 配置
	
	在我们项目的根目录下，创建 `uwsgi.ini` 文件， 内容如下：
	
	``` shell
[uwsgi]
socket = 127.0.0.1:5000
processes = 4
threads = 2
plugins = python3
master = true
venv = venv
pythonpath = .
module = manage
callable = application
	```
	配置参数说明：
	- socket：应用程序所在地址，IP加端口号，当然，也可以有其他的形式
	- processes：开启的进程数量
	- threads：每个进程的线程数
	- plugins：加载的插件
	- module：加载指定的python WSGI模块
	- callable：在收到请求时，uWSGI加载的模块中哪个变量将被调用，默认是名字为“application”的变量。
	
	顺便贴一下我的Flask项目文件：manage.py
	
	``` python 
#!/usr/bin/env python3
# coding:utf-8
from flask.ext.script import Manager
config = 'development'
application = create_app(config)
manager = Manager(application)
...
if __name__ == '__main__':
        manager.run()
	```
	
- 操作
	
	启动：`uwsgi uwsig.ini`。如果启动成功，会看到类似下面的log：
	
	``` shell
(venv)pi@raspberrypi:~/raspi $ uwsgi uwsgi.ini
[uWSGI] getting INI configuration from uwsgi.ini
open("./python3_plugin.so"): No such file or directory [core/utils.c line 3684]
!!! UNABLE to load uWSGI plugin: ./python3_plugin.so: cannot open shared object file: No such file or directory !!!
*** Starting uWSGI 2.0.13.1 (32bit) on [Sun Jul  3 22:35:08 2016] ***
compiled with version: 4.9.2 on 27 May 2016 15:15:57
os: Linux-4.1.19-v7+ #858 树莓派 Tue Mar 15 15:56:00 GMT 2016
nodename: raspberrypi
machine: armv7l
clock source: unix
detected number of CPU cores: 4
current working directory: /home/pi/raspi
detected binary path: /home/pi/raspi/venv/bin/uwsgi
!!! no internal routing support, rebuild with pcre support !!!
your processes number limit is 6831
your memory page size is 4096 bytes
detected max file descriptor number: 65536
lock engine: pthread robust mutexes
thunder lock: disabled (you can enable it with --thunder-lock)
uwsgi socket 0 bound to TCP address 127.0.0.1:5000 fd 3
Python version: 3.4.2 (default, Oct 19 2014, 14:03:53)  [GCC 4.9.1]
Set PythonHome to venv
Python main interpreter initialized at 0xbaf828
python threads support enabled
your server socket listen backlog is limited to 100 connections
your mercy for graceful operations on workers is 60 seconds
mapped 358400 bytes (350 KB) for 8 cores
*** Operational MODE: preforking+threaded ***
added ./ to pythonpath.
WSGI app 0 (mountpoint='') ready in 5 seconds on interpreter 0xbaf828 pid: 3418 (default app)
*** uWSGI is running in multiple interpreter mode ***
spawned uWSGI master process (pid: 3418)
spawned uWSGI worker 1 (pid: 3428, cores: 2)
spawned uWSGI worker 2 (pid: 3429, cores: 2)
spawned uWSGI worker 3 (pid: 3431, cores: 2)
spawned uWSGI worker 4 (pid: 3432, cores: 2)
	```
	
	- 停止
	
		- 如果还在同一个shell中，我们可以直接按 `Ctrl + C` 
		- 如果不在同一个shell中，可以这样结束掉 `killall -9 uwsgi`
	
	启动成功之后就可以访问我们的Web应用了，默认地址是：http://127.0.0.1:5000 （这个地址和端口号是在我们的Flask应用中配配置的，这里不做介绍）。如果出现 `invalid request block size: 21573 (max 4096)...skip`这个错误，请将配置中的 `socket` 改为 `http`，具体可以参考[这里](http://stackoverflow.com/questions/15878176/uwsgi-invalid-request-block-size)。


## 0x02 Nginx

- 安装

	`sudo apt-get install ngnix`
	
- 操作
	
	- 启动
		
		sudo /etc/init.d/nginx start
		sudo service nginx start
		
	- 停止
	
		sudo /etc/init.d/nginx stop
		sudo service nginx stop
		
	- 重启
		
		sudo /etc/init.d/nginx restart
		sudo service nginx restart
	
	如果没问题，这个时候我们就可以打开浏览器访问：127.0.0.1，应该会看到一个静态页面。

- 配置

	nginx 的配置文件时位于 `/etc/nginx/sites-available` 目录下的 `default` 文件，我们复制一份做修改，这里同样给一份最简单的配置：
	
	``` shell
server {
    listen 80;

    server_name ttdevs.vicp.net;

    # access_log logs/access.log compression;

    #默认请求
    location / {
            include uwsgi_params;
            uwsgi_pass 127.0.0.1:5000;
    }
}
	```
	
	配置好这些之后重启我们的nginx服务：`sudo service nginx restart`。即可访问我们的web应用了。	


## 0x03 总结

如果中间没有出错，你的Flask应用已经部署成功。到此，我们来梳理下这个流程。首先，我们安装Nginx，他直接处理用户发送的HTTP请求（HTTP服务器），并将请求按照我们的配置（nginx中的配置）交给uWSGI服务器（Web服务器），最后uWSGI将请求交给我们的Flask应用，由Flask应用进行逻辑处理，处理完之后再将结果返回给用户。这些，就是我对这个流程的理解，相信中间肯定有错误或者不到位的地方，在对这些知识有更进一步理解的时候我会返回及时更新。同时也欢迎各位指正～～


## 0xFF 参考

1. http://docs.jinkan.org/docs/flask/deploying/uwsgi.html
2. http://www.cnblogs.com/zhouej/archive/2012/03/25/2379646.html
3. http://www.cnblogs.com/Ray-liang/p/4173923.html
3. https://github.com/nginx/nginx/blob/master/conf/uwsgi_params
4. http://docs.jinkan.org/docs/flask/deploying/wsgi-standalone.html#deploying-proxy-setups

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)

