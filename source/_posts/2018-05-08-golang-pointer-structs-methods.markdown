---
layout: post
title: "Golang - pointer, structs, methods"
date: 2018-05-08 23:55:02 +0800
comments: true
categories: golang
---

<!-- more -->

* [指標 Pointer](#pointer)
* [結構體 Structs](#structs)
	* [Anonymous structs](#anonymous_structs)
	* [Pointers to structs](#pointers_to_structs)
	* [Nested structs](#nested_structs)
	* [Promoted fields](#promoted_fields)
	* [New struct](#new_struct)
	* [Structs Equality](#equality)
	* [組合 combination](#combination)
	* [建構子 Constructors](#constructors)
* [方法 methods](#methods)
	* [Pointer receivers](#pointer_receivers)
	* [receivers in methods vs arguments in functions](#receivers)
	* [function use pointer or value ??](#pointer_or_value)

# <span id="pointer">指標 Pointer</span>

> A pointer holds the memory address of a value
> 將變數直接指向記憶體位置就叫做Pointer，要修改內容就直接到該記憶體位置修改

* The type `*T` is a pointer to a T value. Its zero value is `nil`.
* The `&` operator generates a pointer to its operand.
* Do not pass a pointer to an array as a argument to a function. Use slice instead.
* Go does not support pointer arithmetic(pointer 位置運算)
* 使用 value receivers，function 會用 copy 的方式，並且改不到原本的值，pointer receivers 則會使用 reference 且會改到原本的值，比較不會浪費記憶體

```go
package main

import (
	"fmt"
)

func main() {
	var i = 8  // i 佔用了一個記憶體空間
	var p *int // 宣告 p 是一個 int 的指標，但此時要指向哪還不知道
	fmt.Printf("Type of p is %T\n", p)
	fmt.Printf("Type of &i is %T\n", &i)
	fmt.Println("p =", p) // The zero value of a pointer is nil

	p = &i          // 將 p 指到 i 的記憶體位置
	fmt.Println("p =", p)  // p 所指到的記憶體位置，就是i
	fmt.Println("&p =", &p) // p 的記憶體位置
	fmt.Println("*p =", *p) // '*' 代表透過 pointer 顯示該記憶體位置的值

	*p = 20         // 透過 pointer 寫入 i 的值
	fmt.Println("*p =", *p)
	fmt.Println("i =", i)
}

/**
Type of p is *int
Type of &i is *int
p = <nil>
p = 0x10414020
&p = 0x1040c128
*p = 8
*p = 20
i = 20
**/
// 當用 * 宣告為指標時，必須再去指向記憶體位置 &
```

### Passing pointer to a function

```go
package main

import (
	"fmt"
)

func change(val *int) {
	*val = 55
}
func main() {
	a := 58
	fmt.Println("value of a before function call is", a)
	b := &a
	change(b)
	fmt.Println("value of a after function call is", a)
}

/**
value of a before function call is 58
value of a after function call is 55
**/
```

* [30天就Go(10)：Pointer](https://ithelp.ithome.com.tw/articles/10187607)
* [[golangbot.com] pointers](https://golangbot.com/pointers/)

# <span id="structs">結構體 Structs</span>

Golang 沒有類別 class，但有建構體 struct
> * 在 Structs 中我們可以為不同項，定義不同的數據類型
> * Structs 是由一系列具有相同類型或不同類型的數據構成的數據集合
> * 且沒有 public、private、protected 的種類
> * structs 是可以比較的，但如果裡面的值是不可比較的 (map)，那就不可以比較

### Declaring a structure

For instance a employee has a firstName, lastName and age. It makes sense to group these three properties into a single structure employee.

```go
package main

import (
	"fmt"
)

type Employee struct {
	firstName, lastName string
	age, salary         int
}

func main() {

	//creating structure using field names
	emp1 := Employee{
		firstName: "Sam",
		age:       25,
		salary:    500,
		lastName:  "Anderson",
	}

	//creating structure without using field names
	emp2 := Employee{"Thomas", "Paul", 29, 800}

	fmt.Println("Employee 1", emp1)
	fmt.Println(emp1.firstName, emp1.age, emp1.salary, emp1.lastName)
	fmt.Println("Employee 2", emp2)
	fmt.Println(emp2.firstName, emp2.age, emp2.salary, emp2.lastName)

}

/**
Employee 1 {Sam Anderson 25 500}
Sam 25 500 Anderson
Employee 2 {Thomas Paul 29 800}
Thomas 29 800 Paul
**/
```

### <span id="anonymous_structs"> Anonymous structs </span>

```go
package main

import (
	"fmt"
)

func main() {
	emp3 := struct {
		firstName, lastName string
		age, salary         int
	}{
		firstName: "Andreah",
		lastName:  "Nikola",
		age:       31,
		salary:    5000,
	}

	fmt.Println("Employee 3", emp3)
}

// Employee 3 {Andreah Nikola 31 5000}
```

### Anonymous fields

`Anonymous fields` 因為沒有 name，預設會將 type 當作是自己的 name


```go
package main

import (
    "fmt"
)

type Person struct {
    string
    int
}

func main() {
    var p Person
    // or p := Person{"Naveen", 50}
    p.string = "leon"
    p.int = 50
    fmt.Println(p)
}

// {leon 50}
```

### <span id="pointers_to_structs"> Pointers to structs </span>

```go
package main

import (
	"fmt"
)

type Vertex struct {
	X int
	Y int
}

func main() {
	v := Vertex{1, 2}
	p := &v
	p.X = 1e9 // 等同於 (*p).X，但這樣太複雜，因此允許可以只用 p.X
	fmt.Println(v)

	var (
		v1 = Vertex{1, 2}  // has type Vertex
		v2 = Vertex{X: 1}  // Y:0 is implicit
		v3 = Vertex{}      // X:0 and Y:0
		p2 = &Vertex{1, 2} // has type *Vertex
	)

	fmt.Println(v1, v2, v3, p2, *p2)

}

/**
{1000000000 2}
{1 2} {1 0} {0 0} &{1 2} {1 2}
**/
```

### <span id="nested_structs"> Nested structs </span>

```go
package main

import (
    "fmt"
)

type Address struct {
    city, state string
}

type Person struct {
    name string
    age int
    address Address
}

func main() {
    var p Person
    p.name = "Naveen"
    p.age = 50
    p.address = Address {
        city: "Chicago",
        state: "Illinois",
    }
    fmt.Println("Name:", p.name)
    fmt.Println("Age:",p.age)
    fmt.Println("City:",p.address.city)
    fmt.Println("State:",p.address.state)
}

/**
Name: Naveen
Age: 50
City: Chicago
State: Illinois
**/
```


### <span id="promoted_fields"> Promoted fields </span>

當 struct 裡面有 `anonymous struct field`，就叫做 `promoted fields`

```go
package main

import (
    "fmt"
)

type Address struct {
    city, state string
}

type Person struct {
    name string
    age  int
    Address // anonymous struct field 因此 name 就會是 Address
}

func main() {
    var p Person
    p.name = "Naveen"
    p.age = 50
    p.Address = Address{
        city:  "Chicago",
        state: "Illinois",
    }
    fmt.Println("Name:", p.name)
    fmt.Println("Age:", p.age)
    fmt.Println("City:", p.city) //city is promoted field
    fmt.Println("State:", p.state) //state is promoted field
}
```

### <span id="new_struct"> New struct </span>

```go
package main

import (
	"fmt"
)

type Vertex struct {
	X, Y int
}

func main() {
	v := new(Vertex)
	// 相當於 var v *Vertex = new(Vertex)
	// 宣告 v 是一個 *Vertex 的指標，並指向 Vertex struct
	fmt.Println(v)
	v.X, v.Y = 11, 9
	fmt.Println(v)
}

// &{0 0}
// &{11 9}
```

### <span id="equality"> Structs Equality </span>

如果 structs 的 fields 是相等的話，就可以比較，還有裡面的 type 是可以比較的(像是  maps 就無法做比較)

```go
package main

import (
    "fmt"
)

type name struct {
    firstName string
    lastName string
}


func main() {
    name1 := name{"Steve", "Jobs"}
    name2 := name{"Steve", "Jobs"}
    if name1 == name2 {
        fmt.Println("name1 and name2 are equal")
    } else {
        fmt.Println("name1 and name2 are not equal")
    }

    name3 := name{firstName:"Steve", lastName:"Jobs"}
    name4 := name{}
    name4.firstName = "Steve"
    if name3 == name4 {
        fmt.Println("name3 and name4 are equal")
    } else {
        fmt.Println("name3 and name4 are not equal")
    }
}

// name1 and name2 are equal
// name3 and name4 are not equal
```

### <span id="combination">組合 combination</span>

golang 沒有繼承，但有組合

* [Composition Instead of Inheritance - OOP in Go](https://golangbot.com/inheritance/)

```go
package main

import (
	"fmt"
)

type author struct {
	firstName string
	lastName  string
	bio       string
}

func (a author) fullName() string {
	return fmt.Sprintf("%s %s", a.firstName, a.lastName)
}

type post struct {
	title   string
	content string
	// Anonymous fields
	// Go會將嵌入字段預設作為屬性名
	author
}

func (p post) details() {
	fmt.Println("Title: ", p.title)
	fmt.Println("Content: ", p.content)
	fmt.Println("Author: ", p.fullName())
	fmt.Println("Bio: ", p.bio)
}

type website struct {
	posts []post
}

func (w website) contents() {
	fmt.Println("Contents of Website")
	for _, v := range w.posts {
		v.details()
		fmt.Println()
	}
}

func main() {
	author1 := author{
		"Naveen",
		"Ramanathan",
		"Golang Enthusiast",
	}
	post1 := post{
		"Inheritance in Go",
		"Go supports composition instead of inheritance",
		author1,
	}
	post2 := post{
		"Struct instead of Classes in Go",
		"Go does not support classes but methods can be added to structs",
		author1,
	}
	post3 := post{
		"Concurrency",
		"Go is a concurrent language and not a parallel one",
		author1,
	}
	w := website{
		posts: []post{post1, post2, post3},
	}
	w.contents()
}
```

### <span id="constructors">建構子 Constructors</span>

Golang 裡因為沒有類別，也就沒有建構子，但可以自己在外部建立一個建構用函式。

```go
package main

import (
	"fmt"
)

type Test struct {
	a string
}

func (t *Test) show() {
	fmt.Println(t.a)
}

// 用來建構 Test 的假建構子
func newTest() (test *Test) {
	test = &Test{a: "foobar"}
	// 這裡會回傳一個型態是 *Test 建構體的 test 變數
	return
}

func main() {
	b := newTest()
	b.show() // 輸出：foobar
}

// foobar
```

##### Example 2

```go
// employee/employee.go
package employee

import (
    "fmt"
)

type employee struct {  // 改成小寫，不需要外面呼叫，必須都要透過 New 來產生
    firstName   string
    lastName    string
    totalLeaves int
    leavesTaken int
}

func New(firstName string, lastName string, totalLeave int, leavesTaken int) employee {
    e := employee {firstName, lastName, totalLeave, leavesTaken}
    return e
}

func (e employee) LeavesRemaining() {
    fmt.Printf("%s %s has %d leaves remaining", e.firstName, e.lastName, (e.totalLeaves - e.leavesTaken))
}
```

```go
// main.go
package main

import "oop/employee"

func main() {
    e := employee.New("Sam", "Adolf", 30, 20)
    e.LeavesRemaining()
}
```

# <span id="methods"> methods </span>

Golang 中 structs 的成員還有方法都是在 structs 外面所定義的

> 在普通函式前面加個接受者（receiver，寫在函式名前面的括號裡面），這樣編譯器就知道這個函式（方法）屬於哪個struct了。


> Remember: a method is just a function with a receiver argument


* golang 並不完全屬於[物件導向語言](https://golang.org/doc/faq#Is_Go_an_object-oriented_language)，但是透過 methods 和 types 使行為像 class 一樣
* function 可以達成跟 methods 一樣的方法，但是 function 不允許有同樣的名稱，methods 可以，只要是不同的 struct
* function 通常只接收一種接受者 receiver，但 methods 可以接受 `value receiver` & `pointer receiver`

```go
// 定義
func (r ReceiverType) funcName(parameters) {
}
// 形象一點說，就是 ReceiverType 類型的所有字段，方法 funcName 都是可以使用的，可以認為 funcName 屬於 ReceiverType
```

```go
package main

import (
	"fmt"
	"math"
)

type Vertex struct {
	X, Y float64
}

func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
	ver := Vertex{3, 4}
	fmt.Println(ver.Abs())
}


// 5
```

###  Declare a method on non-struct types, too.

```go
package main

import (
	"fmt"
	"math"
)

type MyFloat float64 // create a type alias for the built-in type float64s

func (f MyFloat) Abs() float64 {
	if f < 0 {
		return float64(-f)
	}
	return float64(f)
}

func main() {
	f := MyFloat(-math.Sqrt2)
	fmt.Println(f.Abs())
}

// 1.4142135623730951
```


### <span id="pointer_receivers"> Pointer receivers </span>

兩個原因為什麼要用 pointer receiver?

* 更改指標指向的值
* 避免重複每個 method 使用的值

```go
package main

import (
	"fmt"
)

// // 先定義一個 Foobar 建構體，然後有個叫做 a 的字串成員
type Foobar struct {
	a string
}

// 定義一個屬於 Foobar 的 test 方法
func (f *Foobar) test() {
	// 接收來自 Foobar 的 a
	fmt.Println(f.a)
}
func main() {
	a := &Foobar{a: "a: hello, world!"}
	var b = Foobar{a: "b: hello, world!"}
	var c = &Foobar{a: "c: hello, world!"}
	var d *Foobar = &Foobar{a: "d: hello, world!"} // 將 d 宣告為 Foobar 指標
	a.test()
	(&b).test()
	b.test() // 相當於 (&b).test()
	c.test()
	d.test()
}

/**
a: hello, world!
b: hello, world!
b: hello, world!
c: hello, world!
d: hello, world!
**/
```

##### Pointer receive to change value

```go
package main

import (
	"fmt"
)

type Vertex struct {
	X, Y float64
}

func (v Vertex) Abs() float64 {
	return v.X + v.Y
}

func (v *Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func main() {
	v := Vertex{3, 4}
	v.Scale(10)
	fmt.Println(v.Abs())
}

// 70
// 改成沒有 *,  output 會變為 7
// 有 * 用指標，才會更改原本的 X, Y
```

##### rewritten as functions

```go
package main

import (
	"fmt"
)

type Vertex struct {
	X, Y float64
}

func Abs(v Vertex) float64 {
	return v.X + v.Y
}

func Scale(v *Vertex, f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func main() {
	v := Vertex{3, 4}
	Scale(&v, 10)
	fmt.Println(Abs(v))
}

// 70
```

### Methods and pointer indirection

```go
package main

import (
	"fmt"
)

type Vertex struct {
  X, Y float64
}

func (v *Vertex) Scale(f float64) {
  v.X = v.X * f
  v.Y = v.Y * f
}

func ScaleFunc(v *Vertex, f float64) {
  v.X = v.X * f
  v.Y = v.Y * f
}

func (v Vertex) Abs() float64 {
    return v.X + v.Y
}

func AbsFunc(v Vertex) float64 {
    return v.X + v.Y
}

func main() {
  v := Vertex{3, 4}
  v.Scale(2) // (&v).Scale(2)
  ScaleFunc(&v, 10)

  p := &Vertex{4, 3}
  p.Scale(3)
  ScaleFunc(p, 8)

  fmt.Println(v, p)

  a := Vertex{3, 4}
  fmt.Println(a.Abs())
  fmt.Println(AbsFunc(a))

  b := &Vertex{4, 3}
  fmt.Println(b.Abs()) // (*b).Abs()
  fmt.Println(AbsFunc(*b))
}

// {60 80} &{96 72}
// 7
// 7
// 7
// 7
```

### <span id="receivers"> receivers in methods vs arguments in functions </span>

* When a function has a `value/point argument`, it will accept only a value argument.
* When a method has a `value/point` receiver, it will accept both pointer and value receivers.

> * 原因是在於 `p.area()` 會自動解讀為 `(*p).area()`，因此實際上 value receiver 也是只能接收 value
> * 而 Pointer receivers 也是一樣 `r.perimeter()` 會解讀為 `(&r).perimeter()`

##### value

```go
package main

import (
    "fmt"
)

type rectangle struct {
    length int
    width  int
}

func area(r rectangle) {
    fmt.Printf("Area Function result: %d\n", (r.length * r.width))
}

func (r rectangle) area() {
    fmt.Printf("Area Method result: %d\n", (r.length * r.width))
}

func main() {
    r := rectangle{
        length: 10,
        width:  5,
    }
    area(r)
    r.area()

    p := &r
    /*
       compilation error, cannot use p (type *rectangle) as type rectangle
       in argument to area
    */
    //area(p)

    p.area()//calling value receiver with a pointer
}

// Area Function result: 50
// Area Method result: 50
// Area Method result: 50
```

##### point

```go
package main

import (
    "fmt"
)

type rectangle struct {
    length int
    width  int
}

func perimeter(r *rectangle) {
    fmt.Println("perimeter function output:", 2*(r.length+r.width))

}

func (r *rectangle) perimeter() {
    fmt.Println("perimeter method output:", 2*(r.length+r.width))
}

func main() {
    r := rectangle{
        length: 10,
        width:  5,
    }
    p := &r //pointer to r
    perimeter(p)
    p.perimeter()

    /*
        cannot use r (type rectangle) as type *rectangle in argument to perimeter
    */
    //perimeter(r)

    r.perimeter()//calling pointer receiver with a value

}

// perimeter function output: 30
// perimeter method output: 30
// perimeter method output: 30
```

### <span id="methods_on_non_struct_types"> Methods on non struct types </span>

method 必須定義才 local type 才可以，因此像要再 `int` 新增 method 則會出現 error `cannot define new methods on non-local type int`，必須定義新的 type 才行

```go
package main

import "fmt"

type myInt int

// 不可以直接用 int
func (a myInt) add(b myInt) myInt {
    return a + b
}

func main() {
    num1 := myInt(5)
    num2 := myInt(10)
    sum := num1.add(num2)
    fmt.Println("Sum is", sum)
}

// Sum is 15
```

### <span id="pointer_or_value">function use pointer or value ??</span>

> 先說結論: 如果只是要讀值，可以使用 Value 或 Pointer 方式，但是要寫入，則只能用 Pointer 方式。

### 寫入或讀取

果需要對 Struct 內的成員進行修改，那請務必使用 Pointer 傳值，相反的，Go 會使用 Copy struct 方式來傳入，但是用此方式你就拿不到修改後的資料。

### 效能
假設 Struct 內部成員非常的多，請務必使用 Pointer 方式傳入，這樣省下的系統資源肯定比 Copy Value 的方式還來的多。

### 一致性
在開發團隊內，如果有人使用 Pointer 有人使用 Value 方式，這樣寫法不統一，造成維護效率非常低，所以官方建議，全部使用 Pointer 方式是最好的寫法。

* [Go 語言內 struct methods 該使用 pointer 或 value 傳值?](https://blog.wu-boy.com/2017/05/go-struct-method-pointer-or-value/)
* [Should I define methods on values or pointers?](https://golang.org/doc/faq#methods_on_values_or_pointers)

參考文件:

* [golangbot.com](https://golangbot.com/learn-golang-series/)
