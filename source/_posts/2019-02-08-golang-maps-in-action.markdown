---
layout: post
title: "Golang - maps in action"
date: 2019-02-08 10:55:15 +0800
comments: true
categories: golang
---

<!-- more -->

# Introduction

 `Hash Table` 是 Computer Science 中最有用的資料結構，提供了快速尋找，新增，刪除，Golang 透過 `map` type 來實踐。
 
# Declaration and initialization

```go
map[KeyType]ValueType
```

* KeyType 是 type 可以是任意一個可比較的類型
* ValueType 也是可以任意的類型
* 包括 map type 也可以

下面的變數 `m` 是一個 `string keys` to `int values` 的 map

```go
var m map[string]int
```

Map 的類型是 `reference types`, 像是 `pointers` or `slices`，因此上面 `m` 的 value 是 nil

當讀取時 `nil map` 行為類似空的 map，若嘗試寫入 `nil map` 則會造成 `runtime panic` 

因此如果要初始化一個 map 可以用 `make` function

```go
m = make(map[string]int)
```

`make` function 會分配並且初始化一個 `hash map data structure` 並返回指向它(`make`)的 map value

# Working with maps

Set key `route` to `66`

```go
m["route"] = 66
```

Assign `m["route"]` to variable `66`

```go
i := m["route"]
```

value 是 `int`，因此如果 key 不存在，則會回傳 `0`，string 則是回傳空字串

> 前提是要用 make 來建立，否則會 panic

```go
j := m["root"]
// j == 0
```

回傳 map 的 item 長度

```go
n := len(m)
```

`delete` function 根據 key 去做刪除，刪除不會回傳任何東西，如果 key 是不存在則不會做任何事

```go
delete(m, "route")
```

map 也可以用兩個變數來取

* 第一個變數 `i` 是指 `m["route"]` 裡的值，如果沒有 `route` 就回傳 `0`
* 第二個變數 `ok` 則是用來判斷這個 key 存不存在，true 為存在，反之不存在

```go
i, ok := m["route"]
```

如果只是要判斷存不存在，並沒有要使用到 value 可以給一個 `_`

```go
_, ok := m["route"]
```

要取出 map 的 key & value 可以用 range

```go
for key, value := range m {
    fmt.Println("Key:", key, "Value:", value)
}
```

初始化並給值

```go
commits := map[string]int{
    "rsc": 3711,
    "r":   2138,
    "gri": 1908,
    "adg": 912,
}
```

初始化並給空的值，效果跟用 `make` 一樣

```go
m = map[string]int{}
```

# Exploiting zero values

在 map 上利用 0(bool) 值

###  map 利用 bool 來作為一種數據結構的檢測，就不需要多一個變數來處理

```go
package main

import (
	"fmt"
)

func main() {
	// 建立一個 Node 的 struct
	type Node struct {
		Next  *Node
		Value interface{}
	}
	
	second := &Node{
		Next:  nil,
		Value: 2,
	}

	first := &Node{
		Next:  second,
		Value: 1,
	}
	// 故意讓 first 重複，形成迴圈
	second.Next = first
	visited := make(map[*Node]bool)
	for n := first; n != nil; n = n.Next {
		// 如果遇到一個已經變成 true 代表重複了，就 break
		if visited[n] {
			fmt.Println("cycle detected")
			break
		}
		// 只要有遍歷到就將 value 改成 true
		visited[n] = true
		fmt.Println(n.Value)
	}

}
```

### map of slices

不需要 check key 存不存在，因為 appending 一個 nil 的 slice, 會自動分配新的 slice
 
```go
package main

import (
	"fmt"
)

func main() {
	type Person struct {
		Name  string
		Likes []string
	}

	var people []*Person
	people = append(people, &Person{"Leon", []string{"cheese", "bacon"}})
	// 也可以 people := []*Person{&Person{"Leon", []string{"cheese", "bacon"}}}

	likes := make(map[string][]*Person)

	// 取出所有 people
	for _, p := range people {
		// 取出每個 person 喜歡的東西
		for _, l := range p.Likes {
			// 列出喜歡這個東西的人
			likes[l] = append(likes[l], p)
		}
	}
	// 列出喜歡起司的人
	for _, p := range likes["cheese"] {
		fmt.Println(p.Name, "likes cheese.")
	}

	// 列出喜歡培根的人數
	fmt.Println(len(likes["bacon"]), "people like bacon.")
}
```
由於 range 和 len 都將 nil slice 視為零長度的 slice，所以沒有 data 也不會有問題

```go
package main

import "fmt"

type ListNode struct {
	Val  int
	Next *ListNode
}

func main() {
	mapp := make(map[string]ListNode)
	mapp["a"] = ListNode{Val: 1}
	mapp["b"] = ListNode{Val: 2}
	mapp["a"].Next = mapp["b"]
	fmt.Println(mapp["a"].Val)
}
```

# Key types

先前提到 key 可以是任何可以比較的類型(`boolean`, `numeric`, `string`, `pointer`, `channel`, and `interface types`, and `structs` or `arrays`)

注意到這列表上不包括(`slices`, `maps`, and `functions`) 這些類型不能做比較，所以也不能當作 map 的 key

另外 struct key 是比較特別的，因為 struct 的 data 是多維度的面相(可以描述一個 data 的結構)

