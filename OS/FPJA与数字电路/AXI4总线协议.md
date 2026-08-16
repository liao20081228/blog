---
title: AXI4总线协议
tags: FPJA
---

------

<font color=blue>**_版权声明_**</font>：<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。网络转载请注明出处！！！</font>

------

# 1 AXI简介

高级可扩展接口**AXI**(Advanced eXtensible Interface)，是ARM公司1996年提出的微控制器总线家族AMBA中的（Advanced Microcontroller Bus Architecture）一部分，是一种**高性能、高带宽、低延迟的片内总线**，也用来替代以前的AHB和APB总线。第一个版本的AXI（AXI3）包含在2003年发布的AMBA3.0中，AXI的第二个版本AXI（AXI4）包含在2010年发布的AMBA 4.0之中。

Xilinx从6系列的FPGA开始引入AXI标准，主要描述了主设备和从设备之间的数据传输方式。在ZYNQ中继续使用，版本是AXI4，所以我们经常会看到AXI4.0，ZYNQ内部设备都有AXI接口。

* AMBA AXI协议支持支持高性能、高频率系统设计。
	* 适合高带宽低延时设计
	* 无需复杂的桥就能实现高频操作
	* 能满足大部分器件的接口要求
	* 适合高初始延时的存储控制器
	* 提供互联架构的灵活性与独立性
	* 向下兼容已有的AHB和APB接口
* AXI协议具有如下特点：
	* 总线的地址/控制和数据通道是分离的；
	* 用字节线来支持非对齐的数据传输
	* 基于burst的传输，只需传输首地址
	* 分离的读、写数据通道，能提供低功耗DMA
	* 支持多种寻址方式
	* 支持乱序传输
	* 允许添加寄存器级来进行更容易的时序收敛

# 2 AXI4的三种总线

* AXI4-Lite——相当于原来的APB协议，具有轻量级，结极简单的特点，不支持批量传输，读写时一次只能读写一个字（32位），可为外设提供**单个数据传输**，用于**访问一些低速外设和外设控制**，需要地址；

* AXI4——相当于原来的AHB协议，提供高速的系统内部互连通道，支持批量传输，可以连续对一片地址进行一次性读写，支持burst模式，主要**用于处理器访问存储等需要高速数据的场合**

>上面两种均采用内存映射控制方式，即ARM将用户自定义IP编入某一地址进行访问，读写时就像在读写自己的片内RAM，编程也很方便，开发难度较低。代价就是资源占用过多，需要额外的读地址线、写地址线、读数据线、写数据线、写应答线等。

* AXI4-Stream——像FIFO一样，是一种连续流接口，不需要地址线，数据传输的时候**不需要地址**，而是主从设备直接连续读写数据，可以支持burst模式，主要用于如视频、高速AD、PCIe、DMA接口等需要**高速数据传输**的场合，其本质都是针对数值流构建的数据通路，从信源（例如 ARM 内存、DMA、无线接收前端等）到信宿（例如 HDMI 显示器、高速AD 音频输出，等）构建起连续的数据流。这种接口适合做实时信号处理，跟Xilinx原来的Local Link协议类似，

>对于这类 IP，ARM 不能通过上面的内存映射方式控制（FIFO 根本没有地址的概念），必须有一个转换装置，例如 AXI-DMA 模块来实现内存映射到流式接口的转换。

> 下面为几个常AXI接口IP的功能介绍： 
AXI-DMA：实现从PS内存到PL高速传输通道AXI-HP<---->AXI-Stream的转换
AXI-FIFO-MM2S：实现从PS内存到PL通用传输通道AXI-GP<----->AXI-Stream的转换
AXI-Datamover：实现从PS内存到PL高速传输通道AXI-HP<---->AXI-Stream的转换，只不过是完全由PL控制的，PS是完全被动的。
AXI-VDMA：实现从PS内存到PL高速传输通道AXI-HP<---->AXI-Stream的转换，只不过是专门针对视频、图像等二维数据。 
AXI-CDMA：这个是由PL完成的将数据从内存的一个位置搬移到另一个位置，无需CPU参与。

