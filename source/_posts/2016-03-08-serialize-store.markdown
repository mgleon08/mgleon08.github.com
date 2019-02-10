---
layout: post
title: "Serialize & Store 將 object 塞在欄位裡"
date: 2016-03-08 22:33:45 +0800
comments: true
categories: rails
---
當欄位上需要塞比較多 `data` 時就可以使用，相當便利。

<!-- more -->
#Serialize 序列化

可以將一個 object 轉換成一個可被資料庫儲存及傳輸的純文字形態
反之讀出來就是，Deserialize 反序列。

`serialize` 可指定欄位，在存入資料庫時，就會自動序列化成YAML格式。

### 欄位必須是 text

```ruby
t.text types
```

model

```ruby
class User < ActiveRecord::Base
  serialize :types
  #若沒特別指定，任何形式都可存取
  serialize :types Array
  #指定 Array，就不能塞 Hash 進去
end

User.create(profiles: {gender: "male", phone: 12345})
User.last.profiles
#=> {gender: "male", phone: 12345}
User.last.profiles[:gender]
#=> male
```


>缺點是 serialize 後的欄位，就無法用 where 進行查詢。


#Store

上述的 profiles 可以用 `store` 來設定。
就像是一般的變數一樣。

### 欄位必須是 text

```ruby
t.text types
```

```ruby
class User < ActiveRecord::Base
  store :profiles, :accessors => [:gender, :phone]
end


User.create(gender: "male", phone: 12345)

user.profiles
#=> {:gender => "male", :phone => 12345}
user.gender
#=> "male"
user.phone
#=> 12345
```

官方文件：  
[api - serialize](http://api.rubyonrails.org/classes/ActiveRecord/Base.html#M001799)  
[apidock - serialize](http://apidock.com/rails/ActiveRecord/Base/serialize/class)  
[api - Store](http://api.rubyonrails.org/classes/ActiveRecord/Store.html)  

參考文件：  
[ActiveRecord - 進階功能](https://ihower.tw/rails4/activerecord-others.html)
