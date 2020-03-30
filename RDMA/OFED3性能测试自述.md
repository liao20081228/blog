---
title: OFED3性能测试自述
tags: RDMA
---

------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>。</font><font color=red>未经作者允许</font>[《perftest官方文档》](https://github.com/linux-rdma/perftest)。<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。网络转载请注明出处！！！</font>***

------




# 1 概览
这是一个在uverb上编写的测试集合，用于性能微基准测试。这些测试可用于HW或SW调优以及功能测试。

这个测试集含有一系列带宽和延迟基准测试，例如：

* send ——ib_send_bw 和 ib_send_lat
* RDMA Read——ib_read_bw 和 ib_read_lat
* RDMA Write——ib_write_bw和ib_wriet_lat
* RDMA Atomic——ib_atomic_bw和ib_atomic_lat
* Natice Ethernet——raw_ethernet_bw和raw_ethernet_lat（和MOFED2一起工作的时）

请将结果/观察结果发布到openib-general邮件列表。参见“联系我们”<http://openib.org/mailman/listinfo/openib-general和http://www.openib.org>。

# 2 测试方法说明
* 基准测试使用CPU循环计数器获得时间戳，而不需要上下文切换。一些CPU架构(例如，Intel的80486或更老的PPC)没有这样的能力。

* 延迟基准测试测量往返时间，但将其中一半作为单向延迟。这意味着对于非对称配置，结果可能不准确。

* 在所有单向带宽基准测试中，客户端测量带宽。在双向带宽基准测试中，每一方都测量它所发起的流量的带宽，在测量周期结束时，服务器将结果报告给客户端，客户端将它们组合在一起。

* 延迟测试报告延迟测量结果的最小值、中间值和最大值。与平均延迟测量相比，中间值延迟通常对高延迟变化不那么敏感。
通常，由于预热效应，测量的第一个值是最大值。

* 长采样周期对测量精度的影响非常有限。迭代的默认值设置为1000次是非常好的。
注意，程序保持带内存占用的数据结构与迭代次数成比例。设置一个非常高的迭代次数可能会对测量的性能产生负面影响，而这与被测设备无关。
如果需要大量的迭代，建议使用-N标志(没有峰值)。

* 带宽基准测试可以运行多次迭代，也可以运行固定的时间。使用-D标志指示测试运行指定的秒数。run_infinite标志指示程序一直运行，直到被用户中断，并且每5秒打印一次测量的带宽。

* 延迟基准测试中的“-H”选项转储结果的直方图。见xgraph, ygraph, r-base (<http://www.r-project.org/>)， PSPP，或其他统计分析程序。

**重要提示**

在Infiniband fabric上运行基准测试时，子网管理器必须在启动基准测试之前在交换机或fabric中的一个节点上运行。

已经测试的架构:i686、x86_64、ia64

# 3 基准测试描述

这些基准测试生成一个综合的操作流，这对于硬件和软件的基准测试和分析非常有用。

这些基准测试并不是为了模拟任何实际的应用程序流量。实际的应用程序流量可能会受到许多参数的影响，仅根据这些基准测试的结果可能无法预测。

|测试程序名|描述|
|:--|:--|
|ib_send_lat|  发送事物的延迟测试|
|ib_send_bw|  发送事物的带宽测试|
|ib_write_lat| 	RDMA write事物的延迟测试|
|ib_write_bw |   RDMA write事物的带宽测试|
|ib_read_lat |	 RDMA read事物的带宽测试|
|ib_read_bw |	   RDMA read事物的带宽测试|
|ib_atomic_lat|    原子事物的延迟测试|
|ib_atomic_bw|	原子事物的带宽测试|


原始以太网接口基准测试:
|测试程序名|描述|
|:--|:--|
|raw_ethernet_send_lat|基于原始以太网接口的延迟测试|
|raw_ethernet_send_bw|基于原始以太网接口的带宽测试|

# 4 运行测试

前提条件：
* kernel 2.6
* (内核模块) 匹配 libibverbs
* (内核模块) 匹配 librdmacm
* (内核模块) 匹配 libibumad
* (内核模块) 匹配 libmath (lm).


服务端：`./<测试程序> <选项>`
客户端：`./<测试程序> <选项> <服务端IP地址>`
* `<服务端IP地址>`是IPv4或IPv6地址。如果配置了IPoIB，您可以使用IPoIB地址。
* —help列出可用的`<选项>`

所有测试都含有的公共选项：

|短选项|长选项|描述|
|:--|:--|:--|
|-h|--help|显示帮助
|-p \<port>|--port=\<port>|	监听/连接端口\<port>	 (默认: 18515)|
| -R| --rdma_cm	|用rdma_cm连接QPs，并在这些QPs上运行测试|
|-z|--com_rdma_cm|使用rdma_cm模块通信以交换数据——使用常规QPs|
|-m \<mtu>|--mtu=\<mtu>|QP MTU 大小（默认： ibv_devinfo中的active_mtu ）
|-c \<type>|--connection=\<type>|		连接类型， RC/UC/UD/XRC/DC/SRD (默认 RC).
|-d \<dev>	|--ib-dev=\<dev>|使用IB设备\<dev> (默认: 第一个找到的设备|
|-i \<port>	|--ib-port=\<port>|	使用IB设备的网络端口 \<port>(默认: 1)|
|-s \<size>| --size=\<size>	|	交换的消息大小	 (默认: 1) |
|-a| --all		|	运行测试，大小从2到2^23^|
|-n \<iters>	|--iters=\<iters>	|		交换次数 (至少100，默认：1000)|
|-x \<index>	|--gid-index=\<index>	|测试使用从命令中获取GID索引的GID值测试，（默认：IB没有gid，以太网gid为0）	|
|-V|--version	|		显示版本号|
|-e|--events	|			在CQ事件时睡眠 (默认 poll)|
|-F|--CPU-freq|		即使是cpufreq_ondemand模块也不要失败|
|-I \<size>|--inline_size=\<size>	|以内联模式发送的消息的最大大小|
|-u \<timeout>	|--qp-timeout=\<timeout>	|QP超时= (4 微秒)\*(2^timeout^) (默认: 14)
|-S \<sl>	|--sl=\<sl>		|服务级别（默认0）|
|-r \<dep>	|--rx-depth=\<dep>	|接收队列深度	（默认600）|

延迟测试选项：

|短选项|长选项|描述|
|:--|:--|:--|
|-C|--report-cycles	|以CPU周期为单位报告时间|
|-H|--report-histogram|	打印所有结果(默认:只显示总结)|
|-U|--report-unsorted|打印出未排序的结果（默认：已排序）|

带宽测试选项：

|短选项|长选项|描述|
|:--|:--|:--|
|-b|--bidirectional	|测量双向带宽（默认单向）|
|-N| --no peak-bw|取消峰值带宽计算 (默认：计算峰值带宽)
|-Q|--cq-mod|仅在	\<cq-mod>完成后生成Cqe|
|-t \<dep>| --tx-depth=\<dep>|	tx队列的大小 (默认: 128)|
|-O|--dualport|	在双端口模式下运行测试(2 QPs)。两个端口都必须是活动的(默认关闭) |
|-D \<sec> 	| --duration=\<sec> |运行 \<sec>秒测试|
|-f \<sec> | --margin=\<sec>| 在持续时间内，测量结果含有边缘值(默认值:2)|
|-l \<list size>|--post_list=\<list size>|发布\<list size>大小的WQEs列表(而不是单次发布)|
|-q \<num of qp's>|	--qp=\<num of qp's>|进程中运行的QPs数量(默认值:1)|
|  | --run_infinitely|运行测试直到用户中断，每5秒打印一次结果|
	
	
发送测试选项：	

|短选项|长选项|描述|
|:--|:--|:--|
|-r \<dep> | --rx-depth=\<dep>| 接收队列大小 (默认: 带宽测试中512)|
|-g \<num_of_qps>|--mcg=\<num_of_qps>| 向多播组发送消息，并附加\<num_of_qps> QPs|	
| -M \<multicast_gid>|--MGID=\<multicast_gid>|	在多播中，使用\<multicast_gid>作为组MGID|
	  
原子测试选项：
|短选项|长选项|描述|
|:--|:--|:--|
|-A \<type>| --atomic_type=\<type>|原子操作类型，CMP_AND_SWAP/FETCH_AND_ADD|
|-o \<num>	| --outs=\<num>	|未完成的读/原子请求数量——也在READ测试中|


raw_ethernet_send_bw 选项:
|短选项|长选项|描述|
|:--|:--|:--|
|-B|--source_mac|源MAC地址,格式XX:XX:XX:XX:XX:XX:XX(默认采用MAC地址格式GID)|
|-E| --dest_mac	|目的MAC，格式XX:XX:XX:XX:XX:XX:XX，不可省略|
|-J| --server_ip|服务端IP地址，格式 X.X.X.X (用于发送带IP头部的包|
|-j|--client_ip	|客户端IP地址，格式 X.X.X.X (用于发送带IP头部的包|
|-K| --server_port	|服务端端口号，用于发送带UDP头部的包|	
|-k|--client_port|	客户端端口号，用于发送带UDP头部的包|	
|-Z| --server|	当前计算机作为服务器端，不可省略|
|-P|--client|	为当前计算机作为客户端	，不可省略	|

特殊功能在测试中的详细说明：

1. 使用post_list特性 (-l, --post_list=\<list size>).
在这种情况中，每个QP将准备\<list size>大小的 ibv_send_wr(而不是1)，并将它们相互链接。
在链接中，我们平均是分配\<list size>大小的数组，并设置数组中的每个ibv_send_wr的'next'指针指向数组中的下一个元素。数组中最后一个ibv_send_wribv的next指针将指向NULL。
在这种情况中，当发布布送列表中的第一个ibv send wr时，将指示硬件发布所有的WQEs。这意味着每个post_send将发布\<list size>大小的消息。
这个特性很好，我们想知道单个进程中QPs的最大消息率。
由于我们仅限于SW post_send（〜10 Mpps，因为每个SW post_send之间有〜500 ns），因为它不依赖于SW限制 ，因此当设置\<list_size>为64（例如）时，我们可以看到真实的硬件消息速率。

2. RDMA连接模式(CM)
您可以将“-R”标记添加到所有测试中，以用rdma_cm库将每一端的QPs连接起来。
在这种情况下，这个库将连接QPs并使用IPoIB接口来完成测试。当您在两个节点之间没有以太网连接时，它会有所帮助。
您必须提供IPoIB接口作为服务端IP。

3. ib_send_lat和ib_send_bw中的多播支持
send测试在动词级别内置了测试多播性能的功能。您可以使用“-g”来指定附加到这个多播组的QPs数量。“-M”标志允许您选择多播组地址。

# 5 已知问题
1. ib_send_lat和ib_send_bw中的多播支持不稳定。
基准程序可能会挂起或表现出其他意外行为。

2. 在UD或UC模式下运行时，ib_send_bw测试中的双向支持。
在极少数情况下，基准程序可能会挂起。
perftest-2.3发行版包含用于挂起检测的功能，在这种情况下，该功能将在2分钟后退出测试。

3. perftest的不同版本可能彼此不兼容。
请在两侧使用相同的perftest版本，以确保基准测试结果的一致性。

4. 测试版5.3及更高版本不能与早期版本的perftest一起工作。

5. 由于MLNX_OFED-2.2中的API更改，此perftest软件包无法在MLNX_OFED-2.1上编译
为了正确编译，请执行以下操作：
`./configure --disable-verbs_exp`
 `make`
 
6. 在x390x平台虚拟化环境中，程序包测试应用程序显示的结果可能不正确。

7. perftest-2.3版本包括对dualport VPI测试的支持——端口1-以太网、端口2-ib。(除了Eth:Eth, IB:IB之外)。目前，在端口1-IB、端口2-以太网运行dualport仍不能工作。

8. 对于postlist，当使用迭代且num_iter /poslist_val有余数时，测试将不会正确完成。


------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>[《perftest官方文档》](https://github.com/linux-rdma/perftest)。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。网络转载请注明出处！！！</font>***

------
