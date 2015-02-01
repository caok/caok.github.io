---
layout: post
title:  "PostgreSQL"
date:   2014-12-20 22:10:57
categories: postgresql
---

### 安装
{% highlight bash %}
sudo apt-get install postgresql
sudo apt-get install libpq-dev
# for hstore
sudo apt-get install postgresql-contrib
{% endhighlight %}

### 配置用户
{% highlight bash %}
sudo -u postgres psql template1
CREATE USER chideo WITH PASSWORD 'Wagner01'; 
alter user chideo with superuser;
\q
{% endhighlight %}

修改 /etc/postgresql/9.3/main/pg_hba.conf 权限控制，本地 TCP 访问设置为完全信任
{% highlight bash %}
#host    all             all             127.0.0.1/32            md5
host    all             all             127.0.0.1/32            trust
{% endhighlight %}

修改后要重启
{% highlight bash %}
sudo service postgresql restart
{% endhighlight %}

###备份
{% highlight bash %}
Uncompressed:

$ pg_dump -h IP_ADDRESS -p 5432 -U app -N postgis -N topology -d DATABASE_NAME > your_file_name.sql
Compressed:

$ pg_dump -h IP_ADDRESS -p 5432 -U app -a -N postgis -N topology -Fc -d DATABASE_NAME > your_file_name.dump
The variables

IP_ADDRESS = The IP address of your database server

DATABASE_NAME = The database name of your server (found on the Database tab of your app)

The flags

-h = Host
-p = Port
-U = User
-d = Database name
-N = Exclude schema (in particular, exclude the PostGIS and topology schema if you aren’t using any of their geographic functionality)
-Fc = Format compressed
Optional flags

-a = Data only
-c = Clean
{% endhighlight %}

### 导入
{% highlight bash %}
psql -h IP_ADDRESS -p 5432 -U app -d databasename -f your_file_name.sql
{% endhighlight %}

连接数据库, 默认的用户和数据库是postgres
{% highlight bash %}
psql -U user -d dbname
{% endhighlight %}

切换数据库,相当于mysql的use dbname
{% highlight bash %}
\c dbname
{% endhighlight %}

列举数据库，相当于mysql的show databases
{% highlight bash %}
\l
{% endhighlight %}

列举表，相当于mysql的show tables
{% highlight bash %}
\dt
{% endhighlight %}

查看表结构，相当于desc tblname,show columns from tbname
{% highlight bash %}
\d tblname
{% endhighlight %}

