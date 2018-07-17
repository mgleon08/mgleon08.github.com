---
layout: post
title: "Rails 5.2 credentials"
date: 2018-07-14 00:22:53 +0800
comments: true
categories: rails
---

rails5.2 開始 `config/secrets.yml` 已經被移除了，取而代之的是 `credentials`

<!-- more -->

rails5.2 會新增兩個檔案

* `config/credentials.yml.enc` is an encrypted file that will contain all your secret credentials，因為加密過可以很放新的推到 github

* `config/master.key` is a file containing your encryption key

`mater.key` 是用來加解密 `credentials.yml.enc`，記得不要上到 github，要放到 `.gitignore`

```ruby
# .gitignore
# Ignore master key for decrypting credentials and more.
/config/master.key
```

```ruby
# 可以看說明
rails credentials:help
```
因為經過加密，所以打開 `credentials.yml.enc` 會發現一串亂碼

```ruby
# credentials.yml.enc
17NEkGq/xkDO...
```

如果要編輯檔案需要另外執行解密，才能夠進行編輯

```ruby
# EDITOR 可以改用其他編輯器 subl code 都可以
EDITOR=vim rails credentials:edit
```

```ruby
# aws:
#   access_key_id: 123
#   secret_access_key: 345

# Used as the base secret for all MessageVerifiers in Rails, including the one protecting cookies.
secret_key_base: 008174812dc5a309a0de...
```

離開後就會自動儲存

```ruby
# 這時候就會用 master.key 進行加密
New credentials encrypted and saved.
```

讀取檔案

```ruby
Rails.application.credentials.dig(:secret_key_base)
Rails.application.credentials[:secret_key_base]
Rails.application.credentials.aws[:access_key_id]
Rails.application.credentials.aws[:secret_access_key]
Rails.application.credentials.secret_key_base
```

並且向其他 `yml` 檔案一樣，建立一在 `share` 的資料夾中，或是新增一個環境變數 `RAILS_MASTER_KEY`

### Environments

如果想像之前一樣，不同環境有不同的 key，可以設定成跟之前的 `secret.yml` 一樣

```ruby
development:
  aws:
    access_key_id: 123
    secret_access_key: 345
production:
  aws:
    access_key_id: 321
    secret_access_key: 543
```

```ruby
Rails.application.credentials[Rails.env.to_sym][:aws][:access_key_id]
```

### Production

如果在 `production` 環境，要做以下設定

```ruby
#config/environments/production.rb
...
config.require_master_key = true
...
```

參考文件

* [Rails 5.2 credentials](https://medium.com/cedarcode/rails-5-2-credentials-9b3324851336)
* [Storing Secret Credentials in Rails 5.2 and Up](https://www.viget.com/articles/storing-secret-credentials-in-rails-5-2-and-up/)
* [Rails 5.2: encrypted secrets](https://keithpblog.org/post/encrypted-secrets/)
