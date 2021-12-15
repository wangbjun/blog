---
title: ubuntu unity桌面设置攻略
date: 2021-12-11 12:00:00
tags: 
- Linux
- Ubuntu
category: OS
keywords: Linux,Ubuntu,操作系统
---
难道只有我一个人觉得Ubuntu的unity桌面非常好用吗？

最近把台式机上面的Ubuntu 16.04格式化了，装了黑苹果用了一周，不得不说，MacOS确实很精美，软件生态比Linux丰富很多，比Windows简单安全很多，但是我最终还是不想用了，原因很多，究其原因，我觉得是因为Mac是一个为触摸操作优化的系统，没有触摸板就像是断条腿。

所以，我决定重回Linux，继续用Ubuntu，期间尝试了最新的Ubuntu 20.04版本，发现Gnome桌面模仿Mac系统的痕迹太明显了，扁平化的UI设计真的欣赏不来，最要命的是我总感觉一卡一卡，没有unity桌面流畅，真的不是我的错觉，我之前也尝试过18.04版本，那会感觉也是很卡。

最后想了想，还是回到unity桌面吧，不再折腾了，奈何unity已经被抛弃，只能死守16.04这个版本，虽然看起来有点老了，但是用起来完全没有任何问题。装个系统对于我来说是非常轻松的事情，但最麻烦的是我个人的一些特殊设定、常用软件的安装配置比较耗时，所以这篇文章的主要目的就在于记录整个过程，结合我之前写的一些文章，作为一个记录，也可以供需要的参考一下，如果你喜欢unity桌面的话。

> 以下文章的内容仅针对Ubuntu的unity桌面，准确的说是16.04版本，因为之后的版本都是Gnome桌面，基本上不适用了，Only For Ubuntu16.04。

<!--more-->

## 1.系统安装
其实系统的安装非常简单，网上教程也很多，这里就不再赘述了，简单总结一下过程：

- 下载ISO镜像，从Ubuntu官网下载即可
- 制作U盘启动盘，需要一个至少4GB的U盘，在Windows下我一般使用ultraIso软碟通制作，Ubuntu下的话可以用系统自带的启动盘工具
- BIOS设定，如果启动失败可以尝试修改一下，比如关闭安全启动
- 成功引导进入安装界面后，一步步按照引导步骤来就可以
- 安装完成后，拔掉U盘重启即可

如果你是安装双系统的话，现在的Ubuntu安装程序非常智能，可以检测到你已有系统，需要注意在安装的时候选择”与现有系统共存“即可。

<img src="/images/2021/2021-12-11_12-03.png" />

## 2.卸载多余软件
在Ubuntu 16.04的安装程序里面还没有最小安装的选项，所以默认情况下还会安装很多自带的软件应用，我一般都会选择卸载掉，节省空间，系统也更加简洁，建议先从这里开始，主要包括以下软件：

- 文档套件libreOffice，可以使用WPS替代
- 视频播放器Totem，可以使用smplayer或者vlc替代
- 音乐播放器Rhythmbox，不好用
- 邮件软件Thunderbird，我一般都用网页版
- 图片管理器ShotWell，用不上，卸载不影响图片浏览
- 摄像头软件Cheese，用不上，我电脑没摄像头
- BT下载软件Transmission，可以使用QTbittorrent替代

以上软件都是我个人觉得不好用、用不上的，我一般是通过命令行卸载，我把Shell命令总结了一下，需要的可以直接复制：
```shell
#!/bin/bash
sudo apt purge libreoffice-*
sudo apt purge totem
sudo apt purge rhythmbox
sudo apt purge thunderbird
sudo apt purge shotwell
sudo apt purge cheese
sudo apt purge transmission-gtk
```
卸载完之后建议重启一下，因为有些软件的图标缓存没刷新。

