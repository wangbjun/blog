---
title: Golang泛型入门了解
date: 2022-03-17 22:27:00
tags: Golang
category: Golang
---
## 1.什么是泛型？
最近Go官方刚刚发布了很多人期待已久的1.18版本，其中最大的特性就是正式支持泛型。

>泛型程序设计（generic programming）是程序设计语言的一种风格或范式。泛型允许程序员在强类型程序设计语言中编写代码时使用一些以后才指定的类型，在实例化时作为参数指明这些类型。

上面这段描述比较专业，通俗的说泛型就是广泛的类型。。。很多语言都有实现类似特性，比如在Java里面泛型应用非常广泛，在C/C++里面也有宏、函数模板这样的东西可以实现类似的效果。

<!--more-->

举个例子，假设我们需要实现一个数字相加的函数，因为Go是强类型语言，所以必须申明函数参数类型和返回值，如下：
```go
func sum(param []int) int {
    s := 0
    for _, v := range param {
        s += v
    }
    return s
}
```
但是上面这个函数只能实现int类型的数字相加，如果我们还有一些浮点型数字需要相加呢？在Go支持泛型之前，我们只能再写一个类型申明为float64的“重复”函数，但是这样显的很呆。。。怎么办呢？

## 2.简单示例
下面咱们就看一下如果使用泛型来简化上面这个sum函数，用一个函数解决int和float64这2种数值类型的相加。

其实在泛型诞生之前，很多人经常利用 **interface{}** 类型来实现类似泛型的作用，**interface{}** 有点像是泛型，只不过在使用的时候需要断言，如果使用 **interface{}** 实现的话，我们可以使用反射获取参数的真实类型，然后再断言进行相加。

Go的泛型使用 [] 来申明类型，比如 func sum[v int|float64] 表示v可以是int或者float64类型。

示例如下：
```go
func main() {
    fmt.Printf("%v\n", sum([]int{1, 5}))
    fmt.Printf("%v\n", sum([]float64{1, 2.5}))
    fmt.Printf("%v\n", sum([]string{"a", "b", "cdef"}))
}

func sum[v int | float64 | string](param []v) v {
    var s v
    for _, v := range param {
        s += v
    }
    return s
}
//运行结果
6
3.5
abcdef
```

官方这次还新增了3个泛型试验包，其中有针对slice和map的包，主要是实现了一些通用函数操作，比如查找、比较、排序等，但它们的 API 不在 Go1兼容性承诺的保证范围内:
- golang.org/x/exp/constraints:对通用代码有用的约束，例如constraints.Ordered
- golang.org/x/exp/slices:对任何元素类型的切片进行操作的通用函数集合
- golang.org/x/exp/maps:对任何键或元素类型的映射进行操作的通用函数集合

不过目前来看，Go的泛型特性还在继续完善中，暂时不建议使用在生产环境上，估计还得再等等，其实泛型的最大意义在于实现一些复杂又灵活的功能，比如一些框架。

俗话说：“动态语言一时爽，代码重构火葬场”，为什么这么说呢？ 

因为动态语言一般又叫做弱类型语言，比如PHP和JS，它们的特点就是简单灵活，函数不需要申明类型，比如int和float类型可以直接相加，因为解释器会在运行时做自动类型转换。

而Go、Java这样的则被称作静态语言，又叫强类型语言，虽然性能高，但是缺乏足够灵活性，泛型就是为了弥补这个缺陷，尽可能的在性能和灵活性之间取一个平衡。