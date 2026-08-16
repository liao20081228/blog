---
title:  Real-time Linux communications：an evaluation of the Linux communication stack for real-time robotic applications
tags: RDMA
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文翻译自<font color=blue>《Real-time Linux communications: an evaluation of the Linux communication stack for real-time robotic applications》。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对作者的起码的尊重！！！</font>

------


# 0  摘要
As robotics systems become more distributed, the communications between different robot modules play a key role for the reliability of the overall robot control. In this paper, we present a study of the Linux communication stack meant for real-time robotic applications. We evaluate the real-time performance of UDP based communications in Linux on multicore embedded devices as test platforms. We prove that, under an appropriate configuration, the Linux kernel greatly enhances the determinism of communications using the UDP protocol. Furthermore, we demonstrate that concurrent traffic disrupts the bounded latencies and propose a solution by separating the real-time application and the corresponding interrupt in a CPU.

随着机器人系统变得更加分散，不同机器人模块之间的通信对整个机器人控制的可靠性起着关键作用。在本文中，我们提出了一个用于实时机器人应用程序的Linux通信栈的研究。我们评估了Linux在多核嵌入式设备上的基于UDP的通信的实时性能。我们证明，在适当的配置下，Linux内核极大地增强了使用UDP协议的通信的确定性。此外，我们证明并发流量会中断有界延迟，并提出了一种通过将实时应用程序与CPU中相应的中断分开来的解决方案

# 1 引言

The Ethernet communication standard is widely used in robotics systems due to its popularity and reduced cost. When it comes to real-time communications, while historically Ethernet represented a popular contender, many manufacturers selected field buses instead. As introduced in previous work [^1], we are starting to observe a change though. With the arrival of the ‘Time Sensitive Networking’ (TSN) standards, Ethernet is expected to gain wider adoption for real-time robotic applications.  There are currently several communication technologies based on the Ethernet protocols. Some protocols such as Profinet RT [^2] or Powerlink [^3] use a network stack specifically designed for the protocol（These network stacks have been specifically designed to meet the desired real-time capabilities.）. Other protocols, such as the Data Distributed Services (DDS) [^4], OPC-UA [^5] or Profinet are built on top of the well known TCP/IP and UDP/IP OSI layers. This facilitates the process of interoperating with common transport protocols and ensures a high compatibility between devices. However, their corresponding network stacks and drivers are typically not optimized for real-time communications. The real-time performance is limited when compared to other Ethernet alternatives with specific network stacks.

以太网通信标准由于其普及和低成本而广泛用于机器人系统。在实时通信方面，虽然历史上以太网代表了一种流行的竞争者，但许多制造商却选择了现场总线。正如之前的工作[^1]所介绍的那样，我们开始观察到一个变化:随着“时间敏感网络”（TSN）标准的出现，以太网有望在实时机器人应用中得到更广泛的应用。目前有几种基于以太网协议的通信技术。某些协议（如Profinet RT [^2]或Powerlink [^3]）使用专为协议设计的网络堆栈（这些网络栈是专门为满足实时功能需要而设计的）。其他协议，例如数据分布式服务（DDS）[^4]，OPC-UA [^5]或Profinet，建立在众所周知的TCP / IP和UDP / IP OSI层之上。这有助于与公共传输协议进行互操作的过程，并确保设备之间的高兼容性。但是，它们相应的网络堆栈和驱动程序通常不会针对实时通信进行优化。与具有特定网络堆栈的其他以太网替代方案相比，实时性能有限。


注1：现场总线（Field bus）是近年来迅速发展起来的一种工业数据总线，它主要解决工业现场的智能化仪器仪表、控制器、执行机构等现场设备间的数字通信以及这些现场控制设备和高级控制系统之间的信息传递问题。由于现场总线简单、可靠、经济实用等一系列突出的优点，因而受到了许多标准团体和计算机厂商的高度重视。它是一种工业数据总线，是自动化领域中底层数据通信网络。简单说，现场总线就是以数字通信替代了传统4-20mA模拟信号及普通开关量信号的传输，是连接智能现场设备和自动化系统的全数字、双向、多站的通信系统。

In this work, we aim to measure and evaluate the real-time performance of the Linux network subsystem with the UDP and IP protocols. UDP is used by several real-time transport protocols such as the Real Time Publish Subscribe protocol (RTPS). We aim to determine which configuration provides better isolation in a mixed-critical traffic scenario. To achieve real-time performance, the network stack will be deployed in a Real-Time Operating System (RTOS). In the case of Linux, the standard kernel does not provide realtime capabilities. However, with the Real-time Preemption patch (PREEMPT-RT), it is possible to achieve real-time computing capabilities as demonstrated [^6]. Despite the Linux network subsystem not being optimized for bounded maximum latencies, with this work, we expect to achieve reasonable deterministic communications with PREEMPTRT, as well as a suitable configuration.

在这项工作中，我们的目标是使用UDP和IP协议来测量和评估Linux网络子系统的实时性能。 UDP由几种实时传输协议使用，例如实时发布订阅协议（RTPS）。我们的目标是确定哪种配置在混合关键流量方案中提供更好的隔离。为了实现实时性能，网络堆栈将部署在实时操作系统（RTOS）中。Linux的标准内核不提供实时能力。但是，使用Real-time Preemption补丁（PREEMPT-RT），可以实现实时计算功能[^6]。尽管Linux网络子系统没有针对有限的最大延迟进行优化，但通过这项工作，我们希望通过PREEMPTRT实现合理的确定性通信，以及合适的配置。

The content below is structured as follows: section II introduces related previous work. In particular, we review the real-time performance of Linux with PREEMPT-RT and the real-time communications in Linux for robotics. Section III outlines an overview of the key components of the Linux kernel we have used to configure our system to optimize the real-time communications over Ethernet. Section IV discusses the experimental results obtained while evaluating the presented hypotheses. Finally, Section V presents our conclusions and future work.


