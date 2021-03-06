---
layout: post
title: "mysql设置允许外部访问"
date: 2013-02-27 20:34
categories: [DB]
tags: [Mysql]
---

#### 本地登录mysql
{% highlight bash %}
mysql -u root -p
{% endhighlight %}

执行
{% highlight sql %}
grant all on *.* to user@'client_ip' identified by 'password' with grant option;

{% endhighlight %}
例子：所有局域网客户端都可访问
{% highlight sql %}
grant all on *.* to root@'192.168.0.%' identified by 'password' with grant option;
{% endhighlight %}
这里*.*指数据库名，root为mysql的用户名，password为mysql的用户密码

例子：为example数据库设置用户example
{% highlight sql %}
grant all on example.* to example@'192.168.0.%' identified by 'password' with grant option;
{% endhighlight %}

#### 设置mysql外部能访问
在/etc/mysql/my.cnf中注释到下面这行

    bind-address = 127.0.0.1

#### 重启mysql-server

    sudo service mysql restart

#### 修改mysql密码
修改root密码为：yourpassword
{% highlight sql %}
mysql> GRANT USAGE ON *.* TO root@localhost IDENTIFIED BY 'yourpassword';
{% endhighlight %}