## 3.更新系统脚本
你从官网下载的镜像，一般都不是最新的，所以有时候安装完成之后就需要更新，可能已经弹出提示让你更新了，我一般喜欢写一个脚本文件来实现一键更新、卸载、清理，脚本内容如下：
```shell
#!/bin/bash
sudo apt -y update
sudo apt -y upgrade
sudo apt -y dist-upgrade
sudo apt -y autoremove
sudo apt -y autoclean
```
这个几个命令主要是更新仓库缓存、升级软件、删除无用的依赖包、自动清理。

我一般会把这几个命令放到一个文件，比如名字叫update，然后放到一个文件夹里面比如 **/home/jwang/Documents/MyBin**

然后在设置一下环境变量，把这个文件夹目录放到PATH里面，我一般喜欢改 **/etc/environment** 这个文件，这样以后我就可以直接在终端里面用**update**命令去更新系统。

说到更新，不得不说Ubuntu的更新源的问题，如果你安装的时候选择的是中文，并且时区也没问题，默认情况下应该是cn的中文源，这个源的速度还行，但是有时候可能也很慢，这时候就得换第三方源了，比如阿里云的源、清华大学的源等等，国内有很多。

<img src="/images/2021/2021-12-10_21-00.png" />

网上有很多教程是直接修改source文件的，比较麻烦，而且容易出错，这里告诉大家一起最简单的方式，打开软件和更新设置，里面有一个选择更新源的选项，选择其它源就可以看到国内的所有认证过的第三方源，mirrors开头的都是，选择一个就行，也可以点击右边的按钮，自动选择的一个最快的源。

## 4.sudo免密设置
这个主要是为了方便，毕竟咱是作为个人电脑，天天输密码也没啥意义，看个人需求，如果为了安全考虑就别改了，但我每次都会改。

方法很简单，在命令行输入 **sudo visudo** 会打开一个文件的编辑界面，在最后的位置增加一行：
```shell
your_user_name   ALL=(ALL:ALL) NOPASSWD:ALL
```
your_user_name代表你自己的登录用户名，不要照抄。同时我建议注释掉上方3个default开头的配置项，这块主要是为了解决使用sudo找不到命令的问题，详细的原因可以参考我之前的文章

有很多人不会用nano这个编辑器，修改完成之后，只要按ctrl+x，选择y，回车保存即可。

## 5.屏幕缩放比例
这一块是针对高分屏用户的，比如4k分辨率的显示器，我用的就是，默认情况下字体很小，得设置一下缩放。

这里先教大家一个最简单的方式，只需要在显示设置里面，设置一下”菜单和标题缩放比例“即可，根据我的感觉，4k屏幕设置成1.75比较合适，大家根据自己的情况调节，适合自己最好。

<img src="/images/2021/2021-12-10_21-05.png" />

