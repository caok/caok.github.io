---
layout: post
title: "Ubuntu添加sudo用户"
date: 2013-09-06 20:58
comments: true
categories: [Ubuntu]
---


Ubuntu系统默认不允许root用户直接登录。安装系统时，建立的普通用户默认是可以使用sudo的。进入系统后，新添加的用户默认无法使用sudo命令，如何可以使得普通用户可以使用sudo命令呢？具体做法如下：

### 方法1
通过交互的方式增加新的账户clark
```sh
$ sudo adduser clark
```
新增加的账户是没有sudo权限的，针对ubuntu 12.04，可以使得用户clark属于sudo组和adm组
```
$ sudo vi /etc/group
修改
adm:x:4:clark
sudo:x:27:clark
```
<!-- more -->

### 方法2
增加新的账户
```sh
$ useradd -s /bin/bash -mr  ***（你要添加的账号名称）
```
修改密码
```sh
$ paddwd ***(新添加的账号名称)
```
其中，useradd的参数说明可以使用useradd --help查看。

* -m 为创建账号主目录，默认不创建。
* -r 为创建系统管理员账号
* -s 指定shell环境(默认的是sh)

删除用户clark： 
```sh
$ sudo userdel clark
```
