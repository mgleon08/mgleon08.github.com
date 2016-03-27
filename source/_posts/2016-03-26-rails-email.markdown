---
layout: post
title: "Email 寄信通知"
date: 2016-03-26 09:35:21 +0800
comments: true
categories: rails
---

網站中經常會使用寄信來通知用戶，像是註冊信件，確認更改密碼等等之類的。

<!--more-->

#指令
```ruby
rails g mailer TestMailer notify_job_apply
```

#產生檔案
```ruby
# app/mailers/application_mailer.rb
class ApplicationMailer < ActionMailer::Base
  default from: "from@example.com"
  layout 'mailer'
 	
 #寄信給多個收件者
 #default to: Proc.new { Admin.pluck(:email) }, from: 'notification@example.com'
end

#app/mailer/test_mailer.rb
class TestMailer < ApplicationMailer
  # Subject can be set in your I18n file at config/locales/en.yml
  # with the following lookup:
  #
  #   en.test_mailer.notify_job_apply.subject
  #
  def notify_job_apply
    @greeting = "Hi"
	
	#附件檔案
	attachments['filename.jpg'] = File.read('/path/to/filename.jpg')
    mail to: "to@example.org"
    mail to: ["email"], bcc:["sub-email"], subject: "Title"
    
  end
end
```

#信件內容

因為不是每個人都可以顯示 html 檔案，因此要有兩份

```ruby
#app/views/test_mailer/notify_job_apply.html.erb
<h1>TestMailer#notify_job_apply</h1>
<p>
  <%= @greeting %>, find me in app/views/test_mailer/notify_job_apply.html.erb
</p>
<%= image_tag attachments['image.jpg'].url, alt: 'My Photo', class: 'photos' %>
```

純文字檔

```ruby
#app/views/test_mailer/notify_job_apply.text.erb 
TestMailer#notify_job_apply
<%= @greeting %>, find me in app/views/test_mailer/notify_job_apply.text.erb
```

```ruby
#app/model/user.rb
class User < ActiveRecord::Base
  after_save : notify_job_apply_notification, if: : notify_job_apply?
    private    def notify_job_apply_notification
      UserMailer.notify_job_apply(self).deliver 
      #deliver_now! 立即送出
      #deliver_later! 非同步去處理 ex: sidekiq
    end
  edn
```

#設定

```ruby
#config/environments

#忽略任何寄信錯誤
config.action_mailer.raise_delivery_errors = false
#測試用，不會真的寄信
config.action_mailer.delivery_method = :test
#用什麼方式傳遞
config.action_mailer.delivery_method = :smtp
#網站網址
config.action_mailer.default_url_options = { host: "http://localhost:3000" }
#一定要轉 symbol 不然會吃不到
config.action_mailer.smtp_settings = config_for(:email).symbolize_keys
#使用 sidekiq 做背景處理
config.active_job.queue_adapter = :sidekiq 
```

設定擋

```ruby
#config/email.yml
development:
  address: "smtp.mailgun.org"
  port: 587
  domain: "google.com"
  authentication: "plain"
  user_name: "postmaster@leon.tw"
  password: "1234567890"
  enable_starttls_auto: true
```
#預覽測試

[letter_opener](https://github.com/ryanb/letter_opener) 這個 gem 可以不用真的寄信，而是直接在畫面上顯示寄信內容，就可以拿來做測試

```ruby
#config/environments
config.action_mailer.delivery_method = :letter_opener
```

官方文件：  
[Action Mailer](http://guides.rubyonrails.org/action_mailer_basics.html)    
[Action Mailer 中文](http://rails.ruby.tw/action_mailer_basics.html)

參考文件：  
[ActionMailer - E-mail 發送](https://ihower.tw/rails4/actionmailer.html)  
[如何正確發送(大量) Email 信件](https://ihower.tw/blog/archives/3481)

gem：  
[madmimi-gem](https://github.com/madmimi/madmimi-gem)  
[mailgun-ruby](https://github.com/mailgun/mailgun-ruby)  
[letter_opener](https://github.com/ryanb/letter_opener) 

email:  
[madmimi.com](https://madmimi.com/)  
[mailgun](https://www.mailgun.com/)  
[aws](http://aws.amazon.com/tw/ses/)  
[mailchimp](http://mailchimp.com/)  
[SendGrid](http://sendgrid.com/)