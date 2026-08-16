---
title: bootloader和BIOS、uboot和grub、bootmgr的区别
tags: 嵌入式
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《Bootloader和BIOS、Grub、uboot概念》](https://blog.csdn.net/JIA_GUOQIANG/article/details/53149314?locationNum=10&fps=1)。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

# 1 Bootloader和BIOS
## 1.1 BIOS
&emsp;&emsp;BIOS（Basic Input Output System）：基本输入输出系统。它是一组固化到计算机内主板上一个ROM芯片上的程序 ，保存着计算机最重要的基本输入输出的程序、开机后自检程序和系统自启动程序，它可从CMOS中读写系统设置的具体信息。
&emsp;&emsp;BIOS是连接软件与硬件的一座“桥梁”，是计算机的开启时运行的第一个程序，主要功能是为计算机提供最底层的、最直接的硬件设置和控制。BIOS的三个主要功能：

* **自检及初始化程序**，分为三个部分。
 * 加电自检（POST）：功能是检查计算机硬件是否良好，自检中如发现有错误，将按两种情况处理：对于严重故障（致命性故障）则停机，此时由于各种初始化操作还没完成，不能给出任何提示或信号；对于非严重故障则给出提示或声音报警信号，等待用户处理。
 * 初始化：创建中断向量、设置寄存器、对一些外部设备进行初始化和检测等，其中很重要的一部分是BIOS设置，主要是对硬件设置的一些参数，当计算机启动时会读取这些参数，并和实际硬件设置进行比较，如果不符合，会影响系统的启动。
 * 加载引导程序：功能是引导Dos或其他操作系统，此时会在硬盘读取引导记录，然后把计算机的控制权转交给引导记录，由引导记录（IPL与SPL）把操作系统装入电脑，在电脑启动成功后，BIOS的这部分任务就完成了。也就是说**BIOS本身不能引导OS内核**。

* **程序服务处理**；BIOS直接与计算机的I/O设备（Input/Output，即输入/输出设备）打交道，通过特定的数据端口发出命令，传送或者接受各种外部设备的数据，实现软件程序对硬件的直接操作。

* **硬件中断处理**；开机时BIOS会告诉CPU各硬件设备的中断号，当用户发出使用某个设备的指令后，CPU就根据中断号使用相对应的硬件完成工作，再根据中断号跳回原来的工作。

>BIOS分为：Legacy BIOS 和 UEFI BIOS

## 1.2 Bootloader
在典型的嵌入式系统中，引导加载程序（bootloader）完成比BIOS更多的功能，具有如下特点：

* 嵌入式系统中在操作系统内核运行前运行的程序；
* 可以分为单阶段的BootLoader和多阶段的BootLoader，一般从从固态存储设备上启动的Bootloader大多都是 2 阶段的启动过程，如uboot是2阶段的，grub就是单阶段的；

一般而言，这两个阶段完成的功能可以如下分类，但这不是绝对的：

* Bootloader第一阶段的功能。
 * 硬件设备初始化。
 * 为加载Bootloader的第二阶段代码准备RAM空间。
 * 拷贝Bootloader的第二阶段代码到 RAM 空间中。
 * 设置好栈。
 * 跳转到第二阶段代码的C入口点。
 * 在第一阶段进行的硬件初始化一般包括：关闭WATCHDOG、关中断、设置CPU的速度和时钟频率、RAM初始化等。这些并不都是必需的，比如S3C2410/S3C2440的开发板所使用的U-Boot中，就将CPU的速度和时钟频率的设置放在第二阶段。甚至，将第二阶段的代码复制到RAM空间中也不是必需的，对于NORFlash等存储设备，完全可以在上面直接执行代码，只不过这相比在RAM中执行效率大为降低。
* Bootloader第二阶段的功能。
 * 初始化本阶段要使用到的硬件设备
 * 检测系统内存映射(memory map)。
 * 将内核映像和根文件系统映像从Flash上读到RAM空间中。
 * 为内核设置启动参数。
 * 调用内核。

## 1.3 关系
大致说来，嵌入式平台中的Bootloader的作用 相当于 PC中的BIOS+Grub/Bootmgr。

当然嵌入式平台BootLoader还会把设备树之类的文件传给OS内核。


# 2 不同平台的引导加载程序
## 2.1 PC平台的Grub与 bootmgr
&emsp;&emsp;GNU GRUB（GRand Unified Bootloader简称“GRUB”）是一个来自GNU项目的多操作系统启动程序。GRUB是多启动规范的实现，它允许用户可以在计算机内同时拥有多个操作系统，并在计算机启动时选择希望运行的操作系统。GRUB可用于选择操作系统分区上的不同内核，也可用于向这些内核传递启动参数。可用来用来引导不同系统，如windows，linux，通常用于linux，毕竟是GNU的嘛。
&emsp;&emsp;Windows也有类似的工具NTLOADER、Bootmgr；比如我们在机器中安装了Windows 98后，我们再安装一个Windows XP ，在机器启动的会有一个菜单让我们选择进入是进入Windows 98 还是进入Windows XP。NTLOADER就是一个多系统启动引导管理器，NTLOADER 同样也能引导Linux，只是极为麻烦罢了。Bootmgr是Boot Manager的缩写，是在Windows Vista、Windows 7、windows 8/8.1和windows 10中使用的新的启动管理器，以代替Windows NT系列操作系统（Windows XP、Windows 2003）中的启动管理器——NTLOADER。

## 2.2 嵌入式平台的u-boot
&emsp;&emsp;U-Boot，全称为Universal Boot Loader，即通用Bootloader，是遵循GPL条款的开放源代码项目。其前身是由德国DENX软件工程中心的Wolfgang Denk基于8xxROM的源码创建的PPCBOOT工程。后来整理代码结构使得非常容易增加其他类型的开发板、其他架构的CPU(原来只支持 PowerPC)；增加更多的功能，比如启动Linux、下载S-Record格式的文件、通过网络启动、通过PCMCIA/CompactFLash /ATA disk/SCSI等方式启动。增加ARM架构CPU及其他更多CPU的支持后，改名为U-Boot。

* 支持多种嵌入式操作系统内核，如Linux、NetBSD、VxWorks、QNX、RTEMS、ARTOS、LynxOS；
* 支持多个处理器系列，如PowerPC、ARM、x86、MIPS、XScale；


------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《Bootloader和BIOS、Grub、uboot概念》](https://blog.csdn.net/JIA_GUOQIANG/article/details/53149314?locationNum=10&fps=1)。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