|接口 |传输特性  |应用场合|替代接口|
|:--|:--|:--|:--|
|AXI4-Lite |地址，单数据传输 | 低速外设备控制|AHB
|AXI4  |地址，突发批量数据传输  |需要地址的批量传输|APB
|AXI4-Stream | 仅传输数据，突然传输 | 数据流和媒体流传输|Local Link




# 3 AXI4工作原理
## 3.1 通道架构

AXI4包含5个不同的通道：

* 读地址通道，包含ARVALID, ARADDR, ARREADY等信号；
* 写地址通道，包含AWVALID，AWADDR, AWREADY等信号；
* 读数据通道，包含RVALID, RDATA, RREADY, RRESP等信号；
* 写数据通道，包含WVALID, WDATA，WSTRB,WREADY等信号；
* 写应答通道，包含BVALID, BRESP, BREADY等信号；
* 系统通道，包含：ACLK，ARESETn等信号；
	
## 3.2 信号描述

### 3.2.1 系统通道（也叫全局）信号

|信号名|源|描述|
|:--|:--|:--|
|ACLK|　　	时钟源　|　	全局时钟信号|
|ARESETn|　　	复位源|	全局复位信号，低电平有效|



### 3.2.2 写地址通道信号


|信号名|源|描述|
|:--|:--|:--|
|AWID | 主机 |写地址ID，用来标志一组写信号
|AWADDR|主机|	写地址，给出一次写突发传输的写地址
|AWLEN|	主机|	突发长度，给出突发传输的次数
|AWSIZE	|主机|	突发大小，给出每次突发传输的字节数
|AWBURST|主机|	突发类型
|AWLOCK|主机|	总线锁信号，可提供操作的原子性
|AWCACHE|主机|	内存类型，表明一次传输是怎样通过系统的
|AWPROT	|主机|	保护类型，表明一次传输的特权级及安全等级
|AWQOS	|主机|	质量服务QoS
|AWREGION|主机|	区域标志，能实现单一物理接口对应的多个逻辑接口
|AWUSER	|主机|	用户自定义信号
|AWVALID|主机|	有效信号，表明此通道的地址控制信号有效
|AWREADY|从机|	表明“从”可以接收地址和对应的控制信号


### 3.2.3 写数据通道信号

|信号名|源|描述|
|:--|:--|:--|
|WID | 主机 | 一次写传输的ID tag |
|WDATA	|主机|	写数据
|WSTRB	|主机|	写数据有效的字节线，信号为1的bit用来表明WDATA哪8bits数据是有效的
|WLAST	|主机|	表明此次传输是最后一个突发传输
|WUSER	|主机|	用户自定义信号
|WVALID	|主机|	写有效，表明此次写有效
|WREADY	|从机|	表明从机可以接收写数据


### 3.2.4 写响应通道信号

|信号名|源　|　描述　|
|:--|:--|:--|
|BID|	从机|	写响应ID tag|
|BRESP|	从机|	写响应，表明写传输的状态
|BUSER	|从机|	用户自定义
|BVALID	|从机|	写响应有效
|BREADY	|主机|	表明主机能够接收写响应


### 3.2.5 读地址通道信号

|信号名　　|　　	源　|　　　	描述　　　　　　
|:--|:--|:--|
|ARID|	|主机	|读地址ID，用来标志一组写信号
|ARADDR	|主机	|读地址，给出一次写突发传输的读地址
|ARLEN	|主机	|突发长度，给出突发传输的次数
|ARSIZE	|主机	|突发大小，给出每次突发传输的字节数
|ARBURST	|主机	|突发类型
|ARLOCK	|主机	|总线锁信号，可提供操作的原子性
|ARCACHE	|主机	|内存类型，表明一次传输是怎样通过系统的
|ARPROT	|主机	|保护类型，表明一次传输的特权级及安全等级
|ARQOS	|主机	|质量服务QoS
|ARREGION	|主机	|区域标志，能实现单一物理接口对应的多个逻辑接口
|ARUSER	|主机	|用户自定义信号
|ARVALID	|主机	|有效信号，表明此通道的地址控制信号有效
|ARREADY	|从机	|表明“从”可以接收地址和对应的控制信号


