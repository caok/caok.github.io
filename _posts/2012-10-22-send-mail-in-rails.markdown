---
layout: post
title: "send mail in rails"
date: 2012-10-22 17:49
comments: true
categories: [Rails, Web, Mail]
---

#### 1.Create A Notice Mailer
> rails g mailer NoticeMailer decomp_change

    create   app/mailers/notice_mailer.rb
    invoke   erb
    create   app/views/notice_mailer
    create   app/views/notice_mailer/decomp_change.text.erb

<!-- more -->

```
# app/mailers/notice_mailer.rb
class NoticeMailer < ActionMailer::Base
  default from: "from@example.com"

  def decomp_change
    @greeting = "Hi"
    mail to: "to@example.org"
  end
end
```

#### 2.Sending Attachments in Mail
```
#app/mailers/notice_mailer.rb
class NoticeMailer < ActionMailer::Base
  default from: "from@example.com"

  def decomp_change (notice)
    @notice = notice
    @last_tweet = @notice.tweets.last
    attachments['z.pdf'] = File.read("#{Rails.root}/public/notice.pdf")
    mail to: @notice.email, subject: 'Your decomp stage has changed'
  end
end
```

#### 3.MAILER VIEWS
```
#app/views/notice_mailer/decomp_change.text.erb
Greetings <%= @notice.name %>,

  Your decomposition state is now <%= @notice.decomp %> and your
last tweet was: <%= @last_tweet.body %>

Good luck!
```

```
#app/views/notice_mailer/decomp_change.html.erb
<h1>Greetings <%= @notice.name %>,</h1>

<p>Your decomposition state is now <%= @notice.decomp %> and your
last tweet was: <%= @last_tweet.body %></p>

<%= link_to "View yourself", notice_url(@notice) %>
```

#### 4.SENDING MAIL
```
#app/models/notice.rb
class Notice < ActiveRecord::Base
  after_save :decomp_change_notification, if: :decomp_changed?

  private
  def decomp_change_notification
    NoticeMailer.decomp_change(self).deliver
  end
end
```

#### 5.CONFIG
```
#config/environments/development.rb
config.action_mailer.delivery_method = :smtp
  config.action_mailer.smtp_settings = {
    :address              => "smtp.gmail.com",
    :port                 => 587,
    :user_name            => 'poshboytl',
    :password             =>  ENV['GMAIL_PASS'],
    :authentication       => 'plain',
    :enable_starttls_auto => true
  }
```

#### references：
http://railscasts-china.com/episodes/how-to-send-emails-in-rails
