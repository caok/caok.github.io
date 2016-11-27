---
layout: post
title: "form objects"
date: 2013-08-14 22:41
categories: [Rails]
tags: [Rails]
---

### Form Object
* 1.One form, multiple models
* 2.Used in create and update flows
* 3.Quack like Active Records

### Implications
* 1.Layer aggregation onto individual objects
* 2.Limit responsibility of ActiveRecords
* 3.Contextual validations

### When to use
* 1.Aggregating models into a form
* 2.Instead of accepts_nested_attributes_for

当多个 ActiveRecord models 有可能在更新时使用同一个form提交时，可以考虑使用一个Form对象来对其进行封装。这个要比使用accepts_nested_attributes_for更加干净清晰些。

比如“用户注册”过程，可以把注册过程封装成一个单一的可重用的类;所有用户注册的代码就不再不需要属于user model中，这样对user model的测试影响也会更小。

实际上form object就是为了达到Single Responsibility Principle

### Single Responsibility Principle
* There should never be more than one reason for a class to change.
* Each responsibility is an axis of change.

### [416-form-object](http://railscasts.com/episodes/416-form-objects)中的疑问
#### 1.persisted? vs new_record? any relation?
{% highlight ruby %}
def persisted?
  !(new_record? || destoryed?)
end
{% endhighlight %}

persisted?只有在新建或是删除时才为false

#### 2.Why we don't need changing_password as attributes?
原先要确保创建和编辑用户的时候不受original_password, new_password和new_password_confirmation的字段限制，所以加入了一个changing_password的字段来区分是否是修改密码的过程。使用form_object之后，这三个字段也同时被独立出来了，不再会去影响user的创建和更新。

#### 参考:
* http://railscasts.com/episodes/416-form-objects
* http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models
* https://github.com/codeclimate/refactoring-fat-models
* http://panthersoftware.com/blog/2013/05/13/user-registration-using-form-objects-in-rails
* https://www.youtube.com/watch?feature=player_embedded&v=IqajIYxbPOI
* https://github.com/hookercookerman/form_model
* https://github.com/apotonick/reform
