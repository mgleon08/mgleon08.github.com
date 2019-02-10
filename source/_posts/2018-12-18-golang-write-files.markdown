---
layout: post
title: "Golang - Writing Files"
date: 2018-12-18 15:16:53 +0800
comments: true
categories: golang
---

<!-- more -->

* [Writing string to a file](#string)
* [Writing bytes to a file](#bytes)
* [Writing strings line by line to a file](#line)
* [Appending to a file](#appending)
* [Writing to file concurrently](#concurrently)

# <span id="string"> Writing string to a file </span>

* [os.File](https://golang.org/pkg/os/#File)

1. Create the file
2. Write the string to the file

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// 建立新的 txt 檔案
	f, err := os.Create("test.txt")
	if err != nil {
		fmt.Println(err)
		return
	}
	// 將 String 寫入檔案內
	l, err := f.WriteString("Hello World")
	if err != nil {
		fmt.Println(err)
		// 關閉檔案
		f.Close()
		return
	}
	fmt.Println(l, "bytes written successfully")
	// 關閉檔案
	err = f.Close()
	if err != nil {
		fmt.Println(err)
		return
	}
}
// 11 bytes written successfully
```

# <span id="bytes"> Writing bytes to a file </span>

[File.Write](https://golang.org/pkg/os/#File.Write)

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	f, err := os.Create("$GOPATH/src/write/bytes")
	if err != nil {
		fmt.Println(err)
		return
	}
	// slice of bytes
	d2 := []byte{104, 101, 108, 108, 111, 32, 119, 111, 114, 108, 100}
	n2, err := f.Write(d2)
	if err != nil {
		fmt.Println(err)
		f.Close()
		return
	}
	fmt.Println(n2, "bytes written successfully")
	err = f.Close()
	if err != nil {
		fmt.Println(err)
		return
	}
}
// 11 bytes written successfully
```

# <span id="line"> Writing strings line by line to a file </span>

* [Fprintln](https://golang.org/pkg/fmt/#Fprintln)

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	f, err := os.Create("lines")
	if err != nil {
		fmt.Println(err)
		f.Close()
		return
	}
	d := []string{"Welcome to the world of Go1.", "Go is a compiled language.", "It is easy to learn Go."}

	for _, v := range d {
		// Fprintln formats using the default formats for its operands and writes to w. Spaces are always added between operands and a newline is appended
		fmt.Fprintln(f, v)
		if err != nil {
			fmt.Println(err)
			return
		}
	}
	err = f.Close()
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println("file written successfully")
}

// file written successfully
```

# <span id="appending"> Appending to a file </span>

* [OpenFile](https://golang.org/pkg/os/#OpenFile)

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// open the file in append and write only mode
	// 0644 權限 -rw-r--r--
	f, err := os.OpenFile("lines", os.O_APPEND|os.O_WRONLY, 0644)
	if err != nil {
		fmt.Println(err)
		return
	}
	newLine := "File handling is easy."
    // Fprintln formats using the default formats for its operands and writes to w. Spaces are always added between operands and a newline is appended
	_, err = fmt.Fprintln(f, newLine)
	if err != nil {
		fmt.Println(err)
		f.Close()
		return
	}
	err = f.Close()
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println("file appended successfully")
}

// file appended successfully
```

# <span id="concurrently"> Writing to file concurrently </span>

當同時 goroutines 寫進同一個 file 時，會發生 race condition，因此必須避免

### 步驟

1. 建立一個 channel 來傳遞要填寫的內容
2. 一次建立 100 個 producer goroutines，並且同時寫一個亂數到 channel
3. 建立一個 consumer goroutine 用來讀取 channel 裡面的 value，因為只有一個所以可以避免造成 race condition
4. 關閉檔案

```go
package main

import (
	"fmt"
	_ "math/rand"
	"os"
	"sync"
)

// 用來產生亂數，並傳遞給 channel
func produce(data chan int, wg *sync.WaitGroup, number int) {
	// n := rand.Intn(999)
	// data <- n
	// 改用 1 - 99 比較看的出，goroutine 每次完成的順序都不一樣
	data <- number
	// 執行完一次就 -1
	wg.Done()
}

func consume(data chan int, done chan bool) {
	// 建立檔案 concurrent
	f, err := os.Create("concurrent")
	if err != nil {
		fmt.Println(err)
		return
	}
	// 讀取 data channel 裡面的
	for d := range data {
		_, err = fmt.Fprintln(f, d)
		if err != nil {
			fmt.Println(err)
			f.Close()
			done <- false
			return
		}
	}
	// close file
	err = f.Close()
	if err != nil {
		fmt.Println(err)
		done <- false
		return
	}
	// 到這邊就是完成，傳 true 進去
	done <- true
}

func main() {
	data := make(chan int)
	done := make(chan bool)
	wg := sync.WaitGroup{}
	for i := 0; i < 100; i++ {
		// 每次都加一
		wg.Add(1)
		// 建立 100 goroutine
		go produce(data, &wg, i)
	}
	go consume(data, done)

	// 等待 100 個 goroutine 完成
	go func() {
		// 會 wait 到 0 才會繼續下一步
		wg.Wait()
		// 將 channel 關閉，用 range 取出 channel 的值必須關閉
		close(data)
	}()
	// 等到 done 有東西，才會繼續
	d := <-done
	if d == true {
		fmt.Println("File written successfully")
	} else {
		fmt.Println("File writing failed")
	}
}
// File written successfully
```

參考文件:

* [golangbot - Writing Files](https://golangbot.com/write-files/)
