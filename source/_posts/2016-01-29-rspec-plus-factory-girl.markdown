---
layout: post
title: "用 Rspec + Factory Girl 寫測試"
date: 2016-01-29 20:36:46 +0800
comments: true
categories: rails gem rspec
---

程式寫久之後，就會發現測試的重要性!  
因此來介紹 rails 中，比內建測試還好用的 rspec 搭配 factory_girl

<!-- more -->

#測試種類

* 單元測試(Unit test)  
針對每個程式各個最小單位進行測試，像是在 controller 就單單只測試 controller 裡面的 action，而裡面產生的 model,method，都用假的方式來取代，已確保有錯誤時，可以很快知道是哪邊有問題 。
	
* 整合測試(Integration test)  
主要是用來測試，每個 class 的互動，像是 controller 裡面會 call 到 model ，也會 call 到 view ，並測試回傳的值是否正確。


#寫測試的好處
* Instant Feedback 即時反饋（寫測試的時間 < debug的時間）
* 回歸測試及重構 （重構時就不需要再重複的測試）
* 幫助設計API（TDD = 先測試，在實作）
* 一種程式文件（可以讓很快就知道之前api怎麼寫的）

#慣例

* ⼀個 rb 檔案配⼀個同名的 _spec.rb 檔案
* guard 等⼯具容易設定  
[guard-rspec](https://github.com/guard/guard-rspec) 程式⼀修改完存檔，⾃動跑對應的測試（bundle後，輸入 guard init repec 初始化，打guard（bundle exec guard 真正執行））
* editor 有⽀援快速鍵
* describe “#name” 是 instance method
* describe “.name” 是 class method
* 測試spec盡量比較簡單清楚，可以不用DRY，實作才會要DRY

#輸出格式

* rspec filename.rb 預設不產⽣⽂件
* rspec filename.rb -fd 輸出 specdoc ⽂件
* rspec filename.rb -fh 輸出 html ⽂件

#安裝

```ruby
group :development, :test do
  gem 'rspec-rails'
  gem 'factory_girl_rails'
end
```

#設定

### 顏色描述
```ruby
#vi .rspec檔案輸入
--color #顯示顏色
--format #documentation顯示描述
```
###將不需要的檔案關閉
generate 新的 controller 或是 model 時，rails 就會很聰明的順便新增 sepc 檔案，但有時候我們會希望用到的時候再去建立即可，所以需要關閉就輸入以下指令。

`/config/application.rb`

```ruby
config.generators do |g|
  g.view_specs false
  g.helper_specs false
  g.request_specs false
  g.controller_specs false
  g.routing_specs false
end
```

#Rspec


### model
```ruby
require 'rails_helper' #必須載入才能使用裡面的方法

RSpec.describe Post, type: :model do #RSpec 可省略
    it "is accessible" do
        post = Post.create!
        expect(post).to eq(Post.last)
    end

    it "has title and content columns" do
        columns = Post.column_names
        expect(columns).to include("id")
    end
end
```

* `describe`, `context` 描述要測試的是什麼，可以用nested   
* `it`, `specify`, `example` 就是⼀⼩段測試
* `expect(…).to` 或 `expect(…).to_not` 定義期望
* `eq` 預期的是否和自己設定的相等
* `include` 預期的是否有包括自己設定的值
* `describe` 和 `it` 前面加上 x 代表 pending，執行 rspec 就會先跳拓
*  [其他方法](https://www.relishapp.com/rspec/rspec-expectations/v/3-4/docs/built-in-matchers/change-matcher)

###Routing spec syntax
```ruby
expect(:get => "/events").to route_to("events#index")
expect(:get => "/widgets/1/edit").not_to be_routable

expect(:get => "/posts/1").to route_to(
      :controller => "posts",
      :action => "show",
      :id => "1"
      )
```

###Controller spec syntax
```ruby
expect(response).to render_template(:new)
expect(response).to redirect_to(events_url)
expect(response).to have_http_status(200)
expect(assigns(:event)).to be_a_new(Event)
```

###View spec syntax
```ruby
render
expect(rendered).to include("Title")
expect(response).to render_template(partial: "_form")
```

###Helper spec syntax
```ruby
expect(helper.your_method).to eq("")
```

###request
```ruby
RSpec.describe "Users", :type => :request do
  before do
    @user = User.create(name: "hello")
  end

  it "GET /users" do
    get "/users"
    expect(response).to have_http_status(200)
    expect(response).to render_template(:index)
    expect(response.body).to include("hello")
  end
  
  it "GET /user/:id" do
    get "/user", id: @user.id
    expect(response).to have_http_status(200)
    expect(response).to render_template(:index)
    expect(response.body).to include("hello")
  end
```
request 通常直接從網址進行 Get 或 Post ，接著判斷傳回來的值是否正確。

```ruby
before do  
	@user = User.new(name: "hello")
end

before(:all) do
	@user = User.new(name: "hello")
end

#也有 after(:each)，afte(:all)
```

* before(:each) 每段it之前執行
* before(:all) 整段describe前只執行一次
* after(:each) 每段it之後執行
* after(:all) 整段describe後只執行一次
* (:each) 可以不用加，預設為(:each) 

```ruby
let(:user){User.new(:name => "hello")}
```

* 相較於 before(:each) 可增加執⾏速度  
* 有使⽤到才會運算(lazy)，並且在同⼀個 example 測試中多次呼叫會 Memoized 快取起來。  
* let! 則是⾮ lazy 版本  

#Stub

```ruby
allow_any_instance_of(User).to receive(:follow).and_return(false)
```
用stub 假造 method，讓它忽略這個 method，或是指定回傳東西，可以避免在測試時，測試不必要的東西。


#factory_girl

到 `spec/rails_helper.rb` 設定

```ruby
RSpec.configure do |config|
  config.include FactoryGirl::Syntax::Methods
end
```
在 `spec` 底下新增 `factories` 資料夾，接著在裡面新增相對應的物件名稱，像是 `user.rb`

`spec/factories/user.rb`

```ruby
FactoryGirl.define do
  factory :user, class: User" do
    name "hello"
    age  18
    
    # 可以設定create 之後要做哪些動作
    after(:create) do |video|
      create(:photo, photo_id: photo.id)
      create(:photo, size: "500", photo_id: photo.id)
      create(:photo, size: "800", photo_id: photo.id)
    end
    
    # 也可以設定多種條件
    trait :child do
      age 6
      #after(:create) {|user| user.add_role(:admin) } 
      #after(:build)  {|user| user.add_role(:admin) } 
      # 也可以設定 create 之後的設定
    end
  end
end
```
這樣在 spec 裡面就可以直接建立假資料

```ruby
before do  
	@user  = FactoryGirl.create(:user) #FactoryGirl 可省略
	@child = create(:user, :child) #就只替換掉 age
end
```

###為什麼要假物件?
* 無法控制回傳值的外部系統 (例如第三⽅ web service)
* 建構正確的回傳值很⿇煩 (例如得準備很多假資料)
* 可能很慢，拖慢測試速度 (例如耗時的運算)
* 有難以預測的回傳值 (例如亂數⽅法)
* 還沒開始實作 (特別是採⽤ TDD 流程)

#Capybara
RSpec除了可以拿來寫單元程式，我們也可以把測試的層級拉高做整合性測試，以Web應用程式來說，就是去自動化瀏覽器的操作，實際去向網站伺服器請求，然後驗證出來的HTML是正確的輸出。

[capybara](https://github.com/jnicklas/capybara)就是一套可以搭配的工具，用來模擬瀏覽器行為

#CI server
CI(Continuous Integration) 
伺服器的用處是每次有人Commit就會自動執行編譯及測試(Ruby不用編譯，所以主要的用處是跑測試)，並回報結果，如果有人送交的程式搞砸了回歸測試，馬上就有回饋可以知道。

[circleci.com](https://circleci.com)

建立 `circle.yml`

```ruby
machine:
  timezone:
    Asia/Taipei
  ruby:
    version: 2.1.2
dependencies:
  pre:
    - rvm use 2.1.2
    - gem install bundler
    - gem install rubocop
  post:
    - gem update rake
database:
  override:
    - cp config/database.yml.example config/database.yml
    - rake db:create db:migrate --trace
test:
  override:
    - bundle exec rspec --color

```

建立 `config/database.yml.example`  

```ruby
default: &default
  adapter: mysql2
  encoding: utf8
  host: localhost
  username:
  password:

test:
  <<: *default
  database: test_db

development:
  <<: *default
  database: development_db

production:
  <<: *default
  database: production_db
```

接著到 [circleci.com](https://circleci.com) 和 github 帳號做連結。  
接著將要跑的 project 加進去，之後只要 push 到 github 就會自動跑了！  


#大師引言
```
大部份都是寫model的測試
controller偶爾會寫

其他的因為後面有驗收測試也會測試到，所以不浪費時間去寫測試
驗收測試多半是測試子路徑，不會測到所有的條件，所以個別的小項測試，就直接在model寫就好了

工程是寫到request就很棒了
feature 比較像是QA在寫的
```
官方文件：  
[Better Specs](http://betterspecs.org/)  
[Relish](https://www.relishapp.com/)  

Gem：  
[rspec-rails](https://github.com/rspec/rspec-rails)  
[factory_girl_rails](https://github.com/thoughtbot/factory_girl_rails)    
[guard-rspec](https://github.com/guard/guard-rspec)  
[capybara](https://github.com/jnicklas/capybara)

參考文件：  
[自動化測試](https://ihower.tw/rails4/testing.html)  
[RSpec-Rails (基礎篇)](http://motion-express.com/trainings/rspec-rails-1)  
[RSpec-Rails當中自訂methods及helpers](http://motion-express.com/blog/20150320-custom-helpers-in-rspec)  
[RSpec-Rails 針對module進行unit test](http://motion-express.com/blog/20150327-rspec-rails-testing-module)  
[RSpec 讓你愛上寫測試](http://www.slideshare.net/ihower/rspec-7394497)  
[程式設計師升級必練內功：TDD Kata](https://blog.alphacamp.co/2015/03/02/tdd-kata/)  
[codility 練習](https://codility.com/programmers/lessons/)  
[保齡球練習](http://www.sportcalculators.com/bowling-score-calculator)