舉例來說下面是一個 map 包著一個 map，用於統計國家/地區的網頁造訪次數

```go
// 外面 map 的 key 是網頁的路徑 path，裡面的 map 的 key 則是 國家的代碼
hits := make(map[string]map[string]int)
```

澳洲(Australian) 的 documentation page 點擊次數

```go
n := hits["/doc/"]["au"]
```

但這種方法，再新增新的 data 時，會不太好處理，因為每次給外部 map key 時，就必須再檢查裡面的 map 是否存在，不存在在建立

```go
func add(m map[string]map[string]int, path, country string) {
	// 先確認 value 在不在
    mm, ok := m[path]
    // 如果 value 不存在，就建立新的 inner map，每次都要檢查
    if !ok {
        mm = make(map[string]int)
        m[path] = mm
    }
    // 
    mm[country]++
}
add(hits, "/doc/", "au")
```

利用 struct 可以減少上面的複雜性

```go
type Key struct {
    Path, Country string
}
hits := make(map[Key]int)
```

當越南人(Vietnamese) 造訪頁面，增加(或建立新的) 可以用一行就解決

```go
hits[Key{"/", "vn"}]++
```

要看到瑞士(Swiss)有多少人看到 `/ref/spec` 也很簡單

```go
n := hits[Key{"/ref/spec", "ch"}]
```


# Concurrency

[Maps are not safe for concurrent use](https://golang.org/doc/faq#atomic_maps)

並發訪問map是不安全的，會出現未定義行為

如果希望多併發讀取 map，必須提供某種同步機制，可以用 [sync.RWMutex](https://golang.org/pkg/sync/#RWMutex) 讀寫鎖，確保同步機制(synchronization mechanism)

但是透過讀寫鎖控制 map 的並發訪問時，會導致一定的性能問題，不過能保證程序的安全運行。

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var counter = struct {
		sync.RWMutex
		m map[string]int
	}{m: make(map[string]int)}

	// To read from the counter, take the read lock:
	counter.RLock()
	n := counter.m["some_key"]
	counter.RUnlock()
	fmt.Println("some_key:", n)
	
	// To write to the counter, take the write lock:
	counter.Lock()
	counter.m["some_key"]++
	counter.Unlock()
	fmt.Println("some_key:", counter.m["some_key"])
}
```

* [Go 1.9 sync.Map揭秘](https://colobu.com/2017/07/11/dive-into-sync-Map/)
* [go語言坑之並發訪問map](https://www.jianshu.com/p/10a998089486)

# Iteration order

用 `range` 迭代 map 時，並沒有指定每次的順序一樣，也沒有保證下一次的順序會跟上一次的順序一樣，每次都是隨機的

如果希望能夠每次迭代的順序都一樣的話，必須先將 key 單獨分開來做排序，在迭代排序好的 key mapping 回 map

```go
package main

import (
	"fmt"
	"sort"
)

func main() {

	m := map[int]string{
		1: "A",
		2: "B",
		3: "C",
	}
	// 先將 key 取出來排序
	var keys []int
	for k := range m {
		keys = append(keys, k)
	}
	sort.Ints(keys)
	// 改成以下方法，就會倒過來
	// sort.Sort(sort.Reverse(sort.IntSlice(keys)))

	// 迭代排序好的 slice，並指定 map 的 key
	for _, k := range keys {
		fmt.Println("Key:", k, "Value:", m[k])
	}
}
```

# cannot assign to struct field XXX in map

當 struct 作為 map 裡面的值時，不能透過 map[key].xx = "xx" 這種賦值，會出現 `cannot assign to struct field XXX in map` 不予許修改 map 裡的值

```go
package main

import (
	"fmt"
)

type test struct {
	name string
}

func main() {
	a := test{"hello"}
	mapp := make(map[string]test)
	mapp["hey"] = a

	// 因為 map 的 value 是不可尋址的，因此會報錯
	// cannot assign to struct field mapp["test"].name in map
	mapp["hey"].name = "hi"

	// if v, ok := mapp["hey"]; ok {
	// 雖然這樣不會有 error，但實際上 v 是 copy 的值，因此也不會改到原本的 value，所以還是 hello
	// 	v.name = "hi"
	// }

	fmt.Println(mapp["hey"].name)

}
```

### 原因:

map 的 value 是不可尋址的([addressable](https://golang.org/ref/spec#Address_operators))，因為 map 中的值會在記憶體中行動，舊的指針地址在 map 改變時會變得無效。

另外 map 是會自動擴容，因此原來存值是 A 地址，擴容後 A 地址就不是原來的值了，因此如果需要改值，必須改用 `make(map[string]*test)`

參考文件

* [Go maps in action](https://blog.golang.org/go-maps-in-action)
* [Why do I get a “cannot assign” error when setting value to a struct as a value in a map? [duplicate]](https://stackoverflow.com/questions/32751537/why-do-i-get-a-cannot-assign-error-when-setting-value-to-a-struct-as-a-value-i)
* [問一個問題。為啥結構體作為map的值，不能透過map[key].成員屬性 = "Xxx" 這種賦值](https://gocn.vip/question/1714)
* [Golang面試題解析（四）](https://studygolang.com/articles/12714)
