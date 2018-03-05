---
layout: post
title: "樂觀鎖 與 悲觀鎖 Optimistic Locking & Pessimistic Locking"
date: 2017-11-01 12:16:10 +0800
comments: true
categories: rails
---

有時候在 db 中會發現重複的值，在交易頻繁的網站更是如此，那就需要 lock，rails 也提供方便的方法來鎖!

<!-- more -->

## 樂觀鎖

* 每次去拿數據的時候，都認為別人不會修改數據，所以不會對數據上鎖，這樣在你拿數據的時候別人也能拿和你屬於同一條的數據。
* 在更新數據時，會判斷在這期間是否有人更新過數據，如果有，則本次更新失敗；否則成功。
* 由於多個用戶可以同時對同一條數據進行訪問，增加了數據庫的吞吐量。
* 適合在資源爭用不激烈的時候使用。


> 使用樂觀鎖之前需要給數據庫增加一列 :lock_version，Rails 會自動識別這一列，像數據庫提交數據的時候自動帶上。

```ruby
retry_times = 3

begin
    @order.with_lock do
        @order.set_paid!
    end
rescue ActiveRecord::StaleObjectError => e
    retry_times -= 1
    if retry_times > 0
        retry
    else
        raise e
    end
rescue => e
    raise e
end
```

## 悲觀鎖

* 每次去拿數據的時候，都認為別人會修改數據，因此會對數據上鎖，這樣在自己讀寫數據的過程中，別人不能讀寫這條數據，只能等待本次處理結束，才能訪問。
* 嚴謹、有效的保證了數據的有效行
* 不能同時對數據庫中同一條數據進行訪問，大大減少了數據庫的吞吐量。
* 需要持續的與數據庫保持連接，因此不適合web應用
* 實現起來，比較麻煩。
* 在資源爭用比較嚴重的時候比較合適



```ruby
# rails
account = Account.find(1)
Account.transaction do
    account.lock!
    account.balance -= 100
    account.save! 
end

# 和下面是等價的

account.with_lock do
    account.balance -= 100
    account.save!
end
```


參考文件:

* [更新時鎖定記錄](https://rails.ruby.tw/active_record_querying.html#%E6%9B%B4%E6%96%B0%E6%99%82%E9%8E%96%E5%AE%9A%E8%A8%98%E9%8C%84)
* [Rails 中樂觀鎖與悲觀鎖的使用](https://ruby-china.org/topics/28963)
* [簡介隔離層級](https://openhome.cc/Gossip/HibernateGossip/IsolationLevel.html)
* [Rails 中樂觀鎖與悲觀鎖的使用](https://ruby-china.org/topics/28963)
* [ActiveRecord::Locking::Pessimistic](http://api.rubyonrails.org/classes/ActiveRecord/Locking/Pessimistic.html)
* [ActiveRecord::Locking::Optimistic](http://api.rubyonrails.org/classes/ActiveRecord/Locking/Optimistic.html)
