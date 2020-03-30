---
title: the back-end computer system for the medipix based Pi-mega X-ray camera
tags: RDMA
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文翻译自<font color=blue>《the back-end computer system for the medipix[^1] based Pi-mega X-ray camera》。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

# 0 摘要

巴西同步加速器与BrPhotonic和CPqD合作设计和开发了PI-MEGA，这是一种使用Medipix 3RX芯片的新X射线相机，其目标建立一个快速大型的相机以慢速Sirius的新需求。本文描述了用于接收、处理和存储图像的后端电脑系统的设计和测试。这个后端电脑系统将使用基于以太网技术的RDMA，并且必须能够以每个PI-MEGA 元素图像50Gbps~100Gps的速率处理数据。多个PI-MEGA元素可以合并产生一个大的相机。初始应用包括分层解析重构和相干衍射图像技术。

# 2 PI-MEGA概览
新的Pi-mega X射线相机被设计用来满足Sirius的实验需求。在当前原型阶段，pi-mega是1.536X1.536（240万像素，图1）X射线相机，它将以每秒1000帧的速度采集24bits图像。PI-mega相机可以组合成2X2的阵列（940万像素）来增加像素。相机具有一下特点：

* 快速读取（高大2000 fps @ 12bit）
* 深度无图像混合技术
* 55微米的方形像素
* 没有死角
* 易于维护和替换

该相机还处于原型阶段，由LNLS检测器组设计。

