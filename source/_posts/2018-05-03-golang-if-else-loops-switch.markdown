---
layout: post
title: "Golang - if else, loops, switch"
date: 2018-05-03 18:42:50 +0800
comments: true
categories: golang
---

<!-- more -->

* [if else 條件式](#if-else)
* [迴圈 loops (for、while、each)](#loops)
  * [range (each)](#range)
* [switch 條件式](#switch)

# <span id="if-else">if else 條件式</span>

### syntax

* `else` 一定要接續在 `}` 否則會 `syntax error`(因為 golang 會自動在後面加上 `;`)

```go
if condition {  
} else if condition {
} else {
}
```

```go
if statement; condition {  
}
```

### example

```go
package main

import (
	"fmt"
	"math"
)

func pow(x, n, lim float64) float64 {
	// 判斷式前，先宣告變數
	if v := math.Pow(x, n); v < lim {
		return v // v 範圍只在 if 的 scope 裡
	} else if v == lim {
		fmt.Printf("%g = %g\n", v, lim)
	} else {

		fmt.Printf("%g >= %g\n", v, lim)
	}
	return lim // 若改成 v 則會出現 undefined: v
}

func main() {
	fmt.Println(
		pow(3, 2, 10),
		pow(3, 3, 20), // 因為有換行所以必須要加上 ,
	)
}

// 27 >= 20
// 9 20
```

# <span id="loops">迴圈 loops (for、while、each)</span>

Golang 中只有 for 一種迴圈，但能夠達成 for、while、foreach 多種用法

### syntaxs

```go
for initialisation; condition; post {  
}
```

### example

```go
package main

import (
	"fmt"
)

func main() {
	// 必須先定義起始的變數
	// The most basic type, with a single condition.
	// 1, 2, 3
	i := 1
	for i <= 3 {
		fmt.Println(i)
		i = i + 1
	}

	// A classic initial/condition/after `for` loop.
	// 7, 8, 9
	for j := 7; j <= 9; j++ {
		fmt.Println(j)
	}

	// initialisation and post are omitted
	for i <= 10 {
		fmt.Printf("%d ", i)
		i += 2
	}
	
	//multiple initialisation and increment
	for no, i := 10, 1; i <= 10 && no <= 19; i, no = i+1, no+1 {
	    fmt.Printf("%d * %d = %d\n", no, i, no*i)
	}

	// You can also `continue` to the next iteration of
	// the loop.
	for n := 0; n <= 5; n++ {
		if n%2 == 0 {
			continue
		}
		fmt.Println(n)
	}
}
```

### infinite loop (while)

```go
package main

import (
	"fmt"
)

func main() {
	// `for` without a condition will loop repeatedly
	// until you `break` out of the loop or `return` from
	// the enclosing function.
	// loop
	for {
		fmt.Println("loop")
		break
	}
}
```

```go
// 實作 math.Sqrt(x) func, 並驗證與原本的 func 是否相同
package main

import (
	"fmt"
	"math"
)

func Sqrt(x float64) (float64, int) {
	z := 1.0
	index := 1
	for {
		z -= (z*z - x) / (2 * z)
		fmt.Println(z)
		index++
		if z == math.Sqrt(x) {
			break
		}
	}

	return z, index
}

func main() {
	i := float64(169)
	output, index := Sqrt(i)
	fmt.Println("index:", index)
	fmt.Println("output:", output)
}

/**
85
43.49411764705882
23.68985027605849
15.41185354894443
13.188719595702175
13.00135021013767
13.000000070110696
13
index: 9
output: 13
**/
```

###<span id="range">range (each)</span>

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

# <span id="switch">switch 條件式</span>


```go
// 判斷式前，先宣告變數
package main

import (
	"fmt"
	"runtime"
)

func main() {
	fmt.Print("Go runs on ")
	switch os := runtime.GOOS; os {
	case "darwin":
		fmt.Println("OS X.")
	case "linux":
		fmt.Println("Linux.")
	default:
		// freebsd, openbsd,
		// plan9, windows...
		fmt.Printf("%s.", os)
	}
}

// Go runs on nacl.
```
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	fmt.Println("When's Saturday?")
	today := time.Now().Weekday()
	switch time.Saturday {
	case today + 0:
		fmt.Println("Today.")
	case today + 1:
		fmt.Println("Tomorrow.")
	case today + 2:
		fmt.Println("In two days.")
	default:
		fmt.Println("Too far away.")
	}
}

// When's Saturday?
// Too far away.
```

### Expressionless switch

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	t := time.Now()
	switch { // expression is omitted
	case t.Hour() < 12:
		fmt.Println("Good morning!")
	case t.Hour() < 17:
		fmt.Println("Good afternoon.")
	default:
		fmt.Println("Good evening.")
	}
}

```

### Multiple expressions in case

```go
package main

import (
	"fmt"
)

func main() {
	letter := "i"
	switch letter {
	case "a", "e", "i", "o", "u": //multiple expressions in case
		fmt.Println("vowel")
	default:
		fmt.Println("not a vowel")
	}
}
// vowel
```

### fallthrough

可以遇到條件符合之後，繼續往下走

```go
package main

import (
	"fmt"
)

func number() int {
	num := 15 * 5
	return num
}

func main() {

	switch num := number(); { //num is not a constant
	case num < 50:
		fmt.Printf("%d is lesser than 50\n", num)
		fallthrough
	case num < 100:
		fmt.Printf("%d is lesser than 100\n", num)
		fallthrough
	case num < 200:
		fmt.Printf("%d is lesser than 200", num)
	}

}

// 75 is lesser than 100
// 75 is lesser than 200
```

參考文件:

* [golangbot.com](https://golangbot.com/learn-golang-series/)