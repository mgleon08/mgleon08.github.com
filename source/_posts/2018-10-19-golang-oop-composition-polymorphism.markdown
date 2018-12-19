---
layout: post
title: "Golang - OOP, Composition, Polymorphism"
date: 2018-10-19 22:43:55 +0800
comments: true
categories: golang
---

<!-- more -->

* [object oriented programming](#oop)
* [Composition](#composition)
* [Polymorphism](#polymorphism)

# <span id="oop"> object oriented programming </span>

golang 本身並不是純的 OOP，在官網也有寫道 [Is Go an object-oriented language?](https://golang.org/doc/faq#Is_Go_an_object-oriented_language)

但可以透過 golang 的 `struct` & `method` 表現的跟其他語言的 class 類似

### Structs Instead of Classes

這裡的 `employee.go` 就像是 class 一樣，在 `main.go` 裡面可以使用

```go
oop
├── employee
│   └── employee.go
└── main.go
```

```go
// employee.go
package employee

import (  
    "fmt"
)

type Employee struct {  
    FirstName   string
    LastName    string
    TotalLeaves int
    LeavesTaken int
}

func (e Employee) LeavesRemaining() {  
    fmt.Printf("%s %s has %d leaves remaining", e.FirstName, e.LastName, (e.TotalLeaves - e.LeavesTaken))
}
```

```go
// main.go
package main

import "oop/employee"

func main() {  
    e := employee.Employee {
        FirstName: "Sam",
        LastName: "Adolf",
        TotalLeaves: 30,
        LeavesTaken: 20,
    }
    e.LeavesRemaining()
}
```

### New() function instead of constructors

如果上面改成以下，並不會報錯，會顯示 `has 0 leaves remaining` 但是這並不是正確的，因為人名都不見了

```go
// main.go
package main

import "oop/employee"

func main() {  
    var e employee.Employee
    e.LeavesRemaining()
}
```

為了防止這樣，因此我們可以透過 `func` 建立類似 `建構子 constructors` 的方式

```go
// employee.go
package employee

import (  
    "fmt"
)

// 改成小寫，因為只需要內部使用
type employee struct {  
    firstName   string
    lastName    string
    totalLeaves int
    leavesTaken int
}

// 大寫，讓外部要呼叫只能透過 New()
func New(firstName string, lastName string, totalLeave int, leavesTaken int) employee {  
    e := employee {firstName, lastName, totalLeave, leavesTaken}
    return e
}

func (e employee) LeavesRemaining() {  
    fmt.Printf("%s %s has %d leaves remaining", e.firstName, e.lastName, (e.totalLeaves - e.leavesTaken))
}
```

```go
// main.go
package main

import "oop/employee"

func main() {  
    e := employee.New("Sam", "Adolf", 30, 20)
    e.LeavesRemaining()
}
```

# <span id="composition"> Composition </span>

go 並不支援繼承(Inheritance)，但支援組合(Composition)，透過 `embedding structs` 來達成

```go
package main

import (  
    "fmt"
)

type author struct {  
    firstName string
    lastName  string
    bio       string
}

func (a author) fullName() string {  
    return fmt.Sprintf("%s %s", a.firstName, a.lastName)
}

type post struct {  
    title     string
    content   string
    author
}

func (p post) details() {  
    fmt.Println("Title: ", p.title)
    fmt.Println("Content: ", p.content)
    fmt.Println("Author: ", p.author.fullName()) 
    // 這裡就是透過 embedding structs，也可以改寫成 p.fullName()，因為 fullName 並沒有跟 post 裡的 method 衝突到
    fmt.Println("Bio: ", p.author.bio)
}

func main() {  
    author1 := author{
        "Naveen",
        "Ramanathan",
        "Golang Enthusiast",
    }
    post1 := post{
        "Inheritance in Go",
        "Go supports composition instead of inheritance",
        author1,
    }
    post1.details()
}
```

### Embedding slice of structs

```go
package main

import (  
    "fmt"
)

type author struct {  
    firstName string
    lastName  string
    bio       string
}

func (a author) fullName() string {  
    return fmt.Sprintf("%s %s", a.firstName, a.lastName)
}

type post struct {  
    title   string
    content string
    author
}

func (p post) details() {  
    fmt.Println("Title: ", p.title)
    fmt.Println("Content: ", p.content)
    fmt.Println("Author: ", p.fullName())
    fmt.Println("Bio: ", p.bio)
}

type website struct {  
 posts []post
}
func (w website) contents() {  
    fmt.Println("Contents of Website")
    for _, v := range w.posts {
        v.details()
        fmt.Println()
    }
}

func main() {  
    author1 := author{
        "Naveen",
        "Ramanathan",
        "Golang Enthusiast",
    }
    post1 := post{
        "Inheritance in Go",
        "Go supports composition instead of inheritance",
        author1,
    }
    post2 := post{
        "Struct instead of Classes in Go",
        "Go does not support classes but methods can be added to structs",
        author1,
    }
    post3 := post{
        "Concurrency",
        "Go is a concurrent language and not a parallel one",
        author1,
    }
    w := website{
        posts: []post{post1, post2, post3},
    }
    w.contents()
}
```

# <span id="polymorphism"> Polymorphism </span>

go 要達成多態，必須透過 interface 來實現，只要有 type 實作了 interface 的 function，就可以說這個 type 實作了這個 interface，因此透過這個特性，就能達成 Polymorphism

```go
package main

import (  
    "fmt"
)

type Income interface {  
    calculate() int
    source() string
}

type FixedBilling struct {  
    projectName string
    biddedAmount int
}

type TimeAndMaterial struct {  
    projectName string
    noOfHours  int
    hourlyRate int
}

// FixedBilling 實作 interface 的 calculate() & source()
func (fb FixedBilling) calculate() int {  
    return fb.biddedAmount
}

func (fb FixedBilling) source() string {  
    return fb.projectName
}

// TimeAndMaterial 實作 interface 的 calculate() & source()
func (tm TimeAndMaterial) calculate() int {  
    return tm.noOfHours * tm.hourlyRate
}

func (tm TimeAndMaterial) source() string {  
    return tm.projectName
}

// ic 只接收實作了 Income interface 的 type，達成 Polymorphism
func calculateNetIncome(ic []Income) {  
    var netincome int = 0
    for _, income := range ic {
        fmt.Printf("Income From %s = $%d\n", income.source(), income.calculate())
        netincome += income.calculate()
    }
    fmt.Printf("Net income of organisation = $%d", netincome)
}

func main() {  
    project1 := FixedBilling{projectName: "Project 1", biddedAmount: 5000}
    project2 := FixedBilling{projectName: "Project 2", biddedAmount: 10000}
    project3 := TimeAndMaterial{projectName: "Project 3", noOfHours: 160, hourlyRate: 25}
    incomeStreams := []Income{project1, project2, project3}
    calculateNetIncome(incomeStreams)
}
```


參考文件

* [golangbot.com](https://golangbot.com/learn-golang-series/)