文章结构如下：第二节介绍了以前的相关工作。特别是，我们回顾了使用PREEMPT-RT的Linux的实时性能以及Linux中用于机器人的实时通信。第三部分概述了我们用于配置系统以优化以太网实时通信的Linux内核的关键组件。第四节讨论了在评估所提出的假设时获得的实验结果。最后，第五节介绍了我们的结论和未来的工作。

# 2 相关工作
In [^7], Abeni et al. compared the networking performance between a vanilla kernel 3.4.41 and the same version with PREEMPT-RT. They investigated the throughput and latency with different RT patch implementations. They also explained some of the key ideas of the Linux Network architecture in PREEMPT-RT and showed an evaluation of the network latency by measuring the round-trip latency with the UDP protocol. The results showed that the differences between the kernel version tested appeared when the system was loaded. For the kernel versions where the soft Interrupt Requests (IRQ) were served in the IRQ threads the results showed that latency was bounded. However, the influence of concurrent traffic was not discussed in the paper. Khoronzhuk and Valris [^8] showed an analysis of the deterministic performance of the network stack with the aim of using it with TSN. They used a real-time kernel 4.9 with a 1 Gbps Ethernet network interface. They concluded that there is jitter in the order of hundreds of microseconds across the networking stack layers, both in transmission and reception paths.

在[^7]中，Abeni等人。比较了vanilla内核3.4.41与PREEMPT-RT相同版本之间的网络性能。他们使用不同的RT补丁调查了吞吐量和延迟。他们还解释了PREEMPT-RT中Linux网络架构的一些关键思想，并通过使用UDP协议测量往返延迟来评估网络延迟。结果表明，在加载系统时，测试的内核版本之间存在差异。对于在IRQ线程中提供软中断请求（IRQ）的内核版本，结果显示延迟是有限的。但是，本文没有讨论并发流量的影响。 Khoronzhuk和Valris [^8]展示了对网络堆栈的确定性性能的分析，目的是将其与TSN一起使用。他们使用具有1 Gbps以太网网络接口的实时内核4.9。他们得出结论，在传输和接收路径中，网络堆栈层中存在数百微秒的抖动。

For an appropriate setup of our infrastructure, we made active use of the work presented at [^9] and [^10], which provide some guidelines to configure a PREEMPT-RT based OS for real-time networking.

为了适当地设置我们的基础设施，我们积极利用了[^9]和[^10]中提出的工作，这些工作为配置基于PREEMPT-RT的OS进行实时网络提供了一些指导。

# 3 在LInux上设置实时通信
## 3.1  实施抢占补丁

There are currently different approaches to use Linux for real-time applications. A common path is to leave the most critical tasks to an embedded RTOS and give to Linux the highest level commands. A second approach is to use a dual-kernel scheme like Xenomai [^11] and RTAI [^12] which deploy a microkernel running in parallel with a separate Linux kernel. The problem for this kind of solution is that it requires special tools and libraries. 

目前有不同的方法将Linux用于实时应用程序。一个常见的途径是将最关键的任务留给嵌入式RTOS，并为Linux提供最高级别的命令。第二种方法是使用像Xenomai [^11]和RTAI [^12]这样的双内核方案，它们部署与单独的Linux内核并行运行的微内核。这种解决方案的问题在于它需要特殊的工具和库。

A third approach is to use a single-kernel. The Real-Time Linux (RTL) Collaborative Project [^13] is the most relevant open-source solution for this option. The project is based on the PREEMPT-RT patch and aims to create a predictable and deterministic environment turning the Linux kernel into a viable real-time platform. The ultimate goal of the RTL project is to mainline the PREEMPT-RT patch. The importance behind this effort is not related to the creation of a Linux-based RTOS, but to provide the Linux kernel with real-time capabilities. The main benefit is that it is possible to use the Linux standard tools and libraries without the need of specific real-time APIs. Also, Linux is widely used and strongly supported, this helps to keep the OS updated with new technologies and features, something which is often a problem in smaller projects due to resource limitations.


第三种方法是使用单内核。 Real-Time Linux（RTL）协作项目[^13]是此选项最相关的开源解决方案。该项目基于PREEMPT-RT补丁，旨在创建一个可预测的确定性环境，将Linux内核转变为可行的实时平台。 RTL项目的最终目标是PREEMPT-RT补丁。这项工作背后的重要性与创建基于Linux的RTOS无关，而是为Linux内核提供实时功能。主要好处是可以使用Linux标准工具和库，而无需特定的实时API。此外，Linux得到了广泛使用并得到了强有力的支持，这有助于使用新技术和新功能更新操作系统，这在较小的项目中通常会因资源限制而成为问题。


## 3.2  Linux网络架构

While it is possible to bypass the Linux Network Stack using custom drivers or user-space network libraries, we are interested in using the Linux Network Stack; mainly, because it is easier to maintain and integrate with a user-space application or communication middlewares. In addition, the Linux Network Stack supports a wide range of drivers which allow to deploy an application in different devices.

虽然可以使用自定义驱动程序或用户空间网络库绕过Linux网络堆栈，但我们对使用Linux网络堆栈感兴趣; 主要是因为它更容易维护和集成用户空间应用程序或通信中间件。 此外，Linux网络堆栈支持各种驱动程序，允许在不同设备中部署应用程序。

### 3.2.1 Linux流量控制

Linux Traffic Control: An important module of the networking subsystem is the Linux kernel packet scheduler, which is configured with the user-space tool Linux Traffic Control (TC) [^14]. TC provides mechanisms to control the way enqueued packets are sent and received, it provides a set of functionality such as shaping, scheduling, policing and dropping network traffic.

网络子系统的一个重要模块是Linux内核数据包调度程序，它使用用户空间工具Linux Traffic Control（TC）进行配置[^14]。 TC提供了控制排队数据包发送和接收方式的机制，它提供了一组功能，如整形，调度，监管和丢弃网络流量。

The main element of the Linux packet scheduler module are the queuing disciplines (Qdisc), which are network traffic disciplines to create queues and quality of service (QoS) rules for reception and transmission. There are ingress and egress Qdisc for reception and transmission respectively. The egress Qdisc provides shaping, scheduling and filter capabilities for data transmission from the network protocol layers to the network interface ring buffers. On the other hand, the ingress Qdisc provides filter and dropping capabilities for the reception path from the network interface ring buffers to the network protocol layers (although these are commonly less used).

