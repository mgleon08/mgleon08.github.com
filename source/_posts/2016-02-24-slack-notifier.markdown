---
layout: post
title: "用 slack 通知大小事"
date: 2016-02-24 20:24:11 +0800
comments: true
categories: rails gem
---

透過 slack 來通知程式上的人任何事情，相當方便。

<!-- more -->

#申請 webhook

* 到 slack 網站，根據想發送訊息的 channel 申請一個 [Incoming Webhooks](https://api.slack.com/incoming-webhooks#putting_it_all_together)

>Incoming Webhooks 傳到 slack
>Outgoing Webhooks slack 傳出去

* 申請好了會給一串 `url`


#安裝

```ruby
gem 'slack-notifier'
```

#設定

```ruby
def notify_system_manager_by_slack(parameter)
  notifier = Slack::Notifier.new("WebHookUrl")
  notifier.username = "[Sidekiq] Error Notifier"

  message  = "Time:"    + "#{Time.now.strftime('%Y-%m-%d %H:%M:%S')}\n"
  message += "Message:" + "#{parameter}\n"

  setting = {
    text:  message,
    color: "danger"
  }

  notifier.ping "\n", attachments: [setting]
end
```

#再通知加上其他訊息
可以在 slack 上面加上顏色，訊息，或圖片等等的
都可以在網站上找到 [Attachments](https://api.slack.com/docs/attachments)

#錯誤通知
如果希望有錯誤能夠通知你，可以用第三方的工具 + slack 更加方便

[Rollbar](https://rollbar.com/)
[Sentry](https://getsentry.com/welcome/)

gem：
[slack-notifier](https://github.com/stevenosloan/slack-notifier)
