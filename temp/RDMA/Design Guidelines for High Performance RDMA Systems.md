---
title: Design Guidelines for High Performance RDMA Systems
tags: RDMA
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文翻译自<font color=blue>《Design Guidelines for High Performance RDMA Systems》。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

<style>table{word-break:initial;}</style>

# 0 摘要  

Modern RDMA hardware offers the potential for exceptional performance, but design choices including which RDMA operations to use and how to use them significantly affect observed performance. This paper lays out guidelines that can be used by system designers to navigate the RDMA design space. Our guidelines emphasize paying attention to low-level details such as individual PCIe transactions and NIC architecture. We empirically demonstrate how these guidelines can be used to improve the performance of RDMA-based systems: we design a networked sequencer that outperforms an existing design by 50x, and improve the CPU effciency of a prior highperformance key-value store by 83%. We also present and evaluate several new RDMA optimizations and pitfalls, and discuss how they affect the design of RDMA systems.

现代RDMA硬件具有实现卓越性能的潜力，但是设计选择(包括使用哪种RDMA操作以及如何使用它们)会显着影响所观察到的性能。 本文列出了系统设计人员可以用来指导RDMA设计步骤的指南。 我们的准则强调要注意底层细节，例如单个PCIe事务和NIC体系结构。 我们以经验证明了如何使用这些准则来提高基于RDMA的系统的性能：我们设计了一个网络排序器，其性能比现有设计高50倍，并且将以前的高性能键值存储的CPU效率提高了83％。 我们还将介绍和评估几种新的RDMA优化和陷阱，并讨论它们如何影响RDMA系统的设计。

# 1 引言

In recent years, Remote Direct Memory Access (RDMA)-capable networks have dropped in price and made substantial inroads into datacenters. Despite their newfound popularity, using their advanced capabilities to best effect remains challenging for software designers. This challenge arises because of the nearly-bewildering array of options a programmer has for using the NIC （In this paper, we refer exclusively to RDMA-capable network interface cards, so we use the more generic but shorter term NIC throughout）, and because the relative performance of these operations is determined by complex low-level factors such as PCIe bus transactions and the (proprietary and often confidential) details of the NIC architecture.

近年来，支持远程直接内存访问（RDMA）的网络价格下降，并大举进入数据中心。 尽管获得了新的普及，但对软件设计师来说，使用其先进的功能来达到最佳效果仍然是一项挑战。 由于程序员使用NIC（在本文中仅指代支持RDMA的网络接口卡，因此我们在整个过程中使用的是更通用但更短的术语：NIC）的选择几乎令人迷惑，因此出现了这一挑战，并且因为 这些操作的相对性能取决于复杂的低级因素，例如PCIe总线事务以及NIC体系结构的（专有且通常是机密的）详细信息。

Unfortunately, finding an effcient match between RDMA capabilities and an application is important: As we show in Section 5, the best and worst choices of RDMA options vary by a factor of seventy in their overall throughput, and by a factor of 3.2 in the amount of host CPU they consume. Furthermore, there is no onesize- fits-all best approach. Small changes in application requirements significantly affect the relative performance of different designs. For example, using general-purpose RPCs over RDMA is the best design for a networked key-value store (Section 4), but this same design choice provides lower scalability and 16% lower throughput than the best choice for a networked “sequencer” (Section 4; the sequencer returns a monotonically increasing integer to client requests).

不幸的是，找到RDMA功能与应用程序之间的有效匹配很重要：正如我们在第5节中所展示的，RDMA选项的最佳和最差选择在它们的总吞吐量中相差70倍，在它们消耗的主机CPU量方面相差3.2倍。 此外，还没有一种万能的最佳方法。 应用程序需求的细微变化会严重影响不同设计的相对性能。 例如，在RDMA上使用通用RPC是网络键值存储的最佳设计（第4节），但是与网络“序列器”的最佳选择相比，前面的设计选择会导致更低的可扩展性和损失16％的吞吐量（ 第4节；序列器向客户端请求返回单调递增的整数）。


This paper helps system designers address this challenge in two ways. First, it provides guidelines, backed by an open-source set of measurement tools, for evaluating and optimizing the most important system factors that affect end-to-end throughput when using RDMA NICs. For each guideline (e.g., “Avoid NIC cache misses”), the paper provides insight on both how to determine whether this guideline is relevant (e.g., by using PCIe counter measurements to detect excess traffic between the NIC and the CPU), and a discussion of which modes of using the NICs are most likely to mitigate the problem.

本文以两种方式帮助系统设计人员应对这一挑战。 首先，它提供了一套指南，并以一组开源的测量工具为后盾，用于评估和优化在使用RDMA NIC时影响端到端吞吐量的最重要的系统因素。 对于每个准则（例如，“避免NIC缓存未命中”），本文都提供了有关如何确定该准则是否相关的见解（例如，通过使用PCIe计数器测量来检测NIC和CPU之间的多余流量）以及讨论使用哪种NIC模式最有可能缓解该问题。


Second, we evaluate the efficacy of these guidelines by applying them to both microbenchmarks and real systems, across three generations of RDMA hardware. Section 4.2 presents a novel design for a network sequencer that outperforms an existing design by 50x. Our best sequencer design handles 122 million operations/second using a single NIC and scales well. Section 4.3 applies the guidelines to improve the CPU efficiency and throughput of the HERD key-value cache [^20] by 83% and 35% respectively. Finally, we show that today’s RDMA NICs handle contention for atomic operations extremely slowly, rendering designs that use them [^27],[^30], [^11] very slow.

其次，我们将这些准则应用于三代RDMA硬件的微基准和实际系统中，以评估这些准则的有效性。 第4.2节介绍了一种网络定序器的新颖设计，其性能比现有设计高出50倍。 我们最佳的音序器设计使用单个NIC可以处理每秒1.22亿次操作，并且可以很好地扩展。 第4.3节应用了准则，将HERD键值高速缓存[^20]的CPU效率和吞吐量分别提高了83％和35％。 最后，我们证明了当今的RDMA NIC处理原子操作的竞争非常缓慢，从而使使用它们的设计 [^27],[^30], [^11]变得非常慢。

