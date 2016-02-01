---
layout: post
title: "Ruby on Rails - 簡易搜尋功能"
date: 2015-12-18 16:01:02 +0800
comments: true
categories: rails gem
---

在架設網站得時候，不免常常需要用到搜尋功能（增加使用者體驗)
所以在這邊介紹一個簡易的搜尋功能

<!-- more -->

並不需要額外加 router，直接先在 view 裡面加上

```ruby
<%= form_tag books_path, :method => :get do %>
	<%= text_field_tag "keyword", nil, placeholder: '請輸入關鍵字...', :class=>"form-control"%>
	<%= submit_tag "Search" %>
<% end %>
```

之後在 controller 加上

```ruby
if params[:keyword]
	@books = Book.where( [ "name like ? or content like", "%#{params[:keyword]}%", "%#{params[:keyword]}%"] )
end
```

用SQL的語法 `LIKE`，直接在資料庫裡面找尋相關的關鍵字，如果有多個欄位，再加上 `OR` 即可，其中 `name` 和 `content` 就是要搜尋的欄位。

記得前面後面要加上 `%` ，如果沒加上，就一定要完全跟輸入的關鍵字一樣。
加上去之後代表，前面後面都可以加上任意字組，就等於是說只有欄位裡有這個關鍵字就搜尋出來的意思。

`% (百分比符號)：代表零個、一個、或數個字母。`

[SQL萬用字元](http://www.1keydata.com/tw/sql/sql-wildcard.html)

以上是比較簡易的方式來達成，相對的效能上可能也沒那麼好

#進階搜尋
如果要更加複雜，效能更好或是全文搜尋的話
可以用 gem 來取代

* [ransack](https://github.com/activerecord-hackery/ransack)
比較不考慮效能

* [Solr](https://github.com/outoftime/sunspot)
[在 Rails 中使用 Solr 做全文搜尋](http://gogojimmy.net/2012/01/25/full-text-search-in-rails-with-solr/)


* [Elasticsearch](https://www.elastic.co/)
[elasticsearch-rails](https://github.com/elastic/elasticsearch-rails)
[第三方gem: Searchkick](https://github.com/ankane/searchkick)

不過目前都還沒用過啊..改天要用到再來研究一下:D