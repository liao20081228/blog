---
title: Linxu内核镜像的格式与产生过程
tags: Linux内核构建系统
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>《Linux内核官方文档》。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------


# 1 Linux内核镜像格式

&emsp;&emsp;Linux内核有多种格式的镜像，包括vmlinux、vmlinuz，Image、zImage、bzImage、uImage、xipImage、bootpImage等.

![1](https://www.github.com/liao20081228/blog/raw/master/图片/Linux内核镜像的格式与产生过程/1.jpg)


## 1.1 vmlinux
&emsp;&emsp;vmlinuz是可引导的、可压缩的内核镜像，vm代表Virtual Memory。Linux支持虚拟内存，因此得名vm。它是由用户对内核源码编译得到，**实质是elf格式的文件**。也就是说，vmlinux是编译出来的最原始的内核文件，未压缩，比较大。这种格式的镜像文件多存放在PC机上，大约为50MB。
>ELF，Executable and Linkable Format，可执行可链接格式，是UNIX实验室作为应用程序二进制接口而发布的，扩展名为elf。可以简单的认为，在elf格式的文件中，除二进制代码外，还包括该可执行文件的其它某些信息，并不是纯粹的二进制代码。




## 1.2 vmlinuz

&emsp;&emsp;vmlinuz是可引导的、压缩过的内核。“vm”代表“Virtual Memory”。vmlinuz是可执行 的Linux内核，它位于/boot/vmlinuz，它是一个软链接。vmlinuz的建立有两种方式:

* 一是编译内核时通过“make zImage”创建，手动拷贝到/boot目录下面。zImage适用于小内核的情况，它的存在是为了向后的兼容性。
* 二是内核编译时通过命令make bzImage创建，然后手动拷贝至/boot目录下。




## 1.3 Image
&emsp;&emsp;Image是经过objcopy处理的只包含二进制数据的内核代码，它已经不是elf格式了，但这种格式的内核镜像还没有经过压缩。
>GNU使用工具程序objcopy作用是拷贝一个目标文件的内容到另一个目标文件中，也就是说，可以将一种格式的目标文件转换成另一种格式的目标文件。 通过使用binary作为输出目标(-o binary)，可产生一个原始的二进制文件，实质上是将所有的符号和重定位信息都将被抛弃，只剩下二进制数据。

## 1.4 zImage和bzImage
&emsp;&emsp;zImage是ARM linux常用的一种压缩镜像文件，它是由image加上解压代码经gzip压缩而成，命令格式是`make [b]zImage`。这种格式的Linux镜像文件多存放在NAND上。

&emsp;&emsp;bz表示big zImage,其格式与zImage类似，但采用了不同的压缩算法，注意，bzImage的压缩率更高。bzImage不是用bzip2压缩的,bz表示“big zImage”。 bzImage中的b是“big”意思。 

&emsp;&emsp;zImage(vmlinuz)和bzImage(vmlinuz)都是用gzip压缩的。它们不仅是一个压缩文件，而且在这两个文件的开头部分内嵌有gzip解压缩代码。所以你不能用gunzip或gzip –dc解包vmlinuz。

&emsp;&emsp;内核文件中包含一个微型的gzip用于解压缩内核并引导它。两者的不同之处在于，老的zImage解压缩内核到低端内存(第一个 640K)，bzImage解压缩内核到高端内存(1M以上)。如果内核比较小，那么可以采用zImage或bzImage之一，两种方式引导的系统运行 时是相同的。大的内核采用bzImage，不能采用zImage。


## 1.5 uImage
### 1.5.1 uImage简介
&emsp;&emsp;uImage是uboot专用的镜像文件，它是在zImage之前加上一个长度为64B的头信息(tag)，在头信息内说明了该镜像文件的类型、加载 位置、生成时间、大小等信息。换句话说，若直接从uImage的0x40位置开始执行，则zImage和uImage没有任何区别。命令格式是`make uImage`，这种格式的Linux镜像文件多存放在NAND 上。64字节的头结构如下：
```cpp
typedef struct image_header {
	__be32		ih_magic;	/* 魔数	*/
	__be32		ih_hcrc;	/* 整个64字节头的crc校验码 */
	__be32		ih_time;	/* uImage的建立时间	*/
	__be32		ih_size;	/* zImage的字节数 */
	__be32		ih_load;	/* uImage要加载的地址	*/
	__be32		ih_ep;		/* zImage的入口位置 = load + 64	*/
	__be32		ih_dcrc;	/* 整个zImage的crc校验码	*/
	uint8_t		ih_os;		/* 操作系统代码	*/
	uint8_t		ih_arch;	/* 芯片类型 */
	uint8_t		ih_type;	/*  镜像类型 */
	uint8_t		ih_comp;	/* 压缩类型 */
	uint8_t		ih_name[IH_NMLEN];	/* 32字节的名字 */
} image_header_t;

```
所以，uImage和zImage都是压缩后的内核映像。而uImage是用mkimage工具根据zImage制作而来的。mkimage工具介绍如下：u-boot里面的mkimage工具来生成uImage（u-boot源码包/tools/mkimage.c )


