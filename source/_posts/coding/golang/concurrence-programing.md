---
title: 浅谈Golang并发编程
date: 2021-12-30 21:00:00
tags: Golang
category: Golang
---
说到并发和并行，很多人容易混淆，虽然在大部分时候他们俩的意思都差不多，但实际上还是有很大区别。如果非得说明白，我觉得先从并行和串行这个2个单词说起更加容易理解，因为这2个单词很多人都熟悉，在编程里面所谓串行就是一个一个去执行，而并行则是指同时去执行。

举个例子：如果你需要调5个接口获取数据，每个接口耗时1s，那么串行总共需要5s来执行，但是并行顾名思义同时去执行，则可以同时去调用，那么总共只需要1s时间。所以相对于串行来说，并行可以节省很大时间，提高效率。

但是并发的概念和并行有所不同，并发更强调的是“同时”发生，它可以是并行的，也可以是“假”并行。

举个例子：早期CPU只有单核，单核意味着CPU同时一时间只能执行一段代码，那电脑是怎么做到在上网的同时还可以听歌呢？这是因为操作系统是支持并发的，它会把CPU执行时间分为很多小的时间片分给不同的应用程序，然后交替执行代码，所以看上去好像是在同时进行。如今，电脑CPU都是多核的，操作系统可以把一个或者多个核单独分配给一个应用去使用，对于这种情况我们可以认为他是并行。

从理论上说并行只是并发的一种实现方式，并发是更高维度的概念，说白了，并发只要你能实现“同时”执行的效果，至于物理层面是否是同时并不重要。所以我们网上看到大部分文章都会说并发编程，很少有人会说并行编程。

其实在如今，很多语言依然不支持并发编程，比如PHP；还有很多语言通过多进程、多线程的方式来实现，比如C++、Java；而Go则是通过协程的方式实现了并发编程。
<!--more-->

## 1.示例
咱们先看一个串行的例子，一个简单的函数调用，这里用sleep模拟操作耗时1s，然后打印当前时间，可以看到执行完这3次操作必须耗时3s
```go
func main() {
    hello()
    hello()
    hello()
}

func hello() {
    time.Sleep(time.Second)
    fmt.Printf("Hello World: %v\n", time.Now().Unix())
}
/** 运行结果
Hello World: 1640766013
Hello World: 1640766014
Hello World: 1640766015
 */
```
下面再看Go协程并发的版本：
```go
func main() {
    go hello()
    go hello()
    go hello()
    time.Sleep(time.Second * 2)
}

func hello() {
    time.Sleep(time.Second)
    fmt.Printf("Hello World: %v\n", time.Now().Unix())
}
/** 运行结果
Hello World: 1640766082
Hello World: 1640766082
Hello World: 1640766082
 */
```
从打印结果来看，这3次调用几乎是同时执行的，虽然严格意义上说并不是同时开始的，因为协程的启动也有开销的，需要一点时间的，不过也是毫秒级别，从秒的维度看是同时。

## 2.等待协程结束（同步）
可能你看过很大文章说在Go里面启用协程只需要使用go关键字就行，但是实际使用中你还要解决几个问题，其中第一个就是如何等待协程执行完成。

当你使用go启动一个协程之后，它是异步非阻塞的，它会在”后台“默默执行协程里面的代码，同时主进程会继续往下执行，在上面这个例子里面，main里面没有其它代码了，只有一个sleep，之所以要加这个sleep就是为了等待那3个协程执行完，如果去掉这个sleep你会发现没有任何结果输出，因为协程还没来得及运行主进程就结束了。

所以当你在使用协程的时候需要确保所有协程都有足够的时间去执行，当所有协程都结束后才退出。当然如果应用本身就是常驻进程的话，比如HTTP服务，就不需要关心这个问题。

上面这个例子的写法只是为了方便测试，在实际上开发中，肯定不能用sleep，你不可能准确预测到协程执行需要多久时间，官方为了解决这个问题专门弄了包，在sync包里面有一个WaitGroup就是拿来同步不同协程的操作。
```go
// A WaitGroup waits for a collection of goroutines to finish.
// The main goroutine calls Add to set the number of
// goroutines to wait for. Then each of the goroutines
// runs and calls Done when finished. At the same time,
// Wait can be used to block until all goroutines have finished.
//
// A WaitGroup must not be copied after first use.
type WaitGroup struct {
    noCopy noCopy
    state1 [3]uint32
}
```
使用方法非常简单，它只有3个方法，Add、Done、Wait，最开始那个例子如果使用WaitGroup改造的话可以这么写：
```go
func main() {
    wg := new(sync.WaitGroup)
    wg.Add(3)

    go hello(wg)
    go hello(wg)
    go hello(wg)

    wg.Wait()
}

func hello(wg *sync.WaitGroup) {
    defer wg.Done()
    fmt.Printf("Hello World: %v\n", time.Now().Unix())
}
```
其中Add(3)表示有3个协程需要执行，同时我们以指针的形式把wg传入函数内部，在函数结束之前调用Done，最后在主进程里面我们调用Wait，这个函数是阻塞的，它会等待所有协程结束。

