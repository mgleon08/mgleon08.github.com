---
layout: post
title: "Golang - goroutine, channel, worker pool, select, mutex"
date: 2018-05-17 23:10:28 +0800
comments: true
categories: golang
---

<!-- more -->

* [Concurrency](#concurrency)
* [Parallelism](#parallelism)
* [Goroutine](#goroutine)
* [Channel](#channel)
* [Buffered Channels](#buffered_channels)
* [Worker Pool](#worker_pool)
* [Select](#select)
* [sync.Mutex](mutex)

# Concurrency &  Parallelism

> Goroutines are part of making concurrency easy to use.

要了解 Goroutine 可能要先了解一下，Concurrency 跟 parallelism 的差別 

### <span id="concurrency"> Concurrency </span>


> Concurrency is the capability to deal with lots of things at once.

簡單的解釋，一個在跑步的人，因為鞋帶鬆了，所以他停下來綁鞋帶，綁完後繼續跑。

![](http://i.imgur.com/WOthNHb.png)

### <span id="parallelism"> Parallelism </span>

> Parallelism is doing lots of things at the same time. It might sound similar to concurrency but its actually different.

同樣用慢跑來解釋的話，在慢跑的人，同時在用耳機聽音樂，在同一時間做了很多事。

![](http://i.imgur.com/cTonbBE.png)

* [concurrency & parallelism](https://golangbot.com/concurrency/)

# <span id="goroutine"> Goroutine </span>


> Goroutines are functions or methods that run concurrently with other functions or methods. 

* go 本身就有數千個 goroutine 在跑
* goroutine can be thought of as light weight threads.
* 與 threads 相比，goroutine 成本非常小
* goroutine 最多運行 GOMAXPROCS 數量(可以設定)
* main() 也是一個 goroutine 稱為 `main Goroutine`

### Example 1

```go
package main

import (  
    "fmt"
    "time"
)

func hello() {  
    fmt.Println("Hello world goroutine")
}

func main() {  
    go hello()
    // time.Sleep(1 * time.Second)
    fmt.Println("main function")
}

// main function
// 因為 hello() 進入背景處理，但是 main() 已經結束，因此所有的 goroutine 都會直接打斷，程序退出。
// 加入 sleep 1 秒，讓 hello() 有足夠的時間 retuen 回來
```

### Example 2

```go
package main

import (
	"fmt"
	"time"
)

func numbers() {
	for i := 1; i <= 5; i++ {
		time.Sleep(250 * time.Millisecond)
		fmt.Printf("%d ", i)
	}
}
func alphabets() {
	for i := 'a'; i <= 'e'; i++ {
		time.Sleep(400 * time.Millisecond)
		fmt.Printf("%c ", i)
	}
}
func main() {
	go numbers()
	go alphabets()
	time.Sleep(3000 * time.Millisecond)
	fmt.Println("main terminated")
}

// 1 a 2 3 b 4 c 5 d e main terminated
```

![](https://golangbot.com/content/images/2017/07/Goroutines-explained.png)

* [Goroutine是如何工作的](https://tonybai.com/2014/11/15/how-goroutines-work/)
* [[golangbot.com] goroutines](https://golangbot.com/goroutines/)

# <span id="channel"> Channel </span>

> Channels can be thought as pipes using which Goroutines communicate. Similar to how water flows from one end to another in a pipe, data can be sent from one end and received from the another end using channels.

* Channel 是 goroutine 之間相互通信的機制（goroutine之間是相互獨立的，因此需要 Channel 來做溝通）
* Channel 中使用的 type 稱之為 element type，比如 int 類型的 channel 寫作為 `chan int`
* Go 使用 make 內建函式建立 channel
* The zero value of a channel is `nil`

```go
package main

import "fmt"

func main() {
	var a chan int
	if a == nil {
		fmt.Println("channel a is nil, going to define it")
		a = make(chan int)
		fmt.Printf("Type of a is %T", a)
	}
}

// channel a is nil, going to define it
// Type of a is chan int
```

```go
a := make(chan int)  
```

### send、receive、close

```go
data := <- ch // read from channel ch 
<- ch // read from channel ch
ch <- data // write to channel ch
```

### Sends and receives are blocking by default

> * When a data is sent to a channel, the control is blocked in the send statement until some other Goroutine reads from that channel.
> * Similarly when data is read from a channel, the read is blocked until some Goroutine writes data to that channel.
> * 如果另一方一直對沒有動作，會造成 `Deadlock`

##### Example

`<-done` 這行會導致 `main goroutine` blocked 在這邊，直到其他 goroutine 將 data 寫入 done，不然是不會繼續往下走，也意味著就不需要用 sleep 來停止

```go
package main

import (  
    "fmt"
    "time"
)

func hello(done chan bool) {  
    fmt.Println("hello go routine is going to sleep")
    time.Sleep(4 * time.Second)
    fmt.Println("hello go routine awake and going to write to done")
    done <- true
}
func main() {  
    done := make(chan bool)
    fmt.Println("Main going to call hello go goroutine")
    go hello(done)
    // 這邊 done channel，沒有任何東西，因此被 blockeds 住，等有東西到 channel 才會繼續下一行
    <- done 
    fmt.Println("Main received data")
}

// Main going to call hello go goroutine
// hello go routine is going to sleep
// hello go routine awake and going to write to done
// Main received data
```

##### Exampale

```go
package main

import (  
    "fmt"
)

func calcSquares(number int, squareop chan int) {  
    sum := 0
    for number != 0 {
        digit := number % 10
        sum += digit * digit
        number /= 10
    }
    squareop <- sum
}

func calcCubes(number int, cubeop chan int) {  
    sum := 0 
    for number != 0 {
        digit := number % 10
        sum += digit * digit * digit
        number /= 10
    }
    cubeop <- sum
} 

func main() {  
    number := 589
    sqrch := make(chan int)
    cubech := make(chan int)
    go calcSquares(number, sqrch)
    go calcCubes(number, cubech)
    squares, cubes := <-sqrch, <-cubech
    fmt.Println("Final output", squares + cubes)
}

// Final output 1536
```

refactor with close & range

```go
package main

import (  
    "fmt"
)

func digits(number int, dchnl chan int) {  
    for number != 0 {
        digit := number % 10
        dchnl <- digit
        number /= 10
    }
    close(dchnl)
}
func calcSquares(number int, squareop chan int) {  
    sum := 0
    dch := make(chan int)
    go digits(number, dch)
    for digit := range dch {
        sum += digit * digit
    }
    squareop <- sum
}

func calcCubes(number int, cubeop chan int) {  
    sum := 0
    dch := make(chan int)
    go digits(number, dch)
    for digit := range dch {
        sum += digit * digit * digit
    }
    cubeop <- sum
}

func main() {  
    number := 589
    sqrch := make(chan int)
    cubech := make(chan int)
    go calcSquares(number, sqrch)
    go calcCubes(number, cubech)
    squares, cubes := <-sqrch, <-cubech
    fmt.Println("Final output", squares+cubes)
}

// Final output 1536
```

### Deadlock

當 Goroutine send data 到 channel，但沒有其他的 Goroutine 去接收這個 data，就會造成 Deadlock，並且出現錯誤 `panic`

```go
package main

func main() {  
    ch := make(chan int)
    ch <- 5
}
```

### Unidirectional channels (單向 channels)

到目前為止，說的都是 bidirectional channels(雙向 channels)

channel 也可以是單向的，`only send or receive data`

```go
package main

import "fmt"

func sendData(sendch chan<- int) {  
    sendch <- 10
}

func main() {  
    sendch := make(chan<- int) // 單向 channel
    go sendData(sendch)
    fmt.Println(<-sendch)
}

// main.go:11: invalid operation: <-sendch (receive from send-only type chan<- int)
```
以上只有 send 功能，並沒有 receive，因此會報錯，但事實上只有 send 的 channel 也是沒有什麼意義

```go
package main

import "fmt"

func sendData(sendch chan<- int) { // 單向 channel
	sendch <- 10
}

func main() {
	chnl := make(chan int) // 雙向 channel
	go sendData(chnl)
	fmt.Println(<-chnl)
}

// 10
```
將單向改成雙向，並且透過 `func` 來控制單向

### Closing channels and for range loops on channels

* Only the sender should close a channel, never the receiver. Sending on a closed channel will cause a `panic`
* Channels aren't like files; you don't usually need to close them. Closing is only necessary when the receiver must be told there are no more values coming, such as to terminate a `range` loop.

```go
v, ok := <- ch
// ok 
// true: 可以接收的狀態
// false: 沒有任何 value & channel 已經關閉
```

##### Examples

```go
package main

import (
	"fmt"
)

func producer(chnl chan int) {
	for i := 0; i < 3; i++ {
		chnl <- i
	}
	close(chnl) // close
}
func main() {
	ch := make(chan int)
	go producer(ch)
	// 用 range，當 channel close 會自動離開
	for v := range ch {
		fmt.Println("Received ", v)
	}
	/*
	for {
		v, ok := <-ch
		if ok == false {
			break
		}
		fmt.Println("Received ", v, ok)
	}
	*/
}

// Received  0 true
// Received  1 true
// Received  2 true
// 沒有 close 會造成 fatal error: all goroutines are asleep - deadlock!
```

##### Example

```go
package main

import (
	"fmt"
)

func fibonacci(n int, c chan int) {
	x, y := 0, 1
	for i := 0; i < n; i++ {
		c <- x
		x, y = y, x+y
	}
	close(c)
}

func main() {
	c := make(chan int, 10)
	go fibonacci(cap(c), c)
	for i := range c {
		fmt.Println(i)
	}
}

// 0
// 1
// 1
// 2
// 3
// 5
// 8
// 13
// 21
// 34
```

# <span id="buffered_channels"> Buffered Channels </span>

> Provide the buffer length as the second argument to make to initialize a buffered channel:

```go
ch := make(chan int, 100)
// ch := make(chan type, capacity)
// The capacity for an unbuffered channel is 0 
```

* 當 buffer 滿了，sends 就會 block 住 
* 當 buffer 空了，receives 就會 block 住


### Example

```go
package main

import "fmt"

func main() {
	ch := make(chan int, 3)
	ch <- 1
	ch <- 2 
	// ch <- 3 再多傳一個就會 deadlock
	fmt.Println("capacity is", cap(ch))
	fmt.Println("length is", len(ch))
	fmt.Println(<-ch)
	fmt.Println(<-ch)
	// fmt.Println(<-ch) 多接收一個也會 deadlock
	fmt.Println("capacity is", cap(ch))
	fmt.Println("length is", len(ch))
}

/*
capacity is 3
length is 2
1
2
capacity is 3
length is 0
*/
```

# <span id="worker_pool"> Worker Pool </span>

### WaitGroup

透過 WaitGroup，可以讓所有的 Goroutine 都跑完，最後再結束

```go
package main

import (  
    "fmt"
    "sync"
    "time"
)

func process(i int, wg *sync.WaitGroup) {  
    fmt.Println("started Goroutine ", i)
    time.Sleep(2 * time.Second)
    fmt.Printf("Goroutine %d ended\n", i)
    wg.Done() // 執行完一次就 -1
}

func main() {  
    no := 3
    var wg sync.WaitGroup
    for i := 0; i < no; i++ {
        wg.Add(1) // 每次執行都 + 1
        go process(i, &wg) // wg 一定要用 pointer，否則每個 goroutine 都會有各自的 WaitGroup
    }
    wg.Wait() // 會 wait 到 0 才會繼續下一步
    fmt.Println("All go routines finished executing")
}
```

what is [worker pool](https://en.wikipedia.org/wiki/Thread_pool)?

> a worker pool is a collection of threads which are waiting for tasks to be assigned to them. Once they finish the task assigned, they make themselves available again for the next task.

* [WaitGroup](https://golang.org/pkg/sync/#WaitGroup)

### Example

* [Worker Pool Implementation](https://golangbot.com/buffered-channels-worker-pools/)

# <span id="select"> Select </span>

* The select statement lets a goroutine wait on multiple communication operations.
* A select blocks until one of its cases can run, then it executes that case. It chooses one at random if multiple are ready.

> The syntax is similar to switch except that each of the case statement will be a channel operation

```go
package main

import (
	"fmt"
	"time"
)

func server1(ch chan string) {
	time.Sleep(6 * time.Second)
	ch <- "from server1"
}
func server2(ch chan string) {
	time.Sleep(3 * time.Second)
	ch <- "from server2"

}
func main() {
	output1 := make(chan string)
	output2 := make(chan string)
	go server1(output1)
	go server2(output2)
	
	// 等待到其中一個 channel 回來，就執行，如果都有就會隨機
	select {
	case s1 := <-output1:
		fmt.Println(s1)
	case s2 := <-output2:
		fmt.Println(s2)
	}
}

// from server2
```



```go
package main

import "fmt"

func fibonacci(c, quit chan int) {
	x, y := 0, 1
	for {
		select {
		case c <- x:
			x, y = y, x+y
		case <-quit:
			fmt.Println("quit")
			return
		}
	}
}

func main() {
	c := make(chan int)
	quit := make(chan int)
	go func() {
		for i := 0; i < 10; i++ {
			fmt.Println(<-c)
		}
		quit <- 0
	}()
	fibonacci(c, quit)
}

// 0
// 1
// 1
// 2
// 3
// 5
// 8
// 13
// 21
// 34
// quit
```

### Default Selection

The default case in a select is run if no other case is ready.

```go
// Use a default case to try a send or receive without blocking:

select {
case i := <-c:
    // use i
default:
    // receiving from c would block
}
```

```go
package main

import (
	"fmt"
	"time"
)

func process(ch chan string) {
	time.Sleep(10500 * time.Millisecond)
	ch <- "process successful"
}

func main() {
	ch := make(chan string)
	go process(ch)
	for {
		time.Sleep(1000 * time.Millisecond)
		select {
		case v := <-ch:
			fmt.Println("received value: ", v)
			return
		default:
			fmt.Println("no value received")
		}
	}

}
```

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	tick := time.Tick(100 * time.Millisecond)
	boom := time.After(500 * time.Millisecond)
	for {
		select {
		case <-tick:
			fmt.Println("tick.")
		case <-boom:
			fmt.Println("BOOM!")
			return
		default:
			fmt.Println("    .")
			time.Sleep(50 * time.Millisecond)
		}
	}
}
```

# <span id="mutex"> sync.Mutex </span>

### [Critical section](https://zh.wikipedia.org/wiki/%E8%87%A8%E7%95%8C%E5%8D%80%E6%AE%B5)

一個存取共用資源（例如：共用裝置或是共用記憶體）的程式片段，而這些共用資源有無法同時被多個執行緒存取的特性。

當兩個以上的 goroutine 對同一個 value 做計算時，可能會因為前後順序的差異，導致最後的結果不同。(e.g. x = x + 1)



### [Mutex](https://tip.golang.org/pkg/sync/#Mutex)

A Mutex is used to provide a locking mechanism to ensure that only one Goroutine is running the critical section of code at any point of time to prevent race condition from happening.


```go
// Mutex is available in the sync package
mutex.Lock()  
x = x + 1  
mutex.Unlock() 

// 同時間只會有一個 goroutine
```

### Example

```go
package main

import (
	"fmt"
	"sync"
)

var x = 0

func increment(wg *sync.WaitGroup, m *sync.Mutex) {
	m.Lock()
	x = x + 1
	m.Unlock()
	wg.Done()
}
func main() {
	var w sync.WaitGroup
	var m sync.Mutex
	for i := 0; i < 1000; i++ {
		w.Add(1)
		go increment(&w, &m) // 這裡一定要用 address
	}
	w.Wait()
	fmt.Println("final value of x", x)
}

// 1000
// 如果沒加上 lock，同時就會有很多個 goroutine 在跑，導致最後結果不一樣
```

用 buffered channel 的方式

```go
package main

import (
	"fmt"
	"sync"
)

var x = 0

func increment(wg *sync.WaitGroup, ch chan bool) {
	ch <- true // 當前面的 channel 還有東西，就會 block 住，導致後面無法繼續
	x = x + 1
	<-ch
	wg.Done()
}
func main() {
	var w sync.WaitGroup
	ch := make(chan bool, 1)
	for i := 0; i < 1000; i++ {
		w.Add(1)
		go increment(&w, ch)
	}
	w.Wait()
	fmt.Println("final value of x", x)
}
```

### Example 2

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

// SafeCounter is safe to use concurrently.
type SafeCounter struct {
	v   map[string]int
	mux sync.Mutex
}

// Inc increments the counter for the given key.
func (c *SafeCounter) Inc(key string) {
	c.mux.Lock()
	// Lock so only one goroutine at a time can access the map c.v.
	c.v[key]++
	c.mux.Unlock()
}

// Value returns the current value of the counter for the given key.
func (c *SafeCounter) Value(key string) int {
	c.mux.Lock()
	// Lock so only one goroutine at a time can access the map c.v.
	defer c.mux.Unlock()
	return c.v[key]
}

func main() {
	c := SafeCounter{v: make(map[string]int)}
	for i := 0; i < 1000; i++ {
		go c.Inc("somekey")
	}

	time.Sleep(time.Second)
	fmt.Println(c.Value("somekey"))
}
```

參考文件:

* [[golangbot.com]](https://golangbot.com/learn-golang-series/)