Linux数据包调度程序模块的主要元素是排队规则（Qdisc），它是用于创建队列的网络流量规则和用于接收和传输的服务质量（QoS）规则。 有入口Qdisc和出口Qdisc分别用于接收和传输。 出口Qdisc为从网络协议层到网络接口环形缓冲区的数据传输提供整形，调度和过滤能力。 另一方面，入口Qdisc为从网络接口环形缓冲区到网络协议层的接收路径提供过滤和丢弃能力（尽管这些通常较少使用）。

For the egress Qdisc there are two basic types of disciplines: classless Qdisc and classful Qdisc. The classless Qdisc does not contain another Qdisc so there is only one level of queuing. The classless Qdisc only determines whether the packet is classified, delayed or dropped. The classful Qdisc can contain another Qdisc, so there could be several levels of queues. In such case, there may be different filters to determine from which Qdisc packets will be transmitted.

对于出口Qdisc，有两种基本类型的规则：无类Qdisc和有类Qdisc。 无类Qdisc不包含另一个Qdisc，因此只有一个级别的队列。 无类Qdisc仅确定数据包是分类，延迟还是丢弃。 有类Qdisc可以包含另一个Qdisc，因此可能有几个级别的队列。 在这种情况下，可能存在不同的过滤器以确定将从Qdisc分组从哪一个过滤器进行传输。

Qdisc can be used to avoid traffic congestion with non real-time traffic at the transmission path (figure 1). For a classless Qdisc, the default discipline is the PFIFO_FAST, which has three FIFO priority bands. In the case of a classful Qdisc there is the PRIO qdisc which can contain an arbitrary number of classes of differing priority. There are also specific egress Qdisc destined for multiqueue network devices, for example the MQPRIO Qdisc [^15], which is a queuing discipline to map traffic flows to hardware queues based on the priority of the packet. This Qdisc will dequeue packets with higher priority allowing to avoid contention problems in the transmission path. In addition to the priority Qdisc, it is common to attach a shaper to limit low priority traffic bandwidth such as a the ‘Token Bucket Filter’ TBF Qdisc [^16].

Qdisc可用于避免传输路径上非实时流量的流量拥塞（图1）。 对于无类Qdisc，默认规则是PFIFO_FAST，它有三个FIFO优先级。 有类Qdisc默认使用PRIO Qdisc，它可以包含任意数量的不同优先级。 还存在用于多队列网络设备的特定出口Qdisc，例如MQPRIO Qdisc [^15]，它是基于分组的优先级将流量流映射到硬件队列的排队规则。 该Qdisc将使具有更高优先级的分组出列，从而避免传输路径中的争用问题。 除了优先级Qdisc之外，通常还要附加一个整形器以限制低优先级流量带宽，例如“令牌桶过滤器”TBF Qdisc [^16]。

![图1](https://www.github.com/liao20081228/blog/raw/master/图片/实时Linux通信：评估用于实时机器人应用程序的Linux通信堆栈/图1.jpg)

Recently, because of the interest in support TSN in the Linux network stack, new Qdiscs have been created or are currently under development. The IEEE 802.1Q-2014 Credit Based Shaper (CBS) [^17], Qdisc has already been included from kernel 4.15. The CBS is used to enforce a Quality of Service by limiting the data rate of a traffic class. Currently there are two Qdisc under development, the ‘Earliest Transmit Time First (ETF)’ [^18] which provides a per-queue transmit time based scheduling and the ‘Time-Aware Priority Scheduler’ (TAPRIO) which provides per-port scheduling. These Qdisc will allow to create deterministic scheduling in software or to offload the work to the network hardware if it is supported.

最近，由于对Linux网络堆栈中支持TSN的兴趣，已经创建或正在开发新的Qdisc。 IEEE 802.1Q-2014基于信用的整形器（CBS）[^17]，Qdisc已经包含在内核4.15中。 CBS用于通过限制流量类的数据速率来实施服务质量。 目前正在开发两个Qdisc，即提供基于每队列传输时间的调度的“最早传输时间优先（ETF）”[^18]和提供每端口调度的“时间感知优先级调度器”（TAPRIO）。 这些Qdisc将允许在软件中创建确定性调度，或者如果支持，则将工作卸载到网络硬件。

### 3.2.2 流量分类
 In order to steer a traffic flow to a Qdisc or to a ring buffer, the traffic flow must be classified usually by marking the traffic with a priority. There are several ways to set the priority of a specific traffic flow: 
a) from the user-space using socket options SO_PRIORITY and IP_TOS, 
b) with iptables and 
c) with net_prio cgroups.
Setting the priority of a flow maps the traffic from a socket (or an application) to a Socket Buffer (SKB) priority, which is an internal priority for the kernel networking layer. The SKB priority is used by the MQPRIO Qdisc to map the traffic flow to a traffic class of the Qdisc. At the same time, each traffic class is mapped to a TX ring buffer.

为了将流量引导到Qdisc或环形缓冲区，通常必须通过优先级标记流量来对流量进行分类。 有几种方法可以设置特定流量的优先级：
a）从用户空间使用套接字选项SO_PRIORITY和IP_TOS，
b）使用iptables
c）使用net_prio cgroups。
设置流的优先级将流量从套接字（或应用程序）映射到套接字缓冲区（SKB）优先级，这是内核网络层的内部优先级。 MQPRIO Qdisc使用SKB优先级将流量映射到Qdisc的流量类。 同时，每个流量类都映射到TX环缓冲区。


### 3.2.3 网络硬摘中断线程和软中断

