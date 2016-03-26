---
layout: post
title:  "keepalived的最小化配置应用"
categories: blog
---

背景介绍
----------
keepalived构建于lvs之上，并实现了vrrp协议

这里要配置机器A 10.5.5.5/16与机器B 10.5.4.4/16 共用一个虚拟ip 10.5.1.1

[haproxy作者Willy Tarreau谈heartbeat与keepalived选择](http://www.formilux.org/archives/haproxy/1003/3259.html)

安装
---------
因为都是root操作，索性`apt-get install keepalived`


配置
--------
先copy官方的sample配置`sudo cp /usr/share/doc/keepalived/samples/keepalived.conf.sample /etc/keepalived/keepalived.conf`

    ! Configuration File for keepalived

    global_defs {
       notification_email {
         acassen
       }
       notification_email_from Alexandre.Cassen@firewall.loc
       smtp_server 192.168.200.1
       smtp_connect_timeout 30
       router_id LVS_DEVEL
    }

    vrrp_instance VI_1 {
        interface eth0
        virtual_router_id 50
        nopreempt
        priority 100 # 主服务器A上配置为100 从服务器B上配置50
        advert_int 1
        virtual_ipaddress {
            10.5.1.1
        }
    }

1. 配置完毕之后分别在AB上启动keepalived
2. kill A的keepalived
3. 通过`ip addr`命令观察B上ip地址变化
