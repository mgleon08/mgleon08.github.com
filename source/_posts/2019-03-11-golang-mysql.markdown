---
layout: post
title: "Golang - MySQL"
date: 2019-03-11 23:13:41 +0800
comments: true
categories: golang
---

<!-- more -->

golang 在 database 上的存取，建立了一個 sql 抽象介面 [database/sql](https://golang.org/pkg/database/sql/)，接著大家就可以透過這個 interface 去實作裡面相對應的 driver

目前看到比較多人使用的是 [go-sql-driver/mysql](https://github.com/go-sql-driver/mysql)


# install

```go
go get github.com/go-sql-driver/mysql
```

# connect to database

```go
package main

import (
	"database/sql"
	"fmt"
	"strings"

	_ "github.com/go-sql-driver/mysql"
)

// db 設定
const (
	userName = "root"
	password = ""
	host     = "127.0.0.1"
	port     = "3306"
	dbName   = "dbName"
)

func initDB() {
	// [username[:password]@][protocol[(address)]]/dbname[?param1=value1&...&paramN=valueN]
	// "username:password@tcp(host:port)/數據庫?charset=utf8"
	path := strings.Join([]string{userName, ":", password, "@tcp(", host, ":", port, ")/", dbName, "?charset=utf8"}, "")
	fmt.Println(path)

	// 第一個是 driverName 第二個則是 database 的設定 path
	// 也可以用 var DB *sql.DB
	DB, _ := sql.Open("mysql", path)

	// 設定 database 最大連接數
	DB.SetConnMaxLifetime(100)

	//設定上 database 最大閒置連接數
	DB.SetMaxIdleConns(10)

	// 驗證是否連上 db
	if err := DB.Ping(); err != nil {
		fmt.Println("opon database fail:", err)
		return
	}
	fmt.Println("connnect success")
}

func main() {
	initDB()
}
```

# query

### Exec

```go
result, err := db.Exec(
    "INSERT INTO users (name) VALUES (?)",
    "leon",
)
```

### QueryRow

```go
var id int64
row := DB.QueryRow("SELECT id FROM users WHERE name = ?", "leon")
err := row.Scan(&id)
if err != nil {
    log.Fatal(err)
}
fmt.Println(id)
```

### Query

```go
rows, err := DB.Query("SELECT * FROM banner")
if err != nil {
  log.Fatal(err)
}

// 記得要 close
defer rows.Close()
for rows.Next() {
  var id, width, height int64
  if err := rows.Scan(&id, &width, &height); err != nil {
    log.Fatal(err)
  }
  fmt.Printf("id: %v, width: %v, height: %v\n", id, width, height)
}
if err := rows.Err(); err != nil {
  log.Fatal(err)
}
```

### Prepare

```go
width := 640
stmt, err := DB.Prepare("SELECT id FROM banner WHERE width = ?")
if err != nil {
  log.Fatal(err)
}
rows, err := stmt.Query(width)

// 記得要 close
defer rows.Close()
for rows.Next() {
  var id int64
  if err := rows.Scan(&id); err != nil {
    log.Fatal(err)
  }
  fmt.Printf("id: %v\n", id)
}
```

### Begin (transcation)

```go
// 開啟 transaction
tx, err := DB.Begin()
if err != nil {
  fmt.Println("tx fail")
  return
}
// 準備 sql 語句
stmt, err := tx.Prepare("INSERT INTO banner (`id`, `width`, `height`) VALUES (?, ?, ?)")
if err != nil {
  fmt.Println("Prepare fail")
  return
}
// 設定參數以及執行 sql 語句
res, err := stmt.Exec(100, 200, 300)
if err != nil {
  fmt.Println("Exec fail")
  return
}
// 提交 transaction
tx.Commit()
// 取得最後一個 insert 的 id
fmt.Println(res.LastInsertId())
```

# 各種方式效率分析

先參照 [golang學習之旅：使用go語言操作mysql數據庫](https://studygolang.com/articles/3022) 之後有用到再來研究

Reference:

* [database/sql](https://golang.org/pkg/database/sql/)
* [go-sql-driver/mysql](https://github.com/go-sql-driver/mysql)
* [Golang連接mysql數據庫](https://www.jianshu.com/p/ee87e989f149)
* [golang學習之旅：使用go語言操作mysql數據庫](https://studygolang.com/articles/3022)
