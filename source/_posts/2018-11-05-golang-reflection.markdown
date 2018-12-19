---
layout: post
title: "Golang - Reflection"
date: 2018-11-05 15:16:41 +0800
comments: true
categories: golang
---

<!-- more -->

# <span id="what"> What is reflection? </span>

[Reflection](https://golang.org/pkg/reflect/) 是 golang 中的一個 package，可以在執行時檢查變數和值，並找到其所屬的 type。

### reflect.Type and reflect.Value

* `reflect.Type` 可以表示 `interface{}` 具體的 type，透過 [reflect.TypeOf()](https://golang.org/pkg/reflect/#TypeOf) 取得
* `reflect.Value` 可以表示 `interface{}` 底層的 value，透過 [reflect.ValueOf()](https://golang.org/pkg/reflect/#ValueOf) 取得
* `reflect.Kind` 可以表示具體 type 的類型 [reflect.Kind](https://golang.org/pkg/reflect/#Kind)
* 以下範例可能會比較清楚

Generalize query creator and make it work on any struct

```go
package main

import (
	"fmt"
	"reflect"
)

type order struct {
	ordId      int
	customerId int
}

// interface{} 意味著任何型態的值，因為裡面沒有任何 method，也代表所有 type 都 implement
func createQuery(q interface{}) {
	t := reflect.TypeOf(q)
	v := reflect.ValueOf(q)
	k := t.Kind()
	fmt.Println("Type ", t)
	fmt.Println("Value ", v)
	fmt.Println("Kind ", k)
}

func main() {
	o := order{
		ordId:      456,
		customerId: 56,
	}
	createQuery(o)

}

//Type  main.order
//Value  {456 56}
//Kind  struct
```

### NumField() and Field() methods

* `NumField()` 用來算出 reflect.Value 裡的數量
* `Field()` 用來取出 reflect.Value 的值

```go
package main

import (
    "fmt"
    "reflect"
)

type order struct {
    ordId      int
    customerId int
}

func createQuery(q interface{}) {
    if reflect.ValueOf(q).Kind() == reflect.Struct {
        v := reflect.ValueOf(q)
        fmt.Println("Number of fields", v.NumField())
        for i := 0; i < v.NumField(); i++ {
            fmt.Printf("Field:%d type:%T value:%v\n", i, v.Field(i), v.Field(i))
        }
    }

}
func main() {
    o := order{
        ordId:      456,
        customerId: 56,
    }
    createQuery(o)
}
```

### Int() and String() methods

* The methods Int and String help extract the reflect.Value as an int64 and string respectively.

```go
package main

import (
    "fmt"
    "reflect"
)

func main() {
    a := 56
    x := reflect.ValueOf(a).Int()
    fmt.Printf("type:%T value:%v\n", x, x)
    b := "Naveen"
    y := reflect.ValueOf(b).String()
    fmt.Printf("type:%T value:%v\n", y, y)

}

// type:int64 value:56
// type:string value:Naveen
```

### Complete Generalize query creator

```go
package main

import (
    "fmt"
    "reflect"
)

type order struct {
    ordId      int
    customerId int
}

type employee struct {
    name    string
    id      int
    address string
    salary  int
    country string
}

func createQuery(q interface{}) {
    // 先判斷 q 是否為 struct
    if reflect.ValueOf(q).Kind() == reflect.Struct {
      // 取得具體 type，並提取該名稱
      // main.order -> order
      // main.employee -> employee
        t := reflect.TypeOf(q).Name()
        query := fmt.Sprintf("insert into %s values(", t)
      // 取得 value
        v := reflect.ValueOf(q)
      // v.NumField() 取得數量去做迴圈
        for i := 0; i < v.NumField(); i++ {
            switch v.Field(i).Kind() {
            case reflect.Int:
                if i == 0 {
                  // 第一個不需接 ,
                    query = fmt.Sprintf("%s%d", query, v.Field(i).Int())
                } else {
                    query = fmt.Sprintf("%s, %d", query, v.Field(i).Int())
                }
            case reflect.String:
                if i == 0 {
                  // 第一個不需接 ,
                    query = fmt.Sprintf("%s\"%s\"", query, v.Field(i).String())
                } else {
                    query = fmt.Sprintf("%s, \"%s\"", query, v.Field(i).String())
                }
            default:
                // 這個只處理 int & string 不符合就 return
                fmt.Println("Unsupported type")
                return
            }
        }
      // 最後結尾在加上 )
        query = fmt.Sprintf("%s)", query)
        fmt.Println(query)
        return

    }
  // 如果參數不是 struct 就會顯示
    fmt.Println("unsupported type")
}

func main() {
    o := order{
        ordId:      456,
        customerId: 56,
    }
    createQuery(o)

    e := employee{
        name:    "Naveen",
        id:      565,
        address: "Coimbatore",
        salary:  90000,
        country: "India",
    }
    createQuery(e)
    i := 90
    createQuery(i)

}
```

# Should reflection be used?

> Rob Pike: Clear is better than clever. Reflection is never clear.

結論是 reflection 是非常強大且高級的技巧，但是難以寫的乾淨或是很好維護，所以作者是建議盡量可以避免，除非真的有需要的話

參考文件

* [golangbot.com Part 34: Reflection](https://golangbot.com/reflection/)
