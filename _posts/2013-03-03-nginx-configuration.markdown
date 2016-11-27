---
layout: post
title: "nginx configuration"
date: 2013-03-03 20:34
categories: [Nginx]
tags: [Nginx]
---

之前介绍了nginx的安装和简单使用，有必要简单总结下nginx在配置文件的含义。

#### 管理配置文件
Ubuntu安装之后的文件结构大致为：

* 所有的配置文件都在/etc/nginx下，并且每个虚拟主机已经安排在了/etc/nginx/sites-available下
* 程序文件在/usr/sbin/nginx
* 日志放在了/var/log/nginx中
* 并已经在/etc/init.d/下创建了启动脚本nginx
* 默认的虚拟主机的目录设置在了/var/www/nginx-default (有的版本 默认的虚拟主机的目录设置在了/var/www, 请参考/etc/nginx/sites-available里的配置)

安装完成后nginx的主配置文件被放在/etc/nginx/nginx.conf。同时一个默认的备份配置文件存在/etc/nginx/nginx.conf.default。在每次修改配置文件时都做好相应的备份，防止意外。

修改配置文件后通过重启nginx生效。

#### 全局配置(nginx.conf)
{% highlight bash %}
# 工作的子进程数量(通常设置成和CPU数量相等)
worker_processes 1;

# Nginx使用的用户和用户组
user nobody nogroup;

# 指定pid存放文件
pid /tmp/nginx.pid;

# 全局错误日志存放路径
error_log /tmp/nginx.error.log;
{% endhighlight %}

{% highlight bash %}
# 工作模式及连接数上限
events {
  # 允许最大连接数(单个后台worker_process进程的最大并发链接数)
  worker_connections 1024;

  # 使用网络IO模型linux建议epoll，FreeBSD建议采用kqueue
  use epoll;

  accept_mutex off; # "on" if nginx worker_processes > 1
} 
{% endhighlight %}

{% highlight bash %}
# 设定http服务器，利用它的反向代理功能提供负载均衡支持
http {
  # nginx 在构建的时候将会在配置路径中查找该文件
  include mime.types;

  # 默认文件类型
  default_type application/octet-stream;

  # 设定日志格式
  access_log /tmp/nginx.access.log combined;

  # sendfile 指令指定 nginx 是否调用 sendfile 函数(zero copy 方式)来输出文件,对于普通应用,必须设为on,
  # 如果用来进行下载等应用磁盘IO重负载应用,可设置为off,以平衡磁盘与网络I/O处理速度,降低系统的uptime.
  sendfile on;

  # 防止网络阻塞
  tcp_nopush on;   # off may be better for *some* Comet/long-poll stuff
  tcp_nodelay off; # on may be better for some Comet/long-poll stuff

  # gzip指令告诉nginx使用gzip压缩的方式来降低带宽使用和加快传输速度,但它也会增加cpu的使用
  # 开启gzip压缩输出
  gzip on;
  # 压缩版本(默认1.1，前端如果是squid2.5请使用1.0)
  gzip_http_version 1.0;
  gzip_proxied any;
  # 最小压缩文件大小
  gzip_min_length 500;
  # 压缩等级,9代表最多用CPU和1代表最少用CPU
  gzip_comp_level 2;
  gzip_disable "MSIE [1-6]\.";
  # 压缩类型
  gzip_types text/plain text/html text/xml text/css
             text/comma-separated-values
             text/javascript application/x-javascript
             application/atom+xml;

  # 设定负载均衡的服务器列表
  upstream app_server {
    # fail_timeout=0 means we always retry an upstream even if it failed
    # to return a good HTTP response (in case the Unicorn master nukes a
    # single worker for timing out).

    # for UNIX domain socket setups:
    server unix:/tmp/.sock fail_timeout=0;

    # for TCP setups, point these to your backend servers
    # server 192.168.0.7:8080 fail_timeout=0;
    # server 192.168.0.8:8080 fail_timeout=0;
    # server 192.168.0.9:8080 fail_timeout=0;
  }

  # 设定虚拟主机配置
  server {
    # 监听端口
    listen 80 default;
    # 定义使用 example.com 访问,域名可以有多个,用空格隔开
    server_name example.com;
    # 允许客户端请求的最大单文件字节数
    client_max_body_size 4G;
    # 长连接超时时间，单位是秒
    keepalive_timeout 5;
   
    root /u/app/example_app/current/public;
   
    try_files $uri/index.html $uri.html $uri @unicorn;
    location @unicorn {
      # 后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header Host $http_host;
      proxy_redirect off;
      proxy_pass http://localhost:9000;
    }
   
    # 定义错误提示页面，这里设置成用capistrano部署的Rails应用的错误提示页面
    error_page 500 502 503 504 /500.html;
    location = /500.html {
      root /u/app/example_app/current/public;
    }
  }
}
{% endhighlight %}

#### 参考：
* http://nginx.org/en/
* http://wiki.ubuntu.org.cn/Nginx
* http://kingj.iteye.com/blog/1420187
* http://www.nginx.cn/591.html
* http://www.nginx.cn/76.html
* http://www.2cto.com/os/201212/176520.html
* https://github.com/defunkt/unicorn/blob/master/examples/nginx.conf

