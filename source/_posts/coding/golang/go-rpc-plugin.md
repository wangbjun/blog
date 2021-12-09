---
title: 基于RPC实现的Go插件机制
date: 2021-12-08 20:10:00
tags: Golang
category: Golang
---
最近项目里面有需要实现插件机制，所谓插件就是指可以在不用发布新的版本情况下实现对程序功能的扩展，这种插件机制应用特别广泛，常见的比如咱们浏览器的扩展、Nginx扩展、PHP扩展等等。

在Go语言里面，官方自带了一个插件扩展机制，这个我之前有文章介绍过，详情可以点击 [Go Plugin实现插件化编程](https://wangbjun.site/2021/coding/golang/go-plugin.html)

但是Go官方自带这个插件机制很不成熟，更像是一个半成品，存在很多问题，比如插件无法卸载、各自版本依赖必须完全一致，其中对版本依赖必须一致这个要求非常致命，因为在一个大型的项目里面，很难限制插件开发者使用的依赖库版本，很难做到统一。

今天咱们就介绍一个基于RPC实现的插件机制，而且这种方式是经过大规模实践验证的，可用性非常高，值得尝试一下。

<!--more-->

## 1.简介
这个是hashicorp公司开源的项目，Github地址： [https://github.com/hashicorp/go-plugin](https://github.com/hashicorp/go-plugin)

官方的介绍如下：
>go-plugin 是一个基于 RPC 的 Go (golang) 插件系统。 它是 HashiCorp 工具使用了 4 年多的插件系统。 虽然最初是为 Packer 创建的，但它也被 Terraform、Nomad、Vault 和 Boundary 使用。 虽然插件系统基于 RPC，但它目前仅设计用于在本地 [可靠] 网络上工作。 不支持真实网络上的插件，并且会导致意外行为。 该插件系统已在许多不同项目的数百万台机器上使用，并已证明经得起考验，可用于生产。

介绍中提到在多个项目使用过，但是这些我都不熟悉，不过有一个开源的项目很多人都应该听过，名字叫[Grafana](https://grafana.com/)

Grafana是一个开源的监控可视化平台，刚好就是hashicorp公司的项目，目前也提供商业化服务，其后端就是Go写的，也算是Go语言里面非常热门的开源项目，有几十k的star，源码之前看过写的也很不错，其中就用到了上面所说的插件库，其中Grafana面板里面的数据源就是插件的形式，用户可以根据自己需求下载安装对应数据库的插件。

不过在Grafana这个项目里面，这个插件库被封装了很多层，代码非常多，不是那么简单，咱们先不看，首先先了解一下这个插件库所提供的功能，根据官方文档这个插件库有以下特性：

1. 插件是Go接口的实现：这让插件的编写、使用非常自然。对于插件的作者来说，他只需要实现一个Go接口即可；对于插件的用户来说，他只需要调用一个Go接口即可。go-plugin会处理好本地调用转换为gRPC调用的所有细节
2. 跨语言支持：插件可以基于任何主流语言编写，同样可以被任何主流语言消费
3. 支持复杂的参数、返回值：go-plugin可以处理接口、io.Reader/Writer等复杂类型
4. 双向通信：为了支持复杂参数，宿主进程能够将接口实现发送给插件，插件也能够回调到宿主进程
5. 内置日志系统：任何使用log标准库的的插件，都会将日志信息传回宿主机进程。宿主进程会在这些日志前面加上插件二进制文件的路径，并且打印日志
6. 协议版本化：支持一个简单的协议版本化，增加版本号后可以基于老版本协议的插件无效化。当接口签名变化时应当增加版本
7. 标准输出/错误同步：插件以子进程的方式运行，这些子进程可以自由的使用标准输出/错误，并且打印的内容会被自动同步到宿主进程，宿主进程可以为同步的日志指定一个io.Writer
8. TTY Preservation：插件子进程可以链接到宿主进程的stdin文件描述符，以便要求TTY的软件能正常工作
9. 宿主进程升级：宿主进程升级的时候，插件子进程可以继续允许，并在升级后自动关联到新的宿主进程
10. 加密通信：gRPC信道可以加密
11. 完整性校验：支持对插件的二进制文件进行Checksum
12. 插件崩溃了，不会导致宿主进程崩溃
13. 容易安装：只需要将插件放到某个宿主进程能够访问的目录即可

这些特性看上去很多很丰富，功能很强大，然而根据我的了解，实际应用起来还是需要自己做一些封装的，从官方的demo来看，并没有体现出这些特性。

## 2.示例
官方仓库里面有几个example，咱们先看一个最简单的demo


### 1.插件定义
首先先看一下**greeter_interface.go**文件。第一部分咱们可以理解为，先定义这个插件的要实现的接口，约束插件的行为。
```go

// Greeter is the interface that we're exposing as a plugin.
type Greeter interface {
    Greet() string
}
```
然后定义了一个实现这个插件接口的Greeter，这里是通过RPC去实现，所以需要一个rpc client
```go
// Here is an implementation that talks over RPC
type GreeterRPC struct{ client *rpc.Client }

func (g *GreeterRPC) Greet() string {
    var resp string
    err := g.client.Call("Plugin.Greet", new(interface{}), &resp)
    if err != nil {
        // You usually want your interfaces to return errors. If they don't,
        // there isn't much other choice here.
        panic(err)
    }
    return resp
}
```
紧接着，又定义了一个RPCServer包装了一遍
```go
// Here is the RPC server that GreeterRPC talks to, conforming to
// the requirements of net/rpc
type GreeterRPCServer struct {
    // This is the real implementation
    Impl Greeter
}

func (s *GreeterRPCServer) Greet(args interface{}, resp *string) error {
    *resp = s.Impl.Greet()
    return nil
}
```
最后的最后，才是插件的实现，主要是Server和Client这2个方法：
```go
type GreeterPlugin struct {
    // Impl Injection
    Impl Greeter
}

func (p *GreeterPlugin) Server(*plugin.MuxBroker) (interface{}, error) {
    return &GreeterRPCServer{Impl: p.Impl}, nil
}

func (GreeterPlugin) Client(b *plugin.MuxBroker, c *rpc.Client) (interface{}, error) {
    return &GreeterRPC{client: c}, nil
}
```
RPC Plugin接口的定义：
```go
// Plugin is the interface that is implemented to serve/connect to an
// inteface implementation.
type Plugin interface {
    // Server should return the RPC server compatible struct to serve
    // the methods that the Client calls over net/rpc.
    Server(*MuxBroker) (interface{}, error)

    // Client returns an interface implementation for the plugin you're
    // serving that communicates to the server end of the plugin.
    Client(*MuxBroker, *rpc.Client) (interface{}, error)
}
```

最终，在层层包装之下，这个文件定义了一个插件的框架，以及要实现的方法，但是还缺一个实现，实现是在**greeter_impl.go**文件里面

### 2.插件实现
首先定义一个对象实现Greet方法，这个比较简单，这里用到库里面一个logger库：
```go
// Here is a real implementation of Greeter
type GreeterHello struct {
    logger hclog.Logger
}

func (g *GreeterHello) Greet() string {
    g.logger.Debug("message from GreeterHello.Greet")
    return "Hello!"
}
```
然后就是main里面的内容，这块的操作简单说就是设置一些参数，启动一个RPC服务，等待请求的到来：
```go
func main() {
    logger := hclog.New(&hclog.LoggerOptions{
        Level:      hclog.Trace,
        Output:     os.Stderr,
        JSONFormat: true,
    })

    greeter := &GreeterHello{
        logger: logger,
    }
    // pluginMap is the map of plugins we can dispense.
    var pluginMap = map[string]plugin.Plugin{
        "greeter": &example.GreeterPlugin{Impl: greeter},
    }

    logger.Debug("message from plugin", "foo", "bar")

    plugin.Serve(&plugin.ServeConfig{
        HandshakeConfig: plugin.HandshakeConfig{
            ProtocolVersion:  1,
            MagicCookieKey:   "BASIC_PLUGIN",
            MagicCookieValue: "hello",
        },
        Plugins: pluginMap,
    })
}
```
最后别忘了编译插件，插件的编译实际上和普通Go程序没有什么区别，会得到一个二进制文件，后面会用到
```shell
go build -o ./plugin/greeter ./plugin/greeter_impl.go
```
### 3.使用插件
使用插件的代码相对也简单，只需要New一个Client，设置相关参数，然后发起调用：
```go
func main() {
    // We're a host! Start by launching the plugin process.
    client := plugin.NewClient(&plugin.ClientConfig{
        HandshakeConfig: plugin.HandshakeConfig{
            ProtocolVersion:  1,
            MagicCookieKey:   "BASIC_PLUGIN",
            MagicCookieValue: "hello",
        },
        Plugins: map[string]plugin.Plugin{
            "greeter": &example.GreeterPlugin{},
        },
        Cmd: exec.Command("./plugin/greeter"),
        Logger: hclog.New(&hclog.LoggerOptions{
            Name:   "plugin",
            Output: os.Stdout,
            Level:  hclog.Debug,
        }),
    })
    defer client.Kill()

    // Connect via RPC
    rpcClient, err := client.Client()
    if err != nil {
        log.Fatal(err)
    }

    // Request the plugin
    raw, err := rpcClient.Dispense("greeter")
    if err != nil {
        log.Fatal(err)
    }

    // We should have a Greeter now! This feels like a normal interface
    // implementation but is in fact over an RPC connection.
    greeter := raw.(example.Greeter)
    fmt.Println(greeter.Greet())
}
```
这里面需要注意的是一个handshakeConfig里面的配置要和插件实现里面的一致，另外就是设置二进制执行文件的位置。

其次，Plugins是一个map里面存储了插件名和插件定义的映射关系。

## 3.疑问
从上面的demo来看，这个库的插件系统本质上就是本地RPC调用，虽然性能可能低了一点，毕竟本地网络也有开销，但是确实不存在官方插件机制的一些问题。

但是我从这个demo里面并没有看出来是如何实现多个插件共存、以及插件升级更新等功能，实际上在我后续的研究中我发现，这个库并没有实现这些功能。。。

如果要实现这些功能可能得自己去实现，Grafana这个项目在用到这个库得时候就做了大量的封装。

## 4.原理
首先，咱们先看一下插件实现这块，主要就是**plugin.Serve**方法，它需要传入一个插件配置：
```go
func Serve(opts *ServeConfig) {
    // 一些配置
    ...

    // Register a listener so we can accept a connection
    listener, err := serverListener()
    if err != nil {
        logger.Error("plugin init error", "error", err)
        return
    }

    // Close the listener on return. We wrap this in a func() on purpose
    // because the "listener" reference may change to TLS.
    defer func() {
        listener.Close()
    }()
    
    // TLS的配置
    ...

    // Create the channel to tell us when we're done
    doneCh := make(chan struct{})
    
    // Build the server type
    var server ServerProtocol
    switch protoType {
    case ProtocolNetRPC:
        // If we have a TLS configuration then we wrap the listener
        // ourselves and do it at that level.
        if tlsConfig != nil {
            listener = tls.NewListener(listener, tlsConfig)
        }

        // Create the RPC server to dispense
        server = &RPCServer{
            Plugins: pluginSet,
            Stdout:  stdout_r,
            Stderr:  stderr_r,
            DoneCh:  doneCh,
        }

    case ProtocolGRPC:
        // Create the gRPC server
        server = &GRPCServer{
            Plugins: pluginSet,
            Server:  opts.GRPCServer,
            TLS:     tlsConfig,
            Stdout:  stdout_r,
            Stderr:  stderr_r,
            DoneCh:  doneCh,
            logger:  logger,
        }
    default:
        panic("unknown server protocol: " + protoType)
    }

    // Initialize the servers
    if err := server.Init(); err != nil {
        logger.Error("protocol init", "error", err)
        return
    }
    ...

    // Accept connections and wait for completion
    go server.Serve(listener)

    ctx := context.Background()
    if opts.Test != nil && opts.Test.Context != nil {
        ctx = opts.Test.Context
    }
    select {
    case <-ctx.Done():
        listener.Close()
        if s, ok := server.(*GRPCServer); ok {
            s.Stop()
        }
        // Wait for the server itself to shut down
        <-doneCh
    case <-doneCh:
    }
}
```
代码很多,这里只展示了核心代码，其实正做的一件事就是初始化并启动RPC服务，做好接受请求的准备。

更多的代码在插件使用这块，首先我们New了一个Client，这个Client是维护插件的，而且是一个插件一个Client，所以如果你要实现多插件共存，可以去实现一个插件和Client的映射关系即可。
```go
type Client struct {
    config            *ClientConfig
    exited            bool
    l                 sync.Mutex
    address           net.Addr
    process           *os.Process
    client            ClientProtocol
    protocol          Protocol
    logger            hclog.Logger
    doneCtx           context.Context
    ctxCancel         context.CancelFunc
    negotiatedVersion int

    // clientWaitGroup is used to manage the lifecycle of the plugin management
    // goroutines.
    clientWaitGroup sync.WaitGroup

    // stderrWaitGroup is used to prevent the command's Wait() function from
    // being called before we've finished reading from the stderr pipe.
    stderrWaitGroup sync.WaitGroup

    // processKilled is used for testing only, to flag when the process was
    // forcefully killed.
    processKilled bool
}
```
在main里面当我们New完Client之后，依次调用了Client和Dispense2个方法，这个2个方法非常重要：
```go
// Client returns the protocol client for this connection.
// Subsequent calls to this will return the same client.
func (c *Client) Client() (ClientProtocol, error) {
    _, err := c.Start()
    if err != nil {
        return nil, err
    }

    c.l.Lock()
    defer c.l.Unlock()

    if c.client != nil {
        return c.client, nil
    }

    switch c.protocol {
    case ProtocolNetRPC:
        c.client, err = newRPCClient(c)

    case ProtocolGRPC:
        c.client, err = newGRPCClient(c.doneCtx, c)

    default:
        return nil, fmt.Errorf("unknown server protocol: %s", c.protocol)
    }

    if err != nil {
        c.client = nil
        return nil, err
    }

    return c.client, nil
}
```
其中c.Start这个方法干了很多事情，简单说就是根据配置里面的cmd，也就是咱们编译插件之后得到二进制可执行文件，启动插件的rpc服务。

然后再根据协议的不同，启动RPC服务或者GRPC服务，得到一个真正可用Client，相当于就是通道已经打通了，接下来就是发起请求。
```go
func (c *RPCClient) Dispense(name string) (interface{}, error) {
    p, ok := c.plugins[name]
    if !ok {
        return nil, fmt.Errorf("unknown plugin type: %s", name)
    }

    var id uint32
    if err := c.control.Call(
        "Dispenser.Dispense", name, &id); err != nil {
        return nil, err
    }

    conn, err := c.broker.Dial(id)
    if err != nil {
        return nil, err
    }

    return p.Client(c.broker, rpc.NewClient(conn))
}
```
Dispense方法就是根据插件名拿到对应的插件对象，然后又包装了一层拿到一个Client对象，还记得最初定义插件时候那个Client吗？
```go
func (GreeterPlugin) Client(b *plugin.MuxBroker, c *rpc.Client) (interface{}, error) {
    return &GreeterRPC{client: c}, nil
}
```
最后，断言并调用插件的方法，这时候才是真正发起了RPC请求，并获取返回结果：
```go
greeter := raw.(example.Greeter)
greeter.Greet()
```

有一点，我感觉特别奇怪，从这个插件的实现来看，Dispense这步更像是细分了插件里面的插件，因为在我理解，一个二进制文件就是一个插件，一个插件只有一个实现。

但是很明显，这个库并不这么认为，它认为一个插件文件里面可以实现多个插件，所以它增加了一个Plugins来存储插件的映射关系，也就是说你可以在一个插件里面实现多个接口。

这种实现也相当于是一种约束，实际上在Grafana这个项目里面，它通过这种方式声明了可以支持的插件类型，作为插件，根据需求实现部分接口即可。
```go
// getPluginSet returns list of plugins supported
func getPluginSet() goplugin.PluginSet {
    return goplugin.PluginSet{
        "diagnostics": &grpcplugin.DiagnosticsGRPCPlugin{},
        "resource":    &grpcplugin.ResourceGRPCPlugin{},
        "data":        &grpcplugin.DataGRPCPlugin{},
        "stream":      &grpcplugin.StreamGRPCPlugin{},
        "renderer":    &pluginextensionv2.RendererGRPCPlugin{},
    }
}
```
## 5.畅想
写了这么多，对于这个库，个人觉得可用性非常高，经过生产实际检验的，但是只有核心功能，缺少封装，对于使用插件的系统来说，至少要实现以下功能：
1. 多插件的管理。
2. 插件自动、手动更新。
3. 插件健康检查，因为这个插件本质就是一个rpc服务，可能会挂掉，挂掉之后该怎么办呢？

如果真的要想好好用起这个库，确实还得花不少功夫，有一个好得地方是我们可以参考Hashicorp家的其它开源项目代码来完善，其实我也在想能不能稍微把这个库封装一下提供一个简单易用的接口。