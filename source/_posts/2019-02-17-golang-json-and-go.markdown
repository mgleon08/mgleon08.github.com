---
layout: post
title: "Golang - JSON and Go"
date: 2019-02-17 22:40:16 +0800
comments: true
categories: golang
---

<!-- more -->

# Encoding - Marshal 序列化

在 Go 中並不是所有的類型都能進行序列化：

* 必須是 struct 中支援外部引用的 field 才能夠序列化 (集開頭字母大寫的 field)
* JSON object key 只支援 string
* Channel、complex、function 等 type 無法進行序列化
* 不支援循環數據結構，因為序列化時會進行無限迴圈
* `Pointer` 序列化之後是其指向的值或者是 `nil`


```go
func Marshal(v interface{}) ([]byte, error
```

```go
package main

import (
	"encoding/json"
	"fmt"
)

type UserInfo struct {
	Name string
	Say  string
	Age int64 
}

func main() {
	m := UserInfo{"Leon", "Hello", 18}
	b, _ := json.Marshal(m)
	fmt.Println(string(b))
}

// {"Name":"Leon","Say":"Hello","Age":18}
```

# 指定 JSON filed name

### Struct Tag

`Struct tag` 可以決定 `Marshal` 和 `Unmarshal` 函式如何序列化和反序列化數據。

* 一般 `json` 都是小寫，但因為 golang 序列化時，struct 必須是大寫，因此透過 `Struct Tag` 改成小寫

```go
package main

import (
	"encoding/json"
	"fmt"
)

type UserInfo struct {
	Name string `json:"name"`
	Say  string `json:"say"`
	Age int64   `json:"age"`
}

func main() {
	m := UserInfo{"Leon", "Hello", 18}
	b, _ := json.Marshal(m)
	fmt.Println(string(b))
}

// {"name":"Leon","say":"Hello","age":18}
```

# 指定 field 的行為

### omitempty

透過 `omitempty` 如果 field 的值是對應類型的 `zero-value`，序列化之後的 `JSON object` 中不包含此 `field`

```go
package main

import (
	"encoding/json"
	"fmt"
)

type UserInfo struct {
	Name string `json:"name"`
	Say  string `json:"say"`
	Age int64   `json:"age,omitempty"`
}

func main() {
	m := UserInfo{}
	b, _ := json.Marshal(m)
	fmt.Println(string(b))
}

// {"name":"","say":""}
```

### -

透過 `-` 可以忽略掉該 field

```go
package main

import (
	"encoding/json"
	"fmt"
)

type UserInfo struct {
	Name string `json:"name"`
	Say  string `json:"-"`
	Age int64   `json:"age,omitempty"`
}

func main() {
	m := UserInfo{"Leon", "Hello", 18}
	b, _ := json.Marshal(m)
	fmt.Println(string(b))
}

// {"name":"Leon","age":18}
```

# Decoding - Unmarshal 反序列化

預設的 JSON 支援以下幾種 Go 類型：

```go
bool for JSON booleans
float64 for JSON numbers
string for JSON strings
nil for JSON null
```

### 知道類型的反序列化

```go
package main

import (
	"encoding/json"
	"fmt"
)

type UserInfo struct {
	Name string `json:"name"`
	Say string  `json:"say"`
	Age int64   `json:"age"`
}

func main() {
	var jsonString string
	jsonString = `{"name":"Leon","say":"hello","age":18}`
	
	//把 json unmarshal 進去 struct
	u := &UserInfo{}
	err := json.Unmarshal([]byte(jsonString), u)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println(u)
	fmt.Printf("%T\n", u)
	fmt.Printf("name:%s, say:%s, age:%d\n", u.Name, u.Say, u.Age)
}

// &{Leon hello 18}
// *main.UserInfo
// name:Leon, say:hello, age:18
```

### 不知道類型的反序列化

如果不知道類型可以用 `interface{}` 

```go
package main

import (
	"encoding/json"
	"fmt"
)

func main() {
	var jsonString string
	jsonString = `{"name":"Leon","age":18,"cars":["Maserati","BMW"]}`
	
	//把 json unmarshal 進去 struct
	var u interface{}
	err := json.Unmarshal([]byte(jsonString), &u)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println(u)
	fmt.Printf("%T\n", u)
}

// map[cars:[Maserati BMW] name:Leon age:18]
// map[string]interface {}
```

回傳的格式如

```go
map[string]interface{}{
    "Name": "Leon",
    "age":  18,
    "cars": []interface{}{
        "Maserati",
        "BMW",
    },
}
```

