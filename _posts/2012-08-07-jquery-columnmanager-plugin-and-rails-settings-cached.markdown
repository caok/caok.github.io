---
layout: post
title: "jQuery columnManager plugin and rails-settings-cached"
date: 2012-08-07 18:47
comments: true
categories: [Rails, Javascript, Web]
---

在平时页面显示表格时，容易出现要显示的字段过多，而在一页的范围内无法完全显示的困境。通过juery columnanager可以实现显示和隐藏的效果，再通过rails-settings-cached将其与个人偏好设置相关联，使得更加人性化。

### 安装rails-settings-cached
Edit your Gemfile:

    gem "rails-settings-cached"

Generate your settings:

    $ rails g settings <settings_name>

Now just put that migration in the database with:

    rake db:migrate

Settings may be bound to any existing ActiveRecord object. Define this association like this: Notice! is not do caching in this version.

    class User < ActiveRecord::Base
      include RailsSettings::Extend
    end

### 安装jQuery columnManager plugin
下载地址：<a href="http://p.sohei.org/download-manager.php?id=54">jquery.columnmanager.zip</a>
我这使用的是里面的jquery.columnmanager.pack,添加至项目的application.js中

    //= require jquery.columnmanager.pack

### 两者的配合使用
页面上调用
{% highlight ruby %}
<div id="targetcol">
</div>

<table id="tableall">
  .....
</table>

<script type="text/javascript">
  $('#tableall').columnManager({
    listTargetID:'targetcol',
    onClass: 'simpleon',
    offClass: 'simpleoff',
    colsHidden: <%= current_user.hide_columns(request.url) %>,
    onToggle: function(index, state){
      $.ajax({
        url: 'users/hide',
        type: 'post',
        data: "index=" + index + ";state=" + state + ";url=" + "<%= request.url %>"
        })
    }    
  });    
</script>
{% endhighlight %}

这里将点击后的效果通过ajax保存到用户的settings中，其中保存了url、index、state这几个属性值, 并在下次打开页面时通过hide_cloumns取出原先保存的值,起到一个人性化的效果。

编辑routes
{% highlight ruby %}
  resources :users do
    post :hide, :on => :collection
  end
{% endhighlight %}

在users_controller中增加hide处理，提供给ajax调用
{% highlight ruby %}
# users_controller.rb
def hide
  p = current_user.settings.pagesetups
  p = {} unless p
  if p[params[:url]]
    p[params[:url]][params[:index].to_i] = params[:state]
  else
    p[params[:url]] = {params[:index].to_i => params[:state]}
  end
  current_user.settings.pagesetups = p
  render :nothing => true
end
{% endhighlight %}

取出用户对于table中字段隐藏显示的偏好设置
{% highlight ruby %}
# user.rb
def hide_columns(url)
  if settings.pagesetups and settings.pagesetups[url]
    p = self.settings.pagesetups[url] 
  else
    return "[]"              
  end
  lists = p.delete_if {|k,v| v != "false" }.keys
  "[" + lists * "," + "]"    
end
{% endhighlight %}

#### 参考
<a href="http://p.sohei.org/stuff/jquery/columnmanager/demo/demo.html">jQuery columnManager plugin</a>
<a href="https://github.com/huacnlee/rails-settings-cached">rails-settings-cached</a>
