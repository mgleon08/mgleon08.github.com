---
layout: post
title: "Ruby on Rails - json"
date: 2016-01-09 12:26:46 +0800
comments: true
categories: rails gem
---

JSON 是很經常是用到的格式，不管是和程式溝通或是交換資料。

<!--more-->

#什麼是 JSON

JSON 是個以純文字為基底去儲存和傳送簡單結構資料，可以透過特定的格式去儲存任何資料(字串,數字,陣列,物件)，也可以透過物件或陣列來傳送較複雜的資料。

一旦建立了您的 JSON 資料，就可以非常簡單的跟其他程式溝通或交換資料，因為 JSON 就只是純文字個格式。

JSON 的優點如下:

* 相容性高
* 格式容易瞭解，閱讀及修改方便
* 支援許多資料格式 (number,string,booleans,nulls,array,associative array)
* 許多程式都支援函式庫讀取或修改 JSON 資料

參考文件：
[你不可不知的 JSON 基本介紹](https://blog.wu-boy.com/2011/04/%E4%BD%A0%E4%B8%8D%E5%8F%AF%E4%B8%8D%E7%9F%A5%E7%9A%84-json-%E5%9F%BA%E6%9C%AC%E4%BB%8B%E7%B4%B9/)

#Rails 如何傳遞JSON

###respond_to

在 `rails` 當中可以用 `respond_to ` 設定回傳的 `format`。

```ruby
respond_to do |format|
	format.html { redirect_to :back }
	format.json
	format.js
end

#也可以寫成單行
respond_to :html, :json, :js
```
若後面沒指定會去找 `view` 中，後面是 `.json` 或 `.js` 的檔案
但記得因為 `format` 有三種，所以要 json 資料的話就在網址後面加 `.json`

respond_to可以用來回應不同的資料格式。Rails內建支援格式包括有
`:html, :text, :js, :css, :ics, :csv, :xml, :rss, :atom, :yaml, :json`

>如果需要擴充，可以編輯config/initializers/mime_types.rb這個檔案

###render
可以簡單使用 `render json` 的方式，直接強制 html 輸出成 json 格式
`render json: User.info`
這樣連view都不需要，就會直接顯示。

或是直接 `render template` 指定輸出 json 格式
`render template: "api/users/index.json.jbuilder"`

###routes scope設定，指定controller使用json格式輸出
最後是直接設定好 `routes` 的 `default` 格式，這樣就不用再指定要 `render` 什麼!

```ruby
scope :path => '/api/v1/', :defaults => { :format => :json }, :module => "api_v1", :as => 'v1' do
    resources :users #ApiV1::CompaniesController
end
```
`path`：指令網址前面的路徑
`defaults`：指定default的格式
`module`：指定 controller 會是 ApiV1::UsersController
`as`：產生URL helper

參考文件：
[Scope](https://ihower.tw/rails4/routing.html)
[Rails修改預設顯示格式為json](http://motion-express.com/blog/20141124-rails-default-render-json)

#搭配gem - [jbuilder](https://github.com/rails/jbuilder)

再rails當中，很常會用這個 `gem` 來轉 `json`

像是剛才的`render template: "api/users/index.json.jbuilder"`
就會去找這個 template，並且像是 `html.erb` 一樣可以直接使用 `@` 的參數。

```ruby
#api/users/index.json.jbuilder

json.info do
  json.number do
    json.total User.count
  end

  json.data @users do |u|
    json.id u.id
    json.name u.name
  end
end
```
就會生產出以下

```ruby
#json
{
"info": {
  "number": {
     "total": 1
  },
  "data": [
     {
      "id": 1,
      "name": "abc"
			}
		]
	}
}
```

#接收JSON

可以用 [rest-client](https://github.com/rest-client/rest-client) 這個gem
先用 `get` 取得資料，再用 `JSON.parse` 來將 `string` 解析成 `hash`

範例： [Ubike](http://data.taipei/opendata/datalist/apiAccess?scope=resourceAquire&rid=ddb80380-f1b3-4f8e-8016-7ed9cba571d5) 資料，並存取到資料庫。

```ruby
#lib/tasks/dev.rake

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

範例2：[立委資料](http://vote.ly.g0v.tw/api/vote/?page=1)

```ruby
#立委資料，資料相當大，很多分頁
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

參考文件：
[你不可不知的 JSON 基本介紹](https://blog.wu-boy.com/2011/04/%E4%BD%A0%E4%B8%8D%E5%8F%AF%E4%B8%8D%E7%9F%A5%E7%9A%84-json-%E5%9F%BA%E6%9C%AC%E4%BB%8B%E7%B4%B9/)
[Scope](https://ihower.tw/rails4/routing.html)
[Rails修改預設顯示格式為json](http://motion-express.com/blog/20141124-rails-default-render-json)

gem：
[jbuilder](https://github.com/rails/jbuilder)
[rest-client](https://github.com/rest-client/rest-client)