key 是 `string`，value 是 `interface{}`，必須透過 `type assertion` 和 `range` 取得所有的 key

```go
package main

import (
	"encoding/json"
	"fmt"
)

func showInfo(u interface{}){
	m := u.(map[string]interface{})
	for k, v := range m {
		switch vv := v.(type) {
		case string:
			fmt.Println(k, "is string", vv)
		case float64:
			fmt.Println(k, "is float64", vv)
		case []interface{}:
			fmt.Println(k, "is an array:")
			for i, u := range vv {
				fmt.Println(i, u)
			}
		default:
			fmt.Println(k, "is of a type I don't know how to handle")
		}
	}
}

func main() {
	var jsonString string
	jsonString = `{"name":"Leon","age":18,"cars":["Maserati","BMW"]}`

	//把 json unmarshal 進去 struct
	var u interface{}
	err := json.Unmarshal([]byte(jsonString), &u)
	if err != nil {
		fmt.Println(err)
		return
	}
	showInfo(u)
}

// name is string Leon
// age is float64 18
// cars is an array:
// 0 Maserati
// 1 BMW
```

# Reference Types - 反序列化對 slice、map、pointer 的處理

struct 中包含一個 `slice Cars` ，slice 預設是 `nil`，之所以反序列化可以正常進行就是因為 Unmarshal 在序列化時進行了對 `slice Cars` 做了初始化，同理，對 `map` 和 `pointer` 都會做類似的工作

> 比如序列化如果 Pointer 不是 nil 首先進行 dereference 獲得其指向的值，然後再進行序列化，反序列化時首先對 nil pointer 進行初始化

```go
package main

import (
    "encoding/json"
    "fmt"
)

type UserInfo struct {
    Name string 
    Age  int64  
    Cars []string
}

func main() {
    var jsonString string
    jsonString = `{"name":"Leon","age":18,"cars":["Maserati","BMW"]}`
    
    //把 json unmarshal 進去 struct
    u := &UserInfo{}
    err := json.Unmarshal([]byte(jsonString), u)
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Println(u)
    fmt.Printf("%T\n", u)
    fmt.Printf("name:%s, age:%d, cars:%s\n", u.Name, u.Age, u.Cars)
}

// &{Leon 18 [Maserati BMW]}
// *main.UserInfo
// name:Leon, age:18, cars:[Maserati BMW]
```

# nested struct 的序列化

Go 支援對 `nested struct` 進行序列化和反序列化:

```go
package main

import (
    "encoding/json"
    "fmt"
)

type App struct {
	Id string `json:"id"`
}

type Org struct {
	Name string `json:"name"`
}

type AppWithOrg struct {
	App
	Org
}

func main() {
	data := []byte(`{ "id": "123", "name": "My Awesome Org" }`)
	var b AppWithOrg
	json.Unmarshal(data, &b)
	fmt.Printf("%#v\n\n", b)
	
	a := AppWithOrg{
		App: App{ Id: "321" },
		Org: Org{ Name: "My Awesome Org"},
	}
	data, _ = json.Marshal(a)
	fmt.Println(string(data))
}

// main.AppWithOrg{App:main.App{Id:"123"}, Org:main.Org{Name:"My Awesome Org"}}
// {"id":"321","name":"My Awesome Org"}
```

# Stream JSON

Golang 提供 `Decoder` 和 `Encoder` 對 stream JSON 進行處理，常見 request 中的 Body、文件等

```go
package main

import (
	"encoding/json"
	"fmt"
	"io"
	"log"
	"strings"
)

func main() {
	const jsonStream = `
	{"Name": "Ed", "Text": "Knock knock."}
	{"Name": "Sam", "Text": "Who's there?"}
	{"Name": "Ed", "Text": "Go fmt."}
	{"Name": "Sam", "Text": "Go fmt who?"}
	{"Name": "Ed", "Text": "Go fmt yourself!"}
`
	type Message struct {
		Name, Text string
	}
	dec := json.NewDecoder(strings.NewReader(jsonStream))
	for {
		var m Message
		if err := dec.Decode(&m); err == io.EOF {
			break
		} else if err != nil {
			log.Fatal(err)
		}
		fmt.Printf("%s: %s\n", m.Name, m.Text)
	}
}

// Ed: Knock knock.
// Sam: Who's there?
// Ed: Go fmt.
// Sam: Go fmt who?
// Ed: Go fmt yourself!
```


參考文件

* [json](https://golang.org/pkg/encoding/json/#Marshal)
* [理解 Go 中的 JSON](https://sanyuesha.com/2018/05/07/go-json/)
* [JSON and Go](https://blog.golang.org/json-and-go)
