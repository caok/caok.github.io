---
layout: post
title: "caching with rails"
date: 2013-01-30 19:37
categories: [Rails]
tags: [Rails]
---

### 1.Page caching
Page caching是最简单最高效的一种，它会将Action最后的HTML结果存成public/下的HTML文件，也就是静态网页。

{% highlight ruby %}
class ProductsController < ActionController
  caches_page :index
  def index; end
end
{% endhighlight %}

不过缺点也同样明显，由于是静态网页，对于任何的request都会返回同一个结果，适用面比较窄。
比如用kaminari等定义了翻页功能，就将失效，始终停留在第一页。

如果要真的使用的话，可以在有新的product创建的时候更新index。
{% highlight ruby %}
class ProductsController < ActionController
  caches_page :index

  def index
    @products = Products.all
  end

  def create
    expire_page :action => :index
  end
end
{% endhighlight %}

expire_page会在有create操作的时候，清除cache里的资料重新生成。

### 2.Action caching
Rails在development模式下的cache默认是关闭的。

打开Rails的cache很简单，只要在环境对应的配置文件config/environments/*.rb中设置即可：
    config.action_controller.perform_caching = true

action caching与page caching的区别在于request会经过web server并且被rails application接收，直到所有的before filters被处理。
{% highlight ruby %}
class ProductsController < ActionController
  before_filter :authenticate, :only => :create
  caches_action :index

  def index
    @products = Product.all
  end

  def create
    expire_action :action => :index
  end
end
{% endhighlight %}

缺点也和Page caching一样，无法提供不同使用者有不同內容。

expire_action会在有create操作的时候，清除cache里的资料重新生成。

### 3.Fragment Caching
动态web应用程序的页面往往由多个components组成，而每个component经常需要采取不同的cache、expire策略。针对这个问题，rails提供了fragment cache。

Fragment caching可以只缓存HTML中的一小段元素，我們可以自由选择要cache的区域。这种cache发生在View中，所以我們须把cache程式写在View中，用cache包起來要cache的Template：

如果一个页面有多个component被cache，则需要添加suffix来区分它们：
{% highlight ruby %}
<% cache(:action => 'recent', :action_suffix => 'all_products') do %>
  All available products:
{% endhighlight %}

如果你想要一个无需绑定到相应action的cache块，可以赋予一个全局的key
{% highlight ruby %}
<% cache 'all_available_products' do %>
  All available products:
<% end %>
{% endhighlight %}

cache方法接受一个可选参数。这个参数被用作缓存的key(默认情况下，页面的URL会被作为缓存的key)。如果我们把模型(model)当作参数,那么模型的cache_key属性将被作为这个key。这就是说，当article更新的时候这个缓存片段就会过期。
{% highlight ruby %}
<% title @article.name %>  
<% cache @article do %>  
  <p class="author"><em>from <%=h @article.author_name %></em></p>  
  <% for comment in @article.comments %>  
    <div id="comment">  
      <strong><%= link_to_unless comment.site_url.blank?, h(comment.author_name), h(comment.site_url) %></strong>  
      <em>on <%= comment.created_at.strftime('%b %d, %Y at %H:%M') %></em>  
      <%= simple_format comment.content %>  
    </div>  
  <% end %> 
  <!-- Rest of code omitted -->  
<% end %>  
<h3>Add your comment:</h3>  
<%= render :partial => 'comments/form' %>
{% endhighlight %}

cache_key由模型(model)名,模型的id和updated_at属性组成。key的最后一段非常有用，因为这一段组成部分，这个key每次都会应为模型的更新而改变。这样每次模型的任意属性有更改，这个缓存片段都会过期。

在我们的应用程序里一个Article可能有很多Comments。如果我们使用article页面的表单对article添加一条comment，这条comment将不会作为article页面的一部分被显示。这是因为article已经被缓存了，article页面只会显示缓存里面的comments。当一条comment被添加时article的时间戳未被修改，所以缓存片段不会过期。

要想实现当添加或修改comment时article页面显示新的comment，我们仅仅需要对comment模型(model)做一点点修改:
{% highlight ruby %}
class Comment < ActiveRecord::Base  
  belongs_to :article, :touch => true  
end
{% endhighlight %}

给belongs_to关系添加 :touch => true 意味着当创建，更新或者删除一条comment的时候，该comment属于(belongs_to)的article被touched。现在我们添加一条comment，缓存会失效并且页面会更新而且显示刚添加的comment。

touch: true

saves the record with the updated_at/on attributes set to the current time

清理cache
---------
手动清除
    rake tmp:cache:clear
在资料修改或刪除时，在适当的Controller Action中过期这些cache资料
{% highlight ruby %}
expire_fragment(:controller => 'products', :action => 'recent',  :action_suffix => 'all_prods)
expire_fragment(:key => ['all_available_products', @latest_product.created_at].join(':'))
{% endhighlight %}

### 4.Sweepers
但如上面这样到处写expire方法，显然不是最好的方式。Rails针对此提供了sweeper机制：把cache清理移入一个observer类，此类会监测一个对象的变化，并且通过相应的钩子来清理此对象相关的cache。
{% highlight ruby %}
# sweepers/store_sweeper.rb
class StoreSweeper < ActionController::Caching::Sweeper
  # This sweeper is going to keep an eye on the Product model
  observe Product

  # If our sweeper detects that a Product was created call this
  def after_create(product)
      expire_cache_for(product)
  end

  # If our sweeper detects that a Product was updated call this
  def after_update(product)
    expire_cache_for(product)
  end

  # If our sweeper detects that a Product was deleted call this
  def after_destroy(product)
    expire_cache_for(product)
  end

  private
  def expire_cache_for(record)
    # Expire the list page now that we added a new product
    expire_page(:controller => '#{record}', :action => 'list')

    # Expire a fragment
    expire_fragment(:controller => '#{record}', :action => 'recent', :action_suffix => 'all_products')
  end
end
{% endhighlight %}

Cache sweeper在controller里面就是一个after或者aroud filter
{% highlight ruby %}
class ProductsController < ActionController   
  cache_sweeper :store_sweeper, :only => [ :create, :update, :destroy ]   
  caches_page :list
  
  def create
  end

  def list
  end
end
{% endhighlight %}

### 5.Counter cache
如果需要常计算has_many的Model有多少笔记录，例如显示文章列表时，也要显示每篇有多少留言回复。
{% highlight ruby %}
<% @topics.each do |topic| %>
  主題：<%= topic.subject %>
  回复數：<%= topic.posts.size %>
<% end %>
{% endhighlight %}

这时Rails会产生一笔笔的SQL count查询：
    SELECT * FROM `posts` LIMIT 5 OFFSET 0
    SELECT count(*) AS count_all FROM `posts` WHERE (`posts`.topic_id = 1 )
    SELECT count(*) AS count_all FROM `posts` WHERE (`posts`.topic_id = 2 )
    SELECT count(*) AS count_all FROM `posts` WHERE (`posts`.topic_id = 3 )
    SELECT count(*) AS count_all FROM `posts` WHERE (`posts`.topic_id = 4 )
    SELECT count(*) AS count_all FROM `posts` WHERE (`posts`.topic_id = 5 )
Counter cache功能可以把這個數字存進資料庫，不再需要一筆筆的SQL count查詢，並且會在Post數量有更新的時候，自動更新這個值。

首先，你必须要在Topic Model新增一個栏位叫做posts_count，依照慣例是_count结尾，型別是integer，有预设值0。

    rails g migration add_posts_count_to_topic

编辑Migration：
{% highlight ruby %}
class AddPostsCountToTopic < ActiveRecord::Migration
  def self.up
    add_column :topics, :posts_count, :integer, :default => 0

    # 如果是网站上线后才新增这个功能，这里需要先计算设定好初始值
    Topic.find_each do |topic|
      Topic.reset_counters topic.id, :posts
    end
  end

  def self.down
    remove_column :topics, :posts_count
  end
end
{% endhighlight %}

编辑Models，加入:counter_cache => true：
{% highlight ruby %}
class Topic < ActiveRecord::Base
  has_many :posts
end

class Posts < ActiveRecord::Base
  belongs_to :topic, :counter_cache => true
end
{% endhighlight %}

这样同样的@topic.posts.size，就会自动变成使用@topic.posts_count，而不會用SQL count查詢一次。

### 6.rails.cache
{% highlight ruby %}
Rails.cache.read("city")   # => nil
Rails.cache.write("city", "Duckburgh")
Rails.cache.read("city")   # => "Duckburgh"

cache.write("today", "Monday")
cache.fetch("today")  # => "Monday"

cache.fetch("city")   # => nil
cache.fetch("city") do
  "Duckburgh"
end
cache.fetch("city")   # => "Duckburgh"
{% endhighlight %}

#### 参考
* [http://robbinfan.com/blog/38/orm-cache-sumup](http://robbinfan.com/blog/38/orm-cache-sumup)
* [http://guides.rubyonrails.org/caching_with_rails.html](http://guides.rubyonrails.org/caching_with_rails.html)
* http://ihower.tw/rails3/caching.html
* http://broadcastingadam.com/2011/05/advanced_caching_in_rails/
* http://andyhu1007.iteye.com/blog/413172
* http://railscasts.com/episodes/90-fragment-caching
* http://railscasts.com/episodes/172-touch-and-cache
* http://railscasts.com/episodes/23-counter-cache-column
* http://railscasts.com/episodes/387-cache-digests
* http://www.codelearn.org/blog/rails-cache-with-examples
