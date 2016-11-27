---
layout: post
title: "current_user for model"
date: 2015-06-09 17:00
categories: [Rails]
tags: [Rails]
---

{% highlight ruby %}
  class User
    class << self
      def current_user=(user)
        Thread.current[:current_user] = user
      end
  
      def current_user
        Thread.current[:current_user]
      end
    end
  end
  
  class ApplicationController
    before_filter :set_current_user
  
    def set_current_user
      User.current_user = current_user
    end
  end
{% endhighlight %}

调用的时候只需要User.current_user

### 参考
* https://stackoverflow.com/questions/3742785/rails-3-devise-current-user-is-not-accessible-in-a-model
