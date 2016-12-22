---
layout: post
title:  "nginx log by logrotate"
date:   2016-12-20 22:00:00 +0800
categories: [Ubuntu]
tags: [Ubuntu Nginx]
---

在ubuntu的服务器上安装好nginx之后，会发现nginx的日志都保存在/var/log/nginx中，而且是按周为单位进行分割的，如果流量特别大而磁盘空间较小的时候，会发生磁盘写满的情况。所以这个时候我们可以调整分割的频率，让其提早打包，甚至将其转移出去。

这里我们介绍下怎么调整日志的分割频率


##### 修改配置nginx日志配置文件
位置： **/etc/logrotate.d/nginx**

{% highlight shell %}
/var/log/nginx/feedmob-access.log {
        daily
        missingok
        rotate 20
        #size 10G
        dateext
        compress
        delaycompress
        notifempty
        create 0640 www-data adm
        sharedscripts
        olddir /home/deploy/nginx_log
        prerotate
                if [ -d /etc/logrotate.d/httpd-prerotate ]; then \
                        run-parts /etc/logrotate.d/httpd-prerotate; \
                fi \
        endscript
        postrotate
                [ -s /run/nginx.pid ] && kill -USR1 `cat /run/nginx.pid`
        endscript
}
{% endhighlight %}

* weekly: 表示每周处理下日志
* daily: 表示每天处理下日志
* rotate count: 最多保持count个轮转备份
* create: 处理完该日志文件后，新生成一个日志文件，当然尽可能是同名同权限等
* dateext: 默认未加时间戳
* compress: 默认不压缩
* compress: 通过gzip 压缩转储旧的日志
* delaycompress: 和 compress 一起使用时，转储的日志文件到下一次转储时才压缩
* notifempty: 如果是空文件的话，不转储
* olddir directory：转储后的日志文件放入指定的目录，必须和当前日志文件在同一个文件系统
* size size：当日志文件到达指定的大小时才转储，Size 可以指定 bytes (缺省)以及KB (sizek)或者MB

##### 测试
{% highlight shell %}
sudo logrotate -f /etc/logrotate.d/nginx
{% endhighlight %}

#### 备注:
* https://www.cnblogs.com/futeng/p/4785206.html
* http://www.ttlsa.com/linux/logrotate-log-management-tools/
