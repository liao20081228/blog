---
title: RDMA编程手册 第4章 libibverbs
tags: RDMA
---

------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>[《RDMA Aware Networks Programming User Manual v1.7》](https://www.mellanox.com/related-docs/prod_software/RDMA_Aware_Programming_user_manual.pdf)、[《Dotan Barak的RDMAmojo博客》](http://www.rdmamojo.com/)、《man手册v24.0》，我所做的工作就是更新、整理一部分内容，删除了一部分废弃的API，添加了一部分新的API。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

# 2 初始化
## 2.1 ibv_fork_init
**函数原型：**

```cpp
int ibv_fork_init(void)
```

**输入参数：** 无。

**输出参数：** 无。

**返回值：** 成功返回0，失败返回errno（表示错误原因）。

* EINVAL——调用函数太晚。
* ENOMEM——内存不足，无法完成操作。
* ENOSYS——没有对RDMA的内核支持。

**说明：**

在调用libibverbs中的任何其他函数之前，应该先调用ibv_fork_init()。

ibv_fork_init初始化libibverbs的数据结构以正确地处理fork()函数，并避免数据损坏，无论fork()是显式调用还是隐式调用，（例如在system(), popen()调用中）。

在RDMA技术中，RDMA设备知道虚拟地址到物理地址的映射。 当进程显式或隐式调用fork()函数时，由于Linux内核的“写时复制”（copy-on-write）策略，父进程和子进程虚拟页面都将映射到相同的物理内存页面。 当其中一个进程（第一个）写入这样的页面时，它将获得一个具有相同内容的新物理页面。 RDMA设备不知道对于该进程虚拟地址到物理地址的转换已经改变，因此可能会尝试访问旧的物理地址（该地址现在甚至可以由其他进程或内核使用）。 这可能导致数据损坏甚至更糟的情况——内核崩溃。

为了防止此问题的发生，libibverbs添加了fork()保护支持。 它指示Linux内核将原始物理页面保持映射到父进程，而不管两个进程中的哪个首先写入页面（注：也就是说始终应当使用父进程进行RDMA操作）。

libibverbs中的fork()支持并不完整，它假定只有父进程会执行RDMA操作。 如果一个子进程尝试执行RDMA操作，则可能会遇到各种问题。

如果所有父进程线程始终被阻塞直到所有子进程结束，或通过exec()操作更改地址空间，则不必调用ibv_fork_init。

如果在没有调用ibv_fork_init()的情况下就使用fork()，则可能会遇到数据损坏，段错误，缺失工作请求完成，或任何其他现象。

此函数在支持**madvise**()的**MADV_DONTFORK**标志的Linux内核上起作用（2.6.17及更高版本）。

将环境变量**RDMAV_FORK_SAFE**或**IBV_FORK_SAFE**设置为任何值都具有与调用ibv_fork_init()相同的效果（libibverb只检查是否定义了一个环境变量，并且完全忽略了该值）。

将设置环境变量**RDMAV_HUGEPAGES_SAFE**告诉该库检查内核用于内存区域的基础页面大小。如果应用程序直接或间接通过libhugetlbfs之类的库使用巨型页面，则需要这样做。

调用ibv_fork_init()会降低性能，这是因为每个内存注册都要进行额外的系统调用，以及分配给跟踪内存区域的额外内存。确切的性能影响取决于工作负载，通常不会很明显。

没有任何函数调用与设置环境变量**RDMAV_HUGEPAGES_SAFE**具有相同的效果。

设置**RDMAV_HUGEPAGES_SAFE**会增加所有内存注册的开销。

**常见问题：**

Q：如果我不启用fork()支持而仍然使用fork()，会发生什么?
A：可能会发生不好的事情，比如(但不限于)：数据损坏、丢失工作完成、段错误。

Q：调用ibv_fork_init()或设置环境变量RDMAV_FORK_SAFE有不同吗?
A：没有，完全等效。

Q：我将哪个值赋给环境变量RDMAV_FORK_SAFE是有关系的吗？
A：没有，任何值都可以。

Q：我使用的页面很大，并且只启用了RDMAV_FORK_SAFE，fork可以工作吗？
A：不能，应该将环境变量RDMAV_HUGEPAGES_SAFE设置为任何值。

Q：我不知道是否需要fork()支持，为了安全起见，是否可以启用fork()支持？
A：是的,你可以。只需注意，在启用fork支持时将消耗额外的内存。

Q：我没有显式调用fork()，仅调用了可能会调用fork()的其他系统调用，我是否仍需要启用fork()支持？
A：是的，您需要启用fork()支持，无论是直接调用fork()还是任何其他系统调用隐式调用fork()。

Q：我只向我的客户提供一个库，他调用fork()，仍然需要fork()支持吗？
A：是的，如果进程调用fork()，则需要启用fork()支持，无论调用它的人是谁都没关系。 如果用户有时会使用fork()，则可以设置环境变量RDMAV_FORK_SAFE。

Q：我调用了ibv fork init()，它失败了，这是什么意思呢？
A：这意味着您的内核不支持madvise()，您应该咨询支持或更新您的内核。

Q：调用fork()后能否继承RDMA资源？
A：libibverbs有几个文件说明符，在调用fork()之后，它们仍可在子进程中使用。 但是，强烈建议不要尝试使用它们，因为这可能会导致不良后果。

**示例：**

在bash中为fork()保护设置环境变量，以下所有行具有相同的效果:
```bash
# export RDMAV_FORK_SAFE
# export RDMAV_FORK_SAFE=0
# export RDMAV_FORK_SAFE=1
# export RDMAV_FORK_SAFE="no"
# export RDMAV_FORK_SAFE="yes"
# export IBV_FORK_SAFE=0
# export IBV_FORK_SAFE=1
# export IBV_FORK_SAFE="no"
```
在使用大页面时为fork()保护设置环境变量(在bash中)
```bash
# export RDMAV_FORK_SAFE
# export RDMAV_HUGEPAGES_SAFE
```
在C源代码中为fork()保护调用函数:

```cpp
int rc;

rc = ibv_fork_init()
if (rc)
        exit(rc);
```
# 3 获取和释放设备列表
以下命令用于一般设备操作，允许用户查询有关系统上设备的信息以及打开和关闭特定设备。

## 3.1 ibv_get_device_list

**函数原型：**

```cpp
struct ibv_device **ibv_get_device_list(int *num_devices)
```

**输入参数：** 无。

**输出参数：** num_devices——(可选)如果非空，则数组中返回的设备数量将存储在这里。

**返回值：** ibv_get_device_list()成功时返回可用的RDMA设备的数组，这个数组以NULL终止。如果失败，则返回NULL并设置errno显示错误原因。 如果未找到任何设备，则将num_devices设置为0，并返回non-NULL。可能的错误码是：

* EPERM——权限拒绝。
* ENOMEM——内存不足，无法完成操作。
* ENOSYS ——没有对RDMA的内核支持。

**说明：**

在调用libibverb中的任何其他函数之前，应该先调用ibv fork init()。

ibv_get_device_list()返回当前可用的RDMA设备的以NULL终止的数组。数组中每个条目都是指向struct ibv_device的指针。应使用ibv_free_device_list()释放该数组。

数组条目不应直接访问。 相反，它们应与以下服务动词一起使用：ibv_get_device_name()，ibv_get_device_guid()和ibv_open_device()。

在调用 ibv_free_device_list()之前，用户代码应该用ibv_open_device() 打开所有它打算的使用的设备。一旦使用 ibv_free_device_list()释放设备数组，则只有打开的设备才能使用，指向未打开设备的指针不再合法。

如果发现内核动词设备但未找到相应的用户空间驱动程序时，设置环境变量IBV_SHOW_WARNINGS将导致在向stderr发出警告。

ibv_device结构体的数组将保持有效，直到该数组被释放。 调用ibv_get_device_list之后，用户应打开任何所需的设备，并通过ibv_free_device_list命令迅速释放数组。

struct ibv_device定义为：
```cpp
struct ibv_device {
	struct	_ibv_device_ops			ops;							//设备操作函数
	enum	ibv_node_type			node_type;						//节点类型，一个枚举值，详细信息见下文
	enum	ibv_transport_type		transport_type;					//传输层类型，一个枚举值，详细信息见下文	
	char							name[IBV_SYSFS_NAME_MAX];		//内核设备名如“mthca0”
	char							dev_name[IBV_SYSFS_NAME_MAX];	//uverb设备名如“uverbs0”
	char							dev_path[IBV_SYSFS_PATH_MAX];	//向sysfs中infiniband_verb类设备的路径
	char							ibdev_path[IBV_SYSFS_PATH_MAX]; //在sysfs中到infiniband类设备的路径
};
```

```cpp
enum {
	IBV_SYSFS_NAME_MAX	= 64,
	IBV_SYSFS_PATH_MAX	= 256
};
 ```

struct \_ibv_device_ops定义如下：

```cpp
struct _ibv_device_ops {
	struct ibv_context	*(*_dummy1)(struct ibv_device *device, int cmd_fd);
	void				 (*_dummy2)(struct ibv_context *context);
};
```

enum ibv_node_type定义如下：

```cpp
enum ibv_node_type {
	IBV_NODE_UNKNOWN	= -1,	//未知类型
	IBV_NODE_CA			= 1,	//通道适配器
	IBV_NODE_SWITCH,			//交换机
	IBV_NODE_ROUTER,			//路由器
	IBV_NODE_RNIC,				//RDMA网卡
	IBV_NODE_USNIC,				//
	IBV_NODE_USNIC_UDP,			//
};
```

enum ibv_transport_type定义如下：
```cpp
enum ibv_transport_type {
	IBV_TRANSPORT_UNKNOWN	= -1,		//未知类型
	IBV_TRANSPORT_IB		= 0,		//IB
	IBV_TRANSPORT_IWARP,				//IWARP
	IBV_TRANSPORT_USNIC,
	IBV_TRANSPORT_USNIC_UDP,
};
```

**示例：** 参见ibv_query_port。

**常见问题：**

Q：我调用了ibv_get_device_list()，它返回NULL，这是什么意思?
A：这是一个不应该失败的基本动词，请检查是否加载了ib_uverb模块。

Q：我调用了ibv_get_device_list()，它根本没有找到任何RDMA设备(空列表)，这是什么意思?

A：驱动程序找不到任何RDMA设备。
* 如果您的机器中有任何RDMA设备，检查lspci。
* 使用lsmod检查RDMA设备的低级驱动程序是否已加载。
* 检查dmesg/var/log/messages是否有错误。

## 3.2 ibv_free_device_list
**函数原型：**

```cpp
void ibv_free_device_list(struct ibv_device **list)
```

**输入参数：** list——ibv_get_device_list()返回的RDMA设备数组。

**输出参数：** 无。

**返回值：** 无，表示此函数总是成功的。

**说明：** ibv_free_device_list释放由ibv_get_device_list提供的ibv_device结构体数组。 在调用此命令之前，应打开任何所需的设备。 释放驻足后，数组上的所有ibv_device结构体都将无效，无法再使用。

**示例：** 参见ibv_query_port。

# 4 基于设备的操作
## 4.1 获取设备信息
### 4.1.1 ibv_get_device_name

**函数原型：**
```cpp
const char *ibv_get_device_name(struct ibv_device *device)
```
**输入参数：** device——RDMA设备，由ibv_get_device_list()返回的struct ibv_device数组元素。


**输出参数：** 无

**返回值：** 成功返回指向的设备名称的字符串指针，失败返回NULL。

**说明：**

ibv_get_device_name返回指向ibv_device结构体中包含的设备名称的指针。名称由以下字段组成:
* 前缀——说明RDMA设备供应商和模型。
	* cxgb3——Chelsio通信，T3 RDMA系列
	* cxgb4——Chelsio 通信, T4 RDMA系列
	* ehca—— IBM, eHCA 系列
	* ipathverbs——QLogic
	* mlx4——Mellanox 技术，ConnectX系列
	* mthca——Mellanox 技术，InfiniHost 系列
	* nes——Intel, Intel-NE系列
* 索引——一个数字，有助于区分同一台计算机中同一供应商和系列的几种设备

此名称在特定机器内是唯一的(同一名称不能分配给多个设备)。

然而，这个名字在InfiniBand结构中并不是唯一的(这个名字可以在不同的机器中找到)。

当计算机中有多个RDMA设备时，更改计算机中设备的位置（即在PCI总线中）可能会导致与设备关联的名称发生更改。 为了区分设备，建议使用由ibv_get_device_guid()返回的设备GUID。

**示例：** 参见ibv_query_port。

### 4.1.2 ibv_get_device_guid

**函数原型：**
```cpp
__be64 ibv_get_device_guid(struct ibv_device *device)
```
**输入参数：** device——ibv_get_device_list()返回的struct ibv_device数组元素。

**输出参数：** 无。

**返回值：** 成功返回64位GUID。失败返回0。

**说明：**

ibv_get_device_guid以网络字节顺序返回RDMA设备的64位全局惟一标识符(GUID)。

这个GUID在制造过程中由其供应商分配给这个设备，它是唯一的，可以用作RDMA设备的标识符。

从RDMA设备GUID的前缀，可以使用[IEEE OUI](http://standards.ieee.org/develop/regauth/oui/oui.txt)查询谁是该设备的供应商。

**示例：** 参见ibv_query_port。

## 4.2 打开和关闭设备
### 4.2.1 ibv_open_device
**函数原型：**
```cpp
struct ibv_context *ibv_open_device(struct ibv_device *device)
```
**输入参数：** device——RMDA设备，由ibv_get_device_list()返回的struct ibv_device数组元素。

**输出参数：** 无。

**返回值：** 成功返回动词上下文，可用于设备上的未来操作，失败返回NULL。

**说明：**

ibv_open_device()为RDMA设备设备创建一个上下文。 该上下文稍后将用于查询其资源或用于创建资源，并且应使用ibv_close_device()释放。

与动词名称不同，它实际上并没有打开设备，该设备是由内核低级驱动程序打开的，并且可以由其他用户/内核级代码使用。 该动词仅打开上下文，以允许用户级别的应用程序使用它。

设置环境变量RDMAV_ALLOW_DISASSOC_DESTROY可以使该库成功将EIO与destroy命令相关联，因为已经释放了内核资源。 这是为了防止在解除设备关联时用户空间区域中的内存泄漏。 当调用与对象销毁函数的同时，使用此标志的应用程序不能调用ibv_get_cq_event或ibv_get_async_event。

struct ibv_context定义如下：
```cpp
struct ibv_context {
	struct ibv_device		*device;			//RDMA设备结构体，详细信息见ibv_get_device_list
	struct ibv_context_ops	ops;				//设备上下文操作函数，详细信息见下文
	int						cmd_fd;	
	int						async_fd;			//用于(内部)读取异步事件的文件说明符。
	int						num_comp_vectors;	//此RDMA设备可用的完成向量(即MSI-X向量)的数量。
	pthread_mutex_t			mutex;				//锁
	void					*abi_compat;		//应用二进制接口兼容性
};
```

struct ibv_context_ops定义如下：
```cpp
struct ibv_context_ops {
	void *(*_compat_query_device)(void);//查询设备
	int		(*_compat_query_port)(struct ibv_context *context,uint8_t port_num,
				struct _compat_ibv_port_attr *port_attr);	//查询端口，port_attr详细信息见下文
	void *(*_compat_alloc_pd)(void);		//分配PD
	void *(*_compat_dealloc_pd)(void);		//释放PD
	void *(*_compat_reg_mr)(void);			//注册MR
	void *(*_compat_rereg_mr)(void);		//重注册MR
	void *(*_compat_dereg_mr)(void);		//注销MR
	
	struct ibv_mw *(*alloc_mw)(struct ibv_pd *pd, enum ibv_mw_type type);					//分配MW
	int  (*bind_mw)(struct ibv_qp *qp, struct ibv_mw *mw, struct ibv_mw_bind *mw_bind);	//绑定MW
	int  (*dealloc_mw)(struct ibv_mw *mw);													//释放MW
	
	void *(*_compat_create_cq)(void);												//创建CQ
	int		(*poll_cq)(struct ibv_cq *cq, int num_entries, struct ibv_wc *wc);	//轮询CQ
	int		(*req_notify_cq)(struct ibv_cq *cq, int solicited_only);				//通知CQ
	void *(*_compat_cq_event)(void);												//
	void *(*_compat_resize_cq)(void);												//修改CQ的容量
	void *(*_compat_destroy_cq)(void);												//销毁CQ
	
	void *(*_compat_create_srq)(void);		//创建SRQ
	void *(*_compat_modify_srq)(void);		//修改SRQ
	void *(*_compat_query_srq)(void);		//查询SRQ
	void *(*_compat_destroy_srq)(void);	//销毁SRQ
	int		(*post_srq_recv)(struct ibv_srq *srq, struct ibv_recv_wr *recv_wr,
								struct ibv_recv_wr **bad_recv_wr);		//发布RR到SRQ

	void *(*_compat_create_qp)(void);		//创建QP
	void *(*_compat_query_qp)(void);		//查询QP
	void *(*_compat_modify_qp)(void);		//修改QP
	void *(*_compat_destroy_qp)(void);		//销毁QP
	int		(*post_send)(struct ibv_qp *qp, struct ibv_send_wr *wr,
							struct ibv_send_wr **bad_wr);		//发布WR到SQ
	int		(*post_recv)(struct ibv_qp *qp, struct ibv_recv_wr *wr,
							struct ibv_recv_wr **bad_wr);		//发布WR到RQ
									
	void *(*_compat_create_ah)(void);		//创建AH
	void *(*_compat_destroy_ah)(void);		//销毁AH
	
	void *(*_compat_attach_mcast)(void);	//加入多播
	void *(*_compat_detach_mcast)(void);	//离开多播
	
	void *(*_compat_async_event)(void);
};

```
struct \_compat_ibv_port_attr 定义如下：
```cpp
struct _compat_ibv_port_attr {
	enum ibv_port_state		state;
	enum ibv_mtu				max_mtu;
	enum ibv_mtu				active_mtu;
	int							gid_tbl_len;
	uint32_t					port_cap_flags;
	uint32_t					max_msg_sz;
	uint32_t					bad_pkey_cntr;
	uint32_t					qkey_viol_cntr;
	uint16_t					pkey_tbl_len;
	uint16_t					lid;
	uint16_t					sm_lid;
	uint8_t						lmc;
	uint8_t						max_vl_num;
	uint8_t						sm_sl;
	uint8_t						subnet_timeout;
	uint8_t						init_type_reply;
	uint8_t						active_width;
	uint8_t						active_speed;
	uint8_t						phys_state;
	uint8_t						link_layer;
	uint8_t						flags;
};
```

**示例：** 参见ibv_query_port。

### 4.2.2 ibv_close_device

**函数原型：**
```cpp
int ibv_close_device(struct ibv_context *context)
```
**输入参数：** context——ibv_open_device()返回的RDMA设备上下文。


**输出参数：** 无

**返回值：** 成功返回0；出错返回-1。

**说明：**

ibv_close_device关闭先前使用ibv_open_device打开的动词上下文。

ibv_close_device()不会释放与此上下文关联的资源。 用户在调用该动词之前释放它们，以防止资源（例如内存，文件说明符，RDMA对象编号）泄漏。 使用这些孤立资源可能会导致段错误。

但是，当该进程结束时，这些资源将由操作系统自动清除。

**示例：** 见ipv_query_port。

# 5 基于设备上下文的操作
打开设备后，将使用以下命令。 这些命令使您可以获取有关设备或其端口之一的更多特定信息，创建完成队列（CQ），完成通道（CC）和保护域（PD），以用于进一步的操作。
## 5.1 查询设备的RDMA属性

###  5.1.1 ibv_query_device
**函数原型：**
```cpp
int ibv_query_device(struct ibv_context *context, struct ibv_device_attr *device_attr)
```
**输入参数：** context——RDMA设备上下文，由ibv_open_device()返回。

**输出参数：** device_attr——表示设备属性的。结构体详细信息见下文。

**返回值：** 成功返回0；失败返回errno指示失败的原因。ENOMEM，内存不足。

**说明：**

ibv_query_device检索与设备关联的各种属性。 用户应malloc一个struct ibv_device_attr，并将其传递给该命令，在成功返回后它将被填充。 ***用户负责释放此结构体***。

由ibv_query_device()返回的RDMA设备属性是恒定的，不会被设备或SM更改，因此程序可以调用此动词并将其保存以备后面使用。

ibv_query_device()返回的最大值是设备支持的资源的上限。 但是，可能无法使用这些最大值，因为可以创建的任何资源的实际数量可能会受到计算机配置，主机内存量，用户权限以及其他用户或进程已经在使用的资源量的限制。

struct ibv_device_attr的定义如下:

```cpp
struct ibv_device_attr{
	char					fw_ver[64];					//固件版本
	__be64					node_guid;					//节点全局唯一标识符(GUID)
	__be64					sys_image_guid;				//系统映像GUID
	uint64_t				max_mr_size;				//可以注册的最大的连续内存块
	uint64_t				page_size_cap;				//支持的页面大小
	uint32_t				vendor_id;					//供应商ID，根据IEEE
	uint32_t				vendor_part_id;				//供应商提供的部件ID
	uint32_t				hw_ver;						//硬件版本
	int						max_qp;						//队列对(QP)的最大数量
	int						max_qp_wr;					//任意队列未完成工作请求(WR)的最大数量
	unsigned int			device_cap_flags;			//HCA功能掩码，enum ibv_device_cap_flags的枚举值按位或得到
	int						max_sge;					//非RD QPs的每个WR的分散/聚合条目（SGE）的最大值
	int						max_sge_rd;					//RD QPs的每个WR的SGE
	int						max_cq;						//支持的CQ的最大值
	int						max_cqe;					//每个CQ的CQE的最大值
	int						max_mr;						//支持的MR的最大值
	int						max_pd;						//支持的保护域（PD）的最大值
	int						max_qp_rd_atom;				//每个QP未完成RDMA读和原子操作的最大值
	int						max_ee_rd_atom;				//每个端到端上下文(RD)的最大未完成RDMA读和原子操作
	int						max_res_rd_atom;			//用于即将来临的RDMA读和原子操作的最大资源值
	int						max_qp_init_rd_atom;		//每个QP可以启动的RDMA读和原子操作的最大值
	int						max_ee_init_rd_atom;		//每个EE可以启动的RDMA读和原子操作的最大值
	enum ibv_atomic_cap	atomic_cap;					//原子操作支持级别，详细信息见下文
	int						max_ee;						//支持的EE上下文的最大值
	int						max_rdd;					//支持的RD域的最大值
	int						max_mw;						//支持的内存窗口（MW）的最大值
	int						max_raw_ipv6_qp;			//支持的原始IPv6数据报QPs的最大值
	int						max_raw_ethy_qp;			//支持的以太网数据QPS的最大值
	int						max_mcast_grp;				//支持的多播组的最大数量
	int						max_mcast_qp_attach;		//每个多播组可以附加的QPs的最大值
	int						max_total_mcast_qp_attach;	//多播组可以附加的QPs的总和的最大数
	int						max_ah;						//支持的地址句柄（AH）的最大值
	int						max_fmr;					//支持的快速内存注册（FMR）的最大值
	int						max_map_per_fmr;			//需要unmap操作之前，每个FMR的最大重新映射数
	int						max_srq;					//支持的共享接收队列（SRQ）的最大值
	int						max_srq_wr;					//每个SRQ的WR的最大值
	int						max_srq_sge;				//每个SRQ的SGE的最大值
	uint16_t				max_pkeys;					//分区的最大数量
	uint8_t					local_ca_ack_delay;			//本地CA ack延迟
	uint8_t					phys_port_cnt;				//物理端口数
}

```

下面是对struct ibv_device_attr的完整说明：

|成员名|说明|
|:--|:--|
fw_ver|	NULL终止的字符串，说明RDMA设备的固件版本。|
|node_guid|	与RDMA设备关联的GUID（以网络字节顺序）。 这与ibv_get_device_guid()返回的GUID相同。|
|sys_image_guid|与此RDMA设备和作为单个系统一部分的其他设备相关联的GUID(按网络字节顺序)。例如:同一个核心交换机中有多个交换芯片。|
|max_mr_size|此设备可以注册的最大连续内存块的大小（以字节为单位）。|
|page_size_cap|	内存页大小。|
|vendor_id|IEEE分配的设备供应商ID。|
|vendor_part_id|Devic的部件ID，由供应商提供。|
|hw_ver	|设备的硬件版本，由供应商提供。|}
|max_qp|UD/UC/RC传输类型下，QP的最大数量。|
|max_qp_wr|任意发送或接收队列上的未完成工作请求的最大数量。|
|device_cap_flags|设备支持的功能，由enum ibv_device_cap_flags的枚举值按位或得到，见下面详细叙述。|
|max_sge|除RD之外，在QP中，每个发送或接收工作请求的最大分散/收集条目数|
|max_sge_rd|在RD QP中，每个发送或接收工作请求的最大分散/收集条目数，如果不支持RD，则值为0。|	
|max_cq	|CQ的最大数量。|
|max_cqe|每个CQ的CQE的最大数量。|
|max_mr	|MR的最大数量|
|max_pd|PD的最大数量|
|max_qp_rd_atom|以此设备为目标的每个QP中的未完成的RDMA读取和原子操作的最大数量（如果支持的话）|
|max_ee_rd_atom|以此设备为目标的每个EEC中的未完成的RDMA读取和原子操作的最大数量（如果支持的话）|
|max_res_rd_atom|以此设备为目标的，用于RDMA读取和原子操作的资源的最大总数（如果支持的话）|
|max_qp_init_rd_atom|每个QP用于启动RDMA读和原子操作的的最大深度(如果支持的话)|
|max_ee_init_rd_atom|每个EEC用于启动RDMA读和原子操作的的最大深度(如果支持的话)|
|atomic_cap	|设备支持的原子操作级别。值为enum ibv_atomic_cap中定义的枚举值之一。见下面详细叙述|
|max_ee|EE上下文的最大数量，如果此设备不支持RD，则值为0 |
|max_rdd|RDD的最大数量，如果此设备不支持RD，则值为0 |
|max_mw|MW的最大数量。如果此设备不支持MW，则值为0.|
|max_raw_ipv6_qp	|原始IPv6数据报QP的最大数量。如果该设备不支持原始IPv6数据报QP，则值为0。
|max_raw_ethy_qp|原始以太网数据报QP的最大数量。如果该设备不支持原始原始以太网数据报QP，则值为0。|
|max_mcast_grp|多播组的最大数量。如果此设备不支持不可靠的多播，则值为0。|
|max_mcast_qp_attach|每个组播组的可添加的QP的最大数量。 如果此设备不支持不可靠的多播，则值为0。|
|max_total_mcast_qp_attach	|可添加到此设备的多播组的QP的最大总数。如果此设备不支持不可靠的多播，则此值为0。|
|max_ah|AH的最大数量
|max_fmr|FMR的最大数量。如果该设备不支持FMR，则该值为0。|
|max_map_per_fmr|每个FMR映射的最大数量。如果该设备不支持FMR，则该值为0。|
|max_srq|SRQ的最大数量。 如果此设备不支持SRQ，则该值为0|
|max_srq_wr	|SRQ中未完成工作请求的最大数量|
|max_srq_sge|在一个SRQ中，每个接收工作请求的最大散点条目数|
|max_pkeys|	分区的最大数量|
|local_ca_ack_delay|本地CA ACK延迟。此值指定本地设备接收到消息与传输关联的ACK或NAK之间的最大预期时间间隔。
|phys_port_cnt|此设备上的物理端口数|

enum ibv_device_cap_flags定义如下：

```cpp
enum ibv_device_cap_flags {
	IBV_DEVICE_RESIZE_MAX_WR			= 1,			//设备支持修改QP的未完成工作请求的最大数量
	IBV_DEVICE_BAD_PKEY_CNTR			= 1 <<  1,	//设备支持每个端口的错误P_Key计数
	IBV_DEVICE_BAD_QKEY_CNTR			= 1 <<  2,	//设备支持每个端口的无效Q_Key计数
	IBV_DEVICE_RAW_MULTI				= 1 <<  3,	//设备支持原始数据包多播
	IBV_DEVICE_AUTO_PATH_MIG			= 1 <<  4,	//设备支持自动路径迁移
	IBV_DEVICE_CHANGE_PHY_PORT			= 1 <<  5,	//设备支持从SQD状态转换到SQD状态时更改QP的主端口号
	IBV_DEVICE_UD_AV_PORT_ENFORCE		= 1 <<  6,	//设备支持 强制使用AH端口号
	IBV_DEVICE_CURR_QP_STATE_MOD		= 1 <<  7,	//设备支持调用ibv_modify_qp()修改当前QP状态
	IBV_DEVICE_SHUTDOWN_PORT			= 1 <<  8,	//设备支持关闭端口
	IBV_DEVICE_INIT_TYPE				= 1 <<  9,	//设备支持 设置 InitType 和 InitTypeReply
	IBV_DEVICE_PORT_ACTIVE_EVENT		= 1 << 10,	//设备支持 产生IBV_EVENT_PORT_ACTIVE事件
	IBV_DEVICE_SYS_IMAGE_GUID			= 1 << 11,	//设备支持 System Image GUID
	IBV_DEVICE_RC_RNR_NAK_GEN			= 1 << 12,	//设备支持 为RC QPs生成RNR-NAK
	IBV_DEVICE_SRQ_RESIZE				= 1 << 13,	//设备支持修改SRQ的未完成工作请求的最大数量
	IBV_DEVICE_N_NOTIFY_CQ				= 1 << 14,	//设备支持当向CQ添加N个完成(而不是一个)时生成一个请求完成通知
	IBV_DEVICE_MEM_WINDOW				= 1 << 17,	//
	IBV_DEVICE_UD_IP_CSUM				= 1 << 18,	//
	IBV_DEVICE_XRC						= 1 << 20,	//
	IBV_DEVICE_MEM_MGT_EXTENSIONS		= 1 << 21,	//
	IBV_DEVICE_MEM_WINDOW_TYPE_2A		= 1 << 23,	//
	IBV_DEVICE_MEM_WINDOW_TYPE_2B		= 1 << 24,	//
	IBV_DEVICE_RC_IP_CSUM				= 1 << 25,	//
	IBV_DEVICE_RAW_IP_CSUM				= 1 << 26,	//
	IBV_DEVICE_MANAGED_FLOW_STEERING	= 1 << 29//
};
```

enum ibv_atomic_cap定义如下：

```cpp
enum ibv_atomic_cap {
	IBV_ATOMIC_NONE,	//根本不支持原子操作
	IBV_ATOMIC_HCA,		//仅在此设备上的QP之间保证原子性
	IBV_ATOMIC_GLOB		//保证此设备与任何其他组件（例如CPU，IO设备和其他RDMA设备）之间的原子性
};
```

**常见问题：**

Q：ibv_query_device()显示我可以从资源创建X个元素，但是我只能创建Y（其中Y小于X），这对吗？
A：是的，可能会发生这种情况，因为ibv_query_device()报告的值是设备支持的上限。 如果任何其他进程/模块也从该资源创建了元素，则该资源的可用数量将减少。

Q：我可以知道一个资源有多少个元素可以创建吗？
A：不能。这个值取决于很多因素。

Q：我可以知道资源的多少元素是由其他进程/模块创建的吗？
A：不能。 当前RDMA堆栈不支持此功能。


### 5.1.2 ibv_query_device_ex

**函数原型：**
```cpp
 int ibv_query_device_ex(struct ibv_context *context, const struct ibv_query_device_ex_input *input,
							struct ibv_device_attr_ex *attr);
```

**输入参数：**

* context——RDMA设备上下文，由ibv_open_device()返回。
* input——查询属性的掩码，是一个可扩展的输入结构，用于将来可能的扩展。当前无效。

**输出参数：** attr——RDMA设备的扩展属性。结构体详细信息见下文。

**返回值：** 成功返回0；失败返回errno，指示失败的原因。

**说明：**

ibv_query_device_ex()获取设备的属性，用户应分配一个 struct ibv_device_attr_ex ，并传入函数，他将在成功调用时被填充。***用户负责释放此结构体***。

该函数返回的最大值是设备支持的资源上限。 但是，可能无法使用这些最大值，因为可以创建的任何资源的实际数量可能会受到计算机配置，主机内存量，用户权限以及已被其他用户/进程使用的资源量的限制。

 struct ibv_query_device_ex_input 定义如下：
 ```cpp
struct ibv_query_device_ex_input {
	uint32_t	comp_mask;				//可扩展的输入结构，用于将来可能的扩展
};
```

 struct ibv_device_attr_ex 定义如下：
 ```cpp
struct ibv_device_attr_ex {
	struct ibv_device_attr			orig_attr;					//设备属性，结构体详细信息见 ibv_query_device
	uint32_t						comp_mask;					//兼容性掩码，定义以下哪些字段有效，当前未定义
	struct ibv_odp_caps			odp_caps;					//按需分页功能
	uint64_t						completion_timestamp_mask;	//完成时间戳掩码，0 =不支持
	uint64_t						hca_core_clock;				//HCA的频率，以kHZ为单位，0 =不支持
	uint64_t						device_cap_flags_ex;		//扩展设备功能标志，当前未定义
	struct ibv_tso_caps			tso_caps;					//TCP 分段卸载功能
	struct ibv_rss_caps			rss_caps;					//RSS功能
	uint32_t						max_wq_type_rq;				//RQ类型的最大工作队列
	struct ibv_packet_pacing_caps	packet_pacing_caps;			//packet pacing capabilities
	uint32_t						raw_packet_caps;			//原始数据包功能，使用枚举ibv_raw_packet_caps
	struct ibv_tm_caps				tm_caps;					//标签匹配功能
	struct ibv_cq_moderation_caps	cq_mod_caps;				//CQ moderation max capabilities
	uint64_t						max_dm_size;				//可分配的最大设备内存大小，以字节为单位
	struct ibv_pci_atomic_cap		spci_atomic_caps;			//PCI原子操作功能，使用枚举ibv_pci_atomic_op_size
	uint32_t						xrc_odp_caps;				//支持的功能，枚举ibv_odp_transport_cap_bits按位或
};
```
 struct ibv_odp_caps定义如下:

```cpp
struct ibv_odp_caps {
	uint64_t general_odp_caps;			//一般的按需分页功能，使用枚举ibv_odp_general_caps
	struct {
		uint32_t rc_odp_caps;			//RC的按需分页功能，使用枚举 ibv_odp_transport_cap_bits
		uint32_t uc_odp_caps;			//UC的按需分页功能，使用枚举 ibv_odp_transport_cap_bits
		uint32_t ud_odp_caps;			//UD的按需分页功能，使用枚举 ibv_odp_transport_cap_bits
	} per_transport_caps;
};
```
enum ibv_odp_general_cap_bits定义如下：
```cpp
enum ibv_odp_general_caps {
	IBV_ODP_SUPPORT				= 1 << 0,	/* 支持按需分页功能 */
	IBV_ODP_SUPPORT_IMPLICIT	= 1 << 1,	/* 支持隐式的按需分页功能 */
};
```

enum ibv_odp_transport_cap_bits定义如下：
```cpp
enum ibv_odp_transport_cap_bits {
	IBV_ODP_SUPPORT_SEND		= 1 << 0,	/* Send操作支持按需分页 */
	IBV_ODP_SUPPORT_RECV		= 1 << 1,	/* Receive操作支持按需分页*/
	IBV_ODP_SUPPORT_WRITE		= 1 << 2,	/* RDMA-Write 操作支持按需分页 */
	IBV_ODP_SUPPORT_READ		= 1 << 3,	/* RDMA-Read 操作支持按需分页 */
	IBV_ODP_SUPPORT_ATOMIC		= 1 << 4,	/* RDMA-Atomic操作支持按需分页paging */
	IBV_ODP_SUPPORT_SRQ_RECV	= 1 << 5,	/* SRQ receive 操作支持按需分页*/
};

```
struct ibv_tso_caps定义如下：
```cpp
struct ibv_tso_caps {
	uint32_t max_tso;			/* TSO引擎的分段的最大负载（以字节为单位）*/
	uint32_t supported_qpts;	/* 位图显示哪些QP类型支持TOS */
};

```
 struct ibv_rss_caps定义如下：
```cpp
struct ibv_rss_caps {
	uint32_t supported_qpts;					/* 位图显示哪些QP类型支持RSS */
	uint32_t max_rwq_indirection_tables;		/* 最大接收工作队列间接表* / */
	uint32_t max_rwq_indirection_table_size;	/* 最大接收工作队列间接表大小 */
	uint64_t rx_hash_fields_mask;				/* 哪个输入数据包的字段可以参与RX哈希，
													使用枚举ibv_rx_hash_fields,
													详细信息见ibv_create_qp_ex */
	uint8_t  rx_hash_function;					/* 支持哪些hash函数，
													使用枚举ibv_rx_hash_function_flags，
													详细信息见ibv_create_qp_ex */
};
```

struct ibv_packet_pacing_caps 定义如下：


```cpp
struct ibv_packet_pacing_caps {
	uint32_t qp_rate_limit_min;	/* 最小速率限制， kbps */
	uint32_t qp_rate_limit_max;	/* 最大速率限制， kbps */
	uint32_t supported_qpts;		/* 按位或，显示支持哪些QP类型。 */
};

```
enum ibv_raw_packet_caps定义如下：
```cpp
enum ibv_raw_packet_caps {
	IBV_RAW_PACKET_CAP_CVLAN_STRIPPING	= 1 << 0,	/* 支持 CVLAN stripping  */
	IBV_RAW_PACKET_CAP_SCATTER_FCS		= 1 << 1,	/* 支持FCS scattering */
	IBV_RAW_PACKET_CAP_IP_CSUM			= 1 << 2,	/* 支持IP CSUM offload */
	IBV_RAW_PACKET_CAP_DELAY_DROP		= 1 << 3,
};
```

 struct ibv_tm_caps 定义如下：

```cpp
 struct ibv_tm_caps {
	uint32_t	max_rndv_hdr_size;	/* 集合请求标头的最大大小 */
	uint32_t	max_num_tags;		/* TM-SRQ匹配列表中标记缓冲区的最大数量 */
	uint32_t	flags;				/* 来自枚举ibv_tm_cap_flags */
	uint32_t	max_ops;			/* 未完成列表操作的最大数量 */
	uint32_t	max_sge;			/* 标记缓冲区中SGE的最大数量 */
};
```
enum ibv_tm_cap_flags 定义如下：

```cpp
enum ibv_tm_cap_flags {
	IBV_TM_CAP_RC   = 1 << 0,	/* 支持RC传输上的标签匹配 */
};
```

struct ibv_cq_moderation_caps定义如下：
```cpp
struct	ibv_cq_moderation_caps {
	uint16_t	max_cq_count;
	uint16_t	max_cq_period; //usec
};
```

 struct ibv_pci_atomic_caps定义如下：
```cpp
struct ibv_pci_atomic_caps {
	uint16_t fetch_add;		/* 原子获取和添加操作支持的大小，使用enum ibv_pci_atomic_op_size */
	uint16_t swap;				/* 原子无条件交换操作支持的大小，使用enum ibv_pci_atomic_op_size  */
	uint16_t compare_swap;		/* 原子比较和交换操作支持的大小，使用enum ibv_pci_atomic_op_size */
};
```

enum ibv_pci_atomic_op_size定义如下：
```cpp
enum ibv_pci_atomic_op_size {
	IBV_PCI_ATOMIC_OPERATION_4_BYTE_SIZE_SUP	= 1 << 0,
	IBV_PCI_ATOMIC_OPERATION_8_BYTE_SIZE_SUP	= 1 << 1,
	IBV_PCI_ATOMIC_OPERATION_16_BYTE_SIZE_SUP	= 1 << 2,
};
```

扩展设备功能标志（device_cap_flags_ex）：
```cpp
#define IBV_DEVICE_PCI_WRITE_END_PADDING (1ULL << 36)
```

表示设备支持将PCI写填充到完整的缓存行。

将数据包填充到完整的高速缓存行可以减少内存控制器所需的流量，但要在PCI-E端口上创建更多流量。

具有高CPU内存负载和低PCI-E利用率的工作负载将受益于此功能，而具有高PCI-E利用率和小数据包的工作负载将受到损害。

例如，对于128字节的缓存行大小，任何小于128字节的数据包的传输都将需要在PCI上进行完整的128传输，从而可能使所需的PCI-E带宽增加一倍。

可以通过IBV_QP_CREATE_PCI_WRITE_END_PADDING或IBV_WQ_FLAGS_PCI_WRITE_END_PADDING标志以QP或WQ方式启用此功能。

### 5.1.3 ibv_query_port
**函数原型：**
```cpp
int ibv_query_port(struct ibv_context *context, uint8_t port_num, struct ibv_port_attr *port_attr)
```
**输入参数：**

* context——RDMA设备上下文，由ibv_open_device()返回。
* port_num——物理端口号，1是第一个端口。

**输出参数：** port_attr——表示端口属性的。结构体详细信息见下文。

**返回值：** 成功返回0；出错返回errno，指示失败的原因。

* EINVAL——port_num无效
* ENOMEM——内存不足

**说明：**

ibv_query_port检索与端口关联的各种属性。 用户应分配一个struct ibv_port_attr，并将其传递给命令，在成功返回后它将被填充。 ***用户负责释放此结构体***。

bv_query_port()返回的大多数端口属性不是恒定的，可能会更改，主要是通过SM（在InfiniBand中）或由硬件更改。 强烈建议避免保存此查询的结果，或在新的SM（重新）配置子网时刷新它们。

struct ibv_port_attr定义如下：

```cpp
struct ibv_port_attr
{
	enum ibv_port_state	state;				//逻辑端口状态， enum ibv_port_state枚举值之一
	enum ibv_mtu			max_mtu;			//端口支持的最大传输单元(MTU)。			
	enum ibv_mtu			active_mtu;			//实际使用的MTU
	int						gid_tbl_len;		//源全局ID (GID)表的长度
	uint32_t				port_cap_flags;		//该端口支持的功能，enum ibv_port_cap_flags中按位或
	uint32_t				max_msg_sz;			//最大消息大小
	uint32_t				bad_pkey_cntr;		//bad p_key计数器
	uint32_t				qkey_viol_cntr;		//Q_Key无效计数器
	uint16_t				pkey_tbl_len;		//分区表长度
	uint16_t				lid;				//分配给该端口的第一个本地标识符(LID)
	uint16_t				sm_lid;				//子网管理器的LID
	uint8_t					lmc;				//LID掩码控制（当多个LID被赋给端口时使用）
	uint8_t					max_vl_num;			//虚拟通道（VL）最大值
	uint8_t					sm_sl;				//SM服务级别（SL）。没找到定义。
	uint8_t					subnet_timeout;		//子网传播延迟
	uint8_t					init_type_reply;	//SM执行的初始化类型
	uint8_t					active_width;		//当前活动链接宽度
	uint8_t					active_speed;		//当前活动链接速度
	uint8_t					phys_state;			//物理端口状态。没找到定义
	uint8_t					link_layer;			//端口的链路层协议，使用匿名枚举值
	uint8_t					flags;				//端口标志，没找到定义
	uint16_t				port_cap_flags2;	//端口功能，enum ibv_port_cap_flags2中按位或
};
````

下面是struct ibv_port_attr的完整说明：

|成员名|协议|说明|
|:--|:--|:--|
|state|IB/RoCE/iWARP|逻辑端口状态，它是 enum ibv_port_state枚举值之一，见下面详述|
|max_mtu|IB/RoCE/iWARP|端口支持的MTU的最大值。它是 enum ibv_mtu枚举值之一，见下面详述|
|active_mtu|IB/RoCE/iWARP|此端口上允许发送和接收的MTU的真实最大值。 它可以是上述max_mtu指定的枚举值之一。 此值指定可以在UC/RC QP中配置的最大消息大小以及UD QP可以传输的最大消息大小|
|gid_tbl_len|IB/RoCE/iWARP|此端口的源GID表的长度|
|port_cap_flags|IB/RoCE/iWARP|端口支持功能， 其值enum ibv_port_cap_flags中的枚举值按位或。见下面详述|
|max_msg_sz|IB/RoCE/iWARP	|此端口支持的最大消息大小|
|bad_pkey_cntr|IB/RoCE|端口的错误P_Key计数器（如果设备支持）（在dev_cap.device_cap_flags中设置了IBV_DEVICE_BAD_PKEY_CNTR）|
|qkey_viol_cntr|IB/RoCE|端口的无效P_Key计数器（如果设备支持）（在dev_cap.device_cap_flags中设置了IBV_DEVICE_BAD_QKEY_CNTR ）|
|pkey_tbl_len|IB/RoCE/iWARP|该端口分区表的长度|
|lid|IB|此端口的基本LID，仅在状态为IBV_PORT_ARMED或IBV_PORT_ACTIVE时有效|
|sm_lid|IB|管理这个端口的SM的LID|
|lmc|IB|此端口的端口LID掩码（用于多路径），仅在状态为IBV_PORT_ARMED或IBV_PORT_ACTIVE时有效|
|max_vl_num|IB|此端口支持的最大虚拟连接数。该值不是枚举值，数值见下面详述|
|sm_sl|IB|管理这个端口的SM的服务级别|
|subnet_timeout|IB|指定期望的到达子网中的任何其他端口的最大子网传播延迟，该延迟取决于交换机的配置，并且还用于确定SubnTrap()从该端口可以发送的最大速率。持续时间的计算基于：4.096\*2^subnet_timeout^
|init_type_reply|IB|如果设备支持，则在将端口状态更改为IBV_PORT_ACTIVE或IBV_PORT_ARMED状态之前由SM配置的值，表示执行的初始化类型（在dev_cap.device_cap_flags中设置了IBV_DEVICE_INIT_TYPE）|
|active_width|	IB/RoCE/iWARP|该端口的激活链路宽度。 此值不是枚举值，数值见下面详述|
|active_speed|IB/RoCE/iWARP|该端口的激活链路速度。 此值不是枚举值，数值见下面详述|
|phys_state|IB|物理链接状态。该值不是枚举值，数值见下面详述|
|link_layer|IB/RoCE/iWARP|该端口使用的链路层协议。此值是匿名的枚举，见下面详述|
|flags||端口标志
|port_cap_flags2||端口支持功能 。 其值enum ibv_port_cap_flags2中的枚举值按位或。见下面详述

 enum ibv_port_state定义如下：
```cpp
enum ibv_port_state {
	IBV_PORT_NOP				= 0,	//保留
	IBV_PORT_DOWN				= 1,	//逻辑链路中断
	IBV_PORT_INIT				= 2,	//逻辑链接正在初始化
	IBV_PORT_ARMED				= 3,	//逻辑链接已经装备
	IBV_PORT_ACTIVE				= 4,	//逻辑链接是激活的
	IBV_PORT_ACTIVE_DEFER		= 5	//逻辑链接处于“激活延期”状态
};
```

下面是 enum ibv_port_state的完整说明：

|枚举值|说明|
|:--|:--|
|  IBV_PORT_NOP|保留值，不应该被观察到|
|  IBV_PORT_DOWN |逻辑链路中断。该端口的物理链接没有建立。在这种状态下，链路层丢弃所有呈现给它的数据包|
|IBV_PORT_INIT|逻辑链接正在初始化。 端口的物理链路已建立，但是SM尚未配置逻辑链路。 在这种状态下，链路层只能接收和发送SMP和流控制链路数据包，呈现给它的其他类型的数据包将被丢弃|
|IBV_PORT_ARMED|逻辑链接已经装备。 端口的物理链路已建立，但SM尚未完全配置逻辑链路。 在这种状态下，链路层可以接收和发送SMP和流控制链路数据包，可以接收其他类型的数据包，但会丢弃用于发送的非SMP数据包|
| IBV_PORT_ACTIVE|逻辑链接是激活的。链路层可以传输和接收所有的数据包类型|
|IBV_PORT_ACTIVE_DEFER |逻辑链接处于“激活延期”状态。 逻辑链接为激活的，但物理链接出现故障。 如果错误将在超时前恢复，则逻辑链接将返回到IBV_PORT_ACTIVE，否则它将移动到IBV_PORT_DOWN|

 enum ibv_mtu定义如下：

```cpp
enum ibv_mtu {
	IBV_MTU_256		= 1,	//MTU 是256 bytes
	IBV_MTU_512		= 2,	//MTU 是512 bytes
	IBV_MTU_1024	= 3,	//MTU 是1024 bytes
	IBV_MTU_2048	= 4,	//MTU 是2048 bytes
	IBV_MTU_4096	= 5	//MTU 是4096 bytes
};
```

enum ibv_port_cap_flags定义如下：
```cpp
enum ibv_port_cap_flags {
	IBV_PORT_SM							= 1 <<  1,	//管理子网的SM从该端口发送数据包
	IBV_PORT_NOTICE_SUP					= 1 <<  2,
	IBV_PORT_TRAP_SUP					= 1 <<  3,
	IBV_PORT_OPT_IPD_SUP				= 1 <<  4,
	IBV_PORT_AUTO_MIGR_SUP				= 1 <<  5,
	IBV_PORT_SL_MAP_SUP					= 1 <<  6,
	IBV_PORT_MKEY_NVRAM					= 1 <<  7,
	IBV_PORT_PKEY_NVRAM					= 1 <<  8,
	IBV_PORT_LED_INFO_SUP				= 1 <<  9,
	IBV_PORT_SYS_IMAGE_GUID_SUP			= 1 << 11,
	IBV_PORT_PKEY_SW_EXT_PORT_TRAP_SUP	= 1 << 12,
	IBV_PORT_EXTENDED_SPEEDS_SUP		= 1 << 14,
	IBV_PORT_CAP_MASK2_SUP				= 1 << 15,
	IBV_PORT_CM_SUP						= 1 << 16,
	IBV_PORT_SNMP_TUNNEL_SUP			= 1 << 17,	//表示SNMP隧道代理正在监听此端口
	IBV_PORT_REINIT_SUP					= 1 << 18,
	IBV_PORT_DEVICE_MGMT_SUP			= 1 << 19,
	IBV_PORT_VENDOR_CLASS_SUP			= 1 << 20,	//供应商的指定代理正在监听此端口
	IBV_PORT_DR_NOTICE_SUP				= 1 << 21,
	IBV_PORT_CAP_MASK_NOTICE_SUP		= 1 << 22,
	IBV_PORT_BOOT_MGMT_SUP				= 1 << 23,
	IBV_PORT_LINK_LATENCY_SUP			= 1 << 24,
	IBV_PORT_CLIENT_REG_SUP				= 1 << 25,	//这个端口能够生成IBV_EVENT_CLIENT_REREGISTER异步事件
	IBV_PORT_IP_BASED_GIDS				= 1 << 26
};
```

max_vl_num的完整说明如下：

|值|说明|
|:--|:--|
|1|1VL，VL~0~|
|2|2VL,  VL~0~,VL~1~|
|3|4VL,   VL~0~-VL~3~|
|4|8VL,   VL~0~-VL~7~|
|5|15VL,  VL~0~-VL~14~|

active_width的完整说明如下：

|值|说明|
|:--|:--|
|1|1x|
|2|4x|
|4|8x|
|8|12x|

active_speed的完整说明如下：

|值|说明|
|:--|:--|
|1|2.5Gbps|
|2|5Gbps|
|4|105Gbps|
|8|10Gbps|
|16|14Gbps|
|32|32Gbps|

phys_state的完整说明如下：

|值|说明|
|:--|:--|
|1|sleep：端口将其输出驱动到静态电平，并且不响应接收到的数据。 在这种状态下，链路会在不关闭端口电源的情况下 变为不激活|
|2|polling：端口发送训练序列，并响应接收训练序列。|
|3|Disabled：端口将其输出驱动到静态电平，并且不响应接收数据|
|4|PortConfigurationTraining：发送器和接收器均处于激活状态，并且端口正在尝试配置并转变为LinkUp状态
|5|LinkUp：该端口可用于发送和接收数据包|
|6|LinkErrorRecovery：端口尝试重新同步链接并使它恢复正常操作
|7|Phytest：端口允许发送器和接收电路由外部测试设备进行测试，以符合发射器和接收器的规范|


link_layer的定义如下：

```cpp
enum {
	IBV_LINK_LAYER_UNSPECIFIED,		//遗留值，用于表示链路层协议是InfiniBand
	IBV_LINK_LAYER_INFINIBAND,		//链路层协议是InfiniBand
	IBV_LINK_LAYER_ETHERNET,		//链路层协议是以太网，因此可以使用RoCE
};
```


下面是flags的取值的完整说明：

|值|说明|
|:--|:--|
|IBV_QPF_GRH_REQUIRED|设置此标志后，应用程序必须用创建配置的GRH创建所有的AH|


 enum ibv_port_cap_flags2定义如下：

```cpp
enum ibv_port_cap_flags2 {
	IBV_PORT_SET_NODE_DESC_SUP				= 1 << 0,
	IBV_PORT_INFO_EXT_SUP					= 1 << 1,
	IBV_PORT_VIRT_SUP						= 1 << 2,
	IBV_PORT_SWITCH_PORT_STATE_TABLE_SUP	= 1 << 3,
	IBV_PORT_LINK_WIDTH_2X_SUP				= 1 << 4,
	IBV_PORT_LINK_SPEED_HDR_SUP				= 1 << 5,
};
```

**示例：**

```cpp
#include <stdio.h>
#include <infiniband/verbs.h>

int main(void)
{
	struct ibv_context *ctx;
	struct ibv_device **device_list;
	int num_devices;
	int i;
	int rc;

	device_list = ibv_get_device_list(&num_devices);
	if (!device_list) {
		fprintf(stderr, "Error, ibv_get_device_list() failed\n");
		return -1;
	}

	printf("%d RDMA device(s) found:\n\n", num_devices);

	for (i = 0; i < num_devices; ++ i) {
		struct ibv_device_attr device_attr;
		int j;

		ctx = ibv_open_device(device_list[i]);
		if (!ctx) {
			fprintf(stderr, "Error, failed to open the device '%s'\n",
				ibv_get_device_name(device_list[i]));
			rc = -1;
			goto out;
		}

		rc = ibv_query_device(ctx, &device_attr);
		if (rc) {
			fprintf(stderr, "Error, failed to query the device '%s' attributes\n",
				ibv_get_device_name(device_list[i]));
			goto out_device;
}

		printf("The device '%s' has %d port(s)\n", ibv_get_device_name(ctx->device),
				device_attr.phys_port_cnt);

		for (j = 1; j <= device_attr.phys_port_cnt; ++j) {
			struct ibv_port_attr port_attr;

			rc = ibv_query_port(ctx, j, &port_attr);
			if (rc) {
				fprintf(stderr, "Error, failed to query port %d attributes in device '%s'\n",
					j, ibv_get_device_name(ctx->device));
				goto out_device;
			}

			printf("RDMA device %s, port %d state %s active\n", ibv_get_device_name(ctx->device), j,
				(port_attr.state == IBV_PORT_ACTIVE) ? "is" : "isn't");
		}

		rc = ibv_close_device(ctx);
		if (rc) {
			fprintf(stderr, "Error, failed to close the device '%s'\n",
				ibv_get_device_name(ctx->device));
			goto out;
		}
	}
		
	ibv_free_device_list(device_list);

	return 0;

out_device:
	ibv_close_device(ctx);

out:
	ibv_free_device_list(device_list);
	return rc;
}
```
**常见问题：**

Q：我正在使用iWarp / RoCE，是否需要ibv_query_port()返回的所有值？
A:  不。检查协议列，以了解哪些属性与您相关。

Q：我使用的是IB，是否需要ibv_query_port()返回的所有值？
A：不。有些字段您会经常使用（例如state），某些字段可能在调试问题时使用（计数器），而某些字段对于其他服务很有帮助。

Q：当我需要端口属性时，每次调用ibv_query_port()都会花费一些时间，我可以缓存某些属性吗？
A：	是。 表示端口支持的属性（例如，支持的表长度和功能）的属性不会更改，但是由SM配置的其他属性可能会更改，例如state和计数器。如果需要，可以在InfiniBand中缓存返回的结构，并仅在发生无关的异步事件时查询它（稍后将在章节讨论）。


### 5.1.4 ibv_query_gid
**函数原型：**
```cpp
int ibv_query_gid(struct ibv_context *context, uint8_t port_num, int index, union ibv_gid *gid)
```
**输入参数：**

* context——RDMA设备上下文，由ibv_open_device()返回。
* port_num——物理端口号，1是第一个端口。
* index——返回GID表中的哪一项（0是第一个）

**输出参数：** gid——包含gid信息的union ibv_gid。联合体详细信息见下文。

**返回值：** 成功返回0；失败返回errno指示失败的原因。EMFILE	，该进程打开太多文件。

**说明：**

ibv_query_gid在端口的全局标识符（GID）表中检索条目。 子网管理器（SM）为每个端口分配至少一个GID。 GID是有效的IPv6地址，由全球唯一标识符（GUID）和SM分配的前缀组成。 GID [0]是唯一的，并且包含端口的GUID，它是RDMA设备制造商的卖方提供的恒定值。

仅当port_attr.state为IBV_PORT_ARMED或IBV_PORT_ACTIVE时，GID表的内容才有效。 对于端口的其他状态，GID表的值不确定。
用户应分配一个union ibv_gid，并将其传递给命令，在成功返回后它将被填充。 ***用户负责释放该联合体***。

union ibv_gid的定义如下：

```cpp
union ibv_gid {
	uint8_t raw[16];
	struct {
		__be64 subnet_prefix;
		__be64 interface_id;
	} global;
};
```
**示例：**

```cpp
union ibv_gid gid;

rc = ibv_query_gid(ctx, 1, 2, &gid);
if (rc) {
	fprintf(stderr, "Error, failed to query GID index %d of port %d in device '%s'\n",
		2, 1, ibv_get_device_name(ctx->device));
	return -1;
}
```
**常见问题：**

Q：GID有什么好处？
A：GID是在不同子网之间发送包时的全局地址。每个GID有128位，它的值由两部分组合而成：
* 高64位：子网前缀。 标识一组由通用SM管理的终端端口。
* 低64位：GID前缀。由SM配置的EUI-64值。

Q：我在用iWARP/IBoE，我会用这个动词吗?
A：这个动词与InfiniBand和RoCE有关。

Q：当需要GID索引值时，每次调用ibv_query_gid()都需要花费时间，我可以缓存这个值吗?
A：实际上,是的。GID表是由SM配置的，SM可以在初始配置之后更改GID，但大多数情况下不会这样做。如果缓存GID表的值，则在发生IBV_EVENT_GID_CHANGE事件时，应该刷新这些值。

### 5.1.5 ibv_query_pkey
**函数原型：**
```cpp
int ibv_query_pkey(struct ibv_context *context, uint8_t port_num, int index, uint16_t *pkey)
```
**输入参数：**
* context——RDMA设备上下文，由ibv_open_device()返回。
* port_num——物理端口号，1是第一个端口。
* index——返回pkey表中的哪一项（0是第一个）

**输出参数：** pkey——期望的pkey。

**返回值：** 成功返回0；失败返回errno指示失败的原因。EMFILE	，该进程打开太多文件。

**说明：**

ibv_query_pkey在端口的分区密钥（pkey）表中检索条目。 子网管理器（SM）为每个端口分配至少一个pkey。 pkey标识端口所属的分区。 pkey大致类似于以太网网络中的VLAN ID。

仅当port_attr.state为IBV_PORT_ARMED或IBV_PORT_ACTIVE时，P_Key表的内容才有效。 对于端口的其他状态，分区表的值取决于实现。

配置该表的实体是SM，因此ibv_query_pkey()仅与InfiniBand相关。

用户传入一个指向uint16的指针，该指针将被请求的pkey填充。 ***用户有释放释放此uint16***。

**示例：**

```cpp
uint16_t pkey;

rc = ibv_query_pkey(ctx, 1, 2, &pkey);
if (rc) {
	fprintf(stderr, "Error, failed to query P_Key index %d of port %d in device '%s'\n",
		2, 1, ibv_get_device_name(ctx->device));
	return -1;
}
```
**常见问题：**

Q：P_Key有什么好处?
A：P_Key允许在物理网络上创建虚拟网络(就像以太网中的vlan)。
* 高1位：定义成员级别:
  * 1：正式成员
  * 0：受限成员
* 低15位：定义P_Key键值

使用它的规则非常简单:每个QP都与一个P_Key关联。

仅当满足以下两个条件时，QP X（与P_Key值PX相关联）才能与QP Y（与P_Key值PY相关联）进行通信：

* PX.membership == 1 || PY.membership == 1
* PX.key == PY.key


这意味着，如果PX和PY的成员资格都受限，则即使它们的键值相等，它们也无法通信。

Q：我正在使用iWARP / IBoE，会使用这个动词吗？
A：不。这个动词只与IB有关。

Q：当我需要P_Key索引的值时每次调用ibv_query_pkey()都会花费时间，我可以缓存该值吗？
A：是。 P_Key表是由SM配置的，SM可以更改它，但是大多数时候不是。 如果缓存P_Key表的值，则在发生IBV_EVENT_PKEY_CHANGE事件的情况下，应刷新这些值。

## 5.2 创建和销毁CC

### 5.2.1 ibv_create_comp_channel
**函数原型：**
```cpp
struct ibv_comp_channel *ibv_create_comp_channel(struct ibv_context *context)
```
**输入参数：** context——来自ibv_open_device的struct ibv_context。

**输出参数：** 无。

**返回值：** 成功返回指向创建的CC的指针，失败返回NULL。并设置errno指示失败原因：
* EMFILE——该进程打开太多文件。
* ENOMEM——没有足够的内存。

**说明：**

ibv_create_comp_channel为RMDA设备上下文创建一个完成事件通道。

此完成事件通道是libibverbs引入的抽象，在InfiniBand体系结构动词规范或RDMA协议动词规范中不存在。完成事件通道实质上是文件说明符，用于将“工作完成”通知传送到用户空间进程。当为完成队列（CQ）生成“工作完成”事件时，该事件通过附加到该CQ的完成事件通道传递。通过使用多个完成事件通道或将不同的优先级赋予不同的CQ，这可能有助于将完成事件引导到不同的线程。

一个或多个完成队列可以与同一完成事件通道关联。

struct ibv_comp_channel 定义如下：
```cpp
struct ibv_comp_channel {
	struct ibv_context	*context;	//上下文
	int					fd;			//文件说明符
	int					refcnt;		//通道的引用计数
};
```

**示例：** 见ibv_destroy_comp_channel。

**常见问题：**

Q：完成事件通道有什么好处？
A：完成事件通道是一个对象，它在用户空间进程中使用事件模式而不是轮询模式来帮助处理工作完成。

Q：为什么要在事件模式下读取工作完成?
A：如果您的进程不需要低延迟，或者需要降低CPU使用率，那么在事件模式下读取工作完成是非常有用的；您的进程将进入睡眠状态，当一个新的工作完成在与完成事件通道关联的CQ中可用时，你的进程并将被内核唤醒。

Q：可以将多个CQ与同一个完成事件通道关联吗?
A：是的。 多个CQ可以与同一完成事件通道相关联。 一个进程可以使用多个完成事件通道来“聚合”不同的CQ。 通过使用多个完成事件通道或将不同的优先级赋予不同的CQ，这可能有助于将完成事件引导到不同的线程。

Q：我如何使用多个完成事件通道来为不同的CQ分配不同的优先级？
A：您可以将具有相同优先级的所有CQ与相同的完成事件通道相关联，并根据CQs组的优先级来处理“工作完成”事件。

### 5.2.2 ibv_destroy_comp_channel
**函数原型：**
```cpp
int ibv_destroy_comp_channel(struct ibv_comp_channel *channel)
```
**输入参数：** channel——来自 ibv_create_comp_channel的struct ibv_comp_channel。

**输出参数：** 无。

**返回值：** 成功返回0；失败返回errno指示失败的原因。EBUSY——CQs仍然与这个完成事件通道相关联。

**说明：** ibv_destroy_comp_channel释放完成通道。 如果仍然有任何完成队列（CQ）与该完成通道关联，则此命令将失败。

**示例：**

```cpp
struct ibv_comp_channel *event_channel;

event_channel = ibv_create_comp_channel(context);
if (!event_channel) {
	fprintf(stderr, "Error, ibv_create_comp_channel() failed\n");
	return -1;
}

if (ibv_destroy_comp_channel(event_channel)) {
	fprintf(stderr, "Error, ibv_destroy_comp_channel() failed\n");
	return -1;
}
```

**常见问题：**
Q：ibv_destroy_comp_channel()失败了，与它相关的cq会发生什么变化?
A：什么变化都没有。你可以使用他们，没有任何副作用。

Q：ibv_destroy_comp_channel()失败了，我能知道哪些cq与它相关并导致了失败吗?
A：不，目前RDMA堆栈没有这个功能。

## 5.3 创建、修改、销毁CQ

###  5.3.1 ibv_create_cq
**函数原型：**
```cpp
struct ibv_cq *ibv_create_cq(struct ibv_context *context, int cqe, void *cq_context,
					struct ibv_comp_channel *channel, int comp_vector)
```
**输入参数：**
* context——来自ibv_open_device的struct ibv_context。
* cqe——CQ将支持的条目的最小值。取值为：1~dev_cap.max_cqe
* cq_context——（可选）用户定义的上下文，将在cq->cq_context中可用。当使用动词ibv_get_cq_event()等待完成事件通知时，将返回此上下文。
* channel——（可选）来自ibv_create_comp_channel的完成事件通道，用于指示新工作完成添加到此CQ中。NULL表示不使用完成事件通道。。
* comp_vector——（可选）MSI-X完成向量。将用于发送完成事件的信号。如果将这些中断的IRQ关联掩码配置为将每个MSI-X中断分散到不同的核上处理，则可以使用此参数将完成工作负载分散到多个核上。值可以是0~context->num_comp_vectors。

**输出参数：** 无。

**返回值：** 成功返回指向创建的CQ的指针，失败返回NULL。并设置errno指示失败原因.

* EINVAL——cqe，channel，comp_vector无效。
* ENOMEM——内存不足。

**说明：**

ibv_create_cq创建完成队列（CQ）。完成队列包含完成队列条目（CQE）。

每个队列对（QP）都有一个关联的发送CQ和接收CQ。一个CQ可以共享用于发送和接收，也可以在多个QP之间共享。工作完成持有信息以指出QP编号和它来自的队列（发送或接收）。

当发送或接收队列中的未完成工作请求完成时，会将工作完成添加到该工作队列的CQ中。 此工作完成指示未完成的工作请求已完成（不再视为未完成），并提供有关此请求的详细信息（状态，方向，操作码等）。

参数cqe定义队列的最小值。队列的实际大小可能大于或等于指定的值。

参数cq_context是用户定义的值。如果在CQ创建期间指定了该值，那么在使用完成通道（CC）时，该值将作为ibv_get_cq_event中的参数返回。

参数 channel用于指定CC。 CQ只是一个没有内置通知机制的队列。当使用轮询范例进行CQ处理时，CC是不必要的。用户只需定期轮询CQ。但是，如果您希望使用挂起范例，则需要CC。 CC是一种机制，它允许用户通知新的CQE出现在CQ上。

参数comp_vector用于指定用于通知完成事件的完成向量。它必须是0~context->num_comp_vectors。

```cpp
struct ibv_cq {
	struct ibv_context			*context;				//设备上下文，由ibv_create_cq传入
	struct ibv_comp_channel	*channel;				//事件通道，由ibv_create_cq传入
	void						*cq_context;			//用户传入的自定义上下文，由ibv_create_cq传入
	uint32_t					handle;					//句柄
	int							cqe;					//CQ中的CQE数量
	pthread_mutex_t				mutex;					//锁
	pthread_cond_t				cond;					//条件变量
	uint32_t					comp_events_completed;	//已经完成的完成事件
	uint32_t					async_events_completed;	//已经完成的异步事件
};
```
pthread_mutex_t定义如下:
```cpp
//位于<bits/pthreadtypes.h>
typedef union {
	struct __pthread_mutex_s	__data;
	char						__size[__SIZEOF_PTHREAD_MUTEX_T];
	long int					__align;
} pthread_mutex_t;
```
pthread_cond_t定义如下:

```cpp
//位于<bits/pthreadtypes.h>
typedef union {
	struct __pthread_cond_s			__data;
	char								__size[__SIZEOF_PTHREAD_COND_T];
	__extension__ long long int		__align;
} pthread_cond_t;

```

**示例：** 见ibv_destroy_cq。

**常见问题：**

Q：CQ有什么好处？
A：CQ用于为已完成并应产生工作完成的任何工作请求保存工作完成，并提供有关工作完成的详细信息。

Q：我可以在同一QP中对发送/接收队列使用不同的CQ吗？
A：是的。在任何QP中，发送队列的CQ和接收队列的CQ可能相同，也可能不同。这是灵活的，由用户决定。

Q：几个QP可以与同一个CQ相关联吗？
A：是。 多个QP可以在它们的发送或接收队列中或在两个队列中与同一个CQ相关联。

Q：CQ的尺寸应该是多少?
A：一个CQ应该有足够的空间来容纳与该CQ相关的队列中未完成的工作请求的所有工作完成，因此CQ的大小不应小于可能未完成的工作请求的总数。

Q：如果我选择的CQ尺寸太小会怎样？
A：如果出现CQ满的情况，并且向该CQ添加新的工作完成，则CQ会溢出。 带有错误的完成将添加到CQ，并且将生成异步事件IBV_EVENT_CQ_ERR。

Q：我可以使用哪个值作为cq_context?
A：由于cq_context为void*，因此您可以输入所需的任何值。


### 5.3.2 ibv_create_cq_ex

**函数原型：**
```cpp
struct ibv_cq_ex *ibv_create_cq_ex(struct ibv_context *context, struct ibv_cq_init_attr_ex *cq_attr)
```
**输入参数：**

* context——RDMA设备上下文。由ibv_open_device返回。
* cq_attr——CQ扩展属性。详细信息见下文。

**输出参数：** 无。

**返回值：** 成功返回指向创建的CQ_ex的指针，失败返回NULL。并设置errno指示失败原因。ENOSYS——RDMA设备不支持扩展CQ。

**说明：**

ibv_create_cq_ex根据cq_attr请求的属性为RDMA设备山西该文context创建一个CQ。创建的cq的大小可能大于或等于请求的大小。检查返回的CQ中的cqe可以获得实际大小。

struct ibv_cq_init_attr_ex定义如下：

```cpp
struct ibv_cq_init_attr_ex {
	int							cqe;			/* CQ的CQE的最小数量*/
	void						*cq_context;	/* 完成事件返回的用户定义上下文 */
	struct ibv_comp_channel	*channel;		/* 完成通道。如果不使用完成事件则为NULL，
													详细信息见ibv_create_comp_channel*/
	int							comp_vector;	/* 用于通知完成事件的完成向量。
													[0~context-> num_comp_vectors) */
	uint64_t					wc_flags;		/* 应该在ibv_poll_cq_ex中返回的wc_flags。
													按位或enum ibv_create_cq_wc_flags，详细信息见下文 */
	uint32_t					comp_mask;		/* 兼容性掩码（扩展动词），
													按位或enum ibv_cq_init_attr_mask ，详细信息见下文  */
	uint32_t					flags			/* 按位或enum ibv_create_cq_attr_flags，详细信息见下文 */
};
```

enum ibv_create_cq_wc_flags定义如下：

```cpp
enum ibv_create_cq_wc_flags {
	IBV_WC_EX_WITH_BYTE_LEN							= 1 << 0,		/* 在WC中需要字节长度 */
	IBV_WC_EX_WITH_IMM								= 1 << 1,		/* 在WC中需要立即数据 */
	IBV_WC_EX_WITH_QP_NUM							= 1 << 2,		/* 在WC中需要QP编号 */
	IBV_WC_EX_WITH_SRC_QP							= 1 << 3,		/* 在WC中需要源QP */
	IBV_WC_EX_WITH_SLID								= 1 << 4,		/* 在WC中需要slid */
	IBV_WC_EX_WITH_SL								= 1 << 5,		/* 在WC中需要sl */
	IBV_WC_EX_WITH_DLID_PATH_BITS					= 1 << 6,		/* 在WC中需要dlid路径位 */
	IBV_WC_EX_WITH_COMPLETION_TIMESTAMP				= 1 << 7,		/* 在WC中需要完成设备时间戳 */
	IBV_WC_EX_WITH_CVLAN							= 1 << 8,		/* 在WC中需要VLAN信息  */
	IBV_WC_EX_WITH_FLOW_TAG							= 1 << 9,		/* 在WC中需要流标签 */
	IBV_WC_EX_WITH_TM_INFO							= 1 << 10,
	IBV_WC_EX_WITH_COMPLETION_TIMESTAMP_WALLCLOCK	= 1 << 11,	/* 在WC中需要wallclock时间戳 */
};

enum {
	IBV_WC_STANDARD_FLAGS = IBV_WC_EX_WITH_BYTE_LEN		|
	IBV_WC_EX_WITH_IMM										|
	IBV_WC_EX_WITH_QP_NUM									|
	IBV_WC_EX_WITH_SRC_QP									|
	IBV_WC_EX_WITH_SLID										|
	IBV_WC_EX_WITH_SL										|
	IBV_WC_EX_WITH_DLID_PATH_BITS
};

enum {
	IBV_CREATE_CQ_SUP_WC_FLAGS = IBV_WC_STANDARD_FLAGS		|
	IBV_WC_EX_WITH_COMPLETION_TIMESTAMP						|
	IBV_WC_EX_WITH_CVLAN									|
	IBV_WC_EX_WITH_FLOW_TAG									|
	IBV_WC_EX_WITH_TM_INFO									|
	IBV_WC_EX_WITH_COMPLETION_TIMESTAMP_WALLCLOCK
};
```
enum ibv_cq_init_attr_mask定义如下：

```cpp
enum ibv_cq_init_attr_mask {
	IBV_CQ_INIT_ATTR_MASK_FLAGS = 1 << 0,
};
```

enum ibv_create_cq_attr_flags定义如下：

```cpp
enum ibv_create_cq_attr_flags {
	IBV_CREATE_CQ_ATTR_SINGLE_THREADED	= 1 << 0,	//此CQ是单线程使用的，因此不需要锁定
	IBV_CREATE_CQ_ATTR_IGNORE_OVERRUN	= 1 << 1,	/*如果溢出，此CQ不会进入错误状态，CQE始终将写入下一个条目
														必须将应用程序设计为避免CQ溢出，否则CQE可能会丢失。*/
};
```

struct ibv_cq_ex定义如下：

```cpp
struct ibv_cq_ex {
	struct ibv_context			*context;			//设备上下午文，来自ibv_
	struct ibv_comp_channel	*channel;
	void						*cq_context;
	uint32_t					handle;
	int							cqe;

	pthread_mutex_t				mutex;
	pthread_cond_t				cond;
	uint32_t					comp_events_completed;
	uint32_t					async_events_completed;

	uint32_t					comp_mask;
	enum ibv_wc_status			status;		//WC状态，详细信息见下文
	uint64_t					wr_id;
	int							(*start_poll)(struct ibv_cq_ex *current, struct ibv_poll_cq_attr *attr);
	int							(*next_poll)(struct ibv_cq_ex *current);
	void						(*end_poll)(struct ibv_cq_ex *current);
	enum ibv_wc_opcode			(*read_opcode)(struct ibv_cq_ex *current);
	uint32_t					(*read_vendor_err)(struct ibv_cq_ex *current);
	uint32_t					(*read_byte_len)(struct ibv_cq_ex *current);
	__be32						(*read_imm_data)(struct ibv_cq_ex *current);  //__be32 表示大端32位
	uint32_t					(*read_qp_num)(struct ibv_cq_ex *current);
	uint32_t					(*read_src_qp)(struct ibv_cq_ex *current);
	unsigned int				(*read_wc_flags)(struct ibv_cq_ex *current);
	uint32_t					(*read_slid)(struct ibv_cq_ex *current);
	uint8_t						(*read_sl)(struct ibv_cq_ex *current);
	uint8_t						(*read_dlid_path_bits)(struct ibv_cq_ex *current);
	uint64_t					(*read_completion_ts)(struct ibv_cq_ex *current);
	uint16_t					(*read_cvlan)(struct ibv_cq_ex *current);
	uint32_t					(*read_flow_tag)(struct ibv_cq_ex *current);
	void						(*read_tm_info)(struct ibv_cq_ex *current, struct ibv_wc_tm_info *tm_info);
	uint64_t					(*read_completion_wallclock_ns)(struct ibv_cq_ex *current);
};
```

  enum ibv_wc_status 定义如下：

```cpp
enum ibv_wc_status {
	IBV_WC_SUCCESS,
	IBV_WC_LOC_LEN_ERR,
	IBV_WC_LOC_QP_OP_ERR,
	IBV_WC_LOC_EEC_OP_ERR,
	IBV_WC_LOC_PROT_ERR,
	IBV_WC_WR_FLUSH_ERR,
	IBV_WC_MW_BIND_ERR,
	IBV_WC_BAD_RESP_ERR,
	IBV_WC_LOC_ACCESS_ERR,
	IBV_WC_REM_INV_REQ_ERR,
	IBV_WC_REM_ACCESS_ERR,
	IBV_WC_REM_OP_ERR,
	IBV_WC_RETRY_EXC_ERR,
	IBV_WC_RNR_RETRY_EXC_ERR,
	IBV_WC_LOC_RDD_VIOL_ERR,
	IBV_WC_REM_INV_RD_REQ_ERR,
	IBV_WC_REM_ABORT_ERR,
	IBV_WC_INV_EECN_ERR,
	IBV_WC_INV_EEC_STATE_ERR,
	IBV_WC_FATAL_ERR,
	IBV_WC_RESP_TIMEOUT_ERR,
	IBV_WC_GENERAL_ERR,
	IBV_WC_TM_ERR,
	IBV_WC_TM_RNDV_INCOMPLETE,
};
```

**轮询扩展的CQ** ：为了有效地轮询扩展的CQ，用户可以使用以下函数。

*  **完成迭代函数：**
   * `int ibv_start_poll(struct ibv_cq_ex *cq, struct ibv_poll_cq_attr *attr)`
   开始轮询一批工作完成。给出attr是为了使此函数在将来易于扩展。如果成功，此函数将返回0，否则返回错误代码。当CQ上没有可用的完成时，将返回ENOENT，但CQ保持有效状态。成功后，可以使用下面说明的查询函数来查询完成的属性。如果给出了错误代码，则不应调用end_poll。
   * `int ibv_next_poll(struct ibv_cq_ex *cq)`
调用此函数是为了获取下一个工作完成。 必须在start_poll之后和end_poll之前调用它。 如果成功，此函数将返回0，否则返回错误代码。 当CQ上没有可用的完成时，将返回ENOENT，但CQ保持有效状态。 成功后，可以使用下面说明的查询函数来查询完成的属性。 如果给出了错误代码，则仍应调用end_poll，表明这是轮询批处理的结尾。
   * `void ibv_end_poll(struct ibv_cq_ex *cq)`
 此函数指示工作完成的轮询批处理的结束。 调用此函数后，用户应通过调用start_poll开始新的批处理。
* **在完成中轮询字段：** 下面的成员和函数用于轮询当前完成。 当前完成是迭代器指向的完成（start_poll和next_poll使该迭代器前进）。 只能查询用户通过ibv_create_cq_ex中的wc_flags请求的字段。 此外，某些字段仅在某些操作码和状态码中有效。
   * `uint64_t wr_id`
可以从struct bv_cq_ex直接访问。
   * `enum ibv_wc_status`
可以从struct ibv_cq_ex直接访问。
   * `enum ibv_wc_opcode ibv_wc_read_opcode(struct ibv_cq_ex *cq);`
  从当前完成中获取操作码。
   * `uint32_t ibv_wc_read_vendor_err(struct ibv_cq_ex *cq);`
从当前完成中获取供应商错误。
   * `uint32_t ibv_wc_read_byte_len(struct ibv_cq_ex *cq);`
从当前完成中获取字节长度。
   * ` __be32 ibv_wc_read_imm_data(struct ibv_cq_ex *cq);`
   从当前完成中获取立即数据字段。
   * `uint32_t ibv_wc_read_invalidated_rkey(struct ibv_cq_ex *cq);`
   从当前完成中获取SEND_INVAL的无效的rkey。
   * `uint32_t ibv_wc_read_qp_num(struct ibv_cq_ex *cq);`
  从当前完成中获取QP编号字段。
   * ` uint32_t ibv_wc_read_src_qp(struct ibv_cq_ex *cq);`
   从当前完成中获取源QP编号字段。
   * `int ibv_wc_read_wc_flags(struct ibv_cq_ex *cq);`
   从当前完成中获取QP标志字段。
   * `uint16_t ibv_wc_read_pkey_index(struct ibv_cq_ex *cq);`
   从当前完成中获取Pkey索引字段。
   * `uint32_t ibv_wc_read_slid(struct ibv_cq_ex *cq);`
   从当前完成中获取slid字段。
   * `uint8_t ibv_wc_read_sl(struct ibv_cq_ex *cq);`
   从当前完成中获取sl字段。
   * `uint8_t ibv_wc_read_dlid_path_bits(struct ibv_cq_ex *cq);`
  从当前完成中获取dlid_path_bits字段。
   * `uint64_t ibv_wc_read_completion_ts(struct ibv_cq_ex *cq);`
  从当前完成中获取的完成时间戳，单位为HCA时钟。
   * `uint64_t ibv_wc_read_completion_wallclock_ns(struct ibv_cq_ex *cq);`
   从当前完成中获取完成时间戳，并将其由HCA时钟单位转为 wallclock 纳秒。
   * `uint16_t ibv_wc_read_cvlan(struct ibv_cq_ex *cq);`
      从当前完成中获取CLAN字段。
   * `uint32_t ibv_wc_read_flow_tag(struct ibv_cq_ex *cq); `
      从当前完成中获取流标签。
   * `void ibv_wc_read_tm_info(struct ibv_cq_ex *cq, struct ibv_wc_tm_info *tm_info);`
      从当前完成中获取标签匹配信息。
	
struct ibv_wc_tm_info 定义如下：
```cpp
struct ibv_wc_tm_info {
	uint64_t tag;		/* 来自TMH的标签 */
	uint32_t priv;		/* 来自TMH的不透明用户数据 */
};
```
struct ibv_poll_cq_attr 定义如下
```cpp
struct ibv_poll_cq_attr {
	uint32_t comp_mask;
};
```
### 5.3.3 ibv_cq_ex_to_cq
  **函数原型：**
 **
```cpp
static inline struct ibv_cq *ibv_cq_ex_to_cq(struct ibv_cq_ex *cq)
```
**输入参数：** cq——扩展cq。由ibv_create_cq_ex返回。

**输出参数：** 无。‘

**返回值：** 成功返回说明WC状态的字符串。失败返回NULL。

**说明：** 将扩展cq转为普通cq。

### 5.3.4 ibv_resize_cq
**函数原型：**
```cpp
int ibv_resize_cq(struct ibv_cq *cq, int cqe)
```
**输入参数：**

* cq——要重新分配大小的CQ。由ibv_create_cq返回。
* cqe——CQ将支持的条目的最小值。

**输出参数：** 无。

**返回值：** 成功返回0；失败返回errno指示失败的原因。
* EINVAL——无效的cqe。
* ENOMEM——内存不足。
* ENOSYS——设备不支持调整CQ大小。

**说明：**
ibv_resize_cq调整完成队列（CQ）的大小。

如果CQ为空或包含尚未由ibv_poll_cq()轮询的工作完成，则可以调整CQ的大小。 如果在创建任何通知事件之前使用ibv_req_notify_cq()请求了通知请求，则可以调整其大小。 如果一个或多个QP与之关联或没关联，则可以调整它的大小。

如果要减小CQ大小，则新大小会有一些限制：

* 它不能小于CQ中当前存在的“工作完成”数量
* 足以容纳可能会进入此CQ的工作完成数量，否则，CQ会溢出。

用户可以定义CQ的最小大小。 实际大小可以等于或大于此值。 cq的cqe成员将更新为实际大小。

ibv_resize_cq()可以分配大于或等于请求的大小的CQ大小。因此，调用ibv_resize_cq()可能不会做任何事情，因为此动词仅保证将CQ调整为至少与请求的大小一样大的大小。


**示例：** 见ibv_destroy_cq。

**常见问题：**

Q：ibv_resize_cq()失败，与此相关的QP会发生什么？
A：没事 您可以继续使用，而不会产生任何副作用。

Q：ibv_resize_cq()失败，我可以知道此CQ中存在多少工作完成吗？
A：不，RDMA堆栈没有这个功能(它在规范中没有定义)。

Q：我可以调整其中包含工作完成的CQ的大小吗？
A：是的，可以。


Q：我在CQ上调用了ibv_req_notify_cq()，但没有得到任何完成事件。 我可以调整该CQ的大小吗？
A：是的，只要新的尺寸不小于该CQ中当前的工作完成数量即可。

Q：我调用了ibv_resize_cq()并尝试减小CQ大小，但是CQ大小没有改变。 发生了什么？
A：这可能发生。 规则是用户可以定义CQ的最小大小，并且实际大小可以等于或大于此值。 尝试减小CQ大小可能会导致无济于事。 这取决于（低级驱动程序的）实现。

### 5.3.5 ibv_modify_cq
**函数原型：**
```cpp
int ibv_modify_cq(struct ibv_cq *cq, struct ibv_modify_cq_attr *cq_attr。)
```
**输入参数：**

* cq——将要修改的CQ。来自ibv_create_cq。
* cq_attr——cq属性。详细信息见下文。

**输出参数：** cq_attr——cq属性。CQ修改后的实际属性。

**返回值：** 成功返回0；失败返回errno指示失败的原因。ENOSYS——设备不支持修改CQ属性。

**说明：**

ibv_modify_cq基于cq_attr->attr_mask修改CQ属性。

struct ibv_modify_cq_attr定义如下：

```cpp
struct ibv_modify_cq_attr {
	uint32_t				attr_mask ;	//要修改的属性的掩码，使用enum ibv_cq_attr_mask，详细信息见下文.
	struct ibv_moderate_cq	moderate;
};
```
struct ibv_moderate_cq定义如下：
```cpp
struct ibv_moderate_cq {
	uint16_t cq_count;		//CQ计数
	uint16_t cq_period;	//CQ周期，微秒
};

```
enum ibv_cq_attr_mask定义如下：
```cpp
enum ibv_cq_attr_mask {
	IBV_CQ_ATTR_MODERATE	= 1 << 0,
	IBV_CQ_ATTR_RESERVED	= 1 << 1,
};
```

### 5.3.6 ibv_destroy_cq
**函数原型：**
```cpp
int ibv_destroy_cq(struct ibv_cq *cq)
```
**输入参数：** cq，将要销毁的CQ。

**输出参数：** 无。

**返回值：** 成功返回0；失败返回errno指示失败的原因。EBUSY——一个或多个WQ仍然与此CQ关联。

**说明：**

ibv_destroy_cq释放完成队列（CQ）。 如果有任何队列对（QP）仍与指定的CQ关联，则此命令将失败。

如果使用ibv_get_async_event()读取了该CQ上的任何关联异步事件，但仍未使用ibv_ack_async_event()进行确认，则在确认该事件之前，对ibv_destroy_cq()的调用将永远不会结束。

如果CQ为空或包含ibv_poll_cq()尚未轮询的工作完成，则可以销毁CQ。 如果在创建任何通知事件之前使用ibv_req_notify_cq()请求了通知请求，则也可以销毁它。

**示例：**

```cpp
struct ibv_cq *cq;

cq = ibv_create_cq(context, 100, NULL, NULL, 0);
if (!cq) {
	fprintf(stderr, "Error, ibv_create_cq() failed\n");
	return -1;
}

if (ibv_resize_cq(cq, 2000)) {
	fprintf(stderr, "Error, ibv_resize_cq() failed\n");
	return -1;
}

if (ibv_destroy_cq(cq)) {
	fprintf(stderr, "Error, ibv_destroy_cq() failed\n");
	return -1;
}
```

**常见问题：**

Q：ibv_destroy_cq()失败，与此相关的QP会发生什么？
A：没事 您可以继续使用，而不会产生任何副作用。

Q：ibv_destroy_cq()失败，我可以知道哪些QP与之相关并导致了此失败吗？
A：不，目前RDMA堆栈没有这个功能。

Q：我可以销毁带有工作完成的CQ吗？
A：是的，你可以。当CQ被摧毁时，它是否包含工作完成并不重要。

Q：我在CQ上调用了ibv_req_notify_cq()，但没有得到任何完成事件。 我可以销毁该CQ吗？
A：是的，你可以。


Q：我调用了ibv_destroy_cq()，但是它没有结束。 发生了什么？
A：该CQ上至少有一个关联的异步事件被读取，但没有确认。

## 5.4 分配和释放PD
###  5.4.1 ibv_alloc_pd
**函数原型：**
```cpp
struct ibv_pd *ibv_alloc_pd(struct ibv_context *context)
```
**输入参数：** context——来自ibv_open_device的struct ibv_context。

**输出参数：** 无。

**返回值：** 成功返回指向创建的保护域的指针，失败返回NULL。

**说明：**

ibv_alloc_pd创建保护域（PD）。创建的PD被用于：创建 AH, SRQ, QP、注册 MR、分配 MW。

PD限制了哪些队列对（QP）可以访问哪些内存区域，从而提供了一定程度的保护，防止未经授权的访问。 用户必须创建至少一个PD才能使用VPI动词


```cpp
struct ibv_pd {
	struct ibv_context		*context;	//设备上下文
	uint32_t				handle;		//句柄
};

 ```
**示例：** 见 ibv_dealloc_pd。

**常见问题：**

Q：PD有什么好处？
A：PD是一种保护手段，可帮助您创建可以一起工作的一组对象。 如果使用PD1创建了多个对象，而使用PD2创建了其他对象，则将来自group1的对象与来自group2的对象一起使用将最终以错误结束。

### 5.4.2 ibv_dealloc_pd
**函数原型：**
```cpp
int ibv_dealloc_pd(struct ibv_pd *pd)
```
**输入参数：** pd——来自ibv_dealloc_pd的struct ibv_pd。

**输出参数：** 无。

**返回值：** 成功返回0；失败返回errno，指示失败的原因。
* EINVAL，PD的设备上下文无效。
* EBUSY，还有RDMA资源与PD关联。	

**说明：** ibv_dealloc_pd释放保护域（PD）。 如果当前有任何其他对象与指定的PD相关联，则该命令将失败。可以关联的资源有：AH、QP、SRQ、MW、MR。

**示例：**
```cpp
struct ibv_context *context;
struct ibv_pd *pd;

pd = ibv_alloc_pd(context);
if (!pd) {
	fprintf(stderr, "Error, ibv_alloc_pd() failed\n");
	return -1;
}

if (ibv_dealloc_pd(pd)) {
	fprintf(stderr, "Error, ibv_dealloc_pd() failed\n");
	return -1;
}
```

**常见问题：**

Q：我可以通过一个动词调用销毁PD和与其相关的所有RDMA资源吗？
A：不，libibverbs不支持它。 如果用户希望取消分配PD，则需要在调用ibv_dealloc_pd()之前销毁与之关联的所有RDMA资源。

Q：ibv_dealloc_pd()失败，我可以知道分配了哪些RDMA资源并导致此失败吗？
A：不可以，目前RDMA堆栈不具备此功能。

## 5.5 分配和释放DM
### 5.5.1 ibv_adlloc_dm
**函数原型：**
```cpp
struct ibv_dm *ibv_alloc_dm(struct ibv_context *context, struct ibv_alloc_dm_attr *attr)
```
**输入参数：**
* context——设备上下文，由ibv_open_device创建。
* attr——设备内存属性，详细信息见下文。

**输出参数：** 无。

**返回值：** 成功返回指向ibv_dm结构体的指针，失败返回NULL。并设置errno显示失败原因。

**说明：**

ibv_alloc_dm()为RMDA设备上下文context分配一个设备内存缓冲区。

struct ibv_alloc_dm_attr定义如下：

```cpp
struct ibv_alloc_dm_attr {
	size_t		length;			/*期望的设备内存缓冲区的长度 */
	uint32_t	log_align_req;	/*与log2地址对齐要求的 log 2*/
	uint32_t	comp_mask;		/*兼容性掩码，定义了哪些跟随的变量有效 */
};
```

在使用设备内存执行RDMA原子操作的情况下，可能需要地址对齐。

在这种情况下，用户可以使用分配属性结构体中的log_align_req参数指定设备内存的起始地址对齐方式。

如果设备没有剩余的可用设备内存，则ibv_alloc_dm()可能会失败，其中最大的已分配内存量是由ibv_device_attr_ex结构中的max_dm_size属性提供的。


struct ibv_dm定义如下：

```cpp
struct ibv_dm {
	struct ibv_context	*context;													//动词上下文
	int					(*memcpy_to_dm)(struct ibv_dm *dm, uint64_t dm_offset,
										const void *host_addr, size_t length);	//从主机内存拷贝到设备内存
	int					(*memcpy_from_dm)(void *host_addr, struct ibv_dm *dm,
										uint64_t dm_offset, size_t length);		//从设备内存拷贝到主机内存
	uint32_t			comp_mask;													//兼容性掩码
};
```

### 5.5.2 ibv_dealloc_dm
**函数原型：**
```cpp
 int ibv_free_dm(struct ibv_dm *dm)
```
**输入参数：** ibv_dm——设备内存，来自ibv_alloc_dm。

**输出参数：** 无。

**返回值：** 成功返回0，失败返回errno以显示错误原因。

**说明：** 释放设备内存。如果如果任何其他资源（例如MR）仍然与DM关联，则调用失败。

## 5.6 分配和释放TD（差mojo）

### 5.6.1 ibv_alloc_td
**函数原型：**
```cpp
struct ibv_td *ibv_alloc_td(struct ibv_context *context, struct ibv_td_init_attr *init_attr);
```
**输入参数：**

* context——动词上下文，来自ibv_open_device。
* init_attr——初始属性。

**输出参数：** 无。

**返回值：** 成功返回指向struct ibv_td的指针，失败返回NULL，并设置errno显示失败原因。

**说明：**

ibv_alloc_td()为RDMA设备上下文context分配一个线程域。

线程域对象定义动词库和驱动程序将如何使用锁和其他硬件功能来获得最佳性能，以处理多线程或单线程保护。 应用程序在创建动词对象时将动词资源分配给线程域。

如果指定了ibv_td对象，则在该线程域下创建的所有对象都将禁用内部锁，该内部锁定旨在防止多个用户线程同时访问该对象。 默认情况下，无论是否指定线程域，所有动词对象都可安全用于多线程访问。

可以通过ibv_alloc_parent_domain()将struct ibv_td添加到父域中，然后可以使用父域来创建动词对象。

struct ibv_td_init_attr定义如下：

```cpp
struct ibv_td_init_attr {
	uint32_t comp_mask;	//兼容性掩码
}
```

struct ibv_td定义如下：
```cpp
struct ibv_td {
	struct ibv_context *context;	//设备上下文
};
```

### 5.6.2 ibv_dealloc_td
**函数原型：**
```cpp
int ibv_dealloc_td(struct ibv_td *td)
```
**输入参数：** td——线程域，来自ibv_alloc_td。

**输出参数：** 无。

**返回值：** 成功返回0，失败返回errno以显示失败原因。

**说明：** ibv_dealloc_td()将取消分配线程域td。 在取消分配td之前，应销毁使用td创建的所有资源。

## 5.7 打开和关闭XRCD（差mojo）
### 5.7.1 ibv_open_xrcd
**函数原型：**
```cpp
struct ibv_xrcd *ibv_open_xrcd(struct ibv_context *context, struct ibv_xrcd_init_attr *xrcd_init_attr)
```
**输入参数：**

* context——设备上下文。
* xrcd_init_attr——初始化属性。

**输出参数：** 无。

**返回值：** 成功返回指向struct ibv_xrcd的指针，失败返回NULL。

**说明：**

ibv_open_xrcd为RDMA设备上下文context打开一个XRC域。xrcd_init_attr是xrc的初始属性。

struc ibv_xrcd_init_attr定义如下：

```cpp
struct ibv_xrcd_init_attr {
	uint32_t	comp_mask;	//兼容性掩码，用于标识有效字段。enum ibv_xrcd_init_attr_mask按位或
	int			fd;			//与XRCD关联的文件描述符。0或以下值按位或得到。
	int			oflags;		/*期待的创建属性。
								O_CREAT：创建XRCD并将其与给定fd引用的inode关联。 如果存在XRCD，
									则此标志无效，除非下面的O_EXCL另有说明。
								O_EXCl：如果设置了O_EXCL和O_CREAT，则如果存在与inode关联的XRCD，
									则打开将失败。
								如果fd等于-1，则没有inode与XRCD关联。 为了指示应该创建XRCD，
									使用oflag = O_CREAT。*/
};
```
enum ibv_xrcd_init_attr_mask定义如下：
```cpp
enum ibv_xrcd_init_attr_mask {
	IBV_XRCD_INIT_ATTR_FD		= 1 << 0,	//fd字段有效
	IBV_XRCD_INIT_ATTR_OFLAGS	= 1 << 1,	//oflags字段有效
	IBV_XRCD_INIT_ATTR_RESERVED	= 1 << 2	//保留
};
```

struct ibv_xrcd定义如下：
```cpp
struct ibv_xrcd {
	struct ibv_context	*context;	//设备上下文
};
```
### 5.7.2 ibv_close_xrcd
**函数原型：**

```cpp
int ibv_close_xrcd(struct ibv_xrcd *xrcd)
```
**输入参数：** xrcd——来自ibv_opend_xrcd。

**输出参数：** 无。

**返回值：** 成功返回0，失败返回errno以显示失败原因。

**说明：**

ibv_close_xrcd()关闭XRCD域。如果是最后一个引用，则XRCD将被销毁。

如果xrcd被关闭时仍有其他任何资源与其关联，则该调用可能会失败。

## 5.8 创建和销毁计数器（差mojo）

### 5.8.1 ibv_create_counters
**函数原型：** 
```cpp
struct ibv_counters *ibv_create_counters(struct ibv_context *context, 
								struct ibv_counters_init_attr *init_attr);
```
**输入参数：** 

* context——设备上下文，来自ibv_open_device。
* init_attr——计数器初始属性，详细信息见下文。

**输出参数：** 无。

**返回值：** 成功返回指向struct ibv_counters的指针，失败返回NULL，并设置errno显示错误原因。

* ENOSYS——当前设备不支持ibv_create_counters。
* ENOMEM——没有足够的内存。

**说明：** 

ibv_create_counters为RDMA设备上下文创建一个新的计数器句柄。

ibv_一个counters句柄在创建时可以静态添加到动词资源（例如：QP，WQ，Flow）。

例如，在创建新的流期间，通过调用ibv_create_flow()将ibv_counters静态添加到流中（struct ibv_flow）上。

计数器在创建时清空，并且值将单调增加。

struct ibv_counters_init_attr定义如下：
```cpp
struct ibv_counters_init_attr {
	uint32_t	comp_mask;	//有效字段掩码
};
```

struct ibv_counters定义如下：
```cpp
struct ibv_counters {
	struct ibv_context	*context;	//设备上下文，详细信息见ibv_open_device
};
```

**示例：** 见ibv_read_counters。

### 5.8.2 ibv_destroy_counters
**函数原型：** 
```cpp
int ibv_destroy_counters(struct ibv_counters *counters)
```
**输入参数：** 

**输出参数**

**返回值：** 成功返回0，失败返回errno显失败原因。EINVAL——参数非法。

**说明：** ibv_destroy_counters()释放了计数器句柄，用户应在销毁计数器对象之前将其分离。
## 5.9 创建、修改、销毁WQ（差mojo）
### 5.9.1 ibv_create_wq
**函数原型：** 
```cpp
struct ibv_wq *ibv_create_wq(struct ibv_context *context, struct ibv_wq_init_attr *wq_init_attr)
```
**输入参数：** 

* context——设备上下文，来自ibv_open_device。
* wq_init_attr——WQ初始化属性。详细信息见下文。

**输出参数：** wq_init_attr——WQ的真实属性。

**返回值：** 成功返回指向struct ibv_wq的指针，失败返回NULL。

**说明：** 

ibv_create_wq创建一个WQ并与设备上下文context关联。成功创建后，该函数将用创建的WQ的实际值更新wq_init_attr->max_wr and wq_init_attr->max_sge字段，实际值肯大于或等于请求的值。


struct ibv_wq的定义如下：

```cpp
struct ibv_wq {
	struct ibv_context		*context;			//设备上下文,详细信息见ibv_open_device
	void					*wq_context;		//自定义上下文
	struct  ibv_pd			*pd;				//保护域,详细信息见ibv_alloc_pd
	struct  ibv_cq			*cq;				//完成队列,详细信息见ibv_create_cq
	uint32_t				wq_num;
	uint32_t				handle;
	enum ibv_wq_state		state;				//工作队列状态，详细信息见下文
	enum ibv_wq_type		wq_type;			//工作队列类型，详细信息见下文
	int						(*post_recv)(struct ibv_wq *current,struct ibv_recv_wr *recv_wr,
											struct ibv_recv_wr **bad_recv_wr);
	pthread_mutex_t			mutex;
	pthread_cond_t			cond;
	uint32_t				events_completed;
	uint32_t				comp_mask;
};
```
工作队列(WQ)。QP可以在没有内部WQ“打包”的情况下被创建，该QP配置为使用“外部”WQ对象作为其接收/发送队列。

与完成队列关联的WQ（多对一）拥有WQ属性（PD，WQ大小等）。

IBV_WQT_RQ类型的WQ：
* 包含接收的WQE，在这种情况下，它的PD也是分散的。
* 公开发布接收函数，用于将工作请求（WR）列表发布到其接收队列。

enum ibv_wq_state定义如下：

```cpp
enum ibv_wq_state {
	IBV_WQS_RESET,
	IBV_WQS_RDY,
	IBV_WQS_ERR,
	IBV_WQS_UNKNOWN
}; 
```
enum ibv_wq_type定义如下：

```cpp
enum ibv_wq_type {
	IBV_WQT_RQ
};
```
struct ibv_wq_init_attr定义如下：

```cpp
struct ibv_wq_init_attr {
	void				*wq_context;	//与WQ关联的上下文
	enum ibv_wq_type	wq_type;		//WQ类型，详细信息见ibv_create_wq
	uint32_t			max_wr;			//请求的WQ中未完成的WR的最大数量
	uint32_t			max_sge;		//WQ中每个WR请求的最大分散/聚合元素数量
	struct ibv_pd		*pd;			//将要与WQ关联的PD
	struct ibv_cq		*cq;			//将要与WQ关联的CQ
	uint32_t			comp_mask; 	//标识有效字段。使用 enum ibv_wq_init_attr_mask
	uint32_t			create_flags; 	//此WQ的创建标志，使用enum ibv_wq_flags
};
```
enum ibv_wq_init_attr_mask定义如下：
```cpp
enum ibv_wq_init_attr_mask {
	IBV_WQ_INIT_ATTR_FLAGS			= 1 << 0,
	IBV_WQ_INIT_ATTR_RESERVED		= 1 << 1,
}; 
```
enum ibv_wq_flags定义如下：
```cpp
enum ibv_wq_flags {
	IBV_WQ_FLAGS_CVLAN_STRIPPING		= 1 << 0,	//CVLAN字段将从传入数据包中剥离
	IBV_WQ_FLAGS_SCATTER_FCS			= 1 << 1,	//FCS字段将分散到主机内存中
	IBV_WQ_FLAGS_DELAY_DROP				= 1 << 2,	//如果没有接收到的WQE，数据包将不会立即被丢弃
	IBV_WQ_FLAGS_PCI_WRITE_END_PADDING	= 1 << 3,	//传入的数据包将被填充为缓存行的大小
	IBV_WQ_FLAGS_RESERVED				= 1 << 4,	//
};
```
### 5.9.2 ibv_modify_wq(N)

### 5.9.3 ibv_destroy_wq
**函数原型：** 
```cpp
int ibv_destroy_wq(struct ibv_wq *wq)
```
**输入参数：** wq——工作队列。来自ibv_create_wq。

**输出参数：** 无。

**返回值：** 成功返回0，失败返回errno显示错误原因。

**说明：** 销毁已存在的wq。

## 5.10 创建和销毁RWQ IND TBL
### 5.10.1 ibv_create_rwq_ind_table
**函数原型：** 
```cpp
struct ibv_rwq_ind_table *ibv_create_rwq_ind_table(struct ibv_context *context,
													struct ibv_rwq_ind_table_init_attr *init_attr)
```
**输入参数：** 

* context——设备上下文，来自ibv_open_device。
* init_attr——要创建的接收工作队列间接表（WRQ IND TBL）的初始属性。

**输出参数：** 无。

**返回值：** 成功返回指向RWQ IND TABLE的指针，失败返回NULL。

**描述：** 

创建一个RWQ IND TBL， 并与设备上下文context关联。

函数ibv_create_rwq_ind_table()将创建一个包含一个接收工作队列表的RWQ IND TBL。

创建的对象应用作ibv_create_qp_ex()的一部分，以启用基于某些RX哈希配置的传入数据包的分派。

struct ibv_rwq_ind_table 定义如下：
```cpp
struct ibv_rwq_ind_table {
	struct ibv_context	*context;
	int					ind_tbl_handle;
	int					ind_tbl_num;
	uint32_t			comp_mask;			//掩码，使用enum ibv_ind_table_init_attr_mask
};
```

struct ibv_rwq_ind_table_init_attr定义如下：
```cpp
struct ibv_rwq_ind_table_init_attr {
	uint32_t		log_ind_tbl_size;	//间接表日志大小，2的n次方|
	struct ibv_wq	**ind_tbl;			//每个元素都是指向接收工作队列的指针。
	uint32_t		comp_mask;			//标识有效字段。使用enum ibv_ind_table_init_attr_mask
};
```

enum ibv_ind_table_init_attr_mask定义如下：

```cpp
enum ibv_ind_table_init_attr_mask {
	IBV_CREATE_IND_TABLE_RESERVED	= (1 << 0)
};
```
### 5.10.2 ibv_destroy_rwq_ind_table
**函数原型：** 
```cpp
int ibv_destroy_rwq_ind_table(struct ibv_rwq_ind_table *rwq_ind_table)
```
**输入参数：** rwq_ind_tablet——要销毁的接收工作队列间接表。来自ibv_create_rwq_ind_table。

**输出参数：** 无。

**返回值：** 成功返回0，失败返回errno显示失败原因。

**描述：** 销毁接收工作队列间接表（RWQ IND TBL）。

# 6 基于保护域的操作
建立保护域（PD）后，可以在该域中创建对象。 本节介绍了PD上可用的操作。 这些包括注册内存区域（MR），创建队列对（QP）或共享接收队列（SRQ）和地址句柄（AH）。

## 6.1 创建、查询、修改、销毁SRQ
### 6.1.1 ibv_create_srq
**函数原型：**
```cpp
struct ibv_srq *ibv_create_srq(struct ibv_pd *pd, struct ibv_srq_init_attr *srq_init_attr)
```
**输入参数：**
* pd——来自ibv_dealloc_pd的struct ibv_pd，与共享接收队列关联的PD。
* srq_init_attr——创建SRQ所需的初始属性列表。详细信息见下文。

**输出参数：** srq_init_attr——RQ的真实属性值。

**返回值：** 成功返回指向创建的SRQ的指针，失败返回NULL，并设置errno指示错误原因。

* EINVAL——无效pd 或 max_wr或max_sge中提供的值无效。
* ENOMEM——没有足够的资源来完成此操作。
* ENOSYS——此RDMA设备不支持共享接收队列。

**说明：**
ibv_create_srq创建与保护域PD关联的共享接收队列（SRQ）。 读取srq_attr-> max_wr和srq_attr-> max_sge以确定所需的SRQ大小，并将其设置为返回时分配的实际值。 如果ibv_create_srq成功，则max_wr和max_sge将至少与请求的值一样大。

函数ibv_create_srq()将使用创建的SRQ的原始值更新srq_init_attr结构体。 max_wr和max_sge的值将大于或等于请求的值。

这个SRQ稍后将在调用ibv_create_qp()时使用，表示该QP的RQ与其他QP共享。

如果dev_cap.max_srq的值为零，则意味着RDMA设备不支持共享接收队列。

用户可以定义SRQ的最小属性：工作请求的数量和每个工作请求的分散/收集条目数。 实际属性可以等于或高于这些值。

struct ibv_srq_init_attr 说明了新创建的srq的请求属性。定义如下：
```cpp
struct ibv_srq_init_attr {
	void	*srq_context;		//(可选的)SRQ关联的上下文，用户定义值，在srq->srq_context中可用
	struct	ibv_srq_attrattr;	//共享接收队列的属性，详细信息见ibv_modify_srq
};
```

struct ibv_srq定义如下：
```cpp
struct ibv_srq {
	struct ibv_context	*context;		//来自ibv_open_device的struct ibv_context。
	void				*srq_context;	//用户定义SRQ上下文
	struct ibv_pd		*pd;			//保护域，详细信息见ibv_alloc_pd
	uint32_t			handle;
	pthread_mutex_t		mutex;
	pthread_cond_t		cond;
	uint32_t			events_completed;
}
```

**示例：** 见ibv_destroy_srq。

**常见问题：**
Q：SRQ有什么好处？
A：SRQ，顾名思义:是一个共享接收队列。与使用队列对(每个队列都有自己的接收队列)相比，使用SRQs可以减少发送的接收请求的总数。

Q：我可以将一个SRQ与不同的QP一起使用吗？
A：可以。

Q：我可以对不同传输类型的QP使用一个SRQ吗？
A：可以。

Q：如何使用srq_limit?
A：如果SRQ配备了一个值，则当该SRQ中的RR数量降至该值以下时，将生成关联的异步事件IBV_EVENT_SRQ_LIMIT_REACHED。

Q：SRQ应该使用哪些属性?
A：工作请求的数量应足以容纳与此SRQ关联的所有QP中所有传入消息的接收请求。分散/收集元素的数量应该是将发布到此SRQ的任何工作请求中分散/收集元素的最大数量。

Q：我可以使用哪个值作为srq_context?
A：因为srq上下文是一个void \*，所以您可以输入任何您想要的值。



### 6.1.2 ibv_create_srq_ex
**函数原型：**
```cpp
struct ibv_srq *ibv_create_srq_ex(struct ibv_context *context,
						struct ibv_srq_init_attr_ex *srq_init_attr_ex);
```
**输入参数：**
* context——设备上下文，来自ibv_open_device。
* srq_init_attr_ex——srq初始扩展属性。详细信息见下文。

**输出参数：** srq_init_attr_ex——srq实际扩展属性。

**返回值：** 成功返回创建的SRQ的指针；失败返回NULL。如果调用失败，将设置errno来指示失败的原因。ENOSYS——设备不支持。

**说明：**

ibv_create_srq_ex()创建支持基本模式和xrc模式的共享接收队列（SRQ）。

函数ibv_create_srq_ex()将使用创建的SRQ的原始值更新srq_init_attr_ex结构体。 max_wr和max_sge的值将大于或等于请求的值。

struct ibv_srq_init_attr_ex定义如下：
```cpp
struct ibv_srq_init_attr_ex {
	void					*srq_context;	// SRQ的关联上下文
	struct ibv_srq_attr	attr;			// SRQ属性 ，详细信息见ibv_create_srq
	uint32_t				comp_mask;		// 标识有效字段，使用enum ibv_srq_init_attr_mask，详细信息见下文
	enum ibv_srq_type		srq_type;		// 基本，或XRC，或标签匹配
	struct ibv_pd			*pd;			// 与SRQ关联的PD，详细信息见ibv_alloc_pd
	struct ibv_xrcd		*xrcd;			// 与SRQ关联的XRC域，详细信息见ibv_open_xrcd
	struct ibv_cq			*cq;			// 为了XRC模式要与SRQ关联的CQ ，详细信息见ibv_create_cq
	struct ibv_tm_cap		tm_cap;			// 标签匹配属性
};
```

struct ibv_srq_type定义如下：
```cpp
enum ibv_srq_type {
	IBV_SRQT_BASIC,
	IBV_SRQT_XRC,
	IBV_SRQT_TM,
};

```
enum ibv_srq_init_attr_mask定义如下：
```cpp
enum ibv_srq_init_attr_mask {
	IBV_SRQ_INIT_ATTR_TYPE		= 1 << 0,
	IBV_SRQ_INIT_ATTR_PD		= 1 << 1,
	IBV_SRQ_INIT_ATTR_XRCD		= 1 << 2,
	IBV_SRQ_INIT_ATTR_CQ		= 1 << 3,
	IBV_SRQ_INIT_ATTR_TM		= 1 << 4,
	IBV_SRQ_INIT_ATTR_RESERVED	= 1 << 5,
};
```

struct ibv_tm_cap 定义如下：
```cpp
struct ibv_tm_cap {
	uint32_t	max_num_tags;	/* 标签匹配列表大小 */
	uint32_t	max_ops;		/* 未完成的标记列表操作的数量 */
};
```

### 6.1.3 ibv_modify_srq
**函数原型：**
```cpp
int ibv_modify_srq (struct ibv_srq *srq, struct ibv_srq_attr *srq_attr, int srq_attr_mask)
```
**输入参数：**
* srq——要修改的SRQ。来自ibv_create_srq或ibv_create_srq_ex。
* srq_attr——指定要修改的SRQ属性（输入）。详细信息见下文。
* srq_attr_mask——将要被修改SRQ属性的位掩码。值为0或者一个多个枚举值按位或。详细信息见下文。

**输出参数：** srq_attr——struct ibv_srq_attr返回更新后的值。

**返回值：** 成功返回0；失败返回errno来指示失败的原因。

* EINVAL——max_wr、max_sge或srq_limit中提供的值无无效。
* ENOMEM——没有足够的资源来完成这个操作。
* ENOSYS——此RDMA设备不支持共享接收队列修改。

**说明：**
ibv_modify_srq基于掩码srq_attr_mask使用srq_attr中的的属性值来修改SRQ srq的属性。 srq_attr是上面在动词

如果要修改的任何属性是无效的，则不会修改任何属性。 另外，并非所有设备都支持调整SRQ的大小。 要检查设备是否支持调整大小，请检查设备功能标志中的IBV_DEVICE_SRQ_RESIZE位是否已设置。

一旦SRQ中的WRs数量下降到SRQ限制以下，修改SRQ限制将使SRQ产生IBV_EVENT_SRQ_LIMIT_REACHED“low watermark”异步事件。

struct ibv_srq_attr定义如下：

```cpp
struct ibv_srq_attr {
	uint32_t	max_wr;		/* SRQ中请求的未完成工作请求（WR）的最大数量 */
	uint32_t	max_sge;	/* 请求的每个WR散点元素的最大数量 */
	uint32_t	srq_limit;	/* SRQ的限制值（与ibv_create_srq不相关）*/
};
```
struct ibv_srq_attr完整说明如下：

|成员|说明|
|:--|:--|
|max_wr|可以发布到此SRQ的未完成WR的最大数量。值可以是[1~dev_cap.max_srq_wr]|
|max_sge|可以发布到此SRQ的任何WR中的SGE的最大数量。值可以是[1~dev_cap.max_srq_sge]**
|srq_limit|SRQ将配备的值。当SRQ中未完成的WRs数量低于此限制时，将生成关联的异步事件ibv_event_srq_limit_arrived。值可以是[0~可以提交到SRQ的WR数量]。0意味着不会生成SRQ限制事件(因为SRQ中未完成的WRs的数量不能是负数)。该值仅与iWARP有关，在IB中被忽略 ( 与ibv_create_srq无关) |

srq_attr_mask指定要修改的SRQ属性。它是0或位或一个或多个以下标志。

enum ibv_srq_attr_mask定义如下：
	  **
```cpp
enum ibv_srq_attr_mask {
	IBV_SRQ_MAX_WR	= 1 << 0,   /*调整SRQ大小 。字段srq_attr-> max_wr和srq_attr-> max_sge将被使用。
										仅当设备支持此操作时才可以调整SRQ的大小（在
										dev_cap.device_cap_flags中设置了IBV_DEVICE_SRQ_RESIZE）*/
	IBV_SRQ_LIMIT	= 1 << 1		//设置SRQ限制值。字段srq_attr->srq_limit将被使用。
};
```
使用SRQ限制事件机制的建议方法是:

1. 创建SRQ。
2. 将WR发布到该SRQ（在与QP关联之前或之后）。
3. 将SRQ限制设置为小于发布的WR数量的非零值。
4. 当SRQ中未完成的WR数量下降到限制以下时，将生成异步事件IBV_EVENT_SRQ_LIMIT_REACHED。 发生这种情况时，应执行以下操作：
   1. 确认异步事件
   2. 转到2）（即，将更多WR发布到SRQ并设置SRQ限制）

如果将SRQ限制设置为一个值，该值大于SRQ中当前未完成WR的数量，则将立即生成SRQ限制事件。

**示例：** 见ibv_destroy_srq。

**常见问题：**

Q：ibv_modify_srq()失败，该SRQ的属性现在是什么？
A：与调用ibv_modify_srq()之前完全相同，未对SRQ进行任何更改。

Q：我用值X武装了SRQ。可以将此值更改为Y吗？
A：是的你可以。 只需使用不同的SRQ限制值来调用ibv_modify_srq()。

### 6.1.4 ibv_query_srq

**函数原型：**
```cpp
 int ibv_query_srq(struct ibv_srq *srq, struct ibv_srq_attr *srq_attr)
```
**输入参数：** srq——共享接收队列，来自ibv_create_srq。

**输出参数：** srq_attr——SRQ的属性。详细信息见ibv_modify_srq。


**返回值：** 成功返回0；出错返回-1。如果调用失败，将设置errno来指示失败的原因。

* ENOMEM——没有足够的资源来完成这个操作。
* ENOSYS——此RDMA设备不支持查询共享接收队列。

**说明：** ibv_query_srq返回指定的SRQ的属性列表和当前值。 它通过指针srq_attr返回属性，该指针是上面在ibv_create_srq下说明的ibv_srq_attr结构体。 如果srq_attr中的srq_limit的值为0，则表明达到SRQ限制（“low watermark'”）事件没有（或不再）武装（not armed）。 在重新武装事件之前，不会生成任何异步事件。


**示例：**
```cpp
struct ibv_srq *srq;
struct ibv_srq_attr srq_attr;

if (ibv_query_srq(srq, &srq_attr)) {
	fprintf(stderr, "Error, ibv_query_srq() failed\n");
	return -1;
}
```

**常见问题：**

Q：当需要SRQ属性时，每次调用ibv_query_srq()都需要花费一些时间，我可以缓存一些属性吗?
A：实际上,可以。指示SRQ大小(即max_wr和max_sge)的属性不会更改，除非使用ibv_modify_srq()调整此SRQ的大小。srq_limit值可能会改变:当SRQ被武装时，它将保存低水印值。在生成SRQ limit事件之后，它将保留值0(除非重新武装SRQ)。




### 6.1.5 ibv_destroy_srq

**函数原型：**
```cpp
int ibv_destroy_srq(struct ibv_srq *srq)
```
**输入参数：** srq，要销毁的SRQ。来自ibv_create_srq或ibv_create_srq_ex。

**输出参数：** 无。

**返回值：** 成功返回0；出错返回-1。如果调用失败，将设置errno来指示失败的原因。EBUSY——一个或多个QPs仍然与SRQ关联。

**说明：**

ibv_destroy_srq销毁特定的SRQ。如果有任何QP与该SRQ关联则会失败。

如果使用ibv_get_async_event()读取了该SRQ上的任何关联异步事件，但仍未使用ibv_ack_async_event()进行确认，则对ibv_destroy_srq()的调用将永远不会结束，直到该事件被确认为止。

如果SRQ为空或包含未完成的未完成工作请求，则可以销毁SRQ。 同样，如果在生成关联的异步事件IBV_EVENT_SRQ_LIMIT_REACHED之前使用ibv_modify_srq()在该SRQ上请求了限制事件，则可以销毁它。


**示例：**

```cpp
struct ibv_pd *pd;
struct ibv_srq *srq;
struct ibv_srq_init_attr srq_init_attr;
struct ibv_srq_attr srq_attr;

 //创建SRQ
memset(&srq_init_attr, 0, sizeof(srq_init_attr));
srq_init_attr.attr.max_wr  = 1;
srq_init_attr.attr.max_sge = 2;
srq = ibv_create_srq(pd, &srq_init_attr);
if (!srq) {
	fprintf(stderr, "Error, ibv_create_srq() failed\n");
	return -1;
}

 //修改SRQ

memset(&srq_attr, 0, sizeof(srq_attr));
srq_attr.max_wr  = 10;
srq_attr.max_sge = 1;
srq_attr.srq_limit = 10
if (ibv_modify_srq(srq, &srq_attr, IBV_SRQ_MAX_WR| IBV_SRQ_LIMIT)) {
	fprintf(stderr, "Error, ibv_modify_srq() failed when resizing an SRQ\n");
	return -1;
}

//销毁SRQ
if (ibv_destroy_srq(srq)) {
	fprintf(stderr, "Error, ibv_destroy_srq() failed\n");
	return -1;
}
```

**常见问题：**

Q：ibv_destroy_srq()失败，与它相关的QPs会发生什么变化?
A：没事，您可以继续使用，而不会产生任何副作用。

Q：ibv_destroy_srq()失败，我可以知道哪些QP与之相关并导致了此失败吗？
A：不，目前RDMA堆栈没有这个功能。

Q：我可以销毁拥有未完成工作请求的SRQ吗？
A：是的你可以。 销毁SRQ时，是否包含工作请求都无关紧要。

Q：我在一个SRQ上调用ibv_modify_srq()来设置limit事件，但是还没有得到这个事件。我能摧毁SRQ吗?
A：是的,你可以。

Q：我调用了ibv_destroy_srq()，但是它没有结束。发生了什么事?
A：SRQ上，至少有个关联的异步事件被读取，但没有确认。


## 6.2 创建、查询、修改、打开、销毁QP
### 6.2.1 ibv_create_qp
**函数原型：**
```cpp
struct ibv_qp *ibv_create_qp(struct ibv_pd *pd, struct ibv_qp_init_attr *qp_init_attr)
```
**输入参数：**

* pd——来自ibv_dealloc_pd的struct ibv_pd。
* qp_init_attr——队列对的初始属性。详细信息见下文。

**输出参数：** qp_init_attr——队列对的实际属性。

**返回值：** 成功返回指向QP的指针，失败返回NULL。并设置errno只是失败原因：

* EINVAl——pd, send_cq, recv_cq, srq非法，或者max_send_wr, max_recv_wr, max_send_sge, max_recv_sge 或 max_inline_data非法。
* ENOMEM——内存不足。
* ENOSYS——此RDMA设备不支持此传输服务类型的QP。
* EPERM——没有足够的权限创建具有此传输服务类型的QP。

**说明：**

ibv_create_qp创建一个与保护域pd关联的QP。当一个QP被创建时，它将进入RESET状态。

用户可以定义QP的最小属性：发送/接收队列的工作请求数量和每个工作请求的分散/收集条目数。 函数使用实际值更新qp_init_attr->cap结构体，因此实际属性可以等于或高于请求的值。

如果要求创建与SRQ关联的IBV_QPT_RC或IBV_QPT_UD以外的其他类型的QP，则ibv_create_qp()将失败。

如果要将QP与SRQ关联，则ibv_create_qp()将忽略属性max_recv_wr和max_recv_sge。

struct ibv_qp_init_attr定义如下：
```cpp
struct ibv_qp_init_attr {
	void				*qp_context;	//(可选)用户定义的值，将在qp->qp_context中可用
	struct ibv_cq		*send_cq;		//与SQ关联的CQ。必须在调用ibv_create_qp之前创建。由ibv_create_cq返回
	struct ibv_cq		*recv_cq;		/*与RQ关联的CQ。必须在调用ibv_create_qp之前创建。
											由ibv_create_cq返回，它可能与send_cq相同*/
	struct ibv_srq		*srq;			//(可选)从ibv_create_srq返回的SRQ，此QP将与之关联。否则,为NULL
	struct ibv_qp_cap	cap;			/*QP大小的属性，如下表所述。 成功创建队列对后，
											此结构体将保留实际的队列对属性*/
	enum ibv_qp_type	qp_type;		//QP传输服务类型，枚举值之一，详细信息见下文						
	int					sq_sig_all;		//被发布到在此QP的SQ中的WR的信号级别。详细信息见下文	
};
```
InfiniBand规范定义了QP传输类型：可靠数据报。 但是，RDMA软件堆栈既不支持它，也不支持任何RDMA设备。

send_cq和recv_cq可以是相同的CQ，也可以是不同的CQ。

RC和UD QP始终可以与SRQ相关联。 有些RDMA设备也允许UC QP与SRQ关联。 但是，目前没有任何迹象表明RDMA设备支持此功能。


struct ibv_qp_cap的定义如下:
```cpp
struct ibv_qp_cap {
	uint32_t max_send_wr;		//发送队列中未完成的最大发送请求数
	uint32_t max_recv_wr;		//接收队列中未完成的最大发送请求数
	uint32_t max_send_sge;		//发送队列上一个WR中分散/聚集元素(SGE)的最大数量
	uint32_t max_recv_sge;		//接收队列上一个WR中最大的SGEs数量。
	uint32_t max_inline_data;	//发送队列中内联数据的最大字节数
};
```
struct ibv_qp_cap的完整说明如下:
|成员|说明|
|:--|:--|
|max_send_wr	|可以发布到该QP的SQ中的未完成WR的最大数量。值可以是0~dev_cap.max_qp_wr。对于特定的传输类型，可能有RDMA设备支持的未完成WR少于最大报告值。
|max_recv_wr	|可以发布到该QP的RQ中的未完成WR的最大数量。值可以是0~dev_cap.max_qp_wr。对于特定的传输类型，可能有RDMA设备支持的未完成WR少于最大报告值。如果QP与SRQ关联，则忽略此值
|max_send_sge	|可以发布到该QP的SQ中的WR的分散/聚集元素的最大数量。 值可以是0~dev_cap.max_sge。 可能有一些RDMA设备对于特定的传输类型可能支持的分散/聚集元素少于最大报告值。对于特定的传输类型，可能存在支持比最大报告值更少的散射/聚集元素的RDMA设备。
|max_recv_sge	|可以发布到该QP的RQ中的WR的分散/聚集元素的最大数量。 值可以是0~dev_cap.max_sge。 可能有一些RDMA设备对于特定的传输类型可能支持的分散/聚集元素少于最大报告值。对于特定的传输类型，可能存在支持比最大报告值更少的散射/聚集元素的RDMA设备。如果QP与SRQ关联，则忽略此值
|max_inline_data|可以内联发布到SQ的最大消息大小(以字节为单位)。如果没有内联消息请求则为0

发送内联数据是任何RDMA规范中都未定义的实现扩展：它允许在发布到RDMA设备的工作请求（而不是分散/收集条目）中发送数据本身。 保存此消息的内存不必注册。 没有任何动词指定可在QP中内联发送的最大大小。 一些RDMA设备支持它。 在某些RDMA设备中，创建QP将设置max_inline_data为消息大小，这些消息可以被SQ中请求的数量的SGE发送。 如果是其他版本，则应在创建QP之前明确指定要内联发送的消息大小。 对于这些设备，建议尝试创建具有所需消息大小的QP，如果QP创建失败，则继续减小它。


enum ibv_qp_type定义如下：
```cpp
enum ibv_qp_type {
	IBV_QPT_RC			= 2,		//rc
	IBV_QPT_UC,						//uc
	IBV_QPT_UD,						//ud
	IBV_QPT_RAW_PACKET	= 8,
	IBV_QPT_XRC_SEND	= 9,
	IBV_QPT_XRC_RECV,
	IBV_QPT_DRIVER		= 0xff,	//不代表特定服务，而是用于特定于供应商的QP逻辑
};
```


sq_sig_all取值定义如下：
|值|说明|
|:--|:--|
|0|所有提交到SQ的WR都将生成一个工作完成（WC）|
|非0|如果将此值设置为0，在提交到SQ的每个WR中，用户都必须决定是否要为成功完成的WR产生WC（请参阅ibv_post_send）。|


struct ibv_qp的定义如下：

```cpp
struct ibv_qp {
	struct ibv_context	*context;			//动词上下文，详细信息见ibv_open_divece
	void				*qp_context;
	struct ibv_pd		*pd;				//保护域，详细信息见ibv_alloc_pd
	struct ibv_cq		*send_cq;			//发送完成队列
	struct ibv_cq		*recv_cq;			//接收完成队列
	struct ibv_srq		*srq;				//共享接收队列
	uint32_t			handle;				//句柄
	uint32_t			qp_num;				//qp编号
	enum ibv_qp_state	state;				//qp状态，详细信息见ibv_query_qp
	enum ibv_qp_type	qp_type;			//qp类型

	pthread_mutex_t		mutex;				//锁
	pthread_cond_t		cond;				//条件变量
	uint32_t			events_completed;	//已完成的事件
};
```

**示例：** 见 ibv_destroy_qp。

**常见问题：**
Q：QP有什么好处？
A：QP是在RDMA体系结构(类似于套接字)中发送和接收数据的实际对象。

Q：QP与Socket相等吗？
A：不完全是。 套接字是一种抽象，由网络堆栈维护，并且背后没有物理资源。 QP是RDMA设备的资源，并且一个进程可以同时使用一个QP号（类似于与特定TCP或UDP端口号关联的套接字）

Q：我可以将多个QP与同一个SRQ相关联吗？
A：可以。

Q：哪些QP传输类型可以与SRQ相关联？
A：所有RDMA设备都可以将RC和UD QP与SRQ关联。 在某些RDMA设备中，您也可以将UC QP与SRQ关联。

Q：如果我将QP与SRQ相关联，是否需要设置接收队列属性？
A：不，你不必这么做。如果QP与SRQ相关联，则接收队列属性将被完全忽略。

Q：我可以在发送和接收队列中使用相同的CQ吗?
A：可以。


Q：我可以在发送队列中使用一个CQ，在接收队列中使用另一个CQ吗?
A：可以。

Q：我如何知道在QP中内联发送的最大消息大小是多少?
A：你不可能知道这个信息。此信息不可用。你通过反复试验找到了这些信息。


Q：我创建了传输类型为X的QP，并且成功了。 我试图创建传输类型为Y的QP，但QP创建失败。 发生了什么？
A：dev_cap.max_sge和dev_cap.max_qp_wr中的值报告了任何QP传输类型支持的分散/聚集条目和工作请求的支持值。 但是，对于特定的RDMA设备，可能存在无法使用这些最大值创建的QP传输类型。 通过反复试验，应该为该特定的RDMA设备获得正确的属性。

Q：设备功能报告说max_qp_wr/max_sge是X，但是当我尝试使用这些属性创建QP时，它失败了。 发生了什么？
A：dev_cap.max_sge和dev_cap.max_qp_wr中的值报告任何工作队列（发送和接收）支持的分散/聚集条目和工作请求的最大支持值。 但是，对于特定的RDMA设备，对于发送或接收队列，可能还有其他考虑因素，这些因素会阻止使用这些最大值创建QP。 通过反复试验，应该为该特定的RDMA设备获得正确的属性。

### 6.2.2 ibv_create_qp_ex

**函数原型：**
```cpp
struct ibv_qp *ibv_create_qp_ex(struct ibv_context *context,
						struct ibv_qp_init_attr_ex *qp_init_attr);
```
**输入参数：**

* context——设备上下文，来自ibv_open_device。
* qp_init_attr——QP的初始扩展属性。详细信息见下文。

**输出参数：** qp_init_attr——队列对的真实扩展属性。

**返回值：** 成功返回指向QP的指针，失败返回NULL。并设置errno只是失败原因。检查返回的QP中的QP编号（qp_num）。


**说明：**

ibv_create_qp_ex创建一个与保护域pd关联的QP。

函数ibv_create_qp_ex将使用创建的QP的实际QP值更新qp_init_attr_ex-> cap结构。该值将大于或等于请求的值。

如果要将QP与SRQ关联，则ibv_create_qp_ex()将忽略属性max_recv_wr和max_recv_sge。

仅在UD QP上支持属性source_qpn，没有流量控制RX应该是不可能的。

当QP创建属性IBV_QP_INIT_ATTR_SEND_OPS_FLAGS时，请使用ibv_qp_to_qp_ex()获取用于访问send ops迭代器接口的ibv_qp_ex。


struct ibv_qp_init_attr_ex定义如下：

```cpp
struct ibv_qp_init_attr_ex {
	void						*qp_context;	/* QP管理的上下文 */
	struct ibv_cq				*send_cq;		/* 要与SQ关联的CQ，详细信息见ibv_create_cq */
	struct ibv_cq				*recv_cq;		/* 要与RQ关联的CQ，详细信息见ibv_create_cq */
	struct ibv_srq				*srq;			/* 要与QP关联的SRQ，否则为空，详细信息见ibv_create_srq */
	struct ibv_qp_cap			cap;			/* QP功能，详细信息见ibv_create_qp */
	enum ibv_qp_type			qp_type;		/* QP传输服务类型，一个枚举值，详细信息见ibv_create_qp */
	int							sq_sig_all;		/* 被发布到QP的SQ中的WR的信号级别。详细信息见ibv_create_qp */
	uint32_t					comp_mask;		/* 标识合法字段，按位或enum ibv_qp_init_attr_mask，详情见下文*/
	struct ibv_pd				*pd;			/* 要和该QP关联的PD，详细信息见ibv_alloc_pd */
	struct ibv_xrcd			*xrcd;			/* 要与目标QP关联的XRC域，详细信息见ibv_open_xrcd */
	enum ibv_qp_create_flags	create_flags;	/* QP的创建标志，枚举值按位或，详细信息见下文 */
	uint16_t					max_tso_header;	/* TSO头部的最大值 */
	struct ibv_rwq_ind_table	*rwq_ind_tbl;	/* 与QP关联的间接表 ,用于在不同的接收工作队列之间分配传入,
													的数据包。 将接收WQ与不同的CPU内核相关联，可以将工作
													负荷分配到不同的CPU核心。间接表只能包含IBV_WQT_RQ类型的WQ。
													详细信息见ibv_create_rwq_ind_table*/
	struct ibv_rx_hash_conf	rx_hash_conf;  /* 要使用的RX哈希配置*/
	uint32_t					source_qpn;		/* 源QP编号，应使用创建标志IBV_QP_CREATE_SOURCE_QPN*/
	uint64_t					send_ops_flags;	/* 选择将在结构体ibv_qp_ex中定义的QP发送操作。**
													使用枚举ibv_qp_create_send_ops_flags  */
};
```

 enum ibv_qp_init_attr_mask定义如下：
```cpp 
enum ibv_qp_init_attr_mask {
	IBV_QP_INIT_ATTR_PD					= 1 << 0,
	IBV_QP_INIT_ATTR_XRCD				= 1 << 1,
	IBV_QP_INIT_ATTR_CREATE_FLAGS		= 1 << 2,
	IBV_QP_INIT_ATTR_MAX_TSO_HEADER		= 1 << 3,
	IBV_QP_INIT_ATTR_IND_TABLE			= 1 << 4,
	IBV_QP_INIT_ATTR_RX_HASH			= 1 << 5,
	IBV_QP_INIT_ATTR_SEND_OPS_FLAGS		= 1 << 6,
};
```


enum ibv_qp_create_flags定义如下：
```cpp
enum ibv_qp_create_flags {
	IBV_QP_CREATE_BLOCK_SELF_MCAST_LB	= 1 << 1,		/* 防止自多播回路 */
	IBV_QP_CREATE_SCATTER_FCS			= 1 << 8,		/* FCS字段将分散到主机内存中*/
	IBV_QP_CREATE_CVLAN_STRIPPING		= 1 << 9,		/* CVLAN字段将从传入数据包中剥离*/
	IBV_QP_CREATE_SOURCE_QPN			= 1 << 10,	/* 创建的QP将使用source_qpn作为其线路QP号 */
	IBV_QP_CREATE_PCI_WRITE_END_PADDING	= 1 << 11,	/* 传入的数据包将被填充为缓存行大小 */
};
```

struct ibv_rx_hash_conf 定义如下：
```cpp
struct ibv_rx_hash_conf {
	uint8_t		rx_hash_function;		/* RX哈希函数，使用enum ibv_rx_hash_function_flags */
	uint8_t		rx_hash_key_len;		/* RX哈希密钥长度 */
	uint8_t		*rx_hash_key;			/* RX哈希密钥数据 */
	uint64_t	rx_hash_fields_mask;	/* 应该参与哈希处理的RX字段，使用enum ibv_rx_hash_fields */
};

```
enum ibv_rx_hash_function_flags定义如下：

```cpppp
enum ibv_rx_hash_function_flags {
	IBV_RX_HASH_FUNC_TOEPLITZ	= 1 << 0,
};
```
 enum ibv_rx_hash_fields 定义如下：
```cpp
 enum ibv_rx_hash_fields {
	IBV_RX_HASH_SRC_IPV4		= 1 << 0,
	IBV_RX_HASH_DST_IPV4		= 1 << 1,
	IBV_RX_HASH_SRC_IPV6		= 1 << 2,
	IBV_RX_HASH_DST_IPV6		= 1 << 3,
	IBV_RX_HASH_SRC_PORT_TCP	= 1 << 4,
	IBV_RX_HASH_DST_PORT_TCP	= 1 << 5,
	IBV_RX_HASH_SRC_PORT_UDP	= 1 << 6,
	IBV_RX_HASH_DST_PORT_UDP	= 1 << 7,
	IBV_RX_HASH_IPSEC_SPI		= 1 << 8,
						//使用隧道协议时，例如 VXLAN，那么我们有一个内部（封装数据包）和外部。
						//为了将RSS应用于内部数据包，应使用L3/L4字段之一设置以下字段。
	IBV_RX_HASH_INNER			= (1UL << 31),
};
```
 struct ibv_qp_create_send_ops_flags 定义如下：
```cpp
struct ibv_qp_create_send_ops_flags {
	IBV_QP_EX_WITH_RDMA_WRITE			= 1 << 0,
	IBV_QP_EX_WITH_RDMA_WRITE_WITH_IMM	= 1 << 1,
	IBV_QP_EX_WITH_SEND					= 1 << 2,
	IBV_QP_EX_WITH_SEND_WITH_IMM		= 1 << 3,
	IBV_QP_EX_WITH_RDMA_READ			= 1 << 4,
	IBV_QP_EX_WITH_ATOMIC_CMP_AND_SWP	= 1 << 5,
	IBV_QP_EX_WITH_ATOMIC_FETCH_AND_ADD	= 1 << 6,
	IBV_QP_EX_WITH_LOCAL_INV			= 1 << 7,
	IBV_QP_EX_WITH_BIND_MW				= 1 << 8,
	IBV_QP_EX_WITH_SEND_WITH_INV		= 1 << 9,
	IBV_QP_EX_WITH_TSO					= 1 << 10,
};
```


### 6.2.3 ibv_qp_to_qp_ex
**函数原型：**
```cpp
struct ibv_qp_ex *ibv_qp_to_qp_ex(struct ibv_qp *qp);
```
**输入参数：** qp——来自ibv_create_qp的队列对。

**输出参数：** 无。


**返回值：** 成功返回指向ibv_qp_ex的指针；失败返回NULL，并设置errno来指示失败的原因。


**说明：** ibv_qp_to_qp_ex将qp转为对应的qp_ex。

struct ibv_qp_ex定义如下：

```cpp
struct ibv_qp_ex {
	struct ibv_qp	qp_base;
	uint64_t		comp_mask; 

	uint64_t		wr_id;
	unsigned int	wr_flags;	/* bitmask from enum ibv_send_flags */

	void 			(*wr_atomic_cmp_swp)(struct ibv_qp_ex *qp, uint32_t rkey,
						uint64_t remote_addr, uint64_t compare,uint64_t swap);
	void			(*wr_atomic_fetch_add)(struct ibv_qp_ex *qp, uint32_t rkey,
						uint64_t remote_addr, uint64_t add);
	void			(*wr_bind_mw)(struct ibv_qp_ex *qp, struct ibv_mw *mw,uint32_t rkey,
						const struct ibv_mw_bind_info *bind_info);
	void			(*wr_local_inv)(struct ibv_qp_ex *qp, uint32_t invalidate_rkey);
	void			(*wr_rdma_read)(struct ibv_qp_ex *qp, uint32_t rkey,
						uint64_t remote_addr);
	void			(*wr_rdma_write)(struct ibv_qp_ex *qp, uint32_t rkey,uint64_t remote_addr);
	void			(*wr_rdma_write_imm)(struct ibv_qp_ex *qp, uint32_t rkey,
						uint64_t remote_addr, __be32 imm_data);

	void			(*wr_send)(struct ibv_qp_ex *qp);
	void			(*wr_send_imm)(struct ibv_qp_ex *qp, __be32 imm_data);
	void			(*wr_send_inv)(struct ibv_qp_ex *qp, uint32_t invalidate_rkey);
	void			(*wr_send_tso)(struct ibv_qp_ex *qp, void *hdr, uint16_t hdr_sz,uint16_t mss);

	void			(*wr_set_ud_addr)(struct ibv_qp_ex *qp, struct ibv_ah *ah,
						uint32_t remote_qpn, uint32_t remote_qkey);
	void			(*wr_set_xrc_srqn)(struct ibv_qp_ex *qp, uint32_t remote_srqn);

	void			(*wr_set_inline_data)(struct ibv_qp_ex *qp, void *addr, size_t length);
	void			(*wr_set_inline_data_list)(struct ibv_qp_ex *qp, size_t num_buf,
				  		const struct ibv_data_buf *buf_list);
	void			(*wr_set_sge)(struct ibv_qp_ex *qp, uint32_t lkey, uint64_t addr, uint32_t length);
	void			(*wr_set_sge_list)(struct ibv_qp_ex *qp, size_t num_sge, const struct ibv_sge *sg_list);

	void			(*wr_start)(struct ibv_qp_ex *qp);
	int				(*wr_complete)(struct ibv_qp_ex *qp);
	void			(*wr_abort)(struct ibv_qp_ex *qp);
};
```

### 6.2.4 ibv_query_qp
**函数原型：**
```cpp
int ibv_query_qp(struct ibv_qp *qp, struct ibv_qp_attr *attr, 
					enum ibv_qp_attr_mask attr_mask, struct ibv_qp_init_attr *init_attr)
```
**输入参数：**
* qp——来自ibv_create_qp或ibv_create_qp_ex的struct ibv_qp。
* attr_mask——要查询的字段的位掩码， enum ibv_qp_attr_mask按位或，详细信息见下文。

**输出参数：**
* attr——将被请求的属性填充的struct ibv_qp_attr。
* init_attr——将被初始属性填充的struct ibv_qp_init_attr。详细信息见ibv_create_qp。

**返回值：** 成功返回0；失败返回errno指示失败的原因。ENOMEM——内存不足。

**说明：**
ibv_query_qp获取通过队列对（QP）的各种属性，并通过attr和init_attr返回。

参数attr_mask是一个提示，它指定要检索的最小属性列表。 某些RDMA设备可能会返回未请求的其他属性，例如，如果可以低代价地返回该值。 该格式与ibv_modify_qp()中的格式相同。

如果使用ibv_modify_qp()设置了属性值，则这些属性值有效。 有效属性的确切列表取决于QP状态。

多次调用ibv_query_qp()可能会导致以下属性的返回值有所不同：qp_state，path_mig_state，sq_draining，ah_attr（如果启用了APM）。

用户应分配一个struct ibv_qp_attr和一个struct ibv_qp_init_attr，并将它们传递给命令。 成功返回后，将填充这些结构体。 ***用户负责释放这些结构体***。

struct ibv_qp_attr 定义如下：

```cpp
struct ibv_qp_attr {
	enum ibv_qp_state	qp_state;				/* 要转移的QP状态，使用enum ibv_qp_state，详细信息见下文 */
	enum ibv_qp_state	cur_qp_state;			/* 假设这是当前的QP状态，使用enum ibv_qp_state，详细信息见下文*/
	enum ibv_mtu		path_mtu;				/* 路径MTU（仅对RC/UC QP有效），详细信息见ibv_query_port*/
	enum ibv_mig_state	path_mig_state;			/* 路径迁移状态（如果HCA支持APM，则有效) */
	uint32_t			qkey;					/* QP的Q_key（仅对UD QP有效） */
	uint32_t			rq_psn;					/* 用于接收队列的PSN（仅对RC/UC QP有效） */
	uint32_t			sq_psn;					/* 发送队列的PSN（仅对RC/UC QP有效） */
	uint32_t			dest_qp_num;			/* 目标QP编号（仅对RC/UC QP有效） */
	int					qp_access_flags;		/* 启用的远程访问操作的掩码（仅对RC/UC QP有效）,
													按位或enum ibv_access_flags ，详细信息见ibv_reg_mr*/
	struct ibv_qp_cap	cap;					/* QP容量（如果HCA支持调整QP大小则有效），
													详细信息见ibv_create_qp*/
	struct ibv_ah_attr	ah_attr;				/* 主路径地址向量（仅对RC/UC QP有效），详细信息见ibv_create_ah */
	struct ibv_ah_attr	alt_ah_attr;			/* 备用路径地址向量（仅对RC/UC QP有效，详细信息见ibv_create_ah */
	uint16_t			pkey_index;				/* 主 P_Key索引 */
	uint16_t			alt_pkey_index;			/* 备用 P_Key索引*/
	uint8_t				en_sqd_async_notify;	/* 启用SQD。耗尽异步通知（仅在qp_state为SQD时有效） */
	uint8_t				sq_draining;			/* QP正在排尽吗？与ibv_modify_qp()不相关 */
	uint8_t				max_rd_atomic;			/* 目标QP上未完成的RDMA读取和原子操作的数量（仅对RC QP有效）*/
	uint8_t				max_dest_rd_atomic;		/*用于处理传入RDMA读取和原子操作的响应器资源数量（仅对RC QP有效）*/
	uint8_t				min_rnr_timer;			/* 最小RNR NAK计时器（仅对RC QP有效），详细信息见下文*/
	uint8_t				port_num;				/* 主端口号 */
	uint8_t				timeout;				/* 主路径的本地确认超时（仅对RC QP有效），详细信息见下文*/
	uint8_t				retry_cnt;				/* 重试计数（仅对RC QP有效） */
	uint8_t				rnr_retry;				/*  RNR重试（仅对RC QP有效） */
	uint8_t				alt_port_num;			/* 备用端口号 */
	uint8_t				alt_timeout;			/*备用路径的本地确认超时（仅对RC QP有效） */
	uint32_t			rate_limit;				/* Packet pacing的速率限制（kbps）*/
};
```
struct ibv_qp_attr的完整说明如下：
|成员|说明|
|:--|:--|
|qp_state|下一个QP状态。使用enum ibv_qp_state ，详细信息见下文|
|cur_qp_state|假设这是当前的QP状态，使用enum ibv_qp_state ，详细信息见下文。 如果应用程序知道QP状态与低级驱动程序假定的状态不同，这将很有用。 与ibv_query_qp()无关，它是qp_state枚举值之一|
|path_mtu|路径MTU（最大传输单元），即可以在路径中传输的数据包的最大有效负载大小。 对于UC和RC QPs，当需要时，RDMA设备将自动将消息分段到这个大小的数据包。它是一个枚举值，详细信息见ibv_qusey_port|
|path_mig_state|QP路径迁移状态机的状态（如果设备支持：在dev_cap.device_cap_flags中设置了IBV_DEVICE_AUTO_PATH_MIG）。它是一个枚举值，详细信息见下文|
|qkey|将检查传入消息的Q_Key并将其用作传出Q_Key（如果设置了发送请求中q_key的MSB）。 仅与UD QP相关|
|rq_psn|RC和UC QP的已接收数据包的数据包序列号的24位值|
|sq_psn|任何QP的已发送数据包的数据包序列号的24位值|
|dest_qp_num|RC和UC QP的远程QP编号的24位值； 发送数据时，数据包将被发送到该QP编号，而接收数据时，将仅从该QP编号接受数据包|
|qp_access_flags|RC和UC QPs的传入包的远程操作的允许访问标志。按位或enum ibv_access_flags，详细信息见下文|
|cap|QP中的WR的数量。详细信息见ibv_create_qp。
|ah_attr|主路径的地址向量，说明到远程QP的路径信息，详细信息见ibv_create_ah。
|alt_ah_attr|备用路径的地址向量，说明到远程QP的路径信息，详细信息见ibv_create_ah。仅当设备支持时才可以使用（在dev_cap.device_cap_flags中设置了IBV_DEVICE_AUTO_PATH_MIG）
|pkey_index|主P_Key索引。 从这个QP发出的数据包将与P_Key表中的条目的值一起发送，而到这个QP的传入数据包将在主路径中得到验证
|alt_pkey_index|备用P_Key索引。 从这个QP发出的数据包将与P_Key表中的条目的值一起发送，而到这个QP的传入数据包将在备用路径中得到验证
|en_sqd_async_notify|如果不为零，则在QP状态变为SQD时生成关联的异步事件IBV_EVENT_SQ_DRAINED。耗尽，例如发送队列被耗尽。 与ibv_query_qp()无关。
|sq_draining|如果设置，则表示发送队列正在耗尽。仅当QP处于SQD状态时才相关
|max_rd_atomic|作为发起方，在任意时刻，此QP可以处理的RDMA读取和原子操作的数量。 仅与RC QP相关|
|max_dest_rd_atomic|作为目的地，在任意时刻，此QP可以处理的RDMA读取和原子操作的数量。仅与RC QP相关|
|min_rnr_timer|最小RNR NAK计时器字段值。 当到此QP的传入消息应消耗RQ中的一个WR，但该队列上没有未完成WR时，QP将向发起方发送RNR NAK数据包。 它不会影响由于其他原因RNR NAK发送。 因为这些值没有被枚举，详细信息见下文|
|port_num|与此QP关联的主物理端口号|
|timeout|QP在重传数据包之前等待来自远程QP的ACK / NACK的最长时间。 零值是特殊值，这意味着等待无限久（对于调试很有用）。 对于任何其他值，时间计算为：4.096\*2^timeout^ usec。 详细信息见下文。
|retry_cnt|一个3比特位的值，表示由于远程端在主路径中没有应答，QP在报告错误之前尝试重新发送数据包的总次数|
|rnr_retry|一个3比特位的值，表示当远程QP发送RNR NACK时，QP在报告错误之前尝试重新发送数据包的总次数。7是特殊值，表示在RNR时，重传无限次|
|alt_port_num|与此QP关联的备用物理端口号
|alt_timeout|类似timeout，备用路径中，QP在重传数据包之前等待来自远程QP的ACK / NACK的最长时间。详细信息见下文。
| rate_limit|Packet pacing的速率限制（kbps）|

关于QP属性中的一些特定值有几点需要注意:
* 在开发过程中，将attr.retry_cnt和attr.rnr_retry设置为0是一个好习惯。如果代码中存在竞争，最好在开发阶段检测它们。
* attr.timeout值为0意味着等待无限长的ACK或NACK。这意味着，如果消息中的任何数据包丢失，且没有发送ACK或NACK，则不会发生重发，QP将停止发送
* attr.rnr_retry值为7意思是在远程QP发送RNR Nack时，将无限次重试发送消息。


enum ibv_qp_state 定义如下：

```cpp
enum ibv_qp_state {
	IBV_QPS_RESET,		//重置状态
	IBV_QPS_INIT,		//初始化状态
	IBV_QPS_RTR,		//准备接收
	IBV_QPS_RTS,		//准备发送
	IBV_QPS_SQD,		//发送队列耗尽
	IBV_QPS_SQE,		//发送队列出错
	IBV_QPS_ERR,		//错误
	IBV_QPS_UNKNOWN		//未知
};
```
enum ibv_mig_state定义如下：

```cpp
enum ibv_mig_state {
	IBV_MIG_MIGRATED,	//Migrated状态，即迁移的初始状态已经完成
	IBV_MIG_REARM,		//Rarmed状态，即尝试尝试协调远程RC QP以将本地QP和远程QP都移入Armed状态|
	IBV_MIG_ARMED		//Armed状态，本地和远程QPs都准备好执行路径迁移
  };
```

min_rnr_timer取值说明如下：
|值|说明（ms）|值|说明（ms）|值|说明（ms）|值|说明（ms）|
|:--|:--|:--|:--|:--|:--|:--|:--|
|0|655.36|8|5.12|16|0.32|24|40.96|
|1|0.01|9|7.68|17|0.48|25|61.44|
|2|0.02|10|10.24|18|0.64|26|81.92|
|3|0.03|11|15.36|19|0.96|27|122.88|
|4|0.04|12|0.08|20|1.28|28|163.84|
|5|0.06|13|0.12|21|1.92|29|245.76|
|6|2.56|14|0.16|22|20.48|30|327.68|
|7|3.84|15|0.24|23|30.72|31|491.52|

timeout取值说明如下：
|值|说明（us）|值|说明（ms）|值|说明（s）|值|说明（s）|
|:--|:--|:--|:--|:--|:--|:--|:--|
|0|无限|8| 1.048|16|0.268|24|68.7|
|1|8.192|9|2.097|17|0.536|25|137|
|2|16.384|10|4.194|18|1.07|26|275|
|3|32.768|11|8.389|19|2.14|27|550|
|4|65.536|12|16.777|20|4.29|28|1100|
|5|131.072|13|33.554|21|8.58|29|2200|
|6|262.144|14|67.109|22|17.1|30|4400|
|7|524.288|15|134.217|23|34.3|31|8800|


 enum ibv_qp_attr_mask定义如下：
```cpp
enum ibv_qp_attr_mask {
	IBV_QP_STATE				= 1 <<  0,	//填充attr->qp_state
	IBV_QP_CUR_STATE			= 1 <<  1,	//填充 attr->cur_qp_state
	IBV_QP_EN_SQD_ASYNC_NOTIFY	= 1 <<  2,	//填充 attr->en_sqd_async_notify
	IBV_QP_ACCESS_FLAGS			= 1 <<  3,	//填充 attr->qp_access_flags
	IBV_QP_PKEY_INDEX			= 1 <<  4,	//填充attr->pkey_index
	IBV_QP_PORT					= 1 <<  5,	//填充 attr->port_num
	IBV_QP_QKEY					= 1 <<  6,	//填充attr->qkey
	IBV_QP_AV					= 1 <<  7,	//填充 attr->ah_attr
	IBV_QP_PATH_MTU				= 1 <<  8,	//填充 attr->path_mtuattr->timeout
	IBV_QP_TIMEOUT				= 1 <<  9,	//填充attr->timeout
	IBV_QP_RETRY_CNT			= 1 << 10,	//填充 attr->retry_cnt
	IBV_QP_RNR_RETRY			= 1 << 11,	//填充attr->rnr_retry
	IBV_QP_RQ_PSN				= 1 << 12,	//填充attr->rq_psn
	IBV_QP_MAX_QP_RD_ATOMIC		= 1 << 13,	//填充attr->max_rd_atomic
	IBV_QP_ALT_PATH				= 1 << 14,	//填充attr->alt_*
	IBV_QP_MIN_RNR_TIMER		= 1 << 15,	//填充 attr->min_rnr_timer
	IBV_QP_SQ_PSN				= 1 << 16,	//填充 attr->sq_psn
	IBV_QP_MAX_DEST_RD_ATOMIC	= 1 << 17,	//填充attr->max_dest_rd_atomic
	IBV_QP_PATH_MIG_STATE		= 1 << 18,	//填充attr->path_mig_state
	IBV_QP_CAP					= 1 << 19,	//填充 attr->cap
	IBV_QP_DEST_QPN				= 1 << 20,	//填充attr->dest_qp_num
	/* 这些位在较旧的内核上受支持，但从未在libibverbs中公开：
	_IBV_QP_SMAC				= 1 << 21,
	_IBV_QP_ALT_SMAC			= 1 << 22,
	_IBV_QP_VID					= 1 << 23,
	_IBV_QP_ALT_VID				= 1 << 24,
	*/
	IBV_QP_RATE_LIMIT			= 1 << 25,	//填充attr->rate_limit
};**
```
下表指定在每种状态下服务类型为IBV_QPT_UD的QP的有效属性：

|状态|有效属性|
|:--|:--|
|RESET|	IBV_QP_STATE
|INIT|	IBV_QP_STATE, IBV_QP_PKEY_INDEX, IBV_QP_PORT, IBV_QP_QKEY
|RTR|	IBV_QP_STATE, IBV_QP_PKEY_INDEX, IBV_QP_PORT, IBV_QP_QKEY
|RTS|	IBV_QP_STATE, IBV_QP_PKEY_INDEX, IBV_QP_PORT, IBV_QP_QKEY, IBV_QP_SQ_PSN
|SQD|	IBV_QP_STATE, IBV_QP_PKEY_INDEX, IBV_QP_PORT, IBV_QP_QKEY, IBV_QP_SQ_PSN
|SQE|	IBV_QP_STATE, IBV_QP_PKEY_INDEX, IBV_QP_PORT, IBV_QP_QKEY, IBV_QP_SQ_PSN
|ERR|	IBV_QP_STATE

下表指定在每种状态下服务类型为IBV_QPT_UC的QP的有效属性：

|状态|有效属性|
|:--|:--|
|RESET|	IBV_QP_STATE
|INIT|	IBV_QP_STATE, IBV_QP_PKEY_INDEX, IBV_QP_PORT, IBV_QP_ACCESS_FLAGS
|RTR|	IBV_QP_STATE, IBV_QP_PKEY_INDEX, IBV_QP_PORT, IBV_QP_ACCESS_FLAGS, IBV_QP_AV, IBV_QP_PATH_MTU, IBV_QP_DEST_QPN, IBV_QP_RQ_PSN, IBV_QP_ALT_PATH
|RTS|	IBV_QP_STATE, IBV_QP_PKEY_INDEX, IBV_QP_PORT, IBV_QP_ACCESS_FLAGS, IBV_QP_AV, IBV_QP_PATH_MTU, IBV_QP_DEST_QPN, IBV_QP_RQ_PSN, IBV_QP_ALT_PATH, IBV_QP_SQ_PSN, IBV_QP_PATH_MIG_STATE
|SQD|	IBV_QP_STATE, IBV_QP_PKEY_INDEX, IBV_QP_PORT, IBV_QP_ACCESS_FLAGS, IBV_QP_AV, IBV_QP_PATH_MTU, IBV_QP_DEST_QPN, IBV_QP_RQ_PSN, IBV_QP_ALT_PATH, IBV_QP_SQ_PSN, IBV_QP_PATH_MIG_STATE
|SQE|	IBV_QP_STATE, IBV_QP_PKEY_INDEX, IBV_QP_PORT, IBV_QP_ACCESS_FLAGS, IBV_QP_AV, IBV_QP_PATH_MTU, IBV_QP_DEST_QPN, IBV_QP_RQ_PSN, IBV_QP_ALT_PATH, IBV_QP_SQ_PSN, IBV_QP_PATH_MIG_STATE
|ERR|	IBV_QP_STATE

下表指定在每种状态下服务类型为IBV_QPT_RC的QP的有效属性：

|状态|有效属性|
|:--|:--|
|RESET|	IBV_QP_STATE|
|INIT|	IBV_QP_STATE, IBV_QP_PKEY_INDEX, IBV_QP_PORT, IBV_QP_ACCESS_FLAGS
|RTR|	IBV_QP_STATE, IBV_QP_PKEY_INDEX, IBV_QP_PORT, IBV_QP_ACCESS_FLAGS, IBV_QP_AV, IBV_QP_PATH_MTU, IBV_QP_DEST_QPN, IBV_QP_RQ_PSN, IBV_QP_MAX_DEST_RD_ATOMIC, IBV_QP_MIN_RNR_TIMER, IBV_QP_ALT_PATH
|RTS|	IBV_QP_STATE, IBV_QP_PKEY_INDEX, IBV_QP_PORT, IBV_QP_ACCESS_FLAGS, IBV_QP_AV, IBV_QP_PATH_MTU, IBV_QP_DEST_QPN, IBV_QP_RQ_PSN, IBV_QP_MAX_DEST_RD_ATOMIC, IBV_QP_MIN_RNR_TIMER, IBV_QP_ALT_PATH, IBV_QP_SQ_PSN, IBV_QP_TIMEOUT, IBV_QP_RETRY_CNT, IBV_QP_RNR_RETRY, IBV_QP_MAX_QP_RD_ATOMIC, IBV_QP_PATH_MIG_STATE
|SQD|	IBV_QP_STATE, IBV_QP_PKEY_INDEX, IBV_QP_PORT, IBV_QP_ACCESS_FLAGS, IBV_QP_AV, IBV_QP_PATH_MTU, IBV_QP_DEST_QPN, IBV_QP_RQ_PSN, IBV_QP_MAX_DEST_RD_ATOMIC, IBV_QP_MIN_RNR_TIMER, IBV_QP_ALT_PATH, IBV_QP_SQ_PSN, IBV_QP_TIMEOUT, IBV_QP_RETRY_CNT, IBV_QP_RNR_RETRY, IBV_QP_MAX_QP_RD_ATOMIC, IBV_QP_PATH_MIG_STATE
|ERR|	IBV_QP_STATE

 **示例：**

```cpp
struct ibv_qp *qp;
struct ibv_qp_attr attr;
struct ibv_qp_init_attr init_attr;

if (ibv_query_qp(qp, &attr,IBV_QP_STATE, &init_attr)) {
	fprintf(stderr, "Failed to query QP state\n");
	return -1;
}
```

**常见问题：**
Q：当我需要QP属性时，每次调用ibv_query_qp()都需要花费一些时间，我可以缓存一些属性吗?
A：实际是可以的。 除非使用ibv_modify_qp()对其进行了更改，否则大多数属性都是常量。 QP属性结构中的以下字段可能会更改：qp_state，path_mig_state，sq_draining，ah_attr，pkey_index，port_num，timeout。

Q：我可以确切指定ibv_query_qp()填充哪些属性吗？
A：不可以。参数attr_mask只是一个提示，RDMA设备的底层驱动程序可能会（并且在大多数情况下）会填充比attr_mask中请求的属性更多的属性。

Q：所有QP属性都有效吗？
A：否。有效的QP属性取决于QP状态。

Q：如何获得QP状态？
A：至少应该使用IBV_QP_STATE设置标志来调用ibv_query_qp()。


### 6.2.5 ibv_modify_qp
**函数原型：**
```cpp
int ibv_modify_qp(struct ibv_qp *qp, struct ibv_qp_attr *attr, enum ibv_qp_attr_mask attr_mask)
```
**输入参数：**
* qp——来自ibv_create_qp或ibv_create_qp_ex。
* attr——QP属性。详细信息见ibv_query_qp。
* attr_mask——位掩码，用于定义attr中的哪些属性已为此调用设置。详细信息见ibv_query_qp。

**输出参数：** attr——如果QP属性被修改，则attr中保存修改后的实际属性。

**返回值：** 成功返回0；失败返回errno指示失败的原因。

* EINVAL——attr和或attr_mask中有非法参数。
* ENOMEM——内存不足。

**说明：**

ibv_modify_qp根据attr_mask更改QP属性，这些属性之一有可能是QP状态。 它的名称有点用词不当，因为您不能使用此命令随意修改qp属性。 在每个状态转变期间都有一组非常严格的属性可以修改，并且状态转变必须以正确的顺序进行。 以下小节将详细介绍每个状态转变。

如果任何修改属性或修改掩码无效，则不会修改任何属性（包括QP状态）。

并非所有设备都支持调整QP大小。 要检查设备是否支持它，请检查设备功能标志中的IBV_DEVICE_RESIZE_MAX_WR位是否已设置。

并非所有设备都支持备用路径。 要检查设备是否支持它，请检查设备功能标志中的IBV_DEVICE_AUTO_PATH_MIG位是否已设置。


更改的属性说明了QP的发送和接收属性。 在UC和RC QP中，这意味着将QP与远程QP连接。

在Infiniband中，应该执行到子网管理员（SA）的路径查询，以便确定应该为QP配置哪些属性或作为最佳解决方案，请使用Communication Manager（CM）或通用RDMA CM代理（CMA） 连接QP。 但是，有些应用程序倾向于自己连接QP，并通过套接字交换数据来决定使用哪个QP属性。

在RoCE中，对于RC QPs和UC QPs，应该在QP属性中配置GRH，对于UD QP，则应在AH中配置GRH。

在iWARP中，只能使用通用RDMA CM代理（CMA）连接QP。


**示例：** 见下一节QP状态转移。

**常见问题：**
Q：为什么需要调用ibv_modify_qp()?
A：您需要调用ibv_modify_qp()才能将QP修改为可以接收和发送数据的状态。

Q：我可以将QP移到RTR状态并保持在该状态吗？
A：是的你可以。 如果QP只是接收者。

Q：我如何知道要用哪些属性配置QP ?
A：在Infiniband中，应该执行到SA的路径查询，或者使用CM或CMA；在iWARP中，应该使用CMA。

Q：ibv_modify_qp()失败了，该QP的属性现在是什么？
A：与调用ibv_modify_qp()之前完全相同，未对QP进行任何更改。 尽管故障可能是由于错误导致RDMA设备将QP状态转移至SQE或ERROR状态引起的。

Q：在基于ROCE的有连接QP中，ibv_modify_qp()进行INIT-> RTR转换失败，这是什么原因？
A：使用RoCE时，必须配置GRH。在有连接的QP中：作为QP属性的一部分。在UD QP中：作为地址句柄属性的一部分。

### 6.2.6 ibv_open_qp
**函数原型：** 
```cpp
struct ibv_qp *ibv_open_qp(struct ibv_context *context,struct ibv_qp_open_attr *qp_open_attr);

```
**输入参数：** 

* context——设备上下文，来自ibv_open_device
* qp_open_attr——要打开的qp打开属性。详细信息见下文。

**输出参数：** 无。

**返回值：**   成功返回指向打开的QP的指针，如果请求失败，则返回NULL。 检查返回的QP中的QP编号（qp_num）。

**描述：** 

打开与扩展保护域xrcd关联的已有队列对（QP）。

如果要求ibv_open_qp()打开指定xrcd中不存在的qp_num和qp_type的QP，它将失败。
  
ibv_destroy_qp()关闭打开的QP并销毁基础QP（如果没有其他引用）。
  
struct ibv_qp_open_attr定义如下：
```cpp
struct ibv_qp_open_attr {
	uint32_t			comp_mask;		//标识有效字段，使用enum ibv_qp_open_attr_mask
	uint32_t			qp_num;			//QP编号
	struct ibv_xrcd	*xrcd;			//XRC域
	void				*qp_context;	//用户定义不透明值
	enum ibv_qp_type	qp_type;		//QP传输服务类型
};  
```
enum ibv_qp_open_attr_mask定义如下
```cpp
enum ibv_qp_open_attr_mask {
	IBV_QP_OPEN_ATTR_NUM		= 1 << 0,
	IBV_QP_OPEN_ATTR_XRCD		= 1 << 1,
	IBV_QP_OPEN_ATTR_CONTEXT	= 1 << 2,
	IBV_QP_OPEN_ATTR_TYPE		= 1 << 3,
	IBV_QP_OPEN_ATTR_RESERVED	= 1 << 4
}; 
```

### 6.2.7 ibv_destroy_qp
**函数原型：**
```cpp
int ibv_destroy_qp(struct ibv_qp *qp)
```
**输入参数：** qp，来自ibv_create_qp的struct ibv_qp。


**输出参数：** 无。

**返回值：** 成功返回0；失败返回errno指示失败的原因。EBUSY——QP仍然添加到一个或多个多播组。

**说明：**

ibv_destroy_qp释放一个队列对。

当QP被销毁时，RDMA设备将不再处理发送或接收队列中的任何未完成的工作请求，也不会为它们生成工作完成。 用户负责清除那些工作请求的所有关联资源（即内存缓冲区）。

将QP修改为“Error”状态并在销毁队列对之前轮询所有相关的工作完成是一种良好的编程习惯。由于销毁队列对并不能保证销毁后，其工作完成将从CQ中删除。即使工作完成已经在CQ中，也可能无法检索它们。在调整CQ大小，将新的Qp到CQ或重新打开队列对之前，这样做有助于避免CQ溢出。如果队列对与SRQ相关联，建议在将队列对修改为“Error”状态之后和销毁队列对之前，等待关联事件IBV_EVENT_QP_LAST_WQE_REACHED。否则，可能导致工作请求泄漏。销毁队列对时，建议执行以下步骤：

* 将队列对移到Error状态
* 等待关联事件IBV_EVENT_QP_LAST_WQE_REACHED
* 要么，通过调用ibv_poll_cq来排空CQ，然后等待CQ为空或轮询CQ操作数已超过CQ容量大小。
* 要么，发布另一个将在同一CQ上完成的工作请求，并等待该工作请求作为工作完成返回
* 通过调用ibv_destroy_qp()销毁QP

如果不可靠的数据报队列对仍然被添加到多播组，则销毁该QP将失败。

如果使用ibv_get_async_event()在该QP上读取了任何关联的异步事件，但仍未使用ibv_ack_async_event()进行确认，则对ibv_destroy_qp()的调用将永远不会结束，直到该事件被确认为止。

QP可以在任何状态下被销毁。

**示例：**
```cpp
//创建一个发送和接收队列具有相同CQ的QP并将其销毁：
struct ibv_pd *pd;
struct ibv_cq *cq;
struct ibv_qp *qp;
struct ibv_qp_init_attr qp_init_attr;

memset(&qp_init_attr, 0, sizeof(qp_init_attr));

qp_init_attr.send_cq	 cq;
qp_init_attr.recv_cq	 cq;
qp_init_attr.qp_type	 IBV_QPT_RC;
qp_init_attr.cap.max_send_wr	 2;
qp_init_attr.cap.max_recv_wr	 2;
qp_init_attr.cap.max_send_sge	 1;
qp_init_attr.cap.max_recv_sge	 1;

qp	 ibv_create_qp(pd, &qp_init_attr);
if (!qp) {
	fprintf(stderr, "Error, ibv_create_qp() failed\n");
	return -1;
}

if (ibv_destroy_qp(qp)) {
	fprintf(stderr, "Error, ibv_destroy_qp() failed\n");
	return -1;
}


//创建发送和接收队列具有不同CQ的QP
struct ibv_pd *pd;
struct ibv_cq *send_cq;
struct ibv_cq *recv_cq;
struct ibv_qp *qp;
struct ibv_qp_init_attr qp_init_attr;

memset(&qp_init_attr, 0, sizeof(qp_init_attr));

qp_init_attr.send_cq	 send_cq;
qp_init_attr.recv_cq	 recv_cq;
qp_init_attr.qp_type	 IBV_QPT_RC;
qp_init_attr.cap.max_send_wr	 2;
qp_init_attr.cap.max_recv_wr	 2;
qp_init_attr.cap.max_send_sge	 1;
qp_init_attr.cap.max_recv_sge	 1;

qp	 ibv_create_qp(pd, &qp_init_attr);
if (!qp) {
	fprintf(stderr, "Error, ibv_create_qp() failed\n");
	return -1;
}

//创建一个与SRQ关联的QP
struct ibv_pd *pd;
struct ibv_cq *cq;
struct ibv_srq *srq;
struct ibv_qp *qp;
struct ibv_qp_init_attr qp_init_attr;

memset(&qp_init_attr, 0, sizeof(qp_init_attr));

qp_init_attr.send_cq	 send_cq;
qp_init_attr.recv_cq	 recv_cq;
qp_init_attr.srq	 srq;
qp_init_attr.qp_type	 IBV_QPT_RC;
qp_init_attr.cap.max_send_wr	 2;
qp_init_attr.cap.max_send_sge	 1;

qp	 ibv_create_qp(pd, &qp_init_attr);
if (!qp) {
	fprintf(stderr, "Error, ibv_create_qp() failed\n");
	return -1;
}

//创建一个支持内联消息的QP
struct ibv_pd *pd;
struct ibv_cq *cq;
struct ibv_qp *qp;
struct ibv_qp_init_attr qp_init_attr;

memset(&qp_init_attr, 0, sizeof(qp_init_attr));

qp_init_attr.send_cq	 cq;
qp_init_attr.recv_cq	 cq;
qp_init_attr.qp_type	 IBV_QPT_RC;
qp_init_attr.cap.max_send_wr	 2;
qp_init_attr.cap.max_recv_wr	 2;
qp_init_attr.cap.max_send_sge	 1;
qp_init_attr.cap.max_recv_sge	 1;
qp_init_attr.cap.max_inline_data	 512;

qp	 ibv_create_qp(pd, &qp_init_attr);
if (!qp) {
	fprintf(stderr, "Error, ibv_create_qp() failed\n");
	return -1;
}
```

**常见问题：**
Q：ibv_destroy_qp()失败，此QP附加到的多播组会发生什么？
A：没事，您可以继续使用，而不会产生任何副作用。

Q：ibv_destroy_qp()失败，我可以知道它连接到哪个多播组并导致此失败吗？
A：不，目前RDMA堆栈没有这个功能。

Q：我可以销毁拥有未完成工作请求的QP吗？
A：是的你可以。 销毁QP时，无论是否包含工作请求都无关紧要。

Q：我可以在任何状态下摧毁QP吗?
A：是的，QP可以在任何状态下被销毁。

Q：我调用了ibv_destroy_qp()，但是它没有结束。发生了什么事?
A：该QP上至少有一个关联的异步事件被读取，但没有确认。

## 6.3 注册、分配、建议、注销MR
### 6.3.1 ibv_reg_mr
**函数原型：**
```cpp
struct ibv_mr *ibv_reg_mr(struct ibv_pd *pd, void *addr, size_t length, enum ibv_access_flags access)
```
**输入参数：**
* pd——来自ibv_alloc_pd的struct ibv_pd。
* addr——内存基址。
* length——内存区域大小，以字节为单位，至少为1，并小于dev_cap.max_mr_size。
* access——访问标志。详细信息见下文。

**输出参数：** 无。

**返回值：** 成功返回指向创建的内存区的指针；失败返回NULL，并设置error指示错误原因：
* EINVAL——非法的访问权限。
* ENOMEM——没有足够的资源(在操作系统或RDMA设备中)来完成此操作。

当用ibv_post_\*动词发布缓冲区，本地密钥（L_key）字段lkey被用作struct ibv_sge的lkey字段。远程密钥（R_key）rkey被远程进程用于执行RDMA操作和原子操作。远程进程将这个rkey作为传递给ibv_post_send函数的struct ibv_send_wr的rkey字段。


**说明：**

ibv_reg_mr注册一个内存区域（MR），将其与保护域（PD）相关联，并为其分配本地和远程密钥（lkey，rkey）。 所有使用内存的VPI命令都要求通过此命令注册内存。 可以根据用户要求将相同的物理内存映射到不同的MR，甚至允许将不同的权限或PD分配给相同的内存。

通过这样做，允许RDMA设备读取数据并将其写入该内存。 执行此注册需要一些时间，因此，当需要快速响应时，建议不要在数据路径中执行内存注册。

每一次成功的注册都会产生一个MR，它有唯一的(在一个特定的RDMA设备内)lkey和rkey值。

MR的起始地址为addr，大小为length。 可以注册的块的最大大小限制为device_attr.max_mr_size。 调用进程的虚拟空间中的每个内存地址都可以可以注册，包括但不限于：
* 本地内存（变量或数组）
* 全局内存（变量或数组）
* 动态分配内存（使用malloc()或mmap()）
* 共享内存
* 文本段地址

注册的内存缓冲区不必是页面对齐的。

没有任何办法知道可以为特定设备注册的总内存量。

enum ibv_access_flags 定义如下：

```cpp
enum ibv_access_flags {
	IBV_ACCESS_LOCAL_WRITE		= 1,		//允许本地写
	IBV_ACCESS_REMOTE_WRITE		= (1<<1),	//允许在这个QP上传入RDMA 写
	IBV_ACCESS_REMOTE_READ		= (1<<2),	//允许在这个QP上传入RDMA 读
	IBV_ACCESS_REMOTE_ATOMIC	= (1<<3),	//允许在这个QP上传入RDMA 原子操作
	IBV_ACCESS_MW_BIND			= (1<<4),	//允许内存窗口绑定
	IBV_ACCESS_ZERO_BASED		= (1<<5),	//使用从MR开头的字节偏移量来访问此MR，而不是指针地址
	IBV_ACCESS_ON_DEMAND		= (1<<6),	//创建一个按需分页MR
};**
```
enum ibv_access_flags 的完整定义如下：
|枚举值|说明|
|:--|:--|
|IBV_ACCESS_LOCAL_WRITE|启用本地写访问：内存区域可用于接收请求（RR）或在IBV_WR_ATOMIC_CMP_AND_SWP或IBV_WR_ATOMIC_FETCH_AND_ADD在本地写入远程内容的值
|IBV_ACCESS_REMOTE_WRITE|启用远程写访问:可以使用IBV_WR_RDMA_WRITE 或IBV_WR_RDMA_WRITE_WITH_IMM从远程上下文访问内存区域|
|IBV_ACCESS_REMOTE_READ|启用远程读访问：可以使用IBV_WR_RDMA_READ从远程上下文访问内存区域|
|IBV_ACCESS_REMOTE_ATOMIC|启用远程原子操作访问(如果支持):可以使用IBV_WR_ATOMIC_CMP_AND_SWP或IBV_WR_ATOMIC_FETCH_AND_ADD从远程上下文访问内存区域|
|IBV_ACCESS_MW_BIND|允许内存窗口绑定|
|IBV_ACCESS_ZERO_BASED|使用从MR开头的字节偏移量来访问此MR，而不是指针地址|
|IBV_ACCESS_ON_DEMAND|创建一个按需分页MR

如果设置了 IBV_ACCESS_REMOTE_WRITE或 IBV_ACCESS_REMOTE_ATOMIC，则IBV_ACCESS_LOCAL_WRITE必须设置。因为只有在允许本地写的情况下才应该允许远程写。

本地读访问始终是允许的。即可以使用IBV_WR_SEND、IBV_WR_SEND_WITH_IMM、IBV_WR_RDMA_WRITE、IBV_WR_RDMA_WRITE_WITH_IMM从本地读取内存区域。

为了创建一个隐式的ODP MR，应设置IBV_ACCESS_ON_DEMAND， addr应该为0， length应该为SIZE_MAX。

内存注册区域的要求权限可以是该内存块的操作系统权限的全部或子集。 例如：只读内存无法注册为具有写入权限（本地或远程）。

特定进程可以注册一个或多个内存区域。

本地读访问是隐含的和自动的。

任何违反给定内存操作的访问权限的VPI操作都将失败。

请注意，队列对（QP）属性还必须具有正确的权限，否则操作将失败。

struct ibv_mr定义如下：
```cpp
struct ibv_mr	{
	struct ibv_context		*context;
	struct ibv_pd			*pd;
	void					*addr;
	size_t					length;
	uint32_t				handle;
	uint32_t				lkey;	//将被用于在本地访问中引用此MR
	uint32_t				rkey;	//将被用于在远程访问中引用此MR。这两个值可能相等，但这并不总是保证的。
};
```

**示例：** 见ibv_dereg_mr。

**常见问题：**

Q：为什么MR总是有好处？
A：MR注册是RDMA设备获取内存缓冲区并准备将其用于本地 和/或 远程访问的过程。

Q：我可以多次注册同一个内存块吗?
A：是。 可以多次注册同一个内存块或其一部分。 这些内存注册区甚至可以使用不同的访问标志来执行。

Q：可以注册的内存的总大小是多少?
A：没有任何方法可以知道可以注册的内存总大小。 从理论上讲，此值没有任何限制。 但是，如果希望注册大量内存（数百GB），则低级驱动程序的默认值可能还不够；因此，如果您希望将这些值添加到内存中，则可能无法使用。请参阅“设备特性”一节，了解如何更改默认参数值以解决此问题。

Q：我可以使用比操作系统允许的更多权限的RDMA设备访问内存吗?
A：否。在内存注册期间，驱动程序将检查内存块的权限，并检查是否允许该内存块使用所请求的权限。

Q：没有注册就可以在RDMA中使用内存块吗？
A：基本上不行，但是，有些RDMA设备可以读取内存而无需进行内存注册（内联数据发送）。

Q：ibv_reg_mr()失败，原因是什么?
A：ibv_reg_mr()可能会失败，原因如下:
* 错误的属性:错误的权限或错误的内存缓冲区
* 没有足够的资源来注册这个内存缓冲区

如果这是您注册的第一个缓冲区，则可能是由于第一个原因。 如果在已经注册了许多缓冲区之后内存注册失败，则可能是该失败的原因是没有足够的资源来注册此内存缓冲区：大多数情况下，RDMA设备中的资源用于虚拟地址的转换=> 实际地址。 在这种情况下，您可能需要检查如何增加此RDMA设备可以注册的内存量。

Q：我可以在进程中注册多个MR吗？
A：可以。

**设备特性：Mellanox技术**
Q：ibv_reg_mr()失败，原因是什么?
A：如果您使用的是ConnectX HCAs家族中的一个，这是一个配置问题;您应该增加可注册内存的总大小。这可以通过设置模块mlx4_core的参数log_num_mtt的值来实现。将以下行添加到文件`/etc/modprobe.conf或/etc/modprobe.d/mlx4_core.conf`（取决于您使用的Linux发行版）应该可以解决此问题：`options mlx4_core log_num_mtt=24`

### 6.3.2 ibv_rereg_mr
**函数原型：**
```cpp
int ibv_rereg_mr(struct ibv_mr *mr, int  flags,struct ibv_pd * pd,
			void  *addr,size_t  length, int  access);
```

**输入参数：**

* mr——来自ibv_reg_mr的struct ibv_mr。
* flags——enum ibv_rereg_mr_flags按位或得到的位掩码，表示要修改的属性。详细信息见下文。
* pd——要重新注册的保护域，来自ibv_alloc_pd。
* addr——mr的起始地址。
* length——mr的长度。
* access——访问和其它标志。0或enum ibv_access_flags按位或得到。详细信息见ibv_reg_mr。

**输出参数：** 无。

**返回值：** 成功返回0；出错返回错误代码enum ibv_rereg_mr_err_code来指示失败的原因。
```cpp
enum ibv_rereg_mr_err_code {
	IBV_REREG_MR_ERR_INPUT					= -1,	//原始MR有效，libibverbs检测到输入错误
	IBV_REREG_MR_ERR_DONT_FORK_NEW			= -2,	//原始MR有效，但由于未在新的地址范围上fork而失败。
	IBV_REREG_MR_ERR_DO_FORK_OLD			= -3,	//新MR有效，由于旧地址范围上的fork而失败。
	IBV_REREG_MR_ERR_CMD					= -4,	//不应使用MR，命令错误。
	IBV_REREG_MR_ERR_CMD_AND_DO_FORK_NEW	= -5,	//不应使用MR，命令错误，新地址范围上的fork状态无效。
};
```

**说明：**

ibv_rereg_mr()修改一个已经存在的内存区域（MR）mr的属性。从概念上来说，此函数先注销内存区域mr，再注册内存区域mr。在可能的情况在，资源被重用，而不是释放和重新分配。

即使失败了，用户仍然需要在此MR上调用ibv_dereg_mr 进行注销。

 enum ibv_rereg_mr_flags定义如下：
```cpp
enum ibv_rereg_mr_flags {
	IBV_REREG_MR_CHANGE_TRANSLATION	= (1 << 0),	//更改位置和长度，参数addr和length有效。
	IBV_REREG_MR_CHANGE_PD			= (1 << 1),	//更改保护域，参数pd有效。
	IBV_REREG_MR_CHANGE_ACCESS		= (1 << 2),	//更改访问标志，参数access有效。
	IBV_REREG_MR_KEEP_VALID			= (1 << 3),
	IBV_REREG_MR_FLAGS_SUPPORTED	= ((IBV_REREG_MR_KEEP_VALID << 1) - 1)
};
```
### 6.3.3 ibv_alloc_null_mr(N)

### 6.3.4 ibv_advise_mr(N)

### 6.3.5 ibv_dereg_mr
**函数原型：**
```cpp
int ibv_dereg_mr(struct ibv_mr *mr)
```
**输入参数：** mr——来自ibv_reg_mr的struct ibv_mr。

**输出参数：** 无。

**返回值：** 成功返回0；出错返回错误码，并设置errno来指示失败的原因。
* EINVAL——MR上下文非法。
* EBUSY——MW仍然与该MR关联。

**说明：**

ibv_dereg_mr()注销一个内存区域，并在与之关联的RDMA设备中释放该内存区域的资源。

仅在注销内存区域后，用户才应释放与其关联的内存缓冲区。

如果仍然有任何内存窗口绑定到MR，则注销将失败。

由用户决定是否停止使用与此MR关联的密钥（lkey和rkey）。 如果这些密钥将用于本地工作请求或用于远程操作请求，则将导致工作完成错误。

**示例：**

```cpp
struct ibv_pd *pd;
struct ibv_mr *mr;

mr	 ibv_reg_mr(pd, buf, size, IBV_ACCESS_LOCAL_WRITE);
if (!mr) {
	fprintf(stderr, "Error, ibv_reg_mr() failed\n");
	return -1;
}

if (ibv_dereg_mr(mr)) {
	fprintf(stderr, "Error, ibv_dereg_mr() failed\n");
	return -1;
}
```

**常见问题：**
Q：如果在注销之前释放与MR相关的内存缓冲区会怎样？
A：这样做可能会导致段错误。

Q：如果我在注销MR之后使用与MR关联的密钥（密钥），将会发生什么？
A：这样做将导致带有错误的工作完成，因为这些密钥无效。 在注销此MR之前，应确保没有使用这些密钥的任何本地工作请求或远程操作请求。

Q：ibv_dereg_mr()失败，我可以知道分配了哪个MW并导致此失败吗？
A：不，目前RDMA堆栈没有这个功能。

## 6.4 创建和销毁AH

### 6.4.1 ibv_create_ah
**函数原型：**
```cpp
struct ibv_ah *ibv_create_ah(struct ibv_pd *pd, struct ibv_ah_attr *attr)
```
**输入参数：**
* pd——来自ibv_alloc_pd的struct ibv_pd。
* attr——地址句柄要求的属性。详细信息见下文。

**输出参数：** 无。

**返回值：** 成功返回指向创建的地址句柄(AH)的指针，失败返回NULL，并设置errno表示失败原因:

* EINVAL——无效的pd，或attr中提供的值无效。
* ENOMEM——没有足够的资源来完成这个操作。

**说明：**

ibv_create_ah()创建与保护域关联的地址句柄(AH)。 AH包含到达远程目的地的所有必要数据。 在连接的传输模式（RC，UC）中，AH与队列对（QP）相关联。 在数据报传输模式（UD）中，AH与工作请求（WR）相关联。

稍后，当发送请求(SR)被发布到不可靠的数据报（ UD ）QP时，将使用这个AH。

如果设置了端口标志IBV_QPF_GRH_REQUIRED，则 ibv_create_ah必须使用如下定义：
```
struct ibv_ah_attr { .is_global	 1; .grh	 {...}; }
```

struct ibv_ah定义如下：
```cpp
struct ibv_ah {
	struct ibv_context		*context;	//设备上下文，来自ibv_open_device
	struct ibv_pd			*pd;		//保护域，来自ibv_alloc_pd
	uint32_t				handle;		//句柄
};
```

struct ibv_ah_attr定义如下：
```cpp
struct ibv_ah_attr {
	struct ibv_global_route	grh;			//全局路由头GRH属性，定义见下文
	uint16_t					dlid;			//目的地lid
	uint8_t						sl;				//服务级别
	uint8_t						src_path_bits;	//源路径位
	uint8_t						static_rate;	//静态速度
	uint8_t						is_global;		//GRH属性是有效的
	uint8_t						port_num;		//将发送数据包的本地物理端口
};
```
struct ibv_ah_attr完整定义如下：
|字段|说明|
|:--|:--|
|grh|全局路由头（GRH）的属性，如下表所述。 将数据包发送到另一个子网或多播组时，这很有用。|
|dlid|如果目标在同一子网中，则是该子网将数据包传送到的端口的LID。 如果目标在另一个子网中，则是路由器的LID
|sl|4位。要使用的服务级别
|src_path_bits|使用的源路径位。 当在端口中使用LMC时，即每个端口覆盖一个LID范围时，这很有用。 数据包将使用端口的基本LID发送，并与源路径位的值进行按位“或”运算。 值0表示使用了端口的基本LID|
|static_rate|一个值，该值限制发送到子网的数据包的速率。 如果数据包来源的速率高于目的地的速率，这将很有用|
|is_global|如果该值包含除0之外的任何值，则GRH信息存在于此AH中，因此字段GRH有效|
|port_num|将发送数据包的本地物理端口|


struct ibv_global_route的定义如下：
```cpp
struct ibv_global_route {
	union		ibv_gid dgid;	//目的地GID（参见ibv_query_gid）或MGID
	uint32_t	flow_label;		//流标签
	uint8_t		sgid_index;		//源GID索引（参见ibv_query_gid）
	uint8_t		hop_limit;		//跳数限制
	uint8_t		traffic_class;	//流量类别
};
```
struct ibv_global_route完整定义如下：
|字段|说明|
|:--|:--|
|dgid|用于标识包的目标端口的GID|
|flow_label|20位。 如果将此值设置为非零值，则会提示具有多个出站路径的交换机和路由器，这些数据包序列必须按顺序传送，而这些包序列必须位于同一路径上，这样就不会对其进行重新排序 。|
|sgid_index|端口的GID表中的索引，将用于标识数据包的始发者
|hop_limit|数据包在被丢弃之前被允许经过的跳数（即路由器的数量）。 这样可以确保在发生路由循环时，数据包不会在路由器之间无限期循环。 每个路由器在数据包处将该值减一，并且当该值达到0时，将丢弃此数据包。 将该值设置为0或1将确保数据包不会离开本地子网。|
|traffic_class|数据包的始发者使用此值指定路由器处理它们所需的传递优先级|

**示例：** 见ibv_destroy_ah()。

**常见问题：**

Q：AH有什么好处？
A：当对UD QP调用ibv_post_send()时使用AH。 该对象说明了从请求方到响应方的路径。

Q：我可以将一个AH与不同的QP一起使用吗？
A：是。 只要所有这些QP和AH与相同的PD关联，就可以完成此操作。

Q：调用ibv_create_ah()时如何获得AH所需的信息？
A：获取这些信息有几种方法:

* 向子网管理员(SA)执行路径查询
* 与远程节点的带外连接，例如：使用套接字
* 使用众所周知的值：这可以在静态子网（所有地址都已预定义）中完成，也可以使用多播组完成


### 6.4.2 ibv_init_ah_from_wc
**函数原型：**
```cpp
int ibv_init_ah_from_wc(struct ibv_context *context, uint8_t port_num,
							struct ibv_wc *wc, struct ibv_grh *grh,struct ibv_ah_attr *ah_attr);
```
**输入参数：**

* context——从ibv_open_device()返回的RDMA设备上下文，接收到的消息到达该上下文。
* port_num——接收到的消息到达的端口号。
* wc——使用ibv_poll_cq()读取的工作完成
* grh——到UD QP的传入消息的GRH缓冲区。 除非工作完成表明GRH有效，否则将忽略此值。详细信息见下文。

**输出参数：** ah_attr——将被填充的地址句柄属性的结构体。

**返回值：** 成功返回0；出错返回-1。如果在端口的port_num GID表中找不到GRH中的DGID，则此动词可能失败。

**说明：**

ibv_init_ah_from_wc使用端口号port_num以及来自工作完成wc和全局路由头（GRH）结构体grh中的属性为RDMA设备上下文context初始化地址句柄（AH）属性结构体ah_attr。

从ibv_init_ah_from_wc()返回的结构体ah_attr可用于ibv_create_ah()创建新的AH。

当希望将响应发送回不可靠数据报（UD）QP接收到的消息的发送者时，这很有用。 wc是使用ibv_poll_cq()从CQ轮询的该消息的工作完成。 此工作完成必须成功并且属于单播消息。

grh是(可能)包含接收消息的grh的缓冲区(发送到接收队列以指定消息保存位置的接收请求缓冲区的前40个字节)。

struct ibv_grh定义如下：
```cpp
  struct ibv_grh {
	__be32			version_tclass_flow;
	__be16			paylen;
	uint8_t			next_hdr;
	uint8_t			hop_limit;
	union ibv_gid	sgid;
	union ibv_gid	dgid;
  };
```


**示例：**

```cpp
struct ibv_context *context;
struct ibv_pd *pd;
struct ibv_ah *ah;
struct ibv_wc wc;
struct ibv_ah_attr ah_attr;
int ret;

ret	 ibv_init_ah_from_wc(context, port,
                          &wc, grh_buf,
                          &ah_attr);
if (ret) {
	fprintf(stderr, "Error, ibv_init_ah_from_wc() failed\n");
	return -1;
}

ah	 ibv_create_ah(pd, &ah_attr);
if (!ah) {
	fprintf(stderr, "Error, ibv_create_ah() failed\n");
	return -1;
}

if (ibv_destroy_ah(ah)) {
	fprintf(stderr, "Error, ibv_destroy_ah() failed\n");
	return -1;
}
```

**常见问题：**

Q：我可以使用任何工作完成来填充AH属性结构体吗？
A：否。调用ibv_init_ah_from_wc()时可以使用的工作完成有几点限制：

* 工作完成必须是单播消息，而不是多播消息。
* 如果在wc_flags中设置了IBV_WC_GRH，即传入消息具有GRH，则必须提供grh。
* 该工作完成的状态必须为IBV_WC_SUCCESS，否则此工作完成的大多数属性均无效。


### 6.4.3 ibv_create_ah_from_wc

**函数原型：**
```cpp
struct ibv_ah *ibv_create_ah_from_wc(struct ibv_pd *pd, struct ibv_wc *wc,
										struct ibv_grh *grh, uint8_t port_num);
```
**输入参数：**

* pd——从ibv_alloc_pd()返回的保护域，该域与接收到消息的RDMA设备上下文相关联。
* wc——使用ibv_poll_cq()读取的工作完成。
* grh——到UD QP的传入消息的GRH缓冲区。 除非工作完成表明GRH有效，否则将忽略此值
* port_num——收到的消息到达的端口号。

**输出参数：** 无。

**返回值：** 成功返回指向新分配的地址句柄的指针；出错返回NULL并设置errno来指示失败的原因。

* EINVAL——pd无效，或提供的属性无效。
* ENOMEM——没有足够的资源来完成这个操作。

**说明：**

ibv_create_ah_from_wc()使用port_num、工作完成和全局路由头（GRH）缓冲区来创建地址句柄（AH）。

当希望将响应发送回不可靠数据报（UD）QP接收到的消息的发送者时，这很有用。 wc是使用ibv_poll_cq()从CQ轮询的该消息的工作完成。 此工作完成必须成功并且属于单播消息。

grh是(可能)包含接收消息的grh的缓冲区(发送到接收队列以指定消息保存位置的接收请求缓冲区的前40个字节)。

>注：实质上这个函数是ibv_init_ah_attr_from_wc和ibv_create_ah的综合。

**示例：**

```cpp
struct ibv_pd *pd;
struct ibv_ah *ah;
struct ibv_wc wc;
int ret;

ah	 ibv_create_ah_from_wc(pd, &wc, grh_buf,
                            port);
if (!ah) {
	fprintf(stderr, "Error, ibv_create_ah_from_wc() failed\n");
	return -1;
}

if (ibv_destroy_ah(ah)) {
	fprintf(stderr, "Error, ibv_destroy_ah() failed\n");
	return -1;
}
```

**常见问题：**

Q：我可以使用任何工作完成来填充AH属性结构体吗？
A：否。调用ibv_init_ah_from_wc()时可以使用的工作完成有几点限制：

* 工作完成必须是单播消息，而不是多播消息。
* 如果在wc_flags中设置了IBV_WC_GRH，即传入消息具有GRH，则必须提供grh。
* 该工作完成的状态必须为IBV_WC_SUCCESS，否则此工作完成的大多数属性均无效。

### 6.4.4 ibv_destroy_ah
**函数原型：**
```cpp
int ibv_destroy_ah(struct ibv_ah *ah)
```
**输入参数：** ah，来自ibv_create_ah的struct ibv_ah。

**输出参数：** 无。

**返回值：** 成功返回0；出错返回errno来指示失败的原因。EINVAL——AH上下文非法。

**说明：**

ibv_destroy_ah释放地址句柄（AH）。一旦AH被销毁，它不能再用在UD QPs。

用户必须在销毁AH之前等待与该AH一起发布的所有未完成的发送请求完成或清除。 否则，可能会导致工作完成错误或段错误。

**示例：**

```cpp
struct ibv_pd *pd;
struct ibv_ah *ah;
struct ibv_ah_attr ah_attr;

memset(&ah_attr, 0, sizeof(ah_attr));

ah_attr.is_global	 0;
ah_attr.dlid	 dlid;
ah_attr.sl	 sl;
ah_attr.src_path_bits	 0;
ah_attr.port_num	 port;

ah	 ibv_create_ah(pd, &ah_attr);
if (!ah) {
	fprintf(stderr, "Error, ibv_create_ah() failed\n");
	return -1;
}

if (ibv_destroy_ah(ah)) {
	fprintf(stderr, "Error, ibv_destroy_ah() failed\n");
	
```

**常见问题：**
Q：如果仍然有未完成的发送请求使用AH时销毁AH，会发生什么情况？
A：这样做可能导致错误的工作完成或段错误。

Q：我想销毁一个AH，但仍有一些仍在使用的未完成的发送请求，我该怎么办？
A：有两个选择：

* 等待，直到未完成的发送请求完成
* 刷新那些发布了“发送请求''的发送队列（例如，可以将那些发布了“发送请求''的QP的状态更改为IBV_QPS_ERR状态）

## 6.5 分配、绑定、释放MW（差mojo）
### 6.5.1 ibv_alloc_mw
**函数原型：**

```cpp
struct ibv_mw *ibv_alloc_mw(struct ibv_pd *pd,enum ibv_mw_type type)
```
**输入参数：** 

* pd——保护域，来自ibv_alloc_pd。
* type——内存窗口类型。enum ibv_mw_type的详细信息见下文。

**输出参数：** 无。

**返回值：** 成功返回指向struct ibv_mw的指针，失败返回NULL。

**说明：**

ibv_alloc_mw分配一个类型为type的内存窗口，并与与保护域pd关联。

MW创建时未绑定，如果要使用MW，则必须通过ibv_bind_mw（type为1）或特别的WR（type为2）进行绑定。一旦绑定，内存窗口将允许RDMA（远程）访问与其绑定的MR的子集，直到通过以下方式使之无效：

* type为1，ibv_bind_mw中length为0，
* type为2，WR操作码为IBV_WR_LOCAL_INV或IBV_WR_SEND_WITH_INV，
* ibv_dealloc_mw

远程密钥（R_Key）字段被远程进程用来执行Atomic和RDMA操作。 绑定操作期间此密钥将被改变。 远程进程将此rkey放置为传递到ibv_post_send函数的struct ibv_send_wr的rkey字段。

struct ibv_mw定义如下：
```cpp
 struct ibv_mw {
	struct ibv_context	*context;	//设备上下文，来自ibv_open_device
	struct ibv_pd		*pd;		//保护域，来自ibv_alloc_pd
	uint32_t			rkey;		//远程密钥
	uint32_t			handle;		//句柄
	enum ibv_mw_type	type;		//窗口类型，详细信息见下文
 };

```
enum ibv_mw_type 定义如下：

```cpp
enum ibv_mw_type {
	IBV_MW_TYPE_1	= 1,
	IBV_MW_TYPE_2	= 2
};

```

### 6.5.2 ibv_bind_mw
**函数原型：**

```cpp
int ibv_bind_mw(struct ibv_qp *qp, struct ibv_mw *mw, struct ibv_mw_bind *mw_bind)
```

**输入参数：** 

* qp——队列对。来自ibv_create_qp。
* mw——要绑定的内存窗口，来自ibv_alloc_mw。
* mw_bind——要绑定内存串口的MR的信息。详细信息见下文。

**输出参数：** 无。

**返回值：** 成功返回0，失败返回errno以显示失败原因。如果成功，则绑定后的内存窗口的R_key将在mw_bind-> mw-> rkey字段中返回。

**说明：**

ibv_bind_mw()根据mw_bind中的详细信息向队列对qp发送一个绑定内存窗口mw的请求。

函数返回时，绑定不会完成-它只是发布到QP。如果绑定操作的后续CQE指示失败，则用户应保留旧R_key的副本，并修复mw结构。用户可以在同一QP上使用发送请求安全地发送R_key（基于QP排序规则：始终对同一QP的绑定请求之后的发送进行排序），但在读取到成功的CQE之前，不得以任何其他方式将其传输到远程。

请注意，对于类型2的MW，应该使用ibv_post_send直接发布绑定WR到QP。

struct ibv_mw_bind定义如下：
```cpp
struct ibv_mw_bind {
	uint64_t					wr_id;			/* 用户定义的WR IDUser defined WR ID */
	int							send_flags;		/* 使用enum ibv_send_flags，见ibv_post_send */
	struct ibv_mw_bind_info	bind_info;		/* 内存宽口绑定信息 */
}

```

struct ibv_mw_bind_info定义如下：
```cpp
struct ibv_mw_bind_info {
	struct ibv_mr	*mr;				//要绑定到MW的MR，详细信息见ibv_reg_mr
	uint64_t		addr;				//MW应该的起始地址
	uint64_t		length;				//MW应跨越的长度，以字节为单位
	unsigned int	mw_access_flags;	//MW的访问标志，使用enum ibv_access_flags,详细信息见ibv_reg_mr
};
```
对于绑定操作，QP传输服务类型必须为UC，RC或XRC_SEND之一。

属性send_flags描述WR的属性。它是0或以下一个或多个标志的按位或：
* IBV_SEND_FENCE——设置栅栏指示器。仅对IBV_QPT_RC的QP有效
* IBV_SEND_SIGNALED——设置完成通知指示器。 仅当使用sq_sig_all= 0创建QP时相关

绑定成功完成后，mw_access_flags定义允许的对MW的访问。它是0或以下一个或多个标志的按位或：
* IBV_ACCESS_REMOTE_WRITE——允许远程写访问，要求MR的本地写访问。
* IBV_ACCESS_REMOTE_READ——允许远程读访问。
* IBV_ACCESS_REMOTE_ATOMIC——允许远程原子操作访问（如果支持）。要求MR的本地写访问。
* IBV_ACCESS_ZERO_BASED——如果设置，则WR的“ remote_addr”字段设置的地址将从MW的起始地址增加一个偏移。


### 6.5.3 ibv_dealloc_mw
**函数原型：**

```cpp
int ibv_dealloc_mw(struct ibv_mw *mw)
```
**输入参数：** mw——内存窗口。来自ibv_alloc_mw。

**输出参数：** 无。

**返回值：** 成功返回0，失败返回errno以显示失败原因。

**说明：** ibv_dealloc_mw()取消之前绑定绑定的MR，并取消分配内存窗口。


# 7 基于队列对的操作
## 7.1 QP状态转移

队列对（QP）必须先通过递增的状态顺序进行转换，然后才能用于通信。

||Reset|Init|RTR|RTS|SQD|SQE|Error|
|:--|:--|:--|:--|:--|:--|:--|:--|
|发布到接收队列|禁止|允许|允许|允许|允许|允许|允许|
|发布到发送队列|禁止|禁止|禁止|允许|允许|允许|允许|
|接收请求处理|不处理|不处理|处理|处理|处理|处理|因错误刷新|
|发送请求处理|不处理|不处理|不处理|处理|新的WR不处理|因错误刷新|因错误刷新|
|传入的包|静默丢弃|静默丢弃|处理|处理|处理|处理|静默丢弃|
|传出的包|无|无|无|发出|发出|无|无|


>注：有关更多详细信息见《RDMA网络编程手册 第2章 编程概述 第4节》。


### 7.1.1 UD QP状态转移

下表指定了服务类型为IBV_QPT_UD的QP的完全受支持的转换及其接受的属性：

|状态转移类型|	必需属性|	可选属性|
|:--|:--|:--|
|\*->RESET	|IBV_QP_STATE	||
|\*->ERR	|IBV_QP_STATE	||
|RESET->INIT	|IBV_QP_STATE，<br />IBV_QP_PKEY_INDEX，<br /> IBV_QP_PORT，<br /> IBV_QP_QKEY	||
|INIT->INIT	||	IBV_QP_PKEY_INDEX， <br />IBV_QP_PORT，<br /> IBV_QP_QKEY|
|INIT->RTR|	IBV_QP_STATE|	IBV_QP_PKEY_INDEX，<br /> IBV_QP_QKEY|
|RTR->RTS|	IBV_QP_STATE，<br /> IBV_QP_SQ_PSN|	IBV_QP_CUR_STATE，<br /> IBV_QP_QKEY|
|RTS->RTS	|	|IBV_QP_CUR_STATE，<br /> IBV_QP_QKEY|
|RTS->SQD|	IBV_QP_STATE	|IBV_QP_EN_SQD_ASYNC_NOTIFY|
|SQD->RTS|	IBV_QP_STATE	|IBV_QP_CUR_STATE，<br /> IBV_QP_QKEY|
|SQD->SQD|		|IBV_QP_PKEY_INDEX，<br /> IBV_QP_QKEY
|SQE->RTS|	IBV_QP_STATE	|IBV_QP_CUR_STATE，<br /> IBV_QP_QKEY|


### 7.1.2 UC QP状态转移

下表指定了服务类型为IBV_QPT_UC的QP的完全受支持的转换及其接受的属性：
|状态转移类型|	必需属性|	可选属性|
|:--|:--|:--|
|\*->RESET|	IBV_QP_STATE	||
|\*->ERR|	IBV_QP_STATE	||
|RESET->INIT|	IBV_QP_STATE，<br /> IBV_QP_PKEY_INDEX，<br /> IBV_QP_PORT，<br /> IBV_QP_ACCESS_FLAGS||	
|INIT->INIT	||	IBV_QP_PKEY_INDEX，<br /> IBV_QP_PORT， <br /> IBV_QP_ACCESS_FLAGS
|INIT->RTR	|IBV_QP_STATE，<br />IBV_QP_PATH_MTU， <br /> IBV_QP_AV，<br /> IBV_QP_DEST_QPN，<br />IBV_QP_RQ_PSN|	IBV_QP_PKEY_INDEX，<br /> IBV_QP_ALT_PATH，<br />IBV_QP_ACCESS_FLAGS
|RTR->RTS|	IBV_QP_STATE，<br /> IBV_QP_SQ_PSN	|IBV_QP_CUR_STATE，<br />IBV_QP_ACCESS_FLAGS，<br />IBV_QP_ALT_PATH，<br /> IBV_QP_PATH_MIG_STATE|
|RTS->RTS|	|	IBV_QP_CUR_STATE，<br /> IBV_QP_ACCESS_FLAGS，<br /> IBV_QP_ALT_PATH，<br /> IBV_QP_PATH_MIG_STATE
|RTS->SQD|	IBV_QP_STATE	|IBV_QP_EN_SQD_ASYNC_NOTIFY|
|SQD->RTS	|IBV_QP_STATE|	IBV_QP_CUR_STATE，<br /> IBV_QP_ACCESS_FLAGS，<br /> IBV_QP_ALT_PATH，<br /> IBV_QP_PATH_MIG_STATE
|SQD->SQD	|	|IBV_QP_PKEY_INDEX，<br /> IBV_QP_ACCESS_FLAGS，<br />IBV_QP_AV，<br />IBV_QP_ALT_PATH，<br /> IBV_QP_PATH_MIG_STATE
|SQE->RTS|	IBV_QP_STATE	|IBV_QP_CUR_STATE，<br /> IBV_QP_ACCESS_FLAGS


### 7.1.3 RC QP状态转移

下表指定了服务类型为IBV_QPT_RC的QP的完全受支持的转换及其接受的属性：
|状态转移类型|	必需属性|	可选属性|
|:--|:--|:--|
|\*->RESET|	IBV_QP_STATE	||
|\*->ERR|	IBV_QP_STATE	||
|RESET->INIT|	IBV_QP_STATE，<br /> IBV_QP_PKEY_INDEX，<br /> IBV_QP_PORT，<br /> IBV_QP_ACCESS_FLAGS	
|INIT->INIT	||	IBV_QP_PKEY_INDEX，<br /> IBV_QP_PORT，<br /> IBV_QP_ACCESS_FLAGS
INIT->RTR|	IBV_QP_STATE，<br /> IBV_QP_AV，<br /> IBV_QP_PATH_MTU，<br /> IBV_QP_DEST_QPN，<br /> IBV_QP_RQ_PSN，<br /> IBV_QP_MAX_DEST_RD_ATOMIC，<br /> IBV_QP_MIN_RNR_TIMER|	IBV_QP_PKEY_INDEX，<br />IBV_QP_ACCESS_FLAGS，<br />IBV_QP_ALT_PATH
|RTR->RTS|	IBV_QP_STATE，<br /> IBV_QP_SQ_PSN，<br /> IBV_QP_TIMEOUT，<br /> IBV_QP_RETRY_CNT，<br /> IBV_QP_RNR_RETRY，<br /> IBV_QP_MAX_QP_RD_ATOMIC	|IBV_QP_CUR_STATE，<br />IBV_QP_ACCESS_FLAGS，<br />IBV_QP_MIN_RNR_TIMER，<br /> IBV_QP_ALT_PATH，<br /> IBV_QP_PATH_MIG_STATE
|RTS->RTS|	|	IBV_QP_CUR_STATE，<br /> IBV_QP_ACCESS_FLAGS，<br />IBV_QP_MIN_RNR_TIMER，<br />IBV_QP_ALT_PATH，<br /> IBV_QP_PATH_MIG_STATE
|RTS->SQD|	IBV_QP_STATE|IBV_QP_EN_SQD_ASYNC_NOTIFY
|SQD->RTS|	IBV_QP_STATE|	IBV_QP_CUR_STATE，<br />IBV_QP_ACCESS_FLAGS，<br />IBV_QP_MIN_RNR_TIMER，<br /> IBV_QP_ALT_PATH，<br /> IBV_QP_PATH_MIG_STATE
|SQD->SQD|	|	IBV_QP_PKEY_INDEX，<br />IBV_QP_PORT，<br />IBV_QP_ACCESS_FLAGS，<br />IBV_QP_AV，<br />IBV_QP_MAX_QP_RD_ATOMIC，<br />IBV_QP_MIN_RNR_TIMER，<br />IBV_QP_ALT_PATH，<br />IBV_QP_TIMEOUT，<br />IBV_QP_RETRY_CNT，<br />IBV_QP_RNR_RETRY，<br /> IBV_QP_MAX_DEST_RD_ATOMIC，<br />IBV_QP_PATH_MIG_STATE


### 7.1.4 示例
```cpp
//将UD QP从Reset状态修改为RTS状态:
//假设变量my_port, my_psn已声明并用有效值初始化**
struct ibv_qp *qp;
struct ibv_qp_attr attr;

memset(&attr, 0, sizeof(attr));

attr.qp_state	 IBV_QPS_INIT;
attr.pkey_index	 0;
attr.port_num	 my_port;
attr.qkey	 0x22222222;

if (ibv_modify_qp(qp, &attr,
		  IBV_QP_STATE      |
		  IBV_QP_PKEY_INDEX |
		  IBV_QP_PORT       |
		  IBV_QP_QKEY)) {
	fprintf(stderr, "Failed to modify QP to INIT\n");
	return -1;
}

memset(&attr, 0, sizeof(attr));

attr.qp_state		= IBV_QPS_RTR;

if (ibv_modify_qp(qp, &attr, IBV_QP_STATE)) {
	fprintf(stderr, "Failed to modify QP to RTR\n");
	return -1;
}

memset(&attr, 0, sizeof(attr));

attr.qp_state		 IBV_QPS_RTS;
attr.sq_psn		 my_psn;

if (ibv_modify_qp(qp, &attr,
		  IBV_QP_STATE |
		  IBV_QP_SQ_PSN)) {
	fprintf(stderr, "Failed to modify QP to RTS\n");
	return 1;
}

//将UC QP从Reset状态修改为RTS状态:
//假设变量 my_port, my_psn, my_mtu, my_sl, remote_qpn, remote_psn, remote_lid 已声明并用有效值初始化**
struct ibv_qp *qp;
struct ibv_qp_attr attr;

memset(&attr, 0, sizeof(attr));

attr.qp_state	 IBV_QPS_INIT;
attr.pkey_index	 0;
attr.port_num	 my_port;
attr.qp_access_flags	 0;

if (ibv_modify_qp(qp, &attr,
		  IBV_QP_STATE        |
		  IBV_QP_PKEY_INDEX   |
		  IBV_QP_PORT         |
		  IBV_QP_ACCESS_FLAGS)) {
	fprintf(stderr, "Failed to modify QP to INIT\n");
	return -1;
}

memset(&attr, 0, sizeof(attr));

attr.qp_state		= IBV_QPS_RTR;
attr.path_mtu		= my_mtu;
attr.dest_qp_num	= remote_qpn;
attr.rq_psn		= remote_psn;
attr.ah_attr.is_global		 0;
attr.ah_attr.dlid		 remote_lid;
attr.ah_attr.sl			 my_sl;
attr.ah_attr.src_path_bits	 0;
attr.ah_attr.port_num		 my_port;

if (ibv_modify_qp(qp, &attr,
		  IBV_QP_STATE    |
		  IBV_QP_AV       |
		  IBV_QP_PATH_MTU |
		  IBV_QP_DEST_QPN |
		  IBV_QP_RQ_PSN)) {
	fprintf(stderr, "Failed to modify QP to RTR\n");
	return -1;
}

memset(&attr, 0, sizeof(attr));

attr.qp_state		 IBV_QPS_RTS;
attr.sq_psn		 my_psn;

if (ibv_modify_qp(qp, &attr,
		  IBV_QP_STATE  |
		  IBV_QP_SQ_PSN)) {
	fprintf(stderr, "Failed to modify QP to RTS\n");
	return 1;
}

//将RC QP从Reset状态修改为RTS状态:
//假设变量 my_port, my_psn, my_mtu, my_sl, remote_qpn, remote_psn, remote_lid 已声明并用有效值初始化**
struct ibv_qp *qp;
struct ibv_qp_attr attr;

memset(&attr, 0, sizeof(attr));

attr.qp_state	 IBV_QPS_INIT;
attr.pkey_index	 0;
attr.port_num	 my_port;
attr.qp_access_flags	 0;

if (ibv_modify_qp(qp, &attr,
		  IBV_QP_STATE      |
		  IBV_QP_PKEY_INDEX |
		  IBV_QP_PORT       |
		  IBV_QP_ACCESS_FLAGS)) {
	fprintf(stderr, "Failed to modify QP to INIT\n");
	return -1;
}

memset(&attr, 0, sizeof(attr));

attr.qp_state		= IBV_QPS_RTR;
attr.path_mtu		= my_mtu;
attr.dest_qp_num	= remote_qpn;
attr.rq_psn		= remote_psn;
attr.max_dest_rd_atomic	= 1;
attr.min_rnr_timer	= 12;
attr.ah_attr.is_global		 0;
attr.ah_attr.dlid		 remote_lid;
attr.ah_attr.sl			 my_sl;
attr.ah_attr.src_path_bits	 0;
attr.ah_attr.port_num		 my_port;

if (ibv_modify_qp(qp, &attr,
		  IBV_QP_STATE              |
		  IBV_QP_AV                 |
		  IBV_QP_PATH_MTU           |
		  IBV_QP_DEST_QPN           |
		  IBV_QP_RQ_PSN             |
		  IBV_QP_MAX_DEST_RD_ATOMIC |
		  IBV_QP_MIN_RNR_TIMER)) {
	fprintf(stderr, "Failed to modify QP to RTR\n");
	return -1;
}

memset(&attr, 0, sizeof(attr));

attr.qp_state		 IBV_QPS_RTS;
attr.sq_psn		 my_psn;
attr.timeout		 14;
attr.retry_cnt		 7;
attr.rnr_retry		 7; /* infinite */
attr.max_rd_atomic	 1;

if (ibv_modify_qp(qp, &attr,
		  IBV_QP_STATE              |
		  IBV_QP_TIMEOUT            |
		  IBV_QP_RETRY_CNT          |
		  IBV_QP_RNR_RETRY          |
		  IBV_QP_SQ_PSN             |
		  IBV_QP_MAX_QP_RD_ATOMIC)) {
	fprintf(stderr, "Failed to modify QP to RTS\n");
	return 1;
}
```

## 7.2 加入和离开多播组（差mojo）
### 7.2.1 ibv_attach_mcast
**函数原型：**
```cpp
int ibv_attach_mcast(struct ibv_qp *qp, const union ibv_gid *gid, uint16_t lid)
```
**输入参数：**
* qp——附加到多播组的QP。
* gid——多播组GID
* lid——主机字节顺序的多播组LID。

**输出参数：** 无。

**返回值：** 成功返回0，失败errno来指示失败的原因。

**说明：**
ibv_attach_mcast 向多播GID为gid，多播LID为lid的多播组中添加队列对qp。

只能将传输服务类型IBV_QPT_UD的QPs添加到多播组。

为了接收多播消息，必须将对多播组的加入请求发送给子网管理员（SA），以便将网络结构的多播路由配置为将消息传递到本地端口。

如果QP多次连接到同一多播组，则QP仍将接收多播消息的单个副本。


### 7.2.2 ibv_detach_mcast
**函数原型：**
```cpp
int ibv_detach_mcast(struct ibv_qp *qp, const union ibv_gid *gid, uint16_t lid)
```
**输入参数：**
* qp——附加到多播组的QP。
* gid——多播组GID
* lid——主机字节顺序的多播组LID。
**输出参数：** 无。

**返回值：** 成功返回0，失败返回errno来指示失败的原因。

**说明：** ibv_detach_mcast 从多播组GID是gid，多播LID是lid的多播组中移除队列对qp。

## 7.3 创建和销毁流（差mojo）
### 7.3.1 ibv_create_flow
**函数原型：** 
```cpp
struct ibv_flow *ibv_create_flow(struct ibv_qp *qp, struct ibv_flow_attr *flow)
```
**输入参数：** 

* qp——队列对，来自ibv_create_qp。
* flow_attr——流属性，详细信息见下文。

**输出参数：** 无。

**返回值：** 成功发返回指向struct ibv_flow的指针，失败返回NULL并设置errno显示失败原因。

* EINVAL——流规范、QP或优先级无效。
* ENOMEM——没有足够内存。
* ENXIO——当前不支持管理流定向的设备。
* EPERM——没有权限来添加流定向规则。

**说明：**

ibv_create_flow将qp添加到一个指定的流flow中。

这些动词仅适用于支持IBV_DEVICE_MANAGED_FLOW_STEERING的设备，并且仅适用于传输服务类型IBV_QPT_UD或IBV_QPT_RAW_PACKET的QP。

用户必须在使用规范结构体之前将其设置为零。

ibv_flow_eth_filter中的ether_type字段是数据包最后一个VLAN标记之后的ethertype。

仅规则类型IBV_FLOW_ATTR_NORMAL支持IBV_FLOW_ATTR_FLAGS_DONT_TRAP标志。

IBV_FLOW_ATTR_SNIFFER规则类型不需要任何规范。

当设置了IBV_FLOW_ATTR_FLAGS_EGRESS标志时，qp参数仅用作获取设备的手段。

struct ibv_flow定义如下：
```cpp
struct ibv_flow {
	uint32_t			comp_mask;
	struct ibv_context	*context;	//设备上下文，来自ibv_open_deivce
	uint32_t			handle;
};
```

struct ibv_flow_attr定义如下：
```cpp
struct ibv_flow_attr {
	uint32_t					comp_mask;		/* 未来的可扩展性 */
	enum ibv_flow_attr_type	type;			/* 规则类型，详细信息见下文 */
	uint16_t					size;			/* 命令的大小 */
	uint16_t					priority;		/* 规则优先级，详细信息见下文 */
	uint8_t						num_of_specs;	/* ibv_flow_spec_xxx 的数量 */
	uint8_t						port;			/* 上行端口号 */
	uint32_t					flags; 		/* 规则的额外标志，enum ibv_flow_flags，详细信息见下文 */
	/* 以下是根据用户要求的可选层
		* struct ibv_flow_spec_xxx [L2]
		* struct ibv_flow_spec_yyy [L3/L4]
	*/
};
```
enum ibv_flow_attr_type定义如下：

```cpp
enum ibv_flow_attr_type {
	IBV_FLOW_ATTR_NORMAL		= 0x0,	/* 根据规则的规范进行定向 */
	IBV_FLOW_ATTR_ALL_DEFAULT	= 0x1,	/* 默认单播和多播规则-接收所有未定向到任何QP的Eth流量 */
	IBV_FLOW_ATTR_MC_DEFAULT	= 0x2,	/* 默认多播规则-接收未定向到任何QP的所有Eth多播流量 */
	IBV_FLOW_ATTR_SNIFFER		= 0x3,	/* 嗅探器规则-接收所有端口流量 */
};

```

enum ibv_flow_flags定义如下：
```cpp
enum ibv_flow_flags {
	IBV_FLOW_ATTR_FLAGS_ALLOW_LOOP_BACK	= 1 << 0,	/* 将规则应用于从添加的QP通过环回发送的数据包 */
	IBV_FLOW_ATTR_FLAGS_DONT_TRAP		= 1 << 1,	/* 规则不捕获接收到的数据包，从而允许它们匹配优先级较低的规则 */
	IBV_FLOW_ATTR_FLAGS_EGRESS			= 1 << 2,	/* 根据EGRESS规则匹配发送的数据包，并在需要时执行关联的操作 */
};

```
enum ibv_flow_spec_type定义如下：
```cpp
enum ibv_flow_spec_type {
	IBV_FLOW_SPEC_ETH			= 0x20,	/* L2头部的流规范 */
	IBV_FLOW_SPEC_IPV4			= 0x30,	/* IPv4头部的流规范 */
	IBV_FLOW_SPEC_IPV6			= 0x31,	/* IPv6头部的流规范 */
	IBV_FLOW_SPEC_IPV4_EXT		= 0x32,	/* IPv4的扩展流规范 */
	IBV_FLOW_SPEC_ESP			= 0x34,	/* FESP（IPSec）头部的流规范 */
	IBV_FLOW_SPEC_TCP			= 0x40,	/* TCP头部的流规范 */
	IBV_FLOW_SPEC_UDP			= 0x41,	/* UDP头部的流规范 */
	IBV_FLOW_SPEC_VXLAN_TUNNEL	= 0x50,	/* VXLAN头部的流规范 */
	IBV_FLOW_SPEC_GRE			= 0x51,	/* GRE标头部的流规范 */
	IBV_FLOW_SPEC_MPLS			= 0x60,	/* MPLS标头部的流规范 */
	IBV_FLOW_SPEC_INNER			= 0x100,	/* 将L2/L3/L4规范应用于内部头部的标志 */
	IBV_FLOW_SPEC_ACTION_TAG	= 0x1000,	/* 标记匹配数据包的动作 */
	IBV_FLOW_SPEC_ACTION_DROP	= 0x1001,	/* 丢弃匹配数据包的动作 */
	IBV_FLOW_SPEC_ACTION_HANDLE	= 0x1002,	/* 执行ibv_create_flow_action_xxxx动词创建的操作 */
	IBV_FLOW_SPEC_ACTION_COUNT	= 0x1003,	/* 用ibv_counters句柄的计算匹配的包的动作 */
};
```

struct ibv_flow_spec_xxx定义如下：
```cpp
struct ibv_flow_spec_xxx {	//xxx可以为：eth、ipv4、ipv4_ext、ipv6、esp、tcp_udp、gre、mpls、tunnel
	enum ibv_flow_spec_type	type;	/* 流规范类型，详细信息见ibv_create_flow */
	uint16_t					size;	/* 流规范大小= sizeof（struct ibv_flow_spec_xxx */
	struct ibv_flow_xxx_filter	val;
	struct ibv_flow_xxx_filter	mask;	/* 定义在输入数据包中查找匹配项时，过滤器值中哪些位适用 */
};

```
struct ibv_flow_xxx_filter定义如下：
```cpp
struct ibv_flow_eth_filter {
	uint8_t			dst_mac[6];
	uint8_t			src_mac[6];
	uint16_t		ether_type;
	/* 与802.1q相同的布局: prio 3, cfi 1, vlan id 12 */
	uint16_t		vlan_tag;
};

struct ibv_flow_ipv4_filter {
	uint32_t		rc_ip;
	uint32_t		dst_ip;
}

struct ibv_flow_ipv4_ext_filter {
	uint32_t		src_ip;
	uint32_			dst_ip;
	uint8_t			proto;
	uint8_t			tos;
	uint8_t			ttl;
	uint8_t			flags;
}

struct ibv_flow_ipv6_filter {
	uint8_t			src_ip[16];
	uint8_t			dst_ip[16];
	uint32_t		flow_label;
	uint8_t			next_hdr;
	uint8_t			traffic_class;
	uint8_t			hop_limit;
}

struct ibv_flow_esp_filter {
	uint32_t		spi;
	uint32_t		seq;
};

struct ibv_flow_tcp_udp_filter {
	uint16_t		dst_port;
	uint16_t		src_port;
}

struct ibv_flow_gre_filter {
	/* c_ks_res0_ver字段是标准GRE头部的偏移量0中的位0-15：
	 * bit 0 - 校验和存在位
	 * bit 1 - 保留位，置0
	 * bit 2 - 密钥存在位.
	 * bit 3 - 顺序号存在位
	 * bits 4:12 - 保留位，置0
	 * bits 13:15 - GRE版本
	 */
	uint16_t		c_ks_res0_ver;
	uint16_t		protocol;
	uint32_t		key;
}

struct ibv_flow_mpls_filter {
	/* 该字段包含整个MPLS标签：
	 * bits 0:19  - 标签值字段
	 * bits 20:22 - 流量类别字段
	 * bits 23   - 栈底位.
	 * bits 24:31 - ttl字段.
	 */
	uint32_t		label;
};

struct ibv_flow_tunnel_filter {
	uint32_t		tunnel_id;
};
```

每个规范结构体都包含相关的网络层参数以进行匹配。为了强制匹配，用户为每个参数设置一个掩码。

来自网线的数据包与流规范匹配。 如果找到匹配项，则对数据包执行关联的流操作。

在入口的流中，QP参数被视为另一个将数据包分散到指定QP的动作。
       
如果在掩码中设置了该位，则值中的对应位应被匹配。

请注意，大多数供应商都支持全掩码（全“ 1”）或零掩码（全“ 0”）。

相关网络结构体中的网络参数应以网络顺序（big endian）给出。

***流域和优先级***

流定向定义了域和优先级的概念。每个域代表一个可以附加一个流的应用程序。域是优先的。当它们的流规范重叠时，优先级较高的域将始终取代优先级较低的域。
 
IB动词具有较高的优先级域。

除域外，每个域中都有优先级。较低优先级数值（较高优先级）优先于具有较高数字优先级值（较低优先级）的匹配规则。重要的是要注意，流规范的优先级值不仅用于建立冲突流匹配的优先级，还用于抽象流规范测试是否匹配的顺序。优先级较高的流将在优先级较低的流之前进行测试。

***规则定义排序***

应用程序可以按无序方案提供ibv_flow_spec_xxx规则。在这种情况下，每个规范都应定义良好并与特定的网络头层匹配。在某些情况下，当规范列表中存在某些流规范类型时，需要以有序方式提供列表，以便严格定义该流规范类型在协议堆栈中的位置。

当需要排序的某些规范类型驻留在内部网络协议堆栈中（在隧道协议中）时，应将排序应用于内部网络规范，并应使用IBV_FLOW_SPEC_INNER标志将其与内部规范指示相结合。 例如：试图匹配内部网络中MPLS标签的MPLS规范应设置IBV_FLOW_SPEC_INNER标志，其余内部网络规范也应设置IBV_FLOW_SPEC_INNER标志。 最重要的是，应按有序方式提供所有内部网络规范。 这对于表示许多封装隧道协议至关重要。

需要这种排序的流规范类型为：
1. IBV_FLOW_SPEC_MPLS-

由于MPLS头部可以出现在协议栈中的多个位置，也可以封装在不同层的顶部，因此需要根据此规范在协议栈中的确切位置来放置此规范。

**示例：** 

下面的flow_attr，定义了优先级0的规则，以匹配目标mac地址和源ipv4地址。为此，使用了L2和L3规范。

如果此规则有命中，则意味着接收到的数据包具有目标mac：66：11：22：33：44：55和源ip：0x0B86C806，该数据包将被定向到其附加的qp

```cpp
struct raw_eth_flow_attr {
	struct ibv_flow_attr		attr;
	struct ibv_flow_spec_eth	spec_eth;
	struct ibv_flow_spec_ipv4	spec_ipv4;
} __attribute__((packed));

struct raw_eth_flow_attr flow_attr = {
	.attr		= {
		.comp_mask		= 0,
		.type			= IBV_FLOW_ATTR_NORMAL,
		.size			= sizeof(flow_attr),
		.priority		= 0,
		.num_of_specs	= 2,
		.port			= 1,
		.flags			= 0,
	},
	.spec_eth 	= {
		.type			= IBV_FLOW_SPEC_ETH,
		.size			= sizeof(struct ibv_flow_spec_eth),
		.val			= {
			.dst_mac		= { 0x66, 0x11, 0x22, 0x33, 0x44, 0x55},
			.src_mac		= { 0x00, 0x00, 0x00, 0x00, 0x00, 0x00},
			.ether_type		= 0,
			.vlan_tag		= 0,
		},
		.mask			= {
			.dst_mac		= { 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF},
			.src_mac		= { 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF},
			.ether_type		= 0,
			.vlan_tag		= 0,
		}
	},
	.spec_ipv4	= {
		.type			= IBV_FLOW_SPEC_IPV4,
		.size			= sizeof(struct ibv_flow_spec_ipv4),
		.val			= {
			.src_ip			= 0x0B86C806,
			.dst_ip			= 0,
		},
		.mask			= {
			.src_ip			= 0xFFFFFFFF,
			.dst_ip			= 0,
		}
	}
};

```

### 7.3.2 ibv_destroy_flow
**函数原型：** 
```cpp
int ibv_destroy_flow(struct ibv_flow *flow_id)
```
**输入参数：** 

**输出参数**

**返回值：** 成功发返回0，失败返回errno显示失败原因。EINVAL——flowid无效。

**说明：** ibv_destroy_flow销毁指定的流。

# 8 发布工作请求
## 8.1 发布WR到QP
### 8.1.1 ibv_post_recv（差mojo）
**函数原型：**
```cpp
int ibv_post_recv(struct ibv_qp *qp, struct ibv_recv_wr *wr, struct ibv_recv_wr **bad_wr)
```
**输入参数：**

* qp——来自ibv_create_qp的struct ibv_qp。
* wr——包含接收缓冲区的第一个工作请求(WR)。

**输出参数：** bad_wr——指向第一个被拒绝的WR的指针。

**返回值：** 成功返回0；出错返回errno来指示失败的原因。

**说明：**
ibv_post_recv将以wr开始的工作请求（WR）链表发布到队列对qp的接收队列中。 至少一个接收缓冲区应发布到接收队列，以将QP状态转换为RTR。 远程对等方执行SEND，带立即数SEND和带立即数RDMA WRITE入时，将消耗接收缓冲区。 接收缓冲区不用于其他RDMA操作。 在出现第一个错误时，将停止处理WR列表，并通过bad_wr中返回失败的WR。

WR使用的缓冲区只有在WR完全执行了请求，并且已经从相应的完成队列（CQ）中检索到工作完成之后，才能安全地重用。

如果QP qp与一个SRQ关联，则必须使用函数ibv_post_srq_recv()，而不是ibv_post_recv()，因为不会使用QP自己的接收队列。

如果将WR发布到UD QP，则传入消息的全局路由头（GRH）将放置在分散列表中缓冲区的前40个字节中。 如果传入消息中不存在GRH，则前几个字节将不确定。 这意味着在所有情况下，对于UD QP，传入消息的实际数据将以40个字节的偏移量开始进入分散列表中的缓冲区。

struct ibv_recv_wr定义如下:
```cpp
struct ibv_recv_wr {
	uint64_t			wr_id;		//用户分配的工作请求ID
	struct ibv_recv_wr	*next;		//指向下一个WR的指针，如果最后一个,则为空。
	struct ibv_sge		*sg_list;	//这个WR的离散/聚合数组
	int					num_sge;	//sg_list中的条目数
};
```
struct ibv_sge定义如下:

```cpp
struct ibv_sge {
	uint64_t addr;		//本地内存缓冲区的起始地址，或地址是IBV_ZERO_BASED的MR的数量
	uint32_t length;	//缓冲区长度
	uint32_t lkey;		//来自ibv_reg_mr的本地MR的密钥lkey
};
```

### 8.1.2 ibv_post_send（差mojo）
**函数原型：**
```cpp
int ibv_post_send(struct ibv_qp *qp, struct ibv_send_wr *wr, struct ibv_send_wr **bad_wr)
```
**输入参数：**
* qp——来自ibv_create_qp的struct ibv_qp。
* wr——第一个工作请求(WR)。

**输出参数：** bad_wr——指向第一个被拒绝的WR的指针。

**返回值：** 成功返回0；失败返回errno来指示失败的原因。

**说明：**
ibv_post_send将以wr开始的工作请求（WR）的链表发布到队列对qp的发送队列中。 此操作用于启动所有通信，包括RDMA操作。 在出现第一个错误时，将停止处理WR列表（这可以在发布WRs时立即检测到），并通过bad_wr中返回失败的WR。

在请求已完全执行并且从相应的完成队列（CQ）中检索到完成队列条目（CQE）之前，用户不得更改或销毁与WR相关的AH，以避免意外行为。

只有在WR已完全执行并且已从相应的CQ检索到WCE之后，才能安全地重用WR使用的缓冲区。 但是，如果设置了IBV_SEND_INLINE标志，则在调用返回后可以立即重用该缓冲区。**


struct ibv_send_wr定义如下：
```cpp
struct ibv_send_wr {
	uint64_t							wr_id;			//用户分配的WR ID
	struct ibv_send_wr					*next;			//指向下一个WR的指针，如果最后一个,则为NULL
	struct ibv_sge *					sg_list;		//WR的离散/聚合数组，详细信息见ibv_post_recv
	int									num_sge;		//sg_list中的条目数量。
	enum ibv_wr_opcode					opcode;			//操作类型，枚举值之一，详细信息见下文
	enum ibv_send_flags				send_flags;		//WR属性标志，0或按位或得到，详细信息见下文
	union {
		__be32							imm_data;		//当操作数是*_WITH_IMM时，以网络字节顺序发送的立即数据，
		uint32_t						invalidate_rkey;//当操作数是*_WITH_INV时，存放无效的远程密钥
	};
	union {
		struct {
			uint64_t					remote_addr;	//远程内存缓冲区的起始地址
			uint32_t					rkey;			//远程MR的密钥(来自远端的ibv_reg_mr)
		} rdma;										//用于一般RDMA操作
		struct {
			uint64_t					remote_addr;	//远程内存缓冲区的起始地址
			uint64_t					compare_add;	//用于比较和添加操作的值
			uint64_t					swap;			//用于交换的值
			uint32_t					rkey;			//远程MR的密钥
		} atomic;										//用于原子操作
		struct {
			struct ibv_ah				*ah;			//远程节点地址的地址句柄(AH)
			uint32_t					remote_qpn;		//目标QP的QP号
			uint32_t					remote_qkey;	//目标QP的Qkey
		} ud;											//用于UD操作
	} wr;
	union {
		struct {
			uint32_t					remote_srqn;	//目标扩展可靠连接(XRC)的共享接收队列(SRQ)编号。仅用于XRC操作。
		}xrc;
	}qp_type;
	union {
		struct {
			struct ibv_mw				*mw;			//要绑定的MW，详细信息见ibv_alloc_mw
			uint32_t					rkey;			//MW期望的rkey
			struct ibv_mw_bind_info	bind_info;		//MW的附加绑定信息,详细信息见ibv_bind_mw
		} bind_mw;
		struct {
			void						*hdr;			//内联头部的指针地址
			uint16_t					hdr_sz;			//内联头部的大小
			uint16_t					mss;			//每个TSO片段的最大分段大小
		}tso;
	};
};
```

enum ibv_send_flags定义如下：
```cpp
enum ibv_send_flags {
	IBV_SEND_FENCE		= 1 << 0, //设置栅栏指示器。仅对IBV_QPT_RC的QP有效
	IBV_SEND_SIGNALED	= 1 << 1, //设置完成通知指示器。 仅当使用sq_sig_all= 0创建QP时相关
	IBV_SEND_SOLICITED	= 1 << 2, //设置请求的事件指示器。 仅对发送和带立即数的RDMA写有效
	IBV_SEND_INLINE		= 1 << 3, /*将给定聚合列表中的数据作为发送WQE中的内联数据发送。
										仅对发送和RDMA写有效。 L_Key将不会被检查 */
	IBV_SEND_IP_CSUM	= 1 << 4  /*卸载IPv4和TCP/UDP校验和计算。仅当device_attr中的
										device_cap_flags指示当前QP支持校验和卸载时有效*/
  };
```
enum ibv_wr_opcode的定义如下：
```cpp
enum ibv_wr_opcode {
	IBV_WR_RDMA_WRITE,				//写
	IBV_WR_RDMA_WRITE_WITH_IMM,		//带立即数的写
	IBV_WR_SEND,					//发送
	IBV_WR_SEND_WITH_IMM,			//带立即数发送
	IBV_WR_RDMA_READ,				//读
	IBV_WR_ATOMIC_CMP_AND_SWP,		//原子的比较和交换
	IBV_WR_ATOMIC_FETCH_AND_ADD,	//原子的取回和添加
	IBV_WR_LOCAL_INV,				//
	IBV_WR_BIND_MW,					//
	IBV_WR_SEND_WITH_INV,			//带无效的密钥发送
	IBV_WR_TSO,						//
	IBV_WR_DRIVER1,					//用于发出特定驱动程序操作
};
```

不同类型的QP支持的操作码如下：

|操作码(IBV_WR_\*)|IBV_QPT_UD|IBV_QPT_UC|IBV_QPT_RC|IBV_QPT_XRC_SEND|IBV_QPT_RAW_PACKET|
|:--|:--:|:--:|:--:|:--:|:--:|
|SEND|V|V|V|V|V|
|SEND_WITH_IMM|V|V|V|V||
|RDMA_WRITE||V|V|V||
|RDMA_WRITE_WITH_IMM||V|V|V||
|RDMA_READ|||V|V||
|ATOMIC_CMP_AND_SWP|||V|V||
|ATOMIC_FETCH_AND_ADD|||V|V||
|LOCAL_INV||V|V|V||
|BIND_MW||V|V|V||
|SEND_WITH_INV||V|V|V||
|TSO|V||||V|

struct ibv_mw_bind_info定义如下：


## 8.2 发布WR到SRQ

### 8.2.1 ibv_post_srq_recv（差mojo）
**函数原型：**
```cpp
int ibv_post_srq_recv(struct ibv_srq *srq, struct ibv_recv_wr *recv_wr, struct ibv_recv_wr **bad__wr)
```
**输入参数：**
*  srq——将工作请求发布到的SRQ，来自ibv_create_srq。
* recv_wr，要发布到接收队列的工作请求列表， 详细信息见ibv_post_recv。

**输出参数：** bad_wr——指向第一个被拒绝的WR的指针。

**返回值：** 成功返回0，失败返回errno来指示失败的原因。

**说明：**
ibv_post_srq_recv将以wr开始的工作请求链表发布到指定的SRQ srq中。 它在第一次失败时停止处理此列表中的WR（可以在发布请求时立即检测到该WR），并通过bad_wr参数返回失败的WR。

WR使用的缓冲区只有在WR完全执行了请求并且已经从相应的完成队列（CQ）中检索到工作完成之后，才能安全地重用。

如果将WR发布到UD QP，则传入消息的全局路由头（GRH）将放置在分散列表中缓冲区的前40个字节中。 如果传入消息中不存在GRH，则前40个字节将未定义的。 这意味着在所有情况下，对于UD QP，传入消息的实际数据将以40个字节的偏移量开始进入分散列表中的缓冲区。

# 9 获取和确认事件
## 9.1 获取和确认事件
### 9.1.1 ibv_get_cq_event（差mojo）
**函数原型：**
```cpp
int ibv_get_cq_event(struct ibv_comp_channel *channel, struct ibv_cq **cq, void **cq_context)
```
**输入参数：**
* channel——来自ibv_create_comp_channel的struct ibv_comp_channel。

**输出参数：**
* cq——指向与事件关联的完成队列(CQ)的指针。
* cq_context——用户在ibv_create_cq中提供的上下文。

**返回值：** 成功返回0；出错返回-1。如果调用失败，将设置errno来指示失败的原因。

**说明：**
ibv_get_cq_event等待完成通道chnnel上的下一个完成事件， 并用事件的CQ和CQ上下文填充cq和cq_context。 请注意，这是一个阻塞操作。 用户应分配一个指向struct ibv_cq的指针和一个void指针，以传递给函数。 返回时将使用适当的值填充它们。 ***用户负责释放这些指针***。

 ibv_get_cq_event返回的每个完成事件都必须使用ibv_ack_cq_events操作确认。为了避免竞争，销毁CQ将等待所有的完成事件被确认。这保证了获取事件和确认事件一一对应。 由于ibv_destroy_cq操作等待所有事件被确认，因此如果未正确确认事件，它将挂起。

一旦在CC上发送了关于完成队列（CQ）的通知，该CQ现在将被“取消装备”，并且将不会再次向CC发送任何通知，直到通过新的调用ibv_req_notify_cq操作重新装备为止。

此操作仅通知用户CQ具有要处理的完成队列条目（CQE），而实际上不处理CQE。 用户应使用ibv_poll_cq操作来处理CQE。

**示例：** 见ibv_ack_cq_events。

### 9.1.2 ibv_ack_cq_events（差mojo）
**函数原型：**
```cpp
void ibv_ack_cq_events(struct ibv_cq *cq, unsigned int nevents)
```
**输入参数：**
* cq——来自ibv_create_cq的struct ibv_cq。
* nevents——需要确认的事件数量(1…n)。


**输出参数：** 无。

**返回值：** 无。

**说明：**
ibv_ack_cq_events确认从ibv_get_cq_event接收到的事件。 尽管从ibv_get_cq_event收到的每个通知仅算作一个事件，但用户可以通过一次调用ibv_ack_cq_events来确认多个事件。 要确认的事件数以nevents传递，并且至少应为1。由于此操作使用互斥锁，因此它的成本较高，所以一次调用中确认多个事件可能会提供更好的性能。

有关更多详细信息，请参见ibv_get_cq_event。

**示例：**
下列代码展示了一种可能的用法：
* 步骤一：准备
  1. 创建CQ
  2. 在新的（第一个）完成事件时请求通知
* 步骤二：完成处理例程
  3. 等待完成事件、确认
  4. 在下一次完成事件时请求通知
  5. 清空CQ

注意，可能会触发额外事件，而在CQ中不具有相应的完成条目。 如果将一个CQE添加到步骤4和步骤5之间的CQ，然后在步骤5中清空（轮询）CQ，则会发生这种情况。


```cpp
cq	 ibv_create_cq(ctx, 1, ev_ctx, channel, 0);
if (!cq) {
	fprintf(stderr, "Failed to create CQ\n");
	return 1;
}

/* Request notification before any completion can be created */
if (ibv_req_notify_cq(cq, 0)) {
	fprintf(stderr, "Couldn't request CQ notification\n");
	return 1;
}

		.
		.
		.

/* Wait for the completion event */
if (ibv_get_cq_event(channel, &ev_cq, &ev_ctx)) {
	fprintf(stderr, "Failed to get cq_event\n");
	return 1;
}

/* Ack the event */
ibv_ack_cq_events(ev_cq, 1);

/* Request notification upon the next completion event */
if (ibv_req_notify_cq(ev_cq, 0)) {
	fprintf(stderr, "Couldn't request CQ notification\n");
	return 1;
}

/* Empty the CQ: poll all of the completions from the CQ (if any exist) */
do {
	ne	 ibv_poll_cq(cq, 1, &wc);
	if (ne < 0) {
		fprintf(stderr, "Failed to poll completions from the CQ\n");
		return 1;
	}

	/* there may be an extra event with no completion in the CQ */
	if (ne	= 0)
		continue;

	if (wc.status != IBV_WC_SUCCESS) {
		fprintf(stderr, "Completion with status 0x%x was found\n", wc.status);
		return 1;
	}
} while (ne);
```
 下列代码展示了一种非阻塞模型的可能用法：

1. 设置完成事件通道为非阻塞
2. 轮询事件通道知道有一个完成事件
3. 获取事件并确认

```cpp
/* change the blocking mode of the completion channel */
flags	 fcntl(channel->fd, F_GETFL);
rc	 fcntl(channel->fd, F_SETFL, flags | O_NONBLOCK);
if (rc < 0) {
	fprintf(stderr, "Failed to change file descriptor of completion event channel\n");
	return 1;
}

/*
 * poll the channel until it has an event and sleep ms_timeout
 * milliseconds between any iteration
 */
my_pollfd.fd	 channel->fd;
my_pollfd.events	 POLLIN;
my_pollfd.revents	 0;

do {
	rc	 poll(&my_pollfd, 1, ms_timeout);
} while (rc	= 0);
if (rc < 0) {
	fprintf(stderr, "poll failed\n");
	return 1;
}
ev_cq	 cq;

/* Wait for the completion event */
if (ibv_get_cq_event(channel, &ev_cq, &ev_ctx)) {
	fprintf(stderr, "Failed to get cq_event\n");
	return 1;
}

/* Ack the event */
ibv_ack_cq_events(ev_cq, 1);
```
## 9.2 获取和确认异步事件
### 9.2.1 ibv_get_async_event（差mojo）
**函数原型：** 
```c
int ibv_get_async_event(struct ibv_context *context, struct ibv_async_event *event);
```
**输入参数：** 

* context——设备上下文，来自ibv_opend_device。
* event——用于返回异步事件的指针。详细信息见下文。

**输出参数：** event——指向寻找到的异步事件的指针。

**返回值：** 成功返回0，失败返回-1，并设置errno显示失败原因。

**说明：** 

ibv_get_async_event获取RDMA设备上下文context的下一个异步事件，并通过指针event返回它，它是一个ibv_async_event结构体。 最后必须通过ibv_ack_async_event确认由ibv_get_async_event返回的所有异步事件。为避免竞争，销毁对象（CQ，SRQ或QP）将等待所有相关事件被确认。 这样可以避免应用程序在销毁相应对象之后检索关联事件。

ibv_get_async_event()是一个阻塞函数。 如果多个线程同时调用此函数，则在发生异步事件时，只有一个线程将接收该函数，并且无法预测哪个线程将接收它。

struct ibv_async_event定义如下：
```cpp
struct ibv_async_event {
	union {							/* 结构体成员element的哪一个成员将是有效的，具体取决于结构体成员event_type */
		struct ibv_cq		*cq;		/* 获得事件的CQ，详细信息见ibv_create_cq */
		struct ibv_qp		*qp;		/* 获得事件的QP，详细信息见ibv_create_qp */
		struct ibv_srq		*srq;		/* 获得事件的SRQ，详细信息见ibv_create_srq */
		struct ibv_wq		*wq;		/* 获得事件的WQ，详细信息见ibv_create_wq */
		int					port_num;	/* 获得事件的端口号*/
	} element;
	enum ibv_event_type	event_type;	/* 事件类型 */
};

```
enum ibv_event_type定义如下：

```cpp
enum ibv_event_type {
	IBV_EVENT_CQ_ERR,				//CP事件，CQ错误，CQ溢出
	IBV_EVENT_QP_FATAL,				//QP事件，QP发生错误，并转换为Error状态
	IBV_EVENT_QP_REQ_ERR,			//QP事件，无效的请求本地工作队列错误
	IBV_EVENT_QP_ACCESS_ERR,		//QP事件，本地访问违反错误
	IBV_EVENT_COMM_EST,				//QP事件，QP上的通信已建立
	IBV_EVENT_SQ_DRAINED,			//QP事件，进行中的发送队列中已耗尽未完成消息
	IBV_EVENT_PATH_MIG,				//QP事件，连接已迁移到备用路径
	IBV_EVENT_PATH_MIG_ERR,			//QP事件，连接无法迁移到备用路径
	IBV_EVENT_DEVICE_FATAL,			//CA事件，CA处于FATAL状态
	IBV_EVENT_PORT_ACTIVE,			//端口事件，端口上的链接变为激活
	IBV_EVENT_PORT_ERR,				//端口事件，端口上的链接不可用
	IBV_EVENT_LID_CHANGE,			//端口事件，端口上的LID已更改
	IBV_EVENT_PKEY_CHANGE,			//端口事件，端口上的P_key表已更改
	IBV_EVENT_SM_CHANGE,			//端口事件，端口上的SM已更改
	IBV_EVENT_SRQ_ERR,				//SRQ事件，SRQ出现错误
	IBV_EVENT_SRQ_LIMIT_REACHED,	//SRQ事件，达到SRQ限制
	IBV_EVENT_QP_LAST_WQE_REACHED,	//QP事件，上一个WQE到达与SRQ相关联的QP
	IBV_EVENT_CLIENT_REREGISTER,	//端口事件，SM向端口发送了CLIENT_REREGISTER请求
	IBV_EVENT_GID_CHANGE,			//端口事件，端口上GID表已更改
	IBV_EVENT_WQ_FATAL,				//WQ事件，WQ处于FATAL状态
};  
```

**示例:** 见ibv_ack_async_event。


### 9.2.2 ibv_ack_async_event（差mojo）
**函数原型：** 
```c
void ibv_ack_async_event(struct ibv_async_event *event);
```
**输入参数：** event——异步事件，由ibv_get_async_event返回。

**输出参数：** 无。

**返回值：** 无。

**说明：** 确认异步事件。

**示例:**

下面的代码示例演示了一种在非阻塞模式下处理异步事件的可能方法。 它执行以下步骤：

1. 将异步事件队列工作模式设置为非阻塞
2. 轮询队列，直到发生异步事件
3. 获取异步事件并确认

```cpp
/* change the blocking mode of the async event queue */
flags = fcntl(ctx->async_fd, F_GETFL);
rc = fcntl(ctx->async_fd, F_SETFL, flags | O_NONBLOCK);
if (rc < 0) {
	fprintf(stderr, "Failed to change file descriptor of async event queue\n");
	return 1;
}

/*
 * poll the queue until it has an event and sleep ms_timeout
 * milliseconds between any iteration
 */
my_pollfd.fd      = ctx->async_fd;
my_pollfd.events  = POLLIN;
my_pollfd.revents = 0;

do {
	rc = poll(&my_pollfd, 1, ms_timeout);
} while (rc == 0);
if (rc < 0) {
	fprintf(stderr, "poll failed\n");
	return 1;
}

/* Get the async event */
if (ibv_get_async_event(ctx, &async_event)) {
	fprintf(stderr, "Failed to get async_event\n");
	return 1;
}

/* Ack the event */
ibv_ack_async_event(&async_event);

```


# 10 通知和轮询完成队列
### 10.1  ibv_req_notify_cq（差mojo）
**函数原型：**
```cpp
int ibv_req_notify_cq(struct ibv_cq *cq, int solicited_only)
```
**输入参数：**
* cq——来自ibv_create_cq的struct ibv_cq。
* solicited_only——只有当WR被标为请求时才通知。

**输出参数：** 无。

**返回值：** 成功返回0；失败返回errno来指示失败的原因。

**说明：**
ibv_req_notify_cq在完成队列cq上请求一个完成通知。在向cq添加新的CQ条目（CQE）后，完成事件将添加到与CQ关联的完成通道。 如果该CQ中已经有一个CQE，则不会为此事件生成一个事件。 如果设置了solicited_only标志，则只有设置了solicited标志的WR的CQE才会触发通知。

如果参数solicited_only为零，则为任何新的CQE生成完成事件。 如果solicited_only不为零，则仅为被认为是“请求的”新的CQE生成一个完成事件。如果CQE是设置了请求事件头部位的消息的接收完成，或 状态为不成功，则CQE是请求的。所有其他成功的接收完成，或任何成功的发送完成都不是请求的。

用户应使用ibv_get_cq_event操作接收这个通知。

一个请求仅可用于一个通知。每次调用ibv_req_notify_cq()只会生成一个完成事件。

## 10.2 ibv_poll_cq(差mojo)
**函数原型：**
```cpp
int ibv_poll_cq(struct ibv_cq *cq, int num_entries, struct ibv_wc *wc)
```
**输入参数：**
* cq——来自ibv_create_cq的struct ibv_cq。
* num_entries——要返回的CQE的最大数量。

**输出参数：** wc——CQE数组。

**返回值：** 成功返回数组wc中的CQE数量，失败返回-1。

**说明：**
ibv_poll_cq轮询完成队列cq并返回前num_entries个CQE（当CQ中的CQE少于num_entries时，则返回所有可用的CQE）。 用户应分配一个struct ibv_wc数组，并将其传递给调用的wc。

wc中可容纳的条目数应在num_entries中传递。 ***用户负责释放该内存***。

实际获取的CQEs数量作为返回值给出。

必须定期对CQs进行轮询，以防止超出。 如果发生超限，将关闭CQ并发送异步事件IBV_EVENT_CQ_ERR。

struct ibv_wc定义如下：
```cpp
struct ibv_wc {
	uint64_t			wr_id;				//已完成的WR的iD
	enum ibv_wc_status	status;				//操作的状态, 详细信息见ibv_create_cq_ex
	enum ibv_wc_opcode	opcode;				//在已完成的WR中指定的操作码，详细信息见下文
	uint32_t			vendor_err;			//供应商错误综合码
	uint32_t 			byte_len;			//传输的字节数
	union {
		__be32			imm_data;			//当wc_flags&IBV_WC_WITH_IMM为真，存放网络字节序的立即数据
		uint32_t		invalidated_rkey;	//当wc_flags&IBV_WC_WITH_INV为真，存放无效的本地Rkey
	};
	uint32_t			qp_num;				//已完成的WR的本地QP编号
	uint32_t			src_qp;				//已完成的WR的远程QP编号（仅对UD QP有效）
	enum ibv_wc_flags	wc_flags;			//已完成WR的标准
	uint16_t			pkey_index;			//P_key索引（仅对GSI_QPs有效）
	uint16_t			slid;				//源LID
	uint8_t				sl;					//服务级别
	uint8_t				dlid_path_bits;		//DLID路径位（不能用于多播消息）
};
```

enum ibv_wc_opcode定义如下：

```cpp
enum ibv_wc_opcode {
	IBV_WC_SEND,		//几乎与ibv_wr_opcode对应
	IBV_WC_RDMA_WRITE,
	IBV_WC_RDMA_READ,
	IBV_WC_COMP_SWAP,
	IBV_WC_FETCH_ADD,
	IBV_WC_BIND_MW,
	IBV_WC_LOCAL_INV,
	IBV_WC_TSO,
	/*
	* 设置IBV_WC_RECV的值，以便消费者
	* 可以通过测试 (opcode & IBV_WC_RECV)来测试完成是否是接收完成。
	*/
	IBV_WC_RECV	 1 << 7,
	IBV_WC_RECV_RDMA_WITH_IMM,

	IBV_WC_TM_ADD,
	IBV_WC_TM_DEL,
	IBV_WC_TM_SYNC,
	IBV_WC_TM_RECV,
	IBV_WC_TM_NO_TAG,
	IBV_WC_DRIVER1,		//响应IBV_WR_DRIVER1操作码
 };
```

```cpp
enum ibv_wc_flags {
	IBV_WC_GRH				= 1 << 0,	//存在GRH（仅UD QP有效）
	IBV_WC_WITH_IMM			= 1 << 1,	//立即数据有效
	IBV_WC_IP_CSUM_OK		= 1 << IBV_WC_IP_CSUM_OK_SHIFT,	/*已验证IPv4上的TCP/UDP校验和和IPv4头
																	校验和。 仅当device_attr中的device_cap_flags
																	显示当前QP支持校验和卸载时有效。*/
	IBV_WC_WITH_INV			= 1 << 3,	//无效Rkey数据有效
	IBV_WC_TM_SYNC_REQ		= 1 << 4,	//
	IBV_WC_TM_MATCH			= 1 << 5,	//
	IBV_WC_TM_DATA_VALID	= 1 << 6,	//
};
```
不是所有的wc属性都是有效的。如果完成状态不是IBV_WC_SUCCESS，则仅以下属性有效：wr_id，status，qp_num和vendor_err。

IBV_WC_IP_CSUM_OK_SHIFT定义如下：
```
enum {
	IBV_WC_IP_CSUM_OK_SHIFT	= 2
};
```

**示例：** 见ibv_ack_cq_event

# 11 基于扩展QP的操作
**函数原型：**
```cpp
void	ibv_wr_abort(struct ibv_qp_ex *qp);
int		ibv_wr_complete(struct ibv_qp_ex *qp);
void	ibv_wr_start(struct ibv_qp_ex *qp);

void	ibv_wr_atomic_cmp_swp(struct ibv_qp_ex *qp, uint32_t rkey,
								uint64_t remote_addr, uint64_t compare,uint64_t swap);
void	ibv_wr_atomic_fetch_add(struct ibv_qp_ex *qp, uint32_t rkey,
									uint64_t	remote_addr, uint64_t add);

void	ibv_wr_bind_mw(struct ibv_qp_ex *qp, struct ibv_mw *mw, uint32_t rkey,
							const struct ibv_mw_bind_info *bind_info);
void	ibv_wr_local_inv(struct ibv_qp_ex *qp, uint32_t invalidate_rkey);

void	ibv_wr_rdma_read(struct ibv_qp_ex *qp, uint32_t rkey,
							uint64_t remote_addr);
void	ibv_wr_rdma_write(struct ibv_qp_ex *qp, uint32_t rkey,
							uint64_t remote_addr);
void	ibv_wr_rdma_write_imm(struct ibv_qp_ex *qp, uint32_t rkey,
							uint64_t remote_addr, __be32 imm_data);

void	ibv_wr_send(struct ibv_qp_ex *qp);
void	ibv_wr_send_imm(struct ibv_qp_ex *qp, __be32 imm_data);
void	ibv_wr_send_inv(struct ibv_qp_ex *qp, uint32_t invalidate_rkey);
void	ibv_wr_send_tso(struct ibv_qp_ex *qp, void *hdr, uint16_t hdr_sz,uint16_t mss);
void	ibv_wr_set_inline_data(struct ibv_qp_ex *qp, void *addr, size_t length);
void	ibv_wr_set_inline_data_list(struct ibv_qp_ex *qp, size_t num_buf,
									const struct ibv_data_buf *buf_list);
void	ibv_wr_set_sge(struct ibv_qp_ex *qp, uint32_t lkey, uint64_t addr,
						uint32_t length);
void	ibv_wr_set_sge_list(struct ibv_qp_ex *qp, size_t num_sge,
							const struct ibv_sge *sg_list);

void	ibv_wr_set_ud_addr(struct ibv_qp_ex *qp, struct ibv_ah *ah,
							uint32_t remote_qpn, uint32_t remote_qkey);
void	ibv_wr_set_xrc_srqn(struct ibv_qp_ex *qp, uint32_t remote_srqn);

```


**返回值：**

应用程序应以不会造成失败的方式使用此API。 各个API不会返回失败指示以避免分支。

如果在操作过程中检测到失败，例如由于参数无效，那么ibv_wr_complete()将返回失败，并且整个发布将被中止。

**说明：**

动词 工作请求API（ibv_wr_\*）允许使用函数调用代替基于ibv_post_send()方案的架构，将工作高效地发布到发送队列中。 此方法旨在最大程度地减少发布过程中的CPU分支和锁定。

这些API旨在用于访问ibv_post_send()提供的功能之外的其他功能。

ibv_post_send()批处理的WR和这些API批处理的WR可以相互交错，只要它们不在彼此的关键区域内发布。这些API中的关键区域由ibv_wr_start()和ibv_wr_complete()/ibv_wr_abort()形成。

***用法***

要使用这些API，必须使用ibv_create_qp_ex()创建QP，这个函数允许在comp_mask中设置IBV_QP_INIT_ATTR_SEND_OPS_FLAGS。 send_ops_flags应该设置为将发布到QP的工作请求类型的OR运算。

如果QP不支持所有请求的工作请求类型，则QP创建将失败。

将工作请求发布到QP是在ibv_wr_start()和ibv_wr_complete()/ibv_wr_abort()形成的关键区域内完成的（请参见下面的注意事项）。

每个工作请求都是通过调用WR构建函数（请参见下面的表WR Builder）来创建的，以开始创建工作请求，然后再调用下文所述的允许/需要的设置函数。

可以多次调用WR构建器和设置器组合，以有效地在单个关键区域内发布多个工作请求。

每个WR构建函数将使用struct ibv_qp_ex的wr_id成员来设置要在完成时返回的值。 一些操作还将使用wr_flags成员影响操作（请参见下面的标志）。 这些值应在调用WR构建函数之前设置。

例如，一个简单的发送可以形成，如下：
```cpp
qpx->wr_id = 1;
ibv_wr_send(qpx);
ibv_wr_set_sge(qpx, lkey, &data, sizeof(data));

```
“工作请求”部分详细描述了各种WR构建函数和设置函数。

通过调用ibv_wr_complete()或ibv_wr_abort()完成发布工作。 在ibv_wr_complete()返回成功之前，不会对队列执行任何工作。 ibv_wr_abort()将丢弃自ibv_wr_start()以来准备的所有工作。

***工作请求***

许多操作与ibv_post_send()可用的操作码匹配。 每个操作都有一个WR构建函数，一个允许的设置函数列表，一个标志位，这个标志位用于请求struct ibv_qp_init_attr_ex中send_ops_flags的操作（请参见下面的示例）。

|操作|WR构建器|支持的QP类型|设置器
|:--|:--|:--|:--|
|ATOMIC_CMP_AND_SWP|ibv_wr_atomic_cmp_swp()|RC, XRC_SEND|DATA, QP|
|ATOMIC_FETCH_AND_ADD|ibv_wr_atomic_fetch_add()|RC, XRC_SEND|DATA, QP|
|BIND_MW|ibv_wr_bind_mw() |UC, RC, XRC_SEND|NONE|
|LOCAL_INV|ibv_wr_local_inv() |UC, RC, XRC_SEND | NONE|
|RDMA_READ |ibv_wr_rdma_read() |RC, XRC_SEND | DATA, QP|
|RDMA_WRITE| ibv_wr_rdma_write() |UC, RC, XRC_SEND| DATA, QP|
|RDMA_WRITE_WITH_IMM |ibv_wr_rdma_write_imm()|UC, RC, XRC_SEND |DATA, QP|
|SEND| ibv_wr_send() | UD, UC, RC, |XRC_SEND, RAW_PACKET |DATA, QP|
|SEND_WITH_IMM|ibv_wr_send_imm()|UD, UC, RC, SRC SEND|DATA, QP|
|SEND_WITH_INV |ibv_wr_send_inv() | UC, RC, |XRC_SEND| DATA, QP|
|TSO |ibv_wr_send_tso()| UD,  RAW_PACKET|DATA, QP|


* **原子操作**：原子操作仅是原子操作，只要所有对内存的写操作仅通过同一RDMA硬件进行即可。 由CPU或系统中其他RDMA硬件执行的写操作的不是原子的。
  * `ibv_wr_atomic_cmp_swp()`
如果rkey和remote_addr指定的远程64位内存位置等于compare，则将其设置为swap。
  * `ibv_wr_atomic_fetch_add()`
将add添加到rkey和remote_addr的指定的64位内存位置。
* **内存窗口**：内存窗口类2 操作（有关ibv_alloc_mw的信息，请参见手册页）。
  * `ibv_wr_bind_mw()`
绑定由mw指定的MW类型2，设置一个新的rkey并通过bind_info设置其属性。
  * `ibv_wr_local_inv()`
使与rkey关联的MW类型2无效。
* **RDMA**
  * `ibv_wr_rdma_read()`
从rkey和remote_addr指定的远程内存位置读取。读取的字节数以及存储数据的本地位置由此调用后设置的DATA缓冲区确定。
  * `ibv_wr_rdma_write()，ibv_wr_rdma_write_imm()`
写入rkey和remote_addr指定的远程内存位置。写入的字节数以及写入数据的本地位置由此调用后设置的DATA缓冲区确定。\_imm版本使远程端获得包含32位立即数据的IBV_WC_RECV_RDMA_WITH_IMM，
* **消息发送**
  * `ibv_wr_send()，ibv_wr_send_imm()`
发送消息。发送的字节数以及获取数据的本地位置由此调用后设置的DATA缓冲区确定。\_imm版本使远程端获取包含32位立即数据的IBV_WC_RECV_RDMA_WITH_IMM。
  * `ibv_wr_send_inv()`
数据传输与`ibv_wr_send()`相同，但是在传递完成之前，远程端将使invalidate_rkey指定的MR无效。
  * `ibv_wr_send_tso()`
使用TCP Segmentation Offload产生多个SEND消息。 SGE指向TCP流缓冲区，该缓冲区将分成MSS大小的SEND。 hdr包含直到（包括）TCP头的整个网络头，并在每个分段之前添加前缀。
* **QP特定设置器**： 某些QP类型要求每个发布都必须有额外的设置器，这些设置器对于上表中列出的QP设置群器的任何操作都是强制的。
  * UD QP
 必须调用`ibv_wr_set_ud_addr()`来设置工作的目标地址。
  * XRC_SEND QP
必须调用`ibv_wr_set_xrc_srqn()`来设置SRQN字段。
* **数据传输设置器**：对于需要传输数据的工作，应在WR构建器之后调用以下设置器之一：
  * `ibv_wr_set_sge()`
传输数据 到/自 由lkey，addr和length给定的单个缓冲区。这等效于具有单个元素的`ibv_wr_set_sge_list()`。
  * `ibv_wr_set_sge_list()`
传输数据 到/自 逻辑上串联在一起的缓冲区列表。每个缓冲区由struct ibv_sge数组中的一个元素指定。内联设置器将在设置器中复制发送的数据，并允许调用方立即重新使用缓冲区。此行为与IBV_SEND_INLINE标志相同。通常，此副本以优化SEND延迟的方式完成，并且适合于小消息。驱动程序将限制在单个操作中可以支持的数据量。在strcut ibv_qp_init_attr的max_inline_data成员中请求此限制。仅对SEND和RDMA_WRITE有效。
  * `ibv_wr_set_inline_data()`
从addr和length给定的单个缓冲区中复制发送数据。这等效于具有单个元素的`ibv_wr_set_inline_data_list()`。
  * `ibv_wr_set_inline_data_list()`
从逻辑上串联在一起的缓冲区列表中复制发送数据。每个缓冲区由struct ibv_inl_data数组中的一个元素指定。
* **flags**： 可以在wr_flags中指定标志的位掩码，以控制工作请求的行为。定义见ibv_post_send
  * `IBV_SEND_FENCE`
在先前的工作完成之前，不要启动此工作请求。
  * `IBV_SEND_IP_CSUM`
卸载IPv4和TCP/UDP校验和计算
  * `IBV_SEND_SIGNALED`
将在操作的完成队列中生成一个完成。
  `IBV_SEND_SOLICTED`
设置RDMA数据包中的请求位。 这通知另一方在接收到RDMA操作后生成完成事件。

***并发性***

驱动程序将提供锁以确保ibv_wr_start()和ibv_wr_complete()/abort()形成每个QP关键部分，其他线程无法进入该部分。

如果在QP创建期间提供了ibv_td，则不会执行锁定，并且由调用者决定一次只能在关键区域内有一个线程。

**示例：**
```cpp
/* create RC QP type and specify the required send opcodes */
qp_init_attr_ex.qp_type = IBV_QPT_RC;
qp_init_attr_ex.comp_mask |= IBV_QP_INIT_ATTR_SEND_OPS_FLAGS;
qp_init_attr_ex.send_ops_flags |= IBV_QP_EX_WITH_RDMA_WRITE;
qp_init_attr_ex.send_ops_flags |= IBV_QP_EX_WITH_RDMA_WRITE_WITH_IMM;

ibv_qp *qp = ibv_create_qp_ex(ctx, qp_init_attr_ex);
ibv_qp_ex *qpx = ibv_qp_to_qp_ex(qp);

ibv_wr_start(qpx);

/* create 1st WRITE WR entry */
qpx->wr_id = my_wr_id_1;
ibv_wr_rdma_write(qpx, rkey, remote_addr_1);
ibv_wr_set_sge(qpx, lkey, local_addr_1, length_1);

/* create 2nd WRITE_WITH_IMM WR entry */
qpx->wr_id = my_wr_id_2;
qpx->send_flags = IBV_SEND_SIGNALED;
ibv_wr_rdma_write_imm(qpx, rkey, remote_addr_2, htonl(0x1234));
ibv_set_wr_sge(qpx, lkey, local_addr_2, length_2);

/* Begin processing WRs */
ret = ibv_wr_complete(qpx);
```

# 12 基于流的操作

# 13 基于设备内存的操作
## 13.1 拷贝数据
### 13.1.1 ibv_memcpy_from_dm
**函数原型：** 
```cpp
int ibv_memcpy_from_dm(void *host_addr, struct ibv_dm *dm,uint64_t dm_offset, size_t length)
```
**输入参数：** 

* dm——设备内存。来及ibv_alloc_dm。
* dm_offset——要访问的设备内存的自起始地址的偏移量。
* length——要复制的长度，以字节为单位。

**输出参数：** host_addr——主机内存起始地址。

**返回值：** 成功返回0，失败返回errno显示失败原因。

**描述：** 读取设备内存，并写入主机内存中。
### 13.1.2 ibv_memcpy_to_dm
**函数原型：** 
```cpp
int ibv_memcpy_to_dm(struct ibv_dm *dm, uint64_t dm_offset,void *host_addr, size_t length)
```
**输入参数：** 
* host_addr——主机内存地址
* length——要复制的长度，以字节为单位。
* dm_offset——要访问的设备内存的自起始地址的偏移量。

**输出参数：** dm——设备内存。来及ibv_alloc_dm。

**返回值：** 成功返回0，失败返回errno显示失败原因。

**描述：** 读取主机内存，并写入设备内存中。

## 13.2 注册设备内存
### 13.2.1 ibv_reg_dm_mr
**函数原型：** 
```cpp
struct ibv_mr *ibv_reg_dm_mr(struct ibv_pd *pd, struct ibv_dm *dm, 
								uint64_t dm_offset, size_t length, uint32_t access)
```
**输入参数：** 
* pd——要关联的保护域，来自ibv_alloc_pd。
* dm——设备内存。来及ibv_alloc_dm。
* dm_offset——要注册的已分配设备内存的自起始地址的偏移量。
* length——要注册设备内存的大小。
* access——MR访问标志，使用enum ibv_access_flags。详细信息见ibv_reg_mr。对于这种类型的注册，必须设置IBV_ACCESS_ZERO_BASED标志。

**输出参数：** 无。

**返回值：** 成功返回指向struct ibv_mr的指针，失败返回NULL。

**描述：** 

用户可以在发布接收或发送工作请求时将分配的设备内存注册为内存区域，并在sge中使用lkey/rkey。 这种类型的MR被定义为从零开始的，因此对它的任何引用（特别是在sge中）都是从区域的开头偏移一些字节来完成的。
# 15 枚举值转字符串
## 15.1 ibv_node_type_str

**函数原型：**
```cpp
const char *ibv_node_type_str (enum ibv_node_type node_type)
```
**输入参数：** node_type——enum ibv_node_type枚举值，可以是HCA, Switch,Router, RNIC or Unknown。枚举的详细信息见ibv_get_device_list。


**输出参数：** 无。

**返回值：** 成功说明枚举值node_type的常量字符串。如果node_type不是有效的节点类型值，则失败并返回“unknown”。

**说明：** ibv_node_type_str返回一个说明节点类型枚举值node_type的字符串。 此值可以是 InfiniBand HCA, Switch, Router,  an RDMA enabled NIC 或 unknown。

**示例：** 参见ibv_query_port。

## 15.2 ibv_port_state_str
**函数原型：**
```cpp
const char *ibv_port_state_str (enum ibv_port_state port_state)
```
**输入参数：** port_state——请求说明的逻辑端口状态枚举值。枚举详细信息见ibv_query_port。

**输出参数：** 无。

**返回值：** 说明枚举值 port_state的常量字符串。

**说明：**

成功返回一个说明逻辑端口状枚举值的字符串。**说明：** 成功返回一个说明逻辑端口状枚举值的字符串，则失败返回“unknown”。

enum ibv_port_state 定义参见ibv_query_port。

**示例：**

```cpp
const char *descr;
descr = ibv_port_state_str(IBV_PORT_ACTIVE);
printf("The description of the enumerated value %d is %s\n", IBV_PORT_ACTIVE,descr);
```
## 15.3 ibv_wc_status_str
  **函数原型：**
```cpp
const char *ibv_wc_status_str(enum ibv_wc_status status)
```
**输入参数：** status——wc状态，详细信息见ibv_create_cq_ex。

**输出参数：** 无。

**返回值：** 成功返回说明WC状态的字符串。失败返回NULL。

**说明：** 返回说明WC状态的字符串。

## 15.4 ibv_event_type_str
  **函数原型：**
```cpp
const char *ibv_event_type_str(enum ibv_event_type event)
```
**输入参数：** event——事件类型，详细信息见ibv_get_async_event。

**输出参数：** 无。

**返回值：** 成功返回说明event类型的字符串。失败返回NULL。

**说明：** 返回说明event状态的字符串。