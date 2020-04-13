---
title: RDMA编程手册 第0章 术语
tags: RDMA
---

------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>[《RDMA Aware Networks Programming User Manual v1.7》](https://www.mellanox.com/related-docs/prod_software/RDMA_Aware_Programming_user_manual.pdf)、[《Dotan Barak的RDMAmojo博客》](http://www.rdmamojo.com/)、《man手册v24.0》，我所做的工作就是更新、整理一部分内容，删除了一部分废弃的API，添加了一部分新的API。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------



|缩写|英文全名|中文全名|描述|
|:--|:--|:--|:--|
||Access Layer|接入层|用于访问互连结构(VPI™、InfiniBand®、以太网、FCoE)的低级操作系统基础设施(管道)。它包括支持上层网络协议、中间件和管理代理所需的所有基本传输服务。
| AH |Address Handle|地址句柄|在UD QP中用于描述达远程端的路径的对象
|CA| Channel Adapter|通道适配器|一种终止无限带宽链接并执行传输层功能的设备
|CI |Channel Interface|通道接口|通过网络适配器，相关固件和设备驱动程序软件的组合实现的面向动词消费者的通道描述|
|CM|Communication Manager|通信管理器|负责建立、维护和释放通信的实体，用于RC和UC QP服务类型。<br  />服务ID解析协议使UD服务的用户能够找到支持其所需服务的QP。<br  />终端节点的每个IB端口中都有一个CM。|
||Compare & Swap|比较和交换|指示远程QP读取64位值，将其与提供的比较数据进行比较，如果相等，则将其替换为QP中提供的交换数据。|
|CQ|Completion Queue|完成队列|包含CQEs的队列（FIFO）|
|CQE |Completion Queue Entry|完成队列条目|CQ中的一项，描述有关已完成WR的信息（状态大小等）
|DMA|Direct Memory Access|直接内存访问|允许硬件绕过CPU直接将数据块移入和移出内存|
||Fetch & Add|取回和添加| 指示远程QP读取64位值，并将其替换为QP中提供的数据值与该64位值之和。|
|GUID|Globally Unique IDentifier|全局唯一标识符|一个唯一标识子网中设备或组件的64位数字|
|GID|Global IDentifier|全局标识符|128位标识符，用于标识网络适配器上的端口，路由器上的端口或多播组。<br  />GID是有效的128位IPv6地址（根据RFC 2373），在IBA中定义了其他属性/限制，以促进有效的发现，通信和路由。|
|GRH |Global Routing Header|全局路由头部|数据包头，用于跨子网边界传递数据包，也用于传递多播消息。<br  />此数据包头基于IPv6协议。|
||Network Adapter |网络适配器|一种允许在网络中的计算机之间进行通信的硬件设备。|
||host|主机|一种运行了操作系统的计算机平台，该操作系统可以控制一个或多个网络适配器|
|HCA|Host Channel Adapter|主机通道适配器|
|IB|InfiniBand|无限带宽技术|
||Join operation |加入操作|IB端口必须显式通过向SA发送请求来加入多播组以接收多播数据包。
||lkey |本地密钥|注册MR时接收到的数字，它将在本地被WR用于标识存储区域及其关权限。
|LID |Local IDentifier|本地标识符|子网管理器分配给终端节点的16位地址。每个LID在其子网内都是唯一的。
|LLE |Low Latency Ethernet|低延时以太网|基于CEE（融合增强型以太网）的RDMA服务，允许通过以太网进行IB传输。|
|NA| Network Adapter|网络适配器|终止链接并执行传输层功能的设备。|
|MGID |Multicast Group ID|多播组ID|IB多播组用MGID标识，并由SM管理。 SM将MLID与每个MGID关联，并在结构中对IB交换机进行显式编程，以确保加入多播组的所有端口均接收到数据包。|
|MR| Memory Region|内存区域|一组连续的内存缓冲区，这些缓冲区已经通过访问权限进行了注册。这些缓冲区需要注册，以便网络适配器使用它们。在注册期间，将创建L_Key和R_Key并将其与创建的内存区域关联|
|MTU |Maximum Transfer Unit|最大传输单元|可以从端口发送/接收的数据包的有效载荷（不包括头部）的最大大小|
|NIC|network interface controller|网络接口控制器|一块被设计用来允许计算机在计算机网络上进行通讯的计算机硬件。
|MW |Memory Window|内存窗口|一个被分配的资源，在绑定到现有“内存注册”中的指定区域后即可启用远程访问。每个内存窗口都有一个关联的窗口句柄，访问权限集和当前的R_Key。|
||Outstanding Work Request |未完成工作请求|WR已发布到工作队列，但它的完成没有查询到|
|pkey |Partition key|分区密钥|pkey标识端口所属的分区。 pkey大致类似于以太网网络中的VLAN ID。它用于指向端口的分区密钥（pkey）表中的条目。子网管理器（SM）为每个端口分配至少一个pkey。|
|PD |Protection Domain|保护域|其组件只能彼此交互的对象。 AHs与QPs交互，而MRs与WQs交互。
|QP|Queue Pair|队列对|一对独立的WQs（发送队列和接收队列）打包在一个对象中，目的是在网络节点之间传输数据。<br  />发布是用于启动数据的发送或接收。 <br  />QP有三种类型：UD不可靠数据报，不可靠连接和可靠连接。|
|RC |Reliable Connection|可靠连接|基于面向连接的协议的QP传输服务类型。<br  />一个QP（队列对）与另一个单个QP相关联。消息以可靠的方式发送（就信息的正确性和顺序而言）。|
|RDMA |Remote Direct Memory Access|远程内存直接访问|在不调用远程CPU的情况下访问远程端的内存|
|RDMA_CM|Remote Direct Memory Access Communication Manager|远程直接内存访问通信管理器|用于设置可靠的、连接的和不可靠的数据报数据传输的API。 它提供用于建立连接的RDMA传输中立接口。 该API基于套接字，但适用于基于队列对（QP）的语义：通信必须通过特定的RDMA设备进行，并且数据传输基于消息。|
||Requestor |请求者|连接中，将启动数据传输一侧（通过发布发送请求）|
||Responder |响应者|连接中，将响应来自请求者的命令的一侧，其中可能包括写响应者内存或读响应者内存的请求，以及最后请求响应者接收消息的命令。|
||rkey |远端密钥|注册MR时收到的数字，用于对传入的RDMA操作强制执行权限检查|
|RNR|Receiver Not Ready|接收器未就绪|在RC QP流中，通信的两边之间有连接，但接收端不存在RR
|RQ|Receive Queue|接收队列|保留用户发布的RRs的工作队列|
|RR|Receive Request|接收请求|发布到RQ的WR，其中使用发送操作码描述将输入数据写入何处。另请注意，一个即时的RDMA write将消耗一个RR。|
|RTR |Ready To Receive|准备接收|一个QP状态，该状态下可以发布和处理RR|
|RTS |Ready To Send|准备发送|一个QP状态，该状态下可以发布和处理SR|
|SA |Subnet Administrator|子网管理器|查询和操作子网管理数据的接口|
|SGE|Scatter /Gather Elements|分散/聚集元素|指向本地注册内存块的全部或一部分的指针的条目。这个条目保存块的起始地址，大小和lkey（及其关联的权限）。|
||S/G Array |S/G数组|WR中存在的一个S/G元素数组，根据所使用的操作码，该WR可以从多个缓冲区收集数据并将其作为单个流发送，也可以将单个流分解为多个缓冲区|
|SM| Subnet Manager|子网管理器|配置和管理子网的实体。<br  />发现网络拓扑。<br  />分配LID。确定路由方案并设置路由表。<br  />一个主SM，可能还有几个从SM（备用模式）。<br  />管理交换机路由表，从而建立通过结构的路径。|
|SQ| Send Queue|发送队列|保存用户发布的SRs的工作队列|
|SR|Send Request|发送请求|已发布到SQ的WR，其中描述了将要传输多少数据，其方向和方式（操作码将指定传输）|
|SRQ |Shared Receive Queue|共享接收队列|一个队列，其中持有来自与其相关联的任何RC/UC/UD QP中的传入消息的WQE。<br />一个SRQ可以关联多个QP。|
|TCA |Target Channel Adapter|目标适配器|不需要支持动词的通道适配器，通常在I/O设备中使用
|UC |Unreliable Connection|不可靠的连接|基于面向连接的协议的QP传输服务类型，其中QP（队列对）与另一个单个QP相关联。 QPs无法执行可靠的协议，消息可能会丢失。|
|UD |Unreliable Datagram|不可靠数据报|一种QP传输服务类型，其中消息可以是一个包的长度，并且每个UD QP可以从子网中的另一个UD QP发送/接收消息。<br />消息可能会丢失，不能保证顺序。 UD QP是唯一支持多播消息的类型。 UD数据包的消息大小限于路径MTU|
||Verbs|动词|网络适配器功能的抽象描述.<br />使用动词，任何应用程序都可以创建/管理使用RDMA进行数据传输所需的对象。
|VPI|Virtual Protocol Interface|虚拟协议接口|允许用户更改端口的第2层协议。|
|WQ |Work Queue|工作队列| 发送队列或接收队列之一。|
|WQE|Work Queue Element|工作队列元素|WQE，发音为“ wookie”，是工作队列中的一个元素。|
|WR |Work Request|工作请求|用户发布到工作队列的请求。
||input parameter|输入参数|传入了数据的参数。
||output parameter|输出参数|传出了数据的参数。|


------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>[《RDMA Aware Networks Programming User Manual v1.7》](https://www.mellanox.com/related-docs/prod_software/RDMA_Aware_Programming_user_manual.pdf)、[《Dotan Barak的RDMAmojo博客》](http://www.rdmamojo.com/)、《Ubuntu19.10 的rdma-core套件man手册v24.0》，我所做的工作就是更新、整理一部分内容，删除了一部分废弃的API，添加了一部分新的API。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

