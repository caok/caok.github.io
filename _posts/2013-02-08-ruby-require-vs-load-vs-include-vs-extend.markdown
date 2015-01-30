---
layout: post
title: "Ruby require VS load VS include VS extend"
date: 2013-02-08 22:27
comments: true
categories: [Ruby]
---

## load
**load用来多次加载一个库，你必须指定扩展名。**

load的使用方法几乎和require一样，但它不会跟踪是否已经加载该库。当你使用一个load方法时，你必须制定“.rb”（扩展库文件名）来加载库，当然它可以多次加载一个库。

很多时候，当你想要用require而不是load，但如果你想每次库调用库都被重新加载那么就使用load。比如，如果你的module经常改变，你或许想使用load去将这些改变加载到class中。
{% highlight ruby %}
load 'test_library.rb'
{% endhighlight %}

如果模块被定义在一个单独的.rb文件，那么你可以这么使用：
{% highlight ruby %}
# log.rb
module Log 
  def class_type
    "This class is of type: #{self.class}"
  end
end
{% endhighlight %}

{% highlight ruby %}
# test.rb
load 'log.rb'
 
class TestClass 
  include Log 
  # ... 
end
{% endhighlight %}

## require
**require方法让你加载一个库，并防止它被多次加载。**如果您尝试多次加载同一个库，require方法将返回“false”。只有当你要加载的库位于一个分离的文件中时才有必要使用require。使用时不需要加扩展名，一般放在文件的最前面.

它可以跟踪该库是否已经加载与否。你也不必指定“.rb”扩展名的库文件名。
{% highlight ruby %}
require 'test_library'
{% endhighlight %}

## include
当你将module加载(include)进class中，正如如下代码所示，你就可以使用定义在module中的函数并且将其插入到该class中。它也允许‘mixin’行为。这是用来DRY你的代码，以避免重复，例如，如果有多个类，只需要将同样的代码放在模块内。

下面的代码是将设module Log和class TestClass定义在同一个.rb文件中。如果他们分别在独自的单独文件中的话，就需要使用‘load’和‘require’来让class你已经定义的module。

当你的库加载之后，你可以在你的类定义中包含一个module，让module的实例方法和变量成为类本身的实例方法和类变量，它们mix进来了。根据锄头书，include并不会把module的实例方法拷贝到类中，只是做了引用，包含module的不同类都指向了同一个对象。如果你改变了module的定义，即使你的程序还在运行，所有包含module的类都会改变行为。

{% highlight ruby %}
module Log 
  def class_type
    "This class is of type: #{self.class}"
  end
end

class TestClass
  include Log
  # ...
end

tc = TestClass.new.class_type
#The above will print “This class is of type: TestClass”
{% endhighlight %}

{% highlight ruby %}
module MyModule
  def module_method
    puts "module_method"
  end
end

class MyClass
  include MyModule
end

#实际上就等同于
class MyClass
  def module_method
    puts "module_method"
  end
end
{% endhighlight %}

## extend
当使用method方法而不是include的时，你可以将method方法作为一个**类方法**而不是实例方法。
{% highlight ruby %}
module Log 
  def class_type
    "This class is of type: #{self.class}"
  end
end
 
class TestClass 
  extend Log 
  # ... 
end
 
tc = TestClass.class_type
#The above will print “This class is of type: TestClass”
{% endhighlight %}

当在类中使用extend而不是include，如果你尝试实例化TestClass，并在该类中调用class_type方法时，像你在上面的include的例子中一样，你会得到一个NoMethodError。该模块的方法成为类方法。

{% highlight ruby %}
module MyModule
  def module_method
    puts "module_method"
  end
end

class MyClass
  extend MyModule
end

#实际上就等同于
class MyClass
  def self.module_method
    puts "module_method"
  end
end
{% endhighlight %}

参考：[原文](http://ionrails.com/2009/09/19/ruby_require-vs-load-vs-include-vs-extend/)
