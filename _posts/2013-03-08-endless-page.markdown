---
layout: post
title: "endless page"
date: 2013-03-08 20:53
categories: [Rails]
tags: [Rails]
---

经常会看到[瀑布流](http://baike.baidu.com/view/7151782.htm)，随着瀑布流效果页面滚动条向下滚动，这种布局还会不断加载数据块并附加至当前尾部。

参差不齐的多栏布局，可以使用[masonry-rails](https://github.com/kristianmandrup/masonry-rails)，也可以试试[Grid-A-Licious](https://github.com/suprb/Grid-A-Licious)。不过我这里主要总结下endless page的效果。

{% highlight ruby %}
# Gemfile
gem 'kaminari'
{% endhighlight %}

{% highlight ruby %}
# posts_controller.rb
def index
  @posts = Post.page(params[:page])
 
  respond_to do |format|
    format.html
    format.json { render json: @posts }
    format.js
  end
end
{% endhighlight %}

{% highlight ruby %}
# index.html.erb
<table>
  <tr>
    <th></th>
  </tr>

  <%= render @posts %>
</table>

<div class="paginate">
  <%= paginate @posts, :remote => true %>
</div>
{% endhighlight %}

{% highlight ruby %}
# index.js.erb
$("<%= j(render @posts) %>").appendTo($("table"));
$(".paginate").html("<%= j(paginate @posts, :remote => true) %>");
checkScroll();
{% endhighlight %}

{% highlight ruby %}
# posts.js
function checkScroll() {
  if (nearBottomOfPage()) {
    if($(".next a").length != 0){
      $.rails.handleRemote($(".next a"));
    }
  } else {
    setTimeout("checkScroll()", 250);
  }
}

function nearBottomOfPage() {
  return scrollDistanceFromBottom() < 150;
}

function scrollDistanceFromBottom(argument) {
  return pageHeight() - (window.pageYOffset + self.innerHeight);
}

function pageHeight() {
  return Math.max(document.body.scrollHeight, document.body.offsetHeight);
}

$(document).ready(function(){
  checkScroll();
});
{% endhighlight %}

上述的需要在点击“下一页”的时候才会去加载，在[114-endless-page-revised](http://railscasts.com/episodes/114-endless-page-revised)中做了些改变，起到滚动鼠标自动加载的效果.

其实就是通过getScript去发送加载下一页的请求
{% highlight ruby %}
# posts.js
jQuery ->
  if $('.pagination').length
    $(window).scroll ->
      url = $('.pagination .next a').attr('href')
      if url && $(window).scrollTop() > $(document).height() - $(window).height() - 50
        $('.pagination').text("Fetching more posts...")
        $.getScript(url)
    $(window).scroll()
{% endhighlight %}

{% highlight ruby %}
# index.js.erb
$('#posts').append('<%= j render(@posts) %>');
<% if (@posts.next_page %>
  $('.pagination').replaceWith('<%= j paginate(@posts) %>');
<% else %>
  $('.pagination').remove();
<% end %>
<% sleep 1 %> 
{% endhighlight %}


#### 参考：
* http://railscasts.com/episodes/114-endless-page
* http://railscasts.com/episodes/114-endless-page-revised
* http://railscasts.com/episodes/174-pagination-with-ajax
* https://github.com/railscasts/114-endless-page-revised
* http://www.slideshare.net/wildjcrt/gem-partial-ajax
* https://github.com/wildjcrt/endless-page-example
* http://endless-page-example.herokuapp.com/
* https://github.com/fschwahn/endless_page
* https://github.com/pedromtavares/endless_scroll_example
* https://github.com/amatsuda/kaminari/wiki/How-To:-Create-Infinite-Scrolling-with-jQuery
