---
layout: post
title:  "编译安装内核源码命令速记"
categories: linux
tags:  kernel ubuntu16.04
author: ganpeng
---

* content
{:toc}


看了 [LINUX KERNEL IN A NUTSHELL](http://www.kroah.com/lkn/) 中的前面部分内容，于是想自己编译安装内核玩玩，就去官网下了个最新的stable版本(4.19.7)。



命令速记：

1. **make defconfig** : 生成默认配置文件。(要安装flex和bison，词法和语法分析工具，前者用于解析单词，后者用于句子归约)  
2. **make menuconfig** : 根据自己的需求改变默认配置，menuconfig是基于控制台的，还有gconfig是基于GTK+的，xconfig基于Qt。(要安装ncurses)
3. **make -j6** : 编译内核，由于使用机器是6和6线程的cpu，所以参数使用6。(需要安装libelf-dev, libssl-dev)
4. **make modules_install** : 拷贝编译好的模块到/lib/modules/kernel_version目录下。
5. **make install** : 由于ubuntu有installkernel命令，所以可以直接使用make install来安装内核。 
 
make install会完成以下工作：  
1. The kernel build system will verify that the kernel has been successfully built
properly.
2. The build system will install the static kernel portion into the /boot directory
and name this executable file based on the kernel version of the built kernel.
3. Any needed initial ramdisk images will be automatically created, using the
modules that have just been installed during the modules_install phase.
4. The bootloader program will be properly notified that a new kernel is
present, and it will be added to the appropriate menu so the user can select it
the next time the machine is booted.
5. After this is finished, the kernel is successfully installed, and you can safely
reboot and try out your new kernel image. Note that this installation does not
overwrite any older kernel images, so if there is a problem with your new
kernel image, the old kernel can be selected at boot time.

重启之后grub默认第一个选项就是新内核，回车启动。

成功进入桌面环境，然而键盘和鼠标都动不了。。。于是登录不了，尝试ctrl+alt+F1也无法切到控制台。  
虽然键盘鼠标都用不了，但是按关机键的时候屏幕上还是会出现重启or关机的界面，所以电源键还是识别的。

google没找到原因。等有空再折腾一波。