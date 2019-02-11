---
layout: post
title: "Sql 好用的 command"
date: 2018-06-14 18:43:04 +0800
comments: true
categories: sql other
---

最近常常使用 sql，因應各種需求，也發現一些蠻好用的指令~

* [FIELD](#FIELD)
* [GROUP_CONCAT](#GROUP_CONCAT)
* [CONCAT](#CONCAT)
* [IFNULL](#IFNULL)
* [CONVERT](#CONVERT)
* [CASE](#CASE)
* [LOWER() & UPPER()](#LOWER_UPPER)
* [AUTO_INCREMENT](#AUTO_INCREMENT)
* [unix_timestamp](#unix_timestamp)
* [BETWEEN](#BETWEEN)
* [Time](#Time)
* [specify order](#specify_order)
* [Table View](#table_view)

tutorial

* [sqlzoo.net](#https://sqlzoo.net/)

<!-- more -->

# <span id="FIELD"> FIELD() <span>

FIELD 會 return 第一個參數的，position

```sql
-- 第一個參數 "c", 在後面第三個位置
SELECT FIELD("c", "a", "b", "c", "d", "e");
-- 3

-- 如果找不到就會是 0
SELECT FIELD("f", "a", "b", "c", "d", "e");
-- 0
```
可以搭配 order，讓取出來的順序按照原本給的參數

```ruby
user_ids = [3, 5, 1, 2, 4]

users = User.where(id: user_ids).order("FIELD(id, #{user_ids.join(',')})")
```

```sql
SELECT `users`.* FROM `users` WHERE `users`.`id` IN (3, 5, 1, 2, 4)  ORDER BY FIELD(id, 3,5,1,2,4)"
```

* [MySQL FIELD() Function](https://www.w3schools.com/sql/func_mysql_field.asp)

# <span id="GROUP_CONCAT"> GROUP_CONCAT <span>

可以透過 group 將所有的 book 做分類，並且依照分類將 `book name` group 起來

```ruby
# 取出 user 底下所有的 books，並且照 books_type, books_name(會是 string 串聯起來)

User.select("books.type AS book_type, GROUP_CONCAT(DISTINCT books.name SEPARATOR ', ') as book_name")
    .joins("LEFT JOIN books ON users.id = books.user_id")
    .where(id: 8)
    .group("books.type")
```

```sql
SELECT user.id, books.type, GROUP_CONCAT(DISTINCT books.name SEPARATOR ', ') as book_name
FROM users
LEFT JOIN books ON users.id = books.user_id
WHERE user.id = 8
GROUP BY books.type
```

也可以排序

```sql
SELECT GROUP_CONCAT(DISTINCT books.id ORDER BY books.id ASC SEPARATOR ', ') as book_name
```

# <span id="CONCAT"> CONCAT <span>

可以將多的 column 串在一起

```sql
CONCAT(字串1, 字串2, 字串3, ...)
```

# <span id="IFNULL"> IFNULL <span>

```sql
-- 如果x不是NULL(不包含0)，IFNULL()返回x，否則它返回y。
IFNULL(x, y)
```

# <span id="CONVERT"> CONVERT <span>

```sql
-- 將時間轉成 date
CONVERT(created_at, date)
```

# <span id="CASE"> CASE <span>

像 `if else` 一樣，可以根據條件，給不同的值

```sql
SELECT CASE ("欄位名")
  WHEN "條件1" THEN "結果1"
  WHEN "條件2" THEN "結果2"
  ...
  [ELSE "結果N"]
  END
FROM "表格名";
```

# <span id="LOWER_UPPER"> LOWER() &  UPPER() <span>

將字串轉小寫或大寫

```sql
LOWER("HI");
UPPER("hi");
```

# <span id="AUTO_INCREMENT"> AUTO_INCREMENT <span>

```sql
ALTER TABLE `pre_campaign_details_channels` CHANGE `id` `id` INT(11)  NOT NULL  AUTO_INCREMENT , ADD UNIQUE (`id`);
```

# <span id="unix_timestamp"> unix_timestamp <span>


date 轉成 Unix Timestamp

`2018/01/01` -> `1514736000(s)` -> `1514736000000(ms)`

```sql
SELECT unix_timestamp(my_datetime_column) as stamp
-- milliseconds
SELECT unix_timestamp(my_datetime_column) * 1000 as stamp
```

[Convert MySql DateTime stamp into JavaScript's Date format](https://stackoverflow.com/questions/3075577/convert-mysql-datetime-stamp-into-javascripts-date-format)

# <span id="BETWEEN"> BETWEEN <span>

可以直接指定在某個區段的時間

```sql
SELECT *
FROM Store_Information
WHERE Txn_Date BETWEEN 'Jan-06-1999' AND 'Jan-10-1999';
```

[SQL Between](https://www.1keydata.com/tw/sql/sqlbetween.html)

# <span id="Time"> Time <span>

```sql
SELECT NOW(), CURTIME(), CURDATE(), (CURDATE() - INTERVAL 1 DAY), (CURDATE() - INTERVAL 1 MONTH)
-- 2018-09-20 17:20:48
-- 17:20:48
-- 2018-09-20
-- 2018-09-19
-- 2018-10-20
-- INTERVAL - 可以將時間去做加減
```

# <span id="specify_order"> specify order <span>

指定順序

```sql
SELECT `users`.*
FROM `users`
WHERE `users`.`id` IN (11, 13, 3, 5, 7, 9)
ORDER BY
  CASE
  WHEN `users`.`id`=11 THEN 0
  WHEN `users`.`id`=13 THEN 1
  WHEN `users`.`id`=9 THEN 2
  ELSE 3 END ASC
```

# <span id="table_view"> Table View <span>

虛擬表格。它跟表格的不同是，表格中有實際儲存資料，而視觀表是建立在表格之上的一個架構，它本身並不實際儲存資料。

```sql
CREATE VIEW "視觀表名"
AS "SQL SELECT 語句";
```

```sql
CREATE VIEW OR REPLACE "視觀表名"
AS "SQL SELECT 語句";
```

* [MySQL 超新手入門（11）Views](http://www.codedata.com.tw/database/mysql-tutorial-11-views/)
* [sql-create-view](https://www.1keydata.com/tw/sql/sql-create-view.html)
* [sql-view](http://www.runoob.com/sql/sql-view.html)


