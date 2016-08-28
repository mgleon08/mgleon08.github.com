---
layout: post
title: "例外處理 rescue exception, Error Handling, Custom Error Pages"
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


* [rescue-from](http://rails.ruby.tw/action_controller_overview.html#rescue-from)  
* [Action Controller - 控制 HTTP 流程](https://ihower.tw/rails4/actioncontroller.html)  
* [Rescuable](http://edgeapi.rubyonrails.org/classes/ActiveSupport/Rescuable/ClassMethods.html)  
* [Understanding Ruby and Rails: Rescuable and rescue_from](https://simonecarletti.com/blog/2009/12/inside-ruby-on-rails-rescuable-and-rescue_from/)  
* [Rails' rescue_from](http://www.rubytutorial.io/rails-rescue_from/)  

#自訂錯誤類型

```ruby
module Errors
  class WhatError < StandardError
  end
end
```

```ruby
raise Errors::WhatError, 'Message'
```

```ruby
rescue Errors::WhatError => e
```

* [Where to define custom error types in Ruby and/or Rails?](http://stackoverflow.com/questions/5200842/where-to-define-custom-error-types-in-ruby-and-or-rails)  
* [Raise custom Exception with arguments](http://stackoverflow.com/questions/11636874/raise-custom-exception-with-arguments)  
* [RAISING AND RESCUING CUSTOM ERRORS IN RAILS](https://wearestac.com/blog/raising-and-rescuing-custom-errors-in-rails)

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

###Dynamic Error Pages

* [Dynamic Rails Error Pages](https://mattbrictson.com/dynamic-rails-error-pages)  
* [How to redirect to a 404 in Rails?](http://stackoverflow.com/questions/2385799/how-to-redirect-to-a-404-in-rails)  
* [Custom 404 error page with Rails 4](http://thepugautomatic.com/2014/08/404-with-rails-4/)  
* [DYNAMIC ERROR PAGES IN RAILS](https://wearestac.com/blog/dynamic-error-pages-in-rails)  
* [Jutsu #12: Custom Error Pages in Rails 4+](https://samurails.com/jutsu/rails-jutsu/jutsu-12-custom-error-pages-in-rails-4/)  
* [Creating Custom Error Pages With Rails](https://solidfoundationwebdev.com/blog/posts/creating-custom-error-pages-with-rails)

#Don't use rescue Exception => e

```ruby
begin
  do_something()
rescue => e
  puts e # e is an exception object containing info about the error. 
end
```
```ruby
begin
  do_something()
rescue ActiveRecord::RecordNotFound => e
  puts e # Only rescues RecordNotFound exceptions, or classes that inherit from RecordNotFound
end
```

###Don't do this.

```ruby
begin
  do_something()
rescue Exception => e
  # Don't do this. This will swallow every single exception. Nothing gets past it. 
end
```

###why

因為

```ruby
#會將所有的 exception 給 rescue 包括，ruby 自行產生的
rescue Exception => e

#只抓自己專案裡面的 error 都是繼承自 StandardError
rescue => e
等同於
rescue StandardError => e
```

###The Exception Tree
```ruby
Exception
 NoMemoryError
 ScriptError
   LoadError
   NotImplementedError
   SyntaxError
 SignalException
   Interrupt
 StandardError
   ArgumentError
   IOError
     EOFError
   IndexError
   LocalJumpError
   NameError
     NoMethodError
   RangeError
     FloatDomainError
   RegexpError
   RuntimeError
   SecurityError
   SystemCallError
   SystemStackError
   ThreadError
   TypeError
   ZeroDivisionError
 SystemExit
 fatal
```

* [Ruby's Exception vs StandardError: What's the difference?](http://blog.honeybadger.io/ruby-exception-vs-standarderror-whats-the-difference/)

#Raise four step

###Step 1:Call #exception to get the exception

`raise` 主要是會去執行 `self.class` 的 `#exception` 因此可以覆蓋過去

```ruby
def raise(error_class_or_obj, message, backtrace) 
	error = error_class_or_obj.exception(message)# ...end
```

```ruby
require 'net/http'

class Net::HTTPInternalServerError
  def exception(message="Internal server error")
    RuntimeError.new(message)
  end
end

class Net::HTTPNotFound
  def exception(message="哈囉")
    RuntimeError.new(message)
  end
end

response = Net::HTTP.get_response(
  URI.parse("http://avdi.org/notexist"))

if response.code.to_i >= 400
  raise response
end
```

###Step 2: #set_backtrace

追蹤錯誤來源

>in order to get a stack trace which includes the current line, you must call #caller passing 0 for the “start” parameter 

```ruby
def foo
  puts "#caller: "
  puts *caller
  puts "----------"
  puts "#caller(0)"
  puts *caller(0)
end
def bar
  foo
end
bar

##caller:
#3.set_back_trace.rb:9:in `bar'
#3.set_back_trace.rb:11:in `<main>'
#----------
##caller(0)
#3.set_back_trace.rb:6:in `foo'
#3.set_back_trace.rb:9:in `bar'
#3.set_back_trace.rb:11:in `<main>'
```
###Step 3: Set the global exception variable

`$ERROR_INFO` alias for `$!`

```ruby
require 'English'
puts $!.inspect

begin
  raise "Oops"
rescue
  puts $!.inspect
  puts $ERROR_INFO.inspect
end
puts $!.inspect
```

[English](http://ruby-doc.org/stdlib-2.0.0/libdoc/English/rdoc/English.html)

###Step 4: Raise the exception up the call stack

Limitations on exception matchers

```ruby
def errors_with_message(pattern)
  #Generate an anonymous "matcher module" with a custom threequals
  m = Module.new
  (class << m; self; end).instance_eval do
    define_method(:===) do |e|
      pattern === e.message
    end
  end
  m
end

puts "About to raise"
begin
  raise "Timeout while reading from socket"
rescue errors_with_message(/socket/)
  puts "Ignoring socket error"
end
puts "Continuing..."
```
A custom exception matcher.

```ruby
def errors_matching(&block)
  m = Module.new
  (class << m; self; end).instance_eval do
    define_method(:===, &block)
  end
  m
end

class RetryableError < StandardError
  attr_reader :num_tries
  def initialize(message, num_tries)
    @num_tries = num_tries
    super("#{message} (##{num_tries})")
  end
end

puts "About to raise"

begin
  raise RetryableError.new("Connection timeout", 2)
rescue errors_matching{|e| e.num_tries < 3} => e
  puts "Ignoring #{e.message}"
end
puts "Continuing..."
```

官方文件：  
[Exception](http://ruby-doc.org/core-2.2.0/Exception.html)

參考文件：  

* [Ruby - Chapter 09 例外處理(exception)](http://blog.xuite.net/yschu/wretch/104912690-Ruby+-+Chapter+09+%E4%BE%8B%E5%A4%96%E8%99%95%E7%90%86(exception))  
* [Ruby學習筆記(8) – 錯誤與例外處理](http://blog.tonycube.com/2011/07/ruby8.html)  
* [Error Handling in Rails — The Modular Way](https://medium.com/rails-ember-beyond/error-handling-in-rails-the-modular-way-9afcddd2fe1b#.nbq2qjk4a)