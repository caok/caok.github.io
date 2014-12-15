---
layout: post
title: "First 5 Minutes Troubleshooting A Server"
date: 2013-09-03 22:58
comments: true
categories: [Ubuntu]
---

这几天看了一篇“在服务器上排除问题的头五分钟”觉得对自己很有帮助，有必要转载过来。再把里面的命令之类的稍微解释下。

原文地址：

* http://devo.ps/blog/2013/03/06/troubleshooting-5minutes-on-a-yet-unknown-box.html
* http://blog.jobbole.com/36375/

遇到服务器故障，问题出现的原因很少可以一下就想到。我们基本上都会从以下步骤入手：

### 一、尽可能搞清楚问题的前因后果

不要一下子就扎到服务器前面，你需要先搞明白对这台服务器有多少已知的情况，还有故障的具体情况。不然你很可能就是在无的放矢。

<!-- more -->

必须搞清楚的问题有：

* 故障的表现是什么？无响应？报错？
* 故障是什么时候发现的？
* 故障是否可重现？
* 有没有出现的规律（比如每小时出现一次）
* 最后一次对整个平台进行更新的内容是什么（代码、服务器等）？
* 故障影响的特定用户群是什么样的(已登录的, 退出的, 某个地域的…)?
* 基础架构（物理的、逻辑的）的文档是否能找到?
* 是否有监控平台可用? （比如Munin、Zabbix、 Nagios、 New Relic… 什么都可以）
* 是否有日志可以查看?. （比如Loggly、Airbrake、 Graylog…）
* 最后两个是最方便的信息来源，不过别抱太大希望，基本上它们都不会有。只能再继续摸索了。

### 二、有谁在?
```sh
$ w     # 目前登入系统的用户有哪些人，以及他们正在执行的程序
$ last  # 列出目前与过去登入系统的用户相关信息
```
用这两个命令看看都有谁在线，有哪些用户访问过。这不是什么关键步骤，不过最好别在其他用户正干活的时候来调试系统。有道是一山不容二虎嘛。

### 三、之前发生了什么?
```sh
$ history
```
查看一下之前服务器上执行过的命令。看一下总是没错的，加上前面看的谁登录过的信息，应该有点用。另外作为admin要注意，不要利用自己的权限去侵犯别人的隐私哦。

到这里先提醒一下，等会你可能会需要更新 HISTTIMEFORMAT 环境变量来显示这些命令被执行的时间。对要不然光看到一堆不知道啥时候执行的命令，同样会令人抓狂的。

设置如下:

```sh
$ vi .bash_profile

添加以下几行
HISTFILESIZE=2000
HISTSIZE=2000
HISTTIMEFORMAT="[%Y-%m-%d %H:%M:%S] "
export HISTTIMEFORMAT

$ source .bash_profile
```

### 四、现在在运行的进程是啥?
```sh
$ pstree -a
$ ps aux
```
这都是查看现有进程的。 ps aux 的结果比较杂乱， pstree -a 的结果比较简单明了，可以看到正在运行的进程及相关用户。

### 五、监听的网络服务
```sh
$ netstat -ntlp
$ netstat -nulp
$ netstat -nxlp
```
我一般都分开运行这三个命令，不想一下子看到列出一大堆所有的服务。netstat -nalp倒也可以。不过我绝不会用 numeric 选项 （鄙人一点浅薄的看法：IP 地址看起来更方便）。

netstat常用参数:
```
-a (all)显示所有选项，默认不显示LISTEN相关
-t (tcp)仅显示tcp相关选项
-u (udp)仅显示udp相关选项
-n 拒绝显示别名，能显示数字的全部转化成数字。
-l 仅列出有在 Listen (监听) 的服務状态

-p 显示建立相关链接的程序名
-r 显示路由信息，路由表
-e 显示扩展信息，例如uid等
-s 按各个协议进行统计
-c 每隔一个固定时间，执行该netstat命令。
```

