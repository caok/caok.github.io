---
layout: post
title: "在linux中配置开发环境"
date: 2013-01-11 22:39
comments: true
categories: [Ubuntu]
---

最近重装了下系统，换成了Mint Linux,就将看到的一些关于开发环境配置的重新整理了下，主要是参考老大写的
<a href="http://wongyouth.github.com/blog/2012/06/20/setup-new-ubuntu-environment/">博文</a>
，又往里增加了些。
<!-- more -->

###更新源
    sudo apt-get update

###安装系统包
```sh
sudo apt-get -y install git-core curl zsh exuberant-ctags vim autoconf automake openssl \
  build-essential libc6-dev libreadline6 libreadline6-dev zlib1g zlib1g-dev libssl-dev libyaml-dev \
  mysql-server libmysqlclient-dev libsqlite3-0 libsqlite3-dev sqlite3 \
  ncurses-dev libtool bison libxslt1-dev libxml2-dev libqt4-dev
```

nokogiri.gem need libxml2 libxml2-dev libxslt1-dev mysql2.gem need libmysqlclient-dev capybara-webkit.gem need libqt4-dev

###安装PostgreSQL
    sudo apt-get install postgresql-9.1 libpq-dev

修改 /etc/postgresql/9.1/main/pg_hba.conf 权限控制，本地 TCP 访问设置为完全信任
```sh
#host    all             all             127.0.0.1/32            md5
host    all             all             127.0.0.1/32            trust
```

修改后要重启

    sudo service postgresql restart

###安装oh-my-zsh，如果你使用bash可以跳过此步骤
    curl -L https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh | bash

###切换到zsh
    chsh -s `which zsh`

###设置vim文件，安装常用的vim插件
    curl https://raw.github.com/wongyouth/vimfiles/master/install.sh | bash


###安装ruby环境管理工具rbenv
    curl https://raw.github.com/fesplugas/rbenv-installer/master/bin/rbenv-installer | bash
    rbenv bootstrap-ubuntu-12-04

####安装ruby1.9
```sh
rbenv install -l      # 查看Ruby Available versions
rbenv install 1.9.3-p194

rbenv global 1.9.3-p194      # 默认使用 1.9.3-p194
rbenv shell 1.9.3-p194       # 当前的shell使用 1.9.3-p194, 会设置一个 `RBENV_VERSION` 环境变量
rbenv local jruby-1.6.7      # 当前目录使用 jruby-1.6.7, 会生成一个 `.rbenv-version` 文件
```
如果查看Ruby Available versions中没有最新的ruby版本的话，可以更新rbenv和ruby-build
```sh
# 更新rbenv
cd ~/.rbenv
git pull
# 更新ruby-build
cd ~/.rbenv/plugins/ruby-build
git pull
```

####更新gem 并安装 bundler rake
    gem update --system
    gem install bundler rake

####更新rbenv的shim，使rake, bundle命令可以直接使用
    rbenv rehash

###或者可以使用rvm
听别人说rvm不错也试着用了下
####安装rvm
```sh
curl -L get.rvm.io | bash -s stable
source ~/.bashrc
source ~/.bash_profile
```
修改 RVM 的 Ruby 安装源到国内的 淘宝镜像服务器，这样能提高安装速度
    sed -i -e 's/ftp\.ruby-lang\.org\/pub\/ruby/ruby\.taobao\.org\/mirrors\/ruby/g' ~/.rvm/config/db

若遇到rvm无法识别的提示，就在.zshrc或.bashrc文件中加入：
    [[ -s "/home/clark/.rvm/scripts/rvm" ]] && source "/home/clark/.rvm/scripts/rvm"

####ruby的安装与切换
    列出已知的ruby版本
    rvm list known
    安装一个ruby版本
    rvm install 1.9.3
    这里安装了最新的1.9.3, rvm list known列表里面的都可以拿来安装。

####使用一个ruby版本
    rvm use 1.9.3
####查询已经安装的ruby
    rvm list
####卸载一个已安装版本
    rvm remove 1.9.2
####如果想设置为默认版本
    rvm use 1.9.3 --default

####更新gem 并安装 bundler rake
    gem update --system
    gem install bundler rake

项目自动加载gemset
rvm还可以自动加载gemset.

例如我们有一个rails3.1.3项目，需要1.9.3版本ruby.整个流程可以这样。
```sh
rvm install 1.9.3
rvm use 1.9.3
rvm gemset create rails313
rvm use 1.9.3@rails313
```
下面进入到项目目录，建立一个.rvmrc文件。
在这个文件里可以很简单的加一个命令：
    rvm use 1.9.3@rails313

###配置主目录里面常用的dotfiles文件
```sh
git clone git://github.com/wongyouth/dotfiles ~/.dotfiles
cd ~/.dotfiles
rake install
```
###输入法安装
####1.安装
    sudo apt-get install im-switch fcitx fcitx-googlepinyin

####2.将fcitx设为系统默认输入法：
    System Setting >>Language surpport >> Language >> Keyborad input method system >> fcitx
    或者
    im-switch -s fcitx -z default #修改当前用户的默认输入法

####3.fictx开机自启动：
System Setting >> Startup Applications >> Add

    Name: Fcitx
    Command: /usr/bin/fcitx -d
    Comment:Fcitx startup

####4.重启
如果之前已经有装过ibus或者scim之类的输入法，最好先卸载掉，防止冲突

###man命令彩色高亮显示
在~/.bashrc(或者.zshrc)中加入如下代码：
```sh
export LESS_TERMCAP_mb=$'\E[01;31m'
export LESS_TERMCAP_md=$'\E[01;31m'
export LESS_TERMCAP_me=$'\E[0m'
export LESS_TERMCAP_se=$'\E[0m'
export LESS_TERMCAP_so=$'\E[01;44;33m'
export LESS_TERMCAP_ue=$'\E[0m'
export LESS_TERMCAP_us=$'\E[01;32m'
```
    source .bashrc 直接生效
重启启动终端即可。

### 参考：
* http://ruby-china.org/wiki/rbenv-guide
* http://ruby-china.org/wiki/rvm-guide
* https://github.com/sstephenson/rbenv
* https://github.com/huacnlee/init.d
