---
title: RDMA编程手册 第4章 libibverbs
tags: RDMA
---

------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>[《RDMA Aware Networks Programming User Manual v1.7》](https://www.mellanox.com/related-docs/prod_software/RDMA_Aware_Programming_user_manual.pdf)、[《Dotan Barak的RDMAmojo博客》](http://www.rdmamojo.com/)、《man手册v24.0》，我所做的工作就是更新、整理一部分内容，删除了一部分废弃的API，添加了一部分新的API。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

# 1 libibverbs介绍
## 1.1 概述

libibverbs是Infiniband（根据Infiniband规范）和iWARP（根据iWARP动词规范）的RDMA动词的一种实现。它处理创建、修改、查询和销毁资源的控制路径（control path），例如保护域（PD），完成队列（CQ），队列对（QP），共享接收队列（SRQ），地址句柄（AH），内存区域（MR）。它还处理被发布到QP和SRQ的发送数据和接收数据，并使用轮询和完成事件从CQ中获取工作完成。

控制路径是通过对uverbs内核模块的系统调用来实现的，该模块进一步调用了底层硬件驱动程序。数据路径是通过对底层硬件库的调用来实现的，在大多数情况下，这些库与硬件直接交互，从而提供了内核和网络堆栈旁路（保存上下文/模式切换）以及零拷贝和异步I/O模型。

通常，在网络编程和RDMA编程中，存在涉及与远程对等方（例如地址解析和连接建立）和远程实体（例如IB下的路由解析和加入多播组）交互的操作，其中通过IB动词管理的资源(如QP或AH)最终将在此交互中被创建或被实现。在这种情况下，其寻址语义是基于IP的应用程序可以使用与libibverbs协同工作的librdmacm。

## 1.2 线程安全

该库是线程安全的库，动词可以被进程中的每个线程调用，甚至可以从不同的线程处理相同的资源（保证操作的原子性）。 但是，由用户决定在资源被销毁后（由同一线程或任何其他线程）停止使用资源，否则可能导致段错误。

## 1.3 fork安全

一般来说，在使用libibvebrs时，应该避免使用fork()，无论是显式地调用它或隐式地调用它(通过调用fork()的其他系统调用，如system()、popen()等)。

如果必须使用fork()，请阅读ibv_fork_init()的文档。
 
 ## 1.4 库API
库中的函数应当声明为函数，有些函数可能声明为宏。

为了使用libibvebrs，源代码中必须包含头文件verbs.h:

```cpp
#include <infiniband/verbs.h>
```
## 1.5 资源创建依赖

![图1](https://gitee.com/liao20081228/blog_pictures/raw/master/RDMA编程手册_第4章_libibverbs/图1.jpg#pic_center)

 ## 1.6 静态链接

如果libibverbs静态链接到应用程序，则所有供应商驱动程序也必须静态链接。 使用静态链接时，该库不会加载动态链接的供应商驱动程序。

要链接供应商驱动程序，请在编译应用程序时，将RDMA_STATIC_PROVIDERS定义为所需供应商程序的逗号分隔列表。 特殊关键字“ all”将静态链接所有受支持的libibverbs供应商驱动程序。

它旨在与pkg-config（1）一起使用，以为libibverbs链接设置适当的标志。

 如果不这样做，那么ibv_get_device_list将始终返回一个空列表。

 强烈建议对libibverbs应用程序只使用动态链接。

## 1.7 典型错误消息
以下是执行libibverb应用程序时可能打印到stderr的典型错误消息列表，以及解决方法:

 **libibverbs: Fatal: couldn't read uverbs abi version**
 * 原因：libibverb找不到表示内核和libibverbs之间ABI(应用程序二进制接口)版本的文件(/sys/class/infiniband_verb /abi_version)。
 * 来源：这通常发生在模块ib_uverb未加载时。
 * 解决办法：：如果安装了RDMA包(OFED) ，重新启动计算机。否则，使用适当的服务文件加载RDMA堆栈驱动程序。

**libibverbs: Fatal: kernel ABI version X doesn't match library version Y**
 * 原因：可用的RDMA内核堆栈不被libibverbs支持(这就是ABI的错误含义)。
 * 来源：这通常发生在内核部分和libibverbs不是来自同一个源(i.e. OFED/inbox/built manually)时。
 * 解决办法：卸载当前的RDMA包，并安装一个新的OFED发行版或Linux发行版中的包。

**libibverbs: Warning: couldn't open config directory '/etc/libibverbs.d'**
  * 原因：libibverb未能打开保存已安装的用户空间低级驱动程序库信息的目录。
  * 来源：这通常是在使用与用户空间低级驱动程序库不同的参数（为“ configure”提供--sysconfdir）配置和编译libibverbs时发生。
  * 解决办法：卸载用户空间底层驱动程序和libibverb，并从一致的源安装它们，或者使用相同的参数重新编译所有库。

**libibverbs: Warning: fork()-safety requested but init failed**
* 原因：libibverbs尝试根据用户的请求在fork（）安全模式下工作，但失败了。
* 来源：这通常发生在旧的Linux内核中(比2.6.12更老)
* 解决办法：迁移到更新的Linux内核或禁用fork（）请求环境变量/动词。

**libibverbs: Warning: no userspace device-specific driver found**
* 原因：libibverb未能找到特定RDMA设备的用户空间低级驱动程序。
* 来源：缺少此RDMA设备的用户空间低级驱动程序。
* 解决办法：根据计算机中存在的HW安装丢失的低级驱动程序(lspci可能很方便)。

**libibverbs Warning: couldn't load driver**
* 原因：libibverb未能加载特定RDMA设备的用户空间低级驱动程序库。
* 来源：这通常发生在RDMA设备的用户空间低级驱动程序库(.so文件）丢失了，损坏了或者与libibverbs不一致(就支持的特性而言)。
* 解决办法：如果这个RDMA设备的用户空间底层驱动程序库丢失:安装它。如果已经安装了，卸载并从libibverb的相同源重新安装它。

**libibverbs: Warning: RLIMIT_MEMLOCK is 32768 bytes**
 * 原因：libibverb验证了正在运行的进程可以锁定的内存量，并检测到该值为32KB或更少。。
 * 来源：使用RDMA需要锁定系统内存。当创建完成队列、队列对、共享接收队列或内存区域时，低的可被锁定的内存量会导致失败。
 * 解决办法：增加可被任何进程锁定的内存量到一个更高的值(“unlimited”是首选)。