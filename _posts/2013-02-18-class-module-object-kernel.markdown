---
layout: post
title: "Class Module Object Kernel"
date: 2013-02-18 20:26
comments: true
categories: [Ruby]
---

{% highlight ruby %}
Class.class       ==> Class
Module.class      ==> Class
Object.class      ==> Class
Kernel.class      ==> Module
BasicObject.class ==> Class
{% endhighlight %}

Class\Module\Object\BasicObject都是Class,只有Kernel是Module

{% highlight ruby %}
Class.superclass        ==> Module
Module.superclass       ==> Object
Object.superclass       ==> BasicObject
Kernel.superclass       ==> NoMethodError: undefined method `superclass' for Kernel:Module`
BasicObject.superclass  ==> nil
{% endhighlight %}

Class类继承了Module类，Module类继承了Object类，Object类继承了BasicObject类，BasicObject类是所有类的祖先.

Kernel为Module，理解为系统预定义的一些方法，我们可以在所有的对象上使用，使用时不需要使用类型作为前缀，当然我们也可以加上Kernel

{% highlight ruby %}
Class.ancestors         ==> [Class, Module, Object, Kernel, BasicObject]
Module.ancestors        ==> [Module, Object, Kernel, BasicObject]
Object.ancestors        ==> [Object, Kernel, BasicObject]
Kernel.ancestors        ==> [Kernel]
BasicObject.ancestors   ==> [BasicObject]
{% endhighlight %}

BasicObject类是所有类的父类,而Object混入了Kernel这个模块，又由于Object是Ruby中所有类的父类，这样以来，Kernel中内建的核心函数就可以被Ruby中除了BasicObject的所有的类和对象访问。

Object的实例方法由Kernel模块定义。

Kernel模块中定义了private method和public method，所以对于一个普通的对象，可以直接调用Kernel的public method，可以通过通过一下方式查看Kernel.methods, Kernel.public_methods, Kernel.private_methods.

区别：

* 1.模块不能实例化，类不能include
* 2.如果模块和类不在用一个文件中，如果要使用include，先使用require把文件引入(include和require的区别参见[Ruby Require vs Load vs Include vs Extend](http://caok1231.com/blog/2013/02/08/ruby-require-vs-load-vs-include-vs-extend/))
* 3.include不是简单的将模块的实例变量和方法拷贝到类中，而是建立一个由类到所包含模块的引用
* 4.如果有多个include，将依次生成代理类，最后一个include的将是该类的直接超类，依次向上衍生
* 5.含有include的模块或者类定义，可以访问它所包含的常量，类变量和实例方法。如果一个模块被包含，改模块的常量，类变量，实例方法都被绑定到该类的一个匿名超类中，类的对象会响应发送给模块实例方法的消息
* 6.模块里可以定义一个initialize方法，当创建包括模块的类的对象时，满足一下条件之一，则模块的该方法将被调用：a、类没有定义他自己的initialize方法，b、类的initialize方法中调用了super

#### 参考：
* http://blog.163.com/rettar@126/blog/static/121650342201141711395591/
* http://book.douban.com/annotation/16286998/ 
