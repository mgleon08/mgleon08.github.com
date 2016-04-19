---
layout: post
title: "淺談 Backend 課程筆記 3"
date: 2016-04-19 22:29:24 +0800
comments: true
categories: backend
---

課程筆記3

<!-- more -->

#理想的 Backend 系統
###越少模組越好
* 需要監控和管理的模組變越少
* 團隊要掌握的語言/技術變越少

###通用 vs 專精
ex : 是否使用ElasticSearch?

* 需要 tokenization 嗎？(和民居食屋，若打民居是否要顯示出來?)
* 能不能先在 RDBMS 做 search?

###安全 vs 快速開發
ex : 應該把呼叫 GCM 放到獨立的 worker?

* 多一個 worker，多一個 worker 要管，也要建立 MQ server

###沒有 Single Point of Failure

所謂「SPOF」是指當某個零件故障會造成整個系統無法正當運作，那麼這個零件就是整個系統中
的 Single Points Of Failure。

* 每一系統模組(System Component)，都應該要有 standby instance (備用實例)
	* 但 standby instance 只能保護單一隨機性的災難
* 一旦監控系統發現某一模組當掉，就把它停掉，並用 standby instance 取代
* 一旦某模組當掉，你的設計應該讓系統停在 Consistent 狀態
	* 除非為了效能，而故意犧牲 Consistency

/ping/ 回傳的是 204 無法發現有什麼問題，正確是回傳 200

###對 Surge 有抵抗性
* 動態加開 AS 會有幫助，但加開的速度不一定能追上流量
* READ 可以用 Cache 支撐，但 Write 不行
* 吃大量資源的 end point 可以考慮轉用 async 模式
* 別再 AS 上執行 async API 的工作，會害 AS 當掉

###不需要即時人員支援
* 停用 Load Balancer 下，一台 AS 當掉便自動停用，只讓系統損失 1/n 的運算能力
* 監察系統發現 Master RDBMS 當掉後，應該懂得自動把 Slave DB 提升成 Master 並進行運算
* 交給 Job Farm 的工作，在本來的 Worker Instance 當掉後，會自動由其他 Worker 重做

#Backend 架構

###DNS
* 對大型網站，會有多個 data center 的，DNS 會回答該地區最近的 data center 的 IP
* 對大型網站，一個 domain name 可能對應多個 IP
	* 單一 load balancer 有其物理極限的(頻寬/CPU)，因此要多個 load balancer

###Load Balancer
* 通常 LB 會把 HTTPS 轉成 HTTP ，然後才把 Request 轉給 AS
* 高階的 LB 能根據 url(甚至是 Request body) 做 routing
	*  AWS 沒有(陽春的LoadBalance)，HAproxy 有，所以可以在 EC2 上架 HAproxy，就能依 url、request body 做 routing，但是AWS ELB的網路流量比EC2好很多
* AS 架構沒問題的話，round robin （輪詢） 跟 least conn （最少連線數） 沒什麼差別
* 如果 AS 不是 Stateless
	* source 只能在非流動用戶上(手機ip會改) 
	* sticky 需要 LB 解讀 Request 並抽取像 SessionId 的變數，然後根據之前歷史派到特定 AS 上。對 LB 有極高性能要求。

>Stateless（Client與Server的溝通不需依賴狀態，每一個 Request 必須包含所有需要的資訊，而不需依賴其他 Request 的狀態。）

