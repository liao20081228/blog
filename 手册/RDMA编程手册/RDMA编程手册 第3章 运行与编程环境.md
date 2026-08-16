---
title: RDMA编程手册 第3章 运行与编程环境
tags: RDMA
---

------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>[《RDMA Aware Networks Programming User Manual v1.7》](https://www.mellanox.com/related-docs/prod_software/RDMA_Aware_Programming_user_manual.pdf)、[《Dotan Barak的RDMAmojo博客》](http://www.rdmamojo.com/)、《Ubuntu19.10 rdma-core套件 man手册v24.0》，我所做的工作就是更新、整理一部分内容，删除了一部分废弃的API，添加了一部分新的API。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

# 0 简介

# 1 OFED包
# 2 发行包（Ubuntu）

rdma-core
* 描述：RDMA核心用户空间基础设施和文档
  * 该软件包为系统使用Linux内核的远程直接内存访问（RDMA）子系统提供基本的启动时支持，该子系统包括InfiniBand，iWARP和基于聚合以太网（RoCE）的RDMA。
  * 包括几个内核RDMA支持守护程序：
    * 监视RDMA设备更改和/或主机名更改并基于这些更改更新RDMA设备的节点描述的rdma-ndd守护程序。
    * iWARP端口映射程序守护程序（iwpmd），它在用户空间中为iWARP驱动程序提供内核支持服务，以通过标准套接字接口声明TCP端口。
* 依赖：lsb-base , udev, perl, libc6, libnl-3-200 , libsystemd0, libudev1

rdmacm-utils
* 描述：基于librdmacm1开发的实用程序。比如rping和 udaddy。
* 依赖：libc6，libibverbs1，librdmacm1

librdmacm1
* 描述：
  * librdmacm是一个库，使用RDMA适配器时，该库允许应用程序设置可靠的连接和不可靠的数据报传输。它提供了与传输无关的接口，相同的代码可用于InfiniBand和iWARP适配器。该接口基于套接字，但适用于队列对（QP）的基本语义：通信必须使用特定的RDMA设备，并且数据传输基于消息。
  * librdmacm仅提供通信管理（连接设置和拆除），并与libibverbs提供的动词接口配合使用，后者提供用于实际传输数据的接口。
* 依赖：libibverbs1， libc6

librdmacm-dev
* 描述：librdmacm1的开发文件
* 依赖：libibverbs-dev， librdmacm1

libibverbs1
* 描述：允许用户空间进程使用《InfiniBand架构规范》和《RDMA协议动词规范》中描述的动词。iWARP以太网卡基于硬件卸载的TCP/ IP支持RDMA，而InfiniBand是一种高吞吐量，低延迟的网络技术。 InfiniBand主机通道适配器（HCA）和iWARP NIC通常支持从用户空间进行直接硬件访问（内核旁路），且libibverbs可用时支持此操作。
* 依赖：adduser，libc6，libnl-3-200，libnl-route-3-200 

libibverbs-dev
* 描述：libibverbs1的开发文件
* 依赖：libibverbs1 和ibverbs-providers

ibverbs-providers
* 描述：RMDA驱动分为用户空间部分和内核空间部分。这个包主要是用户空间部分的驱动：
  - bnxt_re: Broadcom NetXtreme-E RoCE HCAs
  - cxgb3: Chelsio T3 iWARP HCAs
  - cxgb4: Chelsio T4 iWARP HCAs
  - hfi1verbs: Intel Omni-Path HFI
  - hns: HiSilicon Hip06 SoC
  - i40iw: Intel Ethernet Connection X722 RDMA
  - ipathverbs: QLogic InfiniPath HCAs
  - mlx4: Mellanox ConnectX-3 InfiniBand HCAs
  - mlx5: Mellanox Connect-IB/X-4+ InfiniBand HCAs
  - mthca: Mellanox InfiniBand HCAs
  - nes: Intel NetEffect NE020-based iWARP adapters
  - ocrdma: Emulex OneConnect RDMA/RoCE device
  - qedr: QLogic QL4xxx RoCE HCAs
  - rxe: A software implementation of the RoCE protocol
  - vmw_pvrdma: VMware paravirtual RDMA device
* 依赖：libc6和libibverbs1

ibverbs-utils：
* 描述：基于libibverbs1开发的实用程序。比如ibv_devinfo,，可以显示IB设备的信息。
* 依赖：libc6和libibverbs1

libibmad5
* 描述：InfiniBand管理数据报库。它提供了InfiniBand 诊断和管理程序使用的底层InfiniBand管理数据报功能，包括Management Datagrams (MAD), Subnet Administration (SA), Subnet Management Packets (SMP)和其他基本功能.
* 依赖：libc6，libibumad3

libibmad-dev
* 描述：libibumad5的开发文件。
* 依赖：libibumad5

libibumad3
* 描述：InfiniBand用户空间管理数据报库。它提供用户空间管理数据报功能。这些功能位于内核中的uMAD模块之上，这些功能由InfiniBand诊断和管理工具使用。
* 依赖：libc6

libibumad-dev 
* 描述：libibumad的开发文件。
* 依赖：libibumad3 


------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>[《RDMA Aware Networks Programming User Manual v1.7》](https://www.mellanox.com/related-docs/prod_software/RDMA_Aware_Programming_user_manual.pdf)、[《Dotan Barak的RDMAmojo博客》](http://www.rdmamojo.com/)、《Ubuntu19.10 的rdma-core套件man手册v24.0》，我所做的工作就是更新、整理一部分内容，删除了一部分废弃的API，添加了一部分新的API。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

