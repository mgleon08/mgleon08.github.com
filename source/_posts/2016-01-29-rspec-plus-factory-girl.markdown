---
layout: post
title: "用 Rspec + Factory Girl + CircleCi 寫測試"
date: 2016-01-29 20:36:46 +0800
comments: true
categories: rails gem rspec
---

程式寫久之後，就會發現測試的重要性!
因此來介紹 rails 中，比內建測試還好用的 rspec 搭配 factory_girl

<!-- more -->

###目錄
* [Rspec](#rspec)
* [Factory Girl](#factory_girl)
* [Capybara](#capybara)
* [CI Server](#ci)
* [Other](#other)

#<span id="rspec">RSpec</span>

Ruby 的測試 DSL (Domain-specific language)

- Semantic Code：⽐ Test::Unit 更好讀，寫的⼈ 更容易描述測試⺫的
- Self-documenting：可執⾏的規格⽂件

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

* ⼀個 rb 檔案配⼀個同名的 _spec.rb 檔案 (非常重要，如果檔名後面沒加 _spec 就不會跑)
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

#ruby
gem install rspec
rspec --init
#create   .rspec
#create   spec/spec_helper.rb

#rails
rails generate rspec:install
```

#設定

### 顏色描述
```ruby
#vim .rspec檔案輸入
#Require spec_helper automatically in your *_spec.rb
--require spec_helper

#顯示顏色
--color

#顯示描述
--format documentation 
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

#語法介紹

- describe 或 context 用來描述你要測試的是什麼，可以用nested
- it 每個 it 就是⼀⼩段測試（it, specify, example都是一樣的）
- expect(…).to 或 to_not 所有物件都有這個⽅法來定義你的期望
- before(:each) 每段 it 之前執⾏
- before(:all) 整段 describe 執⾏⼀次 ＊沒加(:xxx)預設是:each
- after(:each) • afte(:all)
- pending 可以先列出要測試的，但不用執行，直接在it前面加上x，xit
- let(:name) { exp }
    - 相較於 before(:each) 可增加執⾏速度
    - 有使⽤到才會運算(lazy)，並且在同⼀個 example 測試中多次呼叫會 Memoized 快取起來。
    - • let! 則是⾮ lazy 版本 (will create before every example)
- 別名
    - describe = context
    - it = specify = example

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
RSpec.describe "posts", :type => :routing do

  expect(:get => "/events").to route_to("events#index")
  expect(:get => "/widgets/1/edit").not_to be_routable

  expect(:get => "/posts/1").to route_to(
        :controller => "posts",
        :action => "show",
        :id => "1"
        )
end
```

###Controller spec syntax
```ruby
RSpec.describe PostsController, type: :controller do
  expect(response).to render_template(:new)
  expect(response).to redirect_to(events_url)
  expect(response).to have_http_status(200)
  expect(assigns(:event)).to be_a_new(Event)
end
```

###View spec syntax
```ruby
RSpec.describe "posts/index.html.erb", type: :view do
  render
  expect(rendered).to include("Title")
  expect(response).to render_template(partial: "_form")
end
```

###Helper spec syntax
```ruby
expect(helper.your_method).to eq("")
```

### model spec
```ruby
describe Material::Banner, type: :model do
end

#驗證

it 'fails validation with no wrong video type' do
  video = Video.new(file: file)
  video.valid?
  expect(video.errors[:file].first).to include('allowed types: mp4, flv')
end

or

expect(video).to be_invalid
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


###預期會執行某一個class的methd

```ruby
 expect(Clsss).to receive(:method).with(params)
```

###測試 Pattern

Four-Phase Test

* Setup （準備測試資料）
* Exercie （實際執行測試）
* Verification （驗證測試結果）
* Teardown （拆解測試）


###double
假物件，可用於 mock 中指定回傳的值  

mock 主要是用來模擬「外部邏輯」，因此可以使用 mock objects，在 RSpec 裡面叫做 double（替身）。

```ruby
require 'rails_helper'

RSpec.describe BooksCalculator do
  describe "#books_count" do 
    it "returns books count" do 
      course = double(:books_count => 5 )
      book_calculator = StudentsCalculator.new(course)
      expect(book_calculator.books_count).to eq(5)
    end
  end
end
```

###Stub

`Stub:`
For replacing a method with code that returns a specified result.  

* 用stub 假造 method，讓它忽略這個 method，或是指定回傳東西，可以避免在測試時，測試不必要的東西。  
* 專注於要測試的東西，如果 method 有呼叫其他 method 就可以 stub 掉
* 製造假物件，指定這個假物件能回應哪些訊息，還有對應的回傳值，讓要測試的主角，在執行過程中一些地方，能獲得一致的結果

* 要測試的 model

```ruby
class Zombie < ActiveRecord::Base
  has_one :weapon

  def decapitate
    weapon.slice(self, :head)
    self.status = "dead again"
  end
end
```

* 我们要測試 decapitate 方法, 但裡面調用了 weapon 的 slice 方法, 所以我們要把它 stub 掉

```ruby
describe Zombie do
  let(:zombie) { Zombie.create }

  context "#decapitate" do
    it "sets status to dead again" do
      allow(zombie.weapon).to receive(:slice) #Rspec 3 之後的語法
      #zombie.weapon.stub(:slice)  #模擬 slice 阻斷掉他 Rspec 2
      zombie.decapitate
      zombie.status.should == "dead again"
    end
  end
end
```

###Mock

`Mock:`
A stub with an expectations that the method gets called.  

* 假造 method，不只忽略這個 method ，並且期望被呼叫到
* 「模擬」與一個「協作者」的互動，設立一個「會收到指定訊息」的期望，去驗證互動是否真的有發生。
* 簡單來說 mock 就是 stub + expectation , 說它是 stub 是因為它也可以像 stub 一樣偽造方法, 阻斷對原來方法的調用, expectation 是說它不僅偽造了這個方法,它還期望你(必須)調用這個方法,如果沒有被調用到,這個 test 就 fail 了
* 一樣製造假物件，但是現在我們改對這個假物件斷言，在主角的執行過程中，應該要收到什麼訊息、收到的訊息應該夾帶什麼參數、訊息收到的次數...等等，但是會有程式碼的流程些微不自然的問題（準備、斷言、（準備）、執行）

使用 mock objects，在 RSpec 裡面叫做 double（替身），來取代「外部邏輯」資料

```ruby
# simulate a not found resource
context "when not found" do
  before { allow(Resource).to receive(:where).with(created_from: params[:id]).and_return(false) }
  it { is_expected.to respond_with 404 }
end
```

```ruby
describe Zombie do
  let(:zombie) { Zombie.create }

  context "#decapitate" do
    it "calls weapon.slice" do
      zombie.weapon.should_receive(:slice)
      zombie.decapitate
    end
  end
end
```

```ruby
allow_any_instance_of(User).to receive(:follow).and_return(false)
```
[mock](http://betterspecs.org/#mock)

範例:

```ruby
#/app/models/zombie.rbdef geolocate  loc = Zoogle.graveyard_locator(self.graveyard)  "#{loc[:latitude]}, #{loc[:longitude]}"end
#/spec/models/zombie_spec.rbit "calls Zoogle.graveyard_locator" do  Zoogle.should_receive(:graveyard_locator).with(zombie.graveyard)    .and_return({latitude: 2, longitude: 3})  zombie.geolocateend￼#stubs the method + expectation with correct param + return value

#/spec/models/zombie_spec.rbit "returns properly formatted lat, long" do  Zoogle.stub(:graveyard_locator).with(zombie.graveyard)     .and_return({latitude: 2, longitude: 3})  zombie.geolocate.should == "2, 3"end
```

```ruby
#/app/models/zombie.rbdef geolocate_with_object  loc = Zoogle.graveyard_locator(self.graveyard)  "#{loc.latitude}, #{loc.longitude}"enddef latitude  return 2end
def longitude  return 3 
end
￼￼￼￼￼#/spec/models/zombie_spec.rbit "returns properly formatted lat, long" do  loc = stub(latitude: 2, longitude: 3)  Zoogle.stub(:graveyard_locator).returns(loc)  zombie.geolocate_with_object.should == "2, 3"end
```

###Spies
* 類似 mocks ，一樣製造假物件，一樣是對假物件斷言，但是透過測試工具的功能，而改善了測試程式碼的可讀性，流程更自然（準備、執行、斷言）

[[RSpec] 進階測試系列概念 - Part 6 Mocking V.S. Spying](http://blog.xdite.net/posts/2016/06/11/rspec-advanced-concept-part-6)

Stubs, Mocks and Spies，都是測試的技巧 or 手法!!

[Stubs, Mocks and Spies in RSpec](https://about.futurelearn.com/blog/stubs-mocks-spies-rspec/)  
[了解 Stubs, Mocks, and Spies](https://github.com/festime/stubs-mocks-spies-in-rspec)  
[对 stub 和 mock 的理解](https://ruby-china.org/topics/10977)  
[[RSpec] subject , expect , context, is_expected, be_xxx](http://blog.xdite.net/posts/2016/06/10/rspec-subject-expect-context-is-expected-be)  
[[RSpec] 進階測試系列概念](http://blog.xdite.net/posts/2016/06/11/rspec-advanced-concept-part-0)

###let & subject

```ruby
#subject 是主要要測的物件
#let 則是測試中主要物件時，提供不同的條件
subject (:zombie) { Zombie.new(name:'john') }
let(:axe){ Weapon.new(name:'axe') }
```

`subject`
>Subject blocks allow you to control the initialization of the subject under test. If you don’t have any custom initialization required, then you’re given a default subject method already. All it does is call new on the class you’re testing.

`let`
>Let blocks allow you to provide some input to the subject block that change in various contexts. This way you can simply provide an alternative let block for a given value and not have to duplicate the setup code for the subject over again. Let blocks also work inside of before :each blocks if you need them.

`let`  只有在用到時才會執行  
`let!` 每個測試前都會執行

```ruby
describe User do
  subject(:user){ described_class.new firstname:firstname } #described_class = User
	
  context '' do 
    let(:firstname) {'test'}
  end
end
```

參考文件：  
[subject](http://betterspecs.org/#subject)  
[let](http://betterspecs.org/#let)  
[RSpec 中 let 和 subject 的区别是什么？](https://ruby-china.org/topics/9271)  
[Difference between subject and let #6](https://github.com/reachlocal/rspec-style-guide/issues/6)  
[Dry Up Your Rspec Files With Subject & Let Blocks](http://benscheirman.com/2011/05/dry-up-your-rspec-files-with-subject-let-blocks/)  
[RSpec before vs let](https://www.launchacademy.com/codecabulary/learn-test-driven-development/rspec/before-vs-let)  
[Rails - RSpec - Difference between “let” and “let!”](http://stackoverflow.com/questions/10173097/rails-rspec-difference-between-let-and-let)

###focus
當想要只跑指定的測試時，可以加上 focus

```ruby
  it '#index',  focus:true do
    get :index, format: :json
    expect(response).to have_http_status 200
  end
```

```ruby
rspec --tag focus
```

要忽略特定的測試，可以加上 slow

```ruby
  it '#index',  slow:true do
    get :index, format: :json
    expect(response).to have_http_status 200
  end
```
可以不就在輸入 tag 就會執行

```ruby
￼#spec/spec_helper.rbRSpec.configure do |config|  config.filter_run focus: true  config.run_all_with_everything_filtered = true
    config.filter_run_excluding slow: true  config.run_all_with_everything_filtered = trueend
```

###matchers

* [rspec](https://relishapp.com/rspec/)

###include

可以將常用的包成 `method` include 進來。

`spec/custom_helper.rb`

```ruby
module CustomHelper
  def my_helper_method(success)
    ...
  end
end
```
要記得 reqyire & include 進來

`spec/rails_helper.rb`

```ruby
require_relative 'custom_helper'

RSpec.configure do |config|
  FactoryGirl::SyntaxRunner.send(:include, CustomHelper) 
  #factory girl 也可以引入這個 helper 進來
  
  config.include CustomHelper
end
```

`factories/post.rb`

```ruby
FactoryGirl.define do
  factory :post do
    title { my_helper_method(attr) } #記得要加 {}
  end
end
```

[Define helper methods in a module](https://www.relishapp.com/rspec/rspec-core/docs/helper-methods/define-helper-methods-in-a-module)

###shared_examples_for

```ruby
describe Vampire do   it_behaves_like 'the undead'
   #let(:undead) { Zombie.new }end

shared_examples_for 'the undead' do  #RSpec.shared_examples  it 'does not have a pulse' do    subject.pulse.should == false 
    #undead.pulse.should == false
  endend
```

```ruby
#spec/models/vampire_spec.rb
describe Zombie do  it_behaves_like 'the undead', Zombie.newend

#spec/support/shared_examples_for_undead.rbshared_examples_for 'the undead' do |undead|  it 'does not have a pulse' doundead.pulse.should == false endend
```

```ruby
describe '#fullname' do
  shared_example_for 'a fullname' do |(first, middle, last), output|
    subject(fullname){ User.fullname }
    
    let(:firstname) {first}
    let(:middlename) {middle}
    let(:lastname)   {last}
	
    it { is_expected.to eq output }
  end 
end

  {
	[nil, 'two', 'three'] => 'two three',
	[one, 'two', 'three'] => 'one two three',
	[nil, 'two', nil]     => 'two',
	[one, 'nil', 'three'] => 'one three',
  }.each do |name_set, output|
    it_behaves_like 'a fullname', name_set, output
  end
end
```

參考文件：  
[DRY up your specs using RSpec's shared_examples_for](https://niallburkley.com/blog/rspecs-shared_examples_for/)  
[Correct way to use shared_examples_for](http://stackoverflow.com/questions/11058502/correct-way-to-use-shared-examples-for)  
[Shared context](https://www.relishapp.com/rspec/rspec-core/docs/example-groups/shared-context)  
[Shared examples](https://www.relishapp.com/rspec/rspec-core/docs/example-groups/shared-examples)  
[Shared example group](https://www.relishapp.com/rspec/rspec-core/v/2-0/docs/example-groups/shared-example-group)  
[Relish](https://www.relishapp.com/rspec/rspec-core/docs/example-groups/shared-examples)   
[betterspecs](http://betterspecs.org/#sharedexamples)  
[Single Table Inheritance with Factory Girl in Rails](http://stackoverflow.com/questions/23470209/single-table-inheritance-with-factory-girl-in-rails)

###custom matcher

```ruby
#spec/models/zombie_spec.rb￼describe Zombie do  it 'validates presence of name' do    zombie = Zombie.new(name: nil)    zombie.should validate_presence_of(:name)  endend
```

```ruby
#spec/support/validate_presence_of.rb module ValidatePresenceOf   class Matcher     def initialize(attribute)       @attribute = attribute
       #default message
       @message = "can't be blank"     end
          def matches?(model)
       @model = model
       #主要的測試       model.valid?       model.errors.has_key?(@attribute)
       #collect errors and find a match
       errors = @model.errors[@attribute]       errors.any? { |error| error == @message }     end 
   end   def validate_presence_of(attribute)     Matcher.new(attribute)   end 
   
   def failure_message      "#{@model.class} failed to validate :#{@attribute} presence."   end
      def negative_failure_message      "#{@model.class} validated :#{@attribute} presence."   end
   
   #override failure message & return self
   def with_message(message)     @message = message     self   end
end
```


```ruby
require 'rspec/expectations'

RSpec::Matchers.define :be_a_multiple_of do |expected|
  match do |actual|
    actual % expected == 0
  emd
end

RSpec.describe 9 do
  it { is_expeced.to be_a_multiple_of(3)}
end
```

###expected_to
* is_expected is defined simply as expect(subject) and is designed for when you are using rspec-expectations with its newer expect-based syntax.

[One-liner syntax](https://www.relishapp.com/rspec/rspec-core/docs/subject/one-liner-syntax)


###RSpec Testing for a JSON API
[RSpec Testing for a JSON API](http://blog.ianmiller.nyc/2014/04/18/rspec-testing-for-a-json-api/)  
[Testing Rails jBuilder JSON APIs With RSpec](http://ahimmelstoss.github.io/blog/2014/07/27/testing-a-rails-api-with-rspec/)

###CURL

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

#<span id="factory_girl">Factory Girl</span>

到 `spec/rails_helper.rb` 設定

```ruby
RSpec.configure do |config|
  config.include FactoryGirl::Syntax::Methods
end
```

設定好後原本需要

```ruby
FactoryGirl.create(:user)
```

就不用加前面的 FactoryGirl 直接

```ruby
create(:user)
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

###注意
factory_girl 產生出來的資料，不會透過 controller ，而是直接再 model 產生，因此會跑出 validation 的驗證。

若是希望能跑 controller action 裡的 method 則是要另外跑

```ruby
trait :user_buy do
  after(:create) do |user|
    user.buy
  end
end
```

###為什麼要假物件?
* 無法控制回傳值的外部系統 (例如第三⽅ web service)
* 建構正確的回傳值很⿇煩 (例如得準備很多假資料)
* 可能很慢，拖慢測試速度 (例如耗時的運算)
* 有難以預測的回傳值 (例如亂數⽅法)
* 還沒開始實作 (特別是採⽤ TDD 流程)

##其他設定
`rails g model` 時，一併產生 factory_girl 的檔案在 `spc/factories `

```ruby
config.generators do |g|
  g.test_framework :rspec, :fixture => true, :views => false, :fixture_replacement => :factory_girl
  g.fixture_replacement :factory_girl, :dir => "spec/factories"
end
```
參考文件：  
[Factory Girl](http://www.slideshare.net/gabevanslv/factory-girl-15924188)

#<span id="capybara">Capybara</span>
RSpec除了可以拿來寫單元程式，我們也可以把測試的層級拉高做整合性測試，以Web應用程式來說，就是去自動化瀏覽器的操作，實際去向網站伺服器請求，然後驗證出來的HTML是正確的輸出。

[capybara](https://github.com/jnicklas/capybara)就是一套可以搭配的工具，用來模擬瀏覽器行為  

如果真的需要打開瀏覽器測試，例如需要測試JavaScript和Ajax介面，可以使用 [seleniumhq](http://docs.seleniumhq.org/) 或[ Watir](http://watir.com/) 工具

[RSpec & Capybara 整合測試(Selenium and Poltergeist Driver)](http://mgleon08.github.io/blog/2016/03/30/rspec-capybara-selenium-poltergeist-driver/)

#<span id="ci">CI server</span>
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

#<span id="other">Other</span>

[guard-rspec](https://github.com/guard/guard-rspec)   
`Continuous Testing` 的工具。程式修改完存檔，自動跑對應的測試。

[shoulda-matchers](https://github.com/thoughtbot/shoulda-matchers)   
提供了更多Rails的專屬Matchers

[SimpleCov](https://github.com/colszowka/simplecov)  
用來顯示，測試涵蓋的範圍，可知道哪些地方沒有測試到

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

* [Better Specs](http://betterspecs.org/)  
* [Relish](https://relishapp.com/rspec/)   
* [shoulda](http://matchers.shoulda.io/)  

Gem：  

* [rspec-rails](https://github.com/rspec/rspec-rails)  
* [factory_girl_rails](https://github.com/thoughtbot/factory_girl_rails)  
* [guard-rspec](https://github.com/guard/guard-rspec)  
* [capybara](https://github.com/jnicklas/capybara)  
* [shoulda-matchers](https://github.com/thoughtbot/shoulda-matchers)  
* [SimpleCov](https://github.com/colszowka/simplecov)  
* [database_cleaner](https://github.com/DatabaseCleaner/database_cleaner)  
* [approvals](https://github.com/kytrinyx/approvals)

參考文件： 
 
* [自動化測試](https://ihower.tw/rails4/testing.html)  
* [RSpec-Rails (基礎篇)](http://motion-express.com/trainings/rspec-rails-1)  
* [RSpec-Rails當中自訂methods及helpers](http://motion-express.com/blog/20150320-custom-helpers-in-rspec)  
* [RSpec-Rails 針對module進行unit test](http://motion-express.com/blog/20150327-rspec-rails-testing-module)  
* [RSpec 讓你愛上寫測試](http://www.slideshare.net/ihower/rspec-7394497)  
* [程式設計師升級必練內功：TDD Kata](https://blog.alphacamp.co/2015/03/02/tdd-kata/)  
* [codility 練習](https://codility.com/programmers/lessons/)  
* [保齡球練習](http://www.sportcalculators.com/bowling-score-calculator)  
* [对 stub 和 mock 的理解](https://ruby-china.org/topics/10977)  
* [RSpec 中 let 和 subject 的区别是什么？](https://ruby-china.org/topics/9271)  
* [[RSpec] subject , expect , context, is_expected, be_xxx](http://blog.xdite.net/posts/2016/06/10/rspec-subject-expect-context-is-expected-be)  
* [[RSpec] 進階測試系列概念](http://blog.xdite.net/posts/2016/06/11/rspec-advanced-concept-part-0)  
* [Rails Race Condition Test With RSpec
](http://blog.mz026.rocks/20160821/rails-race-condition-test-rspec)
* [RSpec全套測試環境搭建從零入門](https://www.rails365.net/articles/rspec)

RailsPacific： 
 
* [#RailsPacific - Taming Chaotic Specs - RSpec Design Patterns by Adam Cuppy](https://speakerdeck.com/acuppy/number-railspacific-taming-chaotic-specs-rspec-design-patterns)  
* [Courageous Software](http://randycoulman.com/blog/categories/getting-testy/)  

Rspec-Style-Guide:  

* [rspec-style-guide(reachlocal)](https://github.com/reachlocal/rspec-style-guide)  
* [rspec-style-guide(howaboutwe)](https://github.com/howaboutwe/rspec-style-guide)
