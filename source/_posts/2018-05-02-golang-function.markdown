---
layout: post
title: "Golang - function, packages"
date: 2018-05-02 18:20:11 +0800
comments: true
categories: golang
---

<!-- more -->

* [函式 function](#function)
  * [匿名函式 Anonymous function](#anonymous)
  * [Defined function types](#types)
  * [Higher-order functionsn](#higher_order_functions)
  * [閉包函式 Closure function](#closure)
  * [初始化函式 Init function ](#init)
  * [Practical use of first class functions](#first_class_function)
* [packages](#packages)

# <span id="function">函式 function</span>

```go
// 定義, Functions are values too
func function_name( [parameter list] ) [return_types list] {
   //function body
}
```

```go
package main

import "fmt"

func foo1(name string) {
  fmt.Println("1. Hi " + name)
}

// 多個傳入值
func foo2(name1 string, name2 string) {
  fmt.Println("2. Hi " + name1 + ", " + name2)
}

// 傳入值同個形態，可只寫一個
func foo3(name1, name2 string) {
  fmt.Println("3. Hi " + name1 + ", " + name2)
}

// return 值的型態定義在後面
func foo4(name string) string {
  var str = "Hi " + name
  return str
}

// 可以直接在 func 的回傳區塊命名回傳變數
func foo5(name string) (str string) {
  str = "Hi " + name
  return
}

// 多重return
func foo6(x, y int) (int, int) {
  return x + y, x - y
}

// 多個傳入值 (s當會有不確定的個數傳入值，當有兩個變數，不確定的要放置在最後)
func foo7(x ...int) int {
  // 收到 Type 為 []int
  var t int
  for _, n := range x {
    t += n
  }
  return t
}

// 傳入slice, slice是類似矩陣的東西
func foo8(x []int) int {
  var t int
  for _, n := range x {
    t += n
  }
  return t
}

// function回傳function，function可以當變數，也可以用來回傳
func foo9() func() string {
  return func() string {
    return "foo9s"
  }
}

func main() {
  // 自動判別型別，必須宣告再 main 裡面
  foo10 := func(name string) {
    fmt.Println("10. Hi " + name)
  }

  foo1("foo1")
  foo2("foo2", "bar2")
  foo3("foo3", "bar3")
  fmt.Println("4.", foo4("foo4"))
  fmt.Println("5.", foo5("foo5"))
  a6, b6 := foo6(1, 2)
  fmt.Println("6.", a6, b6)
  fmt.Println("7.", foo7(1, 2, 3))
  nums := []int{1, 2, 3, 4}
  fmt.Println("8.", foo8(nums))
  fmt.Println("9.", foo9()())
  foo10("foo10")
}

// 1. Hi foo1
// 2. Hi foo2, bar2
// 3. Hi foo3, bar3
// 4. Hi foo4
// 5. Hi foo5
// 6. 3 -1
// 7. 6
// 8. 10
// 9. foo9s
// 10. Hi foo10
```

###  variadic function

當有不確定的參數時，可用 `...` 來代替


> 只能夠放在最後一個參數

```go
package main

import (
	"fmt"
)

func find(num int, nums ...int) {
	fmt.Printf("type of nums is %T\n", nums)
	fmt.Println("nums is", nums)

}
func main() {
	find(89, 89, 90, 95)
	nums := []int{89, 90, 95}
	find(89, nums...) // 必須加上 ... 否則會造成 type error
}

/**
type of nums is []int
nums is [89 90 95]
type of nums is []int
nums is [89 90 95]
**/

// 做法是 golang 會自動將後面的值，轉成 slice，並且根據 type
```

### <span id="anonymous"> 匿名函式 Anonymous function</span>

```go
package main

import (
	"fmt"
)

func main() {
	a := func() {
		fmt.Println("匿名函式")
	}
	a()

	func() {
		fmt.Println("hello world first class function")
	}()

	func(n string) {
		fmt.Println("Welcome", n)
	}("Gophers")
}

/*
匿名函式
hello world first class function
Welcome Gophers
*/
```

### <span id="types"> Defined function types </span>

像定義 struct type 一樣，也可以定義 func type

```go
type add func(a int, b int) int  
```

```go
package main

import (  
    "fmt"
)

// 定義一個 type 是 func，並且裡面要給的參數和 return 的值都已經確認
type add func(a int, b int) int

func main() {  
    var a add = func(a int, b int) int {
        return a + b
    }
    s := a(5, 6)
    fmt.Println("Sum", s)
}
```

### <span id="higher_order_functions"> Higher-order functions </span>

* takes one or more functions as arguments
* returns a function as its result

Passing functions as arguments to other functions

```go
package main

import (
    "fmt"
)

// 變數 a，type 是 a func(a, b int) int
func simple(a func(a, b int) int) {
    fmt.Println(a(60, 7))
}

func main() {
    f := func(a, b int) int {
        return a + b
    }
    simple(f)
}
```

Returning functions from other functions

```go
package main

import (
    "fmt"
)

// return func(a, b int) int

func simple() func(a, b int) int {
    f := func(a, b int) int {
        return a + b
    }
    return f
}

func main() {
    s := simple()
    fmt.Println(s(60, 7))
}
```

### <span id="closure"> 閉包函式 Closure function </span>

```go
// 該閉包涵數接收一個int型參數，其返回值是函式類型
package main

import (
	"fmt"
)

// return func(int) int
func closure(x int) func(int) int {
	fmt.Println("1.", x, &x)
	return func(y int) int {
		fmt.Println("2. x =", x, &x)
		fmt.Println("3. y =", y, &y)
		return x + y
	}
}

func main() {
	f := closure(10)
	fmt.Println("closure(10)\n")

	fmt.Println(f(1))
	fmt.Println("f(1))\n")

	fmt.Println(f(2))
	fmt.Println("f(2)")
}

// 1. 10 0x10414020
// closure(10)

// 此時 x 變數形成了一個 closure
// 2. x = 10 0x10414020
// 3. y = 1 0x10414040
// 11
// f(1))

// 2. x = 10 0x10414020
// 3. y = 2 0x10414050
// 12
// f(2)
```

```go
package main

import (
    "fmt"
)

func appendStr() func(string) string {
    t := "Hello"
    c := func(b string) string {
        t = t + " " + b
        return t
    }
    return c
}

func main() {
    a := appendStr()
    b := appendStr()
    fmt.Println(a("World"))
    fmt.Println(b("Everyone"))

    fmt.Println(a("Gopher"))
    fmt.Println(b("!"))
}

/*
Hello World
Hello Everyone
Hello World Gopher
Hello Everyone !
*/
```

### <span id="init"> 初始化函式 Init function </span>

* The init function should not have any return type and should not have any parameters
* Package level variables are initialised first
* init function is called next. A package can have multiple init functions (either in a single file or distributed across multiple files) and they are called in the order in which they are presented to the compiler.


```go
package main

import "fmt"

func init() {
	fmt.Println("Init")
}

func main() {
	fmt.Println("Main")
}

// Init
// Main
```

### fibonacci

```go
package main

import (
	"fmt"
)

// fibonacci is a function that returns
// a function that returns an int.
func fibonacci() func() int {
  x, y := 0, 1
  fmt.Println("test1 x=", x, "y=", y)
  return func() int {
    fmt.Println("test2 x=", x, "y=", y)
    x, y = y, x + y
    fmt.Println("test3 x=", x, "y=", y)
    return x
  }
}

func main() {
  f := fibonacci()
  for i := 0; i < 10; i++ {
    fmt.Println(f())
  }
}

/**
test1 x= 0 y= 1
test2 x= 0 y= 1
test3 x= 1 y= 1
1
test2 x= 1 y= 1
test3 x= 1 y= 2
1
test2 x= 1 y= 2
test3 x= 2 y= 3
2
test2 x= 2 y= 3
test3 x= 3 y= 5
3
test2 x= 3 y= 5
test3 x= 5 y= 8
5
test2 x= 5 y= 8
test3 x= 8 y= 13
8
test2 x= 8 y= 13
test3 x= 13 y= 21
13
test2 x= 13 y= 21
test3 x= 21 y= 34
21
test2 x= 21 y= 34
test3 x= 34 y= 55
34
test2 x= 34 y= 55
test3 x= 55 y= 89
55
**/
```

### <span id="first_class_function"> Practical use of first class functions </span>

> What are first class functions?
> 
> A language which supports first class functions allows functions to be assigned to variables, passed as arguments to other functions and returned from other functions. Go has support for first class functions.

```go
package main

import (  
    "fmt"
)

type student struct {  
    firstName string
    lastName  string
    grade     string
    country   string
}

func filter(s []student, f func(s student) bool) []student {  
    var r []student
    for _, v := range s {
        if f(v) == true {
            r = append(r, v)
        }
    }
    return r
}

func main() {  
    s1 := student {
        firstName: "Naveen",
        lastName:  "Ramanathan",
        grade:     "A",
        country:   "India",
    }
    s2 := student {
        firstName: "Samuel",
        lastName:  "Johnson",
        grade:     "B",
        country:   "USA",
    }
    stu := []student{s1, s2}
    con := func(s student) bool {
        if s.grade == "B" {
            return true
        }
        return false
    }
    
    f := filter(stu, con)
    
    // 透過此方式，可以很輕易更改 filter 的條件，例如改成 country
    fmt.Println(f)
}

// [{Samuel Johnson B USA}]
```

```go
package main

import (  
    "fmt"
)

func iMap(s []int, f func(int) int) []int {  
    var r []int
    for _, v := range s {
        r = append(r, f(v))
    }
    return r
}
func main() {  
    a := []int{5, 6, 7, 8, 9}
    r := iMap(a, func(n int) int {
        return n * 5
    })
    fmt.Println(r)
}
// [25 30 35 40 45]
```

# <span id="packages"> packages </span>

```
package test
```

宣告程式屬於哪個 `package`，所有的 go 檔案都必須聲明，要 import 這個檔案時，就必須使用 `test` 這個名稱。

而 go 又分兩種專案

* 執行檔 (executable)
* 函式庫 (library)

執行檔一定要宣告為 `main` 套件

```go
// 沒聲明 main 會顯示
go run: cannot run non-main package
```

main 是一個比較特殊的 package，

> Package main is special. It defines a standalone executable program, not a library. Within package main the function main is also special—it’s where execution of the program begins. Whatever main does is what the program does.

因此一定要有個 `main` package 當作是程式入口

* [PackageNames](https://golang.org/doc/code.html#PackageNames)
* [Program_execution](https://golang.org/ref/spec#Program_execution)
* [Package “main” and func “main”](https://stackoverflow.com/questions/42333488/package-main-and-func-main)


### Custom Package

目錄結構

```go
src/
└── geometry
		├── geometry.go
		└── rectangle
				└── rectprops.go

// go install geometry 產生的執行檔會放以下路徑
bin/
└── geometry
```

```go
//geometry.go
package main

import (
    "fmt"
    "geometry/rectangle" //importing custom package
)

func main() {
    var rectLen, rectWidth float64 = 6, 7
    fmt.Println("Geometrical shape properties")
        /*Area function of rectangle package used
        */
    fmt.Printf("area of rectangle %.2f\n", rectangle.Area(rectLen, rectWidth))
        /*Diagonal function of rectangle package used
        */
    fmt.Printf("diagonal of the rectangle %.2f ",rectangle.Diagonal(rectLen, rectWidth))
}
```

* 記得檔案要放在 `$GOPATH/src` 底下，預設會去抓這底下的 `src/geometry/rectangle`
* 在 rectangle 資料夾底下的檔案，package 都必須是 rectangle
* 要 export 的變數都必須是大寫開頭，如以下 `Area` `Diagonal`，改成 `area` `diagonal` 就會變成 private method

```go
//rectprops.go
package rectangle

import "math"

func Area(len, wid float64) float64 {
    area := len * wid
    return area
}

func Diagonal(len, wid float64) float64 {
    diagonal := math.Sqrt((len * len) + (wid * wid))
    return diagonal
}
```

### init function

每個 package 都可以有多個 `init(){}`，並且是一開始就會執行

```go
func init() {
}
```

執行的順序為

1. Package level variables are initialised first
2. init function is called next. A package can have multiple init functions (either in a single file or distributed across multiple files) and they are called in the order in which they are presented to the compiler.

```go
//geometry.go
package main

import (
    "fmt"
    "geometry/rectangle" //importing custom package
    "log"
)
/*
 * 1. package variables
*/
var rectLen, rectWidth float64 = 6, 7

/*
*2. init function to check if length and width are greater than zero
*/
func init() {
    println("main package initialized")
    if rectLen < 0 {
        log.Fatal("length is less than zero")
    }
    if rectWidth < 0 {
        log.Fatal("width is less than zero")
    }
}

func main() {
    fmt.Println("Geometrical shape properties")
    fmt.Printf("area of rectangle %.2f\n", rectangle.Area(rectLen, rectWidth))
    fmt.Printf("diagonal of the rectangle %.2f ",rectangle.Diagonal(rectLen, rectWidth))
}
```

import 其他 package 時，就會執行該 package `init(){}`

有時候只是為了 `init(){}` 並不是真的要使用該 package 就可以用 `_` 來代替

```go
import(
	_ "fmt"
)
```

* import 只能指定資料夾，無法指定檔案
* 同一個資料夾底下只能有一個 package，指的是 test 資料夾裡面所有的 package 都會是 test

```go
// 引入套件，多個可以加括號 ()
import "fmt"

// 希望使用匯入的套件，是為了要觸發那個套件的 init func 而引用的話，可以在前面加上一個底線 _
import _ math

// 如果名字有衝突可以加上 neko
import (
    "github.com/test1/teameow"
    neko "github.com/test2/teameow"
)

// 當前文件同一目錄的 model 目錄，但是不建議這種方式 import
import "./test"

// 載入 GOPATH/src/test1/teameow 模組
import "github.com/test1/teameow"

// 點操作
import(
    . "fmt" // 可以使 fmt.Println("Hi") 省略為 Println("Hi")
)

// 別名操作
import(
    f "fmt" // 就可以改為用 f 來呼叫，f.Println("Hi")
)
```

參考文件:

* [[golangbot.com] functions](https://golangbot.com/functions/)
* [[golangbot.com] packages](https://golangbot.com/packages/)
