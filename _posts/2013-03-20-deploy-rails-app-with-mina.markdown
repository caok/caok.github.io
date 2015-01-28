---
layout: post
title: "deploy rails app with mina"
date: 2013-03-20 21:10
comments: true
categories: [Rails, Mina]
---

capistrano是使用的最多的部署工具，最近社区中不少人开始推荐[mina](https://github.com/nadarei/mina),试着用了下部署的速度的确快了很多。虽然mina最近两个月都没有更新，不过仍然希望这个项目能得到很好的发展。

下面简单总结下使用mina简单部署rails应用的过程。

### 1.安装mina
{% highlight ruby %}
# Gemfile
group :development do
  gem 'mina', :git => 'git://github.com/nadarei/mina.git'
end
{% endhighlight %}

使用mina 0.2.1的时候可能会出现[Mina hangs after entering SSH password](https://github.com/nadarei/mina/issues/88), 而0.2.0的版本没有这个问题,但为了使用最新的版本可以直接从原项目中取(这样的话在执行mina命令时要加上bundle exec).

### 2.初始化mina
{% highlight bash %}
bundle
bundle exec mina init
{% endhighlight %}

它将创建部署文件config/deploy.rb

### 3.创建你的服务器
{% highlight bash %}
$ ssh username@your.server.com

# Once in your server, create the deploy folder:
~@your.server.com$ mkdir /var/www/foobar.com
~@your.server.com$ chown -R username /var/www/foobar.com
{% endhighlight %}

这样可以避免部署时出现的sudo的错误

### 4.配置mina
{% highlight ruby %}
# config/deploy.rb
require 'mina/bundler'
require 'mina/rails'
require 'mina/git'
require 'mina/rbenv'  # for rbenv support. (http://rbenv.org)
# require 'mina/rvm'    # for rvm support. (http://rvm.io)

set :domain, 'foobar.com'                 # 设置你的ip地址或域名
set :deploy_to, '/var/www/foobar.com'     # 设置部署的路径
set :repository, 'git://...'              # git地址
#set :repository, File.expand_path('../../.git/', __FILE__)  #直接取本地的git项目
set :branch, 'master'                     # 确定代码分支
 
# 设置需要共享的文件
set :shared_paths, ['config/database.yml', 'log', 'tmp']
 
# 可选设置
set :user, 'foobar'    # SSH 用户名.
# set :port, '30000'   # SSH 端口，默认22.
 
# 设置对于大多数的命令(mina deploy或mina rake)都需要预先加载的环境
task :environment do
  # 如果使用的是rbenv,这么设置,但需确保.rbenv-version(rbenv local 1.9.3-p374)已经存在于你的项目中
  invoke :'rbenv:load'
 
  # 如果使用rvm，可以这样加载一个RVM version@gemset
  # invoke :'rvm:use[ruby-1.9.3-p374@default]'
end

# mina setup 时会执行的操作
task :setup => :environment do
  queue! %[mkdir -p "#{deploy_to}/shared/log"]              # 创建日志目录
  queue! %[chmod g+rx,u+rwx "#{deploy_to}/shared/log"]      # 设置日志目录权限
 
  queue! %[mkdir -p "#{deploy_to}/shared/config"]           # 创建配置目录
  queue! %[chmod g+rx,u+rwx "#{deploy_to}/shared/config"]   # 设置配置目录权限

  # 生成服务器的database.yml
  queue! %[cd #{deploy_to}/shared && git archive --remote=#{repository} #{branch} config | tar -x config/database.yml]
end
 
# 进行mina deploy会进行的操作
desc "Deploys the current version to the server."
task :deploy => :environment do
  deploy do
    # Put things that will set up an empty directory into a fully set-up
    # instance of your project.
    invoke :'git:clone'
    invoke :'deploy:link_shared_paths'
    invoke :'bundle:install'
    invoke :'rails:db_migrate'
    invoke :'rails:assets_precompile'

    to :launch do
      queue 'touch tmp/restart.txt'
    end
  end
end
{% endhighlight %}

### 5.服务器目录初始化
{% highlight bash %}
bundle exec mina setup
{% endhighlight %}

也可以这样操作，使提示更加详细些
{% highlight bash %}
bundle exec mina setup --verbose
{% endhighlight %}

### 6.进行项目部署
    bundle exec mina deploy
查看其他命令
    mina tasks

#### 参考：
* http://nadarei.co/mina/setting_up_a_project.html
* https://github.com/alfuken/mina-rails-unicorn-nginx-god
* https://gist.github.com/stas/4539489
* http://mjason.github.com/blog/2013/01/08/jian-dan-de-railsbu-shu/
* http://luansantos.com/2012/09/05/deploying-rails-applications-with-mina/
* http://rightsrestricted.com/entries/rails-deployment-with-mina-utilizing-git-slash-ssh