參考文件：  
[Stateful與Stateless](https://dotblogs.com.tw/jimmyyu/2010/10/16/difference-between-stateful-and-stateless)  
[SQL Server 負載平衡架構介紹(Load Balancing)](http://caryhsu.blogspot.tw/2011/03/sql-server-load-balancing.htmll)

###Application Server
* 好的 AS 應該是 Stateless
	*  這樣可以是流量加開 AS
	*  LB 便不用昂貴的 sticky mode 了
* AS中 ，不應跑會吃大量資源的工作
	*  LB才能使用 round robin / random 模式
	*  AS 才不會凍結掉 

###Long Polling Server (LPS)
* 如果要將 message 推送到手機，請使用 GCM/APNS ，別自行架 Server
* 除非系統流量很大，用 AS 來處理 Long Polling 變好
* 使用獨立 LPS 的好處
	* 你能專精負責 long polling 語言
	* 不用擔心 AS execution worker pool 被用光

參考文件：  
[WebSocket 通訊協定簡介：比較 Polling、Long-Polling 與 Streaming 的運作原理](http://blogger.gtwang.org/2014/01/websocket-protocol.html)  

###Main DB
* 別輕易使用 noSQL （noSQL 沒有 transation）
* RAID 只能保護單一 SSD 故障
* Master/Slave Replication 保護不了壞人刪除數據
* 只有離線儲存的備份數據，才真正安全
* 效能，數據安全性，低成本 三者最多只能要兩種

###Cache cluster
* 對小型系統，單獨一台大記憶體的機器做 Caching 也許比較方便
* 對大型系統，單獨一台機器做 Caching 不合成本效益，所以要用多台機器
* 要知道物件放在哪台機器上，最簡單方法 MD5(key) mod n
* 為 Cache cluster 增加/刪減機器時，小心別引發 total cache miss

### Hot Data DB
* 跟 cache 是不同的
* 暫時性的，丟失了也死不了的數據
* 在為了效能犧牲 Consistency 的情況下，要延後寫入 main DB 的數據
* 一般來說 Hot Data 數據量不會太多，所以不需要 Clustering
* 建議做 Replication 去保護資料

###Search DB
* 沒錢別額外架 Search DB
* 數據儲存結構有特別為搜尋做特化
* 有 tokenization 和 auto correction 等等幫助搜尋的重要功能
* 很多時候 Search DB 的數據和 Main DB 的數據會有時間差

###Report DB
* 專門的 Report DB，很可能採用 denormalized schema
	* 需要高度專業性
	* 需要很多很多的錢
* Random Sampling 是否能解決問題

###File DB
* 有人說 File 應該以 BLOB 物件被放到 Main DB，但是..
	* 一般建立後極少被改動
	* RDBMS 一般用上系統中最高級的 SSD，而 File 一般不需要這種效能
* 延伸思考，在檔案名字中額外加上 MD5，能解決很多 File caching 問題 

###Message Queue
* Message
	* 會裝有 Json/XML 格式的工作內容
	* 會有 MessageId
* Producer
	* 訊息生產者，一般來說是 AS 要建立工作
	* 一個 Queue 能有多於一個 Producer 
* Consumer
	* 訊息消耗者，一般來說是 Worker 要執行工作
	* Queue 中沒有 Message 時，Consumer 會被 blocking，不佔用 CPU
	* 一個 Queue 能有多於一個 Consumer
	* 一份訊息，在同一時間內只會讓一個 Consumer 收到
* 有些 MQ 不保證絕對性的 FIFO 和 no-duplicate，使用上請注意(ex:Amazon SQS)
* 如果應用較簡單，Redia 某程度上也能當做 MQ 使用
* 沒有人和 message 佔用 CPU 為 0，因為 worker 都被 blocked

###Worker Farm
* 一個 Message 同一時間只會一個 worker 收到
* 可以視 Queue 中的剩餘 Message 量，動態決定增減 Worker Server

###Cron job worker
* 跟async task worker 有點不同，只在預定時間睡醒，把工作做完便死亡
* 為了不要深夜撲火，應該在多於一台機器上執行
	* 為了一份工作不被多個 worker 執行，需要 execution control 機制
	* Database-as-IPC 是很嚴重的 anti-pattren，但如果有很少量的 data exchange 還算可以接受
* 理想的 Cron job 即使當掉，也無需 data cleanup，單純重跑即可
* 同時間多個 Worker 執行時，也懂的自動分工，不會出錯 

簡報：  
[TritonHo/slides](https://github.com/TritonHo/slides)