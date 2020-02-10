---
title: Web开发中用到的Cache
date: 2019-02-01 12:02:46
tags: Cache
category: 编程开发
---

## 1.什么是Cache？
Cache(音: 侃屎),中文称为缓存，缓存可以说是计算机系统里面一味良药，在很多地方的设计都用到了Cache，比如在CPU里面的一级缓存，二级缓存，好的CPU还有三级缓存。硬盘也有缓存，比如一般1T的机械硬盘会有64M的闪存缓存。

![](http://ww1.sinaimg.cn/large/5f6e3e27ly1fymbinhu4uj20di0avtel.jpg)

在软件系统里面，缓存更是无处不在，比如浏览器本地缓存、网络缓存、CDN缓存、代理缓存...

<!--more-->

缓存是一种设计思想，在现实生活中也有很多应用，比如京东物流，大家都知道在京东上面买东西快递非常快，那是因为京东在很多大城市的周围建立了自己的仓库，京东把销量比较好的商品提前放在仓库里面。当你下单的时候京东直接从附近的仓库给你发货，速度当然快，最快的情况下几个小时就可以收货。

如果不这样做，直接从厂家发货，比如你在北京，买的东西是从广东生产的，估计至少要2-3天！


## 2.Cache解决的是什么问题？
Cache主要解决数据获取成本高的问题,当你获取一个数据特别麻烦，成本非常高，这里的成本高可能是时间成本，比如网络请求时间，或者是验证问题，并且这个数据是重复的，多次获取的数据完全一样，那你就可以使用缓存。

在软件开发中，最常见的用法就是用来替代关系型数据库某些功能，比如在一个电商网站里面，一个商品的数据一旦上架之后很少改动，正常情况下，用户每次刷新页面都需要从数据库多个表里面获取同样的商品数据，如果网站用户非常多，这对数据库压力还是很大的，这时候就可以使用缓存，把每个商品的数据缓存起来，关于缓存在哪里这个问题咱们待会再说。

>最常见的做法就是以key-value的形式缓存数据，比如在上面的例子里面，以商品ID为key，商品信息为value。缓存一方面是为了降低数据库压力，这样就不用每次都查询数据库了，而且还可以提高网站速度，因为很多缓存是存储在内存里面的，这比磁盘的响应和读取速度高很多数量级。

比如浏览器缓存，很多浏览器都会缓存网站的资源文件，比如图片,js,css,fonts，这样我们就不用每次去网站获取，提高了网页加载速度之外还节省了流量！

在很多数据变化不大，或者对数据时效性要求不高的地方我们都可以使用缓存来提供应用速度，比如接口缓存，假如我们需要调用一个外部接口获取一些数据，但是这个接口比较慢而我们又需要重复去获取这些数据，这时候也可以加缓存。但是缓存并不是银弹，缓存用的不好也会带来一些数据错乱问题，影响系统功能。

## 3.Cache时效问题
缓存时效是使用缓存最需要解决的问题，比如上面说到的商品信息缓存，虽然这个商品信息并不是经常改动，但是万一改动了呢？这就会带来数据不一致问题。解决这个问题有2种相对简单的方法，一种是给缓存设置一个有效期，比如说缓存10分钟，10分钟之后缓存就会失效，然后重新从数据库查询数据重新设置缓存。

这样即使数据不一致，也最多只会影响10分钟，这在一些对数据时效性要求不高的应用里面也可以接受，主要是操作简单。

另一种方式则是在修改数据的时候主动更新缓存，这在方式虽然保证了缓存是最新的，但是操作起来并不简单，一个系统里面可能有多个修改数据的入口，如果某一个地方忘记更新缓存...。为了解决这个问题，有些人采用监控数据库binlog日志的方法来更新缓存，因为无论你通过什么方法修改数据，最终都要操作数据库，这样做虽然有效，但是明显更复杂。

当然我们也可以结合这2种方式，既给缓存设置一定的有效期，也在修改数据的时候更新缓存，这样即使忘记更新缓存，也能保证数据最终会一致。

![](http://ww1.sinaimg.cn/large/5f6e3e27ly1fymbog7e6zj20i707m0tf.jpg)

在http协议里面，缓存是非常重要的，为了解决缓存时效性问题，协议定义了很多header，比如 Expires、Cache-Control、Last-Modified、If-Modified-Since、Etag，具体含义和用法这里不过多解释，但是这些协议头只是约定了一些规则，具体怎么实现还得看web服务器以及中间的代理服务器，其最终目的都是为了既能充分利用浏览器缓存提供网页加载速度，也能及时获取最新数据。

## 4.缓存分类
如果从客户端到服务器中间的过程来分，缓存一般分为这几种：
### 1.客户端缓存。
这个最常见的就是浏览器缓存，除此之外，其它很多手机App，客户端App理论上讲都可以使用缓存。

### 2.代理服务器缓存
最常见的就是各种CDN缓存。有些公司或者企业内部可能也有自己的缓存服务器。还有一些第三方宽带运营商，比如长城宽带，宽带通这类一般内部都有自己的缓存服务器，因为这些宽带服务商的的流量需要向电信、联通购买，如果它们内部能够缓存常用的资源，就可以大大节省流量开销，这也是为什么这些宽带便宜的原因之一。

### 3.服务器缓存
一个请求如果在客户端本地，中间代理服务器都没有找到缓存的资源，它就会到达最终服务器，我们一般说的 memcached，redis缓存就是指服务器缓存。服务器缓存根据类型的不同也可以分好几种：
![](http://ww1.sinaimg.cn/large/5f6e3e27ly1fymcn95eewj20ha09174r.jpg)
#### 一.本地缓存
顾名思义，就是指我们把数据缓存在服务器本地，可细分为文件缓存和内存缓存，文件缓存一般适合少量数据，操作简单，读取速度一般，自带持久化，不会比数据库快很多。内存缓存是把数据存储在服务器内存里面，和内存一样，只要Web服务不重启，数据就不会丢失，和文件缓存比，内存缓存要快很多。

但是如果你有多台Web服务器并且做了负载均衡，使用本地缓存可能会带来数据不一致问题，更新起来更麻烦，一般很少用。

#### 二.远程缓存
这是针对本地缓存而言，所谓远程是指有一个服务器专门提供缓存服务，多台服务器使用同一个缓存服务，这就解决了本地缓存的问题。咱们最常用的memcached和redis就是属于远程内存缓存，其中redis支持持久化。

#### 三.分布式缓存
这是第二种类型的扩展，这时候我们不仅仅有多台Web服务器，还有多台缓存服务器，这时候我们需要解决缓存服务器之间的数据同步问题！

## 5.常见内存缓存应用
通常情况下，我们很少自己去实现缓存服务，往往采用成熟的第三方缓存应用，通常有以下2个：
### 1.memcached
功能简单，只支持常用的增删改查操作，只支持string类型，不支持持久化，不支持集群，性能优秀。

### 2.redis
基本上可以说memcached有的redis都有，memcached没有的redis也有，redis支持的数据类型非常多，功能强大，而且支持持久化，自带集群功能，社区活跃。所以基本上现在使用redis居多，memcached用来存储session这样的临时数据比较合适。如果只做缓存的话，memcached性能要比redis好那么一丢丢而已，但是redis的功能可不仅仅是缓存。

