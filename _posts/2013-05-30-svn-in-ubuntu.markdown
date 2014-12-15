---
layout: post
title: "svn in ubuntu"
date: 2013-05-30 09:25
comments: true
categories: [Ubuntu]
---

### 1.安装
```sh
sudo apt-get install subversion
svn --version  #查看是否安装成功
```

### 2.解决svn错误 SSL handshake failed: SSL error
在ubuntu下进行检出的时候出现如下报错：
    SSL handshake failed: SSL error: Key usage violation in certificate has been detected

<!-- more -->

##### 解决方法：

* sudo apt-get remove libneon27

* Download the latest libneon package from http://packages.debian.org/squeeze/libneon27 (at the bottom you can choose the right version for your architecture).
```
32位下载：http://ftp.cn.debian.org/debian/pool/main/n/neon27/libneon27_0.29.3-3_i386.deb
64位下载：http://ftp.cn.debian.org/debian/pool/main/n/neon27/libneon27_0.29.3-3_amd64.deb
```
* Install the required libssl dependency:
```sh
sudo apt-get install libssl0.9.8
```
* Install the downloaded libneon package. E.g. for the 64Bit architecture:
```sh
dpkg -i libneon27_0.29.3-3_amd64.deb 或 libneon27_0.29.3-3_i386.deb
```
* Change the symbolic links again like described above:
```sh
sudo mv /usr/lib/libneon-gnutls.so.27 /usr/lib/libneon-gnutls.so.27.old
sudo ln -s /usr/lib/libneon.so.27 /usr/lib/libneon-gnutls.so.27
```

### 3.简单使用svn
* 取代码
```
svn co --username xxxx --password xxxx   http://192.168.0.100/svn/xx/xx
```
* 查看状态
```
svn status
```
* 更新你的工作拷贝
```
svn update
```
* 检验修改
```
svn status
svn diff
svn revert
```
* 提交修改
```
svn commit -m "xxxxx"
```
* 清除
```
svn cleanup
```
* 导入
```
svn import
```
* 添加新文件
```
svn add test.js
```
* 删除文件
```
svn remove test.js -m "delete test.js"
```

####  参考
* http://wenku.baidu.com/view/c8b31111cc7931b765ce15cf.html
