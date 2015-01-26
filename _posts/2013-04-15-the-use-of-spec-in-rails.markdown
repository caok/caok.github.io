---
layout: post
title: "testing rails app #2"
date: 2013-04-15 21:29
comments: true
categories: [Rails, Rspec, Test]
---

在上一篇简单介绍了下[在rails中安装rspec的方法](http://caok1231.com/blog/2013/04/10/testing-rails-app/)，现在接着介绍下rspec中的一些具体用法。

### 1.基本结构
首先先介绍下spec文件
{% highlight ruby %}
#post_spec.rb
require "spec_helper"  #加载spec的默认设置

describe Post do       #你想测试的类Post
  it "named hello"     #测试案例用“it”来声明，紧跟着“it”的是这个测试案例的名称
end
{% endhighlight %}

它会默认去寻找post.rb，进行相应的测试。

[嵌套结构](https://www.relishapp.com/rspec/rspec-core/v/2-6/docs/example-groups/basic-structure-describe-it)
{% highlight ruby %}
describe Post do
  context "first" do
    it "do something" do
    end
    context "second-level"
      it "do another thing" do
      end
    end
  end

  context "second" do
    it "do something" do
    end
  end
end
{% endhighlight %}

### 2.Matcher
这部分主要定义在[rspec-expectations](https://www.relishapp.com/rspec/rspec-expectations/v/2-13/docs)中, 它用来定义希望的输出结果。这里大致介绍些常用的：

#### [比较](https://www.relishapp.com/rspec/rspec-expectations/v/2-13/docs/built-in-matchers/be-matchers!)
{% highlight ruby %}
post.title.should == "hello"      ==>  should eq("hello")
post.status.should == false       ==>  should be_false
post.status.should == true        ==>  should be_true
posts.count.should > 5
posts.count.should_not == 5
post.title.should =~ /expression/  ==> should match(/expression/)
{% endhighlight %}

若post中定义了一个函数published?返回一个boolean
{% highlight ruby %}
post.published?.should be_true   ==> post.published?.should be
==> post.should be_published     #如果函数定义为published,则be_published报错提示找不到published?

post.published?.should be_false  #published?返回false或者nil时，测试案例通过
post.published?.should be_nil    #published?返回nil时，测试案例通过
{% endhighlight %}

#### [指定对象的类型](https://www.relishapp.com/rspec/rspec-expectations/docs/built-in-matchers/specify-types-of-objects!)
{% highlight ruby %}
recent_post.should be_kind_of(Post)             # recent_post < Post
post.status.should be_an_instance_of(String)    # status的类型为String
{% endhighlight %}

#### [Collection membership](https://www.relishapp.com/rspec/rspec-expectations/docs/built-in-matchers/include-matcher!)
{% highlight ruby %}
post.comments.should include(comment1)
[1,2,3].should include(1, 2)
"this string".should include("is str")
{% endhighlight %}

#### [have(n).items](https://www.relishapp.com/rspec/rspec-expectations/v/2-13/docs/built-in-matchers/have-n-items-matcher!)
{% highlight ruby %}
post.comments.count.should == 2   ==>   post.should have(2).comments
post.should have_at_least(x).comments
post.should have_at_most(x).comments
{% endhighlight %}

#### [change](https://www.relishapp.com/rspec/rspec-expectations/docs/built-in-matchers/expect-change!)
{% highlight ruby %}
post = Post.new(title: "hello")
expect {post.save}.to change {Post.count}.by(1)
==> expect {post.save}.to change {Post.count}.from(0).to(1)
{% endhighlight %}

#### [raise_error](https://www.relishapp.com/rspec/rspec-expectations/v/2-13/docs/built-in-matchers/raise-error-matcher!)
{% highlight ruby %}
post = Post.new
expect {post.save!}.to raise_error(ActiveRecord::RecordInvalid)

expect { raise "oops" }.to raise_error("oops")
expect { Object.new.to_s }.to_not raise_error
{% endhighlight %}

#### [其他](https://www.relishapp.com/rspec/rspec-expectations/v/2-13/docs/built-in-matchers)

#### 参考：
* http://rspec.info
* http://www.codeschool.com/courses/testing-with-rspec
* https://www.relishapp.com/rspec/rspec-rails/v/2-13/docs
* http://rubydoc.info/gems/rspec-rails/frames
* https://www.relishapp.com/rspec/rspec-expectations/docs/built-in-matchers
* http://rubydoc.info/gems/rspec-expectations/frames
