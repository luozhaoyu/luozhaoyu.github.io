---
layout: post
title: "配置haproxy及通过rsyslog记录日志"
categories: blog
---

安装
-----------
推荐使用`apt-get install haproxy`里面自带了重新启动脚本

如果编译的话`apt-get install libpcre3-dev`的pcre可以大幅提高regex表达式的解析速度
然后`make TARGET=generic USE_PCRE=1`


配置
--------
haproxy.cfg

    global
            log 127.0.0.1   local0
            log 127.0.0.1   local1 notice
            #log loghost    local0 info
            maxconn 4096
            #chroot /usr/share/haproxy
            user sysop
            group sysop
            daemon
            #debug
            #quiet
    
    defaults
            log     global
            mode    http
            option  httplog
            option  dontlognull
            option  httpclose # 很重要，确保http真的关闭
            option  forceclose # 没有关闭的强制关闭，保证兼容性
            retries 3
            option redispatch
            maxconn 2000
            contimeout      5000
            clitimeout      50000
            srvtimeout      50000
    
    listen test 0.0.0.0:8000
            balance roundrobin
            server  server1 10.5.5.5:8001 check weight 30
            server  server2 10.5.5.5:8002 check weight 30
            server  server3 10.5.5.5:8003 check weight 30
            stats enable
            stats uri /stats
            stats refresh 3



rsyslog
-----------
首先让rsyslog监听514端口，其次是把haproxy的日志都打入特定文件
`vim /etc/rsyslog.d/haproxy.conf`

    # Create an additional socket in haproxy's chroot in order to allow logging via
    # /dev/log to chroot'ed HAProxy processes
    $AddUnixListenSocket /tmp/haproxylog.sock
    #local0.*                               /tmp/haproxy.log
    #local1.*                               /tmp/haproxy.log

    # Send HAProxy messages to a dedicated logfile
    if $programname startswith 'haproxy' then /tmp/haproxy.log

### 调试rsyslog
`logger -p local0.err thisisanerrormessage`
