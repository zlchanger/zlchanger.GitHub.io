---
layout: post
title: linux服务器配置（3）————MySQL
date: 2018/04/04
categories: blog
tags: [linux配置]
description: 配置服务器——————MySQL。

---
## 配置环境

操作系统：centos；
平台：阿里云服务器；

## linux安装MySQL步骤

1.在/usr/local/下创建mysql文件夹，用来存放mysql。

    cd /usr/local
    mkdir mysql

2.配置YUM源

在MySQL官网中下载YUM源rpm安装包：http://dev.mysql.com/downloads/repo/yum/ 
下载mysql源安装包

    shell> wget http://dev.mysql.com/get/mysql57-community-release-el7-8.noarch.rpm
安装mysql源

    shell> yum localinstall mysql57-community-release-el7-8.noarch.rpm
检查mysql源是否安装成功

    shell> yum repolist enabled | grep "mysql.*-community.*"
    
3.安装MySQL

    shell> yum install mysql-community-server
    
4.启动MySQL服务

    shell> service mysqld start
    shell> systemctl start mysqld
查看MySQL的启动状态

    shell> service mysqld status
    shell> systemctl status mysqld
    
    ● mysqld.service - MySQL Server
       Loaded: loaded (/usr/lib/systemd/system/mysqld.service; disabled; vendor preset: disabled)
       Active: active (running) since 五 2016-06-24 04:37:37 CST; 35min ago
     Main PID: 2888 (mysqld)
       CGroup: /system.slice/mysqld.service
               └─2888 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid
    
    6月 24 04:37:36 localhost.localdomain systemd[1]: Starting MySQL Server...
    6月 24 04:37:37 localhost.localdomain systemd[1]: Started MySQL Server.
    
5.开机启动

    shell> systemctl enable mysqld
    shell> systemctl daemon-reload
    
6.修改root本地登录密码

mysql安装完成之后，在/var/log/mysqld.log文件中给root生成了一个默认密码。通过下面的方式找到root默认密码，然后登录mysql进行修改：
    
    shell> grep 'temporary password' /var/log/mysqld.log
    
    shell> mysql -uroot -p
    mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!'; 
    或者
    
    mysql> set password for 'root'@'localhost'=password('MyNewPass4!');
    
7.授权其他机器登陆

    GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'mypassword' WITH GRANT OPTION;
    FLUSH  PRIVILEGES;
    
—End— 