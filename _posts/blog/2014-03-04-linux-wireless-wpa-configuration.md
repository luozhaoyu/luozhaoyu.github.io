---
layout: post
title: "linux配置wireless无线"
categories: blog
---
查看可用的无线网络 `sudo iwlist wlan0 scan`

配置对应无线网络的密码 `wpa_passphrase USER PASS > /etc/wpa_supplicant/wpa.conf`

配置无线网卡启用时的自定义路由
-----------
    iface wlan0 inet static
        address 1.2.3.4
        netmask 255.255.0.0
        gateway YOURGATEWAY
        pre-up wpa_supplicant -B -iwlan0 -c/etc/wpa_supplicant/wpa.conf
        post-down killall -q wpa_supplicant
        post-up ip route add 192.168.0.0/16 via YOURGATEWAY
        post-up ip route replace 0.0.0.0/0 via YOURGATEWAY # set the default gateway

改变无线网卡的工作模式为master `iwconfig wlan0 mode master`
