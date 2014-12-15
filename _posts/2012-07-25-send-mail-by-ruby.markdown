---
layout: post
title: "send mail by ruby"
date: 2012-07-25 21:30
comments: true
categories: [Ruby, Mail]
---

利用ruby查询当前的外网的ip地址并将其发送到指定的邮箱，同时增加了发送附件的例子。
<!-- more -->

```
#mail_backup.rb
#!/usr/bin/env ruby
#
# ARGV[0] - msg
# ARGV[1] - mailto
# ARGV[2] - filename

require 'open-uri'
require 'rubygems'
require 'action_mailer'

ActionMailer::Base.smtp_settings = {
  :address              => 'smtp.163.com',
  :port                 => 25,
  :domain               => '163.com',
  :user_name            => 'example@163.com',
  :password             => 'password',
  :authentication       => 'login',
  :enable_starttls_auto => true
}

class Notifer < ActionMailer::Base
  default :from => 'example@163.com'

  def mailip(ip, msg, mailto, file)
    attachments["#{file}"] = File.read("#{file}")  #发送附件处理
    mail :to => mailto, :subject => "#{msg} ip is #{ip}" do |format|
      format.text { render :text => ip}
    end.deliver
  end
end

open("http://checkip.dyn.com") do |f|
  ip = f.read.slice /[0-9]+(\.[0-9]+){3}/
  filename = "mysql-" + Time.now.strftime("%Y-%m-%d").to_s + ".sql.gz"   #定义需要发送的附件
  Notifer.mailip ip, ARGV[0], ARGV[1] || 'caok1231@gmail.com', ARGV[2] || filename
end
```

使用:
ruby mail_backup.rb topic example@gmail.com filename

说明：
发送的附件默认为当前文件夹下的文件，也可自己通过最后一个参数指定要发送的文件
