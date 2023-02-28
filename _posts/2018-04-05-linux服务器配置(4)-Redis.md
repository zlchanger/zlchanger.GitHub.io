---
layout: post
title: linux服务器配置（4）————Redis
date: 2018/04/05
categories: blog
tags: [linux配置]
description: 配置服务器——————Redis。

---
## 配置环境

操作系统：centos；
平台：阿里云服务器；

## linux安装Redis步骤

1.在/usr/local/下创建redis文件夹，用来存放redis。

    cd /usr/local
    mkdir redis

2.下载Redis 

    wget http://download.redis.io/releases/redis-3.0.4.tar.gz
    
3.解压Redis 

    tar -zxvf redis-3.0.4.tar.gz
    rm -rf redis-3.0.4.tar.gz
    
4.编译安装Redis 

    cd redis-3.0.4
    make
执行安装命令

    make install
    
make install安装完成后，会在/usr/local/bin目录下生成下面几个可执行文件，它们的作用分别是：

redis-server：Redis服务器端启动程序 
redis-cli：Redis客户端操作工具。也可以用telnet根据其纯文本协议来操作 
redis-benchmark：Redis性能测试工具 
redis-check-aof：数据修复工具 
redis-check-dump：检查导出工具

备注

有的机器会出现类似以下错误：

    make[1]: Entering directory `/root/redis/src'
    You need tcl 8.5 or newer in order to run the Redis test
    ……

这是因为没有安装tcl导致，yum安装即可：

    yum install tcl
    
5.配置Redis 

复制配置文件到/etc/目录： 
    
    cp redis.conf /etc/
           
为了让Redis后台运行，一般还需要修改redis.conf文件：

    vim /etc/redis.conf
    
修改daemonize配置项为yes，使Redis进程在后台运行

    daemonize yes
    
ip修改为0.0.0.0

    bind 0.0.0.0

6.启动Redis 

配置完成后，启动Redis：

    cd /usr/local/bin
    ./redis-server /etc/redis.conf
    
检查启动情况：

    ps -ef | grep redis
    
进入redis客户端
    
    ./bin/redis-server etc/redis.conf
    ./bin/redis-cli
    shutdown/quit/auth
    
7.设置用户密码

进入/usr/local/bin

    ./redis-cli
    
    config get requirepass
    config set requirepass password
    auth password
    config get requirepass
    quit
    
—End—