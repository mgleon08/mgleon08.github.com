---
layout: post
title: "Golang - Interfaces"
date: 2018-05-12 23:39:34 +0800
comments: true
categories: golang
---

<!-- more -->

* [介面 Interfaces](#interfaces)
  * [Stringer Interface](#stringer)
  * [Errors Interface](#errors)
  * [Reader Interface](#reader)
  * [Image Interface](#image)
  * [HTTP interface](#http)

# <span id="interfaces">介面 Interfaces</span>

Golang 本身並沒有泛型程式設計(generic types)，但可以利用 interface 來達成

> 泛型程式設計:   
> 簡單來說就是，編寫的程式碼不是針對特定的類型（比如適用於int, 不適用於string）才有效，而是大部分類型的參數都是可以工作的。

在 Golang 中，interface 是一組 method 的集合，是 [duck typing programming](https://en.wikipedia.org/wiki/Duck_typing) 的一種體現。不關心屬性（數據），只關心行為（方法）


### 實作條件

欲實作的建構體必須要有 Interface 所定義的所有函式、接收參數、回傳值

```go
type Database interface {  
    Read(string) string
    Write(string)
}
// Read 一定要接收一個字串，然後回傳一個字串
// Write 則必須接收一個字串。

type Database interface {  
    Read(name string) string
    Write(data string)
}
// 允許定義 Interface 的時候擺上參數名稱用以辨別，但實作的時候並不需要遵循這個參數名稱。
```

### Why Interface?

> Interface specifies what methods a type should have and the type decides how to implement these methods.

* writing generic algorithm （泛型程式設計）
* hiding implementation detail （隱藏具體實現）
* providing interception points

Golang 裡有兩種 Interface (型態 & 定義)

### 1. 型態

### Empty Interface

`interface{}` 意味著任何型態的值，因為裡面沒有任何 method，也代表所有 type 都 implement


```go
package main

import (
	"fmt"
)

func describe(i interface{}) {
	fmt.Printf("Type = %T, value = %v\n", i, i)
}

func main() {
	var inter interface{}
	describe(inter)
	s := "Hello World"
	describe(s)
	i := 55
	describe(i)
	strt := struct {
		name string
	}{
		name: "Hi",
	}
	describe(strt)
}

/*
Type = <nil>, value = <nil>
Type = string, value = Hello World
Type = int, value = 55
Type = struct { name string }, value = {Hi}
*/
```

### interface{}類型的 slice 是不是就可以接受任何類型的 slice ?

go 不會對 類型是interface{} 的 slice 進行轉換
[InterfaceSlice](https://github.com/golang/go/wiki/InterfaceSlice)

```go
package main

import "fmt"

func printAll(vals []interface{}) { //1
	for _, val := range vals {
		fmt.Println(val)
	}
}
func main() {
	names := []string{"stanley", "david", "oscar"}
	printAll(names)
}

// prog.go:14:10: cannot use names (type []string) as type []interface {} in argument to printAll
```


### Interface value 的賦值

從概念上來講，interface value 有兩部分組成：type 部分是一個 concrete type，vlaue 部分是這個 concrete type 對應的 instance，它們分別稱之為 interface value 的 dynamic type 和 dynamic value。

* [Go interface 詳解 (三) ：interface 的值](http://sanyuesha.com/2017/10/18/go-interface-3/)


### 型態斷言 Type Assertion

* [Go interface 詳解 (四) ：type assertion](http://sanyuesha.com/2017/12/01/go-interface-4/)

Type assertion is used extract the underlying value of the interface.

`i.(T)` is the syntax which is used to get the underlying value of interface `i` whose concrete type is `T`.

`t, ok := i.(T)` i 是 interface 類型的變數，T代表要斷言的類型，t 是 interface 變數存儲的值，ok 是 bool 類型表示是否為該斷言的類型 T

```go
package main

import (
	"fmt"
)

func assert(i interface{}) {
	// Type Switch
	switch v := i.(type) {
	case int:
		fmt.Printf("Twice %v is %v\n", v, v*2)
	case string:
		fmt.Printf("%q is %v bytes long\n", v, len(v))
	default:
		fmt.Printf("I don't know about type %T!\n", v)
	}
}

func main() {
	assert(21)
	assert("hello")
	assert(true)
}

/*
Twice 21 is 42
"hello" is 5 bytes long
I don't know about type bool!
*/
```

##### compare a type to an interface

當一個 type implements 了一個 interface，就可以做比較

```go
package main

import "fmt"

type Describer interface {
	Describe()
}
type Person struct {
	name string
	age  int
}

func (p Person) Describe() {
	fmt.Printf("%s is %d years old", p.name, p.age)
}

func findType(i interface{}) {
	switch v := i.(type) {
	case Describer:
		v.Describe()
	default:
		fmt.Printf("unknown type\n")
	}
}

func main() {
	findType("Naveen")
	p := Person{
		name: "Naveen R",
		age:  25,
	}
	findType(p)
}

/*
unknown type
Naveen R is 25 years old
*/
```

##### 具體類型 vs 介面類型

在 `x.(T)` 當中，`x` 是 interface 介面的表達式，`T`是類型，稱為被斷言類型

> 介面有介面值的概念，其包括 `動態類型 dynamic type` 和 `動態值 dynamic value` 兩部分。

T 有分兩種 `具體類型` `介面類型`

##### 具體類型 concrete type

類型斷言首先檢查 `x` 的動態類型是否是 `T`

* 如果是，類型斷言的結果就是 `x` 的動態值，動態值的 type 就是 T.
* 如果否，操作就會 `panic`
* 對 concrete type 的斷言實際上是獲取 x 的 dynamic value。


```go
// os.Stdout 的類型就是 *os.File
var w io.Writer
w = os.Stdout
f := w.(*os.File) // success: f == os.Stdout
c := w.(*bytes.Buffer) // panic: interface holds *os.File, not *bytes.Buffer
```

##### 介面類型 interface type

類型斷言首先檢查x的動態類型是否滿足T

* 斷言的目的是為了檢測 x 的 dynamic type 是否滿足 T
* 如果是，x 的動態值不會被提取，結果仍然是以前的動態類型和動態值。但結果的類型是介面類型T.
* 換句話説，對介面類型的斷言，結果的類型是T，不是x的類型，改變了類型的表述方式，從而改變了可訪問的方法集合(通常更大)，但會保留x介面值中的動態類型和動態值。

```go
/*
第一個類別型斷言後，
w、rw兩個介面的動態值都是os.Stout，動態類型都是*os.File。

w的類型是io.Writer，只暴露Write方法，
rw的類型是io.ReadWriter,暴露Read和Write方法。
*/
var w io.Writer
w = os.Stdout // w = os.Stdout 賦值 *os.File 類型的 value 給 w
rw := w.(io.ReadWriter) // success: *os.File has both Read and Write
```


##### Testing

* [pointer struct](https://play.golang.org/p/7_uCMhUNGVb)
* [struct](https://play.golang.org/p/1eZxJWA9ok3)
* [interface](https://play.golang.org/p/Uqq6oE7py2Q)

### 型態宣告 Type Casting

若很確定這個 interface{} 是什麼型態，可以直接透過 value.(型態) 來進行型態宣告

倘若該型態不正確，則會出現 `panic` 警告。

```go
package main

import (
	"fmt"
)

func Test(i interface{}) {
	v, k := i.(int)
	fmt.Println(v, k)
}

func Panic(i interface{}) {
	fmt.Println(i.(string))
}

func main() {
	Test("Hi")
	Test(123)
	Panic(111)
}

// 0 false
// 123 true
// panic: interface conversion: interface {} is int, not string
```

### 2. 定義

### Example 1

```go
package main

import (  
    "fmt"
)

type SalaryCalculator interface {  
    CalculateSalary() int //第一個是 method 名稱，第二個是 return type
}

// 長期雇工
type Permanent struct {  
    empId    int
    basicpay int
    pf       int
}

// 合約雇工
type Contract struct {  
    empId  int
    basicpay int
}

// 長期雇工 實作 CalculateSalary()
func (p Permanent) CalculateSalary() int {  
    return p.basicpay + p.pf
}

// 合約雇工實作 CalculateSalary()
func (c Contract) CalculateSalary() int {  
    return c.basicpay
}

/*
total expense is calculated by iterating though the SalaryCalculator slice and summing  
the salaries of the individual employees  
*/

// 必須傳 slice 並且 type 是 SalaryCalculator
func totalExpense(s []SalaryCalculator) {  
    expense := 0
    for _, v := range s {
    	// v 必須都有實作 CalculateSalary()
        expense = expense + v.CalculateSalary()
    }
    fmt.Printf("Total Expense Per Month $%d", expense)
}

func main() {  
    pemp1 := Permanent{1, 5000, 20}
    pemp2 := Permanent{2, 6000, 30}
    cemp1 := Contract{3, 3000}
    employees := []SalaryCalculator{pemp1, pemp2, cemp1}
    totalExpense(employees)
}

// Total Expense Per Month $14050
```

### Implementing interfaces using pointer receivers vs value receivers

```go
package main

import "fmt"

type Describer interface {  
    Describe()
}

type Person struct {  
    name string
    age  int
}

func (p Person) Describe() { //implemented using value receiver  
    fmt.Printf("%s is %d years old\n", p.name, p.age)
}

type Address struct {  
    state   string
    country string
}

func (a *Address) Describe() { //implemented using pointer receiver  
    fmt.Printf("State %s Country %s", a.state, a.country)
}

func main() {  
    var d1 Describer // 聲明 d1 是 Describer(interface) 類型
    p1 := Person{"Sam", 25}
    d1 = p1  // 賦值 p1 到 d1
    d1.Describe()
    p2 := Person{"James", 32}
    d1 = &p2 // 賦值 &p2 到 d1
    d1.Describe()

    var d2 Describer // 聲明 d2 是 Describer(interface) 類型
    a := Address{"Washington", "USA"}
    
    //d2 = a 
    // 當 assign value 時，必須要 implements interface 的內容，否則會 error
    /* 
       cannot use a (type Address) as type Describer
       in assignment: Address does not implement
       Describer (Describe method has pointer
       receiver)
    */
    
    d2 = &a // 賦值 &a 到 d2
    d2.Describe()
}

/*
Sam is 25 years old
James is 32 years old
State Washington Country USA
*/
```

### Embedding interfaces

```go
package main

import (
	"fmt"
)

type SalaryCalculator interface {
	DisplaySalary()
}

type LeaveCalculator interface {
	CalculateLeavesLeft() int
}

type EmployeeOperations interface {
	SalaryCalculator
	LeaveCalculator
}

type Employee struct {
	firstName   string
	lastName    string
	basicPay    int
	pf          int
	totalLeaves int
	leavesTaken int
}

func (e Employee) DisplaySalary() {
	fmt.Printf("%s %s has salary $%d", e.firstName, e.lastName, (e.basicPay + e.pf))
}

func (e Employee) CalculateLeavesLeft() int {
	return e.totalLeaves - e.leavesTaken
}

func main() {
	e := Employee{
		firstName:   "Naveen",
		lastName:    "Ramanathan",
		basicPay:    5000,
		pf:          200,
		totalLeaves: 30,
		leavesTaken: 5,
	}
	var empOp EmployeeOperations = e
	empOp.DisplaySalary()
	fmt.Println("\nLeaves left =", empOp.CalculateLeavesLeft())
}

/*
Naveen Ramanathan has salary $5200
Leaves left = 25
*/
```

##### Example 2

Go語言中也提供了sort函式，原始碼，src/sort/sort.go

裡面定義了一個Interface，包含三個方法：Len(), Less(), Swap()。Interface 作為參數傳遞給Sort。

我們要使用Sort，只需要實現Interface的三個方法就可以使用。

```go
// src/sort/sort.go
package sort

// A type, typically a collection, that satisfies sort.Interface can be
// sorted by the routines in this package.  The methods require that the
// elements of the collection be enumerated by an integer index.
// interface 是一個介面定義，用來讓第三方開發者定義自己的資料庫使用方式，就像一種規範。
type Interface interface {
    // Len is the number of elements in the collection.
    Len() int
    // Less reports whether the element with
    // index i should sort before the element with index j.
    Less(i, j int) bool
    // Swap swaps the elements with indexes i and j.
    Swap(i, j int)
}

...

// Sort sorts data.
// It makes one call to data.Len to determine n, and O(n*log(n)) calls to
// data.Less and data.Swap. The sort is not guaranteed to be stable.
func Sort(data Interface) {
    // Switch to heapsort if depth of 2*ceil(lg(n+1)) is reached.
    n := data.Len()
    maxDepth := 0
    for i := n; i > 0; i >>= 1 {
        maxDepth++
    }
    maxDepth *= 2
    quickSort(data, 0, n, maxDepth)
}
```
```go
package main

import (
    "fmt"
    "sort"
)

type Person struct {
    Name string
    Age  int
}

func (p Person) String() string {
    return fmt.Sprintf("%s: %d", p.Name, p.Age)
}

// ByAge implements sort.Interface for []Person based on
// the Age field.
type ByAge []Person

func (a ByAge) Len() int           { return len(a) }
func (a ByAge) Swap(i, j int)      { a[i], a[j] = a[j], a[i] }
func (a ByAge) Less(i, j int) bool { return a[i].Age < a[j].Age }

func main() {
    people := []Person{
        {"Bob", 31},
        {"John", 42},
        {"Michael", 17},
        {"Jenny", 26},
    }

    fmt.Println(people)
    sort.Sort(ByAge(people))
    fmt.Println(people)
}

// [Bob: 31 John: 42 Michael: 17 Jenny: 26]
// [Michael: 17 Jenny: 26 Bob: 31 John: 42]
```







### <span id="stringer">Stringer Interface</span>

Make the IPAddr type implement fmt.Stringer to print the address as a dotted quad.

> most ubiquitous interfaces is `Stringer` defined by the `fmt` package.

```go
type Stringer interface {
    String() string
}
```

##### [範例 1](tour.golang.org/methods/17)

```go
package main

import "fmt"

type Person struct {
  Name string
  Age  int
}

func (p Person) String() string {
  return fmt.Sprintf("%v (%v years)", p.Name, p.Age)
}

func main() {
  a := Person{"Arthur Dent", 42}
  z := Person{"Zaphod Beeblebrox", 9001}
  fmt.Println(a, z)
}
// Arthur Dent (42 years) Zaphod Beeblebrox (9001 years)s
```

##### [範例 2](https://tour.golang.org/methods/18)

```go
package main

import "fmt"

type IPAddr [4]byte

func (ip IPAddr) String() string {
  return fmt.Sprintf("%v.%v.%v.%v", ip[0], ip[1], ip[2], ip[3])
}

func main() {
   // key 為 string type
   // value 為 IPAddr type 
   // 因此只有 IPAddr 會轉換成上方定義的 func 形式
  hosts := map[string]IPAddr{
    "loopback":  {127, 0, 0, 1},
    "googleDNS": {8, 8, 8, 8},
  }
  for name, ip := range hosts {
    fmt.Printf("%v: %v\n", name, ip)
  }
}
```

### <span id="errors">Errors Interface</span>

The error type is a built-in interface similar to fmt.Stringer

```go
type error interface {
    Error() string
}
```
##### [範例 1](https://tour.golang.org/methods/19)

```go
package main

import (
  "fmt"
  "time"
)

type MyError struct {
  When time.Time
  What string
}

func (e *MyError) Error() string {
  return fmt.Sprintf("at %v, %s",
    e.When, e.What)
}

func run() error {
  return &MyError{
    time.Now(),
    "it didn't work",
  }
}

func main() {
  if err := run(); err != nil {
    fmt.Println(err)
  }
}

// at 2009-11-10 23:00:00 +0000 UTC m=+0.000000001, it didn't work
```

##### [範例 2](https://tour.golang.org/methods/20)

```go
package main

import (
  "fmt"
  "math"
)

type ErrNegativeSqrt float64

func (e ErrNegativeSqrt) Error() string {
  return fmt.Sprintf("Sqrt: negative number %g", e)
}

const delta = 1e-10

func Sqrt(f float64) (float64, error) {
  if f < 0 {
    return 0, ErrNegativeSqrt(f)
  }
  z := f
  for {
    n := z - (z*z-f)/(2*z)
    if math.Abs(n-z) < delta {
      break
    }
    z = n
  }
  return z, nil
}

func main() {
  fmt.Println(Sqrt(2))
  fmt.Println(Sqrt(-2))
}

// 1.4142135623730951 <nil>
// 0 cannot Sqrt negativ number: -2
```

### <span id="reader">Reader Interface</span>

The io package specifies the io.Reader interface, which represents the read end of a stream of data.

[Reader interface](https://golang.org/pkg/io/#Reader)

```go
type Reader interface {
        Read(p []byte) (n int, err error)
}
```

##### [範例 1](https://tour.golang.org/methods/21)

```go
package main

import (
  "fmt"
  "io"
  "strings"
)

func main() {
  r := strings.NewReader("Hello, Reader!")

  b := make([]byte, 8)
  for {
    n, err := r.Read(b)
    fmt.Printf("n = %v err = %v b = %v\n", n, err, b)
    fmt.Printf("b[:n] = %q\n", b[:n])
    // end-of-file (EOF)
    if err == io.EOF {
      break
    }
  }
}

// n = 8 err = <nil> b = [72 101 108 108 111 44 32 82]
// b[:n] = "Hello, R"
// n = 6 err = <nil> b = [101 97 100 101 114 33 32 82]
// b[:n] = "eader!"
// n = 0 err = EOF b = [101 97 100 101 114 33 32 82]
// b[:n] = ""
```

##### [範例 2](https://tour.golang.org/methods/22)

[reader](https://github.com/golang/tour/blob/master/reader/validate.go)

```go
package main

import "golang.org/x/tour/reader"

type MyReader struct{}

func (r MyReader) Read(b []byte) (int, error) {
  for i := range b {
    b[i] = 'A'
  }
  return len(b), nil
}

func main() {
  reader.Validate(MyReader{})
}

// OK!
```

##### [範例 3](https://tour.golang.org/methods/23)

rot13Reader
s 
> Wiki [`ROT13`](https://en.wikipedia.org/wiki/ROT13)

```go
package main

import (
  "io"
  "os"
  "strings"
)

// 轉換 ROT13
func rot13(b byte) byte {
  var a, z byte
  switch {
  // 判斷是小寫還是大寫的範圍
  case 'a' <= b && b <= 'z': 
    a, z = 'a', 'z'
  case 'A' <= b && b <= 'Z':
    a, z = 'A', 'Z'
   // 特殊符號就直接回傳
  default:
    return b 
  }
  // a & b 會自動轉成 ASCII 碼數字
  return (b-a+13)%(z-a+1) + a
}

// 建立一個 rot13Reader 並且有 Read method
type rot13Reader struct {
  rr io.Reader
}

func (r rot13Reader) Read(p []byte) (n int, err error) {
// n 為 s 字串的長度
  n, err = r.rr.Read(p)
  for i := 0; i < n; i++ {
    p[i] = rot13(p[i])
  }
  return
}

func main() {
  s := strings.NewReader(
    "Lbh penpxrq gur pbqr!")
  r := rot13Reader{s}
  io.Copy(os.Stdout, &r)
}

// You cracked the code!
```

### <span id="image">Image Interface</span>

[Image Interface](https://golang.org/pkg/image/#Image)

```go
package image

type Image interface {
    ColorModel() color.Model
    Bounds() Rectangle
    At(x, y int) color.Color
}
```

##### [範例 1](https://tour.golang.org/methods/25)

```go
package main

import (
	"code.google.com/p/go-tour/pic"
	"image"
	"image/color"
)

type Image struct {
	width  int
	height int
}

func (img Image) ColorModel() color.Model {
	return color.RGBAModel
}

func (img Image) Bounds() image.Rectangle {
	return image.Rect(0, 0, img.width, img.height)
}

func (img Image) At(x, y int) color.Color {
	img_func := func(x, y int) uint8 {
		//return uint8(x*y)
		//return uint8((x+y) / 2)
		return uint8(x ^ y)
	}
	v := img_func(x, y)
	return color.RGBA{v, v, 255, 255}
}

func main() {
	m := Image{256, 64}
	pic.ShowImage(m)
}
```
### <span id="http"> HTTP Interface</span>

[HTTP Interface](https://golang.org/pkg/net/http/)

```go
package http

type Handler interface {
    ServeHTTP(w ResponseWriter, r *Request)
}
```

##### 範例 1

```go
package main

import (
	"fmt"
	"net/http"
)

type Hello struct{}

func (h Hello) ServeHTTP(
	w http.ResponseWriter,
	r *http.Request) {
	fmt.Fprint(w, "Hello!")
}

func main() {
	var h Hello
	http.ListenAndServe("localhost:4000", h)
}
```

##### 範例 2

```go
package main

import (
    "fmt"
    "net/http"
)

type String string

func (s String) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, s)
}


type Struct struct {
    Greeting string
    Punct string
    Who string
}

func (s *Struct) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, s.Greeting, s.Punct, s.Who)
}


func main() {
    http.Handle("/string", String("I'm a frayed knot."))
    http.Handle("/struct", &Struct{"Hello", ":", "Gophers!"})
    http.ListenAndServe("localhost:4000", nil)
}
```

參考文件:

* [golangbot.com](https://golangbot.com/learn-golang-series/)
* [[golangbot.com] polymorphism](https://golangbot.com/polymorphism/)
* [解釋 Golang 中的 Interface 到底是什麼](https://yami.io/golang-interface/)
* [深入理解 Go Interface](http://legendtkl.com/2017/06/12/understanding-golang-interface/)
* [go"泛型程式設計"](http://legendtkl.com/2015/11/25/go-generic-programming/)
* [談一談Go的interface和reflect](http://legendtkl.com/2015/11/28/go-interface-reflect/)
* [理解 go interface 的 5 個關鍵點](http://sanyuesha.com/2017/07/22/how-to-understand-go-interface/)
* [Go interface 詳解(一) ：介紹](http://sanyuesha.com/2017/10/10/go-interface-1/)
* [Go interface 詳解(二) ：定義和使用](http://sanyuesha.com/2017/10/12/go-interface-2/)
* [Go interface 詳解 (三) ：interface 的值](http://sanyuesha.com/2017/10/18/go-interface-3/)
* [Go interface 詳解 (四) ：type assertion](http://sanyuesha.com/2017/12/01/go-interface-4/)
* [[video] Google Understanding Go Interfaces](https://www.youtube.com/watch?v=F4wUrj6pmSI&t=2319s)
* [[slider] Google Understanding Go Interfaces](https://github.com/gopherchina/conference/blob/master/2017/1.4%20interface.presented.pdf)
* [7.10. 類型斷言](http://gobook.io/read/github.com/lunny/gopl-zh/ch7/ch7-10.html)
* [Go 的類型斷言type assertion](https://hk.saowen.com/a/07c058b7f0424b619737a3cf71710500b3d493edb3f72b80e2478b2f43b631b1)
* [Go Data Structures: Interfaces](https://research.swtch.com/interfaces)