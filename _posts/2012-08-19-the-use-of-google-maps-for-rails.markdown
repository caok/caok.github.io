---
layout: post
title: "the use of Google-Maps-for-Rails"
date: 2012-08-19 17:40
comments: true
categories: [Rails, GoogleMaps]
---

在rails中调用google maps，以此可以来实现一个LBS网站。
这里使用的是<a href="https://github.com/apneadiving/Google-Maps-for-Rails">Google-Maps-for-Rails</a>这个gem。RailsCast中也介绍了一个gem也有相关的功能，<a href="http://railscasts.com/episodes/273-geocoder">geocoder</a>.

这里我简单的介绍下Google-Maps-for-Rails的一些使用。

###事先的准备
1.Gemfile

    gem 'gmaps4rails'

2.将assets添加到自己的应用中

    rails generate gmaps4rails:install

3.css

    <%= stylesheet_link_tag 'gmaps4rails' %>

4.javascript

    <%= yield :scripts %>

查看下css和javascript添加的位置
{% highlight ruby %}
# application.html.erb
<!DOCTYPE html>
<html>
  <head>
    <title>xxxx</title>
    <%= stylesheet_link_tag    "application", :media => "all" %>
    <%= javascript_include_tag "application" %>
    <%= csrf_meta_tags %>
    <%= stylesheet_link_tag 'gmaps4rails' %>
  </head>
  <body>
    <%= yield %>
    <%= yield :scripts %>
  </body>
</html>
{% endhighlight %}

###基本的配置
1.生成你的表结构，里面包含country,city,street等若干字段

    add_column :xxx, :country, :string
    add_column :xxx, :city, :string
    add_column :xxx, :street, :string
    add_column :xxx, :latitude,  :float
    add_column :xxx, :longitude, :float
    add_column :xxx, :gmaps, :boolean

2.在你的model中，增加：

    acts_as_gmappable
    def gmaps4rails_address
      "#{self.street}, #{self.city}, #{self.country}" 
    end

###显示出google地图
1.在controller中

    @json = xxx.all.to_gmaps4rails

2.在view中

    <%= gmaps4rails(@json) %>

此时在页面上就能看到google地图了，还行吧,哈哈。
在输入coutry、city、street之后，这个gem能自动识别其经纬度，并且在地图上显示出来。

如果你使用了Twitter Bootstrap了，可能会对谷歌地图的实现有一定的影响，不过解决方法很简单。
在你的gmaps4rails.css中你需要添加img { max-width :none; }到.map_container或者.gmaps4rails_map中的任意一个。比如：

    .map_container img{max-width: none;}

