---
title: RDMA网络编程手册 第6章 编程示例
tags: RDMA
---

------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>[《RDMA Aware Networks Programming User Manual v1.7》](https://www.mellanox.com/related-docs/prod_software/RDMA_Aware_Programming_user_manual.pdf)、[《Dotan Barak的RDMAmojo博客》](http://www.rdmamojo.com/)、《Ubuntu19.10 rdma-core套件 man手册v24.0》，我所做的工作就是更新、整理一部分内容，删除了一部分废弃的API，添加了一部分新的API。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------



# 1 使用IBV动词编程

# 2  使用RDMA动词编程

## 2.1 自动路径迁移（APM）
```cpp
8.1 Automatic Path Migration (APM)
/*
 * 编译命令：
 * gcc apm.c -o apm -libverbs -lrdmacm
 *
 * 描述：
 * 这个例子说明了自动路径迁移（APM）。基本流程如下：
 * 1. 在客户端和服务端之间建立连接
 * 2. 在连接的每一侧设置备用路径详细信息
 * 3. 在客户端和服务器之间来回执行发送操作
 * 4. 导致路径被迁移（手动或自动）
 * 5. 使用备用路径完成发送
 *
 * 有两种方法可以导致路径迁移
 * 1. 使用ibv_modify_qp动词设置，path_mig_state = IBV_MIG_MIGRATED
 * 2.假设连接的至少一侧有两个端口，并且每个端口都有到另一个主机的路径，
 * 请拔出原始端口的电缆，然后观察它迁移到 另一个端口。
 * 
 * 运行示例：
 * 此示例需要特定的IB网络配置才能正确演示APM。 需要两台主机，
 * 一台用于客户端，一台用于服务器。 这两个主机中的至少一个必须具有带两个端口的IB卡。
 * 这两个端口都应该连接到同一子网，并且每个端口都具有通过IB交换机到另一台主机的路由。
 * 该可执行文件可以作为客户端或服务器应用程序运行。 首先在一个主机上启动服务器端，
 * 然后在另一台主机上启动客户端。 使用默认参数，客户端和服务器将在100秒内交换100个发送。 
 * 在此期间，手动拔下连接到有两个端口的主机的原始端口的电缆，然后观察路径被迁移到另一个端口。 
 * 迁移路径最多可能需要一分钟。 要查看通过软件迁移的路径，请使用客户端上的-m选项。
 *
 * 服务端：
 * ./apm -s
 *
 * 客户端（-a是远程接口的IP）：
 * ./apm -a 192.168.1.12
 *
 */
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <errno.h>
#include <getopt.h>
#include <rdma/rdma_verbs.h>

#define VERB_ERR(verb, ret)  fprintf(stderr, "%s returned %d errno %d\n", verb, ret, errno)

/* 默认参数值 */
#define DEFAULT_PORT "51216"
#define DEFAULT_MSG_COUNT 100
#define DEFAULT_MSG_LENGTH 1000000
#define DEFAULT_MSEC_DELAY 500

/* 示例中使用的资源 */
struct context
{
	/* 用户参数 */
	int 			server;
	char			*server_name;
	char 			*server_port;
	int 			msg_count;
	int 			msg_length;
	int 			msec_delay;
	uint8_t 		alt_srcport;
	uint16_t 		alt_dlid;
	uint16_t		my_alt_dlid;
	int 			migrate_after;
	/* 资源 */
	struct rdma_cm_id 	*id;
	struct rdma_cm_id 	*listen_id;
	struct ibv_mr 		*send_mr;
	struct ibv_mr 		*recv_mr;
	char 				*send_buf;
	char 				*recv_buf;
	pthread_t 			async_event_thread;
};

/*
* 函数：async_event_thread
*
* 输入：arg，struct context对象
*
* 输出：无

* 返回值：NULL
*
* 描述：读取在发送数据期间发生的任何异步事件，并打印出事件的详细信息。 特别是与迁移相关的事件。
* 
*/
static void *async_event_thread(void *arg)
{
	struct ibv_async_event event;
	int ret;
	struct context *ctx = (struct context *) arg;
	while (1) 
	{
		ret = ibv_get_async_event(ctx->id->verbs, &event);
		if (ret)
		{
			VERB_ERR("ibv_get_async_event", ret);
			break;
		}
		switch (event.event_type) 
		{
			case IBV_EVENT_PATH_MIG：
				printf("QP path migrated\n");
				break;
			case IBV_EVENT_PATH_MIG_ERR：
				printf("QP path migration error\n");
				break;
			default：
				printf("Async Event %d\n", event.event_type);
				break;
		}
		ibv_ack_async_event(&event);
	}
	return NULL;
}

/*
 * 函数：get_alt_dlid_from_private_data
 *
 * 输入：event， 含有私有数据的RDMA事件
 *
 * 输出：dlid，在私有数据中进行发送的。
 *
 * 返回值：成功返回值0，失败返回值非0.
 * 
 * 描述：获取从远端发送的私有数据并返回值私有数据中包含的目的地LID。
 * 
 */
int get_alt_dlid_from_private_data(struct rdma_cm_event *event, uint16_t *dlid)
{
	if (event->param.conn.private_data_len < 4) 
	{
		printf("unexpected private data len： %d",
		event->param.conn.private_data_len);
		return -1;
	}

	*dlid = ntohs(*((uint16_t *) event->param.conn.private_data));
	return 0;
}

/*
 * 函数：get_alt_port_details
 *
 * ：ctx，struct context对象
 *
 * 输出：
 无
 *
 * 返回值s：
 * 成功返回值0，失败返回值非0
 *
 * 描述：
 * 首先，查询设备以确定是否支持路径迁移。 
 * 接下来，查询设备上的所有端口，以确定是否有与当前端口不同的端口可用作备用端口。
 * 如果是这样，请复制端口号和dlid到上下文中，以便在加载备用路径时可以使用它们。
 *
 * 注意：
 * 此功能假定，如果发现另一个端口处于活动状态，
 * 该端口与初始端口连接到同一子网，并且
 * 有一条路由到其他主机的备用端口。
 *
 */

int get_alt_port_details(struct context *ctx)
{
	int ret, i;
	struct ibv_qp_attr qp_attr;
	struct ibv_qp_init_attr qp_init_attr;
	struct ibv_device_attr dev_attr;
	/* 本示例假定我们要使用的备用端口在同一HCA上。 
	* 来自其他HCA的端口也可以用作备用路径。 
	* 使用ibv_get_device_list或rdma_get_devices获取设备列表。*/
	ret = ibv_query_device(ctx->id->verbs, &dev_attr);
	if (ret) 
	{
		VERB_ERR("ibv_query_device", ret);
		return ret;
	}
	/* 验证HCA支持APM */
	if (!(dev_attr.device_cap_flags | IBV_DEVICE_AUTO_PATH_MIG))
	{
		printf("device does not support auto path migration!\n");
		return -1;
	}
	/* 查询QP以确定我们绑定到哪个端口 */
	ret = ibv_query_qp(ctx->id->qp, &qp_attr, 0, &qp_init_attr);
	if (ret)
	{
		VERB_ERR("ibv_query_qp", ret);
		return ret;
	}
	for (i = 1; i <= dev_attr.phys_port_cnt; i++)
	{
		/*查询所有端口，直到找到一个处于活动状态的端口，该端口不是我们当前连接的端口 */
		struct ibv_port_attr port_attr;
		ret = ibv_query_port(ctx->id->verbs, i, &port_attr);
		if (ret)
		{
			VERB_ERR("ibv_query_device", ret);
			return ret;
		}
		if (port_attr.state == IBV_PORT_ACTIVE) 
		{
			ctx->my_alt_dlid = port_attr.lid;
			ctx->alt_srcport = i;
			if (qp_attr.port_num != i)
				break;
		}
	}
	return 0;
}


/*
* 函数： load_alt_path
*
* 输入： ctx ，struct context 对象
*
* 输出：无
*
* 返回值：成功返回值0，失败返回值非0
*
* 描述：使用ibv_modify_qp加载备用路径信息并将路径迁移状态设置为rearm。
* 
*/
int load_alt_path(struct context *ctx)
{
	int ret;
	struct ibv_qp_attr qp_attr;
	struct ibv_qp_init_attr qp_init_attr;
	/* query to get the current attributes of the qp */
	ret = ibv_query_qp(ctx->id->qp, &qp_attr, 0, &qp_init_attr);
	if (ret) 
	{
		VERB_ERR("ibv_query_qp", ret);
		return ret;
	}

	/* 使用当前路径属性初始化备用路径属性 */
	memcpy(&qp_attr.alt_ah_attr, &qp_attr.ah_attr, sizeof (struct ibv_ah_attr));
	/* 将备用路径属性设置为一些基本值 */
	qp_attr.alt_pkey_index = qp_attr.pkey_index;
	qp_attr.alt_timeout = qp_attr.timeout;
	qp_attr.path_mig_state = IBV_MIG_REARM;
	/* 如果提供了备用路径，请设置源端口和dlid */
	if (ctx->alt_srcport)
		qp_attr.alt_port_num = ctx->alt_srcport;
	else
		qp_attr.alt_port_num = qp_attr.port_num;
	if (ctx->alt_dlid)
		qp_attr.alt_ah_attr.dlid = ctx->alt_dlid;
	printf("loading alt path - local port： %d, dlid： %d\n",
			qp_attr.alt_port_num, qp_attr.alt_ah_attr.dlid);
	ret = ibv_modify_qp(ctx->id->qp, &qp_attr,IBV_QP_ALT_PATH | IBV_QP_PATH_MIG_STATE);
	if (ret)
	{
		VERB_ERR("ibv_modify_qp", ret);
		return ret;
	}
}

/*
 * 函数： reg_mem
 *
 * 输入：ctx ，struct context 对象
 *
 * 输出：无。
 *
 * 返回值：成功返回值0，失败返回值非0
 *
 * 描述：注册用于我们的数据传输的内区域
 * 
 */
int reg_mem(struct context *ctx)
{
	ctx->send_buf = (char *) malloc(ctx->msg_length);
	memset(ctx->send_buf, 0x12, ctx->msg_length);
	ctx->recv_buf = (char *) malloc(ctx->msg_length);
	memset(ctx->recv_buf, 0x00, ctx->msg_length);
	ctx->send_mr = rdma_reg_msgs(ctx->id, ctx->send_buf, ctx->msg_length);
	if (!ctx->send_mr) 
	{
		VERB_ERR("rdma_reg_msgs", -1);
		return -1;
	}
	ctx->recv_mr = rdma_reg_msgs(ctx->id, ctx->recv_buf, ctx->msg_length);
	if (!ctx->recv_mr) 
	{
		VERB_ERR("rdma_reg_msgs", -1);
		return -1;
	}
	return 0;
}
/*
 * 函数： getaddrinfo_and_create_ep
 *
 * 输入：ctx ，struct context 对象
 *
 * 输出：无
 *
 * 返回值：成功返回值0，失败返回值非0
 *
 * 描述：获取地址信息并创建我们的端点
 * 
 */
int getaddrinfo_and_create_ep(struct context *ctx)
{
	int ret;
	struct rdma_addrinfo *rai, hints;
	struct ibv_qp_init_attr qp_init_attr;
	memset(&hints, 0, sizeof (hints));
	hints.ai_port_space = RDMA_PS_TCP;
	if (ctx->server == 1)
	hints.ai_flags = RAI_PASSIVE; /* 这使得它成为服务端 */
	printf("rdma_getaddrinfo\n");
	ret = rdma_getaddrinfo(ctx->server_name, ctx->server_port, &hints, &rai);
	if (ret)
	{
		VERB_ERR("rdma_getaddrinfo", ret);
		return ret;
	}
	memset(&qp_init_attr, 0, sizeof (qp_init_attr));
	qp_init_attr.cap.max_send_wr = 1;
	qp_init_attr.cap.max_recv_wr = 1;
	qp_init_attr.cap.max_send_sge = 1;
	qp_init_attr.cap.max_recv_sge = 1;
	printf("rdma_create_ep\n");

	ret = rdma_create_ep(&ctx->id, rai, NULL, &qp_init_attr);
	if (ret)
	{
		VERB_ERR("rdma_create_ep", ret);
		return ret;
	}
	rdma_freeaddrinfo(rai);
	return 0;
}

/*
* 函数： get_connect_request
*
* 输入： ctx ，struct context 对象
*
* 输出：无
*
* 返回值：成功返回值0，失败返回值非0
*
* 描述：等待客户端的连接请求
*
*/
int get_connect_request(struct context *ctx)
{
	int ret;
	printf("rdma_listen\n");
	ret = rdma_listen(ctx->id, 4);
	if (ret)
	{
		VERB_ERR("rdma_listen", ret);
		return ret;
	}
	ctx->listen_id = ctx->id;
	printf("rdma_get_request\n");
	ret = rdma_get_request(ctx->listen_id, &ctx->id);
	if (ret)
	{
		VERB_ERR("rdma_get_request", ret);
		return ret;
	}
	if (ctx->id->event->event != RDMA_CM_EVENT_CONNECT_REQUEST)
	{
		printf("unexpected event： %s",
		rdma_event_str(ctx->id->event->event));
		return ret;
	}
	/*如果未在命令行上设置备用路径信息，请从私有数据获取  */
	if (ctx->alt_dlid == 0 && ctx->alt_srcport == 0) 
	{
		ret = get_alt_dlid_from_private_data(ctx->id->event, &ctx->alt_dlid);
		if (ret)
		{
			return ret;
		}
	}
	return 0;
}

/*
* 函数： establish_connection
*
* 输入： ctx ，struct context 对象
*
* 输出：无
*
* 返回值s：成功返回值0，失败返回值非0
*
* 描述：创建连接。 对于客户端，请调用rdma_connect。
* 对于服务器，已经收到连接请求，因此只需执行rdma_accept即可完成连接。
*
*/
int establish_connection(struct context *ctx)
{
	int ret;
	uint16_t private_data;
	struct rdma_conn_param conn_param;
	/* 发布接收以捕获第一个发送 */
	ret = rdma_post_recv(ctx->id, NULL, ctx->recv_buf, ctx->msg_length,
	ctx->recv_mr);
	if (ret)
	{
		VERB_ERR("rdma_post_recv", ret);
		return ret;
	}
	/*在私用数据中发送备用端口的dlid */
	private_data = htons(ctx->my_alt_dlid);
	memset(&conn_param, 0, sizeof (conn_param));
	conn_param.private_data_len = sizeof (int);
	conn_param.private_data = &private_data;
	conn_param.responder_resources = 2;
	conn_param.initiator_depth = 2;
	conn_param.retry_count = 5;
	conn_param.rnr_retry_count = 5;
	if (ctx->server) 
	{
		printf("rdma_accept\n");
		ret = rdma_accept(ctx->id, &conn_param);
		if (ret) 
		{
			VERB_ERR("rdma_accept", ret);
			return ret;
		}
	}
	else
	{
		printf("rdma_connect\n");
		ret = rdma_connect(ctx->id, &conn_param);
		if (ret) 
		{
			VERB_ERR("rdma_connect", ret);
			return ret;
		}
		if (ctx->id->event->event != RDMA_CM_EVENT_ESTABLISHED) 
		{
			printf("unexpected event： %s",
			rdma_event_str(ctx->id->event->event));
			return -1;
		}
		/* 如果未在命令行上设置备用路径信息，请从私有数据获取 */
		if (ctx->alt_dlid == 0 && ctx->alt_srcport == 0) 
		{
			ret = get_alt_dlid_from_private_data(ctx->id->event,
			&ctx->alt_dlid);
			if (ret)
				return ret;
		}
	}
	return 0;
}

/*
* 函数： send_msg
*
* 输入：ctx ，struct context 对象
*
* 输出：无
*
* 返回值：成功返回值0，失败返回值非0
*
* 描述：执行发送并获取完成
* 
*/
int send_msg(struct context *ctx)
{
	int ret;
	struct ibv_wc wc;

	ret = rdma_post_send(ctx->id, NULL, ctx->send_buf, ctx->msg_length,
	ctx->send_mr, IBV_SEND_SIGNALED);
	if (ret)
	{
		VERB_ERR("rdma_send_recv", ret);
		return ret;
	}
	ret = rdma_get_send_comp(ctx->id, &wc);
	if (ret < 0)
	{
		VERB_ERR("rdma_get_send_comp", ret);
		return ret;
	}
	return 0;
}
/*
* 函数： recv_msg
*
* 输入：ctx ，struct context 对象
*
* 输出：无
*
* 返回值：成功返回值0，失败返回值非0
*
* 描述：等待接收完成并发布新的接收缓冲区
* 
*/
int recv_msg(struct context *ctx)
{
	int ret;
	struct ibv_wc wc;
	ret = rdma_get_recv_comp(ctx->id, &wc);
	if (ret < 0) 
	{
		VERB_ERR("rdma_get_recv_comp", ret);
		return ret;
	}
	ret = rdma_post_recv(ctx->id, NULL, ctx->recv_buf, ctx->msg_length,
	ctx->recv_mr);
	if (ret) 
	{
		VERB_ERR("rdma_post_recv", ret);
		return ret;
	}
	return 0;
}
/*
* 函数： main
*
* 输入： ctx ，struct context 对象
*
* 输出：无
*
* 返回值：成功返回值0，失败返回值非0
*
* 描述：
*
*/
int main(int argc, char** argv)
{
	int ret, op, i, send_cnt, recv_cnt;
	struct context ctx;
	struct ibv_qp_attr qp_attr;
	memset(&ctx, 0, sizeof (ctx));
	memset(&qp_attr, 0, sizeof (qp_attr));
	ctx.server = 0;
	ctx.server_port = DEFAULT_PORT;
	ctx.msg_count = DEFAULT_MSG_COUNT;
	ctx.msg_length = DEFAULT_MSG_LENGTH;
	ctx.msec_delay = DEFAULT_MSEC_DELAY;
	ctx.alt_dlid = 0;
	ctx.alt_srcport = 0;
	ctx.migrate_after = -1;
	while ((op = getopt(argc, argv, "sa：p：c：l：d：r：m：")) != -1)
	{
		switch (op)
		{
			case 's'：
				ctx.server = 1;
				break;
			case 'a'：
				ctx.server_name = optarg;
				break;
			case 'p'：
				ctx.server_port = optarg;
				break;
			case 'c'：
				ctx.msg_count = atoi(optarg);
				break;
			case 'l'：
				ctx.msg_length = atoi(optarg);
				break;
			case 'd'：
				ctx.alt_dlid = atoi(optarg);
				break;
			case 'r'：
				ctx.alt_srcport = atoi(optarg);
				break;
			case 'm'：
				ctx.migrate_after = atoi(optarg);
				break;
			case 'w'：
				ctx.msec_delay = atoi(optarg);
				break;
			default：
				printf("usage： %s [-s or -a required]\n", argv[0]);
				printf("\t[-s[erver mode]\n");
				printf("\t[-a ip_address]\n");
				printf("\t[-p port_number]\n");
				printf("\t[-c msg_count]\n");
				printf("\t[-l msg_length]\n");
				printf("\t[-d alt_dlid] (requires -r)\n");
				printf("\t[-r alt_srcport] (requires -d)\n");
				printf("\t[-m num_iterations_then_migrate] (client only)\n");
				printf("\t[-w msec_wait_between_sends]\n");
				exit(1);
		}
	}
	printf("mode： %s\n", (ctx.server) ? "server" ： "client");
	printf("address： %s\n", (!ctx.server_name) ? "NULL" ： ctx.server_name);
	printf("port： %s\n", ctx.server_port);
	printf("count： %d\n", ctx.msg_count);
	printf("length： %d\n", ctx.msg_length);
	printf("alt_dlid： %d\n", ctx.alt_dlid);
	printf("alt_port： %d\n", ctx.alt_srcport);
	printf("mig_after： %d\n", ctx.migrate_after);
	printf("msec_wait： %d\n", ctx.msec_delay);
	printf("\n");
	if (!ctx.server && !ctx.server_name) 
	{
		printf("server address must be specified for client mode\n");
		exit(1);
	}
	/* 这两个都必须设置或都不应该设置 */
	if (!((ctx.alt_dlid > 0 && ctx.alt_srcport > 0) || (ctx.alt_dlid == 0 && ctx.alt_srcport == 0))) 
	{
		printf("-d and -r must be used together\n");
		exit(1);
	}
	if (ctx.migrate_after > ctx.msg_count)
	{
		printf("num_iterations_then_migrate must be less than msg_count\n");
		exit(1);
	}
	ret = getaddrinfo_and_create_ep(&ctx);
	if (ret)
		goto out;
	if (ctx.server)
	{
		ret = get_connect_request(&ctx);
		if (ret)
			goto out;
	}
	/* 仅在未在命令行上指定信息的情况下查询备用端口  */
	if (ctx.alt_dlid == 0 && ctx.alt_srcport == 0) 
	{
		ret = get_alt_port_details(&ctx);
		if (ret)
			goto out;
	}
	/* 创建一个线程来处理异步事件 */
	pthread_create(&ctx.async_event_thread, NULL, async_event_thread, &ctx);
	ret = reg_mem(&ctx);
	if (ret)
		goto out;
	ret = establish_connection(&ctx);
	/* 创建连接后，加载备用路径。 这可以在连接时完成，但是必须使用所有ib动词来创建和建立连接*/
	ret = load_alt_path(&ctx);
	if (ret)
		goto out;
	send_cnt = recv_cnt = 0;
	for (i = 0; i < ctx.msg_count; i++) 
	{
		if (ctx.server) 
		{
			if (recv_msg(&ctx))
				break;
			printf("recv： %d\n", ++recv_cnt);
		}
		if (ctx.msec_delay > 0)
			usleep(ctx.msec_delay * 1000);
		if (send_msg(&ctx))
			break;
		printf("send： %d\n", ++send_cnt);
		if (!ctx.server) 
		{
			if (recv_msg(&ctx))
				break;
			printf("recv： %d\n", ++recv_cnt);
		}

		/* 指定发送的编号之后，如果需要，可以手动迁移路径 */
		if (!ctx.server && i == ctx.migrate_after)
		{
			qp_attr.path_mig_state = IBV_MIG_MIGRATED;
			ret = ibv_modify_qp(ctx.id->qp, &qp_attr, IBV_QP_PATH_MIG_STATE);
			if (ret) 
			{
				VERB_ERR("ibv_modify_qp", ret);
				goto out;
			}
		}
	}
	rdma_disconnect(ctx.id);
out：
	if (ctx.send_mr)
		rdma_dereg_mr(ctx.send_mr);
	if (ctx.recv_mr)
		rdma_dereg_mr(ctx.recv_mr);
	if (ctx.id)
		rdma_destroy_ep(ctx.id);
	if (ctx.listen_id)
		rdma_destroy_ep(ctx.listen_id);
	if (ctx.send_buf)
		free(ctx.send_buf);
	if (ctx.recv_buf)
		free(ctx.recv_buf);
	return ret;
}

```

## 2.2 使用RDMA CM多播
```cpp
/*
* 编译命令：
* gcc mc.c -o mc -libverbs -lrdmacm
*
* 描述：
* 发送方和接收方都将创建UD队列对，并加入指定的多播组（ctx.mcast_addr）。 
* 如果加入成功，则发送者必须创建一个地址句柄（ctx.ah）。 
* 然后，发件人将指定数量的发送（ctx.msg_count）发送到多播组。
* 接收者等待接收每个发送，然后双方离开多播组和清除资源。
* 
*
* 运行示例：
* 该可执行文件可以作为发送方或接收方应用程序运行。 
* 可以在两个节点的简单结构上进行演示，发送者应用程序在一个节点上运行，
* 而接收者应用程序在另一个节点上运行。
* 必须将每个节点配置为支持IPoIB，并且必须为IB接口（例如ib0）分配IP地址。 
* 最后，必须使用OpenSM初始化结构。
*
* 接收方（-m是多播地址，通常是接收方的IP）：
* ./mc -m 192.168.1.12
*
* 发送方（-m是多播地址，通常是接收方的IP）：
* ./mc -s -m 192.168.1.12
*
*/

#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <errno.h>
#include <getopt.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <rdma/rdma_verbs.h>

#define VERB_ERR(verb, ret) fprintf(stderr, "%s returned %d errno %d\n", verb, ret, errno)

/* 默认参数值 */
#define DEFAULT_PORT "51216"
#define DEFAULT_MSG_COUNT 4
#define DEFAULT_MSG_LENGTH 64

/*示例中使用的资源 */
struct context
{
	/* 用户参数 */
	int 						sender;
	char 						*bind_addr;
	char 						*mcast_addr;
	char 						*server_port;
	int 						msg_count;
	int 						msg_length;
	/* 资源 */
	struct sockaddr 			mcast_sockaddr;
	struct rdma_cm_id 			*id;
	struct rdma_event_channel	*channel;
	struct ibv_pd 				*pd;
	struct ibv_cq 				*cq;
	struct ibv_mr 				*mr;
	char *buf;
	struct ibv_ah 				*ah;
	uint32_t 					remote_qpn;
	uint32_t 					remote_qkey;
	pthread_t 					cm_thread;
};


/*
* 函数： cm_thread
*
* 输入：arg，struct context对象
*
* 输出：无
*
* 返回值：NULL
*
* 描述：读取发送数据过程中发生的所有CM事件，并打印出事件的详细信息
*/

static void *cm_thread(void *arg)
{
	struct rdma_cm_event *event;
	int ret;
	struct context *ctx = (struct context *) arg;
	while (1) 
	{
		ret = rdma_get_cm_event(ctx->channel, &event);
		if (ret) 
		{
			VERB_ERR("rdma_get_cm_event", ret);
			break;
		}
		printf("event %s, status %d\n",
		rdma_event_str(event->event), event->status);
		rdma_ack_cm_event(event);
	}
	return NULL;
}
/*
* 函数： get_cm_event
*
* 输入： channel，事件通道
*        type，期待的事件类型 
*
* 输出： out_ev，如果需要，该事件将被传递回调用者。将其设置为NULL，事件将被自动确认，
*                否则，调用者必须使用rdma_ack_cm_event确认该事件。
* 
* 返回值：成功返回0，失败返回非0
*
* 描述：等待下一个CM事件，并检查是否与期望的类型匹配。
*/

int get_cm_event(struct rdma_event_channel *channel, enum rdma_cm_event_type type,
				struct rdma_cm_event **out_ev)
{
	int ret = 0;
	struct rdma_cm_event *event = NULL;
	ret = rdma_get_cm_event(channel, &event);
	if (ret) 
	{
		VERB_ERR("rdma_resolve_addr", ret);
		return -1;
	}
	/* 验证事件是预期的类型*/
	if (event->event != type) 
	{
		printf("event： %s, status： %d\n",
		rdma_event_str(event->event), event->status);
		ret = -1;
	}
	/* 如果需要，将事件传递回用户*/
	if (!out_ev)
		rdma_ack_cm_event(event);
	else
		*out_ev = event;
	return ret;
}
/*
* 函数： resolve_addr
*
* 输入：ctx，strcut context对象
* 
* 输出：无
*
* 返回值：成功返回0，失败返回非0
*
* 描述：解析多播地址，并绑定源地址（如果上下文中提供了）
*/
int resolve_addr(struct context *ctx)
{
	int ret;
	struct rdma_addrinfo *bind_rai = NULL;
	struct rdma_addrinfo *mcast_rai = NULL;
	struct rdma_addrinfo hints;
	memset(&hints, 0, sizeof (hints));
	hints.ai_port_space = RDMA_PS_UDP;
	if (ctx->bind_addr) 
	{
		hints.ai_flags = RAI_PASSIVE;
		ret = rdma_getaddrinfo(ctx->bind_addr, NULL, &hints, &bind_rai);
		if (ret) 
		{
			VERB_ERR("rdma_getaddrinfo (bind)", ret);
			return ret;
		}
	}
	hints.ai_flags = 0;
	ret = rdma_getaddrinfo(ctx->mcast_addr, NULL, &hints, &mcast_rai);
	if (ret)
	{
		VERB_ERR("rdma_getaddrinfo (mcast)", ret);
		return ret;
	}
	if (ctx->bind_addr)
	{
		/*如果需要，请绑定到特定适配器  */
		ret = rdma_bind_addr(ctx->id, bind_rai->ai_src_addr);
		if (ret) 
		{
			VERB_ERR("rdma_bind_addr", ret);
			return ret;
		}
		/*绑定时会创建一个PD。 将其复制到上下文中，以便以后使用 */
		ctx->pd = ctx->id->pd;
	}
	ret = rdma_resolve_addr(ctx->id, (bind_rai) ? bind_rai->ai_src_addr ： NULL,
	mcast_rai->ai_dst_addr, 2000);
	if (ret) 
	{
		VERB_ERR("rdma_resolve_addr", ret);
		return ret;
	}
	ret = get_cm_event(ctx->channel, RDMA_CM_EVENT_ADDR_RESOLVED, NULL);
	if (ret) 
	{
		return ret;
	}
	memcpy(&ctx->mcast_sockaddr,
	mcast_rai->ai_dst_addr,
	sizeof (struct sockaddr));
	return 0;
}

/*
* 函数： create_resources
*
* 输入：ctx，strcut context对象
*
* 输出：无
*
* 返回值：成功返回0，失败返回非0
*
* 描述：创建PD，CQ，QP和MR
* 
*/

int create_resources(struct context *ctx)
{
	int ret, buf_size;
	struct ibv_qp_init_attr attr;
	memset(&attr, 0, sizeof (attr));
	/*如果我们绑定到一个地址，则PD已分配给CM ID */
	if (!ctx->pd)
	{
		ctx->pd = ibv_alloc_pd(ctx->id->verbs);
		if (!ctx->pd) 
		{
			VERB_ERR("ibv_alloc_pd", -1);
			return ret;
		}
	}
	ctx->cq = ibv_create_cq(ctx->id->verbs, 2, 0, 0, 0);
	if (!ctx->cq) 
	{
		VERB_ERR("ibv_create_cq", -1);
		return ret;
	}
	attr.qp_type = IBV_QPT_UD;
	attr.send_cq = ctx->cq;
	attr.recv_cq = ctx->cq;
	attr.cap.max_send_wr = ctx->msg_count;
	attr.cap.max_recv_wr = ctx->msg_count;
	attr.cap.max_send_sge = 1;
	attr.cap.max_recv_sge = 1;
	ret = rdma_create_qp(ctx->id, ctx->pd, &attr);
	if (ret) 
	{
		VERB_ERR("rdma_create_qp", ret);
		return ret;
	}
	/*接收器必须在接收缓冲区中留出足够的空间用于GRH  */

	buf_size = ctx->msg_length + (ctx->sender ? 0 ： sizeof (struct ibv_grh));
	ctx->buf = calloc(1, buf_size);
	memset(ctx->buf, 0x00, buf_size);
	/*注册我们的内存区*/
	ctx->mr = rdma_reg_msgs(ctx->id, ctx->buf, buf_size);
	if (!ctx->mr) 
	{
		VERB_ERR("rdma_reg_msgs", -1);
		return -1;
	}
	return 0;
}
/*
* 函数： destroy_resources
*
* 输入： ctx，strcut context对象
*
* 输出：无
*
* 返回值：成功返回0，失败返回非0
*
* 描述：销毁AH，QP，CQ，MR，PD和ID
* 
*/
void destroy_resources(struct context *ctx)
{
	if (ctx->ah)
		ibv_destroy_ah(ctx->ah);
	if (ctx->id->qp)
		rdma_destroy_qp(ctx->id);
	if (ctx->cq)
		ibv_destroy_cq(ctx->cq);
	if (ctx->mr)
		rdma_dereg_mr(ctx->mr);
	if (ctx->buf)
		free(ctx->buf);
	if (ctx->pd && ctx->id->pd == NULL)
		ibv_dealloc_pd(ctx->pd);
	rdma_destroy_id(ctx->id);
	}
/*

* 函数： post_send
*
* 输入：ctx，strcut context对象
*
* 输出：无
*
* 返回值：成功返回0，失败返回非0
*
* 描述：发布一个UD send到多播地址
*
*/
int post_send(struct context *ctx)
{
	int ret;
	struct ibv_send_wr wr, *bad_wr;
	struct ibv_sge sge;
	memset(ctx->buf, 0x12, ctx->msg_length); /*将数据设置为非零 */
	sge.length = ctx->msg_length;
	sge.lkey = ctx->mr->lkey;
	sge.addr = (uint64_t) ctx->buf;
	/* 组播要求消息与即时数据一起发送，并且QP编号是即时数据的内容 */
	wr.next = NULL;
	wr.sg_list = &sge;
	wr.num_sge = 1;
	wr.opcode = IBV_WR_SEND_WITH_IMM;
	wr.send_flags = IBV_SEND_SIGNALED;
	wr.wr_id = 0;
	wr.imm_data = htonl(ctx->id->qp->qp_num);
	wr.wr.ud.ah = ctx->ah;
	wr.wr.ud.remote_qpn = ctx->remote_qpn;
	wr.wr.ud.remote_qkey = ctx->remote_qkey;
	ret = ibv_post_send(ctx->id->qp, &wr, &bad_wr);
	if (ret) 
	{
		VERB_ERR("ibv_post_send", ret);
		return -1;
	}
return 0;
}
/*
* 函数： get_completion
*
* 输入：ctx，strcut context对象
*

* 输出：无
*
* 返回值：成功返回0，失败返回非0
*
* 描述：等待完成并验证操作是否成功
*/
int get_completion(struct context *ctx)
{
	int ret;
	struct ibv_wc wc;
	do
	{
		ret = ibv_poll_cq(ctx->cq, 1, &wc);
		if (ret < 0)
		{
			VERB_ERR("ibv_poll_cq", ret);
			return -1;
		}
	}
	while (ret == 0);
	if (wc.status != IBV_WC_SUCCESS)
	{
		printf("work completion status %s\n",
		ibv_wc_status_str(wc.status));
		return -1;
	}
	return 0;
}
/*
* 函数： main
*
* 输入：argc ，参数个数
*       argv，命令行参数
*
* 输出：无
*
* 返回值：成功返回0，失败返回非0
*
* 描述：主程序演示多播功能。 
* 发送方和接收方都将创建UD队列对，并加入指定的多播组（ctx.mcast_addr）。 
* 如果加入成功，则发送者必须创建一个地址句柄（ctx.ah）。
*  然后，发送者发布指定数量的发送（ctx.msg_count）到多播组。 
*  接收者等待接收每个发送，然后双方离开多播组和清除资源。
*/

int main(int argc, char** argv)
{
	int ret, op, i;
	struct context ctx;
	struct ibv_port_attr port_attr;
	struct rdma_cm_event *event;
	char buf[40];
	memset(&ctx, 0, sizeof (ctx));
	ctx.sender = 0;
	ctx.msg_count = DEFAULT_MSG_COUNT;
	ctx.msg_length = DEFAULT_MSG_LENGTH;
	ctx.server_port = DEFAULT_PORT;
	// 从命令行读取选项
	while ((op = getopt(argc, argv, "shb：m：p：c：l：")) != -1) 
	{
		switch (op) 
		{
		case 's'：
			ctx.sender = 1;
			break;
		case 'b'：
			ctx.bind_addr = optarg;
			break;
		case 'm'：
			ctx.mcast_addr = optarg;
			break;
		case 'p'：
			ctx.server_port = optarg;
			break;
		case 'c'：
			ctx.msg_count = atoi(optarg);
			break;
		case 'l'：
			ctx.msg_length = atoi(optarg);
			break;
		default：
			printf("usage： %s -m mc_address\n", argv[0]);
			printf("\t[-s[ender mode]\n");
			printf("\t[-b bind_address]\n");
			printf("\t[-p port_number]\n");
			printf("\t[-c msg_count]\n");
			printf("\t[-l msg_length]\n");
			exit(1);
	}
	}
	if(ctx.mcast_addr == NULL) 
	{
		printf("multicast address must be specified with -m\n");
		exit(1);
	}
	ctx.channel = rdma_create_event_channel();
	if (!ctx.channel) 
	{
		VERB_ERR("rdma_create_event_channel", -1);
		exit(1);
	}
	ret = rdma_create_id(ctx.channel, &ctx.id, NULL, RDMA_PS_UDP);
	if (ret) 
	{
		VERB_ERR("rdma_create_id", -1);
		exit(1);
	}
	ret = resolve_addr(&ctx);
	if (ret)
		goto out;
	/*验证缓冲区长度不大于MTU  */
	ret = ibv_query_port(ctx.id->verbs, ctx.id->port_num, &port_attr);
	if (ret) 
	{
		VERB_ERR("ibv_query_port", ret);
		goto out;
	}
	if (ctx.msg_length > (1 << port_attr.active_mtu + 7)) 
	{
		printf("buffer length %d is larger then active mtu %d\n",
		ctx.msg_length, 1 << (port_attr.active_mtu + 7));
		goto out;
	}
	ret = create_resources(&ctx);
	if (ret)
		goto out;
	if (!ctx.sender)
	{
		for (i = 0; i < ctx.msg_count; i++)
		{
			ret = rdma_post_recv(ctx.id, NULL, ctx.buf,ctx.msg_length + sizeof (struct ibv_grh),ctx.mr);
			if (ret)
			{
				VERB_ERR("rdma_post_recv", ret);
				goto out;
			}
		}
	}
	/* 加入多播组 */
	ret = rdma_join_multicast(ctx.id, &ctx.mcast_sockaddr, NULL);
	if (ret)
	{
		VERB_ERR("rdma_join_multicast", ret);
		goto out;
	}
	/* 确认我们已成功加入多播组 */
	ret = get_cm_event(ctx.channel, RDMA_CM_EVENT_MULTICAST_JOIN, &event);
	if (ret)
		goto out;
	inet_ntop(AF_INET6, event->param.ud.ah_attr.grh.dgid.raw, buf, 40);
	printf("joined dgid： %s, mlid 0x%x, sl %d\n", buf,
	event->param.ud.ah_attr.dlid, event->param.ud.ah_attr.sl);
	ctx.remote_qpn = event->param.ud.qp_num;
	ctx.remote_qkey = event->param.ud.qkey;
	if (ctx.sender) 
	{
		/* 为发送者创建地址句柄 */
		ctx.ah = ibv_create_ah(ctx.pd, &event->param.ud.ah_attr);
		if (!ctx.ah) 
		{
			VERB_ERR("ibv_create_ah", -1);
			goto out;
		}
	}
	rdma_ack_cm_event(event);
	/*创建一个线程以在交换消息时处理所有CM事件 */
	pthread_create(&ctx.cm_thread, NULL, cm_thread, &ctx);
	if (!ctx.sender)
	printf("waiting for messages...\n");
	for (i = 0; i < ctx.msg_count; i++)
	{
		if (ctx.sender)
		{
			ret = post_send(&ctx);
			if (ret)
				goto out;
		}
		ret = get_completion(&ctx);
		if (ret)
			goto out;
		if (ctx.sender)
			printf("sent message %d\n", i + 1);
		else
			printf("received message %d\n", i + 1);
	}
out：
	ret = rdma_leave_multicast(ctx.id, &ctx.mcast_sockaddr);
	if (ret)
		VERB_ERR("rdma_leave_multicast", ret);
	destroy_resources(&ctx);
	return ret;
}
```

## 2.3 共享接收队列

```cpp
/*
* 编译命令：
* gcc srq.c -o srq -libverbs -lrdmacm
*
* 描述：
* 客户端和服务器都使用SRQ。 创建了许多队列对（QP）（ctx.qp_count），并且每个QP使用SRQ。 客户端和服务器之间的连接是使用命令行中传递的IP地址的详细信息建立的。 建立连接后，客户端将开始向服务器发送突发消息，并在发送最大工作请求（ctx.max_wr）后停止。 服务器接收到所有发送后，将向客户端执行发送以告知其继续。 重复该过程，直到执行了请求的发送数量（ctx.msg_count）。
*
* 运行示例：
* 该可执行文件可以作为客户端或服务器应用程序运行。 可以在两个节点的简单结构上进行演示，服务器应用程序在一个节点上运行，而客户端应用程序在另一个节点上运行。 必须将每个节点配置为支持IPoIB，并且必须为IB接口（例如ib0）分配IP地址。 最后，必须使用OpenSM初始化结构。
*
* 服务器（-a是本地接口的IP）：
* ./srq -s -a 192.168.1.12
*
* 客户端（-a是远程接口的IP）：
* ./srq -a 192.168.1.12
*
*/
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <errno.h>
#include <getopt.h>
#include <rdma/rdma_verbs.h>

#define VERB_ERR(verb, ret) fprintf(stderr, "%s returned %d errno %d\n", verb, ret, errno)

/* 默认参数值 */
#define DEFAULT_PORT "51216"
#define DEFAULT_MSG_COUNT 100
#define DEFAULT_MSG_LENGTH 100000
#define DEFAULT_QP_COUNT 4
#define DEFAULT_MAX_WR 64

/* 示例中使用的资源 */
struct context
{
	/* 用户参数*/
	int server;
	char *server_name;
	char *server_port;
	int msg_count;
	int msg_length;
	int qp_count;
	int max_wr;
	/*资源 */
	struct rdma_cm_id *srq_id;
	struct rdma_cm_id *listen_id;
	struct rdma_cm_id **conn_id;
	struct ibv_mr *send_mr;
	struct ibv_mr *recv_mr;
	struct ibv_srq *srq;
	struct ibv_cq *srq_cq;
	struct ibv_comp_channel *srq_cq_channel;
	char *send_buf;
	char *recv_buf;
};
/*
* 函数： init_resources
*
* 输入：ctx，struct context对象
*       rai ，连接的RDMA地址信息
*
* 输出：无
*
* 返回值：成功返回0，失败返回非0
*
* 描述：此函数初始化客户端和服务器功能共有的资源。它创建我们的SRQ，注册内存区域，发布接收缓冲区，并创建单个完成队列，该队列将用于每个队列对上的接收队列。
* 
*/

int init_resources(struct context *ctx, struct rdma_addrinfo *rai)
{
	int ret, i;
	struct rdma_cm_id *id;
	/* 创建用于创建/访问我们的SRQ的ID */
	ret = rdma_create_id(NULL, &ctx->srq_id, NULL, RDMA_PS_TCP);
	if (ret)
	{
		VERB_ERR("rdma_create_id", ret);
		return ret;
	}
	/* 我们需要将ID绑定到特定的RDMA设备，这是通过解析地址或绑定到该地址来完成的 */
	if (ctx->server == 0) 
	{
		ret = rdma_resolve_addr(ctx->srq_id, NULL, rai->ai_dst_addr, 1000);
		if (ret) 
		{
			VERB_ERR("rdma_resolve_addr", ret);
			return ret;
		}
	}
	else 
	{
		ret = rdma_bind_addr(ctx->srq_id, rai->ai_src_addr);
		if (ret)
		{
			VERB_ERR("rdma_bind_addr", ret);
			return ret;
		}
	}
	/*创建此示例中使用的内存区域  */
	ctx->recv_mr = rdma_reg_msgs(ctx->srq_id, ctx->recv_buf, ctx->msg_length);
	if (!ctx->recv_mr) 
	{
		VERB_ERR("rdma_reg_msgs", -1);
		return -1;
	}
	ctx->send_mr = rdma_reg_msgs(ctx->srq_id, ctx->send_buf, ctx->msg_length);
	if (!ctx->send_mr)
	{
		VERB_ERR("rdma_reg_msgs", -1);
		return -1;
	}
	/*创建我们的共享接收队列  */
	struct ibv_srq_init_attr srq_attr;
	memset(&srq_attr, 0, sizeof (srq_attr));
	srq_attr.attr.max_wr = ctx->max_wr;
	srq_attr.attr.max_sge = 1;
	ret = rdma_create_srq(ctx->srq_id, NULL, &srq_attr);
	if (ret)
	{
		VERB_ERR("rdma_create_srq", ret);
		return -1;
	}
	/* 在我们的上下文中保存SRQ，以便我们以后可以将其分配给其他QP */
	ctx->srq = ctx->srq_id->srq;
	/* 在SRQ上发布我们的接收缓冲区 */
	for (i = 0; i < ctx->max_wr; i++) 
	{
		ret = rdma_post_recv(ctx->srq_id, NULL, ctx->recv_buf, ctx->msg_length,
		ctx->recv_mr);
		if (ret)
		{
			VERB_ERR("rdma_post_recv", ret);
			return ret;
		}
	}
	/* 创建用于SRQ CQ的完成渠道 */
	ctx->srq_cq_channel = ibv_create_comp_channel(ctx->srq_id->verbs);
	if (!ctx->srq_cq_channel) 
	{
		VERB_ERR("ibv_create_comp_channel", -1);
		return -1;
	}
	/*创建一个CQ以用于所有使用SRQ的连接（QP） */
	ctx->srq_cq = ibv_create_cq(ctx->srq_id->verbs, ctx->max_wr, NULL,
	ctx->srq_cq_channel, 0);
	if (!ctx->srq_cq)
	{
		VERB_ERR("ibv_create_cq", -1);
		return -1;
	}
	/*确保我们在第一次完成时得到通知 */
	ret = ibv_req_notify_cq(ctx->srq_cq, 0);
	if (ret) 
	{
		VERB_ERR("ibv_req_notify_cq", ret);
		return ret;
	}
	return 0;
}
/*
* 函数： destroy_resources
*
* 输入：ctx，struct context对象
*
* 输出：无
*
* 返回值：成功返回0，失败返回非0
*
* 描述：此功能清除应用程序使用的资源
*/
void destroy_resources(struct context *ctx)
{
	int i;
	if (ctx->conn_id) 
	{
		for (i = 0; i < ctx->qp_count; i++)
		{
			if (ctx->conn_id[i]) 
			{
				if (ctx->conn_id[i]->qp && ctx->conn_id[i]->qp->state == IBV_QPS_RTS) 
				{
					rdma_disconnect(ctx->conn_id[i]);
				}
				rdma_destroy_qp(ctx->conn_id[i]);
				rdma_destroy_id(ctx->conn_id[i]);
			}
		}
		free(ctx->conn_id);
	}
	if (ctx->recv_mr)
		rdma_dereg_mr(ctx->recv_mr);
	if (ctx->send_mr)
		rdma_dereg_mr(ctx->send_mr);
	if (ctx->recv_buf)
		free(ctx->recv_buf);
	if (ctx->send_buf)
		free(ctx->send_buf);
	if (ctx->srq_cq)
		ibv_destroy_cq(ctx->srq_cq);
	if (ctx->srq_cq_channel)
		ibv_destroy_comp_channel(ctx->srq_cq_channel);
	if (ctx->srq_id) 
	{
		rdma_destroy_srq(ctx->srq_id);
		rdma_destroy_id(ctx->srq_id);
	}
}
/*
* 函数： await_completion
*
* 输入：ctx，struct context对象
*
* 输出：无
*
* 返回值：成功返回0，失败返回非0
*
* 描述： 等待SRQ CQ完成
*
*/
int await_completion(struct context *ctx)
{
	int ret;
	struct ibv_cq *ev_cq;
	void *ev_ctx;
	/*等待CQ事件到达通道 */
	ret = ibv_get_cq_event(ctx->srq_cq_channel, &ev_cq, &ev_ctx);
	if (ret) 
	{
		VERB_ERR("ibv_get_cq_event", ret);
		return ret;
	}
	ibv_ack_cq_events(ev_cq, 1);
	/*重新加载事件通知 */
	ret = ibv_req_notify_cq(ctx->srq_cq, 0);
	if (ret) 
	{
		VERB_ERR("ibv_req_notify_cq", ret);
		return ret;
	}
	return 0;
}
/*
* 函数： run_server
*
* 输入：ctx，struct context对象
*       rai ，连接的RDMA地址信息
*
* 输出：无
*
* 返回值： 成功返回0，失败返回非0
*
* 描述：执行示例的服务器端
*/

int run_server(struct context *ctx, struct rdma_addrinfo *rai)
{
	int ret, i;
	uint64_t send_count = 0;
	uint64_t recv_count = 0;
	struct ibv_wc wc;
	struct ibv_qp_init_attr qp_attr;
	ret = init_resources(ctx, rai);
	if (ret)
	{
		printf("init_resources returned %d\n", ret);
		return ret;
	}
	/* 由于已经设置srq_id，所以把它作为listen_id */
	ctx->listen_id = ctx->srq_id;
	ret = rdma_listen(ctx->listen_id, 4);
	if (ret)
	{
		VERB_ERR("rdma_listen", ret);
		return ret;
	}
	printf("waiting for connection from client...\n");
	for (i = 0; i < ctx->qp_count; i++)
	{
		ret = rdma_get_request(ctx->listen_id, &ctx->conn_id[i]);
		if (ret)
		{
			VERB_ERR("rdma_get_request", ret);
			return ret;
		}
		/* 创建QP */
		memset(&qp_attr, 0, sizeof (qp_attr));
		qp_attr.qp_context = ctx;
		qp_attr.qp_type = IBV_QPT_RC;
		qp_attr.cap.max_send_wr = ctx->max_wr;
		qp_attr.cap.max_recv_wr = ctx->max_wr;
		qp_attr.cap.max_send_sge = 1;
		qp_attr.cap.max_recv_sge = 1;
		qp_attr.cap.max_inline_data = 0;
		qp_attr.recv_cq = ctx->srq_cq;
		qp_attr.srq = ctx->srq;
		qp_attr.sq_sig_all = 0;
		ret = rdma_create_qp(ctx->conn_id[i], NULL, &qp_attr);
		if (ret)
		{
			VERB_ERR("rdma_create_qp", ret);
			return ret;
		}
		/*设置新连接以使用我们的SRQ */
		ctx->conn_id[i]->srq = ctx->srq;
		ret = rdma_accept(ctx->conn_id[i], NULL);
		if (ret) 
		{
			VERB_ERR("rdma_accept", ret);
			return ret;
		}
	}
	while (recv_count < ctx->msg_count) 
	{
		i = 0;
		while (i < ctx->max_wr && recv_count < ctx->msg_count) 
		{
			int ne;
			ret = await_completion(ctx);
			if (ret) 
			{
				printf("await_completion %d\n", ret);
				return ret;
			}
			do 
			{
				ne = ibv_poll_cq(ctx->srq_cq, 1, &wc);
				if (ne < 0) 
				{
					VERB_ERR("ibv_poll_cq", ne);
					return ne;
				}
				else if (ne == 0)
					break;
				if (wc.status != IBV_WC_SUCCESS)
				{
					printf("work completion status %s\n",
					ibv_wc_status_str(wc.status));
					return -1;
				}
				recv_count++;
				printf("recv count： %d, qp_num： %d\n", recv_count, wc.qp_num);
				ret = rdma_post_recv(ctx->srq_id, (void *) wc.wr_id,
				ctx->recv_buf, ctx->msg_length,
				ctx->recv_mr);
				if (ret)
				{
					VERB_ERR("rdma_post_recv", ret);
					return ret;
				}
				i++;
			}while (ne);
		}
		ret = rdma_post_send(ctx->conn_id[0], NULL, ctx->send_buf,
		ctx->msg_length, ctx->send_mr, IBV_SEND_SIGNALED);
		if (ret) 
		{
			VERB_ERR("rdma_post_send", ret);
			return ret;
		}
		ret = rdma_get_send_comp(ctx->conn_id[0], &wc);
		if (ret <= 0) 
		{
			VERB_ERR("rdma_get_send_comp", ret);
			return -1;
		}
		send_count++;
		printf("send count： %d\n", send_count);
	}
	return 0;
}
/*
* 函数： run_client
*
* 输入：ctx，struct context对象
*       rai ，连接的RDMA地址信息
*
* 输出：无
*
* 返回值： 成功返回0，失败返回非0
*
* 描述：执行示例的客户端
*/
int run_client(struct context *ctx, struct rdma_addrinfo *rai)
{
	int ret, i, ne;
	uint64_t send_count = 0;
	uint64_t recv_count = 0;
	struct ibv_wc wc;
	struct ibv_qp_init_attr attr;
	ret = init_resources(ctx, rai);
	if (ret) 
	{
		printf("init_resources returned %d\n", ret);
		return ret;
	}
	for (i = 0; i < ctx->qp_count; i++) 
	{
		memset(&attr, 0, sizeof (attr));
		attr.qp_context = ctx;
		attr.cap.max_send_wr = ctx->max_wr;
		attr.cap.max_recv_wr = ctx->max_wr;
		attr.cap.max_send_sge = 1;
		attr.cap.max_recv_sge = 1;
		attr.cap.max_inline_data = 0;
		attr.recv_cq = ctx->srq_cq;
		attr.srq = ctx->srq;
		attr.sq_sig_all = 0;
		ret = rdma_create_ep(&ctx->conn_id[i], rai, NULL, &attr);
		if (ret) {
		VERB_ERR("rdma_create_ep", ret);
		return ret;
	}
	ret = rdma_connect(ctx->conn_id[i], NULL);
	if (ret) 
	{
		VERB_ERR("rdma_connect", ret);
		return ret;
	}
	}
	while (send_count < ctx->msg_count) 
	{
		for (i = 0; i < ctx->max_wr && send_count < ctx->msg_count; i++) 
		{
			/* perform our send to the server */
			ret = rdma_post_send(ctx->conn_id[i % ctx->qp_count], NULL,
			ctx->send_buf, ctx->msg_length, ctx->send_mr,
			IBV_SEND_SIGNALED);
			if (ret) 
			{
				VERB_ERR("rdma_post_send", ret);
				return ret;
			}
			ret = rdma_get_send_comp(ctx->conn_id[i % ctx->qp_count], &wc);
			if (ret <= 0)
			{
				VERB_ERR("rdma_get_send_comp", ret);
				return ret;
			}
			send_count++;
			printf("send count： %d, qp_num： %d\n", send_count, wc.qp_num);
		}
		/* wait for a recv indicating that all buffers were processed */
		ret = await_completion(ctx);
		if (ret) 
		{
			VERB_ERR("await_completion", ret);
			return ret;
		}
		do
		{
			ne = ibv_poll_cq(ctx->srq_cq, 1, &wc);
			if (ne < 0)
			{
				VERB_ERR("ibv_poll_cq", ne);
				return ne;
			}
			else if (ne == 0)
				break;
			if (wc.status != IBV_WC_SUCCESS)
			{
				printf("work completion status %s\n",
				ibv_wc_status_str(wc.status));
				return -1;
			}
			recv_count++;
			printf("recv count： %d\n", recv_count);
			ret = rdma_post_recv(ctx->srq_id, (void *) wc.wr_id,
			ctx->recv_buf, ctx->msg_length, ctx->recv_mr);
			if (ret) 
			{
				VERB_ERR("rdma_post_recv", ret);
				return ret;
			}
		}while (ne);
	}
	return ret;
}


/*
* 函数： main
*
* 输入：argc ，参数个数
*       argv，命令行参数
*
* 输出：无
*
* 返回值：成功返回0，失败返回非0
*
* 描述：主程序演示SRQ功能。 客户端和服务器都使用SRQ。 
* 创建了ctx.qp_count个QP，并且每个QP都使用SRQ。 
* 连接之后，客户端将开始向服务器启动ctx.max_wr突发的发送。 
* 服务器接收到所有发送后，将向客户端执行发送以告知其可以继续。 
* 重复执行该过程，直到执行了ctx.msg_count发送为止。
*/
int main(int argc, char** argv)
{
	int ret, op;
	struct context ctx;
	struct rdma_addrinfo *rai, hints;
	memset(&ctx, 0, sizeof (ctx));
	memset(&hints, 0, sizeof (hints));
	ctx.server = 0;
	ctx.server_port = DEFAULT_PORT;
	ctx.msg_count = DEFAULT_MSG_COUNT;
	ctx.msg_length = DEFAULT_MSG_LENGTH;
	ctx.qp_count = DEFAULT_QP_COUNT;
	ctx.max_wr = DEFAULT_MAX_WR;
	/* 从命令行读取选项 */
	while ((op = getopt(argc, argv, "sa：p：c：l：q：w：")) != -1) 
	{
		switch (op) 
		{
			case 's'：
				ctx.server = 1;
				break;
			case 'a'：
				ctx.server_name = optarg;
				break;
			case 'p'：
				ctx.server_port = optarg;
				break;
			case 'c'：
				ctx.msg_count = atoi(optarg);
				break;
			case 'l'：
				ctx.msg_length = atoi(optarg);
				break;
			case 'q'：
				ctx.qp_count = atoi(optarg);
				break;
			case 'w'：
				ctx.max_wr = atoi(optarg);
				break;
			default：
				printf("usage： %s -a server_address\n", argv[0]);
				printf("\t[-s server mode]\n");
				printf("\t[-p port_number]\n");
				printf("\t[-c msg_count]\n");
				printf("\t[-l msg_length]\n");
				printf("\t[-q qp_count]\n");
				printf("\t[-w max_wr]\n");
				exit(1);
		}
	}
	if (ctx.server_name == NULL) 
	{
		printf("server address required (use -a)!\n");
		exit(1);
	}
	hints.ai_port_space = RDMA_PS_TCP;
	if (ctx.server == 1)
		hints.ai_flags = RAI_PASSIVE; /* this makes it a server */
	ret = rdma_getaddrinfo(ctx.server_name, ctx.server_port, &hints, &rai);
	if (ret)
	{
		VERB_ERR("rdma_getaddrinfo", ret);
		exit(1);
	}
	/*为我们的QP和发送/接收缓冲区分配内存 */
	ctx.conn_id = (struct rdma_cm_id **) calloc(ctx.qp_count,sizeof (struct rdma_cm_id *));
	memset(ctx.conn_id, 0, sizeof (ctx.conn_id));
	ctx.send_buf = (char *) malloc(ctx.msg_length);
	memset(ctx.send_buf, 0, ctx.msg_length);
	ctx.recv_buf = (char *) malloc(ctx.msg_length);
	memset(ctx.recv_buf, 0, ctx.msg_length);
	if (ctx.server)
		ret = run_server(&ctx, rai);
	else
		ret = run_client(&ctx, rai);
	destroy_resources(&ctx);
	free(rai);
	return ret;
}
```




------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>[《RDMA Aware Networks Programming User Manual v1.7》](https://www.mellanox.com/related-docs/prod_software/RDMA_Aware_Programming_user_manual.pdf)、[《Dotan Barak的RDMAmojo博客》](http://www.rdmamojo.com/)、《Ubuntu19.10 的rdma-core套件man手册v24.0》，我所做的工作就是更新、整理一部分内容，删除了一部分废弃的API，添加了一部分新的API。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

