---
layout: post
title: "semi static pages"
date: 2013-11-24 16:23
categories: [Rails]
tags: [Rails]
---

在日常的使用中静态页面也是比较常见，railscast的[117](http://railscasts.com/episodes/117-semi-static-pages-revised)中介绍了两种处理方法，这里把它总结成文字，方便以后的查询。

### 1.直接通过命令生成三个静态页面
{% highlight ruby %}
rails g controller info about privacy license
{% endhighlight %}

{% highlight ruby %}
get "info/about"

<%= link_to_unless_current "About Us", info_about_path %>
{% endhighlight %}
此时我们的访问路径是localhost:3000/info/about

这个url看着挺别扭的，我们可以改造下routes来将about页面的访问路径修改为localhost:3000/about
{% highlight ruby %}
get "about", to: "info#about"

<%= link_to_unless_current "About Us", about_path %>
{% endhighlight %}
把其他两个privacy和license的路径也同样这么修改，这时我们的代码就可以写成这样
{% highlight ruby %}
%w[about privacy license].each do |page|
  get page, controller: "info", action: page
end 
{% endhighlight %}

### 2.将静态页面内容存入数据库
将静态页面的内容存入到数据库，这样更加的灵活，能很方便的在后期添加其他的静态页面。处理起来也很简单，先生成一张表page用来存放数据
{% highlight ruby %}
rails g scaffold page name permalink:string:index content:text --skip-stylesheets
{% endhighlight %}

{% highlight ruby %}
# models/page.rb
  validates_uniqueness_of :permalink
  
  def to_param
    permalink
  end 
{% endhighlight %}

这里的to_param的用法的话，可以参考：http://apidock.com/rails/ActiveRecord/Base/to_param 和 http://guides.ruby-china.org/active_support_core_extensions.html#2-7

controller中将find(params[:id])修改为find_by_permalink!(params[:id]),这样在添加一个about之后，我们就可以通过page_path("about")来访问about页面
{% highlight ruby %}
<%= link_to_unless_current "About Us", page_path("about") %>
{% endhighlight %}

但这个时候的url是localhost:3000/pages/about,同样这里我们也改造一下，将url改为访问localhost:3000/about就能获得about页面
{% highlight ruby %}
  resources :pages, except: :show 
  get ':id', to: 'pages#show', as: :page
{% endhighlight %}
