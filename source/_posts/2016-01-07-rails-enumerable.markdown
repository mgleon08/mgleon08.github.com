---
layout: post
title: "Ruby On Rails - 好用的 Enumerable"
date: 2016-01-07 20:40:22 +0800
comments: true
categories: ruby ruby語法 rails rails語法 api
---

這次主要來介紹一些好用的 Enumerable  
可以很方便的將需要的資料整合在一起  

<!-- more -->

#Map/Collect

對 block 每個值進行運算，並回傳成一個新的 `array`  
處理 `hash` 時，也可以分開處理 key 和 value


`map` 和 `collect` 其實是一樣的東西，主要是因為其他語言很多都是用 `collect`。

```ruby
array = [1,2,3]
array.map {|v| v * 2}
# => [2, 4, 6]

hash = {:name => "abc", :age => 18}
hash.map {|k, v| v }
# => ["abc", 18]
```

#Select

對物件，挑出指定欄位的值，並回傳 `ActiveRecord::Relation`

```ruby
User.all.select(:id)
=> #<ActiveRecord::Relation [#<User id: 1>, #<User id: 2>, #<User id: 3>,...]>
```

可以再搭配 `map` 變成一個 `array`。

```ruby
User.all.select(:id).map(&:id)
# => [1,2,3]
```

或是直接針對 array 去篩選

```ruby
my_array = [1,2,3,4,5,6,7,8,100]
my_array.select{|item| item%2==0 }
# => [2,4,6,8,100]
```

hash

```ruby
my_hash = {"Joe" => "male", "Jim" => "male", "Patty" => "female"}
my_hash.select{|name, gender| gender == "male" }
# => {"Joe" => "male", "Jim" => "male"}

也可以
my_hash.select{|k,v| ["Joe", "Jim"].include?(k)}

#改成 map 會變成，回傳 boolean值，並且回傳 array
my_hash.map{|name, gender| gender == "male" }
# => [true, true, false]
```
只回傳 ture 的選項

進階用法
	
```ruby
#在Hash裡定義新的方法 沒有特定對象的通常可以放在lib資料夾
class Hash
  def keep(*args)
    select {|k,v| args.include?(k)}
  end
end

my_hash.keep("Joe", "Jim")
# => {"Joe" => "male", "Jim" => "male"}
```



#Pluck

對物件，挑出指定欄位的值，並回傳一個新的 `array`  
像是 `map` 和 `select` 合在一起的指令

Approach - map

```ruby
User.pluck(:id)
#=> [1, 2, 3]

puts Benchmark.measure {User.pluck(:id)}
#0.000000   0.000000   0.000000 (  0.000857)
```
```ruby
User.all.map{|a| a.id}
#=> [1, 2, 3]

puts Benchmark.measure {User.all.map{|a| a.id}}
# 0.000000   0.020000   0.020000 (  0.026401)
```
```ruby
User.select(:id)
=> #<ActiveRecord::Relation [#<User id: 1>, #<User id: 2>, #<User id: 3>]>

puts Benchmark.measure {User.select(:id).to_a}
#0.000000   0.000000   0.000000 (  0.001549)
```
顯然效能上還是 `pluck` 最快  

`map` 會將所有欄位找出來，再根據 block 的值，回傳新的 `array`  

`pluck` 和 `select ` 則是只將需要的欄位選出來，但 `select` 回傳的是 `ActiveRecord::Relation` 必須再透過 map 轉成 `array`  

