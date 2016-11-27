---
layout: post
title: "testing rails app #3"
date: 2013-04-17 21:01
categories: [Rails, Test]
tags: [Rails, Rspec, Test]
---

上一篇总结了下rspec中[基本结构和Matcher](http://caok1231.com/blog/2013/04/15/the-use-of-spec-in-rails/), 现在接着往下，当然我写的这些更多的像是读书笔记，更加详细和权威的可以仔细研究[rspec官方文档](http://rspec.info/)。

### 1.[subject](https://www.relishapp.com/rspec/rspec-core/v/2-6/docs/subject/explicit-subject)
subject即为某个方法调用(此调用的结果是一个对象，这一点毋庸置疑，因为一切都是对象)绑定一个“名字”(一般用 symbol)，于是在后面的测试样例中，我们可以用这个名字来指代它。最直接的好处就是可以让代码更精炼，提高可读性，减少重复。当然可能有人会疑惑 subject 和 let 的区别，其实这两个是同出一源，具体可见：[http://ruby-china.org/topics/9271](http://ruby-china.org/topics/9271), [nightire](http://ruby-china.org/nightire) 给出了非常清晰的解释。

先看一个不用subject的例子

{% highlight ruby %}
describe Post do
  it 'title is Tom' do
    post = Post.new title: 'Tom'
    post.title.should == 'Tom'
  end

  it 'published?' do
    post = Post.new
    post.published?.should be_true
  end
end
{% endhighlight %}

这段测试代码中，我们能发现重复的部分，可以通过subject来重构下
{% highlight ruby %}
describe Post do
  subject { Post.new title: 'Tom' }
  it 'title is Tom' do
    subject.title.should == 'Tom'
  end

  it 'published?' do
    subject.published?.should be_true
  end
end
{% endhighlight %}

这样看着还是比较...,看看还有没有精简的方法

* 1.上一篇[基本结构和Matcher](http://caok1231.com/blog/2013/04/15/the-use-of-spec-in-rails/)对published？还能写的更简单些，还有印象没？
* 2.subject与let的一个不同之处：可以用来配合 should 进行隐式调用的

{% highlight ruby %}
describe Post do
  subject { Post.new title: 'Tom' }
  it 'title is Tom' do
    subject.title.should == 'Tom'
  end

  it 'published?' do
    should be_published
  end
end
{% endhighlight %}

试着再继续下呢，将 do...end 也直接去了
{% highlight ruby %}
describe Post do
  subject { Post.new title: 'Tom' }

  its(:title) { should == 'Tom' }
  it { should be_published }
end
{% endhighlight %}

这里直接用[its方法](https://www.relishapp.com/rspec/rspec-core/v/2-6/docs/subject/attribute-of-subject)来直接读取subject的属性，是不是更加简洁了，挤挤还是有的，哈哈。

如果项目不是太紧张的话，在实现功能的前提下，不断的去重构自己的代码，还是很有必要的，这更是一个不断思考、不断进步的过程。

### 2.[let](https://www.relishapp.com/rspec/rspec-core/v/2-13/docs/helper-methods/let-and-let!)
在上面的例子中有提到let，那就简单介绍下：

使用 let 来定义一个 [memoized](http://www.iteye.com/topic/810957) 的辅助方法。可以在一个example group（describe或context的block范围）内定义 helper 方法，这个方法可以被当前 example group 以及它的 sub example group 调用，但不可以被 parent example group 调用。 

subject是主题的意思，我们一般用它来处理要测试的主题，对于那些重复多次但又不是测试主题的对象，就应该使用let。
{% highlight ruby %}
describe "Checking Account initialization" do
  let(:starting_balance) { Money.new(50, :USD) }
  subject(:account) { CheckingAccount.new(starting_balance) }

  it "has $50 balance" do
    expect(account).to have_a_balance_of(starting_balance)
  end

  it "has a balance attribute which equals the starting balance" do
    expect(account.balance).to eq(starting_balance)
  end
end

{% endhighlight %}
借用个例子说明下，这里测试主题很明显是 account，starting_balance 重复了很多次，就使用 let。

可能有些人会对 let 和 let! 有疑问，他们之间的区别：let 的 block 里的代码是延迟到该方法第一次被调用时执行，而 let! 的 block 里的代码则是在每个 example 执行之前被隐式的 before 调用。

### 3.[Hooks](https://www.relishapp.com/rspec/rspec-core/v/2-6/docs/hooks/before-and-after-hooks)
对于那些每个测试案例中都有的处理，我们可以使用before或者after去处理。

{% highlight ruby %}
before(:each) # run before each example
before(:all)  # run one time only, before all of the examples in a group

after(:each) # run after each example
after(:all)  # run one time only, after all of the examples in a group
{% endhighlight %}


#### 参考：
* https://www.relishapp.com/rspec/rspec-core/v/2-6/docs
* http://ruby-china.org/topics/9271
* http://blog.firsthand.ca/2010/09/other-rspec-methods-subject-specify-and.html
* http://yuan.iteye.com/blog/1044398
