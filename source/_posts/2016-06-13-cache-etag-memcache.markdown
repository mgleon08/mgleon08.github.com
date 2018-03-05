---
layout: post
title: "Cache Etag Memcache"
date: 2016-06-13 23:22:01 +0800
comments: true
categories: rails
---

當有資料不常去做更動，就能夠用 cache 去讀取增加速度!

<!-- more -->

* cache 處太多，程式會變複雜，增加維護的難度
* cache 會增加除錯難度，資料不再只有唯一的資料庫版本
* cache 如果沒寫好，可能會產生資料不一致的Bug、時間顯示相關的Bug(例如顯示資料的時間雖然時間不會變，但是如果是要顯示多少小時以前，就會變動了)等等
* cache 增加了寫程式的難度，像是Expire過期資料、資料的安全性(放在cache 層的資料也需要被保護注意安全)
* 會增加撰寫UI的難度，因為 cache 相關的程式可能會混在樣本中

#Cache Store

* 預設的 memory_store 只適合單機開發，重啟 Rails cache 資料就不見了。
* 正式上線的網站會推薦使用 `Memcached`
* 它是一套Name-Value Pair(NVP)分散式記憶體快取系統，當你有多個Rails伺服器的時候，也可以很方便的共用快取資料。


#Etag

ETag 是用來辨識當前的頁面是否有所改變（Rails 預設就有使用了）

第一次

1. Client 發 Request
2. Rails 流程
	* 1. Render body
	* 2. Create ETag
	* 3. Body & ETag included in response(`headers['ETag'] = Digest::MD5.hexdigest(body)`)
3. Rails 發 Response
	* 200 Success Head 裡包含 ￼headers['ETag']
4. Client 接收並 caches response

第二次

1. Client 發 Request 並帶著 `headers['If-None-Match']` 就是 Etag 但不知道為什麼要叫這名字
2. Rails 流程
	* 1. Render body
	* 2. Create ETag
	* 3. Compare ETag
	* 4. If ETags match then body not included in response
3. Rails 發 Response
	* 304 Not Modified
4. Client 接收並 reads response from cache

以上雖然就不需要再重新 response body，但是每次 request 都必須再把整個 body 去 generate 一次 ETag 這是相當沒有效率的事

所以可以透過 Customer Etag 來改善

```ruby
class ItemsController < ApplicationController
  etag {current_user.id}

  def show
    @item = Item.find(params[:id])
￼    fresh_when(@item)
	#headers['ETag'] = Digest::MD5.hexdigest(@item.cache_key)
	#加上面的 etag {current_user.id} 等於 fresh_when([@item, current_user.id])，可以有更多參數去判斷是否有改變
  end
end
```

這樣一來就只要 id 和 update_at 兩個就可以知道到底整個頁面有沒有東西變動過


#memcache store uses dalli

Dalli is a high performance pure Ruby client for accessing memcached servers

1. Approximately 20% faster than memcache-client.
2. Provides proper failover with recovery and adjustable timeouts.
3. Easier to integrate with monitoring tools like NewRelic RPM in order to track usage. 4. ThreadSafe.
5. Backwards compatible with the previous memcache-client API.


* 安裝Memcached

```ruby
brew install memcached
```

* 加上memcached的函式庫

```ruby
gem "actionpack-action_caching"
gem "dalli"
```

編輯`config/environments/development.rb`和`production.rb`加上

```ruby
config.action_controller.perform_caching = true
config.cache_store = :dalli_store
```

```ruby
# 存入
Rails.cache.write('first', Post.first)
# 取出
Rails.cache.read('first')
# 删除
Rails.cache.delete('first')
# 查看
Rails.cache.exist?('first')
# fetch: 檢查 key 是否存在, 不存在, 把 block 里的结果作為 value, 并返回结果. 如果存在, 就不去執行 block 的内容
Rails.cache.fetch('last_post') { Post.last }
# => blah
# 还可以设定过期时间
Rails.cache.fetch('time', expires_in: 4.seconds) { Time.now }
```

### 使用時機

1.資料庫裡不常變動的資料，ex:所有國家

使用

```ruby
def cached_skills
  Rails.cache.fetch([self.class.name, id, :skills], expires_in: 240.hours) {
    skills.to_a
  }
end
```

* [self.class.name, id, :skills]這是存儲的關鍵
* cache 將會在240小時後過期
* 如果不轉 array (to_a)直接將 active-record 的 relations 儲存到 cache 的話，每次訪問 cache 這個 relations 還是會查詢資料庫的


2.某個每天需要更新的 cache 資料

```ruby
# Use a different cache store in production.
config.cache_store = :dalli_store, { value_max_bytes: 3100000 }
```

```ruby
class TestController < ApplicationController
  skip_before_action :authenticate_user!
  caches_action :index, cache_path: -> { request.domain }, expires_in: 1.day

  def index
    render template: "test/index.json.jbuilder"
  end

  def clear_cache
    Rails.cache.delete_matched("_cache_page.json")

    render json: "Cache page has been deleted."
  end
end

```

### 參考文件

Gem

* [dalli](https://github.com/petergoldstein/dalli)
* [actionpack-action_caching](https://github.com/rails/actionpack-action_caching)


官方文件

* [Rails 缓存简介](http://guides.ruby-china.org/caching_with_rails.html)
* [Caching with Rails: An overview](http://guides.rubyonrails.org/caching_with_rails.html)
* [memcached](https://memcached.org/)
* [caching_with_rails](https://doc.bccnsoft.com/docs/rails-guides-4.1-cn/caching_with_rails.html)

參考文件：

* [快取](https://ihower.tw/rails4/caching.html)
* [Google Developers: HTTP 快取](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching)
* [數據緩存提升性能gem - Dalli介紹](http://grantcss.com/blog/2014/12/04/introduce-dalli/)
* [如何使用 memcached 做快取](https://ihower.tw/blog/archives/1768)
* [Rails中使用memcached內存緩存技術](https://danielzhangqinglong.github.io/2015/03/19/rails-caching/)
* [译 ~ 介绍Rails中的条件HTTP缓存](https://danielzhangqinglong.github.io/2015/04/03/intro-http-caching-with-rails/)
* [Rails 3 和 Rails 4 中 ETags 工作原理](https://ruby-china.org/topics/24996)
* [淺談memory cache](http://enginechang.logdown.com/posts/249025-discussion-on-memory-cache)
* [总结 Web 应用中常用的各种 Cache](https://ruby-china.org/topics/19389)
* [Rails Cache](http://rails-everyday.group.iteye.com/group/wiki/1160)
* [Speed Up Your Rails App by 66% - The Complete Guide to Rails Caching](https://www.nateberkopec.com/2015/07/15/the-complete-guide-to-rails-caching.html)
* [Accelerating Rails, Part 1: Built-in Caching](https://www.fastly.com/blog/accelerating-rails-part-1-built-caching)
* [Accelerating Rails, Part 2: Dynamic HTTP Caching](https://www.fastly.com/blog/accelerating-rails-part-2-dynamic-http-caching)
