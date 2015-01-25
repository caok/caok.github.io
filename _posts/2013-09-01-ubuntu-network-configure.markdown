---
layout: post
title: "ubuntu network configure"
date: 2013-09-01 07:45
comments: true
categories: [Ubuntu]
---

最近经手的一台服务器主板换了，导致了一些连接网络的问题，把相应的处理总结一下，以便下次再遇到时能不用在纠结那么久。

#### 问题描述：
无法上网，ping 127.0.0.1能通，但局域网无法ping通。

### 安装网卡驱动
刚开始认为是更换主板后网卡未能识别，需要安装相应的网卡驱动，于是就朝这方面去努力。

#### 1.识别当前的网卡
{% highlight ruby %}
lspci  #查看网卡类型
{% endhighlight %}

从中我发现该服务器的网卡为Broadcom NetExtreme II 5722,当然如果你知道你服务器类型的话也可以直接上官网查询，顺带把相应的网卡驱动给下载了。

#### 2.rpm转换为deb
找到网卡驱动后，发现只提供了rpm的安装，所以可以通过alien转换为deb的包，这个过程中遇到了一些麻烦，服务器是64位的，而自己的电脑是32位的，弄得只能在vps上去处理。

{% highlight ruby %}
uname -m                                    # 查看系统位数
sudo alien xxxx.deb                         # 将rpm转换为deb
scp xxx.rpm  yourname@192.168.xxx.xxx:~/    # 上传文件到服务器
scp yourname@192.168.xxx.xxx:~/xxx.deb .    # 从服务器下载文件到本地
{% endhighlight %}

#### 3.通过U盘拷贝文件到服务器
由于没有网络，只能通过u盘拷贝驱动程序到服务器

##### a.首先识别磁盘设备
{% highlight ruby %}
sudo fdisk -l
{% endhighlight %}

查出我们的U盘为/dev/sdb1

##### b.挂载
{% highlight ruby %}
id yourname   #查当用户的uid和pid
mkdir usb     #设置将挂载的哪个文件夹
sudo mount /dev/sdb1 ~/usb -o uid=1000,gid=1000 #将U盘挂载到~/usb这个文件夹中
{% endhighlight %}

如果挂载时不指定uid和gid，访问U盘时，U盘中文件的权限不对应，会出现一些不必要的问题

##### c.安装
{% highlight ruby %}
sudo dpkg -i xxx.deb
{% endhighlight %}

#### 3.重启网络
{% highlight ruby %}
sudo /etc/init.d/networking restart
{% endhighlight %}

### 真正的问题
至此原本以为问题解决了，但现实是残酷的，ifconfig时仍只有一个lo，通过ifconfig -a我发现还有个没有激活的网络连接居然叫"eth1"，但我在/etc/network/interface中配置的明明是"eth0",那是不是name和mac地址没对应上造成的呢？于是我把 /etc/network/interface 和 /etc/udev/rules.d/70-persistent-net.rules 修改的一致，都改为"eth1",重启网络后发现ok了，于是关机睡觉。

第二天再次开机时，发现 ifconfig -a 中原本的 "eth1" 又变成了 "eth0"，擦。。。哥怒了。。。

仔细在网上找了下，发现原来是：尽管我们更换了主板(或网卡)，但是原先的网卡信息依旧会存储下来

我用 dmesg 查询开机过程的讯息，看有没有跟网卡相关的，结果发现原来是 udev 服务在开机过程中把其名称给换了："eth1" --> "eth0"

知道问题后，就好解决了，方法有两种：

* 1.删除/etc/udev/rules.d/70-persistent-net.rules
* 2.在/etc/network/interfaces的配置里同时补上"eth0"和"eth1"的

重启发现一切ok

#### 参考:
* http://clark1231.iteye.com/blog/1561400
* http://clark1231.iteye.com/blog/1561970
* http://www.cnblogs.com/laipDIDI/articles/2202262.html
* http://wenku.baidu.com/view/a883aa49fe4733687e21aaa7.html
