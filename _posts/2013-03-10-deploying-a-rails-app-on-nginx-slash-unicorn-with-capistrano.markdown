---
layout: post
title: "Deploying a Rails app on Nginx/Unicorn with Capistrano"
date: 2013-03-10 17:00
categories: [Rails]
tags: [Rails, Nginx, Capistrano, Unicorn]
---

之前分别介绍了下capistrano和nginx的使用，这里汇总一下，大致介绍下用上述工具部署rails项目的设置。

#### 1.加载capistrano和unicorn
{% highlight ruby %}
# Gemfile
gem "capistrano"
gem "unicorn"
{% endhighlight %}

    bundle install

#### 2.安装nginx
{% highlight bash %}
sudo apt-get install nginx
{% endhighlight %}

#### 3.设置unicorn
这里我们需要unicorn的一个配置文件: config/unicorn.rb
{% highlight ruby %}
#config/unicorn.rb

# encoding: utf-8
# Set your full path to application.
application = "example_app"
app_path = "/u/apps/#{application}"
shared_path = "#{app_path}/shared"
current_path = "#{app_path}/current"
 
# Set unicorn options
worker_processes 4
preload_app true   # Preload our app for more speed
timeout 180
# 可同时监听 Unix 本地 socket 或 TCP 端口
# listen 9000, :tcp_nopush => true
listen "/tmp/unicorn.example_app.sock", :backlog => 64
 
# Spawn unicorn master worker for user apps (group: apps)
user ENV['USER'] || 'ruby', ENV['USER'] || 'ruby'
 
# Fill path to your app
working_directory current_path
 
# Should be 'production' by default, otherwise use other env 
rails_env = ENV['RAILS_ENV'] || 'production'
 
# Log everything to one file
stderr_path "log/unicorn.log"
stdout_path "log/unicorn.log"
 
# Set master PID location
pid "#{current_path}/tmp/pids/unicorn.pid"
 
if GC.respond_to?(:copy_on_write_friendly=)
    GC.copy_on_write_friendly = true
end

before_fork do |server, worker|
  ActiveRecord::Base.connection.disconnect!
 
  old_pid = "#{server.config[:pid]}.oldbin"
  if File.exists?(old_pid) && server.pid != old_pid
    begin
      Process.kill("QUIT", File.read(old_pid).to_i)
    rescue Errno::ENOENT, Errno::ESRCH
      # someone else did our job for us
    end
  end
end
 
after_fork do |server, worker|
  ActiveRecord::Base.establish_connection
end
 
# 修正无缝重启unicorn后更新的Gem未生效的问题，原因是config/boot.rb会优先从ENV中获取BUNDLE_GEMFILE，
# 而无缝重启时ENV['BUNDLE_GEMFILE']的值并>未被清除，仍指向旧目录的Gemfile
before_exec do |server|
  ENV["BUNDLE_GEMFILE"] = "#{current_path}/Gemfile"
end
{% endhighlight %}

