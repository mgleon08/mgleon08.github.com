---
layout: post
title: "RSpec & Capybara 整合測試(Selenium and poltergeist Driver)"
date: 2016-03-30 00:04:46 +0800
comments: true
categories: rspec gem
---
capybara 是一個可以模擬瀏覽器行為的測試，還蠻好玩的~  
搭配不同 driver，用不同的方式來跑

<!-- more -->

因為是要模擬人真正在使用網頁的狀態，因次每一動，都要定義出來。

#設定
```ruby
#spec/rails_help.rb
require 'capybara/rspec'
require 'capybara/rails'
require 'capybara/poltergeist'

RSpec.configure do |config|
  #要設定為 false ，就會實際存在 database，因此需要 database_cleaner 來清
  config.use_transactional_fixtures = false
  
  #devise controller 設定
  config.include Devise::TestHelpers, :type => :controller
  #devise capybara 設定
  config.include Warden::Test::Helpers
  
  #database_clean 設定 suite -> 整個 rspec
  config.before(:suite) do
    DatabaseCleaner.clean_with :truncation
  end

  config.before(:each) do |example|
    DatabaseCleaner.strategy = example.metadata[:js] ? :truncation : :transaction
    DatabaseCleaner.start
  end
 
  config.after(:each) do |example|
    DatabaseCleaner.clean
  end
  
  #可以設定環境變數，再跑指令時，下 DRIVER=selenium rspec spec/features 才會使用預設 :rack_test 跑，但沒有支援 js 因此才需要用到 selenium(跑瀏覽器)，poltergeist(跑console)
  unless ENV["DRIVER"] == "selenium"
    Capybara.javascript_driver = :poltergeist
  end
  
  Capybara.register_driver :poltergeist do |app|
    Capybara::Poltergeist::Driver.new(app,
      js_errors: true,
      timeout: 15,
      phantomjs_logger: File.new("#{Rails.root}/log/temp.log","w")
    )
  end
end
```


#spec

```ruby
#spec/features/user_spec.rb

require 'rails_helper'

describe 'home page' do
  before do
    @user  = create(:user, :admin)
    # mock CSRF 
    allow_any_instance_of(ActionController::Base).to receive(:protect_against_forgery?).and_return(true)
  end

  it 'User is admin can login and see data', :js => true do
    visit '/backend'#後台首頁
    fill_in 'email', :with => 'admin@gmail.com' #輸入email
    fill_in 'password', :with => 'password' #輸入密碼
    click_button 'submit' #點擊登入
    expect(page).to have_content('後台') #到後台看到後台的字
  end

  it 'logout', :js=> true do
    login_as(@user) #登入
    visit '/backend/posts'  #在內頁
    find("#logout").click #點擊登出，記得要下 id
    expect(current_path).to eq('/') #回到首頁
    expect(page).to have_content('首頁') #頁面上也首頁的字
  end
end
```



gem：  
[capybara](https://github.com/jnicklas/capybara)   
[selenium](https://github.com/SeleniumHQ/selenium)  
[poltergeist](https://github.com/teampoltergeist/poltergeist)  
[database_cleaner](https://github.com/DatabaseCleaner/database_cleaner)
  
參考文件：  
[Test CSRF in feature test using Capybara](http://blog.tomoyukikashiro.me/post/test-csrf-in-feature-test-using-capybara/)        
[Capybara (and Selenium) with RSpec & Rails 3: quick tutorial](http://www.opinionatedprogrammer.com/2011/02/capybara-and-selenium-with-rspec-and-rails-3/)  
[Devise: How To: Test with Capybara](https://github.com/plataformatec/devise/wiki/How-To:-Test-with-Capybara)  
[Difference between truncation, transaction and deletion database strategies](http://stackoverflow.com/questions/10904996/difference-between-truncation-transaction-and-deletion-database-strategies)