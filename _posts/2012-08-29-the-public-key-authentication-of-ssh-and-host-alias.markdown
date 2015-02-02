---
layout: post
title: "SSH公钥认证和主机别名"
date: 2012-08-29 19:01
comments: true
categories: [Ubuntu, SSH]
---

### 1 SSH 公钥认证
(1)生成公钥

    ssh-keygen

(2)实现无口令登录远程服务器，即用公钥认证取代口令认证

    ssh-copy-id -i .ssh/id_rsa.pub user@server

### 2 SSH 主机别名
(1)创建指定名称的公钥/私钥对

    ssh-keygen -f ~/.ssh/<filename>

注：

> a.将 <filename> 替换为有意义的名称。

> b.会在 ~/.ssh 目录下创建指定的公钥/私钥对。 文件 <filename> 是私钥，文件 <filename>.pub 是公钥。

(2)将新生成的公钥添加到远程主机的 .ssh/authorized_keys 文件中，建立新的公钥认证

    ssh-copy-id -i .ssh/<filename>.pub user@server

SSH的客户端配置文件 ~/.ssh/config 可以通过创建主机别名，在连接主机时，使用特定的公钥。例如 ~/.ssh/config 文件中的下列配置：

    host bj
      user git
      hostname bj.ossxp.com
      port 22
      identityfile ~/.ssh/jiangxin

当执行

    $ ssh bj

或者执行

    $ git clone bj:path/to/repo.git

含义为：

    登录的 SSH 主机为 bj.ossxp.com 。
    登录时使用的用户名为 git 。
    认证时使用的公钥文件为 ~/.ssh/jiangxin.pub 。

#### 参考：
http://www.ossxp.com/doc/git/gitolite.html