如果想要让该项目的unicorn开机启动，可以参考[unicorn_init.sh](https://gist.github.com/happypeter/3977557).

#### 4.设置capistrano
初始化
    capify .

配置
{% highlight ruby %}
#config/deploy.rb

# encoding: utf-8
# add config/deploy to load_path
$: << File.expand_path('../deploy/', __FILE__)
 
require 'bundler/capistrano'
require 'capistrano_database'
 
set :application, "fruitwood"
#set :repository, "git://github.com/caok/example.git"
#set :branch, "master"
set :repository, ENV['REPO'] || File.expand_path('../../.git/', __FILE__)
 
set :scm, :git
 
set :deploy_to, "/u/apps/#{application}" # default
set :deploy_via, :remote_cache # 不要每次都获取全新的repository
 
set :user, ENV['DEPLOY_USER'] || ENV['USER'] || "ruby"
set :use_sudo, true
default_run_options[:pty] = true
 
set :rbenv_version, ENV['RBENV_VERSION'] || "1.9.3-p392"
set :default_environment, {
  'PATH' => "/home/#{user}/.rbenv/shims:/home/#{user}/.rbenv/bin:$PATH",
  'RBENV_VERSION' => "#{rbenv_version}",
}

set :deploy_server, ENV['DEPLOY_SERVER'] || 'localhost'

role :web, "#{deploy_server}"                          # Your HTTP server, Apache/etc
role :app, "#{deploy_server}"                          # This may be the same as your `Web` server
role :db,  "#{deploy_server}", :primary => true        # This is where Rails migrations will run
#role :db,  "your slave db-server here"

before "db:setup", "deploy:chown"

namespace :deploy do
  desc "Start Application"
  task :start, :roles => :app do
    run "cd #{current_path}; RAILS_ENV=production bundle exec unicorn_rails -c config/unicorn.rb -D"
  end

  desc "Stop Application"
  task :stop, :roles => :app do
    run "kill -QUIT `cat #{current_path}/tmp/pids/unicorn.pid`"
  end

  desc "Restart Application"
  task :restart, :roles => :app do
    run "kill -USR2 `cat #{current_path}/tmp/pids/unicorn.pid`"
  end

  namespace :assets do
    desc "deploy the precompiled assets"
    task :precompile, :roles => :web, :except => { :no_release => true } do
      run_locally("bundle exec rake assets:clean assets:precompile RAILS_ENV=#{rails_env} #{asset_env}")
      top.upload("public/assets", "#{release_path}/public/", :via => :scp, :recursive => true)
      run_locally("rm -rf public/assets")
    end
  end

  task :chown, :roles => :app do
    run "#{try_sudo} chown -R #{user}:#{user} #{deploy_to}"
  end

  desc "create db:seed"
  task :seed, :roles => :app do
    run "cd #{current_path} && rake db:seed RAILS_ENV=#{rails_env}"
  end
end
{% endhighlight %}

这里的“$: << File.expand_path('../deploy/', __FILE__)”设置是为了让数据更加安全些，具体做法参考[capistrano and unicorn](http://caok1231.com/blog/2013/02/28/capistrano-and-unicorn/)或者[capistrano and database.yml](http://www.simonecarletti.com/blog/2009/06/capistrano-and-database-yml/)

#### 5.部署代码
预先生成好数据库

{% highlight bash %}
mysql -h HOSTNAME -u USERNAME -p
create database DATABASE_NAME default character set utf8;
{% endhighlight %}

{% highlight bash %}
cap deploy:setup
cap deploy:cold
{% endhighlight %}

此时你可以打开loccalhost:9000查看你的应用

#### 6.设置nginx
{% highlight bash %}
#/etc/nginx/sites-available/rails_nginx.conf

upstream app_server {
  # for UNIX domain socket setups:
  server unix:/tmp/unicorn.example_app.sock fail_timeout=0;

  # for TCP setups, point these to your backend servers
  # server 192.168.0.7:8080 fail_timeout=0;
  # server 192.168.0.8:8080 fail_timeout=0;
}

server {
  listen 80 default;
  server_name example.com;
  client_max_body_size 4G;
  keepalive_timeout 5;

  root /u/app/example_app/current/public;

  try_files $uri/index.html $uri.html $uri @unicorn;
  location @unicorn {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;
    proxy_pass http://app_server;
  }

  # Rails error pages
  error_page 500 502 503 504 /500.html;
  location = /500.html {
    root /u/app/example_app/current/public;
  }
}
{% endhighlight %}

留意这里的app_server在upstream和location中都要替换掉才行
{% highlight bash %}
cd /etc/nginx/sites-enabled
ls -l
sudo rm default
sudo ln -s /etc/nginx/sites-available/rails_nginx.conf rails
{% endhighlight %}

重启nginx，使设置生效
    sudo service nginx restart

此时你打开localhost就可查看你的应用

##### 参考：
* http://ruby-china.org/topics/4709
* http://ruby-china.org/topics/471
* http://railscasts.com/episodes/293-nginx-unicorn
* http://happycasts.net/episodes/42
* https://github.com/defunkt/unicorn/tree/master/examples