At the reception path, the processing of the packets is driven by the kernel interrupt handling mechanism and the “New API” (NAPI) network drivers. NAPI is a mechanism designed to improve the performance for high network loads. When there is a considerable incoming traffic, a high number of interrupts will be generated. Handling each interrupt to process the packets is not very efficient when there are many packets already queued. For this reason NAPI uses interrupt mitigation when high bandwidth incoming packets are detected. Then, the kernel switches to a polling-based processing, checking periodically if there are queued packets. When there is not as much load, the interrupts are re-enabled again. In summary, the Linux kernel uses the interrupt-driven mode by default and only switches to polling mode when the flow of incoming packets exceeds a certain threshold, known as the “weight” of the network interface. This approach works very well as a compromise between latency and throughput, adapting its behavior to the network load status. The problem is that NAPI may introduce additional latency, for example when there is bursty traffic.

在接收路径上，分组的处理由内核中断处理机制和“NEW API”（NAPI）网络驱动程序驱动。 NAPI是一种旨在提高高网络负载性能的机制。当存在相当大的传入流量时，将产生大量中断。当有许多数据包已经排队时，处理每个中断以处理数据包的效率不高。因此，当检测到高带宽传入数据包时，NAPI会使用中断缓解。然后，内核切换到基于轮询的处理，定期检查是否存在排队的数据包。当没有那么多负载时，将再次重新启用中断。总之，Linux内核默认使用中断驱动模式，只有当传入数据包流量超过某个阈值（称为网络接口的“权重”）时才切换到轮询模式。这种方法可以很好地作为延迟和吞吐量之间的折衷，使其行为适应网络负载状态。问题是NAPI可能会引入额外的延迟，例如当有突发流量时。

There are some differences between how PREEMPT-RT and a normal kernel handle interrupts and consequently how packets are handled at the reception path. The modifications of PREEMPT-RT allow to configure the system to improve the networking stack determinism. 


PREEMPT-RT和普通内核处理中断的方式和在接收路径上如何处理数据包 有一些差异。 PREEMPT-RT的修改允许配置系统以改善网络堆栈确定性。

In PREEMPT-RT, most interrupt request (IRQ) handlers are forced to run in threads specifically created for that interrupt. These threads are called IRQ threads [^19]. Handling IRQs as kernel threads allows priority and CPU affinity to be managed individually. IRQ handlers running in threads can themselves be interrupted, so that the latency due to interrupts are mitigated. For a multiqueue NIC, there is an IRQ for each TX and RX queue of the network interface, allowing to prioritize the processing of each queue individually. For example, it is possible to use a queue for real-time traffic and raise the priority of that queue above the other queue IRQ threads.

在PREEMPT-RT中，大多数中断请求（IRQ）处理程序被强制在专门为该中断创建的线程中运行。 这些线程称为IRQ线程[^19]。 将IRQ作为内核线程处理可以单独管理优先级和CPU亲和性。 在线程中运行的IRQ处理程序本身可以被中断，从而减少由于中断引起的延迟。 对于多队列NIC，网络接口的每个TX和RX队列都有一个IRQ，允许单独处理每个队列的优先级。 例如，可以将队列用于实时流量，并将该队列的优先级提高到其他队列IRQ线程之上。

Another important difference is the context where the softirq are executed. From version 3.6.1-rt1, the soft IRQ handlers are executed in the context of the thread that raised that Soft IRQ [^20]. This means that the NET_RX soft IRQ will be normally executed in the context of the network device IRQ thread, which allows a fine control of the networking processing context. However, if the network IRQ thread is preempted or it exhausts its NAPI weight time slice, it is executed in the ksoftirqd/n (where n is the logical number of the CPU). 

另一个重要的区别是执行softirq的上下文。从版本3.6.1-rt1开始，软IRQ处理程序在引发Soft IRQ [^20]的线程的上下文中执行。这意味着NET_RX软IRQ将在网络设备IRQ线程的上下文中正常执行，这允许精确控制网络处理上下文。 但是，如果网络IRQ线程被抢占或耗尽其NAPI权重时间片，则在ksoftirqd / n中执行（其中n是CPU的逻辑号）。

Processing packets in the ksoftirqd/n context is troublesome for real-time because this thread is used by different processes for deferred work and can add latency. Also, as the ksoftirqd runs with SCHED_OTHER policy, the thread execution can be easily preempted. In practice, the soft IRQs are normally executed in the context of NIC IRQ threads and in the ksoftirqd/n thread for high network loads and under heavy stress (CPU, memory, I/O, etc..).

在ksoftirqd / n上下文中处理数据包对于实时来说是麻烦的，因为这个线程被不同的进程用于延迟工作并且可以增加延迟。 此外，由于ksoftirqd使用SCHED_OTHER策略运行，因此可以轻松地抢占线程执行。 实际上，软IRQ通常在NIC IRQ线程和ksoftirqd / n线程的上下文中执行，用于高网络负载和高压力（CPU，内存，I / O等）。

### 3.2.4 套接字分配

One of the current limitations of the network stack for bounded latency is socket memory allocation. Every packet in the network stack needs a sckbuff struct which holds meta-data of the packet. This struct needs to be allocated for each packet, and the time required for the allocation represents a large part of the overhead for processing the packet and jitter source.

网络堆栈的有限延迟的当前限制之一是套接字内存分配。 网络堆栈中的每个数据包都需要一个sckbuff结构，它保存数据包的元数据。 需要为每个数据包分配此结构，并且分配所需的时间代表处理数据包和抖动源的大部分开销。


One of the last projects of the Linux network developers is the XDP or express Data Path [^21] which aims to provide
a high performance, programmable network data path in the Linux kernel. XDP will provide faster packet processing by eliminating the socket meta-data allocation. Despite realtime communications are not the main motivation behind
this project, XDP looks like an interesting feature to be used as an express data path for real-time communications[^22].

Linux网络开发人员的最后一个项目是XDP（express数据路径）[^21]，它旨在在Linux内核中提供高性能，可编程的网络数据路径。 通过消除套接字元数据分配，XDP将提供更快的数据包处理。 尽管实时通信并不是该项目背后的主要动机，但XDP看起来像是一个有趣的功能，可用作实时通信的快速数据路径[^ 22]。

# 4 实验设置和结果

To evaluate the real-time performance of the network stack,we used two embedded devices, measuring the latencies of
a round-trip test.

