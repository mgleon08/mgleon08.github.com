---
layout: post
title: "使用 explain 優化 SQL 語句"
date: 2017-09-01 14:33:17 +0800
comments: true
categories: sql
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
	* const
	* eq_ref
	* ref
	* fulltext
	* ref_or_null
	* index_merge
	* unique_subquery
	* index_subquery
	* range
	* index
	* ALL
5. prossible_keys: 能在該表中使用哪些索引有助於查詢
6. key:實際使用的索引
7. key_len：索引的長度，在不損失精確性的情況 下,長度越短越好
8. ref：索引的哪一列被使用了
9. rows:返回的結果的行數
10. Extra:其他說明 

每個欄位詳細的一些 type 可以到參考文件裡的文章去找尋!

參考文件:

* [使用explain和show profile來分析SQL語句實現優化SQL語句](http://www.shixinke.com/mysql/mysql-sql-optimization-with-using-explain-and-show-profile)
* [在MySQL使用Explain做SQL SELECT語法效能測試](http://blog.kejyun.com/2012/12/Using-EXPLAIN-SQL-To-Analysis-Efficient-On-MySQL.html)