---
layout: post
title:  "nginx nr agent"
date:   2016-12-10 22:00:00 +0800
categories: [Ubuntu]
tags: [Ubuntu Nginx]
---

#### 1.安装nginx-nr-agent
{% highlight shell %}
wget http://nginx.org/packages/ubuntu/pool/nginx/n/nginx-nr-agent/nginx-nr-agent_2.0.0-9_all.deb
sudo apt-get install python-daemon python-setproctitle
sudo dpkg -i nginx-nr-agent_2.0.0-9_all.deb
{% endhighlight %}

#### 2.修改配置
修改 /etc/nginx-nr-agent/nginx-nr-agent.ini

{% highlight shell %}
newrelic_license_key=YOUR_LICENSE_KEY_HERE

# data sources settings
[source1]
name=nginxtips.com
url=http://www.scalescale.com/tips/nginx/status
{% endhighlight %}

#### 3.重启服务
{% highlight shell %}
sudo service nginx-nr-agent start
{% endhighlight %}
