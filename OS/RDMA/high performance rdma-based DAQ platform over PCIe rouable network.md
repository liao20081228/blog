---
title:  high performance rdma-based DAQ platform over PCIe rouable network
tags: RDMA
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文翻译自<font color=blue>《 high performance rdma-based DAQ platform over PCIe routable network》。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

# 0 摘要
Current and upcoming 2D X-ray detectors for synchrotron radiation applications are capable of producing data rates in the range of 1-100 GB/s. Existing industrial protocols do not provide suitable data acquisition solutions for handling efficiently such a high-throughput data streams. A generic and scalable RDMA-based data acquisition platform called RASHPA, designed to address these detector needs, is introduced in this paper. The FPGA implementation as well as the Linux-based software stack is detailed. Finally, the paper presents a demonstrator integrating RASHPA over a routable PCIe network.

当前和即将推出的用于同步加速器辐射应用的2D X射线探测器能够产生1-100 GB / s的数据速率。 现有的工业协议没有提供用于有效地处理这种高吞吐量数据流的合适的数据采集解决方案。 本文介绍了一种通用的，可扩展的基于RDMA的数据采集平台，称为RASHPA，旨在满足这些检测器的需求。 详细介绍了FPGA实现以及基于Linux的软件堆栈。 最后，本文提出了在可路由PCIe网络上集成RASHPA的演示。


# 1 引言

The improvements of integrated circuits manufacturing technologies and processes, applied to the last generations of 2D X-ray detectors, result in a significant increase in the produced data rates. The ESRF has undertaken the implementation of a generic and scalable data acquisition framework as one of key essential components for the development of new advanced high performance detectors for scientific applications [^1].

应用于最新几代2D X射线探测器的集成电路制造技术和工艺的改进导致产生的数据速率显著提高。 ESRF已经实施了通用且可扩展的数据采集框架，将其作为开发用于科学应用的新型先进高性能探测器的关键的组成部分之一[^1]。

One of the key and specific features this new framework is the use of remote direct memory access (RDMA) for fast data transfer. RDMA consists on the transfer of data from the memory of one host or device into that of another one without any intervention of the CPU. This permits high-throughput, low-latency networking. Companies are investing more and more into this feature, already applied to high performance computing, by integrating it into their network cards and communication adapters. Some of the available technical solutions are Infiniband [^2], RDMA over Converged Ethernet (RoCE) [^3] and internet Wide Area RDMA Protocol (iWARP) [^4], to name few.

这个新框架的关键和特定功能之一是使用远程直接内存访问（RDMA）进行快速数据传输。 RDMA包括将数据从一台主机或设备的内存传输到另一台主机或设备的内存，而无需CPU的干预。 这允许高吞吐量，低延迟的网络。 通过将功能集成到其网卡和通信适配器中，公司正在对该功能进行越来越多的投资(已经应用在高性能计算)。 一些可用的技术解决方案包括无线带宽RDMA（IB） [^2] ，融合以太网RDMA（RoCE）[^3] 和 互联网广域RDMA协议（iWARP）[^4] ，仅举几例。

The Peripheral Component Interconnect Express (PCIe) [^5] bus is a high-speed serial computer expansion bus standard, which uses shared parallel bus architecture, in which the PCI host and all devices share a common set of addresses. In other words, PCIe have a direct access to the memory of the system. For RDMA based applications, PCIe is an ideal option due to its reliability, scalability, low latency and native integration. In fact, over the years, PCIe has become the default peripheral interconnect of x86 based platforms, it has a built-in support for control flow, data integrity and packet ordering, thus no need for additional protocol layers. In addition to that, its bandwidth can be adjusted depending on the number of used lanes. Many initiatives have been launched to start using the PCIe-based RDMA to accelerate communications in data centres [^6][^7][^8].

外围组件互连快速（PCIe）[^5]总线是高速串行计算机扩展总线标准，它使用共享的并行总线体系结构，其中PCI主机和所有设备共享一组公共地址。 换句话说，PCIe可以直接访问系统的内存。 对于基于RDMA的应用程序，PCIe是其理想的选择，因为它具有可靠性，可伸缩性，低延迟和本机集成。 实际上，多年来，PCIe已成为基于x86平台的默认外围互连，它内置了对控制流，数据完整性和数据包排序的支持，因此无需其他协议层。 除此之外，它的带宽可以根据使用的通道数进行调整。 已经启动了许多计划来开始使用基于PCIe的RDMA来加速数据中心的通信 [^6][^7][^8]。