![图1](https://raw.githubusercontent.com/liao20081228/blog/master/图片/the_back-end_computer_system_for_the_medipix_based_Pi-mega_X-ray_camera/图1.jpg)

# 2 单个Pi-mega后端设计
对于单个PI-mega，可以设计一个含有1000Gbps的网卡和高端GPU的后端电脑。在这个系统中数据流如图2所示。

1. 原始图像数据离开Pi-mega到达网卡
2. 然后通过某些路径（例如存放到主机内存然后拷贝到GPU）到达GPU
3. 之后GPU在原始数据上做一些处理，并生成结果
4. 这个结果通过某些路径返回网卡
5. 数据离开网卡然后到达存储设备

![图2](https://raw.githubusercontent.com/liao20081228/blog/master/图片/the_back-end_computer_system_for_the_medipix_based_Pi-mega_X-ray_camera/图2.jpg)

在数据流中，到来的原始数据可能高达56.6Gbps（240w像素X24bitX1000fps）。

作为使上述数据流尽可能高效的一种方式，RDMA[^2][^3]技术用于从前端到网卡。RDMA将数据包处理从CPU转到硬件，这是为高性能、低开销网络传输特别设计的。在RDMA中，CPU只需处理控制流，请求传输和从网卡接受完成事件。数据包的建立和处理都由网卡完成[^4]。

有不同类型的网卡支持RDMA传输，具有不同的有点和缺点。

* InfiniBand：所有的网卡与InfiniBand网络支持的RDMA兼容[^2]。
* RoCE v1：基于融合以太网的RDMA是一种用于以太网卡的硬件加速RMDA的实现[^5]。RoCE V1在以太网上实现了InfiniBand网络和传输协议。有了以太网支持，他可以很容易的在FPGA中实现。
*  RoCE v2[^6]：类似于RoCE v1，但是是在以太网之上的UDP/IP的基础上实现InfiniBand传输协议，它是可路由的，且容易在硬件上实现。
* iWARP：在MAP/TCP(Marker PDU Aligned  Framing for TCP， 一个TCP之上的分组协议 )或者SCTP之上实现RDMA的网卡。iWARP网卡的主要优点是重用了TCP和具有它的优点，例如使用了熟悉的可靠的受支持的流控协议，缺点是硬件上实现复杂。
* OmniPath[^7]：支持硬件加速RDMA的Intel所有技术。
* SoftRoCE：一个基于软件的RoCE实现，对于测试RDMA但没有硬件加速以太网卡时很有用。它在Linux4.8[^8]中被合并入内核。

Pi-mega选用RoCE v1 有两个原因：首先，不像其他技术，预计以太网标准在未来几十年不会过时，其次在Pi-mega前端实现最简单。

# 3 多个Pi-mega后端

多个pi-mega实中，NVlink互连[^9]被选作用本地总线。最初目标是一个2X2阵列的pi-meag相机，这个相机将与一个具有两个PCIe网卡和4个NVlink GPU的计算机连接。在这个设置中，两个pi-mega在网络交换机的帮助下发送数据到各自的网卡。相应地降低了最大允许帧速率，以适应200 Gbps带宽中来自所有4个Pi-mega的数据。NVLink的主要优点是能够快速的接收网卡的数据和在GPU之间进行数据交换。测试的机器是 IBM/OpenPower S822LC HPC[^10],支持4 NVIDIA Pascal P100 GPU，GPU之间的NVLink连接为320Gbps，GPU与CPU之间的连接也是320Gbps。（图3）

![图3](https://raw.githubusercontent.com/liao20081228/blog/master/图片/the_back-end_computer_system_for_the_medipix_based_Pi-mega_X-ray_camera/图3.jpg)


# 4 测试
性能测试是在USA.NY的IBM 基准测试中心远程组织的。这里有带有4X EDR（100Gbps）InfiniBand链接的S822SLC HPC集群。开发了一个基准应用程序，该应用程序能够同时使机器数据路径（包括网卡PCIe总线，GPU）饱和，并在主机内存中完成处理（但不包括对存储的最终写入）。 用多台机器对多次运行进行了仿真，其中一台模拟一台pi-mega相机生成数据，另一台表示后端接收和处理数据。 在最大吞吐量模式下，后端计算机接收256GiB数据，并将其复制到GPU。GP计算校验和，并将数据和校验和拷贝到CPU内存。 计算的校验和已通过简单但缓慢的纯CPU程序进行了验证，以确保GPU代码中没有软件错误。 最终获得的吞吐量为78 Gbps，超过了单个pi-mage所需的吞吐量。

![图4](https://raw.githubusercontent.com/liao20081228/blog/master/图片/the_back-end_computer_system_for_the_medipix_based_Pi-mega_X-ray_camera/图4.jpg)

在线提供了称为rdmacp的基准程序，该程序还提供了通用RDMA库[^11]。

# 5 下一步

IBM中心的存储仅通过隔离的基准测试程序进行了测试，并且需要集成到rdmacp中。 还必须扩展rdma库以支持多个pi-mega，以便可以进行更实际的带宽测试，让pi-mega争夺网络带宽。




[^1]: R. Ballabriga et al., “The Medipix3RX: a high resolution, zero dead-time pixel detector readout chip allowing spectroscopic imaging”, Feb. 2013

[^2]: InfiniBand Architecture Specification Volume 1 Release 1.3, InfiniBand Trade Association, Mar. 2015, pp. 97-101.

[^3]:  A Remote Direct Memory Access Protocol Specification, Network Working Group IETF, Oct. 2007, pp. 4-6.

[^4]: RDMA Aware Networks Programming User Manual, Rev 1.7, Mellanox Technologies, May 2015.

[^5]: Supplement to InfiniBand Architecture Specification Volume 1 Release 1.2.1, Annex A16: RDMA over Converged

Ethernet (RoCE), InfiniBand Trade Association, Apr. 2010.

[^6]: Supplement to InfiniBand Architecture Specification Volume 1 Release 1.2.1, Annex A17: RoCEv2, InfiniBand Trade Association, Sep. 2014.

[^7]: M. S. Birrittella et al., “Intel Omni-Path Architecture Technology Overview”, Aug. 2015.

[^8]: Linux 4.8, Linux Kernel Newbies, Oct. 2016;https://kernelnewbies.org/Linux_4.8

[^9]: What is NVLink?, NVIDIA Official Blog, Nov. 2014;https://blogs.nvidia.com/blog/2014/11/14/what-is-nvlink/

[^10]: A. B. Caldeira, V. Haug, and S. Vetter, IBM Power System S822LC for High Performance Computing Introduction and Technical Overview, Oct. 2016

[^11]: RDMA copy sample file transfer application, LNLS SOL Group; https://github.com/lnls-sol/rdmacp/

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文翻译自<font color=blue>《the back-end computer system for the medipix based Pi-mega X-ray camera》。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------