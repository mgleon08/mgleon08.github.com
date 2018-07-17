---
layout: post
title: "Transactions 交易 - Isolation level"
date: 2016-01-31 10:58:13 +0800
comments: true
categories: rails
---

當兩件事情必須確實執行完畢，才存取到資料庫！

<!-- more -->

# Transactions交易

像是銀行匯款，必須一方確實扣款，另一方確實有新增款項，這筆交易在算成功!

```ruby
ActiveRecord::Base.transaction do
  User.create!(:name => 'hello')
  Feed.create!
end
```

在 transaction 裡面必須使用加上 `!` 才會丟例外，讓交易失敗。

另外，資料要在 transaction 完成後，才會存取到資料庫，因此有用 `after_save` 回呼，可能就會失敗。

因此必須改用 `after_commit`這個回呼，才能確保讀取到交易完成後的資料。

```ruby
class User < ActiveRecord::Base  def foo    self.class.transaction do￼￼      self.update!(name: 'bar')    end 
  endend
```

# Isolation level

Transactions 又可以分成以下四種 level

| 隔離層級                     | Dirty Read | Unrepeatable Read | Phantom Read |
|------------------------------|------------|-------------------|--------------|
| 未提交讀（Read uncommitted） | YES        | YES               | YES          |
| 已提交讀（Read committed）   | NO         | YES               | YES          |
| 可重複讀（Repeatable read）  | NO         | NO                | YES          |
| 可串行化（Serializable ）    | NO         | NO                | NO           |


### 提交讀(Read Committed)


* 能讀取到已經 commit 的 data。
* Oracle等多數數據庫預設都是該級別
* 換句話說，在這樣個 isolation level 下，不會有 "讀到別人還沒 commit 的 data" 這回事(dirty read)

### 未提交讀（Read uncommitted)

* 允許髒讀(dirty read)，也就是可能讀取到其他會話中未提交事務修改的數據

### 可重複讀(Repeated Read)

* 可重複讀。在同一個事務內的查詢都是事務開始時刻一致的，InnoDB 預設級別。在SQL標準中，該隔離級別消除了不可重複讀，但是還存在幻象讀
* 對於該 transaction 內讀過的 data, 在 transaction 的過程中不允許有別的 transaction 改動到這些 data。

### 串行讀(Serializable)

* 完全串行化的讀，每次讀都需要獲得表級共享鎖，讀寫相互都會阻塞
* 這是最進階的 isolation level。基本上可以想像使用這樣 isolation level 的 transaction，概念上可以視為同時間只有它一個 transaction 存在。


rails 指定 `isolation level`

```ruby
MyRecord.transaction(isolation: :read_committed) do
  # do your transaction work
end

# :read_uncommitted
# :read_committed
# :repeatable_read
# :serializable
```


參考文件：  

* [簡介隔離層級](https://openhome.cc/Gossip/HibernateGossip/IsolationLevel.html)
* [ActiveRecord - 進階功能](https://ihower.tw/rails4/activerecord-others.html)
* [透過 Payment Service 與 DB Isolation Level 成為莫逆之交](https://blog.mz026.rocks/20180320/db-isolation-level-w-payment-service)
* [Innodb中的事務隔離級別和鎖的關係](https://tech.meituan.com/innodb-lock.html)
* [transaction api doc](https://apidock.com/rails/ActiveRecord/ConnectionAdapters/DatabaseStatements/transaction)
