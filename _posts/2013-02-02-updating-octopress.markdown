---
layout: post
title: "updating octopress"
date: 2013-02-02 19:38
comments: true
categories: [Octopress]
---

最近更新了下octopress，更新过程中由于不注意遇到了一些问题，特此将octopress的更新过程大致介绍下。

必须在source分支下进行，不要切换到master分支;每次push时也只要push到origin source即可。

若你的octopress的博客代码是克隆到本地的，可能remote中没有octopress(创建或是更新时会用到)，所以更新前检查下。

输入：

    git remote

此时你应该能看到octopress和origin两个，如果没有octopress，那么就要添加一下
{% highlight bash %}
git remote add octopress git://github.com/imathis/octopress.git
{% endhighlight %}

更新过程
{% highlight ruby %}
git pull octopress master     # 获取最新的octopress
bundle install                # 更新gems
rake update_source            # 更新模板代码
rake update_style             # 更新模板样式
{% endhighlight %}

更新过后会生成sass.old和source.old存放以前的记录。而这两个文件夹会被git所忽略，可以查看.gitignore文件。
不放心的话可以切换到master分支看下，master分支中并没有更新，不过别忘了切换回source分支。

#### 参考：
http://octopress.org/docs/updating/
