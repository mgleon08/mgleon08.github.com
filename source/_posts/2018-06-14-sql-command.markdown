---
layout: post
title: "Sql 好用的 command"
date: 2018-06-14 18:43:04 +0800
comments: true
categories: sql
---

最近常常使用 sql，因應各種需求，也發現一些蠻好用的指令~

<!-- more -->

# FIELD()

FIELD 會 return 第一個參數的，position

```sql
// 第一個參數 "c", 在後面第三個位置
SELECT FIELD("c", "a", "b", "c", "d", "e");
// 3

// 如果找不到就會是 0
SELECT FIELD("f", "a", "b", "c", "d", "e");
// 0
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

# GROUP_CONCAT

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

# CONCAT

可以將多的 column 串在一起

```sql
CONCAT(字串1, 字串2, 字串3, ...)
```

# IFNULL

```sql
// 如果x不是NULL(不包含0)，IFNULL()返回x，否則它返回y。
IFNULL(x, y)
```

# CONVERT

```sql
// 將時間轉成 date
CONVERT(created_at, date)
```