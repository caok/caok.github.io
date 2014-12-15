---
layout: post
title: "STI and Polymorphic Associations"
date: 2013-10-06 15:01
comments: true
categories: [Rails]
---

### STI(单表继承)
关于继承，Rails內建了其中最简单的一个方法，只用一个数据表来存储继承关系，搭配使用一个名为type的字段来指定这个资料的类别名称。这种方法就是STI(Single-table inheritance,单表继承).

要使用STI功能，依照Rails惯例只需要有一个叫 type 的字符串类型的字段就行。假设以下的users表有个字段叫type，那这三个Models实际上就会共用users这张表。
```ruby
class User < ActiveRecord::Base
end

class Guest < User
end

class Member < User
end
```
在new或create的时候rails就会根据是使用的是Guest还是Member，自动去设定type字段为Guest或Member。

<!-- more -->

但也由于Rails惯例的情况，我们不能把type挪作他用。STI最大的问题在于字段的浪费，如果继承关系中交集的字段不多，那么使用STI就会非常浪费空间。如果有较多的不共用的字段，就建议不要使用STI，而是使用Polymorphic Associations，各自使用各自的表。要关闭STI，只要在父类别中加上self.abstract_class = true

有以下三种方式判断其类型
```ruby
User.first.type == Guest # true
User.first.is_a? Guest # true
User.first.class == Guest # true
```
在处理代码路径的时候我们还可以这么处理，使得结构更加清晰
```
* app
*   models
*     user.rb
*     users
*       pc.rb
*       mac.rb
```
当然Rails中不会自动识别这个路径，我们可以这么处理:
```ruby config/application.rb
# Load Subfolder Models
config.autoload_paths += Dir[Rails.root.join('app', 'models', '{**}')]
```
如果这些对象有相似的属性不同的行为，或者要对这些对象要做数据库查询，这时用STI比较合适;如果这些对象有许多不同的属性字段就建议使用Polymorphic Associations,以避免字段的浪费。可以看下这篇[Single Table Inheritance in Rails](http://www.alexreisner.com/code/single-table-inheritance-in-rails)

### Polymorphic Associations(多态关联)
Polymorphic Associations可以让一个 Model 不一定关联到某一个特定的 Model，秘诀在于除了整数类型的_id外部键之外，再加一个字符串类型的_type字段来说明是哪一种Model。

例如一个Comment model，我們可以通过多态关联让它belongs_to到各种不同的Model上，假设我们已经有了Article與与Photo这两个Model，然后我们希望这两个Model都可以被留言。不用多态关联的话，你得分别建立ArticleComment和PhotoComment两个model。用多态关联的话，无论有多少种需要被留言的model，都只需要一个Comment model就可以了。
```ruby
class CreateComments < ActiveRecord::Migration
  def change
    create_table :comments do |t|
      t.text :content
      t.integer :commentable_id
      t.string :commentable_type

      t.timestamps
    end
  end
end
```
这个Migration中，我们用content这个字段来存储留言的內容，commentable_id用来储存被留言的物件的id而commentable_type则用來储存被留言物件的种类，以这样来说明被留言的对象就是Article与Photo这两种Model，这个Migration文件也可以改写成下面这样：
```ruby
class CreateComments < ActiveRecord::Migration
  def change
    create_table :comments do |t|
      t.text :content
      t.belongs_to :commentable, :polymorphic => true

      t.timestamps
    end
  end
end
```
在model指定其关系
```ruby
class Comment < ActiveRecord::Base
  belongs_to :commentable, :polymorphic => true
end

class Article < ActiveRecord::Base
  has_many :comments, :as => :commentable
end

class Photo < ActiveRecord::Base
  has_many :comments, :as => :commentable
end
```
```ruby
article = Article.first

# 透过关联关系新增留言
comment = article.comments.create(:content => "First Comment")

# 你可以发现 Rails 很聪明的帮我们指定了被留言物件的种类和id
comment.commentable_type => "Article"
comment.commentable_id => 1

# 也可以透过 commentable 反向回查关联的物件
comment.commentable => #<Article id: 1, ....>
```

### 参考:
* http://railscasts.com/episodes/394-sti-and-polymorphic-associations
* http://ihower.tw/rails3/activerecord-others.html
* http://robots.thoughtbot.com/post/159809241/whats-the-deal-with-rails-polymorphic-associations
* http://www.alexreisner.com/code/single-table-inheritance-in-rails
* http://www.martinfowler.com/eaaCatalog/singleTableInheritance.html
* http://rails-bestpractices.com/posts/45-use-sti-and-polymorphic-model-for-multiple-uploads
* http://blog.thirst.co/post/14885390861/rails-single-table-inheritance
* http://www.therailworld.com/posts/18-Single-Table-Inheritance-with-Rails
