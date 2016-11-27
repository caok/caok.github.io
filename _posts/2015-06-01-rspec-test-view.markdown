---
layout: post
title: "Rspec test for view"
date: 2015-06-01 16:30
categories: [Rails, Test]
tags: [Rails, Test]
---

在rspec中可以对view进行简单的测试,我们这简单介绍下

{% highlight ruby %}
#spec/views/widgets/index.html.erb_spec.rb
require "rails_helper"

RSpec.describe "widgets/index" do
  it "displays all the widgets" do
    assign(:widgets, [
      Widget.create!(:name => "slicer"),
      Widget.create!(:name => "dicer")
    ])

    render

    expect(rendered).to match /slicer/
    expect(rendered).to match /dicer/
  end
end
{% endhighlight %}

这里通过assign分配给页面需要的变量, 然后通过render渲染出页面来

再来一个例子，这里我们把nest resource给考虑进去
{% highlight ruby %}
# posts/:post_id/comments
it "should show the post's comment new page" do
  post = Post.create()
  assign(:post, post)
  controller.request.path_parameters[:post_id] = post.id
  stub_template "shared/_footer.html.erb" => ""
  render :template => "posts/comments/new.html.erb"
  expect(rendered).to match /new comment/ 
end
{% endhighlight %}

这里我们测试的页面时嵌套在post里面的，所以首先我们通过“controller.request.path_parameters[:post_id]”来告诉它“:post_id”, 否则它会不知道post_id是多少

再然后这里还用到了一个stub_template, 相当于奖"shared/_footer.html.erb"给mock掉，在这个partial有其他变量，但我们又不想对其测试的情况下，我们就可以使用stub_template



#### 参考
* http://www.relishapp.com/rspec/rspec-rails/v/3-2/docs/view-specs/view-spec
