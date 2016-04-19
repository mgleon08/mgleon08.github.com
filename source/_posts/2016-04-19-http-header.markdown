---
layout: post
title: "HTTP HEADER"
date: 2016-04-19 22:22:30 +0800
comments: true
categories: rails
---

透過 HTTP HEADER 有很多資訊可以做傳遞

<!-- more -->

#mime_types

MIME(Multipurpose Internet Mail Extensions) 格式

主要用在 HTTP 通訊協定中的請求標頭 Accept 和回應標頭 Content-Type 中，來說明此文件的格式。

Rails 會在 Controller 的 respond_to 方法中辨識並回應所請求的格式樣板，例如瀏覽器請求 application/json 就會回應 json 格式

```ruby
Mime::SET.collect(&:to_s)
=> ["text/html",
 "text/plain",
 "text/javascript",
 "text/css",
 "text/calendar",
 "text/csv",
 "text/vcard",
 "image/png",
 "image/jpeg",
 "image/gif",
 "image/bmp",
 "image/tiff",
 "video/mpeg",
 "application/xml",
 "application/rss+xml",
 "application/atom+xml",
 "application/x-yaml",
 "multipart/form-data",
 "application/x-www-form-urlencoded",
 "application/json",
 "application/pdf",
 "application/zip"]
```

#Accept-Language

可以根據瀏覽器的語言，來切換網站的語言

```ruby
￼class ApplicationController < ActionController::Base￼  protect_from_forgery with: :exception  before_action :set_locale￼  
  protected
  def set_locale    I18n.locale = request.headers['Accept-Language']  endend
```

搭配 `http_accept_language` gem

```ruby
class ApplicationController < ActionController::Base  protect_from_forgery with: :exception  before_action :set_locale
  
  ￼protected  def set_locale￼￼￼    locales = I18n.available_locales    I18n.locale = http_accept_language.compatible_language_from(locales)  end
end
```
#USING THE ACCEPT HEADER

```ruby
#config/routes.rb
require 'api_version'

scope defaults: { format: 'json' } do  scope module: :v1, constraints: ApiVersion.new('v1') do￼￼￼    resources :zombies  end￼￼￼￼  scope module: :v2, constraints: ApiVersion.new('v2', true) do    resources :zombies￼￼￼  end 
end
```
```ruby
#lib/api_version.rb
class ApiVersion  def initialize(version, default=false)    @version, @default = version, default￼￼￼￼￼￼  end
  private  def check_headers(headers)    accept = headers['Accept']  end￼  def matches?(request)    @default || check_headers(request.headers)    accept && accept.include?("application/vnd.apocalypse.#{@version}+json")  endend

application
#payload is application-specificvnd.apocalypse#media type is vendor-specific[.version]
#API version+json#response format should be JSON
```

#Token
```ruby
#app/models/user.rb
class User < ActiveRecord::Base
  before_create :set_auth_token
  
  private
  ￼def set_auth_token    return if auth_token.present?    self.auth_token = generate_auth_token
  end
  
  def generate_auth_token
    loop do      token = SecureRandom.hex
      break token unless self.class.exists?(auth_token: token)    end
￼  end
end
```

```ruby
#app/controllers/posts_controller.rb
class EpisodesController < ApplicationController   before_action :authenticate
   
   ￼protected
   def authenticate     authenticate_token || render_unauthorized
   end
   
   ￼def authenticate_token     authenticate_with_http_token do |token, options|
       User.find_by(auth_token: token)     end
   end
end
```
gem：  
[http_accept_language](https://github.com/iain/http_accept_language)

參考文件：  
[mime types](https://ihower.tw/rails4/environments-and-bundler.html)  
[gem介紹及源碼分析之http_accept_language(四)](http://www.rails365.net/articles/gem-jie-shao-yuan-ma-fen-xi-http-accept-language-si)