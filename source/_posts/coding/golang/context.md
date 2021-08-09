---
title: Golang并发调用接口-Context
date: 2021-08-07 20:19:00
tags: 
- Golang
category: Golang
---
Context这个词在编程中经常见到，一般被翻译成上下文，这个翻译既抽象又形象。在日常生活中，普通人说的上下文一般指的是文章的上下文，指的是我们在看文章的时候要结合上下文去分析某句话的意思。

举个例子，有一句话叫： 这个东西挺好吃，小明吃了一斤。光看这句话，你并不知道小明到底吃的是什么？毕竟这个可能是任何东西，你必须翻看文章前面的语句去看这个到底代表的是啥意思，这里上下文表示的就是一个前后的依赖关系。

在CPU里面，context也有类似的意思，我们常说CPU上下文切换，因为单核CPU一次只能执行一个运算，我们之所以可以同时听歌、写代码，那是因为有一种CPU时间片轮转机制在不同进程之间切换，由于其速度非常快，导致你觉得好像是在同时运行。在这个切换进程的过程中，CPU就需要保存上下文，这里的上下文其实就是寄存器、程序计数器等数据。

其实，虽然上下文的叫法理论上讲没太大毛病，但是其实翻译成语境、或者环境更加贴切。言归正传，下面咱就说说Go标准库里面的Context是啥、以及能干啥？

<!---more--->

## 1.手动终止协程
在Golang里面，Context往往是和协程紧密联系在一起的，因为Context是并发安全的，可以在不同协程之间同步状态。

举个例子： 主进程里面启动多个协程去处理数据，当我们关闭程序的时候，有一些协程还在处理中，这时候你该咋办？如果直接不管可能导致某些数据丢失，这时候你就可以通过Context去通知协程做一些终止前的准备工作。

比如下面这个例子里面，我们就启动了3个协程，5s之后通过context发送取消信号，子协程在收到信息之后依次退出。
```go
func main() {
    ctx, cancelFunc := context.WithCancel(context.Background())

    for _, i := range []int{1, 2, 3} {
        go doSomething(ctx, i)
    }

    // 5s之后通知所有协程关闭
    time.Sleep(time.Second * 5)
    cancelFunc()

    // 纯粹是留点时间给协程打印退出信息
    time.Sleep(time.Second * 5)
}

func doSomething(ctx context.Context, i int) {
    for {
        select {
        case <-ctx.Done(): //收到退出信号
            fmt.Printf("Prepare to shutdown: %d\n", i)
            return
        default:
            fmt.Printf("Do Something: %d\n", i)
            time.Sleep(time.Second)
        }
    }
}
```
此外，Context库还有一个```WithTimeout```，其实他们俩用法差不多，唯一的区别就是```WitchCancel```是需要你手动调用取消函数，而且Timeout这个顾名思义，是定时自动取消。

## 2.定时自动终止
以上面的例子为基础，我们只需要把main里面稍微改动一下，使用```WithTimeout```即可。这个cancelFunc是可以忽略的，但是最好用defer调用一下，否则可能导致内存泄露。
```go
func main() {
    ctx, cancelFunc := context.WithTimeout(context.Background(), time.Second * 5)
    defer cancelFunc()
    for _, i := range []int{1, 2, 3} {
        go doSomething(ctx, i)
    }
    time.Sleep(time.Second * 10)
}
```
除此之外，还有一个```WithDeadLine```，意思就是设置一个截止时间，到了这个时间就自动取消，不过在我们这个例子里面，基本上和Timeout等价，稍微改动即可：
```go
ctx, cancelFunc := context.WithDeadline(context.Background(), time.Now().Add(time.Second * 5))
```
实际还得看应用场景去选择使用那个函数。

下面，咱们来看一个比较贴合实际的应用场景： 假设我们需要并发的调用3个接口，而且还需要获取返回结果，并且设置一个超时时间，比如3s。

