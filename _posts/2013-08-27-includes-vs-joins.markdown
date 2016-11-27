---
layout: post
title: "includes vs joins"
date: 2013-08-27 07:03
categories: [Rails]
tags: [Rails]
---

### includes
{% highlight ruby %}
posts = Post.includes(:category)
posts.each do |post|
  post.category.name
end
{% endhighlight %}

通过使用 includes, 我们就可以直接访问category中的字段，而不再需要额外的查询。

includes会把连接在一起的表的所有字段都查询出来，存放在内存中。当你想取连接表的数据时，不需要再次查询数据库。

通过这种方式能有一定的性能提升，它还解决了 [N+1 queries problem](http://guides.rubyonrails.org/active_record_querying.html#eager-loading-associations)

### joins
{% highlight ruby %}
Post.joins(:comments)
{% endhighlight %}

形成的sql语句:
{% highlight ruby %}
select posts.* from posts inner join comments on comments.post_id = posts.id
{% endhighlight %}

joins只会把主表的数据查询出来放在内存中，若需要查询连接表的数据时，还需再次查询数据库。

若只想查询连接表的部分字段时，建议使用joins + select查询，比如：
{% highlight ruby %}
Post.joins(:comments).select("posts.*, comments.body as comment_body")
{% endhighlight %}

#### 参考:
* http://railscasts.com/episodes/22-eager-loading-revised
* http://railscasts.com/episodes/181-include-vs-joins
* http://guides.rubyonrails.org/active_record_querying.html#joining-tables
