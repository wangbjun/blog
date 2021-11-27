---
title: Golang官方HTTP库路由解析
date: 2021-11-27 23:00:00
tags: Golang
category: Golang
---
Golang之所以非常适合用于网络编程的原因之一就是其自带网络库可以非常简单快速的建立一个基于http或者tcp的服务应用。
```go
package main

import "net/http"

func main() {
    http.HandleFunc("/", func(writer http.ResponseWriter, r *http.Request) {
        _, _ = writer.Write([]byte("Hello"))
    })
    err := http.ListenAndServe(":8888", nil)
    if err != nil {
        panic(err)
    }
}
```
官方库这个路由并不区分GET、POST等请求方式，所以很多时候咱们都会使用其它的HTTP框架，比如Gin，不过偶尔也可以用一下，今天咱就看看这个官方库的路由实现。

<!--more-->

让我们点开源码，看看这几行代码到底干了啥，首先看一下个 ```HandlFunc()```，ServerMux这个结构体很重要，它有4个成员属性，其中mu是一个读写互斥锁；m是一个map，其key是一个string（实际上也是路由），value是一个muxEntry；这个muxEntry则是代表了handler和pattern，其中pattern就是咱说的路由，又叫请求path。
```go
type ServeMux struct {
    mu    sync.RWMutex
    m     map[string]muxEntry
    es    []muxEntry // slice of entries sorted from longest to shortest.
    hosts bool       // whether any patterns contain hostnames
}
type muxEntry struct {
    h       Handler
    pattern string
}
// DefaultServeMux is the default ServeMux used by Serve.
var DefaultServeMux = &defaultServeMux

var defaultServeMux ServeMux

// HandleFunc registers the handler function for the given pattern
// in the DefaultServeMux.
// The documentation for ServeMux explains how patterns are matched.
func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
    DefaultServeMux.HandleFunc(pattern, handler)
}
```

HandleFunc最终调用了Handle方法，其主要目的是把路由和handler函数做一个映射关系，实现很简单，就是一个map加一把锁，但是有一个特殊的地方，就是es这个对象的作用，当你的路由以"/"结尾的时候，比如 **/hello/**，会把这个路由放到es里面，并且触发一个排序操作，从最长到最短，这么做是有原因的，咱们后面再看。

```go
// Handle registers the handler for the given pattern.
// If a handler already exists for pattern, Handle panics.
func (mux *ServeMux) Handle(pattern string, handler Handler) {
    mux.mu.Lock()
    defer mux.mu.Unlock()
    
    // 一些异常处理判断。。。
    
    if mux.m == nil {
        mux.m = make(map[string]muxEntry)
    }
    e := muxEntry{h: handler, pattern: pattern}
    mux.m[pattern] = e
	// 如果路由以/结尾，那么多就会触发一个排序操作
    if pattern[len(pattern)-1] == '/' {
        mux.es = appendSorted(mux.es, e)
    }
    
    if pattern[0] != '/' {
        mux.hosts = true
    }
}
```
最终一个请求到来，在解析的时候会调用match这个方法，它首先会精准匹配，从map里面找到，如果找不到则会去es里面从最长到最短依次匹配。
```go
// Find a handler on a handler map given a path string.
// Most-specific (longest) pattern wins.
func (mux *ServeMux) match(path string) (h Handler, pattern string) {
    // Check for exact match first.
    v, ok := mux.m[path]
    if ok {
        return v.h, v.pattern
    }

    // Check for longest valid match.  mux.es contains all patterns
    // that end in / sorted from longest to shortest.
    for _, e := range mux.es {
        if strings.HasPrefix(path, e.pattern) {
            return e.h, e.pattern
        }
    }
    return nil, ""
}
```
举个例子，如果你定义了“/hello"这样的路由，你访问”/hello/“，则会找不到，但是如果定义了"/hello/"，访问“/hello"则会被重定向到"/hello/"。这一点看上去非常奇怪，但是确实是这样。

另外，如果你定义了“/hello/"路由，访问”/hello/1"也会被匹配上，包括”/hello/1/2"等路径。

所以，总结一下，Go官方库这个自带的路由默认是精准匹配，如果你定义的路由不以"/"结尾的话，如果你定义的路由以“/"结尾，则遵循一个最长匹配原则，其实这个有点模糊匹配的味道。