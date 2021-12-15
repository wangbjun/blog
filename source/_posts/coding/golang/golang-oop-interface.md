---
title: Golang面向对象接口编程
date: 2021-12-15 22:00:00
tags: Golang
category: Golang
---

说起面向对象（OOP），很多人都听说过封装、继承、多态这些特性，本质上面向对象只是一种软件编程思想。由此衍生到面向对象语言这个概念，其中Java是最典型的代表，在语言层面就有类和对象的设计。

严格来说Go不是一门面向对象的语言，但是在某种层面上也可以实现面向对象的部分特性，说白了，任何软件工程的主要目标都是为了实现重用性、灵活性和扩展性，Go也不例外。

举个例子：假设你需要把一个大象放到冰箱里面，需要几步？

- 第一步.打开冰箱

- 第二步.把大象放进冰箱

- 第三步,关上冰箱

这3个步骤用面向过程的方式去实现可能就是3个函数，比如openFridge、placeElephant、closeFridge，我们只需要依次调用即可。

但是从面向对象的思维来看，冰箱作为一个对象，它应该有2个函数：open、close，而大象作为一个对象应该有一个函数：walk，我们只需要组合这2个对象的函数就完成这些步骤。

## 1.Go的面向对象
Go里面没有类这个概念，只有结构体struct，结构体可以有属性，如：
```go
type Fridge struct {
    Name   string
    Status string
}
```
虽然结构体里面并不能定义函数，但是我们可以给这个结构体定义方法，通过这种形式：
```go
func (i Fridge) Open()  {
    // open
}

func (i Fridge) Close()  {
    // close
}
```
通过这种方式我们认为Open和Close是属于Fridge这个结构体的方法，这些加在一起可以比作是面向对象语言里面类、类变量、类方法的概念。非常简单易懂，没有其它面向对象语言里面比如静态类、静态属性等等其它特性。

## 2.函数还是方法？
函数英文是function，方法英文是method，很多人对这2个概念不是非常理解，往往都是混着叫，函数方法不分，虽然本质上都是一段代码块。

严格来说，方法是面向对象的概念，它必须属于一个对象，比如Java是一门完全面向对象的语言，所以只有方法，没有函数。 而函数则是很传统的概念，比如C语言里面函数是一等公民，所以C里面只有函数

回到Go里面，其实也应该区分一下，一般我们说函数，指的是这种不属于任何结构体的函数，可以直接调用：
```go
func Open()  {
    // open
}
```
而方法则是属于某个结构体的，不能直接调，你得先New一个对象出来，然后再调用这个对象的方法。

很多语言，比如PHP，既有函数，也有方法，非常灵活，所以最好还是区分一下，虽然意思大家都懂。

## 3.Go的接口
这里说的接口不是指API接口，而是面向对象里面的接口，也叫interface，如上所说Go虽然不是一个完全面向对象的语言，但是依然提供了接口，虽然Go的接口和其它语言接口不太一样。

Go的接口一般被称为是```Duck Type```，当你看到一只鸟走起来像鸭子、游泳起来像鸭子、叫起来也像鸭子，那么这只鸟就可以被称为鸭子。在鸭子类型中，关注点在于对象的行为，能作什么，而不是关注对象所属的类型。

在很多面向对象的语言里面，如果你要实现一个接口你就必须实现其所有定义的抽象方法，这是强制要求，而Go则不是这样，Go甚至连implement这个关键字都没有，你不能“实现”！
```go
type Duck interface {
    Walk()
    Swim()
}
```
只要一个结构体实现了接口定义的所有方法，我们就认为实现了这个接口
```go
type Dog struct {
}

func (i Dog) Walk()  {
}

func (i Dog) Swim()  {
}
```
## 4.为什么需要接口？
其实这个问题也困扰我很久，很多时候我们在写业务代码几乎用不到接口，大多数都是一些方法和函数的调用，但是在看一些底层库源码的时候却发现处处是接口。

到底什么时候该用接口? 这是一个非常值得思考的问题

因为接口这种设计，本质上还是为了灵活性和扩展性，什么时候去用还是得看具体情况，比如一个配置文件库，需要支持json、yaml、ini等多种格式，一个日志库需要支持console、file、api各种输出方法。

而过多的使用interface也会导致代码过于冗余，阅读难度增加，变相增加了后续维护成本，实际工作中，公司开发人员水平层次不齐，最简单直白的代码反而更容易被其它人接手维护。

在我看来，在实际业务开发中，接口最实际的意义其实在于方便写单测，配置依赖注入这种实现模式，可以分割不同层之间的依赖，单独对每一层做单测，从而提高代码质量。

比如在开发中，一个模块依赖另一个模块去实现功能，如果不使用接口做隔离，就很难单独的去做测试：
```go
type ArticleService struct{}

func NewArticleService() ArticleService {
    return ArticleService{}
}

func (i *ArticleService) GetArticles() ([]byte, error) {
    articles, err := NewApi().GetArticles()
    if err != nil {
        return nil, err
    }
    return articles, err
}
```
在这段代码里面，ArticleService是依赖Api去获取测试的，他们之间是耦合的，这样写就很难去单独测试ArticleService的逻辑。
```go
type Api struct{}

func NewApi() Api {
    return Api{}
}

func (i Api) GetArticles() ([]byte, error) {
    get, err := http.Get("https://www.baidu.com")
    if err != nil {
        return nil, err
    }
    defer get.Body.Close()
    all, err := ioutil.ReadAll(get.Body)
    if err != nil {
        return nil, err
    }
    return all, nil
}
```

如果用依赖注入加上接口的方式去改造，可以这么写：
```go
// 定义一个接口
type ApiInterface interface {
    GetArticles() ([]byte, error)
}

type ArticleService struct {
    api ApiInterface
}

func NewArticleService(api ApiInterface) ArticleService {
    return ArticleService{api}
}

func (i *ArticleService) GetArticles() ([]byte, error) {
    articles, err := i.api.GetArticles()
    if err != nil {
        return nil, err
    }
    return articles, err
}
```
我们定义一个接口，它有一个方法，然后ArticleService依赖这个接口，并且我们在New方法里面通过参数的方式注入这个依赖。

在使用的时候区别并不大，我们只需要先初始化Api对象，而且作为参数传入ArticleService内部，然后调用就行了。
```go
func main() {
    articleService := service.NewArticleService(service.NewApi())
    res, err := articleService.GetArticles()
    if err != nil {
        panic(err)
    }
    fmt.Printf("%s\n", res)
}
```
但是其实际意义也不仅如此，一个是ArticleService依赖的是一个接口，不是一个具体的对象，这就是所谓的“面向接口编程，而不是实现”。另外，我们可以单独针对ArticleService做测试，可以Mock一个Api对象，实现解耦。
```go
type mockApi struct {
}

func (mockApi) GetArticles() ([]byte, error) {
    return []byte(""), nil
}

func TestGetArticles(t *testing.T) {
    service := NewArticleService(mockApi{})
    articles, err := service.GetArticles()

    if err != nil {
        t.Fatal("should be nil")
    }
    if len(articles) > 0 {
        t.Fatal("should be 0")
    }
}
```
这种写法可以屏蔽依赖对象对测试结果的影响，专注于自身逻辑的测试，这里只是简单的展示这种用法，实际开发中可以使用一些mock库更加方便的测试各种情况。