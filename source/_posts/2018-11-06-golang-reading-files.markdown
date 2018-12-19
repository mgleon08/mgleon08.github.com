---
layout: post
title: "Golang - Reading Files"
date: 2018-11-06 15:16:53 +0800
comments: true
categories: golang
---

<!-- more -->

* [Reading an entire file into memory ](#entire)
* [Reading a file in small chunks](#chunk)
* [Reading a file line by line](#line)

# <span id="entire"> Reading an entire file into memory </span>

Use [ioutil](https://golang.org/pkg/io/ioutil/) package

* [ReadFile](https://golang.org/pkg/io/ioutil/#ReadFile)

```go
// 先建立檔案

filehandling
├── filehandling.go
└── test.txt
```

```go
// filehandling.go
package main

import (
    "fmt"
    "io/ioutil"
)

func main() {
    data, err := ioutil.ReadFile("test.txt")
    if err != nil {
        // 當找不到 test file 時，會顯示此訊息
        fmt.Println("File reading error", err)
        return
    }
    // return data 是 slice 透過 string 轉
    // 或是改用 fmt.Printf("Contents of file: %s", data)
    fmt.Println("Contents of file:", string(data))
}
```

當你用 `go install` or `go build` 出 binary 的 file，file 移到不同位置就會找不到檔案，有幾種方式可以解決

### 1. 用絕對路徑
### 2. 把路徑當參數，在執行檔案時傳入

必須用到 [flag](https://golang.org/pkg/flag/) package

```go
// flag.go
package main

import (
    "flag"
    "fmt"
)

func main() {
    // 第一個參數 flag name, 第二個 default value, 第三個 flag description
    fptr := flag.String("fpath", "test.txt", "file path to read from")
    // 必須先執行，parse flag
    flag.Parse()
    fmt.Println("value of fpath is", *fptr)
}
```

```go
go run flag.go -fpath=/path-of-file/test.txt
```

更改後為

```go
package main

import (
    "flag"
    "fmt"
    "io/ioutil"
)

func main() {
    fptr := flag.String("fpath", "test.txt", "file path to read from")
    flag.Parse()
    data, err := ioutil.ReadFile(*fptr)
    if err != nil {
        fmt.Println("File reading error", err)
        return
    }
    fmt.Println("Contents of file:", string(data))
}
```

3. 在編譯的時候把檔案一起編譯進去

可以透過 [packr](https://github.com/gobuffalo/packr) package 來實現

```go
package main

import (
	"fmt"

	"github.com/gobuffalo/packr"
)

func main() {
	box := packr.NewBox("../filehandling")
	data, err := box.FindString("test.txt")
	if err != nil {
		fmt.Println(err)
	} else {
		fmt.Println("Contents of file:", data)
	}
}
```

```go
// 用 go 安裝，編譯成絕對路徑，因此任何地方都可以執行 filehandling
// 但是只要將 test.txt 刪除或改名，就會找不到
go install -v

// 用 packr 安裝，則是連 file 也編譯進去，因此即使把檔案刪除也可以執行
packr install -v filehandling

/*
building box ../filehandling
packing file filehandling.go
packed file filehandling.go
packing file test.txt
packed file test.txt
built box ../filehandling with ["filehandling.go" "test.txt"]
filehandling
*/
```

# <span id="chunk"> Reading a file in small chunks </span>

前面都是透過 memory 來一次讀取 file，但檔案太大，或是 memory 不夠的時候，就會導致錯誤，因此可以透過另一種方式來執行 [bufio](https://golang.org/pkg/bufio/) 將檔案內容做分割

```go
package main

import (
	"bufio"
	"flag"
	"fmt"
	"log"
	"os"
)

func main() {
  // 先透過 flag 讀取參數
	fptr := flag.String("fpath", "test.txt", "file path to read from")
  // 解析參數
	flag.Parse()

  // 打開檔案
	f, err := os.Open(*fptr)
	if err != nil {
		log.Fatal(err)
	}
  // function 結束前關閉 file
	defer func() {
		if err = f.Close(); err != nil {
			log.Fatal(err)
		}
	}()
  // returns a new Reader whose buffer has the default size.
	r := bufio.NewReader(f)
  // 建立 byte slice，容量 3，用來一次讀取 3 個
	b := make([]byte, 3)
	for {
		_, err := r.Read(b)
		if err != nil {
			fmt.Println("Error reading file:", err)
			break
		}
		fmt.Println(string(b))
	}
}s
```

# <span id="line"> Reading a file line by line </span>

上面是一次要讀取多少個字元，這次改用一行一行去做讀取，一樣用 [bufio](https://golang.org/pkg/bufio/)

```go
package main

import (
	"bufio"
	"flag"
	"fmt"
	"log"
	"os"
)

func main() {
	// 先透過 flag 讀取參數
	fptr := flag.String("fpath", "test.txt", "file path to read from")
	// 解析參數
	flag.Parse()

	// 打開檔案
	f, err := os.Open(*fptr)
	if err != nil {
		log.Fatal(err)
	}
	// function 結束前關閉 file
	defer func() {
		if err = f.Close(); err != nil {
			log.Fatal(err)
		}
	}()
	// 透過 Scanner 將一行一行印出
	s := bufio.NewScanner(f)
	for s.Scan() {
		fmt.Println(s.Text())
	}
	err = s.Err()
	if err != nil {
		log.Fatal(err)
	}
}
```

參考文件:

* [golangbot - Reading Files](https://golangbot.com/read-files/)
