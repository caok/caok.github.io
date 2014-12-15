---
layout: post
title: "Security in Rails"
date: 2013-03-30 22:35
comments: true
categories: [Rails]
---

平时开发时一般都很少考虑网站的安全性，但这一块却又是开发应用时必须要考虑到的。Rails默认已经帮我们做了很多这方面的处理，但我们也有必要去了解下。
<!-- more -->

### 1.XSS：跨站脚本(Cross-site scriptin)
恶意攻击者往Web页面里插入恶意html代码，当用户浏览该页之时，嵌入其中Web里面的html代码会被执行，从而达到恶意用户的特殊目的。

要防范这个问题的办法，就是要逸出使用者输入的内容，例如将
    <script>变成&lt;script&gt;

使它显示出来的时候不会让浏览器去执行。

Rails中预设将任何View中的字串，都做了HTML逸出。在老的Rails项目中经常会看到h(string)的写法(实际上就是html_escape())，它的作用就是对其进行专一替换。在Rails 3之后已经默认对其进行了处理，就不需要再添加h()。

如果你知道你写的代码是安全的, 不要逸出，你可以这样处理：
```
"<p>safe</p>".html_safe
或
raw("<p>safe</p>")
```
Rails中也提供了sanitize()来过滤逸出

### 2.CSRF：跨站请求伪造(Cross-site request forgery)
攻击者冒充他人利用他人的权限去执行网站上的操作。

要防范CSRF，首先要区分GET和POST的HTTP请求。所有的读取和查询操作都应该使用GET，而修改或删除的操作，则要使用POST、PUT或DELETE。

在表单中，Rails自动加入一个叫做authenticity_token的隐藏字段。这个字段的值是用户session的唯一键值key,是基于一个随机字符串被保存在session中。对于任何一个单独的 POST,PUT,DELETE请求，RAILS都将自动检查这个键值。这会保证每个请求都是由我们自己网站上的用户发出的，而不是由其他站点的伪装用户发出的。对于GET请求，这个token不会被检查，所以我们需要确认GET请求不会改变或者删除我们应用的数据。

此外Rails中通过protect_from_forgery(application_controller.rb)内建了一层CSRF防御，即所有的post请求都必须加上一个安全验证码。Layout中也有一段<%= csrf_meta_tags %>是给JavaScript读取验证码用的。

如果POST请求没有带争取的验证码，Rails就会丢出一个ActionController:InvalidAuthenticityToken的错误.

### 3.SQL注入
SQL注入，就是通过把SQL命令插入到Web表单递交或输入域名或页面请求的查询字符串，最终达到欺骗服务器执行恶意的SQL命令。

那Rails中一般在什么情况下会出现SQL注入的漏洞呢。如果你写出了类似这样的代码，你就需要特别当心了
```ruby
Project.where("name = '#{params[:name]}'")
```
那么攻击者只要输入：
```sql
x'; DROP TABLE users; --
```
这样执行的SQL就会变成(users表就在不知不觉中被人恶意删除了...)：
```sql
SELECT * FROM projects WHERE name = 'x'; DROP TABLE users; --’
```
那如何去避免呢？抵挡SQL注入的正确方法是：绝对不要用Ruby的#{...}机制直接把字符窜直接插入到SQL语句中，而应该用Rails提供的变量绑定工具。Rails ActiveRecord的where方法中使用Hash或Array就会自动帮你做逸出处理,比如刚才的那个查询可以这么写：
```ruby
Project.where("name = ?", params[:name])
或
Project.where(:name => params[:name])
```
将查询变成模型对象，这样也可以避免SQL注入。
```ruby
Project.find_by_name(params[:name])
```
当然有人有疑问，前阵子爆出的[Rails SQL注入漏洞](http://www.csdn.net/article/2013-01-06/2813449-rails-sql-injection-facts)，问题就是出在find_by_xxx上，那是有一定的前提条件的，具体请看[这里](http://www.csdn.net/article/2013-01-06/2813449-rails-sql-injection-facts).

### 4.Mass-assignment
相信这个问题很多人都曾听说过，[github也曾因此被hack过](http://www.2cto.com/Article/201203/122344.html)。
> 如果不做一些预防措施，Modle.new(params[:model])允许攻击者设置任何数据库字段的值。

假设在注册时用户传上来的参数是这样的：
```ruby
params[:user] #=> {:name => “ow3ned”, :admin => true}
```
这样会把这个用户置为一个admin用户，这是一个挺危险的处理。
如果Modle中包含一些敏感属性，例如admin是判断是否为管理员的Boolean，它不应该让使用者进行修改。这时我们就必须用attr_protected(黑名单)或attr_accessible(白名单)来保护这些属性。

Rails如今默认设置了一个空的白名单，具体可以参看：
```ruby application.rb
# Enforce whitelist mode for mass assignment.
# This will create an empty whitelist of attributes available for mass-assignment for all models
# in your app. As such, your models will need to explicitly whitelist or blacklist accessible
# parameters by using an attr_accessible or attr_protected declaration.
config.active_record.whitelist_attributes = true
```
自定义白名单,这样你就只能对user中的name就行赋值
```ruby
class User < ActiveRecord::Base
  attr_accessible :name
end
```
那些被保护的属性如果要赋值，就必须手动处理，而且通常会在不同的Controller中，例如只会出现在后台管理中：
```ruby
params[:user] #=> {:name => "ow3ned", :admin => true}
@user = User.new(params[:user])
@user.admin #=> false # not mass-assigned
@user.admin = true
@user.admin #=> true
```
也可以看看DHH和wycats给出的方法：

* DHH:[https://gist.github.com/dhh/1975644](https://gist.github.com/dhh/1975644)
* wycats:[https://gist.github.com/wycats/1974187](https://gist.github.com/wycats/1974187)

### 5.不要相信ID参数
```ruby
def show
  @order = Order.find(params[:id], :conditions => ["user_id = ?", params[:user_id]])
rescue
  redirect_to :action => "index"
end
```
更好的办法是基于集合的查找方法。比如说user和order之间存在一对多的关联，代码可改为：
```ruby
def show
  @order = current_user.orders.find(params[:id])
rescue
  redirect_to :action => "index"
end
```

### 6.不要暴露controller中的方法
控制器中多有的public方法都是可以被浏览器调用到的action，除此之外的任何方法都应该是protected或private的.

#### 参考：
* http://ihower.tw/rails3/security.html
* http://guides.rubyonrails.org/security.html
* http://blog.sina.com.cn/s/blog_8184e033010156nd.html
* http://www.cnblogs.com/siqi/archive/2012/11/19/2777224.html
* http://railscasts.com/episodes/178-seven-security-tips
* http://railscasts.com/episodes/204-xss-protection-in-rails-3
* http://sunfengcheng.iteye.com/blog/232636
* http://www.csdn.net/article/2013-01-06/2813449-rails-sql-injection-facts
