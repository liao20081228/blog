---
title: U-boot详解（未完）
tags: 嵌入式
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

# 1  简介
&emsp;&emsp; U-Boot，全称 Universal Boot Loader，是遵循GPL条款的开放源码项目。U-Boot的作用是系统引导。U-Boot从FADSROM、8xxROM、PPCBOOT逐步发展演化而来。其源码目录、编译形式与Linux内核很相似，事实上，不少U-Boot源码就是根据相应的Linux内核源程序进行简化而形成的，尤其是一些设备的驱动程序，这从U-Boot源码的注释中能体现这一点。
 &emsp;&emsp;U-Boot不仅仅支持嵌入式Linux系统的引导，它还支持NetBSD, VxWorks, QNX, RTEMS, ARTOS, LynxOS, android嵌入式操作系统。其目前要支持的目标操作系统是OpenBSD, NetBSD, FreeBSD,4.4BSD, Linux, SVR4, Esix, Solaris, Irix, SCO, Dell, NCR, VxWorks, LynxOS, pSOS, QNX, RTEMS, ARTOS, android。这是U-Boot中Universal的一层含义，另外一层含义则是U-Boot除了支持PowerPC系列的处理器外，还能支持MIPS、 x86、ARM、NIOS、XScale等诸多常用系列的处理器。这两个特点正是U-Boot项目的开发目标，即支持尽可能多的嵌入式处理器和嵌入式操作系统。就目前来看，U-Boot对PowerPC系列处理器支持最为丰富，对Linux的支持最完善。其它系列的处理器和操作系统基本是在2002年11 月PPCBOOT改名为U-Boot后逐步扩充的。从PPCBOOT向U-Boot的顺利过渡，很大程度上归功于U-Boot的维护人德国DENX软件工程中心Wolfgang Denk本人精湛专业水平和执着不懈的努力。当前，U-Boot项目正在他的领军之下，众多有志于开放源码BOOT LOADER移植工作的嵌入式开发人员正如火如荼地将各个不同系列嵌入式处理器的移植工作不断展开和深入，以支持更多的嵌入式操作系统的装载与引导。


* 2008年8月及以前，按版本号命名：u-boot-1.3.4.tar.bz2
* 2008年8月以后均按日期命名。目前最新版本：u-boot-2011.06.tar.bz2
# 2 目录结构
| 目录|描述|
|:--|:--|
|api|存放硬件架构无关的接口函数。从u-boot-1.3.2开始出现|
|arch | 与体系结构相关的代码，从u-boot-2010.06版本开始，每种体系结构下统一有三个目录: cpu、lib、include。cpu目录:存放采用相应体系结构处理器的具体分类，作用与旧版本的cpu目录下的对应目录相同；lib目录:存放对相应的体系结构CPU通用的文件，等价于旧版本的lib_xxx； include:存放与相应体系结构对应的头文件，等价于旧版本根目录下include/xxx-asm目录
| board  |  根据不同开发板定制的代码，代码也不少，每个子目录表示一个开发板
|cmd| uboot命令，每个命令对应一个文件，旧版本存放于common目录
|common|  架构无关的命令杂项，每个命令对应一个文件
|configs|		各开发板对应的默认配置文件
| disk   |  磁盘分区相关代码
| doc   | 文档，一堆README.xxx的文件
| drivers  | 驱动，很丰富，每种类型的设备驱动占用一个子目录
|dts|包含用于构建U-Boot内部 fdt的Makefile。
|env| 环境设置命令。由common目录，旧版本存放于common目录
| examples|  示例程序
| fs  | 文件系统，每种fs一个目录，从linux源代码中移植过来
| include  |通用头文件，与旧版本相比，去除了跟平台相关的头文件
|lib  |  通用库文件，从u-boot-2010.06版本开始，等价于旧版本的Lib_generic
|Licenses|各种许可文件
|net |  该目录下保存着与网络协议相关的代码
|post|加电自检程序
|scripts|各种构建脚本和Makefile
|test|各种单元测试文件
|tools  | 该目录下保存着用于生成u-boot的工具,用于编译和检查uboot目标文件
|**文件**|**描述**
| config.mk|
| Kbuild|
|Kconfig |
|Makefile|  

.

# 3 工作模式
* 启动加载模式 ：该模式是Bootloader的正常工作模式，嵌入式产品发布时，Bootloader必项工作在这种模式下，Bootloader将嵌入式操作系统从FLASH中加载到SDRAM中运行，整个过程是自动的。 
* 下载模式 ：该模式就是Bootloader通过某些通信手段将内核映像或根文件系统映像等从PC机中下载到目标板的FLASH中。用户可以利用Bootloader提供的一些命令接口来完成自己想要的操作。


# 4 启动流程
* 第零阶段（BL0）
	*  加载说明
		*  上电后启动的第一个阶段。其代码固化在ROM中，也就是出厂后已经完成，无法进行修改。上电后CPU会直接从ROM上的BL0的代码上开始执行。
	* 主要工作
		*  初始化系统时钟、特殊设备的控制器、启动设备、看门狗、堆栈、SRAM等等
		*  验证BL1镜像
		*  从存储介质上加载BL1代码到对应内部SRAM上跳转到BL1镜像所在的地址上
* 第一阶段（BL1）
	 * 硬件设备初始化
	 * 加载第二阶段代码到RAM，（由spl完成） 
	 * 设置好栈
	 * 跳转到二阶段代码入口
* 第二阶段（BL2）
	 * 初始化本阶段的硬件设备
	 * 检测系统内存映射
	 * 将内核从Flash读取到RAM中
	 * 为内核设置启动参数
	 * 调用内核

![1](https://www.github.com/liao20081228/blog/raw/master/图片/U-boot/1.JPG)

 

 

 


------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
