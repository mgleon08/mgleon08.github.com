---
layout: post
title: "Golang - function"
date: 2018-05-02 18:20:11 +0800
comments: true
categories: golang
---

<!-- more -->

* [函式 function](#function)
  * [匿名函式 Anonymous function](#anonymous)
  * [Higher-order functionsn](#higher_order_functions)s
  * [閉包函式 Closure function](#closure) 
  * [初始化函式 Init function ](#init)

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

### Class Functions

```go
package main

import (  
    "fmt"
)

type add func(a int, b int) int

func main() {  
    var a add = func(a int, b int) int {
        return a + b
    }
    s := a(5, 6)
    fmt.Println("Sum", s)
}

// Sum 11
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
Hello Everyone !s
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

參考文件:

* [[golangbot.com] functions](https://golangbot.com/functions/)
* [[golangbot.com] functions](https://golangbot.com/packages/)