---
layout: post
title: "Golang - context"
date: 2019-04-10 21:30:20 +0800
comments: true
categories: golang
---

<!-- more -->

`context` 是控制並發的一個 package，之前在 [Worker Pool](https://mgleon08.github.io/blog/2018/05/17/golang-goroutine-channel-worker-pool-select-mutex/#worker_pool) 也有提到過另一個 `WaitGroup`，那為什麼需要兩種，來了解一下

# WaitGroup

當所有的 `goroutine` 都完成後，才算完成

### 實際場景

有個監控程式跑很多 `gorountine`，當要停止時，就必須通知每個 `gorountine` 並等待所有都完成，才算完成，否則會造成記憶體洩漏

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func process(i int, wg *sync.WaitGroup) {
	fmt.Println("監控開始 ", i)
	time.Sleep(2 * time.Second)
	fmt.Printf("監控 %d 結束\n", i)
	wg.Done() // 執行完一次就 -1
}

func main() {
	no := 2
	var wg sync.WaitGroup
	for i := 0; i < no; i++ {
		wg.Add(1)          // 每次執行都 + 1
		go process(i, &wg) // wg 一定要用 pointer，否則每個 goroutine 都會有各自的 WaitGroup
	}
	wg.Wait() // 會 wait 到 0 才會繼續下一步
	fmt.Println("所有監控完成")
}
```

大部分的 `gorountine` 啟動後，就會一直跑，大部分情況是等待它自己結束，如果是不會結束的 `gorountine`，就會一直跑下去，比較笨的方式就是，用一個變數去判斷是否要結束，但這樣就必須一直去檢查這個變數

因此可以改用 `chan + select` 來通知

# chan + select

這方式很好的解決上述的問題，並且可以透過給予 `chan` 值來決定是否要停止，但還是有其他問題

* 如果有很多 `goroutine` 都需要控制結束怎麼辦? 
* `goroutine` 又衍生了其它更多的goroutine怎麼?

即使定義很多 `chan` 也很難解決這個問題，因為 `goroutine` 的關係鏈就導致了這種場景非常複雜，因此就有另一個方式 `context`

```go
package main

import (
	"fmt"
	"time"
)

func process(stop chan bool) {
	for {
		select {
		case <-stop:
			fmt.Println("監控退出，停止了...")
			return
		default:
			fmt.Println("goroutine 監控中...")
			time.Sleep(2 * time.Second)
		}
	}
}

func main() {
	stop := make(chan bool)
	go process(stop)
	time.Sleep(5 * time.Second)
	stop <- true
	fmt.Println("所有監控完成")
}
```

# context

* [context](https://golang.org/pkg/context/)

上面提到的情境是多層級的 `goroutine`，因此要控制就必須跟蹤 `groutine`

因此 golang 提供了 `context` 來控制，也就是 `groutine` 的上下文

### Context 使用原則

* 不要把 Context 放在結構體中，要以參數的方式傳遞
* 以 Context 作為參數的函式方法，應該把 Context 作為第一個參數，放在第一位。
* 給一個函式方法傳遞 Context 的時候，不要傳遞 nil，如果不知道傳遞什麼，就使用 `context.TODO`
* Context 的 Value 相關方法應該傳遞必須的數據，不要什麼數據都使用這個傳遞
* Context 是執行緒安全的，可以放心的在多個 goroutine 中傳遞

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func process(ctx context.Context) {
	for {
		select {
		case <-ctx.Done():
			fmt.Println("監控退出，停止了...")
			return
		default:
			fmt.Println("goroutine監控中...")
			time.Sleep(2 * time.Second)
		}
	}
}

func main() {
	ctx, cancel := context.WithCancel(context.Background())
	
	go process(ctx)
	time.Sleep(5 * time.Second)
	fmt.Println("通知監控停止")
	cancel()
	time.Sleep(5 * time.Second)
	fmt.Println("所有監控完成")
}
```

* `context.Background()` 回傳一個非 nil 的空 context，不會被取消也沒有值或時間，就是 context 的根節點
* `context.WithCancel(parent)` 建立一個可取消的 `子 context` 當作參數來追蹤 `goroutine`，另外一個是 `cancle` 用於取消這個 `子 context`
* `ctx.Done` 判斷是否要結束，透過 `cancel()` 來取消


目前為止似乎跟 `chan` 沒什麼兩樣?接著試著控制多個 goroutine


```go
package main

import (
	"context"
	"fmt"
	"time"
)

func process(ctx context.Context, n int) {
	for {
		select {
		case <-ctx.Done():
			fmt.Println(n, "監控退出，停止了...")
			return
		default:
			fmt.Printf("goroutine %v 監控中...\n", n)
			time.Sleep(2 * time.Second)
		}
	}
}

func main() {
	ctx, cancel := context.WithCancel(context.Background())
	n := 3
	for i := 0; i < n; i++ {
		go process(ctx, i)
	}

	time.Sleep(5 * time.Second)
	fmt.Println("通知監控停止")
	cancel()
	time.Sleep(5 * time.Second)
	fmt.Println("所有監控完成")
}
```

可以看到，上面即使有多個 `goroutine` 依樣只要一個 `cancel()` 就全部都取消了

如果是用之前 `chan`，變成只有一個 `true` 只能夠取消一個，就結束了，導致剩下的 memory 沒辦法釋放，或是一次放三個 `true` 去控制，但是當一多起來，就會變得很複雜

# Context interface

```go
type Context interface {
	Deadline() (deadline time.Time, ok bool)

	Done() <-chan struct{}

	Err() error

	Value(key interface{}) interface{}
}
```

* `Deadline()` 是獲取設定的截止時間的意思
	* `deadline` 截止時間，到了這個時間點，Context 會自動發起取消請求
	* `ok` 如果是 false，表示沒有設定截止時間，如果需要取消的話，需要呼叫取消函式進行取消
* `Done` 返回一個 `只讀的 chan`，類型為 struct{}
	* 如果可讀取，表示 `parent context` 已發起取消的請求(`cancel`)，收到信號時就會開始清理操作，然後退出 goroutine，釋放資源
* `Err` 返回取消的錯誤原因，因為什麼 Context 被取消
* `Value` 獲取該 Context 上繫結的值，是一個鍵值對，所以要透過一個 Key 才可以獲取對應的值，這個值一般是執行緒安全的


# Other

* [WithCancel](https://golang.org/pkg/context/#example_WithCancel) 主動取消
* [WithDeadline](https://golang.org/pkg/context/#example_WithDeadline) 截止時間取消
* [WithTimeout](https://golang.org/pkg/context/#example_WithTimeout) 超時取消
* [WithValue](https://golang.org/pkg/context/#example_WithValue) 傳值
* 也可以不包上面的 With，直接使用 `context.Background()`，可以看到上面的 With 最後回傳的也是 `context Context`

Reference

* [context](https://golang.org/pkg/context/)
* [Go Context](https://zhuanlan.zhihu.com/p/58967892)
* [快速掌握 Golang context 包，簡單示例](https://deepzz.com/post/golang-context-package-notes.html)
* [Golang Context 是好的設計嗎？](https://segmentfault.com/a/1190000017394302)
* [Golang Context深入理解](https://juejin.im/post/5a6873fef265da3e317e55b6)
