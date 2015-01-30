---
layout: post
title: "exception notifier"
date: 2013-02-16 18:59
comments: true
categories: [Rails]
---

在web应用中很多时候会遇到系统例外的异常，第一时间将出错信息发送给维护者/开发者是很重要的，所以这里记录下如何处理这一问题。
{% highlight ruby %}
# Gemfile
gem 'exception_notification'               #Exception Notifier Plugin for Rails
gem 'letter_opener', group: :development   #Preview mail in the browser instead of sending
{% endhighlight %}

这里预览mail的话可以使用[letter_opener](https://github.com/ryanb/letter_opener)或者[mails_viewer](https://github.com/tomlion/mails_viewer)

可以在config.ru文件中配置exception_notification的选项，或者在你想要启动的环境配置文件中(production.rb 或 development.rb)
{% highlight ruby %}
  config.middleware.use ExceptionNotifier,
    :sender_address => %{"notifier" <notifier@example.com>},
    :exception_recipients => %w{exceptions@example.com},
    :ignore_exceptions => ExceptionNotifier.default_ignore_exceptions

  config.action_mailer.delivery_method = :letter_opener
{% endhighlight %}

这里的话只会在浏览器中显示邮件，而没有真正发送到邮箱中。
如果要发送到邮箱的话，可以这么配置：
{% highlight ruby %}
  config.middleware.use ExceptionNotifier,
    :sender_address => 'from@163.com',
    :exception_recipients => %w{to@163.com},
    :ignore_exceptions => ExceptionNotifier.default_ignore_exceptions 

  config.action_mailer.delivery_method = :smtp

  config.action_mailer.smtp_settings = {
    :address              => 'smtp.163.com',
    :port                 => 25,
    :domain               => '163.com',
    :user_name            => 'from@163.com',
    :password             => 'password',
    :authentication       => 'login',
    :enable_starttls_auto => true
  }
{% endhighlight %}

#### 参考：
* https://github.com/smartinez87/exception_notification
* http://railscasts.com/episodes/104-exception-notifications-revised
* https://github.com/railscasts/104-exception-notifications-revised
* https://github.com/ryanb/letter_opener
* https://github.com/tomlion/mails_viewer
