---
layout: post
title: "Golang - defer, error handling, custom errors, panic, recover"
date: 2018-05-29 15:39:48 +0800
comments: true
categories: golang
---

<!-- more -->

* [Defer](#defer)
* [Error Handling](#error_handling)
* [Custom Errors](#custom_errors)
* [Panic and Recover](#panic_recover)

# <span id="defer"> Defer </span>

defer後面的表達式會被放入一個列表中，在當前方法返回的時候，列表中的表達式就會被執行。

> * 在Golang中，defer表達式通常用於資源清理和釋放、文件關閉、解鎖以及記錄時間等操作。
> * 即使函式發生嚴重錯誤也會執行
> * 透過與匿名函式配合可在return之後修改函式計算結果
> * Go沒有異常機制，但有 panic/recover 模式來處理錯誤
> * Panic 可以在任何地方引發，但 recover 只有在 defer 呼叫的函式中有效

```go
package main

import (
	"fmt"
)

func main() {
	defer fmt.Println("world")

	fmt.Println("hello")
}


// hello 
// world
```

### Last-In-First-Out (LIFO)

> defer 是讓 fmt.Println(0) , fmt.Println(1) , fmt.Println(2) , fmt.Println(3) , fmt.Println(4) 依序放到清單中, 等到 func f 結束前, 再依據 Last-In-First-Out (LIFO) 的順序 call 清單中的 function.

```go
package main

import (
	"fmt"
)

func main(){
    for i := 0 ; i < 5 ; i++{
        defer fmt.Println(i)
    }
    fmt.Println("f finish")
}


// f finish
// 4
// 3
// 2
// 1
// 0
```

```go
package main

import (  
    "fmt"
)

func main() {  
    name := "Naveen"
    fmt.Printf("Orignal String: %s\n", string(name))
    fmt.Printf("Reversed String: ")
    for _, v := range []rune(name) {
        defer fmt.Printf("%c", v)
    }
}

// Orignal String: Naveen  
// Reversed String: neevaN  
```

### Arguments evaluation

```go
package main

import (  
    "fmt"
)

func printA(a int) {  
    fmt.Println("value of a in deferred function", a)
}

func main() {  
    a := 5
    defer printA(a) // 收到的 arg 會是這時候當下的 arg
    a = 10
    fmt.Println("value of a before deferred function call", a)
}

// value of a before deferred function call 10
// value of a in deferred function 5
```

### defer with return

Ex1. 

```go
package main

import (
	"fmt"
)

func f() (result int) {
	// 相當於 result = 0，最後再執行 result++，因此最後回傳會是 1
	defer func() {
		result++
	}()
	return 0
}

func main() {
	fmt.Println(f())
}

// 1
```
Ex2.

```go
package main

import (
	"fmt"
)

func f() (r int) {
	t := 5
	// 相當於 r = t，因為最後回傳的參數為 r，但 defer 裡面是針對 t + 5，因此不受影響
	defer func() {
		t = t + 5
	}()
	return t
}

func main() {
	fmt.Println(f())
}

// 5
```

Ex3.

```go
package main

import (
	"fmt"
)

func f() (r int) {
	// 相當於 r = 1
	defer func(r int) { //這裡改的r是傳值傳進去的r，不會改變要返回的那個r值
		r = r + 5
	}(r)
	return 1
}

func main() {
	fmt.Println(f())
}
// 1
```

參考文件:

* [[golangbot.com]](https://golangbot.com/learn-golang-series/)
* [[golangbot.com]defer](https://golangbot.com/defer/)
* [Defer_statements](https://golang.org/ref/spec#Defer_statements)
* [Golang中defer的那些事](https://xiaozhou.net/something-about-defer-2014-05-25.html)
* [defer panic recover](https://hsinyu.gitbooks.io/golang_note/content/defer_panic_recover.html)
* [3.4 defer關鍵字](https://tiancaiamao.gitbooks.io/go-internals/content/zh/03.4.html)



# <span id="error_handling"> Error Handling </span>

### 1. Asserting the underlying struct type and getting more information from the struct fields


```go
package main

import (  
    "fmt"
    "os"
)

func main() {  
    f, err := os.Open("/test.txt")
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Println(f.Name(), "opened successfully")
}

// open /test.txt: No such file or directory  
```

```go
package main

import (  
    "fmt"
    "os"
)

func main() {  
    f, err := os.Open("/test.txt")
    // type assertion 斷言 err 是 *os.PathError type，斷言的結果就是 err 的動態值，動態值的 type 就是 *os.PathError  
    // //get the underlying *os.PathError value from err  
    if err, ok := err.(*os.PathError); ok {
        fmt.Println("File at path", err.Path, "failed to open")
        return
    }
    fmt.Println(f.Name(), "opened successfully")
}

// File at path /test.txt failed to open  
```

##### Golang 的 `err.(*os.PathError)` 究竟是什麼？

os.Open return 一個 err 的值，但這個 error 本身是一個 interface

```go
type error interface { Error() string } 
```

* 任何碰巧具有 Error 方法的數據類型都將實現該介面並且可以被分配。在大多數情況下，僅打印錯誤就足夠了
* 聲明 `e, ok := err.(*os.PathError)` 是一個 type asserting 。 
* 它將檢查介面值 err 包含 `*os.PathError` 作為具體類型並將返回該值。 

### 2. Asserting the underlying struct type and getting more information using methods

```go
package main

import (  
    "fmt"
    "net"
)

func main() {  
    addr, err := net.LookupHost("golangbot123.com")
    if err, ok := err.(*net.DNSError); ok {
        if err.Timeout() {
            fmt.Println("operation timed out")
        } else if err.Temporary() {
            fmt.Println("temporary error")
        } else {
            fmt.Println("generic error: ", err)
        }
        return
    }
    fmt.Println(addr)
}
```

### 3. Direct comparison

```go
package main

import (  
    "fmt"
    "path/filepath"
)

func main() {  
    files, error := filepath.Glob("[")
    if error != nil && error == filepath.ErrBadPattern {
        fmt.Println(error)
        return
    }
    fmt.Println("matched files", files)
}
```

# <span id="custom_errors"> Custom Errors </span>

### 1. Creating custom errors using the New function

```go
package main

import (  
    "errors"
    "fmt"
    "math"
)

func circleArea(radius float64) (float64, error) {  
    if radius < 0 {
        return 0, errors.New("Area calculation failed, radius is less than zero")
    }
    return math.Pi * radius * radius, nil
}

func main() {  
    radius := -20.0
    area, err := circleArea(radius)
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Printf("Area of circle %0.2f", area)
}
// Area calculation failed, radius is less than zero
```

### 2. Adding more information to the error using Errorf

```go
package main

import (  
    "fmt"
    "math"
)

func circleArea(radius float64) (float64, error) {  
    if radius < 0 {
        return 0, fmt.Errorf("Area calculation failed, radius %0.2f is less than zero", radius)
    }
    return math.Pi * radius * radius, nil
}

func main() {  
    radius := -20.0
    area, err := circleArea(radius)
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Printf("Area of circle %0.2f", area)
}
// Area calculation failed, radius -20.00 is less than zero
```

### 3. Providing more information about the error using struct type and fields

```go
package main

import (  
    "fmt"
    "math"
)

type areaError struct {  
    err    string
    radius float64
}

func (e *areaError) Error() string {  
    return fmt.Sprintf("radius %0.2f: %s", e.radius, e.err)
}

func circleArea(radius float64) (float64, error) {  
    if radius < 0 {
        return 0, &areaError{"radius is negative", radius}
    }
    return math.Pi * radius * radius, nil
}

func main() {  
    radius := -20.0
    area, err := circleArea(radius)
    if err != nil {
        if err, ok := err.(*areaError); ok {
            fmt.Printf("Radius %0.2f is less than zero", err.radius)
            return
        }
        fmt.Println(err)
        return
    }
    fmt.Printf("Area of rectangle1 %0.2f", area)
}
// Radius -20.00 is less than zero
```

### 4. Providing more information about the error using methods on struct types

```go
ackage main

import "fmt"

type areaError struct {  
    err    string  //error description
    length float64 //length which caused the error
    width  float64 //width which caused the error
}

func (e *areaError) Error() string {  
    return e.err
}

func (e *areaError) lengthNegative() bool {  
    return e.length < 0
}

func (e *areaError) widthNegative() bool {  
    return e.width < 0
}

func rectArea(length, width float64) (float64, error) {  
    err := ""
    if length < 0 {
        err += "length is less than zero"
    }
    if width < 0 {
        if err == "" {
            err = "width is less than zero"
        } else {
            err += ", width is less than zero"
        }
    }
    if err != "" {
        return 0, &areaError{err, length, width}
    }
    return length * width, nil
}

func main() {  
    length, width := -5.0, -9.0
    area, err := rectArea(length, width)
    // 這裡回傳回來的 err，是 interface type，還不知道是什麼 dynamic value & type
    //因此 err.length, err.width, lengthNegative(), widthNegative()，都沒辦法使用，只能使用 Error()
    if err != nil {
    	// 利用 type type assertion to get the underlying value of the error interface
        if err, ok := err.(*areaError); ok {
        // 因此這裡的 err，可以使用 err.length, err.width, lengthNegative(), widthNegative()
            if err.lengthNegative() {
                fmt.Printf("error: length %0.2f is less than zero\n", err.length)
            }
            if err.widthNegative() {
                fmt.Printf("error: width %0.2f is less than zero\n", err.width)
            }
            return
        }
        fmt.Println(err)
        return
    }
    fmt.Println("area of rect", area)
}

/*
error: length -5.00 is less than zero  
error: width -9.00 is less than zero  
*/
```

# <span id="panic_recover"> Panic and Recover </span>

### When should panic be used?

One important factor is that you should avoid panic and recover and use errors where ever possible. Only in cases where the program just cannot continue execution should a panic and recover mechanism be used.

* An unrecoverable error where the program cannot simply continue its execution. 
* A programmer error. (一個接收 pointer 參數的 function，卻傳了 nil 給他)

```go
func panic(interface{})  
```

```go
package main

import (  
    "fmt"
)

func fullName(firstName *string, lastName *string) {  
    if firstName == nil {
        panic("runtime error: first name cannot be nil")
    }
    if lastName == nil {
        panic("runtime error: last name cannot be nil")
    }
    fmt.Printf("%s %s\n", *firstName, *lastName)
    fmt.Println("returned normally from fullName")
}

func main() { 
    firstName := "Elon"
    fullName(&firstName, nil)
    fmt.Println("returned normally from main")
}
/*
panic: runtime error: last name cannot be nil

goroutine 1 [running]:
main.fullName(0x1042ff98, 0x0)
	/tmp/sandbox191402046/main.go:12 +0x140
main.main()
	/tmp/sandbox191402046/main.go:20 +0x40
*/
```

### defer

如果有 `defer`，會等所有的 defer 跑完，最後才會執行 panic

```go
package main

import (  
    "fmt"
)

func fullName(firstName *string, lastName *string) {  
    defer fmt.Println("deferred call in fullName")
    if firstName == nil {
        panic("runtime error: first name cannot be nil")
    }
    if lastName == nil {
        panic("runtime error: last name cannot be nil")
    }
    fmt.Printf("%s %s\n", *firstName, *lastName)
    fmt.Println("returned normally from fullName")
}

func main() {  
    defer fmt.Println("deferred call in main")
    firstName := "Elon"
    fullName(&firstName, nil)
    fmt.Println("returned normally from main")
}

/*
deferred call in fullName  
deferred call in main  
panic: runtime error: last name cannot be nil

goroutine 1 [running]:  
main.fullName(0x1042bf90, 0x0)  
    /tmp/sandbox060731990/main.go:13 +0x280
main.main()  
    /tmp/sandbox060731990/main.go:22 +0xc0
*/
```

### Recover

recover is a builtin function which is used to regain control of a panicking goroutine.

> Recover is useful only when called inside deferred functions and  called from the same goroutine.

```go
func recover() interface{}  
```

```go
package main

import (  
    "fmt"
)

func recoverName() {  
    if r := recover(); r!= nil {
        fmt.Println("recovered from ", r)
    }
}

func fullName(firstName *string, lastName *string) {  
    defer recoverName()
    if firstName == nil {
        panic("runtime error: first name cannot be nil")
    }
    if lastName == nil {
        panic("runtime error: last name cannot be nil")
    }
    fmt.Printf("%s %s\n", *firstName, *lastName)
    fmt.Println("returned normally from fullName")
}

func main() {  
    defer fmt.Println("deferred call in main")
    firstName := "Elon"
    fullName(&firstName, nil)
    fmt.Println("returned normally from main")
}

/*
recovered from  runtime error: last name cannot be nil  
returned normally from main  
deferred call in main  
*/
```

### Getting stack trace after recover

There is a way to print the stack trace using the `PrintStack` function of the `Debug` package


```go
package main

import (  
    "fmt"
    "runtime/debug"
)

func r() {  
    if r := recover(); r != nil {
        fmt.Println("Recovered", r)
        debug.PrintStack()
    }
}

func a() {  
    defer r()
    n := []int{5, 7, 4}
    fmt.Println(n[3])
    fmt.Println("normally returned from a")
}

func main() {  
    a()
    fmt.Println("normally returned from main")
}
```

參考文件:

* [golangbot.com](https://golangbot.com/learn-golang-series/)