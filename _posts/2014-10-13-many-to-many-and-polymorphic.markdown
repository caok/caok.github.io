---
layout: post
title: "many to many and polymorphic"
date: 2014-10-13 17:14
categories: [Rails]
tags: [Rails]
---

最近在项目中遇到一个情景: 多对多 ＋ 多态, 在实现的过程中遇到了一些问题，这里将其整理出来。

#### 情景
"用户"去"订阅""视频"和"文章"。

即: 用户可以订阅多个视频和文章，同样的，视频(文章)可以被多个人订阅。

我们一点点来整理，首先用户可以订阅多个视频，考虑想到订阅多种对象，所以subscribe中试着去使用polymorphic.

user通过user_id有多个subscribe，每个subscribe通过多态belongs_to多种对象

首先我们定义一个subscribe的结构
{% highlight ruby %}
create_table "subscribes", force: true do |t|
  t.integer  "user_id"
  t.integer  "subscribable_id"
  t.string   "subscribable_type"
end
{% endhighlight %}

model中user和subscribe的关系(has_many)
{% highlight ruby %}
class User < ActiveRecord::Base
  has_many :subscribes
end

class Subscribe < ActiveRecord::Base
  belongs_to :user
end
{% endhighlight %}

model中video和subscribe的关系(polymorphic)
{% highlight ruby %}
class Subscribe < ActiveRecord::Base
  belongs_to :user
  belongs_to :subscribable, polymorphic: true     # 通过polymorphic关联到video和article
end

class Video < ActiveRecord::Base
  has_many :subscribes, as: :subscribable
end

class Article < ActiveRecord::Base
  has_many :subscribes, as: :subscribable
end
{% endhighlight %}

这样通过多态，我们就能分别在video和article中分别找到相应的subscribes
{% highlight ruby %}
user.subscribes
user.subscribes.first.subscribable ==> video/article
video.subscribes
video.subscribes.first.user
{% endhighlight %}

接下来我们加人video和user直接的直接关联，可以通过video.users来访问到user，很简单，通过subscribe就行
{% highlight ruby %}
class Video < ActiveRecord::Base
  has_many :subscribes, as: :subscribable
  has_many :users, through: :subscribes
end
{% endhighlight %}

反过来，通过user.videos来访问到video，会麻烦点，我们一点点解释下
{% highlight ruby %}
class User < ActiveRecord::Base
  has_many :videos, through: :subscribes, source: :subscribable, source_type: "Video"
end
{% endhighlight %}

* :source 选项指定 has_many :through 关联的关联源名字。只有无法从关联名种解出关联源的名字时才需要设置这个选项。
* :source_type 选项指定 has_many :through 关联中用来处理多态关联的关联源类型。
* :through 选项指定用来执行查询的连接模型。

这里还是比较怪的，这里的videos实际上是该用户订阅的videos，所以我们这再改写下
{% highlight ruby %}
has_many :subscribed_videos, through: :subscribes, class_name: "Video", source: :subscribable, source_type: "Video"
{% endhighlight %}

### 完整的代码如下
{% highlight ruby %}
create_table "subscribes", force: true do |t|
  t.integer  "user_id"
  t.integer  "subscribable_id"
  t.string   "subscribable_type"
end

class User < ActiveRecord::Base
  has_many :subscribes
  has_many :subscribed_videos, through: :subscribes, class_name: "Video", source: :subscribable, source_type: "Video"
  has_many :subscribed_articles, through: :subscribes, class_name: "Article", source: :subscribable, source_type: "Article"
end

class Subscribe < ActiveRecord::Base
  belongs_to :subscribable, polymorphic: true     # 通过polymorphic关联到video和article
  belongs_to :user                                # 通过user_id来找到user
end

class Video < ActiveRecord::Base
  has_many :subscribes, as: :subscribable
  has_many :users, through: :subscribes
end

class Article < ActiveRecord::Base
  has_many :subscribes, as: :subscribable
  has_many :users, through: :subscribes
end
{% endhighlight %}


#### 备注:
* http://blog.dharanasoft.com/2011/09/06/polymorphic-many-to-many-associations-in-rails/
* http://guides.rubyonrails.org/association_basics.html#polymorphic-associations
* http://www.gotealeaf.com/blog/understanding-polymorphic-associations-in-rails?utm_source=rubyweekly&utm_medium=email
