---
layout: post
title: "make your site ssl enabled"
date: 2014-07-13 08:21
categories: [Web]
tags: [Web, SSL]
---

ssl是什么，ssl就是你在网址里面看到的https。区别于http，如果是ssl的，那么他所传输的数据是保密的， 别人无法根据监听网络偷取你的信用卡数据，登录密码。现在国内的银行等大型网站基本上都支持ssl了。
SSL是Secure Socket Layer的简称，具体的作用就是在部署了SSL证书的网站跟用户浏览器之间建立一个安全的会话。以防止信息在互联网任何中间节点上被盗用。其工作原理可见[这里](https://zh.wikipedia.org/wiki/%E5%AE%89%E5%85%A8%E5%A5%97%E6%8E%A5%E5%B1%82)

你可以上[rapidsslonline](https://www.rapidsslonline.com/)或Godadday上购买SSL证书.

### 现在以Godadday为例,简单介绍下整个的步骤:

* 1.去Godadday上购买SSL证书
* 2.在部署的服务器上生成csr文件

{% highlight bash %}
openssl req -new -newkey rsa:2048 -nodes -keyout domain.key -out domain.csr
{% endhighlight %}

不同的web服务器生成的方式可能有所不同,可参考[ssl-certificates-csr-generation-instructions](http://support.godaddy.com/help/category/746/ssl-certificates-csr-generation-instructions)

#### 3.將生成的csr文件中的內容拷貝到你的ssl账户中
过一段时间后,你就能收到最终的证书(两个crt文件)

#### 4.在nginx中配置SSL certificate chains
{% highlight bash %}
cat domain.crt bundle.crt > domain.chained.crt
{% endhighlight %}

{% highlight bash %}
server {
  listen 443 ssl;
  server_name domain.com;

  ssl                 on;
  ssl_certificate     /etc/nginx/ssl/domain.chained.crt;
  ssl_certificate_key /etc/nginx/ssl/domain.key;
  ssl_protocols       SSLv3 TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers         HIGH:!aNULL:!MD5;
  ssl_session_cache    shared:SSL:10m;
  ssl_session_timeout  10m;
}
{% endhighlight %}

如果想把http的请求转到https的话：
{% highlight bash %}
server {
  listen      80;
  server_name domain.com;
  rewrite     ^ https://$server_name$request_uri? permanent;
}
{% endhighlight %}

#### 5.检测
最后你可以在[ssllabs](https://www.ssllabs.com/ssltest/index.html)上检测ssl最终的情况.

#### 6.提升评分等级
##### Fixing the "Weak Diffie-Hellman"
{% highlight bash %}
openssl dhparam -out /etc/ssl/dhparams2048.pem 2048
{% endhighlight %}
在nginx配置中增加:
{% highlight bash %}
ssl_dhparam /etc/ssl/dhparams2048.pem
{% endhighlight %}

所以nginx中关于ssl的完整配置为
{% highlight bash %}
listen 443 ssl;
server_name caok1231.com;
ssl on;
ssl_certificate /etc/nginx/ssl/*.crt;
ssl_certificate_key /etc/nginx/ssl/*.key;
ssl_session_timeout 1d;
ssl_session_cache shared:SSL:50m;

# Diffie-Hellman parameter for DHE ciphersuites, recommended 2048 bits
ssl_dhparam /etc/ssl/dhparams2048.pem;

# intermediate configuration
ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
ssl_ciphers 'ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS';
ssl_prefer_server_ciphers on;

# OCSP
ssl_stapling on;
ssl_stapling_verify on;

# HSTS (ngx_http_headers_module is required) (15768000 seconds = 6 months)
add_header Strict-Transport-Security max-age=15768000;
{% endhighlight %}


### 备注:
* http://wongyouth.com/blog/2013/01/05/make-your-site-ssl-enabled/
* https://hisea.me/p/ssl-config-with-nginx
* http://nginx.org/en/docs/http/configuring_https_servers.html#chains
* http://support.godaddy.com/help/article/3601/generating-a-certificate-signing-request-nginx
* https://www.codeproject.com/articles/1107743/webcontrols/
* http://www.claudioborges.org/?p=521
* https://www.peterdavehello.org/2015/10/build-an-a-plus-best-practice-https-web-server-via-nginx-chinese-version/
