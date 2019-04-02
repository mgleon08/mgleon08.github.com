---
layout: post
title: "Golang - testing"
date: 2019-03-28 18:24:12 +0800
comments: true
categories: golang
---

<!-- more -->

# Testing

* 檔案的結尾必須是，`測試檔案名稱` + `_test`
* 測試函式必須是 `Test` 開頭，且必須第一個字母大寫(public)
* 測試函式 `TestXxx()` 的參數是 `testing.T`，我們可以使用該型別來記錄錯誤或者是測試狀態
* 要使用 `testing` package
* 測試案例會按照原始碼中寫的順序依次執行
* 函式中透過呼叫 `testing.T` 的 `Error`, `Errorf`, `FailNow`, `Fatal`, `FatalIf` 方法，說明測試不透過，呼叫 `Log` 方法用來記錄測試的資訊。

### main.go

```go
// main.go
package main

import (
	"errors"
	"fmt"
)

func Division(a, b float64) (float64, error) {
	if b == 0 {
		return 0, errors.New("b can not be 0")
	}
	return a / b, nil
}

func main() {
	fmt.Println(Division(60, 2))
}
```


### main_test.go

```go
// main_test.go
package main

import (
	"testing"
)

func Test_division_1(t *testing.T) {
	if i, e := division(6, 2); i != 3 || e != nil {
		t.Error("失敗")
	} else {
		t.Log("成功")
	}
}

func Test_division_2(t *testing.T) {
	t.Error("失敗")
}

func Test_division_table(t *testing.T) {
	tables := []struct {
		x float64
		y float64
	}{
		{3, 1},
		{6, 2},
		{9, 3},
		{8, 2},
	}

	for _, table := range tables {
		if i, e := division(table.x, table.y); i != 3 || e != nil {
			t.Error("失敗")
		} else {
			t.Log("成功")
		}
	}
}
```

### Run test

```go
// go test -v -cover

=== RUN   Test_Division_1
--- PASS: Test_Division_1 (0.00s)
    main_test.go:11: 成功
=== RUN   Test_Division_2
--- FAIL: Test_Division_2 (0.00s)
    main_test.go:16: 失敗
=== RUN   Test_Division_table
--- FAIL: Test_Division_table (0.00s)
    main_test.go:34: 成功
    main_test.go:34: 成功
    main_test.go:34: 成功
    main_test.go:32: 失敗
FAIL
coverage: 50.0% of statements
exit status 1
FAIL    github.com/mgleon08/packages/015.test   0.007s
```

# Benchmark testing

* 檔案的結尾必須是，`測試檔案名稱` + `_test`
* 測試函式必須是 `Benchmark ` 開頭，且必須第一個字母大寫(public)
* `for` 循環內要放置要測試的程式碼
* `b.N` 是 go 語言內建提供的循環，根據一秒鐘的時間計算
* 測試函式 `Benchmark_xxx()` 的參數是 `b *testing.B`

### webbench_test.go

```go
// webbench_test.go

package main

import (
	"testing"
)

func Benchmark_Division(b *testing.B) {
	for i := 0; i < b.N; i++ { //use b.N for looping
		Division(4, 5)
	}
}

func Benchmark_TimeConsumingFunction(b *testing.B) {
	b.StopTimer() //呼叫該函式停止壓力測試的時間計數

	//做一些初始化的工作，例如讀取檔案資料，資料庫連線之類別的,
	//這樣這些時間不影響我們測試函式本身的效能

	b.StartTimer() //重新開始時間
	for i := 0; i < b.N; i++ {
		Division(4, 5)
	}
}
```

### Run benchmark test

跑測試 `go test -v -bench=. -run=none`

```go
// go test -v -bench=. -run=none . -cover 

goos: darwin
goarch: amd64
pkg: github.com/mgleon08/packages/015.test
Benchmark_Division-4                    2000000000               0.43 ns/op
Benchmark_TimeConsumingFunction-4       2000000000               0.42 ns/op
PASS
coverage: 50.0% of statements
ok      github.com/mgleon08/packages/015.test   1.802s
```

* `-bench` - 要跑 Benchmark
* `-run=none` - 不要跑一般的測試
* `-4` - 目前的 CPU 核心數，也就是 GOMAXPROCS 的值
*  `-benchtime=2s` - 加上這個代表跑 2 秒


# Example testing

* 檔案的結尾必須是，`測試檔案名稱` + `_test`
* 測試函式必須是 `Example ` 開頭，且必須第一個字母大寫(public)
* `Output:` 是固定格式

```go
package main

import (
	"fmt"
)

func ExampleDivision() {
	i, _ := Division(6, 1)
	fmt.Print(i)
	// Output: 6
}

func ExampleDivision2() {
	i, _ := Division(6, 2)
	x, _ := Division(12, 3)
	fmt.Println(i)
	fmt.Println(x)
	// Output:
	// 3
	// 4
}
```

Reference

* [11.3 Go 怎麼寫測試案例](https://github.com/astaxie/build-web-application-with-golang/blob/master/zh-tw/11.3.md)
* [如何在 Go 專案內寫測試](https://blog.wu-boy.com/2018/05/how-to-write-testing-in-golang/)
* [如何在 Go 語言內寫效能測試](https://blog.wu-boy.com/2018/06/how-to-write-benchmark-in-go/)
* [Example 示例函式](https://ithelp.ithome.com.tw/articles/10206088)
