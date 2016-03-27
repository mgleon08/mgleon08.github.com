---
layout: post
title: "Lib vs Service Object"
date: 2016-03-26 09:31:32 +0800
comments: true
categories: rails
---

大家都知道，fat models, skinny controllers  
但要將 code 放在哪邊，才會比較好維護?這就有很多方式了  

<!-- more -->

#/lib:
* It is not coupled to my app's domain models.
* It can be reused on other projects.
* It can potentially become its own gem. Thus, putting it in lib/ is the first step in that direction.

簡單的來說，`/lib` 放的比較像是，很多 project 都可以重複使用，並沒有專屬某個 project，並且可以包成一個 gem 給大家使用。

舉例: `FacebookAuth` `GithubAuth`
 
#/app/services:
* They tend to know a decent amount about the inner workings of domain models.
* Perform work that is specific to business domain in my app.
* Tend to be coupled to specific models.

而 `/app/services` 比較像是，專屬於某個 `project`，並且與 business logic 息息相關，就像是某個服務一樣。

舉例: `UserAuthenticator` `開立發票` `寄密碼提醒信` 

#Service Object使用時機

* The action is complex (e.g. closing the books at the end of an accounting period)
* The action reaches across multiple models (e.g. an e-commerce purchase using Order, CreditCard and Customer objects)
* The action interacts with an external service (e.g. posting to social networks)
* The action is not a core concern of the underlying model (e.g. sweeping up outdated data after a certain time period).
* There are multiple ways of performing the action (e.g. authenticating with an access token or password). This is the Gang of Four Strategy pattern.

大致上就是

1. 邏輯複雜
2. 牽扯到很多 model 關係 (ex: 兩個 model 的資料運算)
3. 與外部服務相關 (ex: slack 通知)
4. 與核心無關 (ex: 定時清理過期數據)
5. 會經常重複使用


###約定

* 一個Service Object 只做一件事。
* 每個Service Object 一個文件，統一放在app/services 目錄下。
* 命名採用動作，比如SignEstimate ，而不是EstimateSigner 。
* instance級別實現兩個接口，initialize負責傳入所有依賴，call負責調用。
* class級別實現一個接口call，用於簡單的實例化Service Object然後調用call 。
* call的返回值默認為true/false，也可以有更複雜的形式，比如StatusObject 。
 
>以上約定主要都是看個人習慣，也有人會放在 `app/controller` 底下  
[A Way to Organize POROs in Rails](http://vrybas.github.io/blog/2014/08/15/a-way-to-organize-poros-in-rails/)


#concern vs service object

* concern
 - 簡單說就是，有許多 model 有共用的邏輯片段，可以拆出來 	
* service object
 - 與 concern 不同的地方是，存放的是比較完成的 code，簡單多你丟什麼給它，他就會丟相對應的資料給你。

* 兩個搭配使用
 - 將許多 service object 搬到 concern

參考文件：  
[什麼時候使用Concerns，什麼時候使用Services？](https://ruby-china.org/topics/18401)  
[Service Object 整理和小結](https://ruby-china.org/topics/23892)  
[Gourmet Service Objects](http://brewhouse.io/blog/2014/04/30/gourmet-service-objects.html)  
[7 Patterns to Refactor Fat ActiveRecord Models](http://blog.codeclimate.com/blog/2012/10/17/7-ways-to-decompose-fat-activerecord-models/)  
[Rails code 整理系列- Service Object 初探](http://motion-express.com/blog/20141007-rails-service-object)  
[Rails service objects vs lib classes](http://stackoverflow.com/questions/16159021/rails-service-objects-vs-lib-classes)  
[Lib Folder vs. Services in Rails](http://www.johnnyji.me/rails/2015/05/19/lib-folder-vs-services-in-rails.html)  
[Service Objects 整理架構](http://tech.gadii.net/blog/2014/08/25/Service-Objects-%E6%95%B4%E7%90%86%E6%9E%B6%E6%A7%8B/)  
[Using Services to Keep Your Rails Controllers Clean and DRY](https://blog.engineyard.com/2014/keeping-your-rails-controllers-dry-with-services)   
[Slimming Down Your Models and Controllers with Concerns, Service Objects, and Tableless Models](https://www.viget.com/articles/slimming-down-your-models-and-controllers)