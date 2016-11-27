---
layout: post
title: "Capistrano and Unicorn"
date: 2013-02-28 20:22
categories: [Rails]
tags: [Rails, Capistrano, Unicorn]
---

* [capistrano](https://github.com/capistrano/capistrano) 是最常用来完成 rails 项目的远程多服务器自动化部署的工具。
* [unicorn](http://unicorn.bogomips.org/) 是一款很常用的Rack应用的HTTP服务器。

### 1.安装
{% highlight ruby %}
# Gemfile
group :development do
  gem 'capistrano-unicorn', :require => false
end

group :production do
  gem 'unicorn'
end
{% endhighlight %}

由于我用的是[rbenv](https://github.com/sstephenson/rbenv)而不是rvm，所以在选择gem时挑选了[capistrano-unicorn](https://github.com/sosedoff/capistrano-unicorn),它集成了unicon的任务，可以让我不用再去另外写相应的处理。
此外如果用rvm的话，推荐[rvm-capistrano](https://github.com/wayneeseguin/rvm-capistrano)。

然后自然是
    bundle install

### 2.初始化
在你的项目中运行：
    capify .
它将会创建两个文件：Capfile和config/deploy.rb

### 3.配置
{% highlight ruby %}
set :application, "set your application name here"
set :repository,  "set your repository location here"
{% endhighlight %}

如果想通过代码服务器上的代码来进行自己的代码部署，可这么设置
{% highlight ruby %}
set :repository, "git://github.com/xxxx/xxxxx.git"
set :branch, "master"     # 设置索取代码的哪个分支
{% endhighlight %}

如果是想通过本地的代码直接进行部署：
{% highlight ruby %}
set :repository, File.expand_path('../../.git/', __FILE__)
{% endhighlight %}

这样不设置branch的话，本地处在哪个分支上，就会直接部署这个分支上的代码。

deploy_to可以修改部署的路径, 默认部署在/u/apps/下面
{% highlight ruby %}
set :deploy_to, "/u/apps/#{application}" # default
set :deploy_via, :remote_cache # 不要每次都获取全新的repository
{% endhighlight %}

##### 配置部署的服务器信息
{% highlight ruby %}
set :deploy_server, ENV['DEPLOY_SERVER'] || 'localhost'

role :web, "#{deploy_server}"                    # Your HTTP server, Apache/etc
role :app, "#{deploy_server}"                    # This may be the same as your `Web` server
role :db,  "#{deploy_server}", :primary => true  # This is where Rails migrations will run
#role :db,  "your slave db-server here"
{% endhighlight %}

{% highlight ruby %}
set :user, ENV['USER'] || "ruby"
set :use_sudo, true
default_run_options[:pty] = true
{% endhighlight %}
为了让 #{try_sudo}生效，所以 set :use_sudo, true。这样是不科学的，因为下次我们去执行 cap deploy:setup 时所有的目录创建等操作将会默认由 root 用户去执行，这在后续的操作中可能会引起很多 Permission denied!。
解决方案: set :use_sudo, true 中 true 改为 false，以后用到 #{try_sudo} 的地方，用 #{sudo} 就可以了。

##### 设置rbenv
{% highlight ruby %}
set :rbenv_version, ENV['RBENV_VERSION'] || "1.9.3-p392"
set :default_environment, {
  'PATH' => "/home/#{user}/.rbenv/shims:/home/#{user}/.rbenv/bin:$PATH",
  'RBENV_VERSION' => "#{rbenv_version}",
}
{% endhighlight %}

RBENV_VERSION通过rbenv shell 1.9.3-p392设置,如果使用的rvm的话，可以省略这步设置。

##### 设置unicorn
{% highlight ruby %}
require 'capistrano-unicorn'
 
after 'deploy:start', 'unicorn:start'
after 'deploy:stop', 'unicorn:stop'
{% endhighlight %}

它会调用config/unicorn/production.rb，可以参考capistrano-unicorn中的[sxample](https://github.com/sosedoff/capistrano-unicorn/blob/master/examples/rails3.rb)

本身的database设置不是很安全，可以更加安全的管理database.yml,参考：http://www.simonecarletti.com/blog/2009/06/capistrano-and-database-yml/

### 4.部署
在/u/apps/生成目录结构(只在第一次部署时需要)
    cap deploy:setup
执行完之后就会在部署路径中创建一些新增目录
{% highlight ruby %}
myapp/
      releases/ shared/log shared/pids shared/system
{% endhighlight %}

更新代码并冷启动（适用于服务没有启动）
    cap deploy:cold

更新代码并热启动（使用与服务正在运行）
    cap deploy

更新代码，数据库并热启动（使用与服务正在运行）
    cap deploy:migrations

版本升级出错，返回上一版本
    cap deploy:rollback

更多命令查看
    cap -T

#### 参考：
* https://github.com/capistrano/capistrano/wiki
* http://happycasts.net/episodes/42
* http://www.oschina.net/translate/using-capistrano
* https://help.github.com/articles/deploying-with-capistrano
* https://github.com/sosedoff/capistrano-unicorn
* https://github.com/wayneeseguin/rvm-capistrano
* http://www.simonecarletti.com/blog/2009/06/capistrano-and-database-yml/
* https://github.com/saberma/19wu/blob/master/config/deploy.rb
* https://github.com/ruby-china/ruby-china/blob/master/config/deploy.rb
* https://github.com/wongyouth/wabao/blob/master/config/deploy.rb
* https://github.com/sosedoff/capistrano-unicorn/blob/master/examples/rails3.rb
* http://unicorn.bogomips.org
