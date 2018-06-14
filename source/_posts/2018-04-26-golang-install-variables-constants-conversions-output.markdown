---
layout: post
title: "Golang 初探 - install, variables, constants, conversions And output"
date: 2018-04-26 22:38:09 +0800
comments: true
categories: golang
---

<!-- more -->

* [安裝 Golang install](#install)
* [變數 variables](#variables)
* [常數 Constants](#constants)
* [轉型 conversions](#conversions)
* [輸出 output](#output)

# Golang 介紹

Golang 是 Google 建立的新語言，主要特色有

* open source
* 靜態語言
* 可編譯

# 選擇用 Golang 的原因

* Concurrency 並發，可以簡單的就可以使用多線程
* 可編譯
* 有固定的語言規範[(spec)](https://golang.org/ref/spec)，不需要擔心要怎麼去寫
* 支援靜態的連結，也可以簡單的部署在 server 上面不需要任何的依賴


# <span id="install">安裝 Golang install </span>


### 安裝 (mac)

1. `brew update && brew upgrade`
2. `brew install go`

### 設定 $GOPATH

GOPATH 就是 golang 的 Workspace

設定在`.bashrc` or `.zshrc`

```
export GOPATH="$HOME/Golang"
export GOBIN="$GOPATH/bin" 
export PATH="$PATH:$GOBIN"
```

設定好可以打 `go env`

```ruby
GOARCH="amd64"
GOBIN="/Users/username/Golang/bin"
GOEXE=""
GOHOSTARCH="amd64"
GOHOSTOS="darwin"
GOOS="darwin"
GOPATH="/Users/username/Golang"
GORACE=""
GOROOT="/usr/local/Cellar/go/1.8.3/libexec"
GOTOOLDIR="/usr/local/Cellar/go/1.8.3/libexec/pkg/tool/darwin_amd64"
GCCGO="gccgo"
CC="clang"
GOGCCFLAGS="-fPIC -m64 -pthread -fno-caret-diagnostics -Qunused-arguments -fmessage-length=0 -fdebug-prefix-map=/var/folders/xh/qq6rsrxs7_v15q07sq5rhqtm0000gn/T/go-build862412071=/tmp/go-build -gno-record-gcc-switches -fno-common"
CXX="clang++"
CGO_ENABLED="1"
PKG_CONFIG="pkg-config"
CGO_CFLAGS="-g -O2"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-g -O2"
CGO_FFLAGS="-g -O2"
CGO_LDFLAGS="-g -O2"
```

### 目錄結構

GOPATH中 會在細分三個資料夾

`$HOME/Golang/`

* `src` - This directory contains source files organized as packages. You will write your Go applications inside this directory.
* `pkg` -  This directory contains Go package objects.
* `bin` - This directory contains executable programs.

### Golang Example

```go
// 宣告程式屬於哪個 package，所有的 go 檔案都必須聲明
package main

// 引入套件，多個可以加括號 ()
import "fmt"

// 希望使用匯入的套件，是為了要觸發那個套件的 main() 函式而引用的話，可以在前面加上一個底線 _
import _ math

// 如果名字有衝突可以加上 neko
import (  
    "github.com/test1/teameow"
    neko "github.com/test2/teameow"
)

// 程式執行入口，main 在 golang 中是特殊的 function
func main() {
// 使用 fmt 套件印出字串 word
 fmt.Println("hello world")
}

// Outside a function, every statement begins with a keyword (var, func, and so on) and so the := construct is not available.
```

 `go run main.go` 就可以直接執行
 
 另外也可以先 `build` 產生執行檔於當前目錄
 
 在該目錄跑 `go build main.go` 後執行 `./main`


### command

```go
go run
// 直接執行
go build
// 產生執行檔於當前目錄，並可以直接執行
go install
// 如果沒有錯誤則產生執行檔於 GOPATH/bin
go clean
// 執行後會將 build 產生的檔案都刪除( install 的不會刪)
go tool
// 顯示目前能用的 tool
```

### 命名規則

```go
1. golang use camelcase
2. 字首大寫代表可讓其他 package 使用，可理解為大寫 public 小寫 private
3. 在 func 外面一定要加上 var OR func 才可以去做定義
4. 常數const(constants) 通常也會第一個字大寫
5. 沒用到的參數可用 blank identifier (_) 來代替
```

### 單引號 / 雙引號 / 反引號


* 雙引號

用來建立可解析的字串字面量(支援轉義，但不能用來引用多行)

```go
fmt.Printf("\u65e5\u672c\u8a9e") // 日本語
```

* 反引號

用來建立原生的字串字面量，這些字串可能由多行組成(不支援任何轉義序列)，原生的字串字面量多用於書寫多行訊息、HTML以及正則表達式

```go
fmt.Printf(`\u65e5\u672c\u8a9e`) // \u65e5\u672c\u8a9e
```

* 單引號

表示 Golang 的一個特殊類型：rune(int32的別稱)，類似其他語言的byte但又不完全一樣，是指：碼點字面量（Unicode code point），不做任何轉義的原始內容。

* [Unicode Character Set and UTF-8, UTF-16, UTF-32 Encoding](https://naveenr.net/unicode-character-set-and-utf-8-utf-16-utf-32-encoding/)
* [UTF-8 encoder/decoder](https://mothereff.in/utf-8#%C3%B1)

# <span id="variables"> 變數 variables </span>

```go
// = 使用必須使用先var聲明
var a 
// 不定型別的變數
var a int 
// 宣告成 int
var a int = 10 
// 初始化同時宣告
var a, b int 
// a 跟 b 都是 int，沒有給值 int 預設是 0
var a, b int = 100, 50 
// 同時宣告一樣 type 並給值
var a = 10 
// 自動推斷型別
var a, b = 0, "test"
// 同時宣告自動判斷 type (必須是同時給值)
var (
   a bool = false 
   b int
   c = "hello"
)
// 多個同時宣告和給值，可以用括號包再一起 (不能同行，不然會錯)
```

### Short hand declaration

```go
// := 是聲明並賦值，並且系統自動推斷類型，必須放在 main function 裡面
// := only works in functions and the lower case 't' is so that it is only visible to the package (unexported).

a := 0
a, b, c := 0, true, "test"

a, b := 1, 2
b, c := 3, 4 // 重複宣告，因為 c 是新的變數，因此可以通過
fmt.Println(a, b, c)
// 1, 3, 4

a, b := 1, 2
a, b := 3, 4 // 重複宣告，因為沒有新的參數，因此會報錯
fmt.Println(a, b)
// no new variables on left side of :=

age := 29
age = "naveen" // golang 是強型別，一但定義就無法轉換成其他型別
// cannot use "test" (type string) as type int in assignment

sum := float64(0) // 宣告並給值
```

# <span id="constants"> 常數 Constants </span>

```go
package main

import "fmt"

func main() {
	// Constants cannot be declared using the := syntax.
	const Pi = 3.14
	const World = "世界"
	const (
		// Create a huge number by shifting a 1 bit left 100 places.
		// In other words, the binary number that is 1 followed by 100 zeroes.
		Big = 1 << 100
		// Shift it right again 99 places, so we end up with 1<<1, or 2.
		Small = Big >> 99
	)
	fmt.Println("Pi", Pi)
	fmt.Println("World", World)
	//fmt.Println(Big)
	fmt.Println(Small)
}

// Pi 3.14
// World 世界
// 2
```

### Assign

在 golang 當中，不允許不同 type 的 assign

```go
package main

func main() {  
    var defaultName = "Sam" //allowed
    type myString string
    var customName myString = "Sam" //allowed
    customName = defaultName //not allowed
}

// cannot use defaultName (type string) as type myString in assignment
// myString 是新建立的 type，相當於 string 的 alias，但因為是不同的 type，因此不能夠直接 assign
```

但是在 const 沒有特別聲明是哪種型態時，是屬於 untyped

```go
package main

import (
	"fmt"
)

func main() {
	const a = 5  // 改成 vat 就會 error
	var intVar int = a
	var int32Var int32 = a
	var float64Var float64 = a
	var complex64Var complex64 = a
	fmt.Println("intVar", intVar, "\nint32Var", int32Var, "\nfloat64Var", float64Var, "\ncomplex64Var", complex64Var)
}

// intVar 5 
// int32Var 5 
// float64Var 5 
// complex64Var (5+0i)
```

# <span id="conversions"> 轉型 conversions  </span>

### Basic Types

```go
- string
- bool
- Numeric Types
  - int  int8  int16  int32  int64
  - uint uint8 uint16 uint32 uint64
  - float32 float64
  - complex64 complex128
  - byte // alias for uint8
  - rune // alias for int32，represents a Unicode code point
```

```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	i := 123
	i64 := int64(i)                               // int to int64
	f64 := float64(i)                             // int to float64
	is := strconv.Itoa(i)                         // int to string
	i64s := strconv.FormatInt(i64, 10)            //int64 to string 10 進位
	f64s := strconv.FormatFloat(f64, 'f', -1, 64) // float64 to string
	f64i64 := int64(f64)                          // float64 to int64
	letter := "abc"                               // string to byte
	si, sierr := strconv.Atoi(is)                 // string to int
	si64, si64err := strconv.ParseInt(is, 10, 64) // string to int64 [func ParseInt(s string, base int, bitSize int) (i int64, err error)]
	sf64, sf64err := strconv.ParseFloat(is, 64)   //  string to float64 [func ParseFloat(s string, bitSize int) (f float64, err error)]
	sb, sberr := strconv.ParseBool("true")        // string to bool [func ParseBool(str string) (value bool, err error)]

	fmt.Printf("i      Type: %T, Value: %v\n", i, i)
	fmt.Printf("i64    Type: %T, Value: %v\n", i64, i64)
	fmt.Printf("f64    Type: %T, Value: %v\n", f64, f64)
	fmt.Printf("is     Type: %T, Value: %v\n", is, is)
	fmt.Printf("i64s   Type: %T, Value: %v\n", i64s, i64s)
	fmt.Printf("f64s   Type: %T, Value: %v\n", f64s, f64s)
	fmt.Printf("f64i64 Type: %T, Value: %v\n", f64i64, f64i64)
	fmt.Printf("letter Type: %T, Value: %v\n", []byte(letter), []byte(letter))
	fmt.Printf("si     Type: %T, Value: %v Err: %v\n", si, si, sierr)
	fmt.Printf("si64   Type: %T, Value: %v Err: %v\n", si64, si64, si64err)
	fmt.Printf("sf64   Type: %T, Value: %v Err: %v\n", sf64, sf64, sf64err)
	fmt.Printf("sb     Type: %T, Value: %v Err: %v\n", sb, sb, sberr)

}

/*
i      Type: int, Value: 123
i64    Type: int64, Value: 123
f64    Type: float64, Value: 123
is     Type: string, Value: 123
i64s   Type: string, Value: 123
f64s   Type: string, Value: 123
f64i64 Type: int64, Value: 123
letter Type: []uint8, Value: [97 98 99]
si     Type: int, Value: 123 Err: <nil>
si64   Type: int64, Value: 123 Err: <nil>
sf64   Type: float64, Value: 123 Err: <nil>
sb     Type: bool, Value: true Err: <nil>
*/
```

### string

```go
package main

import (
	"fmt"
)

func printBytes(s string) {
	for i := 0; i < len(s); i++ {
		fmt.Printf("%x ", s[i])
	}
}

func printChars(s string) {
	runes := []rune(s)
	for i := 0; i < len(runes); i++ {
		fmt.Printf("%c ", runes[i])
	}
}

func printCharsAndBytes(s string) {
	for index, rune := range s {
		fmt.Printf("%c starts at byte %d\n", rune, index)
	}
}

func main() {
	name := "哈囉"
	fmt.Println(len(name)) // 6，中文字串是用3個位元組存的
	printBytes(name)
	fmt.Printf("\n")
	printChars(name)
	fmt.Printf("\n")
	printCharsAndBytes(name)

	fmt.Printf("\n\n")

	name = "Señor"
	fmt.Println(len(name))
	printBytes(name)
	fmt.Printf("\n")
	printChars(name)
	fmt.Printf("\n")
	printCharsAndBytes(name)

	fmt.Printf("\n")

	byteSlice := []byte{0x43, 0x61, 0x66, 0xC3, 0xA9}
	str := string(byteSlice)
	fmt.Println(str)

	fmt.Printf("\n")

	runeSlice := []rune{0x0053, 0x0065, 0x00f1, 0x006f, 0x0072}
	runestr := string(runeSlice)
	fmt.Println(runestr)
}

/**
6
e5 93 88 e5 9b 89 
哈 囉 
哈 starts at byte 0
囉 starts at byte 3


6
53 65 c3 b1 6f 72 
S e ñ o r 
S starts at byte 0
e starts at byte 1
ñ starts at byte 2
o starts at byte 4
r starts at byte 5

Café

Señor
**/
```

* [[golangbot] strings](https://golangbot.com/strings/)

### complex

```go
package main

import (  
    "fmt"
)

func main() {  
    c1 := complex(5, 7)
    c2 := 8 + 27i
    cadd := c1 + c2
    fmt.Println("sum:", cadd)
    cmul := c1 * c2
    fmt.Println("product:", cmul)
}

// sum: (13+34i)
// product: (-149+191i)
```
# <span id="output"> 輸出 output </span>

`fmt` 是 golang 的一個套件，一開始必須 import 進來才可以使用

```go
fmt.Println("Hello") // Hello

A := "Hello"  
fmt.Println(A) // Hello

B := "Hello"  
fmt.Printf("%s, world!", B) // Hello, world!

C := []int{1, 2, 3}  
fmt.Println(C) // [1 2 3]  

var Foo bool = false
fmt.Printf("Type: %T Value: %v\n", Foo, Foo) // Type: bool Value: false

var (
  i int
  f float64
  b bool
  s string
)
fmt.Printf("%v %v %v %q\n", i, f, b, s) // 0 0 false ""
fmt.Printf("%T %T %T %T\n", i, f, b, s) // int float64 bool string
// 0 for numeric types
// false for the boolean type
// "" (the empty string) for string
```

參考文件: 

* [[golangbot.com] learn-golang-series](https://golangbot.com/learn-golang-series/)
* [strconv pkg](https://golang.org/pkg/strconv/)
* [reflect](https://golang.org/pkg/reflect/)
* [golang fmt格式「佔位符」](https://studygolang.com/articles/2644)
* [pkg fmt](https://golang.org/pkg/fmt/)
* [學習Golang語言(4):類型--字串](https://zybuluo.com/codemanship/note/21183)
* [string rune byte 的關係](https://golangtc.com/t/528cc004320b52227200000f)
* [字串編碼筆記：ASCII，Unicode 和 UTF-8](http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html)