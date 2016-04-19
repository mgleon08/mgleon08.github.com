---
layout: post
title: "淺談 Backend 課程筆記 1"
date: 2016-04-19 22:29:19 +0800
comments: true
categories: backend
---

課程筆記1

<!-- more -->

#淺談MVC
* M - 系統中各種抽象物件，一改動會主動告知view 
* V - 把 model 顯示給用戶看，有控制項，可將用戶行動傳送給 controller
* C - 根據用戶的行動，去改變model
* 
#M+V+C!= MVC
* 在MVC中，model 改動後需要以 Observer Pattern 去主動告知 view 更新; 但在HTTP中，極少進行 Server Push，而是由 Controller 回應給 View
* MVC 針對的是單機軟體溝通（不需網路，用 function call 即可），而現今都是以 HTTP  RESTful來去溝通
* 而且每家 framework 的 MVC 都不盡相同

>不要提自己是用MVC，只要說有架構分開


#淺談 Backend 基本概念
* backend 目前最注重 `throughput(系統總容量，處理能力)`，每一個 request 處理速度相對的比較不重要 
>當 `execution time` 壓到某一個程度，`network latency` 會變成主因 
* `throughput(系統總容量)` = `concurrent execution` / `execution time per task`
* 對 Application Server 來說，增加機台便能無痛增加 `throughput`
* 而 `RDBMS` 只有單一的 `Master` 通常會成為系統效能瓶頸

>所以不要說什麼語言慢，而是要去增加機台!!

#Strong Type vs Weakly Type
* ST 多花時間在 `coding`，但是少花時間在 `testing`
* 對老手說 WT 可能會比較快一點點

>所以看要多花時間在 `coding` 還是在 `testing`  
>真正吐血的是在 Race Condition/Design Flaw

#Good Code
* 降低 `bug rate`
* 主動將有危險的 code 做註記

#CRUD 到 backend framework
[backdemo](https://github.com/TritonHo/demo)
###問題
* 密碼放在程式碼中 
* `URL Routing`資訊雜亂
	*要知道 /v1/cats/(GET)，必須知道 `main.go`，再細看 `cat.go`
* 要自己編寫簡單的SQL
* 輸入處理和商業邏輯混在一起
* Partial Update 的輸入處理冗長
* 輸入檢查散落在 `Create` 和 `Update`
* 輸出時，大量的重複的程式碼
* Handler 共用 `global variable`

>總結 DRY 將同樣邏輯的封裝在一起

###精簡程式
* 引入 `mux library` 支援以 Regular Expression 做 routing
* 建立 `lib/config` 用來讀取環境變數，把密碼，用戶名稱都放到環境變數中
* 20/80 80% automate ，集中精力解決剩下的 20%
* 使用 ORM 也好，還是要好好學 SQL（用 ORM 解決簡單的，用 SQL 來解決複雜的)
* 使用了 ORM 之後改動 table schema ，只要改動 Model Class
* 建立 `lib/httputil` 來做處理輸入
* 使用 reflection 來支持 Partial Update 並且跟 ORM 整合
* 簡單的 single attribute checking 能放到 struct tag，然後在 input binding 檢查
* 理想的 handler 應該只有 business login，輸入輸出都跟它沒關係
	* 只會返回 statusCode, error/output object
	* middleware 接受以上資料在做 HTTP Response
* handler 需要的是 transaction 不是 db
* 乾脆將 db 丟到 middleware，然後以 parameter 方式將 transaction 丟給 Handler，即使 Handler 遇上問題，也不用自己Rollback，因為在 middleware 那層解決
* 將每個 endpoint，應該要抽出來變成 procedure

###authorization
* 除非有特殊原因，auth token 請放在 `Http Authorization Header`
* 別把 user password 存到資料庫

###Application Server 中主要的模組
* Router
* Middleware
* Auth
* Input Binder
* Basic Validater
* Transaction Manager
* Database connector/ORM
* Output Utlity
* MQ connector
* Logger
* File Manager

#others
###Connection Pool
Connection Pool(連線池) 是一種資料庫連線管理的機制，它介於應用程式與資料庫之間；
集中管理資料庫的連線，能有效提升應用程式存取資料庫的效能及減少連線的錯誤。

* get 到 tomcat，後會先去檢查已經建好的 Connection Pool 有哪個 Connection 是有空的，就將 request 丟過去
* 為什麼 Connection Pool 要用 `global variable`
	* 生長週期是整個server
* 一個 application 都有一個 Connection Pool

使用Connection Pool的好處有:  

1. Pool會keep住與DB的連線。程式需要使用時跟pool要即可。
不用再重複地跟DB建立連線然後又釋放掉。
2. 可設定與DB最大的連線數，避免超過DB所能負擔的連線數。
3. Pool可幫忙驗證Connectin是否還正常，若不正常時，便再與DB建立好的正常連線，
確保程式取得的Connection都是正常可使用的。
4. 額外功能的提供。如幫忙檢查Connection State或幫忙關閉Statement等。
不同的Connection Pool其額外提供的功能當然也會有所不同。

參考文件：  
[連接池(Connection Pool)介紹](http://peggg327.blogspot.tw/2014/11/connection-pool.html)


###reflection
* Reflection的作用：得知自己的外觀，甚至自我修改與複製
* 簡單的來說，能讓程式在 runtime 時知道一個 `structure object` 資訊
	* 例如連帶有什麼 methods 和 attributes
* 雖然會讓效能變慢，但很多 library 都不能不用 
* 除非有特殊原因，一般不會在 handler 中使用 reflection (如果不是在寫 library 不該使用 reflection)

參考文件：  
[Reflection增加了程式的彈性](http://www.ithome.com.tw/node/57227)  
[第 16 章 反射（Reflection）](https://github.com/JustinSDK/JavaSE6Tutorial/blob/master/docs/CH16.md)

簡報：  
[TritonHo/slides](https://github.com/TritonHo/slides)