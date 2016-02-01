---
layout: post
title: "Ruby on Rails - helper?partial?yield?"
date: 2015-12-21 11:08:08 +0800
comments: true
categories: rails
---

在 Rails 中 View 是負責放 html 的地方，因此會盡量讓它單純呈現畫面，邏輯的東西則是放在別的地方。
但是有時候還是不免會有許多重複的 html ，或是判斷和邏輯的東西必須擺放。
因此 rails 就有提供了幾個方法可以解決這些問題。

<!-- more -->

#helper
Helper 主要是來整理 view 中，包含邏輯的部份，指的是可以在 Template 中使用的輔助方法。

像是

* link_to：可以轉換成，HTML 的 `<a>` 標籤
* image_tag：可以轉換成，HTML 的 `<img>` 標籤
* simple_format：可以將內容中 `\n` 換行字元換成HTML的 `<br>` 標籤
* truncate：可以將過長的內容，指定擷取前幾個字元，後面則變成 ...
* strip_tags：移除HTML標籤

以上這些都是內建好的一些 helper
當然我們也可以自訂自己的 helper 出來

###範例

判斷現在登入的使用者，是否為此篇文章的使用者，是的話才顯示刪除按鈕。

```ruby
# 如果指定用户是當前用户，返回 true
def current_user?(user)
	user == current_user
end
```
這樣在 view 中就可以

```ruby
<% if current_user?(@post.user) %>
	<%= link_to "delete", post_path, method: :delete, data: { confirm: "You sure?" } %>
<% end %>
```

>注意 helper 檔案，會預設跟 controller 和 view 一樣的名稱，但是並沒有限制只有該名稱的 view 才能使用，而是所有 view 都能使用。controller 則無法使用。

若是希望 controller 可以使用，可以在 controller 檔案加上 `include PostsHelper`

application_controller.rb

```ruby
class ApplicationController < ActionController::Base
  protect_from_forgery with: :exception
  include PostsHelper
end
```

或是在 controller 加上 `view_context`

```ruby
class PostsController
  def show
    @post =  view_context.truncate(@post.desc, :lenght => 50 )
  end
end
```

最後是 helper 也可以放 html 進去
只要加上 `content_tag`

```ruby
content_tag(:p, "Hello world!")
 # => <p>Hello world!</p>
content_tag(:div, content_tag(:p, "Hello world!"), class: "strong")
 # => <div class="strong"><p>Hello world!</p></div>
content_tag(:div, "Hello world!", class: ["strong", "highlight"])
 # => <div class="strong highlight">Hello world!</div>
content_tag("select", options, multiple: true)
 # => <select multiple="multiple">...options...</select>

<%= content_tag :div, class: "strong" do -%>
  Hello world!
<% end -%>
 # => <div class="strong">Hello world!</div>
```

[content_tag](http://apidock.com/rails/ActionView/Helpers/TagHelper/content_tag)

#partial(局部樣板)
partial 主要是來整理 view 中，重複出現的部分。

###範例

_post_list.html.erb

```ruby
<% @posts.each do |post| %>
  <li><%= post.id %></li>
  <li>Title: <%= link_to(post.title, post_path(post)) %></li>
  <li>Content: <%= post.content %></li>
<% end %>
```
記得照 rails 的慣例，partial 檔案前面要加上 `_`

之後再 view 中只要 `render` 指定的位置，就可以了

index.html.erb

```ruby
<%= render 'post_list' %>
```

###collection partial

另外一種是直接傳遞參數進去的 collection partial，上述可改成

_post_list.html.erb

```ruby
<li><%= post.id %></li>
<li>Title: <%= link_to(post.title, post_path(post)) %></li>
<li>Content: <%= post.content %></li>
```
index.html.erb

```ruby
<ul><%= render :partial => "post_list", :collection => @posts, :as => :post %></ul>
```
or

```ruby
<% @posts.each do |p|
  <%= render :partial => "post_list", :locals => { :post => p } %>
<% end %>
```

將參數直接丟進去，就不用在 view 裡面包 block

#yield

yield 主要是會被替換成樣板的地方。
通常是使用在 layout 裡面的 `application.html.erb`。
會將上下板固定，而中間有 `<%= yield %>` 的地方，就是顯示其他所有的 html.erb 檔案的內容

好處是可以將網站的版型固定，只在需要出現內容的地方用 yield 引進來就可以了。

另外的作用是像是，網站標題，或是fb的Open Graph設定等等，都可以使用這個方式。

###網站標題

先在 `helper` 設定

```ruby
  def full_title(page_title = '')
    base_title = "Ruby on Rails"
    if page_title.empty?
      base_title
    else
      page_title + " | " + base_title
    end
  end
```

接著在 `application.html.erb`

```ruby
 <title><%= full_title(yield(:title)) %></title>
```

之後就可以在每個想呈現不同標題的地方加上

```ruby
<% provide(:title, "About") %>
# 也可以改 <% content_for(:title, "About") %>
```

### Facebook Open Graph

在 `application.html.erb`

```ruby
<%= yield :head %>
```

再到要加 Open Graph 設定的頁面加上

```ruby
<%= content_for :head do %>
    <%= tag(:meta, :content => @post.name, :property => "og:title") %>
    <%= tag(:meta, :content => truncate(@post.about, :length => 150 ), :property => "og:description") %>
    <%= tag(:meta, :content => "post", :property => "og:type") %>
    <%= tag(:meta, :content => post_url(@post), :property => "og:url") %>
<% end %>
```

總結

* partial 負責經常性重複的東西，或是比較大片HTML的東西。
* helper 負責處理跟邏輯判斷有關的東西。
* yield 負責替換樣板的東西。

>建議在 Helper 與 Controller 的 code 不要互相混來呼叫來呼叫去。
>讓 View 歸 View，Controller 歸 Controller。
>若真有業務上的需求需要「到處都可以用」。建議寫 Module 掛在 lib 用 mixin 技巧混入。


官方文件：
[Guides](http://guides.rubyonrails.org/layouts_and_rendering.html#structuring-layouts)
[Guides 中文](http://rails.ruby.tw/layouts_and_rendering.html#%E7%B5%84%E7%B9%94%E7%89%88%E5%9E%8B)

APIdock：
[partial](http://apidock.com/rails/ActionView/Partials)
[helper](http://apidock.com/rails/ActionController/Helpers)

參考資料：
[Ruby on Rails 實戰聖經](https://ihower.tw/rails4/actionview.html)
[rails102](https://rocodev.gitbooks.io/rails-102/content/chapter1-mvc/v/what-is-view.html)
[如何運用 / 設計 Rails Helper (1)](http://blog.xdite.net/posts/2011/12/09/how-to-design-helpers)
[如何運用 / 設計 Rails Helper (2)](http://blog.xdite.net/posts/2011/12/10/how-to-design-helpers-2)
[如何運用 / 設計 Rails Helper (3)](http://blog.xdite.net/posts/2012/01/16/how-to-design-helper-3)