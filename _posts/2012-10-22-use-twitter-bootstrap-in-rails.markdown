---
layout: post
title: "use twitter bootstrap in rails"
date: 2012-10-22 17:50
comments: true
categories: [Rails, Bootstrap, Web]
---

### 1.Installing Gem
Include the Twitter Bootstrap Rails gem in Gemfile to install it from RubyGems.org

<!-- more -->

```
group :assets do
  gem 'twitter-bootstrap-rails'
end
```

You can run bundle from command line

    bundle install

install bootstrap

    rails g bootstrap:install

忽略默认的样式生成scaffold

    rails g scaffold product name price:decimal --skip-stylesheets

### 2.generates Twitter Bootstrap compatible layout
you can use

    rails g bootstrap:layout [LAYOUT_NAME] [*ﬁxed or ﬂuid]

or define it by yourself

```
#layouts/application.html.erb
<!DOCTYPE html>
<html>
  <head>
    <title><%= Company.last.try :name %></title>
    <!--[if lt IE 9]>
      <script src="http://html5shim.googlecode.com/svn/trunk/html5.js" type="text/javascript"></script>
    <![endif]-->
    <%= stylesheet_link_tag    "application", :media => "all" %>
    <%= javascript_include_tag "application" %>
    <%= csrf_meta_tags %>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
  </head>
  <body>
    <div class="navbar navbar-fixed-top">
      <div class="navbar-inner">
        <div class="container">
          <a class="btn btn-navbar" data-toggle="collapse" data-target=".nav-collapse">
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
          </a>
          <%= link_to "Home", root_path, :class => 'brand' %>
          <div class="nav-collapse">
            <ul class="nav">
              <li class='<%= 'active' if params[:controller] == 'home' %>'><%= link_to "首页", root_path %></li>
              <li class='<%= 'active' if params[:controller] == 'products' %>'><%= link_to "产品介绍", products_path %></li>
            </ul>
            <ul class='nav pull-right'>
              <% if user_signed_in? -%>
                <li class='dropdown'>
                  <a class="dropdown-toggle" data-toggle="dropdown">
                    <%= current_user.name %>
                    <b class='caret'></b>
                  </a>
                  <ul class='dropdown-menu'>
                    <li><%= link_to "个人资料", :controller => "my", :action => "information" %></li>
                    <li><%= link_to "编辑个人资料", :controller => "my", :action => "edit" %></li>
                    <li><%= link_to '修改密码', :controller => "my", :action => "password" %></li>
                    <li><%= link_to "退出", destroy_user_session_path, :method => :delete %></li>
                  </ul>
                </li>
              <% else -%>
                <li><%= link_to "登录", new_user_session_path %></li>
              <% end -%>
            </ul>
          </div>
        </div>
      </div>
    </div>
    <div class="container">
      <%= render 'layouts/messages' %>
      <%= yield %>
      <hr>
      <footer>
      <p>&copy;2012 <a href="http://xxxxx.com" target="_blank" >xxxxxxxxx</a></p>
      </footer>
    </div>
  </body>
</html>
```

```
app/views/layouts/_messages.html.erb
<% flash.each do |name, msg| %>
  <% if msg.is_a?(String) %>
    <div class="alert alert-<%= name == :notice ? "success" : "error" %>">
      <a class="close" data-dismiss="alert">&#215;</a>
      <%= content_tag :div, msg, :id => "flash_#{name}" %>
    </div>
  <% end %>
<% end %>
```

### 3.Themed (generates Twitter Bootstrap compatible scaffold views.)
    rails g bootstrap:themed products -f

### 4.Add validation's alert
    gem 'simple_form'
    bundle install
    rails g simple_form:install --bootstrap

{% codeblock lang:ruby %}
<div class="control-group">
  <%= f.label :name, :class => 'control-label' %>
  <div class="controls">
    <%= f.text_field :name, :class => 'text_field' %>
  </div>
</div>
{% endcodeblock %}

修改为

    <%= f.input :name %>

### 5.Use Twitter Bootstrap kaminari
    https://github.com/dleavitt/bootstrap_kaminari
    https://github.com/seyhunak/twitter-bootstrap-rails

### 6.Using Less with Bootstrap
https://github.com/twitter/bootstrap/blob/master/less/variables.less

for example
add in the bootstrap_and_overrides.css.less

    @navbarBackground:                #555;
    @navbarBackgroundHighlight:       #888;
    @navbarText:                      #eee;
    @navbarLinkColor:                 #eee;

#### references:
* https://github.com/twitter/bootstrap
* https://github.com/seyhunak/twitter-bootstrap-rails
