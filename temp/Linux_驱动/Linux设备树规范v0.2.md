---
title: 设备树规范v0.2
tags: Linux驱动
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文翻译自<font color=blue>[设备树官网](https://www.devicetree.org/)。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

<style>table{word-break:initial;}</style>





<style>table{word-break:initial;}</style>

**版权**
版权所有2008,2011 Power.org，Inc。
版权所有2008,2011 Freescale Semiconductor，Inc。
版权所有2008,2011 International Business Machines Corporation。
版权所有2016,2017 Linaro，Ltd。
版权所有2016,2017 Arm，Ltd。

Linaro和devicetree.org文字标记以及Linaro和devicetree.org徽标及相关标记是Linaro Ltd.许可的商标和服务标记。本文档某些元素的实施可能需要第三方知识产权获得许可，包括但不限于专利权。 Linaro及其会员不以任何方式对识别或未能识别的任何或所有第三方知识产权负责。

Power Architecture和Power.org文字标记以及Power和Power.org徽标及相关标记是Power.org许可的商标和服务标记。 本文档某些元素的实施可能需要第三方知识产权获得许可，包括但不限于专利权。Power.org及其会员不以任何方式对识别或未能识别的任何或所有第三方知识产权负责。

本规范“按原样”提供，不作任何形式的保证，包括但不限于任何非侵权，适销性或适用于特定用途的表述或暗示性保证。 在任何情况下，linaro或任何linaro成员均不对任何直接，间接，特殊，示范，惩罚或性损性后果害承担责任，包括但不限于利润损失，即使被告知可能发生此类损害。


与本文件或其条款或条件有关的问题应提交至：
Linaro, Ltd
Harston Mill,
Royston Road,
Harston CB22 7GG
Attn： Devicetree.org Board Secretary


**许可证信息**

根据Apache许可证2.0版获得许可;除非符合许可，否则您不得使用此文件。您可以在<http://www.apache.org/licenses/LICENSE-2.0>获取许可证的副本

除非适用的法律要求书面同意，否则，该许可证下的软件分发按“原样”分发，不附带任何明示或暗示的保证或条件。有关该许可证下的特定语言许可和权限，请参阅许可证。

**致谢**
devicetree.org技术指导委员会要感谢许多做出贡献的个人和公司，他们通过写作，技术讨论和评论来对本规范的制定做出了贡献。

我们要感谢开发和发布ePAPR的power.org平台架构技术小组委员会。ePAPR被用作本文件的起点。

Devicetree规范的重要概念基于Open Firmware Working Group所做的工作，该工作组开发了IEEE-1275的绑定（binding）。 我们要感谢他们的贡献。

我们还要感谢开发和实现扁平设备树概念的PowerPC和ARM Linux社区的贡献。

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;表1 版本历史
|版本|时间|描述|
|:--|:--|:--|
|0.1|2016-05-24|最初的预发布版本。导入ePAPR到结构文本格式，溢出 Power ISA 特定元素。
|0.2|2017-12-20|增加更多推荐的通用节点名<br />增加了中断扩展<br />额外的phy times<br />填写源语言章节中的详细信息<br />编辑变化<br />添加了更改栏版本到发布文档

# 1 介绍
# 1.1 目的和范围

为了初始化和引导计算机系统，各种软件组件进行交互。在将控制权交给软件（如操作系统，引导加载程序或管理程序（hypervisor））之前，固件可能会执行系统硬件的低级初始化。反过来，引导加载程序和管理程序可以加载和转移控制到操作系统。标准，一致的接口和约定有助于这些软件组件之间的交互。在该文档中，术语引导程序一般地指初始化系统状态并执行称为客户端程序的另一软件组件的软件组件。引导程序的示例包括：固件，引导加载程序和管理程序。客户端程序的示例包括：引导加载程序，管理程序，操作系统和专用程序。一个软件可以既是客户端程序又是引导程序（例如，管理程序）。

设备树规范（DTSpec）为客户端程序接口定义提供了完整的引导程序，系统要求最低，这有助于各种系统开发。该规范是针对嵌入式系统。 嵌入式系统通常由系统硬件，操作系统和应用软件组成，这些软件是定制设计用于执行固定的特定任务集。 这与通用计算机不同，通用计算机为具有各种软件和I / O设备的用户设计。 嵌入式系统的其他特性可能包括：

*  一组固定的I / O设备，可能是针对应用程序高度定制的
*  针对尺寸和成本进行优化的系统板
*  有限的用户接口
*  资源限制，如有限的内存和有限的非易失性存储
*  实时约束
*  使用各种操作系统，包括Linux，实时操作系统以及自定义或专有操作系统

**本文件的组织**

*  第1章介绍了DTSpec指定的体系结构。
*  第2章介绍了devicetree概念并描述了其逻辑结构和标准属性。
*  第3章指定了DTSpec兼容设备所需的基本设备节点集的定义。
*  第4章介绍了某类设备和特定设备类型的设备绑定（binding）。
*  第5章指定了设备的DTB编码。
*  第6章指定DTS源语言。

**本文档中使用的约定**
“shall”用于表示严格遵守的强制性要求，以符合标准和不允许偏差（shall等于要求）。

“should”一词用于表示在几种可能性中，有一个特别适合因而被推荐，不提及或排除其他可能性; 或者某种行动方案是首选但不一定是必需的; 或者（以否定的形式）某种行为被弃用但不被禁止（should等于推荐）。

“may”一词用于表示在标准范围内允许的动作（may等于允许）。

设备树构造的示例经常以Devicetree Syntax形式显示。 有关此语法的概述，请参见第6节。

## 1.2 与IEEE1275^TM^和ePAPR的关系
DTSpec与IEEE 1275 Open Firmware标准松散相关（松散相关是紧密相关的反义词）。IEEE1275标准是引导（初始化配置）固件的标准：核心要求和实践[[IEEE1275]](#IEEE1275)。

最初的IEEE 1275规范及其衍生产品如CHRP [[CHRP]](#CHRP)和PAPR [[PAPR]](#PAPR)解决了通用计算机的问题，例如单个版本的操作系统如何在同一系列中的几台不同计算机上工作以及从用户安装的I / O设备加载操作系统。

由于嵌入式系统的性质，开放式通用计算机所面临的一些问题不会发生。 DTSpec中省略的IEEE 1275规范的显著特征包括：

* 插件设备驱动程序
* FCode
* 基于Forth的可编程Open Firmware用户界面
* FCode调试
* 操作系统调试
 
 从IEEE 1275保留的是来自设备树架构的概念，通过该概念，引导程序可以描述并将系统硬件信息传送到客户端程序，从而消除了客户端程序具有系统硬件的硬编码（hard-code）描述的需要。
 
该规范部分取代了ePAPR [[EPAPR]](#EPAPR)规范。 ePAPR记录了Power ISA如何使用设备树，并涵盖了一般概念以及Power ISA特定绑定（binding）。 本文档的文本源自ePAPR，但要么删除特定于体系结构的绑定（binding），要么将它们移动到附录中。


## 1.3 32位和64位支持
DTSpec支持具有32位和64位寻址功能的CPU。 在适用的情况下，DTSpec的各个部分描述了32位和64位寻址的任何要求或注意事项。

## 1.4 术语定义
**AMP** 非对称多处理。计算机可用的CPU被分区为组，每个组运行不同的操作系统映像。 CPU可能（may）相同也可能不相同。

**引导CPU（boot CPU）** 引导程序指向客户端程序入口点的第一个CPU。

**Book III-E** 嵌入式环境。 Power ISA的一部分，定义了嵌入式Power处理器实现中使用的监控指令和相关设施。

**引导程序（boot program）** 通常指初始化系统状态并执行另一个软件组件的软件组件。引导程序的示例包括：固件，引导加载程序和管理程序。

**客户程序（client program）**  典型的客户程序包含应用程序或操作系统软件。客户端程序的示例包括：引导加载程序，管理程序，操作系统和专用程序。

**cell** 32位信息单元（cell）。

**DMA** 直接内存访问。

**DTB** devicetree blob。设备树的二进制形式。

**DTC** 设备树编译器，用于从DTS文件创建DTB文件的一个开源工具。

**DTS** Devicetree syntax。设备树的文本表示。参见附录A 设备树源文件格式（版本1）

**有效地址（effective addres）**  由处理器存储访问或分支指令计算的内存地址。


**物理地址（physical address）** 处理器用于访问外部设备的地址，典型如：内存控制器。

**Power ISA** Power 指令集架构。

**中断说明符（interrupt specifier）**  描述中断的属性值。 典型地，包扩中断号、灵敏度、触发机制。

**次级CPU（secondary CPU ）** 除引导CPU之外的属于客户程序的CPU。

**SMP** 对称多处理。 一种计算机体系结构，其中两个或多个相同的CPU可以共享内存 和IO并在单个操作系统下运行。

**SoC** 片上系统。 单个计算机芯片集成了一个或多个CPU核心以及许多其他外围设备。

**单元（cell）地址（unit address）**  节点名的一部分，指定节点在父节点的地址空间中的地址。

**静止CPU（quiescent CPU）** 静止CPU是一种状态，它不会干扰其他CPU正常操作，也不会被其它正在运行的CPU的正常操作影响，除非用显式方法 启用或重新启用静态CPU。

# 2 设备树
## 2.1 概述
DTSpec指定一种设备树的构造来描述系统硬件。引导程序将设备加载到客户程序的内存中，并将指向设备树的指针传递给客户程序。

本章描述了设备树的逻辑结构，并指定了一组用于描述设备树节点的基本属性集。第3章要求符合DTSpec规范的的设备树需要的某些设备节点。第6章描述了DSpec定义的设备绑定（binding）——表示某些设备类型或类别的要求。 第8章描述了设备树的内存中编码。

设备树是具有描述系统中的设备的节点的数据结构。每个节点都有属性/值对，用于描述所表示设备的特征。 每个节点只有一个父节点，但根节点除外，它没有父节点。

符合设备树规范的设备树描述在系统中客户端程序不一定能动态检测到的设备信息。例如，PCI的体系结构使客户程序能够测试和检测连接的设备，因此可能不需要描述PCI设备的设备树节点。但是，如果无法探测和检测到则需要设备节点来描述系统中的PCI主桥设备。

**示例**
图2.1显示了一个简单设备树的示例表示，该设备树几乎完全足以启动简单的操作系统，具有平台类型、CPU、内存和单个UART。 每个节点内都有属性和值。

![1](https://www.github.com/liao20081228/blog/raw/master/图片/设备树规范v0.2/1.JPG)

## 2.2 设备树结构和约定
### 2.2.1 节点名称
**节点名称要求**

设备树中的每个节点都根据以下约定命名：

&emsp;node-name@unit-address

node-name指定节点的名称。 它的长度应为（shall）1到31个字符，仅由表2.1中的字符集中的字符组成。

 ![3](https://www.github.com/liao20081228/blog/raw/master/图片/设备树规范v0.2/3.JPG)
 
node-name必须（shall）以小写或大写字符开头，并应（should）描述设备的通用类别。
 
unit-address特指节点所在的总线类型。它由表2.1中的一个或多个ASCII字符组成。unit-address必须与节点的reg属性中指定的第一个地址匹配。 如果节点没有reg属性，则必须省略@unit-address，并且node-name单独将节点与树中同一级别的其他节点区分开来。 特定总线的绑定（binding）可能（may）对reg和unit-address的格式指定更多，更具体的要求。

根节点没有node-name或unit-address。 它由正斜杠（/）标识。

![4](https://www.github.com/liao20081228/blog/raw/master/图片/设备树规范v0.2/4.JPG)

在图2.2中：

* 名称为cpu的节点按其单元（cell）地址值0和1进行区分。
* 名称为ethernet的节点通过其单元（cell）地址值Fe002000和Fe003000进行区分。

### 2.2.2 通用名称建议
节点的名称应该（should）是通用的，反映了设备的功能而不是其精确的编程模型。 如果合适，名称应该（should）是以下选项之一：

* adc
* accelerometer
* atm
* audio-codec
* audio-controller
* backlight
* bluetooth
* bus
* cache-controller
* camera
* can
* charger
* clock
* clock-controller
* compact-flash
* can
* cpu
* cpus
* crypto
* disk
* display
* dma-controller
* dsp
* eeprom
* efuse
* endpoint
* ethernet
* ethernet-phy
* fdc
* flash
* gpio
* gpu
* gyrometer
* hdmi
* i2c
* ide
* interrupt-controller
* isa
* keyboard
* key
* keys
* lcd-controller
* led
* leds
* led-controller
* light-sensor
* magnetometer
* mailbox
* mdio
* memory
* memory-controller
* mmc
* mmc-slot
* mouse
* nand-controller
* nvram
* oscillator
* parallel
* pc-card
* pci
* pcie
* phy
* pinctrl
* pmic
* pmu
* port
* ports
* pwm
* regulator
* reset-controller
* rtc
* sata
* scsi
* serial
* sound
* spi
* sram-controller
* ssi-controller
* syscon
* temperature-sensor
* timer
* touchscreen
* usb
* usb-hub
* usb-phy
* video-codec
* vme
* watchdog
* wifi
###  2.2.3 路径名称
通过指定从根节点经过所有后代节点到目标节点的完整路径，可以唯一地识别设备树中的节点。

指定设备路径的约定是：

&emsp;/node-name-1/node-name-2/node-name-N

例如，在图2.2中，cpu＃1的设备路径为：

&emsp;/cpus/cpu@1

根节点的路径是/。

如果节点的完整路径是明确的，则可以省略单元（cell）地址。

如果客户程序遇到不明确的路径，则其行为未定义。

### 2.2.4 属性
设备树中的每个节点都具有描述节点特征的属性。 属性由名称和值组成。

**属性名称**

属性名称是由表2.2中的字符组成的长度为1到31的字符串。

![5](https://www.github.com/liao20081228/blog/raw/master/图片/设备树规范v0.2/5.JPG)

非标准属性名称应指定唯一的字符串前缀，例如股票代码，标识定义属性的公司或组织的名称。例子：

&emsp;&emsp;fsl,channel-fifo-len
&emsp;&emsp;ibm,ppc-interrupt-server#s
&emsp;&emsp;linux,network-index**

**属性值**
属性值是零个或多个字节的数组，其中包含与属性关联的信息。

如果为真假信息，则属性可能（may）为空值。 在这种情况下，该属性是否存就足够描述了。

表2.3描述了DTSpec定义的基本值类型。

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;表2.3 属性值

|值|描述|
|:--|:--|
|\<empty\>|值为空。 属性本身是否存在就足以用于传达真假信息。
|\<u32\>|大端模式的32位整数。 例如：  0x11223344 在内存表示为：<br />&emsp;&emsp;&emsp;address 11<br />&emsp;&emsp;address+1 22<br />&emsp;&emsp;address+2 33<br />&emsp;&emsp;address+3 44
|\<u64\>|大端模式的64位整数。 由两个u32值组成，其中第一个值包含高有效位，第二个值包含低有效位。示例：64位值0x1122334455667788将表示为两个单元（cell）格：\<0x11223344 0x55667788\>。该值将在内存中表示为：<br />&emsp;&emsp;&emsp;address 11<br />&emsp;&emsp;address+1 22<br />&emsp;&emsp;address+2 33<br />&emsp;&emsp;address+3 44<br />&emsp;&emsp;address+4 55<br />&emsp;&emsp;address+5 66<br />&emsp;&emsp;address+6 77<br />&emsp;&emsp;address+7 88<br />&emsp;&emsp;
|\<string\>|字符串可打印，并且以null结尾。 示例：字符串“hello”将在内存中表示为：<br />&emsp;&emsp;&emsp;address 68 'h'<br />&emsp;&emsp;address+1 65 'e'<br />&emsp;&emsp;address+2 6C 'l'<br />&emsp;&emsp;address+3 6C 'l'<br />&emsp;&emsp;address+4 6F 'o'<br />&emsp;&emsp;address+5 00 '\0'
|\<prop-encoded-array>|特定属性格式 。 请参阅属性定义。|
|\<phandle\>|一个u32值。 一个phandle值是在设备树中引用另一个节点的一种方法。可以引用的任何节点都定了具有唯一的u32值的phandle属性。该数字用于具有phandle值类型的属性的值。
|\<stringlist> |字符串列表。例如：字符串列表 “hello”,”world” 将在内存中表示为：<br />&emsp;&emsp;&emsp;address 68 'h'<br />&emsp;&emsp;address+1 65 'e'<br />&emsp;&emsp;address+2 6C 'l'<br />&emsp;&emsp;address+3 6C 'l'<br />&emsp;&emsp;address+4 6F 'o'<br />&emsp;&emsp;address+5 00 '\0'<br />&emsp;&emsp;address+6 77 'w'<br />&emsp;&emsp;address+7 6f 'o'<br />&emsp;&emsp;address+8 72 'r'<br />&emsp;&emsp;address+9 6C 'l'<br />&emsp;&emsp;address+10 64 'd'<br />&emsp;&emsp;address+11 00 '\0'<br />&emsp;&emsp;

## 2.3 标准属性
DTSpec为设备节点指定一组标准属性。 本节将详细介绍这些属性。DTSpec定义的设备节点（参见第3章）可以（may）指定有关标准属性使用的附加要求或约束。 第4章描述了特定设备的表示，还可以（may）指定其他要求。

>注意：本文档中设备树节点的所有示例都使用DTS格式来指定节点和属性。

### 2.3.1 compatible
**属性名**：compatible
**值类型**：\<stringlist>
**描&emsp;述**：

* compatible属性由定义设备的特定编程模型的一个或多个字符串组成。客户端程序应（should）使用此字符串列表来选择设备驱动程序。属性值由一系列以null结尾的字符串构成，从特殊到一般。它们允许设备表达与一系列类似设备的兼容性，可能允许单个设备驱动程序与多个设备匹配。

* 推荐的格式是 "manufacturer,model"，其中manufacturer是描述制造商名称的字符串（比如股票代码），model指定模型型号。

**示&emsp;例**：
&emsp;&emsp;compatible = "fsl,mpc8641", "ns16550";
在此示例中，操作系统将首先尝试查找支持fsl，mpc8641的设备驱动程序。 如果没有找到，那么将尝试查找支持一般的ns16550设备类型的驱动。


### 2.3.2 model
**属性名**：modle
**值类型**：\<string>
**描&emsp;述**：

* model属性值是一个字符串，用于指定制造商的设备型号。

* 推荐的格式是“制造商，型号”，其中制造商是描述制造商名称的字符串（比如股票代码），型号指定模型型号。

**示&emsp;例**：
&emsp;&emsp;model = "fsl,MPC8349EMITX";

###  2.3.3 phandle
**属性名**：phandle
**值类型**：\<u32>
**描&emsp;述**：phandle属性为设备树中的唯一节点制定了数字化的标识符。phandle属性的值是被其它需要引用该节点的节点在与相应的属性值处使用。
**示&emsp;例**：请参阅以下设备树片段：
```dts
pic@10000000 {
	phandle = <1>;
	interrupt-controller;
};
```
定义了一个为1的phandle。 另一个设备节点可以引用具有值为1的phandle的pic节点：
```dts
another-device-node {
	interrupt-parent = <1>;
};
```
>注意：旧版本的设备树可能会遇到叫做 linux，phandle的属性，它其实是phandle的弃用形式。 为了兼容性，如果没有phandle属性，客户程序可能支持linux，phandle。 这两个属性的含义和用法是相同的。

>注意：DTS中的大多数设备树（参见附录A）不包含显式的phandle属性。 当DTS编译为二进制DTB格式时，DTC工具会自动插入phandle属性。

### 2.3.4 status

**属性名**： status
**值类型**：\<string>
**描&emsp;述**：status属性指示设备的运行状态。 下表中列出并定义了有效值

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;表 2.4： status 属性的值
|值 |描述|
|:--|:--|
|"okay" |表示设备正在运行l
|"disabled"|表示设备目前没有运行，但未来可以运行（例如插入或关掉了什么东西） ，有关给定设备的禁用方法的详细信息，请参阅设备绑定（binding）。
|"fail" |表示设备无法运行。 设备中检测到严重错误，未经修复就不可能运行。
|"fail-sss" |表示设备无法运行。 在设备中检测到严重错误，未经修复就不可能运行。 值的sss部分特定于设备并指示检测到的错误原因。


### 2.3.5 #address-cells and #size-cells
**属性名**： #address-cells，#size-cells
**值类型**：\<u32>
**描&emsp;述**：
* #address-cells和#size-cells属性可用于在设备树层次结构中具有子节点的任何设备节点，并描述应如何寻址子设备节点。#address-cells属性定义用于编码子节点的reg属性的地址字段的\<u32>单元（cell）的数量。#size-cells属性定义用于对子节点的reg属性中的size字段进行编码的\<u32>单元（cell）的数量。

* #address-cells和＃size-cells属性不是从设备树中的祖先继承的。 它们应明确定义。

* 符合DTSpec标准的引导程序应在所有有子节点的节点上提供＃address-cells和＃size-cells。 如果缺少，客户端程序应该对＃address-cells假定默认值为2，对于#size-cells应该为1。

**示&emsp;例**：请参阅以下设备树片段：
```dts
soc {
	#address-cells = <1>;
	#size-cells = <1>;
	serial {
		compatible = "ns16550";
		reg = <0x4600 0x100>;
		clock-frequency = <0>;
		interrupts = <0xA 0x8>;
		interrupt-parent = <&ipic>;
	};
};
```
在此示例中，soc节点的＃address-cells和#size-cells属性都设置为1。此设置指定需要一个单元（cell）来表示子节点的reg属性中的地址，并且需要一个单元（cell）来表示子节点的reg属性中的大小。 

串行设备reg属性必须遵循父节点（soc）中设置的规范——地址由1个单元（cell）（0x4600）表示，大小由1个单元（cell）（0x100）表示。

### 2.3.6 reg
**属性名**：reg
**值类型**：\<prop-encoded-array> 编码为任意数量的（地址，长度）对。
**描&emsp;述**：
* reg属性描述了由其父总线定义的地址空间内的设备资源的地址。最常见的是，这意味着内存映射IO寄存器块的偏移量和长度，但在某些总线类型上可能有不同的含义。 根节点定义的地址空间中的地址是CPU实际地址。

* 该属性的值是<prop-encoded-array>，由任意数量的地址和长度对组成，\<地址 长度>。指定地址和长度所需的\、\<u32>单元（cell）的数量是特定于总线的，并由设备节点的父节点中的#address-cells和#size-cells属性指定。 如果父节点为#size-cells指定值0，则应省略reg值的长度字段。

**示&emsp;例**：假设SoC的器件有两个寄存器块，32字节块在SoC中偏移量为0x3000，256字节块的偏移量为0xFE00。 reg属性将编码如下（假设＃address-cells和＃size-cells值为1）：
&emsp;&emsp;reg = <0x3000 0x20 0xFE00 0x100>;


### 2.3.7 virtual-reg
**属性名**：virtual-reg
**值类型**：\<u32>
**描&emsp;述**：virtual-reg属性指定一个有效地址，该地址映射到设备节点的reg属性中指定的第一个物理地址。 此属性使引导程序能够为客户程序提供已设置的虚拟地址到物理地址的映射。

### 2.3.8 ranges

**属性名**：ranges
**值类型**：\<empty>或\<prop-encoded-array>编码为任意数量的（子总线地址，父总线地址，长度）三元组。
**描&emsp;述**：
* ranges属性提供了一种定义总线地址空间（子地址空间）和总线节点的父节点的地址空间（父地址空间）之间的映射或转换的方法。

* ranges属性值的格式是任意数量的（子总线地址，父总线地址，长度）三元组。

	* 子总线地址是子总线地址空间中的物理地址。 表示地址的单元（cell）数取决于总线，可以从该节点（出现ranges属性的节点）的#address-cells确定。
	* 父总线地址是父总线地址空间内的物理地址。 表示父地址的单元数（cell）取决于总线，可以从定义父地址空间的节点的＃address-cells属性中确定。
	* 长度是子地址空间中的范围的大小。 表示大小的单元数可以从该节点（出现ranges属性的节点）的#size-cells确定。

* 如果使用\<empty>值定义属性，则它指定父地址空间和子地址空间相同，并且不需要地址转换。

* 如果该属性不存在于总线节点中，则假定该节点的子节点与父地址空间之间不存在映射。

**示&emsp;例**：
```dts
soc {
	compatible = "simple-bus";
	#address-cells = <1>;
	#size-cells = <1>;
	ranges = <0x0 0xe0000000 0x00100000>;
	serial {
		device_type = "serial";
		compatible = "ns16550";
		reg = <0x4600 0x100>;
		clock-frequency = <0>;
		interrupts = <0xA 0x8>;
		interrupt-parent = <&ipic>;
	};
};
```
soc节点指定一个ranges属性：

&emsp;&emsp;<0x0 0xe0000000 0x00100000>;

此属性值指定了1024KB的地址空间范围，在子节点的物理地址0x0处寻址时映射到父节点的物理地址0xe0000000。通过此映射，serial设备节点可以通过在地址0xe0004600处的载入或存储来被寻址，0xe0004600为子地址空间中的偏移量为0x4600（在reg中指定）加上ranges中映射的父地址0xe0000000。

### 2.3.9 dma-ranges



**属性名**：dma-ranges
**值类型**：\<empty>或\<prop-encoded-array>编码为任意数量的（子总线地址，父总线地址，长度）三元组。
**描&emsp;述**：
* dma-ranges属性用于描述内存映射总线的直接内存访问（DMA）结构，其设备树父节点可以从源自总线的DMA操作访问。它提供了一种定义总线的物理地址空间和总线父节点的物理地址空间之间的映射或转换的方法。

* dma-ranges属性值的格式是任意数量的（子总线地址，父总线地址，长度）三元组。每个三元组特定描述连续的DMA地址范围。

	* 子总线地址是子总线地址空间中的物理地址。 表示地址的单元（cell）数取决于总线，可以从该节点（出现dma-ranges属性的节点）的#address-cells确定。
	* 父总线地址是父总线地址空间内的物理地址。 表示父地址的单元数（cell）取决于总线，可以从定义父地址空间的节点的＃address-cells属性中确定。
	* 长度是子地址空间中的范围的大小。 表示大小的单元数可以从该节点（出现dma-ranges属性的节点）的#size-cells确定。

### 2.3.10 name（弃用）
**属性名**：name
**值类型**：\<string>
**描&emsp;述**：name属性是一个指定节点名称的字符串。 此属性已经废弃，不建议使用它。 但是，它可能用于较旧的非DTSpec兼容的设备。 操作系统应根据节点名称的node-name字段确定节点的名称（请参阅第2.2.1节）。

### 2.3.11 device_type (弃用)
**属性名**： device_type
**值类型**：\<string>
**描&emsp;述**：在IEEE 1275中使用device_type属性来描述设备的FCode编程模型。 由于DTSpec没有FCode，因此不推荐使用该属性，并且**只应将其在cpu和内存节点上**，以便与IEEE 1275派生的设备树兼容。

## 2.4 中断与中断映射

DTSpec采用在Open Firmware Recommended Practice： Interrupt Mapping, Version 0.9 [[b7]](#b7)中指定的表示中断的中断树模型。在设备树内，存在逻辑中断树，其表示平台硬件中的中断的层次结构和中断路径。 虽然通常称为中断树，但它在技术上实际是有向无环图。



中断源到中断控制器的物理连线在设备树中用interrupt-parent属性表示。表示产生中断的设备的节点包含一个interrupt-parent属性，该属性的phandle值指向设备的中断路由到的设备，通常是一个中断控制器。 如果中断生成设备没有interrupt-parent属性，则假定其中断父设备是其设备树中的父设备。

每个中断生成设备包含一个interrupts属性，其值描述该设备的一个或多个中断源。每个中断源都用称为中断说明符的信息表示。中断说明符的格式和含义是特定于中断域的，它取决于中断域根节点上的属性。＃interrupt-cells属性由中断域的根节点用于定义编码中断说明符所需的\<u32>值的数量。例如，对于Open PIC中断控制器，中断说明符采用两个32位值表示，并由中断编号和中断的电平/敏感信息组成。

中断域是解释中断说明符的上下文。 中断域的根节点是一个中断控制器（interrupt controller）或中断连接（interrupt nexus）。

* 中断控制器是物理设备，需要驱动程序来处理路由到它的中断。它也可以级联到另一个中断域。中断控制器由设备树中该节点上的interrupt-controller属性指定。
* 中断连接（interrupt nexus）定义了一个中断域与另一个中断域之间的转换。转换基于特定中断域的信息和特定总线的信息。中断域之间的这种转换是使用interrupt-map属性执行的。例如，PCI控制器设备节点可以是定义从PCI中断命名空间（INTA，INTB等）到具有中断请求号（IRQ）的中断控制器的转换的中断连接。

当遍历中断树到达一个没有 interrupts属性和显式中断父节点的中断控制器节点时，他就被确定为中断树的根节点。

有关设备树中的中断父关系的图形表示，请参见图2.3。它显示了设备树的自然结构以及每个节点在逻辑中断树中的位置。

![6](https://www.github.com/liao20081228/blog/raw/master/图片/设备树规范v0.2/6.JPG)

在图2.3所示的示例中：

*  open-pic中断控制器是中断树的根。
*  中断树根节点有三个子设备，它们将中断直接路由到open-pic
	*   device1
	*   PCI host controller
	*   GPIO Controller
*   存在三个中断域; 一个以open-pic节点为根，一个以pci-host节点为根，一个以gpioctrl节点为根。
*   有两个中断连接节点; 一个位于pci-host节点，一个位于gpioctrl节点。

###  2.4.1 中断生成设备的属性
**属性名**：interrupts
**值类型**：\<prop-encoded-array>编码为任意数量的中断说明符。
**描&emsp;述**：
* 设备节点的interrupts属性定义设备生成的中断。interrupts属性的值由任意数量的中断说明符组成。中断说明符的格式由中断域的根节点的绑定（binding）定义。

* interrupts会被 interrupts-extended属性覆盖，通常只应使用其中一种。

**示&emsp;例**：开放式PIC兼容中断域中的中断说明符的通用定义包括两个单元; 中断号和电平/敏感信息。请参阅以下示例，该示例定义了一个中断说明符，中断号为0xA，电平/敏感信息编码为8。

&emsp;&emsp;interrupts = \<0xA 8>;

**属性名**：interrupt-parent
**值类型**：\<phandle>
**描&emsp;述**：由于中断树中节点的层次结构可能与设备树中的节点层次结构不匹配，因此可以使用interrupt-parent属性显式定义中断父节点。该属性的值是中断父节点的phandle。**如果设备中缺少此属性，则假定其中断父节点是其设备树中的父节点**。


**属性名**：interrupts-extended
**值类型**：\<phandle> \<prop-encoded-array>
**描&emsp;述**：interrupts-extended属性列出了设备生成的中断。当设备连接到多个中断控制器时，应使用interrupts-extended而不是interrupts，因为它使用每个中断说明符对父phandle进行编码。
**示&emsp;例**：这个例子展示了一个有两个中断输出连接到两个独立的中断控制器的设备如何使用interrupts-extended属性描述连接。pic是一个中断控制器，其#interrupt-cells属性为2，而gic是一个中断控制器，其#interruptts-cells属性为1。
&emsp;&emsp;interrupts-extended = <&pic 0xA 8>, <&gic 0xda>;

>interrupts和interrupts-extended属性是互斥的。 设备节点应使用其中一个。仅当需要与不理解interrupts-extended的软件兼容时，才允许使用两者。如果二者同时存在，则interrupts-extended优先。

### 2.4.2 中断控制器的属性

**属性名**：#interrupt-cells
**值类型**：\<u32>
**描&emsp;述**：＃interrupt-cells属性定义了编码中断域的中断说明符所需的单元(cell)数。



**属性名**：interrupt-controller
**值类型**：\<empty>
**描&emsp;述**：interrupt-controller属性把节点定义为中断控制器节点。

### 2.4.3 中断连接的属性

中断连接节点应该具有#interrupt-cells属性。

**属性名**：interrupt-map
**值类型**：\<prop-encoded-array> 编码为任意数量的中断映射条目。
**描&emsp;述**：
*  interrupt-map是中断连接（interrupt nexus）节点上的属性，它将一个中断域与一组父中断域桥接，并指定子域中的中断说明符如何映射到其各自的父中断域中。

*  中断映射是一个表，其中每一行是一个映射条目，由五个字段组成：中断子节点单元地址，中断子节点中断说明符，中断父节点，中断父节点单元地址，中断父节点中断说明符。
	*  **中断子节点单元地址** 要映射的子节点的单元地址。此字段所需的32位单元（cell）的数量由中断子节点所在的总线节点的#address-cells属性描述。
	*  **中断子节点中断说明符** 要映射的子节点的中断说明符。此字段所需的32位单元数由该节点（包含interrupt-map属性的nexus节点）的#interrupt-cells属性描述。
	*  **中断父节点** 单个\<phandle>值，指向子中断域要映射到的中断父节点。
	*  **中断父节点单元地址** 中断父节点的单元地址。 此字段所需的32位单元数由中断父节点字段指向的节点的#address-cells属性描述（(注意，不是中断父设备所在总线的#address-cells属性)）。
	*  **中断父节点中断说明符** 中断父节点的中断说明符。此字段所需的32位单元数由中断父节点字段指向的节点的#interrupt-cells属性描述。

* 在中断映射表上将单元地址/中断说明符对与中断映射中的字段进行匹配。由于单元中断说明符中的某些字段可能不相关，因此在查找完成之前应用掩码。 该掩码在interrupt-map-mask属性中定义（参见第2.4.3.2节）。
>注意：中断子节点和中断父节点都需要定义#address-cells和#interrupt-cells属性。 如果不需要单元地址字段，则#address-cells应明确定义为零。

>译者注：中断映射就是把中断子节点产生的中断转为中断连接节点产生的中断。


**属性名**：interrupt-map-mask
**值类型**：\<prop-encoded-array> 编码为位掩码。
**描&emsp;述**：为中断树中的中断连接节点指定了interrupt-map-mask属性。此属性指定一个掩码，该掩码应用于在interrupt-map属性指定的表中查找的中断说明符时。


**属性名**：#interrupt-cells
**值类型**：\<u32>
**描&emsp;述**：#interrupt-cells属性定义了编码中断域的中断说明符所需的单元(cell)数。


### 2.4.4 中断映射示例
下面显示了具有PCI总线控制器和一个中断映射的设备树片段，该中断映射用于描述两个PCI插槽（IDSEL 0x11,0x12）的中断路由。插槽1和2的INTA，INTB，INTC和INTD引脚连接到Open PIC中断控制器。

```dts
soc {
	compatible = "simple-bus";
	#address-cells = <1>;
	#size-cells = <1>;
	
	open-pic {
		clock-frequency = <0>;
		interrupt-controller;
		#address-cells = <0>;
		#interrupt-cells = <2>;
	};
	
	pci {
		#interrupt-cells = <1>;
		#size-cells = <2>;
		#address-cells = <3>;
		interrupt-map-mask = <0xf800 0 0 7>;
		interrupt-map = <
			/ * IDSEL 0x11 - PCI slot 1* /
			0x8800 0 0 1 &open-pic 2 1 / * INTA * /
			0x8800 0 0 2 &open-pic 3 1 / * INTB * /
			0x8800 0 0 3 &open-pic 4 1 / * INTC * /
			0x8800 0 0 4 &open-pic 1 1 / * INTD * /
			/ * IDSEL 0x12 - PCI slot 2* /
			0x9000 0 0 1 &open-pic 3 1 / * INTA * /
			0x9000 0 0 2 &open-pic 4 1 / * INTB * /
			0x9000 0 0 3 &open-pic 1 1 / * INTC * /
			0x9000 0 0 4 &open-pic 2 1 / * INTD * /
		>;
	};
};
```
描述了一个open PIC中断控制器，并将其标识为具有interrupt-controller属性的中断控制器。

中断映射表中的每一行由五部分组成：**中断子节点单元地址**和**中断子节点中断说明符**，它映射到具有指定**中断父节点单元地址**和**中断父节点中断说明符**的**中断父节点**。

例如，interrupt-map表的第一行指定了插槽1的INTA的映射。该行的字段如下所示：

&emsp;&emsp;子节点单元地址: 0x8800 0 0
&emsp;&emsp;子节点中断说明符: 1
&emsp;&emsp;中断父节点: &open-pic
&emsp;&emsp;父节点单元地址: 空(因为open-pic中#address-cells = <0>)
&emsp;&emsp;父节点中断说明符: 2 1

* 中断子节点单元地址\<0x8800 0 0>。 该值由三个32位单元编码，该字段由PCI控制器的#address-cells属性确定（注：因为pci节点下中断产生设备可由软件可以动态识别所以不需要设备树描述，这些节点所在的总线就是pci）。 这三个单元代表如PCI总线的绑定（binding）所述的PCI地址。这个编码包括总线号 (0x0 << 16), 设备号(0x11 << 11), 功能号(0x0 << 8).
* 中断子节点中断说明符是\<1>，它指定PCI绑定（binding）所描述的INTA。 它需要一个32位单元，由PCI控制器的#interrupt-cells属性（值为1）指定，它是子中断域。
* 中断父节点由一个phandle指定，该phandle指向插槽的中断父节点，即Open PIC中断控制器。
* 中断父节点没有单元地址，因为父中断域（open-pic节点）的#address-cells值为\<0>。
* 中断父节点中断说明符是\<2 1>。 表示中断说明符（两个32位单元）的单元数由中断父节点（open-pic节点）上的#interrupt-cells属性决定。值\<2 1>是由Open PIC中断控制器的设备绑定指定的值（请参阅第4.5节）。 值\<2>指定INTA所连接的中断控制器上的物理中断源编号。 值\<1>指定 电平/敏感 编码。

在此示例中，interrupt-map-mask属性的值为\<0xf800 0 0 7>。 在中断映射表中执行查找之前，此掩码应用于子节点单元的中断说明符。

为查找INTB的IDSEL 0x12（插槽2），功能0x3的open-pic中断源编号，以下步骤将被执行：

* 子节点单元地址和中断说明符形成值\<0x9300 0 0 2>。
	* 地址的编码包括总线编号（0x0 << 16），设备编号（0x12 << 11）和功能编号（0x3 << 8）。
	* 中断说明符是2，它是根据PCI绑定的INTB编码。
	* 应用interrupt-map-mask值\<0xf800 0 0 7>，结果为\<0x9000 0 0 2>。
	* 在中断映射表中查找，映射到父中断说明符\<4 1>。

# 3 设备节点要求
## 3.1 基本设备节点类型

以下各节指定了DTSpec兼容设备树中所需的基本设备节点集的要求。

所有设备树都应具有根节点，并且所有设备树的根节点下应包含以下节点：
 * 一个/cpus 节点
* 至少一个 /memory 节点

## 3.2 根节点
设备树具有单个根节点，其中所有其他设备节点都是其后代。 根节点的完整路径是/。

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;表 3.1 根节点属性

|属性名|用法|值类型|定义|
|:--|:--|:--|:--|
|#address-cells| R| \<u32>| 指定\<u32>单元的数量，以表示根节点的子节点中reg属性中的地址。|
|#size-cells| R |\<u32>| 指定\<u32>单元的数量，以表示根节点的子节点中reg属性中的长度。|
|model| R| \<string>| 指定唯一标识系统主板模型的字符串。 推荐的格式是“manufacturer，model-number”。
|compatible| R| \<stringlist>| 指定与此平台兼容的平台体系结构列表。 操作系统可以在选择特定于平台的代码时使用此属性。推荐形式是：“manufacturer，model-number”，例如：compatible = "fsl,mpc8572ds"

>用法图例：R =必需，O =可选，OR =可选但推荐，SD =请参阅定义

>2.3节中的所有其它标准属性都是允许的，但是可选的。

## 3.3 /aliases节点

设备树可以有定义了一个或多个alias属性的别名节点（/aliases）。 别名节点应位于设备树的根目录，并且节点名为 /aliases。

/aliases节点的每个属性都定义了一个别名。 属性名称指定别名，属性值指定设备树中节点的完整路径。 例如，属性 serial0 = "/simple-bus@fe000000/serial@llc500"定义了别名serial0。

别名应以下字符集中小写文本字符，长为1到31。

![7](https://www.github.com/liao20081228/blog/raw/master/图片/设备树规范v0.2/7.JPG)

别名值是设备路径，并编码为字符串。 该值表示节点的完整路径，但该路径不需要引用叶节点。

客户端程序可以使用alias属性名将完整设备路径称为其字符串值的全部或部分。客户端程序在将字符串视为设备路径时，应（shall）检测并使用别名。

**示例**：
```dts
aliases {
serial0 = "/simple-bus@fe000000/serial@llc500";
ethernet0 = "/simple-bus@fe000000/ethernet@31c000";
};
```
给定别名serial0，客户程序可以查看/aliases节点并确定这个别名是指向设备路径/simple-bus@fe000000/serial@llc500。

## 3.4 /memory节点
所有设备树都需要memory设备节点，并描述系统的物理内存布局。 如果系统具有多个内存，则可以创建多个内存节点，或者可以在单个内存节点的reg属性中指定。

节点名称的unit-name（参见第2.2.1节）应（shall）为memory。

客户程序可以使用它选择的任何存储属性访问任何内存预留未覆盖的内存（请参阅第5.3节）。但是，在更改用于访问实际页面的存储属性之前，客户端程序负责执行体系结构和实现所需的操作，可能包括从缓存中刷新实际页面。

引导程序负责确保在不采取与存储属性更改相关的任何操作的情况下，客户端程序可以安全地访问所有内存（包括内存预留所覆盖的内存），如WIMG = 0b001x。 那是：
* 没有write through要求
* 没有缓存禁止
* 内存连续
* Required either not Guarded or Guarded

如果支持VLE存储属性，则VLE=0 。

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;表 3.3 /memory节点属性

|属性名|用法|值类型|定义|
|:--|:--|:--|:--|
|device_type |R |\<string>| 值为 “memory”|
|reg |R|\<prop-encoded-array>|由任意数量的（地址 大小）对组成，用于指定内存的物理地址和大小。
|initial-mapped-area| O |\<prop-encoded-array>| 指定初始映射区域的地址和大小，是由三元组（有效地址，物理地址，大小）组成的prop-encoded-array。有效和物理地址应为64位（\<u64>），大小应为32位（\<u32>）。

>用法图例：R =必需，O =可选，OR =可选但推荐，SD =请参阅定义

>2.3节中的所有其它标准属性都是允许的，但是可选的。

**示例**
给定具有以下物理内存布局的64位Power系统：

* RAM: 起始地址0x0, 长度 0x80000000 (2GB)
* RAM: 起始地址 0x100000000, 长度 0x100000000 (4GB)

memory节点可以定义如下，假设#address-cells = \<2>，#size-cells = \<2>。

**示例#1** 
```dts
memory@0 {
	device_type = "memory";
	reg = <0x000000000 0x00000000 0x00000000 0x80000000
	0x000000001 0x00000000 0x00000001 0x00000000>;
};
```
**示例#2** 
```dts
memory@0 {
	device_type = "memory";
	reg = <0x000000000 0x00000000 0x00000000 0x80000000>;
};
memory@100000000 {
	device_type = "memory";
	reg = <0x000000001 0x00000000 0x00000001 0x00000000>;
};
```

reg属性用于定义两个内存范围的地址和大小。 跳过了2GB I/O区域。请注意，根节点的#address-cells和#size-cells属性指定值2，这意味着需要两个32位单元来定义内存节点的reg属性的地址和长度。

## 3.5 /chosen节点

/chosen节点不是系统中的实际设备，而是描述可选参数或在运行时由系统固件指定的参数。 它应该是根节点的子节点。

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;表 3.4 /chosen节点属性

|属性名|用法|值类型|定义|
|:--|:--|:--|:--|
|bootargs| O |\<string>| 一个字符串，它指定客户程序的引导参数。 如果不需要引导参数，则该值可能是空字符串。
|stdout-path| O |\<string>| 一个字符串，指定要用于引导控制台输出的设备的节点的完整路径。 如果值中存在字符“：”，则它终止路径。 该值可以是别名。 如果未指定stdin-path属性，则应假定stdout-path定义输入设备。
|stdin-path| O| \<string> |一个字符串，指定要用于引导控制台输入的设备的节点的完整路径。 如果值中存在字符“：”，则它终止路径。 该值可以是别名。


>用法图例：R =必需，O =可选，OR =可选但推荐，SD =请参阅定义

>2.3节中的所有其它标准属性都是允许的，但是可选的。

**示例** 
```dts
chosen {
	bootargs = "root=/dev/nfs rw nfsroot=192.168.1.1 console=ttyS0,115200";
};
```


旧版本的设备树可能会遇到名为linux,stdout-path的属性，它是stdout-path属性的弃用形式。为了兼容性，如果不存在stdout-path属性，客户程序可能希望支持linux,stdout-path。 这两个属性的含义和用法是相同的。

## 3.6 /cpus节点

所有设备树都需要一个/cpus节点。 它不代表系统中的真实设备，而是充当系统CPU的子CPU节点的容器。

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;表 3.5 /cpus节点属性

|属性名|用法|值类型|定义|
|:--|:--|:--|:--|
|#address-cells| R| \<u32> |该值指定此节点的子节点中的reg属性的地址占用的单元格数。
|#size-cells| R |\<u32>| 值必须（shall）为0. 指定此节点的子节点中的reg属性不需要大小。

>用法图例：R =必需，O =可选，OR =可选但推荐，SD =请参阅定义

>2.3节中的所有其它标准属性都是允许的，但是可选的。

/cpus节点可能包含多个cpu节点的相同的的属性。 详情请参阅第3.7节。 有关示例，请参见第3.8.1节。

## 3.7 /cpus/cpu\*节点

cpu节点表示一个足够独立的硬件执行块，它能够运行操作系统而不会干扰可能运行其他操作系统的其他CPU。

共享MMU的硬件线程通常表示在一个cpu节点下。 如果设计了其他更复杂的CPU拓扑图，则CPU的绑定（binding）必须描述拓扑图（例如，不共享MMU的线程）。

CPU和线程通过统一的数字空间编号，该空间应尽可能地匹配CPU/线程的中断控制器的编号。

跨cpu节点具有相同值的属性可以放在/cpus节点中。客户程序必须首先检查特定的cpu节点，但是如果找不到期望的属性，那么它应该查看/cpus节点。 这导致在所有CPU中相同的属性使用更少的表述。

每个CPU节点的node-name应为cpu。

### 3.7.1 /cpus/cpu\*节点的普通属性

下表描述了cpu节点的普通属性。 表3.6中描述的一些属性是具有特定适用细节的标准属性。

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;表 3.6  /cpus/cpu\*节点的普通属性

|属性名|用法|值类型|定义|
|:--|:--|:--|:--|
|device_type|R |\<string>|值必须为“cpu”。|
|reg |R |\<prop-encoded-array>|reg的值是\<prop-encoded-array>，它为CPU节点表示的CPU/线程定义了 唯一CPU/线程ID。<br /><br />如果CPU支持多个线程（即多个执行流），则reg属性是每线程1个元素的数组。 /cpus节点上的#address-cells指定数组中每个元素占用的单元格(cell)数。软件可以通过将reg的大小除以/cpus的#address-cells来确定线程数。<br /><br />如果CPU/线程可以作为外部中断的目标，则reg属性值必须是可由中断控制器寻址的唯一CPU/线程ID。<br /><br />如果CPU /线程不能成为外部中断的目标，则reg必须是唯一的并且超出中断控制器寻址的范围。<br /><br />如果CPU/线程的PIR（挂起中断寄存器）是可修改的，则客户程序应（should）修改PIR以匹配reg属性值。如果PIR不能修改并且PIR值与中断控制器编号空间不同，则CPU绑定可以根据需要定义绑定特定表示的PIR值。
|clock-frequency|R |\<prop-encoded-array>|以赫兹为单位指定CPU的当前时钟速度。 该值是两种形式之一的\<prop-encoded-array>：<br />&emsp;&emsp;•  一个\<u32>。<br />&emsp;&emsp;•  一个\<u64>
|timebase-frequency|R |array|指定更新时基（timebase register）寄存器和递减器寄存器（decrementer registers ）的当前频率（以赫兹为单位）。 该值是两种形式之一的\<prop-encoded-array>：<br />&emsp;&emsp;•  一个\<u32>。<br />&emsp;&emsp;•  一个\<u64>
|status| SD| \<string> |描述CPU状态的标准属性。 对于对称多处理（SMP）配置中的CPU的节点，此属性应存在。对于CPU节点，“okay”和“disabled”值的含义如下：<br />&emsp;&emsp;• “okay”：CPU正在运行。<br />&emsp;&emsp;• “disabled”：CPU处于静止状态。<br />静止CPU是一种状态，它不会干扰其他CPU正常操作，也不会被其它正在运行的CPU的正常操作影响，除非用显式方法 启用或重新启用静态CPU（请参阅 enable-method属性）。<br />特别的，正在运行的CPU应能够在不影响静态CPU的情况下发出广播TLB无效。<br />示例：静态CPU可以处于自旋循环中，保持在复位状态，并与系统总线或其他实现相关状态电隔离。
|enable-method|SD |\<stringlist>|描述启用disabled状态的CPU的方法。 具有值为“disabled”的status属性的CPU需要此属性。该值由一个或多个字符串组成，这些字符串定义释放此CPU的方法。 如果客户程序能识别任何一种方法，则可以使用它。值应为以下之一：<br />&emsp;&emsp;“spin-table”：使用DT-Spec中定义的自旋表（spin table）方法启用CPU。<br />&emsp;&emsp;"[vendor],[method]" :实现取决于描述CPU从“disabled”状态释放的方法的字符串。所需格式为：“[vendor]，[method]”，其中vendor是描述制造商名称的字符串，method是描述供应商特定机制的字符串。示例：“fsl，MPC8572DS”<br /><br />注意：其他方法可能会添加到DTSpec规范的后续版本中。
|cpu-release-addr|SD| \<u64>|enable-method属性的值为“spin-table”的cpu节点需要cpu-release-addr属性。该值指定从其自旋循环释放次级（secondary）CPU的自旋表条目的物理地址。



>用法图例：R =必需，O =可选，OR =可选但推荐，SD =请参阅定义

>2.3节中的所有其它标准属性都是允许的，但是可选的。




&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;表 3.7  /cpus/cpu\*节点的Power ISA 属性

|属性名|用法|值类型|定义|
|:--|:--|:--|:--|
|power-isa-version|O| \<string>|一个字符串，指定Power ISA版本字符串的数字部分。 例如，对于符合Power ISA版本2.06的实现，此属性的值将为“2.06”。
|power-isa-* |O |\<empty>|如果power-isa-version属性存在，则对于显示的Power ISA版本的Book I的Categories部分中的每个类别，存在名为power-isa-[CAT]的属性，其中[CAT]是将所有大写字母转换为小写的类别名缩写，表示该实现支持该类别。<br /> <br />例如，如果power-isa-version属性存在且其值为“2.06”且power-isa-e.hv属性存在，则该实现支持Power ISA版本2.06中定义的[Category:Embedded.Hypervisor]。
|cache-op-block-size|SD |\<u32>|指定高速缓存块指令操作的块大小（以字节为单位）（例如，dcbz）。 如果与L1高速缓存块大小不同，则为必需。
|reservation-granule-size|SD |\<u32>|指定此处理器支持的预留粒度大小（以字节为单位）。
|mmu-type |O |\<string> |指定cpu的MMU类型，合法的值如下： "mpc8xx"、"ppc40x"、 "ppc440"、 "ppc476"、 "power-embedded"、 "powerpc-classic"、 "power-server-stab"、 "power-server-slb"、 "none"


>用法图例：R =必需，O =可选，OR =可选但推荐，SD =请参阅定义

>2.3节中的所有其它标准属性都是允许的，但是可选的。

旧版本的设备树，可能会在cpu节点中遇到bus-frequency 属性。 为了兼容性，客户程序可能希望支持bus-frequency。 该值的格式与clock-frequency的格式相同。 建议的做法是使用clock-frequency属性表示总线节点上总线频率。


### 3.7.2 TLB属性
cpu节点的以下属性描述了处理器MMU中的转换后备（translate look-aside）缓冲区。


&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;表 3.8  /cpus/cpu\*节点的Power ISA TLB属性


|属性名|用法|值类型|定义|
|:--|:--|:--|:--|
|tlb-split| SD|\<empty>|如果该属性存在，则指定TLB具有分开的配置用于指令和数据的独立TLB。 如果不存在，则指定TLB具有统一配置。对于具有用于分离配置TLB的CPU，该属性是必需的|
|tlb-size| SD |\<u32>| 指定TLB中的条目数。 对于具有用于指令和数据地址的统一TLB的CPU，该属性是必需的。|
|tlb-sets |SD| 	\<u32>| 指定TLB中的关联集的数量。对于具有用于指令和数据地址的统一TLB的CPU，该属性是必需的。|
|d-tlb-size| SD |	\<u32> |指定数据TLB中的条目数。对于具有用于分离配置TLB的CPU，该属性是必需的
|d-tlb-sets |SD| 	\<u32>| 指定数据TLB中的关联集的数量。对于具有用于分离配置TLB的CPU，该属性是必需的|
|i-tlb-size| SD |	\<u32> |指定指令TLB中的条目数。对于具有用于分离配置TLB的CPU，该属性是必需的
|i-tlb-sets |SD| 	\<u32>| 指定指令TLB中的关联集的数量。对于具有用于分离配置TLB的CPU，该属性是必需的|

>用法图例：R =必需，O =可选，OR =可选但推荐，SD =请参阅定义

>2.3节中的所有其它标准属性都是允许的，但是可选的。

### 3.7.3 内存cache（L1）属性
cpu节点的以下属性描述了处理器的内部（L1）缓存。


&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;表 3.9  /cpus/cpu\*节点的Power ISA cache属性



|属性名|用法|值类型|定义|
|:--|:--|:--|:--|
|cache-unified| SD| \<empty>|如果存在该属性，则指定缓存具有统一的组织（unified organization）。 如果不存在，则指定缓存具有哈佛架构，哈佛架构具有用于指令和数据的单独缓存。|
|cache-size| SD| \<u32> |指定统一高速缓存的大小（以字节为单位）。如果缓存是统一的（组合指令和数据），则该属性必需。|
|cache-sets| SD |\<u32> |指定统一高速缓存中的关联集的数量。如果缓存是统一的（组合指令和数据），则该属性必需。|
|cache-block-size |SD |\<u32>|指定统一高速缓存的块大小（以字节为单位）。如果缓存是统一的（组合指令和数据），则该属性必需。|
|cache-line-size |SD |\<u32>|如果不同于高速缓存块大小，则指定统一高速缓存的行大小（以字节为单位）如果处理器具有统一缓存（组合指令和数据），则为必需。|
|i-cache-size| SD| \<u32> |指定指令缓存的大小（以字节为单位）。如果处理器具有独立的指令缓存，则为必需。|
|i-cache-sets| SD |\<u32> |指定指令缓存中的关联集的数量。如果处理器具有独立的指令缓存，则为必需。|
|i-cache-block-size |SD |\<u32>|指定指令缓存的块大小（以字节为单位）。如果处理器具有独立的指令缓存，则为必需。|
|i-cache-line-size |SD |\<u32>|如果不同于指令高速缓存块大小，则指定指令缓存的行大小（以字节为单位）。如果处理器具有独立的指令缓存，则为必需。|
|d-cache-size| SD| \<u32> |指定数据缓存的大小（以字节为单位）。如果处理器具有独立的数据缓存，则为必需。|
|d-cache-sets| SD |\<u32> |指定数据缓存中的关联集的数量。如果处理器具有独立的数据缓存，则为必需。|
|d-cache-block-size |SD |\<u32>|指定数据缓存的块大小（以字节为单位）。如果处理器具有独立的数据缓存，则为必需。|
|d-cache-line-size |SD |\<u32>|如果不同于数据高速缓存块大小，则指定数据缓存的行大小（以字节为单位）。如果处理器具有独立的数据缓存，则为必需。|
|next-level-cache |SD |\<phandle>|如果该属性存在，则表示存在另一级别的缓存。 该值是下一级缓存的phandle。 phandle值类型在2.3.3节中有详细描述。|

>用法图例：R =必需，O =可选，OR =可选但推荐，SD =请参阅定义

>2.3节中的所有其它标准属性都是允许的，但是可选的。


可能遇到旧版本的设备树，其中包含名为l2-cache的next-level-cache属性的弃用形式。 为了兼容性，如果不存在next-level-cache属性，则客户端程序可能希望支持l2-cache。 这两个属性的含义和用法是相同的。

### 3.7.4 示例

```dts
cpus {
	#address-cells = <1>;
	#size-cells = <0>;
	cpu@0 {
		device_type = "cpu";
		reg = <0>;
		d-cache-block-size = <32>; // L1 - 32 bytes
		i-cache-block-size = <32>; // L1 - 32 bytes
		d-cache-size = <0x8000>; // L1, 32K
		i-cache-size = <0x8000>; // L1, 32K
		timebase-frequency = <82500000>; // 82.5 MHz
		clock-frequency = <825000000>; // 825 MHz
	};
};
```



## 3.8 多级和共享缓存节点(/cpus/cpu * /l?-cache)

处理器和系统可以实现额外级别的缓存层次结构。 例如，第二级（L2）或第三级（L3）高速缓存。 这些高速缓存可以紧密集成到CPU中，也可以在多个CPU之间共享。

compatile值为“cache”的设备节点描述了这些类型的高速缓存。

缓存节点应定义一个phandle属性，并且关联和或共享这些缓存的所有cpu节点或缓存节点都应包含一个next-level-cache属性，该属性指定指向这些缓存节点的phandle。

缓存节点可以出现在CPU节点或设备树中的任何其他适当位置。

多级和共享高速缓存使用表3-9中的属性表示。 L1缓存的属性在表3-8中描述。


&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;表 3.10  /cpu/cpu\*/l?-cache节点Power ISA 多级和共享缓存节点属性


|属性名|用法|值类型|定义|
|:--|:--|:--|:--|
|compatible| R |\<string> |标准属性。 值应必须包括字符串“cache”。
|cache-level |R |\<u32> |指定缓存层次结构中的级别。 例如，L2级缓存的值为2。


>用法图例：R =必需，O =可选，OR =可选但推荐，SD =请参阅定义

>2.3节中的所有其它标准属性都是允许的，但是可选的。

### 3.8.1 示例
请参阅以下两个CPU的设备表示的示例，每个CPU都有自己的L2缓存和共享的L3缓存。
```dts
cpus {
	#address-cells = <1>;
	#size-cells = <0>;
	cpu@0 {
		device_type = "cpu";
		reg = <0>;
		cache-unified;
		cache-size = <0x8000>; // L1, 32KB
		cache-block-size = <32>;
		timebase-frequency = <82500000>; // 82.5 MHz
		next-level-cache = <&L2_0>; // phandle to L2
		L2_0:l2-cache {
			compatible = "cache";
			cache-unified;
			cache-size = <0x40000>; // 256 KB
			cache-sets = <1024>;
			cache-block-size = <32>;
			cache-level = <2>;
			next-level-cache = <&L3>; // phandle to L3
			L3:l3-cache {
				compatible = "cache";
				cache-unified;
				cache-size = <0x40000>; // 256 KB
				cache-sets = <0x400>; // 1024
				cache-block-size = <32>;
				cache-level = <3>;
			};
		};
	};
	cpu@1 {
		device_type = "cpu";
		reg = <1>;
		cache-unified;
		cache-block-size = <32>;
		cache-size = <0x8000>; // L1, 32KB
		timebase-frequency = <82500000>; // 82.5 MHz
		clock-frequency = <825000000>; // 825 MHz
		next-level-cache = <&L2_1>; // phandle to L2
		L2_1:l2-cache {
			compatible = "cache";
			cache-level = <2>;
			cache-unified;
			cache-size = <0x40000>; // 256 KB
			cache-sets = <0x400>; // 1024
			cache-line-size = <32>; // 32 bytes
			next-level-cache = <&L3>; // phandle to L3
		};
	};
};
```

# 4 设备绑定

本章包含在设备树中有关如何表示特定类型和类的设备的要求（称为绑定）。 设备节点的compatible属性描述节点需要遵守的特定绑定。

绑定可以定义为其他每个的扩展。 例如，可以将新的总线类型定义为简单总线绑定的扩展。 在这种情况下，compatible属性将包含几个标识每个绑定的字符串 —— 从最具体到最一般（参见第2.3.1节，compatible）。

## 4.1绑定指南

### 4.1.1通用原则

为设备创建新的设备树时，完全描述设备必要属性和值得绑定也应该创建。这组属性应充分描述以为设备驱动程序提供需要的设备属性。

一些推荐的做法包括：


1. 使用2.3.1节中描述的约定兼容字符串。
2. 使用适用于新设备的标准属性（在2.3和2.4节中定义）。 此用法通常至少包括reg和interrupt属性。
3. 如果新设备符合DTSpec定义的设备类，请使用第4节（设备绑定）中指定的约定。
4. 如果适用，使用4.1.2节中指定的各种属性约定。
5. 如果绑定需要新属性，则建议的属性名称格式为：“\<company>，\<property-name>“，其中\<company>是OUI或短的唯一字符串，比如股票代码，用于标识绑定的创建者。示例：“ibm，ppc-interrupt-server＃s”

### 4.1.2 杂项属性
本节定义了可能适用于许多类型和类的设备的有用属性列表。 在此定义为便于其名称和用法的标准化。

**clock-frequency 属性** 

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;表 4.1 clock-frequency属性

|属性名|值类型|描述|
|:--|:--|:--|
|clock-frequency|\<prop-encoded-array>|以Hz为单位指定时钟频率。 该值是以两种形式之一的<prop-encoded-array>：<br />(1)32位整数；(2)64位整数。



**reg-shift属性**

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;表 4.2 reg-shift属性

|属性名|值类型|描述|
|:--|:--|:--|
|reg-shift|\<u32>|reg-shift属性提供了一种机制来表示除了寄存器之间的字节数不同之外在大多数方面相同的设备。reg-shift属性以字节为单位指定离散设备寄存器彼此分开的距离。 使用以下公式计算各个寄存器位置：“register << reg-shift“。 如果未指定，则默认值为0。例如，在16540 UART寄存器位于地址0x0,0x4,0x8,0xC，0x10,0x14,0x18和0x1C的系统中，将使用reg-shift = <2>属性来指定寄存器位置。

**label属性**

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;表 4.3 label属性

|属性名|值类型|描述|
|:--|:--|:--|
|label|\<string>|label属性定义人类可读的描述设备的字符串。给定设备的绑定指定该设备的属性的确切含义。|

## 4.2 串口设备
### 4.2.1 串口类绑定
串行设备类包括各种类型的点对点串行线设备。 串行线路设备的示例包括8250 UART，16550 UART，HDLC 设备和BISYNC设备。在大多数情况下，与RS-232标准兼容的硬件适合串行设备类。

I^2^C和SPI（串行外设接口）设备不应表示为串行端口设备，因为它们具有自己的特定表示。

**clock-frequency属性**


&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;表 4.4 clock-frequency属性

|属性名|值类型|描述|示例|
|:--|:--|:--|:--|
|clock-frequency|\<u32>|指定波特率发生器输入时钟的频率（以赫兹为单位）。| clock-frequency = \<100000000>;


**current-speed属性** 

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;表 4.5 current-speed属性

|属性名|值类型|描述|示例|
|:--|:--|:--|:--|
|current-speed|\<u32>|说明以每秒位数指定串行设备的当前速度。 如果已初始化串行设备，则引导程序应设置此属性。|115,200 Baud: current-speed = <115200>;







### 4.2.2 NS16450/16550兼容UART

与National Semiconductor 16450/16550 UART（通用异步接收器发送器）兼容的串行设备应使用以下属性在设备中表示。

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;表 4.6 ns16550 UART属性


|属性名|用法|值类型|定义|
|:--|:--|:--|:--|
|compatible |R |\<string list> |值必须包括“ns16550”|
|clock-frequency |R |\<u32> |指定波特率发生器输入时钟的频率（以Hz为单位）|
|current-speed| OR |\<u32> |指定当前串行设备速度，以每秒位数为单位
|reg |R |\<prop-encode-darray>|指定父总线地址空间内寄存器设备的物理地址
|interrupts |OR |\<prop-encoded-array>|指定此设备生成的中断。 interruptts属性的值由一个或多个中断说明符组成。 中断说明符的格式由节点中断父节点的绑定文档定义。
|reg-shift |O|\<u32> |reg-shift属性以字节为单位指定离散设备寄存器彼此分开的距离。 使用以下公式计算各个寄存器位置：“register << reg-shift“。 如果未指定，则默认值为0。
|virtual-reg |SD |\<u32> or \<u64> |见2.3.7节。 指定映射到reg属性中指定的第一个物理地址的有效地址。 如果此设备节点是系统的控制台，则此属性是必需的。

>用法图例：R =必需，O =可选，OR =可选但推荐，SD =请参阅定义

>2.3节中的所有其它标准属性都是允许的，但是可选的。

## 4.3 网络设备
网络设备是面向分组的通信设备。 该类中的设备假设实现七层OSI模型的数据链路层（第2层）并使用媒体访问控制（MAC）地址。 网络设备的示例包括以太网，FDDI，802.11和令牌环。

### 4.3.1 网络类设备绑定


**address-bits属性**
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;表 4.7 address-bits属性

|属性名|值类型|描述|示例|
|:--|:--|:--|:--|
|address-bits|\<u32>|指定寻址此节点描述的设备所需的地址位数。 此属性指定MAC地址中的位数。 如果未指定，则默认值为48。|address-bits = \<48>;

**local-mac-address属性**
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;表 4.8 local-mac-address属性

|属性名|值类型|描述|示例|
|:--|:--|:--|:--|
|local-mac-address|\<prop-encoded-array> 编码为16进制数组|指定分配给包含此属性的节点描述的网络设备的MAC地址。|local-mac-address = [ 0x00 0x00 0x12 0x34 0x56 0x78];


**mac-address属性**  
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;表 4.9 mac-address属性

|属性名|值类型|描述|示例|
|:--|:--|:--|:--|
|mac-address|\<prop-encoded-array> 编码为16进制数组|指定引导程序上次使用的MAC地址。 如果引导程序为设备分配的MAC地址与local-mac-address属性不同，则应使用此属性。 仅当值与local-mac-address属性值不同时，才应使用此属性。|mac-address = [ 0x00 0x00 0x12 0x34 0x56 0x78];


**max-frame-size属性**  
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;表 4.10 max-frame-size属性

|属性名|值类型|描述|示例|
|:--|:--|:--|:--|
|max-frame-size|\<u32> |指定物理接口可以发送和接收的最大数据包长度（以字节为单位）。|max-frame-size = \<1518>;



### 4.3.2 以太网特定注意事项

除了指定网络设备类的属性之外，基于LAN标准的IEEE 802.3集合（统称为以太网）的网络设备可以使用以下属性在设备树中表示。

本节中列出的属性会扩充网络设备类中列出的属性。

**max-speed属性**  
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;表 4.11 max-speed属性

|属性名|值类型|描述|示例|
|:--|:--|:--|:--|
|max-speed|\<32>|指定设备支持的最大速度（以Mb/s为单位）。|max-speed = \<1000>;|


**max-speed属性**  
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;表 4.12 phy-connection-type属性

|属性名|值类型|描述|示例|
|:--|:--|:--|:--|
|phy-connection-type|  \<string>|指定以太网设备和物理层（PHY）设备之间的接口类型。 此属性的值特定于实现。建议值如下表所示。|phy-connection-type = "mii";

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;表 4.13 phy-connection-type属性定义的值
|连接类型|值|
|:--|:--|
|Media Independent Interface| mii|
|Reduced Media Independent Interface |rmii|
|Gigabit Media Independent Interface |gmii|
|Reduced Gigabit Media Independent |rgmii|
|rgmii with internal delay| rgmii-id|
|rgmii with internal delay on TX only| rgmii-txid|
|rgmii with internal delay on RX only |rgmii-rxid|
|Ten Bit Interface| tbi|
|Reduced Ten Bit Interface| rtbi|
|Serial Media Independent Interface |smii|
|Serial Gigabit Media Independent Interface| sgmii|
|Reverse Media Independent Interface |rev-mii|
|10 Gigabits Media Independent Interface |xgmii|
|Multimedia over Coaxial |moca|
|Quad Serial Gigabit Media Independent Interface |qsgmii
|Turbo Reduced Gigabit Media Independent Interface| trgmii|


**phy-handle属性**  
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;表 4.14  phy-handle属性

|属性名|值类型|描述|示例|
|:--|:--|:--|:--|
|phy-handle|\<phandle>|指定对表示连接到此以太网设备的物理层（PHY）设备的节点的引用。如果以太网设备连接到物理层设备，则需要此属性。|phy-handle = \<&PHY0>;|



## 4.4 Power ISA开放式PIC中断控制器

本节规定了表示Open PIC兼容中断控制器的要求。 Open PIC中断控制器实现Open PIC架构（由AMD和Cyrix联合开发），并在开放式可编程中断控制器（PIC）寄存器接口规范修订版1.2 [[b18]](#b18)中指定。



Open PIC中断域中的中断说明符由两个单元（cell）编码。 第一个单元定义中断号。第二个单元定义了敏感和电平信息。

敏感和电平信息必须（shall）在中断说明符中按如下方式编码：

&emsp;&emsp;0 = 上升沿敏感
&emsp;&emsp;1 = 低电平敏感
&emsp;&emsp;2 = 高电平敏感
&emsp;&emsp;3 = 下降沿敏感

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;表 4.15 open-pic属性


|属性名|用法|值类型|定义|
|:--|:--|:--|:--|
|compatible| R |\<string> |值必须（shall）包含 "open-pic"|
|reg |R|\<prop-encoded-array>|指定父总线地址空间内寄存器设备的物理地址|
|interrupt-controller |R|\<empty> |指定此节点是中断控制器|
|#interrupt-cells |R| \<u32> |值必须（shall）为 2|
|#address-cells |R| \<u32> |值必须（shall）为 0|

>用法图例：R =必需，O =可选，OR =可选但推荐，SD =请参阅定义

>2.3节中的所有其它标准属性都是允许的，但是可选的。

## 4.5 simple-bus兼容值
片上系统处理器可能具有无法探测的内部I/O总线。 无需额外配置即可直接访问总线上的设备。 这种类型的总线表示为compatible为值“simple-bus”的节点。


&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;表 4.16 simple-bus兼容节点属性


|属性名|用法|值类型|定义|
|:--|:--|:--|:--|
|compatible| R |\<string> |值必须（shall）包含 "simple-bus"|
|ranges| R| \<prop-encoded-array>|此属性表示父地址与子地址空间之间的映射（请参阅第2.3.8节，ranges）。|


>用法图例：R =必需，O =可选，OR =可选但推荐，SD =请参阅定义

>2.3节中的所有其它标准属性都是允许的，但是可选的。

# 5 扁平设备树（DTB）格式

设备树Blob（DTB）格式是设备树数据的平面二进制编码。 它用于在软件程序之间交换设备树数据。 例如，在引导操作系统时，固件会将DTB传递给OS内核。

>注意：IEEE1275开放固件[IEEE1275]未定义DTB格式。 在大多数符合Open Firmware的平台上，通过调用固件方法来提取设备树以遍历树结构。


DTB格式在单个线性无指针数据结构内对设备树数据进行编码。 它由一个header（见5.2节）和三个可变大小的部分组成：memory reservation block（见5.3节），structure block（见第5.4节）和strings block（见5.5节）。 这些应按顺序存在于扁平设备树中。 因此，当按地址处加载到内存中时，整个设备树结构将类似于图5.1中的图（较低地址位于图的顶部）。

![8](https://www.github.com/liao20081228/blog/raw/master/图片/设备树规范v0.2/8.JPG)

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;图5.1 设备树.dtb结构

free space 部分可能不存在，但在某些情况下，它们可能需要满足各个块的对齐约束（参见第5.6节）。


## 5.1 版本控制
自DTB格式的原始定义以来，已经定义了几种版本的扁平设备树结构。 header中的字段给出版本，以便客户程序可以确定设备树是否以兼容格式编码。

本文档仅描述DTB格式的第17版。 符合DTSpec的引导程序应提供版本17或更高版本的设备树，并应提供与版本16向后兼容的版本的设备树。符合DTSpec标准的客户端程序应接受与版本17向后兼容的任何版本的设备树，并且也可以接受其他版本。

注意：版本是关于设备树的二进制结构，而不是其内容。

## 5.2 header
设备树的header布局由以下C结构定义。 所有头字段都是32位整数，以大端格式存储。

```c
struct fdt_header {
	uint32_t magic;
	uint32_t totalsize;
	uint32_t off_dt_struct;
	uint32_t off_dt_strings;
	uint32_t off_mem_rsvmap;
	uint32_t version;
	uint32_t last_comp_version;
	uint32_t boot_cpuid_phys;
	uint32_t size_dt_strings;
	uint32_t size_dt_struct;
};
```
**magic** 该字段应包含值0xd00dfeed（大端模式）。

**totalsize** 该字段必须包含设备树数据结构的总大小（以字节为单位）。 该大小应包含结构的所有部分：头部，内存保留块，结构块和字符串块，以及块之间或最终块之后的任何空闲内存区。

**off_dt_struct** 该字段必须包含从头部的起始位置开始计算的结构块的偏移量（参见5.4节）。

**off_dt_strings** 该字段必须包含从头部的起始位置开始计算的字符串块的偏移量（参见5.5节）。

**off_mem_rsvmap** 该字段必须包含从头部的起始位置开始计算的内存保留块的偏移量（参见5.3节）

**version** 该字段必须包含设备树数据结构的版本。 如果使用本文档中定义的结构，则版本为17。 DTSpec引导程序可以提供更高版本的设备树，在这种情况下，该字段应包含在后面的文档中定义的版本号，该文档给出该版本的详细信息。

**last_comp_version** 该字段必须包含当前设备树数据结构可以兼容的最低版本。 因此，对于本文档（版本17）中定义的结构，此字段应包含16，因为版本17兼容版本16，但不兼容早期版本。 根据5.1节，DTSpec引导程序应提供与版本16兼容的格式的设备树，因此该字段应始终包含16。

**boot_cpuid_phys**  该字段必须包含系统引导CPU的物理ID。 它应与设备树中该CPU节点的reg属性中给出的物理ID相同。

**size_dt_strings** 该字段应包含设备树的字符串块部分的字节长度。

**size_dt_struct**  该字段应包含设备树的结构块部分的字节长度。



## 5.3 内存保留块
### 5.3.1 目的

内存保留块为客户程序提供了一个保留的物理内存的列表; 也就是说，它不能用于一般的内存分配。它用于保护重要数据结构不被客户程序覆盖。例如，在某些具有IOMMU的系统上，需要以这种方式保护由DTSpec引导程序初始化的TCE（转换控制条目）表。同样，需要保留在客户端程序运行时期间使用的任何引导程序代码或数据（例如，Open Firmware平台上的RTAS）。DTSpec不要求引导程序提供任何此类运行时组件，但它不禁作为扩展实现。


更具体地说，客户端程序不应访问保留区域中的内存，除非引导程序提供的其他信息明确指示它应该这样做。然后，客户程序可以以指示的方式访问保留内存的指示部分。引导程序向客户机程序指示预留内存的特定用途的方法可以出现本文档、可选的扩展或特定于平台的文档中。

>原文为：More specifically, a client program shall not access memory in a reserved region unless other information provided by the boot program explicitly indicates that it shall do so. The client program may then access the indicated section of the reserved memory in the indicated manner. Methods by which the boot program can indicate to the client program specific uses for reserved memory may appear in this document, in optional extensions to it, or in platform-specific documentation.

由引导程序提供的保留区域可以但不是必须包含设备树二进制本身。 客户程序应确保在使用之前不会覆盖此数据结构，无论它是否在保留区域内。

在内存节点中声明并由引导程序访问或在客户程序进入后由引导程序访问的任何内存都必须保留。 这种类型的访问的示例包括（例如，通过非保护虚拟页面的推测性内存读取）。

此要求是必要的，因为具有任意存储属性的客户程序可以访问未保留的任何内存。


引导程序对保留内存的访问或由引导程序造成的对保留内存的访问必须被完成，就像没有缓存禁止和内存一致性要求一样，另外对于Book III-S实现，就像没有Write Through要求（即WIMG = 0b001x）。此外，如果支持VLE存储属性，则必须以VLE = 0完成对保留存储器的所有访问。
>原文为： Any accesses to reserved memory by or caused by the boot program must be done as not Caching Inhibited and Memory Coherence Required (i.e., WIMG = 0bx01x), and additionally for Book III-S implementations as not Write Through Required (i.e., WIMG = 0b001x). 

 此要求是必要的，因为允许客户端程序映射内存，其存储属性指定为没有Write Through要求，没有缓存禁止和内存一致性要求（即WIMG = 0b001x），并且支持VLE = 0。客户程序可以使用包含保留内存的大型虚拟页面，但是，客户端程序可能不会修改保留的内存，因此引导程序可以执行对保留内存的访问，如同Write Through Required一样，其中此存储属性的冲突值在架构上是允许的。
  
 >原文为：This requirement is necessary because the client program is permitted to map memory with storage attributes specified as not Write Through Required, not Caching Inhibited, and Memory Coherence Required (i.e., WIMG = 0b001x), and VLE=0 where supported. The client program may use large virtual pages that contain reserved memory. However, the client program may not modify reserved memory, so the boot program may perform accesses to reserved memory as Write Through Required where conflicting values for this storage attribute are architecturally permissible.
  
 ### 5.3.2 格式
 
 内存保留块由64位大端整数的对（pair）列表组成，每对由下面的C结构表示。
 ```c
struct fdt_reserve_entry {
	uint64_t address;
	uint64_t size;
};
 ```
每对（pair）给出保留内存区域的物理地址和大小（以字节为单位）。 这些给定区域不应相互重叠。 保留块列表应以地址和大小均等于0的条目终止。请注意，地址和大小值始终为64位。 在32位CPU上，忽略该值的高32位。

内存保留块中的每个uint64_t以及整个内存保留块应位于距设备树二进制开头的8字节对齐的偏移处（参见5.6节）。

## 5.4 结构块
结构块描述了设备树本身的结构和内容。 它由如下所述的一系列带有数据的标记组成。 它们被组织成线性树结构，如下所述。

结构块中的每个标记，以及结构块本身，应位于距设备树二进制开头的4字节对齐的偏移处（见5.6）。

### 5.4.1 词汇结构

结构块由一系列片段组成，每个片段以标记开头，它是一个大端的32位整数。一些标记后面跟着额外的数据，其格式由标记值决定。 所有标记都应在32位边界上对齐，这可能需要在前一个标记的数据之后插入填充字节（值为0x0）。 五种令牌类型如下：

**FDT_BEGIN_NODE (0x00000001)** FDT_BEGIN_NODE表示节点的开始。它后面必须（shall）是节点的单元名称作为额外数据。该名称存储为以空字符结尾的字符串，并且应包括单元地址（请参阅第2.2.1节）（如果有）。如果需要进行对齐，则节点名称后跟填充的零填充字节，然后是下一个标记，可以是除FDT_END之外的任何标记。

**FDT_END_NODE (0x00000002)** FDT_END_NODE表示节点的结束。 此标记没有额外数据; 所以紧接着是下一个家伙，可以是除FDT_PROP之外的任何标记。

**FDT_PROP (0x00000003)** FDT_PROP表示设备树中一个属性的开始。 随后必须（shall）有描述该属性的额外数据。 此数据首先包含属性的长度和名称，表示为以下C结构：
```c
struct {
	uint32_t len;
	uint32_t nameoff;
}
```
这个结构中的两个字段都是32位大端整数。
* len 给出了属性值的长度（以字节为单位）（可以为零，表示空属性，请参见第2.2.4.2节）。
* nameoff 给出了在字符串块中的偏移量（参见第5.5节），在字符串块中，属性的名称存储为以空字符结尾的字符串。

在此结构之后，属性的值以长度为len的字符串给出。 该值之后是零填充字节（如果需要）以对齐到下一个32位边界，然后是下一个标记，可以是除FDT_END之外的任何标记。

**FDT_NOP (0x00000004)** FDT_NOP将被解析设备树的任何程序忽略。此标记没有额外数据; 所以紧接着是下一个标记，它可以是任何有效的标记。可以使用FDT_NOP覆盖树中定义的属性或节点，以将其从设备树中移除，而无需在设备树二进制中移动设备树的表示的其他部分。

**FDT_END (0x00000009)** FDT_END标记结构块的结尾。一个设备树中只有一个FDT_END标记，它应该是结构块中的最后一个标记。 它没有额外的数据; 所以紧接在FDT_END标记之后；所以从结构块的开头算起的紧接在FDT_END标记之后的字节的偏移量等于设备树二进制的头中的size_dt_struct字段的值。

### 5.4.2 树结构
设备树结构表示为线性树：每个节点的表示以FDT_BEGIN_NODE标记开始，以FDT_END_NODE标记结束。 节点的属性和子节点（如果有）在FDT_END_NODE之前表示，以至于这些子节点的FDT_BEGIN_NODE和FDT_END_NODE标记嵌套在父节点的FDT_BEGIN_NODE和FDT_END_NODE标记中。

整个结构块由根节点的表示（包含所有其他节点的表示）和 后跟一个标记结构块结束的FDT_END标记组成。

更确切地说，每个节点的表示由以下组件组成：

* (可选)任意数量的FDT_NOP标记
* FDT_BEGIN_NODE 标记
	*  节点的名称：以null结尾的字符串
	*   [零填充字节以对齐4字节边界]
* 对于节点的每个属性：
	* (可选)任意数量的FDT_NOP标记
	* FDT_PROP标记
		* 5.4.1节中给出的属性信息
		* [零填充字节以对齐4字节边界]
* 以此格式表示所有子节点
* (可选)任意数量的FDT_NOP标记
* FDT_END_NODE令牌

请注意，此过程要求特定节点的所有属性定义都在该节点的任何子节点定义之前。 虽然如果属性和子节点混合在一起，结构也不会模糊，但是这个要求简化了处理扁平树所需的代码。

## 5.5 字符串块

字符串块包含设备树中使用的所有属性名称的字符串。 这些以空终止的字符串在本块中简单地连接在一起，并结构块中的偏移进行引用。

字符串块没有对齐约束，可能出现在距设备树二进制开头的任何偏移处。

## 5.6 对齐

对于没有未对齐内存访问的内存保留和结构块中的数据，它们应该位于适当对齐的内存地址。具体地说，内存保留块必须与8字节边界对齐，结构块必须与4字节边界对齐。

此外，可以重定位设备树二进制而不破坏子块的对齐。

如前面部分所述，结构和字符串块应与设备树二进制的开头具有对齐的偏移量。
为了确保块的内存中对齐，足以确保整个设备树被加载到与任何子块的最大对齐对齐的地址，即8字节边界。

为了确保内存中块的对齐，整个设备树加载到与所有子块中的最大对齐字节边界对齐的地址，即8字节边界就足够了。符合DTSpec标准的引导程序应在将设备树blob传递给客户端程序之前将其加载到这样的对齐地址。 如果DTSpec客户程序将设备树二进制重新定位到内存中，则它应该只对另一个8字节对齐的地址执行此操作。

# 6 扁平设备树（DTS）格式（版本1）

设备树源（DTS）格式是设备树的文本表示形式，可以由dtc处理为内核期望的形式的二进制设备树。 以下描述不是DTS的正式语法定义，但描述了用于表示设备树的基本结构。

DTS文件的名称应以“.dts”结尾。

## 6.1编译器指令
其他源文件可以包含在DTS文件中。 包含文件的名称应以“.dtsi”结尾。 包含的文件又可以包含其他文件。
```dts?linenums=false
/include/ "FILE.dtsi"
```
## 6.2 标签
设备树源格式允许将标签附加到设备树中的任何节点或属性值。 可以通过引用标签而不是显式指定phandle值或节点的完整路径来自动生成Phandle和path引用。 标签仅用于设备树源格式，不会编码为DTB二进制文件。

标签长度应在1到31个字符之间，仅由表6.1中的字符组成，并且必须
不是以数字开头。

通过在标签名称后附加冒号（':'）来创建标签。 通过在标签名称前加上＆符号（'＆'）来创建引用。

![9](https://www.github.com/liao20081228/blog/raw/master/图片/设备树规范v0.2/9.JPG)


## 6.3节点和属性定义 
设备树节点使用node-name和unit-address，其中大括号标记节点定义的开始和结束。 它们之前可能有标签。

```dts?linenums=false
[label:] node-name[@unit-address] {
	[properties definitions]
	[child nodes]
};
```

节点可以包含属性定义和子节点定义。 如果两者都存在，则属性应在子节点之前。

可以删除先前定义的节点。
```dts?linenums=false
/delete-node/ node-name;
/delete-node/ &label;
```
属性定义是表单中的名称值对：
```dts?linenums=false
[label:] property-name = value;
```

具有空（零长度）值的属性，其形式为：

```dts?linenums=false
[label:] property-name;
```

可以删除先前定义的属性。


```dts?linenums=false
/delete-property/ property-name;
```
属性值可以定义为32位整数单元（cell）的数组，可以是以空字符结尾的字符串，可以是字节串或这些的组合。

* 单元的数组由围绕空格分隔的C语言样式的整数列表的尖括号表示。 例：`interrupts = <17 0xc>;`

* 值可以在括号内表示为算术，按位或逻辑表达式。运算符号包括`+ - * / | & ^ ~ << >> || && ! < <= > >= == != ?: (condition ? value_if_true : value_if_false)`

* 64位值用两个32位单元表示。 例：`clock-frequency = <0x00000001 0x00000000>;`

* 使用双引号表示以null结尾的字符串值（属性值被视为包含终止NULL字符）。 例：`compatible = "simple-bus";`

* 字节字符括在方括号[]中，每个字节由两个十六进制数字表示。 每个字节之间的空格是可选的。 例：`local-mac-address = [00 00 12 34 56 78];` 或`local-mac-address = [000012345678];`

* 值可能有几个以逗号分隔的组件，它们连接在一起。 例：`compatible = "ns16550", "ns8250";
`或`example = <0xf00f0000 19>, "a strange property format";`

* 在单元（cell）数组中，对另一个节点的引用将展开为该节点的phandle。 引用可以是＆后跟节点的标签。 例：`interrupt-parent = < &mpic >;` 或者＆后跟一个大括号中的完整路径。 例：`interrupt-parent = < &{/soc/interrupt-controller@40000} >;`

* 在单元数组外部，对另一个节点的引用将展开为该节点的完整路径。 例：`ethernet0 = &EMAC0;` 

* 标签也可能出现在属性值的任何组件之前或之后，或单元数组的单元格之间或字节串的字节之间。 例子：
```dts?linenums=false
reg = reglabel: <0 sizelabel: 0x1000000>;`
prop = [ab cd ef byte4: 00 ff fe];
str = start: "string value" end: ;
```

## 6.4 文件布局
版本1 DTS文件具有整体布局：
```dts?linenums=false
/dts-v1/;
[memory reservations]
/ {
[property definitions]
[child nodes]
};
```
/ dts-v1 /; 必须存在以将文件标识为版本1 DTS（没有此标记的dts文件将被dtc视为处于过时版本0中，除了其他小但不兼容的更改之外，其使用不同的整数格式）。

内存保留定义了DTB的内存保留表的条目。 它们具有以下形式：例如，`/memreserve/ <address> <length>;`其中`<address>`和`<length>`是64位C样式的整数。

* `/{};` 定义了设备树的根节点。
* 支持C样式（/\* ... \*/）和C++样式（//）注释。

# 7参考文献

<span id='b1'>[b1] Power ISA™, Version 2.06 Revision B, July 23, 2010. It is available from power.org (http://power.org)</span>

<span id='IEEE1275'>[IEEE1275] Boot (Initialization Configuration) Firmware: Core Requirements and Practices, 1994, This is the core standard (also known as IEEE 1275) that defines the devicetree concept adopted by the DTSpec and ePAPR. It is available from Global Engineering (http://global.ihs.com/).

<span id='b1'>[b3] PowerPCProcessorBindingtoIEEE1275-1994StandardforBoot(Initialization, Configuration)Firmware, Version 2.1, Open Firmware Working Group,(http://playground.sun.com/1275/bindings/ppc/release/ppc-2_1.html), 1996,This document specifies the PowerPC processor specific binding to the base standard.

[b4] booting-without-of.txt, Ben Herrenschmidt, Becky Bruce, et al., From the Linux kernel source tree (http://www.kernel.org/), Describes the devicetree as used by the Linux kernel.

[b5] Device Trees Everywhere, David Gibson and Ben Herrenschmidt (http://ozlabs.org/~dgibson/home/papers/dtc-paper.pdf), An overview of the concept of the devicetree and devicetree compiler.

[b6] PCI Bus Binding to: IEEE Std 1275-1994 Standard for Boot (Initialization Configuration) Firmware, Revision 2.1, Open Firmware Working Group, 1998 (http://playground.sun.com/1275/bindings/pci/pci2_1.pdf)

<span id='b7'>[b7] Open Firmware Recommended Practice: Interrupt Mapping, Version 0.9, Open Firmware Working Group, 1996(http://playground.sun.com/1275/practice/imap/imap0_9d.pdf)</span>

[b8] Open Firmware Recommended Practice: Device Support Extensions, Version 1.0, Open Firmware Working Group,1997, (http://playground.sun.com/1275/practice/devicex/dse1_0a.html) This document describes the binding for various device types such as network, RTC, keyboard, sound, etc.

[b9] Open Firmware Recommended Practice: Universal Serial Bus Binding to IEEE 1275, Version 1, Open Firmware Working Group, 1998 (http://playground.sun.com/1275/bindings/usb/usb-1_0.ps)

[CHRP] PowerPC Microprocessor Common Hardware Reference Platform (CHRP) Binding, Version 1.8, OpenFirmware Working Group, 1998 (http://playground.sun.com/1275/bindings/chrp/chrp1_8a.ps). This document specifies the properties for Open PIC-compatible interrupt controllers.

[b11] CHRP ISA Interrupt Controller Device Binding, Unapproved Draft version 1.1, Open Firmware Working Group,Aug 19, 1996 (http://playground.sun.com/1275/bindings/devices/postscript/isa-pic-1_1d.ps)

[b12] The Open Programmable Interrupt Controller (PIC) Register Interface Specification, Revision 1.2, Advanced Micro Devices and Cyrix Corporation, October 1995

[b13] PCI Local Bus Specification, Revision 2.2, PCI Special Interest Group

[b14] PCI Express Base Specification, Revision 1.0a, PCI Special Interest Group

[b15] PCI-Express Binding to OF, P1275 Openboot Working Group Proposal, 18 August 2004

[PAPR] Power.org Standard for Power Architecture Platform Requirements, power.org

[b17] System V Application Binary Interface, Edition 4.1, Published by The Santa Cruz Operation, Inc., 1997

<span id='b18'>[b18] The Open Programmable Interrupt Controller (PIC) Register Interface Specification Revision 1.2, AMD and Cyrix,October 1995</span>

[b19] RFC 2119, Key words for use in RFCs to Indicate Requirement Levels, http://www.ietf.org/rfc/rfc2119.txt

[b20] 64-bit PowerPC ELF Application Binary Interface Supplement 1.9, Ian Lance Taylor, 2004

<span id='EPAPR'>[EPAPR] Power.org Standard for Embedded Power Architecture Platform Requirements, power.org, 2011, https://www.power.org/documentation/power-org-standard-for-embedded-power-architecture-platform-requirements-epapr-v1-1-2/</span>

[ARMv8] ARM DDI 0487 ARM(c) Architecture Reference Manual, ARMv8 for ARMv8-A architecture profile, ARM,http://infocenter.arm.com/help/topic/com.arm.doc.ddi0487a.h/index.html




------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文翻译自<font color=blue>[设备树官网](https://www.devicetree.org/)。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------


