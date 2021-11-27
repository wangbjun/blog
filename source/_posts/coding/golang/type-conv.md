---
title: Golang各类型转换
date: 2021-11-21 10:20:00
tags: 
- Golang
category: Golang
---
在使用Golang做业务的时候经常遇到各个类型之间的转换，今天就稍微总结一下常见类型之间的转换方式，主要是int、string、float64、[]byte这几个类型。

<!--more-->

## 1.int和string互转
这个用的还是比较多的，特别是在Web开发的时候，前端传值一般都是string居多，转换主要使用一个库```strconv```来操作：

```go
// StringToInt string => int
func StringToInt(str string) int  {
    i, err := strconv.Atoi(str)
    if err != nil {
        return 0
    }
    return i
}

// IntToString int => string
func IntToString(i int) string {
    return strconv.Itoa(i)
}
```
很多人在转string的时候喜欢用```fmt.Sprintf("%d", i)```，结果一样。

至于int和int64之间的互转，直接```int64(i)```这种写法就可以了，但是可能会存在一个精度丢失，但是Go的int会根据不同的系统架构自动变化，在64位机器上就是int64，所以一般情况下这么转没问题。

## 2.float和string互转
string转float还是用strconv里面的函数，但是float转string的时候的函数用法也着实复杂，所以很多人喜欢用```fmt.Sprintf("%f", f)```，结果一样。
```go
// StringToFloat string => int 
// 注意这个bitSize要么是32，要么是64，如果瞎写的话默认给你算64
func StringToFloat(str string) float64  {
    f, err := strconv.ParseFloat(str, 64)
    if err != nil {
        return 0
    }
    return f
}

// FloatToString float64 => string
func FloatToString(f float64) string  {
    return strconv.FormatFloat(f, 'g', -1, 64)
}
```

## 3.其它类型
1.[]byte和string互转。string本质上就是字节数组[]byte，所以它们俩直接强转就行了，没有什么好说的。

2.int和float64互转。这2个类型之间也可以强制转换，但是也存在精度问题，比如100.9转换成int就变成100了，浮点数都会丢失。

3.int和[]byte互转。理论上这种需求很少，如果非要在做的话，涉及到[]byte的类型转换都可以先转成string，然后再强制转[]byte。