---
layout: post
title: "deploy rails app with cloud foundry"
date: 2013-03-23 14:17
comments: true
categories: [Rails, Vmc]
---

[Cloud Foundry](http://baike.baidu.com/view/8193015.htm)是VMware于2011年4月12日推出的业界第一个开源PaaS云平台，它支持多种框架、语言、运行时环境、云平台及应用服务，使开发人员能够在几秒钟内进行应用程序的部署和扩展，无需担心任何基础架构的问题。

我去年刚听说Cloud Foundry的时候就试用了下，隔了有些日子了，今天重新去使用的时候遇到了一些问题，记录下。
<!-- more -->

vmc需事先安装好了ruby
### 1.安装vmc
```sh
gem install vmc
```
如果刚安装后出现"command not found: vmc"的情况
```sh
rbenv rehash
```
我这使用的是rbenv

### 2.与Cloud Foundry建立连接
    vmc target api.cloudfoundry.com

### 3.登录cloud foundry
当然你已经注册好了Cloud Foundry的帐号，如果没有的话赶紧去[Cloud Foundry](http://my.cloudfoundry.com/signup)去注册一个
```
vmc login
```
登录成功后就可以开始部署了.

### 4.部署rails应用
cloudfoundry上不支持sqlite3，如果你使用的还是sqlite3的话，可以在Gemfile这么修改下
```ruby Gemfile
# If you use a different database in development, hide it from Cloud Foundry.
group :development do
  gem 'sqlite3'
end

# Rails 3.1 can use the latest mysql2 gem.
group :production do
  gem 'mysql2'
end
```
配置信息处也需要修改下
```ruby config/environments/production.rb
config.serve_static_assets = true
```
Bundle your application:
```sh
bundle package
bundle install
```
Assets
```sh
rake assets:precompile
```
Deploy
```
vmc push --runtime ruby19
```
发布过程
```sh
Name> example             # 设置应用的名称

Instances> 1

1: rails3
2: other
Framework> rails3

1: 64M
2: 128M
3: 256M
4: 512M
5: 1G
Memory Limit> 256M

Creating example... OK

1: example.cloudfoundry.com
2: none
Domain> example.cloudfoundry.com

Updating example... OK

Create services for application?> y

1: mongodb 2.0
2: mysql 5.1
3: postgresql 9.0
4: rabbitmq 2.4
5: redis 2.2
6: redis 2.4
7: redis 2.6
What kind?> 2

Name?> example_datebase   # 设置数据库名称

Creating service example_datebase... OK
Binding example_datebase to example... OK
Create another service?> n

Bind other services to application?> n

Save configuration?> y

Saving to manifest.yml... OK
Uploading example... OK
Starting example... OK
Checking example...
  0/1 instances: 1 starting
  0/1 instances: 1 down
  0/1 instances: 1 down
  0/1 instances: 1 flapping
Application failed to start.
```
这里解释下，vmc push后会产生一个manifest.yml的文件，刚才所有的设置都会记录在其中。

很明显这里应用没有启动成功，那问题处在哪里呢？

首先我查看下应用的状态
```
vmc apps
```
```
Getting applications... OK

name        status    usage      runtime   url                             
example     0%        1 x 256M   ruby19    example.cloudfoundry.com
```
```
vmc services
```
```
Getting services... OK

name                 service   version
example_datebase     mysql     5.1  
```
这里我们可以看到数据库和应用都是正确的，只是在启动的时候发生的意外。这时我们可以直接查看下日志
```
vmc logs example
```
```
Using manifest file manifest.yml

Getting logs for example #0... OK

Reading logs/migration.log... OK
/var/vcap/data/dea/apps/example-0-db496108f915e5d9ec906e10a6ee9f12/app/rubygems/ruby/1.9.1/gems/bundler-1.2.1/lib/bundler/source.rb:801:in `rescue in load_spec_files': git://github.com/nadarei/mina.git (at master) is not checked out. Please run `bundle install` (Bundler::GitError)
  from /var/vcap/data/dea/apps/example-0-db496108f915e5d9ec906e10a6ee9f12/app/rubygems/ruby/1.9.1/gems/bundler-1.2.1/lib/bundler/source.rb:799:in `load_spec_files'
...........
```
这里我们可以发现是mina造成的bundle的错误，修改一下相应的错误后重新发布
```
vmc push
```
```
Using manifest file manifest.yml

Uploading example... OK
Stopping example... OK

Starting example... OK
Checking example...
  0/1 instances: 1 starting
  0/1 instances: 1 starting
  0/1 instances: 1 starting
  1/1 instances: 1 running
OK
```
这次很顺利没有再出现问题，它直接调用manifest.yml保存的部署设置。访问下example.cloudfoundry.com就可以直接看到你刚部署的应用。

### 5.vmc其他命令
```
vmc help
vmc update [APP]
vmc stop [APP]
vmc start [APP]
vmc restart [APP]
vmc delete [APP]
vmc logs [APP]
vmc instances [APP]      # 列出你有的instances
vmc scale [APP]          # 更新应用的instances/memory limit
```

#### 参考：
* http://clark1231.iteye.com/blog/1770692
* http://docs.cloudfoundry.com/frameworks/ruby/rails-3-1.html
* http://docs.cloudfoundry.com/tools/vmc/vmc-quick-ref.html
* http://docs.cloudfoundry.com/tools/deploying-apps.html
* http://stackoverflow.com/questions/13808863/vmc-instances-appname-3-gives-error-unknown-app-3
