---
layout: post
title: "使用 explain 優化 SQL 語句"
date: 2017-09-01 14:33:17 +0800
comments: true
categories: sql other
---

有時候發現 query 速度很慢，但又不知道要怎麼提升的時候，就可以用 explain 的方式，來看看 sql 語法，哪裡可以做優化!

<!-- more -->

當 query 速度很慢時，很多時候可能是少加了 index，但有時又不是能夠這麼快的馬上看出來，這時候就可以將 sql 語法前面加上 `explain` 來解析

```sql
explain SELECT * FROM user
```


| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra       |
|----|-------------|-------|------------|-------|---------------|---------|---------|------|------|----------|-------------|
| 1  | SIMPLE      | users | NULL       | range | PRIMARY       | PRIMARY | 4       | NULL | 2148 | 100      | Using where |

1. id: 沒有意義
2. select_type: 查詢類型，是單表查詢、聯合查詢還是子查詢等
3. table: 查詢的表名
4. type: 連接使用的類型(重要項) 顯示連接使用的類型,按最 優到最差的類型排序
	* System
		* 表只有一行，這是一個 const type 的特殊情況
	* const
		* 使用主鍵或者唯一索引的時候，當查詢的表僅有一行時，使用 System
	* eq_ref
		* MySQL在連接查詢時，會從最前面的資料表，對每一個記錄的聯合，從資料表中讀取一個記錄，在查詢時會使用索引為主鍵或唯一鍵的全部
	* ref
		* 只有在查詢使用了非唯一鍵或主鍵時才會發生
	* fulltext
		* 使用全文索引的時候才會出現
	* ref_or_null
		* 查詢類型和ref很像，但是 MySQL 會做一個額外的查詢，來看哪些行包含了NULL。這種類型常見於解析子查詢的優化
	* index_merge
		* 在一個查詢裡面很有多索引用被用到，可能會觸發 index_merge 的優化機制
	* unique_subquery
		* 比 eq_ref 複雜的地方是使用了 in 的子查詢，而且是子查詢是主鍵或者唯一索引
	* index_subquery
		* 和 unique_subquery 類似，但是它在子查詢裡使用的是非唯一索引。
	* range
		* 使用索引返回一個範圍的結果，例如：使用大於 > 或小於 < 查詢時發生。
	* index
		* 全表掃瞄，此為針對索引中的資料進行查詢，主要優點就是避免了排序，但是開銷仍然非常大
	* ALL
		* 針對每一筆記錄進行完全掃瞄，此為最壞的情況，應該儘量避免
5. prossible_keys: 能在該表中使用哪些索引有助於查詢
6. key:實際使用的索引
7. key_len：索引的長度，在不損失精確性的情況 下,長度越短越好
8. ref：索引的哪一列被使用了
9. rows:返回的結果的行數
10. Extra:其他說明

# composite index
另外有時候我們會做 composite index，但其實這種 index 是有順序的!

例如:

```ruby
add_index :lookup, [:name, :email, :phone], name: "lookup_index", unique: true
```

當在 `where` 的時候，就必須有順序 `name` `email` `phone` 這個 `index` 才會有效

ex:

* `name`
* `name` `email`
* `name` `email` `phone`
* `name` `phone`

`where` 的時候不管順序，只要有對應到就可以

像是 `email` or `phone` or `email` + `phone` 就不會用到這個 index

參考文件:

* [使用explain和show profile來分析SQL語句實現優化SQL語句](http://www.shixinke.com/mysql/mysql-sql-optimization-with-using-explain-and-show-profile)
* [在MySQL使用Explain做SQL SELECT語法效能測試](http://blog.kejyun.com/2012/12/Using-EXPLAIN-SQL-To-Analysis-Efficient-On-MySQL.html)
* [MySQL 中 EXPLAIN 命令詳解](https://overtrue.me/articles/2014/10/mysql-explain.html)
* [最官方的 mysql explain type 字段解讀](https://mengkang.net/1124.html)
