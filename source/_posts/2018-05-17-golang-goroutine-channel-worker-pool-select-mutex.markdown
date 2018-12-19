---
layout: post
title: "Golang - goroutine, channel, worker pool, select, mutex"
date: 2018-05-17 23:10:28 +0800
comments: true
categories: golang
---

<!-- more -->

* [Concurrency &  Parallelism](#concurrency_parallelism)
* [Goroutine](#goroutine)
* [Channel](#channel)
	* [Unidirectional channels (單向 channels)](#unidirectional_channels)
	* [Closing channels and for range loops on channels](#closing_channels)
* [Buffered Channels](#buffered_channels)
* [Worker Pool](#worker_pool)
* [Select](#select)
* [sync.Mutex](#mutex)

# <span id="concurrency_parallelism"> Concurrency &  Parallelism </span>

> Goroutines are part of making concurrency easy to use.

要了解 Goroutine 可能要先了解一下，Concurrency 跟 parallelism 的差別

### Concurrency

> Concurrency is the capability to deal with lots of things at once.

簡單的解釋，一個在跑步的人，因為鞋帶鬆了，所以他停下來綁鞋帶，綁完後繼續跑。

![](http://i.imgur.com/WOthNHb.png)

concurrency 只能在單一 CPU 核裡執行

### Parallelism

> Parallelism is doing lots of things at the same time. It might sound similar to concurrency but its actually different.

同樣用慢跑來解釋的話，在慢跑的人，同時在用耳機聽音樂，在同一時間做了很多事。

![](http://i.imgur.com/cTonbBE.png)

Parallelism 可以同時多核處理

### 總結

> 用 CPU 來解釋
> 
> If the person is doing running on 1 core and tying his laces on another core its Parallelism. If he is running on on 1 core and then switching/stopping to tie his laces on the same core then its concurrent

但這不代表 Parallelism 速度會比較快，因為相對於 Concurrency，可能要花更多時間去做溝通

像是要在執行完下載，要跳出訊息顯示成功，用 concurrency 就非常單純，但用 Parallelism 就必須判斷什麼時間點要去通知

* [concurrency & parallelism](https://golangbot.com/concurrency/)

# <span id="goroutine"> Goroutine </span>


> Goroutines are functions or methods that run concurrently with other functions or methods.

* go 本身就有數千個 goroutine 在跑
* goroutine 可以想像成是輕量級的 threads.
* 與 threads 相比，goroutine 成本非常小，通常只有幾 kb，並且不像 threads 會固定 size，而是會根據狀況成長和收縮
* 一個 threads 可能就會有幾千個 goroutine，因此開 OS thread 的量會比較少
* 當 thread 被阻塞時，可以開新 thread 並將剩餘的 goroutine 轉移過去
* goroutine 最多運行 GOMAXPROCS 數量(可以設定)
* main() 也是一個 goroutine 稱為 `main Goroutine`
* Goroutine 可以透過 channel 來進行溝通，防止同時訪問共享的資源，造成競爭

### syntax

```go
// keyword go
go hello()
```

### start a Goroutine

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
    time.Sleep(1 * time.Second)
    fmt.Println("main function")
}

// Hello world goroutine
// main function
// 因為 hello() 進入 goroutine，但是 main() 已經結束，因此所有的 goroutine 都會直接打斷，程序退出。
// 加入 sleep 1 秒，讓 hello() 有足夠的時間 retuen 回來，1秒後已經先 return 回來再顯示 "main function"
```

### Starting multiple Goroutines

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
* Channel 中使用的 type 稱之為 element type，比如 int 類型的 channel 寫作為 `chan int`，無法允許不同 type data 傳輸
* The zero value of a channel is `nil`，這種 nil chaneel 無法使用，必須向 `maps` 和 `slices` 一樣，使用 'make'
* 若沒填寫第二個參數，預設就為 0，則為 unbuffered channel，意思是 sender 和 receiver 必須是同步的才能發送

> * [make](https://golang.org/pkg/builtin/#make)
> * Slice: The size specifies the length. The capacity of the slice is
equal to its length. A second integer argument may be provided to
specify a different capacity; it must be no smaller than the
length. For example, make([]int, 0, 10) allocates an underlying array
of size 10 and returns a slice of length 0 and capacity 10 that is
backed by this underlying array.

>* Map: An empty map is allocated with enough space to hold the
specified number of elements. The size may be omitted, in which case
a small starting size is allocated.

>* Channel: The channel's buffer is initialized with the specified
buffer capacity. If zero, or the size is omitted, the channel is
unbuffered.


```go
var a chan int
a := make(chan int)  
```

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

##### 將上面範例 sleep 改用 channel 改寫

`<-done` 這行會導致 `main goroutine` blocked 在這邊，直到其他 goroutine 將 data 寫入 done，不然是不會繼續往下走，也意味著就不需要用 sleep 來停止

```go
package main

import (
    "fmt"
    "time"
)
// 接收 bool 的 cahnnel
func hello(done chan bool) {
    fmt.Println("hello go routine is going to sleep")
    time.Sleep(4 * time.Second)
    fmt.Println("hello go routine awake and going to write to done")
    done <- true
}
func main() {
	//用 make 建立一個不為 nil 的 channel
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

* `<-done` 會等到 channel data 回來才會繼續執行下一行
* `<-done` 左邊並沒有任何 variable 去接收，因為這邊只是會了要讓他先執行 `hello()` 並不是要回傳的 value

##### 另一個範例

```go
/*
squares = (1 * 1) + (2 * 2) + (3 * 3) 
cubes = (1 * 1 * 1) + (2 * 2 * 2) + (3 * 3 * 3) 
output = squares + cubes = 50
*/
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
    // 最後將 sum 傳送到 squareop channel
    squareop <- sum
}

func calcCubes(number int, cubeop chan int) {
    sum := 0
    for number != 0 {
        digit := number % 10
        sum += digit * digit * digit
        number /= 10
    }
    // 最後將 sum 傳送到 cubeop channel
    cubeop <- sum
}

func main() {
    number := 589
    sqrch := make(chan int)
    cubech := make(chan int)
    go calcSquares(number, sqrch)
    go calcCubes(number, cubech)
    // 會等到 sqrch & cubech data 回來才會繼續執行
    squares, cubes := <-sqrch, <-cubech
    fmt.Println("Final output", squares + cubes)
}

// Final output 1536
```

### refactor with close & range

```go
package main

import (
    "fmt"
)

// 將數字分成個別數字 123 -> 1, 2, 3 丟到 channel
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
    // range 會自動將 channel 的值，一個一個取出
    for digit := range dch {
        sum += digit * digit
    }
    // 將總數 send 到 main 裡面的 channel
    squareop <- sum
}

func calcCubes(number int, cubeop chan int) {
    sum := 0
    dch := make(chan int)
    go digits(number, dch)
    // range 會自動將 channel 的值，一個一個取出
    for digit := range dch {
        sum += digit * digit * digit
    }
    // 將總數 send 到 main 裡面的 channel
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

當 Goroutine send data 到 channel，但沒有其他的 Goroutine 去接收這個 data，就會造成 Deadlock，並且出現錯誤 panic `fatal error: all goroutines are asleep - deadlock!`

```go
package main

func main() {
    ch := make(chan int)
    ch <- 5
}
```

### <span id="unidirectional_channels"> Unidirectional channels (單向 channels) </span>


到目前為止，說的都是 bidirectional channels(雙向 channels)

channel 也可以是單向的，`only send or receive data`

```go
ch := make(chan int) // 雙向 channel
sendch := make(chan<- int) // 單向 channel
var send chan<- int //只能發送 data 到 channel
var receive <-chan int //只能接收 chaneel 裡的 data
```

```go
package main

import "fmt"

// send only channel
func sendData(sendch chan<- int) {
    sendch <- 10
}

func main() {
    sendch := make(chan<- int) // 單向 channel
    go sendData(sendch)
    fmt.Println(<-sendch) // error 因為只有單向進去，沒有出來
    // invalid operation: <-sendch (receive from send-only type chan<- int)
}

// main.go:11: invalid operation: <-sendch (receive from send-only type chan<- int)
```
以上只有 send 功能，並沒有 receive，因此會報錯，但事實上只有 send 的 channel 也是沒有什麼意義

```go
package main

import "fmt"

// send only channel
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

### <span id="closing_channels"> Closing channels and for range loops on channels </span>

* sender 可以關閉 channel 已告知 receivers，已經沒有 dat
* 只有 sender 要關閉 channel，如果沒有 close channel 可能會導致 panic `fatal error: all goroutines are asleep - deadlock!`
* Channels aren't like files; you don't usually need to close them. Closing is only necessary when the receiver must be told there are no more values coming, such as to terminate a `range` loop.

```go
v, ok := <- ch
// ok
// true: 可以接收的狀態
// false: 沒有任何 value & channel 已經關閉

close(ch) // 關閉 channel，用 range 取出 channel 東西時，必須用 close() 告知已經沒東西了，否則會 deadlock
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
	close(chnl) // range 就不需要這行去關閉
}
func main() {
	ch := make(chan int)
	go producer(ch)
	for {
		v, ok := <-ch // 當沒有 close(chnl) 時，這邊就會產生 decklock，因為裡面並沒有 data 了
		if ok == false {
			break
		}
		fmt.Println("Received ", v, ok)
	}
	/*
	用 range，當 channel close 會自動離開
	for v := range ch {
		fmt.Println("Received ", v)
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
// The capacity for an unbuffered channel is 0，代表必須同步
```

* 當 buffer 滿了，sends 就會 block 住
* 當 buffer 空了，receives 就會 block 住


### Example

```go
package main

import "fmt"

func main() {
	ch := make(chan int, 2)
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
capacity is 2
length is 2
1
2
capacity is 2
length is 0
*/
```


### Example2

```go
package main

import (
    "fmt"
    "time"
)

func write(ch chan int) {  
    for i := 0; i < 5; i++ {
        ch <- i
        fmt.Println("successfully wrote", i, "to ch")
    }
    close(ch)
}
func main() {  
    ch := make(chan int, 2)
    go write(ch)
    time.Sleep(2 * time.Second)
    for v := range ch {
        fmt.Println("read value", v,"from ch")
        // 另外如果沒有特別設定，就會一瞬間完成
        time.Sleep(2 * time.Second)

    }
}

/*
successfully wrote 0 to ch  
successfully wrote 1 to ch  
read value 0 from ch  
successfully wrote 2 to ch  
read value 1 from ch  
successfully wrote 3 to ch  
read value 2 from ch  
successfully wrote 4 to ch  
read value 3 from ch  
read value 4 from ch  
*/
```

* ch buffered channel 容量設定 2，當丟給 `write` 跑 for loop，將 0, 1, 2, 3 丟到 ch 裡面，但因為容量只有 2，所以丟到 1 之後，ch 就會被 block 住，直到 ch 的東西被 reader 取出來，因此一開始顯示兩行就停住

```go
successfully wrote 0 to ch
successfully wrote 1 to ch
```

* 因為 reader 在 sleep 後面，所以等兩秒後，`main()` range 就會開始取出，取出一個就會停兩秒，因此同時間，`write()` 發現 ch 又有容量，就會繼續塞，因此又滿了

```go
read value 0 from ch
successfully wrote 2 to ch
```

* 接著就會一直重複

```go
read value 1 from ch  
successfully wrote 3 to ch  
read value 2 from ch  
successfully wrote 4 to ch  
read value 3 from ch  
read value 4 from ch  
```

# <span id="worker_pool"> Worker Pool </span>

### WaitGroup

透過 WaitGroup，可以讓所有的 Goroutine 都跑完，最後再結束

> * WaitGroup is a struct type and we are creating a zero value variable  
> * The way WaitGroup works is by using a counter

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

/*
output 有可能都不一樣，因為每個 Goroutines 執行的順序不同
started Goroutine  2  
started Goroutine  0  
started Goroutine  1  
Goroutine 0 ended  
Goroutine 2 ended  
Goroutine 1 ended  
All go routines finished executing  
*/
```

what is [worker pool](https://en.wikipedia.org/wiki/Thread_pool)?

> a worker pool is a collection of threads which are waiting for tasks to be assigned to them. Once they finish the task assigned, they make themselves available again for the next task.

* [WaitGroup](https://golang.org/pkg/sync/#WaitGroup)

### Example

Core functionalities of our worker pool

* Creation of a pool of Goroutines which listen on an input buffered channel waiting for jobs to be assigned
* Addition of jobs to the input buffered channel
* Writing results to an output buffered channel after job completion
* Read and print results from the output buffered channel

```go
package main

import (  
    "fmt"
    "math/rand"
    "sync"
    "time"
)

type Job struct {  
    id       int
    randomno int
}

type Result struct {  
    job         Job
    sumofdigits int
}

// buffered channel 最多 10 個，沒有取出會 block 住
var jobs = make(chan Job, 10)  
var results = make(chan Result, 10)

// 用來運算 123 = 1 + 2 + 3 = 6
func digits(number int) int {  
    sum := 0
    no := number
    for no != 0 {
        digit := no % 10
        sum += digit
        no /= 10
    }
    time.Sleep(2 * time.Second)
    return sum
}

func worker(wg *sync.WaitGroup) {  
// 從 jobs buffered channel 一個一個取出
    for job := range jobs {
        output := Result{job, digits(job.randomno)}
        // 將 output 的結果，丟到 results buffered channel
        results <- output
    }
    // 每完成一個就 done
    wg.Done()
}

func createWorkerPool(noOfWorkers int) {  
    var wg sync.WaitGroup
    for i := 0; i < noOfWorkers; i++ {
    	//每執行一個 worker 就加1
        wg.Add(1)
        // 每個 Goroutines 執行的時間都不一樣
        go worker(&wg)
    }
    // 等待到全部 wotker 都執行完畢
    wg.Wait()
    // 告知關閉 results channel，用 range 取出 channel 的值必須關閉
    close(results)
}

// 主要是將要執行的 job 存放到 channel 裡面
func allocate(noOfJobs int) {  
    for i := 0; i < noOfJobs; i++ {
    	//產生最高為 998 的亂數
        randomno := rand.Intn(999)
        job := Job{i, randomno}
        // 將 job 丟到 buffered channel
        jobs <- job
    }
    // 告知關閉 jobs channel，用 range 取出 channel 的值必須關閉
    close(jobs)
}

func result(done chan bool) {  
    for result := range results {
        fmt.Printf("Job id %d, input random no %d , sum of digits %d\n", result.job.id, result.job.randomno, result.sumofdigits)
    }
    done <- true
}

func main() {  
    startTime := time.Now()
    noOfJobs := 100
    // 丟 100 個 job 進去
    go allocate(noOfJobs)
    done := make(chan bool)
    go result(done)
    // 同時有 10 個 worker 在跑
    noOfWorkers := 10
    createWorkerPool(noOfWorkers)
    <-done
    endTime := time.Now()
    diff := endTime.Sub(startTime)
    fmt.Println("total time taken ", diff.Seconds(), "seconds")
}
```

* [Worker Pool Implementation](https://golangbot.com/buffered-channels-worker-pools/)

# <span id="select"> Select </span>

* The select statement lets a goroutine wait on multiple communication operations.
* A select blocks until one of its cases can run, then it executes that case. It chooses one at random if multiple are ready.
* The syntax is similar to switch except that each of the case statement will be a channel operation
* 當有個任務需要即時的 output，並且有兩台 server 有同樣的 api 可以呼叫，就可以用 `select` 選擇最快 response 那台來執行

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

	// 等待到其中一個 channel 回來，就執行，如果都有就會隨機，執行完後，接續 terminal 就會結束，因此另一個就不會有 output 了
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

* The default case in a select is run if no other case is ready.
* 加上 default 就不會有 deadlock 情形發生

```go
// Use a default case to try a send or receive without blocking:

select {
case i := <-c:
    // use i
default:
    // receiving from c would block
}
```

不會 deadlock

```go
package main

import "fmt"

func main() {  
    ch := make(chan string)
    // var ch chan string nil 也不會有問題
    select {
    case <-ch:
    default:
        fmt.Println("default case executed")
    }
}
```

##### Example 1

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

##### Example 2

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

Before jumping to mutex, it is important to understand the concept of [Critical section](https://en.wikipedia.org/wiki/Critical_section) in concurrent programming

一個存取共用資源（例如：共用裝置或是共用記憶體）的程式片段，而這些共用資源有無法同時被多個執行緒存取的特性。

舉例來說，當兩個以上的 goroutine 對同一個 value 做計算時，可能會因為前後順序的差異，導致最後的結果不同。(e.g. x = x + 1)

```go
// 情境1
1. goroutine1 取得 x = 0，並加上 1
2. goroutine2 取的 x = 0，並加上 1
3. goroutine1 將結果存回 x
4. goroutine2 將結果存回 x
5. 結果 x = 1

// 情境2
1. goroutine1 取得 x = 0，並加上 1
2. goroutine1 將結果存回 x
3. goroutine2 取的 x = 1，並加上 1，在存回 x
4. goroutine2 將結果存回 x
4. 結果 x = 2

```

### [Mutex](https://tip.golang.org/pkg/sync/#Mutex)

A Mutex is used to provide a locking mechanism to ensure that only one Goroutine is running the critical section of code at any point of time to prevent race condition from happening.


```go
// Mutex is available in the sync package
mutex.Lock()
x = x + 1
mutex.Unlock()

// 同時間只會有一個 goroutine
```

### Solving the race condition using mutex

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

// 必須在 local 跑
// 1000
// 如果沒加上 lock，同時就會有很多個 goroutine 在跑，導致最後結果不一樣
```

### Solving the race condition using buffered channel

```go
package main

import (
	"fmt"
	"sync"
)

var x = 0

func increment(wg *sync.WaitGroup, ch chan bool) {
	ch <- true // 當前面的 channel 還有東西，就會 block 住，導致後面無法繼續
	x = x + 1 // 因此同時只會有一個 goroutine 執行這行
	<-ch // 等這邊釋放出來後，才能夠繼續塞
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