### 3.2.6 读数据通道信号

|信号名　|　　　	源　|　　　	描述|
|:--|:--|:--|
|RID|	从机|	读ID tag
|RDATA|	从机|	读数据
|RRESP|	从机|	读响应，表明读传输的状态
|RLAST|	从机|	表明读突发的最后一次传输
|RUSER|	从机|	用户自定义
|RVALID	|从机|	表明此通道信号有效
|RREADY|	主机|	表明主机能够接收读数据和响应信息


### 3.2.7  低功耗接口信号

|信号名　　　|　	源　　　|　	描述|
|:--|:--|:--|
|CSYSREQ|	时钟控制器|系统退出低功耗请求，此信号从“时钟控制器”到“外设”
|CSYSACK|	外设|	退出低功耗状态确认
|CACTIVE|	外设|	外设请求时钟有效


## 3.3 时钟复位
 时钟：每个AXI组件使用一个时钟信号ACLK，所有输入信号在ACLK上升沿采样，所有输出信号必须在ACLK上升沿后发生。
 
 复位：AXI使用一个低电平有效的复位信号ARESETn，复位信号可以异步断言，但必须和时钟上升沿同步去断言。

 复位期间对接口有如下要求：
* 主机接口必须驱动ARVALID，AWVALID，WVALID为低电平；
* 从机接口必须驱动RVALID，BVALID为低电平；
* 所有其他信号可以被驱动到任意值。
* 在复位后，主机可以在时钟上升沿驱动ARVALID，AWVALID，WVALID为高电平。


## 3.4 握手过程

AXI 协议主要描述了主设备和从设备之间的信息传输方式。**AXI4所采用的是一种READY，VALID握手通信机制**，即主从模块进行数据通信前，根据操作对所用到的数据、控制地址通道（AXI4-Stream 中没有地址通道）进行握手。当从设备准备好接收信息时，会发出 READY 信号。当主设备的信息准备好时，会发出和维持VALID信号，表示信息有效。**VALID和READY信号并没有固定的先后顺序**，谁准备好了就可以发送。**信息只有在VALID和READY信号都有效的时候才开始传输**。当这两个信号持续保持有效，主设备会继续传输下一个信息。主设备撤销 VALID信号或从设备撤销 READY 信号终止传输。


* 5个传输通道均使用VALID/READY信号对传输过程的地址、数据、控制信号进行握手。使用双向握手机制，传输仅仅发生在VALID、READY同时有效的时候。
* VALID、READY并没有固定的先后顺序，谁准备好了就可以发送。如图所示：

