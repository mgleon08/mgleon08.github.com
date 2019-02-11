---
layout: post
title: "簡單理解 CDN 原理"
date: 2018-10-29 15:06:53 +0800
comments: true
categories: cdn other
---

近期遇到 CDN 的問題，順便瞭解一下 CDN 整個運作模式

<!-- more -->

原先以為 CDN 會自動做快取直到我們去 purge CDN，但實際上 CDN 是會根據 `Response Headers` 的

```
cache-control: max-age=60, public
```

```ruby
# rails 中設定 max-age 的方式
expires_in 1.minutes, public: true # public
expires_in 5.minutes # private
```

去做對應的 cache (必須用 public)，另外做了 `max-age` 之後，不只 CDN 會有設定，使用者的瀏覽器也會有設定

```
server -> CDN -> Browser
60         60       60
```

因此如果設定 `max-age` 60 秒

* CDN cache 60 秒，在這 60 秒就不會再打 server，直到 60秒過 or `purge CDN`
* Browser cache 60 秒，在這 60 秒就不會再打 CDN，直到 60秒過 or `clear browser cache`

> 這邊有個重點，當測試的時候，必須開不同的 browser 去做測試，不然直接打 CDN 跟 HTML 打 API 的 url cache 會同步

# cache-control

> HTTP 1.1 才出現 Cache-Control

接著再根據 cache-control 瞭解一下行為，當 browser 設定

### max-age

```ruby
cache-control: max-age=60, public
```

* 第一次訪問，200
* 第二次訪問時，則會顯示 200(from cache)，後來 google 有更新，就有分 `from disk cache(磁盤緩存)` 和 `from memory cache(內存緩存)`

```ruby
Status Code: 200  (from disk cache)
# 記得必須是用 code 去呼叫才會，不是直接打 api，之前測試直接打會變成 304，改用 script 成功了
```

* 但什麼時候是哪種緩存，目前看起來是會根據 google 來判斷 [求教大神瀏覽器是根據什麼決定from disk cache 與 from memory cache？](https://www.zhihu.com/question/64201378)
* 如果有 expires(HTTP1.0) 也存在的話，max-age 會蓋過 expires

> 「public」和「private」
如果回應標記為「public」，即使具備關聯的 HTTP 認證，甚至回應狀態碼無法正常快取，回應也可以供使用者快取。在大多數情況下，「public」並不是必要項目，因為明確的快取資訊 (例如「max-age」) 已表示 回應可供快取。

> 另一方面，瀏覽器可以快取「private」回應，但是通常只開放給單一使用者快取，因此不允許任何中繼快取對其進行快取，例如使用者瀏覽器可以快取包含使用者私人資訊的 HTML 網頁，但是 CDN 不能快取。
>[HTTP 快取](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching?hl=zh-tw)

### no-cache

當設定以下 `no-cache`，代表每次都會發送 Request 去確認是否有新的檔案，沒有就會  return `304`

```ruby
cache-control: no-cache
# 相當於 cache-control: max-age=0
```

### no-store

當設定以下 `no-store`，代表完全不使用快取

```ruby
cache-control: no-store
```

# Purge CDN

通常廠商會提供一隻 API，可以做 purge CDN 的動作，當資料有更新時就可以透過這支 API 把 CDN 清除，讓它重新跟 server 拉取，但有個重點是，`max-age` 就不能夠設定太長，太長會導致 brower 那邊一直 cache 舊的，即使我們 purge 掉 CDN，brower 也是會等到 `max-age` 時間到，才會重新去拉 CDN。

# params cache object

一開始 url 本來是希望帶 params 的方式做 cache，在測試的時候發現，雖然可以做 cache，但在 purge 的時候卻無法清除，看 log 發現顯示 `none cache-existing`
代表根本沒有清掉 cache，詢問了 cdn 的廠商，原來要有帶 params 的 url，當作是 unique object 算是特殊需求，要另外開啟。

但這家開啟後，萬一之後要換別家，會不會別家沒有支援或是要另外開啟? 所以後來還是改用 `path_params` 的方式，以防換別家的時候又會有問題

# File Extension

CDN cache 是根據整個 url，包括副檔名，因此 `http://localhost:3000/test.js` 和 `http://localhost:3000/test.json` 會是不同的 url，purge 時也必須指定對應的 url 才能夠 purge

# http/https

purge 的 url 必須要打一開始設定的 url，`http/https` 會有差別，如果設定的是 `http` 就必須針對 `http` 去做 purge 而不是 `https`

參考文件:

* [HTTP 快取](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching?hl=zh-tw)
* [HTTP Caching](https://cythilya.github.io/2018/07/27/http-caching/?fbclid=IwAR1jJfxG2o25i06Pqueb8yE0copC4VSJpbXmr4lG76wPTcDQzGa7ncE6iTk)
* [循序漸進理解 HTTP Cache 機制](https://blog.techbridge.cc/2017/06/17/cache-introduction/)
* [你應該知道的瀏覽器緩存知識](https://excaliburhan.com/post/things-you-should-know-about-browser-cache.html)
