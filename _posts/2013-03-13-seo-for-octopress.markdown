---
layout: post
title: "Seo for Octopress"
date: 2013-03-13 18:29
categories: [Web]
tags: [Octopress]
---

使用Octopress已经一段时间了，整了个域名，就想着如何才能给自己的博客提升下点击量做点[SEO](http://baike.baidu.com/view/1047.htm).

Octopress默认为每个页面添加meta description，其内容为当前文章的前150个字符，如果是首页就会是第一篇文章的前150个字符。

### 为每篇文章增加keywors和description
{% highlight ruby %}
---
layout: post
title: "SEO for Octopress"
date: 2013-03-13 18:29
comments: true
categories: [seo,octopress]
keywords: seo,octopress
description: How to optimize Octopress for SEO
---
{% endhighlight %}

这样这篇文章keywords和description也会调用这里的设置，生成的结果为：
{% highlight html %}
<title>SEO for Octopress</title>
<meta name="author" content="Caok">
<meta name="description" content="How to optimize Octopress for SEO">
<meta name="keywords" content="seo,octopress">
{% endhighlight %}

### 为主页增加meta信息
上述方法只会为每一篇文章创建keywords和description，但它不会对主页起作用，为了达到这个目的可以这么修改。

这个可以通过修改_includes/head.html来完成。替换以下相关的内容：
{% highlight html %}
<meta name="author" content="{{ site.author }}">
{% capture description %}{% if page.description %}{{ page.description }}{% elsif site.description %}{{ site.description }}{%else%}{{ content | raw_content }}{% endif %}{% endcapture %}
<meta name="description" content="{{ description | strip_html | condense_spaces | truncate:150 }}">
{% if page.keywords %}<meta name="keywords" content="{{ page.keywords }}">{%else%}<meta name="keywords" content="{{ site.keywords }}">{% endif %}
{% endhighlight %}

在_config.yml中增加keywords和description
{% highlight ruby %}
# _config.yml
description: Caok's personal website published personal blog articles and study notes.
keywords: Ubuntu, Ruby, Rails, web, octpress, git, tricks, shell, javascript, Programmer
{% endhighlight %}

### 站外SEO优化
1.将网站提交到各大搜索网站，[网站搜索引擎登录口大全](http://tool.lusongsong.com/addurl.html)
2.在相应的论坛等发帖，链接到自己的网站
3.通过[站长工具](http://tool.chinaz.com)和[gtmetrix](http://gtmetrix.com)分析自己的网站，找到不足之处。

#### 参考：
* http://www.yatishmehta.in/seo-for-octopress
* http://codemacro.com/2012/09/06/octopress-seo
* http://www.oschina.net/translate/fantastic-front-end-performance
* http://www.admin5.com/article/20100917/272215.shtml
