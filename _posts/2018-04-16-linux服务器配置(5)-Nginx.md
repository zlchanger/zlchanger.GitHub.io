---
layout: post
title: linux服务器配置（5）————Nginx
date: 2018/04/16
categories: blog
tags: [linux配置]
description: 配置服务器——————Nginx。

---
## 配置环境

操作系统：centos；
平台：阿里云服务器；

## linux安装Nginx步骤

1.添加Nginx到YUM源,添加CentOS 7 Nginx yum资源库,打开终端,使用以下命令:
               
    sudo rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
    
2.使用yum命令从Nginx源服务器中获取来安装Nginx：

    sudo yum install -y nginx
    
3.以上Nginx已安装在服务器上了，进入/etc/nginx文件夹下可修改Nginx配置

    cd /etc/nginx
      
4.启动Nginx，浏览器输入ip，可看到Nginx欢迎页。

    sudo systemctl start nginx.service        
    
5.CentOS 7 开机启动Nginx

    sudo systemctl enable nginx.service
    
-End-