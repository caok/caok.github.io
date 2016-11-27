---
layout: post
title: "jquery parent() parents() closest()区别"
date: 2014-10-04 23:49
categories: [Web]
tags: [Web, Jquery]
---

#### parent是找当前元素的第一个父节点，不管匹不匹配都不继续往下找

#### parents是找当前元素的所有父节点 

#### closest是找当前元素的所有父节点 ，直到找到第一个匹配的父节点

{% highlight html %}
<ul id="menu" style="width:100px;">   
  <li>   
    <ul>   
      <li>
        <a href="#">Home</a>
      </li>  
    </ul>   
  </li>  
  <li>End</li>  
</ul>    
  
<script type="text/javascript">  
  //点击Home时   
  $("#menu a").click(function() {  
    $(this).parent("ul").css("background", "yellow"); //0   
    $(this).parent("li").parent("ul").css("background", "yellow"); //1  
    $(this).parents("ul").css("background", "yellow"); //2   
    $(this).closest("ul").css("background", "yellow"); //3
    return false;   
  });   
</script> 
{% endhighlight %}

{% highlight html %}
1.parent()方法从**指定类型的直接父节点**开始查找，在"0"中，<a>的直接父节点是<li>所以在这里找不到<ul>父节点。在"2"中先找到了<li>，接着找到<ul>，并将它的背景色设置为yellow。**parent()返回一个节点**。

它和parents()不同的是，它只向上搜索一层，而parents()会搜索整个DOM树.

2.parents()方法查找方式同parent()方法类似，不同的一点在于，**当它找到第一的父节点时并没有停止查找，而是继续查找，最后返回多个父节点**，如在"2"中，使得id为menu的ul整个背景色变成了yellow。

3.closest()方法查找时从包含自身的节点找起，它同parents()很类似，不同点就在于它只返回一个节点如在"3"中，实现的功能同1相同。但它使得代码量减小，同"2"相比又**只返回单一的一个节点**。

它和parents()的区别：

* closest()从自身开始向上遍历，直到找到一个适合的节点，返回的jQuery对象包含0个或者1个对象；
* parents()从自身的父节点开始向上遍历，返回所有祖先节点，并根据选择器对这些节点进行筛选，最终返回的jQuery对象可能包含0、1或者多个对象。
{% endhighlight %}

### 寻找子节点和兄弟节点

* children(expr)        //查找所有子元素，只会找到直接的孩子节点，不会返回所有子孙
* contents()            //查找下面的所有内容，包括节点和文本。
* prev()                //查找上一个兄弟节点，不是所有的兄弟节点
* prevAll()             //查找所有之前的兄弟节点
* next()                //查找下一个兄弟节点，不是所有的兄弟节点
* nextAll()             //查找所有之后的兄弟节点
* siblings()            //查找兄弟节点，不分前后
* find()                //会在div元素内 寻找 class为classname的元素。(子元素找)
* filter()              //则是筛选div的class为classname的元素。(平级找)

### 参考:
* http://senir.blog.163.com/blog/static/104118183201317114914406
* http://www.cnblogs.com/ini_always/archive/2011/11/09/2243671.html
* http://blog.csdn.net/lidiansheng/article/details/8634700
* http://www.cnblogs.com/lianzi/archive/2011/07/31/2122828.html
