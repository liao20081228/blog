---
title: RDMA编程手册 第5章 librdmacm库
tags: RDMA
---

------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>[《RDMA Aware Networks Programming User Manual v1.7》](https://www.mellanox.com/related-docs/prod_software/RDMA_Aware_Programming_user_manual.pdf)、[《Dotan Barak的RDMAmojo博客》](http://www.rdmamojo.com/)、《man手册 v24》，我所做的工作就是更新、整理一部分内容，删除了一部分废弃的API，添加了一部分新的API。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

# 1 librdmacm介绍
## 1.1 概述

RDMA_CM用于在RDMA传输层建立链接。

RDMA CM是一个通信管理器，用于设置可靠的有连接的和不可靠数据报的数据传输。它提供用于建立连接的RDMA传输层中性接口（相同代码可以用于InfiniBand和iWARP）。这些API的慨念基于套接字，但适用于基于队列对（QP）的语义：通信必须通过特定的RDMA设备进行，数据传输基于消息。

RDMA CM可以控制一个RDMA API的QP和通信管理（连接的建立或拆除）部分，或者仅控制通信管理部分。它与libibverbs库定义的动词API结合使用，当然RDMA CM原本也封装了一些libibverbs的动词接口，叫做RDMA动词。 libibverbs库提供了发送和接收数据所需的底层接口。

RDMA CM可以异步或同步运行。用户通过在特定调用中使用rdma_cm事件通道参数来控制操作模式。如果提供了事件通道，则rdma_cm标识符将在该通道上报告其事件数据（例如，连接结果）。如果未提供通道，则所选rdma_cm标识符的所有rdma_cm操作都将阻塞，直到完成。

## 1.2 RDMA动词
rdma_cm支持libibverbs库提供的所有动词。 但是，它也为一些更常用的动词提供了包装函数。 完整的抽象动词调用集：
|动词|描述|
|:--|:--|
|rdma_reg_msgs| 注册一个用于发送和接收数据的缓冲区数组|
|rdma_reg_read|注册一个用于RDMA读取操作的缓冲区|
|rdma_reg_write|注册一个用于RDMA写操作的缓冲区|
|rdma_dereg_mr|注销一个内存区域|
|rdma_post_recv |发布一个缓冲区以接收消息|
|rdma_post_send|发布一个缓冲区以发送消息|
|rdma_post_read |发布一个RDMA读以将数据读入缓冲区|
| rdma_post_write| 发布一个RDMA写以从缓冲区发送数据|
|rdma_post_recvv |发布一个缓冲区向量以接收消息|
|rdma_post_sendv |发布一个缓冲区向量以发送消息|
|rdma_post_readv| 发布一个缓冲区向量以接收RDMA read|
|rdma_post_writev| 发布一个缓冲区向量以发送RDMA write|
|rdma_post_ud_send |发布一个缓冲区以在UD QP上发送消息|
|rdma_get_send_comp| 获取一个send或RDMA操作的完成状态
|rdma_get_recv_comp| 获取关于已完成接收请求的信息|


## 1.3 客户端操作

本节概述了通信的主动端（客户端）的基本操作。 该流程假定异步操作，并显示了低级调用详细信息。 对于同步操作，将消除对`rdma_create_event_channel`，`rdma_get_cm_event`，`rdma_ack_cm_event`和`rdma_destroy_event_channel`的调用。 抽象调用（例如`rdma_create_ep`）将这些调用中的几个封装在单个API下。 用户也可以参考示例应用程序以获得代码示例。 一般的连接流程为：

|序号|API|描述|
|:--|:--|:--|
|1|rdma_getaddrinfo|检索目的地的地址信息|
|2|rdma_create_event_channel|创建接收事件的通道|
|3|rdma_create_id|分配一个rdma_cm_id，类似于套接字|
|4| rdma_resolve_addr|获取本地RDMA设备以到达远程地址
|5|rdma_get_cm_event|等待RDMA_CM_EVENT_ADDR_RESOLVED事件
|6|rdma_ack_cm_event|确认事件
|7|rdma_create_qp|为通信分配QP|
|8|rdma_resolve_route|确定到远程地址的路由|
|9|rdma_get_cm_event|等待RDMA_CM_EVENT_ROUTE_RESOLVED事件|
|10|rdma_ack_cm_event|确认事件|
|11|rdma_connect|连接到远程服务器|
|12|rdma_get_cm_event|等待RDMA_CM_EVENT_ESTABLISHED事件|
|13|rdma_ack_cm_event|确认事件|
|14|RDMA动词|**通过连接执行数据传输**|
|15|rdma_disconnect|断开连接|
|16|rdma_get_cm_event|等待RDMA_CM_EVENT_DISCONNECTED事件|
|17|rdma_ack_cm_event|确认事件|
|18|rdma_destroy_qp|销毁QP|
|19|rdma_destroy_id|释放rdma_cm_id|
|20|rdma_destroy_event_channel| 释放事件通道|

几乎相同的过程用于在节点之间建立不可靠的数据报（UD）通信。 QP之间没有实际的连接，因此不需要断开连接。

尽管此示例显示了客户端启动断开连接，但连接的任一端都可以启动断开连接。

## 1.4 服务端操作

本节概述了被动端或服务器端通信的基本操作。一般的连接流程为：
|序号|API|描述|
|:--|:--|:--|
|1|rdma_create_event_channel|创建接收事件的通道|
|2|rdma_create_id|分配一个rdma_cm_id，类似于套接字|
|3|rdma_bind_addr|设置要监听的本地端口号
|4|rdma_listen|开始监听连接请求|
|5|rdma_get_cm_event|等待新的rdma_cm_id的<br /> RDMA_CM_EVENT_CONNECT_REQUEST事件|
|6|rdma_create_qp|为新的rdma_cm_id上的通信分配QP|
|7|rdma_accept|接受连接请求|
|8|rdma_ack_cm_event|确认事件|
|9|rdma_get_cm_event|等待RDMA_CM_EVENT_ESTABLISHED事件|
|10|rdma_ack_cm_event|确认事件|
|11|RDMA动词|通过连接执行数据传输|
|12|rdma_get_cm_event|等待RDMA_CM_EVENT_DISCONNECTED事件|
|13|rdma_ack_cm_event|确认事件|
|14|rdma_disconnect|断开连接|
|15|rdma_destroy_qp|销毁QP|
|16|rdma_destroy_id|释放连接的rdma_cm_id|
|17|rdma_destroy_id|释放监听的rdma_cm_id
|18|rdma_destroy_event_channel|释放事件通道|

## 1.5 返回值

大多数的librdmacm函数返回0表示成功，返回-1表示失败。如果函数是异步操作，则返回值0表示该操作已成功启动，但该操作仍可能错误完成，用户应检查相关事件的状态。如果返回值为-1，则errno将包含有关失败原因的其他信息。有关更多详细信息，请参见errno。

在与ENOMEM，ENODEV，ENODATA，EINVAL和EADDRNOTAVAIL相关的某些情况下，该库的早先版本将返回-errno而不是设置errno。想要检查这些错误代码并与以前的库版本兼容的应用程序必须将errno手动设置为返回码的负数（如果它<-1）。


## 1.6 库API
库中的函数应该声明为函数，有些函数可以声明为宏。
为了使用librdmacm的连接管理函数，源代码中必须包含头文件rdma_cma.h:

```cpp
#include <rdma/rdma_cma.h>
```
为了使用librdmacm的动词函数，源代码中必须包含头文件rdma_verbs.h:

```cpp
#include <rdma/rdma_verbs.h>
```

# 2 RDMA连接管理API
## 2.1 事件通道的创建和销毁
### 2.1.1 rdma_create_event_channel

**原型**：

``` cpp
struct rdma_event_channel * rdma_create_event_channel (void)
```
**输入参数**：无。

**输出参数**：无。

**返回值**：成功返回指向创建的事件通道的指针，失败返回NULL并设置errno以指示失败原因。

**描述**：

打开一个事件通道，用于用于报告通信事件。 异步事件通过事件通道报告给用户。

事件通道用于在struct rdma_cm_id上定向所有事件。 对于许多客户端来说，单个事件通道可能就足够了，但是，在管理大量连接或cm_id时，用户可能会发现将不同cm_id的事件定向到不同事件通道进行处理很有用。

所有创建的事件通道都必须通过调用rdma_destroy_event_channel进行销毁。 用户应调用rdma_get_cm_event检索事件通道上的事件。

每个事件通道都映射到一个文件描述符。 可以像其他任何fd一样使用和操作关联的文件描述符，以更改其行为。 用户可以将fd设为非阻塞，poll或sellect 该fd，等。

struct channel定义如下：
```ceylon
struct rdma_event_channel 
{
	int fd; //文件描述符
};  
```

### 2.1.2 rdma_destroy_event_channel

**原型**：

``` cpp
void rdma_destroy_event_channel (struct rdma_event_channel *channel)
```
**输入参数**：channel——要摧毁的事件通道。结构体详细信息见rdma_create_event_channel。

**输出参数**：无。

**返回值**：无。

**描述**：关闭事件通道。 释放与事件通道关联的所有资源，并关闭关联的文件描述符。

**注意事项**：与事件通道关联的所有rdma_cm_id都必须被销毁，并且所有返回的事件必须在调用此函数之前被确认。


## 2.2 ID的创建、销毁、迁移、设置

### 2.2.1 rdma_create_id

**原型**：

``` cpp
int rdma_create_id(struct rdma_event_channel *channel, struct rdma_cm_id **id, 
					void *context, enum rdma_port_space ps)
```

**输入参数**：

* channel——与分配的rdma_cm_id相关联的事件的报告通道。可以为NULL。结构体详细信息见rdma_create_event_channel。
* ps——RDMA端口空间。 枚举详细信息见下文。
* context——用户指定的与rdma_cm_id相关联的上下文。

**输出参数**： id——将在其中返回分配的通信标识符的引用。

**返回值**：成功时为0，失败返回-1并设置errno以指示失败的原因。

**描述**：

创建用于跟踪通信信息的标识符。

对于RDMA通信，rdma_cm_id在概念上等效于套接字。区别在于RDMA通信需要在通信发生之前显式绑定到指定的RDMA设备，并且大多数操作天生就是异步的。通过关联的事件通道报告rdma_cm_id上的异步通信事件。 如果channel参数为NULL，则rdma_cm_id将被置于同步操作中。 在同步操作时，导致事件的调用将阻塞，直到操作完成。该事件将通过rdma_cm_id结构体返回给用户， 并且在另一个rdma_cm调用发生前，可以访问该事件。

用户必须调用rdma_destroy_id来释放rdma_destroy_id。

struct rdma_cm_id定义如下：
```c
struct rdma_cm_id
{
	struct ibv_context			*verbs;				//设备上下文，详细信息见ibv_open_device
	struct rdma_event_channel	*channel;			//事件通道 ,详细信息见rdma_create_event_channel	
	void						*context;			//用户定义上下文
	struct ibv_qp				*qp;				//QP，详细信息见ibv_create_qp，ibv_modify_qp
	struct rdma_route			route;				//路由，详细信息见下文
	enum rdma_port_space		ps;					//端口空间，枚举值之一，详细信息见下文。
	uint8_t						port_num;			//端口号
	struct rdma_cm_event		*event;				//事件，详细信息见rdma_get_cm_event
	struct ibv_comp_channel		*send_cq_channel;	//发送完成通道，详细信息见ibv_create_comp_channel
	struct ibv_cq				*send_cq;			//发送完成队列，详细信息见ibv_create_cq
	struct ibv_comp_channel		*recv_cq_channel;	//接收完成通道，详细信息见ibv_create_comp_channel
	struct ibv_cq				*recv_cq;			//接收完成队列，详细信息见ibv_create_cq
	struct ibv_srq				*srq;				//共享接收队列，详细信息见ibv_create_srq
	struct ibv_pd				*pd;				//保护域，详细信息见ibv_alloc_pd
	enum ibv_qp_type			qp_type;			//QP类型，枚举值之一，详细信息见ibv_create_qp
};
```

enum rdma_port_space定义如下：
```cpp
enum rdma_port_space 
{
	RDMA_PS_IPOIB	=	0x0002,
	RDMA_PS_TCP		=	0x0106,	/*提供可靠的，有连接的QP通信。
								不像TCP，RDMA端口空间提供基于消息的通信，而不是基于流*/
	RDMA_PS_UDP		=	0x0111,	//提供不可靠的无连接QP通信， 支持数据报和多播通信
	RDMA_PS_IB		=	0x013F,	//提供任何IB服务，提供任何IB服务（UD，UC，RC，XRC等）
};
```

struct route定义如下：
```cpp
struct rdma_route 
{
	struct rdma_addr		addr;
	struct ibv_sa_path_rec	*path_rec;
	int						num_paths;
}; 

struct ibv_sa_path_rec 
{
	/* reserved */
	/* reserved */
	union ibv_gid	dgid;						//目标gid，详情见ibv_query_gid
	union ibv_gid	sgid;						//源gid
	__be16			dlid;						//目标lid
	__be16			slid;						//源lid
	int				raw_traffic;
	/* reserved */
	__be32			flow_label;					//流标签
	uint8_t			hop_limit;					//跳数限制
	uint8_t			traffic_class;				//流量级别
	int				reversible;
	uint8_t			numb_path;
	__be16			pkey;						//分区密钥
	/* reserved */
	uint8_t			sl;							//服务级别
	uint8_t			mtu_selector;				//mtu选择子
	uint8_t			mtu;						//mtu
	uint8_t			rate_selector;				//速率选择子
	uint8_t			rate;						//速率
	uint8_t			packet_life_time_selector;	//分组生命周期选择子
	uint8_t			packet_life_time;			//分组生命周期
	uint8_t			preference;					//偏好
}; 

struct rdma_addr 
{
	union 
	{
		struct sockaddr			src_addr;	//源地址
		struct sockaddr_in		src_sin;	//源地址，ipv4
		struct sockaddr_in6		src_sin6;	//源地址，ipv6
		struct sockaddr_storage	src_storage;
	};
	union 
	{
		struct sockaddr			dst_addr;	//源地址
		struct sockaddr_in		dst_sin;	//源地址，ipv4
		struct sockaddr_in6		dst_sin6;	//源地址，ipv6
		struct sockaddr_storage	dst_storage;
	};
	union 
	{
		struct rdma_ib_addr		ibaddr;
	} addr;
};

struct rdma_ib_addr 
{
	union ibv_gid	sgid;	//源gid
	union ibv_gid	dgid;	//目标gid
	__be16			pkey;	//分区密钥
};

typedef uint16_t sa_family_t;
struct sockaddr 
{
	sa_family_t		sa_family;		//地址族
	char			sa_data[14];	//14字节的协议地址，包含该 socket 的 IP 地址和端口号
};

typedef uint16_t in_port_t;
struct sockaddr_in 
{ 
	sa_family_t		sin_family;		/*地址族*/ 
	in_port_t		sin_port;		/*端口号，网络字节序*/ 
	struct in_addr	sin_addr;		/*IP 地址，网络字节序*/ 
	unsigned char	sin_zero[8];	/*填充0 以保持与 struct sockaddr 同样大小*/ 
}; 

typedef uint32_t in_addr_t;
struct in_addr 
{ 
	in_addr_t		s_addr;			/*32 位IPv4 地址，网络字节序 */ 
}; 


struct sockaddr_in6
{
	sa_family_t		sin6_family;  	/* 家族协议 */
	in_port_t		sin6_port;		/* 端口号 */
	uint32_t		sin6_flowinfo;	/* IPv6 flow information ipv6中的流标签字段 */
	struct in6_addr	sin6_addr;		/* ipv6的地址信息 */
	uint32_t		sin6_scope_id;	/* ipv6的接口范围 */
};


struct in6_addr
{
	union
	{
		uint8_t __u6_addr8[16];   // 128 bit
#if defined __USE_MISC || defined __USE_GNU
		uint16_t __u6_addr16[8];  // 64 bit
		uint32_t __u6_addr32[4];  // 32 bit
#endif
	} __in6_u;
	#define s6_addr				__in6_u.__u6_addr8
#if defined __USE_MISC || defined __USE_GNU
	# define s6_addr16			__in6_u.__u6_addr16
	# define s6_addr32			__in6_u.__u6_addr32
#endif
};
```

### 2.2.2 rdma_destroy_id

**原型**：
``` cpp
int rdma_destroy_id (struct rdma_cm_id *id)
```
**输入参数**：id——要销毁的通信标识符。结构体详细信息见rdma_create_id。

**输出参数**：无。

**返回值**：成功时为0，失败返回-1并设置errno以指示失败的原因。

**描述**：

销毁指定的rdma_cm_id并取消任何未完成的异步操作。

在调用这个函数之前，用户必须释放rdma_cm_id关联的任何QP，并确认相关事件。

### 2.2.3 rdma_migrate_id
**原型**：
``` cpp
int rdma_migrate_id(struct rdma_cm_id *id, struct rdma_event_channel *channel)
```
**输入参数**：

* id——要迁移的已存在的RDMA通信标识符。结构体详细信息见rdma_create_id。
* channel——与rdma_cm_id关联的事件的报告通道。可以为NULL。结构体详细信息见rdma_create_event_channel。

**输出参数**：无。

**返回值**：成功时为0，失败返回-1并设置errno以指示失败的原因。

**描述**：

rdma_migrate_id将通信标识符迁移到其他事件通道，并将与rdma_cm_id关联的所有挂起事件移至新通道。

用户在通道之间迁移时，不应轮询rdma_cm_id当前事件通道上的事件，也不应在rdma_cm_id上调用其他函数。 当前事件通道上有任何未确认的事件时，此调用将被阻塞。
 
如果channel参数为NULL，则指定的rdma_cm_id将进入同步操作模式。 此id上的所有调用都将阻塞，直到操作完成。

### 2.2.4 rdma_set_option
**原型**：
``` cpp
int rdma_set_option(struct rdma_cm_id *id, int level, int optname, void *optval, size_t optlen)
```
**输入参数**：

* id——RDMA通信标识符。结构体详细信息见rdma_create_id。
* level——要设置的协议级别。取值为枚举值之一。详细信息见下文。
* optname——要设置的选项名，与level有关，取值为枚举值之一。详细信息见下文。
* optval——指向选项的数据的指针。数据取决于level和optname。
* optlen——选项数据(optval)缓冲区的大小。

**输出参数**：无。

**返回值**：成功返回0，失败返回-1，并设置errno以指示失败的原因。

**描述**：

rdma_set_option设置rdma_cm_id的通信选项。 这个调用用于重写默认的系统设置。选项级别level和详细信息可以在相关头文件的枚举中找到。

level可取值定义如下：
```cpp
enum
{
	RDMA_OPTION_ID	  = 0,		//ID级别
	RDMA_OPTION_IB	  = 1		//IB级别
};
```

optname的可取值定义如下：
```cpp
enum
{
	RDMA_OPTION_ID_TOS			= 0,	/* 指定连接提供的服务质量。 optlen=sizeof（是uint8_t）*/
	RDMA_OPTION_ID_REUSEADDR	= 1,	/* 将rdma_cm_id绑定到可重用地址。 这将允许其他用户绑定到相同的地址。
										optlen=sizeof（int） */
	RDMA_OPTION_ID_AFONLY		= 2,	/*设置IPV6_V6ONLY套接字。 optlen=sizeof（int） */
	RDMA_OPTION_ID_ACK_TIMEOUT	= 3		/* 设置QP ACK超时。 根据公式4.096 * 2^ack_timeout 微秒计算。 */ 
};

enum
{
	RDMA_OPTION_IB_PATH			= 1		/*设置IB路径记录数据。optlen=sizeof（struct ibv_path_data []） */
};
```

## 2.3 QP的创建和销毁

### 2.3.1 rdma_create_qp
**原型**：
``` cpp
int rdma_create_qp (struct rdma_cm_id *id, struct ibv_pd *pd, struct ibv_qp_init_attr *qp_init_attr)
```
**输入参数**：

* id——RDMA标识符。结构体详细信息见rdma_create_id。
* pd——QP的保护域（可选的）。结构体详细信息见ibv_alloc_pd。
* qp_init_attr——QP初始属性。结构体详细信息见ibv_create_qp。

**输出参数**：qp_init_attr——通过此结构体返回创建的QP的实际功能和属性。

**返回值**：成功时为0，失败返回-1并设置errno以指示失败的原因。

**描述**：

分配与指定的rdma_cm_id关联的QP，并将其转换以进行发送和接收。

在调用此函数之前，必须将rdma_cm_id绑定到本地RDMA设备，并且保护域必须用于同一设备。 分配给rdma_cm_id的QP的状态将被librdmacm在其状态间自动转换。分配后，QP将准备好处理发布接收请求。如果QP不是面向连接，它将准备发布发送请求。

如果未提供保护域（pd参数为NULL），则将使用默认保护域创建rdma_cm_id。每个RDMA设备分配一个默认的保护域。

初始QP属性由qp_init_attr参数指定。 ibv_qp_init_attr中的send_cq和recv_cq字段是可选的。如果未指定发送或接收完成队列，则rdma_cm将为QP分配CQ以及相应的完成通道。 rdma_cm创建的完成通道和CQ数据通过rdma_cm_id结构体向用户公开。

创建的QP的实际功能和属性将通过qp_init_attr参数返回给用户。 rdma_cm_id只能与单个QP关联。

### 2.3.2 rdma_create_qp_ex（未完成）

**原型**：
``` cpp
int rdma_create_qp_ex(struct rdma_cm_id *id, struct ibv_qp_init_attr_ex *qp_init_attr);
```
**输入参数**：

* id——RDMA标识符。结构体详细信息见rdma_create_id。
* qp_init_attr——QP初始属性，带有扩展属性。

**输出参数**：qp_init_attr——通过此结构体返回创建的QP的实际功能和属性。

**返回值**：成功时为0，失败返回-1并设置errno以指示失败的原因。

**描述**：

分配与指定的rdma_cm_id关联的QP，并将其转换以进行发送和接收。

在调用此函数之前，必须将rdma_cm_id绑定到本地RDMA设备，并且保护域必须用于同一设备。 分配给rdma_cm_id的QP的状态将被librdmacm在其状态间自动转换。分配后，QP将准备好处理发布接收请求。如果QP不是面向连接，它将准备发布发送请求。

如果扩展属性中未提供保护域，则将使用默认保护域创建rdma_cm_id。每个RDMA设备分配一个默认的保护域。

初始QP属性由qp_init_attr参数指定。 ibv_qp_init_attr中的send_cq和recv_cq字段是可选的。如果未指定发送或接收完成队列，则rdma_cm将为QP分配CQ以及相应的完成通道。 rdma_cm创建的完成通道和CQ数据通过rdma_cm_id结构体向用户公开。

创建的QP的实际功能和属性将通过qp_init_attr参数返回给用户。 rdma_cm_id只能与单个QP关联。

### 2.3.3 rdma_destroy_qp
**原型**：
``` cpp
void rdma_destroy_qp (struct rdma_cm_id *id)
```
**输入参数**：id——RDMA标识符。结构体详细信息见rdma_create_id。

**输出参数**：无。

**返回值**：无。

**描述**：

rdma_destroy_qp销毁在rdma_cm_id上分配的QP。

用户必须销毁与rdma_cm_id关联的任何QP，然后再销毁ID。

 struct rdma_cm_id详细信息见rdma_create_id。
 
### 2.3.4 rdma_init_qp_attr

**原型**：
``` cpp
int rdma_init_qp_attr(struct rdma_cm_id *id, struct ibv_qp_attr *qp_attr, int *qp_attr_mask);

```
**输入参数**：

* id——RDMA标识符。结构体详细信息见rdma_create_id。
* qp_attr——指向包含响应信息的qp属性结构体的指针。详细信息见ibv_modify_qp。
* qp_attr_mask——指向包含响应信息的qp属性掩码的指针。详细信息见ibv_modify_qp。

**输出参数**：qp_attr——返回初始化的qp属性。

**返回值**：成功时为0，失败返回-1并设置errno以指示失败的原因。

**描述**：

rdma_init_qp_attr（）返回rdma_cm_id的qp属性。

有关qp属性和qp属性掩码的信息是通过qp_attr和qp_attr_mask参数返回的。

## 2.4 绑定地址、解析地址和解析路由
### 2.4.1 rdma_resolve_addr
**原型**：
``` cpp
int rdma_resolve_addr (struct rdma_cm_id *id, struct sockaddr *src_addr, struct sockaddr *dst_addr, int timeout_ms)
```
**输入参数**：

* id——RDMA标识符。结构体详细信息见rdma_create_id。
* src_addr——源地址信息。可以为NULL。详细信息见rdma_create_id。
* dst_addr——目标地址信息。
* timeout_ms——等待解析完成的时间。

**输出参数**：无。

**返回值**：成功时为0，失败返回-1并设置errno以指示失败的原因。

**描述**：

rdma_resolve_addr将目标地址和可选的源地址从IP地址解析为RDMA地址。如果成功，则指定的rdma_cm_id将绑定到本地设备。

该调用用于将一个给定的目标IP地址映射到可用的RDMA地址。 IP到RDMA地址的映射是使用本地路由表或通过ARP完成的。如果指定了源地址，则将rdma_cm_id绑定到该地址，就像调用rdma_bind_addr一样。如果没有给出源地址，并且rdma_cm_id尚未绑定到设备，则rdma_cm_id将基于本地路由表绑定到源地址。在此调用之后，rdma_cm_id将绑定到RDMA设备。通常在调用rdma_resolve_route和rdma_connect之前从连接的主动端（客户端）进行此调用。

InfiniBand规范：此调用将目标IP地址和源IP地址映射到GID。为了执行映射，IPoIB必须在本地和远程节点上运行。

### 2.4.2 rdma_resolve_route
**原型**：
``` cpp
 int rdma_resolve_route (struct rdma_cm_id *id, int timeout_ms)
```
**输入参数**：

* id——RDMA标识符。结构体详细信息见rdma_create_id。
* timeout_ms——等待解析完成时间。

**输出参数**：无。

**返回值**：成功时为0，失败返回-1并设置errno以指示失败的原因。

**描述**：

rdma_resolve_route解析到目标地址的RDMA路由以建立连接。 目的地址必须已经通过调用rdma_resolve_addr解析。 因此，该函数在rdma_resolve_addr之后但在调用rdma_connect之前在客户端被调用。 对于InfiniBand连接，该调用获取该连接使用的路径记录。

InfiniBand规范：此调用获取连接使用的路径记录。

### 2.4.3 rdma_bind_addr

**原型**：
``` cpp
int rdma_bind_addr (struct rdma_cm_id *id, struct sockaddr *addr);
```
**输入参数**：

* id——RDMA标识符。结构体详细信息见rdma_create_id。
* addr——本地地址信息。允许通配符。结构体详细信息见rdma_create_id。

**输出参数**：无。

**返回值**：成功时为0，失败返回-1并设置errno以指示失败的原因。

**描述**：

rdma_bind_addr将源地址与rdma_cm_id相关联。该地址可能是通配符。如果绑定到指定的本地地址，则rdma_cm_id也将绑定到本地RDMA设备。

 通常，在调用rdma_listen之前，调用此函数以绑定到指定端口号，但是，也可以在一个连接的主动端（客户端）调用 rdma_resolve_addr之前，调用这个函数以绑定到指定地址。

如果用于绑定到端口0，则rdma_cm将选择一个可用端口，这个端口可以使用rdma_get_src_port进行检索。

### 2.4.4 rdma_getaddrinfo
**原型**：
``` cpp
int rdma_getaddrinfo (const char *node, const char *service, const struct rdma_addrinfo *hints, 
							struct rdma_addrinfo **res)
```
**输入参数**：

* node——名称，点分十进制的IPv4或十六进制的IPv6地址，可选的。
* service——服务名称或地址的端口号。
* hints——指向rmda_addrinfo结构体的指针，该结构体包含有关调用者支持的服务类型的暗示。结构体详细信息见下文。
* res——指向包含响应信息的rdma_addrinfo结构体的链表的指针。结构体详细信息见下文。

**输出参数**：res——rdma_addrinfo结构体，该结构体返回建立通信所需的信息。

**返回值**：成功时为0，失败返回-1并设置errno以指示失败的原因。

**描述**：

rdma_getaddrinfo提供与传输无关的地址转换。它解析目标节点和服务地址，并返回建立设备通信所需的信息。它与getaddrinfo的函数等效。

必须提供参数node、service或hints中的一个。 如果提供了hints，则该操作将由hints.ai_flags控制。 如果指定了RAI_PASSIVE，则调用将解析地址信息以便l连接的被动端（服务端）使用。 如果提供了node，则rdma_getaddrinfo将尝试解析到给定节点的RDMA地址，路由和连接数据。 如果提供了hints参数，则可以使用它来控制结果输出，如下所示。 如果未提供node，则rdma_getaddrinfo将尝试基于hints.ai_src_addr，hints.ai_dst_addr或hints.ai_route解析RDMA寻址信息。

 struct rdma_addrinfo定义如下：
 ```cpp
 struct rdma_addrinfo {
	int					ai_flags; 			//提示标志，用于控制操作。它是几个常量的按位或结果。
	int					ai_family;			//源地址和目标地址的地址族
	int					ai_qp_type;			//使用的RDMA QP类型
	int					ai_port_space;		//使用的RDMA端口空间，宏常量按位或，详细信息见下文
	socklen_t			ai_src_len;			//ai_src_addr引用的源地址的长度
	socklen_t			ai_dst_len;			//ai_dst_addr引用的目标地址的长度
	struct sockaddr		*ai_src_addr;		//本地RDMA设备的地址（如果提供），见rdma_create_id
	struct sockaddr		*ai_dst_addr;		//目标RDMA设备的地址（如果提供），见rdma_create_id
	char					*ai_src_canonname;	//源的规范名
	char					*ai_dst_canonname;	//目标的规范名
	size_t				ai_route_len;		//引用的路由信息缓冲区的大小
	void					*ai_route;			//RDMA传输的路由信息
	size_t				ai_connect_len;		//ai_connect引用的连接信息的大小
	void					*ai_connect;		//交换数据，是建立连接的一部分
	struct rdma_addrinfo	*ai_next;			//指向链表中下一个rdma_addrinfo结构体的指针。
};  
 ```

下面是struct rdma_addrinfo的完整描述：

|成员名|描述|
|:--|:--|
|ai_flags|用于控制操作的提示标志。它是一个或多个常量的按位或结果。见下面详细描述。|
|ai_family|源地址和目标地址的地址族。它是一个常量。见下面详细描述。|
|ai_qp_type|用于通信的RDMA QP的类型。支持的类型为：IBV_UD（不可靠的数据报）和IBV_RC（可靠连接）。|
|ai_port_space|使用的RDMA端口空间。支持的值为：RDMA_PS_UDP、RDMA_PS_TCP和RDMA_PS_IB。详细信息见rdma_create_id（）|
|ai_src_len |ai_src_addr引用的源地址的长度。如果找不到给定目标的适当源地址，则该值为0。|
|ai_dst_len |ai_dst_addr引用的目标地址的长度。如果将RAI_PASSIVE标志指定为hints的一部分，则该值为0。|
|ai_src_addr|如果提供，则为本地RDMA设备的地址。详细信息见`man 2 bind`|
|ai_dst_addr|如果提供，则为目标RDMA设备的地址。详细信息见`man 2 bind`|
|ai_src_canonname|源的规范名字|
|ai_dst_canonname|目标的规范名字|
|ai_route_len|ai_route引用的路由信息缓冲区的大小。如果底层传输不需要路由数据，或者无法解析，则为0。|
|ai_route |RDMA传输的路由信息，RDMA传输需要路由数据作为连接建立的一部分。路由数据的格式取决于底层传输。如果使用了Infiniband传输，且路由数据可用，ai_route将在输出中引用struct ibv_path_data数组。可以通过在rdma_getaddrinfo的输入参数中设置所需的路由数据字段来限制路由路径。对于Infiniband，hints.ai_route可以在输入中引用struct ibv_path_record或struct ibv_path_data数组。|
|ai_connect_len|ai_connect引用的连接信息的大小。如果基础传输不需要其他连接信息，则该值为0。|
|ai_connect |交换的数据是连接建立过程的一部分。如果提供，ai_connect数据必须作为私有数据传输，任何用户提供的私有数据都紧随其后。|
|ai_next  |指向链表中下一个rdma_addrinfo结构体的指针。如果没有更多结构体，则为NULL。|


ai_flags的可取值定义如下：
```cpp
#define RAI_PASSIVE		0x00000001	//结果将用于连接的被动/监听侧。
#define RAI_NUMERICHOST	0x00000002	//参数node(如果提供)必须是数字网络地址。该标志禁止任何冗长的地址解析
#define RAI_NOROUTE		0x00000004	//如果设置，则此标志抑制任何冗长的路由解析
#define RAI_FAMILY		0x00000008	//如果设置，则ai_family设置应该用作解释node参数的输入提示。
```

下面是ai_family的可取值的完整描述：
|宏|值|描述|
|:--|:--|:--|
|AF_INET|2|IPv4
|AF_INET6|23|IPv6
|AF_UNSPEC|0|协议无关|
|AF_IB|27|IB


### 2.4.5 rdma_freeaddrinfo
**原型**：
``` cpp
void rdma_freeaddrinfo(struct rdma_addrinfo *res)
```
**输入参数**：res——要释放的rdma_addrinfo结构体。结构体详细信息见 rdma_getaddrinfo。

**输出参数**：无。

**返回值**：无。

**描述**：rdma_freeaddrinfo释放函数rdma_getaddrinfo返回的rdma_addrinfo（res）结构体。 请注意，如果ai_next不为NULL，则rdma_freeaddrinfo将释放addrinfo结构体的整个链表。

## 2.5	连接的监听、连接、接受、拒绝、断开

### 2.5.1 rdma_listen
**原型**：
``` cpp
int rdma_listen (struct rdma_cm_id *id, int backlog)
```
**输入参数**：

* id——RDMA标识符。结构体详细信息见rdma_create_id。
* backlog——传入的连接请求的积压。

**输出参数**：无。

**返回值**：成功时为0，失败返回-1并设置errno以指示失败的原因。

**描述**：   

启动对传入的连接请求或数据报服务查找的监听。监听将仅限于本地绑定的源地址。

请注意，在调用rdma_listen之前，必须先通过调用rdma_bind_addr来将rdma_cm_id绑定到本地地址。 如果rdma_cm_id绑定到指定的IP地址，则监听将限于该地址和关联的RDMA设备。 如果rdma_cm_id仅绑定到RDMA端口号，则监听将在所有RDMA设备上进行。

### 2.5.2 rdma_connect
**原型**：
``` cpp
int rdma_connect (struct rdma_cm_id *id, struct rdma_conn_param *conn_param)
```
**输入参数**：

* id——RDMA通信标识符。结构体详细信息见rdma_create_id。
* conn_param——可选的连接参数。结构体详细信息见rdma_accept。

**输出参数**：无。

**返回值**：成功时为0，失败返回-1并设置errno以指示失败的原因。

**描述**：

rdma_connect启动一个主动端（客户端）的连接请求。对于类型为RDMA_PS_TCP的rdma_cm_id，该调用会发起到远程目标的连接请求。对于类型为RDMA_PS_UDP的rdma_cm_id，它会启动对提供数据报服务的远程QP的查找。

调用这个函数之前，用户必须已经通过调用rdma_resolve_route或rdma_create_ep来解析到目标地址的路由。

struct rdma_conn_param的定义如下：

``` cpp

struct rdma_conn_param 
{
	const void	*private_data;		//用户控制的数据缓冲区.
	uint8_t		private_data_len;	//用户控制的数据缓冲区的大小
	uint8_t		responder_resources;//本地端接收的来自远程端的未完成的RDMA读取和原子操作的最大数量
	uint8_t		initiator_depth;	//本地端拥有的对远程端的未完成RDMA读取和原子操作的最大数量
	uint8_t		flow_control;		//指定硬件流控制是否可用
	uint8_t		retry_count;		//发生错误时在连接上重试数据传输操作的最大次数，接受连接时忽略。
	uint8_t		rnr_retry_count;	//收到RNR错误后，远程对等方应在连接上重试发送操作的最大次数。
	//如果QP是在rdma_cm_id上创建的，则忽略以下两项
	uint8_t		srq;  				//指定与连接关联的QP是否正在使用共享接收队列。
	uint32_t	qp_num; 			//指定与连接关联的QP编号。如果QP是在rdma_cm_id上创建的，则忽略它
};       
```
下面是struct rdma_conn_param 的完整描述：

以下属性用于配置通信，并在发起连接请求或建立数据报通信时由conn_param参数指定。

|成员名|描述
|:--|:--|
|private_data|引用一个用户控制的数据缓冲区。缓冲区的内容被复制，并被作为通信请求的一部分透明地传递到远程端。如果不需要private_data，则可以为NULL。|
|private_data_len|指定用户控制的数据缓冲区的大小。请注意，传输到远程端的实际数据量取决于传输，并且可能大于要求的数据量。|
|responder_resources |本地端将接受的来自远程端的未完成的RDMA读和原子操作的最大数量。仅适用于RDMA_PS_TCP。此值必须小于或等于本地RDMA设备的属性max_qp_rd_atom和远程RDMA设备属性max_qp_init_rd_atom。接受连接时，远端可以调整此值|
|initiator_depth|本地端发出的对远程端的未完成RDMA读取和原子操作的最大数量。 仅适用于RDMA_PS_TCP。 此值必须小于或等于本地RDMA设备属性max_qp_init_rd_atom和远程RDMA设备属性max_qp_rd_atom。 接受连接时，远程端点可以调整此值。|
|flow_control |指定硬件流控制是否可用。该值与远程对等方交换，不用于配置QP。仅适用于RDMA_PS_TCP。|
|retry_count |发生错误时在连接上重试数据传输操作的最大次数。此设置控制发生超时时重试send，RDMA读写和原子操作的次数。仅适用于RDMA_PS_TCP。|
|rnr_retry_count |收到接收者未准备好（RNR）错误后，远程对等方应在连接上重试发送操作的最大次数。当发送请求到达时，而缓冲区还没有被发布以接收传入数据，将生成RNR错误。仅适用于RDMA_PS_TCP。
|srq |指定与连接关联的QP是否正在使用共享接收队列。 如果在rdma_cm_id上创建了QP，则库将忽略此字段。 仅适用于RDMA_PS_TCP。|
|qp_num|指定与连接关联的QP编号。 如果在rdma_cm_id上创建了QP，则库将忽略此字段。 仅适用于RDMA_PS_TCP。|

InfiniBand规范：

除了上述定义的连接属性外，InfiniBand QP还配置了最小RNR NAK定时器和本地ACK超时值。 最小RNR NAK定时器值设置为0，延迟为655ms。 本地ACK超时值是根据包的生存期和本地HCA ACK 延时来计算的。 包的生存期由InfiniBand子网管理员决定，是连接主动端（服务端）获得的路由（路径记录）信息的一部分。 HCA ACK延时是本地使用的HCA的属性。

重试计数和RNR重试计数为3比特位的值。

用户提供的私有数据的长度对于RDMA_PS_TCP限制为56个字节，对于RDMA_PS_UDP限制为180个字节

iWarp规范：当前，通过iWarp RDMA设备建立的连接要求连接的主动端（客户端）发送第一条消息。

responder_resources和initiator_depth的最大值定义为：
```cpp
enum
{
	RDMA_MAX_RESP_RES = 0xFF,	//responder_resources最大值
	RDMA_MAX_INIT_DEPTH = 0xFF //initiator_depth最大值
};
```

### 2.5.3 rdma_establish

**原型**：
``` cpp
int rdma_establish(struct rdma_cm_id *id)
```
**输入参数**：id——RDMA通信标识符。结构体详细信息见rdma_create_id。

**输出参数**：无。

**返回值**：成功时为0，失败返回-1并设置errno以指示失败的原因。

**描述**：

rdma_ establish确认传入的连接响应事件并完成连接建立。

如果尚未在rdma_cm_id上创建QP，那么在获得连接响应事件后，主动端（客户端）应调用此函数以完成连接，

这将触在被动端（服务端）触发连接建立事件。

不得在已创建QP的rdma_cm_id上使用此函数。

译者注：通常用于不使用RDMA CM而用libibverbs来创建QP的情况，

### 2.5.4 rdma_get_request
**原型**：
``` cpp
int rdma_get_request (struct rdma_cm_id *listen, struct rdma_cm_id **id)
```
**输入参数**：listen——监听的rdma_cm_id。结构体详细信息见rdma_create_id。

**输出参数**：id——与新连接关联的rdma_cm_id。结构体详细信息见rdma_create_id。

**返回值**：成功时为0，失败返回-1并设置errno以指示失败的原因。

**描述**：

rdma_get_request检索一个挂起的连接请求事件。 该调用只能用于同步运行的rdma_cm_ids监听中。 如果调用成功，代表连接请求的新rdma_cm_id（id）将返回给用户。 新的rdma_cm_id将引用与该请求关联的事件信息，直到用户在新创建的RDMA标识符上调用rdma_reject，rdma_accept或rdma_destroy_id为止。 有关事件数据的描述，请参见rdma_get_cm_event。

如果QP与监听的rdma_cm_id关联，则返回的rdma_cm_id也将引用已分配的QP。

### 2.5.5 rdma_accept
**原型**：
``` cpp
int rdma_accept (struct rdma_cm_id *id, struct rdma_conn_param *conn_param)
```
**输入参数**：

* id——与请求关联的RDMA通信标识符。结构体详细信息见rdma_create_id。
* conn_param——建立连接所需要的信息，可选的。结构体详细信息见下文。

**输出参数**：无。

**返回值**：成功时为0，失败返回-1并设置errno以指示失败的原因。

**描述**：

从监听端调用rdma_accept以接受连接请求或数据报服务查找请求。

与套接字接受函数不同，不是在监听的rdma_cm_id上调用rdma_accept。而是在新的rdma_cm_id上调用rdma_accept：在调用rdma_listen之后，用户等待RDMA_CM_EVENT_CONNECT_REQUEST事件发生。连接请求事件为用户提供了一个新创建的rdma_cm_id，类似于新的套接字，这个rdma_cm_id被绑定到指定的RDMA设备。

struct rdma_conn_param的定义如下：

``` cpp

struct rdma_conn_param 
{
	const void	*private_data;		//用户控制的数据缓冲区.
	uint8_t		private_data_len;	//用户控制的数据缓冲区的大小
	uint8_t		responder_resources;//本地端接受的来自远程端的未完成的RDMA读取和原子操作的最大数量
	uint8_t		initiator_depth;	//本地端发出的对远程端的未完成RDMA读取和原子操作的最大数量
	uint8_t		flow_control;		//指定硬件流控制是否可用
	uint8_t		retry_count;		//发生错误时在连接上重试数据传输操作的最大次数，接受连接时忽略。
	uint8_t		rnr_retry_count;	//收到RNR错误后，远程对等方应在连接上重试发送操作的最大次数。
	//如果QP是在rdma_cm_id上创建的，则忽略以下两项
	uint8_t		srq;  				//指定与连接关联的QP是否正在使用共享接收队列。
	uint32_t	qp_num; 			//指定与连接关联的QP编号。如果QP是在rdma_cm_id上创建的，则忽略它
};       
```
下面是struct rdma_conn_param 的完整描述：

以下属性用于配置通信，并在接受连接请求或数据报通信请求时由conn_param参数指定。 用户在接受时应使用连接请求事件中报告的rdma_conn_param值来确定这些字段的适当值。 用户可以直接引用连接事件中的rdma_conn_param结构体，也可以引用自己的rdma_conn_param结构体。 如果引用了连接请求事件的rdma_conn_param结构体，则在此调用返回之前，该事件不能被确认。

如果conn_param参数为NULL，则使用连接请求事件中报告的值，并根据本地硬件限制向下调整。

|成员名|描述
|:--|:--|
|private_data|引用一个用户控制的数据缓冲区。缓冲区的内容被复制，并被作为通信请求的一部分透明地传递到远程端。如果不需要private_data，则可以为NULL。|
|private_data_len|指定用户控制的数据缓冲区的大小。请注意，传输到远程端的实际数据量取决于传输，并且可能大于要求的数据量。|
|responder_resources |本地端将接受的来自远程端的未完成的RDMA读和原子操作的最大数量。仅适用于RDMA_PS_TCP。此值必须小于或等于本地RDMA设备的属性max_qp_rd_atom，但最好大于或等于连接请求事件中报告的responder_resources值。|
|Initiator_depth|本地端发出的对远程端的未完成RDMA读和原子操作的最大数量。 仅适用于RDMA_PS_TCP。 此值必须小于或等于本地RDMA设备的属性max_qp_init_rd_atom和连接请求事件中报告的initiator_depth值|
|flow_control |指定硬件流控制是否可用。该值与远程对等方交换，不用于配置QP。仅适用于RDMA_PS_TCP。
|retry_count |此致被忽略|
|rnr_retry_count |在收到接收端未准备好（RNR）错误后，远程对等方应在连接上重试发送操作的最大次数。当发送请求到达时，而缓冲区还没有被发布到接收队列以接收传入数据，将生成RNR错误。仅适用于RDMA_PS_TCP。
|srq |指定与连接关联的QP是否正在使用共享接收队列。 如果QP是在rdma_cm_id上创建的，则库将忽略此字段。 仅适用于RDMA_PS_TCP。|
|qp_num|指定与连接关联的QP编号。 如果QP是在rdma_cm_id上创建的，则库将忽略此字段。|

InfiniBand规范：

除了上述定义的连接属性外，InfiniBand QP还配置了最小RNR NAK定时器和本地ACK超时值。 最小RNR NAK定时器值设置为0，延迟为655ms。 本地ACK超时值是根据包的生存期和本地HCA ACK 延时来计算的。 包的生存期由InfiniBand子网管理员决定，是连接主动端（服务端）获得的路由（路径记录）信息的一部分。 HCA ACK延时是本地使用的HCA的属性。

RNR重试计数为3比特位的值。

用户提供的私有数据的长度对于RDMA_PS_TCP限制为196个字节，对于RDMA_PS_UDP限制为136个字节.

### 2.5.6 rdma_reject
**原型**：
``` cpp
int rdma_reject (struct rdma_cm_id *id, const void *private_data, uint8_t private_data_len)
```
**输入参数**：

* id——与连接请求关联的RDMA标识符。结构体详细信息见rdma_create_id。
* private_data——与拒绝消息一起发送私有数据（可选的）。
* private_data_len——发送的私有数据的大小(以字节为单位)

**输出参数**：无。

**返回值**：成功时为0，失败返回-1并设置errno以指示失败的原因。

**描述**：

监听端调用rdma_reject拒绝连接请求或数据报服务查找请求。

收到连接请求事件后，用户可以调用rdma_reject拒绝请求。 如果底层RDMA传输支持在拒绝消息中的携带私有数据，则私有数据将传递到远程端。


### 2.5.7 rdma_disconnect

**原型**：
``` cpp
int rdma_disconnect (struct rdma_cm_id *id)
```
**输入参数**：id——RDMA标识符。结构体详细信息见rdma_create_id。

**输出参数**：无。

**返回值**：成功时为0，失败返回-1并设置errno以指示失败的原因。

**描述**：rdma_disconnect断开连接，并将所有关联的QP转换为error状态。 此操作将导致所有已发布的工作请求都被刷新到完成队列中。 一个连接的客户端和服务端都可以调用rdma_disconnect。 成功断开连接后，将在连接的两端生成RDMA_CM_EVENT_DISCONNECTED事件。

## 2.6 综合操作
### 2.6.1 rdma_create_ep
**原型**：
``` cpp
int rdma_create_ep(struct rdma_cm_id **id, struct rdma_addrinfo *res, struct ibv_pd *pd,
					struct ibv_qp_init_attr *qp_init_attr)
```
**输入参数**：

* res——rdma_getaddrinfo返回的与rdma_cm_id关联的地址信息。结构体详细信息见rdma_getaddrinfo。
* pd——如果QP与rdma_cm_id关联，则指定QP的保护域（可选的）。结构体详细信息见ibv_alloc_pd。
* qp_init_attr——初始QP属性（可选的）。结构体详细信息见ibv_create_qp。

**输出参数**：id——用于返回已分配的通信标识符的引用。结构体详细信息见rdma_create_id。

**返回值**：成功时为0，失败返回-1并设置errno以指示失败的原因。

**描述**：

rdma_create_ep创建用于跟踪通信信息的标识符以及QP（QP是可选的）。

使用rdma_getaddrinfo解析地址信息后，用户可以使用此调用并基于返回的结果来分配rdma_cm_id。

如果rdma_cm_id将被用于在连接的主动端（客户端），这意味着`res->ai_flag`未设置RAI_PASSIVE标志，如果qp_init_attr不为NULL，则rdma_create_ep将自动在rdma_cm_id上创建一个QP。 QP将与指定的保护域（如果提供了pd参数）关联，或者与默认的保护域（如果未提供pd参数）关联。用户应该查看rdma_create_qp以获得有关pd和qp_init_attr参数的详细信息。调用rdma_create_ep之后，可以通过调用rdma_connect连接返回的rdma_cm_id。主动端（客户端）调用rdma_resolve_addr和rdma_resolve_route是不需要的。

如果rdma_cm_id被用于连接的被动端（服务端），这意味着`res->ai_flag`设置了RAI_PASSIVE标志，则此调用将保存提供的pd和qp_init_attr参数。通过调用rdma_get_request检索新的连接请求时，与新连接关联的rdma_cm_id将自动与使用pd和qp_init_attr参数的QP关联。调用rdma_create_ep之后，可以通过立即调用rdma_listen将返回的rdma_cm_id置于监听状态。被动端（服务端）不需要调用rdma_bind_addr。之后可以通过调用rdma_get_request检索连接请求。

新创建的rdma_cm_id将设置为使用同步操作(rdma_connect, rdma_listen，rdma_get_request)。希望异步操作的用户必须使用rdma_migrate_id将rdma_cm_id迁移到用户创建的事件通道。

用户必须通过调用rdma_destroy_ep释放创建的rdma_cm_id。

### 2.6.2 rdma_destroy_ep
**原型**：
``` cpp
void rdma_destroy_ep (struct rdma_cm_id *id);
```
**输入参数**：id——要销毁的通信标识符。结构体详细信息见rdma_create_id。

**输出参数**：无。

**返回值**：无。

**描述**：rdma_destroy_ep销毁指定的rdma_cm_id和所有相关资源，包括与id关联的QP和SRQ。

## 2.7 查询通信双方地址和端口
### 2.7.1 rdma_get_src_port
**原型**：
``` cpp
__be16 rdma_get_src_port (struct rdma_cm_id *id)
```
**输入参数**：id——RDMA标识符。结构体详细信息见rdma_create_id。

**输出参数**：无。

**返回值**：返回与本地端点关联的16位端口标识符。 如果rdma_cm_id未绑定到端口，则返回值为0。

**描述**：返回已绑定到本地地址的rdma_cm_id的本地端口号。 如果rdma_cm_id未绑定到端口，则返回值为0。

### 2.7.2 rdma_get_dst_port
**原型**：
``` cpp
__be16 rdma_get_dst_port (struct rdma_cm_id *id);
```
**输入参数**：id——RDMA标识符。结构体详细信息见rdma_create_id。

**输出参数**：无。

**返回值**： 返回与远程对等端点关联的16位端口标识符。 如果rdma_cm_id未连接，则返回值为0。
**描述**：返回已绑定到远程地址的rdma_cm_id的远程端口号。如果rdma_cm_id id没有连接，则返回0。
  
### 2.7.3 rdma_get_local_addr
**原型**：
``` cpp
struct sockaddr * rdma_get_local_addr (struct rdma_cm_id *id)
```
**输入参数**：id——RDMA标识符。结构体详细信息见rdma_create_id。

**输出参数**：无。

**返回值**：返回一个指向rdma_cm_id的本地sockaddr地址的指针。 如果id未绑定到地址，则sockaddr结构体的内容将设置为全零。结构体详细信息见`man 2 bind`。

**描述**：rdma_get_local_addr返回已绑定到本地设备的rdma_cm_id的本地IP地址。

### 2.7.4 rdma_get_peer_addr
**原型**：
``` cpp
struct sockaddr * rdma_get_peer_addr (struct rdma_cm_id *id)
```
**输入参数**：id——RDMA标识符。结构体详细信息见rdma_create_id。

**输出参数**：无。

**返回值**： 返回指向有连接对等方的sockaddr地址的指针。 如果rdma_cm_id未连接，则sockaddr结构的内容将设置为全零。结构体详细信息见`man 2 bind`

**描述**：rdma_get_peer_addr返回rdma_cm_id关联的远程IP地址。

## 2.8 设备列表的获取和销毁

### 2.8.1 rdma_get_devices
**原型**：
``` cpp
struct ibv_context ** rdma_get_devices (int *num_devices)
```
**输入参数**：num_devices——如果非NULL，则设置为返回的RDMA设备数量。

**输出参数**：num_devices——当前可用的RDMA设备数量。

**返回值**：  返回可用RDMA设备的数组，如果请求失败，则返回NULL。 失败时，将设置errno以指示失败原因。结构体详细信息见ibv_open_device。

**描述**：

返回以NULL终止的已打开的RDMA设备的数组。 调用者可以使用此函数在指定的RDMA设备上分配资源，这些设备将在多个rdma_cm_id之间共享。

加载librdmacm时，设备保持打开状态，必须通过调用rdma_free_devices释放返回的数组。
 
### 2.8.2 rdma_free_devices
**原型**：
``` cpp
void rdma_free_devices (struct ibv_context **list)
```
**输入参数**：list——rdma_get_devices返回的RDMA设备数组。结构体详细信息见ibv_open_device。

**输出参数**：无。

**返回值**：无。

**描述**： rdma_free_devices释放rdma_get_devices返回的RDMA设备数组。

## 2.9 创建和销毁SRQ
### 2.9.1 rdma_create_srq
**原型**：
``` cpp
int rdma_create_srq (struct rdma_cm_id *id, struct ibv_pd *pd, struct ibv_srq_init_attr *attr)
```
**输入参数**：

* id——RDMA通信标识符。结构体详细信息见rdma_create_id。
* pd——共享接收队列（SRQ）的保护域（可选的）。结构体详细信息见ibv_alloc_pd。
* attr——初始SRQ属性。结构体详细信息见ibv_create_srq。

**输出参数**：attr——通过此结构体返回创建的SRQ的实际功能和属性。

**返回值**：成功返回0，错误返回-1。 如果发生错误，将设置errno来指示失败原因。

**描述**：

rdma_create_srq分配与指定的rdma_cm_id相关的SRQ。

在调用此函数之前，必须将rdma_cm_id绑定到本地RDMA设备，保护域（如果提供了）必须用于同一设备。分配SRQ后，SRQ将准备好处理发布的接收请求。 

如果未提供保护域（pd参数为NULL），则将使用默认保护域创建rdma_cm_id。每个RDMA设备分配了一个默认的保护域。

初始SRQ属性由attr参数指定。 ibv_srq_init_attr中的ext.xrc.cq字段是可选的。如果未为XRC SRQ指定完成队列，则rdma_cm将为SRQ分配CQ以及相应的完成通道。 rdma_cm创建的完成通道和CQ数据通过rdma_cm_id结构体向用户公开。

创建的SRQ的实际功能和属性将通过attr参数返回给用户。 一个rdma_cm_id只能与一个SRQ关联。

### 2.9.2 rdma_create_srq_ex

**原型**：
``` cpp
int rdma_create_srq_ex(struct rdma_cm_id *id, struct ibv_srq_init_attr_ex *attr)
```
**输入参数**：

* id——RDMA通信标识符。结构体详细信息见rdma_create_id。
* attr——初始SRQ属性（扩展的）。结构体详细信息见ibv_create_srq_ex。

**输出参数**：attr——SRQ的实际功能和属性（扩展的）

**返回值**：无。

**描述**：rdma_destroy_srq销毁在rdma_cm_id上分配的SRQ。 必须先销毁与rdma_cm_id相关的任何SRQ，然后再销毁rdma_cm_id。

### 2.9.3 rdma_destroy_srq

**原型**：
``` cpp
void rdma_destroy_srq (struct rdma_cm_id *id)
```
**输入参数**：id——与要销毁的SRQ关联的RDMA标识符。结构体详细信息见rdma_create_id。

**输出参数**：无。

**返回值**：无。

**描述**：rdma_destroy_srq销毁在rdma_cm_id上分配的SRQ。 必须先销毁与rdma_cm_id相关的任何SRQ，然后再销毁rdma_cm_id。


## 2.10 多播的加入和退出
### 2.10.1 rdma_join_multicast
**原型**：
``` cpp
int rdma_join_multicast (struct rdma_cm_id *id, struct sockaddr *addr, void *context)
```
**输入参数**：

* id——与请求关联的通信标识符。结构体详细信息见rdma_create_id。
* addr——要加入的多播地址标识符。结构体详细信息见`man 2  bind`。
* context——与加入请求关联的用户定义上下文。

**输出参数**：无。

**返回值**：成功时为0，失败返回-1并设置errno以指示失败的原因。

**描述**：

加入多播组并将关联的QP添加到该组。

加入多播组之前，必须通过调用rdma_bind_addr或rdma_resolve_addr将rdma_cm_id绑定到RDMA设备。除非提供了指定的源地址，否则rdma_resolve_addr的使用要求本地路由表将多播地址解析到RDMA设备。用户必须调用rdma_leave_multicast离开多播组并释放任何多播资源。加入操作完成后，如果QP与rdma_cm_id关联，则当用户检索到多播事件时，它将自动添加到多播组，否则，用户负责调用ibv_attach_mcast将QP绑定到多播组。加入上下文通过rdma_cm_event中的private_data字段返回给用户。

### 2.10.2 rdma_join_multicast_ex
**原型**：
``` cpp
int rdma_join_multicast_ex (struct rdma_cm_id *id, struct rdma_cm_join_mc_attr_ex *mc_join_attr,
							void *context)
```
**输入参数**：

* id——与请求关联的通信标识符。结构体详细信息见rdma_create_id。
* mc_join_attr——要加入的多播组的属性。结构体详细信息见下文。
* context——与加入请求关联的用户定义上下文。

**输出参数**：无。

**返回值**：成功时为0，失败返回-1并设置errno以指示失败的原因。

**描述**：

带扩展选项加入多播组（MCG）。当前支持带有指定加入标志的多播加入。

加入多播组之前，必须通过调用rdma_bind_addr或rdma_resolve_addr将rdma_cm_id绑定到RDMA设备。除非提供了指定的源地址，否则使用rdma_resolve_addr要求本地路由表将多播地址解析到RDMA设备。用户必须调用rdma_leave_multicast离开多播组并释放任何多播资源。加入操作完成后，如果QP与rdma_cm_id关联，则当用户检索到多播事件时它将自动添加到多播组，否则，用户负责调用ibv_attach_mcast将QP绑定到多播组。加入上下文通过rdma_cm_event中的private_data字段返回给用户。

 struct rdma_cm_join_mc_attr_ex定义如下：

```cpp
struct rdma_cm_join_mc_attr_ex
{
	uint32_t		comp_mask;		/* enum rdma_cm_join_mc_attr_mask枚举值按位或得到，详细信息见下文 */
	uint32_t		join_flags;		/* enum rdma_cm_mc_join_flags枚举值之一 ，详细信息见下文*/  
	struct sockaddr	*addr;			/* 标识要加入的多播组的地址，见`man 2 bind` */
};
```
enum rdma_cm_join_mc_attr_mask定义如下：
```cpp
enum rdma_cm_join_mc_attr_mask
{
	RDMA_CM_JOIN_MC_ATTR_ADDRESS		= 1 << 0,
	RDMA_CM_JOIN_MC_ATTR_JOIN_FLAGS		= 1 << 1,
	RDMA_CM_JOIN_MC_ATTR_RESERVED		= 1 << 2,
};
 ```
 enum rdma_cm_mc_join_flags定义如下：
 ```cpp
enum rdma_cm_mc_join_flags
{
	RDMA_MC_JOIN_FLAG_FULLMEMBER,  		   //创建多播组，向MCG发送多播消息，从MCG接收多播消息。       
	RDMA_MC_JOIN_FLAG_SENDONLY_FULLMEMBER,  //创建多播组，将多播消息发送到MCG，不接收来自MCG的多播消息
	RDMA_MC_JOIN_FLAG_RESERVED,   		   //保留的       
};
```

在InfiniBand上以“只发送成员”身份启动多播组加入需要子网管理器支持，否则加入将失败。

在RoCEv2/ETH上以“只发送成员”身份发起多播组加入不会像正式成员加入多播组那样发送任何IGMP消息。 使用“只发送成员”时，QP将不会添加到MCG。

### 2.10.3 rdma_leave_multicast
**原型**：
``` cpp
 int rdma_leave_multicast (struct rdma_cm_id *id, struct sockaddr *addr)
```
**输入参数**：

* id——与请求关联的通信标识符。结构体详细信息见rdma_create_id。
* addr——标识要离开的多播组的地址。结构体详细信息见`man 2 bind`。

**输出参数**：无。

**返回值**：成功时为0，失败返回-1并设置errno以指示失败的原因。

**描述**：

离开多播组，并从该组分离关联的QP。

在完全加入一个多播组之前调用此函数会导致加入操作被取消。 用户应注意，从多播组接收到的消息在离开多播组后可能仍会排队等待立即的完成处理。 销毁rdma_cm_id将自动离开所有多播组。

## 2.11 事件的通知、获取、确认
### 2.11.1 rdma_notify
**原型**：
``` cpp
int rdma_notify (struct rdma_cm_id *id, enum ibv_event_type event)
```
**输入参数**：

* id——RDMA标识符。结构体详细信息见rdma_create_id。
* event——异步事件。枚举值详细信息见第4节。

**输出参数**：无。

**返回值**：成功时为0，失败返回-1并设置errno以指示失败的原因。如果errno设置为EISCONN（传输端点已连接），则表明底层的通信管理器在处理对rdma_notify的调用之前已建立连接。 在这种情况下，可以安全地忽略该错误。

**描述**：

rdma_notify用于通知librdmacm异步事件已在与rdma_cm_id关联的QP上发生。

通过用户的设备事件处理程序报告在QP上发生的异步事件。该函数用于通知librdmacm通信事件。在大多数情况下，不需要使用此函数，但是，如果连接建立是在带外完成的（例如通过InfiniBand完成），则有可能在尚未被视为已连接的QP上接收数据。在这种情况下，此函数会强制连接进入已建立状态，以便处理罕见的情况，即连接永远不会自行形成。调用此函数可确保将RDMA_CM_EVENT_ESTABLISHED事件传递到应用程序。应该报告给CM的事件是：IB_EVENT_COMM_EST。


### 2.11.2 rdma_get_cm_event
**原型**：
``` cpp
int rdma_get_cm_event (struct rdma_event_channel *channel, struct rdma_cm_event **event)
```
**输入参数**：

* chanel——要查找的事件通道。结构体详细信息见rdma_create_event_channel。

**输出参数**：event——下一个通信事件相关的已分配信息。结构体详细信息见下文。

**返回值**：成功时为0，失败返回-1并设置errno以指示失败的原因。

**描述**：   

检索通信事件。如果没有待处理的事件，则默认情况下，该调用将一直阻塞，直到收到事件为止。

可以通过修改与给定通道关联的文件描述符来更改此函数的默认同步行为。必须通过调用rdma_ack_cm_event确认所有报告的事件。 rdma_cm_id的销毁将阻塞直到确认了相关事件为止。


struct rdma_cm_event定义如下：

```cpp
struct rdma_cm_event
{
	struct rdma_cm_id			*id;			//rdma ID
	struct rdma_cm_id			*listen_id;		//监听ID
	enum rdma_cm_event_type		event;			//事件类型，它是一个枚举值
	int							status;			//与事件关联的所有异步错误信息
	union
	{
		struct rdma_conn_param	conn;			//与有连接的QP服务相关的事件参数
		struct rdma_ud_param	ud;				//与不可靠数据报服务相关的事件参数
	} param;
};
```
通信事件的详细信息在rdma_cm_event结构体中返回。该结构由体rdma_cm分配，并由rdma_ack_cm_event函数释放。

下面是struct rdma_cm_event的完整描述：
|成员名|描述|
|:--|:--|
|id|与事件关联的rdma_cm标识符。如果事件类型为RDMA_CM_EVENT_CONNECT_REQUEST，则它引用该通信的新ID。|
|listen_id|对于RDMA_CM_EVENT_CONNECT_REQUEST事件类型，这引用了相应的监听请求标识符。
|event|指定发生的通信事件的类型。它是enum rdma_cm_event_type的一个枚举值。见下面第4节RDMA CM事件详细描述。|
| status|返回与事件关联的所有异步错误信息。如果操作成功，则状态为零，否则状态值为非零，并且设置为errno或传输的特定值。有关传输特定状态值的详细信息，请参阅下面的事件类型信息。|
|param|根据事件类型提供其他详细信息。用户应基于与事件关联的rdma_cm_id的rdma_port_space来选择conn或ud子字段。请参阅下面的struct rdma_conn_param 和 struct rdma_ud_param|。


struct struct rdma_conn_param定义如下：

```cpp
struct struct rdma_conn_param {   
	const void *private_data;		//与事件关联的任何由用户指定的数据
	uint8_t private_data_len;		//私有数据缓冲区的大小。
	uint8_t responder_resources;	//接收者请求的响应者资源数量。
	uint8_t initiator_depth;		//接收者的未完成的最大RDMA读/原子操作的最大值
	uint8_t flow_control;			//发送方是否提供硬件级别的流控制。
	uint8_t retry_count;	    //接收者应重试发送操作的次数，接受连接时忽略
	uint8_t rnr_retry_count;		//遇到RNR NACK错误，接收者应重 的次数。
       									//如果QP在rdma_cm_id上创建，则被忽略下面两个字段|
	uint8_t srq;					//指定发送者是否正在使用共享接收队列
	uint32_t qp_num;				//连接的远程QP编号。
}; 
```
与有连接的QP服务RDMA_PS_TCP相关的事件参数。除非另有说明，否则与连接有关的事件数据对RDMA_CM_EVENT_CONNECT_REQUEST和RDMA_CM_EVENT_ESTABLISHED事件有效。

下面是struct rdma_conn_param 的完整描述：
|成员名|描述|
|:--|:--|
| private_data|引用与事件关联的任何由用户指定的数据。此字段引用的数据与远程端调用rdma_connect或rdma_accept时指定的数据匹配。如果事件不包含私有数据，则此字段为NULL。调用rdma_ack_cm_event时，将释放此指针引用的缓冲区。|
|private_data_len| 私有数据缓冲区的大小。用户应注意，私有数据缓冲区的大小可能大于对端发送的私有数据量。缓冲区中的任何其他空间将被清零。|
|responder_resources|接收者请求的响应者资源数量。调用rdma_connect和rdma_accept时，此字段匹配远程节点指定的发起者深度。|
|initiator_depth|接收者的未完成的最大RDMA读/原子操作的最大值。调用rdma_connect和rdma_accept时，此字段与远程节点指定的响应者资源匹配。|
| flow_control|指示发送方是否提供硬件级别的流控制。|
| retry_count |仅对于RDMA_CM_EVENT_CONNECT_REQUEST事件，指示接收者应重试发送操作的次数。|
|rnr_retry_count| 遇到RNR NACK错误，接收者应重试的次数。|
|srq|指定发送者是否正在使用共享接收队列。
|qp_num|指示连接的远程QP编号。|

struct rdma_ud_param定义如下：
```cpp
struct rdma_ud_param
{
	const void			*private_data;
	uint8_t				private_data_len;
	struct ibv_ah_attr	ah_attr;
	uint32_t			qp_num;
	uint32_t			qkey; //默认为#define RDMA_UDP_QKEY 0x01234567
}; 
```
与不可靠的数据报（UD）服务RDMA_PS_UDP和RDMA_PS_IPOIB相关的事件参数。除非另有说明，否则UD事件数据对RDMA_CM_EVENT_ESTABLISHED和RDMA_CM_EVENT_MULTICAST_JOIN事件有效。

下面是struct rdma_ud_param完整描述：
|成员名|描述|
|:--|:--|
|private_data|引用与RDMA_CM_EVENT_CONNECT_REQUEST或RDMA_CM_EVENT_ESTABLISHED事件关联的任何由用户指定的数据。此字段引用的数据与远程端调用rdma_connect或rdma_accept时指定的数据匹配。 如果事件不包含私有数据，则此字段为NULL。 调用rdma_ack_cm_event时，将释放此指针引用的缓冲区。|
|private_data_len|私有数据缓冲区的大小。 用户应注意，私有数据缓冲区的大小可能大于远端发送的私有数据量。 缓冲区中的任何其他空间将被清零。|
|ah_attr |将数据发送到远程端点所需的地址信息。 用户分配地址句柄时应使用此结构体。|
|  qp_num   |  远程端点或多播组的QP号。|
|qkey       |将数据发送到远程端点需要的Qkey|

RDMA CM为默认UDP QP和多播组创建的全局QKEY定义为：
```cpp
#define RDMA_UDP_QKEY 0x01234567
```
### 2.11.3 rdma_ack_cm_event
**原型**：
``` cpp
int rdma_ack_cm_event (struct rdma_cm_event *event)
```
**输入参数**：event——要释放的事件。结构体详细信息见rdma_get_cm_event。

**输出参数**：无。

**返回值**：成功时为0，失败返回-1并设置errno以指示失败的原因。

**描述**：rdma_ack_cm_event释放通信事件。 必须释放由rdma_get_cm_event分配的所有事件，成功的get和acks之间应该一一对应。 此调用将释放事件结构体及其引用的任何内存。

### 2.11.4 rdma_event_str
**原型**：
``` cpp
char * rdma_event_str (enumrdma_cm_event_type event )
```
**输入参数**：event——异步事件。枚举详细信息见第4节。

**输出参数**：物。

**返回值**：返回一个指向与事件对应的静态字符串的指针。

**描述**：rdma_event_str返回异步事件的字符串表示形式。

# 3 RDMA动词API
RMDA动词API实际上基本都是在设置一些参数后直接调用IB verbs API。调用关系见下表。
|librmacm API|底层libibverbs API|
|:--|:--|
|rdma_reg_msgs|ibv_reg_mr|
|rdma_reg_read|ibv_reg_mr|
|rdma_reg_write|ibv_reg_mr|
|rmda_dereg_mr|ibv_dereg_mr|
|rdma_post_recvv|ibv_post_recv/ibv_post_srq_recv|
|rdma_post_sendv|ibv_post_send|
|rdma_post_readv|ibv_post_send|
|rdma_post_writev|ibv_post_send|
|rdma_post_recv|ibv_post_recv/ibv_post_srq_recv|
|rdma_post_send|ibv_post_send|
|rdma_post_read|ibv_post_send|
|rdma_post_write|ibv_post_send|
|rdma_post_ud_send|ibv_post_send|
|rdma_get_send_comp|ibv_poll_cq、 ibv_req_notify_cq、ibv_get_cq_event、ibv_ack_cq_events|
|rdma_get_recv_comp|ibv_poll_cq、 ibv_req_notify_cq、ibv_get_cq_event、ibv_ack_cq_events|


## 3.1 内存注册操作
### 3.1.1 rdma_reg_msgs

**原型**：
``` cpp
 struct ibv_mr * rdma_reg_msgs (struct rdma_cm_id *id, void *addr, size_t length);

```
**输入参数**：
 
 * id——对将使用消息缓冲区的通信标识符的引用。结构体详细信息见rdma_create_id。
 * addr——要注册的内存缓冲区的地址。
 * length——要注册的内存的总长度。


**输出参数**：无。

**返回值**：如果成功，则返回对已注册内存区域的引用；如果错误，则返回NULL。如果发生错误，将设置errno来指示失败原因。

**描述**：

注册用于发送/接收消息或RDMA操作的内存缓冲区数组。 使用rdma_reg_msgs注册的内存缓冲区可以使用rdma_post_send或rdma_post_recv发布到rdma_cm_id，或者指定为RDMA读取操作的目标或RDMA写入请求的源。

rdma_reg_msgs用于注册数据缓冲区数组，该缓冲区数组将用于在与rdma_cm_id关联的QP上发送或接收消息。 内存缓冲区被注册到与标识符相关联的保护域。 数据缓冲区数组的起始地址是通过addr参数指定的，而数组的总大小由length给出。

所有数据缓冲区被作为工作请求发布之前，应先对其进行注册。 用户必须通过调用rdma_dereg_mr取消注册所有已注册的内存。

  

### 3.1.2 rdma_reg_read
**原型**：
``` cpp
struct ibv_mr * rdma_reg_read (struct rdma_cm_id *id, void *addr, size_t length)
```
**输入参数**：

 * id——对将使用消息缓冲区的通信标识符的引用。结构体详细信息见rdma_create_id。
 * addr——要注册的内存缓冲区的地址。
 * length——要注册的内存的总长度。
 
**输出参数**：无。

**返回值**： 如果成功，则返回对已注册内存区域的引用；如果错误，则返回NULL。如果发生错误，将设置errno来指示失败原因。

**描述**：

注册一个将由远程RDMA 读取操作访问的内存缓冲区。 使用rdma_reg_read注册的内存缓冲区可以作为RDMA读取请求或类似调用的目标，从而允许在RDMA连接的远程端将缓冲区指定为rdma_post_read的remote_addr。

rdma_reg_read用于在与rdma_cm_id关联的队列对上注册一个数据缓冲区，该缓冲区将是RDMA读取操作的目标。内存缓冲区被注册到与标识符相关联的保护域。 数据缓冲区数组的起始地址是通过addr参数指定的，而数组的总大小由length给出。 

所有数据缓冲区被作为工作请求发布之前，应先对其进行注册。 用户必须通过调用rdma_dereg_mr取消注册所有已注册的内存。

### 3.1.3 rdma_reg_write
**原型**：
``` cpp
struct ibv_mr * rdma_reg_write (struct rdma_cm_id *id, void *addr, size_t length)
```
**输入参数**：

 * id——对将使用消息缓冲区的通信标识符的引用。结构体详细信息见rdma_create_id。
 * addr——要注册的内存缓冲区的地址。
 * length——要注册的内存的总长度。

**输出参数**：无。

**返回值**：  如果成功，则返回对已注册内存区域的引用；如果错误，则返回NULL。如果发生错误，将设置errno来指示失败原因。

**描述**：

注册一个将由远程RDMA 写入操作访问的内存缓冲区。 使用rdma_reg_write注册的内存缓冲区可以作为RDMA写入请求或类似调用的目标，从而允许在RDMA连接的远程端将缓冲区指定为rdma_post_write的remote_addr。

rdma_reg_write用于在与rdma_cm_id关联的队列对上注册一个数据缓冲区，该缓冲区将是RDMA写入操作的目标。内存缓冲区被注册到与标识符相关联的保护域。 数据缓冲区数组的起始地址是通过addr参数指定的，而数组的总大小由length给出。 

所有数据缓冲区被作为工作请求发布之前，应先对其进行注册。 用户必须通过调用rdma_dereg_mr取消注册所有已注册的内存。

### 3.1.4 rdma_dereg_mr
**原型**：
``` cpp
 int rdma_dereg_mr (struct ibv_mr *mr)
```
**输入参数**：mr—— 对已注册内存缓冲区的引用。结构体详细信息见ibv_reg_mr。

**输出参数**：无。

**返回值**：成功返回0，错误返回-1。 如果发生错误，将设置errno来指示失败原因。

**描述**：

注销已为RDMA或消息操作注册的内存缓冲区。 用户应在销毁rdma_cm_id之前为与rdma_cm_id关联的所有已注册内存调用rdma_dereg_mr。

向rdma_cm_id注册的所有内存都被关联到与该ID关联的保护域。 用户必须先注销所有注册的内存，然后才能销毁保护域。


## 3.2 活动队列对操作

### 3.2.1 rdma_post_recvv

**原型**：
``` cpp
int rdma_post_recvv (struct rdma_cm_id *id, void *context, struct ibv_sge *sgl, int nsge)
```
**输入参数**：

* id——对将在其中发布消息缓冲区的通信标识符的引用。结构体详细信息见rdma_create_id。
* context——与请求关联的用户定义上下文。
* sgl——作为单个请求发布的内存缓冲区的分散聚合列表。结构体详细信息见ibv_post_recv。
* nsge——sgl数组中的分散聚合条目的数量。

**输出参数**：无。

**返回值**：成功返回0，错误返回-1。如果发生错误，将设置errno来指示失败原因。

**描述**：

将单个工作请求发布到与rdma_cm_id关联的队列对的接收队列中。 已发布的缓冲区将排队等待接收远程对等方发送的传入消息。

请注意，这个函数支持多分散聚合条目。用户负责确保在对等方发布相应的发送消息之前，接收请求被发布，并且总缓冲区空间足以容纳所有已发送的数据。 消息缓冲区必须在发布之前已被注册，并且缓冲区必须保持注册直到接收完成。

仅在将队列对与消息关联后，才可以将消息发布到rdma_cm_id。 如果使用rdma_create_id分配了rdma_cm_id，则在调用rdma_create_ep或rdma_create_qp之后，将一个队列对绑定到rdma_cm_id。

与接收请求关联的用户定义上下文将通过工作完成wr_id（工作请求标识符）字段返回给用户。


### 3.2.2 rdma_post_sendv
**原型**：
``` cpp
int rdma_post_sendv (struct rdma_cm_id *id, void *context, struct ibv_sge *slg, int nsge, int flags)
```
**输入参数**：

* id——对将在其中发布消息缓冲区的通信标识符的引用。结构体详细信息见rdma_create_id。
* context——与请求关联的用户定义上下文。
* sgl——作为单个请求发布的内存缓冲区的分散聚合列表。结构体详细信息见ibv_post_recv。
* nsge——sgl数组中的分散聚合条目的数量。
* flags——标志，用于控制发送操作的可选标志。枚举的详细信息见ibv_post_send。

**输出参数**：无。

**返回值**：成功返回0，错误返回-1。如果发生错误，将设置errno来指示失败原因。

**描述**：

将工作请求发布到与rdma_cm_id关联的队列对的发送队列中。 已发布缓冲区的内容将发送到连接的远程对等方。

用户负责确保在发出发送操作之前，远程对等方已将接收请求排队。 有关支持的标志的列表，请参见ibv_post_send。 除非发送请求使用内联数据，否则消息缓冲区必须在发布之前已注册，并且缓冲区必须保持注册状态，直到发送完成。

在建立连接之前，，可能不会将其发送到rdma_cm_id或相应的队列对。

与发送请求关联的用户定义上下文将通过工作完成wr_id（工作请求标识符）字段返回给用户。


### 3.2.3 rdma_post_readv
**原型**：
``` cpp
int rdma_post_readv (struct rdma_cm_id *id, void *context, struct ibv_sge *sgl, 
					int nsge, int flags, uint64_t remote_addr, uint32_t rkey)
```
**输入参数**：

* id——对将在其中发布消息缓冲区的通信标识符的引用。结构体详细信息见rdma_create_id。
* context——与请求关联的用户定义上下文。
* sgl——读取的目标缓冲区的分散聚合列表。结构体详细信息见ibv_post_recv。
* nsge——sgl数组中的分散聚合条目的数量。
* flags——标志，用于控制读取操作的可选标志。枚举的详细信息见ibv_post_send。
* remote_addr——要读取的远程已注册内存的地址。
* rkey——与远程地址关联的已注册内存密钥。

**输出参数**：无。

**返回值**：成功返回0，错误返回-1。如果发生错误，将设置errno来指示失败原因。

**描述**：

rdma_post_readv将工作请求发布到与rdma_cm_id关联的队列对的发送队列中。 remote_addr中远程存储区域的内容将被读入sgl数组中给定的本地数据缓冲区中。

有关支持的标志的列表，请参见ibv_post_send。 在发出读取之前，必须已注册了远程和本地数据缓冲区，并且缓冲区必须保持注册状态，直到读取完成。

在建立连接之前，，可能无法将读取操作发布到rdma_cm_id或相应的队列对。

与读取请求关联的用户定义上下文将通过工作完成wr_id（工作请求标识符）字段返回给用户。


### 3.2.4 rdma_post_writev
**原型**：
``` cpp
int rdma_post_writev (struct rdma_cm_id *id, void *context, struct ibv_sge *sgl, 
						int nsge, int flags, uint64_t remote_addr, uint32_t rkey)
```
**输入参数**：

* id——对将在其中发布消息缓冲区的通信标识符的引用。结构体详细信息见rdma_create_id。
* context——与请求关联的用户定义上下文。
* sgl——写入的目标缓冲区的分散聚合列表。
* nsge——sgl数组中的分散聚合条目的数量。结构体详细信息见ibv_post_recv。
* flags——标志，用于控制读取操作的可选标志。枚举的详细信息见ibv_post_send。
* remote_addr——要写入的远程已注册内存的地址。
* rkey——与远程地址关联的已注册内存密钥。

**输出参数**：无。

**返回值**：成功返回0，错误返回-1。如果发生错误，将设置errno来指示失败原因。

**描述**：


rdma_post_writev将工作请求发布到与rdma_cm_id关联的队列对的发送队列中。 sgl数组中本地数据缓冲区的内容将被写入remote_addr的远程存储区域。

有关支持的标志的列表，请参见ibv_post_send。 在发出写入之前，必须已注册了本地数据数据缓冲区，并且缓冲区必须保持注册状态，直到写入完成。远程缓冲区必须始终被注册。

在建立连接之前，，可能无法将写入操作发布到rdma_cm_id或相应的队列对。

与写入请求关联的用户定义上下文将通过工作完成wr_id（工作请求标识符）字段返回给用户。

### 3.2.5 rdma_post_recv
**原型**：
``` cpp
int rdma_post_recv (struct rdma_cm_id *id, void *context, void *addr, size_t length, struct ibv_mr *mr)
```
**输入参数**：

* id——对将在其中发布消息缓冲区的通信标识符的引用。结构体详细信息见rdma_create_id。
* context——与请求关联的用户定义的上下文。
* addr——要发布的内存缓冲区的地址。
* length——内存缓冲区的长度。
* mr——与发布的缓冲区关联的注册内存区域。结构体详细信息见ibv_reg_mr。

**输出参数**：无。

**返回值**：成功返回0，错误返回-1。如果发生错误，将设置errno来指示失败原因。

**描述**：

将工作请求发布到与rdma_cm_id关联的队列对的接收队列中。已发布的缓冲区将排队等待接收远程对等方发送的传入消息。

用户负责确保在对等方发布相应的发送消息之前，发布了一个接收缓冲区并且其大小足以容纳所有已发送的数据。消息缓冲区必须在发布之前已注册，并且mr参数引用了该注册区域。缓冲区必须保持注册状态，直到接收完成。

仅在将队列对与消息关联后，才可以将消息发布到rdma_cm_id。如果使用rdma_create_id分配了rdma_cm_id，则在调用rdma_create_ep或rdma_create_qp之后，将一个队列对绑定到rdma_cm_id。

与接收请求关联的用户定义上下文将通过工作完成wr_id（工作请求标识符）字段返回给用户。

请注意，这是一个简单的接收调用。 此处不涉及分散/聚合列表。

### 3.2.6 rdma_post_send
**原型**：
``` cpp
int rdma_post_send (struct rdma_cm_id *id, void *context, 
					void *addr, size_t length, struct ibv_mr *mr, int flags)
```
**输入参数**：

* id——对将在其中发布消息缓冲区的通信标识符的引用。结构体详细信息见rdma_create_id。
* context——与请求关联的用户定义的上下文。
* addr——要发布的内存缓冲区的地址。
* length——内存缓冲区的长度。
* mr——与发布的缓冲区关联的注册内存区域（可选的）。结构体详细信息见ibv_reg_mr.
* flags——用于控制发送操作的标志（可选的）。结构体详细信息见ibv_post_send。

**输出参数**：无。

**返回值**：成功返回0，错误返回-1。如果发生错误，将设置errno来指示失败原因。

**描述**：

rdma_post_send将工作请求发布到与rdma_cm_id，id关联的队列对的发送队列中。已发布缓冲区的内容将发送到连接的远程对等方。

用户负责确保在发出发送操作之前，远程对等方已将接收请求排队。另外，除非发送请求使用内联数据，否则消息缓冲区必须在发布之前已经被注册，并且mr参数引用该注册区域。缓冲区必须保持注册状态，直到发送完成。

在建立连接之前，发送操作可能不会发布到rdma_cm_id或相应的队列对中。

与发送请求关联的用户定义上下文将通过工作完成wr_id（工作请求标识符）字段返回给用户。



### 3.2.7 rdma_post_read
**原型**：
``` cpp
int rdma_post_read (struct rdma_cm_id *id, void *context, void *addr, size_t length, 
					struct ibv_mr *mr, int flags, uint64_t remote_addr, uint32_t rkey)
```
**输入参数**：

* id——对将在其中发布消息缓冲区的通信标识符的引用。结构体详细信息见rdma_create_id。
* context——与请求关联的用户定义上下文。
* addr——读取请求的本地目标的地址。
* length——读取操作的长度。
* mr——与本地缓冲区关联的注册内存区域。结构体详细信息见ibv_reg_mr.
* flags——标志，用于控制读取操作的可选标志。结构体详细信息见ibv_post_send。
* remote_addr——要读取的远程已注册内存的地址。
* rkey——与远程地址关联的已注册内存密钥。


**输出参数**：无。

**返回值**：成功返回0，错误返回-1。如果发生错误，将设置errno来指示失败原因。

**描述**：

rdma_post_read将工作请求发布到与rdma_cm_id关联的队列对的发送队列中。远程存储区的内容将被读入本地数据缓冲区。

有关支持的标志的列表，请参见ibv_post_send。用户必须确保在发出读取之前，必须已注册了远程和本地数据缓冲区，并且缓冲区必须保持注册状态直到读取完成。

在连接建立之前，可能无法将读取操作发布到rdma_cm_id或相应的队列对。

与读取请求关联的用户定义上下文将通过工作完成wr_id（工作请求标识符）字段返回给用户。


### 3.2.8 rdma_post_write
**原型**：
``` cpp
int rdma_post_write (struct rdma_cm_id *id, void *context, void *addr, size_t length, 
					struct ibv_mr *mr, int flags, uint64_t remote_addr, uint32_t rkey)
```
**输入参数**：

* id——对将在其中发布消息缓冲区的通信标识符的引用。结构体详细信息见rdma_create_id。
* context——与请求关联的用户定义上下文。
* addr——写入请求的源的本地地址。
* length——写入操作的长度。
* mr——与本地缓冲区关联的注册内存区域。结构体详细信息见ibv_reg_mr。
* flags——标志，用于控制写入操作的可选标志。结构体详细信息见ibv_post_send。
* remote_addr——要写入的远程已注册内存的地址。
* rkey——与远程地址关联的已注册内存密钥。

**输出参数**：无。

**返回值**：成功返回0，错误返回-1。如果发生错误，将设置errno来指示失败原因。

**描述**：

rdma_post_write将工作请求发布到与rdma_cm_id关联的队列对的发送队列中。本地数据缓冲区的内容将被写入远程内存区。

有关支持的标志的列表，请参见ibv_post_send。除非指定内联数据，否则在发出写入之前必须已注册本地数据缓冲区，并且缓冲区必须保持注册状态，直到写入完成。远程缓冲区必须始终被注册。。

除非指定内联数据，否则在发出写入之前必须已注册本地数据缓冲区，并且缓冲区必须保持注册状态，直到写入完成。远程缓冲区必须始终被注册。

在建立连接之前，写操作可能不会发布到rdma_cm_id或相应的队列对中。

与写请求关联的用户定义上下文将通过工作完成wr_id（工作请求标识符）字段返回给用户。



### 3.2.9 rdma_post_ud_send
**原型**：
``` cpp
int rdma_post_ud_send (struct rdma_cm_id *id, void *context, void *addr, size_t length, 
						struct ibv_mr *mr, int flags, struct ibv_ah *ah, uint32_t remote_qpn)
```
**输入参数**：

* id——对将在其中发布消息缓冲区的通信标识符的引用。结构体详细信息见rdma_create_id。
* context——与请求关联的用户定义的上下文。
* addr——要发布的内存缓冲区的地址。
* length——内存缓冲区的长度。
* mr——与发布的缓冲区关联的注册内存区域（可选的）。结构体详细信息见ibv_reg_mr。
* flags——用于控制发送操作的标志（可选的）。结构体详细信息见ibv_post_send。
* ah——描述远程节点地址的地址句柄。结构体详细信息见ibv_create_ah。
* remote_qpn——目标队列对的编号。

**输出参数**：无。

**返回值**：成功返回0，错误返回-1。如果发生错误，将设置errno来指示失败原因。

**描述**：

rdma_post_ud_send将工作请求发布到与rdma_cm_id关联的队列对的发送队列中。已发布缓冲区的内容将发送到指定的目标队列对。

用户负责确保在发出发送操作之前，目标队列对已将接收请求排队。除非发送请求使用内联数据，否则消息缓冲区必须在发布之前已注册，且mr参数引用该注册区域。缓冲区必须保持注册状态，直到发送完成。

与发送请求关联的用户定义上下文将通过工作完成wr_id（工作请求标识符）字段返回给用户。




### 3.2.10 rdma_get_send_comp
**原型**：
``` cpp
int rdma_get_send_comp (struct rdma_cm_id *id, struct ibv_wc *wc)
```
**输入参数**：

* id——对要检查完成的通信标识符的引用。结构体详细信息见rdma_create_id。
* wc——对要填充的工作完成结构体的引用。结构体详细信息见ibv_poll_cq。

**输出参数**：wc——对工作完成结构体的引用。 函数返回时，该结构将体包含有关已完成请求的信息

**返回值**：一个非负值（0或1），等于成功时发现的完成数，失败时则为-1。 如果调用失败，将设置errno来指示失败的原因。

**描述**：

rdma_get_send_comp为一个发送操作、RDMA读取操作或RDMA写入操作检索一个完成的工作请求。 通过wc参数返回有关已完成请求的信息，并将wr_id设置为请求的上下文。 有关工作完成结构体ibv_wc的详细信息，请参见ibv_poll_cq。

请注意，此调用轮询与rdma_cm_id关联的发送完成队列。 如果未找到完成，则调用将阻塞，直到请求完成为止。 因此，这意味着仅应在不与其他rdma_cm_id共享CQ的rdma_cm_id上使用该调用，并为发送和接收完成维护单独的CQ。


### 3.2.11 rdma_get_recv_comp
**原型**：
``` cpp
int rdma_get_recv_comp (struct rdma_cm_id *id, struct ibv_wc *wc)
```
**输入参数**：

* id——对要检查完成的通信标识符的引用。结构体详细信息见rdma_create_id。
* wc——对要填充的工作完成结构体的引用。结构体详细信息见ibv_poll_cq。

**输出参数**：成功返回返回的完成数（0或1），错误返回-1。 如果发生错误，将设置errno来指示失败原因。

**返回值**：一个非负值（0或1），等于成功时发现的完成数，失败时则为-1。 如果调用失败，将设置errno来指示失败的原因。

**描述**：

rdma_get_recv_comp为一个接收操作完成检索一个完成的工作请求。 通过wc参数返回有关已完成请求的信息，并将wr_id设置为请求的上下文。 有关工作完成结构体ibv_wc的详细信息，请参见ibv_poll_cq。

请注意，此调用轮询与rdma_cm_id关联的接收完成队列。 如果未找到完成，则调用将阻塞，直到请求完成为止。 因此，这意味着仅应在不与其他rdma_cm_id共享CQ的rdma_cm_id上使用该调用，并为发送和接收完成维护单独的CQ。



# 4  RDMA CM事件

RDMA CM事件是enum rdma_cm_event_type的一个枚举值，它的定义如下：
```cpp
enum rdma_cm_event_type {
	RDMA_CM_EVENT_ADDR_RESOLVED,	//地址解析完成
	RDMA_CM_EVENT_ADDR_ERROR,		//地址解析出错
	RDMA_CM_EVENT_ROUTE_RESOLVED,	//路由解析完成
	RDMA_CM_EVENT_ROUTE_ERROR,		//路由解析出错
	RDMA_CM_EVENT_CONNECT_REQUEST,	//有新的连接请求，被动端
	RDMA_CM_EVENT_CONNECT_RESPONSE,	//有新的连接响应，主动端
	RDMA_CM_EVENT_CONNECT_ERROR,	//连接出错
	RDMA_CM_EVENT_UNREACHABLE,		//无法访问，主动端
	RDMA_CM_EVENT_REJECTED,			//连接被拒绝
	RDMA_CM_EVENT_ESTABLISHED,		//已经建立连接
	RDMA_CM_EVENT_DISCONNECTED,		//连接断开
	RDMA_CM_EVENT_DEVICE_REMOVAL,	//RDMA设备移除
	RDMA_CM_EVENT_MULTICAST_JOIN,	//加入多播组
	RDMA_CM_EVENT_MULTICAST_ERROR,	//加入多播出错
	RDMA_CM_EVENT_ADDR_CHANGE,		//地址改变
	RDMA_CM_EVENT_TIMEWAIT_EXIT		//过了TimeWait状态，可以重用
};  
```

下面是enum rdma_cm_event_type的完整描述：

**RDMA_CM_EVENT_ADDR_RESOLVED——客户端事件**
地址解析（rdma_resolve_addr）成功完成。在客户端（主动）端响应rdma_resolve_addr()时生成此事件。当系统能够解析客户端提供的服务器地址时，将产生该事件。

**RDMA_CM_EVENT_ADDR_ERROR——客户端事件**
地址解析（rdma_resolve_addr）失败。此事件在客户端（主动）端生成。在发生错误的情况下，它是响应rdma_resolve_addr()而生成的。例如，如果找不到设备（例如，用户提供了不正确的设备），则可能会发生这种情况。具体来说，如果远程设备同时具有以太网和IB接口，并且客户端提供了以太网设备名称而不是服务器端的IB设备名称，则将生成RDMA_CM_EVENT_ADDR_ERROR。

**RDMA_CM_EVENT_ROUTE_RESOLVED——客户端事件**
路由解析（rdma_resolve_route）成功完成。在客户端（主动）端响应rdma_resolve_route()时生成此事件。 当系统能够解析客户端提供的服务器地址时，将产生该事件。

**RDMA_CM_EVENT_ROUTE_ERROR——客户端事件**
路由解析（rdma_resolve_route）失败。rdma_resolve_route()失败时，将生成此事件。

**RDMA_CM_EVENT_CONNECT_REQUEST——服务端事件**
在被动端生成，以通知用户新的连接请求。它表示已收到一个连接请求。

**RDMA_CM_EVENT_CONNECT_RESPONSE——客户端事件**
在主动端侧生成，以通知用户对被动端对连接请求的成功响应。它仅在没有关联QP的rdma_cm_id上生成。

**RDMA_CM_EVENT_CONNECT_ERROR——两端事件**
表示尝试建立连接(rdma_establish)或发起连接(rdma_connect)时发生错误。可能在连接的主动或被动端生成。

**RDMA_CM_EVENT_UNREACHABLE——客户端事件**
在主动端生成，通知用户远程服务器不可访问或无法响应连接请求。如果响应InfiniBand上的UD QP解析请求时生成此事件，则事件status字段将包含errno（如果为负数）或IB CM SIDR REP消息中携带的状态结果。

**RDMA_CM_EVENT_REJECTED——客户端事件**
此事件可能在客户端（主动）端生成，表示连接请求或响应被远程设备拒绝。事件status字段将包含传输指定的拒绝原因（如果有）。例如，如果尝试与错误端口上的远程端点进行连接，则可能会发生这种情况。在InfiniBand下，这是IB CM REJ消息中携带的拒绝原因。

**RDMA_CM_EVENT_ESTABLISHED——两端事件**
表示已与远程端点建立连接。在连接的两侧都生成此事件。

**RDMA_CM_EVENT_DISCONNECTED——两端事件**
在连接的两侧都生成此事件以响应rdma_disconnect（）。将生成该事件表示本地和远程设备之间的连接已断开。任何关联的QP将转换为Error状态。所有已发布的工作请求都将被刷新。用户必须将任何此类QP的状态更改为“Reset”以进行恢复。

**RDMA_CM_EVENT_DEVICE_REMOVAL**
当与rdma_cm_id关联的本地RDMA设备已被删除时生成此事件。收到此事件后，用户必须销毁相关的rdma_cm_id。

**RDMA_CM_EVENT_MULTICAST_JOIN**
此事件是对 rdma_join_multicast()的响应，表示多播加入操作（rdma_join_multicast）已成功完成。

**RDMA_CM_EVENT_MULTICAST_ERROR**
此事件表示，企图加入多播组或连接现有多播组（如果已加入该组）时发生错误。当事件发生时，指定的多播组不再可访问，如果需要，应重新加入。

**RDMA_CM_EVENT_ADDR_CHANGE**
此事件表示，通过地址解析与此ID关联的网络设备更改了其硬件地址，例如在绑定故障转移之后。对于希望把用于其RDMA会话的链接与网络堆栈对齐的应用程序，此事件可以作为提示。

**RDMA_CM_EVENT_TIMEWAIT_EXIT**
此事件表示：与连接关联的QP已退出其timewait状态，现在可以重新使用了。断开QP后，它会保持在timewait状态，以允许任何飞行中的数据包退出网络。 timewait状态完成后，rdma_cm将报告此事件。

------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>[《RDMA Aware Networks Programming User Manual v1.7》](https://www.mellanox.com/related-docs/prod_software/RDMA_Aware_Programming_user_manual.pdf)、[《Dotan Barak的RDMAmojo博客》](http://www.rdmamojo.com/)、《man手册 v24》，我所做的工作就是更新、整理一部分内容，删除了一部分废弃的API，添加了一部分新的API。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------