---
layout: post
title: 如何自在的上网
date: 2019/01/28
categories: blog
tags: [VPS-Shadowsocks]
description: 实用消息中间件

---

## 1、购买与连接VPS

首先我们需要买一台VPS，国外的VPS服务器提供商有[Vultr](https://www.vultr.com/),Linode,Hostwinds,InterServer,SiteGround,DreamHost等。
我选取Vultr作为我的VPS购买服务商。

注册Vultr账号，使用payPal，微信或者信用卡支付购买你选选择的VPS型号。

## 2、部署shadowsocks

Shadowsocks 需要同时具备客户端和服务器端，所以它的部署也需要分两步。

### 2.1、部署 Shadowsocks 服务器端

这里使用 teddysun 的一键安装脚本。

    wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh
    chmod +x shadowsocks.sh
    ./shadowsocks.sh 2>&1 | tee shadowsocks.log

最后一步输完，你应该会看到下图中内容──是要你为 Shadowsocks 服务设置一个个人密码。

![shadowsock1.JPEG](https://raw.githubusercontent.com/zlchanger/PictureForMarkDown/master/sahdowsock/shadowsock1.jpeg)

输好回车后会让你选择一个端口，输入 1–65535 间的数字都行。

![shadowsock2.JPG](https://raw.githubusercontent.com/zlchanger/PictureForMarkDown/master/sahdowsock/shadowsock2.jpg)

之后可以选择加密方式，这个就看个人喜好了，默认是 aes-256-gcm。

![shadowsock3.PNG](https://raw.githubusercontent.com/zlchanger/PictureForMarkDown/master/sahdowsock/shadowsock3.png)

遵照上图指示，按任意键开始部署 Shadowsocks。这时你什么都不用做，只需要静静地等它运行完就好。结束后就会看到你所部署的 Shadowsocks 的配置信息。

![shadowsock4.JPG](https://raw.githubusercontent.com/zlchanger/PictureForMarkDown/master/sahdowsock/shadowsock4.jpg)

记住其中黄框中的内容，也就是服务器 IP、服务器端口、你设的密码和加密方式。

### 2.2、TCP Fast Open

实际上只要具备上述四个信息，你就可以在自己的任意设备上进行登录使用了。但是为了更好的连接速度，你还需要多做几步。

首先是打开 TCP Fast Open，输入以下命令，意为用 nano 这个编辑器打开一个文件。

    nano /etc/rc.local
    
你的「终端」会刷新一下，出现下图。

![shadowsock5.JPG](https://raw.githubusercontent.com/zlchanger/PictureForMarkDown/master/sahdowsock/shadowsock5.jpg)

别慌张，它就是个文本编辑器。用方向键把光标移到最末端，粘贴下面这一行内容，然后按 Ctrl + X 退出。

    echo 3 > /proc/sys/net/ipv4/tcp_fastopen
    
输入“Y”并回车确认退出。

![shadowsock6.JPEG](https://raw.githubusercontent.com/zlchanger/PictureForMarkDown/master/sahdowsock/shadowsock6.jpeg)

然后依法炮制，输入：

    nano /etc/sysctl.conf

在文末加上下面的内容，保存退出。

    net.ipv4.tcp_fastopen = 3
    
再打开一个 Shadowsocks 配置文件。

    nano /etc/shadowsocks.json
    
把其中 “fast_open” 一项的 false 替换成 true。

    "fast_open":true
    
如果你希望添加多用户的话，可以将 “password” 字段如下图修改。其中，“22345”:“password1”意为该用户使用 22345 端口、以“password1”为密码连接登录 Shadowsocks。

![shadowsock7.JPEG](https://raw.githubusercontent.com/zlchanger/PictureForMarkDown/master/sahdowsock/shadowsock7.jpeg)

如果添加了多用户，需要更改防火墙规则，开放刚刚新增的端口。centOS 6 系统开放办法如下。保存退出后，输入以下命令，注意一次输入一行，并把 <newport> 替换为刚添加的端口：

    iptables -I INPUT -m state — state NEW -m tcp -p tcp — dport <newport> -j ACCEPT
    iptables -I INPUT -m state — state NEW -m udp -p udp — dport <newport> -j ACCEPT
    /etc/init.d/iptables save
    /etc/init.d/iptables restart

保存退出。最后，输入以下命令重启 Shadowsocks。

    /etc/init.d/shadowsocks restart
    
## 3、安装 BBR

和 2.3 中的步骤一样，首先需要用 SSH 登录 VPS。

    ssh root@<host>
    
这里依然使用 @teddysun 的一键安装脚本，输入以下命令。

    wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh && chmod +x bbr.sh && ./bbr.sh
    
按任意键开始安装，安装需要一段时间，等待一会即可。

![bbr1.jpg](https://raw.githubusercontent.com/zlchanger/PictureForMarkDown/master/sahdowsock/bbr1.png)

安装完成后，脚本会提示需要重启 VPS，输入 y 并回车后重启。

重新使用 SSH 登录 VPS，这时 BBR 应该已经开启了。你可以使用以下两行命令验证一下。

    uname -r
    lsmod | grep bbr
    
如果输出的内核版本为 4.9 以上版本，且返回值有 tcp_bbr 模块的话，说明 bbr 已启动。

![bbr2.jpg](https://raw.githubusercontent.com/zlchanger/PictureForMarkDown/master/sahdowsock/bbr2.png)

至此，整个搭建过程就大功告成了！接下来，尽情地享受自由的网络吧😄

