# golang

golang知识点

## Goroutine

不要以共享内存的方式来通信，相反，要通过通信来共享内存。

### Channels

当通道处于nil状态时，通道上尝试的任何发送或接收都将被阻止。 当通道处于打开状态时，可以发送和接收信号。 当通道处于关闭状态时，它将不再能够发送信号，但是仍然可以接收信号。

## sync
### sync.Atomic
sync/atomic 可以在并发场景下对变量进行非侵入式的操作。可以保证并发安全，虽然使用 sync.Mutex 可以实现，但是使用sync/atomic不仅是轻量级的，而且代码也更加简洁。

与Mutex相比，它的优势主要有以下几点：
* 更高效，因为atomic是直接作用与内存的锁，所以更底层，更高效。在Demo中的用时也可以看出。
* 更简洁，atomic避免了加锁解锁的过程，一行代码就可以完成这个操作，使代码更简洁，更具有可读性。

```go
package main

import (
	"fmt"
	"sync"
	"sync/atomic"
	"time"
)

func main() {
	test1()
	test2()
}

// count++ 并发不安全
func test1() {
	var wg sync.WaitGroup
	count := 0
	t := time.Now()
	for i := 0; i < 100000; i++ {
		wg.Add(1)
		go func(i int) {
			count++ // count不是并发安全的
			wg.Done()
		}(i)
	}
	wg.Wait()

	fmt.Printf("test1 花费时间：%d, count的值为：%d \n", time.Now().Sub(t), count)
}

// atomic.AddInt64(&count,1)  // 原子操作
func test2() {
	var wg sync.WaitGroup
	count := int64(0)
	t := time.Now()
	for i := 0; i < 100000; i++ {
		wg.Add(1)
		go func(i int) {
			atomic.AddInt64(&count, 1) // 原子操作
			wg.Done()
		}(i)
	}
	wg.Wait()

	fmt.Printf("test2 花费时间：%d, count的值为：%d \n", time.Now().Sub(t), count)
}
```

### sync.Once
sync.Once 是 Golang package 中使方法只执行一次的对象实现，作用与 init 函数类似。
* init 函数是在文件包首次被加载的时候执行，且只执行一次。
* sync.Once 是在代码运行中需要的时候执行，且只执行一次。
```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var once sync.Once
	onceBody := func() {
		fmt.Println("Only once")
	}

	done := make(chan bool)
	for i := 0; i < 10; i++ {
		go func() {
			once.Do(onceBody)
			done <- true
		}()
	}

	for i := 0; i < 10; i++ {
		<-done
	}
}
```
> 仅输出一次Only once

源码解析：
```go
package sync

import (
	"sync"
	"sync/atomic"
)

// Once is an object that will perform exactly one action.
type Once struct {
	m    sync.Mutex
	done uint32
}

func (o *Once) Do(f func()) {
	if atomic.LoadUint32(&o.done) == 1 {
		return
	}

	// Slow-path.
	o.m.Lock()
	defer o.m.Unlock()
	if o.done == 0 {
		defer atomic.StoreUint32(&o.done, 1)
		f()
	}
}
```

### sync.Pool
Pool 是可伸缩、并发安全的临时对象池，用来存放已经分配但暂时不用的临时对象，通过对象重用机制，缓解 GC 压力，提高程序性能。

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	p := &sync.Pool{
		New: func() interface{} {
			return 0
		},
	}

	a := p.Get().(int)
	p.Put(1)
	b := p.Get().(int)
	fmt.Println(a, b)
}
```

> 注意，sync.Pool 是一个临时的对象池，适用于储存一些会在 goroutine 间共享的临时对象，其中保存的任何项都可能随时不做通知地释放掉，所以不适合用于存放诸如 socket 长连接或数据库连接的对象。

### sync.WaitGroup
阻塞等待所有任务完成之后再继续执行。

```go
package main

import (
	"sync"
)

type httpPkg struct{}

func (httpPkg) Get(url string) {}

var http httpPkg

func main() {
	var wg sync.WaitGroup
	var urls = []string{
		"http://www.golang.org/",
		"http://www.google.com/",
		"http://www.somestupidname.com/",
	}
	for _, url := range urls {
		wg.Add(1)
		go func(url string) {
			defer wg.Done()
			http.Get(url)
		}(url)
	}
	wg.Wait()
}

```

### sync.Mutex
互斥锁。

#### map并发
map在并发执行中会直接报错，尽量不要做map的并发，如果用并发要加锁，保证map的操作要么读，要么写。

##### 一、加锁方式
```go
package main

import (
	"fmt"
	"sync"
	"time"
)

var lock sync.Mutex

func main() {
	m := make(map[int]int)

	go func() {
		for i := 0; i < 10000; i++ {
			lock.Lock()
			m[i] = i
			lock.Unlock()
		}
	}()
	go func() {
		for i := 0; i < 10000; i++ {
			lock.Lock()
			fmt.Println(m[i])
			lock.Unlock()
		}
	}()

	time.Sleep(time.Second * 20)
}
```

##### 二、channel方式
```go
package main

