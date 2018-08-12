---
layout: post
title:  "以runlevel3启动ubuntu"
categories: cuda
tags:  runlevel ubuntu16.04 cudainstall
author: ganpeng
---

* content
{:toc}


由于学习的原因需要安装cuda。早上下好了runfile的cuda9.2（非apt），安装的时候提示x windows在运行中，必须关闭。

然后cuda文档上说明了需要使用runlevel3启动系统，如果仅仅使用ctrl+alt+F1快捷键切换到tty1是不行的，因为图形界面还是在tty7中运行，实际尝试也验证了这个想法。



google搜索如何使用runlevel3启动ubuntu16.04。发现方法无非就是如下几种：

1. 打开/etc/init/rc-sysinit.conf文件，将下方参数设成3：  
env DEFAULT_RUNLEVEL=2
2. 把grub配置文件中的 **GRUB_CMDLINE_LINUX** 设为 `3`
3. 使用`systemctl set-default multi-user.target`命令将`/etc/systemd/system/default.target`符号链接到`/lib/systemd/system/multi-user.target`
  
一开始尝试的第一种，发现没有任何效果，`DEFAULT_RUNLEVEL`无论是3还是2还是5都进的图形界面，然后我改回了2（以防有其他副作用，所以改回原值）。

接着又搜到了第二种方法，发现仍然是没有任何作用，开机还是进了图形界面。

又尝试第三种方法，发现电脑grub选择启动项之后就卡住了，等了蛮久都没反应。于是进不去系统了。

到这一步需要使用grub启动项的高级选项，里面有recovery mode。进去后使用root shell把刚才的更改恢复。

直到找到这个 [How do I disable X at boot time so that the system boots in text mode?](https://askubuntu.com/questions/16371/how-do-i-disable-x-at-boot-time-so-that-the-system-boots-in-text-mode)

高票答案做法是要把内核启动参数`quiet splash`改成`text`，并且使用systemd的系统（如ubuntu16.04）还要执行一遍第三种方法。

按照该方法能以runlevel3运行ubuntu16.04。

接着我又试了不同的设置。发现如果`/etc/systemd/system/default.target`是链接到`/lib/systemd/system/graphical.target`的话无论内核启动参数是不是text都会启动图形界面。而如果连接到了`/lib/systemd/system/multi-user.target`的话，则当内核启动参数是text进入控制台界面，参数是`quiet splash`则会卡住。

第二种方法的作用其实就是把`3`作为启动参数传递给内核，在grub界面按下`e`键后可以看到具体参数。

另外在grub界面按下`e`键可以直接修改启动参数，不必每次修改grub配置文件并update-grub。

安装cuda之后在跑示例程序时失败了，开始没找到原因，因为所有的命令都是按照官方的installation guide上执行的，后来把在ubuntu设置界面Additional Drivers安装的NVIDIA闭源驱动卸载了再次重新安装cuda，这次示例程序可以正确运行。并且这个界面的驱动变成了下面这样：

![additional dervers](/static/additional drivers.png)

基于以上情况猜测第一次cuda没能正确运行是因为之前已经有了一个驱动。