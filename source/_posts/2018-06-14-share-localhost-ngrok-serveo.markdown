---
layout: post
title: "分享內網 localhost to everybody - ngrok, serveo"
date: 2018-06-14 19:06:43 +0800
comments: true
categories: other
---

當有時候必須請其他人來看 local 上面的狀況時，可以用以下工具，分享給其他人來看

<!-- more -->

工具有兩種

# 1. [ngrok](https://ngrok.com/)

* 先安裝 or [官網下載](https://ngrok.com/download)

```ruby
brew cask install ngrok
# brew cask裝的大多是有gui界面的app以及驅動，brew cask是brew的一個官方源。 像是 chrome
```

* 如果 share 的是 5000 port

```ruby
ngrok http 5000
```

```ruby
Session Status                online
Session Expires               7 hours, 59 minutes
Version                       2.2.8
Region                        United States (us)
Web Interface                 http://127.0.0.1:4040
Forwarding                    http://1ee7a4e7.ngrok.io -> localhost:5000
Forwarding                    https://1ee7a4e7.ngrok.io -> localhost:5000

Connections                   ttl     opn     rt1     rt5     p50     p90
                              0       0       0.00    0.00    0.00    0.00
```

可以看到 `http://1ee7a4e7.ngrok.io` 指向到我們剛剛指定的 `localhost:5000` 就可以將這串網址給別人來看囉


* 也可以加上密碼或是 `subdomain` 不過就是要付費囉


# 2. [serveo](https://serveo.net/)

這個是同事發現的，連安裝都不需要

```ruby
ssh -R 80:localhost:5000 serveo.net
```

* 自訂subdomain

```ruby
ssh -R leon.serveo.net:80:localhost:5000 serveo.net
```

這樣 `https://leon.serveo.net` 就可以連到了~~

參考文件:

* [怎麼將內網的 localhost 讓外面的人都看得到呢？用用 ngrok 吧！](https://tenten.co/blog/how-to-use-ngrok-to-connect-your-localhost/)
* [利用serveo把local開發環境發佈到internet中](https://guahsu.io/2018/06/expose-local-servers-to-the-internet-by-serveo/)