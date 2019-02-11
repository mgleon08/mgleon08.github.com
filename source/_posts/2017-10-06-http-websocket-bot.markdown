---
layout: post
title: "Http Websocket Bot"
date: 2017-10-06 12:09:51 +0800
comments: true
categories: other
---

簡單的 `Http` `Websocket` `Bot` 介紹

<!-- more -->

* HTTP is hypertext transfer protocol
* HTTP is an application protocol
* HTTP is the foundation of data communication for the World Wide Web.

> HTTP 是一個無狀態（Stateless）的協議，對於事務處理沒有記憶能力，伺服器不知道客戶端是什麼狀態。發送 HTTP 請求之後，伺服器根據請求，會給我們發送數據過來，但是，發送完，不會記錄任何訊息。

* Http 1.0
	* HTTP 的生命週期透過 Request 來界定，也就是一個Request 一個Response，那麼在HTTP1.0中，這次HTTP請求就結束了
* Http 1.1
	* 多出一個keep-alive，在一個HTTP連接中，可以發送多個Request，接收多個Response。
	* 但是 Request = Response ， 在HTTP中永遠是這樣，也就是說一個request只能有一個response。而且這個response也是被動的，不能主動發起。

![](https://upload.wikimedia.org/wikipedia/commons/thumb/d/d5/HTTP_persistent_connection.svg/600px-HTTP_persistent_connection.svg.png)

[如何理解HTTP協議的「無連接，無狀態」特點?](https://read01.com/ePjEn2.html)

### 傳統即時性網頁技術

* Polling 輪詢
  * 瀏覽器每隔一段時間就自動送出一個 HTTP 請求給 server ，獲取最新的網頁資料
  * 在 server 沒有新資料時，瀏覽器也是會自動送出請求，造成網路資源浪費
* Long-Polling 長時間輪詢
  * server 在接收到瀏覽器所送出的 HTTP 請求後， server 會等待一段時間，若在這段時間裡 server 有新的資料，它就會把最新的資料傳回給瀏覽器
  * 如果等待的時間到了之後也沒有新資料的話，就會送一個回應給瀏覽器，告知瀏覽器資料沒有更新
  * 如果在資料更新很頻繁的狀況下，長時間輪詢並不會比傳統的輪詢有效率，而且有時候資料量很大時，會造成連續的 polls 不斷產生，反而會更糟糕。
* Streaming
  * 讓 server 在接收到瀏覽器所送出 HTTP 請求後，立即產生一個回應瀏覽器的連線，並且讓這個連線持續一段時間不要中斷，而 server 在這段時間內如果有新的資料，就可以透過這個連線將資料馬上傳送給瀏覽器。
  * 由於是建立在 HTTP 協定上的一種傳輸機制，所以有可能會因為代理 server（proxy）或防火牆（firewall）將其中的資料存放在緩衝區中，造成資料回應上的延遲，因此許多使用串流的 Comet 實作會在偵測到有代理 server 的狀況時，改用 Long-Polling 的方式處理。

# WebSocket

> WebSocket protocol 定義在 HTML5 標準中的一個新的網頁傳輸協議。

* 是一種建立在單一 TCP 連線上的全雙工（full-duplex）通訊管道，可以讓網頁應用程式與伺服器之間做即時性、雙向的資料傳遞。
* 瀏覽器與伺服器之間若要建立一條 WebSocket 連線，在一開始的交握（handshake）階段中，要先從 HTTP 協定升級為 WebSocket 協定

# EventMachine

> [EventMachine](https://travisliu.gitbooks.io/learn-eventmachine/content/index.html) 是一套事件驅動(event-driven IO) 的框架，基於Reactor Pattern 達到輕量化的併發處理

* Reactor模式
	* 一個處理服務請求的並發程式設計模型。多個服務請求同時發往一個服務句柄(Service Handler)。服務句柄(Service Handler)多路分用到來的請求並把它們同步轉發給相關的請求處理器。


Github：

* [eventmachine](https://github.com/eventmachine/eventmachine) doc [rubydoc](http://www.rubydoc.info/github/eventmachine/eventmachine/index)
* [faye-websocket-ruby](https://github.com/faye/faye-websocket-ruby)
* [em-websocket](https://github.com/igrigorik/em-websocket)
* [Widely-used open source libraries](https://api.slack.com/community)


官網：

* [websocket.org](http://websocket.org/)
* [hubot](https://hubot.github.com/)
* [faye](https://faye.jcoglan.com/)

參考文件：

* WebSocket 介紹
	* [WebSocket 通訊協定簡介：比較 Polling、Long-Polling 與 Streaming 的運作原理](https://blog.gtwang.org/web-development/websocket-protocol/)
	* [WebSocket – 新一代網路傳輸技術](http://www.syscom.com.tw/ePaper_New_Content.aspx?id=368&EPID=194&TableName=sgEPArticle)
	* [WebSocket 是什麼原理？為什麼可以實現持久連接？](https://www.zhihu.com/question/20215561)
* [learn-eventmachine](https://travisliu.gitbooks.io/learn-eventmachine/content/index.html)
* [Building a Slackbot with Ruby and Sinatra](https://www.sitepoint.com/building-a-slackbot-with-ruby-and-sinatra/)
* [EventMachine: scalable non-blocking i/o in ruby](https://zh.scribd.com/document/28253878/EventMachine-scalable-non-blocking-i-o-in-ruby)
* [EventMachine簡介](http://blog.csdn.net/zdq0394123/article/details/7901932)
* [rails + websocket](http://railsfun.tw/t/rails-websocket/498)
* [Using WebSockets on Heroku with Ruby](https://devcenter.heroku.com/articles/ruby-websockets)
* [Getting Started with Ruby and WebSockets](https://blog.engineyard.com/2013/getting-started-with-ruby-and-websockets)
* [Ruby SSE Server 動手做](http://tonytonyjan.net/2015/11/05/concurrent-ruby/)
* [websocket序列文章目錄](https://www.rails365.net/articles/websocket-xu-lie-wen-zhang-mu-lu)
* [EventMachine和多執行序模型](https://ihower.tw/rails4/deployment.html)
* [socket.io搭建多聊天室](http://blog.hugzh.com/2016/01/05/socket.io%E6%90%AD%E5%BB%BA%E5%A4%9A%E8%81%8A%E5%A4%A9%E5%AE%A4/)
* [Using WebSockets on Heroku with Ruby](https://devcenter.heroku.com/articles/ruby-websockets#functionality)
* [BUILDING A SLACK SLASH COMMAND WITH SINATRA, FINCH AND HEROKU](https://wearestac.com/blog/building-a-slack-slash-command-with-sinatra-finch-and-heroku)
* [實現一個 Slack Slash Command](http://tech.deepdevelop.com/shi-xian-ge-slack-slash-command/)
* [Serverless! 使用 AWS 開發 Slack Slash Commands](http://blog.amowu.com/2015/12/serverless-aws-slack-slash-commands.html)

### Video
* [Your First Slack Bot Service (Ruby)](http://code.dblock.org/2016/03/11/your-first-slack-bot-service-video.html)
* [Eventmachine Websocket 實戰](http://www.slideshare.net/ryudoawaru/rt28-29828529)  [(Video)](https://www.youtube.com/watch?v=5X5WEFRTehE)
* [Say Hello To Your First Slackbot (Js)](https://www.youtube.com/watch?v=BWaTYiTbv7Q)

### Hubot
* [Hubot Scripts](https://github.com/hubot-scripts)
* [hubot-scripts](https://github.com/github/hubot-scripts)
* [[心得] Hubot, 一套 bot framework](http://huli.logdown.com/posts/417258-hubot-a-bot-framework)
* [製作一個 Hubot 的噗浪 Adapter](http://blog.frost.tw/posts/2012/03/18/create-a-hubot-plurk-adapter/)
* [在 slack 建立 hubot](http://code.kpman.cc/2016/04/18/%E5%9C%A8-slack-%E5%BB%BA%E7%AB%8B-hubot/#)
* [如何製作 Hubot Script 推上 npm](https://asoul.github.io/2016/04/24/create-hubot-script-and-publish-to-npm/)
* [Hubot 聊天機器人簡單架設教學](https://npes87184.github.io/%E7%A0%94%E7%A9%B6%E9%9B%9C%E8%A8%98/2016/07/08/hubotExample.html)
* [使用Hubot建立屬於自己的機器人 (Build Your Own Robot With Hubot)](https://github.com/twtrubiks/mybot)
* [基於Hubot打造自己的聊天機器人服務](http://www.jianshu.com/p/3ff7a48be02d)

### other
* [INSIDE 有隻硬塞 Bot，聊天、訂閱、搜尋樣樣行！（請勿拍打餵食）](http://www.inside.com.tw/2016/08/26/how-to-build-an-inside-bot)
* [chatfuel](https://chatfuel.com/)


### deploy
* [heroku](https://www.heroku.com/)
* [uptimerobot](http://uptimerobot.com/) 一直戳 heroku 防止 server 進入休眠

```ruby
# create new heroku project
heroku create

# push heroku
git push heroku master

# env config setting
heroku config:add SLACK_API_TOKEN=xxxxxx-xxxxxxxxx-xxxxxxxxxxx

heroku config:add SLACK_API_TOKEN=xxoxb-74720840001-leOhdqKE1mX4hYknfHAeV049

heroku config:remove SLACK_API_TOKEN

# log
heroku logs --tail

# scale
heroku ps:scale web=2
```

### 實做簡易 slack bot

* [simple_slack_bot](https://github.com/mgleon08/simple_slack_bot)
* [simple_line_bot](https://github.com/mgleon08/simple_line_bot)