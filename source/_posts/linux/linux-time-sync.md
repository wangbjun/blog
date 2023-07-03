---
title: ubuntu双系统时间同步问题
date: 2023-07-03 20:30:022
category: Linux
tags: 
    - Ubuntu
---
估计用过windows和linux双系统切换的人都遇到过这个问题，2个系统之间一切换时间就不对劲了，虽然说可以手动同步，但是每次这么搞有点费劲，不如一劳永逸，彻底解决这个问题！

根本原因：linux系统与win系统对于时间的管理方式不同。

- linux认为硬件时间为GMT+0时间，是世界标准时间，而中国上海是东八区时间，显示时间为GMT+8。

- windows系统认为硬件时间就是本地时间，而这个时间已经被linux设置为GMT+0时间，因此win系统下时间比正常时间慢8个小时。

<!--more-->

有一个快速解决方案，在ubuntu下安装ntpupdate同步时间，然后将本地时间更新到硬件上，命令如下：
```bash
sudo apt-get install ntpdate
sudo ntpdate time.windows.com
sudo hwclock --localtime --systohc
```
上面这个方案亲测可用，也很方便快捷，还有一个方案是执行： ```timedatectl set-local-rtc 1``` 命令让ubuntu将系统时间和BIOS时间同步，但是这个方案会有一个副作用，但是应该对大多人都没影响，问题不大。

当然上面这2种方案本质上都是把windows和ubuntu2个系统对时间的处理方式统一为同一个，只是这里我们只是修改了ubuntu的，没有动windows。

<img src="/images/2023/2023-07-03 20-49-23.png" />

如果你不想这么做，或者这么做有其它的影响，那可以分别在windows、ubuntu上面设置自动时间同步的机制，每次开机的时候自动联网同步时间，缺点就是有点慢，可能开机进入桌面之后等上1-3s才完成同步时间。

- 在windows里面可以使用计划任务，新建一个计划任务，每次用户登录的时候执行，命令是：```w32tm /resync```。

- 在ubuntu里面的话，需要安装ntpdate，然后执行```timedatectl set-ntp on```这样的话会有一个service服务在后台运行自动同步时间。
