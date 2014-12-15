---
layout: post
title: "Rails Application Templates"
date: 2013-02-25 22:26
comments: true
categories: [Rails]
---

在初始化每个项目的时候都会做一些重复的工作，可以使用Rails的template避免重复劳动。

应用程序模板(template)是简单的ruby文件，该文件包含的DSL用来对刚创建的或已经存在的rails项目添加gems或进行初始化等。
<!-- more -->

### 1.用法
直接用template创建项目
```sh
rails new blog -m ~/template.rb
rails new blog -m http://example.com/template.rb
```
或者在已经创建的项目中
```sh
rake rails:template LOCATION=~/template.rb
rake rails:template LOCATION=http://example.com/template.rb
```

### 2.Template API
##### 2.1 gem
增加一个gem到生成的应用程序的Gemfile中。
```ruby
gem 'slim-rails'
gem 'devise'
```
##### 2.2 gem_group
```ruby
gem_group :development, :test do
  gem 'factory_girl_rails'
end

gem "rspec-rails", group: [:test, :development]

gem_group :assets do
  gem 'turbo-sprockets-rails3'
end
```

##### 2.3 run(command)
执行任意的命令
```ruby
run 'bundle install'
run 'rm -rf public/index.html'
```

##### 2.4 rake
在Rails应用程序运行所提供的rake任务
```
rake "db:migrate"
rake "db:migrate", env: 'production'
```

##### 2.5 generate(what, *args)
```ruby
generate(:controller, "home index")
generate(:scaffold, "person", "name:string", "address:text", "age:number")
```

##### 2.6 route(routing_code)
在config/routes.rb中增加路由条目
```ruby
route "root to: 'person#index'"
```

##### 2.7 environment/application
添加新的代码到config/application.rb
```ruby
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
```

##### 2.8 append_file
增加新的代码到指定的文件中
```ruby
append_file 'app/assets/javascripts/application.js', <<-CODE, verbose: false
//= require rails.validations
//= require rails.validations.simple_form
CODE
```

##### 2.9 git(:command)
Rails的模板可以让你运行任何Git命令
```ruby
git :init
git add: "."
append_file ".gitignore", "config/database.yml"    #将database.yml加入到要忽略的名单中,也可用gsub_file
run "cp config/database.yml config/database.yml.example"
git commit: "-am 'Initial commit'"
```

##### 2.10 加载预先准备好的文件
```ruby
def template_file path
  template_path = File.expand_path("../templates/#{path}", __FILE__)
  if template_path =~ /https:/
    template_path = template_path.scan(/https:.*/).first if template_path =~ /https:/
    template_path.sub!(/https:/, 'https:/') # workaround to add missing slash for https:// 
  end
  file path, open(template_path).read
end
```
加载以下两个文件到相同的路径中
```
template_file 'app/views/common/_menu.html.slim'
template_file 'app/views/layouts/application.html.slim'
```

##### 2.11 文件处理
```ruby
remove_file "README.rdoc"           # 删除文件
create_file "README.md", "TODO"     # 新建文件
```

##### 2.12 交互式生成root
```ruby
if yes? "Do you want to generate a root controller?"
  name = ask("what should it be called?").underscore
  generate :controller, "#{name} index"
  route "root to: '#{name}#index'"
  remove_file "public/index.html"      # rails4中不再有public/index.html文件
end
```

#### 参考：
* http://edgeguides.rubyonrails.org/rails_application_templates.html
* https://github.com/caok/bootstrap-template/blob/master/template.rb
* http://bbs.chinaunix.net/thread-1861565-1-1.html
* http://railscasts.com/episodes/148-custom-app-generators-revised
* http://railscasts.com/episodes/148-app-templates-in-rails-2-3
