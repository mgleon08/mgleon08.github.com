---
layout: post
title: "Web server / Application server / Rack / Process / Thread
"
date: 2016-10-23 16:05:22 +0800
comments: true
categories: server other
---

這幾個名詞之前一直搞不太清楚，所以透過這邊來順便理解一下。

<!-- more -->

#Web Server

> 專門只處理 HTTP request 與 response，當收到 HTTP request 之後，需要business logic 的部分就從 application server 取，最後把 result 轉為 HTTP response

* Apache
* Nginx

###Proxy Server(代理伺服器)

主要的工作是去各個 Web Server 抓取資料回來放在伺服器上來供用戶讀取下載，如此一來可以大幅減少到各個 Web server 抓取資料的時間。

###Reverse Proxy Server(反向代理伺服器)

和 Proxy Server 剛好相反，負責將用戶端的資料傳送（HTTP）給藏在 Reverse Proxy Server 後面的 Web Server，這些躲在後面的 Web Server 不會、也不能直接被用戶直接連結，只能經由 Reverse Proxy Server 代理傳送和接收資料，如此不僅可以保護後方 Web Server 被攻擊，同時還可提供負載平衡、快取以及資料加密的功能。

#Application Serve

> 專門用來處理 business logic 的，較常見的用法是接受 web server 的 request，執行完business logic (過程中視需要去access DB tier)之後把 result 回給 web server

* WEBrick
* Passenger
* Unicorn
	* `single-threaded multi-process`
	* ` process monitoring`
	* 能讓所有處理序都監聽同一個共享的socket，而不是每個處理序使用單獨的socket
* Puma
	* `purely multi-threaded`
* Thin
	* `evented I/O`
	* `no process monitoring`
	* 它的集群節點沒有處理序監控，所以需要去監控處理序是否崩潰。每個處理序監聽各自的socket，不像 Unicorn 一樣共享 socket
* Rainbows `multi-threaded`

#Rack

> 處理一個抽象過的HTTP請求和響應。而這個 HTTP 抽象層就稱之為Rack。

* 所有的 Ruby app server (ex: Unicorn, puma, Passenger) 都實現了Rack介面, 因此能夠與所有 Ruby web frameworkm，像 Rails，Sinatra 等之間進行切換