并发调3个接口并不是太大问题，我们只需要启动3个协程即可，获取响应结果的话，可以采用chan。在超时这块，实际上如果只是调接口的话，实际上Go的标准库里面httpClient可以设置一个超时时间。

所以有一个简单的写法：
```go
func main() {
    var (
        // 设置请求超时时间
        client = http.Client{Timeout: time.Second * 3}
        ch     = make(chan []byte)
    )
    go request(client, "https://www.baidu.com", ch)
    go request(client, "https://www.taobao.com", ch)
    go request(client, "https://www.google.com", ch)

    i := 0
    for {
        resp := <-ch
        fmt.Printf("%d\n", len(resp))

        // 接收3个之后关闭chan
        i++
        if i == 3 {
            close(ch)
            break
        }
    }
    // more code....
}

func request(client http.Client, apiUrl string, res chan []byte) {
    req, err := http.NewRequest("GET", apiUrl, nil)
    if err != nil {
        res <- nil
        return
    }
    resp, err := client.Do(req)
    if err != nil {
        res <- nil
        return
    }
    defer resp.Body.Close()
    all, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        res <- nil
        return
    }
    res <- all
}
```
这种依赖httpClient的写法大部分情况下没问题，但是假设子协程出问题了，没有往chan里面送东西，这个程序就会一直阻塞在那里。

对于这种情况，有2种解决方案：

### 一、接收端超时
接收端指的是上面代码里面接收结果的chan，我们可以设置一个超时时间，比如下面这个例子里面，我设置了3s，超过3s我就不等了，该干啥干啥去。
```go
func main() {
    var ch = make(chan []byte)
    go request("https://www.baidu.com", ch)
    go request("https://www.taobao.com", ch)
    go request("https://www.google.com", ch)

    i := 0
    timer := time.NewTimer(time.Second * 3)
L:
    for {
        select {
        case <-timer.C:
            break L
        case resp := <-ch:
            fmt.Printf("%d\n", len(resp))
            // 接收3个之后关闭chan
            i++
            if i == 3 {
                close(ch)
                break L
            }
        }
    }
    // more code...
}
```

### 二、执行端超时
这种方式意思就是我们要保证协程肯定能返回结果，即使超时也要返回一个nil，那这里就还得用到Context来做这件事。
在子协程里面，我们通过go启动一个协程，当收到取消信号的时候就返回一个nil，表示已经超时了。
```go
func main() {
    ctx, cancelFunc := context.WithTimeout(context.Background(), time.Second*3)
    defer cancelFunc()

    var ch = make(chan []byte)
    go request(ctx, "https://www.baidu.com", ch)
    go request(ctx, "https://www.taobao.com", ch)
    go request(ctx, "https://www.google.com", ch)

    i := 0
    for {
        resp := <-ch
        fmt.Printf("%d\n", len(resp))
        // 接收3个之后关闭chan
        i++
        if i == 3 {
            close(ch)
            break
        }
    }
    // more code...
}

func request(ctx context.Context, apiUrl string, res chan []byte) {
    go func() {
        select {
        case <-ctx.Done():
            fmt.Println("timeout...")
            res <- nil
        }
    }()
    resp, err := http.Get(apiUrl)
    if err != nil {
        res <- nil
        return
    }
    defer resp.Body.Close()
    all, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        res <- nil
        return
    }
    // 模拟超时
    time.Sleep(time.Second * 10)
    res <- all
}
```
执行结果如下：
```
timeout...
0
timeout...
0
timeout...
0
```

## 3.传递值
```WithValue```这个用的不多，效果和函数之间传参差不多，但是context的最大特点就是可以继承，当你的协程链条特别长的时候，这个就比较有意义了，比如说在不同子协程之间传递一些通用的配置变量。

纸上得来终觉浅，绝知此事要躬行，网上关于Context实现的原理的文章挺多，这里我就不献丑了，自己研究也不深，更多的研究其实际应用场景，加深自己的理解。希望上面的一些例子对你实际开发有用。