A lesson from our work is that low-level details are surprisingly important for RDMA system design. Our underlying goal is to provide researchers and developers with a roadmap through these details without necessarily becoming RDMA gurus. We provide simple models of RDMA operations and their associated CPU and PCIe costs, plus open-source software to measure and analyze them (<https://github.com/efficient/rdma_bench>).

从我们的工作中学到的一点是，低级细节对于RDMA系统设计非常重要。 我们的基本目标是为研究人员和开发人员提供详细的路线图，而不必成为RDMA专家。 我们提供RDMA操作及其相关的CPU和PCIe成本的简单模型，以及用于测量和分析它们的开源软件（<https://github.com/efficiency/rdma_bench>）。

We begin our journey into high-performance RDMA based systems with a review of the relevant capabilities of RDMA NICs, and the PCIe bus that frequently arises as a bottleneck.

通过回顾RDMA NIC的相关功能以及经常成为瓶颈的PCIe总线，我们开始了基于高性能RDMA的系统之旅。


# 2 背景
![图1](https://raw.githubusercontent.com/liao20081228/blog/master/图片/Design_Guidelines_for_High_Performance_RDMA_Systems/图1.jpg)

Figure 1 shows the relevant hardware components of a machine in an RDMA cluster. A NIC with one or more ports connects to the PCIe controller of a multi-core CPU. The PCIe controller reads/writes the L3 cache to service the NIC’s PCIe requests; on modern Intel servers [^4], the L3 cache provides counters for PCIe events.

图1显示了RDMA群集中机器的相关硬件组件。 具有一个或多个端口的NIC连接到多核CPU的PCIe控制器。 PCIe控制器读取/写入L3高速缓存以服务NIC的PCIe请求； 在现代的英特尔服务器上[4]，L3缓存为PCIe事件提供了计数器。


## 2.1 PCIe
The current fastest PCIe link is PCIe “3.0 x16,” the 3rd generation PCIe protocol, using 16 lanes. The bandwidth of a PCIe link is the per-lane bandwidth times the number of lanes. PCIe is a layered protocol, and the layer headers add overhead that is important to understand for efficiency. RDMA operations generate 3 types of PCIe transaction layer packets (TLPs): read requests, write requests, and read completions (there is no transaction-layer response for a write). Figure 2a lists the bandwidth and header overhead for the PCIe generations in our clusters. Note that the header overhead of 20–26 bytes is comparable to the common size of data items used in services such as memcached [^25] and RPCs [^15].

当前最快的PCIe链接是使用16条通道的第三代PCIe协议PCIe“ 3.0 x16”。 PCIe链路的带宽是每通道带宽乘以通道数。 PCIe是分层协议，并且层头部增加了开销，这对于了解效率至关重要。 RDMA操作会生成3种类型的PCIe事务层数据包（TLP）：读取请求，写入请求和读取完成（写入没有事务层响应）。 图2a列出了集群中PCIe产生的带宽和头部开销。 请注意，20-26字节的报头开销与服务中使用的数据项（例如memcached [^25]和RPC [^15]）的常用大小相当。

![图2](https://raw.githubusercontent.com/liao20081228/blog/master/图片/Design_Guidelines_for_High_Performance_RDMA_Systems/图2.jpg)

**MMIO writes vs. DMA reads** There are important differences between the two methods of transferring data from a CPU to a PCIe device. CPUs write to mapped device memory (MMIO) to initiate PCIe writes. To avoid generating a PCIe write for each store instruction, CPUs use an optimization called “write combining,” which combines stores to generate cache line–sized PCIe transactions. PCIe devices have DMA engines and can read from DRAM using DMA. DMA reads are not restricted to cache lines, but a read response larger than the CPU’s read completion combining size (C~rc~) is split into multiple completions. Crc is 128 bytes for the Intel CPUs used in our measurements (Table 2); we assume 128 bytes for the AMD CPU [^4], [^3]. A DMA read always uses less host-to-device PCIe bandwidth than an equal-sized MMIO; Figure 2b shows an analytical comparison. This is an important factor, and we show how it effiects performance of higher-layer protocols in the subsequent sections.

**MMIO写入与DMA读取之间的比较** 将数据从CPU传输到PCIe设备的两种方法之间存在重要区别。 CPU写入映射的设备内存（MMIO）以启动PCIe写入。 为了避免为每个存储指令生成PCIe写操作，CPU使用了一种称为“写合并”的优化，该优化将存储合并以生成高速缓存行大小的PCIe事务。 PCIe设备具有DMA引擎，可以使用DMA从DRAM读取。 DMA读取不限于高速缓存行，而是将大于CPU读取完成合并大小（C~rc~）的读取响应分为多个完成。 测量中使用的Intel CPU的Crc为128字节（表2）； 我们假设AMD CPU为128字节[^4]，[^3]。 与同等大小的MMIO相比，DMA读取始终使用更少的主机到设备PCIe带宽。 图2b显示了分析比较。 这是一个重要因素，我们将在随后的部分中说明它如何影响高层协议的性能。



**PCIe counters**  Our contributions rely on understanding the PCIe interaction between NICs and CPUs. Although precise PCIe analysis requires expensive PCIe analyzers or proprietary/confidential NICs manuals, PCIe counters available on modern CPUs can provide several useful insights.(The CPU intercepts cache line-level activity between the PCIe controller and the L3 cache, so the counters can miss some critical information. For example, the counters indicate 2 PCIe reads when the NIC reads a 4-byte chunk straddling 2 cache lines.) For each counter, the number of captured events per second is its counter rate. Our analysis primarily uses counters for DMA reads (PCIeRdCur) and DMA writes (PCIeItoM).

**PCIe计数器** 我们的贡献取决于对NIC和CPU之间的PCIe交互的了解。 尽管精确的PCIe分析需要昂贵的PCIe分析器或专有/机密NIC手册，但是现代CPU上可用的PCIe计数器可以提供一些有用的见解。(CPU拦截了PCIe控制器和L3缓存之间的缓存行级活动，因此计数器可能会丢失一些关键信息。 例如，当NIC读取跨越2个高速缓存行的4字节块时，计数器指示2个PCIe读取。)对于每个计数器，每秒捕获的事件数就是其计数器速率。我们的分析主要将计数器用于DMA读取（PCIeRdCur）和DMA写入（PCIeItoM）。

## 2.2 RDMA

RDMA is a network feature that allows direct access to the memory of a remote computer. RDMA-providing networks include InfiniBand, RoCE (RDMA over Converged Ethernet), and iWARP (InternetWide Area RDMA Protocol). RDMA networks usually provide high bandwidth and low latency: NICs with 100 Gbps of per-port bandwidth and ~2µs round-trip latency are commercially available. The performance and scalability of an RDMAbased communication protocol depends on several factors including the operation (verb) type, transport, optimization flags, and operation initiation method.

RDMA是一项网络功能，可以直接访问远程计算机的内存。 提供RDMA的网络包括InfiniBand，RoCE（融合以太网上的RDMA）和iWARP（Internet广域RDMA协议）。 RDMA网络通常提供高带宽和低延迟：具有100 Gbps每端口带宽和大约2µs往返延迟的NIC可以从市场上买到。 基于RDMA的通信协议的性能和可扩展性取决于几个因素，包括操作（动词）类型，传输，优化标志和操作启动方法。

### 2.2.1 RDMA动词和传输

RDMA hosts communicate using queue pairs (QPs); hosts create QPs consisting of a send queue and a receive queue, and post operations to these queues using the verbs API. We call the host initiating a verb the *requester* and the destination host the *responder*. For some verbs, the responder does not actually send a response. On completing a verb, the requester’s NIC optionally signals completion by DMA-ing a completion entry (CQE) to a completion queue (CQ) associated with the QP. Verbs can be made unsignaled by setting a flag in the request; these verbs do not generate a CQE, and the application detects completion using application-specific methods.


RDMA主机使用队列对（QPs）进行通信； 主机创建由发送队列和接收队列组成的QPs，并使用动词API将操作发布到这些队列。 我们称发起动词的主机为*请求者*，目标主机为*响应者*。 对于某些动词，响应者实际上并未发送响应。 完成动词后，请求者的NIC可选地通过将完成项（CQE）DMA到与QP关联的完成队列（CQ）来发信号以通知动词完成。 通过在请求中设置标记，可以使动词不带信号； 这些动词不会生成CQE，并且应用程序使用特定于应用程序的方法来检测完成。

The two types of verbs are memory verbs and messaging verbs. Memory verbs include RDMA reads, writes,and atomic operations. These verbs specify the remote address to operate on and bypass the responder’s CPU. Messaging verbs include the send and receive verbs. These verbs involve the responder’s CPU: the send’s payload is written to an address specified by a receive that was posted previously by the responder’s CPU. In this paper, we refer to RDMA read, write, send, and receive verbs as READ, WRITE, SEND, and RECV respectively.

动词的两种类型是内存动词和消息传递动词。 内存动词包括RDMA读取，写入和原子操作。 这些动词指定了用于操作并绕过响应者CPU的远程地址。 消息动词包括发送和接收动词。 这些动词涉及响应者的CPU：发送的有效负载被写入指定的地址，该地址由响应者的CPU在之前发布。 在本文中，我们将RDMA的读，写，发送和接收动词分别称为READ，WRITE，SEND和RECV。

![表1](https://raw.githubusercontent.com/liao20081228/blog/master/图片/Design_Guidelines_for_High_Performance_RDMA_Systems/表1.jpg)

RDMA transports are either reliable or unreliable, and either connected or unconnected (also called datagram). With reliable transports, the NIC uses acknowledgments to guarantee in-order delivery of messages. Unreliable transports do not provide this guarantee. However, modern RDMA implementations such as InfiniBand use a lossless link layer that prevents congestion-based losses using link layer flow control [^1], and bit error–based losses using link layer retransmissions [^8]. Therefore, unreliable transports drop packets very rarely. Connected transports require one-to-one connections between QPs, whereas a datagram QP can communicate with multiple QPs. We consider two types of connected transports in this paper: Reliable Connected (RC) and Unreliable Connected (UC). Current RDMA hardware provides only 1 datagram transport: Unreliable Datagram (UD). Different transports support different subsets of verbs: UC does not support RDMA reads, and UD does not support memory verbs. Table 1 summarizes this.

RDMA传输可以可靠的，也可以是还不可靠的；并且是有连接或者无连接的（也称为数据报）。通过可靠的传输，NIC使用确认来保证消息按顺序传递。不可靠的运输不提供此保证。但是，诸如InfiniBand之类的现代RDMA实现使用无损链路层，该层使用链路层流控制[^1]防止基于拥塞的损耗，而使用链路层重传[^8]防止基于误码的损耗。因此，不可靠的传输很少丢弃数据包。连接的传输需要QP之间进行一对一连接，而数据报QP可以与多个QP通信。在本文中，我们考虑了两种类型的连接传输：可靠连接（RC）和不可靠连接（UC）。当前的RDMA硬件仅提供一种数据报传输：不可靠的数据报（UD）。不同的传输支持动词的不同子集：UC不支持RDMA读取，而UD不支持记忆动词。表1对此进行了总结。


### 2.2.2 RDMA WQEs

To initiate RDMA operations, the user-mode NIC driver at the requester creates Work Queue Elements (WQEs) in host memory; typically, WQEs are created in a pre-allocated, contiguous memory region, and each WQE is individually cache line–aligned. (We discuss methods of transferring WQEs to the device in Section 3.1.) The WQE format is vendor-specific and is determined by the NIC hardware.

为了启动RDMA操作，请求者处的用户模式NIC驱动程序会在主机内存中创建工作队列元素（WQE）。 通常，WQE是在预分配的连续内存区域中创建的，并且每个WQE都是分别按行对齐的高速缓存。 （我们将在第3.1节中讨论将WQE传输到设备的方法。）WQE格式是特定于供应商的，并且由NIC硬件确定。

**WQE size** depends on several factors: the type of RDMA operation, the transport, and whether the payload is referenced by a pointer field or inlined in the WQE (i.e., the WQE buffer includes the payload). Table 1 shows the WQE header size for Mellanox NICs for three transports. For example, with a 36-byte WQE header, the size of a WRITE WQE with an x-byte inlined payload is 36 + x bytes. UD WQEs have larger, 68-byte headers to store additional routing information.

WQE大小取决于几个因素：RDMA操作的类型，传输以及有效载荷是由指针字段引用还是内联在WQE中（即WQE缓冲区包括有效载荷）。 表1显示了用于三种传输的Mellanox NIC的WQE标头大小。 例如，对于36字节的WQE标头，带有x字节内联有效负载的WRITE WQE的大小为36 + x字节。 UD WQE具有较大的68字节标头，用于存储其他路由信息。



### 2.2.3 技术和假设

![图3](https://raw.githubusercontent.com/liao20081228/blog/master/图片/Design_Guidelines_for_High_Performance_RDMA_Systems/图3.jpg)
We distinguish between inbound and outbound verbs because their performance differs significantly (Section 5): memory verbs and SENDs are outbound at the requester and inbound at the responder; RECVs are always inbound. Figure 3 summarizes this. As our study focuses on small messages, all WRITEs and SENDs are inlined by default. We define the padding-to-cache-line-alignment function $x'= ⌈x/64⌉ * 64$. We denote the per-lane bandwidth, request header size, and completion header size of PCIe 3.0 by Pbw, Pr , and Pc , respectively.

我们区分入站和出站动词，因为它们的性能差异很大（第5节）：记忆动词和SEND在请求者处出站，在响应者处入站； RECV始终是入站的。 图3对此进行了总结。 由于我们的研究重点是小消息，因此默认情况下会内联所有WRITE和SEND。 我们定义padding-to-cache-line-alignment函数$x'= ⌈x/64⌉ * 64$。我们分别通过Pbw，Pr和Pc表示PCIe 3.0的每通道带宽，请求标头大小和完成标头大小。 

# 3 RDMA设计指南

We now present our design guidelines. Along with each guideline, we present new optimizations and briefly describe those presented in prior literature. We make two assumptions about the NIC hardware that are true for all currently available NICs. First, we assume that the NIC is PCIe-based device. Current network interfaces (NIs) are predominantly discrete PCIe cards; vendors are beginning to integrate NIs on-die [^2], [^5], or on-package [^6], but these NIs still communicate with the PCIe controller using the PCIe protocol, and are less powerful than discrete NIs. Second, we assume that the NIC has internal parallelism using multiple processing units (PUs)—this is generally true of high-speed NIs [^19]. As in conventional parallel programming, this parallelism provides both optimization opportunities (Section 3.3) and pitfalls (Section 3.4).

To discuss the impact on CPU and PCIe use of the optimizations below, we consider transferring N WQEs of size D bytes from the CPU to the NIC.


现在，我们介绍我们的设计准则。 连同每个指南，我们提出了新的优化方法，并简要介绍了先前文献中提出的优化方法。 我们对NIC硬件做出两个假设，这对于所有当前可用的NIC都是正确的。 首先，我们假设NIC是基于PCIe的设备。 当前的网络接口（NI）主要是独立的PCIe卡。 供应商开始在裸片[^2]，[^5]或在封装[^6]上集成NI，但是这些NI仍然使用PCIe协议与PCIe控制器通信，并功能不如独立的NI。 其次，我们假设NIC具有内部并行性，用于用多个处理单元（PU）——对于高速NI通常是正确的[^19]。 与传统的并行编程一样，这种并行性提供了优化机会（第3.3节）和陷阱（第3.4节）。 


To discuss the impact on CPU and PCIe use of the optimizations below, we consider transferring N WQEs of size D bytes from the CPU to the NIC.

为了讨论以下优化对CPU和PCIe使用的影响，我们考虑将大小D字节的N个WQE从CPU传输到NIC。

## 3.1 减少CPU-initiated MMIOs

![图4](https://raw.githubusercontent.com/liao20081228/blog/master/图片/Design_Guidelines_for_High_Performance_RDMA_Systems/图4.jpg)

Both CPU effciency and RDMA throughput can improve if MMIOs are reduced or replaced with the more CPU-and bandwidth-efficient DMAs. CPUs initiate network operations by sending a message to the NIC via MMIO. The message can (1) contain the new work queue elements, or (2) it can refer to the new WQEs by using information such as the address of the last WQE. In the first case, the WQEs are transferred via 64-byte write-combined MMIOs. In the second case, the NIC reads the WQEs using one or more DMAs.3（In this paper, we assume that the NIC reads all new WQEs in one DMA, as is done by Mellanox’s Connect-IB and newer NICs. Older Mellanox NICs read one or more WQEs per DMA, depending on the NIC’s proprietary prefetching logic.）We refer to these methods as WQE-by-MMIO and Doorbell respectively. (Different technologies have different terms for these methods. Mellanox uses “BlueFlame” and “Doorbell,” and Intel®Omni-Path Architecture uses “PIO send” and “SDMA,” respectively.) Figure 4 summarizes this. The WQE-by-MMIO method optimizes for low latency and is typically the default. Two optimizations can improve performance by reducing MMIOs:

如果减少MMIO或将其替换为CPU和带宽效率更高的DMA，则可以提高CPU效率和RDMA吞吐量。 CPU通过MMIO向NIC发送消息来启动网络操作。 该消息可以（1）包含新的工作队列元素，或者（2）它可以通过使用诸如最后一个WQE的地址之类的信息来引用新的WQE。 在第一种情况下，WQE是通过64字节写组合MMIO传输的。 在第二种情况下，NIC使用一个或多个DMA读取WQE。（在本文中，我们假定NIC像Mellanox的Connect-IB和较新的NIC一样，在一个DMA中读取所有新的WQE。较早的Mellanox NIC每个DMA读取一个或多个WQE，具体取决于NIC的专有预取逻辑。）我们将这些方法分别称为WQE-by-MMIO和Doorbell。 （不同的技术对这些方法使用不同的术语。Mellanox使用“ BlueFlame”和“ Doorbell”，英特尔®Omni-path体系结构分别使用“ PIO send”和“ SDMA”。）图4对此进行了总结。 WQE-by-MMIO针对低延迟进行了优化，通常是默认设置。 两种优化可以通过减少MMIO来提高性能：


**Doorbell batching** If an application can issue multiple WQEs to a QP, it can use one Doorbell MMIO for the batch. CPU: Doorbell reduces CPU-generated MMIOs from $N * D'/  64$ with WQE-by-MMIO to 1. PCIe: For N = 10 and D = 65 and PCIe 3.0, Doorbell transfers 1534 bytes, whereas WQE-by-MMIO transfers 1800 bytes (Appendix A). 

**Doorbell 批处理**  如果应用程序可以向QP发出多个WQE，则可以使用一个Doorbell MMIO进行批处理。CPU：Doorbell将WQE-by-MMIO的CPU生成的MMIO从$N * D'/  64$ 减少到1。PCIe：对于N = 10和D = 65和PCIe 3.0，Doorbell传输1534个字节，而WQE-MMIO传输1800个字节（附录A）。

In this paper, we refer to Doorbell batching as batching—a batched WQE transfer via WQE-by-MMIO is identical to a sequence of individual WQE-by-MMIOs, so batching is only useful for Doorbell. 

在本文中，我们将Doorbell批处理称为批处理——通过WQE-by-MMIO进行的批量WQE传输与顺序的单个WQE-by-MMIOs相同，因此批处理仅对Doorbell有用。

**WQE shrinking** Reducing the number of cache lines used by a WQE can improve throughput drastically. For example, consider reducing WQE size by only 1 byte from 129 B to 128 B, and assume that WQE-by-MMIO is used. CPU: CPU-generated MMIOs decreases from 3 to 2. PCIe: Number of PCIe transactions decreases from 3 to 2. Shrinking mechanisms include compacting the application payload, or overloading unused WQE header fields with application data.

**WQE缩减**  减少WQE使用的高速缓存行数可以大大提高吞吐量。。 例如，考虑将WQE大小从129 B减少到128 B仅减少1个字节，并假定使用了WQE-by-MMIO。 CPU：CPU生成的MMIO从3减少到2。PCIe：PCIe事务数从3减少到2。缩减机制包括压缩应用程序有效负载，或使用应用程序数据重载未使用的WQE标头字段。



## 3.2 减少NIC-initiated DMAs

Reducing DMAs saves NIC processing power and PCIe bandwidth, improving RDMA throughput. Note that the batching optimization above adds a DMA read, but it avoids multiple MMIOs, which is usually a good tradeoff. Known optimizations to reduce NIC-initiated DMAs include unsignaled verbs which avoid the completion DMA write (Section 2.2.1), and payload inlining which avoids the payload DMA read (Section 2.2.2). The two optimizations in Section 3.1 affect DMAs, too: batching with large N requires fewer DMA reads than smaller N; WQE shrinking further makes these reads smaller.

减少DMA可以节省NIC的处理能力和PCIe带宽，从而提高RDMA吞吐量。 请注意，上面的批处理优化添加了DMA读取，但是避免了多个MMIO，通常这是一个不错的折衷方案。 减少NIC启动DMA的已知优化包括避免完成DMA写操作的无信号动词（第2.2.1节）和避免负载DMA读取的负载内联（第2.2.2节）。 第3.1节中的两个优化也影响DMA：N较大的批处理比N较小的批处理需要更少的DMA读取； WQE的缩小进一步减小了这些读取。

NICs must DMA a completion queue entry for completed RECVs [^1]; this provides an additional optimization opportunity, as discussed below. Unlike CQEs of other verbs that only signal completion and are dispensible, RECV CQEs contain important metadata such as the size of received data. NICs typically generate two separate DMAs for payload and completion, writing them to application- and driver-owned memory respectively. We later show that the corresponding performance penalty explains the rule-of-thumb that messaging verbs are slower than memory verbs, and using the DMA-avoiding optimizations below challenges this rule-of-thumb for some payload sizes. We assume here that the corresponding SEND for a RECV carries an X-byte payload.

NIC为完成RECVs必须DMA一个CQE[^1]； 如下所述，这提供了额外的优化机会。 与其他动词的CQE仅表示完成并且是可有可无的不同，RECV CQE包含重要的元数据，例如接收到的数据的大小。 NIC通常为有效负载和完成生成两个单独的DMA，分别将它们写入应用程序和驱动程序拥有的内存。 稍后我们将展示相应的性能损失来解释消息动词比内存动词慢的经验法则，并且在某些有效负载大小下，使用下面的DMA避免优化挑战了这一经验法则。 在此我们假设RECV的相应SEND携带X字节有效负载。

**Inline RECV** If X is small (~64 for Mellanox’s NICs), the NIC encapsulates the payload in the CQE, which is later copied by the driver to the application-specified address. CPU: Minor overhead for copying the small payload. PCIe: Uses 1 DMA instead of 2. 

**内联RECV** 如果X小（对于Mellanox的NIC，约为64），则NIC将有效负载封装在CQE中，然后由驱动程序将其复制到应用程序指定的地址。 CPU：复制小的有效负载所需的开销很小。 PCIe：使用1个DMA代替2个。

**Header-only RECV** If X = 0 (i.e., the RDMA SEND packet consists of only a header and no payload), the payload DMA is not generated at the receiver. Some information from the packet’s header is included in the DMA-ed CQE, which can be used to implement application protocols. We call SENDs and RECVs with X = 0 header-only, and regular otherwise. PCIe: Uses 1 DMA instead of 2.

**只有标头的RECV** 如果X = 0（即RDMA SEND数据包仅由一个报头组成，没有任何有效载荷），则有效载荷DMA不会在接收机处生成。 数据包标头中的某些信息包含在DMA版本的CQE中，可用于实现应用协议。 我们将X=0的SEND和RECV称为只有标头，否则称为常规。 PCIe：使用1个DMA代替2个。

![图5](https://raw.githubusercontent.com/liao20081228/blog/master/图片/Design_Guidelines_for_High_Performance_RDMA_Systems/图5.jpg)

![图6](https://raw.githubusercontent.com/liao20081228/blog/master/图片/Design_Guidelines_for_High_Performance_RDMA_Systems/图6.jpg)

Figure 5 and Figure 6 summarize these two guidelines for UD SENDs and RECVs, respectively. These two verbs are used extensively in our evaluation.

图5和图6分别总结了这两个关于UD SEND和RECV的准则。 这两个动词在我们的评估中被广泛使用。


## 3.3 占用多个NIC PUs

Exploiting NIC parallelism is necessary for high performance, but requires explicit attention. A common RDMA programming decision is to use as few queue pairs as possible, but doing so limits NIC parallelism to the number of QPs. This is because operations on the same QP have ordering dependencies and are ideally handled by the same NIC processing unit to avoid cross-PU synchronization. For example, in datagram-based communication, one QP per CPU core is sufficient for communication with all remote cores. Using one QP consumes the least NIC SRAM to hold QP state, while avoiding QP sharing among CPU cores. However, it “binds” a CPU core to a PU and may limit core throughput to PU throughput. This is likely to happen when per-message application processing is small (e.g., the sequencer in Section 4.2) and a high-speed CPU core overwhelms a less powerful PU. In such cases, using multiple QPs per core increases CPU efficiency; we call this the multi-queue optimization.

利用NIC并行性对于高性能是必要的，但需要特别注意。 一个常见的RDMA编程决策是使用尽可能少的队列对，但是这样做会将NIC并行性限制为QP的数量。 这是因为对同一QP的操作具有顺序依赖性，并且最好由同一NIC处理单元处理，以避免跨PU同步。 例如，在基于数据报的通信中，每个CPU核心一个QP足以与所有远程核心进行通信。 使用一个QP消耗最少的NIC SRAM来保持QP状态，同时避免了CPU核心之间的QP共享。 但是，它将CP核心“绑定”到PU并可能将核心吞吐量限制为PU吞吐量。 当每个消息的应用程序处理量较小时（例如，第4.2节中的序列器），而高速CPU核心淹没了功能较弱的PU，则可能会发生这种情况。 在这种情况下，每个核心使用多个QP可以提高CPU效率。 我们称其为多队列优化。


## 3.4 避免NIC PUs之间的争用

RDMA operations that require cross-QP synchronization introduce contention among PUs, and can perform over an order of magnitude worse than uncontended operations. For example, RDMA provides atomic operations such as compare-and-swap and fetch-and-add on remote memory. To our knowledge, all NICs available at the time of writing (including the recently released ConnectX-4 [^7]) use internal concurrency control for atomics: PUs acquire an internal lock for the target address and issue read-modify-write over PCIe. Note that atomic operations contend with non-atomic verbs too. Future NICs may use PCIe’s atomic transactions for higher performing, cache coherence-based concurrency control.

需要跨QP同步的RDMA操作会在PU之间引入竞争，并且执行性能会比无竞争操作差一个数量级。例如，RDMA提供原子操作，例如在远程内存上进行比较-交换以及获取-添加。据我们所知，在撰写本文时，所有可用的NIC（包括最近发布的ConnectX-4 [^7]）都对原子使用内部并发控制：PU获取目标地址的内部锁，并通过PCIe发出读-修改-写操作。注意，原子操作也与非原子动词竞争。未来的NIC可能会使用PCIe的原子事务来实现更高性能的，基于缓存一致性的并发控制。


Therefore, the NIC’s internal locking mechanism, such as the number of locks and the mapping of atomic addresses to these locks, is important; we describe experiments to infer this in Section 5.4. Note that due to the limited SRAM in NICs, the number of available locks is small, which amplifies contention in the workload.


因此，NIC的内部锁定机制很重要，例如锁定的数量以及原子地址到这些锁定的映射。我们将在5.4节中介绍实验来推断这一点。请注意，由于NIC中的SRAM受限制，可用锁的数量很少，这会放大工作负载中的争用。


## 3.5 避免NIC缓存缺失

NICs cache several types of information; it is critical to maintain a high cache hit rate because a miss translates to a read over PCIe. Cached information includes (1) virtual to physical address translations for RDMA-registered memory, (2) QP state, and (3) a work queue element cache. While the first two are known [^13], the third is undocumented and was discovered in our experiments. Address translation cache misses can be reduced by using large (e.g., 2 MB) pages, and QP state cache misses by using fewer QPs [^13]. We make two new contributions in this context:

NIC缓存几种类型的信息。保持高速缓存命中率非常重要，因为未命中会转化为对PCIe的读取。缓存的信息包括（1）RDMA注册的内存的虚拟到物理地址转换，（2）QP状态和（3）工作队列元素缓存。虽然前两个是已知的[^13]，但第三个没有已知记录，是在我们的实验中发现的。通过使用较大的页面（例如2 MB），可以减少地址转换缓存的丢失，而通过使用更少的QP可以减少QP状态缓存的丢失[^13]。在这种情况下，我们做出了两项新贡献：

**Detecting cache misses** All types of NIC cache misses are transparent to the application and can be difficult to detect. We demonstrate how PCIe counters can be leveraged to accomplish this, by detecting and measuring WQE cache misses (Section 5.3.2). In general, subtracting the application’s expected PCIe reads from the actual reads reported by PCIe counters gives an estimate of cache misses. Estimating expected PCIe reads in turn requires PCIe models of RDMA operations (Section 5.1).

**检测高速缓存未命中** 所有类型的NIC高速缓存未中对应用程序都是透明的，并且可能难以检测。 我们通过检测和测量WQE缓存未命中（第5.3.2节）演示了如何利用PCIe计数器来完成此任务。 通常，从PCIe计数器报告的实际读取中减去应用程序的预期PCIe读取，即可估算出缓存未命中。 反过来，估计预期的PCIe读取需要RDMA操作的PCIe模型（第5.1节）。

**WQE cache misses** The initial work queue element transfer from CPU to NIC triggers an insertion of the WQE into the NIC’s WQE cache. When the NIC eventually processes this WQE, a cache miss can occur if it was evicted by newer WQEs. In Section 5.3.2, we show how to measure and reduce these misses.

**WQE缓存未命中** 从CPU到NIC的初始WQE传输会触发WQE插入NIC的WQE缓存。 当NIC最终处理此WQE时，如果它被较新的WQE驱逐了，则会发生高速缓存未命中的情况。 在5.3.2节中，我们展示了如何测量和减少这些遗漏。

# 4 改进系统设计

We now demonstrate how these guidelines can be used to improve the design of whole systems. We consider two systems: networked sequencers, and key-value stores.

现在，我们演示如何将这些准则用于改进整个系统的设计。 我们考虑两个系统：网络序列器和键值存储。

![表2](https://raw.githubusercontent.com/liao20081228/blog/master/图片/Design_Guidelines_for_High_Performance_RDMA_Systems/表2.jpg)

**Evaluation setup** We perform our evaluation on the three clusters described in Table 2. We name the clusters with the initials of their NICs, which is the main hardware component governing performance. CX3 and CIB run Ubuntu 14.04 with Mellanox OFED 2.4; CX runs Ubuntu 12.04 with Mellanox OFED 2.3. Throughout the paper, we use WQE-by-MMIO for non-batched operations and Doorbell for batched operations. However, when batching is enabled but the available batch size is one, WQE-by-MMIO is used. (Doorbell provides little CPU savings for transferring a single small WQE, and uses an extra PCIe transaction.) For brevity, we primarily use the state-ofthe- art CIB cluster in this section; Section 5 evaluates our optimizations on all clusters.

**评估设置** 我们对表2中描述的三个群集进行评估。我们用NIC的缩写来命名群集，这是控制性能的主要硬件组件。 CX3和CIB使用Mellanox OFED 2.4，运行Ubuntu 14.04; CX使用Mellanox OFED 2.3，运行Ubuntu 12.04。 在整篇文章中，我们使用WQE-by-MMIO进行非批处理操作，并使用Doorbell进行批处理操作。 但是，批处理快可用时且可用批处理大小为1时，将使用WQE-by-MMIO。 （Doorbell在传输单个小型WQE时节省的CPU很少，并且使用了额外的PCIe事务。）为简便起见，在本节中，我们主要使用最先进的CIB集群。 第5节评估我们在所有集群上的优化。


## 4.1 HERD RPCs概述

We use HERD’s RPC protocol for communication between clients and the sequencer/key-value server. HERD RPCs have low overhead at the server and high number-of-clients scalability. Protocol clients use unreliable WRITEs to write requests to a request memory region at the server. Before doing so, they post a RECV to an unreliable datagram QP for the server’s response. A server thread (a worker) detects a new request by polling on the request memory region. Then, it performs application processing and responds using a UD SEND posted via WQE-by-MMIO. 

我们使用HERD的RPC协议在客户端与序列器/键值服务器之间进行通信。 HERD RPC在服务器上具有较低的开销，并且在客户端数量方面具有较高的可扩展性。 协议客户端使用不可靠的WRITE将请求写入服务器上的请求内存区域。 在这样做之前，他们将RECV发布到不可靠的数据报QP中，以获取服务器的响应。 服务器线程（工作线程）通过轮询请求内存区域来检测新请求。 然后，它执行应用程序处理，并使用通过WQE-by-MMIO发布的UD SEND进行响应。

We apply the following two optimizations to HERD RPCs in general; we present specific optimizations for each system later.
* **Batching** Instead of responding after detecting one request, the worker checks for one request from each of the C clients, collecting $N <= C$ requests. Then, it SENDs N responses using a batched Doorbell.
* **Multi-queue** Each worker alternates among a tuneable number of UD queue pairs across the batched SENDs.

Note that batching does not add significant latency because we do it opportunistically [^23], [^21]; we do not wait for a number of requests to accumulate. We briefly discuss the latency added by batching in Section 4.2.2.

通常，我们将以下两个优化应用于HERD RPC： 我们稍后会为每个系统提供特定的优化。

* **批处理** 工作线程不会在检测到一个请求之后做出响应，而是从每个C客户端检查一个请求，并收集$N <= C$个请求。 然后，它使用批处理的Doorbell发送N个响应。
* **多队列** 批处理的SEND中，每个工作程序在可调整数量的UD队列对之间交替。

请注意，批处理并不会增加明显的延迟，因为我们是适时地这样做[^23]，[^21]; 我们不等待大量请求积累。 我们在第4.2.2节中简要讨论通过批处理添加的延迟。




## 4.2 网络序列器

Centralized sequencers are useful building blocks for a variety of network applications, such as ordering operations in distributed systems via logical or real timestamps [^11], or providing increasing offsets into a linearly growing memory region [^10]. A centralized sequencer can be the bottleneck in high-performance distributed systems, so building a fast sequencer is an important step to improving whole-system performance.

集中序列器是各种网络应用程序的有用构建块，例如通过逻辑或实时时间戳对分布式系统中的命令进行操作[^11]，或在线性增长的内存区域中提供增加的偏移量[^10]。 集中式序列器可能是高性能分布式系统中的瓶颈，因此构建快速音序器是提高整个系统性能的重要一步。

Our sequence server runs on a single machine and provides an increasing 8-byte integer to client processes running on remote machines. The baseline design usesHERD RPCs. The worker threads at the server share an 8-byte counter; each client can send a sequencer request to any worker. The worker’s application processing consists of atomically incrementing the shared counter by one. When Doorbell batching is enabled, we use an additional application-level optimization to reduce contention for the shared counter: after collecting N requests, a worker atomically increments the shared counter by N, thereby claiming ownership of a sequence of N consecutive integers. It then sends these N integers to the clients using a batched Doorbell (one integer per client).

我们的序列服务器在单台计算机上运行，并为在远程计算机上运行的客户端进程提供递增的8字节整数。 基线设计使用HERD RPC。 服务器上的工作线程共享一个8字节计数器； 每个客户端可以向任何工作线程发序列器请求。 工作线程的应用程序处理包括原子地将共享计数器加1。 启用Doorbell批处理后，我们将使用附加的应用程序级优化来减少共享计数器的争用：收集N个请求后，工作线程原子地将共享计数器增加N，从而声明N个连续整数序列的所有权。 然后，它使用批处理的门铃（每个客户端一个整数）将这N个整数发送给客户端。

![图7](https://raw.githubusercontent.com/liao20081228/blog/master/图片/Design_Guidelines_for_High_Performance_RDMA_Systems/图7.jpg)

Figure 7 shows the effect of batching and multi-queue on the HERD RPC-based sequencer’s throughput with an increasing number of server CPU cores. We run 1 worker thread per core and use 70 client processes on 5 client machines. Batching increases single-core throughput from 7.0 million requests per second (Mrps) to 16.6 Mrps. In this mode, each core still uses 2 response UD queue pairs—one for each NIC port—and is bottlenecked by the NIC processing units handling the QPs; engaging more PUs with multi-queue (3 per-port QPs per core) increases core throughput to 27.4 Mrps. With 6 cores and both optimizations, throughput increases to 97.2 Mrps and is bottlenecked by DMA bandwidth: The DMA bandwidth limit for the batched UD SENDs used by our sequencer is 101.6 million operations/s (Section 5.2.1). At 97.2 Mrps, the sequencer is within 5% of this limit; we attribute the gap to PCIe link- and physical-layer overheads in the DMA-ed requests, which are absent in our SEND-only benchmark. When more than 6 cores are used, throughput drops because the response batch size are smaller: With 6 cores (97.2 Mrps), there are 15.9 responses per batch; with 10 cores (84 Mrps), there are 4.4 responses per batch.

图7显示了批处理和多队列对基于HERD RPC的序列器的吞吐量的影响（随着服务器CPU内核数量的增加）。我们每个核心运行1个工作线程，并在5台客户端计算机上使用70个客户端进程。批处理将单核吞吐量从每秒700万个请求（Mrps）增加到16.6 Mrps。在这种模式下，每个核心仍使用2个响应UD队列对（每个NIC端口一个），并且受到处理QP的NIC处理单元的瓶颈；将更多的PU与多队列配合使用（每个核心3个端口-QPs）可将核心吞吐量提高到27.4 Mrps。具有6个内核并同时进行了优化，吞吐量增加到97.2 Mrps，并受到DMA带宽的限制：我们的序列器使用的批处理UD SEND的DMA带宽限制为1.016亿次操作/秒（第5.2.1节）。在97.2Mrps时，定序器在此限制的5％以内；我们将此差距归因于DMA请求中的PCIe链路层和物理层开销，而这仅在SEND基准测试中是不存在的。当使用6个以上的内核时，由于响应批处理器的大小较小，吞吐量会下降：对于6个内核（97.2 Mrps），每批处理响应15.9。具有10个内核（84 Mrps），每个批处理响应4.4。

## 4.2.1 序列器特定优化

The above design is a straightforward adoption of general purpose RPCs for a sequencer, and inherits the limitations of the RPC protocol. First, the connected QPs used for writing requests require state in the server’s NIC and limit scalability to a few hundred RPC clients [^20]. Higher scalability necessitates exclusive use of datagram transport which only supports SEND/RECV verbs. The challenge then is to use SEND/RECV instead of WRITEs for sequencer requests without sacrificing server performance. Second, it uses PCIe inefficiently: UD SEND work queue elements on Mellanox’s NICs span >= 2 cache lines because of their 68-byte header (Table 1); sending 8 bytes of useful sequencer data requires 128 bytes (2 cache lines) to be DMA-ed by the NIC.

以上是通用RPC直接用于定序器的设计，并且继承了RPC协议的局限性。 首先，用于写入请求的有连接QPs需要服务器NIC中的状态，并将可扩展性限制为数百个RPC客户端[^20]。 更高的可扩展性要求专用仅支持SEND / RECV动词的数据报传输。 之后的挑战是在不牺牲服务器性能的情况下使用SEND / RECV而不是WRITE进行序列器请求。 其次，它低效地使用PCIe：Mellanox NIC上的UD SEND WQE 由于其68字节的标头而跨越>= 2个缓存行（表1）； 发送8字节有用的序列器数据需要NIC对DMA进行128字节（2个高速缓存行）的处理。


We exploit the specific requirements of the sequencer to overcome these limitations. We use header-only SENDs for both requests and responses to solve both problems:

1. The client’s header-only request SENDs generate header-only, single-DMA RECVs at the server (Figure 6), which are as fast as the WRITEs used earlier.
2. The server’s header-only response SEND WQEs use a header field for application payload and fit in 1 cache line (Figure 5), reducing the data DMA-ed per response by 50% to 64 bytes.


我们利用定序器的特定要求来克服这些限制。 我们将仅标头的SEND用于请求和响应，以解决两个问题：

1.客户端的仅标头请求SENDs在服务器上生成仅标头的单DMA RECVs（图6），其速度与之前使用的WRITE一样快。
2.服务器的仅标头响应SEND WQE使用标头字段存储应用程序有效负载并放入1个缓存行（图5），从而将每个响应的DMA数据减少50％至64字节。

Using header-only SENDs requires encoding application information in the SEND packet header; we use the 4-byte immediate integer field of RDMA packets [^1]. Our 8-byte sequencer works around the 4-byte limit as follows: Clients speculate the 4 higher bytes of the counter and send it in a header-only SEND. If the client’s guess is correct,
the server sends the 4 lower bytes in a header-only SEND, else it sends the entire 8-byte value in a regular, non header-only SEND which later triggers an update of the client’s guess. Only a tiny fraction (<=C/2^32^ with C clients) of SENDs are regular. We discuss this speculation technique further in Section 5.

使用仅标头的SEND需要在SEND数据包标头中编码应用程序信息。 我们使用RDMA数据包的4字节立即整数字段[1]。 我们的8字节定序器在4字节的限制范围内工作，如下所示：客户端推测计数器的前4个字节并将其发送到仅标头的SEND中。 如果客户的猜测是正确的，则服务器会在仅标头的SEND中发送低4个字节，否则，它将在常规的非仅标头SEND中发送整个8字节的值，稍后会触发客户猜测的更新。 只有很少一部分（<= C / 2^32^，C个客户端）的SEND是常规的。 我们将在第5节中进一步讨论这种推测技术。

We call this datagram-only sequencer Spec-S0 (speculation with header-only SENDs). Figure 7 shows its throughput with increasing server CPU cores. Spec-S0’s DMA bandwidth limit is higher than HERD RPCs because of smaller response WQEs; it achieves 122 Mrps and is limited by the NIC’s processing power instead of PCIe bandwidth. Spec-S0 has lower single-core throughput than the HERD RPC-based sequencer because of the additional CPU overhead of posting RECVs.

我们将此仅数据报的序列器称为Spec-S0（仅标头SEND的推测）。 图7显示了随着服务器CPU核心数量的增加其吞吐量变化。 由于较小响应WQE，因此Spec-S0的DMA带宽限制高于HERD RPC。 它可以达到122 Mrps，并且受NIC的处理能力（而不是PCIe带宽）的限制。 Spec-S0具有比基于HERD RPC的序列器更低的单核吞吐量，这是因为发布RECV会增加CPU开销。

### 4.2.2 延迟

![图8](https://raw.githubusercontent.com/liao20081228/blog/master/图片/Design_Guidelines_for_High_Performance_RDMA_Systems/图8.jpg)

Figure 8 shows the average end-to-end latency of Spec-S0 with and without response batching. Both modes receive a batch of requests from the NIC; the two modes differ only in the method used to send responses. The non-batched mode sends responses one-by-one using WQE-by-MMIO whereas the batched mode uses Doorbell when multiple responses are available to send. We batch atomic increments to the shared counter in both modes. We use 10 server CPU cores, which is the minimum required to achieve peak throughput. We measure throughput with increasing client load by adding more clients, and by increasing the number of outstanding requests per client. Batching adds up to 1 µs of latency because of the additional DMA read required with the Doorbell method. We believe that the small additional latency is acceptable because of the large throughput and CPU efficiency gains from batching.

图8显示了带和不带响应批处理的Spec-S0的平均端到端延迟。 两种模式都从NIC接收一批请求。 两种模式的区别仅在于发送响应的方法不同。 非批处理模式使用WQE-by-MMIO一个个地发送响应，而批处理模式在有多个响应可发送时使用Doorbell。 我们在两种模式下将原子增量批处理到共享计数器。 我们使用10个服务器CPU核心，这是达到峰值吞吐量所需的最低要求。 我们通过增加客户端数量以及增加每个客户端的未完成请求数量来衡量客户端负载增加时的吞吐量。 由于Doorbell方法需要额外的DMA读取，因此批处理总计需要1 µs的延迟。 我们认为，由于批处理可提高吞吐量和CPU效率，因此较小的附加延迟是可以接受的。

### 4.2.3 基于原子操作的序列器

![表3](https://raw.githubusercontent.com/liao20081228/blog/master/图片/Design_Guidelines_for_High_Performance_RDMA_Systems/表3.jpg)

Atomic fetch-and-add over RDMA is an appealing method to implement a sequencer: Binnig et al. [^11] use this design for the timestamp server in their distributed transaction protocol. However, lock contention for the counter among the NIC’s PUs results in poor performance. The effects of contention are exacerbated by the duration for which locks are held—several hundred nanoseconds for PCIe round trips. Our RPC-based sequencers have lower contention and shorter lock duration: the programmability of general-purpose CPUs allows us to batch updates to the counter which reduces cache line contention, and proximity to the counter’s storage (i.e., core caches) makes these updates fast. Figure 7 shows the throughput of our atomics-based sequencer: it achieves only 2.24 Mrps, which is 50x worse than our optimized design, and 12.2x worse than our single-core throughput. Table 3 summarizes the performance of our sequencers.

通过RDMA进行原子取回和添加是一种实现序列器的诱人方法：Binnig等。 [^11]将此设计用于其分布式事务协议中的时间戳服务器。 但是，NIC的PU之间计数器的锁争用会导致差的性能。 保持锁的持续时间加剧了争用的影响，PCIe往返的持续时间为数百纳秒。 我们基于RPC的定序器具有较低的争用和较短的锁定时间：通用CPU的可编程性使我们能够批量更新计数器，从而减少了缓存行的争用，并且由于与计数器存储（即核心缓存）接近，使得这些更新快速进行 。 图7显示了我们基于原子的序列器的吞吐量：仅达到2.24 Mrps，比优化设计低50倍，比单核吞吐量低12.2倍。 表3总结了我们序列器的性能。

## 4.3 键值存储

Several designs have been proposed for RDMA-based key-value storage. The HERD key-value cache uses HERD RPCs for all requests and does not bypass the remote CPU; other key-value designs bypass the remote CPU for key-value GETs (Pilaf [^24] and FaRMKV [^13]), or for both GETs and PUTs (DrTM-KV [^30] and Nessie [^27]). Our goal here is to demonstrate how our guidelines can be used to optimize or find flaws in RDMA system designs in general; we do not compare across different key-value systems. We first present performance improvements for HERD. Then, we show how the performance of atomics-based key-value stores is adversely affected by lock contention inside NICs.

已经提出了几种基于RDMA的键值存储的设计。 HERD键值高速缓存对所有请求都使用HERD RPC，并且不会绕过远程CPU。 其他键值设计绕过远程CPU进行键值GET（Pilaf [^24]和FaRMKV [^13]），或者GET和PUT（DrTM-KV [^30]和Nessie [^27]）。 我们的目标是论证通常如何使用我们的指南来优化或发现RDMA系统设计中的缺陷； 我们不会在不同的键值系统之间进行比较。 我们首先介绍HERD的性能改进。 然后，我们展示了基于原子的键值存储的性能如何受到NIC内部锁争用的不利影响。

### 4.3.1 提升HERD的性能

We apply batching to HERD as follows. After collecting N <= C requests, the server-side worker performs the GET or PUT operations on the backing datastore. It uses prefetching to hide the memory latency of accesses to the storage data structures [^32], [^23]. Then, the worker sends the responses either one-by-one using WQE-by-MMIO, or as a single batch using Doorbell.

我们将批处理应用于HERD，如下所示。 收集N <= C个请求后，服务器端工作线程将在备份数据存储上执行GET或PUT操作。 它使用预取来隐藏对存储数据结构的访问的内存延迟[^32]、[^23]。 然后，工作线程可以使用WQE-by-MMIO一对一地发送响应，也可以使用Doorbell来成批发送。

For evaluation, we run a HERD server with a variable number of workers on a CIB machine; we use 128 client threads running on eight client machines to issue requests. We pre-populate the key space partition owned by each worker with 8 million key-value pairs, which map 16-byte keys to 32-byte values. The workload consists of 95% GET and 5% PUT operations, with keys chosen uniformly at random from the inserted keys.

为了进行评估，我们在一台CIB机器上运行了HERD服务器，其中的工作线程数量可变。 我们使用在八台客户端计算机上运行的128个客户端线程来发出请求。 我们使用800万个键值对预先填充每个工作线程拥有的键空间分区，这些键值对将16字节的键映射到32字节的值。 工作负载由95％的GET和5％的PUT操作组成，键是从插入的键中随机选择的。

![图9](https://raw.githubusercontent.com/liao20081228/blog/master/图片/Design_Guidelines_for_High_Performance_RDMA_Systems/图9.jpg)

Figure 9 shows the throughput achieved in the above experiment. We also include the maximum throughput achievable by a READ-based key-value store such as Pilaf or FaRM-KV that uses >= 2 small READs per GET (one READ for fetching the index entry, and one for fetching the value). We compute this analytically by halving CIB’s peak inbound READ throughput (Section 5.3.1). We make three observations:

* Batching improves HERD’s per-core throughput by 83% from 6.7 Mrps to 12.3 Mrps. This improvement is smaller than for the sequencer because the CPU processing time saved from avoiding MMIOs is smaller relative to per-request processing in HERD than in the sequencer.
* Batching improves peak throughput by 35% from 72.8 Mrps to 98.3 Mrps. Batched throughput is bottlenecked by PCIe DMA bandwidth.
* With batching, HERD’s throughput is up to 63% higher than a READ-based key-value store. While HERD’s original non-batched design requires 12 cores to outperform a READ-based design, only 7 cores are needed with batching. This highlights the importance of including low-level factors such as batching when comparing RDMA system designs.

图9显示了上述实验中实现的吞吐量。我们还包括通过基于READ的键值存储（例如Pilaf或FaRM-KV）实现的最大吞吐量，该存储使用 >= 2个小READ每个GET（一个READ用于获取索引项，一个READ用于获取值）。我们通过将CIB的峰值入站READ吞吐量减半来进行分析计算（第5.3.1节）。我们进行三个观察：

* 批量处理将HERD的每核吞吐量提高了83％，从6.7 Mrps增加到12.3 Mps。 这种改进比序列器要小，因为HERD中避免MMIO节省的CPU处理时间相对小于按每个请求处理的时间。
* 批处理可将峰值吞吐量从72.8 Mrps提高35％至98.3 Mrps。 PCIe DMA带宽是批量吞吐量的瓶颈。
* 通过批量处理，HERD的吞吐量比基于READ的键值存储高出63％。 HERD的原始非批处理设计需要12个核才能胜过基于READ的设计，而批处理仅需要7个核。 这凸显了比较RDMA系统设计时引入批处理等低级因素的重要性。


### 4.3.2 基于原子操作的键值存储

DrTM-KV [^30] and Nessie [^27], [^28] use RDMA atomics to bypass the remote CPU for both GETs and PUTs. However, these projects do not consider the impact of the NIC’s concurrency control on performance, and present performance for either GET-only (DrTM-KV) or GET mostly workloads (Nessie). We now show that locking inside the NIC results in low PUT throughput, and degrades throughput even when only a small fraction of key-value operations are PUTs.

DrTM-KV [^30]和Nessie [^27]，[^28]使用RDMA原子绕过GET和PUT的远程CPU。 但是，这些项目没有考虑NIC的并发控制对性能的影响，而仅针对GET（DrTM-KV）或GET大多数工作负载（Nessie）的当前性能。 现在，我们显示NIC内的锁会导致PUT吞吐量降低，并且即使只有一小部分键值操作是PUT，也会降低吞吐量。

We discuss DrTM-KV here because of its simplicity, but similar observations apply to Nessie. DrTM-KV caches some fields of its key-value index at all clients; GETs for cached keys use one READ. PUT operations lock, update, and unlock key-value items; locking and unlocking is done using atomics. Running DrTM-KV’s codebase on CIB requires significant modification because CIB’s dual-port NIC are connected in a way that does not allow cross-port communication. To
overcome this, we wrote a simplified emulated version of DrTM-KV: we emulate GETs with 1 READ and PUTs with 2 atomics, and assume a 100% cache hit rate.

由于其简单性，我们在此讨论DrTM-KV，但是类似的观察也适用于Nessie。 DrTM-KV在所有客户端缓存其键值索引的某些字段； 缓存键的GET使用一次READ。 PUT操作锁定，更新和解锁键值项目； 锁定和解锁是使用原子完成的。 在CIB上运行DrTM-KV的代码需要进行重大修改，因为CIB的双端口NIC的连接方式不允许跨端口通信。 为了克服这个问题，我们编写了DrTM-KV的简化仿真版本：我们模拟具有1个READ的GET和具有2个原子的PUT，并假定100％的高速缓存命中率。

![图10](https://raw.githubusercontent.com/liao20081228/blog/master/图片/Design_Guidelines_for_High_Performance_RDMA_Systems/图10.jpg)

Figure 10 shows the throughput of our emulated DrTM-KV server with different fractions of PUT operations in the workload. The server hosts 16 million items with 16-byte keys and 32-byte values. Clients use randomly chosen keys and we use as many clients as required to maximize throughput. Although throughput for a 100% GET workload is high, adding only 10% PUTs degrades it by 72% on CX3 and 31% on CIB. Throughput with 100% PUTs is a tiny fraction of GET-only throughput: 4% on CX3 and 12% on CIB. Note that the degradation for CIB is more gradual than for CX3 because CIB has a better locking mechanism, as shown in Section 5.

图10显示了仿真DrTM-KV服务器在工作负载中具有不同比例的PUT操作的吞吐量。 该服务器托管1600万个项目，其中包含16个字节的键和32个字节的值。 客户使用随机选择的键，而我们需要使用尽可能多的客户以最大化吞吐量。 尽管100％GET工作负载的吞吐量很高，但仅添加10％PUT会使CX3的性能降低72％，而CIB的性能降低31％。 100％PUT的吞吐量是仅GET吞吐量的一小部分：CX3为4％，CIB为12％。 请注意，因为CIB具有更好的锁定机制，所以CIB的降级比CX3的降级更为缓慢，如第5节所示。

# 5 RDMA的低层级因素

![表4](https://raw.githubusercontent.com/liao20081228/blog/master/图片/Design_Guidelines_for_High_Performance_RDMA_Systems/表4.jpg)

Our guidelines and system designs are based on an improved understanding of low-level factors that affect RDMA performance, including I/O initiation mechanisms, PCIe, and NIC architecture. These factors are complicated and there is little existing literature describing them or studying their relevance to networked systems. We attempt to fill this void by presenting clarifying performance measurements, experiments, and models; Table 4 shows a partial summary for CIB. Additionally, we discuss the importance of these factors to general RDMA system design beyond the two systems in Section 4.

我们的准则和系统设计基于对影响RDMA性能的低级因素（包括I/O启动机制，PCIe和NIC体系结构）的深入了解。 这些因素很复杂，现有文献很少描述或研究它们与网络系统的相关性。 我们尝试通过提供明确的性能指标，实验和模型来填补这一空白。 表4显示了CIB的部分总结。 此外，在第4节中，我们将讨论这些因素对除了第4节中的两个系统以外的RDMA系统总体设计的重要性。

We divide our discussion into three common use cases that highlight different low-level factors: (1) batched operations, (2) non-batched operations, and (3) atomic operations. For each case, we present a performance analysis focusing on hardware bottlenecks and optimizations, and discuss implications on RDMA system design.

我们将讨论分为三个常见的用例，这些用例强调了不同的低层因素：（1）批处理操作，（2）非批处理操作和（3）原子操作。 对于每种情况，我们都会针对硬件瓶颈和优化进行性能分析，并讨论对RDMA系统设计的影响。

## 5.1 PCIe模型
We have demonstrated that understanding the PCIe behavior is critical for improving RDMA performance. How-ever, deriving analytical models of PCIe behavior without access to proprietary/confidential NIC manuals and our limited resources—per–cache line PCIe counters and undocumented driver software—required extensive experimentation and analysis. Our derived models are presented in a slightly simplified form at several points in this paper (Figures 5, 6, 13). The exact analytical models are complicated and depend on several factors such as the NIC, its PCIe capability, the verb and transport, the level of Doorbell batching, etc. To make our models easily accessible, we instrumented the datapath of two Mellanox drivers (ConnectX-3 and Connect-IB) to provide statistics about PCIe bandwidth use (<https://github.com/efficient/rdma_bench>). Our models and drivers are restricted to requester-side PCIe behavior. We omit responder-side PCIe behavior because it is the same as described in our previous work [^20]: inbound READs and WRITEs generate one PCIe read and write, respectively; inbound SENDs trigger a RECV completion—we discuss the PCIe transactions for the RECV.

我们已经证明，了解PCIe行为对于提高RDMA性能至关重要。但是，在没有访问专有/机密NIC手册和我们有限的资源（每个缓存行PCIe计数器和未记录的驱动程序软件）的情况下，得出PCIe行为的分析模型需要大量的实验和分析。我们的派生模型在本文的几个地方以稍微简化的形式呈现（图5、6、13）。 精确的分析模型很复杂，并且取决于NIC，其PCIe功能，动词和传输，Doorbell批处理的级别等多个因素。为了使我们的模型易于访问，我们检测了两个Mellanox驱动程序（ConnectX-3和Connect-IB）的数据路径，以提供有关PCIe带宽使用的统计信息（<https://github.com/efficiency/rdma_bench>）。我们的模型和驱动程序仅限于请求方PCIe行为。 我们省略了响应方的PCIe行为，因为它与我们先前的工作[^20]中描述的相同：入站READ和WRITE分别生成一个PCIe读写。 入站SEND触发RECV完成-我们讨论RECV的PCIe事务。

## 5.2 批处理操作

**A limitation of batching** on current hardware makes it useful mainly for datagram transport: all operations in a batch must use the same queue pair because Doorbells are per QP. This limitation seems fundamental to the parallel architecture of NICs: In a hypothetical NIC design where Doorbells contained information relevant for multiple queue pairs (e.g., a compact encoding of “2 and 1 new WQEs for QP 1 and QP 2, respectively”), sending the Doorbell to the NIC processing units handling these QPs would require an expensive selective broadcast inside the NIC. These PUs would then issue separate DMAs for WQEs, losing the coalescing advantage of batching. This limitation makes batching less useful for connected QPs, which provide only one-to-one communication between two machines: the chances that a process has multiple messages for the same remote machine are low in large deployments. We therefore discuss batching for UD transport only.

**批处理的限制** 在当前硬件上，它主要用于数据报传输：批处理中的所有操作都必须使用相同的队列对，因为Doorbell是每个QP。 此限制对于NIC的并行体系结构似乎至关重要：在假设的NIC设计中，Doorbell包含与多个队列对相关的信息（例如，“QP 1两个WQE和QP 2 一个WQE”的紧凑编码），发送Doorbell到处理这些QPs的NIC处理单元将需要在NIC内部进行昂贵的选择性广播。 然后，这些PUs将为WQE发出单独的DMA，从而失去批处理的合并优势。 此限制使批处理对于有连接的QPs不太有用，QPs仅在两台计算机之间提供一对一的通信：在大型部署中，一个进程具有针对同一远程计算机的多个消息的机会很小。 因此，我们仅讨论UD传输的批处理。

## 5.2.1 UD SENDs

![图11](https://raw.githubusercontent.com/liao20081228/blog/master/图片/Design_Guidelines_for_High_Performance_RDMA_Systems/图11.jpg)

Figure 11 shows the throughput and PCIe bandwidth limit of batched and non-batched UD SENDs on CIB. We use one server to issue SENDs to multiple client machines. With batching, we use batches of size 16 (i.e., the NIC DMAs 16 work queue elements per Doorbell). Otherwise, the CPU writes WQEs by the WQE-by-MMIO method. We use as many cores as required to maximize throughput. Batching improves peak SEND throughput by 27% from 80 million operations/s (Mops) to 101.6 Mops.

图11显示了CIB上批处理和非批处理的吞吐量和PCIe带宽限制。 我们使用一台服务器将SEND发送到多台客户端计算机。 对于批处理，我们使用大小为16的批处理（每个Doorbell的NIC DMA 16个WQE）。 否则，CPU通过WQE-by-MMIO方法写入WQE。 我们使用了尽可能多的核心以最大化吞吐量。 批处理将峰值SEND吞吐量从8000万次/秒提高了27％，达到101.6次/秒。

**Bottlenecks** Batched throughput is limited by DMA bandwidth. For every DMA completion of size Crc bytes, there is header overhead of Pc bytes (Section 2.1), leading to 13443 MB/s of useful DMA read bandwidth on CIB. As UD WQEs span at least 2 cache lines, the maximum WQE transfer rate is 13443=128 = 105 million/s, which is within 5% of our achieved throughput; we attribute the difference to link- and physical-layer PCIe overheads.

**瓶颈** 批处理吞吐量受DMA带宽限制。 对于每一个Crc字节大小的DMA完成，都会有Pc字节的报头开销（第2.1节），从而在CIB上产生13443 MB/s的有用DMA读取带宽。 由于UD WQE至少跨越2条高速缓存行，因此最大WQE传输速率为13443 /128 = 1.05亿/ s，这是我们实现的吞吐量的5％之内； 我们将差异归因于链路层和物理层PCIe开销。

Non-batched throughput is limited by MMIO bandwidth. The write-combining MMIO rate on CIB is (16 x P~bw~)/(64 + P~r~ ) = 175 million cache lines/s. UD SEND WQEs with non-zero payload span at least 2 cache lines (Table 1), and achieve up to 80 Mops. This is within 10% of the 87.5 Mops bandwidth limit.

非批处理吞吐量受MMIO带宽限制。 CIB上的写入组合MMIO速率为(16 x P~bw~)/(64 + P~r~ ) = 1.75亿个缓存行/秒。 具有非零有效负载的UD SEND WQE至少跨越2条缓存行（表1），并达到80Mops。 这在87.5 Mops带宽限制的10％以内。

**Multi-queue optimization** Figure 11b shows singlecore throughput for batched and non-batched 60-byte UD SENDs—the largest payload size for which the WQEs fit in 2 cache lines. Interestingly, batching decreases core throughput if only one QP is used: with one QP, a core is coupled to a NIC processing unit (Section 3.3), so throughput depends on how the PU handles batched and non-batched operations. Batching has the expected effect when we break this coupling by using multiple QPs. Batched throughput increases by  ~2x on all clusters with 2 QPs, and between 2–3.2x with 4 QPs. Nonbatched (WQE-by-MMIO) throughput does not increase with multiple QPs (not shown in graph), showing that it is CPU-limited.

**多队列优化** 图11b显示了批处理和非批处理的60字节UD SEND的单核吞吐量，这是WQE在2个缓存行中适合的最大有效负载大小。 有趣的是，如果仅使用一个QP，则批处理会降低核心吞吐量：如果使用一个QP，则将一个核心绑定到NIC处理单元（第3.3节），因此，吞吐量取决于PU处理批处理和非批处理操作的方式。 当我们通过使用多个QP打破这种耦合时，批处理具有预期的效果。 在具有2个QP的所有集群上，批处理的吞吐量提高了约2倍，而在具有4个QP的集群中，批量处理的吞吐量提高了2–3.2倍。 非批处理（按MMIO的WQE）吞吐量不会随着多个QP的增加而增加（图中未显示），表明它受CPU限制。

**Design implications** RDMA-based systems can often choose between CPU-bypassing and CPU-involving designs. For example, clients can access a key-value store either by READing directly from the server’s memory [^24], [^13], [^30], [^28], or via RPCs as in HERD [^20]. Our results show that achieving peak performance on even the most powerful NICs does not require a prohibitive amount of CPU power: only 4 cores are needed to saturate the fastest PCIe links. Therefore, CPU-involving designs will not be limited by CPU processing power, provided that their application-level processing permits so.

**设计意义** 基于RDMA的系统通常可以在绕过CPU和调用CPU之间进行选择。 例如，客户可以通过直接从服务器的内存 [^24], [^13], [^30], [^28]进行读取或通过HERD [20]中的RPC访问键值存储。 我们的结果表明，即使在功能最强大的NIC上也能达到峰值性能，也不需要大量的CPU功能：只需4个核即可使最快的PCIe链路饱和。 因此，只要应用程序级处理允许，涉及CPU的设计就不会受到CPU处理能力的限制。

### 5.2.2 UD RECVs

![图12](https://raw.githubusercontent.com/liao20081228/blog/master/图片/Design_Guidelines_for_High_Performance_RDMA_Systems/图12.jpg)

![表5](https://raw.githubusercontent.com/liao20081228/blog/master/图片/Design_Guidelines_for_High_Performance_RDMA_Systems/表5.jpg)

Table 5 compares the throughput of header-only and payload-carrying regular RECVs (Figure 6). In our experiment, multiple client machines issue SENDs to one server machine that posts RECVs. On CIB, avoiding the payload DMA with header-only RECVs increases throughput by 49% from 82 Mops to 122 Mops, and makes them as fast as inbound WRITEs (Figure 12a).4（Inline-receive improves regular RECV throughput from 22 Mops to 26 Mops on CX3, but is not yet supported for UD on CIB.）Table 5 also compares header-only and regular SENDs (Figure 5). Header-only SENDs use single-cache line WQEs and achieve 54% higher throughput.

表5比较了仅标头和有效载荷常规RECV的吞吐量（图6）。 在我们的实验中，多台客户端计算机向一台发布RECV的服务器计算机发出SEND。 在CIB上，避免使用仅带有标头的RECV的有效负载DMA将吞吐量从82 Mops增加到122 Mops，提高了49％，使其与入站WRITE一样快（图12a）。（内联receive将CX3上的常规RECV吞吐量从22 Mops提高到26 Mops，但是CIB上的UD尚不支持。）表5还比较了仅标头和常规SEND（图8a）。 5）。 仅标头的SEND使用单缓存行WQE，并实现54％的更高高的吞吐量。

**Design implications** Developers avoid RECVs at performance critical machines as a rule of thumb, favoring the faster READ/WRITE verbs [^24], [^20]. Our work provides the exact reason: RECVs are slow due to the CQE DMA; they are as fast as inbound WRITEs if it is avoided.

**设计意义** 开发人员通常会避免在性能关键型计算机上使用RECV，而倾向于使用更快的READ / WRITE动词[^24]，[^20]。 我们的工作提供了确切的原因：由于CQE DMA，RECV速度很慢； 如果避免的话，它们和入站WRITE一样快。

**Speculation**  Current RDMA implementations allow 4 bytes of application data in the packet header of header-only SENDs. For applications that require larger messages,header-only SEND/RECV can be used if speculation is possible; we demonstrated such a design for an 8-byte sequencer in Section 4.2. In general, speculation works as follows: clients transmit their expected response along with requests, and get a small confirmation response in the common case. For example, in a key-value store with client-side caching, clients can send GET requests with the key and its cached version number (using a WRITE or regular SEND). The server replies with a header-only “OK” SEND if the version is valid.

**推测** 当前的RDMA实现在仅标头的SEND的数据包标头中允许4字节的应用程序数据。 对于需要较大消息的应用程序，如果可以推测，则可以使用仅标头的SEND / RECV。 我们在4.2节中证明了8字节序列器的这种设计。 通常，推测的工作方式如下：客户端将其预期的响应与请求一起发送，并在通常情况下获得少量确认响应。 例如，在具有客户端缓存的键值存储中，客户端可以发送带有键及其缓存版本号的GET请求（使用WRITE或常规SEND）。 如果版本有效，服务器将以仅标头的“ok”SEND答复。

There are applications for which 4 bytes of per-message data suffices. For example, some database tables in the TPC-C [^29] benchmark have primary key size between 2 and 3 bytes. A table access request can be sent using a header-only SEND (using the remaining 1 byte to specify the table ID), while the response may need a larger SEND.


在某些应用中，每个消息的数据只需4字节即可。 例如，TPC-C [^29]基准测试中的某些数据库表的主键大小在2到3个字节之间。 可以使用仅标头的SEND（使用剩余的1个字节指定表ID）发送表访问请求，而响应可能需要更大的SEND。

## 5.3 非批处理操作
### 5.3.1 入站READs和WRITEs

Figure 12 shows the measured throughput of inbound READs and UC WRITEs, and the InfiniBand bandwidth limit of inbound READs. We do not show the InfiniBand limit for WRITEs and the PCIe limits as they are higher。

图12显示了测得的入站READ和UC WR的吞吐量，以及入站READ的InfiniBand带宽限制。 我们没有显示WRITE的InfiniBand限制和PCIe的限制，因为它们更高。

**Bottlenecks**  On our clusters, inbound READs and WRITEs are initially bottlenecked by the NIC’s processing power, and then by InfiniBand bandwidth. The payload size at which bandwidth becomes a bottleneck depends on the NIC’s processing power relative to bandwidth. For READs, the transition point is approximately 128 bytes, 256 bytes, and 64 bytes for CX, CX3, and CIB, respectively. CIB NICs are powerful enough to saturate 112 Gbps with 64-byte READs, whereas CX3 NICs require 256-byte READs to saturate 56 Gbps.

**瓶颈** 在我们的集群中，入站读写首先受到NIC处理能力的瓶颈，然后受到InfiniBand带宽的瓶颈。带宽成为瓶颈的有效负载大小取决于NIC相对于带宽的处理能力。 对于READ，CX，CX3和CIB的过渡点分别约为128字节，256字节和64字节。 CIB NIC的足以强大通过64字节的READ达到112 Gbps的饱和带宽，而CX3 NIC需要256字节的READ达到56 Gbps的饱和带宽。

**Implications** The transition point is an important factor for systems that make tradeoffs between the size and number of READs: For key-value lookups of small items (~32 bytes), FaRM’s key-value store [^13] can use one large (~256-byte) READ. In a client-server design where inbound READs determine GET performance, this designperforms well on CX3 because 32- and 256-byte READs have similar throughput; other designs such as DrTMKV [^0] and Pilaf [^24] that instead use 2–3 small READs may provide higher throughput on CIB.

**推测**对于要在READ的大小和数量之间进行权衡的系统，转换点是一个重要因素：对于小项目（〜32字节）的键值查找，FaRM的键值存储[^13]可以使用一个大的值（〜256- 字节）READ。 在客户端服务器设计中，入站READ决定GET性能，该设计在CX3上表现良好，因为32字节和256字节READ具有相似的吞吐量。 其他设计（例如DrTMKV [^30]和Pilaf [^24]）使用2–3个较小的READ可以提供更高的CIB吞吐量。

### 5.3.2 出站READs和WRITEs
For brevity, we only present a summary of the performance of non-batched outbound operations on CIB. Outbound UC WRITEs larger than 28 bytes, i.e., WRITEs with WQEs spanning more than one cache line (Table 1), achieve up to 80 Mops and are bottlenecked by PCIe MMIO throughput, similar to non-batched outbound SENDs (Figure 11a). READs achieve up to 88 Mops and are bottlenecked by NIC processing power.

为简便起见，我们仅介绍CIB上非批量出站操作的性能摘要。 大于28个字节的出站UC WRITE，即WQE跨越多个缓存行的WRITE（表1），达到80Mops，并且受到PCIe MMIO吞吐量的瓶颈，类似于非批量出站SEND（图11a）。 读取最多可达到88Mops，并且受NIC处理能力的限制。

![图13](https://raw.githubusercontent.com/liao20081228/blog/master/图片/Design_Guidelines_for_High_Performance_RDMA_Systems/图13.jpg)
**Achieving high outbound throughput** requires maintaining multiple outstanding requests via pipelining. When the CPU initiates an RDMA operation, the work queue element is inserted into the NIC’s WQE cache. However, if the CPU injects new WQEs faster than the NIC’s processing speed, this WQE can be evicted by newer WQEs. This can cause cache misses when the NIC eventually processes this WQE while generating its RDMA request packets, while servicing its RDMA response, or both. Figure 13a summarizes this model.

**实现高出站吞吐量** 需要通过流水线维护多个未完成的请求。 当CPU启动RDMA操作时，工作队列元素将插入NIC的WQE缓存中。 但是，如果CPU注入新WQE的速度快于NIC的处理速度，则可以由更新的WQE驱逐此WQE。 当NIC在生成其RDMA请求数据包，为RDMA响应提供服务，或同时为两者提供服务时，最终处理此WQE时，这可能导致缓存丢失。 图13a总结了该模型。

To quantify this effiect, we conduct the following experiment on CIB: 14 requester threads on a server issue windows of N 8-byte READs or WRITEs over reliable transport to 14 remote processes. In Figures 13b and 13c, we show the cumulative RDMA request rate, and the extent of WQE cache misses using the PCIeRdCur counter rate. Each thread waits for the N requests to complete before issuing the next window. We use all 14 cores on the server to generate the maximum possible request rate, and RC transport to include cache misses generated while processing ACKs for WRITEs. We make the following observations, showing the importance of the WQE cache in improving and understanding RDMA throughput:

* The optimal window size for maximum throughput is not obvious: throughput does not always increase with increasing window size, and is dependent on the NIC. For example, N = 16 and N = 512 maximize READ throughput on CX3 and CIB respectively.

* Higher RDMA throughput may be obtained at the cost of PCIe reads. For example, on CIB, both READ throughput and PCIe read rate increases as N increases. Although the largest N is optimal for a machine that only issues outbound READs, it may be suboptimal if it also serves other operations.

* CIB’s NIC can handle the CPU’s peak WQE injection rate for WRITEs and never suffiers cache misses. This is not true for READs, indicating that they require more NIC processing than reliable WRITEs.

为了量化这种效果，我们在CIB上进行了以下实验：服务器上的14个请求者线程发出N个可靠传输的8字节READ或WRITE窗口到14个远程进程。 在图13b和13c中，我们使用PCIeRdCur计数器速率显示了累积的RDMA请求速率以及WQE缓存未命中的程度。 每个线程在发出下一个窗口之前等待N个请求完成。 我们使用服务器上的所有14个核心来生成最大可能的请求速率，并使用RC传输来包含在处理WRITE的ACK时生成的缓存未命中。 我们进行以下观察，以显示WQE缓存在改善和理解RDMA吞吐量方面的重要性：

* 最大吞吐量的最佳窗口大小并不明显：吞吐量并不总是随窗口大小的增加而增加，而是取决于NIC。 例如，N = 16和N = 512分别使CX3和CIB的READ吞吐量最大化。

* 可以以PCIe读取为代价获得更高的RDMA吞吐量。 例如，在CIB上，读取吞吐量和PCIe读取速率都随着N的增加而增加。 尽管最大的N对于仅发出出站READ的机器是最佳的，但如果它还用于其他操作，则可能不是最佳选择。

* CIB的NIC可以处理WRITE的CPU峰值WQE注入速率，并且永远不会使高速缓存未命中。 对于READ而言并非如此，这表明与可靠的WRITE相比，它们需要更多的NIC处理。



## 5.4 原子操作

![图14](https://raw.githubusercontent.com/liao20081228/blog/master/图片/Design_Guidelines_for_High_Performance_RDMA_Systems/图14.jpg)
NIC processing units contend for locks during atomic operations (Section 3.4). The performance of atomics depends on the amount of parallelism in the workload with respect to the NIC’s internal locking scheme. To vary the amount of parallelism, we create an array of Z 8 byte counters in a server’s memory, and multiple remote client processes issue atomic operations on counters chosen randomly at each iteration. Figure 14 shows the total client throughput in this experiment. For CX3, it remains 2.7 Mops irrespective of Z; for CIB, it rises to 52 Mops.

NIC处理单元在原子操作期间争用锁（第3.4节）。 原子操作的性能取决于与NIC内部锁方案有关的工作负载中的并行度。 为了改变并行度，我们在服务器的内存中创建了一个Z 8字节计数器数组，并且多个远程客户端进程对每次迭代随机选择的计数器发出原子操作。 图14显示了此实验中的总客户端吞吐量。 对于CX3，与Z无关，它仍然为2.7 Mops； 对于CIB，上升到52Mops。

**Inferring the locking mechanism** The flatness of CX3’s throughput graph indicates that it serializes all atomic operations. For CIB, we measured performance with randomly chosen pairs of addresses and observed lower performance for pairs where both addresses have the same 12 LSBs. This strongly suggests that CIB uses 4096 buckets to slot atomic operations by address—a new operation waits until its slot is empty.

**推断锁定机制** CX3吞吐量图表的平坦度表明它已序列化了所有原子操作。 对于CIB，我们使用随机选择的地址对来测量性能，而对于两个地址具有相同的12 LSB的对，则观察到较低的性能。 这有力地表明，CIB使用4096个桶按地址分配原子操作——一个新的操作会一直等到其插槽为空。

**Bottlenecks and implications*  Throughput on CX3 is limited by PCIe latency because of serialization. For CIB, buffering and computation needed for PCIe read-modifywrite makes NIC processing power the bottleneck.

**瓶颈和影响** 由于串行化，CX3的吞吐量受到PCIe延迟的限制。 对于CIB，PCIe读-改-写所需的缓冲和计算使NIC处理成为瓶颈。

The abysmal throughput for Z = 1 on both NICs reaffirms that atomics are a poor choice for a sequencer; our optimized sequencer in Section 4 provides 12.2x higher performance with a single server CPU core. A lock service for data stores, however, might use a larger Z. Atomics could perform well if such an application used CIB, but they are very slow with CX3, which is the NIC used in prior work [27, 30]. With CIB, careful lock placement is still necessary. For example, if page-aligned data records have their lock variables at the same offset in the record, all lock requests will have the same 12 LSBs and will get serialized. A deterministic scheme that places the lock at different offsets in different records, or a scheme that keeps locks separate from the data will perform better.

两个NIC上Z = 1的极高吞吐率都再次证明了原子操作对于序列器不是一个好的选择。 我们在第4节中优化的序列器通过单个服务器CPU核心提供了12.2倍的更高性能。 但是，用于数据存储的锁定服务可能使用较大的Z。
如果这样的应用程序使用CIB，则原子可能会表现良好，但使用CX3的速度会非常慢，而CX3是先前工作中使用的NIC [27，30]。 对于CIB，仍然需要仔细放置锁。 例如，如果页面对齐的数据记录的锁定变量在记录中的偏移量相同，则所有锁定请求将具有相同的12个LSB，并将被序列化。 将锁放在不同记录中的不同偏移量处的确定性方案，或将锁与数据分开的方案的性能会更好。

# 6 相关工作
**High-performance RDMA systems**
Designing high performance RDMA systems is an active area of research. Recent advances include several key-value storage systems[^24]、[^13]、[^20]、[^30]、[^28] and distributed transaction processing systems [^30]、[^12]、[^14]、[^11]. A key design decision in each of these systems is the choice of verbs, made using a microbenchmark-based performance comparison. Our work shows that there are more dimensions to these comparisons than these projects explore: two verbs cannot be exhaustively compared without exploring the space of low-level factors and optimizations, each of which can offset verb performance by several factors.

**高性能RDMA系统** 设计高性能RDMA系统是一个活跃的研究领域。 最新进展包括几个关键值存储系统
[^24]、[^13]、[^20]、[^30]、[^28]和分布式事务处理系统[^30]、[^12]、[^14]、[^11]。 这些系统中每个系统的关键设计决策是动词的选择，动词的选择是基于基于微基准的性能比较而做出的。 我们的工作表明，进行这些比较的范围要比这些项目要探讨的要多：两个动词如果不探索低级因素和优化的空间就无法进行详尽的比较，而每种因素都会使动词的表现受到几个因素的影响。

**Low-level factors in network I/O**Although there is a large body of work that measures the throughput and CPU utilization of network communication [^18], [^16], [^26], [^13], [^20], there is less existing literature on understanding the lowlevel behavior of network cards. NIQ [^15] presents a highlevel picture of the PCIe interactions between an Ethernet NIC and CPUs, but does not discuss the more subtle interactions that occur during batched transfers. Lee et al. [^22] study the PCIe behavior of Ethernet cards using
a PCIe protocol analyzer, and divide the PCIe traffic into Doorbell traffic, Ethernet descriptor traffic, and actual data traffic. Similarly, analyzing RDMA NICs using a PCIe analyzer may reveal more insights into their behavior than what is achievable using PCIe counters.

**网络I / O中的底层因素** 尽管有大量工作测量网络通信的吞吐量和CPU利用率[^18], [^16], [^26], [^13], [^20]，但是关于理解网卡低级行为的现有文献很少。NIQ [^15]概述了以太网NIC和CPU之间的PCIe交互，但没有讨论批量传输期间发生的更细微的交互。 Lee等。 [^22]使用PCIe协议分析器研究以太网卡的PCIe行为，并将PCIe流量分为Doorbell流量，以太网描述符流量和实际数据流量。 同样，使用PCIe分析器分析RDMA NIC可能比使用PCIe计数器获得更多有关其行为的见解。


# 7 结论
Designing high-performance RDMA systems requires a deep understanding of low-level RDMA details such as PCIe behavior and NIC architecture: our best sequencer is ~50x faster than an existing design and scales perfectly, our optimized HERD key-value store is up to 83% faster than the original, and our fastest transmission method is up to 3.2x faster than the commonly-used baseline. We believe that by presenting clear guidelines, significant optimizations based on these guidelines, and tools and experiments for low-level measurements on their hardware, our work will encourage researchers and developers to develop a better understanding of RDMA hardware before using it in high-performance systems.

设计高性能RDMA系统需要深入了解低级RDMA详细信息，例如PCIe行为和NIC架构：我们最好的序列器比现有设计快约50倍，并且可以完美扩展，我们优化的HERD键值存储库最多比原始速度快83％，并且我们最快的传输方法比常用基准速度快3.2倍。我们相信，通过提供清晰的指导方针，基于这些指导方针的重大优化以及用于对其硬件进行低级测量的工具和实验，我们的工作将鼓励研究人员和开发人员在将RDMA硬件用于高性能系统之前更好地理解它。 

# 致谢
We are tremendously grateful to Joseph Moore and NetApp for providing access to theCIB cluster. We thank Hyeontaek Lim and Sol Boucher for providing feedback, and Liuba Shrira for shepherding. Emulab [^31] and PRObE [^17] resources were used in our experiments. PRObE is supported in part by NSF awards CNS-1042537 and CNS-1042543 (PRObE). This work was supported by funding from the National Science Foundation under awards 1345305 and 1314721, and by Intel via the Intel Science and Technology Center for Cloud Computing (ISTC-CC).

我们非常感谢Joseph Moore和NetApp提供对CIB群集的访问。 我们感谢Hyeontaek Lim和Sol Boucher提供的反馈，以及Liuba Shrira的指导。 在我们的实验中使用了Emulab [31]和PRObE [17]资源。 PRObE部分受到NSF奖励CNS-1042537和CNS-1042543（PRObE）支持。 这项工作得到了美国国家科学基金会（National Science Foundation）授予的1345305和1314721奖项的资助，以及英特尔通过英特尔云计算技术中心（ISTC-CC）的资助。

# 附录
**WQE-by-MMIO and Doorbell PCIe use** We denote the doorbell size by d. The total data transmitted from CPU to NIC with the WQE-by-MMIO method is T~bf~ = 10 \* (⌈65/64⌉ \* ( 64 + P~r~ )) bytes. With cache line padding, 65-byte WQEs are laid out in 128-byte slots in host memory; assuming C~rc~ = 128, T~db~ = (d+P~r~ )+(10\*(128+P~c~ )) bytes. We ignore the PCIe linklayer traffic since it is small compared to transaction-layer traffic: it is common to assume 2 link-layer packets (1 flow control update and 1 acknowledgment, both 8 bytes) per 4-5 TLPs [^9], making the link-layer overhead < 5%. Substituting d = 8 gives T~bf~ = 1800, and T~db~ = 1534.

我们用d表示Doorbell尺寸。 使用WQE-by-MMIO方法从CPU传输到NIC的总数据为 T~bf~ = 10 \* (⌈65/64⌉ \* ( 64 + P~r~ )) 字节。 通过缓存行填充，可以将65字节的WQE布置在主机内存的128字节的插槽中。 假设C~rc~ = 128, T~db~ = (d+P~r~ )+(10\*(128+P~c~ ))个字节。 我们忽略PCIe链路层流量，因为它比事务层流量小：通常每4-5个TLP假设2个链路层数据包（1个流量控制更新和1个确认，均为8个字节）[^ 9]， 使链路层开销<5％。 代入d = 8得到T~bf~ = 1800, and T~db~ = 1534。

[^1]: Infiniband architecture specification volume 1.https://cw.infinibandta.org/document/dl/7859.

[^2]: Intel Atom Processor C2000 Product Family for Microserver. http://www.intel.in/content/dam/www/public/us/en/documents/datasheets/atom-c2000-microserver-datasheet.pdf.

[^3]: Intel Xeon Processor E5-1600/2400/2600/4600 v3 Product Families. http://www.intel.com/content/dam/www/public/us/en/documents/datasheets/xeon-e5-v3-datasheet-vol-2.pdf.

[^4]:  Intel Xeon Processor E5-1600/2400/2600/4600(E5-Product Family) Product Families.http://www.intel.com/content/dam/www/public/us/en/documents/datasheets/xeon-e5-1600-2600-vol-2-datasheet.pdf.

[^5]:  Intel Xeon Processor D-1500 Product Family.http://www.intel.in/content/dam/www/public/us/en/documents/product-briefs/xeon-processor-d-brief.pdf.

[^6]: Intel Xeon Phi Processor Knights Landing Architectural Overview. https://www.nersc.gov/assets/Uploads/KNL-ISC-2015-Workshop-Keynote.pdf.

[^7]: Mellanox ConnectX-4 product brief. http://www.mellanox.com/related-docs/prod_silicon/PB_ConnectX-4_VPI_Card.pdf.

[^8]: Mellanox OFED for linux user manual.http://www.mellanox.com/related-docs/prod_software/Mellanox_OFED_Linux_User_Manual_v2.2-1.0.1.pdf.

[^9]: Understanding Performance of PCI Express Systems. http://www.xilinx.com/support/documentation/white_papers/wp350.pdf.

[^10]: M. Balakrishnan, D. Malkhi, V. Prabhakaran,T. Wobber, M. Wei, and J. D. Davis. CORFU: a shared log design for flash clusters. In Proc. 9th USENIX NSDI, Apr. 2012.

[^11]: C. Binnig, U. Çetintemel, A. Crotty, A. Galakatos,T. Kraska, E. Zamanian, and S. B. Zdonik. The end of slow networks: It’s time for a redesign. CoRR,abs/1504.01048, 2015. URL http://arxiv.org/abs/1504.01048.

[^12]: Y. Chen, X. Wei, J. Shi, R. Chen, and H. Chen. Fast and general distributed transactions using RDMA and HTM. In Proc. 11th ACM European Conference on Computer Systems (EuroSys), Apr. 2016.

[^13]: A. Dragojevi´c, D. Narayanan, O. Hodson, and M. Castro. FaRM: Fast remote memory. In Proc. 11th USENIX NSDI, Apr. 2014.

[^14]: A. Dragojevi´c, D. Narayanan, E. B. Nightingale, M. Renzelmann, A. Shamis, A. Badam, and M. Castro. No compromises: Distributed transactions with consistency, availability, and performance. In Proc. 25th ACM Symposium on Operating Systems Principles (SOSP), Oct. 2015.

[^15]: M. Flajslik and M. Rosenblum. Network interface design for low latency request-response protocols. In Proc. USENIX Annual Technical Conference, June 2013.

[^16]: S. Gallenmüller, P. Emmerich, F. Wohlfart, D. Raumer, and G. Carle. Comparison of frameworks for high-performance packet io. In ANCS, 2015.

[^17]: G. Gibson, G. Grider, A. Jacobson, and W. Lloyd. PRObE: A Thousand-Node Experimental Cluster f
or Computer Systems Research.

[^18]: S. Han, K. Jang, K. Park, and S. Moon. Packet-Shader: a GPU-accelerated software router. In Proc. ACM SIGCOMM, Aug. 2010.

[^19]: S. Hauger, T. Wild, A. Mutter, A. Kirstaedter, K. Karras, R. Ohlendorf, F. Feller, and J. Scharf. Packet processing at 100 Gbps and beyond - challenges and perspectives. In Photonic Networks, 2009 ITG Symposium on, 2009.

[^20]: A. Kalia, M. Kaminsky, and D. G. Andersen. Using RDMA efficiently for key-value services. In Proc.ACM SIGCOMM, Aug. 2014.

[^21]: A. Kalia, D. Zhou, M. Kaminsky, and D. G. Andersen. Raising the bar for using GPUs in software packet processing. In Proc. 12th USENIX NSDI, May 2015.

[^22]:S. Larsen and B. Lee. Platform io dma transaction acceleration. In CACHES. ACM, 2011.

[^23]: H. Lim, D. Han, D. G. Andersen, and M. Kaminsky. MICA: A holistic approach to fast in-memory keyvalue storage. In Proc. 11th USENIX NSDI, Apr.


[^24]: C. Mitchell, Y. Geng, and J. Li. Using one-sided RDMA reads to build a fast, CPU-ecient key-value store. In Proc. USENIX Annual Technical Conference, June 2013.

[^25]: R. Nishtala, H. Fugal, S. Grimm, M. Kwiatkowski, H. Lee, H. C. Li, R. McElroy, M. Paleczny, D. Peek, P. Saab, D. Stafford, T. Tung, and V. Venkataramani. Scaling Memcache at Facebook. In Proc. 10th USENIX NSDI, Apr. 2013.

[^26]: L. Rizzo. netmap: a novel framework for fast packet I/O. In Proceedings of the 2012 USENIX conference on Annual Technical Conference, June 2012.

[^27]: T. Szepesi, B. Wong, B. Cassell, , and T. Brecht. Designing a low-latency cuckoo hash table for writeintensive workloads. In WSRC, 2014.

[^28]: T. Szepesi, B. Cassell, B. Wong, T. Brecht, and X. Liu. Nessie: A decoupled, client-driven, keyvalue store using RDMA. Technical Report CS-2015-09, University of Waterloo, David R. Cheriton School of Computer Science, Waterloo, Canada, June 2015.

[^29]: TPC-C. TPC benchmark C. http://www.tpc.org/tpcc/.

[^30]: X. Wei, J. Shi, Y. Chen, R. Chen, and H. Chen. Fast in-memory transaction processing using RDMA and HTM. In Proceedings of the 25th Symposium on Operating Systems Principles (SOSP), 2015.

[^31]: B. White, J. Lepreau, L. Stoller, R. Ricci, S. Guruprasad, M. Newbold, M. Hibler, C. Barb, and A. Joglekar. An integrated experimental environment for distributed systems and networks. In Proc. 5th USENIX OSDI, pages 255–270, Dec. 2002.

[^32]: D. Zhou, B. Fan, H. Lim, D. G. Andersen, and M. Kaminsky. Scalable, High Performance Ethernet Forwarding with CuckooSwitch. In Proc. 9th International Conference on emerging Networking EXperiments and Technologies (CoNEXT), Dec. 2013.








------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文翻译自<font color=blue>《Design Guidelines for High Performance RDMA Systems》。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------