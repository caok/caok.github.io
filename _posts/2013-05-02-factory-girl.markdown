---
layout: post
title: "factory girl"
date: 2013-05-02 16:33
comments: true
categories: [Rails, Test]
---

很多时候我们都会使用 factory-girl 去构建测试数据，但如何去定义表之间的关系，特别是那种用户角色、权限之类的，接下来就稍微总结下如何用[factory girl](https://github.com/thoughtbot/factory_girl) 去定义测试数据中的用户角色。

### 1.Many-to-Many
假定用户角色是通过many-to-many的关系定义的，比如结构是如下定义的：

{% highlight ruby %}
class User < ActiveRecord::Base
  has_many :user_roles
  has_many :roles, :through => :user_roles
end

class UserRole < ActiveRecord::Base
  belongs_to :user             
  belongs_to :role
end

class Role < ActiveRecord::Base
  attr_accessible :name
 
  has_many :user_roles
  has_many :users, :through => :user_roles
end
{% endhighlight %}

#### 1.1通过seeds.rb
先介绍下一种简单的处理，通过加载seeds.rb，预先生成“role”
{% highlight ruby %}
# seeds.rb
Role.delete_all
Role.create!([{ name: 'admin' }, { name: 'member' }])
{% endhighlight %}

在spec_helper.rb中将其加载
{% highlight ruby %}
# spec_helper.rb
RSpec.configure do |config|
  .....
  config.before(:all) do
    DatabaseCleaner.start
    # execute seed.rb
    load File.expand_path('db/seeds.rb', Rails.root)
  end
  .....
end
{% endhighlight %}

介绍两种调用方式：

第一种：
{% highlight ruby %}
factory :user do
  name "xxx"
  trait :admin do
    after(:build) {|u| u.roles << Role.find_by_name('admin') }
  end
end

//测试中创建admin角色的user
create :user, :admin
{% endhighlight %}

另外一种:
{% highlight ruby %}
factory :user do
  name "xxx"
  trait :admin do
    after(:build) {|u| u.roles << Role.find_by_name('admin') }
  end
  factory :admin, traits: [:admin]
end

//测试中创建admin角色的user
create :admin
{% endhighlight %}

Traits 允许你组合属性，然后将它们应用到任何 factory 中。

#### 1.2直接构建 many-to-many
上面的方法有些讨巧，实际上这个就是要解决如何通过 factory-girl 去构建 many-to-many 的关系
{% highlight ruby %}
factory :user do
  name "xxx"

  factory :admin_user do
    roles {[ create(:admin_role) ]}
  end

  factory :member_user do
    roles {[ create(:member_role) ]}
  end
end  
          
factory :role do
  factory :admin_role do
    name "admin"
  end 
        
  factory :member_role do
    name "member"
  end 
end

//测试中创建admin角色的user
create :admin_user
{% endhighlight %}

### 2.has-many
{% highlight ruby %}
factory :article do
  body 'article body'

  factory :article_with_comment do
    after(:create) do |article|
      create(:comment, article: article)
    end
  end
end

factory :comment do
  body 'Great article!'
end

// 调用
article = create(:article_with_comment)
{% endhighlight %}

更多的比如多态之类的关系可以参考[Factory Girl callbacks](http://robots.thoughtbot.com/post/254496652/aint-no-calla-back-girl)

### 3.补充
定义角色还有种更常见的方式，就是通过 [Role Based Authorization](https://github.com/ryanb/cancan/wiki/Role-Based-Authorization) 这样去定义用户角色的，可以这么处理
{% highlight ruby %}
FactoryGirl.define do
  factory :user do
    name "xxx"
    .....
 
    # create role user for all roles
    #   
    # trait :admin do
    #   roles ["admin"]
    # end
    #   
    # factory :admin_user, :traits => [:admin]
    User::ROLES.each do |r| 
      trait r do
        roles r
      end 
 
      factory "#{r}_user", :traits => [r] 
    end 
  end 
end
{% endhighlight %}

调用时直接这样来：
{% highlight ruby %}
create :admin_user
{% endhighlight %}


#### 参考：
* https://github.com/thoughtbot/factory_girl/blob/master/GETTING_STARTED.md
* http://robots.thoughtbot.com/post/254496652/aint-no-calla-back-girl
* http://stackoverflow.com/questions/14162344/rails-3-factory-girl-many-to-many-relationships
* http://stackoverflow.com/questions/1484374/how-to-create-has-and-belongs-to-many-associations-in-factory-girl
