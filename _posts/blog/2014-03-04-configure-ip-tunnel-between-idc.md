---
layout: post
title: "linux配置idc之间的ip tunnel"
categories: blog
---
假设有机房A的机器10.1.0.10/16 1.2.3.4与机房B的机器192.168.1.10/24 5.6.7.8要搭建基于internet的内网隧道

机房A机器的`/etc/network/intefaces`配置为192.168.100.12

    auto eth1
    iface eth1 inet static
        address 10.1.0.10
        netmask 255.255.0.0
        pre-up ip tunnel add idca_idcb mode gre remote 5.6.7.8
        pre-up ip link set idca_idcb up
        pre-up ip addr add 192.168.99.12 dev idca_idcb
        pre-up ip route replace 192.168.99.21 dev idca_idcb
        pre-up ip route replace 192.168.1/24 via 192.168.99.21
        post-down ip tunnel del idca_idcb

机房B机器的`/etc/network/intefaces`配置为192.168.100.21

    auto eth0
    iface eth0 inet static
        address 192.168.1.10
        netmask 255.255.255.0
        pre-up ip tunnel add idcb_idca mode gre remote 1.2.3.4
        pre-up ip link set idcb_idca up
        pre-up ip addr add 192.168.100.21 dev idcb_idca
        pre-up ip route replace 192.168.100.12 dev idcb_idca
        pre-up ip route replace 10.1.0.0/16 via 192.168.100.12
        post-down ip tunnel del idcb_idca
