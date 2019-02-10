---
layout: post
title: "Golang - Interfaces"
date: 2018-05-12 23:39:34 +0800
comments: true
categories: golang
---

<!-- more -->

* [介面 Interfaces](#interfaces)
* [Why Interface?](#why_interface)
* [Practical use of interface](#practical_use_of_interface)
* [Interface internal representation](#interface_internal_representation)
* [Empty Interface](#empty_interface)
* [Type Assertion](#type_assertion)
* [Type Switch](#type_switch)
* [Implementing interfaces using pointer receivers vs value receivers](#pointer_receivers_vs_value_receivers)
* [Implementing multiple interfaces](#implementing_multiple_interfaces)
* [Embedding interfaces](#embedding_interfaces)
* [範例: 實作 sort interface](#sort_interface)
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

### <span id="why_interface"> Why Interface? </span>

簡單的來說就是 `interface defines the behaviour of an object` 當一個類型提供了 interface 定義的所有 method，就可以說這個類型實作了這個 interface

interface 指定 type 應該要有的方法， type 決定如何實現這些方法

> 像是 WashingMachine 的 interface，包含了 Cleaning() 和 Drying()，任何有提供 Cleaning() 和 Drying() 的 type，都可以說是實作了 WashingMachine 的 interface

* writing generic algorithm （泛型程式設計）
* hiding implementation detail （隱藏具體實現）
* providing interception points

##### Declaring and implementing an interface


定義了 VowelsFinder 的 interface，裡面有個 `FindVowels` 的 method，因此 MyString 的 type，提供了 `FindVowels` method，因此可以說 MyString 實作了 VowelsFinder 的 interface

```go
package main

import (
    "fmt"
)

//interface definition
type VowelsFinder interface {
    FindVowels() []rune // 必須回傳 []rune type
}

type MyString string

//MyString implements VowelsFinder
func (ms MyString) FindVowels() []rune {
    var vowels []rune
    for _, rune := range ms {
        if rune == 'a' || rune == 'e' || rune == 'i' || rune == 'o' || rune == 'u' {
            vowels = append(vowels, rune)
        }
    }
    return vowels
}

func main() {
    name := MyString("Sam Anderson")
    var v VowelsFinder // 宣告 v 是 VowelsFinder interface type
    v = name // possible since MyString implements VowelsFinder
    fmt.Printf("Vowels are %c", v.FindVowels())

}

// Vowels are [a e o]
```

### <span id="practical_use_of_interface"> Practical use of interface </span>

上面範例比較無法體現出為什麼要用 interface，即使用 `name.FindVowels()` 也是可以，以下是比較實際上的用法

> If a variable has an interface type, then we can call methods that are in the named interface.

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

// 必須傳 slice 並且 type 是 SalaryCalculator interface
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

好處在於，當之後有新增新的員工，薪資計算方式也不一樣，只需要新增新的 struct 並實作 `CalculateSalary()`，就加到 `[]SalaryCalculator` 上面就可以直接擴充了

### <span id="interface_internal_representation"> Interface internal representation </span>

An interface can be thought of as being represented internally by a tuple `(type, value)`

`type` is the underlying concrete type of the interface and value holds the value of the concrete type.

```go
package main

import (
    "fmt"
)

type Test interface {
    Tester()
}

type MyFloat float64

func (m MyFloat) Tester() {
    fmt.Println(m)
}

func describe(t Test) {
    fmt.Printf("Interface type %T value %v\n", t, t)
}

func main() {
    var t Test
    f := MyFloat(89.7)
    t = f // 這裡將 type 是 MyFloat assign 給 type 是 Test interface 的 t
    describe(t) // 但印出 type 的時候會是原本底層具體的 type MyFloat 而不是 Test
    t.Tester()
}

// Interface type main.MyFloat value 89.7
// 89.7
```

### <span id="empty_interface"> Empty Interface </span>

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

##### interface{} 類型的 slice 是不是就可以接受任何類型的 slice ?

go 不會對類型是interface{} 的 slice 進行轉換
[InterfaceSlice](https://github.com/golang/go/wiki/InterfaceSlice)

* 因為 `[]interface{}` 並不是 interface 而是 slice

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

### <span id="type_assertion"> Type Assertion </span>

Type assertion 可以用來取出 interface 底層的 value

```go
i.(T)
// used to get the underlying value of interface i whose concrete type is T.
```

```go
package main

import (
    "fmt"
)

func assert(i interface{}) {
    s := i.(int) //get the underlying int value from i
    // 如果改成 string 則會 error
    // panic: interface conversion: interface {} is int, not string
    fmt.Println(s)
}
func main() {
    var s interface{} = 56
    assert(s)
}

// 56
```

`t, ok := i.(T)` i 是 interface 類型的變數，T代表要斷言的類型，t 是 interface 變數存儲的值，ok 是 bool 類型表示是否為該斷言的類型 T


```go
package main

import (
    "fmt"
)

func assert(i interface{}) {
    v, ok := i.(int) // 如果底層的 type 是 int，就會回傳 value & true
    fmt.Println(v, ok)
}
func main() {
    var s interface{} = 56
    assert(s)
    var i interface{} = "Steven Paul"
    assert(i)
}
// 56 true
// 0 false
```

### <span id="type_switch"> Type Switch </span>

當有多個或不確定的底層 type 時，可以用 switch 去做個別處理

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

實現該 interface 的 type，也可以用 interface 來做比對

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
	case Describer: // 這邊可以用 Describer or Person
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

### <span id="pointer_receivers_vs_value_receivers"> Implementing interfaces using pointer receivers vs value receivers </span>

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
    // 原因在於 a 並沒有實作 Describe()，實作 Describe() 的是 pointer receiver，因此不能說 a 實作 Describer interface
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

### <span id="implementing_multiple_interfaces"> Implementing multiple interfaces </span>


type 可以實作多個 interface

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

type Employee struct {
    firstName string
    lastName string
    basicPay int
    pf int
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
    e := Employee {
        firstName: "Naveen",
        lastName: "Ramanathan",
        basicPay: 5000,
        pf: 200,
        totalLeaves: 30,
        leavesTaken: 5,
    }
    var s SalaryCalculator = e
    s.DisplaySalary()
    var l LeaveCalculator = e
    fmt.Println("\nLeaves left =", l.CalculateLeavesLeft())
}

// Naveen Ramanathan has salary $5200
// Leaves left = 25
```

### <span id="embedding_interfaces"> Embedding interfaces </span>

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

// 跟上面比較，多了一個這個
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

### <span id="sort_interface"> 範例: 實作 sort interface </span>

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

// Person struct method String()
func (p Person) String() string {
    return fmt.Sprintf("%s: %d", p.Name, p.Age)
}

// ByAge implements sort.Interface for []Person based on
// the Age field.
type ByAge []Person // Declare a method on non-struct types.

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
    sort.Sort(ByAge(people)) // ByAg(people) because non-struct types
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

##### [範例 1](https://tour.golang.org/methods/17)

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
