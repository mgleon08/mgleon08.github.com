---
layout: post
title: "rails routes設定"
date: 2016-03-26 09:37:30 +0800
comments: true
categories: rails
---
要如何設計網站的 router 是非常重要的一件事。  
在 rails 當中也有許多方法可以做設定。

<!-- more -->



```ruby
#直接設定
get '/patients/:id', to: 'patients#show'

#REST
#設定 scope 有點像資料夾的概念，可用來區分後台可控制範圍
namespace do 
  #也可用單數 resource (會少了 index，然後不用帶id)
  resources :articles, only: [:index, :show] do 
    member do #Acts on a single resource
      post :follow
    end
    collection do #Acts on a collection of resources
      get :follow
    end
  end
end
```

#scope

```ruby
get 'foo/meetings/:id', :to => 'events#show'
post 'foo/meetings', :to => 'events#create'

可以改寫成

scope :controller => "events", :path => "/foo", :as => "bar" do
  get 'meetings/:id' => :show, :as => "meeting"
  post 'meetings' => ':create	, :as => "meetings"
end

scope :path => '/api/v1/', :module => "api_v1", :as => 'v1' do
  resources :projects
end
```
`:as` 增加 vi_path(相對路徑) 和 v1_url(絕對路徑)  
`: path `網址 `/api/v1/projects`   
`:controller` 指定 controller 是哪個  
`:module` 指定 controller 對應到 ApiV1::ProjectsController


#導向

```ruby
#靜態
get "/welcome" to: redirect("/hello")

#動態
get '/stories/:name', to: redirect('/articles/%{name}')
```
`redirect` 將網址導向到另一個網址

#設定沒有指定的 path 都導向同一個頁面

```ruby
get "*path", to: "welcome#welcome", :defaults => { :format => :json }
```
`defaults` 設定此 routes 輸出都是 json 格式

#特殊條件限定

```ruby
#過濾id
get 'photos/:id', to: 'photos#show', id: /[A-Z]\d{5}/

#黑名單
class BlacklistConstraint
  def initialize
    @ips = Blacklist.retrieve_ips
  end
 
  def matches?(request)
    @ips.include?(request.remote_ip)
  end
end
 
Rails.application.routes.draw do
  get '*path', to: 'blacklist#index',
    constraints: BlacklistConstraint.new
end

or

Rails.application.routes.draw do
  get '*path', to: 'blacklist#index',
    constraints: lambda { |request| Blacklist.retrieve_ips.include?(request.remote_ip) }
end
```

###子網域

```rubyresources :posts, constraints: { subdomain: 'api' }
```

```ruby￼constraints subdomain: 'api' do  resources :posts  #http://api.mgleon08.com/posts￼end
```

```ruby
namespace :api do
  constraints subdomain: 'api' do
    resources :posts
  end
end

#產生兩個 api
#http://api.mgleon08.com/api/posts

constraints subdomain: 'api' do 
	namespace :api, path: '/' do #加上 path 就可以消去後面的     resources :zombies   endend
#上下一樣
namespace :api, path: '/', constraints: { subdomain: 'api' } do
  resources :zombiesend

#http://api.mgleon08.com/posts
```

#DRY

### concern
```ruby
concern :sociable do |options| 
  resources :comments, options
  resources :categories, options 
  resources :tags, optionsend
resources :messages, concerns: :sociableresources :posts,    concerns: :sociable￼resources :items do  concerns :sociable, only: :createend
```

###or

```ruby
concern :sociable, Sociable
resources :messages, concerns: :sociable 
resources :posts, concerns: :sociable 
resources :items do  concerns :sociable, only: :createend
```

```ruby
#app/concerns/sociable.rbclass Sociable  def self.call(mapper, options)    mapper.resources :comments, options 
    mapper.resources :categories, options 
    mapper.resources :tags, options  end 
end
```


### 合併

```ruby
resources :zombies, only: :index 
resources :humans, only: :inde
```

```ruby
￼with_options only: :index do |list_only|   list_only.resources :zombies   list_only.resources :humans   list_only.resources :medical_kits￼￼￼￼end
```

#將 module 名稱改為大寫

```ruby
#app/controllers/api/posts_controller.rb
￼module Api #通常是camelcase   class PostController < ApplicationController   end
end
```

```ruby
#config/initializers/inflections.rb
￼ActiveSupport::Inflector.inflections(:en) do |inflect|￼  inflect.acronym 'API'￼end
```
#CURL

可以用 command line 來測試 get

```ruby
curl -i localhost:3000/posts
#-i 詳細資訊

curl -IH "Accept: application/json" localhost:3000/posts
curl -IH "Accept: application/xml" localhost:3000/posts
#H HEAD

curl -i -X POST -d 'episode[title]=ZombieApocalypseNow' http://localhost:3000/posts

#-X the -X option specifies the method
#-d use -d to send data on the request

curl -Iu 'carlos:secret' http://localhost:3000/posts
#上下一樣
curl -I http://carlos:secret@localhost:3000/episodes
#send Basic Auth credentials with the -u option

curl -IH "Accept: application/json" -u 'carlos:fakesecret' http://localhost:3000/posts
#client asks for JSON

curl -IH "Authorization: Token token=16d7d6089b8fe0c5e19bfe10bb156832" http://localhost:3000/posts
#Set token on Authorization header
```

官方文件：  
[Rails Routing from the Outside In](http://guides.rubyonrails.org/routing.html)  
[Rails 路由：深入淺出](http://rails.ruby.tw/routing.html)

參考文件：  
[路由(Routing)](https://ihower.tw/rails4/routing.html)  