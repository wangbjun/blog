---
title: 使用Go Plugin实现插件化编程
date: 2021-10-29 23:43:00
tags: Golang
category: Golang
---
说到插件这个东西，很多人都不陌生，一般来说，插件化有几个好处，一个是增加程序扩展性，丰富功能。另外，还可以实现热更新，有些大型应用，动辄几个GB的安装程序，如果一个小小的更新就需要重新下载整个程序，这时候我们就可以把经常更新的模块插件化，这样更新的时候只需要下载一个小更新文件。比如说平时咱们Chrome浏览器都会装一些插件，可以扩展浏览器实现更多的功能，还能灵活的安装卸载。

Golang在1.8版本之后提供了一个 Plugin 的机制，可以动态的加载so文件，实现插件化，虽然并不是非常成熟，但是在特定的情况下还是非常好用。

>Currently plugins are only supported on Linux, FreeBSD, and macOS.


<!--more-->

## 1.快速开始
插件代码和普通代码没什么区别，只是在编译的时候不一样，但是要求是必须只有一个main包
```go
package main

var Name = "Plugin Name"

func GetName() string {
    return Name
}
```
使用```go build -buildmode=plugin```编译，会得到一个so文件，怎么使用这个文件呢？

很简单，分三步：

1.先打开so文件，如果一个插件已经被打开了，那么会返回已存在的plugin

2.使用Lookup查找需要调用的变量或者函数，名字必须大写开头

3.断言后调用
```go
func main() {
    //打开加载插件，参数是插件的存储位置，可以是相对路径
    open, err := plugin.Open("/home/jwang/Documents/plg.so")
    if err != nil {
        panic(err)
    }
    //查找标识符
    lookup, err := open.Lookup("GetName")
    if err != nil {
        panic(err)
    }
    res := lookup.(func() string)()
    fmt.Printf("%v\n", res)

    name, err := open.Lookup("Name")
    if err != nil {
        panic(err)
    }
    fmt.Printf("%v\n", *name.(*string))
}
```

从上面的代码可以看到，插件的使用方式非常朴实无华，简单易懂。

一般来说，为了实现插件化，可以事先定义好一些接口，然后由插件去实现这些接口，这样才能保证一致性，但是接口的定义不能写在插件包或者调用包里面。这时候就需要定义一个专门的公共包，把接口的定义写在里面，这样插件包和调用包都可以引用。

## 2.注意事项
之所以说这个插件方案不成熟，主要是由于主程序和插件程序之间存在很强的依赖性，比如：

1.编译的GO版本必须完全一致

2.双方依赖的公共第三方库版本必须完全一致

3.GOPATH也得保持一致，这一点可以在编译时候使用trimpath参数解决

4.插件加载之后无法卸载

这些问题短时间内好像官方也没有解决的意思，或者说无法解决。总之，Go plugin目前的应用很少，毕竟作为网络编程语言，在容器化大行其道的环境下，更新程序是一件很轻松的事情，除非有特殊需要。