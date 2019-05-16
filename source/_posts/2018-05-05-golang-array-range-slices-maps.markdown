---
layout: post
title: "Golang - array, range, slices, maps"
date: 2018-05-05 22:17:57 +0800
comments: true
categories: golang
---

<!-- more -->

* [陣列 Array ](#array)
* [Range (each)](#range)
* [Slices](#slices)
* [Maps](#maps)
* [Variadic Functions](#variadic)

# <span id="array"> 陣列 Array </span>

> * Array 的長度是其類型的一部分，因此陣列不能改變大小。
> * 當定義為 int Array，之後就只能放 int，不能放其他 type
> * int default 值是 0, string 是 nil
> * 當設定一定長度時，內容不一定要填滿，但不能超過當初定義的長度。
> * size 不一樣的 Array 是不同的 Type

```go
var a [10]int // 宣告一個變數 a 為一個 int type 的 Array 並且長度只有10.(int 預設是 0, string 則為 nil)

a := [3]int{1}
// [1 0 0]

// 宣告一個變數 ｂ 為 string type, 並且裡面的值是 A, B 並且長度只有 2
b := [2]string{"A", "B"}
b := [...]string{"A", "B"} // ... makes the compiler determine the length

a := [3][2]string // Multidimensional arrays

a := [3][2]string{
        {"lion", "tiger"},
        {"cat", "dog"},
        {"pigeon", "peacock"}, //this comma is necessary. The compiler will complain if you omit this comma
    }

s := [6]struct {
    i int
    b bool
  }{
    {2, true},
    {3, false},
    {5, true},
    {7, true},
    {11, false},
    {13, true},
  }
  
// 透過 1: 2: 可指定 value 要塞的位置
b := [3]int{
	0:111,
	1:222,
	2:333,
}
// [111 222 333]

b := [...]int{
	3:333,
}
// [0 0 0 333]

b := [...]int{
	5:111,
	222,
	0:333,
}
// [333 0 0 0 0 111 222]
```

```go
package main

import "fmt"

func main() {
	var s [2]string
	s[0] = "Hello"
	s[1] = "World"
	fmt.Println(s)

	primes := [6]int{2, 3, 5, 7, 11, 13}
	fmt.Println(primes)

	a := [...]float64{67.7, 89.8, 21, 78}
	for i := 0; i < len(a); i++ {
		fmt.Printf("%d th element of a is %.2f\n", i, a[i])
	}
}

/**
[Hello World]
[2 3 5 7 11 13]
0 th element of a is 67.70
1 th element of a is 89.80
2 th element of a is 21.00
3 th element of a is 78.00
**/
```

### Passing a array to a function

傳遞給 function 不會改變原來的值，因為 array 不是 reference types 而是 value types

```go
package main

import "fmt"

func changeLocal(num [5]int) {
    num[0] = 55
    fmt.Println("inside function ", num)

}
func main() {
    num := [...]int{5, 6, 7, 8, 8}
    fmt.Println("before passing to function ", num)
    changeLocal(num) //num is passed by value
    fmt.Println("after passing to function ", num)
}

/**
before passing to function  [5 6 7 8 8]
inside function  [55 6 7 8 8]
after passing to function  [5 6 7 8 8]
**/
```

#<span id="range"> Range (each) </span>

可以達成 foreach 的方式

```go
func main() {
  data := []string{"a", "b", "c"}

  for index, value := range data {
    fmt.Printf("%d%s|", index, value) // 輸出：0a|1b|2c|
  }

  for index := range data {
    fmt.Printf("%d|", index) // 輸出：0|1|2|
  }

  for _, value := range data {
    fmt.Printf("%s|", value) // 輸出：a|b|c|
  }
}
```

如果是在 map，就會是 key, value

```go
func main() {
	kvs := map[string]string{"a": "apple", "b": "banana"}
	for k, v := range kvs {
		fmt.Printf("%s -> %s\n", k, v)
	}
}
```

也可以只要 index or key

```go
func main(){
    kvs := map[string]string{"a": "apple", "b": "banana"}
	for k := range kvs {
		fmt.Println("key:", k)
	}
}
```

當是 string 時，則是會轉成 Unicode code `rune` type

```go
func main() {
	// `range` on strings iterates over Unicode code
	// points. The first value is the starting byte index
	// of the `rune` and the second the `rune` itself.
	for i, c := range "go" {
		fmt.Println(i, c)
	}
}

// 0 103
// 1 111
```

# <span id="slices"> Slices </span>

不用定義其最大長度，而且可以直接賦予值

> * An array has a fixed size. A slice, on the other hand, is a dynamically-sizeds
> * slice 的零值是 nil，一個 nil 的 slice 的長度和容量是 0，使用 make 就不會是 nil
> * Slice 是用 reference

* nil slice 和 empty slice 是不一樣的，但 length 都一樣是 0
* slice 只能跟 nil 去做比較，兩個 slice 要比較，必須要用 loop 去一個一個比對

```go
package main

import (
	"fmt"
)

func main() {
	var slice1 []string
	slice2 := []string{}

	fmt.Printf("%#v, %#v\n", slice1, slice2)
	fmt.Printf("%t\n", len(slice1) == len(slice2))
	fmt.Printf("%t\n", slice1 == nil)
	fmt.Printf("%t\n", slice2 == nil)
}


// []string(nil), []string{}
// true
// true
// false
```

* [make](https://golang.org/pkg/builtin/#make)

> * Slice: The size specifies the length. The capacity of the slice is
equal to its length. A second integer argument may be provided to
specify a different capacity; it must be no smaller than the
length. For example, make([]int, 0, 10) allocates an underlying array
of size 10 and returns a slice of length 0 and capacity 10 that is
backed by this underlying array.

>* Map: An empty map is allocated with enough space to hold the
specified number of elements. The size may be omitted, in which case
a small starting size is allocated.

>* Channel: The channel's buffer is initialized with the specified
buffer capacity. If zero, or the size is omitted, the channel is
unbuffered.

```go
// 範圍 low ~ hight -1
a[low : high]

// 也可以在加上 cap
a[low : high : cap]

[]int{1, 2, 3} //creates and array and returns a slice reference
[]int{3: 123, 4: 321, 5: 123456} // 指定 index，其餘的會是 0

numa := [3]int{78, 79 ,80}
nums1 := numa[:] //creates a slice which contains all elements of the array
// tart and end are 0 and len(numa)
```

```go
package main

import (
	"fmt"
)

func main() {
	// 不需定義長度，因此可以一直擴充，但相對的會比較浪費資源
	primes := []int{2, 3, 5, 7, 11, 13}

	fmt.Println(primes[1:4]) // 通過指定索引 1:4 去做 slice，包含第一個，但不包含最後一個
	fmt.Println(primes[1:])  // 索引 1 到最後
	fmt.Println(primes[:2])  // 第一個到索引 2
	fmt.Println(primes[1])   // 索引 1
	fmt.Println(primes[1:1])
	fmt.Println(primes[:])   // 全部
}

/**
[3 5 7]
[3 5 7 11 13]
[2 3]
3
[]
[2 3 5 7 11 13]
**/
```

### Slices are like references to arrays

```go
// slices 都是用 references 的方式，因此改值也會影響到原本的值
package main

import (
	"fmt"
)

func main() {
	numbers := [4]int{1, 2, 3, 4}
	fmt.Println(numbers)

	a := numbers[0:2]
	b := numbers[1:3]
	fmt.Println("a =", a, "b =", b)
	b[0] = 0
	fmt.Println("a =", a, "b =", b)
	fmt.Println(numbers)
}

/**
[1 2 3 4]
a = [1 2] b = [2 3]
a = [1 0] b = [0 3]
[1 0 3 4]
**/
```

### length and capacity

* len is the number of elements in the slice.
* cap is the number of elements in the underlying array starting from the index from which the slice is created.

```go
package main

import (
    "fmt"
)

func main() {
    s := []int{2, 3, 5, 7, 11, 13}
    printSlice(s)
    s = s[:0] // Slice the slice to give it zero length.
    printSlice(s)
    s = s[:4] // Extend its length.
    printSlice(s)
    s = s[:cap(s)] // re-sliced
    printSlice(s)
    s = s[2:] // Drop its first two values.
    printSlice(s)

}

func printSlice(s []int) {
    fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
}


/**
len=6 cap=6 [2 3 5 7 11 13]
len=0 cap=6 []
len=4 cap=6 [2 3 5 7]
len=6 cap=6 [2 3 5 7 11 13]
len=4 cap=4 [5 7 11 13]
**/
```

### re-sliced upto its capacity

```go
package main

import (
    "fmt"
)

func main() {
    fruitarray := [...]string{"apple", "orange", "grape", "mango", "water melon", "pine apple", "chikoo"}
    fruitslice := fruitarray[1:3]
    fmt.Printf("length of slice %d capacity %d\n", len(fruitslice), cap(fruitslice))
    fruitslice = fruitslice[:cap(fruitslice)]
	fmt.Println("After re-slicing length is",len(fruitslice), "and capacity is",cap(fruitslice))
}

// length of slice 2 capacity 6
// After re-slicing length is 6 and capacity is 6
```

### Slices of slices 二維陣列

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	// Create a tic-tac-toe board.
	board := [][]string{
		[]string{"_", "_", "_"},
		[]string{"_", "_", "_"},
		[]string{"_", "_", "_"},
	}

	// The players take turns.
	board[0][0] = "X"
	board[2][2] = "O"
	board[1][2] = "X"
	board[1][0] = "O"
	board[0][2] = "X"

	for i := 0; i < len(board); i++ {
		fmt.Printf("%s\n", strings.Join(board[i], " "))
	}
}

// X _ X
// O _ X
// _ _ O
```

* [strings.Join](https://golang.org/pkg/strings/#Join)

### Append

當空間不夠時，會自動擴充 capacity(*2)，實際上是建立一個新的 array，將原有的值 copy 過去，新的 slice 在 reference 到新的 array 上面

```go
package main

import (
	"fmt"
)

func main() {
	var s []int //zero value of a slice is nil
	printSlice(s)

	// append works on nil slices.
	s = append(s, 0)
	printSlice(s)

	// The slice grows as needed.
	s = append(s, 1)
	printSlice(s)

	// We can add more than one element at a time.
	s = append(s, 2)
	printSlice(s)
}

func printSlice(s []int) {
	fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
}

// len=0 cap=0 []
// len=1 cap=2 [0]
// len=2 cap=2 [0 1]
// len=3 cap=4 [0 1 2]
```

### use make function to create dynamically-sized arrays.

```go
func make([]T, len, cap)

// make(類型, 長度, 容量)
```

```go
package main

import (
	"fmt"
)

func main() {
	a := make([]int, 5) // a := [5]int{}
	printSlice("a", a)
	b := make([]int, 0, 5)
	printSlice("b", b)
	c := b[:2]
	printSlice("c", c)
	d := c[2:5]
	printSlice("d", d)
}

func printSlice(s string, x []int) {
	fmt.Printf("%s len=%d cap=%d %v\n",
		s, len(x), cap(x), x)
}

/**
a len=5 cap=5 [0 0 0 0 0]
b len=0 cap=5 []
c len=2 cap=5 [0 0]
d len=3 cap=3 [0 0 0]
**/
```

* [make](https://golang.org/pkg/builtin/#make)
* [Go Slices: usage and internals](https://blog.golang.org/go-slices-usage-and-internals)

### Passing a slice to a function

因為 slice 是用 reference，因此丟到 function 還是一樣會改值

```go
package main

import (
    "fmt"
)

func subtactOne(numbers []int) {
    for i := range numbers {
        numbers[i] -= 2
    }

}
func main() {
    nos := []int{8, 7, 6}
    fmt.Println("slice before function call", nos)
    subtactOne(nos)                               //function modifies the slice
    fmt.Println("slice after function call", nos) //modifications are visible outside

}

/**
slice before function call [8 7 6]
slice after function call [6 5 4]
**/
```

### Memory Optimisation

用 slices 時，會 reference 到 array，此時會造成 array 無法被 garbage collected，因此會造成 memory 的浪費

因此要解決的話可以利用 [copy](https://golang.org/pkg/builtin/#copy)，copy 會產生新的 slices 跟 array ，而原本的 array 就可以被 garbage collected

```go
package main

import (
    "fmt"
)

func countries() []string {
    countries := []string{"USA", "Singapore", "Germany", "India", "Australia"}
    neededCountries := countries[:len(countries)-2]
    countriesCpy := make([]string, len(neededCountries))
    copy(countriesCpy, neededCountries) //copies neededCountries to countriesCpy
    return countriesCpy
}
func main() {
    countriesNeeded := countries()
    fmt.Println(countriesNeeded)
}

// [USA Singapore Germany]
```

* [copy](https://golang.org/pkg/builtin/#copy)

也可以用 `[]int(nil) + append` 的方式來 copy, 因為 nil 的 slice 沒有 reference 任何 array，寫法也比較簡潔

> nil is typeless value, so use []int() to convert nil value to slice value

```go
package main

import (
	"fmt"
	
)

func main() {
	evens := []int{2, 4}
	fmt.Printf("evens: = %v, %[1]p, %d, %d\n", evens, len(evens), cap(evens))
	
	test := make([]int, len(evens))
	copy(test, evens)
	fmt.Printf("test: = %v, %[1]p, %d, %d\n", test, len(test), cap(test))
	
	test2 := append([]int(nil), evens...)
	fmt.Printf("test: = %v, %[1]p, %d, %d\n", test2, len(test2), cap(test2))
}

// evens: = [2 4], 0x414020, 2, 2
// test: = [2 4], 0x414048, 2, 2
// test: = [2 4], 0x414060, 2, 2
```

# <span id="maps"> Maps </span>

A map maps keys to values. (相當於其當語言的 hash)

> * Similar to slices, maps are reference types
> * Only be compared to nil
> * The zero value of a map is nil
> * nil 的 map 無法賦值，必須用 make

```go
make(map[key]value)
value, ok := map[key] // 判斷 key 的 value 在不在
delete(map, key)
```

> * Map: An empty map is allocated with enough space to hold the
specified number of elements. The size may be omitted, in which case
a small starting size is allocated.

```go
package main

import (
	"fmt"
)

func main() {
	// 聲明 map 的 key 和 value 的 type
	var m1 map[string]int
	fmt.Println("m1 == nil:", m1 == nil) // true
	// 使用make函式建立一個非nil的map，nil map不能賦值
	m1 = make(map[string]int)
	fmt.Println("m1 == nil:", m1 == nil) // false
	// 最後給已聲明的map賦值
	m1["a"] = 1
	fmt.Println("m1 =", m1)
	fmt.Println("m1[a] =", m1["a"])

	// 直接建立
	// map[keyType]valueTypes
	m2 := make(map[string]string)
	// 然後賦值
	m2["a"] = "aa"
	fmt.Println("m2 =", m2)

	// 初始化 + 賦值一體化
	m3 := map[string]string{
		"a": "aa",
		"b": "bb",
		"c": "cc",
	}
	fmt.Println("m3 =", m3)

	// 查找鍵值是否存在，ok 為 true / false, v 為 value
	if v, ok := m1["c"]; ok {
		fmt.Println(v)
	} else {
		fmt.Println("Key Not Found")
	}

	// for loop map
	for k, v := range m3 {
		fmt.Println(k, v)
	}

	// delete
	delete(m3, "a")
	fmt.Println(m3)

	//Length of the map
	fmt.Println(len(m3))
}

/**
m1 == nil: true
m1 == nil: false
m1 = map[a:1]
m1[a] = 1
m2 = map[a:aa]
m3 = map[a:aa b:bb c:cc]
Key Not Found
a aa
b bb
c cc
map[b:bb c:cc]
2
**/
```

```go
package main

import "fmt"

type Vertex struct {
  x, y float64
}

// 先聲明map
var m map[string]Vertex

func main() {
  // 再使用make函式建立一個非nil的map，nil map不能賦值
  m = make(map[string]Vertex)
  m["A"] = Vertex{
    40.68433, -74.39967,
  }
  fmt.Println(m["A"])
  fmt.Println(m["B"])
  fmt.Println(m)
}

// {40.68433 -74.39967}
// {0, 0}
// map[A:{40.68433 -74.39967}]
```

* [map (1)](https://hsinyu.gitbooks.io/golang_note/content/map_1.html)
* [Golang map 如何進行刪除操作？](http://blog.cyeam.com/json/2017/11/02/go-map-delete#map-%E7%9A%84%E5%88%A0%E9%99%A4%E6%93%8D%E4%BD%9C)


# <span id="variadic"> Variadic Functions </span>

`Variadic Functions` 是一個可接受任意的參數，一定要放最後面一個

```go
...Type // 代表可以接收任意參數
func append(slice []Type, elems ...Type) []Type
// elems 可以接收任意參數
```

### example 1

```go
package main

import (
    "fmt"
)

func find(num int, nums ...int) {
    fmt.Printf("type of nums is %T\n", nums)
    found := false
    for i, v := range nums {
        if v == num {
            fmt.Println(num, "found at index", i, "in", nums)
            found = true
        }
    }
    if !found {
        fmt.Println(num, "not found in ", nums)
    }
    fmt.Printf("\n")
}
func main() {
    find(89, 89, 90, 95)
    find(45, 56, 67, 45, 90, 109)
    find(78, 38, 56, 98)
    find(87)
    // 相當於
    nums := []int{89, 90, 95}
    find(89, nums...) // 要加 ... 否則 type 會是 []int，造成 error
}

/**
type of nums is []int
89 found at index 0 in [89 90 95]

type of nums is []int
45 found at index 2 in [56 67 45 90 109]

type of nums is []int
78 not found in  [38 56 98]

type of nums is []int
87 not found in  []s
**/
```

原理是會將，後面的參數轉成 slice，所以 type 會變成 `[]int`，但是因為 `find` 後面接收的參數 type 是 int，也不能直接將 `[]int` 帶進去，因此 `[]int` 透過 `...` 語法糖，帶到 Variadic Functions 裡

### example 2

```go
package main

import (
    "fmt"
)

func change(s ...string) {
    s[0] = "Go"
    s = append(s, "playground")
    fmt.Println(s) // [Go world playground]
}

func main() {
    welcome := []string{"hello", "world"}
    change(welcome...)
    fmt.Println(welcome) // [Go world]
}
```

* 一開始是用 slice(reference) 傳入，因此改變值，外面也會跟著改變
* append 則是空間不夠時會建立一個新的 array， 因此不會影響到原來的值

參考文件:

* [golangbot.com](https://golangbot.com/learn-golang-series/)
