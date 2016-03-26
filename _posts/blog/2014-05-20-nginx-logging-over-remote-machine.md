---
layout: post
title: "Configure nginx with rsyslog to assemble your log files"
---

Rsyslog Configurations
-----------
We need a local rsyslog to detect local nginx log change and a remote rsyslog to receive logs.

local relay
=======
in `/etc/rsyslog.conf`:

    $ModLoad imudp
    $UDPServerRun 514

    template(name="nginxForwardFormat" type="string"
    string="%rawmsg%"
    )

    $WorkDirectory /var/spool/rsyslog

    module(load="imfile" PollingInterval="10")
    input(type="imfile"
    File="/var/log/nginx/access.log"
    Tag="nginx"
    StateFile="nginx_imfile"
    Facility="local7"
    Severity="info"
    )

    if $syslogtag == "nginx"
    then @10.5.5.5:514;nginxForwardFormat

remote center rsyslog
==========

    template(name="nginxFormat" type="string" string="%rawmsg% [\"%timereported:::rfc3339%\" %hostname% %syslogtag% %pri-text%]\n")
    # One legacy hack to filter out nginx log
    :msg, contains, "HTTP" /var/log/rsyslog_nginx.log;nginxFormat

debug
==========
* `logger -p local0.err thisisanerrormessage`
* `nc -w0 -u 127.0.0.1 514 <<< "test messages"`

Nginx
----------
In fact, to reduce the PollingInterval, nginx could write its log to a named pipe.
But you should be careful about the network fluctuation to correct missing parts.

Logrotate
---------

    /var/log/nginx/access.log {
        daily
        missingok
        # make sure only one old log for us to rename
        rotate 1
        notifempty
        create 0644 sysop sysop
        sharedscripts
        prerotate
            if [ -d /etc/logrotate.d/httpd-prerotate ]; then \
                    run-parts /etc/logrotate.d/httpd-prerotate; \
            fi \
        endscript
        postrotate
        datename=$(date +%Y_%m_%d_%H_%M)
            [ -s /run/nginx.pid ] && kill -USR1 `cat /run/nginx.pid`
        # ensure new log's privilege is right
        chown sysop.sysop /var/log/nginx/access.log.1
        mv /var/log/nginx/access.log.1 /var/log/nginx/access_$datename.log
        endscript
    }
