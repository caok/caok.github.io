---
layout: post
title: "svn in ubuntu"
date: 2013-05-30 09:25
categories: [Ubuntu]
tags: [Ubuntu]
---

### 1.安装
{% highlight bash %}
sudo apt-get install subversion
svn --version  #查看是否安装成功
{% endhighlight %}

### 2.解决svn错误 SSL handshake failed: SSL error
在ubuntu下进行检出的时候出现如下报错：
    SSL handshake failed: SSL error: Key usage violation in certificate has been detected

##### 解决方法：

* sudo apt-get remove libneon27

* Download the latest libneon package from http://packages.debian.org/squeeze/libneon27 (at the bottom you can choose the right version for your architecture).
{% highlight bash %}
32位下载：http://ftp.cn.debian.org/debian/pool/main/n/neon27/libneon27_0.29.3-3_i386.deb
64位下载：http://ftp.cn.debian.org/debian/pool/main/n/neon27/libneon27_0.29.3-3_amd64.deb
{% endhighlight %}

* Install the required libssl dependency:
{% highlight bash %}
sudo apt-get install libssl0.9.8
{% endhighlight %}

* Install the downloaded libneon package. E.g. for the 64Bit architecture:
{% highlight bash %}
dpkg -i libneon27_0.29.3-3_amd64.deb 或 libneon27_0.29.3-3_i386.deb
{% endhighlight %}

* Change the symbolic links again like described above:
{% highlight bash %}
sudo mv /usr/lib/libneon-gnutls.so.27 /usr/lib/libneon-gnutls.so.27.old
sudo ln -s /usr/lib/libneon.so.27 /usr/lib/libneon-gnutls.so.27
{% endhighlight %}


### 3.简单使用svn
* 取代码
{% highlight bash %}
svn co --username xxxx --password xxxx   http://192.168.0.100/svn/xx/xx
{% endhighlight %}

* 查看状态
{% highlight bash %}
svn status
{% endhighlight %}

* 更新你的工作拷贝
{% highlight bash %}
svn update
{% endhighlight %}

* 检验修改
{% highlight bash %}
svn status
svn diff
svn revert
{% endhighlight %}

* 提交修改
{% highlight bash %}
svn commit -m "xxxxx"
{% endhighlight %}

* 清除
{% highlight bash %}
svn cleanup
{% endhighlight %}

* 导入
{% highlight bash %}
svn import
{% endhighlight %}

* 添加新文件
{% highlight bash %}
svn add test.js
{% endhighlight %}

* 删除文件
{% highlight bash %}
svn remove test.js -m "delete test.js"
{% endhighlight %}


####  参考
* http://wenku.baidu.com/view/c8b31111cc7931b765ce15cf.html