![](http://ohcoder.com/assets/raptor/rack.jpg)

官方文件：

* [Rack: a Ruby Webserver Interface](http://rack.github.io/)

參考文件：

* [為什麼我們需要 Rack ?](https://ruby-china.org/topics/21517)
* [rack介紹與原理](https://www.rails365.net/articles/rack-jie-shao-yu-yuan-li)
* [rack與中間件](https://www.rails365.net/articles/rack-yu-zhong-jian-jian)
* [Rails on Rack](http://rails.ruby.tw/rails_on_rack.html)
* [Advanced Rack](http://gabebw.com/blog/2015/08/10/advanced-rack)

Video：

* [Rebuilding a Ruby web server](https://vimeo.com/user12143456/review/69109140/c72efbd052)

#Process, Threaded, Evented

###多處理序阻塞I/O（Muti-process bloking I/O）

* 一個 process 處理一個 client
* concurrency 透過構造多個 process來實現
* 當對方沒有數據發送，那麼 read 操作就會被阻塞，如果對方接收數據太慢，那麼 write 操作也會被阻塞


優點：

* 工作原理非常簡單
* 不會有 therad safe

缺點：

* process 很吃記憶體，因此不太適合用來做 I/O concurrency

###多執行緒阻塞I/O(Muti-threaded blocking I/O)


* 一個 process 有多個 threaded，每個 threaded 都可以處理 client
* concurrency 透過構造小數量的 process，每一個 process 包含有多個 threaded 來實現

優點：

* concurrency 比較不吃記憶體

缺點：

* 要注意 therad safe

###事件I/O（Evented I/O）

* 使用單一 process 和 therad
* 絕對不會被阻塞，類似 javascript 的 callback
* 持續監聽 Evented I/O，當對方沒有數據發送，或是對方接收數據太慢，I/O 呼叫只返回特定的錯誤訊息，以避免阻塞

> thread safe: 一個 process 有多個 thread 在跑，因為變數是共享的，所以可能造成對一個變數，做重複的操作，導致錯誤的結果

* [Ruby 中的多进程与多线程](http://mp.weixin.qq.com/s?__biz=MzI0NjIzNDkwOA==&mid=2247483676&idx=1&sn=1df45612132f3f96037b04d62f72d0cf&scene=0)
* [Ruby 中的多進程與多線程](https://read01.com/GoNKk4.html)
* [Thread-Safe的理解與分析](http://aftcast.pixnet.net/blog/post/23786004-thread-safe%E7%9A%84%E7%90%86%E8%A7%A3%E8%88%87%E5%88%86%E6%9E%90)
* [Program/Process/Thread 差異](https://medium.com/@totoroLiu/program-process-thread-%E5%B7%AE%E7%95%B0-4a360c7345e5)
* [Java 面試 01 - Program、Process 和 Thread](https://blog.marksylee.com/2016/09/13/java-interview-01-program-process-thread/)
* [Program,Process,Thread](https://programming.im.ncnu.edu.tw/J_Chapter9.htm)
* [[請益] 要如何讓人搞懂Process與Thread](https://www.ptt.cc/bbs/Soft_Job/M.1406625146.A.971.html)

#GIL/GVL

> MRI裡有個東西叫全局解釋器鎖(global interpreter lock)。這個鎖環繞著Ruby程式碼的執行。即是說在一個多執行緒的上下文中，在任何時候只有一個執行緒可以執行Ruby程式碼。 因此，假如一台8核機器上跑著8個執行緒，在特定的時間點上也只有一個執行緒和一個核心在忙碌。GIL一直保護著Ruby內核，以免競爭條件造成數據混亂。把警告和優化放一邊

* [無人知曉的 GIL](https://ruby-china.org/topics/28415)
* [Python的GIL是什麼鬼，多執行緒性能究竟如何](http://cenalulu.github.io/python/gil-in-python/)
* [python 執行緒，GIL 和 ctypes](http://zhuoqiang.me/python-thread-gil-and-ctypes.html)

參考文件：

* [How we've made Phusion Passenger 5 (「Raptor」) up to 4x faster than Unicorn, up to 2x faster than Puma, Torquebox](http://www.rubyraptor.org/how-we-made-raptor-up-to-4x-faster-than-unicorn-and-up-to-2x-faster-than-puma-torquebox/)    [(中譯)](http://ohcoder.com/blog/2014/11/11/raptor-part-1/)
* [Web server / Application server 傻傻分不清楚 ？](http://michaelhsu.tw/2013/07/04/server/)
* [Ruby on Rails 伺服器的選擇](http://blog.chh.tw/posts/ruby-on-rails-server-options/)
* [App server, Web server: What's the difference?](http://www.javaworld.com/article/2077354/learn-java/app-server-web-server-what-s-the-difference.html)
* [A Web Server vs. An App Server](http://www.justinweiss.com/articles/a-web-server-vs-an-app-server/)
* [Ruby on Rails Server options [closed]](http://stackoverflow.com/questions/4113299/ruby-on-rails-server-options) [Ruby 伺服器對比](https://ruby-china.org/topics/25276)
* [Ruby 的多執行緒應用伺服器介紹](https://ruby-china.org/topics/10832)
* [大戰 Rails Connection Leak](http://blog.mz026.rocks/20160917/rails-connection-leak)
* [Rails 部署工具，原來是這樣](https://5xruby.tw/posts/rails-deploy)
* [What is a Reverse Proxy vs. Load Balancer?](https://www.nginx.com/resources/glossary/reverse-proxy-vs-load-balancer/)
