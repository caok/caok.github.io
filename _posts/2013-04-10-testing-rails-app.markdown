---
layout: post
title: "testing rails app #1"
date: 2013-04-10 21:27
categories: [Rails, Test]
tags: [Rails, Rspec, Test]
---

一直都在回避测试，直接写功能，总觉着写测试代码挺别扭的不好写，但逃避不是办法。在社区中混的时间越长，越能发现越来越多的招聘贴都有写测试的要求，可见测试的重要性。在实际的开发过程中，功能一多，通过手动测试是很难每次考虑全面，测试也是比较重复的工作，不可能每次增加功能就手动将所有的测试都重新测一遍，以保证项目ok(对以前的功能也没影响)。那如何去避免这样的情况呢，那就只能通过测试去增加自己对自己代码的信心。

这里先简单介绍下以下两个工具的简单使用，在以后的博客中做继续的补充。

* rspec: BDD 测试框架, 替代 Rails 默认的 TestUnit
* factory_girl: 用于方便的创建测试数据, 替代 Rails 默认的 Test Fixture

### 1.创建rails项目
    rails new example -T
    或者
    rails new example --skip-test-unit --skip-bundle
忽略原先的 Rails 默认的 TestUnit

### 2.配置
{% highlight ruby %}
gem_group :development, :test do
  gem 'rspec-rails'
  gem 'factory_girl_rails'
end
{% endhighlight %}

    bundle install

{% highlight ruby %}
# config/application.rb
  class Application < Rails::Application
    config.generators do |g| 
      g.fixture_replacement :factory_girl
      g.test_framework :rspec, :fixture => true
    end 
    .....
  end
{% endhighlight %}

初始化rspec
    rails generate rspec:install
这个命令会创建两个文件: 

* .spec 保存运行 rspec 时的需要使用的选项; 
* spec/spec_helper.rb 是 rspec 的加载和配置文件。

在rspec中加入factory-girl
{% highlight ruby %}
# spec/spec_helper.rb
RSpec.configure do |config|
  .....
  # Mix in FactoryGirl methods
  config.include FactoryGirl::Syntax::Methods
  .....
end

{% endhighlight %}
这样在spec测试代码中，我们就可以直接调用FactoryGirl的方法

### 3.简单的测试案例
{% highlight ruby %}
rails scaffold book name:string publishing_house:string pages:integer ISBN:string content:text
{% endhighlight %}

如果照上面配置好之后，你就能发现执行这个命令后，会默认生成一些测试的模板(也可以在application.rb中指定生成那些测试模板)，我们可以在那个上面再做修改。

然后执行数据库迁移命令和测试准备命令:

{% highlight ruby %}
rake db:migrate
rake db:test:prepare # 在数据库结构发生变化时, 需要执行这句来保证用 rspec 执行测试时不会出错
{% endhighlight %}

修改测试用的数据
{% highlight ruby %}
# spec/factories/books.rb
FactoryGirl.define do          
  factory :book do         
    name "Aglic Web Development with Rails"
    publishing_house "DHH" 
    pages 500                  
    ISBN "978-7-121-11096-2"
    content "MyText"           
  end                      
end
{% endhighlight %}

测试需求：name不能为空，不能重复
{% highlight ruby %}
# spec/models/book_spec.rb
require 'spec_helper'
    
describe Book do
  let(:book) { build :book }
    
  it "passes validation with all valid informations" do
    expect(book).to be_valid
  end
    
  context "fails validation" do 
    it "with a blank name" do 
      book.name = ''
      expect(book.save).to be_false
    end

    it "with a duplicated name" do
      create :book, :name => book.name
      expect(book.save).to be_false
    end 
  end
end 
{% endhighlight %}

此时跑下测试rspec spec/models/book_spec.rb, 此时你会发现测试没通过(能通过才有鬼呢，哈哈)。

这时就可以修改你的model了，来达到你原先想要达到的目的
{% highlight ruby %}
# app/models/book.rb
class Book < ActiveRecord::Base
  # attributes
  attr_accessible :ISBN, :content, :name, :pages, :publishing_house, :cover

  # validation
  validates :name, :uniqueness => true
  validates :name, :presence => true
end
{% endhighlight %}

此时再跑测试时，你会得到这样的效果：
{% highlight ruby %}
Rack::File headers parameter replaces cache_control after Rack 1.5.
...

Finished in 0.8842 seconds
3 examples, 0 failures
{% endhighlight %}

总算是通过了，接下来可以自己试试加点其他的。

### 4.执行测试的方式
在 Rails 应用执行 rspec 测试有两种方式:
使用 rake 命令. 示例:

{% highlight ruby %}
rake test             # 执行所有测试, 也可以用 rake spec, 或直接用 rake
rake spec:models      # 执行 spec/models 下面的测试
rake spec:helpers     # 执行 spec/helpers 下面的测试
rake spec:requests    # 执行 spec/requests 下面的测试 (验收测试)
{% endhighlight %}

第一次执行这个命令时, 有可能会提示要先执行 rake db:migrate.
使用 rspec 命令. 示例:
{% highlight ruby %}
rspec                     # 执行 /spec 下所有的 "*_spec.rb" 文件, 即执行所有测试
rspec spec/models         # 执行 spec/models 下面的测试
rspec spec/helpers        # 执行 spec/helpers 下面的测试

rspec spec/models/post_spec.rb    # 只测试一个文件
rspec spec/models/post_spec.rb:7  # 只执行这个文件第7行所在的单个测试 (每个 "it" 块是一个测试)
{% endhighlight %}


#### 参考：
* http://xhh.me/blog/2012/10/testing-rails-app-with-rspec-capybara-and-zeus/
* http://ruby-china.org/topics/7771
* http://ruby-china.org/topics/7770
