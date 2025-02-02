---
title: 为什么PHP都在转Go?
date: 2021-11-14 12:39:00
tags: PHP
category: PHP
---
作为一名以Web开发为主的程序员，上一次写PHP已经是2年前的时候了，不出意外今后应该都不会再去写PHP代码了，原因很简单：Go太好用了。

这个话题看上去像是语言之争，听起来没有意义，因为大家都说真正牛逼的人什么语言都能写，语言只是工具，最重要的是编程思想。话说的没错，但是从实际开发来说，不同语言的生态环境还是有点区别的，比如很多库包装和调用的形式不一样，来回切换成本很高。

举个例子，常用的http库，如果你写PHP你立马会想到Guzzle这个强大的库，如果是Go呢，标准库自带的功能就已经足够了，如果是Python，应该是用requests这个库，但是Java呢，我还真不清楚。。。

从实际工作的角度来说，一个公司的技术栈往往都是以某些语言为主，比如字节都是Go为主、百度偏向PHP、阿里京东基本上都是Java。原因有很多，但是大多都是规范、维护、内部学习等，毕竟你招聘的时候岗位上面都会说明主要语言。

<!--more-->

### 1.PHP那些年
曾经有一段时间，那是PHP最辉煌的时代，是一个前端尚未独立，前后端不分的时代，那时候网页的HTML内容都是后端处理返回，这一过程简称“套模板”。

那时候对于前端来说，只需要给后端一个静态的HTML，后端会使用模板语言去渲染数据，生成一个动态内容的页面。那个年代不仅有PHP、还有JSP（Java的模板语言）、ASP，不过在这场模板语言之战中，PHP是最大的赢家，成为Web页面开发的王者。

<img src="/images/2021/2021-11-14_14-47.png"/>

那时候很多公司对于PHP的定位就是前端，PHPer主要的工作就是从后端服务（Java）获取数据，简单处理一下，然后渲染HTML模板返回就行了，这导致至今还有很多人依然以为PHP是写前端的。。。

然而这个辉煌时代没有维持多久，前端变天了，随着react、vue、angular等前端MVVC框架的诞生，前后端分离已经是大势所趋。现在不再需要后端去渲染模板了，后端只需要写接口，把数据以JSON的形式给到前端就行了。

很多人喜欢使用PHP原因是因为其作为一门弱类型语言，语法非常灵活，所以在套模板这方面确实快，简单的处理数据也方便，在当年，配合Apache服务器，简直是利器。

### 2.来自Go的狙击
在前后端分离的时代，虽然不用PHP套模板了，PHP还可以去写接口，配合一些PHP Web框架，写起接口来也是非常快的，三五行代码开启一个Web服务，相当于其它语言比如Java来说太简单方便了。

但是这一优势在Go面前算不上太大优势，Go开启的一个Web服务更简单，连nginx都帮你省了，而且在协程的加持下，性能非常高。

下面我从几个方面来说一下Go和PHP的对比：
#### 1.语言设计
虽然把动态弱类型和静态强类型语言放在一起对比本身就不太公平，但是Go出身名门，由知名大佬操刀设计，这么多年来，在语言设计上面非常谨慎，所以Go的特性非常少，学习使用成本非常低，上手快。

PHP历史包袱有点重，且不论PHP几个大版本之间的兼容性问题，光语言设计本身的坑也不少，相信写过几年PHP的人都深有感触，说白了，背后没有大公司支撑操刀设计。

#### 2.PHP本质是C
如果你了解PHP你会发现很多PHP大佬本质上是C程序员，因为PHP本质上是C写的一个Web库，PHP的源码就是C，你写的每一行PHP代码都会被解释器翻译成C代码去执行。

所以如果你想精通PHP，那么你必须精通C语言，很多PHP程序员就是图着简单易用来的，你让去研究C语言，大部分人都做不到。这2者的跨度太大了，通常搞Web开发的都不会写C，精通C的人一般不会搞Web开发。本人不才，写了几年PHP也只是停留在语言本身层面，没有深入了解过其底层设计，看了就忘。

PHP的运行依赖大量C扩展，比如常见的mysql、curl、json、mbstring扩展等等，如果你的业务需要高性能计算，就得自己去写扩展，这意味着你还是离不开C语言。

#### 3.资源占用
这2个语言之间除了性能的差异之外，对资源的占用更加明显，PHP非常耗资源，当你的并发上来之后你会发现CPU和内存可能会成为瓶颈。

虽然说Web层往往可以横向扩展，内存不够用无非就是多跑几台服务器的事情，但是当你的服务器数量达到成千上万规模，使用Go可以节省数倍的资源开销。

这时候有人会说，PHP有swoole，常驻内存，高性能，也有协程可以实现并发编程，资源占用低。个人感觉swoole完全偏离了PHP的初衷，使用swoole意味着你要抛弃PHP生态里面大量成熟的库，更像是断臂求生。还有一点，swoole掌握在商业公司手里，相信大家前一段时间也听说过一些争论，圈子有点乱，用Go的话放心Google不会收费的。换句话说，都用上了swoole了，学习成本也不低，何不尝试一下新语言Go呢？
                             
### 3.大势所趋
实际上，从目前国内市场来看，PHP转Go已经是大势所趋，很多培训班现在连PHP的课程都不开了，都在教Go。

![](/images/2021/2021-11-14_14-58.png)

另外，很多正在用PHP的公司也正在尝试使用Go去做一些东西，一些老项目可能不会动，但是新项目用Go的很多很多，虽然说短期内PHP依然有市场，但是趋势不可逆。

随着Go的生态不断完善，第三方库越来越多，在Web开发方面，Go可能成为Java最有力的竞争者，而PHP则可能慢慢没落。

很早就有人说，Go是现代C语言，但现在看起来C依然还是那个C，在高性能领域的地位无可撼动。Go自带GC和runtime，在性能和易用性之间取一个平衡，这一点非常成功。

你说呢？