### 1.5.2 uImage创建工具——`mkimage`
**命令**：mkimage -l uimage file name
&emsp;&emsp;&emsp;mkimage [options] -f image_tree_source_file uimage_file_name
&emsp;&emsp;&emsp;mkimage [options] -F uimage_file_name
&emsp;&emsp;&emsp;mkimage [options] (legacy mode)
**描述**：mkimage命令用于创建与 U-Boot引导加载程序一起使用的映像。这些映像可以包含linux内核，设备树blob，根文件系统映像，固件映像等，可以单独使用，也可以组合使用。mkimage支持两种不同的格式：

* 旧的传统镜像格式连接各个部分（例如，内核映像，设备树blob和ramdisk映像），并添加一个64字节的标头，其中包含有关目标体系结构，操作系统，图像类型，压缩方法，入口点，时间戳，校验和等。
* 新的FIT（平面镜像树）格式允许更灵活地处理各种类型的镜像，并且还通过更强校验来增强镜像的完整性保护。 它还支持启动验证。

<style>table{word-break:initial;}</style>
|选项|描述|
|:--|:--|
|-l uimage_file_name|显示image_header结构体中的头信息|
|**创建传统镜像**|
|-A architecture|设置体系架构。 
|-O os|设置操作系统。 u-boot的bootm命令按os类型更改引导方法。
|-T image type|设置镜像类型。  
|-C compression type|设置压缩类型。 
|-a load addess| 设置十六进制表示的载入地址
|-e entry point|设置十六进制表示的进入点
|-n image name|设置镜像名称
|-d image data file| 使用镜像数据来源
|-x     |设置XIP旗标。即不进行文件的拷贝，在当前位置执行
|**创建FIT镜像**|
|-c comment|指定签名时要添加的注释。
|-D dtc_options| 为用于创建映像的设备树编译器提供特殊选项。
|-f image_tree_source_file|描述FIT镜像结构和内容的镜像树源文件
|-F |表示现有的FIT镜像应修改。不执行dtc编译，不应给出-f标志。这可用于在初始镜像创建后使用附加KEY对镜像进行签名。
|-k key_directory| 指定含有用于签名的key的目录，该目录应该含有用于签名的私钥name.key和用于验证的整数name.crt
|-K key_destination|指定dtb文件以写入公钥。
|-r  |指定用于签署FIT的密钥是必需的。 这意味着必须验证它们才能启动映像。 如果没有此选项，验证将是可选的。
|&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;||

## 1.6 xipImage
&emsp;&emsp;这种格式的Linux镜像文件多存放在NorFlash上，且运行时不需要拷贝到内存SDRAM中，可以直接在NorFlash中运行。

# 2 内核镜像的产生过程
&emsp;&emsp;在嵌入式Linux中，内核的启动过程分为两个阶段。其中，第一阶段启动代码放在arch/arm/kernel/head.S文件中，该文件与体系结构相关，与用户的开发板无关，主要是初始化ARM内核等。第二阶段启动代码是init目录下的main.c。现以执行命令`make zImage`为例来说明，arm-linux内核镜像的产生过程。

* 当用户对Linux内核源码进行编译时，kernel的第1和2阶段代码会生成可执行文件vmlinux，该文件是未被压缩的镜像文件，非常大，不能直接下载到NAND中，通常放在PC机上，这也是最原始的Linux镜像文件。试验时该文件约50M。

* 镜像文件vmlinux由于很大，肯定不能直接烧入NAND中，因此需要进行二进制化，即经过objcopy处理，使之只包含二进制数据的内核代码，去除不需要的文件信息等，这样就制作成了image镜像文件。该镜像文件也是未压缩，只是经过了二进制化而变小。试验时该文件约5M。

* 一般来说，内核镜像是经过压缩的，只是在运行时再将其解压。所以，编译时会先使用gzip将镜像文件image进行压缩(压缩比约为 2:1),再将压缩后的镜像文件和源码中的两个文件arch/arm/boot/compressed/head.S、arch/arm/boot/compressed/misc.c一起链接生成压缩后的镜像文件compress/vmlinuz。试验时该文件约为2.5M。注意，这两个源码文件 是解压程序，用于将内存SDRAM中的压缩镜像zImage进行解压。

* 压缩后的镜像文件compress/vmlinuz经过二进制化，最终生成镜像文件zImage，试验时该文件约为2.5M。当然，在内存 SDRAM中运行压缩镜像文件zImage时，会首先调用两个解压程序arch/arm /boot/compressed/head.S、arch/arm/boot/compressed/misc.c将自身解压，然后再执行kernel 的第一阶段启动代码arch/arm/kernel/head.S。简而言之，在内存中运行内核时，kernel先自身解压，再执行第一阶段启动代码。试 验时运行在内存中的镜像文件约为5M，与image镜像文件大小相同。
------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>《Linux内核官方文档》。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
