---
layout: post
title: "Rails Application Templates"
date: 2013-02-25 22:26
categories: [Rails]
tags: [Rails]
---

在初始化每个项目的时候都会做一些重复的工作，可以使用Rails的template避免重复劳动。

应用程序模板(template)是简单的ruby文件，该文件包含的DSL用来对刚创建的或已经存在的rails项目添加gems或进行初始化等。
<!-- more -->

### 1.用法
直接用template创建项目
{% highlight bash %}
rails new blog -m ~/template.rb
rails new blog -m http://example.com/template.rb
{% endhighlight %}

或者在已经创建的项目中
{% highlight bash %}
rake rails:template LOCATION=~/template.rb
rake rails:template LOCATION=http://example.com/template.rb
{% endhighlight %}

### 2.Template API
##### 2.1 gem
增加一个gem到生成的应用程序的Gemfile中。
{% highlight ruby %}
gem 'slim-rails'
gem 'devise'
{% endhighlight %}

##### 2.2 gem_group
{% highlight ruby %}
gem_group :development, :test do
  gem 'factory_girl_rails'
end

gem "rspec-rails", group: [:test, :development]

gem_group :assets do
  gem 'turbo-sprockets-rails3'
end
{% endhighlight %}

##### 2.3 run(command)
执行任意的命令
{% highlight ruby %}
run 'bundle install'
run 'rm -rf public/index.html'
{% endhighlight %}

##### 2.4 rake
在Rails应用程序运行所提供的rake任务
{% highlight ruby %}
rake "db:migrate"
rake "db:migrate", env: 'production'
{% endhighlight %}

##### 2.5 generate(what, *args)
{% highlight ruby %}
generate(:controller, "home index")
generate(:scaffold, "person", "name:string", "address:text", "age:number")
{% endhighlight %}

##### 2.6 route(routing_code)
在config/routes.rb中增加路由条目
{% highlight ruby %}
route "root to: 'person#index'"
{% endhighlight %}

##### 2.7 environment/application
添加新的代码到config/application.rb
{% highlight ruby %}
environment do
<<-CODE
config.time_zone = 'Beijing'
    config.i18n.default_locale = 'zh-CN'
    config.generators do |g|
      g.fixture_replacement :factory_girl
      g.test_framework :rspec, :fixture => true
      g.stylesheets false
      g.javascripts false
      g.helper false
      g.helper_specs false
      g.view_specs false
    end
CODE
end
{% endhighlight %}

##### 2.8 append_file
增加新的代码到指定的文件中
{% highlight ruby %}
append_file 'app/assets/javascripts/application.js', <<-CODE, verbose: false
//= require rails.validations
//= require rails.validations.simple_form
CODE
{% endhighlight %}

##### 2.9 git(:command)
Rails的模板可以让你运行任何Git命令
{% highlight ruby %}
git :init
git add: "."
append_file ".gitignore", "config/database.yml"    #将database.yml加入到要忽略的名单中,也可用gsub_file
run "cp config/database.yml config/database.yml.example"
git commit: "-am 'Initial commit'"
{% endhighlight %}

##### 2.10 加载预先准备好的文件
{% highlight ruby %}
def template_file path
  template_path = File.expand_path("../templates/#{path}", __FILE__)
  if template_path =~ /https:/
    template_path = template_path.scan(/https:.*/).first if template_path =~ /https:/
    template_path.sub!(/https:/, 'https:/') # workaround to add missing slash for https:// 
  end
  file path, open(template_path).read
end
{% endhighlight %}

加载以下两个文件到相同的路径中
{% highlight ruby %}
template_file 'app/views/common/_menu.html.slim'
template_file 'app/views/layouts/application.html.slim'
{% endhighlight %}

##### 2.11 文件处理
{% highlight ruby %}
remove_file "README.rdoc"           # 删除文件
create_file "README.md", "TODO"     # 新建文件
{% endhighlight %}

##### 2.12 交互式生成root
{% highlight ruby %}
if yes? "Do you want to generate a root controller?"
  name = ask("what should it be called?").underscore
  generate :controller, "#{name} index"
  route "root to: '#{name}#index'"
  remove_file "public/index.html"      # rails4中不再有public/index.html文件
end
{% endhighlight %}

#### 参考：
* http://edgeguides.rubyonrails.org/rails_application_templates.html
* https://github.com/caok/bootstrap-template/blob/master/template.rb
* http://bbs.chinaunix.net/thread-1861565-1-1.html
* http://railscasts.com/episodes/148-custom-app-generators-revised
* http://railscasts.com/episodes/148-app-templates-in-rails-2-3
