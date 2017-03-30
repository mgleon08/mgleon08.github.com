---
layout: post
title: "Cookie & Session"
date: 2016-06-12 20:34:20 +0800
comments: true
categories: rails
---
用於讓 server 記住 client 的行為與資料。

<!-- more -->

#Cookie 

由於 HTTP 協定是無狀態((Stateless)的，server 端無法得知使用者在前端的操作內容，阻礙了前端與後端對話，因此 cookie 就是被設計來解決這個問題的機制。

* 當 server 想要儲存使用者的某些狀態時，就可以發送 cookie 給 client
* http header 裡面其中一個欄位
* key / value 的形式儲
* 檔案大小最多 4Kb
* 以明文的方式在網際網路上傳輸及在 Client 端電腦儲存，所以不建議儲存敏感資料
* 儲存在 client 的瀏覽器
* 儲存位置
	* 記憶體 cookie：由瀏覽器維護，當瀏覽器關閉，cookie也隨之消失，為非持久性的
	* 硬碟 cookie：存在用戶端硬碟中，有一個過期時間，除非使用者手動清理cookie或是超過過期時間，否則cookie不會消失，為持久性的


#Session

* 儲存在 server 端
* session 需要 cookie 的輔助才能產生運作
* server 會傳送帶有 session id 的 cookie 給 client，當下一個 request 及cookie 再被送至 server 端時，會用 cookie 中的 session id 來辨認每個使用者所儲存的狀態與 data。
* 相對於 cookies，session 多用來儲存敏感的資料，也常常成為攻擊的目標 [Session Hijacking](http://guides.rubyonrails.org/security.html#session-hijacking)


#rails 裡的 Session & Cookie 

###session

```ruby
#設定
session[:current_user_id] = user.id
#移除
session[:current_user_id] = nil
session.delete(: current_user_id)
#整個 session 清掉
reset_session
```

###session store
`config/initializers/session_store.rb` 裡面可以設定  

* `:cookie_store`（預設） 
* `:active_record_store` 使用資料庫來儲存（安全性較高）
	* 需搭配 activerecord-session_store gem，產生sessions資料表
* `:mem_cache_store` 使用Memcached快取系統來儲存，適合逾須兼顧效能的高流量網站

> Rails預設採用 Cookies session storage 來儲存 Session 資料，它是將 Session 資料透過 config/secrets.yml 的 secret_key_base 編碼後放到瀏覽器的 Cookie 之中

###cookie

```ruby
#設定
cookies[:commenter_name] = @comment.author
#移除
cookies.delete(:commenter_name)
#保護不能讓使用者亂改 signed
cookies.signed[:user_preference] = @current_user.preferences
#盡可能永遠留在使用者瀏覽器的資料 permanent
cookies[:remember_token] = { value:   remember_token,
                             expires: 20.years.from_now.utc }
#上下相等
cookies.permanent[:remember_me] = [current_user.id, current_user.salt]
#permanent 預設為 20 年後過期 = 20.years.from_now

#可一起用
cookies.permanent.signed[:remember_me] = [current_user.id, current_user.salt]
```

```ruby
#Rails 3 是 digitally signed cookie 可以用這方法 decode
#Rails 4 則改為 Encrypted cookies 就無法
require 'rack'cookie = "BAh7CUkiD3Nlc3Npb25faWQGOgZFRkkiJ(...)"Rack::Session::Cookie::Base64::Marshal.new.decode(cookie)
```

官方文件：  

* [session](http://rails.ruby.tw/action_controller_overview.html#session)  
* [cookie](http://rails.ruby.tw/action_controller_overview.html#cookies)  
[RailsGuide](http://guides.rubyonrails.org/security.html#sessions)  

參考文件：  

* [Cookie 和 Session 的神秘關係](http://blog.andikan.me/2012/10/03/cookie-and-session/)  
* [Action Controller - 控制 HTTP 流程](https://ihower.tw/rails4/actioncontroller.html)  
* [Web 技術中的 Session 是什麼？](http://fred-zone.blogspot.tw/2014/01/web-session.html)  
* [session / cookie 解釋](http://railsfun.tw/t/session-cookie/380)  
* [Rails - Sessions, Cookies and Flash](http://lucaswu.logdown.com/posts/735841-rails-sessions-cookies-and-flash)
* [[译] Rails Sessions 是如何工作的？](http://grantcss.com/blog/2015/03/23/how-rails-sessions-work/)  
* [cookie原理与实现(rails篇)](http://www.rails365.net/articles/cookie-yuan-li-yu-shi-xian-rails-pian)  
* [session原理与实现(rails篇)](http://www.rails365.net/articles/session-yuan-li-yu-shi-xian-rails-pian)  
* [COOKIE與SESSION比較](https://read01.com/NyARK.html)  
* [第 8 章 登录和退出](http://railstutorial-china.org/rails42/chapter8.html#logging-in)
* [详说 Cookie, LocalStorage 与 SessionStorage | 咀嚼之味](http://jerryzou.com/posts/cookie-and-web-storage/)
* [Cache / Session / Cache 簡單說](https://medium.com/@renhades/%E7%B0%A1%E5%96%AE%E8%AA%AA-cache-session-cache-4de6f3c77aa1#.qdbr3vq94)
* [九种浏览器端缓存机制知多少 | ouvenzhang的博客](http://jixianqianduan.com/frontend-javascript/2015/12/28/nine-browser-cache-methods.html)
* [讓我們來談談 CSRF](http://blog.techbridge.cc/2017/02/25/csrf-introduction/)