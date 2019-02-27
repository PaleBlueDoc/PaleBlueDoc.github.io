---
title: Nginx反向代理初步学习
date: 2019-02-26
categories: 技术杂记
tags: 
	- nginx
---

nginx是一个轻量级，性能强大的web和反向代理服务器，Nginx流行也有很长一段时间了，而最近公司的项目也都使用了Nginx的反向代理。借此机会我也学习一下Nginx的使用。

## Nginx安装

### Windows
windows系统的nginx安装就是下载后直接解压运行EXE文件即可，不过运行后不会像tomcat一样有一个控制台一直运行，二十运行起来后就隐藏了，需要启动CMD运行命令（nginx -s stop）进行关闭。或者直接任务管理器结束进程。

### Linux
Linux中Nginx的安装步骤可以参照吴秦（Tyler）的[Nginx安装与使用](http://www.cnblogs.com/skynet/p/4146083.html)，使用GUN的安装规范进行安装。安装后会将Nginx安装到/usr/local/nginx下，运行时使用命令“nginx -c 配置文件目录”启动Nginx，关闭时使用“nginx -s stop”命令

<!--more-->

## Nginx配置文件
我使用的是本地的虚拟机安装的Nginx,虚拟机分配的局域网IP为192.168.6.56，在虚拟机的tomcat中运行了一个testproject项目。Nginx的配置文件也是非常简洁的，以下是我自己最初设置的Nginx配置，我的理解是只要监听好域名及端口，在默认访问域名时只要直接跳转到tomcat的项目地址就可以了。


	worker_processes  1;		##工作进程数量，一般以CPU数量为准
	
	events {
	    worker_connections  1024;
	}
	
	http {
	    include       mime.types;
	    default_type  application/octet-stream;
	
	    sendfile        on;
	    keepalive_timeout  65;
	
	
	    server {				##监听服务
	        listen      80 ;	##监听端口
	        server_name  192.168.6.56;	#监听域名
	
	        location / {		##匹配规则 使用正则表达式
	                proxy_pass http://localhost:8080/testproject/;			#匹配规则的连接反向代理到的连接
	                proxy_redirect default;
	        }
	    }
	
	}


我希望得到的结果是在本机访问虚拟机的IP时可以直接进入到tomcat的项目，这样如果我有一个域名，只要把域名绑定到虚拟机IP就可以实现通过域名直接访问项目了。
但事情没有这么容易，在我用上面配置运行时，访问的结果是浏览器出现了错误提示“重定向次数过多”，我理解应该是nginx循环重定向了，但我又不明白为什么会如此，百度搜索了半天也没找到问题的合理解释，但却莫名找到了解决办法。修改配置文件如下：
	
	worker_processes  1;
	
	events {
	    worker_connections  1024;
	}
	
	http {
	    include       mime.types;
	    default_type  application/octet-stream;
	
	    sendfile        on;
	    keepalive_timeout  65;
	
	
	    server {
	        listen      80 ;
	        server_name  192.168.6.56;
	
	        location / {
	                proxy_pass http://localhost:8080/testproject/;
	                proxy_set_header    REMOTE-HOST $remote_addr;
	                proxy_set_header   Host $host;
	                proxy_set_header   X-Real-IP $remote_addr;
	                proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
	        }
	       
	    }
	}

添加了proxy_set_header参数，用来修改反向代理到tomcat连接时的header。而我对HTTP请求的理解并不深入，所以也没太看懂具体是什么原理。后续搞明白了再写一遍文章来记录吧。
按照上面的配置，在规则中都重新设置请求的header。重新访问时就不会出现“重定向次数过多”的问题。直接通过IP就可以反向代理到tomcat的项目中，不需要修改tomcat的任何配置。

PS:在公司的项目中配置时，我们处理同一个tomcat中多个项目分配不同域名是通过配置tomcat的多个HOST，然后使用Nginx反向代理每个HOST绑定的域名来实现的。而使用这种配置方式时，就可以不用修改tomcat配置，在Nginx中设置多个server绑定多个域名反向代理tomcat中不同的项目也可以实现。
