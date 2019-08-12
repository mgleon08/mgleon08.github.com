---
layout: post
title: "Golang Tricks"
date: 2019-06-18 22:11:30 +0800
comments: true
categories: golang
---

<!-- more -->

1. [Defer 執行順序](#defer1)
2. [Range & Foreach](#range)
3. [golang 執行的隨機性和閉包](#closure)
4. [golang 的組合繼承](#combination_inherit)
5. [Select 隨機性](#select)
6. [Defer 執行順序](#defer2)
7. [make 預設值和 append](#make_append)
8. [map 執行緒安全](#map_concurrent)
9. [chan緩存池](#chan_buffer)
10. [interface 實作方式](#interface_implement)
11. [interface內部結構](#interface_internal)
12. [Type Assertion](#type_assertion)
13. [Return Value Naming](#return_value_naming)
14. [defer和函式返回值](#defer3)
15. [new & make](#new_make)
16. [slice append](#slice_append)
17. [Struct Compare](#struct_compare)
18. [Return Type](#return_type)
19. [iota](#iota)
20. [Short variable declarations](#short_variable_declarations)
21. [Const Address](#const_address)
22. [goto 位置](#goto)
23. [Type Alias](#type_alias)
24. [if 變數作用域](#variable_scope)

# <span id='defer1'> Defer 執行順序 </span>

```go
package main

import (
    "fmt"
)

func main() {
    defer_call()
}

func defer_call() {
    defer func() { fmt.Println("1") }()
    defer func() { fmt.Println("2") }()
    defer func() { fmt.Println("3") }()

    panic("exception")
}
```

https://play.golang.org/p/Zpbiau9RuGm

* defer 採用後進先出(Last In First Out (LIFO))
* panic 需等所有的 defer 結束後才會執行

# <span id='range'> Range & Foreach </span>

```go
package main

import (
	"fmt"
)

type student struct {
	Name string
	Age  int
}

func main() {
	m := make(map[string]*student)
	stus := []student{
		{Name: "zhou", Age: 24},
		{Name: "li", Age: 23},
		{Name: "wang", Age: 22},
	}
	for _, stu := range stus {
		m[stu.Name] = &stu
	}

	for k, v := range m {
		fmt.Println(k, "=>", v.Name)
	}

}
```

https://play.golang.org/p/ocNj5lp0W0_4

* range 是用 copy 的方式，因此每個 key 都指向同一個 point，最後的值就會是最後一個指向的 value
* 解決方式，改用 index 的方式

  ```go
  for i := 0; i < len(stus); i++ {
    m[stus[i].Name] = &stus[i]
  }
  
  for k, v := range m {
    fmt.Println(k, "=>", v.Name)
  }

  for i, _ := range stus {
    m[stus[i].Name] = &stus[i]
  }

  for k, v := range m {
    fmt.Println(k, "=>", v.Name)
  }
  ```

# <span id='closure'> golang 執行的隨機性和閉包 </span>
  
```go
package main

import (
	"fmt"
	"runtime"
	"sync"
)

func main() {
	runtime.GOMAXPROCS(1)
	wg := sync.WaitGroup{}
	wg.Add(20)
	for i := 0; i < 10; i++ {
		go func() {
			fmt.Printf("A address: %p, value: %v\n", &i, i)
			wg.Done()
		}()
	}
	for i := 0; i < 10; i++ {
		go func(i int) {
			fmt.Printf("B address: %p, value: %v\n", &i, i)
			wg.Done()
		}(i)
	}
	wg.Wait()
}
```

https://play.golang.org/p/EzCWf8r1wnt

* 第一個 func 沒有將 i 帶入，取得的 i 是 for 外部的 i，並且 i 會一直加 1 直到 10，因此最後輸出的 A 全部都是 10
* 第二個 func 則是每次都會帶入 i，每個 i 都會做 copy 帶入 func，因此外部的 ++ 不會影響到

# <span id='combination_inherit'> golang 的組合繼承 </span>

```go
package main

import (
	"fmt"
)

type People struct{}

func (p *People) ShowA() {
	fmt.Println("showA")
	p.ShowB()
}
func (p *People) ShowB() {
	fmt.Println("showB")
}

type Teacher struct {
	People
}

func (t *Teacher) ShowB() {
	fmt.Println("teacher showB")
}

func main() {
	t := Teacher{}
	t.ShowA()
}
```

https://play.golang.org/p/eSGvASIBn5K

* 透過組合的方式，來實現 OO 的繼承，當 struct 有 `anonymous struct field`，就會被當成 `promoted fields`，簡單的來說原本 people 的方法(必須是匿名)就可以直接被 teacher 給調用
* 當呼叫了 showA() 裡面在 call showB() 時，此時 People 類型並不知道自己會被什麼類型組合，因此無法去使用未知的組合者 Teacher 類型的 func

# <span id='select'> Select隨機性 </span>

```go
package main

import (
	"fmt"
	"runtime"
)

func main() {
	runtime.GOMAXPROCS(1)
	int_chan := make(chan int, 1)
	string_chan := make(chan string, 1)
	int_chan <- 1
	string_chan <- "hello"
	select {
	case value := <-int_chan:
		fmt.Println(value)
	case value := <-string_chan:
		panic(value)
	}
}
```

https://play.golang.org/p/ntESh6QYaYT

* select 擁有隨機性，因此都有可能觸發
* 當只要有個 case 可以 return 就可以立即執行
* 都能 return 則隨機
* 都不行則跑 default

# <span id='defer2'> Defer 執行順序 </span>

```go
package main

import "fmt"

func calc(index string, a, b int) int {
	ret := a + b
	fmt.Println(index, a, b, ret)
	return ret
}

func main() {
	a := 1
	b := 2
	defer calc("A", a, calc("C", a, b))
	a = 3
	defer calc("B", a, calc("D", a, b))
	b = 4
}
```

https://play.golang.org/p/BrhIs-_chN_y

* defer 採用後進先出(Last In First Out (LIFO))

# <span id='make_append'> make 預設值和 append </span>

```go
package main

import "fmt"

func main() {
	s := make([]int, 5)
	s = append(s, 1, 2, 3)
	fmt.Println(s)
}
```

https://play.golang.org/p/7VHcPYjenWw

* make 初始化時，後面直有一個值的話，代表 len 和 cap，因此裡面預設就有 5 個 default 0，所以在 append 時，會再繼續加上去

# <span id='map_concurrent'> map 執行緒安全 </span>

```go
package main

import (
	"fmt"
	"sync"
)

type UserAges struct {
	ages map[string]int
	sync.Mutex
}

func (ua *UserAges) Add(name string, age int) {
	ua.Lock()
	defer ua.Unlock()
	ua.ages[name] = age
}

func (ua *UserAges) Get(name string) int {
	if age, ok := ua.ages[name]; ok {
		return age
	}
	return -1
}

func main() {
	userAge := &UserAges{
		ages: make(map[string]int),
	}
	userAge.Add("leon", 28)
	age := userAge.Get("leon")
	fmt.Println(age)
}
```

https://play.golang.org/p/KwtbBryhP5x

* 在讀(Get)的時候，沒有加上 Lock，可能會導致 `fatal error: concurrent map read and map write`
* 更改為
  ```go
  func (ua *UserAges) Get(name string) int {
    ua.Lock()
    defer ua.Unlock()
    if age, ok := ua.ages[name]; ok {
        return age
    }
    return -1
  }
  ```
  
  
# <span id='chan_buffer'> chan緩存池 </span>

```go
package main

import (
	"fmt"
	"sync"
)

type threadSafeSet struct {
	sync.RWMutex
	s []interface{}
}

func (set *threadSafeSet) Iter() <-chan interface{} {
	ch := make(chan interface{})
	go func() {
		set.RLock()

		for _, elem := range set.s {
			ch <- elem
			fmt.Println("Iter:", elem)
		}

		close(ch)
		set.RUnlock()

	}()
	return ch
}

func main() {

	th := threadSafeSet{
		s: []interface{}{"1", "2"},
	}
	v := <-th.Iter()
	fmt.Printf("%s%v", "ch", v)
}
```

https://play.golang.org/p/2n2GBeo35Jo

* chan 不是 buffer chan 因此，會造成阻塞，並且顯示不了所有的 value，當 range 將第一個 "1" 放進 chan 後，再跑第二次要放進 "2" 時，必須等待第一個值被取出來，因此會停在那邊，但是當取出來後，就會繼續進行下面，導致直接結束
* 改用 buffer chan
  ```go
  ch := make(chan interface{},len(set.s))
  ```
  
# <span id="interface_implement"> interface 實作方式 </span>

```go
package main

import (
	"fmt"
)

type People interface {
	Speak(string) string
}

type Stduent struct{}

func (stu *Stduent) Speak(think string) (talk string) {
	if think == "say" {
		talk = "You are a good boy"
	} else {
		talk = "hi"
	}
	return
}

func main() {
	var peo People = Stduent{}
	think := "say"
	fmt.Println(peo.Speak(think))
}
```

https://play.golang.org/p/u1qbY4CbVNQ

* 因為實作 interface 的是 pointer receiver，所以會造成 error
  ```go
  var peo People = &Stduent{}
  ```
  
  
# <span id="interface_internal"> interface內部結構 <span>

```go
package main

import (
	"fmt"
)

type People interface {
	Show()
}

type Student struct{}

func (stu *Student) Show() {

}

func live() People {
	var stu *Student
	return stu
}

func main() {
	if live() == nil {
		fmt.Println("AAAAAAA")
	} else {
		fmt.Println("BBBBBBB")
	}
}
```

https://play.golang.org/p/2nBU-dMwW6b


```go
package main

import "fmt"

func Foo(x interface{}) {
	if x == nil {
		fmt.Println("empty interface")
		return
	}
	fmt.Println("non-empty interface")
}

func main() {
	var x *int = nil
	Foo(x)
}
```

https://play.golang.org/p/lx6W61O66P

* golang 中的 interface 分兩種，但底層結構稍微不同，因此 data 指向了 `nil` 並不代表 interface 是 `nil`
  * ```go
    var in interface{}
    ```
  * ```go
    type People interface {
        Show()
    }
    ```
* 第二個改成 `var x interface{}` 則為 `empty interface`
* [Go語言interface底層實現](https://i6448038.github.io/2018/10/01/Golang-interface/)
* [Go介面詳解](https://zhuanlan.zhihu.com/p/27055513)

# <span id="type_assertion"> Type Assertion <span>

```go
package main

import "fmt"

func main() {
	i := GetValue()

	switch i.(type) {
	case int:
		fmt.Println("int")
	case string:
		fmt.Println("string")
	case interface{}:
		fmt.Println("interface")
	default:
		fmt.Println("unknown")
	}

}

func GetValue() int {
	return 1
}
```

https://play.golang.org/p/-OiFMCcYttw

* Type Assertion 只能用於 interface

# <span id="return_value_naming"> Return Value Naming <span>

```go
package main

import "fmt"

func main() {
	i, _ := funcMui(1, 2)
	fmt.Println(i)

}

func funcMui(x,y int)(sum int, error){
    return x+y,nil
}
```

https://play.golang.org/p/nH6KTCel6ns

* 只要有一個返回值有命名，全部就必須命名
* 如果兩個 type 一樣，則可以只寫一個


# <span id="defer3"> defer和函式返回值 <span>

```go
package main

func main() {
	println(DeferFunc1(1))
	println(DeferFunc2(1))
	println(DeferFunc3(1))
}

func DeferFunc1(i int) (t int) {
	t = i
	defer func() {
		t += 3
	}()
	return t
}

func DeferFunc2(i int) int {
	t := i
	defer func() {
		t += 3
	}()
	return t
}

func DeferFunc3(i int) (t int) {
	defer func() {
		t += i
	}()
	return 2
}
```

https://play.golang.org/p/HpOb3huRV-0

* defer 需要在 func 結束前執行
* func return 名字會在函式起始處被初始化為對應類型的零值並且作用域為整個函式 
* DeferFunc1 - t 作用域為整個 func，因此 return 前會將 t 更改
* DeferFunc2 - t 作用域為裡面的 func，因此不影響外面的 t
* DeferFunc3 - t 作用域為整個 func，跟 Func 1 一樣

# <span id="new_make"> new & make <span>

```go
package main

import "fmt"

func main() {
	list := new([]int)
	list = append(list, 1)
	fmt.Println(list)
}
```

https://play.golang.org/p/CEwGhsmQfZx

* new 可以用來初始化泛型，並且返回儲存位址
* new 會自動用 zeroed value 來初始化型別
  * `string` => ""
  * `int`, `float` => 0
  * `channel`, `func`, `map`, `slice` => `nil`
* 必須改用 make([]int, 0), 初始化 slice (make 不會回傳指標)
* [Go 語言中的 make 和 new](https://draveness.me/golang-make-and-new)
* [golang 筆記：make 與 new 的差別](https://medium.com/d-d-mag/golang-%E7%AD%86%E8%A8%98-make-%E8%88%87-new-%E7%9A%84%E5%B7%AE%E5%88%A5-68b05c7ce016)
* [Allocation with new](https://golang.org/doc/effective_go.html#allocation_new)
* [Allocation with make](https://golang.org/doc/effective_go.html#allocation_make)


# <span id="slice_append"> slice append <span>

```go
package main

import "fmt"

func main() {
	s1 := []int{1, 2, 3}
	s2 := []int{4, 5}
	s1 = append(s1, s2)
	fmt.Println(s1)
}
```

https://play.golang.org/p/xOWsWj0Evni

* append slice 必須加上 `...` 展開 `s2...`

# <span id="struct_compare"> Struct Compare <span>

```go
package main

import "fmt"

func main() {
	sn1 := struct {
		age  int
		name string
	}{age: 11, name: "qq"}

	sn2 := struct {
		age  int
		name string
	}{age: 11, name: "qq"}

	if sn1 == sn2 {
		fmt.Println("sn1 == sn2")
	}

	sm1 := struct {
		age int
		m   map[string]string
	}{age: 11, m: map[string]string{"a": "1"}}

	sm2 := struct {
		age int
		m   map[string]string
	}{age: 11, m: map[string]string{"a": "1"}}

	if sm1 == sm2 {
		fmt.Println("sm1 == sm2")
	}
}
```

https://play.golang.org/p/3cEX9aYtvsR

* struct 只能在相同類型才能做比較，順序也會有差
  * 這樣也不能做比較
  * ```go
    sn3 := struct {
		age  int
		name string
	  }{age: 11, name: "qq"}
    ```
* map 和 slice 則不可比較，如果要比較要用 `reflect.DeepEqual`
  * ```go
    if reflect.DeepEqual(sm1, sm2) {
		  fmt.Println("sm1 == sm2")
	  }
    ```
    
# <span id="return_type"> Return Type <span>

```go
package main

import "fmt"

func GetValue(m map[int]string, id int) (string, bool) {
	if _, exist := m[id]; exist {
		return "存在數據", true
	}
	return nil, false
}

func main() {
	intmap := map[int]string{
		1: "a",
		2: "bb",
		3: "ccc",
	}

	v, err := GetValue(intmap, 3)
	fmt.Println(v, err)
}
```

https://play.golang.org/p/noHSLgTvdEq

* `nil` 可以用作 `interface`、`function`、`pointer`、`map`、`slice` 和 `channel` 的"空值"。

# <span id="iota"> iota </span>

```go
package main

import "fmt"

const (
	x = iota
	y
	z = "zz"
	k
	p = iota
)

func main() {
	fmt.Println(x, y, z, k, p)
}
```

https://play.golang.org/p/XwtR_I0fQdp

# <span id="short_variable_declarations"> Short variable declarations </span>

```go
package main

import "fmt"

var(
    size := 1024
    max_size = size*2
)

func main()  {
    fmt.Println(size,max_size)
}
```

https://play.golang.org/p/AUfpBHZg-F4

* `:=` 是聲明並賦值，並且系統自動推斷類型，必須放在 main function 裡面


# <span id="const_address"> Const Address </span>

```go
package main

import "fmt"

const cl  = 100
var bl    = 123

func main()  {
    fmt.Println(&bl,bl)
    fmt.Println(&cl,cl)
}
```

https://play.golang.org/p/dbK6-maWceJ

* 常量不同於變數的在運行期分配記憶體，常量通常會被編譯器在預處理階段直接展開，作為指令數據使用
* [pointers - Find address of constant in go](https://stackoverflow.com/questions/35146286/find-address-of-constant-in-go)


# <span id="goto"> goto 位置 </span>

```go
package main

import "fmt"

func main() {

	for i := 0; i < 10; i++ {
	loop:
		fmt.Println(i)
	}
	goto loop
}
```

https://play.golang.org/p/-0KuesWC3kM

* goto 必須放在 func 裡面

# <span id="type_alias"> Type Alias </span>

```go
package main

import "fmt"

func main() {
	type MyInt1 int
	type MyInt2 = int
	var i int = 9
	var i1 MyInt1 = i
	var i2 MyInt2 = i
	fmt.Println(i1, i2)
}
```

https://play.golang.org/p/ELp_QFf5-o2

* 基於一個類別型建立一個新類型，稱之為 defintion
* 基於一個類別型建立一個別名，稱之為 alias
* MyInt1 為稱之為 defintion，雖然底層類型為 int 類型，但是不能直接賦值，需要強轉
* MyInt2 稱之為 alias，可以直接賦值

```go
package main

import "fmt"

type User struct {
}

type MyUser1 User
type MyUser2 = User

func (i MyUser1) m1() {
	fmt.Println("MyUser1.m1")
}
func (i User) m2() {
	fmt.Println("User.m2")
}

func main() {
	var i1 MyUser1
	var i2 MyUser2
	i1.m1()
	i2.m2()
}
```

https://play.golang.org/p/2fsGAqTmUrH

* `i2.m1()` 會 error，因為 m1() 是 `MyUser1` 才有的 func，不是 User

```go
package main

import "fmt"

type T1 struct {
}

func (t T1) m1() {
	fmt.Println("T1.m1")
}

type T2 = T1

type MyStruct struct {
	T1
	T2
}

func main() {
	my := MyStruct{}
	my.m1()
}
```

https://play.golang.org/p/zwvjqUFLHax

* 重複會不知道該 call 哪一個導致 error (`ambiguous selector my.m1`)，把其中一個拿掉就可以 work，或是改為 `my.T1.m1()` + `my.T2.m1()` 


# <span id="variable_scope"> if 變數作用域 </span>

```go
package main

import (
	"fmt"
)

func main() {
	a, b := 1, 2
	fmt.Printf("%p\n", &a)
	fmt.Printf("%p\n", &b)
	
	if true {
		b, c := 3, 4
		fmt.Printf("%p\n", &b)
		fmt.Printf("%p\n", &c)
	}
}
```

https://play.golang.org/p/CvqxnzGSAG9

* if 的變數會遮罩函式作用域內的變數，導致裡面的 b 是新的 address


# <span id="closure"> closure </span>


```go
package main

import (
	"fmt"
)

func test() []func() {
	var funs []func()
	for i := 0; i < 2; i++ {
		funs = append(funs, func() {
			fmt.Println(&i, i)
		})
	}
	return funs
}

func main() {
	funs := test()
	for _, f := range funs {
		f()
	}
}
```

https://play.golang.org/p/8RL7gO5_4EF

* `funx` 裡的 i 都是同一個，要不一樣要先 assign 到新的 value
  ```go
  for i:=0;i<2 ;i++  {
        x:=i
        funs = append(funs, func() {
            println(&x,x)
        })
  }
  ```
  
  
```go
package main

import (
	"fmt"
)

func test(x int) (func(), func()) {
	return func() {
			fmt.Println(x)
			x += 10
		}, func() {
			fmt.Println(x)
		}
}

func main() {
	a, b := test(100)
	a()
	b()
	
}
```

https://play.golang.org/p/5ReC78kUM3c

* 閉包引用相同變數

# Reference

* [Golang面試題解析](https://my.oschina.net/qiangmzsx/blog/1478739)
* [Go面試題答案與解析](https://bit.ly/2WSSgGe)
* [Golang面試題解析（二）](https://my.oschina.net/qiangmzsx/blog/1515173)
* [Golang面試題解析（三) ](https://my.oschina.net/qiangmzsx/blog/1533839)
