---
layout: post
title: "rails render partial"
date: 2014-05-17 14:08
comments: true
categories: [Rails]
---

### 1. 默认参数
```ruby
<%= render partial: "account" %>  
```
默认本地有个变量@account传递过去，render到的partial(./_account.erb)有个对应的变量account.

### 2. 单独参数 locals 
locals传递一组hash参数hash 值是本地的变量，hash的key是partial里的变量 
```ruby
<%= render partial: "account", locals: { account: @buyer } %>
  
<% for ad in @advertisements %>
  <%= render partial: "ad", locals: { ad: ad } %>
<% end %>
```
上面两个render分别
* 传递本地变量@buyer到_account.erb里叫account
* 传递本地变量ad到_ad.erb里叫ad

<!-- more -->

##### 根据1默认参数下面两个是一样的
```ruby
<%= render partial: "contract", locals: { contract: @contract } %>

<%= render partial: "contract" %>
```

### 3. as
用来改变传递之后，partial里变量的名称，如下，render方式是一样的。
```ruby
<%= render partial: "contract", as: :agreement
等同于
<%= render partial: "contract", locals: { agreement: @contract }  
```
传递@contract到_contract.erb里，partial里的变量名为agreement 

### 4. object
object最简单，就是把一个**变量原名**传递到partial里，所以什么都记不清楚的时候，就用object多写点都能表达 
```ruby
<%= render partial: "account", object: @buyer %>  

<% for ad in @advertisements %>  
  <%= render partial: "ad", object: ad %>  
<% end %>  
```
* 传递@buyer到_account.erb的partial里变量名还是@buyer
* 传递ad到_ad.erb的partial里变量名还是ad 

### 5. object和as合用 
```ruby
<%= render partial: "contract", object: @contract, as: :contract %>  
等同于
<%= render partial: "contract" %>  
```

### 6. collection 
collection选项用于指定被传递给partial的集合对象
```ruby
<%= render partial: "ad", collection: @advertisements %>  

<%= render partial: "ad", collection: @advertisements, as: :item %>  
```
这里不得不提一下,与4中通过for循环来render虽然能达到一样的效果,不过渲染速度上却差了好几倍,所以建议使用collection的方式.

#### 下标索引值
在 设置:collection选项的时候，rails同时提供了一个counter变量给partial模板，变量名以partial名(不带下划线)开，以_counter结尾，并且经试验，这个变量名不受:as选项影响(也就是说在上面的代码中，这个变量名应该是ad_counter而不是item_counter)。其值为collection对象的索引值(从0开始)。
#### :spacer_template
:spacer_template选项用于指定填充于collection每个member之间的模板：
```ruby
<%= render partial: "ad", collection: @advertisements, spacer_template: "ad_ruler" %>
```
上面的代码中，_ad_ruler.html.erb的内容将被填充到每一对_ad partial之间。

和:object一样，:collection也有简写形式：
```ruby
<%= render partial: @advertisements %>
```

### 7. layout 
```ruby
<%# app/views/users/index.html.erb &>  
Here's the administrator:  
<%= render partial: "user", layout: "administrator", locals: { user: administrator } %>  
  
Here's the editor:  
<%= render partial: "user", layout: "editor", locals: { user: editor } %>  
  
<%# app/views/users/_user.html.erb &>
Name: <%= user.name %>

<%# app/views/users/_administrator.html.erb &>
<div id="administrator">
  Budget: $<%= user.budget %>
  <%= yield %>
</div>
  
<%# app/views/users/_editor.html.erb &>
<div id="editor">
  Deadline: <%= user.deadline %>
  <%= yield %>
</div>
```

### 8. 默认 
```ruby
#@account是一条记录  
<%= render partial: "accounts/account", locals: { account: @account} %>  
等同于
<%= render partial: @account %>  
```
```ruby
# @posts是一组记录  
<%= render partial: "posts/post", collection: @posts %>  
等同于
<%= render partial: @posts %>  
```
* 看你要partial的变量是一组记录还是一条记录，会对应约定用locals和collection
* 如果要传递给partial的实例变量名==partial名=model名，可以简写

### 9. 一些漂亮的简写
```ruby
#<%= render partial: "account" %>可用下面代替  
<%= render "account" %>

#<%= render partial: "account", locals: { account: @buyer } %>可用下面代替  
<%= render "account", account: @buyer %>

# @account是一条记录  
# <%= render partial: "accounts/account", locals: { account: @account } %>可用下面代替  
<%= render(@account) %>

# @posts是一组记录  
# <%= render partial: "posts/post", collection: @posts %>可用下面代替  
<%= render(@posts) %>
```

#### 参考:
* http://hlee.iteye.com/blog/1084702
* http://apidock.com/rails/ActionController/Base/render
* http://www.tuicool.com/articles/n222Q3
* http://blog.sina.com.cn/s/blog_812973c301011ct3.html