这个库源码的话比较简单，简单说就是一个原子计数操作，Add就是+1，Done就是-1，然后开启一个for循环阻塞，在循环里面等待值变为0就退出。

除此之外，针对单个协程的话也可以利用Go管道的特性来实现类似功能，但是针对多个协程之间的同步还是WaitGroup更好好用。
```go
func main() {
    ch := make(chan int64)
    go hello(ch)
    <-ch
}

func hello(ch chan int64) {
    defer func() {
        ch <- 1
    }()
    fmt.Printf("Hello World: %v\n", time.Now().Unix())
}
```

## 3.协程通信
在Go里面channel是协程之间通信的桥梁，在Go编程里面有一句非常知名的话：”不要使用共享内存去通信，而应该通过通信来共享内存“，这里面的通信指的就是channel这种方式。

即使你使用多进程或者多线程这种并发编程模型，都存在在不同进程或者线程之间通信的需求，我们很容易就想到可以使用全局变量+锁这种方式，这种方式本质上就是共享内存，Go的编程哲学并不推荐，虽然Go也能这么写。

以上面的例子为基础，我们现在假设现在hello这个函数不再打印一个结果，我们需要“返回”当前时间戳作为结果，如果你直接return，主进程是没法接收返回值的。

如果你使用共享内存这种方式，可以先定义一个全局的slice来存储结果，同时需要一把全局锁来解决并发数据竞争问题，比如下面这个例子：
```go
var (
    ts     []int64
    locker = sync.RWMutex{}
)

func main() {
    wg := new(sync.WaitGroup)
    wg.Add(3)

    go hello(wg)
    go hello(wg)
    go hello(wg)

    wg.Wait()

    fmt.Printf("%v\n", ts)
}

func hello(wg *sync.WaitGroup) {
    defer wg.Done()
    defer locker.Unlock()
    locker.Lock()
    ts = append(ts, time.Now().Unix())
}
/** 运行结果
[1640854306 1640854306 1640854306]
 */
```
虽然Go官方并不推荐这种写法，但是这么写是完全没有问题的，但如果使用channel的话更好，下面我们再看看使用channel的写法：
```go
func main() {
    ch := make(chan int64)

    go hello(ch)
    go hello(ch)
    go hello(ch)

    for {
        fmt.Printf("%v\n", <-ch)
    }
}

func hello(ch chan int64) {
    ch <- time.Now().Unix()
}
```
我们先创建一个int64类似的channel，然后作为参数传入函数内部，在函数里面把结果放到管道里面，同时在主进程里面使用for...range获取channel里面的数据，这里大家可以注意到可以不用WaitGroup了，因为从channel里面获取数据也是阻塞的。

但是运行的话你会发现不仅打印出了3个正确结果，还报错了: ```fatal error: all goroutines are asleep - deadlock!```

这个死锁问题在使用channel的时候需要注意的问题，详细说起来还比较复杂，简单说就是如果你要从一个channel里面取数据，那么必须有一个地方会往里面放数据，在这个例子里面，Go运行时检测到了你取完3个数据之后，没有其它地方会往channel里面放数据了，这样for循环会一直等下去。。。

想要解决这个问题，就必须在合理的时候关闭channel，在上面这个例子，我们可以明确知道只有3个结果，所以我们可以在for循环里面加一个判断，主动close channel，结束for循环，继续执行后面的代码逻辑。
```go
i := 0
for {
    fmt.Printf("%v\n", <-ch)
    i++
    if i == 3 {
        close(ch)
        break
    }
}
```
## 4.协程调度
Go协程调度相关的东西非常多，这里只是抛砖引玉说一点，话说在计算机系统里面，但凡是涉及到资源（CPU、GPU、内存、存储）的使用都需要调度，所谓调度无非是如何去分配资源，因为资源是有限的，但是使用方却很多，给谁用，用多少是一个问题。

举个例子，电脑CPU只有8核，但是上面跑的进程可能有几十上百个，那操作系统是如何分配CPU资源的呢？是按照先后顺序还是平均分配？ 这就涉及到操作系统调度的问题了，说起来也非常复杂，一个成熟的调度算法必须兼顾公平和效率。

回到Go这边，Go的协程也被称为是用户态线程，在多线程并发模型里面，进程和线程是直接参与CPU调度的，是由操作系统来完成调度这件事，但是Go的协程并不直接参与系统调度，那问题来了，如果我开了100个协程，但是我的CPU只有8核，Go是怎么做到“同时”运行这些协程的呢？

<img src="/images/2021/gmp.jpeg" />

这个问题的答案就是Go协程的调度所用到的GMP模型，这个调度模型也是经过多次版本迭代后的结果，网上也很多深度解析的文章介绍，感兴趣的可以找找看一看，这里只借用一张图。

