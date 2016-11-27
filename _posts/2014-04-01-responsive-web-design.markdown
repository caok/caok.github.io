---
layout: post
title: "Responsive Web Design"
date: 2014-04-01 18:06
categories: [Web]
tags: [Web]
---

眼下移动设备越来越多，在做网页设计的时候需要考虑的也随着增加。如何能在不同分辨率大小的设备上将同一个页面较好的呈现出来，成为很多web程序员不得不面对的问题。针对这个问题，人们开始提出[Responsive Web Design](http://alistapart.com/article/responsive-web-design),即让同一张网页自动适应不同大小的屏幕，根据屏幕宽度，自动调整布局。

可以在[mediaqueri.es](http://mediaqueri.es/)查看到这类的例子。


这里介绍下在做自适应网页的时候需要注意的几个点:

### 1.viewport标签
在网页代码的头部加入viewport标签,增加viewport meta标签告诉浏览器视口宽度等于设备屏幕宽度，且不进行初始缩放。
{% highlight html %}
<meta name="viewport" content="width=device-width, initial-scale=1" />
{% endhighlight %}
这行代码的意思是，网页宽度默认等于屏幕宽度（width=device-width），原始缩放比例（initial-scale=1）为1.0，即网页初始大小占屏幕面积的100%。主流的浏览器都支持这个设置，但ie6,7,8的话就要例外了。。

但如果把 user-scalable=no 或者 maximum-scale=1 结合 initial-scale=1一起使用,这会禁用站点的缩放的功能。

### 2.不使用px
在设定width时不要使用px来定义，这会使它的宽度是绝对的。这将不利于"自适应"。所以我们在定义宽度时要尽量使用百分比，或者auto;
{% highlight css %}
width: 50%;
width: auto;
{% endhighlight %}
当然字的大小也是如此，使用em来取代px。
{% highlight css %}
body {
  font: normal 100% Helvetica, Arial, sans-serif;
}

h1 {
  font-size: 1.5em; 
}
{% endhighlight %}
这里面h1的字体大小就是默认body字体的1.5倍

图片的处理也是如此:

img标签的话，只需要设置 max-width: 100%;或width:100%; 语句为：img { max-width: 98%; }

css加载的background-image如何自适应大小呢，其实CSS3中是可以实现的，添加如下语句：background-size:100% 100%;

### 3.使用box-sizing: border-box
在你的css文件的顶部增加:
{% highlight css %}
*, *:before, *:after {
  -moz-box-sizing: border-box;
  -webkit-box-sizing: border-box;
  box-sizing: border-box;
}
{% endhighlight %}
区别在哪里呢？

没使用box-sizing: border-box
{% highlight css %}
margin, borders 和 padding 都在content的外部
{% endhighlight %}
使用box-sizing: border-box
{% highlight css %}
borders 和 padding 在content的内部，只有margin在content的外部
{% endhighlight %}
这个主要也是用来解决ie这个特例的...

### 4.使用流动布局
让每个区块的位置都是浮动的，而不是固定不变的。这个时候绝对定位（position: absolute）的使用，就要非常小心。

float的好处是，如果宽度太小，放不下两个元素，后面的元素会自动滚动到前面元素的下方，不会在水平方向overflow（溢出），避免了水平滚动条的出现。

### 5.选择加载css
自动探测屏幕宽度，然后加载相应的CSS文件。
{% highlight html %}
<link rel="stylesheet" type="text/css"
    media="screen and (max-device-width: 400px)"
　　href="tinyScreen.css" />
{% endhighlight %}
如果屏幕宽度小于400像素（max-device-width: 400px），就加载tinyScreen.css文件。
{% highlight html %}
<link rel="stylesheet" type="text/css"
　　media="screen and (min-width: 400px) and (max-device-width: 600px)"
　　href="smallScreen.css" />
{% endhighlight %}
如果屏幕宽度在400像素到600像素之间，则加载smallScreen.css文件。


### 参考:
* http://www.ruanyifeng.com/blog/2012/05/responsive_web_design.html
* http://www.adamkaplan.me/grid
* http://www.cnblogs.com/softlover/archive/2012/11/20/2779900.html
* http://www.xyhtml5.com/html5-viewport-tags.html
