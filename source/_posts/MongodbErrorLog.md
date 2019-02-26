---
title: "Mongodb启动异常处理记录"
date: 2019-02-26
categories: 技术杂记
tags: 
	- mongodb
---

# Mongodb启动异常处理记录

## 问题描述

> 在使用yum安装mongodb3之后一直没有手动重启过，也没修改过配置文件，在使用java连接时，出现了socket连接失败。

## 解决步骤

### Linux防火墙端口未开放问题

> 初步考虑到是由于通过非本地地址访问mongodb的27017端口，可能是被linxu防火墙拦截了，使用的是centos7d但手动开启了iptables，具体设置可以根据[iptables的详细介绍及配置方法](http://my.oschina.net/shipley/blog/299025) 但开启之后扔发现无法连接。

### mongodb端口监听

> 找不到原因考虑可能是mongodb自身的配置问题，找到mongodb配置文件/ect/mongod.conf 发现里面 bindIP 的默认配置为127.0.0.1 ，查询发现bindIp为绑定地址，默认只能本地访问，其他配置可查询[mongodb配置文件详解](http://my.oschina.net/pwd/blog/399374)修改后重启mongdb服务发现mongodb起不来了，状态一直是failed。

### mongod.pid文件地址不存在

> 从此查询了许多包括发现了一个命令 journalctl -xe 能查看系统启动日志，可以发现里面什么服务启动异常。但最终发现这不是问题的所在，真正的问题还是在看了/var/log/mongodb/mongodb.log mongodb的日志中发现根本原因是因为启动出现了异常 “ERROR: Cannot write pid file to /var/run/mongodb/mongod.pid: No such file or directory” 在mongodb的配置文件中有一个是pidFilePath 配置了此处目录但实际上/var/run中并没有mongodb这个目录。所以应该就是在这了，而我手动创建mongodb目录和文件之后依然不行，发现mongdb在安装时默认创建了mongod用户和组来管理mongodb而我用root权限创建的文件夹mongodb没有权限访问，所以仿照文章[ fixed init.d script to create directory for pid file](https://github.com/mongodb/mongo/commit/50ca596ace0b1390482408f1b19ffb1f9170cab6) 解决了此问题

### Failed to unlink socket file /tmp/mongodb-27017.sock errno:1 Operation not permitted

> 本以为终于可以解决这个搞了一天的问题，但事情尚未结束，mongodb的日志文件又给了我这样一个错误，根据文章[启动MongoDB时出现Failed to unlink socket file /tmp/mongodb-27017.sock errno:1 Operation not permitted](http://blog.csdn.net/doegoo/article/details/51906165)发现问题在于我用了root账户重启mongodb,生成的文件/tmp/mongodb-27017.sock也是root用户的，所以需要把文件删除后切换到mongod 账户重启mongodb服务发现终于启动成功

### mongod.pid自动生成

> 第二天发现重新启动mongodb的时候又再次出现了mongod.pid文件无法创建的问题，再查找到了一篇文章，按照上面说的创建了一个/lib/tmpfiles.d/mongodb.conf 内容是“d /var/run/mongodb 0755 mongod mongod” 然后执行sudo systemd-tmpfiles --create mongodb.conf就可以创建/etc/run/mongodb文件夹并付好权限，但每次启动都执行这句话不是太费劲了么，思考是否可以把这个命令写到mongdb的启动命令中。所以修改/etc/init.d/mongo文件

```
start()
{
  # Make sure the default pidfile directory exists
  if [ ! -d $PIDDIR ]; then
    # 测试在没有创建/etc/run/mongod文件夹的时候自动创建
    sudo systemd-tmpfiles --create mongodb.conf
    install -d -m 0755 -o $MONGO_USER -g $MONGO_GROUP $PIDDIR
  fi
```
> 这样执行重启后发现不需要在单独执行mongodb.conf文件，但开机启动时mongodb状态依然是faild需要后续在研究

### mongodb开机启动时faild问题解决

> 查看日志发现错误代码

``` 
2016-08-24T21:52:16.742-0400 E NETWORK  [initandlisten] listen(): bind() failed errno:99 Cannot assign requested address for socket: 192.168.6.56:27017
2016-08-24T21:52:16.742-0400 E STORAGE  [initandlisten] Failed to set up sockets during startup.

```

> 发现是mongodb在绑定端口192.168.6.56:27017的时候无法绑定发生了异常，想到可能是防火墙未开启27017端口问题，查看iptables状态发现果然iptables未开启，设置的开放27017端口自然也就没有起效。后续虽然设置了iptables服务开机启动但重启后发现仍然是关闭的，再百度发现firewalld服务与iptables只能使用一个。centos7默认是使用firewalld服务的，当时未搞明白firewalld的使用方式所以就重新开启了iptables结果忘记将firewalld服务设置不开机启动了。最终将firewall设置为不开机启动,重启后一切正常了，iptables与mongodb都能正常启动，至此事情完结了。