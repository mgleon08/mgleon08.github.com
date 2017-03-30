---
layout: post
title: "自己定義 Rake Tasks"
date: 2016-03-10 21:29:39 +0800
comments: true
categories: rails
---

在寫 rails 當中，經常會用到 rake xxx，現在也可以自己定義了!

<!-- more -->

檔案放在 `/lib/tasks/xxx.rake`
記得後面副檔名是 `rake`

#[Ubike](http://data.taipei/opendata/datalist/apiAccess?scope=resourceAquire&rid=ddb80380-f1b3-4f8e-8016-7ed9cba571d5)

get 台北 Uike 資料，並存取到資料庫。

可以用 [rest-client](https://github.com/rest-client/rest-client) 這個gem
先用 `get` 取得資料，再用 `JSON.parse` 來將 `string` 解析成 `hash`

`rake dev:fetch_ubike`

```ruby
#lib/tasks/dev.rake

desc "get Ubike date" # 新增訊息在 rake --tasks 裡面
namespace :dev do

  task :fetch_ubike => :environment do
    puts "fetching ubike"

    url = "http://data.taipei/opendata/datalist/apiAccess?scope=resourceAquire&rid=ddb80380-f1b3-4f8e-8016-7ed9cba571d5"

    raw_content = RestClient.get(url)

    data = JSON.parse( raw_content )

    data["result"]["results"].each do |u|
      a = Ubike.find_by_ubike_id( u["_id"] )

      if a == nil
        # maybe update it!
        Ubike.create( :ubike_id => u["_id"], :name => u["sna"])
      else
        Ubike.update( :ubike_id => u["_id"], :name => u["sna"])
      end
    end

  end

end
```
這樣只要打 `rake dev:fetch_ubike` 就會自動跑了!

#[立委資料](http://vote.ly.g0v.tw/api/vote/?page=1)

`rake vote:fetch_raw_vote`

```ruby
#立委資料，資料相當大，很多分頁

desc "get vote data" # 新增訊息在 rake --tasks 裡面
namespace :vote do

 task :fetch_raw_vote => :environment do
   puts "fetching raw_vote"

   url = "http://vote.ly.g0v.tw/api/vote/?page=1"
   raw_content = RestClient.get(url)
   data = JSON.parse( raw_content )
     while data["next"] != nil
        data["results"].each do |r|

           Vote.create( :url => r["url"],
                           :uid => r["uid"],
                           :sitting_id => r["sitting_id"],
                           :vote_seq => r["vote_seq"],
                           :content => r["content"],
                           :conflict => r["conflict"],
                           :results => r["results"],
                           :result => r["result"])
         end

         url = data["next"]
         raw_content = RestClient.get(url)
         data = JSON.parse( raw_content )
      end

 end

end
```

#secret.yml

或是定義可以直接下指令新增 `secret.yml`

`rake generate:secrets`

```ruby
def paste_secrets
  text = "
    # Be sure to restart your server when you modify this file.

    # Your secret key is used for verifying the integrity of signed cookies.
    # If you change this key, all old signed cookies will become invalid!

    # Make sure the secret is at least 30 characters and all random,
    # no regular words or you'll be exposed to dictionary attacks.
    # You can use `rake secret` to generate a secure secret key.

    # Make sure the secrets in this file are kept private
    # if you're sharing your code publicly.

    development:
      secret_key_base: #{`rake secret`.strip}

    test:
      secret_key_base: #{`rake secret`.gsub('\n','')}

    # Do not keep production secrets in the repository,
    # instead read values from the environment.
    production:
      secret_key_base: #{`rake secret`.gsub('\n','')}
  "
  path = "#{Rails.root}/config/secrets.yml"
  File.open(path,"w"){|file|
    file.write(text)
  }
end

desc "Add secrets.yml" # 新增訊息在 rake --tasks 裡面
namespace :generate do
  task :secrets do
    puts "Generating: #{Rails.root}/config/secrets.yml"
    paste_secrets
    puts "Done!"
  end
end
```

#Rake

`rake -T` 可以看到定義好 `desc` 裡的內容

[Ruby 語法放大鏡之「常在終端機裡下 rake db:migrate 指令，這個 rake 是什麼，後面那個 db:migrate 又是怎麼回事?」](http://kaochenlong.com/2016/04/30/rake/)

之前寫的Json
[Json](http://mgleon08.github.io/blog/2016/01/09/ruby-on-rails-json/)

參考資料：
[錦囊妙計](https://ihower.tw/rails4/rails-recipes.html)
