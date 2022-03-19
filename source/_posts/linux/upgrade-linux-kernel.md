---
title: Ubuntu下升级Linux内核版本
date: 2022-03-19 13:00:00
category: Linux
tags: 
    - Ubuntu
---
众所周知，Ubuntu的发行版默认的Linux内核版本都不是最新的，但是通常比较稳定，是和发行版契合的，一般情况下我不建议升级，除非说你有明确的需求。

比如说 Ubuntu16.04 目前的内核版本还是4.15，确实比较老，像我个人比较喜欢unity桌面，所以也一直没有去升级到20.04的发行版（新发行版使用5.0以上版本），但是我们可以单独升级内核。

<img src="/images/2022/screenfetch.png" /> 

本人升级内核主要是折腾、尝鲜，个人感觉升级版本之后感觉确实流畅了一点，其它没有什么变化。

先说结论：目前 Ubuntu16.04桌面版本，可以完美支持的Linux内核版本是5.4，更新一点的版本在安装的时候会有一些headers依赖错误，虽然不致命，但是总感觉有问题，这些底层依赖库自己想要去升级版本的话是非常麻烦的，必须从源码编译，而且可能会有导致系统崩溃的风险。

<!--more-->

## 1.Longterm 版本
Linux内核更新还是非常快，小版本一个接一个，我们可以找一个相对稳定持续更新的版本，打开 ```https://www.kernel.org/``` 官方主页，我们会看到最近几个 **longterm** 的版本：

<img src="/images/2022/longterm.png" /> 

建议升级的适合优先选择这些版本，因为这些版本会持续更新很久。。。当然也不是说其它小版本就不能用了。

这些我们只是看一下版本号，备用，除非你选择自己编译内核，这将非常复杂和耗时，接下来我们将去下载已经编译的好的内核。

## 2.预编译的内核
在 ```https://kernel.ubuntu.com/~kernel-ppa/mainline/``` 这里我们可以下载到编译的适合Ubuntu的内核，每一个版本都有，从其命名格式上面也能看出，前面2个数字是大版本号，最后一个数字代表是小版本，我们可以选择自己需求的版本：

<img src="/images/2022/ubuntu-ppa.png" /> 

点击链接会打开一个新的页面，可以看到上面还分为amd64、arm64、i386等不同架构，这里以64位系统为例，里面有多个deb安装包文件，下载的时候需要注意Linux内核还分为 **lowlatency 和 generic**，也就是通用版本和低延迟版本，据我了解低延迟版本一般适用于对延迟要求比较低的需求场景，比如录音。

这里咱们就选择下载名字里面包含generic的几个安装包，全部下载下来放到一个文件夹里面，然后使用```sudo dpkg -i *.deb```命令安装即可。

<img src="/images/2022/ubuntu-amd64.png" /> 

如果不报错，顺利安装成功的话，会默认启用该内核，重启生效。

卸载的话，只需要使用```apt purge```删除这几个软件即可，以5.4内核为例则是下面这个软件：
```shell
linux-headers-5.4.185-0504185
linux-headers-5.4.185-0504185-generic
linux-image-unsigned-5.4.185-0504185-generic
linux-modules-5.4.185-0504185-generic
```
最后不要忘了，使用```sudo update-grub2```生成新的启动image文件。
```shell
jwang@jwang:~$ sudo update-grub2
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-5.4.185-0504185-generic
Found initrd image: /boot/initrd.img-5.4.185-0504185-generic
Found linux image: /boot/vmlinuz-4.15.0-171-generic
Found initrd image: /boot/initrd.img-4.15.0-171-generic
Found linux image: /boot/vmlinuz-4.15.0-169-generic
Found initrd image: /boot/initrd.img-4.15.0-169-generic
Found Windows Boot Manager on /dev/nvme0n1p1@/EFI/Microsoft/Boot/bootmgfw.efi
Adding boot menu entry for EFI firmware configuration
done
```

最后说一个副作用，我在升级的过程中发现之前的 **virtualbox** 用不了，提示使用 /sbin/vboxconfig 命令配置也没有成功，看日志是因为系统里面缺少合适的header文件。

个人猜测是因为virtualbox这种虚拟机软件比较特殊，它依赖了内核里面的某种机制，可能官方的安装包是针对Ubuntu16.04的内核版本设置的，尝试了多种方法，没有解决。如果你比较依赖virtualbox虚拟机的话，可能要谨慎一点。