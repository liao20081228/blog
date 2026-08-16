---
title: zynq-7000中GPIO的实现方式
tags: Xilinx
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《zynq中各种GPIO方式的区别》](https://blog.csdn.net/qq_36373500/article/details/54628831 "点击跳转")、[《基于Zynq的MIO与EMIO的区别和应用》](https://blog.csdn.net/limoon1212/article/details/45483491)。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

<style>table{word-break:initial;}</style>



&emsp;&emsp;ZYNQ可以使用多种方式提供GPIO的能力，ZYNQ中GPIO有四种，其中PS中MIO/EMIO两种，而PL中同样有两种情况，AXI_GPIO和AXI_LITE自定义的GPIO；下面就这四种情况进行说明：
# 1 PS的MIO实现的GPIO
* MIO实现的GPIO需要在做PCB板卡之前就对功能有所限制：

  * 其中MIO和EMIO是直接挂在PS上的GPIO。而AXI_GPIO是通过AXI总线挂在PS上的GPIO上。

  * 其中MIO分布在BANK0，BANK1，而EMIO则分布在BANK2、BANK3。
 
* 注意一下几项：

  * 首先、MIO在zynq上的管脚是固定的，而EMIO，是通过PL部分扩展的，所以**使用EMIO时候需要在约束文件中分配管脚**，所以设计EMIO的程序时，需要生成PL部分的bit文件，烧写到FPGA中。
 
  * 其次、MIO共占54bit，而EMIO占64bit。其中MIO占用IO号为0-53。而EMIO占用IO号为54-117。
 
  * 无论是EMIO还是MIO都属于PS上的IO，直接由PS操作。在调用头文件，只调用`#include "xgpiops.h"`即可，而在调用AXI_GPIO时，则需要`#include "xgpio.h"`。


>**MIO与EMIO概念**
MIO：多功能IO接口，属于Zynq的PS部分，在芯片外部有54个引脚。这些引脚可以用在GPIO、SPI、UART、TIMER、Ethernet、USB等功能上，每个引脚都同时具有多种功能，故叫多功能。
EMIO：扩展MIO，依然属于Zynq的PS部分，只是连接到了PL上，再从PL的引脚连到芯片外面实现数据输入输出。

>**MIO与EMIO的使用例程**:双串口、双网口等的实现
实现数据直接从PS的MIO口接收与发送
通过PL口实现数据的接收与发送
实现数据的双串口/双网口接收与发送（即PS可同时从两个UART口或网口接收或发送数据）

# 2 PS的EMIO实现GPIO
EMIO实现GPIO也使用PS，但是有FPGA的灵活

EMIO实现是在平台设置

![1](https://www.github.com/liao20081228/blog/raw/master/图片/zynq-7000中GPIO的实现方式/1.JPG)



EMIO设置之后如果实现为IO方式就需要添加一个自定义的iobuf,如图

![2](https://www.github.com/liao20081228/blog/raw/master/图片/zynq-7000中GPIO的实现方式/2.JPG)

IOBUF的实现代码如下
```verilog
  
module ad_iobuf
(
    input   [1:0]  dio_t, 
    input   [1:0]  dio_i, 
    output  [1:0]  dio_o,
	inout   [1:0]  dio_p
);
	
  genvar n;
 
  generate
	for (n = 0; n < 2; n = n + 1) 
		begin: g_iobuf
			assign dio_o[n] = dio_p[n];
			assign dio_p[n] = (dio_t[n] == 1'b1) ? 1'bz : dio_i[n];
		end
  endgenerate
 
endmodule

```


以上两种在SDK软件层面实现
```cpp
static XGpioPs psGpioInstancePtr;   //定义PS的GPIO指针，如果用到MIO和EMIO也只要定义这一个就行
XGpioPs_Config *GpioConfigPtr; //XGpioPs结构体中还包含一个结构体，查bsp中的h文件
int xStatus;

GpioConfigPtr =XGpioPs_LookupConfig(XPAR_PS7_GPIO_0_DEVICE_ID);   
if(GpioConfigPtr == NULL)
       returnXST_FAILURE;
xStatus =XGpioPs_CfgInitialize(&psGpioInstancePtr,GpioConfigPtr,GpioConfigPtr->BaseAddr);
if(XST_SUCCESS != xStatus)
       print("PS GPIO INIT FAILED \n\r");

XGpioPs_SetDirectionPin(&psGpioInstancePtr,iPinNumber,uPinDirection);
XGpioPs_SetOutputEnablePin(&psGpioInstancePtr,iPinNumber,1);
XGpioPs_WritePin(&psGpioInstancePtr,iPinNumber,0); //这里是写某一个pin，也有写一个bank的，陆书上P111就是写bank的。

 
//举例：以LD9（在zed板上是MIO7），另外加上一位EMIO，即54，估计EMIO是从54位开始，有待验证（mark）。


static XGpioPs psGpioInstancePtr;   //定义PS的GPIO指针，如果用到MIO和EMIO也只要定义这一个就行
XGpioPs_Config *GpioConfigPtr; //XGpioPs 结构体中还包含一个结构体，查bsp中的h文件
int xStatus;
static int iPinNumber = 7; /*Led LD9 isconnected to MIO pin 7*/
static int iPinNumberEMIO = 54;

GpioConfigPtr =XGpioPs_LookupConfig(XPAR_PS7_GPIO_0_DEVICE_ID);   
if(GpioConfigPtr == NULL)
       returnXST_FAILURE;
xStatus =XGpioPs_CfgInitialize(&psGpioInstancePtr,GpioConfigPtr,GpioConfigPtr->BaseAddr);
if(XST_SUCCESS != xStatus)
       print("PS GPIO INIT FAILED \n\r");

XGpioPs_SetDirectionPin(&psGpioInstancePtr,7,1);  0输入，1输出
XGpioPs_SetOutputEnablePin(&psGpioInstancePtr,7,1);   0 为dis，1为enable
XGpioPs_WritePin(&psGpioInstancePtr,7,0);
XGpioPs_SetDirectionPin(&psGpioInstancePtr,54,0);  0输入，1输出
XGpioPs_SetOutputEnablePin(&psGpioInstancePtr,54,0);   0 为dis，1为enable
//注明：7可以用iPinNumber代替，54可以用iPinNumberEMIO代替。
```

# 3 PL的AXI_GPIO实现GPIO


# 4 PL的AXI_LITE自定义组件实现GPIO




------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《zynq中各种GPIO方式的区别》](https://blog.csdn.net/qq_36373500/article/details/54628831 "点击跳转")、[《基于Zynq的MIO与EMIO的区别和应用》](https://blog.csdn.net/limoon1212/article/details/45483491)。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
