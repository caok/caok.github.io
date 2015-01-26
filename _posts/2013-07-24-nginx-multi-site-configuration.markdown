---
layout: post
title: "Nginx multi-site configuration"
date: 2013-07-24 09:43
comments: true
categories: [Nginx, Ubuntu]
---

简单介绍下nginx配置多个站点的情况。

这里以配置2个站点（2个域名）为例，n 个站点可以相应增加调整，假设：
{% highlight bash %}
IP地址: 202.55.1.100
域名1 example1.com 放在 /www/example1
域名2 example2.com 放在 /www/example2
{% endhighlight %}

最简单的处理:
{% highlight bash %}
http {
  index index.html;
 
  server {
    server_name example1.com;
    root /www/example1;
  }
 
  server {
    server_name example2.com;
    root /www/example2;
  }
}
{% endhighlight %}

但这样站点一多不好管理，可以给每个站点都创建一个单独的配置文件,主要是通过virtual去设置。

配置 nginx virtual hosting 的基本思路和步骤如下：
{% highlight bash %}
把2个站点 example1.com, example2.com 放到 nginx 可以访问的目录 /www/
给每个站点分别创建一个 nginx 配置文件 example1.com.conf，example2.com.conf, 并把配置文件放到 /etc/nginx/vhosts/
然后在 /etc/nginx.conf 里面加一句 include 把步骤2创建的配置文件全部包含进来（用 * 号）
重启 nginx
{% endhighlight %}

查看nginx.conf这个配置文件后，发现其实我们可以将每个站点的配置文件放在sites-enabled中即可
{% highlight bash %}
  include /etc/nginx/conf.d/*.conf;
  include /etc/nginx/sites-enabled/*;
{% endhighlight %}

### 具体过程
#### 1、在 /etc/nginx/sites-available/ 里创建一个名字为 example1.conf 的文件，把以下内容拷进去
{% highlight bash %}
# example1.conf
server {
  server_name example1.com;

  root /www/example1;
  index index.html;
}
{% endhighlight %}

#### 2、在 /etc/nginx/sites-available/ 里创建一个名字为 example2.conf 的文件，把以下内容拷进去
{% highlight bash %}
# example2.conf
server {
  server_name example2.com;

  root /www/example2;
  index index.html;
}
{% endhighlight %}

#### 3、将之前的配置关联进sites-enabled
{% highlight bash %}
cd /etc/nginx/sites-enabled
ls -l
sudo ln -s /etc/nginx/sites-available/example1.conf example1
sudo ln -s /etc/nginx/sites-available/example2.conf example2
{% endhighlight %}

#### 4、重启 Nginx
{% highlight bash %}
sudo /etc/init.d/nginx restart
or
sudo service nginx restart
{% endhighlight %}

想要立刻看到效果的话，可以在/etc/hosts中将域名指向本机，即可查看刚才配置的效果

#### 参考:
* http://www.jb51.net/article/27533.htm
* http://www.open-open.com/lib/view/open1347676848681.html
* http://www.open-open.com/bbs/view/1319455592515