## 5.控制协程数量
虽然Go协程的开销很小，但是如果不加以控制可能会产生一些意想不到的结果，举个例子，你需要调用一个接口，使用1000个不同的参数，每次调用接口需要1s时间，聪明的你肯定想到如果串行的调用则需要1000s时间，太慢了，但是使用Go协程开启1000个协程很快就搞定了。
```go
func main() {
    wg := new(sync.WaitGroup)
    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go hello(wg)
    }
    wg.Wait()
}

func hello(wg *sync.WaitGroup) {
    defer wg.Done()
    resp, err := http.Get("https://www.baidu.com")
    if err != nil {
        return
    }
    fmt.Printf("resp status: %s\n", resp.Status)
}
```
思路没问题，但是你有没有想过这个接口的能力能不能支持这么大并发量，如果不支持，轻则导致你调用报错、重则导致对方服务挂掉。所以从安全可用的角度来说，我们需要控制一下并发数量，比如说100个。

只要稍加改造，可以利用Go管道的阻塞特性实现这一功能，控制协程的并发数量：
```go
func main() {
    wg := new(sync.WaitGroup)
    ch := make(chan bool, 100)

    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go hello(wg, ch)
    }
    wg.Wait()
}

func hello(wg *sync.WaitGroup, ch chan bool) {
    defer wg.Done()
    defer func(ch chan bool) {
        <-ch
    }(ch)
    
    ch <- true
    resp, err := http.Get("https://www.baidu.com")
    if err != nil {
        return
    }
    fmt.Printf("resp status: %s\n", resp.Status)
}
```
这段代码的原理就是有缓冲管道是可以缓存一定数据的，我们先创建一个缓存大小为100的channel，把它作为参数传递到函数内部，在执行函数逻辑之前往channel里面放一个数据，在执行完逻辑之后取出一个数据。因为缓存大小是100，所以最多可以有100个协程同时执行。

不过这种方案的缺点就是在一开始就已经启用了1000个协程，只不过这些协程卡在等待执行逻辑的过程中，如果协程数据量超级大的话可能有一些内存开销。

## 6.ErrGroup
这是一个Go官方提供的一个同步扩展库，它可以将一个大任务拆分成几个小任务并发执行，提高程序效率。其代码非常简洁，主要有三个方法，WithContext、Go、Wait。
```go
func main() {
    var g errgroup.Group

    // 启动第一个子任务,它执行成功
    g.Go(func() error {
        time.Sleep(5 * time.Second)
        fmt.Println("exec #1")
        return nil
    })
    // 启动第二个子任务，它执行失败
    g.Go(func() error {
        time.Sleep(10 * time.Second)
        fmt.Println("exec #2")
        return errors.New("failed to exec #2")
    })

    // 启动第三个子任务，它执行成功
    g.Go(func() error {
        time.Sleep(15 * time.Second)
        fmt.Println("exec #3")
        return nil
    })
    // 等待三个任务都完成
    if err := g.Wait(); err == nil {
        fmt.Println("Successfully exec all")
    } else {
        fmt.Println("failed:", err)
    }
}
```
本质上这是一个对WaitGroup的封装，使用起来更加方便，它不仅可以等待所有协程执行完成，还可以返回第一个错误，同时还可以实现当其中某一个协程出错，取消其它协程的功能。

其源码非常简单，总共只有几十行代码，包含非常丰富的注释，贴上来大家一起欣赏一下：
```go
// Copyright 2016 The Go Authors. All rights reserved.
// Use of this source code is governed by a BSD-style
// license that can be found in the LICENSE file.

// Package errgroup provides synchronization, error propagation, and Context
// cancelation for groups of goroutines working on subtasks of a common task.
package errgroup

import (
    "context"
    "sync"
)

// A Group is a collection of goroutines working on subtasks that are part of
// the same overall task.
//
// A zero Group is valid and does not cancel on error.
type Group struct {
    cancel func()

    wg sync.WaitGroup

    errOnce sync.Once
    err     error
}

// WithContext returns a new Group and an associated Context derived from ctx.
//
// The derived Context is canceled the first time a function passed to Go
// returns a non-nil error or the first time Wait returns, whichever occurs
// first.
func WithContext(ctx context.Context) (*Group, context.Context) {
    ctx, cancel := context.WithCancel(ctx)
    return &Group{cancel: cancel}, ctx
}

// Wait blocks until all function calls from the Go method have returned, then
// returns the first non-nil error (if any) from them.
func (g *Group) Wait() error {
    g.wg.Wait()
    if g.cancel != nil {
        g.cancel()
    }
    return g.err
}

// Go calls the given function in a new goroutine.
//
// The first call to return a non-nil error cancels the group; its error will be
// returned by Wait.
func (g *Group) Go(f func() error) {
    g.wg.Add(1)

    go func() {
        defer g.wg.Done()

        if err := f(); err != nil {
            g.errOnce.Do(func() {
                g.err = err
                if g.cancel != nil {
                    g.cancel()
                }
            })
        }
    }()
}
```