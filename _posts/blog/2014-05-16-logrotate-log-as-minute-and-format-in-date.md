---
layout: post
title: "Use logrotate: rotate as minute and customize new log filename"
categories: blog
---

logrotate.d
------------
Create a configuration file in `/etc/logrotate.d/` like:

    /var/log/nginx/YOURLOGFILE {
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
            mv /var/log/nginx/YOURLOGFILE.1 /var/log/nginx/CUSTOMIZED_$datename.log
        endscript
    }

crontab
-----------
Since logrotate does not support rotate every minute, I have to log in as *root* then modify its crontab like:
`* * * * * /usr/sbin/logrotate -f /etc/logrotate.d/YOURCONF`

Be especially careful about yout logrotate's absolute path. You could debug your logrotate behavior using `logrotate -df /etc/logrotate.d/YOURCONF`
