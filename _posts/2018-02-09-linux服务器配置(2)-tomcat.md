---
layout: post
title: linux服务器配置（2）————tomcat
date: 2018/02/09
categories: blog
tags: [linux配置]
description: 配置服务器——————tomcat。

---
## 配置环境

操作系统：centos；
平台：阿里云服务器；

## linux安装tomcat步骤

1.在/usr/local/下创建tomcat文件夹，用来存放tomcat。

    cd /usr/local
    mkdir tomcat

2.下载自己所要的tomcat安装包，使用wget命令。如：apache-tomcat-9.0.4.tar.gz

    wget http://mirrors.hust.edu.cn/apache/tomcat/tomcat-9/v9.0.4/bin/apache-tomcat-9.0.4.tar.gz
    
3.解压下载的toncat安装包，并删除安装包

    tar -zxvf apache-tomcat-9.0.4.tar.gz
    rm -rf apache-tomcat-9.0.4.tar.gz
    
4.配置环境变量，使用vi命令打开/etc/profile

    vim /etc/profile

添加如下配置

    #set tomcat
    CATALINA_BASE=/usr/local/tomcat/apache-tomcat-9.0.4
    CATALINA_HOME=/usr/local/tomcat/apache-tomcat-9.0.4
    TOMCAT_HOME=/usr/local/tomcat/apache-tomcat-9.0.4
    export CATALINA_BASE CATALINA_HOME TOMCAT_HOME

保存并退出
 
4.(1)使用vi打开server.xml，可以修改port端口号，默认8080
   
5.启动Tomcat（记得服务器安全组打开8080端口）

    cd /usr/local/tomcat/bin
    ./startup.sh
    
在浏览器输入ip:8080，可以访问tomcat服务器

6.关闭Tomcat

    ./shutdown.sh
    
—End—
