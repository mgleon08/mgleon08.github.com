---
layout: post
title: "例外處理 rescue exception"
date: 2016-02-04 21:51:38 +0800
comments: true
categories: rails
---

在 rails 當中，當發生例外時就會爆錯，畫面就會不見。  
但有時我們並不希望讓它這樣，因此可以用 rescue 才處理掉這些例外發生時，該執行的動作。

<!-- more -->

#例外處理

```ruby
begin
  # 有可能發生例外的處理動作
rescue => e
  # 例外發生時的處理措施
ensure
  # 無論有沒有發生例外，這一段都一定會執行
end
```

#分開處理例外

可以給予不同例外，執行不同動作

順序應為最特殊為第一位，以此類推  
若要在最後包含所有例外，可以使用rescue Exception

```ruby
begin
  # 有可能發生例外的處理動作
rescue ArgumentError => e
  # 例外發生時的處理措施
rescue TypeError => e
  # 例外發生時的處理措施
rescue Exception => e
  # 例外發生時的處理措施
end
```
如果沒有指定變數，例外物件會自動存放在：`$!`及`$@`變數中  
`$!`：最後發生例外的物件  
`$@`：呈現最後例外所發生的位置和資計


#重來

例外發生後，再重新執行一次

```ruby
begin
  # 有可能發生例外的處理動作
rescue => e
  retry #重新再跑
end
```

#例外語法的簡化

如果例外 begin & end 的範圍剛好就是整個方法的範圍，就可以省略。

```ruby
def rescue
  #有可能發生例外的處理動作
rescue
  #例外發生時的處理措施
ensure
  #無論是否發生例外都會執行
end

```

#自行產生例外

```ruby
def test
  raise StandardError, "test error"
  #丟例外出來 raise(例外名稱, 例外訊息)
rescue => e
  binding.pry
end
```

#rescue from 
可以透過 rescue from 將所有例外集中處理，這樣就不用每個地方都要 rescue

```ruby
class ApplicationController < ActionController::Base
  rescue_from User::NotAuthorized, :with => :deny_access # self defined exception
  rescue_from ActiveRecord::RecordInvalid, :with => :show_errors

  rescue_from 'MyAppError::Base' do |exception|
    render :xml => exception, :status => 500
  end

  protected
    def deny_access
      ...
    end

    def show_errors(exception)
      exception.record.new_record? ? ...
    end
end
```

rescue_from 通常是用在像是某些第三方函式庫，會丟出一些例外，必須來處理

例如在 pundit 這個檢查權限的套件，如果發生權限不夠的情況，會丟出Pundit::NotAuthorizedError 的例外，這時候就可以捕捉這個例外，改成回到首頁：

```ruby
rescue_from Pundit::NotAuthorizedError, with: :user_not_authorized

protected

def user_not_authorized
 flash[:alert] = I18n.t(:user_not_authorized)
 redirect_to(request.referrer || root_path)
end
```

>不要做 rescue_from Exception 或 rescue_from StandardError，除非有很好的理由。因為這會帶來嚴重的副作用（譬如無法得知異常的細節、無法在開發時追蹤 Backtrace）

[rescue-from](http://rails.ruby.tw/action_controller_overview.html#rescue-from)  
[Action Controller - 控制 HTTP 流程](https://ihower.tw/rails4/actioncontroller.html)  
[Rescuable](http://edgeapi.rubyonrails.org/classes/ActiveSupport/Rescuable/ClassMethods.html)  
[Understanding Ruby and Rails: Rescuable and rescue_from](https://simonecarletti.com/blog/2009/12/inside-ruby-on-rails-rescuable-and-rescue_from/)  
[Rails' rescue_from](http://www.rubytutorial.io/rails-rescue_from/)  

#自訂錯誤頁面

可以使用 Controller 與 View 來自己客製化錯誤處理的版面。首先定義顯示錯誤頁面的路由。

config/application.rb

```ruby
config.exceptions_app = self.routes
```

config/routes.rb

```ruby
match '/404', via: :all, to: 'errors#not_found'
match '/422', via: :all, to: 'errors#unprocessable_entity'
match '/500', via: :all, to: 'errors#server_error'
```

* 建立 Controller 與 View。

app/controllers/errors_controller.rb

```ruby
class ErrorsController < ActionController::Base
  layout 'error'
 
  def not_found
    render status: :not_found
  end
 
  def unprocessable_entity
    render status: :unprocessable_entity
  end
 
  def server_error
    render status: :server_error
  end
end
```

app/views

```ruby
errors/
  not_found.html.erb
  unprocessable_entity.html.erb
  server_error.html.erb
layouts/
  error.html.erb
```

參考文件：  
[Ruby - Chapter 09 例外處理(exception)](http://blog.xuite.net/yschu/wretch/104912690-Ruby+-+Chapter+09+%E4%BE%8B%E5%A4%96%E8%99%95%E7%90%86(exception))  
[Ruby學習筆記(8) – 錯誤與例外處理](http://blog.tonycube.com/2011/07/ruby8.html)