---
layout: post
title: "Gitolite 服务架设"
date: 2012-08-29 18:13
comments: true
categories: [Git]
tags: [Git, Ubuntu]
---

### 1. git安装

    安装Git-Core:
    sudo apt-get install git-core
    设置用户信息：
    git config --global user.name "Your Name"
    git config --global user.email your@email.com

### 2. 安装gitolite
##### (1)服务器端创建专用帐号

    sudo adduser --system --shell /bin/bash --group git

创建用户 git，并设置用户的 shell 为可登录的 shell，如 /bin/bash，同时添加同名的用户组。
有的系统，只允许特定的用户组(如 ssh 用户组)的用户才可以通过 SSH 协议登录，这就需要将新建的 git 用户添加到 ssh 用户组中。

    sudo adduser git ssh

为 git 用户设置用户密码。当整个 git 服务配置完成，运行正常后，建议取消 git 的口令，只允许公钥认证。

    sudo passwd git

管理员在客户端使用下面的命令，建立无口令登录：

    ssh-copy-id git@server

#### (2)gitolite的安装和升级
用git用户登录

    su git
    cd ~

安装gitolite

    git clone git://github.com/sitaramc/gitolite
    mkdir -p /home/git/bin
    gitolite/install -ln /home/git/bin

添加path

    export PATH=/home/git/bin:$PATH
    或者echo "PATH=$PATH:$HOME/bin" >> ~/.bashrc
    echo $PATH 查看（可能需要退出一下使环境变量生效）

设置gitolite的管理员（admin）

    gitolite setup -pk admin.pub

### 3. 更新gitolite

    Update your clone of the gitolite source.
    Repeat the install command you used earlier (make sure you use the same arguments as before).
    Run "gitolite setup".

### 4. 管理员克隆 gitolite-admin 管理库

    git clone git@192.168.0.106:gitolite-admin.git

#### 参考：
* https://github.com/sitaramc/gitolite
* http://www.ossxp.com/doc/git/gitolite.html

