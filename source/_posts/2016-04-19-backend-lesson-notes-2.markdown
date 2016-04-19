---
layout: post
title: "淺談 Backend 課程筆記 2"
date: 2016-04-19 22:29:22 +0800
comments: true
categories: backend
---

課程筆記2

<!-- more -->

#淺談系統安全
* 所謂的安全，都是基於 `Design Flaw（設計錯誤）` 導致的
* 大多都是網路上看別人範例，指理解一半然後再自行創作，導致嚴重安全問題
	* 前人的架構大都經過充分論證，相對上不易有漏洞（ex:https）
* Denfense-in-depth
	* 多種的安全措施，即使一種安全措施被攻陷，還是有其他安全措施能夠將傷害減到最低
	* 心理戰，讓 hacker 需要一直花時間來破解讓他不耐煩
* 沒人會花時間去 hack 系統，除非有錢!!（氣象局 vs 銀行）

#淺談 HTTPS
* HTTPS = HTTP + TLS（前身 SSL）
* 2016年標準，所有網站都應該使用 HTTPS
	* Google 搜尋也會加分
* 能有效防止各式各樣的攻擊，但不要..
	* 使用太舊的加密方法，ex：DES
	* CNNIC 

參考文件：  
[傳輸層安全協議TLS](https://zh.wikipedia.org/wiki/%E5%82%B3%E8%BC%B8%E5%B1%A4%E5%AE%89%E5%85%A8%E5%8D%94%E8%AD%B0)

###HTTPS流程
* 在TCP handshake 後 server 把證書(公開)傳給 client
* Client 收到證書後，根據發行者資訊，再拿出存於 Client 的發行者 public key 對證書驗證
* Client 產生對稱鑰匙，並且以證書中的 public key 進行加密，傳給 Server
	* 對稱式比非對稱式所使用的 CPU 還小 
* Server 使用證書中的 private key 進行解密，拿到 Client 傳來的對稱鑰匙
* 指此 TLS handshaking 已經完成。之後通訊權使用對稱鑰匙加密。其他人則無法。

>現在 https 不安全是因為中國的 CNNIC，所以要將中國的 CNNIC 憑證都刪除!! 中國的銀行也都是用美國的憑證


#Load Balancing
* 有用 Load Balancing 會將 HTTPS 放在 LB，在用 HTTP 和 Application Server 連線

參考文件：  
[負載平衡是分散式資源串連之鑰](http://www.ithome.com.tw/node/81790)

#MITM 中間人攻擊

參考文件：  
[MITM](https://zh.wikipedia.org/wiki/%E4%B8%AD%E9%97%B4%E4%BA%BA%E6%94%BB%E5%87%BB)  
[神不知，鬼不覺，你的帳號密碼是如何被竊聽的？─中間人攻擊〈Man-in-the-middle Attack〉初探](http://mmdays.com/2008/11/10/mitm/)

#One way hashing （H）
* hash 接受任意長度的資料，然後輸出固定長度結果的東西
* One way 的特性，只看 `function output` 和 `function` 本身，不能輕易找出 `funciton input`
* 將用戶密碼進行 H 然後將結果存到資料庫，之後用戶登入，把用戶的password 進行一次 hashing 跟資料庫的 password 比對即可

###Rainbow attack
* 將大量的 string 進行 H，只要這個表夠大，就能夠去比對

[彩虹表Rainbow Table](https://zh.wikipedia.org/wiki/%E5%BD%A9%E8%99%B9%E8%A1%A8) 是壞人建立表時，大幅節省空間的手法

參考文件：  
[Rainbow Hash Table 與密碼破解](http://recover.pixnet.net/blog/post/4535917-rainbow-hash-table-%E8%88%87%E5%AF%86%E7%A2%BC%E7%A0%B4%E8%A7%A3)  
[（總結）密碼破解之王：Ophcrack彩虹表(Rainbow Tables)原理詳解（附：120G彩虹表下載）](http://www.ha97.com/4009.html)

###防止Rainbow attack
* 建立新用戶識，先產生一個隨機（最好使用線上的 random funciton 不要自己來弄）的 string（salt），然後把 H（salt+password） 和 salt 存到資料庫
* 用戶登入，系統就先將 salt 拿出來，再來比對 H（salt+password）和資料庫的值
* 這樣壞人就需要為每一個 salt 建立獨立的一張表，所以 CPU 和 Memory 成本變成本來的 N 倍

#預防 SQL injection
* 記住 parameter escaping 不是絕對安全的方式
* 只有 Parameterization(使用參數或參數標記代替常量值的操作) 才是真正安全
	* 請善用其底層是用 Parameterization 來工作的 ORM，輕鬆又安全

#多層資料庫權限
* 一個好的資料庫，至少有這三種用戶，admin_user, normal_user, readonly_user
* admin_user
	* 資料庫的擁有者，有所有權限
	* 除了要更動 database schema，或是新減資料庫，否則絕不使用
	* 最信任的人員持有
* normal_user
	* 給 Application server 使用
	* 沒有建立刪除資料庫權限
	* 沒有 truncate table 權限
	* 只有 select/insert/update/delete 權限
	* 只有對 stored procedure 執行權限
*  readonly_user
	* 一般不應輕易連上 production database 來除錯，除非是最後手段
	* 別拿 admin_user & normal_user 來除錯

#Audit table
* 對於敏感資料，我們會想記錄其所有的改動
	* 例如想紀錄 user_balances 的改動，便會建立 user_balances_audit 來紀錄
	* 然後在 user_balances 建立 on insert, on update, on delete 的 trigger 把改動自動抄到 user_balances_audit
	* normal_user 不應對 audit tables 有任何權限
	* trigger 擁有者是 admin_user
	* trigger 權限要基於建立者而不是執行者
		* 不同的 RDBMS 具體指令會有分別，但理念相同
		* ex: PostgreSQL 建立 trigger function 時需要 `SECURITY DEFINER` 關鍵字

#SessionId
* 用戶登入流程
	* 產生一組SessionId
	* 把 SessionId->(UserId) 存到 Redis，並且 TTL 設定為 30 分鐘
	* 把 SessionId 傳回用戶
* 之後用戶所有的 request 都會把 SessionId 放在 http authorization header
	* 伺服器利用 SessionId 從 Redis 中找回 Userid 便能知道發出 request 的用戶
	* 然後伺服器再把 Redis 中該 SessionId 重新設定 TTL 為 30 分鐘
*  缺點
	* 所有的 Request 都要先檢查 Redis 去查 SessionId 是否正確
	* 萬一 Redis 當掉，所有用戶都需登入
	* 一旦壞人突破防火牆接觸到 Redis 便能建立受害用戶的 Session，並偽裝成用戶

#JWT
[jwt](https://jwt.io/)  

* Header
 	* alg : 說明 jwt signature 所使用的演算法（建議 RS256/RS512）

```
{
  "alg": "HS256",
  "typ": "JWT"
}
```
* Payload
	* 放什麼都可以，一般至少放上 userid 和 exp(到期時間)
	* jwt 到期時間

```
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```
* Signature 	
	* 把 header, payload 和 secret(HMAC)/private key(RSA) 一起計算出的 SHA256/SHA512
	* 因為 secret(HMAC)/private key(RSA) 只會放在 server, 所以用戶沒法自行產生，用戶偷改 header, payload 其 Signature 也不會吻合

###jwt 流程
* 用戶登入系統，伺服器會檢查用戶密碼是否正確
	* 建立jwt
	* 將 jwt 傳給用戶
* 之後用戶的所有 request，都會將 jwt 放在 http authorization header
	* 伺服器利用 public key 驗證 jwt 是否正確，並檢查 exp 是否過期，在從 payload 中找回 UserId，便能知道發出 request 的用戶
* jwt 快到期前，client side 會發出 renew jwt request，像 server 拿新的 jwt （exp 現在為 30分鐘）
	* 不建議在每個 response 都傳回新的 jwt   

###jwt 好壞
* 好
	* 沒有了 Redis，不用當心 redis 當掉，Session 流失問題
* 壞
	* Client side 要定期更新 jwt
	* 一旦壞人拿到 secret(HMAC)/private key(RSA) 便能偽裝用戶
	* 建議用 RS 演算法，並把 jwt 發行放在獨立機器
	
參考文件：  
[JWT 在前後端分離中的應用與實踐](http://blog.rainy.im/2015/06/10/react-jwt-pretty-good-practice/)  
[OBeyTech JWT - JSON Web Token](http://obeyo-blog.logdown.com/posts/208001)
	
#進階API設計概念
### API 版本控制
* API = 規格，定下來就不要一直更改
* 要進行更改可以利用 v1/v2/v3 版本來去做新的更動，才不會導致錯誤，即使原本的版本有 bug，但對方可能已經自行解決或將 bug 當作 feature
* 因此要先新舊版本 API 並行，確定轉到新版 API 在刪除舊版本的

### middleware
* 用戶丟 `/v1/emailAuth?token={token}`
* 我們可以在用戶和 Application Server 建立一層 middleware ，將 `/v1/emailAuth?token={token}`（GET）轉乘 `/v1/emailAuth` 並且把 token 放進 HTTP header，然後再把 Request 丟給 Application Server
* 一版來說企業客戶會有很多保安要求，為每個客戶在主城市增加額外的 API 只會讓程式越來越亂，這時就應該交給 middleware 解決

###違反 RESTful
* 用 uniform interface 的 API 便會有 `/v1/cats` 和 `/v1/cats/{catId}/catFoods/{catFoolId}`
* 這就會有很多 POST request 造成大量的 network over head
* 改成 `/v1/catAndCatFoods`(post)
	* 雖然違反 uniform interface，但更關鍵的 Stateless protocol 並沒有違反
* 正常系統設計流程
	* 用戶使用流程
	* 討論
	* API 設計
* 完整的 request 是基於 商業邏輯和用戶體驗的 

#Idempotent
* 不管多少次的 resquest ， response 都是一樣的
* POST 是 non-Idempotent
	* 要改成 Idempotent 就將 response 透過 Redis 來處理，後續 response 都會先去 redis 看有沒有相同，有相同就回覆一樣的 response

#Optimistic lock
[樂觀鎖定（Optimistic Locking](http://openhome.cc/Gossip/HibernateGossip/OptimisticLocking.html)

#Stateless protocol

#Long polling
[WebSocket 通訊協定簡介：比較 Polling、Long-Polling 與 Streaming 的運作原理](http://blogger.gtwang.org/2014/01/websocket-protocol.html)

#Asynchronous API

#2PC(Two-phase Commit)
[Two-phase Commit](https://zh.wikipedia.org/wiki/%E4%BA%8C%E9%98%B6%E6%AE%B5%E6%8F%90%E4%BA%A4)

簡報：  
[TritonHo/slides](https://github.com/TritonHo/slides)