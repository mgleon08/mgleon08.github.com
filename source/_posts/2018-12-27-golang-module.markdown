---
layout: post
title: "Golang - Module"
date: 2018-12-27 23:01:48 +0800
comments: true
categories: golang
---

<!-- more -->

go module 是 1.11 版本出的新功能，先介紹之前使用套件的方式

先下載一個套件

```go
go get -u github.com/appleboy/com/random
// -u 如果檔案已經存在，有更新的話就會更新
// 下載後會存在 $GOPATH/src/github.com/appleboy/com/random
```

新增檔案 `$GOPATH/src/testgomod/main.go`

```go
// $GOPATH/src/testgomod/main.go
package main

import (
	"fmt"

	"github.com/appleboy/com/random"
)

func main() {
	fmt.Println(random.String(10))
}
```

### 問題

* 套件無法分版本，因為都是 `$GOPATH/src/github.com/appleboy/com/random`
* 專案必須要在 `$GOPATH` 底下才的 work


# govendor

以前都是用 [govendor](https://github.com/kardianos/govendor) 套件來處理

```go
go get -u github.com/kardianos/govendor
```

在原本的專案底下執行 `govendor init`

```go
// 會出現新的 vendor 資料夾
.
├── main.go
└── vendor
    └── vendor.json
```

在執行 `govendor fetch github.com/appleboy/com/random`

```go
// 後面可以加上版號 govendor fetch github.com/appleboy/com/random@1.0
// 會產生對應的檔案在 vendor 裡面
.
├── main.go
└── vendor
    ├── github.com
    │   └── appleboy
    │       └── com
    │           ├── LICENSE
    │           └── random
    │               └── random.go
    └── vendor.json
```

套件相關資訊都會記錄在 `vendor.json`

```go
// $GOPATH/src/testgomod/vendor/vendor.json

{
	"comment": "",
	"ignore": "test",
	"package": [
		{
			"checksumSHA1": "ZPkL/ha7+FVjS65ZmmlDGH8d12A=",
			"path": "github.com/appleboy/com/random",
			"revision": "c0b5901f9622d5256343198ed54af65af5d08cff",
			"revisionTime": "2018-04-10T03:06:38Z"
		}
	],
	"rootPath": "testgomod"
}
```

### 問題

* 專案必須要在 `$GOPATH` 底下才的 work

# Go module

golang 在版本1.11 新出的功能，初始化專案時必須加上 `export GO111MODULE=on` 原本預設是 `auto` 就是 disable

在原本有用 govendor 專案底下執行，就會自動複製產生

```go
go mod init
// go: creating new go.mod: module testgomod
// go: copying requirements from vendor/vendor.json
```

如果是`新專案`則是

```go
go mod init <packname>
```

多出一個 `go.mod`

```go
// go.mod
// 記錄使用哪個套件，版本，sha1
module testgomod

// 新專案就不會有這條
require github.com/appleboy/com v0.0.0-20180410030638-c0b5901f9622
```

接著 import 的地方如果有需要下載的 package，執行以下指令就會自動去抓取

```go
go mod download
// or
go run main.go
// or
go build -v -o main .
```

會產生 `go.sum`

```go
// go.sum
github.com/appleboy/com v0.0.0-20180410030638-c0b5901f9622 h1:ozHD8HTq7ivv8vTJRCAzjA4wEA8BMGekxMDZrFdqz5M=
github.com/appleboy/com v0.0.0-20180410030638-c0b5901f9622/go.mod h1:rtwjPnHClMOJw4K5oW3ASx9BCPCJ1SDbFbzJjY4Ebqw=
```

實際上會存取在 `$GOPATH/pkg/mod/`

這時候可以嘗試將 govendor 的資料夾刪除，並改 module 成 auto

```go
rm -rf vendor
export GO111MODULE=auto
```

跑 

```go
go run main.go
// or
go build -v -o main .
// -o output
// -v print the names of packages as they are compiled.
```

會發現建立不了，再試著將 module 改成 on 去跑

```go
export GO111MODULE=on
```

```go
go run main.go
// or
go build -v -o main .

// 就不依賴 goverdor，專案也不一定要在 $GOPATH 底下
```

### 好處

* 不需在 $GOPATH 底下才能建立專案
* 可以切換不同版本

### 清除

清除 pkg 底下內容

```go
go clean -i -x -modcache
```

### 指令

```go
go download    download modules to local cache
go edit        edit go.mod from tools or scripts
go graph       print module requirement graph
go init        initialize new module in current directory
go tidy        add missing and remove unused modules
go vendor      make vendored copy of dependencies
go verify      verify dependencies have expected content
go why         explain why packages or modules are needed
```

參考文件

* [Go 語言 1.11 版本推出 go module](https://blog.wu-boy.com/2018/10/go-1-11-support-go-module/)
* [跳出Go module的泥潭](https://colobu.com/2018/08/27/learn-go-module/)
* [Introduction to Go Modules](https://roberto.selbach.ca/intro-to-go-modules/)
* [govendor](https://github.com/kardianos/govendor)
