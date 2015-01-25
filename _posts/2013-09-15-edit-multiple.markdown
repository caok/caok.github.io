---
layout: post
title: "edit multiple"
date: 2013-09-15 20:54
comments: true
categories: [Rails]
---

railscast的[165 Edit Multiple (revised)](http://railscasts.com/episodes/165-edit-multiple-revised)中介绍下编辑多条数据库记录的方法

### 方法一(inline)
允许你一次选中多个product，然后每个被选中的product都会被设置成"discontinued"。
{% highlight ruby %}
# products/index.html
<%= form_tag discontinue_products_path, method: :put do %>
  <table>
    <tr>                         
      <th></th>                  
      ........
    </tr>
    <% @products.each do |product| %>
      <tr>
        <td><%= check_box_tag "product_ids[]", product.id %></td>
        .........
      </tr>                      
    <% end %>
  </table>
  <%= submit_tag "discountinue checked", data: { disable_with: "Please wait..." } %>
<% end %>
{% endhighlight %}

{% highlight ruby %}
# routes.rb
resources :products do
  collection do
    put :discontinue
  end
end
{% endhighlight %}

{% highlight ruby %}
# products_controller.rb
def discontinue
  Product.update_all({discontinued: true}, {id: params[:product_ids]})
  redirect_to products_url
end
{% endhighlight %}

#### 补充
form_tag的写法，不同于form_for没有将属性与model绑定起来，而是直接写属性名。每个helper方法都有两个参数，一个是域的名字，另一个是域的值。

submit_tag:创建一个以文本值作为标题的提交按钮。

submit_tag中的:disable_with选项的作用是:在button被点击之后把它disable掉，并且把button的文字替换成“Please wait...”。一个简单的选项带来了显而易见的好处:不仅避免了多次点击的问题，而且显式地告诉了用户表单正在被提交当中。

创建一个复选框表单

check_box_tag(name, value = "1", checked = false, options = {})
{% highlight ruby %}
# 示例
check_box_tag 'eula', 'accepted', false, disabled: true
# => <input disabled="disabled" id="eula" name="eula" type="checkbox" value="accepted" />
{% endhighlight %}

check_box_tag的name，它是以一个方括号结尾的，这是所有传上去的值将会被集合成一个数组。
{% highlight ruby %}
<%= check_box_tag "product_ids[]", product.id %>
等同于页面上的效果
<input id="product_ids_" name="product_ids[]" type="checkbox" value="1">
{% endhighlight %}

### 方法二(individually)
让用户选择更新一批记录的多个属性，而不是之前的单个属性.首先利用checkbox选择要修改的product,我们先把显示所有product的table标签用form包起来,如果我们没有为form指定提交方式，所以它默认是POST，但是我们需要通过这个form进入到一个显示所有选中的product的页面,所以我们需要用的应该是GET
{% highlight ruby %}
# products/index.html
<%= form_tag edit_multiple_products_path, method: :get do %>
  <table>
    <tr>                         
      <th></th>                  
      ........
    </tr>
    <% @products.each do |product| %>
      <tr>
        <td><%= check_box_tag "product_ids[]", product.id %></td>
        .........
      </tr>                      
    <% end %>
  </table>
  <%= submit_tag "edit checked" %>
<% end %>
{% endhighlight %}

{% highlight ruby %}
# routes.rb
resources :products do
  collection do
    get :edit_multiple
    put :update_multiple
  end
end
{% endhighlight %}

{% highlight ruby %}
# products/edit_multiple.html.erb
<h1>Edit Checked Products</h1>
<%= form_tag update_multiple_products_path, method: :put do %>
  <% @products.each do |product| %>
    <h2><%= product.name %></h2>
    <%= fields_for "products[]", product do |f| %>
      <% if product.errors.any? %>
        <div id="error_explanation">
          <h2><%= pluralize(product.errors.count, "error") %> prohibited this product from being saved:</h2>
          <ul>
          <% product.errors.full_messages.each do |msg| %>
            <li><%= msg %></li>
          <% end %>
          </ul>
        </div>
      <% end %>
      <div class="field">
        <%= f.label :name %><br />
        <%= f.text_field :name %>
      </div>
      <div class="field">
        <%= f.label :price %><br />
        <%= f.text_field :price %>
      </div>
      <div class="field">
        <%= f.label :category_id %><br />
        <%= f.collection_select :category_id, Category.order("name"), :id, :name %>
      </div>
      <div class="field">
        <%= f.check_box :discontinued %>
        <%= f.label :discontinued %>
      </div>
    <% end %>
  <% end %>
  <div class="actions">
    <%= submit_tag "update" %>
  </div>
<% end %>
{% endhighlight %}

{% highlight ruby %}
# products_controller.rb
def edit_multiple
  @products = Product.find(params[:product_ids])
end

def update_multiple
  Product.update(params[:products].keys, params[:products].values)
  redirect_to products_url
end
{% endhighlight %}

增加对字段限制的处理判断
{% highlight ruby %}
# products_controller.rb
def update_multiple
  @products = Product.update(params[:products].keys, params[:products].values)
  @products.reject! { |p| p.errors.empty? }
  if @products.empty?
    redirect_to products_url
  else
    render "edit_multiple"
  end 
end 
{% endhighlight %}

### 方法三(single-fieldset)
{% highlight ruby %}
# products/edit_multiple.html.erb
<h1>Edit Checked Products</h1>
           
<%= form_tag update_multiple_products_path, method: :put do %>
  <ul>  
    <% @products.each do |product| %>
      <li>
        <%= hidden_field_tag "product_ids[]", product.id %>
        <%= product.name %>
        <ul class="errors">
          <% product.errors.full_messages.each do |msg| %>
            <li><%= msg %></li>
          <% end %>
        </ul>
      </li>
    <% end %>
  </ul> 
  <%= fields_for :product do |f| %>
    <div class="field">
      <%= f.label :name %><br />
      <%= f.text_field :name %>
    </div>
    <div class="field">
      <%= f.label :price %><br />
      <%= f.text_field :price %>
    </div>
    <div class="field">
      <%= f.label :category_id %><br />
      <%= f.collection_select :category_id, Category.order("name"), :id, :name, include_blank: true %>
    </div>
    <div class="field">
      <%= f.label :discontinued %><br/>
      <%= f.select :discontinued, [["Yes", true], ["No", false]], include_blank: true %>
    </div>
  <% end %>
  <div class="actions">
    <%= submit_tag "update" %>
  </div>
<% end %>
{% endhighlight %}

{% highlight ruby %}
# products_controller.rb
def update_multiple
  @products = Product.find(params[:product_ids])
  @products.reject! do |product|
    product.update_attributes(params[:product].reject { |k,v| v.blank? })
  end 
  if @products.empty?
    redirect_to products_url
  else
    @product = Product.new(params[:product])
    render "edit_multiple"
  end 
end 
{% endhighlight %}

### 参考:
* http://railscasts.com/episodes/165-edit-multiple-revised
* http://railscasts.com/episodes/165-edit-multiple
* http://www.infoq.com/cn/articles/high-quality-web-apps-on-rails/
* http://railscasts.com/episodes/52-update-through-checkboxes
* http://cn.asciicasts.com/episodes/165-edit-multiple
* http://cn.asciicasts.com/episodes/16-virtual-attributes
