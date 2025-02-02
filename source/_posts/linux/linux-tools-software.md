---
title: 闲谈Linux桌面系统下常用工具
date: 2018-12-08 10:03:09
category: Linux
tags: 
    - Linux
    - Ubuntu
---

## 前言
作为一个编程开发人员，Linux操作系统的诞生就是一个传奇故事，如今Linux内核系统更是遍布在咱们日常生活中各种电子设备，比如智能路由器（openWrt）、安卓手机（Android）、服务器。。。而Linux桌面系统的使用率其实也不低，尤其在国外，毕竟Linux开源免费，而国内由于Windows盗版横行，用Linux的相对来说少一点！

本人使用了一年多的Ubuntu和Mint桌面发行版，主要做PHP开发，今天来谈谈自己经常用的工具，欢迎大家点评！

<!--more-->

## 一、开发工具篇
这估计是大家最关心的，毕竟用Linux的大多还是搞编程的程序员，一般人我还真不建议使用Linux，而这方面Linux绝对够用！由于不同语言使用的IDE不同，我说说我自己使用过的。

### 1.JetBrains 全家桶

<img src="/images/old/3571187-2dd1696efe7b0182.png"/> 

有人说，此工具一出，此篇终结，没啥说的了。。。因为这个全家桶支持很多语言，C/C++、Java、Ruby、Python、Php、JavaScript、Go、Mysql、.Net 差不多囊括了常见的编程语言了吧？JetBrains系列都是基于JRE，也就是跨平台的，功能十分丰富，还可以安装插件，缺点就是运行速度相对比较慢（毕竟是Java虚拟机），大家可以去其官网看看，如果有钱可以买正版支持一下！

### 2.Atom/Sublime Text
轻量级的文本编辑器，适合各种编程语言，速度快，安装插件之后功能也非常丰富，然而Sublime的Linux版本有致命缺陷，无法输入中文（搜狗输入法），民间有解决方案。Atom是完全开源免费的，功能也很多丰富，关键是可以完美输入中文！

### 3.数据库管理
我经常用的有Navicat，Windows下面的估计大家都用过，Linux下的是Wine版本，基本功能都有，挺方便。还有一个MySQL Workbench也不错，也有很多人喜欢用phpmyadmin。

### 4.Remarkable
一款MarkDown编辑器，经常写MarkDown可以试试。

### 5.Vi/Vim
这个难度有点高，我也就是偶尔原来修改个配置啥滴，想把vim玩溜那得花不少功夫，据说还有一个上古神器Emacs，从来没有见过身边有人用过。。。不过只要你肯折腾，配置好用起来也就很溜的哦。

### 6.Visual Studio Code
最近才发现的一个编辑器，界面看上去和Atom有点像，功能非常强大，微软出品，适合写C/C++

## 二、日常生活篇
### 1.浏览器
首推 Google Chrome、其次Firefox，不解释，一定要注册/登录Google账号哦。

### 2.聊天
QQ是个Bug，网上有wine版本，功能很残缺，建议大家不要折腾了，实在需要就用Windows虚拟机吧！而微信则可以登录网页版，发送图片文件都没问题。

这里更正一下，QQ有一个wine版本非常不错，功能堪称完美，据说是提取自国产深度Linux系统，其本质上是用了一个收费的wine工具crossover。微信也有客户端版本，是一个第三方的开源项目，名字叫electronic，可以去github看看。---2017-10-14

### 3.虚拟机
VMware 和 Virtualbox, 一个收费，一个开源免费，功能上来说基本差不多，虚拟一个Windows系统有时候还是有点用的。

### 4.下载
Linux下的BT客户端其实很多，比如Transmission、qBitTorrent。。。然而国内这网络状况，基本上下载不动，还是迅雷好使点，我一般是在虚拟机里面用迅雷下载。

### 5.同步网盘
国内有个叫坚果云Nutstore挺好使，每月有免费流量，同步一些小文件没问题，文件多了就要花钱买流量了。

### 6.音乐
网易云音乐，不解释。

### 7.视频
除了系统自带的播放器，我一般用vlc播放器，感觉效果好点。

### 8.文档
WPS当之无愧，而且还没广告，良心之作，卸载掉自带的LibreOffice吧。

### 9.输入法
搜狗输入法也算是良心之作，没广告，自动更新词库，还能换皮肤，不过偶尔会出问题，删除用户配置文件就能解决。

### 10.截图
系统自带截图，PrtSc键全屏截图，shift+PrtSc局部截图，也可以安装一个第三方截图软件Shutter。

使用Linux系统至今，未发现有什么没法解决的问题，Windows能干的事情，Linux也能干，实在干不了的事情虚拟机干，随着Qt的流行，以后跨平台的软件应该会越来越多，相信Linux系统也会变的越来越流行！