Despite these benefits, the limited availability of PCIe over cable products and the lack of standardization of optical cabling form is still an issue [^1].

尽管有这些好处，但PCIe在光缆产品上的可用性有限，光缆形式的标准化仍是一个问题[^1]。

RASHPA (RDMA-based Acquisition System for High Performance Applications) is the generic framework for detector data acquisition currently under development at the ESRF. It is optimised for the transfer of 2D detector data, i.e images, and it relies completely on RDMA mechanisms. When implemented over a low latency PCIe over cable network, RASHPA is able to push data, at very high speed into the address space of one or several backend computers. The scheme provides a high standardizationlevel in the data transmission pipeline from the detector up to the software application for further processing, visualization or storage.

RASHPA（用于高性能应用的基于RDMA的采集系统）是ESRF当前正在开发的用于探测器数据采集的通用框架。它针对2D检测器数据(即图像)的传输进行了优化，并且完全依赖RDMA机制。当通过光缆网络上的低延迟PCIe实现时，RASHPA能够以很高的速度将数据推送到一台或几台后端计算机的地址空间中。该方案在从检测器到软件应用程序的数据传输管道中提供了更高的标准化水平，以进行进一步的处理，可视化或存储。

This paper details the FPGA implementation of RASHPA at the detector side, as well as its carefully designed Linux software stack. In addition to that, a demonstrator integrating RASHPA over a routable PCIe over cable network is also presented.

本文详细介绍了RASHPA在检测器端的FPGA实施及其精心设计的Linux软件堆栈。除此之外，还展示了在电缆网络上通过可路由PCIe集成RASHPA的演示。

The paper is organised as follows: Section 2 introduces the RASHPA concept and architecture. Section 3, presents the hardware and software implementation of RASHPA and the interaction between both. Section 4 shows the RASHPA prototype and experimental results. Conclusions and future perspectives are discussed in section 

本文的组织方式如下：第2节介绍RASHPA的概念和体系结构。第3节介绍RASHPA的硬件和软件实现以及两者之间的交互。第4部分显示了RASHPA原型和实验结果。结论和未来展望将在第5节中讨论。

# 2 RASHPA的概念

As a framework, RASHPA defines a set of functional concepts as well as the hardware and software interfaces. It also implements a generic, non hardware specific middleware running on the backend computers. For practical implementation, the ESRF is also developing a set of hardware blocks that allow building and integrating the required functionality in the in-house developed detectors in conformity with the RASHPA specifications.

作为一个框架，RASHPA定义了一组功能概念以及硬件和软件接口。它还实现了在后端计算机上运行的通用，非硬件特定的中间件。对于实际实施，ESRF还开发了一组硬件模块，这些模块允许按照RASHPA规范在内部开发的检测器中构建和集成所需的功能。

