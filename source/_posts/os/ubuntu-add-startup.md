---
title: ubuntu添加开机启动
date: 2021-12-13 20:30:00
tags:
- Linux
- Ubuntu
category: OS
keywords: Linux,Ubuntu,开机自启
---
简单说一下Linux设置开机自启的几种方式，以前用Windows很多软件巴不得开机启动，导致开机巨慢，所以每次安装完系统都要各自优化，禁用启动项。

但是Linux下大部分软件都比较守规矩，没有设置自启，对于一些后台进程类应用，还需要自己手动添加开机自启，下面我就针对Ubuntu桌面演示一下。

举个例子，Ubuntu桌面下的鼠标移动速度太快，很灵敏，即使在鼠标设置里面把移动速度拉到最低还是很快，这时候只能祭出神器: **xinput**。

假设最终解决方式是需要使用命令行设置属性，如下：
```shell
xinput set-prop 10 "Device Accel Velocity Scaling" 1.2
```
不过这个命令的效果不是永久的，每次电脑重启就失效了。如果你不嫌麻烦你就每次开机的时候执行一下，但是这显的很蠢。

在Ubuntu下实现这种开机自启、自动执行的效果有很多种方式，下面咱就一一介绍。

<!--more-->

## 1.rc.local
这是我认为最简单的一种方式，rc.local是属于systemctl系统里面的一个配置文件，其完整路径是 **/etc/rc.local**。systemctl这个东西讲起来就有点复杂了，这里就不多说了，你只需要知道，但凡是写在这个文件里面的命令行在开机启动的时候都会被执行。
```shell
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.

exit 0
```
默认情况下，这个文件是空的，只有一些注释，我们只需要把要执行的命令写在 exit 0 之前就可以了。

## 2.开机自启应用
上面所说的方式很多Linux发行版都支持，而这条是针对Ubuntu桌面而言的，系统自带了一个名为“启动应用程序”的软件，打开这个软件会发现里面列出了一些启动程序。

<img src="/images/2021/2021-12-13_20-31.png" />

在右边还有3个按钮，添加、编辑、删除，功能十分简单，只要在命令里面填入想要执行的命令即可，或者选择需要运行的脚本文件（先把命令写到脚本文件里面）。

可能有人好奇，这个保存到哪里去了？我研究了一下发现是保存到 **~/.config/autostart** 目录里面了，是一个desktop文件，你甚至可以手动创建一个这样的文件放进去。

## 3.systemctl
systemctl是一个非常复杂的Linux管理系统，咱们也不用了解太多。如果想实现开机自启，只需要自己创建一个自定义的service，添加到系统里面即可。

这种方式不仅可以实现开机自启，还可以实现 **service xxx re\start**、**service xxx stop** 这种形式的管理，非常方便。

在Ubuntu里面，service文件有2个存放的地方，一个是 **/etc/systemd/system**，另一个是 **/lib/systemd/system**，如果你打开看一下，会发现里面已经有很多服务在了，不少老熟人。其实Linux里面大多数系统级别的服务都是通过这种形式启动的。

咱们先看一个案例，比如 **nginx.service** 文件的内容：
```shell
[Unit]
Description=A high performance web server and a reverse proxy server
After=network.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t -q -g 'daemon on; master_process on;'
ExecStart=/usr/sbin/nginx -g 'daemon on; master_process on;'
ExecReload=/usr/sbin/nginx -g 'daemon on; master_process on;' -s reload
ExecStop=-/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx.pid
TimeoutStopSec=5
KillMode=mixed

[Install]
WantedBy=multi-user.target
```
这个service文件是由3个部分组成，其中Unit里面字段主要是描述服务以及声明服务的依赖关系，而Install里面只是声明一个安装的时候的依赖，最核心的是Service里面定义的字段。

篇幅有限，这里咱也不一一介绍了，如果你想知道这些字段的详细含义，请参考一份专业的文档：http://www.jinbuguo.com/systemd/systemd.service.html

总之，如果你只想实现开机自动执行这么一个最简单的功能，只需要这么写：
```shell
[Unit]
Description=简单的Foo服务

[Service]
ExecStart=/usr/sbin/foo-daemon

[Install]
WantedBy=multi-user.target
```
创建一个"foo.service"文件，写入上面的内容，然后把这个文件移动到 **/etc/systemd/system** 目录里面，再执行enable命令开启服务，不出问题的话，应该会有以下信息：
```shell
jwang@jwang:~$ systemctl enable foo.service
Created symlink from /etc/systemd/system/multi-user.target.wants/foo.service to /etc/systemd/system/foo.service.
```
其实大多数情况下使用service这种方式更多的是为了更改方便的启动、停止、重启应用，而不仅仅是为了实现自启，单纯为了实现自启我还是推荐前面2种更加简单的方式。
