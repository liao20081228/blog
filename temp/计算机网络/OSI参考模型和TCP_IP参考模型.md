---
title: OSI参考模型和TCP_IP参考模型
tags: 计算机网络
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文章参考了<font color=blue >谢希仁的《 计算机网络》、好就分享的[《OSI七层协议模型、TCP/IP四层模型学习笔记》](https://www.cnblogs.com/Robin-YB/p/6668762.html)。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------



# 1 OSI七层协议模型
|分层|数据格式|功能与连接方式|典型设备
|:--|:--|:--|:--|
|应用层|报文|网络服务与使用者应用程序间的一个接口|终端设备|
|表示层 |报文|数据表示、数据安全、数据压缩|终端设备|
|会话层| 报文|会话层连接到传输层的映射；会话连接的流量控制；数据传输；会话连接恢复与释放；会话连接管理、差错控制|终端设备|
|传输层| 段|用一个寻址机制来标识一个特定的应用程序（端口号）|终端设备|
|网络层|分组|基于网络层地址（IP地址）进行不同网络系统间的路径选择|网关、路由器|
|数据链路层|帧|在物理层上建立、撤销、标识逻辑链接和链路复用以及差错校验等功能。通过使用接收系统的硬件地址或物理地址来寻址|网桥、交换机|
|物理层|比特|建立、维护和取消物理连接|光纤、同轴电缆、双绞线、网卡、中继器、集线器|
|&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;|&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;|&emsp;|&emsp;|

![图1](https://www.github.com/liao20081228/blog/raw/master/图片/OSI参考模型和TCP_IP参考模型/1.jpg)

## 1.1 物理层（Physical Layer）
&emsp;&emsp;物理层是OSI分层结构体系中最重要、最基础的一层，它建立在传输媒介基础上，起建立、维护和取消物理连接作用，实现设备之间的物理接口。物理层只接收和发送一串比特(bit)流，不考虑信息的意义和信息结构。

* 物理层包括对连接到网络上的设备描述其各种机械的、电气的、功能的规定。
	* 机械特性规定了网络连接时所需接插件的规格尺寸、引脚数量和排列情况等
	* 电气特性规定了在物理连接上传输bit流时线路上信号电平的大小、阻抗匹配、传输速率距离限制等；
	* 功能特性是指对各个信号先分配确切的信号含义，即定义了DTE（数据终端设备）和DCE（数据通信设备）之间各个线路的功能；
	* 过程特性定义了利用信号线进行bit流传输的一组操作规程，是指在物理连接的建立、维护、交换信息时，DTE和DCE双方在各电路上的动作系列。
* 物理层的数据单位是比特。
* 属于物理层定义的典型规范代表包括：EIA/TIARS-232、EIA/TIARS-449、V.35、RJ-45等。
* 物理层的主要功能：
	* 为数据端设备提供传送数据的通路，数据通路可以是一个物理媒体，也可以是多个物理媒体连接而成。一次完整的数据传输，包括激活物理连接，传送数据，终止物理连接。所谓激活，就是不管有多少物理媒体参与，都要在通信的两个数据终端设备间连接起来，形成一条通路。
	* 传输数据。物理层要形成适合数据传输需要的实体，为数据传送服务：
		1. 保证数据按位传输的正确性；
		2. 向数据链路层提供一个透明的位传输；
		3. 提供足够的带宽(带宽是指每秒钟内能通过的比特(BIT)数)，以减少信道上的拥塞。传输数据的方式能满足点到点，一点到多点，串行或并行，半双工或全双工，同步或异步传输的需要。

	* 完成物理层的一些管理工作，如在数据终端设备、数据通信和交换设备等设备之间完成对数据链路的建立、保持和拆除操作。

* 物理层的典型设备：光纤、同轴电缆、双绞线、中继器和集线器

## 1.2 数据链路层（Data Link Layer）
&emsp;&emsp;在物理层提供比特流服务的基础上，将比特信息封装成数据帧Frame，起到在物理层上建立、撤销、标识逻辑链接和链路复用以及差错校验等功能。通过使用接收系统的硬件地址或物理地址来寻址。建立相邻结点之间的数据链路，通过差错控制提供数据帧（Frame）在信道上无差错的传输，同时为其上面的网络层提供有效的服务。

* 数据链路层在不可靠的物理介质上提供可靠的传输。该层的作用包括：物理地址寻址、数据的成帧、流量控制、数据的检错、重发等。
* 数据的传输单位称为帧（frame）。
* 数据链路层协议的代表包括：SDLC、HDLC、PPP、STP、帧中继等。
* 链路层的主要功能：
	* 链路层的功能是实现系统实体间二进制信息块的正确传输
	* 为网络层提供可靠无错误的数据信息
	* 在数据链路中解决信息模式、操作模式、差错控制、流量控制、信息交换过程和通信控制规程的问题
* 链路层是为网络层提供数据传送服务的，这种服务要依靠本层具备的功能来实现。链路层应具备如下功能：
   * 链路连接的建立，拆除，分离。
   * 帧定界和帧同步。链路层的数据传输单元是帧，协议不同，帧的长短和界面也有差别，但无论如何必须对帧进行定界。
   * 顺序控制，指对帧的收发顺序的控制。
   * 差错检测和恢复。还有链路标识，流量控制等等。差错检测多用方阵码校验和循环码校验来检测信道上数据的误码，而帧丢失等用序号检测。各种错误的恢复则常靠反馈重发技术来完成。
* 数据链路层的典型设备：二层交换机、网桥、网卡

## 1.3 网络层（Network Layer）
&emsp;&emsp;网络层也称通信子网层，是高层协议之间的界面层，用于控制通信子网的操作，是通信子网与资源子网的接口。在计算机网络中进行通信的两个计算机之间可能会经过很多个数据链路，也可能还要经过很多通信子网。网络层的任务就是选择合适的网间路由和交换结点，确保数据及时传送。网络层将解封装数据链路层收到的帧，提取数据包，包中封装有网络层包头，其中含有逻辑地址信息源站点和目的站点地址的网络地址。
&emsp;&emsp;如果你在谈论一个IP地址，那么你是在处理第3层的问题，这是“数据包”问题，而不是第2层的“帧”。IP是第3层问题的一部分，此外还有一些路由协议和地址解析协议（ARP）。有关路由的一切事情都在第3层处理。地址解析和路由是3层的重要目的。网络层还可以实现拥塞控制、网际互连、信息包顺序控制及网络记账等功能。

* 在网络层交换的数据单元的单位是分组。
* 网络层协议的代表包括：IP、IPX、OSPF等。
* 网络层主要功能是基于网络层地址（IP地址）进行不同网络系统间的路径选择。
* 网络层为建立网络连接和为上层提供服务，应具备以下主要功能：
	* 路由选择和中继
	* 激活，终止网络连接
	* 在一条数据链路上复用多条网络连接，多采取分时复用技术
	* 差错检测与恢复
	* 排序，流量控制
	* 服务选择
	* 网络管理
* 网络层典型设备：网关、路由器

## 1.4 传输层（Transport Layer）
&emsp;&emsp;传输层建立在网络层和会话层之间，实质上它是网络体系结构中高低层之间衔接的一个接口层。用一个寻址机制来标识一个特定的应用程序（端口号）。传输层不仅是一个单独的结构层，它还是整个分层体系协议的核心，没有传输层整个分层协议就没有意义。
&emsp;&emsp;传输层是两台计算机经过网络进行数据通信时，第一个端到端的层次，具有缓冲作用。当网络层服务质量不能满足要求时，它将服务加以提高，以满足高层的要求；当网络层服务质量较好时，它只用很少的工作。传输层还可进行复用，即在一个网络连接上创建多个逻辑连接。
&emsp;&emsp;传输层也称为运输层。传输层只存在于端开放系统中，是介于低三层通信子网和高三层资源子网之间的一层，但是很重要的一层。因为它是源端到目的端对数据传送进行控制从低到高的最后一层。
&emsp;&emsp;世界上各种通信子网在性能上存在着很大差异。例如电话交换网、分组交换网、公用数据交换网、局域网等通信子网都可互连，但它们提供的吞吐量、传输速率、数据延迟通信费用各不相同。对于会话层来说，却要求有一性能恒定的界面。传输层就承担了这一功能。它采用分流/合流、复用/介复用技术来调节上述通信子网的差异，使会话层感受不到。
&emsp;&emsp;此外传输层还要具备差错恢复、流量控制等功能，以此对会话层屏蔽通信子网在这些方面的细节与差异。传输层面对的数据对象已不是网络地址和主机地址，而是和会话层的界面端口。上述功能的最终目的是为会话提供可靠的、无误的数据传输。传输层的服务一般要经历传输连接建立阶段、数据传送阶段、传输连接释放阶段3个阶段才算完成一个完整的服务过程。而在数据传送阶段又分为一般数据传送和加速数据传送两种。传输层服务分成5种类型。基本可以满足对传送质量、传送速度、传送费用的各种不同需要。

* 传输层的数据单元是段
* 传输层协议的代表包括：TCP、UDP、SPX等。
* 传输层获得下层提供的服务包括：
	* 发送和接收正确的数据块分组序列，并用其构成传输层数据；
	* 获得网络层地址，包括虚拟信道和逻辑信道。
* 传输层向上层提供的服务包括：
   * 无差错的有序的报文收发；
   * 提供传输连接；
   * 进行流量控制。
   * 传输层为上层提供端到端的透明的、可靠的数据传输服务，所谓透明的传输是指在通信过程中传输层对上层屏蔽了通信传输系统的具体细节。

* 传输层的主要功能是从会话层接收数据，根据需要把数据切成较小的数据片，并把数据传送给网络层，确保数据片正确到达网络层，从而实现两层数据的透明传送。

## 1.5 会话层（Session Layer）
&emsp;&emsp;这一层也可以称为会晤层或对话层，在会话层及以上的高层次中，数据传送的单位不再另外命名，统称为报文。会话层不参与具体的传输，它提供包括访问验证和会话管理在内的建立和维护应用之间通信的机制。如服务器验证用户登录便是由会话层完成的。
&emsp;&emsp;会话层提供的服务可使应用建立和维持会话，并能使会话获得同步。会话层使用校验点可使通信会话在通信失效时从校验点继续恢复通信。这种能力对于传送大的文件极为重要。会话层、表示层、应用层构成开放系统的高3层，面对应用进程提供分布处理，对话管理，信息表示，恢复最后的差错等。会话层同样要担负应用进程服务要求，而运输层不能完成的那部分工作，给运输层功能差距以弥补。主要的功能是对话管理，数据流同步和重新同步。要完成这些功能，需要由大量的服务单元功能组合，已经制定的功能单元已有几十种。

* 会话层的主要功能： 
	* 会话层连接到传输层的映射；
   * 会话连接的流量控制；
   * 数据传输；
   * 会话连接恢复与释放；
   * 会话连接管理、差错控制。
* 为会话实体间建立连接、为给两个对等会话服务用户建立一个会话连接，应该做如下几项工作：
   * 将会话地址映射为传输地址；
   * 选择需要的传输服务质量参数(QOS)；
   * 对会话参数进行协商；
   * 识别各个会话连接；
   * 传送有限的透明用户数据；
   * 数据传输阶段。这个阶段是在两个会话用户之间实现有组织的，同步的数据传输。用户数据单元为SSDU，而协议数据单元为SPDU。会话用户之间的数据传送过程是将SSDU转变成SPDU进行的。
   * 连接释放。连接释放是通过"有序释放"、"废弃"、"有限量透明用户数据传送"等功能单元来释放会话连接的。会话层标准为了使会话连接建立阶段能进行功能协商，也为了便于其它国际标准参考和引用，定义了12种功能单元。各个系统可根据自身情况和需要，以核心功能服务单元为基础，选配其他功能单元组成合理的会话服务子集。会话层的主要标准有"DIS8236:会话服务定义"和"DIS8237:会话协议规范"。

## 1.6 表示层（Presentation Layer）
&emsp;&emsp;表示层向上对应用层提供服务，向下接收来自会话层的服务。表示层是为在应用过程之间传送的信息提供表示方法的服务，它关心的只是发出信息的语法与语义。

* 表示层要完成某些特定的功能要有：
   * 不同数据编码格式的转换，
   * 提供数据压缩、解压缩服务，
   * 对数据进行加密、解密
* 表示层为应用层提供服务包括语法选择、语法转换等。
   * 语法选择是提供一种初始语法和以后修改这种选择的手段。
   * 语法转换涉及代码转换和字符集的转换、数据格式的修改以及对数据结构操作的适配。

## 1.7 应用层（Application Layer）
&emsp;&emsp;网络应用层是通信用户之间的窗口，为用户提供网络管理、文件传输、事务处理等服务。其中包含了若干个独立的、用户通用的服务协议模块。网络应用层是OSI的最高层，为网络用户之间的通信提供专用的程序。应用层的内容主要取决于用户的各自需要，这一层设计的主要问题是分布数据库、分布计算技术、网络操作系统和分布操作系统、远程文件传输、电子邮件、终端电话及远程作业登录与控制等。至2011年应用层在国际上没有完整的标准，是一个范围很广的研究领域。在OSI的7个层次中，应用层是最复杂的，所包含的应用层协议也最多，有些还在研究和开发之中。

* 应用层为操作系统或网络应用程序提供访问网络服务的接口。
* 应用层协议的代表包括：Telnet、FTP、HTTP、SNMP、DNS等。

# 2 TCP/IP四层模型
![2](https://www.github.com/liao20081228/blog/raw/master/图片/OSI参考模型和TCP_IP参考模型/2.jpg)

![3](https://www.github.com/liao20081228/blog/raw/master/图片/OSI参考模型和TCP_IP参考模型/3.jpg)

* 应用层
应用层对应于OSI参考模型的高层，为用户提供所需要的各种服务，例如：FTP、Telnet、DNS、SMTP等.
* 传输层
传输层对应于OSI参考模型的传输层，为应用层实体提供端到端的通信功能，保证了数据包的顺序传送及数据的完整性。该层定义了两个主要的协议：传输控制协议（TCP）和用户数据报协议（UDP).
   * TCP协议提供的是一种可靠的、通过“三次握手”来连接的数据传输服务；
   * 而UDP协议提供的则是不保证可靠的（并不是不可靠）、无连接的数据传输服务.
* 网际互联层
网际互联层对应于OSI参考模型的网络层，主要解决主机到主机的通信问题。它所包含的协议设计数据包在整个网络上的逻辑传输。注重重新赋予主机一个IP地址来完成对主机的寻址，它还负责数据包在多种网络中的路由。该层有三个主要协议：网际协议（IP）、互联网组管理协议（IGMP）和互联网控制报文协议（ICMP）。
IP协议是网际互联层最重要的协议，它提供的是一个可靠、无连接的数据报传递服务。
* 网络接口层（物理链路层）
网络接入层与OSI参考模型中的物理层和数据链路层相对应。它负责监视数据在主机和网络之间的交换。事实上，TCP/IP本身并未定义该层的协议，而由参与互连的各网络使用自己的物理层和数据链路层协议，然后与TCP/IP的网络接入层进行连接。地址解析协议（ARP）工作在此层，即OSI参考模型的数据链路层。


# 3 OSI和TCP/IP的关系

* OSI引入了服务、接口、协议、分层的概念，TCP/IP借鉴了OSI的这些概念建立TCP/IP模型。

*  OSI先有模型，后有协议，先有标准，后进行实践；而TCP/IP则相反，先有协议和应用再提出了模型，且是参照的OSI模型。

* OSI是一种理论下的模型，而TCP/IP已被广泛使用，成为网络互联事实上的标准。

<table align='center' border="1">
    <caption>OSI模型和TCP/IP模型对比</caption>
    <tr>
        <th align='left'>OSI模型</th>
        <th align='left'> TCP/IP模型</th>
        <th align='left'>对应网络协议</th>
    </tr>
    <tr>
        <td align='left'>应用层</td>
        <td align='left' rowspan="3">应用层</td>
        <td align='left'>HTTP、TFTP、FTP、NFS、SMTP、WWW、Telnet、Gateway、SNMP</td>
    </tr>
    <tr>
        <td align='left'>表示层</td>
        <td align='left'>GIF、JPEG、ASCII、MPEG、MIDI,HTML</td>
    </tr>
    <tr>
        <td align='left'>会话层</td>
        <td align='left'>RPC、SQL、NetBIOS、ADSP、ASP、SCP、SSH、SDP、RTCP</td>
    </tr>
    <tr>
        <td align='left'>传输层</td>
        <td align='left'>传输层</td>
        <td align='left'>TCP（传输控制协议）UDP（ 用户数据报协议）</td>
    </tr>
    <tr>
        <td align='left'>网络层</td>
        <td align='left'>网络层（网际互连层）</td>
        <td align='left'>IP、 ICMP、IGMP</td>    
    </tr>
    <tr>
         <td align='left'>数据链路层</td>
        <td align='left' rowspan="2">物理链路层<br>也叫网络接口层</td>
        <td align='left'>Ethernet、PPP、STP，IEEE 802系列，ARP、RARP</td>
    </tr>
    <tr>
        <td align='left'>物理层</td>
        <td align='left'>EIA/TIARS-232、EIA/TIARS-449、RJ-45</td> 
    </tr>
</table>

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文章参考了<font color=blue >谢希仁的《 计算机网络》、好就分享的[《OSI七层协议模型、TCP/IP四层模型学习笔记》](https://www.cnblogs.com/Robin-YB/p/6668762.html)。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------