![握手机制](http://img-blog.csdn.net/20180610153442285?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpYW8yMDA4MTIyOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 3.5 乱序传输
axi总线支持交叉的，乱序的传输，传输的正确性依靠id去保证。
![乱序的传输](http://img-blog.csdn.net/20180610115818640?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpYW8yMDA4MTIyOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


## 3.6 通道信号要求

* 通道握手信号：每个通道有自己的xVALID/xREADY握手信号对。

* 写地址通道：当主机驱动有效的地址和控制信号时，主机可以断言AWVALID，一旦断言，需要保持AWVALID的断言状态，直到时钟上升沿采样到从机的AWREADY。AWREADY默认值可高可低，推荐为高（如果为低，一次传输至少需要两个周期，一个用来断言AWVALID，一个用来断言AWREADY）；当AWREADY为高时，从机必须能够接受提供给它的有效地址。

* 写数据通道：在写突发传输过程中，主机只能在它提供有效的写数据时断言WVALID，一旦断言，需要保持断言状态，直到时钟上升沿采样到从机的WREADY。WREADY默认值可以为高，这要求从机总能够在单个周期内接受写数据。主机在驱动最后一次写突发传输是需要断言WLAST信号。

* 写响应通道：从机只能它在驱动有效的写响应时断言BVALID，一旦断言需要保持，直到时钟上升沿采样到主机的BREADY信号。当主机总能在一个周期内接受写响应信号时，可以将BREADY的默认值设为高。

* 读地址通道：当主机驱动有效的地址和控制信号时，主机可以断言ARVALID，一旦断言，需要保持ARVALID的断言状态，直到时钟上升沿采样到从机的ARREADY。ARREADY默认值可高可低，推荐为高（如果为低，一次传输至少需要两个周期，一个用来断言ARVALID，一个用来断言ARREADY）；当ARREADY为高时，从机必须能够接受提供给它的有效地址。

* 读数据通道：只有当从机驱动有效的读数据时从机才可以断言RVALID，一旦断言需要保持直到时钟上升沿采样到主机的BREADY。BREADY默认值可以为高，此时需要主机任何时候一旦开始读传输就能立马接受读数据。当最后一次突发读传输时，从机需要断言RLAST。

## 3.7 通道间关系

* AXI协议要求通道间满足如下关系：
	* 写响应必须跟随最后一次burst的写传输
	* 读数据必须跟随数据对应的地址
	* 通道握手信号需要确认一些依赖关系

* 通道握手信号的依赖关系
	* 为防止死锁，通道握手信号需要遵循一定的依赖关系。
	* VALID信号不能依赖READY信号。
	* AXI接口可以等到检测到VALID才断言对应的READY，也可以检测到VALID之前就断言READY。
* 下面有几个图表明依赖关系，单箭头指向的信号能在箭头起点信号之前或之后断言；双箭头指向的信号必须在箭头起点信号断言之后断言。

![依赖关系](http://img-blog.csdn.net/20180610162533755?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpYW8yMDA4MTIyOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)



## 3.8 地址传输结构

AXI协议是基于burst的，**主机只给出突发传输的第一个字节的地址，从机必须计算突发传输后续的地址。突发传输不能跨4KB边界**（防止突发跨越两个从机的边界，也限制了从机所需支持的地址自增数）。

### 3.8.1 burst传输规则
burst传输具有如下规则：

* WRAP型 burst ，burst长度必须是2,4,8,16
* burst不能跨4KB边界
* 不支持提前终止burst传输
* 所有的组件都不能提前终止一次突发传输。然而，主机可以通过解断言所有的写的strobes来使非所有的写字节来减少写传输的数量。读burst中，主机可以忽略后续的读数据来减少读个数。也就是说，不管怎样，都必须完成所有的burst传输。

>注：对于FIFO，忽略后续读数据可能导致数据丢失，必须保证突发传输长度和要求的数据传输大小匹配。

### 3.8.2 突发长度

ARLEN[7:0]是读传输的突发长度，AWLEN[7:0]是写传输的突发长度。

AXI3只支持1~16次的突发传输（Burst\_length=AxLEN[3:0]+1），AXI4扩展突发长度，INCR突发类型为1~256次传输，对于其他的传输类型依然保持1~16次突发传输（Burst\_Length=AxLEN[7:0]+1）。
### 3.8.3 突发大小
ARSIZE[2:0]是读突发传输大小；AWSIZE[2:0]是写突发传输大小。

|AxSIZE[2:0]| bytes in transfer|AxSIZE[2:0]|    bytes in transfer|
|:--|:--|:--|:--|
|‘b000   |　               1|‘b100|　　　　　　16|
|‘b001　|　　　　　2|‘b101|　　　　　　32|
|‘b010　|　　　　　4|‘b110|　　　　　　64|
|‘b011　|　　　　　8|‘b111|　　　　　　128|

### 3.8.4 突发类型
 ARBURST[1:0]是读突发传输类型；AWBURST[1:0]是写突发传输类型。
* FIXED：突发传输过程中地址固定，用于FIFO访问
* INCR：增量突发，传输过程中，地址递增。增加量取决AxSIZE的值。
* WRAP：回环突发，和增量突发类似，但会在特定高地址的边界处回到低地址处。回环突发的长度只能是2,4,8,16次传输，传输首地址和每次传输的大小对齐。最低的地址整个传输的数据大小对齐。回环边界等于（AxSIZE*AxLEN）。


|AxBURST[1:0]|　　　　burst type|
|:--|:--|
|‘b00|　　　　　　　　　　FIXED|
|‘b01　|　　　　　　　　　INCR|
|‘b10　|　　　　　　　　　WRAP|
|‘b11　|　　　　　　　　　Reserved|

### 3.8.5 计算方法
 * Start_Address=AxADDR
 * Number_Bytes=2^AxSIZE
 * Burst_Length=AxLEN+1
 * Aligned_Addr=（INT（Start_Address/Number_Bytes))xNumber_Bytes。//INT表示向下取整。
 * 对于INCR突发和WRAP突发但没有到达回环边界，地址由下述方程决定：Address_N=Aligned_Address+(N-1)xNumber_Bytes
 * WRAP突发，突发边界：Wrap_Boundary=(INT(Start_Address/(Number_Bytes x Burst_Length)))x(Number_Bytes x Burst_Length)

 


## 3.9 数据传输结构

WSTRB[n:0]对应于对应的写字节，WSTRB[n]对应WDATA[8n+7:8n]。

WVALID为低时，WSTRB可以为任意值，WVALID为高时，WSTRB为高的字节线必须指示有效的数据。

### 3.9.1 窄传输

* 当主机产生比它数据总线要窄的传输时，由地址和控制信号决定哪个字节被传输：
	* INCR和WRAP，不同的字节线决定每次burst传输的数据
	* FIXED，每次传输使用相同的字节线。

* 下图给出了5次突发传输，起始地址为0，每次传输为8bit，数据总线为32bit，突发类型为INCR。![5次突发传输](http://img-blog.csdn.net/20180610172402877?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpYW8yMDA4MTIyOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
* 下图给出3次突发，起始地址为4，每次传输32bit，数据总线为64bit。![3次突发](http://img-blog.csdn.net/20180610172410999?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpYW8yMDA4MTIyOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)




### 3.9.2 非对齐传输

AXI支持非对齐传输。在大于一个字节的传输中，第一个自己的传输可能是非对齐的。如32-bit数据包起始地址在0x1002，非32bit对齐。主机可以
* 使用低位地址线来表示非对齐的起始地址；
* 提供对齐的起始地址，使用字节线来表示非对齐的起始地址。

对齐非对齐传输示例1-32bit总线

![32](http://img-blog.csdn.net/20180610172844818?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpYW8yMDA4MTIyOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

对齐非对齐传输示例2-64bit总线

![64](http://img-blog.csdn.net/20180610172852392?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpYW8yMDA4MTIyOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
对齐的回环传输示例

![回环](http://img-blog.csdn.net/20180610172859797?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpYW8yMDA4MTIyOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 3.10 读写响应结构

*读传输的响应信息是附加在读数据通道上的，写传输的响应在写响应通道。

RRESP[1:0]是读传输响应结构；BRESP[1:0],写传输响应结构。

|标志|XRESP[1:0]|描述|
|:--|:--|:--|
|OKAY|‘b00|正常访问成功
|EXOKAY|‘b01|Exclusive 访问成功
|SLVERR|‘b10|从机错误。表明访问已经成功到了从机，但从机希望返回一个错误的情况给主机。
|DECERR|‘b11|译码错误。一般由互联组件给出，表明没有对应的从机地址。

## 3.11 读写模型
![读写模型](http://img-blog.csdn.net/20180610173525953?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpYW8yMDA4MTIyOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
## 3.12 读写过程
读操作：主与从进行读地址通道握手并传输地址内容，然后在读数据通道握手并传输所读内容以及读取操作的回应，时钟上升沿有效。如图所示：

![读操作](http://img-blog.csdn.net/2018061017353457?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpYW8yMDA4MTIyOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

写操作：主与从进行写地址通道握手并传输地址内容，然后在写数据通道握手并传输所读内容，最后再写回应通道握手，并传输写回应数据，时钟上升沿有效。如图所示：

![写操作](http://img-blog.csdn.net/20180610173540977?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpYW8yMDA4MTIyOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


# 4 AXI4-Lite工作模式 
# 5 AXI4-Stream工作模式
**AXI4-Stream跟AXI4的区别就是AXI4-Stream去除了地址线，这样就不涉及读写数据的概念了，只有简单的发送与接收说法，减少了延时。**

## 5.1 信号描述:
|信号及传输结构|源|描述|
|:--|:--|:--|
|ACLK|时钟源|全局时钟信号。所有信号在ACLK信号上升沿采样。
|ARESETn|复位源|全局复位信号。ARESETn低电平有效。
|TVALID|主|TVALID表示主设备正在驱动一个有效的传输。当TVALID和TREADY都置位时，发生一个传输。
|TREADY|从|TREADY表示从设备在当前周期能够接收一次传输。
|TDATA[(8n-1):0]|主|TDATA是基本的有效载荷，用来提供跨越接口的数据。数据为整数个字节。可选宽度32,64,128,256bit
|TSTRB[(n-1)：0]|主|TSTRB位字节修饰符。用来描述TDATA相关字节内容作为一个数字字节或者一个位置字节被处理。宽度为tdata/8
|TKEEP[(n-1)：0]|主|TKEEP是字节修饰符。用来表明TDATA相关字节的内容是否作为数据流的一部分被处理。TKEEP字节修饰符未被确认的那些相关字节是空字节，可以从数据流中去除。
|TLAST|主|TLAST表明了包的边界。
|TID[(i-1):0]|主|TID是数据流的标识符，用来表明不同的数据流。
|TDEST[(d-1):0]|主|TDEST为据流提供路由信息。
|TUSER[(n-1):0]|主|TUSER是用户定义的边带信息，这它能伴随数据，宽度为128bit



## 5.2 传输过程
![sad](http://www.eefocus.com/include/picture/500/400/data/12-03/1331916395_6a78b023.jpg)




# 6  AXI 互联
AXI协议严格的讲是一个点对点的主从接口协议，**当多个外设需要互相交互数据时，我们需要加入一个AXI Interconnect模块，也就是AXI互联矩阵，作用是提供将一个或多个AXI主设备连接到一个或多个AXI从设备的一种交换机制**（有点类似于交换机里面的交换矩阵）。Xilinx为我们提供了实现这种互联矩阵的IP核axi_interconnect_1，在前面的例子中，我们在XPS中可以看到。这个IP核**最多可以支持16个主设备、16个从设备**，如果需要更多的接口，可以多加入几个IP核。关于AXI Interconnect更多的知识，可参考Xilinx官方文档DS768。

![互联](http://img-blog.csdn.net/20180610120003883?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpYW8yMDA4MTIyOA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


* AXI协议提供单一的接口定义，能用在三种接口之间：master/interconnect、slave/interconnect、master/slave。

* 可以使用以下几种典型的系统拓扑架构：
	* 共享地址与数据总线
	* 共享地址总线，多数据总线
	* multilayer多层，多地址总线，多数据总线

>在大多数系统中，地址通道的带宽要求没有数据通道高，因此可以使用共享地址总线，多数据总线结构来对系统性能和互联复杂度进行平衡。

* 寄存器片（Register Slices）：
	* 每个AXI通道使用单一方向传输信息，并且各个通道直接没有任何固定关系。因此可以在任何通道任何点插入寄存器片，当然这会导致额外的周期延迟。
	* 使用寄存器片可以实现周期延迟（cycles of latency）和最大操作频率的折中；使用寄存器片可以分割低速外设的长路径。




------

<font color=blue>**_版权声明_**</font>：<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。网络转载请注明出处！！！</font>

------
