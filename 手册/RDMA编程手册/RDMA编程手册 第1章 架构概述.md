---
title: RDMA编程手册 第1章 架构概述
tags: RDMA
---

------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>[《RDMA Aware Networks Programming User Manual v1.7》](https://www.mellanox.com/related-docs/prod_software/RDMA_Aware_Programming_user_manual.pdf)、[《Dotan Barak的RDMAmojo博客》](http://www.rdmamojo.com/)、《Ubuntu19.10 rdma-core套件 man手册v24.0》，我所做的工作就是更新、整理一部分内容，删除了一部分废弃的API，添加了一部分新的API。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------


# 1 InfiniBand
InfiniBand（IB）是一种高速，低延迟，低CPU开销，高效且可扩展的服务器和存储互连技术。 InfiniBand的关键功能之一是它对本机远程直接内存访问（RDMA）的支持。 InfiniBand支持服务器之间以及服务器与存储之间的数据传输，而在数据路径中无需调用主机CPU。 InfiniBand使用I/O通道进行数据通信（每台主机最多1600万个），其中每个通道提供虚拟化NIC或HCA的语义（安全性，隔离性等）。 InfiniBand使用铜缆和光纤连接，可提供各种技术或解决方案速度，范围从每个端口10Gb/s（SDR）到最高56Gb/s（FDR）。 InfiniBand的效率和可扩展性使其成为全球领先的高性能计算，云，Web 2.0，存储，数据库以及金融数据中心和应用程序的最佳性能和成本/性能互连解决方案。 InfiniBand是由IBTA组织定义和规范的标准技术。


# 2 虚拟协议互连（VPI）
Mellanox虚拟协议互连（VPI）架构为支持InfiniBand和以太网语义的网络适配器和交换机之间的通信提供了高性能，低延迟和可靠的方式。 可以将VPI适配器或交换机设置为在每个端口上传递InfiniBand或以太网语义。 例如，可以将双端口VPI适配器配置为以下选项之一：

* 具有两个InfiniBand端口的适配器（HCA）
* 具有两个以太网端口的NIC
* 一个适配器，该适配器同时具有一个InfiniBand端口和一个以太网端口

同样，VPI交换机可以仅有InfiniBand的端口，仅有以太网的端口，或具有同时工作的InfiniBand和以太网端口的混合。

基于Mellanox的VPI适配器和交换机同时支持InfiniBand RDMA和以太网RoCE解决方案。

# 3  基于聚合以太网的RDMA（RoCE）
RoCE是以太网RDMA的标准，也是IBTA组织定义和规范的标准。 RoCE为以太网提供了真正的RDMA语义，因为它不需要复杂且性能低下的TCP传输（例如，iWARP需要TCP）。

RoCE是当今最高效的低延迟以太网解决方案。 它需要非常低的CPU开销，并利用数据中心桥接以太网中的优先级流控制来实现无损连接。 自OFED 1.5.1发布以来，Open Fabrics软件已完全支持RoCE。

# 4 RDMA技术的比较
当前，共有三种支持RDMA的技术：InfiniBand，以太网RoCE和以太网iWARP。 这三种技术共享本文档中定义的通用用户API，但是具有不同的物理层和链接层。

谈到以太网解决方案，RoCE在性能，吞吐量和CPU开销方面都比iWARP具有明显的性能优势。 RoCE得到了许多领先解决方案的支持，并集成在Windows Server软件（以及InfiniBand）中。

RDMA技术基于传统网络中的网络概念，但是它们与IP网络中的对等技术有所不同。 关键区别在于RDMA提供消息传递服务，应用程序可以使用该消息传递服务直接访问远程计算机上的虚拟内存。 消息传递服务可用于进程间通信（IPC），与远程服务器的通信以及使用上层协议（ULP）与存储设备进行通信，例如RDMA的iSCSI扩展（ISER）和SCSI RDMA协议（SRP），存储消息 Block（SMB），Samba，Lustre，ZFS等。

RDMA通过堆栈绕过和避免复制提供了低的延迟，降低了CPU利用率，减少了内存带宽瓶颈，并提供了高带宽利用率。 RDMA传递的主要好处来自将RDMA消息传递服务呈现给应用程序的方式以及用于传输和传递这些消息的基础技术。 RDMA提供基于通道的IO。 该通道允许使用RDMA设备的应用程序直接读写远程虚拟内存。

在传统的套接字网络中，应用程序通过API来从操作系统请求网络资源，该API代表它们进行事务处理。 但是，RDMA使用OS建立通道，然后允许应用程序直接交换消息，而无需进一步的OS干预。 消息可以是RDMA  READ，RDMA WRITE操作或 SEND/REVICE操作。 IB和RoCE都支持多播传输。

IB链路层提供功能，例如用于拥塞控制的基于信用的流量控制机制。 它还允许使用虚拟通道（VL），从而简化高层协议和高级服务质量。 它保证了VL中沿给定路径的强有序性。 IB传输层提供可靠性和交付保证。

IB使用的网络层具有的功能使即使在物理上位于不同服务器上的应用程序之间也可以轻松地在应用程序的虚拟内存之间直接传输消息。 因此，最好将IB传输层与软件传输接口的组合视为RDMA消息传输服务。包括软件传输接口在内的整个堆栈都包含IB消息传递服务。

![图1](https://raw.githubusercontent.com/liao20081228/blog/master/图片/RDMA_Aware网络编程用户手册/图1.jpg)

最重要的一点是，每个应用程序都可以直接访问结构中设备的虚拟内存。 这意味着应用程序不需要向操作系统发出传输消息的请求。 与此形成对比的是传统网络环境，在传统网络环境中，共享网络资源由操作系统拥有，而用户应用程序无法访问。 因此，应用程序必须依赖于操作系统的参与，才能将数据从应用程序的虚拟缓冲区空间中移出，并通过网络堆栈移到网线上。 同样，在另一端，应用程序必须依靠操作系统来代表它在网络上检索数据并将其放置在其虚拟缓冲区空间中。

![图2](https://raw.githubusercontent.com/liao20081228/blog/master/图片/RDMA_Aware网络编程用户手册/图2.jpg)

TCP/IP /以太网是面向字节流的传输，用于在套接字应用程序之间传递信息字节。 TCP/IP在设计上是有损的，但使用传输控制协议（TCP）来实现可靠性方案。 TCP/IP需要对每个操作进行操作系统（OS）干预，包括在线路两端复制缓冲区。 在面向字节流的网络中，丢失了消息边界的观点。 当应用程序要发送数据包时，操作系统将字节放入属于操作系统的主内存中的匿名缓冲区中，当字节传输完成时，操作系统会将其缓冲区中的数据复制到应用程序的接收缓冲区中。 每次到达数据包时都会重复此过程，直到接收到整个字节流为止。 TCP负责重新传输由于拥塞而丢失的任何数据包。

在IB中，完整的消息直接传递到应用程序。 一旦应用程序请求传输RDMA读或写，IB硬件就会根据需要将出站消息划分为大小由结构路径最大传输单元确定的数据包。 这些数据包通过IB网络传输，并直接传递到接收应用程序的虚拟缓冲区中，在此将它们重新组合为完整的消息。 一旦接收到整个消息，就会通知接收应用程序。 因此，直到整个消息传递到接收应用程序的缓冲区之前，发送应用程序和接收应用程序都不会涉及。

# 5  关键组件

仅在部署IB和RoCE的优势的背景下介绍这些内容。我们不讨论电缆和连接器。

**主机通道适配器**

HCAs提供了IB终端节点（例如，服务器）连接到IB网络的点。这些等效于以太网（NIC）卡，但它们的功能更多。 HCA在操作系统的控制下提供地址转换机制，该机制允许应用程序直接访问HCA。相同的地址转换机制是HCA代表用户级应用程序访问内存的方式。该应用程序引用虚拟地址，而HCA可以将这些地址转换为物理地址，以影响实际的消息传输。

**范围扩展器**

InfiniBand范围扩展是通过将InfiniBand流量封装到WAN链路上并扩展足够的缓冲区信用来确保WAN上的完整带宽来实现的。

**子网管理器**

InfiniBand子网管理器为连接到InfiniBand结构的每个端口分配本地标识符（LID），并根据分配的LID开发路由表。 IB子网管理器是软件定义网络（SDN）的概念，它消除了互连的复杂性，并允许创建超大规模的计算和存储基础结构。

**交换机**

IB交换机在概念上类似于标准网络交换机，但旨在满足IB性能要求。 它们实现了IB链路层的流控制，以防止数据包丢失，并支持避免拥塞和自适应路由功能以及高级服务质量。 许多交换机都包含一个子网管理器。 至少需要一个Subnet Manager才能配置IB结构。

# 6 对现有的应用程序和ULPs的支持

使用IB上的IP（IPoIB）或IB上的以太网（EoIB）或RDS ULP，可以使IP应用程序在InfiniBand架构上运行。 通过iSER，SRP，RDS，NFS，ZFS，SMB等支持存储应用程序。 MPI和Network Direct也都支持ULP，但不在本文档范围之内。

# 7 引用

* IBTA Intro to IB for End Users
<http://members.infinibandta.org/kwspub/Intro_to_IB_for_End_Users.pdf>
* Mellanox InfiniBandFAQ_FQ_100.pdf
<http://www.mellanox.com/pdf/whitepapers/InfiniBandFAQ_FQ_100.pdf>
* Mellanox WP_2007_IB_Software_and_Protocols.pdf
<http://www.mellanox.com/pdf/whitepapers/WP_2007_IB_Software_and_Protocols.pdf>
* Mellanox driver software stacks and firmware are available for download from Mellanox
Technologies’ Web pages: <http://www.mellanox.com>


------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>[《RDMA Aware Networks Programming User Manual v1.7》](https://www.mellanox.com/related-docs/prod_software/RDMA_Aware_Programming_user_manual.pdf)、[《Dotan Barak的RDMAmojo博客》](http://www.rdmamojo.com/)、《Ubuntu19.10 的rdma-core套件man手册v24.0》，我所做的工作就是更新、整理一部分内容，删除了一部分废弃的API，添加了一部分新的API。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

