---
layout: post
title: "Golang - conversions"
date: 2018-04-27 22:38:09 +0800
comments: true
categories: golang
---

<!-- more -->

# <span id="conversions"> 轉型 conversions  </span>

* [fmt](https://golang.org/pkg/fmt/)
* [Unicode Character Set and UTF-8, UTF-16, UTF-32 Encoding](https://naveenr.net/unicode-character-set-and-utf-8-utf-16-utf-32-encoding/)
* [UTF-8 encoder/decoder](https://mothereff.in/utf-8#%C3%B1)


```go
Type(value)
```

```go
// default type
i := 42      // int
f := 3.14    // float64
b := true    // bool
s := "Hello" // string
r := 'A'     // rune
```

```go
package main

import "fmt"

func main(){
	a, b := 100, 2.5
	// float64 轉 int 會有小數點不見得問題
	a = a * int(b)
	fmt.Println(a)

	a, b = 100, 2.5
	// 因為 a type 是 int 所以最後必須再轉回 int
	a = int(float64(a) * b)
	fmt.Println(a)
}
```

* 當有運算子類型確定，則會轉為該類型再運算
* 當兩個運算子都為 typeless 且不同 type，會自動轉為需要的類型 (前提是兩種 type 是可以互相轉換)

```go
a := 1 + 2.3
a := float64(1 + 2.3)
```

* 當兩個運算子都為 typeless 相同 type，兩個各別轉換為 `default type`

```go
const min = 1
max := 5 + min
max := int(5) + int(min)
```


```go
package main

import (
	"fmt"
)

func main() {
	// 類型確定為 int
	var num1 int = 7
	fmt.Println(num1 / 2)
	// 3
	fmt.Println(num1 / 2.0)
	// 3
	
	// 類型確定為 float
	var num2 float64 = 7.0
	fmt.Println(num2 / 2)
	// 3.5
	fmt.Println(num2 / 2.0)
	// 3.5
	
	// 自動推導為 int
	var num3 = 7
	fmt.Println(num3 / 2)
	// 3
	fmt.Println(num3 / 2.0)
	// 3
	
	// 自動推導為 float
	var num4 = 7.0
	fmt.Println(num4 / 2)
	// 3.5
	fmt.Println(num4 / 2.0)
	// 3.5
	
	// typeless
	const num5 = 7
	fmt.Println(num5 / 2) // int(num5 / 2)
	// 3
	fmt.Println(num5 / 2.0) // float64(num5 / 2.0)
	// 3.5
	
	// typeless
	const num6 = 7.0
	fmt.Println(num6 / 2) // float64(num6 / 2)
	// 3.5
	fmt.Println(num6 / 2.0) // float64(num6 / 2)
	// 3.5
	
	fmt.Println(7 / 2)
	// 3
	fmt.Println(7 / 2.0) // float64(7 / 2.0)
	// 3.5
	fmt.Println(7.0 / 2) // float64(7.0 / 2)
	// 3.5
}
```

### Typeless

當沒有特別指定 type 時，為 typeless, `1`, `3.14`, `const a = 1`.. 並且和有 type 的值做運算時，會自動轉換為該 type

```go
const min = 1
var b byte = min

// Go implicitly converts a typeless constant to a typed value
var b = byte(min)
```

Example

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	t := 10
	fmt.Println(time.Second * t)
}

// invalid operation: time.Second * t (mismatched types time.Duration and int)
```

此時 `t := 10 (自動判斷 deault type int)` type 為 `int`，因此不同 type 是無法相乘，要改成 

```go
// 直接轉換
time.Duration(t)

// typeless，因此會被轉換成 Duration
const t = 10
// or 
fmt.Println(time.Second * 10)
```

### Basic Types

> `*` 為比較常使用

```go
- *string
- *bool
- Numeric Types
  - *int  int8  int16  *int32(rune)  int64
  - uint *uint8(byte) uint16 uint32 uint64
  - float32 *float64
  - complex64 complex128
  - byte // alias for uint8
  - rune // alias for int32，represents a Unicode code point
```

### strconv 轉換不同 type

```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	i := 123
	i64 := int64(i)                               // int to int64
	f64 := float64(i)                             // int to float64
	is := strconv.Itoa(i)                         // int to string
	i64s := strconv.FormatInt(i64, 10)            //int64 to string 10 進位
	f64s := strconv.FormatFloat(f64, 'f', -1, 64) // float64 to string
	f64i64 := int64(f64)                          // float64 to int64
	letter := "abc"                               // string to byte
	si, sierr := strconv.Atoi(is)                 // string to int
	si64, si64err := strconv.ParseInt(is, 10, 64) // string to int64 [func ParseInt(s string, base int, bitSize int) (i int64, err error)]
	sf64, sf64err := strconv.ParseFloat(is, 64)   //  string to float64 [func ParseFloat(s string, bitSize int) (f float64, err error)]
	sb, sberr := strconv.ParseBool("true")        // string to bool [func ParseBool(str string) (value bool, err error)]

	fmt.Printf("i      Type: %T, Value: %v\n", i, i)
	fmt.Printf("i64    Type: %T, Value: %v\n", i64, i64)
	fmt.Printf("f64    Type: %T, Value: %v\n", f64, f64)
	fmt.Printf("is     Type: %T, Value: %v\n", is, is)
	fmt.Printf("i64s   Type: %T, Value: %v\n", i64s, i64s)
	fmt.Printf("f64s   Type: %T, Value: %v\n", f64s, f64s)
	fmt.Printf("f64i64 Type: %T, Value: %v\n", f64i64, f64i64)
	fmt.Printf("letter Type: %T, Value: %v\n", []byte(letter), []byte(letter))
	fmt.Printf("si     Type: %T, Value: %v Err: %v\n", si, si, sierr)
	fmt.Printf("si64   Type: %T, Value: %v Err: %v\n", si64, si64, si64err)
	fmt.Printf("sf64   Type: %T, Value: %v Err: %v\n", sf64, sf64, sf64err)
	fmt.Printf("sb     Type: %T, Value: %v Err: %v\n", sb, sb, sberr)

}

/*
i      Type: int, Value: 123
i64    Type: int64, Value: 123
f64    Type: float64, Value: 123
is     Type: string, Value: 123
i64s   Type: string, Value: 123
f64s   Type: string, Value: 123
f64i64 Type: int64, Value: 123
letter Type: []uint8, Value: [97 98 99]
si     Type: int, Value: 123 Err: <nil>
si64   Type: int64, Value: 123 Err: <nil>
sf64   Type: float64, Value: 123 Err: <nil>
sb     Type: bool, Value: true Err: <nil>
*/
```

### underlying type

* [time/Duration](https://golang.org/pkg/time/#Duration)

在 `time` package 定義了一個 type `Duration` 底層的 type 是 int64

```go
type Duration int64
```

但是當 `Duration` 和 `a` 相乘會發生 `invalid operation: h *= a (mismatched types time.Duration and int64)` 因為雖然底層都是 `int64` 但 golang 是強型別，不允許不同的 type 作處理，因此必須再透過 `time.Duration(a)
` 轉成 `Duration type`

> 只要底層一樣就可以做轉換 `time.Duration(a)` `ounce(g)`


```go
package main

import (
	"fmt"
	"time"
)

func main() {
	h, _ := time.ParseDuration("4h30m")
	fmt.Printf("%T\n", h)
	fmt.Printf("I've got %.1f hours of work left.\n", h.Hours())
	
	var a int64 = 2
	// 必須轉換成 time.Duration type 才可以相乘
	h *= time.Duration(a)
	fmt.Println(h)
}
```

```go
package main

import (
	"fmt"
)

type gram float64
type ounce float64

func main() {
	var g gram = 1000
	var o ounce
	// 必須轉換成 ounce type 才可以相乘
	o = ounce(g) * 0.035274
	fmt.Printf("%g grams is %0.2f ounces\n", g, o)
}
```

### String

> Since a string is a slice of bytes, its possible to access each byte of a string

```go
package main

import (
	"fmt"
)

func printBytes(s string) {
	for i := 0; i < len(s); i++ {
		// %x base 16, lower-case, two characters per byte
		fmt.Printf("%x ", s[i])
	}
}

func printChars(s string) {
	// 這裡需要用 rune(int32)，因為有些字像中文會佔據 UTF-8 3個 byte，無法直接用 %c 對應到相對應的字，必須要一個 byte 才可以對應
	runes := []rune(s)
	// 這邊可以改用 range，range 會直接轉成 rune 就不需要另外轉
	for i := 0; i < len(runes); i++ {
		// %c the character represented by the corresponding Unicode code point
		fmt.Printf("%c ", runes[i])
	}
}

func printCharsAndBytes(s string) {
	for index, rune := range s {
		fmt.Printf("%c starts at byte %d\n", rune, index)
	}
}

func main() {
	name := "哈囉"
	fmt.Println(len(name)) // 6，中文字串是用3個位元組存的
	printBytes(name)
	fmt.Printf("\n")
	printChars(name)
	fmt.Printf("\n")
	printCharsAndBytes(name)

	fmt.Printf("\n\n")

	name = "Señor"
	fmt.Println(len(name))
	printBytes(name)
	fmt.Printf("\n")
	printChars(name)
	fmt.Printf("\n")
	printCharsAndBytes(name)

	fmt.Printf("\n")

	// Constructing string from slice of bytes
	byteSlice := []byte{0x43, 0x61, 0x66, 0xC3, 0xA9}ㄋ
	// 也可以改用 10 進位 []byte{67, 97, 102, 195, 169} 結果一樣
	str := string(byteSlice)
	fmt.Println(str)

	fmt.Printf("\n")

	// Constructing a string from slice of runes
	runeSlice := []rune{0x0053, 0x0065, 0x00f1, 0x006f, 0x0072}
	runestr := string(runeSlice)
	fmt.Println(runestr)
}

/**
6
e5 93 88 e5 9b 89
哈 囉
哈 starts at byte 0
囉 starts at byte 3


6
53 65 c3 b1 6f 72
S e ñ o r
S starts at byte 0
e starts at byte 1
ñ starts at byte 2
o starts at byte 4
r starts at byte 5

Café

Señor
**/
```

### Length of the string

必須用到 `utf8` package

```go
package main

import (
    "fmt"
    "unicode/utf8"
)

func length(s string) {
    fmt.Printf("length of %s is %d\n", s, utf8.RuneCountInString(s))
}
func main() {
    word1 := "Señor"
    length(word1)
    word2 := "Pets"
    length(word2)
}
```

也可以用 `len()` 但是 `len` 是 `length of a string in bytes`，所以有些字如果是 `2byte` 像是 `é` 則會長度計算為 2

```go
package main

import (
	"fmt"
)

func main() {
	a := "Hello"
	fmt.Println(len(a))
}
```

### Strings are immutable

string 是不可變的

```go
package main

import (
    "fmt"
)

func mutate(s []rune) string {
    s[0] = 'a' // 改變後
    return string(s) // 在組成 string 傳回來
}

func main() {
    h := "hello"
    // mutate(h) 直接將 string 帶進去會出現 cannot assign to s[0]
    fmt.Println(mutate([]rune(h))) // 必須先轉成 rune slice type
}
```

* [[golangbot] strings](https://golangbot.com/strings/)

### complex

```go
package main

import (
    "fmt"
)

func main() {
    c1 := complex(5, 7)
    c2 := 8 + 27i
    cadd := c1 + c2
    fmt.Println("sum:", cadd)
    cmul := c1 * c2
    fmt.Println("product:", cmul)
}

// sum: (13+34i)
// product: (-149+191i)
```


### float

```go
package main

import (
	"fmt"
)

func main() {
	// float/float
	fmt.Println(7 / 2.0)
	s := []byte("1234567")
	// int/int
	fmt.Println(len(s) / 2.0)
}
// 3.5
// 3
```

* [Conversions](https://golang.org/ref/spec#Conversions)
* [go：整數除以浮點數的問題](https://segmentfault.com/q/1010000011519048)
