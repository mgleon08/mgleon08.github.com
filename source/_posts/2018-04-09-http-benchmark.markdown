---
layout: post
title: "HTTP benchmark 工具 Wrk"
date: 2018-04-09 17:09:37 +0800
comments: true
categories: http
---

wrk 可以預先模擬多人來網站時，效能會如何!

<!-- more -->

這類的工具有很多，大概如下

* 複雜
	* jmeter
	* LoadRunner
* 簡單
	* wrk	
	* ab

那這次主要介紹是 wrk

# 安裝

```ruby
brew install wrk
```

# Command

```ruby
# 跟服務器建立並保持的TCP連接數量 
-c, --connections: total number of HTTP connections to keep open with
                   each thread handling N = connections/threads
# 壓測時間
-d, --duration:    duration of the test, e.g. 2s, 2m, 2h

# 使用多少個線程進行壓測   
-t, --threads:     total number of threads to use

# 指定Lua腳本路徑
-s, --script:      LuaJIT script, see SCRIPTING

# 為每一個HTTP請求添加HTTP頭
-H, --header:      HTTP header to add to request, e.g. "User-Agent: wrk"
# 在壓測結束後，打印延遲統計信息
    --latency:     print detailed latency statistics
# 超時時間
    --timeout:     record a timeout if a response is not received within
                   this amount of time.
```


# Exmple

```ruby
wrk -t12 -c400 -d30s -T30s --latency http://localhost:3000
# -t12 用 12 個線程
# -c400 模擬 400 個併發連接
# -d30s 持續 30 秒
# -T30s 設定超過 30 秒就算連接超時 
# --latency 響應時間的分佈情況
```

```ruby
Running 30s test @ http://localhost:3000
  4 threads and 1000 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.38ms    1.09ms  84.91ms   91.92%
    Req/Sec     3.62k   555.58     4.48k    81.00%
  Latency Distribution
     50%    1.22ms
     75%    1.69ms
     90%    2.27ms
     99%    3.93ms
  108206 requests in 30.08s, 118.88MB read
  Socket errors: connect 0, read 883, write 1, timeout 0
Requests/sec:   3597.39
Transfer/sec:      3.95MB
```

* `Latency`: 響應時間
* `Req/Sec`: 每個線程每秒鐘的完成的請求數
* `Avg`: 平均值
* `Stdev (Standard Deviation)`: 即標準偏差，是統計學的一個名詞，這裡表示請求響應時間的離散程度，值越大代表請求響應時間的差距越大，系統的響應約不穩定。
* `Max`: 最大值
* `+/- Stdev`: 正負一個標準差佔比
* `Latency Distribution`: 50% 在 1.22ms 以內完成 / 99% 在 3.93ms 以內完成
* `Socket errors`: 分為 `連接錯誤`, `讀取錯誤`, `寫入錯誤`, `超時錯誤`
* `Requests/sec`: 每秒請求數量，也就是並發能力

# 腳本 scroipt 測試複雜場景

wrk 支援 `lua` 的腳本

先建立一個 `test.lua` 的 file

```lua
-- example HTTP POST script which demonstrates setting the
-- HTTP method, body, and adding a header

wrk.method = "POST"
wrk.body   = "foo=bar&baz=quux"
wrk.headers["Content-Type"] = "application/x-www-form-urlencoded"
```

執行

```ruby
wrk -t12 -c100 -d30s -T30s --script=post.lua --latency http://localhost:3000 
```

### wrk 接受的屬性

```ruby
local wrk = {  
   scheme  = "http",  
   host    = "localhost",  
   port    = nil,  
   method  = "GET",  
   path    = "/",  
   headers = {},  
   body    = nil,  
   thread  = nil,  
}  
```
### wrk 提供的 hook 函數

* setup 函數 

這個函數在目標 IP 地址已經解析完, 並且所有 thread 已經生成, 但是還沒有開始時被調用. 每個線程執行一次這個函數. 
可以通過thread:get(name),  thread:set(name, value)設置線程級別的變量. 

* init 函數 

每次請求發送之前被調用. 
可以接受 wrk 命令行的額外參數. 通過 -- 指定. 

* delay函數 

這個函數返回一個數值, 在這次請求執行完以後延遲多長時間執行下一個請求. 可以對應 thinking time 的場景. 

* request函數 

通過這個函數可以每次請求之前修改本次請求的屬性. 返回一個字符串. 這個函數要慎用, 會影響測試端性能. 

* response

每次請求返回以後被調用，可以根據響應內容做特殊處理，比如遇到特殊響應停止執行測試，或輸出到控制台等等。

```lua
-- 實現每個請求前會有隨機的延遲
-- example script that demonstrates adding a random
-- 10-50ms delay before each request
function delay()
   return math.random(10, 50)
end
```

```lua
-- 每個線程要先進行認證，認證之後獲取token以進行壓測
token = nil
path  = "/authenticate"

request = function()
   return wrk.format("GET", path)
end

response = function(status, headers, body)
   if not token and status == 200 then
      token = headers["X-Token"]
      path  = "/resource"
      wrk.headers["X-Token"] = token
   end
end
```

```lua
-- 壓測支持HTTP pipeline的服務
-- 通過在init方法中將三個HTTP request請求拼接在一起，實現每次發送三個請求，以使用HTTP pipeline。
init = function(args)
   local r = {}
   r[1] = wrk.format(nil, "/?foo")
   r[2] = wrk.format(nil, "/?bar")
   r[3] = wrk.format(nil, "/?baz")

   req = table.concat(r)
end

request = function()
   return req
end
```

* [更多 scripts 範本](https://github.com/wg/wrk/tree/master/scripts)

參考文件：

* [wrk git](https://github.com/wg/wrk)
* [使用 wrk 來測試 HTTP 性能](https://www.restran.net/2016/09/27/wrk-http-benchmark/)
* [wrk -- 小巧輕盈的 http 性能測試工具](http://zjumty.iteye.com/blog/2221040)
* [使用ab和wrk對OSS進行benchmark測試](https://yq.aliyun.com/articles/35251)
* [性能測試應該怎麼做？](https://coolshell.cn/articles/17381.html)
* [Http壓測工具wrk使用指南](http://zhaox.github.io/benchmark/2016/12/28/wrk-guidelines)
* [更多 scripts 範本](https://github.com/wg/wrk/tree/master/scripts)