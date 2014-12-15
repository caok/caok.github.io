---
layout: post
title: "dynamic forms"
date: 2013-09-20 07:59
comments: true
categories: [Rails]
---

在日常开发中经常会遇到需要留给客户(或者自己)自定义的字段，以便满足在正式使用时某些字段无法记录的情况。简单一些的话，我们可以做些个预留字段，在需要时只需要在修改下页面就ok了。当然这样的话只能满足一些简单的情况，如果字段需要受输入限制或者字段类型不一样的话，上述做法就很难实现了。

在[railscast 403 Dynamic Forms](http://railscasts.com/episodes/403-dynamic-forms)中介绍了另外一种实现方式能较好的达到这种效果，总结下,以备以后的不时之需。

首先我们创建一个ProductType用来存储“属性扩展方式”，这样让可以让用户自定义。比如说我们有个product，但product的类型有多种比如说图书，衣服等等，像这些需要存储的字段除了一些基本的外其他的会有所不同，图书可能要记录下页码，作者之类的，而衣服可能需要记录尺寸大小。我们通过ProductType来记录是图书还是衣服。而图书这种产品类型有那些不同的字段需要扩展，我们就把它存储在ProductField中.这样我们就可以将product的基础属性直接保存在product那张表中，而其他的不同的字段我们就对其进行另外的存储处理。

### 1.创建扩展属性的结构
```sh
rails g scaffold ProductType name --skip-stylesheets
rails g model ProductField name field_type required:boolean product_type:belongs_to
```
这里我们在ProductField的name用来存字段的名称，field_type用来存字段的类型，required确定是否需要做字段输入限制，product_type的话指定它是哪种“商品属性扩展方式”.

<!-- more -->
每个productType都有多个ProductField
```ruby models/product_type.rb
class ProductType < ActiveRecord::Base
  attr_accessible :name, :fields_attributes
  has_many :fields, class_name: "ProductField"
  accepts_nested_attributes_for :fields, allow_destroy: true
end
```

### 2.完成扩展属性的页面功能
在_form.html.erb页面增加以下代码
```ruby views/product_types/_form.html.erb
<%= f.fields_for :fields do |builder| %>       
  <%= render 'field_fields', f: builder %>     
<% end %>                                      
<%= link_to_add_fields 'Add Field', f, :fields %> 
```
通过js动态增加或删除field:
```ruby views/product_types/_field_fields.html.erb
<fieldset>
  <%= f.select :field_type, %w[text_field check_box] %>
  <%= f.text_field :name, placeholder: 'field_name' %>
  <%= f.check_box :required %> <%= f.label :required %>
  <%= f.hidden_field :_destroy %>
  <%= link_to "[remove]", "#", class: "remove_fields" %>
</fieldset>
```
```ruby application_helper.rb
def link_to_add_fields(name, f, association)
  new_object = f.object.send(association).klass.new
  id = new_object.object_id
  fields = f.fields_for(association, new_object, child_index: id) do |builder|
    render(association.to_s.singularize + "_fields", f: builder)
  end
  link_to(name, '#', class: "add_fields", data: {id: id, fields: fields.gsub("\n", "")})
end
```
```javascript assets/javascripts/product_types.js.coffee
$(document).on 'click', 'form .remove_fields', (event) ->
  $(this).prev('input[type=hidden]').val('1')
  $(this).closest('fieldset').hide()
  event.preventDefault()

$(document).on 'click', 'form .add_fields', (event) ->
  time = new Date().getTime()
  regexp = new RegExp($(this).data('id'), 'g')
  $(this).before($(this).data('fields').replace(regexp, time))
  event.preventDefault()
```

### 3.在product中加入“属性扩展”
#### a.创建链接
处理product，使product和product_type相互关联，在products/index中增加如下代码
```ruby views/products/index.html.erb
<%= form_tag new_product_path, method: :get do %>
  <%= select_tag :product_type_id, options_from_collection_for_select(ProductType.all, :id, :name) %>
  <%= submit_tag "New product", name: nil %>
<% end %> 
```
当然相应的controller也需要处理下，将product_type_id的值传递给页面，以方便在创建的时候能取到product_type_id的值
```ruby controllers/products_controller.erb
def new
  @product = Product.new(product_type_id: params[:product_type_id])
end
```

#### b.结构
我们在product中设置了一个字段properties，用来需要存储的字段值，其次设置一个product_type_id用来存储扩展的属性类型
```sh
rails g migration add_type_to_products product_type_id:integer properties:text
```
```ruby models/product.rb
class Product < ActiveRecord::Base
  attr_accessible :name, :price, :product_type_id, :properties
  belongs_to :product_type     
  serialize :properties, Hash
end
```
#### 这里的serialize:
序列化(Serialize)通常指的是将一个物件转换成一个可被资料库存储及传输的纯文字形态，反之将这笔资料从资料库中读出后转回物件的动作我们就将其称为反序列(Deserialize)，Rails提供了serialize让你指定需要序列化资料的栏位，任何物件在存入资料库时就会自动序列化成YAML格式，而当从资料库取出时就会自动帮你反序列成原先的物件。在上面的例子中，properties通常是text形态让我们有更大的空间可以存储资料，然后我们将一个Hash物件序列化之后存储到properties里面：
```ruby serialize示例
> product = Product.create(:properties => { "author" => "jack", "url" => "foo" })
> Product.find(product.id).properties # => { "author" => "jack", "url" => "foo" }
```
虽然序列化很方便可以让你存储任意的物件，但是缺点是序列化资料就失去了透过资料库查询索引的功效，你无法在SQL的where条件中指定序列化后的资料。

#### c.form页面
在products/_form.html.erb中增加properties的处理，就是将扩展的属性值存储到properties中去
```ruby views/products/_form.html.erb
<%= f.hidden_field :product_type_id %>
<%= f.fields_for :properties, OpenStruct.new(@product.properties) do |builder| %>
  <% @product.product_type.fields.each do |field| %>
    <%= render "products/fields/#{field.field_type}", field: field, f: builder %>
  <% end %>
<% end %>
```
[OpenStruct](http://www.ruby-doc.org/stdlib-2.0/libdoc/ostruct/rdoc/OpenStruct.html)用于动态快速的将很多属性一起绑定到对象上。也可以看下这篇：[Struct和OpenStruct](http://www.iteye.com/topic/546347)。

```ruby views/products/fields/_text_field.html.erb
<div class="field">
  <%= f.label field.name %><br />
  <%= f.text_field field.name %>
</div>
```

```ruby views/products/fields/_check_box.html.erb
<div class="field">
  <%= f.check_box field.name %>
  <%= f.label field.name %>
</div>
```

#### d."扩展字段"的限制
加入字段限制，在product.rb中增加以下代码，主要是通过对required这个值的判断
```ruby models/product.rb
  validate :validate_properties
  
  def validate_properties
    product_type.fields.each do |field|
      if field.required? && properties[field.name].blank?
        errors.add field.name, "must not be blank"
      end
    end
  end
```

#### e.详情页面显示"扩展字段"
在product的详情页面增加扩展字段的显示，增加以下代码
```ruby views/products/show.html.erb
<% @product.properties.each do |name, value| %>
  <p>       
    <b><%= name.humanize %>:</b>
    <%= value %>
  </p>   
<% end %> 
```

### 其他
####缺点：
* 1.受text类型的限制，不能存放太多的字段信息
* 2.在做表间关联存储的时候比较麻烦，dynamic forms适合用于扩展单个表的字段属性

### 参考:
* http://railscasts.com/episodes/403-dynamic-forms
* http://ihower.tw/rails3/activerecord-others.html
* http://www.iteye.com/topic/546347
* http://www.ruby-doc.org/stdlib-2.0/libdoc/ostruct/rdoc/OpenStruct.html
