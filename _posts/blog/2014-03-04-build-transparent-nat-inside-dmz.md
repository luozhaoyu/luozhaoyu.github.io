---
layout: post
title: "linux为DMZ的机器通过透明nat配置公网ip"
categories: blog
---
处于DMZ之后的内网机器往往只配了一块网卡，所以往往只有一个内网ip(就算配了外网ip，外网的机器也ping不到，所以没有作用)(除非再去上游配置路由)

总之，如果能在防火墙或gateway A统一为内网机器B配上公网ip那便是极好的。这里我们需要在A上通过iptables配置DNAT和SNAT来篡改ip包目标或源地址

假设gateway A替B保管了一个公网ip 1.2.3.4, 机器B的内网ip是10.5.0.70/16

1. 在A上设置B的DNAT`sudo iptables -t nat -A PREROUTING -d 1.2.3.4 -j DNAT --to-destination 10.5.0.70`
- 其次再设置SNAT `sudo iptables -t nat -A POSTROUTING -d 10.5.0.0/16 -j MASQUERADE` 注意这里是*-d*而不是*-s*，*-s*的含义是SNAT，伪装内网出去的出具包，这里需要是把外网进来数据包伪造成A的内网ip以便机器B能够回复给A的内网ip，而不是直接回复给源头

备注
----------
可以给机器B配一些其它的内网ip专门用来接收公网访问

    vim /etc/network/interfaces
    auto eth0:0
    iface eth0:0 inet static
        address 10.5.0.70
        netmask 255.255.0.0

在gateway A上可以用`iptables-save`来保存iptables设置