更多的netstat的使用可以参考: [http://www.cnblogs.com/ggjucheng/archive/2012/01/08/2316661.html](http://www.cnblogs.com/ggjucheng/archive/2012/01/08/2316661.html)

找到所有正在运行的服务，检查它们是否应该运行。查看各个监听端口。在netstat显示的服务列表中的PID 和 ps aux 进程列表中的是一样的。

如果服务器上有好几个Java或者Erlang什么的进程在同时运行，能够按PID分别找到每个进程就很重要了。

通常我们建议每台服务器上运行的服务少一点，必要时可以增加服务器。如果你看到一台服务器上有三四十个监听端口开着，那还是做个记录，回头有空的时候清理一下，重新组织一下服务器。

### 六、CPU 和内存
```sh
$ free    # 内存使用状况
$ uptime  # 用于获取主机运行时间和查询linux系统负载
$ top     # 进程的状况
$ htop    # 交互式的进程浏览器，可以用来替换Linux下的top命令
```
注意以下问题:

* 还有空余的内存吗? 服务器是否正在内存和硬盘之间进行swap?
* 还有剩余的CPU吗? 服务器是几核的? 是否有某些CPU核负载过多了?
* 服务器最大的负载来自什么地方? 平均负载是多少?

```
$ uptime
显示结果为：
10:19:04 up 257 days, 18:56,  12 users,  load average: 2.10, 2.10,2.09

显示内容说明：
10:19:04             # 系统当前时间
up 257 days, 18:56   # 主机已运行时间,时间越大，说明你的机器越稳定。
12 user              # 用户连接数，是总连接数而不是用户数
load average         # 系统平均负载，统计最近1，5，15分钟的系统平均负载
```

### 七、硬件
```sh
$ lspci      # 显示系统中所有PCI总线设备或连接到该总线上的所有设备
$ dmidecode  # 查看机器所有硬件信息
$ dmesg      # 查看开机启动信息
$ ethtool    # 查看网卡信息
$ cat /proc/cpuinfo    # CPU相关的参数
$ getconf LONG_BIT     # 查看CPU的位数
$ cat /proc/partitions # 查看磁盘信息
$ fdisk -l             # 系统上的磁盘(包括U盘)的分区以及大小相关信息
```
有很多服务器还是裸机状态，可以看一下：

找到RAID卡(是否带BBU备用电池?)、CPU、空余的内存插槽。根据这些情况可以大致了解硬件问题的来源和性能改进的办法。

网卡是否设置好? 是否正运行在半双工状态? 速度是10MBps? 有没有 TX/RX 报错?

### 八、IO 性能
```sh
$ iostat -kx 2
$ vmstat 2 10
$ mpstat 2 10
$ dstat --top-io --top-bio
```
这些命令对于调试后端性能非常有用。

* 检查磁盘使用量：服务器硬盘是否已满?
* 是否开启了swap交换模式 (si/so)?
* CPU被谁占用：系统进程? 用户进程? 虚拟机?
* dstat 是我的最爱。用它可以看到谁在进行 IO： 是不是MySQL吃掉了所有的系统资源? 还是你的PHP进程?
 
### 九、挂载点 和 文件系统
```sh
$ mount
$ cat /etc/fstab
$ vgs
$ pvs
$ lvs
$ df -h
$ lsof +D / /* beware not to kill your box */
```
* 一共挂载了多少文件系统?
* 有没有某个服务专用的文件系统? (比如MySQL?)
* 文件系统的挂载选项是什么： noatime? default? 有没有文件系统被重新挂载为只读模式了？
* 磁盘空间是否还有剩余?
* 是否有大文件被删除但没有清空?
* 如果磁盘空间有问题，你是否还有空间来扩展一个分区？
 
### 十、内核、中断和网络
```sh
$ sysctl -a | grep ...
$ cat /proc/interrupts
$ cat /proc/net/ip_conntrack /* may take some time on busy servers */
$ netstat
$ ss -s
```
* 你的中断请求是否是均衡地分配给CPU处理，还是会有某个CPU的核因为大量的网络中断请求或者RAID请求而过载了？
* SWAP交换的设置是什么？对于工作站来说swappinness 设为 60 就很好, 不过对于服务器就太糟了：你最好永远不要让服务器做SWAP交换，不然对磁盘的读写会锁死SWAP进程。
* conntrack_max 是否设的足够大，能应付你服务器的流量?
* 在不同状态下(TIME_WAIT, …)TCP连接时间的设置是怎样的？
* 如果要显示所有存在的连接，netstat 会比较慢， 你可以先用 ss 看一下总体情况。
* 你还可以看一下 Linux TCP tuning 了解网络性能调优的一些要点。

### 十一、系统日志和内核消息
```sh
$ dmesg
$ less /var/log/messages
$ less /var/log/secure
$ less /var/log/auth
```
* 查看错误和警告消息，比如看看是不是很多关于连接数过多导致？
* 看看是否有硬件错误或文件系统错误?
* 分析是否能将这些错误事件和前面发现的疑点进行时间上的比对。
 
### 十二、定时任务
```sh
$ ls /etc/cron* + cat
$ for user in $(cat /etc/passwd | cut -f1 -d:); do crontab -l -u $user; done
```
* 是否有某个定时任务运行过于频繁?
* 是否有些用户提交了隐藏的定时任务?
* 在出现故障的时候，是否正好有某个备份任务在执行？
 

### 十三、应用系统日志
这里边可分析的东西就多了, 不过恐怕你作为运维人员是没功夫去仔细研究它的。关注那些明显的问题，比如在一个典型的LAMP（Linux+Apache+Mysql+Perl）应用环境里:

* Apache & Nginx: 查找访问和错误日志, 直接找 5xx 错误, 再看看是否有 limit_zone 错误。
* MySQL:          在mysql.log找错误消息，看看有没有结构损坏的表， 是否有innodb修复进程在运行，是否有disk/index/query 问题.
* PHP-FPM:        如果设定了 php-slow 日志, 直接找错误信息 (php, mysql, memcache, …)，如果没设定，赶紧设定。
* Varnish:        在varnishlog 和 varnishstat 里, 检查 hit/miss比. 看看配置信息里是否遗漏了什么规则，使最终用户可以直接攻击你的后端？
* HA-Proxy:       后端的状况如何？健康状况检查是否成功？是前端还是后端的队列大小达到最大值了？
 
### 结论

经过这5分钟之后，你应该对如下情况比较清楚了：

* 在服务器上运行的都是些啥？
* 这个故障看起来是和 IO/硬件/网络 或者 系统配置 (有问题的代码、系统内核调优, …)相关。
* 这个故障是否有你熟悉的一些特征？比如对数据库索引使用不当，或者太多的apache后台进程。
