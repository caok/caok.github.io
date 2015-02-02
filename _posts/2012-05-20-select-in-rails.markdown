---
layout: post
title: "select in rails"
date: 2012-05-20 12:52
comments: true
categories: [Rails, Select, Web]
---

一般下拉选择框
--------------

{% highlight ruby %}
# index.html.haml
= search_form_for @q do |f|
  = f.label :warehouse_id_eq
  = f.collection_select :warehouse_id_eq, Warehouse.all, :id, :name, :include_blank => true
  = f.label :recycled_eq
  = f.select :recycled_eq, Item::RECYCLED_TYPES.map {|k, v| [v, k]}, :include_blank => true
  = f.submit
{% endhighlight %}

{% highlight ruby %}
# _form.html.haml
= semantic_form_for resource do |f|
  = render "shared/error_messages", :target => resource
  = f.inputs do
    = f.input :warehouse, :collection => Warehouse.all
    = f.input :recycled, :collection => Item::RECYCLED_TYPES.map {|k, v| [v, k]}, :as => :select
  = f.buttons do
    = f.commit_button
{% endhighlight %}

二级下拉选择框
--------------

{% highlight ruby %}
# index.html.haml
= search_form_for @q do |f|
  = f.label :category_id_in
  = f.grouped_collection_select :category_id_in, Category.roots, :descendants, :name, :id, :name, :include_blank => true
  = f.submit
{% endhighlight %}

{% highlight ruby %}
# _form.html.haml
= semantic_form_for(@item) do |f|
  = render "shared/error_messages", :target => @item
  = f.inputs do
    = f.input :category, :collection => option_groups_from_collection_for_select(Category.roots, :descendants, :name, :id, :name, @delivery.category_id)
  = f.buttons do
    = f.commit_button
{% endhighlight %}

> option_groups_from_collection_for_select(collection, group_method, group_label_method, option_key_method, option_value_method, selected_key = nil)

Auto-Complete Association
------------------------
> 针对有些情况下下拉菜单过长导致选择不便，此处我将演示通过输入文字自动补全来选择

> 我这里要在product的view中自动补全thing

{% highlight ruby %}
gem 'rails3-jquery-autocomplete'
{% endhighlight %}

{% highlight ruby %}
resources :products do
  collection do
    get :autocomplete_thing_name
  end
end
{% endhighlight %}

> 这里想显示name、material、size三个字段

{% highlight ruby %}
def name_with_material_and_size
  "#{name} - #{material} - #{size}"
end
{% endhighlight %}

{% highlight ruby %}
# products_controller.rb
autocomplete :thing, :name, :full => true, :display_value => :name_with_material_and_size, :extra_data => [:name, :material, :size]
{% endhighlight %}

{% highlight ruby %}
# products.js.coffee
jQuery ->
  $("#autocomplete_thing_name").bind('railsAutocomplete.select', (e,data)->
    $("#product_thing_id").val(data.item.id)
  )
{% endhighlight %}

{% highlight ruby %}
# the view of product
= f.hidden_field :thing_id
= f.input :thing_id, :as => :autocomplete, :url => autocomplete_thing_name_products_path, :input_html => {:id => 'autocomplete_thing_name', :name => "thing_name"}
{% endhighlight %}

#### 缺点

> 1.默认只能显示十个选项，所以对于名字相同的较多的情况不大适合

> 2.输入两个字及其以上时才开始自动匹配

> 3.输入了文字但是不选择或是未选中而直接提交，会递交失败

二级联动选择
------------

>  考虑到上述情况，设置二个选择框，第二个选择框中的根据第一个选择框中选中的内容而定

{% highlight ruby %}
# routes.rb
resources :products do
  get 'load_things', :on => :collection
end
{% endhighlight %}

{% highlight ruby %}
# products_controller.rb
def load_things
  if request.xhr? and params[:category_id]
    things = Thing.where(:category_id => params[:category_id])
    render :partial => 'thing', :locals => { :product => Product.find(params[:product_id]), :things => things, :disabled => false }
  else
    render :text => 'No parameters got sent.'
  end
end
{% endhighlight %}

{% highlight ruby %}
# products.js.coffee
jQuery ->
  $('#product_category_id').change( (e)->
    category_entry = $('#product_category_id')
    product_entry = $('#product_id')
    placeholder = $('#product_thing_id_input')
    error_text = '<p>错误</p>'

    $('#product_thing_id').remove()

    $.get(
      '/products/load_things',
      {category_id: category_entry.val(), product_id: product_entry.val()}
      (data) ->
        placeholder.replaceWith data
      'text'
    ).error( ->
      placeholder.append error_text
    )

    e.preventDefault()
  )
{% endhighlight %}

{% highlight ruby %}
# the view of product
= semantic_form_for(@product, :url => {:action => "add_thing_to_product"}, :html => {:method => :post}) do |f|
  = f.inputs do
    = f.hidden_field :id
    = f.hidden_field :thing_id
    = f.input :category_id, :collection => option_groups_from_collection_for_select(Category.roots, :descendants, :name, :id, :name, @product.category_id)

    = render :partial => 'thing', :locals => { :product => @product, :things => @things, :disabled => @product.persisted? }

  = f.buttons do
    = f.commit_button
{% endhighlight %}

{% highlight ruby %}
# _thing.html.haml
= semantic_fields_for :product, product do |f|
  = f.input :thing_id, :collection => things, :member_label => :name_with_material_and_size, :input_html => { :disabled => disabled }
{% endhighlight %}
