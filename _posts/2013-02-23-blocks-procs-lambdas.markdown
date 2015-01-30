---
layout: post
title: "blocks procs lambdas"
date: 2013-02-23 21:17
comments: true
categories: [Ruby]
---

Ruby提供了三种不同的定义函数的方法:blocks, procs 和 lambdas -- 他们都是 Closure (闭包)

<!-- more -->

## Blocks(代码块)
block有两种方式{...}和do...end, 接收Block的方法和Block的首定义如do或{ 必须在同一行, 否则会报语法错误.
{% highlight ruby %}
array = [1,2,3,4]
array.map! do |n|
  n * n
end
# => [1, 4, 9, 16]    #这里array计算的同时，也会将计算的结果重新赋值给array

array = [1,2,3,4]
array.map! { |n| n * n }
# => [1, 4, 9, 16]
{% endhighlight %}

block背后最神奇的要数yield，为了计算block它推迟了调用方法的执行。通过在该方法中的任何除yield之外剩余的代码计算出block的结果。yield语句也可以接受参数，然后将这些传入block中并进行计算。

yield不带参数：
{% highlight ruby %}
def time
  start = Time.now
  result = yield
  puts "Completed in #{Time.now - start} seconds."
  result
end
 
time do
  sleep 2
end
# => Completed in 2.001109 seconds.
{% endhighlight %}

yield带参数：
{% highlight ruby %}
def contrived_example(n)
  yield n
end

contrived_example(3) do |x|
  x + 3
end
# => 6
{% endhighlight %}

看下下面两段代码，实现的效果是一样的。
{% highlight ruby %}
class Array
  def iterate!
    self.each_with_index do |n, i|
      self[i] = yield(n)
    end
  end
end
{% endhighlight %}

{% highlight ruby %}
class Array
  def iterate!(&code)
    self.each_with_index do |n, i|
      self[i] = code.call(n)
    end
  end
end
{% endhighlight %}

这两段代码就只有两个不同之处：

* 1、为iterate!传递一个参数&code，&表示这个参数是block(参数名之前加一个“&”,表示将代码块作为闭包传递给函数)。
* 2、在iterate!中没有使用yield而是call。

效果相同，为什么还要这种不同的语法呢？让我们先来看一个到底什么是blocks吧？
{% highlight ruby %}
def what_am_i(&block)
  block.class
end

puts what_am_i {}
# => Proc
{% endhighlight %}

block也是一个Proc！那Proc是什么?

## Procs(过程)
Proc对象是一个绑定了很多本地参数的blocks代码块，一旦绑定后，这些代码你就能在不同的上下文调用，使用这些参数。

Block与Proc惟一的不同是：block是不能保存的Proc，是一性使用的解决方案。
{% highlight ruby %}
squared = Proc.new { |n| n * n }

class Array
  def map!(proc_object)
    self.each_with_index do |value, index|
      self[index] = proc_object.call(value)
    end
  end
end

array = [1,2,3,4]
array.map!(squared)
# => [1, 4, 9, 16]
{% endhighlight %}

传递多个闭包给一个方法
{% highlight ruby %}
def callbacks(procs)
  procs[:starting].call
  puts "Still going"
  procs[:finishing].call
end

callbacks(:starting => Proc.new { puts "Starting" },
          :finishing => Proc.new { puts "Finishing" })
# => Starting
# => Still going
# => Finishing
{% endhighlight %}

什么时候用blocks而不用Procs呢？

* 1、Block:方法把一个对象拆分成很多片段，并且你希望你的用户可以与这些片段做一些交互。
* 2、Block:希望自动运行多个语句，如数据库迁移(database migration)。
* 3、Proc:希望多次复用一段代码。
* 4、Proc:方法有一个或多个回调方法(callbacks)。

## Lambda(表达式)
lambda是定义在Kernel中的函数，因此它的作用域是全局的。使用lambda用来创建闭包的方式和使用Proc非常类似,lambda和proc除了两个关键的不同之处外几乎是一样的。
{% highlight ruby %}
class Array
  def iterate!(code)
    self.each_with_index do |n, i|
      self[i] = code.call(n)
    end
  end
end

array = [1, 2, 3, 4]
array.iterate!(lambda { |n| n ** 2 })
# => [1, 4, 9, 16]
{% endhighlight %}

区别1：lambda检查参数的个数，Proc不会。
{% highlight ruby %}
p = Proc.new { |x, y| puts "x: #{x.class} y: #{y.class}" }
p.call 2
# => x: Fixnum y: NilClass

l = lambda { |x, y| puts "x: #{x.class} y: #{y.class}" }
l.call 2
# => ArgumentError: wrong number of arguments (1 for 2)
{% endhighlight %}

区别2：return不同。lambda的return是返回值给方法，方法会继续执行；Proc的return会终止方法并返回得到的值。
{% highlight ruby %}
def proc_math
  Proc.new { return 1 + 1 }.call
  return 2 + 2
end
def lambda_math
  lambda { return 1 + 1 }.call
  return 2 + 2
end
proc_math   # => 2
lambda_math # => 4
{% endhighlight %}

* proc_math中，执行到Proc.new中的return时，直接返回2，不继续执行。
* lambda_math中，执行到lambda中的return时，返回2，方法继续执行, 最后返回4。

{% highlight ruby %}
def generic_return(code)
  one, two    = 1, 2
  three, four = code.call(one, two)
  return "Give me a #{three} and a #{four}"
end

puts generic_return(lambda { |x, y| return x + 2, y + 2 })
puts generic_return(Proc.new { |x, y| return x + 2, y + 2 })
puts generic_return(Proc.new { |x, y| x + 2; y + 2 })
puts generic_return(Proc.new { |x, y| [x + 2, y + 2] })

# => Give me a 3 and a 4
# => *.rb:8: unexpected return (LocalJumpError)
# => Give me a 4 and a 
# => Give me a 3 and a 4
{% endhighlight %}

Procs在Ruby里是代码片段，不是方法。因为这个，Proc return的就是proc_return这个方法的return，因为Proc那段就是那个方法里的代码片段。而lambdas的行为像一个方法，它检查参数的个数，而且不会覆盖调用方法的return。由于这个原因，你最好把lambdas理解为写一个方法(method)的另类方式，只不过是匿名的而已。

## 总结
* Blocks是单独使用的(代码片段)
* Procs是作为对象存在的(代码片段)
* Lambdas有严格的参数检查(方法)

#### 参考：
* http://www.robertsosinski.com/2008/12/21/understanding-ruby-blocks-procs-and-lambdas/
* http://blog.csdn.net/aabbcc456aa/article/details/8486923
* http://net.tutsplus.com/tutorials/ruby/ruby-on-rails-study-guide-blocks-procs-and-lambdas/
* http://www.oschina.net/translate/know-your-closures-blocks-procs-and-lambdas
* http://blackanger.blog.51cto.com/140924/123034
* http://blog.163.com/digoal@126/blog/static/1638770402012221105853250/