![图1](https://raw.githubusercontent.com/liao20081228/blog/master/图片/high_performance_rdma-based_DAQ_platform_over_PCIe_rouable_network/图1.jpg)

RASHPA allows detectors to push data (images, regions of interest (ROI), metadata, events etc...) directly into one or more backend computers as depicted in Figure1. RASHPA’s main properties are its scalability, flexibility and high performance. It is intended to have an adjustable bandwidth that can be compatible with any backend computer. Although the examples in this paper are based on PCIe, RASPHA is in principle independent of a particular type of data link, as long as the data link supports RDMA and routability over a network to dispatch the data to multiple destinations.

RASHPA允许检测器将数据（图像，感兴趣区域（ROI），元数据，事件等）直接推送到一台或多台后端计算机中，如图1所示。 RASHPA的主要属性是其可扩展性，灵活性和高性能。它旨在具有可与任何后端计算机兼容的可调带宽。尽管本文中的示例基于PCIe，但是RASPHA原则上独立于特定类型的数据链路，只要数据链路支持RDMA和网络上的可路由性以将数据分派到多个目的地。

![图2](https://raw.githubusercontent.com/liao20081228/blog/master/图片/high_performance_rdma-based_DAQ_platform_over_PCIe_rouable_network/图2.jpg)

RASHPA allows Detector Modules (DM) to push data into Backend Computers (BC). Although the usual data destinations are RAM system memory buffers, other possible destinations under investigation are Graphical Processing Units (GPU), coprocessors and disk controllers.BC’s receiving data from the DM are called Data Receivers (DR). BC’s configuring and initializing RASHPA are called System Managers (SM) which can act as a DR as well. librashpa, a software library is responsible of providing to the detector software application the required functions to configure initialize and control RASHPA hardware. librashpa should be integrated in all the DRs and the SM. It does not provide any kind of intercommunications between multiple DRs and does not have any link with the DM. This behaviour is illustrated in Figure 2.


RASHPA允许检测器模块（DM）将数据推入后端计算机（BC）。尽管通常的数据目标是RAM系统内存缓冲区，但其他可能要研究的目标是图形处理单元（GPU），协处理器和磁盘控制器。BC从DM接收数据的过程称为数据接收器（DR）。 BC的配置和初始化RASHPA的过程称为系统管理器（SM），它也可以表现为DR。 librashpa，一个软件库，负责向检测器软件应用程序提供配置初始化和控制RASHPA硬件所需的功能。 librashpa应该集成在所有DR和SM中。它不提供多个DR之间的任何类型的互通，并且与DM没有任何链接。此行为如图2所示。



![图3](https://raw.githubusercontent.com/liao20081228/blog/master/图片/high_performance_rdma-based_DAQ_platform_over_PCIe_rouable_network/图3.jpg)

Figure 3 is a conceptual diagram of how the RASHPA framework fits the data transmission pipeline of a basic detector system consisting of a unique DM and a single BC. From a hardware point of view, the RASHPA controller consists of specific logic interfacing the detector readout electronics as well as a set of hardware blocks handling data transmission. These blocks are known as channels. Two types of configurable channels can be identified in RASHPA: data and event channels. Each data channel, for which there can be multiple instances in a single RASHPA controller, is responsible of transferring detector data to a pre-configured address space within one or several DRs. A single event channel should acknowledge the data receiver or system manager about any event occurring at the RASHPA controller. Typical asynchronous events are errors, end of transmission conditions, etc.


图3是RASHPA框架如何适合由唯一DM和单个BC组成的基本检测器系统的数据传输管道的概念图。从硬件的角度来看，RASHPA控制器包括与检测器读出电子设备接口的特定逻辑，以及一组处理数据传输的硬件模块。这些块称为通道。在RASHPA中可以识别两种类型的可配置通道：数据通道和事件通道。每个数据通道（单个RASHPA控制器中可能有多个实例）负责将检测器数据传输到一个或几个DR内的预配置地址空间。单个事件通道应向数据接收器或系统管理器确认RASHPA控制器上发生的任何事件。典型的异步事件包括错误，传输条件终止等。


The main RASHPA components in the detector head are:

* A scheduler in charge of generating and dispatching
memory transfer requests (data and event
channels).
* DMA engines to access workstation memory.
* An embedded CPU in charge of handling configuration
and control requests from the workstation.

In addition to that, RASHPA defines and implement a librashpa on the DR that initialises and manages the data transfer process and provides a standard programming interface to client applications.


探测器头中的RASHPA主要组件是：

* 调度程序：负责生成和调度内存传输请求（数据和事件通道）。
* DMA引擎：访问工作站内存。
* 嵌入式CPU：负责处理来自工作站的配置和控制请求。

除此之外，RASHPA在DR上定义并实现一个librashpa，以初始化和管理数据传输过程，并为客户端应用程序提供标准的编程接口。

# 3  RASHPA实现
Two types of implementation should be identified: hardware and software implementations.

应该区分两种类型的实现：硬件和软件实现。


From the hardware point of view, the FPGA implementation of RASHPA is independent of the type of high speed data link used to perform the data acquisition process.

从硬件的角度来看，RASHPA的FPGA实现与用于执行数据采集过程的高速数据链路的类型无关。

The FPGA implementation presented in the following paragraphs is based on an AXI subsystem implemented on a Xilinx KC705 development board [^9]. It is considered as a separate intellectual property (IP) that can be easily integrated within the ESRF control systems. In this implementation, a PCIe over cable data link was chosen for the reasons mentioned in the introduction.

以下段介绍的FPGA实现基于在Xilinx KC705开发板上实现的AXI子系统[^9]。 它被视为独立的知识产权（IP），可以轻松地集成到ESRF控制系统中。 在此实现中，出于引言中提到的原因，选择了光缆数据链路上的PCIe。

## 3.1 FPGA实现

![图4](https://raw.githubusercontent.com/liao20081228/blog/master/图片/high_performance_rdma-based_DAQ_platform_over_PCIe_rouable_network/图4.jpg)

The AXI subsystem implementing RASHPA on the KC705 development board, Figure 4, consists of several IPs, some from Xilinx and others designed ad-hoc:

 * Detector Interface Unit (DIU): This unit reads the image data and store them in a DDR3 memory on board.
 
 * Block Generation Unit (BGU): It contains all the data channels that are configured prior starting the data transfer process. This unit outputs 256-bit packets containing all the information about each transfer, such as source address in the DDR, destination address index of a physical address in an address translation unit, bytes to transfer, etc.

* Address Translation Unit (ATU): This is an internal memory containing the physical address of the available memory buffers on each DR.

* Block Split Unit (BSU): It is responsible of reading and analyzing BGU’s output data, getting corresponding addresses from the ATU, and configuring the Central Direct Memory Access (CDMA), and the PCIe IPs.
* CDMA: a Xilinx built-in direct memory access IP.

* PCIe: a Xilinx built-in transfer layer IP.

All the previously described IPs are configured through the e-bone interconnect [^10] which is the ESRF standard interconnect used for control applications.



在KC705开发板上实现RASHPA的AXI子系统（图4）由几个IP组成，其中一些来自Xilinx，另一些则是专门设计的：

* 检测器接口单元（DIU）：该单元读取图像数据并将其存储在板上的DDR3内存中。

* 块生成单元（BGU）：它包含在开始数据传输过程之前配置的所有数据通道。该单元输出256位数据包，其中包含有关每次传输的所有信息，例如DDR中的源地址，地址转换单元中物理地址的目标地址索引，要传输的字节等。

* 地址转换单元（ATU）：这是一个内部存储器，包含每个DR上可用内存缓冲区的物理地址。

* 块拆分单元（BSU）：它负责读取和分析BGU的输出数据，从ATU获取相应的地址，并配置中央直接内存访问（CDMA以及 PCIe IP。

* CDMA：Xilinx内置的直接内存访问IP。

* PCIe：Xilinx内置的传输层IP。

所有先前描述的IP都是通过e-bone互连[^10]配置的，e-bone互连是ESRF用于控制应用的标准互连。

## 3.2 基于电缆的PCIe

A RASHPA system must be able of sending data to multiple destinations, thus it requires a routable network. As this implementation has adopted PCIe for data transmission, one requires to use PCIe switches for packet routing. Such components are not so common, but one could find some of them such as the Dolphin IXS600 from dolphinics [11] and OSS-PCIe-1U-SW-x4-2.0 from one stop systems [^12].

RASHPA系统必须能够将数据发送到多个目的地，因此它需要可路由的网络。 由于此实现采用PCIe进行数据传输，因此需要使用PCIe交换机进行数据包路由。 这样的组件并不常见，但是可以从dolphinics[^11]中找到，如 Dolphin IXS600 [^11]，或从一站式系统中找到，如OSS-PCIe-1U-SW-x4-2.0 [^12]。

For the demonstration purpose, we made the decision to use the PXH810 board [^13], presented in Figure 5 which integrates a PLX8749 PCIe switch.

出于演示目的，我们决定使用图5所示的PXH810板[^13]，该板集成了PLX8749 PCIe交换机。

![图5](https://raw.githubusercontent.com/liao20081228/blog/master/图片/high_performance_rdma-based_DAQ_platform_over_PCIe_rouable_network/图5.jpg)

The board comes with a Linux driver that is not compatible for RASHPA’s application. We have then reimplemented the driver of the PLX switch in order to fit the RASHPA needs.

该评估板带有与RASHPA的应用程序不兼容的Linux驱动程序。 然后，我们重新实现了PLX开关的驱动程序，以满足RASHPA的需求。

The PXH810 board is plugged in a BC, with one PCIe cable adapter board. The adapter board links the current BC to the DM, whereas the PXH810 connects the current BC to another BC. This way, RASHPA can send data to two BCs via the PCIe switch.

PXH810板通过一个PCIe电缆适配器板插入BC。 适配器板将当前BC连接到DM，而PXH810将当前BC连接到另一个BC。 这样，RASHPA可以通过PCIe交换机将数据发送到两个BC。


## 3.3  工作站中间件

![图6](https://raw.githubusercontent.com/liao20081228/blog/master/图片/high_performance_rdma-based_DAQ_platform_over_PCIe_rouable_network/图6.jpg)

The workstation middleware called librashpa, a Linux library, is in charge of providing the client application with data transmission services through a well defined application programming interface (API). An example of client is the LIMA data acquisition and detector control library widely used at ESRF and other synchrotron radiation facilities [^14]. The middleware layers depicted in Figure 6 support the following characteristics:

* Configuration: this layer is in charge of building description of the hardware such as when and where to transmit data.

* Device abstraction: Since RAHSPA is link independant, the PCIe endpoint discussed earlier can be replaced with any other data link such as Ethernet, Infiniband, etc... The device abstraction layer hides low-level details and provides the software with a generic interface to access the underlying devices. While most of the layer is implemented in user space, special operations such as interrupt handling require a thin driver.

* Memory management: RDMA transfer requires to work with the BC’s physical address space, however typical client application buffers are allocated in the process virtual address space. The middleware thus provides a virtual memory allocator on top of an internally managed physical memory allocator.

* Unified Event Model: There are multiple sources of events that would make sequential programming difficult. For instance,  data transmission completion is signalled by the detector using an interrupt along with auxiliary data;  LINUX informs about PCIe device adding or removal using system notifications; Timers and callbacks may be registered by the client application itself... For those reasons, the middleware programming model is largely event based and provides software abstractions that unify the different event sources.


名为librashpa的工作站中间件（Linux库）负责通过定义良好的应用程序编程接口（API）为客户端应用程序提供数据传输服务。客户端的一个例子是在ESRF和其他同步加速器辐射设施中广泛使用的LIMA数据采集和检测器控制库[^14]。图6中描绘的中间件层支持以下特征：

* 配置：此层负责构建硬件描述，例如何时何地传输数据。

* 设备抽象：由于RAHSPA是独立于链路的，因此前面讨论的PCIe端点可以替换为任何其他数据链路，例如以太网，Infiniband等。设备抽象层隐藏了底层细节，并为软件提供了通用接口访问底层设备。尽管大多数层是在用户空间中实现的，但是诸如中断处理之类的特殊操作需要精简的驱动程序。

* 内存管理：RDMA传输需要使用BC的物理地址空间，但是典型的客户端应用程序缓冲区是在进程虚拟地址空间中分配的。因此，中间件在内部管理的物理内存分配器之上提供了虚拟内存分配器。

* 统一事件模型：有多种事件源，这将使顺序编程变得困难。例如，检测器使用中断和辅助数据来发出数据传输完成的信号。 LINUX使用系统通知来通知有关PCIe设备添加或删除的信息；计时器和回调可能由客户端应用程序本身注册...出于这些原因，中间件编程模型主要基于事件，并提供统一不同事件源的软件抽象。

# 4 RSAHPA高级原型

The first RASHPA prototype was developed in the frame of the European project CRISP [1]. In the first version, RASHPA supported the data transfer from a DM to one BC. It has been tested and validated using a data generator emulating the detector behaviour.

第一个RASHPA原型是在欧洲项目CRISP [^1]的框架中开发的。在第一个版本中，RASHPA支持从DM到一个BC的数据传输。它已经使用模拟探测器行为的数据生成器进行了测试和验证。

![图7](https://raw.githubusercontent.com/liao20081228/blog/master/图片/high_performance_rdma-based_DAQ_platform_over_PCIe_rouable_network/图7.jpg)

In the current version, Figure 7, RASHPA has been integrated to the SMARTPIX, a Medipix3 based detector currently under development at the ESRF. In that perspective, the KC705 development board has been replaced with a commercial PFPKX7 board from Techway [^15]. The new of prototype version also supports the multiple destinations feature thanks to the use of PCIe switches. Copper cables were used to build and test the routable network although fibre optics cabling is also available. In this implementation, two types of PC are used: a so-called detector PC having 64GB of internal DDR, and Gen3x16 PCIe endpoint, and an industrial PC with only 4GB on internal DDR and Gen1x1 PCIe endpoint. The PFPkx7 supports Gen2x4 PCIe, thus the link connecting the detector to the detector PC is negotiated to Gen2x4 whereas the virtual link connecting the detector to the industrial PC is limited to Gen1x1.

在当前版本中，图7，RASHPA已集成到SMARTPIX中，SMARTPIX是ESRF目前正在开发的基于Medipix3的检测器。从这个角度来看，KC705开发板已被Techway [^15]的商用PFPKX7板取代。由于使用了PCIe交换机，因此新的原型版本还支持多目标功能铜缆用于构建和测试可路由网络，虽然也可以使用光纤电缆。在此实现中，使用两种类型的PC：具有64GB内部DDR和Gen3x16 PCIe端的所谓检测器PC，以及在1GB内部DDR和Gen1x1 PCIe端的工业PC。 PFPkx7支持Gen2x4 PCIe，因此将检测器连接到检测器PC的链接协商为Gen2x4，而将检测器连接到工业PC的虚拟链接则限制为Gen1x1。

![图8](https://raw.githubusercontent.com/liao20081228/blog/master/图片/high_performance_rdma-based_DAQ_platform_over_PCIe_rouable_network/图8.jpg)

Due to the difficulty of producing real detector images with the current SMARTPIX hardware without an X-ray source, a Java image generator application was developed, where the ESRF logo was used as a reference image. A screen shot of an experiment is illustrated in Figure 8. In this experiment two data channels where configured to send the original image to the detector PC and a region of interest of that image to the industrial PC based on some configuration sent by the system manager to the RASHPA controller.

由于在没有X射线源的情况下使用当前的SMARTPIX硬件难以生成真实的探测器图像，因此开发了Java图像生成器应用程序，其中将ESRF徽标用作参考图像。图8显示了一个实验的屏幕截图。在该实验中，两个数据通道配置为基于系统管理器发送给RASHPA控制器的某些配置将原始图像发送到检测器PC和将该图像的感兴趣区域发送到工业PC。

Table 1, shows the data throughput of the data acquisition process. Two ways were investigated to measure the data throughput. The first one measures the throughput internally within the FPGA, whereas the second one detects whenever data are available to the application.

表1显示了数据采集过程的数据吞吐量。 研究了两种方法来测量数据吞吐量。 第一个在FPGA内部测量吞吐量，而第二个在数据可用于应用程序时进行检测。

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;**表1 测试数据吞吐量**

||FPGA内带宽|数据可用于应用程序时带宽|
|:--|:--|:--|
|检测器PC GEN2X4|10.88Gbps<br />68%|8.18Gbps<br />51.15%|
|工业PC GEN1X1|1.44Gbps<br />71.99%|1.12Gbps<br />56.28%|

Measured throughput when Gen2x4 PCIe endpoint is the target is 68% of the Gen2x4 maximum bandwidth when measured within the FPGA and 51.15% when measured at the application level. Similarly measured bandwidth at the Gen1x1 is 71.99% and 56.28% when measured at the FPGA and application level respectively. These losses in the data throughput are due to the configuration of the DMA which is imposed by RASHPA as well as the restriction of the PCIe packet size limited to 4Kbytes. Some other latency can be added when throughput is measured at the application level due to the processor execution time.

当在FPGA中进行测量时，以Gen2x4 PCIe端点为目标的测量吞吐量为Gen2x4最大带宽的68％，在应用程序级测量为51.15％。 类似地，当在FPGA和应用程序级别进行测量时，在Gen1x1处测量的带宽分别为71.99％和56.28％。 数据吞吐量的这些损失是由于RASHPA施加的DMA配置以及PCIe数据包大小限制为4 KB所致。 当由于处理器执行时间而在应用程序级别测量吞吐量时，可能会添加一些其他延迟。

# 5 结论和未来工作

In this paper was described a generic and scalable RDMA-based data acquisition platform called RASHPA and its FPGA implementation as well as its carefully designed Linux-based software stack. A demonstrator integrating RASHPA with the ESRF SMARTPIX 2D X-ray detector over a routable PCIe network was presented.

在本文中，描述了一种称为RASHPA的通用且可扩展的基于RDMA的数据采集平台和其中的FPGA实现以及经过精心设计的基于Linux的软件堆栈。展示了通过可路由PCIe网络将RASHPA与ESRF SMARTPIX 2D X射线探测器集成在一起的演示器。

The development and the realisation of the prototypes allowed to validate the basic RASHPA concepts and objectives. The obtained results showed the capability of RASHPA to send detector images as well as a region of interest of these images to multiple destinations simultaneously. Data throughput was around 53% when measuring the bandwidth at the application level whereas when measuring it at the FPGA level it reached 70%.

原型的开发和实现可以验证RASHPA的基本概念和目标。获得的结果表明RASHPA能够将检测器图像以及这些图像的关注区域同时发送到多个目的地。在应用程序级别测量带宽时，数据吞吐量约为53％，而在FPGA级别测量带宽则达到70％。

In future work is planned prototyping an RDMA-based protocol over 100G Ethernet network in order to replace the PCIe link to overcome the strong limitations of available commercial off-the-shelf hardware, in particular high performance switches. Another aspect under study is the application of the RDMA techniques in RASHPA to send data directly from the detector to GPUs in the data receivers.

在将来的工作中，计划在100G以太网上对基于RDMA的协议进行原型设计，以取代PCIe链路，以克服现有商用现货硬件（尤其是高性能交换机）的强大限制。研究的另一方面是RASHPA中RDMA技术的应用，可将数据直接从检测器发送到数据接收器中的GPU。

# 6 致谢
This project has received funding from the European Union’s Horizon 2020 research and innovation programme under grant agreement No. 654220.

该项目已获得欧盟Horizon 2020研究与创新计划的资助，资助号为654220。


------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文翻译自<font color=blue>《 high performance rdma-based DAQ platform over PCIe routable network》。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

[^1]:F. Le Mentec et al., RASHPA : A data acquisition framework for 2D X-Ray detectors, ICALEPCS proceeding, San
Francisco, 2013

[^2]: Mellanox Technologies, Introduction to Infiniband, white paper, Document Number 2003WP.

[^3]: Mellanox Technologies, RDMA/ROCE solutions,https://community.mellanox.com/docs/DOC-2283.

[^4]: Intel corp., Understanding iWARP : Delivering low latency Ethernet, 2015.

[^5]: PCI-SIG, PCI Express base specification revision 3.0, November 10, 2010.

[^6]: Zang et al., PROP: using PCIe-Based RDMA to accelerate rack-scale communications in data centers, IEEE International conference on Parallel and Distributed Systems (ICPADS),, January 2016, DOI:10.1109/ICPADS.2015.65

[^7]: Boffi et al., PCIe-based network architectures over optical fiber links: an insight from the advent project, 18th Italian National Conference on Photonic technologies (Fotonica 2016), June 2016, Rome, Italy, DOI:10.1049/cp.2016.0883

[^8]: J.F. Zazo et al., A PCIe DMA engine to support the virtualization of 40 Gbps FPGA-accelerated network appliances,IEEE International Conference on Reconfigurable Computing and FPGAs (ReConFig), Dec. 2015, Mexico City, Mexico, DOI:10.1109/ReConFig.7393334

[^9]: https://www.xilinx.com/products/boards-andkits/ek-k7-kc705-g.html

[^10]: https://www.ohwr.org/projects/e-bone

[^11]: http://www.dolphinics.com/products/IXS600.html

[^12]: https://www.onestopsystems.com/product/pcie-x4-gen2-10-port-switch

[^13]: http://www.dolphinics.com/products/PXH810.html

[^14]: A. Homs et al., LIMA: Acquiring data with imaging detectors, in Proc. ICALEPCS’11, Grenoble, France, Oct. 2011,paper WEMAU011, pp. 676-679.

[^15]: http://www.techway.fr/fichedetaillee?id_categorie=1&id_produit=118