---
layout: post
title: "Golang - slices usage and internals"
date: 2019-02-03 10:08:15 +0800
comments: true
categories: golang
---

<!-- more -->

# Introduction

golang 提供很方便 slice，去處理序列化的數據. slice 類似其他語言的 array，但具有特別的屬性

# Arrays

slices 類型是建立在 array 類型上面，因此要了解 slice 必須先瞭解 array

array 必須指定 type 和 size，像是 `[4]int` 代表裡面的值必須是 `int`，並且只能放 4 個值

array size 是固定的，像是 `[4]int` 和 `[5]int` 就不一樣，而且是不相容的類型

```go
var a [4]int
a[0] = 1
i := a[0]
// i == 1
```

array 不需要顯示的初始化 (initialized explicitly)

> 是指說不需要有 assign 值，像 int 就會是 0，string 就是 “”，不像 slice 或 map 需要用 make()

```go
// a[2] == 0, the zero value of the int type
```

`[4]int` 指的是，照順序排列的 4 個整數值

![](https://blog.golang.org/go-slices-usage-and-internals_slice-array.png)

Go's arrays are values.

array variable 表示整個 array，並不是指向 array 的第一個元素，這表示當 assign 值給 array 時，將會複製整個 array 的內容 (為了避免複製，可以傳遞一個指向 array 的 pointer)

```go
package main

import (
	"fmt"
)

func assign(num [4]int) {
	fmt.Printf("assign %p\n", &num)
	fmt.Printf("assign %p\n", &num[0])
	fmt.Printf("assign %p\n", &num[1])
}

func main() {
	var a [4]int

	fmt.Printf("main %p\n", &a)
	fmt.Printf("main %p\n", &a[0])
	fmt.Printf("main %p\n", &a[1])
	assign(a)

}

// 可以發現當 assign 之後，address 就不一樣
// main 0x416020
// main 0x416020
// main 0x416024
// assign 0x416050
// assign 0x416050
// assign 0x416054
```

另一種思考是將 array 當成一個 struct，但是索引是 `index` 而不是 `name`

```go
b := [2]string{"Penn", "Teller"}
```

也可以用 `...` 讓編譯器，在編譯的時候幫你計算

```go
b := [...]string{"Penn", "Teller"}
```

上面兩個範例的 `b` 都代表 `[2]string`

# Slices

上面 array size 都是固定，很不彈性，所以在 golang 當中比較少會使用到，比較常看到的反而是 `slices` 以 array 為基礎，但是更加彈性方便

```go
// 不需要像 array 給 length
[]T
```

slice 宣告方式，跟 array 很像

```go
letters := []string{"a", "b", "c", "d"}
```

可以透過 make 來建立 slice

```go
func make([]T, len, cap) []T
```

用 make 建立時，要給定 `type`，`len`，`cap`，make 就會分配一個 array，並回傳 reference 到這個 array 的 slice

```go
var s []byte
s = make([]byte, 5, 5)
// s == []byte{0, 0, 0, 0, 0}
```

`cap` 可以不寫，默認為 `cap` 跟 `len` 一樣

```go
s := make([]byte, 5)
len(s) == 5
cap(s) == 5
```

sliec 和 array 可以透過 `[:]` 來建立另一個 slice，透過前後數字，表示要切的長度

```go
b := []byte{'g', 'o', 'l', 'a', 'n', 'g'}
// b[1:4] == []byte{'o', 'l', 'a'}, sharing the same storage as b
```

slice 前後的索引，是可選的，前面不存在代表從 0 開始，後面不存在代表該長度

```go
// b[:2] == []byte{'g', 'o'}
// b[2:] == []byte{'l', 'a', 'n', 'g'}
// b[:] == b
```

兩個都不寫也可以，就是原本的數組

```go
x := [3]string{"Лайка", "Белка", "Стрелка"}
s := x[:] // a slice referencing the storage of x
```

# Slice internals

A slice is a descriptor of an array segment.

slice 的組成包含 element 的指標(pointer)、長度(len)、容量(cap)

![](https://blog.golang.org/go-slices-usage-and-internals_slice-struct.png)

`make([]byte, 5)` 像是下面這樣，指標指向 slice 的第一個 element，並對應到 array 中 element

![](https://blog.golang.org/go-slices-usage-and-internals_slice-1.png)

length 是 slice 引用的元素數，capacity 則是 array 底層可以使用到的數量(是指 slice pointer 指到的位置開始數起)

```go
s = s[2:4]
// 範圍是 2~4(不包含 4 的位置)，所以長度會是 2
```

以這張圖表示，slice pointer 指向 array[2]，並且 length = 2, capacity = 3

![](https://blog.golang.org/go-slices-usage-and-internals_slice-2.png)

slicing 不會 copy 原本的 slice，而是將 pointer 指向到原來的 array，這也讓 slice 可以那麼有效率的原因

但這也代表，不同 slice 會對應到同一個 array，因此改變 array 的值，所有有指到該 array 的值都會變

```go
d := []byte{'r', 'o', 'a', 'd'}
e := d[2:] 
// e == []byte{'a', 'd'}
e[1] = 'm'
// e == []byte{'a', 'm'}
// d == []byte{'r', 'o', 'a', 'm'}
```

之前把 length 切的比 capacity 還要短，可以透過在 slicing 一次，把長度加長

```go
s = s[:cap(s)]
```

![](https://blog.golang.org/go-slices-usage-and-internals_slice-3.png)

slice 不能超過其 capacity，否則會 `runtime panic`，cap 會由 slice 的起點到最後面，因此每次起點都不是 0 cap 則會一直變小

從 0 開始切

```go
a := []int{1, 2, 3, 4}
b := a[:2]
fmt.Println(cap(b[:]))
fmt.Println(b)
// 4
// [1 2]
```

從 2 開始切

```go
a := []int{1, 2, 3, 4}
b := a[2:]
fmt.Println(cap(b[:]))
fmt.Println(b)
// 2
// [3 4]
```

# Growing slices (the copy and append functions)

要新增 slice 的 capacity 的方式，就是建立一個新的 slice，並將原本的內容複製過去(這種動態數組實現技術，是來自其他語言)

建立新的 t slice，將 s 的內容複製到 t，再將 t 分配給 s，就能使 s 對應到加倍 cap 的 t

```go
t := make([]byte, len(s), (cap(s)+1)*2) // +1 in case cap(s) == 0
for i := range s {
        t[i] = s[i]
}
s = t
```

loop 複製的部分，可以透過內建的 `copy` 簡單完成

```go
func copy(dst, src []T) int
```

也可以在不同長度的 slice 進行 copy

```go
// s 複製到 t
t := make([]byte, len(s), (cap(s)+1)*2)
copy(t, s)
s = t
```

更常見的操作，是將 data append 在後面，必要時會產生新的 slice

```go
func AppendByte(slice []byte, data ...byte) []byte {
	 // 原本 slice 的長度
    m := len(slice)
    // 原本 slice 的長度 + 新的 data 的長度
    n := m + len(data)
    // 如果加總的長度，比原本的 slice 的容量還長，就建立新的 slice
    if n > cap(slice) { // if necessary, reallocate
        // allocate double what's needed, for future growth.
        newSlice := make([]byte, (n+1)*2)
        copy(newSlice, slice)
        slice = newSlice
    }
    // 將 slice 切到跟加總的長度一樣
    slice = slice[0:n]
    // 將新的 data 複製到，slice 的後面
    copy(slice[m:n], data)
    return slice
}
```

像是這樣

```go
p := []byte{2, 3, 5}
p = AppendByte(p, 7, 11, 13)
// p == []byte{2, 3, 5, 7, 11, 13}
```

像 AppendByte 的 function 非常有用，因為可以控制 slice 擴充方式

golang 內建就有 `append`

```go
func append(s []T, x ...T) []T
```

`func append` 可以將 x 元素，append 在 data 後面，如果需要更大的容量，會自動分配新的 slice

```go
a := make([]int, 1)
// a == []int{0}
a = append(a, 1, 2, 3)
// a == []int{0, 1, 2, 3}
```

要 append slice to slice 可以用 `...`，將參數作為列表

```go
a := []string{"John", "Paul"}
b := []string{"George", "Ringo", "Pete"}
a = append(a, b...) // equivalent to "append(a, b[0], b[1], b[2])"
// a == []string{"John", "Paul", "George", "Ringo", "Pete"}
```

```go
package main

import (
	"fmt"
)

// Filter returns a new slice holding only
// the elements of s that satisfy fn()
func Filter(s []int, fn func(int) bool) []int {
	var p []int // == nil
	for _, v := range s {
		if fn(v) {
			p = append(p, v)
		}
	}
	return p
}

func main() {
	p := []int{2, 3, 5, 11, 13, 15}
	f := func(x int) bool {
		return x > 10
	}
	p = Filter(p, f)
	fmt.Println(p)
}

// [11 13 15]
```


# A possible "gotcha"

之前提到的 re-slice，不會複製到底層的 array，因此完整的 array 會一直留存在記憶體，導致只需要一小部分數據時，卻將所有數據保存在 memory 中

以下為存文件載入到 memory，並回傳 slice 包含第一組的連續數字

```go
var digitRegexp = regexp.MustCompile("[0-9]+")

func FindDigits(filename string) []byte {
    b, _ := ioutil.ReadFile(filename)
    return digitRegexp.Find(b)
}
```

此行為就會造成，雖然只要第一組的連續數字，但是 slice 卻會 references 到原本的 array(完整的文件)，導致 memory 的浪費，並且 garbage collector 無法去做釋放

要解決此問題，要先將需要的字串複製到新的 slice，這樣原本的 slice 就可以做釋放

```go
func CopyDigits(filename string) []byte {
    b, _ := ioutil.ReadFile(filename)
    b = digitRegexp.Find(b)
    // 建立新的 slice，長度容量設定為需要的長度即可
    c := make([]byte, len(b))
    copy(c, b)
    return c
}
```

也可以使用 `append` 建立更簡潔的 function

```go
func CopyDigits(filename string) []byte {
    b, _ := ioutil.ReadFile(filename)
    b = digitRegexp.Find(b)
    // 建立新的 slice，長度容量設定為需要的長度即可
    c := make([]int, 1)
    append(c, b...)
    return c
}
```

參考文件

* [Go Slices: usage and internals](https://blog.golang.org/go-slices-usage-and-internals)
* [深入解析 Go 中 Slice 底層實現](https://halfrost.com/go_slice/)