type SafeMap struct {
	Data map[int]string
	ch  chan func()
}

func NewSafeMap() *SafeMap {
	m := &SafeMap{
		Data: make(map[int]string),
		ch:   make(chan func()),
	}
	go func() {
		for {
			(<-m.ch)()
		}
	}()
	return m
}
func (m *SafeMap) add(num int, data string) {
	m.ch <- func() {
		m.Data[num] = data
	}
}
func (m *SafeMap) del(num int) {
	m.ch <- func() {
		delete(m.Data, num)
	}
}
func (m *SafeMap) find(num int) (data string) {
	m.ch <- func() {
		if res, ok := m.Data[num]; ok {
			data = res
		}
	}
	return
}
```

#### slice并发
slice在并发执行中不会报错，但是数据会丢失。

##### 一、加锁方式
```go
package main

import "sync"

var s []int
var lock sync.Mutex

func appendValue(i int) {
	lock.Lock()
	s = append(s, i)
	lock.Unlock()
}
```

##### 二、channel方式
```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	num := 10000

	var wg sync.WaitGroup
	wg.Add(num)

	c := make(chan int)
	for i := 0; i < num; i++ {
		go func() {
			c <- 1 // channl是协程安全的
			wg.Done()
		}()
	}

	// 等待关闭channel
	go func() {
		wg.Wait()
		close(c)
	}()

	// 读取数据
	var a []int
	for i := range c {
		a = append(a, i)
	}

	fmt.Println(len(a))
}
```

##### 三、索引方式
```go
package main

import "sync"

func main() {
	num := 10000
	a := make([]int, num, num)

	var wg sync.WaitGroup
	wg.Add(num)

	for i := 0; i < num; i++ {
		k := i // 必须使用局部变量
		go func(index int) {
			a[index] = 1
			wg.Done()
		}(k)
	}

	wg.Wait()

	count := 0
	for i := range a {
		if a[i] != 0 {
			count++
		}
	}
}
```


## 并发模型

IO多路复用、多进程、多线程

* 进程是资源分配的最小单位，线程是CPU调度的最小单位
* select, poll, epoll 都是I/O多路复用的具体的实现
* 协程是一种用户态的轻量级线程

> golang程序可以轻松支持10w级别的Goruntine，而线程数量达到1k时，内存占用就已经达到2G。

epoll 可以说是I/O 多路复用最新的一个实现，epoll 修复了poll 和select绝大部分问题, 比如：

* epoll 现在是线程安全的。
* epoll 现在不仅告诉你sock组里面数据，还会告诉你具体哪个sock有数据，你不用自己去找了。

## Go Channel 
无缓冲的通道（unbuffered channel）是指在接收前没有能力保存任何值的通道。
有缓冲的通道（buffered channel）是一种在被接收前能存储一个或者多个数据值的通道。
无缓冲的通道保证进行发送和接收的goroutine会在同一时间进行数据交换。

有关 channel 的关闭，你需要注意以下事项:

* 关闭一个未初始化(nil) 的 channel 会产生 panic。
* 重复关闭同一个 channel 会产生 panic。
* 向一个已关闭的 channel 中发送消息会产生 panic。
* 从已关闭的 channel 读取消息不会产生 panic，且能读出 channel 中还未被读取的消息，若消息均已读出，则会读到类型的零值。从一个已关闭的 channel 中读取消息永远不会阻塞，并且会返回一个为 false 的 ok-idiom，可以用它来判断 channel 是否关闭。
* 关闭 channel 会产生一个广播机制，所有向 channel 读取消息的 goroutine 都会收到消息。

### for range
range c产生的迭代值为Channel中发送的值，它会一直迭代直到channel被关闭。
```go
package main

import (
	"fmt"
)

func main() {
	ch := make(chan int)

	go func() {
		for i := 0; i < 5; i++ {
			ch <- i
		}
		close(ch)
	}()

	for num := range ch {
		fmt.Println("num = ", num)
	}
}
```

### select
同时有多个channel可以接收数据时，选择一个case处理。
* 如果没有case需要处理，则会选择default去处理，如果default case存在的情况下。
* 如果没有default case，则select语句会阻塞，直到某个case需要处理。
* nil channel上的操作会一直被阻塞，如果没有default case, 只有nil channel的select会一直被阻塞。
* select语句和switch语句一样，它不是循环，它只会选择一个case来处理，如果想一直处理channel，你可以在外面加一个无限的for循环。

### timeout
超时处理
```go
package main

import (
	"fmt"
	"time"
)

func main() {
	ch := make(chan string, 1)

	go func() {
		time.Sleep(time.Second * 2)
		ch <- "result 2"
	}()

	select {
	case res := <-ch:
		fmt.Println(res)
	case <-time.After(time.Second * 1):
		fmt.Println("timeout 1")
	}
}
```