这里调节的话基本上能够解决90%软件的缩放问题，但是还是有很多软件不生效，比如QT开发的、Wine下的软件，这里推荐大家看一下我以前总结的文章：[Ubuntu 4K显示器缩放设置](https://wangbjun.site/2019/linux/ubuntu-4k-scale.html)

## 6.默认设置调节
这里是针对系统默认设置的一些调节，主要是根据我个人习惯，大家随意，适合自己就好，并不是必须的设置，主要包括以下：

- 安全和隐私。关闭最近文件使用记录、关闭休眠后需要密码选项，因为本人在家使用，图个方便。

- 外观。在行为里面开启自动隐藏启动器，把灵敏度调到最低，如果需要显示启动器只要按一下win键即可，个人感觉这样更简洁。关于窗口菜单这块，我喜欢在窗口标题显示，Ubuntu默认是在最上面，和Mac一样，个人感觉不方便，特别是当你有一个大屏幕，打开一些窗口化的小应用的时候会发现菜单离你好远。不过好在Ubuntu提供了设置选项，不像Mac直接定死了，不可修改。

- 时间和日期。可以设置显示月日以及星期，方便查看

- 账号里面可以设置一下自动登录，这样每次重启的时候就不用输密码，还是为了方便，我是家用，如果在公司可不要这么干。

<img src="/images/2021/2021-12-10_21-18.png" />

## 7.快捷键调节
在系统设置键盘设置里面可以设置一些快捷键，但是这些都无关精要，我实际上想实现一个**Win+D**显示桌面的快捷键，这里无法实现，必须得上神器**compiz**
```shell
sudo apt install compiz-plugins compiz-plugins-extra compizconfig-settings-manager
```
安装完成后打开compiz设置管理器，这个软件功能十分强大，建议不懂的话不要乱动，我这里只需要改一丁点地方，找到**Ubuntu Unity Plugin**,在里面找到Show Deskto的设置项，其默认值是Ctrl+Super+D，其中Super就是Win键，我们这里给改成Win+D，这是Windows默认的快捷键，保持统一，约定大于规则。

其它快捷键有需要的自行修改，我一般还喜欢改一下启动器的快捷键，之前说过默认是Win键，我喜欢改成Alt+Q

## 8.unity-tweak-tool
这里顺便说一下另一个神器**unity-tweak-tool**，功能十分强大，可以修改很多系统设置选项里面没有暴露出来的东西
```shell
sudo apt install unity-tweak-tool
```
比如说工作区的一些调节选项，工作区是Ubuntu非常厉害的功能，也是Mac系统杀手锏，感兴趣的同学可以了解一下，这里不细说了。

<img src="/images/2021/2021-12-10_21-36.png" />

这个软件还可以调节系统的主题、字体等设置，非常有用。

## 9.网速监控插件
Linux上面的网速监控插件有很多，但是能和Ubuntu系统完美结合的我只找到这个:

<img src="/images/2021/2021-12-10_21-45.png" />

这个插件功能简单，2个箭头，一个上传一个下载，可以选择监控哪个网卡，也可以选择监控所有网卡，没有其它多余功能！安装的话需要加一个ppa源：
```shell
sudo add-apt-repository ppa:nilarimogard/webupd8
sudo apt update
sudo apt-get install indicator-netspeed
```
安装重启即可，非常实用，对于我来说是必备

## 10.鼠标灵敏度问题
默认情况下，你的鼠标可能会很飘...特别灵敏，就算你在鼠标设置里面把灵敏度拉到最低依然很快，如果是这样你就需要用命令行去调节一下了，Ubuntu有一个xinput命令 可以列出所有设备和其属性，找到你鼠标的id,然后使用下面命令设置灵敏度：
```shell
xinput --set-prop 10 "Device Accel Constant Deceleration" 1.2
xinput --set-prop 10 "Device Accel Velocity Scaling" 1
```
10是我的鼠标在系统里面的id，后面一个 1.2 和 1 这2个参数大家可以自己调整，找到适合自己的就行，为了每次开机自动设置，我们还需要配置一个开机自启脚本，这里面提供一个简单方法 Ubuntu自带一个名叫”启动应用程序“的软件，比较简单，我们可以添加自己编写的脚本：

另外，你可能发现你的鼠标滚轮滚起来非常慢，特别是在浏览网页的时候，很可惜Ubuntu的鼠标设置界面并没有提供修改的地方，但是也有办法解决：
```shell
sudo apt install imwheel
```
然后在用户目录下创建一个vim .imwheelrc配置文件，写入一下配置：
```
".*"
None,      Up,   Button4, 4
None,      Down, Button5, 4
Control_L, Up,   Control_L|Button4
Control_L, Down, Control_L|Button5
Shift_L,   Up,   Shift_L|Button4
Shift_L,   Down, Shift_L|Button5
```
最主要的就是前面2个行，后面几行可以不用管，其中“4”设置的就是滚动速度，大家可以根据自己的需要设置合适的值，保存之后可以通过**killall imwheel && imwheel**重新加载配置。但是为了每次重启后还能生效，你还需要放到系统开机自启里面。

