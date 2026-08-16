---
title: Linux设备树
tags: Linux驱动
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文章参考了<font color=blue >[《Device Tree Usage》](http://www.devicetree.org/Device_Tree_Usage)、[《uboot之fdt介绍》](https://www.cnblogs.com/leaven/p/6295999.html)、[《蜂窝科技的几篇文章》](http://www.wowotech.net/device_model/dt_basic_concept.html)，[《Linux设备树文件结构与解析深度分析》](https://blog.csdn.net/woyimibayi/article/details/77574736)、[《我眼中的Linux设备树》](https://www.cnblogs.com/targethero/p/5053591.html)。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

# 1 设备树简介
&emsp;&emsp;Linus Torvalds在2011年3月17日的ARM Linux邮件列表宣称“this whole ARM thing is a fucking pain in the ass”，引发ARM Linux社区的地震，随后ARM社区进行了一系列的重大修正。在过去的ARM Linux中，`arch/arm/plat-xxx和arch/arm/mach-xxx`中充斥着大量的垃圾代码，相当多数的代码只是在描述板级细节，而这些板级细节对于内核来讲，不过是垃圾，如板上的platform设备、resource、i2c_board_info、spi_board_info以及各种硬件的platform_data。社区必须改变这种局面，于是PowerPC等其他体系架构下已经使用的Flattened Device Tree（FDT）进入ARM社区的视野。

&emsp;&emsp;Device Tree是一种描述硬件的数据结构，它起源于 OpenFirmware (OF)。采用Device Tree后，许多硬件的细节可以直接透过它传递给Linux 内核，而不再需要在kernel中进行大量的冗余编码。设备树是从软件使用的角度描述硬件的，不是从硬件设计的角度描述的。我们在写设备树时没有必要按照硬件逻辑生搬硬套，也不要指望通过阅读设备树弄清楚硬件是如何设计的。对于软件可以自动识别的硬件，如USB设备，PCI设备，也是没有必要通过设备树描述的。

&emsp;&emsp;内核中关于设备树的文档位于`Documentation/devicetree/`目录。设备树是Power.org组织定义的一套名为[EPAPR](https://www.power.org/documentation/epapr-version-1-1/)的规范。如果想快速了解下设备树怎么用，可以参考[《Device Tree Usage》](http://www.devicetree.org/Device_Tree_Usage)




&emsp;&emsp;设备树包括三个部分DTS，DTC和DTB。我们首先根据硬件编写DTS文件，然后在通过DTC工具将DTS文件转换成 DTB文件，然后将DTB文件烧写到机器上(如emmc，磁盘等存储介质)。系统启动时，Bootloader（如u-boot）在启动内核前将DTB文件读到内存中，跳转到内核执行的同时将DTB起始地址传给内核。之后内核会解析Device Tree并创建和注册相关的设备。



![设备树](https://www.github.com/liao20081228/blog/raw/master/图片/Linux设备树/1.jpg "devicetree")

* DTS(Device tree source)是设备树源文件，为了方便阅读及修改，采用文本格式。DTSI类似于C语言的头文件
* DTC(Device tree compiler)是一个小工具，负责将DTS编译成DTB。
* DTB(Device tree blob)是DTS的二进制形式，供内核使用。


# 2 设备树源文件

&emsp;&emsp;**EPAPR**中规定了设备树源文件的语法，可以分为两个层次的。第一层是关于设备树组织形式的，如设备树结构，节点名字的构成等，第一个层次是基础，是理解第二个层次的前提。第二层是关于设备树内容的，如多核CPU怎样描述，一个具体的设备如何描述。第二层可以看成是第一层的具体应用。相对来说第二层内容更多，更具体，根据描述的内容不同，定义规范的方式也有差别，比如关于CPU，内存，中断这些基础的内容，是在epapr中说明的，而关于外设的规范是在专门的地方说明的。说设备树的规范可以分成两个层次，是针对DTS的，关于DTB的结构不在此范围内。

&emsp;&emsp;**dts**文件是一种ASCII文本格式的Device Tree描述，此文本格式非常人性化，适合人类的阅读习惯。基本上，在ARM Linux在，一个.dts文件对应一个ARM的machine，一般放置在内核的`arch/arm/boot/dts/`目录。由于一个SoC可能对应多个machine（一个SoC可以对应多个产品和电路板），势必这些dts文件需包含许多共同的部分，Linux内核为了简化，把SoC公用的部分或者多个machine共同的部分一般提炼为**dtsi**，类似于C语言的头文件，使用`/include/ "file.dtsi"` 或 `#include "file.dtsi"`。其他的machine对应的dts就include这个dtsi。

## 2.1 基本结构
```dts
/dts-v1/; #指定设备树版本，缺少会报错
/{
	node1 {
		a-string-property = "A string";
		a-string-list-property = "first string", "second string";
		a-binary-data-property = [0x01 0x23 0x34 0x56];
		
		child-node1 {
			first-child-property=<&label>;
			second-child-property = <1>;
			a-string-property = "Hello, world";
		};
		
		child-node2 {
		};
	};
	
	labe1: node2 {
		an-empty-property;
		a-cell-property = <1 2 3 4>; /* each cell is a 32 bit*/
		child-node1 {
		};
	};
};
```
上面的dts文件并没有实际意义，但它表示了一个设备树源文件的基本结构：

* dts/dtsi文件由结点组成；
* 每个设备树都有一个根结点"/"开始，根节点只有一个；
* 每个结点下面可以有一系列子结点，子节点下又可含有一系列子结点；
* 节点包含若干属性。
* 节点用节点名标识，节点名的格式是：**node-name@unit-address**
	* 根节点必须是“/”。
	* node-name为节点名，不超过31字符，包括`“0-9 a-z A-Z , . _ + -”`。规范定义了通用的节点名：`atm、cache-controller、compact-flash、can、cpu、crypto、disk、display、dma-controller、ethernet、ethernet-phy、fdc、flash、gpio、i2c、ide、interrupt-controller、isa、keyboard、mdio、memory、memory-controller、mouse、nvram、parallel、pc-card、pci、pcie、rtc、sata、scsi、serial、sound、spi、timer、usb、vme、watchdog`
	* unit-address为节点地址，用来区别多个相同的设备。 相同的设备，node-name相同，而unit-address不同，unit-address的具体格式和设备挂在哪个bus上相关。例如：
		* 对于cpu，其unit-address就是从0开始编址，以此加一。
		* 以太网控制器，其unit-address就是寄存器地址。
	* 如果节点有unit-address，那么节点下边必须有一个叫reg的属性，并且该地址必须和reg的属性的第一个地址相同。如果节点没有reg属性，那么unit-address及前边的@都不能有。
	* **引用节点**，有两种方法
		* 使用全路径full path。全路径就是由根节点开始，各级节点用“/”隔开，如`/cpus/cpu@0`
		* 为节点设置一个标签label，然后用`&label`来引用该节点，这种引用是通过phandle属性进行的。这种phandle节点，在经过DTC工具编译之后，`&label`会变成一个特殊的整型数字n，假设n值为0x00000001，那么在被引用的节点下自动生成两个属性:`linux,phandle = <0x00000001>;phandle = <0x00000001>;`
* 属性由`属性名=值`定义
	* 属性名由1到31个字符组成。和节点名有些区别，不允许有大写字母，增加了`？`和`#`两个字符。为了容易区分以及避免重复，标准未定义的属性名字应该用公司或组织名称开头，比如：`ibm,ppc-interrupt-server#s`。
	* 属性值有5种：
		* 空，如"an-empty-property"；
		* 字符串，如 "a-string-property"；
		* 字符串数组 ，用双引号括起来，如"a-string-property-list"；
		* Cells（单位为u32bit），如"second-child-property"，
		* 无符号整数（u32，u64），如“a-binary-data-property"。
	* EPAPR规范预定义了一些标准的属性:
		* 设备型号有关：`compatible、model、device_type`……		
		* 地址相关：`#address-cells、#size-cells、reg、range、dma-range`……
		* 中断相关：`interrupts、interrupt-controller、#interrupt-cells、interrupt-map、interrupt-map-mask、interrupt-parrent`……
		* 设备状态相关：`status`……
## 2.2 常见属性
### 2.2.1 基本信息
* “compatible”，“model”,"device_type"都是用来表示节点基本信息的。
	* “compatible”属性是用来匹配驱动的，他的类型是字符串数组，每个字符串表示一种设备的类型，从具体到一般。列表中的第一个字符串指定了`<manufacturer>,<model>`格式的节点代表的确切设备，第二个字符串代表了与该设备兼容的其他设备。如：`compatible = "fsl,mpc8349-uart", "ns16550"`。
	* "model"属性用来表示设备的型号，用字符串表示，不像"compatible"用多个字符串，只需一个就够了。
	* "device_type"属性用来表示设备类型，用字符串表示。
 
 * "`phandle`"属性是专门为方便引用节点设计的，想要引用哪个节点就在该节点下边增加一个'phandle'属性，设定值为一个u32，如`phandle = <1>`，引用的地方直接使用数字1就可以引用该节点，如`interrupt-parent = <1>`。以上是规范中描述的方法，实际上这样也不方便，我在实际的代码中没有看到这么用的。还记得节点那节说过节点名字前边可以定义一个标签吧，实际情况是都用标签引用，比如节点标签为intc1，那么用`interrupt-parent = <&intc1>`就可以引用了。

*  “status”属性用来表示节点的状态的，其实就是硬件的状态，用字符串表示。'okay'表示硬件正常工作，“disabled”表示硬件当前不可用，“fail”表示因为出错不可用，“fail-sss”表示因为某种原因出错不可用，sss表示具体的出错原因。实际中，基本只用'okay'和'disabled'。
### 2.2.2 编址

"#address-cells","#size-cells","reg","ranges","dma-ranges"属性都是和地址有关的。不同的平台，不同的总线，地址长度可能不同，有32位地址，有64位地址，为了适应这个，规范规定一个32位的长度为一个cell。
* "#address-cells"属性用来表示地址需要几个cell表示，该属性本身是u32类型的。 
* "#size-cells"属性用来表示地址空间的长度需要几个cell表示，属性本身的类型也是u32。
* "reg"属性用来表示节点地址资源的，比如常见的就是寄存器的起始地址及大小。要想表示一块连续地址，必须包含起始地址和空间大小两个参数，如果有多块地址，那么就需要多组这样的值表示，如`reg = <address1 length1 [address2 length2] ... >`。每个元素是一个二元组，包含起始地址和大小。地址和大小用几个cell表示由父节点的"#address-cells","#size-cells"属性确定。
* "ranges"属性用来表示如何转换设备在总线上的地址和总线本身的地址。'ranges'属性的每个元素是三元组，按照前后顺序分别是`<子总线地址，父总线地址，大小>`。子总线地址需要几个u32表示由'ranges'属性所在节点的'#address-cells'属性决定，父总线地址需要几个u32表示由上一级节点的'#address-cells'属性决定，大小需要几个u32表示由当前节点的'#size-cells'属性确定。
* 'dma-ranges'属性的结构和定义与'ranges'属性完全相同，唯一不同的是地址是dma使用的地址，'ranges'中的地址是cpu使用的地址。

### 2.2.3 中断

* 中断产生设备节点的属性
	* "interrupt-parent"：实质是一个指向该设备所连接的中断控制器的pHandle，但一般使用标签来进行引用，如`interrupt-parent=<&intc>`。那些没有interrupt-parent属性的节点则从它们的父节点继承该属性，也就是说如果设备树的父节点就是中断父节点，那么可以不用设置interrupt-parent属性。
	* "interrupts "：中断源的属性，中断说明符列表，对应于该设备上的每个中断输出信号。每个元素由几个Cell表示由中断父节点的 "#interrupt-cells "属性说明。
* 中断控制器节点的属性
	* "interrupt-controller "： 一个空的属性表示该节点为中断控制器。
	* "#interrupt-cells "： 中断控制器节点或者interrupt nexus节点的属性。它声明了中断产生设备的一个中断说明符需要多少个cell。假如需要2个cell表示，`<中断号，中断触发方式>`，那么#interrupt-cells就设置成2。需要3个cell表示，`<中断类型，中断号，中断触发方式>`，那么#interrupt-cells就设置成3。
* 中断关系节点(用于说明中断控制器中的一个中断与中断产生设备中的多个中断源之间的对应关系)的属性
	* "interrupt-map"：interrupt nexus节点的属性，是cell类型的，每个元素表示一个中断映射关系，`<中断子设备地址，中断子设备中断说明符，中断父设备，中断父设备地址，中断父设备中断说明符>`。中断子设备地址由中断子设备所在总线的#address-cells属性决定，中断子设备中断说明符由该interrupt nexus节点下的#interrupt-cell决定的。中断父设备是一个指向中断父设备的`<phandle>`属性，一般情况下是中断控制器，但是按照中断树的逻辑，也可能是更高一级的interrupt nexus节点。中断父设备地址是由中断父设备节点下的#address-cells属性决定的(注意，不是中断父设备所在总线的#address-cells属性)。中断父设备中断源说明符由几个Cell表示是由中断父设备的#interrupt-cells属性决定的。
	* 中断设备的中断源和中断控制器的中断可能是多对一的关系，如果每个子中断都用interrupt-map中的一行表示，那么interrupt-map属性将非常大。为了让多个子中断共享映射关系，引入了“interrupt-map-mask”属性，该属性的类型也是cell类型，包含中断子设备地址和中断子设备中断源的bit mask，给定一个子中断源，那么首先和interrupt-map-mask做与运算，运算结果再通过interrupt-map属性查找对应的中断父设备中断源。这就是我们前边为什么说interrupt-map属性的一行是一个“中断映射关系”，而不是“一个中断”映射关系的原因。
	
	
&emsp;&emsp;整个中断树的最底层是中断产生设备(也可能是从interrupt nexus节点)，中断产生设备用interrupts属性描述他能产生的中断。因为他的中断父设备可能和设备树的父设备不同，那么用interrupt-parent属性指向他的中断父设备。他的中断父设备可能是中断控制器(如果中断产生设备的中断和中断控制器的中断是一一对应的，或者最底层是interrupt nexus节点)，也可能是interrupt nexus节点(如果最底层是中断产生设备，且需要映射)。interrupt nexus节点及他的所有直接子节点构成了一个interrupt domain，在该interrupt domain下中断源怎样表示由#interrupt-cells属性决定，如何由中断子设备中断源找到中断父设备中断源由interrupt-map和interrupt-map-mask属性决定。interrupt nexus的父节点可能还是一个interrupt nexus父节点，也可能是一个中断控制器，当向上找到最后一个中断控制器，并且该中断控制器再也没有中断父设备时，整个中断树就遍历完成了。中断控制器用interrupt-controller属性表示自己是中断控制器，并且用#interrupt-cells属性表示他所直接管理的interrupt domain用几个u32表示一个中断源。根据中断树的特性，一个设备树中是有可能有多个中断树的。


### 2.2.4 启动

&emsp;&emsp;“bootargs”，该属性是chosen节点的属性，类型是字符串，用来向Linux内核传递cmdline。规范中还定义了stdout-path和stdin-path两个可选的、字符串类型的属性，这两个属性的目的是用来指定标准输入输出设备的，在linux中，这两个属性基本不用。
## 2.3 示例
&emsp;&emsp;为了帮助理解device tree的用法，我们从一个简单的计算机开始， 手把手创建一个device tree来描述它。假设有这样一台计算机（基于ARM Versatile）,由“Acme”制造并命名为"Coyote's Revenge"：

* 1个双核ARM Cortex-A9 32位处理器；
* 256MB SDRAM，起始地址为0
* ARM的processor local bus上的内存映射区域分布了：
	* 2个串口，起始地址0x101F1000 和 0x101F2000
	* GPIO控制器，位于0x101F3000
	* SPI控制器，位于0x10150000，并挂载以下设备：
		* MMC插槽，SS管脚连接到GPIO #1
	* 中断控制器，位于0x10140000
	* External bus桥，External bus桥上又连接了：
		* SMC SMC91111 Ethernet，位于0x10100000
		* I2C控制器，位于0x10160000，I2C总线上又连接了
			* Maxim DS1338实时钟，I2C地址为1101000 （0x58）
		* 64MB NOR Flash，位于0x30000000

### 2.3.1 初始结构
&emsp;&emsp;第一步，先构建一个计算机的基本架构，即一个有效设备树的最小架构。在这一步，要唯一地标志这台计算机。
```dts
/{  
    compatible = "acme,coyotes-revenge";  
};  
```
&emsp;&emsp;compatible属性以"`<manufacturer>,<model>`"的格式指定了系统名称。指定具体设备和制造商名称来避免命名空间的冲突，一个操作系统可以使用compatible值来决定如何运行这台计算机，在该属性中填入正确的数据很重要。


### 2.3.2 添加CPU
&emsp;&emsp;接下来就要描述各个CPU了。先添加一个“cpus”容器节点，再将每个CPU作为子节点添加。在本例中，系统是基于ARM的双核Cortex A9系统。
```dts
/dts-v1/;
/ {  
    compatible = "acme,coyotes-revenge";  
  
  
    cpus {  
        cpu@0 {  
            compatible = "arm,cortex-a9";  
        };  
        cpu@1 {  
            compatible = "arm,cortex-a9";  
        };  
    };  
}; 
```
&emsp;&emsp;各个CPU节点的compatible属性是一个字符串，与顶层compatible属性类似，该字符串以“`<manufacturer>,<model>`”的格式指定了CPU的确切型号。


### 2.3.3 添加其他设备
&emsp;&emsp;系统中的每个设备由device tree的一个节点来表示，接下来将为设备树添加设备节点。在我们讲到如何寻址和如何处理中断之前，暂时将新节点置空。
```dts
/dts-v1/;
/ {  
    compatible = "acme,coyotes-revenge";  
  
  
    cpus {  
        cpu@0 {  
            compatible = "arm,cortex-a9";  
        };  
        cpu@1 {  
            compatible = "arm,cortex-a9";  
        };  
    };  
  
  
    serial@101F0000 {  
        compatible = "arm,pl011";  
    };  
  
  
    serial@101F2000 {  
        compatible = "arm,pl011";  
    };  
  
  
    gpio@101F3000 {  
        compatible = "arm,pl061";  
    };  
  
  
    interrupt-controller@10140000 {  
        compatible = "arm,pl190";  
    };  
  
  
    spi@10115000 {  
        compatible = "arm,pl022";  
    };  
  
  
    external-bus {  
        ethernet@0,0 {  
            compatible = "smc,smc91c111";  
        };  
  
  
        i2c@1,0 {  
            compatible = "acme,a1234-i2c-bus";  
            rtc@58 {  
                compatible = "maxim,ds1338";  
            };  
        };  
  
  
        flash@2,0 {  
            compatible = "samsung,k8f1315ebm", "cfi-flash";  
        };  
    };  
};  
```
&emsp;&emsp;在上面的设备树中，系统中的设备节点已经添加进来，树的层次结构反映了设备如何连到系统中。外部总线上的设备就是外部总线节点的子节点，i2c设备是i2c总线控制节点的子节点。总的来说，**层次结构表现的是从CPU视角来看的系统视图**。在这里这棵树是依然是无效的。它缺少关于设备之间的连接信息。稍后将添加这些数据。在这个设备树中应当注意：
* 每个设备节点有一个compatible属性。
* flash节点的compatible属性有两个字符串。 
* 之前提到的，节点命名应当反映设备的类型，而不是特定型号。请参考EPAPR规范2.2.2节的通用节点命名，应优先使用这些命名。

&emsp;&emsp;树中的每一个代表了一个设备的节点都要有一个compatible属性。compatible是OS用来匹配设备驱动的关键。compatible是字符串的列表。列表中的第一个字符串指定了"`<manufacturer>,<model>`"格式的节点代表的确切设备，第二个字符串代表了与该设备兼容的其他设备。例如，Freescale MPC8349 SoC有一个串口设备实现了National Semiconductor ns16550寄存器接口。因此MPC8349串口设备的compatible属性为：`compatible = "fsl,mpc8349-uart", "ns16550"`。在这里，fsl,mpc8349-uart指定了确切的设备，ns16550表明它与National Semiconductor 16550 UART是寄存器级兼容的。
>&emsp;&emsp;注：由于历史原因，ns16550没有制造商前缀，但所有新的compatible值都应使用制造商的前缀。这种做法使得现有的设备驱动程序可以绑定到一个新设备上，同时仍能唯一准确的识别硬件。
>&emsp;&emsp;警告：不要使用带通配符的compatible值，如"fsl,mpc83xx-uart"等类似表达，芯片厂商总会改变并打破你的通配符假设，到时候再想修改就为时已晚了。相反，你应当选择一个特定的芯片实现，并与所有后续芯片保持兼容。

 

### 2.3.4  编址
* 可编址的设备使用下列属性来将地址信息编码进设备树：
	* reg
	* #address-cells
	* #size-cells
	* ranges
* 每个可寻址的设备有一个reg属性，即以下面形式表示的元组列表：`reg = <address1 length1 [address2 length2] [address3 length3] ... >`
* 每个元组表示该设备的地址范围。每个地址值由一个或多个32位整数列表组成，被称做cells。同样地，长度值也可以是cells列表或为空。
* 由于address和length字段是大小可变的变量，**父节点的`#address-cells`和`#size-cells`属性用来说明子节点的各个字段有多少个cells**。换句话说，正确解释一个子节点的reg属性需要父节点的#address-cells和#size-cells值。

#### 2.3.4.1 CPU编址

&emsp;&emsp;谈到编址，最简单的例子就是CPU节点。每个CPU被分配了一个唯一的ID，并且不存在与CPU ids的相关大小信息。
```dts
cpus {  
    #address-cells = <1>;  
    #size-cells = <0>;  
    cpu@0 {  
        compatible = "arm,cortex-a9";  
        reg = <0>;  
    };  
    cpu@1 {  
        compatible = "arm,cortex-a9";  
        reg = <1>;  
    };  
};
```
&emsp;&emsp;在cpus节点中，`#address-cells`为1，`#size-cells`为0，这意味着子寄存器值是一个uint32，是一个不包含大小字段的地址。在本例中，两个CPU被分配为地址0和1。cpu节点的`#size-cells`为0，是因为只为每个CPU分配了地址值。你一定注意到了，reg值与节点名中的值是匹配。按照惯例，如果一个节点有reg属性，则节点名称必须包含unit-address属性，unit-address属性值是reg属性中的第一个地址值。

>注：ePAPR中对cell的定义是”一个包含32bit信息的单元“。

#### 2.3.4.2 内存映射设备
&emsp;&emsp;与CPU节点中的单一地址值不同，内存映射设备会被分配一个它能响应的地址范围。`#size-cells`用来说明每个子节点中reg元组的大小的长度。在下面的示例中，每个地址值是1 cell (32位) ，并且每个的长度值也为1 cell，这在32位系统中是非常典型的。64位计算机可以在设备树中使用2作为`#address-cells`和`#size-cells`的值来实现64位寻址。
```dts
/dts-v1/;
/ {  
    #address-cells = <1>;  
    #size-cells = <1>;  
  
  
    ...  
  
  
    serial@101f0000 {  
        compatible = "arm,pl011";  
        reg = <0x101f0000 0x1000 >;  
    };  
  
  
    serial@101f2000 {  
        compatible = "arm,pl011";  
        reg = <0x101f2000 0x1000 >;  
    };  
  
  
    gpio@101f3000 {  
        compatible = "arm,pl061";  
        reg = <0x101f3000 0x1000  
               0x101f4000 0x0010>;  
    };  
  
  
    interrupt-controller@10140000 {  
        compatible = "arm,pl190";  
        reg = <0x10140000 0x1000 >;  
    };  
  
  
    spi@10115000 {  
        compatible = "arm,pl022";  
        reg = <0x10115000 0x1000 >;  
    };  
  
  
    ...  
  
  
};  
```
&emsp;&emsp;每个设备都被分配了一个基地址及该区域大小。本例中的GPIO设备地址被分成两个地址范围:`0x101f3000~0x101f3fff`和`0x101f4000~0x101f400f`。
&emsp;&emsp;有些挂载于总线上的设备有不同的编址方案。例如，设备可以通过独立片选线连接到外部总线。因为父节点定义了它的子节点的地址范围，所以可根据需要选择地址映射来最佳地描述该系统，如下所示：
```dts
external-bus {  
        #address-cells = <2>  
        #size-cells = <1>;  
  
  
        ethernet@0,0 {  
            compatible = "smc,smc91c111";  
            reg = <0 0 0x1000>;  
        };  
  
  
        i2c@1,0 {  
            compatible = "acme,a1234-i2c-bus";  
            reg = <1 0 0x1000>;  
            rtc@58 {  
                compatible = "maxim,ds1338";  
            };  
        };  
  
  
        flash@2,0 {  
            compatible = "samsung,k8f1315ebm", "cfi-flash";  
            reg = <2 0 0x4000000>;  
        };  
    }; 
```	
&emsp;&emsp;在上例中，外部总线用了2个cells来表示地址值;一个是片选号，一个是基于片选的偏移量。长度字段还是一个cell，这是因为只有地址的偏移部分需要一个范围。所以，在本例中，每个reg条目包含3个cell；片选号码，偏移，长度。由于地址范围包含节点及其子节点，父节点可以自由定义任何对该总线而言有意义的编址方案。直接父节点和子节点之外的其他节点，通常不关心本地节点地址域，因而地址不得不从一个域映射到另一个域。


#### 2.3.4.3 非内存映射设备

&emsp;&emsp;其他设备没有映射到处理器本地总线上。虽然这些设备可以有地址范围，但是不能直接被CPU访问，而是由父设备的驱动代表CPU来执行间接访问。

&emsp;&emsp;以I2C设备为例，每个设备分配一个地址，但没有与它相关的长度或范围，这与CPU地址分配很相似。
```dts
i2c@1,0 {  
           compatible = "acme,a1234-i2c-bus";  
           #address-cells = <1>;  
           #size-cells = <0>;  
           reg = <1 0 0x1000>;  
           rtc@58 {  
               compatible = "maxim,ds1338";  
               reg = <58>;  
           };  
       }; 
```	   



#### 2.3.4.4 地址转换

&emsp;&emsp;我们已经讨论过如何分配地址给设备，但在这些地址只是设备节点的本地地址，还没有描述如何将这些地址映射成CPU可使用的地址。
&emsp;&emsp;根节点总是从CPU的角度描述地址空间。如果根节点的子节点已经使用了CPU地址域，就不需要任何显式映射了，例如，串口serial@101f0000被直接分配到地址0x101f0000。
&emsp;&emsp;如果根节点的间接子节点没有使用CPU的地址域，为了获得一个内存映射地址，设备树必须指定如何翻译地址。
```dts
/dts-v1/;
/ {  
    compatible = "acme,coyotes-revenge";  
    #address-cells = <1>;  
    #size-cells = <1>;  
    ...  
    external-bus {  
        #address-cells = <2>;  
        #size-cells = <1>;  
        ranges = <0 0  0x10100000   0x10000     // Chipselect 1, Ethernet  
               1 0  0x10160000   0x10000     // Chipselect 2, i2c controller  
               2 0  0x30000000   0x4000000>;  // Chipselect 3, NOR Flash  
  
  
        ethernet@0,0 {  
            compatible = "smc,smc91c111";  
            reg = <0 0 0x1000>;  
        };  
  
  
        i2c@1,0 {  
            compatible = "acme,a1234-i2c-bus";  
            #address-cells = <1>;  
            #size-cells = <0>;  
            reg = <1 0 0x1000>;  
            rtc@58 {  
                compatible = "maxim,ds1338";  
                reg = <58>;  
            };  
        };  
  
  
        flash@2,0 {  
            compatible = "samsung,k8f1315ebm", "cfi-flash";  
            reg = <2 0 0x4000000>;  
        };  
    };  
}; 
```
&emsp;&emsp;ranges是地址翻译表，由3个字段组成，即`<子地址，父地址，区域大小>`，分别对应子节点的`#address-cells`值，父节点的`#address-cells`值，子节点的`#size-cells`值确定。对于本例中的外部总线，子节点的地址是2个单元，父节点的地址是1个单元，子节点的区域大小是1个单元。3个ranges被转换：

* 从片选0偏移0被映射到地址范围0x10100000~0x1010ffff
* 从片选1偏移0被映射到地址范围0x10160000~0x1016ffff
* 从片选2偏移0被映射到地址范围为0x30000000~0x30ffffff

&emsp;&emsp;另外，如果父节点和子节点的地址空间是相同的，那么一个节点可以添加一个空的ranges属性。一个空的ranges属性的存在意味着子节点地址空间的地址1：1地映射到父地址空间。你可能会问，为如果可以都用一一映射，为什么还需要地址转换？这是因为一些总线（如PCI ）具有完全不同的地址空间，其细节需要暴露给操作系统。其他有DMA engine的计算机需要知道总线上的真实地址。有时设备需要组合在一起，因为他们都有着相同的软件可编程的物理地址映射。是否需要一一映射取决于操作系统所需要的信息，以及硬件设计。
&emsp;&emsp;你也应该注意到，在i2c@1,0节点上没有ranges属性。这样做的原因是，不像外部总线，I2C总线上的设备不是内存映射到CPU地址域的。相反，CPU通过i2c@1,0设备间接访问rtc@58设备。**ranges属性为空意味着设备不能被除了父设备以外的任何设备直接访问**。

&emsp;&emsp;另举一例说明ranges属性：
```dts
soc{

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

		interrupt-parent = < &ipic >;
	}

}
```
&emsp;&emsp;soc的ranges属性指定了，从物理地址为0x0大小为1024KB的子节点映射到了物理地址为0xe0000000的父地址空间，有个这层映射关系，串口设备节点就可以通过load/store地址0xe0004600来访问，即0x4600的偏移+在ranges中指定的映射0xe0000000。
 
&emsp;&emsp;当然在64位操作系统中也会看到这样的映射，不要感到吃惊啦，"0xf 0x00000000"一起组成父节点地址即f00000000。
``` dts
dcsr: dcsr@f00000000 {
	ranges = <0x0 0xf 0x00000000 0x01072000>;
 };
```

### 2.3.5 中断如何工作

&emsp;&emsp;与地址范围转换遵循树的天然结构不同，一台计算机的任何设备都可以发起和终止中断信号。不像设备编址，中断信号表现为独立于树的节点之间的链接。描述中断连接有4个属性：

* interrupt-controller ： 一个空的属性定义该节点为接收中断信号的设备

* #interrupt-cells ： 这是中断控制器节点的属性。它声明了中断控制器的中断说明符有多少个cell（类似#address-cells和#size-cells） 。

* interrupt-parent：设备节点的属性，包含一个指向该设备所连接中断控制器的pHandle。那些没有interrupt-parent属性节点则从它们的父节点继承该属性。

* interrupts ：设备节点属性，包含中断说明符列表，对应于该设备上的每个中断输出信号。

&emsp;&emsp;中断说明符是一个或多个cell的数据（由#interrupt-cells指定），指定设备连接到哪些中断输入。下面的例子中，大多数设备只有一个中断输出，但也有一个设备上有多个中断输出的情况。一个中断符的含义完全取决于绑定的中断控制器设备。每个中断控制器可以决定它需要多少cell来唯一地确定一个中断输入。下面的代码将中断添加到我们的Coyote's Revenge：

```dts
/dts-v1/;
/ {  
    compatible = "acme,coyotes-revenge";  
    #address-cells = <1>;  
    #size-cells = <1>;  
    interrupt-parent = <&intc>;  
  
  
    cpus {  
        #address-cells = <1>;  
        #size-cells = <0>;  
        cpu@0 {  
            compatible = "arm,cortex-a9";  
            reg = <0>;  
        };  
        cpu@1 {  
            compatible = "arm,cortex-a9";  
            reg = <1>;  
        };  
    };  
  
  
    serial@101f0000 {  
        compatible = "arm,pl011";  
        reg = <0x101f0000 0x1000 >;  
        interrupts = < 1 0 >;  
    };  
  
  
    serial@101f2000 {  
        compatible = "arm,pl011";  
        reg = <0x101f2000 0x1000 >;  
        interrupts = < 2 0 >;  
    };  
  
  
    gpio@101f3000 {  
        compatible = "arm,pl061";  
        reg = <0x101f3000 0x1000  
               0x101f4000 0x0010>;  
        interrupts = < 3 0 >;  
    };  
  
  
    intc: interrupt-controller@10140000 {  
        compatible = "arm,pl190";  
        reg = <0x10140000 0x1000 >;  
        interrupt-controller;  
        #interrupt-cells = <2>;  
    };  
  
  
    spi@10115000 {  
        compatible = "arm,pl022";  
        reg = <0x10115000 0x1000 >;  
        interrupts = < 4 0 >;  
    };  
  
  
    external-bus {  
        #address-cells = <2>  
        #size-cells = <1>;  
        ranges = <0 0  0x10100000   0x1000     // Chipselect 1, Ethernet  
                  1 0  0x10160000   0x1000     // Chipselect 2, i2c controller  
                  2 0  0x30000000   0x4000000>; // Chipselect 3, NOR Flash  
  
  
        ethernet@0,0 {  
            compatible = "smc,smc91c111";  
            reg = <0 0 0x1000>;  
            interrupts = < 5 2 >;  
        };  
  
  
        i2c@1,0 {  
            compatible = "acme,a1234-i2c-bus";  
            #address-cells = <1>;  
            #size-cells = <0>;  
            reg = <1 0 0x1000>;  
            interrupts = < 6 2 >;  
            rtc@58 {  
                compatible = "maxim,ds1338";  
                reg = <58>;  
                interrupts = < 7 3 >;  
            };  
        };  
  
  
        flash@2,0 {  
            compatible = "samsung,k8f1315ebm", "cfi-flash";  
            reg = <2 0 0x4000000>;  
        };  
    };  
};
```
&emsp;&emsp;需要注意的是：这台机器只有一个中断控制器interrupt-controller@10140000，标签'intc:'被添加到中断控制器节点，该标签用于给根节点中的interrupt-parent属性分配一个phandle。这个interrupt-parent值将成为系统默认值，所有子节点都继承它，除非它被显式覆盖。每个设备用一个interrupt属性来指定一个不同的中断输入线。#interrupt-cells是2，所以每个中断符有2个单元。本例使用通用格式，第一个cell代表中断行号码，第二个cell代表标志位，如高电平触发/低电平触发，边沿触发/水平触发。对于一个给定的中断控制器，请参阅控制器binding文档，以了解如何对说明符进行编码。

 

### 2.3.6 设备特有的数据

&emsp;&emsp;除了公共属性，一个节点可以添加任意属性和子节点，只要遵循一些规则，操作系统所需要的任何数据可以被添加。
* 首先，设备新的特有属性名应当使用制造商前缀，这样它们不会与现有的标准属性名称冲突。
* 第二，属性和子节点的含义必须有文档来约束，这样一个设备驱动程序的作者才能知道如何解释这些数据。约束记录了一个特定的compatible值代表什么，它应该有什么样的属性，它可能有什么子节点，代表什么设备。每个唯一的compatible值应该有自己的约束（或者声明与其他compatible值的兼容性）。绑定新设备都记录在本wiki页。
* 第三，使用devicetree-discuss@lists.ozlabs.org邮件列表发布新的binding以进行审查。新的binding的审查可以发现了很多未来会导致问题的常见错误。

 

### 2.3.7 特殊节点

#### 2.3.7.1 别名节点
&emsp;&emsp;引用一个特定的节点通常通过完整路径，如`/external-bus/ethernet@0,0`，但当用户真正想要知道的是哪个设备是eth0时，这很不具有易读性，别名节点可分配一个短的alias给一个完整的设备路径。例如：
```dts
aliases {  
    ethernet0 = &eth0;  
    serial0 = &serial0;  
};  

#或者

aliases {  
    ethernet0 = 节点全路径;  
    serial0 = 节点全路径;  
};
```
&emsp;&emsp;分配标识符给设备时，使用别名是受操作系统欢迎的。这里使用了一个新的语法`property = &label`;该语法指定通过标签引用的完整节点路径为一个字符串属性。这与phandle = <&label>;不同，它是把一个pHandle值插入到一个cell。

#### 2.3.7.2 可选节点
&emsp;&emsp;可选节点并不代表真正的设备，而是作为固件和操作系统之间传递数据的地方，如启动参数。选择节点中的数据并不代表硬件。通常情况下，选择节点在DTS源文件中为空，并在开机时填充。在我们的示例系统中，固件可以添加以下选择节点：
```dts
chosen {  
    bootargs = "root=/dev/nfs rw nfsroot=192.168.1.1 console=ttyS0,115200";  
}  
```


# 3 设备树二进制文件
&emsp;&emsp;经过Device Tree Compiler编译，Device Tree source file变成了Device Tree Blob（又称作flattened device tree）的格式。Device Tree Blob的数据组织如下图所示：

![dtb结构图](https://www.github.com/liao20081228/blog/raw/master/图片/Linux设备树/2.gif "dtb结构")

![4](https://www.github.com/liao20081228/blog/raw/master/图片/Linux设备树/4.png)

&emsp;&emsp; alignment gap为填充域，使用0进行填充。
## 3.1 DTB header
&emsp;&emsp;DTB header主要描述设备树的一些基本信息，例如设备树大小，结构块偏移地址，字符串块偏移地址等。偏移地址是相对于设备树头的起始地址计算的。
```cpp
/* ./scripts/dtc/libfdt/libfdt_env.h */
typedef uint16_t FDT_BITWISE fdt16_t;
typedef uint32_t FDT_BITWISE fdt32_t;
typedef uint64_t FDT_BITWISE fdt64_t;

/* ./include/linux/libfdt_env.h */
typedef __be16 fdt16_t;   
typedef __be32 fdt32_t;
typedef __be64 fdt64_t;



/* ./scripts/dtc/libfdt/fdt.h */
struct fdt_header {
   fdt32_t magic;           //设备树魔数，固定为FDT_MAGIC=0xd00dfeed(大端)
   fdt32_t totalsize;        //整个设备树的大小
   fdt32_t off_dt_struct;      //DT structure block在整个设备树中的偏移
   fdt32_t off_dt_strings;     //DT string block在设备树中的偏移
   fdt32_t off_mem_rsvmap;     //保留内存区，该区保留了不能被内核动态分配的内存空间
   fdt32_t version;          //设备树版本
   fdt32_t last_comp_version;   //向下兼容版本号
   fdt32_t boot_cpuid_phys;    //在多核处理器中用于启动的主cpu的物理id
   fdt32_t size_dt_strings;    //DT string block大小
   fdt32_t size_dt_struct;     //DT structure block大小
};



#define FDT_MAGIC   0xd00dfeed
#define FDT_TAGSIZE  sizeof(fdt32_t)
```


|字段	|描述|
|--|--|
|magic	|用来识别DTB的。通过这个magic，kernel可以确定bootloader传递的参数block是一个DTB还是tag list。|
|totalsize|DTB的total size|
|off_dt_struct|	structure block的offset
|off_dt_strings| strings block的offset
|off_mem_rsvmap| memory reserve map的offset。有些系统，我们也许会保留一些memory有特殊用途（例如DTB或者initrd image），或者在有些DSP+ARM的SOC platform上，有写memory被保留用于ARM和DSP进行信息交互。这些保留内存不会进入内存管理系统。
|version|	该DTB的版本。
|last_comp_version|兼容版本信息。如果设备树版本为1~3，则设置为1.如果设备树版本为16或17则设置为16|
|boot_cpuid_phys|	我们在哪一个CPU（用ID标识）上booting。只适用于版本2
|size_dt_strings|strings block的size。它和off_dt_strings一起确定了strings block在内存中的位置，只有版本3及以后才有该字段
|size_dt_struct|structure block的size。它和off_dt_struct一起确定了 structure block在内存中的位置，只有版本17及以后才有该字段
|&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;||

## 3.2 memory reserve map
 ```c
 /* ./scripts/dtc/libfdt/fdt.h */:
 struct fdt_reserve_entry {
	fdt64_t address;
	fdt64_t size;
};
```
&emsp;&emsp;这个区域包括了若干的reserve memory描述符。每个reserve memory描述符是由address和size组成。其中address和size都是用U64来描述。


## 3.3 DT structure block

&emsp;&emsp;设备树结构块是一个线性化的结构体，是设备树的主体，以节点的形式保存了主板上的设备信息。在结构块中，以宏`FDT_BEGIN_NODE`标志一个节点的开始，以宏`FDT_END_NODE`标识一个节点的结束，整个结构块以宏`FDT_END` 结束。在`scripts/dtc/libfdt/fdt.h`中有相关定义，我们把这些宏称之为tag。共计有5种tag，定义如下：
```c
#define FDT_BEGIN_NODE  0x1    /*描述一个node的开始位置，紧挨着该tag的就是node name（包括unit address）*/
#define FDT_END_NODE    0x2     /*描述了一个node的结束位置。*/
#define FDT_PROP    0x3  /*描述了一个property的开始位置，该tag之后是两个u32的数据，
                   分别是length和name offset。length表示该property 
				   value data的size。name offset表示该属性字符串在device tree 
				   strings block的偏移值。length和name offset之后就是长度为length
				   具体的属性值数据。*/
#define FDT_NOP     0x4  /*NOP*/
#define FDT_END     0x9  /*标识了一个DTB的结束位置。*/

#define FDT_V1_SIZE (7*sizeof(fdt32_t))   /*不同版本fdt_header具有不同的大小*/
#define FDT_V2_SIZE (FDT_V1_SIZE + sizeof(fdt32_t))
#define FDT_V3_SIZE (FDT_V2_SIZE + sizeof(fdt32_t))
#define FDT_V16_SIZE FDT_V3_SIZE
#define FDT_V17_SIZE (FDT_V16_SIZE + sizeof(fdt32_t))
```
```csharp?linenums
/*节点信息使用struct fdt_node_header结构体描述。*/
struct fdt_node_header  {
    fdt32_t  tag;     	/*一般为FDT_BEGIN_NODE或FDT_END_NODE*/
    char  name[0];

};

 
/*属性信息使用struct fdt_property结构体描述*/
struct  fdt_property  {
    fdt32_t  tag;     /*一般为FDT_PROP*/
    fdt32_t  len;     /*len为属性的值的长度（包括‘\0’，单位：字节）*/
    fdt32_t  nameoff;   /*nameoff为属性名称存储位置相对于off_dt_strings的偏移地址*/
    char  data[0];          
};
```

一个节点主要由以下几部分组成：

1. 节点开始标志：一般为FDT_BEGIN_NODE。
2. 节点路径或者节点的单元名(version小于3以节点全路径表示，version>=16以节点单元名表示)
3. 填充字段（对齐到四字节）
4. 节点属性。每个属性以宏FDT_PROP开始，后面依次为属性值的字节长度(4字节)、属性名称在字符串块中的偏移量(4字节)、属性值和填充（对齐到四字节）。
5. 如果存在子节点，则定义子节点。
6. 节点结束标志FDT_END_NODE。

节点使用`struct fdt_node_header`组织数据，属性使用`struct  fdt_property `组织数据。


![3](https://www.github.com/liao20081228/blog/raw/master/图片/Linux设备树/3.jpg)

![5](https://www.github.com/liao20081228/blog/raw/master/图片/Linux设备树/5.jpg)

## 3.4 DT strings block
&emsp;&emsp;通过节点的定义知道节点都有若干属性，而不同的节点的属性又有大量相同的属性名称，因此将这些属性名称提取出一张表，当节点需要应用某个属性名称时直接在属性名字段保存该属性名称在字符串块中的偏移量。





# 4 设备树编译与调试
&emsp;&emsp;DTC为编译工具，它可以将.dts文件编译成.dtb文件。DTC的源码位于内核的`scripts/dtc`目录，内核选中CONFIG_OF，编译内核的时候，可执行程序DTC就会被编译出来。 即scripts/dtc/Makefile中
```makefile
hostprogs-y := dtc
always := $(hostprogs-y) 
```
&emsp;&emsp;在内核的arch/arm/boot/dts/Makefile中，若选中某种SOC，则与其对应相关的所有dtb文件都将编译出来。在linux下，`make dtbs`可单独编译dtb。以下截取了TEGRA平台的一部分。
```makefile
ifeq ($(CONFIG_OF),y)
dtb-$(CONFIG_ARCH_TEGRA) += tegra20-harmony.dtb tegra30-beaver.dtb tegra114-dalmore.dtb tegra124-ardbeg.dtb 
```
## 4.1 dtc 
**命令**：dtc [options] input_file
**描述**：设备树编译器dtc将给定格式的设备树作为输入，输出另一种格式设备树，用于在嵌入式系统上引导内核。



|选项|描述|
|:--|:--|
| -h   |  显示帮助|
|-q  |不显示警告
|-qq| 不显示错误
|-qqq|不显示警告和错误|
|-I input_format|输入格式:dts ,dtb， fs |
|-o output_file|输出文件
| -O output_format|输出格式：dts ,dtb，asm|
|-V output version| 在dtb文件中保存blob版本，，默认值为17（仅与dtb和asm输出相关）。
|-d dependency_file|输出依赖文件
| -R num|for  num reserve map entries留出空间（仅与dtb和asm输出相关）
|-S bytes|使blob大小至少为bytes|
| -p bytes|填充到bytes长的blob
|-b number|设置启动CPU的编号。
|-F| 即使输入设备树树有错误，也尝试生成输出。
|-s |在输出之前对节点和属性进行排序（仅对比较树有用）
|-v|打印DTC版本并退出。|
| -H phandle_format|legacy , epapr ,both |

## 4.2 fdtget和fdtput
**命令**：fdtget [options] dt_file [node  property]...
&emsp;&emsp;&emsp;fdtget -p [options] dt file [node]...
**描述**：从二进制设备树中读取数据

|选项|长选项|描述|
|:--|:--|:--|
| -h   |  --help|显示帮助|
|-V|--version|显示版本号|
|-t type|--type arg| s=string, i=int, u=unsigned, x=hex，可选前缀:hh/b=1B, h=2B, l=4B(default)
|-p|--properties|列出每个节点的属性|
|-l|--list|列出每个节点的子节点|
|-d arg|--default arg|当没有指定属性时，要显示的默认值|

**命令**：fdtput [options] dt_file [node property [value...]]
&emsp;&emsp;&emsp;fdtput -c [options] dt_file [node...]
**描述**：写数据到二进制设备树中



|短选项|长选项|描述|
|:--|:--|:--|
| -h   |--help|  显示帮助|
| -c| --create |  如果节点不存在创建节点
|-p|--auto-path|  根据需要节点的路径自动创建节点
|-v|--verbose|显示详细过程
|-t type|--type arg| s=string, i=int, u=unsigned, x=hex，可选前缀:hh/b=1B, h=2B, l=4B(default)


## 4.3 fdtdump
* **命令**：fdtdump [options] DTB_file_name
* **描述**：输出二进制设备树的可读版本
|短选项|长选项|描述|
|:--|:--|:--|
|-d| --debug |解码文件时转储调试信息|
|-s|--scan   |浏览设备树文件|
|-h|--help   | 显示帮助并退出|
|-V|--version |显示版本号并退出|

# 5 设备树工作原理




```


```

<5> machine_desc结构
内核提供了一个重要的结构体struct machine_desc ，这个结构体在内核移植中起到相当重要的作用，内核通过machine_desc结构体来控制系统体系架构相关部分的初始化。machine_desc结构体通过MACHINE_START宏来初始化，在代码中， 通过在start_kernel->setup_arch中调用setup_machine_fdt来获取。
```cpp
struct machine_desc {
    unsigned int nr; /* architecture number */
    const char *name; /* architecture name */
    unsigned long atag_offset; /* tagged list (relative) */
    const char *const *dt_compat; /* array of device tree* 'compatible' strings */
    unsigned int nr_irqs; /* number of IRQs */
    
#ifdef CONFIG_ZONE_DMA
    phys_addr_t dma_zone_size; /* size of DMA-able area */
#endif
    
    unsigned int video_start; /* start of video RAM */
    unsigned int video_end; /* end of video RAM */
    
    unsigned char reserve_lp0 :1; /* never has lp0 */
    unsigned char reserve_lp1 :1; /* never has lp1 */
    unsigned char reserve_lp2 :1; /* never has lp2 */
    enum reboot_mode reboot_mode; /* default restart mode */
    struct smp_operations *smp; /* SMP operations */
    bool (*smp_init)(void);
    void (*fixup)(struct tag *, char **,struct meminfo *);
    void (*init_meminfo)(void);
    void (*reserve)(void);/* reserve mem blocks */
    void (*map_io)(void);/* IO mapping function */
    void (*init_early)(void);
    void (*init_irq)(void);
    void (*init_time)(void);
    void (*init_machine)(void);
    void (*init_late)(void);
#ifdef CONFIG_MULTI_IRQ_HANDLER
    void (*handle_irq)(struct pt_regs *);
#endif
    void (*restart)(enum reboot_mode, const char *);
};
 
<6>设备节点结构体
struct device_node {
    const char *name;    //设备name
    const char *type; //设备类型
    phandle phandle;
    const char *full_name; //设备全称，包括父设备名
 
    struct property *properties; //设备属性链表
    struct property *deadprops; //removed properties
    struct device_node *parent; //指向父节点
    struct device_node *child; //指向子节点
    struct device_node *sibling; //指向兄弟节点
    struct device_node *next; //相同设备类型的下一个节点
    struct device_node *allnext; //next in list of all nodes
    struct proc_dir_entry *pde; //该节点对应的proc
    struct kref kref;
    unsigned long _flags;
    void *data;
#if defined(CONFIG_SPARC)
    const char *path_component_name;
    unsigned int unique_id;
    struct of_irq_controller *irq_trans;
#endif
};
 
<7>属性结构体
struct property {
    char *name;        //属性名
    int length;        //属性值长度
    void *value;        //属性值
    struct property *next; //指向下一个属性
    unsigned long _flags; //标志
    unsigned int unique_id;
	struct bin_attribute attr；
};
```


![3](https://www.github.com/liao20081228/blog/raw/master/图片/Linux设备树/30145634_1428507890cwwV.png)
![1](https://www.github.com/liao20081228/blog/raw/master/图片/Linux设备树/20170829102235526.jpg)



------


&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文章参考了<font color=blue >[《Device Tree Usage》](http://www.devicetree.org/Device_Tree_Usage)、[《uboot之fdt介绍》](https://www.cnblogs.com/leaven/p/6295999.html)、[《蜂窝科技的几篇文章》](http://www.wowotech.net/device_model/dt_basic_concept.html)，[《Linux设备树文件结构与解析深度分析》](https://blog.csdn.net/woyimibayi/article/details/77574736)、[《我眼中的Linux设备树》](https://www.cnblogs.com/targethero/p/5053591.html)。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------