參考文件：  
[Rails Pluck vs Select and Map/Collect](http://rubyinrails.com/2014/06/05/rails-pluck-vs-select-map-collect/)  
[Getting to Know Pluck and Select](http://gavinmiller.io/2013/getting-to-know-pluck-and-select/)
[Pluck vs. map and select](http://ohm.sh/2014/02/09/pluck-vs-map-and-select.html)

```ruby
Person.pluck(:id)
# SELECT people.id FROM people
# => [1, 2, 3]

Person.pluck(:id, :name)
# SELECT people.id, people.name FROM people
# => [[1, 'David'], [2, 'Jeremy'], [3, 'Jose']]

Person.pluck('DISTINCT role')
# SELECT DISTINCT role FROM people
# => ['admin', 'member', 'guest']

Person.where(age: 21).limit(5).pluck(:id)
# SELECT people.id FROM people WHERE people.age = 21 LIMIT 5
# => [2, 3]

Person.pluck('DATEDIFF(updated_at, created_at)')
# SELECT DATEDIFF(updated_at, created_at) FROM people
# => ['0', '27761', '173']
```
#reject
回傳 block 為 `false` 的值，成一個新的 `array`。

```ruby
# Remove even numbers
(1..30).reject { |n| n % 2 == 0 }
# => [1, 3, 5, 7, 9, 11, 13, 15, 17, 19, 21, 23, 25, 27, 29]

# Remove years dividable with 4 (this is *not* the full leap years rule)
(1950..2000).reject { |y| y % 4 != 0 }
# => [1952, 1956, 1960, 1964, 1968, 1972, 1976, 1980, 1984, 1988, 1992, 1996, 2000]

# Remove users with karma below arithmetic mean
total = users.inject(0) { |total, user| total += user.karma }
mean = total / users.size
good_users = users.reject { |u| u.karma < mean }
```

#inject

inject 方法可以先給予初始值(數字，hash，array 都可以)，之後給予指定的元素，不斷的迭代。

```ruby
(5..10).inject(1) {|init, n| init * n }
# => 151200
(5..10).inject(1, :*)                         
#=> 151200
```
```ruby
(5..10).inject {|sum, n| sum * n }
# => 45
(5..10).inject(:+)                            
#=> 45
```
也可以拿來做比較。

```ruby
%w{ cat sheep bear }.inject do |memo,word|
   memo.length > word.length ? memo : word
end
# => "sheep"
```

如果給予 inject 的參數為一個空區塊，那麼 inject 會將結果整理成 Hash。

```ruby
User.all.inject({}) do |hash, user| 
	hash[user.name] = user.id  
	hash # 需要回傳運算結果
end
# => {"A"=>1, "B"=>2, "C"=>3}
```
但要注意的是，由於每跑一次，都會取用最後的回傳值，當做這次的初始值，因此最後必須再加個 `hash` ，否則會出錯。

>也可以直接給予有值的 hash 再繼續加上後面的值，取代掉 inject({}) 中的空 hash

也可改用 reduce 跟 inject 一模一樣  
[Is inject the same thing as reduce in ruby?](http://stackoverflow.com/questions/13813243/is-inject-the-same-thing-as-reduce-in-ruby)

###額外說明
也可以用 map 方式，湊成上面的值。

```ruby
Hash[User.all.map {|user| [user.name, user.id ]}]
# => {"A"=>1, "B"=>2, "C"=>3}

User.all.map {|user| [user.name, user.id ]}.to_h
# => {"A"=>1, "B"=>2, "C"=>3}
```

#each_with_object

跟 inject 非常類似，，主要差別在於你不用回傳運算結果，還有參數是顛倒過來的。

```ruby
User.all.each_with_object({}) do | user, hash | 
	hash[user.name] = user.id  
end
```
#merge
可以將另一個 hash 合併在一起

```ruby
h1 = { "a" => 100, "b" => 200 }
h2 = { "b" => 254, "c" => 300 }
h1.merge(h2)   
#=> {"a"=>100, "b"=>254, "c"=>300}

h1.merge(h2){|key, oldval, newval| newval - oldval}
#=> {"a"=>100, "b"=>54,  "c"=>300}

h1             
#=> {"a"=>100, "b"=>200}
```

```ruby
Hash.new(0).merge(name: "abc", phone: 09xxxxx)
```

#each_with_index

用來加上索引。

```ruby
hash = Hash.new
%w(cat dog wombat).each_with_index {|item, index|
  hash[item] = index
}
#=> ["cat", "dog", "wombat"]

hash
#=> {"cat"=>0, "dog"=>1, "wombat"=>2}
```
[Ruby’s inject/reduce and each_with_object](http://www.bbs-software.com/blog/2013/11/22/rubys-injectreduce-and-each_with_object/)

也可以用來將複數的的 position 印出來。

```ruby
["Cool", "chicken!", "beans!", "beef!"].each_with_index do |item, index|
	print "#{item} " if index%2==0
end
Cool beans!  # => ["Cool", "chicken!", "beans!", "beef!"]
```

#sum
可以算出集合的加總

```ruby
payments.sum { |p| p.price * p.tax_rate }
payments.sum(&:price)
```

數字，字串，陣列都可以，其實就是用 `+` 的方法

```ruby
[5, 15, 10].sum # => 30
['foo', 'bar'].sum # => "foobar"
[[1, 2], [3, 1, 5]].sum #=> [1, 2, 3, 1, 5]
```
#group_by

可以依照指定的欄位分組出來。

```ruby
latest_transcripts.group_by(&:day).each do |day, transcripts|
  p "#{day} -> #{transcripts.map(&:class).join(', ')}"
end

# "2006-03-01 -> Transcript"
# "2006-02-28 -> Transcript"
# "2006-02-27 -> Transcript, Transcript"
# "2006-02-26 -> Transcript, Transcript"
# "2006-02-25 -> Transcript"
# "2006-02-24 -> Transcript, Transcript"
# "2006-02-23 -> Transcript"
```

```ruby
names = ["James", "Bob", "Joe", "Mark", "Jim"]
names.group_by{|name| name.length}
# => {5=>["James"], 3=>["Bob", "Joe", "Jim"], 4=>["Mark"]} 
```

#grep

根據指定的條件塞選

```ruby
names = ["James", "Bob", "Joe", "Mark", "Jim"]
names.grep(/J/)
#=> ["James", "Joe", "Jim"]
```


#index_by

index_by可以指定欄位做為鍵值整理成Hash。

```ruby
User.index_by(&:phone)
# => {'0912xxxxxx' => <User ...>, '0919xxxxxx' => <User ...>, ...}
```

鍵值通常必須是唯一的，若不是唯一的話，會以最後出現的元素做為判斷值。


#any?

只要有任何條件符合，就回傳true

```ruby
%w{ant bear cat}.any? {|word| word.length >= 3}   
#=> true
%w{ant bear cat}.any? {|word| word.length >= 4}   
#=> true
[ nil, true, 99 ].any?                            
#=> 只要有一個不是 nil 和 false 就是 true
```
主要都是集合的方法

可參考之前的  
[.nil? .empty? .blank? .present? 傻傻分不清楚？](http://mgleon08.github.io/blog/2015/12/16/ruby-on-rail-nil-empty-blank-present/)

#&:

```ruby
User.all.map(&:name)
```

 `&:` 代表代入一個Proc  
 `(&:name)` = `{|name| user.name}` 的概念XD。

#Benchmark

上面其實很多都很類似，主要差異的話就是速度吧  
所以可以用以下的方式來測試每種執行出來的速度。

[Benchmark](http://ruby-doc.org/stdlib-2.0.0/libdoc/benchmark/rdoc/Benchmark.html)  
[benchmark-ips](https://github.com/evanphx/benchmark-ips)


官方文件：  
[Enumerable](http://ruby-doc.org/core-2.1.0/Enumerable.html)  
[map/collect](http://apidock.com/ruby/Array/map)  
[reject](http://ruby-doc.org/core-2.2.3/Enumerable.html#method-i-reject)  
[inject](http://apidock.com/ruby/Enumerable/inject)  
[select](http://apidock.com/rails/ActiveRecord/QueryMethods/select)  
[pluck](http://apidock.com/rails/ActiveRecord/Calculations/pluck)    
[reduce](http://apidock.com/ruby/Enumerable/reduce)  
[each_with_object](http://apidock.com/rails/Enumerable/each_with_object)  
[each_with_index](http://apidock.com/ruby/v1_9_3_392/Enumerable/each_with_index)  
[merge](http://ruby-doc.org/core-1.9.3/Hash.html#method-i-merge)  
[sum](http://apidock.com/rails/Enumerable/sum)  
[group_by](http://apidock.com/rails/Enumerable/group_by)  
[index_by](http://apidock.com/rails/v4.2.1/Enumerable/index_by)  
[many?](http://apidock.com/rails/Enumerable/many%3F)  
[any?](http://apidock.com/ruby/Enumerable/any%3F)  

參考文件：  
[Rails Pluck vs Select and Map/Collect](http://rubyinrails.com/2014/06/05/rails-pluck-vs-select-map-collect/)  
[Getting to Know Pluck and Select](http://gavinmiller.io/2013/getting-to-know-pluck-and-select/)  
[Pluck vs. map and select](http://ohm.sh/2014/02/09/pluck-vs-map-and-select.html)  
[Ruby Explained: Map, Select, and Other Enumerable Methods](http://www.eriktrautman.com/posts/ruby-explained-map-select-and-other-enumerable-methods)  
[each_with_object vs inject](https://gist.github.com/cupakromer/3371003)  
[ActiveSupport - 工具函式庫](https://ihower.tw/rails4/activesupport.html)  
[Ruby 用 inject 和 each_with_object 來組 hash
](http://motion-express.com/blog/20141027-ruby-inject-each-with-object-hash)
[What does map(&:name) mean in Ruby?](http://stackoverflow.com/questions/1217088/what-does-mapname-mean-in-ruby)