为了评估网络堆栈的实时性能，我们使用了两个嵌入式设备，测量往返测试的延迟。

## 4.1 往返回路测试

The network latency is measured as the round-trip time (RTT), also called ping-pong test. For the test we use a client in one of the devices and a server in the other. The roundtrip latency is measured as the time it takes for a message to travel from the client to the server, and from the server back to the client.

网络延迟测量为往返时间（RTT），也称为乒乓测试。 对于测试，我们在其中一个设备中使用客户端，在另一个设备中使用服务器。 往返延迟测量为消息从客户端传输到服务器以及从服务器返回到客户端所花费的时间。

![图2](https://www.github.com/liao20081228/blog/raw/master/图片/实时Linux通信：评估用于实时机器人应用程序的Linux通信堆栈/图2.jpg)

For the client and server one we used a modified version of cyclictest [^23] which allows us to save the statistics and create latency histograms which show the amount of jitter and worst-case latency of the test. Additionally, we count the number of missed deadlines for a 1 millisecond target loop time.

对于客户端和服务器，我们使用了cyclictest [^23]的修改版本，它允许我们保存统计信息并创建延迟直方图，这用来显示测试的抖动量和最坏情况延迟。 此外，我们计算1毫秒目标循环时间的错过最后期限的数量。


For the timing loop we used the clock_nanosleep primitive. We also used memory locking, the FIFO scheduler and set a real-time priority of 80. In all the tests, we marked the traffic as priority traffic using the socket option SO_PRIORITY. To generate load in the system, we used the program stress and, to generate traffic, the program iperf.

对于时序循环，我们使用了clock_nanosleep原语。 我们还使用了内存锁定，FIFO调度程序并设置了80的实时优先级。在所有测试中，我们使用套接字选项SO_PRIORITY将流量标记为优先级流量。 为了在系统中生成负载，我们使用程序stree，为了生成流量，使用了程序iperf。

### 4.1.1 实验设置

The main characteristics of the embedded device are:
Processor: ARMv7 Processor (2 cores).
Kernel version: 4.9.30.
PREEMPT-RT patch (when used): rt21.
Link capacity: 100/1000 Mbps, Full-Duplex.

嵌入式设备的参数主要如下：
处理器：ARMv7 Processor (双核).
内核版本：4.9.30
实时抢占补丁：rt21
链路容量：00/1000 Mbps,全双功.

Each device has a switched endpoint as network interface. The devices are connected in linear topology with a PC. The PC is simply used to generate contending traffic, also referred in this work as background traffic.

每个设备都有一个交换端点作为网络接口。 这些器件采用线性拓扑结构与PC连接。 PC仅用于生成竞争流量，在本工作中也称为后台流量。


The traffic used to measure the end-to-end latency is marked with higher priority than the background traffic. The embedded device network interface has 3 queues (tx_q0, tx_q1, tx_q3). In our case, the network drivers use the Priority Code Point (PCP) of the Ethernet header to map a frame with a TX queue. Priority 4 is mapped to tx_q0, priorities 2 and 3 to tx_q1 and the rest of priorities to tx_q2. We configure a MQPRIO qdisc to map the SKB priorities with the same classification criteria than the one used by the network device drivers. The Qdisc queues are mapped one to one with each hardware queue, so we can avoid traffic contention at the network layer in the transmission path.

用于衡量端到端延迟的流量标记的优先级高于后台流量。 嵌入式设备网络接口有3个队列（tx_q0，tx_q1，tx_q3）。 在我们的示例中，网络驱动程序使用以太网报头的优先级代码点（PCP）来映射具有TX队列的帧。 优先级4映射到tx_q0，优先级2和3映射到tx_q1，其余优先级映射到tx_q2。 我们配置MQPRIO qdisc以使用与网络设备驱动程序使用的分类标准相同的分类标准来映射SKB优先级。 Qdisc队列与每个硬件队列一对一映射，因此我们可以避免传输路径中网络层的流量争用。

---

tc qdisc replace dev eth1 root mqprio num_tc 3 \
map 2 2 1 1 0 2 2 2 2 2 2 2 2 2 2 2
queues 1@0 1@1 1@2 hw 0

---

This command maps the SKB priority 4 with queue 0, SKB priorities 2 and 3 with queue 1 and the rest of priorities with queue 2. Checking the classes setting, we get the following output:

此命令将SKB优先级4映射到队列0，将SKB优先级2和3映射到队列1，将其余优先级映射到队列2.检查类设置，我们得到以下输出：

---

root@DUT1:~# tc -g qdisc show dev eth1
qdisc mqprio 8001: root tc 3 map 2 2 1 1 0 2 2 2 2 2 2 2 2 2 2 2
queues:(0:0) (1:1) (2:2)
qdisc pfifo_fast 0: parent 8001:3 bands 3 priomap 1 2 2 2 1 2 0 0 1 1 1 1 1 1 1 1
qdisc pfifo_fast 0: parent 8001:2 bands 3 priomap 1 2 2 2 1 2 0 0 1 1 1 1 1 1 1 1
qdisc pfifo_fast 0: parent 8001:1 bands 3 priomap 1 2 2 2 1 2 0 0 1 1 1 1 1 1 1 1

---

Apart from the networking layer mapping, we need to map the SKB priorities with the priority used by the Ethernet layer, which is the PCP. We set a one to one configuration. This sets the priority in VLAN field of the Ethernet header according to the SKB priority configured in the user-space. The PCP is used by the Network interface Card (NIC) to map traffic to the hardware queues.


除了网络层映射之外，我们还需要将SKB优先级映射到以太网层使用的优先级，即PCP。 我们设置了一对一的配置。 这根据用户空间中配置的SKB优先级设置以太网报头的VLAN字段中的优先级。 网络接口卡（NIC）使用PCP将流量映射到硬件队列。


---

root@DUT1:~# ip link set eth1.2 type vlan egress 0:0 1:1 2:2 3:3 4:4 5:5 6:6 7:7

---













## 4.2 任务、中断亲和性和CPU屏蔽
In real-time systems, real-time tasks and interrupts can be pinned to a specific CPU to separate their resources from non real-time tasks. This is an effective way to prevent non real-time processes interferences. 

在实时系统中，可以将实时任务和中断固定到特定CPU，以将其资源与非实时任务分开。 这是防止非实时过程干扰的有效方法。

There are several ways to set the affinity of tasks and IRQs with CPUs. For the experiments, we decided to compare two levels of isolation. In the first case, we pin the real-time task the IRQ of the real-time traffic queue to the same CPU. We use “pthread_setaffinity_np” and “smp irq affinity” to set the priorities of the IRQs.


有几种方法可以设置任务和IRQ与CPU的亲和力。 对于实验，我们决定比较两个级别的隔离。 在第一种情况下，我们将实时任务和实时流量队列的IRQ固定到同一CPU。 我们使用“pthread_setaffinity_np”和“smp irq affinity”来设置IRQ的优先级。

In the second case, we use cpusets [^24], which is part of the Linux cgroups to assign a CPU for real-time tasks. With this method, we can also migrate all processes running in the isolated CPU, so that only the real-time task is allowed to run in that CPU（Only some kernel processes will run in the isolated CPU which will
reduce the system jitter considerably）. We also set the affinity of all the IRQs to the non real-time CPU, while the IRQs of the real-time queue (of the network device) are set with affinity in the isolated CPU.

在第二种情况下，我们使用cpusets（它是linux cgoups的一部分）来为史诗任务分配一个cpu[^24]。 使用此方法，我们还可以迁移在隔离CPU中运行的所有进程，以便只允许在该CPU中运行实时任务（只有一些内核进程将在隔离的CPU中运行，这将大大减少系统抖动。）。 我们还将所有IRQ的亲和性设置为非实时CPU，而实时队列（网络设备）的IRQ的亲和性设置在隔离的CPU中。

In the experimental setup, we use the described methods to isolate applications sending and receiving real-time traffic. The tests are run using different configurations: no-rt, rt-normal, rt-affinities and rt-isolation. In the first case, no-rt, we use a vanilla kernel. In the second case, rt-normal, we use a PREEMPT-RT kernel without binding the round-trip programs and network IRQs to any CPU. In the third case, rt-affinities, we bind the IRQ thread of the priority queue and the client and server programs to CPU 1 of each device. Finally, in the fourth case, rt-isolation, we run the roundtrip application in an isolated CPU. In all the cases, we set the priority of the RTT test client and server to a 80 value.

在实验设置中，我们使用所描述的方法来隔离发送和接收实时流量的应用程序。 测试使用不同的配置运行：no-rt，rt-normal，rt-affinities和rt-isolation。 在第一种情况下，no-rt，我们使用香草内核。 在第二种情况下，rt-normal，我们使用PREEMPT-RT内核，而不将往返程序和网络IRQ绑定到任何CPU。 在第三种情况下，rt-affinities，我们将优先级队列的IRQ线程以及客户端和服务器程序绑定到每个设备的CPU 1。 最后，在第四种情况下，rt-isolation，我们在一个隔离的CPU中运行往返应用程序。 在所有情况下，我们将RTT测试客户端和服务器的优先级设置为80值。

In order to have an intuition about the determinism of each configuration, we ran a 1 hour cyclictest, obtaining the following worst-case latencies: no-rt: 13197 s, rtnormal/ rt-affinities: 110 s and rt-isolation: 88 s.


为了对每种配置的确定性有直觉，我们进行了1小时的循环测试，获得了以下最坏情况的延迟：no-rt：13197 s，rtnormal / rt-affinities：110 s和rt-isolation：88 s。

### 4.2.1 系统和网络负载

For each case, we run thetests in different load conditions: idle, stress, tx-traffic and rx-traffic:
* idle: No other user-space program running except the client and server.
* stress: We generate some load to stress the CPU and memory and block memory.3
* tx-traffic: We generate some concurrent traffic in the transmission path of the client. We send 100 Mbps traffic from the client to the PC.
* rx-traffic: We generate some concurrent traffic in the reception path of the server. We send 100 Mbps traffic from the PC to the server.


对于每种情况，我们在不同的负载条件下运行测试：空闲，压力，tx流量和rx流量：

* 空闲：除了客户端和服务端，没有其他用户程序正在运行
* 压力：我们产生一些负载来stress CPU和block 内存。（Command used: stress -c 2 -i 2 -m 2 –vm-bytes 128M -d 2 –hdd-bytes 15M）
* tx流量： 我们在客户端的传输路径中生成一些并发流量。 我们从客户端向PC发送100 Mbps流量。
* rx流量：我们在服务器的接收路径中生成一些并发流量。 我们从PC向服务器发送100 Mbps流量。

When generating concurrent traffic, there is also congestion in the MAC queues of both devices. However, as the  traffic of the test is prioritized, the delay added by the link layer is not meaningful for the test.

在生成并发流量时，两个设备的MAC队列中也存在拥塞。 但是，由于测试的流量优先，链路层添加的延迟对测试没有意义。

## 4.3 结果

We have compared the results obtained for a round-trip test of 3 hours of duration, sending UDP packets of 500 Bytes at a 1 millisecond rate. The Tables I, II, III, IV show the statistics of the different configuration used under different conditions. For real-time benchmarks, the most important metrics are the worst-cases (Max), packet loss and the number of missed deadlines. In this case, we decided to set a 1 millisecond deadline to match it with the sending rate.

我们比较了3小时持续时间的往返测试结果，以1毫秒的速率发送500字节的UDP数据包。 表I，II，III，IV显示了在不同条件下使用的不同配置的统计数据。 对于实时基准测试，最重要的指标是最坏情况（最大），数据包丢失和错过最后期限的数量。 在这种情况下，我们决定设置1毫秒的截止时间，使其与发送速率相匹配。

As it can be seen in Table I, the non real-time kernel results in what seems to be the best average performance, but in contrast, it has a high number of missed deadlines and a high maximum latency value; even when the system is idle. The latency suffers specially when the system is stressed due to the lack of preemption in the kernel.

从表I中可以看出，非实时内核导致了最佳的平均性能，但相比之下，它具有大量错过的期限和高的最大延迟值; 即使系统处于空闲状态。 由于内核中没有抢占，系统受到压力时，延迟会受到影响。

For rt-normal (Table II), the latencies are bounded when the system is stressed. When generating concurrent traffic, we observe higher latency values and some missed deadliness

对于rt-normal（表II），当系统受到压力时，延迟是有界的。 在生成并发流量时，我们会观察到更高的延迟值和一些错过的截止时间。

For rt-affinities, we can see an improvement compared to the previous scenario. Specially for concurrent traffic (Table III). We can also see that when pinning the round-trip threads and the Ethernet IRQs for the priority to the same CPU, the latency seems to be bounded.

对于rt亲和力，我们可以看到与之前的情景相比有所改善。 特别适用于并发流量（表III）。 我们还可以看到，当将往返线程和以太网IRQ固定为同一CPU的优先级时，延迟似乎是有限的。

In the case of no-isolation (table IV), we appreciate a similar behavior when compared to the affinity case. We can see that stressing the non isolated CPU has some impact on the tasks of the isolated core. However, in the idle case, for short test we observed very low jitter. To the best of our knowledge, one of the main contributions of such latency was the scheduler ticker, which is generated each 10 milliseconds（This effect was observed using kernel tracer ‘ftrace’ and correlating the
latency measurements with the traces during the test.）. While it is possible to avoid it in the client because it runs in a timed loop, in the server side, it is not possible to avoid the ticker. As both devices are not synchronized, at some point, the clock of the server side drifts from the client and the scheduler ticker interferes in the server execution. This effect can be seen in Figure 4.

在不隔离的情况下（表IV），我们认为与亲和力案例相类似行为。 我们可以看到，压力非隔离CPU对隔离内核的任务有一些影响。 但是，在空闲情况下，对于短测试，我们观察到非常低的抖动。 据我们所知，这种延迟的主要贡献之一是调度程序自动收报机，它每10毫秒生成一次 （使用内核示踪剂'ftrace'观察到这种效应，并将延迟测量值与测试期间的迹线相关联。）。 虽然可以在客户端中避免它，因为它在定时循环中运行，但在服务器端，不可能避免使用自动收报机。 由于两个设备都不同步，在某些时候，服务器端的时钟从客户端漂移，并且调度程序自动收报机干扰服务器执行。 这种效果可以在图4中看到。

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;**表1 往返延迟的结果：No RT, Kernel version: 4.9.30**

||Min(us) |Avg(us) |Max(us) |Missed deadline |Packet loss|
|--|--|--|--|--|--|
|Idle |193| 217 |1446 |15 / 600000| 0 / 600000|
|Stress |262 |560 |46742| 20979 / 600000| 0 / 600000
|TX traffic at 100 Mbps| 195 |378 |7160 |2298 / 600000 |0 / 600000
|RX traffic at 100 Mbps| 192| 217 |1426| 22 / 600000| 0 / 600000

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;**表2 往返延迟的结果： RT Normal, Kernel version: 4.9.30-rt21**

||Min(us) |Avg(us) |Max(us) |Missed deadline |Packet loss|
|--|--|--|--|--|--|
|Idle |251| 266 |522 |0 / 600000| 0 / 600000|
|Stress |254| 341| 618| 0 / 600000 |0 / 600000|
|TX traffic at 100 Mbps| 265 |320| 25727 |20 / 600000| 0 / 600000|
|RX traffic at 100 Mbps |263 |292| 898| 9 / 600000| 0 / 600000|


&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;**表3 往返延迟的结果： RT , CPU Affinity, Kernel version: 4.9.30-rt21**

||Min(us) |Avg(us) |Max(us) |Missed deadline |Packet loss|
|--|--|--|--|--|--|
|Idle |275 |289| 644 |0 / 600000| 0 / 600000|
|Stress |277 |358| 828 |0 / 600000 |0 / 600000|
|TX traffic at 100 Mbps |277 |322| 568| 0 / 600000 |0 / 600000|
|RX traffic at 100 Mbps |274 |287| 592| 0 / 600000 |0 / 600000|

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;**表4 往返延迟的结果： RT , CPU Isolated, Kernel version: 4.9.30-rt21**

||Min(us) |Avg(us) |Max(us) |Missed deadline |Packet loss|
|--|--|--|--|--|--|
|Idle |297 |311 |592 |0 / 600000| 0 / 600000|
|Stress| 298 |355 |766 |0 / 600000 |0 / 600000|
|TX traffic at 100 Mbps |301 336 617 0 / 600000 0 / 600000
|RX traffic at 100 Mbps| 296 312 542 0 / 600000 0 / 600000

When running the test with 200 Mbps RX-traffic, we observed that best-effort traffic is processed in the ksoftirqd/0 context continuously. This generates high latency spikes in all the cases, even for the isolation case. To trace the source of these latency spikes, we should trace the kernel taking a snapshot when the latency occurs.

当使用200 Mbps RX流量运行测试时，我们观察到在尽力而为的流量k是在softirqd / 0上下文中连续处理的。 这会在所有情况下产生高延迟峰值，即使对于隔离情况也是如此。 为了跟踪这些延迟峰值的来源，我们应该在发生延迟时跟踪内核拍摄快照。


# 5 结论和未来的工作

The results obtained prove that the Linux real-time setup presented improves greatly the determinism of communications using the UDP protocol. First, we confirm that the communication delay caused when the system is under heavy load is mitigated by making use of a real-time kernel and by running the application with real-time priority.

结果证明，所提出的Linux实时设置极大地改善了使用UDP协议的通信的确定性。首先，我们确认通过使用实时内核和运行具有实时优先级的应用程序来减轻系统处于高负载时引起的通信延迟。


Second, we demonstrate that, whenever there is concurrent traffic, simply setting the priorities of the real-time process is not enough. Separating the real-time application and the corresponding interrupt in a CPU seems to be an effective approach to avoid high latencies. For higher concurrent traffic loads, however, we can still see unbounded latency

其次，我们证明，只要存在并发流量，仅仅设置实时流程的优先级是不够的。在CPU中分离实时应用程序和相应的中断似乎是避免高延迟的有效方法。但是，对于更高的并发流量负载，我们仍然可以看到无限延迟

and further research is required to overcome this limitation with our current setup.We conclude that, under certain circumstances and for a variety of stress and traffic overload situations, Linux can indeed meet some real-time constraints for communications. Hereby, we present an evaluation of the Linux communication stack meant for real-time robotic applications. Future work should take into account that the network stack has not been fully optimized for low and bounded latency; there is certainly room for improvement. It seems to us that there is some ongoing work inside the Linux network stack, such as the XDP [^21] project, showing promise for an improved real-time performance. In future work, it might be interesting to test some of these features
and compare the results.

我们当前的设置需要进一步的研究来克服这个限制。我们得出结论，在某些情况下，对于各种压力和流量过载情况，Linux确实可以满足一些实时的通信限制。在此，我们提出了一个针对实时机器人应用程序的Linux通信堆栈的评估。未来的工作应该考虑到网络堆栈尚未针对低延迟和有限延迟进行全面优化;肯定有改进的余地。在我们看来，Linux网络堆栈中正在进行一些正在进行的工作，例如XDP [^21]项目，它显示了提高实时性能的希望。在将来的工作中，测试其中一些功能并比较结果可能会很有趣。


------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文翻译自<font color=blue>《Real-time Linux communications: an evaluation of the Linux
communication stack for real-time robotic applications》。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对作者的起码的尊重！！！</font>

------



[^1]: C.S.V.Gutiérrez, L.U.S.Juan, I. Z. Ugarte, and V. M. Vilches,“Time-sensitive networking for robotics,” CoRR, vol. abs/1804.07643,2018. [Online]. Available: http://arxiv.org/abs/1804.07643
[^2]: “PROFINET the leading industrial ethernet standard,” https://www.profibus.com/technology/profinet/, accessed: 2018-04-12.
[^3]: “POWERLINK - powerlink standarization group,” https://www.ethernet-powerlink.org/powerlink/technology, accessed: 2018-04-12.
[^4]: “Data Distribution Service Specification, version 1.4,” https://www.omg.org/spec/DDS/1.4/, accessed: 2018-04-12.
[^5]: “OPC-UA - the opc unified architecture (ua),” https://opcfoundation.org/about/opc-technologies/opc-ua/, accessed: 2018-04-12.
[^6]: H. Fayyad-Kazan, L. Perneel, and M. Timmerman, “Linuxpreempt-rtvs. commercial rtoss: how big is the performance gap?” GSTFJournal on Computing (JoC), vol. 3, no. 1, 2018. [Online]. Available:http://dl6.globalstf.org/index.php/joc/article/view/1088
[^7]: L. Abeni and C. Kiraly, “Investigating the network performance of areal-time linux kernel.”
[^8]: I. Khoronzhuk and P. Varis, “Realtime Performance of NetworkingStack – SFO17209,”http://connect.linaro.org/resource/sfo17/sfo17-209/, accessed: 2018-04-12.
[^9]: “IRQs: the Hard, the Soft, the Threaded and the Preemptible alisonchaiken,”https://events.static.linuxfound.org/sites/events/files/slides/Chaiken_ELCE2016.pdf, accessed: 2018-04-12.
[^10]: “Real-time Ethernet (UDP) worst-case roundtriptime monitoring - osadl,” https://www.osadl.org/Real-time-Ethernet-UDP-worst-case-roun.qa-farm-rt-ethernet-udp-monitor.0.html, accessed: 2018-04-12.
[^11]: “Xenomai project home page,” April 2018, [Accessed: 2018-04-12].[Online]. Available: https://xenomai.org/
[^12]: “Rtai project home page,” April 2018, [Accessed: 2018-04-12].[Online]. Available: https://www.rtai.org/
[^13]: “The RTL Collaborative Project,” https://wiki.linuxfoundation.org/realtime/rtl/start, accessed: 2018-04-12.
[^14]: “Traffic control - linux queuing disciplines,” http://man7.org/linux/man-pages/man8/tc.8.html, accessed: 2018-04-12.
[^15]: “Multiqueue Priority Qdisc - linux man pages,” https://www.systutorials.com/docs/linux/man/8-tc-mqprio/, accessed: 2018-04-12.
[^16]: “Token Bucket Filter - linux queuing disciplines,” https://www.systutorials.com/docs/linux/man/8-tc-tbf/, accessed: 2018-04-12.
[^17]: “ CBS - Credit Based Shaper (CBS) Qdisc,” http://man7.org/linux/man-pages/man8/tc-cbs.8.html, accessed: 2018-04-12.
[^18]: “Scheduled packet transmission: Etf,” [Accessed: 2018-04-12].[Online]. Available: https://lwn.net/Articles/758592/
[^19]: J. Edge, “Moving interrupts to threads,” October 2008, [Accessed:2018-04-12]. [Online]. Available: https://lwn.net/Articles/302043/
[^20]: J. Corbet, “Software interrupts and realtime,” October 2012,[Accessed: 2018-04-12]. [Online]. Available: https://lwn.net/Articles/520076/
[^21]: “XDP (eXpress Data Path) documentation,” https://lwn.net/Articles/701224/, accessed: 2018-04-12.
[^22]: “The road towards a linux tsn infrastructure, jesus sanchezpalencia,”April 2018, [Accessed: 2018-04-12]. [Online]. Available:https://elinux.org/images/5/56/ELC-2018-USA-TSNonLinux.pdf
[^23]: “Cyclictest,” April 2018, [Accessed: 2018-04-12]. [Online]. Available:https://wiki.linuxfoundation.org/realtime/documentation/howto/tools/cyclictest
[^24]: S. Derr, “CPUSETS,” https://www.kernel.org/doc/Documentation/cgroup-v1/cpusets.txt, accessed: 2018-04-12.




