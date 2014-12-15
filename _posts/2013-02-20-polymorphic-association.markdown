---
layout: post
title: "多态关联"
date: 2013-02-20 20:17
comments: true
categories: [Rails]
---

所谓多态关联，就是一个model可以通过单一的关联从属于多个model。比如article、photo、event等都需要评论(comment)，如果没有多态关联，就需要为每个model都建立属于自己的comment model，这样代码会有很多的重复。通过[polymorphic-associations](http://guides.rubyonrails.org/association_basics.html#polymorphic-associations)就可以只建立一个comment model，然后让每一条comment知道自己属于哪个model就可。

<!-- more -->

### 1.创建comment model
```
rails g model Comment content:text commentable_id:integer commentable_type:string
```
通过此命令生成的migration文件为：
```ruby
class CreateComments < ActiveRecord::Migration
  def change          
    create_table :comments do |t| 
      t.text :content
      t.integer :commentable_id
      t.string  :commentable_type
      t.timestamps 
    end               
  end                 
end  
```
这个migration可以通过t.references来简化，并且对其增加索引来提速。
```ruby
class CreateComments < ActiveRecord::Migration
  def change          
    create_table :comments do |t| 
      t.text :content
      t.references :commentable, :polymorphic => true
      t.timestamps
    end
    add_index :comments, [:commentable_id, :commentable_type]
  end
end
```
给相应的model增加关联
```ruby app/model/comment.rb
class Comment < ActiveRecord::Base
  belongs_to :commentable, :polymorphic => true
end
```
在article、photo、event的model中增加
```ruby
has_many :comments, :as => :commentable
```

### 3.创建comment controller
```
rails g controller comment index new
```
```ruby app/controllers/comments_controller.rb
class CommentsController < ApplicationController
  before_filter :load_commentable 
 
  def index
    @comments = @commentable.comments
  end
  
  def new
    @comment = @commentable.comments.new
  end
      
  def create
    @comment = @commentable.comments.new(params[:comment])
    if @comment.save
      redirect_to [@commentable, :comments], :notice => "Comment created."
    else
      render :new
    end
  end
  
  private
  # 获取正确的评论
  def load_commentable
    resource, id = request.path.split('/')[1, 2]                        # /articles/1
    @commentable = resource.singularize.classify.constantize.find(id)   # Artiale.find(1)
  end
end
```
### 4.设置[nested resources](http://guides.rubyonrails.org/routing.html#nested-resources)
```ruby config/routes.rb
resources :articles do
  resources :comments
end
#同理处理photo和event
```
这样我们就可以通过URL(/articles/1/comments)访问一篇文章article的所有评论

### 5.增加相应的views
```ruby app/views/comments/index.html.slim
h1 Comments
= render 'comments'
p= link_to "New comment", [:new, @commentable, :comment]
```
```ruby app/views/comments/_comments.html.slim
- @comments.each do |comment|
  .comment
    = simple_format comment.content
    = link_to "Change", [:edit, @commentable, comment]
    = link_to "Delete", [@commentable, comment], :method => :delete
```
```ruby app/views/comments/new.html.slim
h1 Comments#new
= render 'form'
```
```ruby app/views/comments/_form.html.slim
= form_for [@commentable, @comment] do |f|
  .field
    = f.text_area :content
  = f.submit
```
这里将index和create中的部分给抽出来，方便在其他地方使用

### 6.在article中增加comments
```ruby app/views/articles/show.html.slim
h1= @article.name
= simple_format @article.content
p= link_to "Back to Articles", articles_path

h2 Comments
= render "comments/comments"
= render "comments/form"
```
相应的需要在article的show中处理下
```ruby app/controllers/articles_controller.rb
def show
  @article = Article.find(params[:id])
  @commentable = @article
  @comments = @commentable.comments
  @comment = Comment.new
end
```
这样就可以直接在articles/1中直接增加评论，当然需要修改下comment创建成功后的跳转。
```ruby
def create
  @comment = @commentable.comments.new(params[:comment])
  if @comment.save
    redirect_to @commentable, :notice => "Comment created."
  else
    render :new              
  end
end
```

#### 参考：
* http://guides.rubyonrails.org/association_basics.html#polymorphic-associations
* http://guides.rubyonrails.org/routing.html#nested-resources
* http://cn.asciicasts.com/episodes/154-polymorphic-association
* http://railscasts.com/episodes/154-polymorphic-association-revised
* https://github.com/railscasts/154-polymorphic-association-revised
