---
title: libmodbus中文手册
tags: modbus
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《libmodbus 3.1.4官方文档》](https://libmodbus.org/docs/v3.1.4/modbus_new_tcp.html)。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

<style>table{word-break:initial;}</style>


# 1 名称
libmodbus是一个快速和可移植的Modbus库。

# 2 概要
```c
//引入头文件
#include<modbus.h>

//在linux中如果使用apt install 安装libmodbus，请使用#include<modbus/modbus.h>

//编译
CC   files -l libmodbus

```

# 3 描述

 libmodbus是一个在设备中使用的遵守modbus协议的发送/接收数据的库。该库包含各种后端，可通过不同的网络进行通信(如RTU模式下的串行或TCP/IPv6模式下的以太网)。<http://www.modbus.org>网站提供了该协议的文件<http://www.modbus.org/specs.php>。

libmodbus提供了较低通信层的抽象，并在所有支持的平台上提供了相同的应用编程接口。

本文档概述了libmodbus的概念，描述了libmodbus如何抽象与不同硬件和平台的modbus通信，并为libmodbus库提供的函数供了参考手册。


## 3.1 环境

Modbus协议包含许多变体（例如，串行RTU或Ehternet TCP），为了简化变体的实现，该库设计为每个变体使用一种后端。这些后端也是一种满足其他要求（例如，实时操作）的便捷方式 每个后端都提供了一个特定的函数来创建一个新的modbus_t环境。 modbus_t环境是一个不透明的结构，包含根据所选变体与其他Modbus设备建立连接的所有必要信息。

您可以根据需要选择最佳环境：

### 3.1.1  RTU环境

RTU（远终端单元）被用于串行通信，其利用数据的紧凑的二进制表示来进行协议通信。 RTU格式遵循带有循环冗余校验校验和的命令/数据，作为错误校验机制，以确保数据的可靠性。Modbus RTU是Modbus最常用的实现方式。 Modbus RTU消息必须连续传输而没有字符间的犹豫（摘自维基百科，Modbus，<http://en.wikipedia.org/wiki/Modbus>（截至2011年3月13日，格林尼治标准时间20:51）)。

Modbus RTU帧调用一个从机(处理Modbus请求的设备/服务)和一个主机(发送请求的客户机)。通信总是由主机发起。

许多Modbus设备可以在同一个物理链接上连接在一起，因此需要用modbus_set_slave(3)定义消息所关心的是从哪个从机。如果正在运行从机，则从机序号用于筛选消息。

RTU的libmodbus实现并不像原始Modbus规范中所述的那样基于时间，而是尽可能快地发送所有字节，当接收到所有预期的字符时，将认为响应或指示已经完成。这个实现提供了非常快速的通信，但是您必须注意设置从机的响应超时小于主机的响应超时(否则，当其中一个从服务器没有响应时，其他从机可能会忽略主机请求)。

#### 3.1.1.1  创建RTU环境

```C
modbus_t *modbus_new_rtu(const char *device, int baud, char parity, int data_bit, int stop_bit);
```

**功能**：为RTU创建一个modbus环境。
**描述**：分配并初始化modbus_t结构，以便在串行线路上以RTU模式进行通信。
* device参数指定OS处理的串行端口的名称，例如` /dev/ttyS0或/dev/ttyUSB0`。 在Windows上，必须在COM编号大于9的情况下为COM名称添加`\\.\` ，例如 `\\\\.\\COM10`。 有关详细信息，请参见<http://msdn.microsoft.com/en-us/library/aa365247(v=vs.85).aspx>
* baud参数指定通信的波特率，例如9600、19200、57600、115200等。
* parity参数设置校验方式，取值如下：
	* N 无校验
	* E 偶校验
	* O 奇校验 
* data_bits参数指定数据位数，允许的值为5,6,7,8。
* stop_bit参数指定停止位数，允许的值为1,2。

**返回值**：成功返回指向modbus_t的指针，失败返回NULL并将errno设置为下面定义的值之一。
**错误**：EINVAL，表示非法参数。
**示例**：
```C
modbus_t *ctx;

ctx = modbus_new_rtu("/dev/ttyUSB0", 115200, 'N', 8, 1);
if (ctx == NULL) {
    fprintf(stderr, "Unable to create the libmodbus context\n");
    return -1;
}
```
#### 3.1.1.2 获取/设置串口模式
```c
int modbus_rtu_get_serial_mode(modbus_t *ctx)
```
**功能**：获取当前串口模型。
**描述**：返回当前libmodbus环境使用的串行模式。
* MODBUS_RTU_RS232
串口设置为RS232通信。 RS-232（推荐标准232）是连接DTE（数据终端设备）和DCE（数据电路终端设备）之间串行二进制单端数据和控制信号的一系列标准的传统名称。 它通常用于计算机串行端口
* MODBUS_RTU_RS485
串口设置为RS485通信。 EIA-485，也称为TIA / EIA-485或RS-485，定义了用于平衡数字多点系统的接收器和驱动器的电气特性的标准。 该标准广泛用于工业自动化中的通信，因为它可以在长距离和电噪声环境中有效地使用。
* 此函数只在Linux内核2.6.28以后可用，并且只能与使用RTU后端的环境一起使用。

**返回值**：成功返回MODBUS_RTU_RS232或MODBUS_RTU_RS485。失败返回-1并将errno设置为下面定义的值之一。
**错误**：EINVAL，表示的相关后端不是RTU。

```c
int modbus_rtu_set_serial_mode(modbus_t *ctx)
```
**功能**：设置串口模型。
**描述**：返回当前libmodbus环境使用的串行模式。此函数只在Linux内核2.6.28以后可用，并且只能与使用RTU后端的环境一起使用。
* MODBUS_RTU_RS232
串口设置为RS232通信。 RS-232（推荐标准232）是连接DTE（数据终端设备）和DCE（数据电路终端设备）之间串行二进制单端数据和控制信号的一系列标准的传统名称。 它通常用于计算机串行端口
* MODBUS_RTU_RS485
串口设置为RS485通信。 EIA-485，也称为TIA / EIA-485或RS-485，定义了用于平衡数字多点系统的接收器和驱动器的电气特性的标准。 该标准广泛用于工业自动化中的通信，因为它可以在长距离和电噪声环境中有效地使用。



**返回值**：成功返回0，失败返回-1并将errno设置为下面定义的值之一。
**错误**：
* EINVAL，表示的那关后端不是RTU。
* ENOTSUP，您的平台不支持该函数。
* 如果调用ioctl()失败，将返回ioctl的错误代码。

```c
int modbus_rtu_get_rts(modbus_t *ctx);
```
**功能**：在RTU 环境中获取当前RTS模式。
**描述**：获取当前modbus环境的RTS（一种流控请求）。此函数只能与使用RTU后端的环境一起使用。可能的返回值是：
* MODBUS_RTU_RTS_NONE
* MODBUS_RTU_RTS_UP
* MODBUS_RTU_RTS_DOWN



**返回值**：成功返回RTS模式，失败返回-1并将errno设置为下面定义的值之一。
**错误**：EINVAL，后端非RTU。

```c
int modbus_rtu_set_rts(modbus_t *ctx, int mode)
```
**功能**：设置RTU 环境中的RTS模式。
**描述**：

* 设置请求发送模式用于在RS485串行总线上进行通讯，默认模式为MODBUS_RTU_RTS_NONE，在把数据写入线路之前不会有信号发出。
* 要启用RTS 模式, 必须使用 MODBUS_RTU_RTS_UP或MODBUS_RTU_RTS_DOWN, 这些模式启用 RTS 模式并同时设置极性。使用MODBUS_RTU_RTS_UP时, RTS 标志位置为使能，并进行一个 ioctl 调用, 然后在1毫秒的延迟后，数据写入总线，然后将 RTS 标志位置为非使能，并进行另一个 ioctl 调用, 并再次延迟1毫秒。MODBUS_RTU_RTS_DOWN模式与之类似, 但使用相反的 RTS 标志位。

**返回值**：成功返回0，失败返回-1并将errno设置为下面定义的值之一。
**错误**：EINVAL，后端非RTU或mode参数非法。
**示例**：

```c
modbus_t *ctx;
uint16_t tab_reg[10];

ctx = modbus_new_rtu("/dev/ttyS0", 115200, 'N', 8, 1);
modbus_set_slave(ctx, 1);
modbus_rtu_set_serial_mode(ctx, MODBUS_RTU_RS485);
modbus_rtu_set_rts(ctx, MODBUS_RTU_RTS_UP);

if (modbus_connect(ctx) == -1) {
    fprintf(stderr, "Connexion failed: %s\n", modbus_strerror(errno));
    modbus_free(ctx);
    return -1;
}

rc = modbus_read_registers(ctx, 0, 7, tab_reg);
if (rc == -1) {
    fprintf(stderr, "%s\n", modbus_strerror(errno));
    return -1;
}

modbus_close(ctx);
modbus_free(ctx);

```


```c
int modbus_rtu_set_custom_rts(modbus_t *ctx, void (set_rts) (modbus_t ctx, int on))
```

**功能**：设置用于自定义RTS实现的函数
**描述**：
* 设置要在传输之前和之后设置RTS引脚时要调用的自定义函数。 默认情况下，它设置为使用ioctl来切换RTS引脚的内部函数，
* 请注意，此功能符合RTS模式，必须使用值MODBUS_RTU_RTS_UP或MODBUS_RTU_RTS_DOWN来调用该功能。
* 此函数只能与使用RTU后端的上下文一起使用。

**返回值**：成功返回0，失败返回-1并将errno设置为下面定义的值之一。
**错误**：EINVAL，后端非RTU


```c
int modbus_rtu_get_rts_delay(modbus_t *ctx);
```


**功能**：获取RTU中的当前RTS延迟。
**描述**：获取lmodbus环境的当前RTS（请求发送）延迟时间。此函数只能与使用RTU后端的环境一起使用。
**返回值**：成功返回以微秒为单位的延迟时间，失败返回-1并将errno设置为下面定义的值之一。
**错误**：EINVAL，后端非RTU

```c
int modbus_rtu_set_rts_delay(modbus_t *ctx, int us);
```


**功能**：设置RTU中得RTS延迟。
**描述**：设置modbus环境中的RTS延迟时间。此函数只能与使用RTU后端的环境一起使用。
**返回值**：成功返回0，失败返回-1并将errno设置为下面定义的值之一。
**错误**：EINVAL，后端非RTU或us为负数。
**示例**：



### 3.1.2  TCP（IPV4）环境

TCP后端实现用于TCP / IPv4网络通信的Modbus变体。 它不需要校验和计算，因为较低层负责相同的操作。

```c
modbus_t *modbus_new_tcp(const char *ip, int port);
```
**功能**：创建一个IPV4的modbus 环境。
**描述**：分配并初始化modbus_t结构，以与Modbus TCP / IPv4服务器通信。
* ip参数指定客户端想要建立连接的服务器的IP地址。
* port参数是要使用的TCP端口。 将端口设置为MODBUS_TCP_DEFAULT_PORT以使用默认端口（502）。 使用大于或等于1024的端口号很方便，因为它不必具有管理员权限。

**返回值**：成功返回指向modbus_t的指针，失败返回NULL并将errno设置为下面定义的值之一。
**错误**：EINVAL，表示非法IP地址。
**示例**：
```C
modbus_t *ctx;

ctx = modbus_new_tcp("127.0.0.1", 1502);
if (ctx == NULL) {
    fprintf(stderr, "Unable to allocate libmodbus context\n");
    return -1;
}

if (modbus_connect(ctx) == -1) {
    fprintf(stderr, "Connection failed: %s\n", modbus_strerror(errno));
    modbus_free(ctx);
    return -1;
}
```
### 3.1.3 TCP PI环境
TCP PI (Protocol Indepedent)后端实现了一个Modbus变体，用于在TCP IPv4和IPv6网络上通信。它不需要校验和计算，因为下层处理相同的问题。

与只使用TCP IPv4的后端相反，TCP PI后端提供主机名解析，但是它消耗大约1Kb的额外内存。
```c
modbus_t *modbus_new_tcp_pi(const char *node, const char *service);
```
**功能**：创建一个TCP Pi的modbus环境。
**描述**：分配和初始化modbus t结构，以便与modbus TCP IPv4或Ipv6服务器通信。

* node参数指定要连接的主机的主机域名或IP地址。
* service参数是要连接的服务名称/端口号。要使用默认Modbus端口，请使用字符串“502”。在许多Unix系统上，使用大于或等于1024的端口号是很方便的，因为不需要有管理员特权。

**返回值**：成功返回指向modbus_t的指针，失败返回NULL并将errno设置为下面定义的值之一。
**错误**：EINVAL，节点字符串为空或已被截断。服务字符串为空或已被截断。
**示例**：

```C
modbus_t *ctx;

ctx = modbus_new_tcp_pi("::1", "1502");
if (ctx == NULL) {
    fprintf(stderr, "Unable to allocate libmodbus context\n");
    return -1;
}

if (modbus_connect(ctx) == -1) {
    fprintf(stderr, "Connection failed: %s\n", modbus_strerror(errno));
    modbus_free(ctx);
    return -1;
}
```

## 3.2 通用
在使用任何libmodbus函数之前，调用者必须使用上面解释的函数分配和初始化modbus环境，然后用以下函数来修改和释放modbus环境：

### 3.2.1 释放modbus环境
```c
void modbus_free(modbus_t *ctx);
```
**功能**：释放modbus环境。
**描述**：释放已分配的modbus_t结构体。
**返回值**：无。

### 3.2.2 设置从机ID
```c
int modbus_set_slave(modbus_t *ctx, int slave);
```

**功能**：在modbus环境中设置从机序号
**描述**：在modbus环境中设置从机序号，其行为取决于网络和设备的角色:
* RTU
定义远程设备的从机ID以在主机模式下进行通信，或在从机模式下设置内部从机ID。 根据协议，Modbus设备只接受持有其从机号或特殊广播号码的消息。
* TCP
如果消息必须到达串行网络上的设备，则在TCP中仅需要从机号码。 可以在TCP模式下使用特殊值MODBUS_TCP_SLAVE（0xFF）来恢复默认值。
*  广播地址为MODBUS_BROADCAST_ADDRESS。 当您希望网络中的所有Modbus设备都收到请求时，必须使用此特殊值。

**返回值**：成功返回0，失败返回-1并将errno设置为下面定义的值之一。
**错误**：EINVAL，从机号非法

**示例**：
```c
modbus_t *ctx;

ctx = modbus_new_rtu("/dev/ttyUSB0", 115200, 'N', 8, 1);
if (ctx == NULL) {
    fprintf(stderr, "Unable to create the libmodbus context\n");
    return -1;
}

rc = modbus_set_slave(ctx, YOUR_DEVICE_ID);
if (rc == -1) {
    fprintf(stderr, "Invalid slave ID\n");
    modbus_free(ctx);
    return -1;
}

if (modbus_connect(ctx) == -1) {
    fprintf(stderr, "Connection failed: %s\n", modbus_strerror(errno));
    modbus_free(ctx);
    return -1;
}
```
### 3.2.3 启用debug模式

```c
void modbus_set_debug(modbus_t *ctx, int flag);
```

**功能**：设置modbus环境的debug标识。
**描述**：使用参数boolean设置modbus t环境的调试标志。当flag设置为TRUE时，许多详细的消息将显示在stdout和stderr上。例如，此标志用于显示Modbus消息的字节。
**返回值**：无


### 3.2.4 超时设置
```c
void modbus_get_byte_timeout(modbus_t *ctx, struct timeval *timeout); //V3.0.6

int modbus_get_byte_timeout(modbus_t *ctx, uint32_t *to_sec, uint32_t *to_usec); //V3.1.4
```
**功能**：获取连续字节间的超时间隔。
**描述**：在to_sec 和 to_usec参数中存储同一消息的两个连续字节之间的超时间隔。
**返回值**：无
**示例**：
```c

//V3.0.6
struct timeval byte_timeout;

/* Save original timeout */
modbus_get_byte_timeout(ctx, &byte_timeout);


//V3.1.4
int32_t to_sec;
uint32_t to_usec;

/* Save original timeout */
modbus_get_byte_timeout(ctx, &to_sec, &to_usec);

```


```c
void modbus_set_byte_timeout(modbus_t *ctx, struct timeval *timeout);//V3.0.6

int modbus_set_byte_timeout(modbus_t *ctx, uint32_t to_sec, uint32_t to_usec); //V3.1.4
```
**功能**：设置连续字节间的超时间隔。
**描述**：
* 设置同一消息的两个连续字节之间的超时间隔。timeout是在select（）函数返回之前所经过的时间量的上限, 如果该时间超过定义的timeout, 则等待响应的函数将引发ETIMEDOUT错误。
* to_usec参数的值必须在范围0到999999之间。
* 如果to_sec和to_usec都为零, 则不会使用此超时设置。在这种情况下, modbus_set_response_timeout ()控制响应的整个处理, 必须在响应超时过期之前接收到完整的确认响应。当设置了字节超时时, 响应超时仅用于等待响应的第一个字节。

**返回值**：如果成功，函数将返回0。否则将返回-1并设置errno。



```c
void modbus_get_response_timeout(modbus_t *ctx, struct timeval *timeout);//v3.0.6

int modbus_get_response_timeout(modbus_t *ctx, uint32_t *to_sec, uint32_t *to_usec);//v3.1.4
```

**功能**：获取响应超时间隔。
**描述**：将用于等待响应的超时间隔存储在timeout中。
**返回值**：如果成功，函数将返回0。否则将返回-1并设置errno。

```c++
//v3.0.6
struct timeval old_response_timeout;
struct timeval response_timeout;

/* Save original timeout */
modbus_get_response_timeout(ctx, &old_response_timeout);

/* Define a new and too short timeout! */
response_timeout.tv_sec = 0;
response_timeout.tv_usec = 0;
modbus_set_response_timeout(ctx, &response_timeout);


//v3.1.4
uint32_t old_response_to_sec;
uint32_t old_response_to_usec;

/* Save original timeout */
modbus_get_response_timeout(ctx, &old_response_to_sec, &old_response_to_usec);

/* Define a new and too short timeout! */
modbus_set_response_timeout(ctx, 0, 0);
```

```c
void modbus_set_response_timeout(modbus_t *ctx, struct timeval *timeout);//v3.0.6

int modbus_set_response_timeout(modbus_t *ctx, uint32_t to_sec, uint32_t to_usec);//v3.1.4
```

**功能**：设置响应超时间隔。
**描述**：设置用于等待响应的超时间隔。 如果在接收响应之前等待的时间超过给定的超时间隔，则会引发错误。
设置了字节超时时，如果第一个响应字节的经过时间长于给定的超时，则等待响应的函数将引发ETIMEDOUT错误。 当禁用字节超时时，必须在响应超时到期之前收到完整的确认响应。to_usec参数的值必须在0到999999之间。
**返回值**：如果成功，函数将返回0。否则将返回-1并设置errno。
**错误**：EINVAL，ctx为null，或者 to_sec和 to_usec 为0, 或者to_usec大于 1000000.
**示例**：
```c++
//v3.0.6
struct timeval old_response_timeout;
struct timeval response_timeout;

/* Save original timeout */
modbus_get_response_timeout(ctx, &old_response_timeout);

/* Define a new and too short timeout! */
response_timeout.tv_sec = 0;
response_timeout.tv_usec = 0;
modbus_set_response_timeout(ctx, &response_timeout);

//v3.1.4
uint32_t old_response_to_sec;
uint32_t old_response_to_usec;

/* Save original timeout */
modbus_get_response_timeout(ctx, &old_response_to_sec, &old_response_to_usec);

/* Define a new timeout of 200ms */
modbus_set_response_timeout(ctx, 0, 200000);
```

### 3.2.5 错误恢复模式

```c
int modbus_set_error_recovery(modbus_t *ctx, modbus_error_recovery_mode error_recovery);
```
**功能**：设置错误恢复模式
**描述**：设置在连接失败或收到非预期字节时的错误恢复模式。 参数error_recovery可以按以下常数中的零个或多个进行按位或运算。
* MODBUS_ERROR_RECOVERY_NONE，默认无错误恢复模式以便 应用程序负责控制libmodbus函数返回的错误值，并在必要时处理它们。
* MODBUS_ERROR_RECOVERY_LINK ， 库将在经过modbus环境中定义的响应超时的延迟后尝试重新连接。 此模式将尝试无限关闭/连接循环，直到发送调用成功，并且将尝试一次在select / read调用上重新建立连接（如果connecton关闭，除了 对于从机/服务器，重新连接后读取的值肯定不再可用，）。 在某些情况下（例如，select调用超时），此模式还将在基于当前响应超时的延迟之后运行刷新请求。如果到远程目标设备的网络关闭，可能会挂起几秒钟。
* MODBUS_ERROR_RECOVERY_PROTOCOL ，睡眠眠和刷新序列将用于清除正在进行的通信，这可能发生在消息长度无效、TID错误或接收的函数代码不是预期的时候。响应超时延迟将用于睡眠。
* 模式是掩码值，因此它们是互补的。
* 建议不要为从站/服务器启用错误恢复。

**返回值**：成功返回0，失败返回-1并将errno设置为下面定义的值之一。
**错误**：EINVAL， error_recovery的值为负。
**示例**：
```c
modbus_set_error_recovery(ctx, MODBUS_ERROR_RECOVERY_LINK | MODBUS_ERROR_RECOVERY_PROTOCOL);
```
### 3.2.6 获取/设置内部socket

```c
int modbus_get_socket(modbus_t *ctx);
```

**功能**：获取环境的当前套接字
**描述**：返回modbus环境的当前套接字或文件描述符。
**返回值**：环境的当前套接字或文件描述符。


```c
void modbus_set_socket(modbus_t *ctx, int socket);//V3.0.6

int modbus_set_socket(modbus_t *ctx, int s);//V3.1.4
```
**功能**：设置modbus环境的socket
**描述**：在modbus环境中设置套接字或文件描述符。 此功能对于管理同一服务器的多个客户端连接非常有用。
**返回值**：如果成功，函数将返回0。否则将返回-1并设置errno。

```c
ctx = modbus_new_tcp("127.0.0.1", 1502);
server_socket = modbus_tcp_listen(ctx, NB_CONNECTION);

FD_ZERO(&rdset);
FD_SET(server_socket, &rdset);

/* .... */

if (FD_ISSET(master_socket, &rdset)) {
    modbus_set_socket(ctx, master_socket);
    rc = modbus_receive(ctx, query);
    if (rc != -1) {
        modbus_reply(ctx, query, rc, mb_mapping);
    }
}
```



### 3.2.7 头部信息

```c
int modbus_get_header_length(modbus_t *ctx);
```

**功能**：获取当前头部长度
**描述**：从后端取得当前头部的长度。此函数便于操作消息，因此仅限于低级操作。
**返回值**：成功返回一个整数值。



### 3.2.8 数据操作宏
```C
MODBUS_GET_HIGH_BYTE(data) //提取一个int16的高位字节


MODBUS_GET_LOW_BYTE(data) //提取一个int16的低位字节

MODBUS_GET_INT64_FROM_INT16(tab_int16, index),//用tab_int16[index]~tab_int16[index+3]构建一个int64 

MODBUS_GET_INT32_FROM_INT16 (tab_int16, index)  //用tab_int16[index]和tab_int16[index+1]构建一个int32 

MODBUS_GET_INT16_FROM_INT8 (tab_int8, index))  //用tab_int8[index]和tab_int8[index+1]构建一个int16

MODBUS_SET_INT16_TO_INT8 tab_int8, index, value) //将一个int16分拆为两个int8

MODBUS_SET_INT32_TO_INT16(tab_int16, index, value)// 将一个int32分拆为两个int16

MODBUS_SET_INT64_TO_INT16(tab_int16, index, value),//将一个int64分拆为四个int16
```



### 3.2.9 处理位和字节
```c
void modbus_set_bits_from_byte(uint8_t *dest, int index, const uint8_t value);
```


**功能**：把一个字节由低位向高位的每一位设置为单独的字节。这些字节的值为0或1。
**描述**：value中由低位向高位，一共8位，从index位置开始写入dest数组中。例如：把0b11010101 分拆为1,0,1,0,1,0,1,1
**返回值**：无返回值。


```c
void modbus_set_bits_from_bytes(uint8_t *dest, int index, unsigned int nb_bits, const uint8_t *tab_byte);
```

**功能**：把一个字节数组的每个字节从前向后，由低位向高位的每一位设置为单独的字节。
**描述**：tab_byte数组从前向后，每个字节由低位向高位 ，一共nb_bits位，从index开始写入dest数组中。例如：把0b11010101 ,0b11111010分拆为1,0,1,0,1,0,1,1,0,1,0,1,1,1,1,1。
**返回值**：无返回值


```c
uint8_t modbus_get_byte_from_bits(const uint8_t *src, int index, unsigned int nb_bits);
```

**功能**：从用一个字节表示一位的数组获得用一个字节表示的值。数组从前向后，从低位向高位。数组元素只能为0或1。
**描述**：从许多位中提取值。 来自src的index处的所有nb_bits位将被读取为单个字节值。 要获得（最大只能获得）一个字节的值，请将nb_bits设置为8。
**返回值**：返回一个包含读取位的字节。

### 3.2.10 设置或获取浮点数

```c
float modbus_get_float_abcd(const uint16_t *src);
```

**功能**：以ABCD字节顺序从2个寄存器（寄存器指的unit16_t）中获取一个浮点值
**描述**： 以通常的Modbus格式从4个字节获得一个浮点数。 src数组必须是两个16位值的指针，例如，如果第一个字设置为0x0020而第二个字设置为0xF147，则浮点值将读为123456.0。
**返回值**：无


```c
void modbus_set_float_abcd(float f, uint16_t *dest);
```

**功能**：使用ABCD字节顺序在两个寄存器中设置浮点值
**描述**： 以通常的Modbus格式将浮点数设置为4字节。dest数组必须是两个16位值上的指针，才能存储转换的完整结果。
**返回值**：无



```c
float modbus_get_float_badc(const uint16_t *src);
```

**功能**：以BADC字节顺序从2个寄存器中获取一个浮点值
**描述**： 的Modbus格式从4个字节获得一个浮点数。 src数组必须是两个16位值的指针，例如，如果第一个字设置为0x2000而第二个字设置为0x47F1，则浮点值将读为123456.0。
**返回值**：无



```c
void modbus_set_float_badc(float f, uint16_t *dest);
```

**功能**：以BADC字节顺序从2个寄存器中获取一个浮点值
**描述**： 以交换字节的Modbus格式将浮点数设置为4字节。dest数组必须是两个16位值上的指针，才能存储转换的完整结果。
**返回值**：无



```c
float modbus_get_float_cdab(const uint16_t *src);
```

**功能**：以CDAB字节顺序从2个寄存器中获取一个浮点值
**描述**： 以交换寄存器顺序的Modbus格式从4个字节获得一个浮点数。 src数组必须是两个16位值的指针，例如，如果第一个字设置为0xF147而第二个字设置为0x0020，则浮点值将读为123456.0。
**返回值**：无



```c
void modbus_set_float_cdab(float f, uint16_t *dest);
```

**功能**：以CDAB字节顺序从2个寄存器中获取一个浮点值
**描述**：  以交换寄存器顺序的Modbus格式将浮点数设置为4字节。dest数组必须是两个16位值上的指针，才能存储转换的完整结果。
**返回值**：无


```c
float modbus_get_float_dcba(const uint16_t *src);
float modbus_get_float(const uint16_t *src);//过时
```

**功能**：以完全逆序的字节顺序从2个寄存器中获取一个浮点值
**描述**：  以完全逆序的字节的Modbus格式从4个字节获得一个浮点数。 src数组必须是两个16位值的指针，例如，如果第一个字设置为0x47F1 而第二个字设置为0x2000，则浮点值将读为123456.0。
**返回值**：无


```c
void modbus_set_float_dcba(float f, uint16_t *dest);
void modbus_set_float(float f, uint16_t *dest);//过时
```
**功能**：使用DCBA字节顺序在2个寄存器中存放一个浮点值
**描述**： 以完全逆序的Modbus格式将浮点数设置为4字节。dest数组必须是两个16位值上的指针，才能存储转换的完整结果。
**返回值**：无



## 3.3 连接
提供的以下函数以建立和关闭与Modbus设备的连接：
###  3.3.1 建立连接
```c
int modbus_connect(modbus_t *ctx);
```

**功能**：建立modbus连接
**描述**：使用参数中给出的modbus环境的上下文信息的网络或总线建立到Modbus主机的连接，
**返回值**：如果成功，函数将返回0。否则，它将返回-1并将errno设置为底层平台的系统调用定义的值之一。
**错误**：
**示例**：
```c
modbus_t *ctx;

ctx = modbus_new_tcp("127.0.0.1", 502);
if (modbus_connect(ctx) == -1) {
    fprintf(stderr, "Connection failed: %s\n", modbus_strerror(errno));
    modbus_free(ctx);
    return -1;
}
```
###  3.3.2 关闭连接
```c
void modbus_close（modbus_t * ctx）;
```

**功能**：关闭Modbus连接
**描述**：关闭与modbus环境中的设置的后端的连接。
**返回值**：无
**示例**：
```c
modbus_t *ctx;

ctx = modbus_new_tcp("127.0.0.1", 502);
if (modbus_connect(ctx) == -1) {
    fprintf(stderr, "Connection failed: %s\n", modbus_strerror(errno));
    modbus_free(ctx);
    return -1;
}

modbus_close(ctx);
modbus_free(ctx);
```

### 3.3.3  刷新连接

```c
int modbus_flush(modbus_t *ctx);

```

**功能**：刷新未传输的数据
**描述**：丢弃接收到了，但未从中读取到与modbus环境ctx中相关联的套接字或文件描述符的数据。
**返回值**：如果成功，函数将返回0或刷新字节数。否则将返回-1并设置errno。

## 3.4 客户端（主机）
Modbus协议定义了不同的数据类型和函数，用于从远程设备读写数据。客户端使用以下函数发送Modbus请求:

### 3.4.1 读数据
```c
int modbus_read_bits(modbus_t *ctx, int addr, int nb, uint8_t *dest);
```

**功能**：读多个位
**描述**：
* 读远程设备的 addr 地址开始的共nb 位（线圈）的状态，读取的结果以无符号的字节 (8 位) 设置为TRUE或FALSE存储在目的数组dest中。
* 您必须注意分配足够的内存以将结果存储在dest位置，至少是nb* sizeof (uint8_t) 的内存大小。
* 该函数使用0x01 modbus 功能码（读取线圈状态）。
* 注：线圈也就是coil，其实就是位，8个线圈就是8位，

**返回值**：成功返回读取位的数目即nb，失败返回-1并设置errno。
**错误**：EMBMDATA，请求太多位。
**示例**：该函数主要用于取得一组逻辑线圈的当前状态（1/0）。

```c
int modbus_read_input_bits(modbus_t *ctx, int addr, int nb, uint8_t *dest);
```

**功能**：读取多个输入位
**描述**：
* 用于读远程设备的 addr 地址开始的共nb 位输入位的状态。读取的结果以无符号的字节 (8 位) 设置为TRUE或FALSE存储在目的数组dest中。
* 您必须注意分配足够的内存以将结果存储在dest位置，至少是nb* sizeof (uint8_t) 的内存大小。
* 该函数使用0x02 modbus 功能码(读取输入状态)。

**返回值**：成功返回读取输入位的数目即nb，失败返回-1并设置errno
**错误**：EMBMDATA，请求太多离散输入位。
**示例**：主要用于取得一组开关输入的当前状态（1/0）。

```c
int modbus_read_registers(modbus_t *ctx, int addr, int nb, uint16_t *dest);
```

**功能**：读取多个保持寄存器
**描述**： 
* 用于读远程设备的 addr 地址开始的共nb 位（保持寄存器）的状态。读取结果以uint (16 位) 的形式存储在dest数组中。
* 您必须注意分配足够的内存以将结果存储在dest位置，至少是nb* sizeof (uint16_t) 的内存大小。
* 该函数使用0x03 功能码 (读取保持寄存器)。

**返回值**：成功返回读取输入位的数目即nb，失败返回-1并设置errno为
**错误**：EMBMDATA，请求太多寄存器。
**示例**：
```c
modbus_t *ctx;
uint16_t tab_reg[64];
int rc;
int i;

ctx = modbus_new_tcp("127.0.0.1", 1502);
if (modbus_connect(ctx) == -1) {
    fprintf(stderr, "Connection failed: %s\n", modbus_strerror(errno));
    modbus_free(ctx);
    return -1;
}

rc = modbus_read_registers(ctx, 0, 10, tab_reg);
if (rc == -1) {
    fprintf(stderr, "%s\n", modbus_strerror(errno));
    return -1;
}

for (i=0; i < rc; i++) {
    printf("reg[%d]=%d (0x%X)\n", i, tab_reg[i], tab_reg[i]);
}

modbus_close(ctx);
modbus_free(ctx);
```
```c
int modbus_read_input_registers(modbus_t *ctx, int addr, int nb, uint16_t *dest);
```

**功能**：读取多个输入寄存器
**描述**： 
* 用于读远程设备的 addr 地址开始的共nb 位（输入寄存器）的状态。读取结果以uint (16 位) 的形式存储在dest数组中。
* 您必须注意分配足够的内存以将结果存储在dest位置，至少是nb* sizeof (uint16_t) 的内存大小。
* 该函数使用0x04 功能码 (读取保持寄存器)。保持寄存器和输入寄存器具有不同的历史意义，但现在更常见的是只使用保持寄存器。

**返回值**：成功返回读取输入位的数目即nb，失败返回-1并设置errno
**错误**：EMBMDATA，请求太多输入寄存器。

```c
int modbus_report_slave_id(modbus_t *ctx, int max_dest, uint8_t *dest);
```

**功能**：返回控制器的描述
**描述**：向控制器发送请求以获取控制器描述。存储在dest中的响应包括：

* 从机（slave）ID ,此ID实际上完全不是唯一的，所以不能靠它来知道信息在响应中如何打包。
* 运行指示器状态（0x00 = OFF, 0xFF = ON）
* 特定于每个控制器的附加数据，例如： libmodbus 以字符串形式返回库的版本号。
* 此函数返回最多max_dest字节数据到dest,所以要确保dest空间足够。
* 该函数使用0x11 功能码 (报告从机ID)。

**返回值**：成功读取数据的数量，如果输出因为max_dest限制而被截断，则返回值会返回若dest空间足够情况下应该会写入到dest的字节数，因此，大于max_dest的返回值意味着响应数据被截断。失败返回-1并设置errno。
**示例**：
```c
uint8_t tab_bytes[MODBUS_MAX_PDU_LENGTH];

...

rc = modbus_report_slave_id(ctx, MODBUS_MAX_PDU_LENGTH, tab_bytes);
if (rc > 1) {
    printf("Run Status Indicator: %s\n", tab_bytes[1] ? "ON" : "OFF");
}
```



### 3.4.2 写数据
```c
int modbus_write_bit(modbus_t *ctx, int addr, int status);
```

**功能**：写一个比特
**描述**：
* 将status的状态写入远程设备addr地址处，值必须是TRUE或者FLASE。 
* 该功能使用Modbus功能代码0x05（写一个线圈）。
* 
**返回值**：如果成功，函数将返回1。否则将返回-1并设置errno。

```c
int modbus_write_register(modbus_t *ctx, int addr, int value);
```

**功能**：写一个寄存器
**描述**：
* 将value的值写入远程设备的地址addr处的保存寄存器。
* 该功能使用Modbus功能代码0x06（写单个保持寄存器）。
* 
**返回值**：成功返回1，失败返回-1并设置errno。

```c
int modbus_write_bits(modbus_t *ctx, int addr, int nb, const uint8_t *src);
```

**功能**：写多个位
**描述**：
* 将src中nb位（线圈）的状态写入远程设备addr地址处，src数组元素的值必须是TRUE或者FLASE。 
* 该函数使用0x0F功能码（强置多线圈)。

**返回值**：成功返回写入位数nb，失败返回-1并设置errno。
**错误**：EMBMDATA，写入太多位。

```c
int modbus_write_registers(modbus_t *ctx, int addr, int nb, const uint16_t *src);
```

**功能**：写多个寄存器。
**描述**：
* 将src中的nb个保持寄存器写入到远程设备addr地址处。
* 该函数使用Modbus功能代码0x10(预先设置多个寄存器)。
* 
**返回值**：成功返回写入寄存器数nb，失败返回-1并设置errno。

### 3.4.3 读写数据
```c
int modbus_write_and_read_registers(modbus_t *ctx, int write_addr, int write_nb, const uint16_t *src, int read_addr, int read_nb, const uint16_t *dest);
```

**功能**：在一次传输中写入和读取多个寄存器。
**描述**：
* 将src中的write_nb个保持寄存器写入到远程设备write_addr地址处，然后从远程设备的read_addr处读取read_nb个保持寄存器到dest数组中。
* 您必须注意分配足够的内存来存储dest(至少nb * sizeof(uint16 t))中的结果。
* 该功能使用Modbus功能代码0x17(写/读寄存器)。

**返回值**：如果成功，函数将返回读寄存器的数目。否则将返回-1并设置errno。
**错误**：EMBMDATA，请求读/写的寄存器太多。

### 3.4.4 原始请求
```c
int modbus_send_raw_request(modbus_t *ctx, uint8_t *raw_req, int raw_req_length);
```

**功能**：发送一个原始请求
**描述**：
* 通过环境ctx的套接字（socket）发送一个请求。此函数只用于调试目的，你必须小心的手动提出有效的请求。此函数只是增加了消息、所选后端的报头或者CRC，因此raw_req的开始必须包含至少一个从站/单元的ID和一个功能代码。此函数可用于发送未由库处理的请求。
* libmodbus 库的公共头部提供了支持的Modbus功能列表以帮助建立原始请求，其前缀为`MODBUS_FC_`, 例如`MODBUS_FC_READ_HOLDING_REGISTERS`。

**返回值**：如果成功，函数将返回完整的消息长度，并计算与后端相关的额外数据。否则将返回-1并设置errno。

**示例**：
```c
modbus_t *ctx;
/* Read 5 holding registers from address 1 */
uint8_t raw_req[] = { 0xFF, MODBUS_FC_READ_HOLDING_REGISTERS, 0x00, 0x01, 0x0, 0x05 };
int req_length;
uint8_t rsp[MODBUS_TCP_MAX_ADU_LENGTH];

ctx = modbus_new_tcp("127.0.0.1", 1502);
if (modbus_connect(ctx) == -1) {
    fprintf(stderr, "Connection failed: %s\n", modbus_strerror(errno));
    modbus_free(ctx);
    return -1;
}

req_length = modbus_send_raw_request(ctx, raw_req, 6 * sizeof(uint8_t));
modbus_receive_confirmation(ctx, rsp);

modbus_close(ctx);
modbus_free(ctx);
```
```c
int modbus_receive_confirmation(modbus_t *ctx, uint8_t *rsp);
```

**功能**：接收确认请求
**描述**：
* 通过环境ctx的套接口（socket）接受请求，此函数只用于调试，因为收到的响应不会根据初试请求进行检查，此函数可用于接受未由库处理的请求。
* 响应的最大大小取决于使用的后端, 在 RTU 中,rsp必须是MODBUS_RTU_MAX_ADU_LENGTH字节, 在 TCP 中必须是MODBUS_TCP_MAX_ADU_LENGTH字节。如果要编写与两者兼容的代码, 可以使用常量MODBUS_MAX_ADU_LENGTH (所有 libmodbus 后台的最大值)。注意分配足够的内存以存储响应以避免服务器崩溃。

**返回值**：将确认请求存储于rsp中，并在成功时返回响应长度，如果指示请求被忽略，返回的请求长度可以是0（例如，在RTU模式下对另一个从机slave的查询）。否则返回-1并设置errno。

**示例**：
```c
uint8_t rsp[MODBUS_MAX_ADU_LENGTH];
rc = modbus_receive_confirmation(ctx, rsp);
### 3.4.5 响应异常

```c
int modbus_reply_exception(modbus_t *ctx, const uint8_t *req, unsigned int exception_code);
```

**功能**：发送一个异常响应
**描述**：发送一个基于exception_code的异常响应。libmodbus 提供了以下异常代码:

* MODBUS_EXCEPTION_ILLEGAL_FUNCTION (1)
* MODBUS_EXCEPTION_ILLEGAL_DATA_ADDRESS (2)
* MODBUS_EXCEPTION_ILLEGAL_DATA_VALUE (3)
* MODBUS_EXCEPTION_SLAVE_OR_SERVER_FAILURE (4)
* MODBUS_EXCEPTION_ACKNOWLEDGE (5)
* MODBUS_EXCEPTION_SLAVE_OR_SERVER_BUSY (6)
* MODBUS_EXCEPTION_NEGATIVE_ACKNOWLEDGE (7)
* MODBUS_EXCEPTION_MEMORY_PARITY (8)
* MODBUS_EXCEPTION_NOT_DEFINED (9)
* MODBUS_EXCEPTION_GATEWAY_PATH (10)
* MODBUS_EXCEPTION_GATEWAY_TARGET (11)

初始请求req是构建有效响应所必需的。

**返回值**：如果成功，函数将返回发送的响应的长度。否则将返回-1并设置errno。
**错误**：EINVAL，异常代码非法。

## 3.5 服务端（从机）
服务端是正在等待来自客户端的请求，并且当它与请求有关时必须回答。 libmodbus提供以下函数来处理请求：
### 3.5.1 数据映射
```c
modbus_mapping_t modbus_mapping_new(int nb_bits, int nb_input_bits, int nb_registers, int nb_input_registers);*
```

**功能**：分配四个元素的数组来存放位和寄存器
**描述**：
* 分配四数组来存储位（线圈）、输入位、（保持）寄存器和输入寄存器。指针存储在 modbus_mapping_t 结构中。数组的所有值都初始化为零。
* 该函数等用于调用 modbus_mapping_new_start_address(3)。该函数的所有起始地址都为0.
* 如果没有必要为特定类型的数据分配数组, 则可以在参数中传递零值, 关联的指针将赋值为 NULL。
* 此函数便于处理在服务器(从机)中的请求。



**返回值**：成功，则返回新分配的结构，否则返回NULL并设置errno
**错误**：ENOMEM,内存不足
**示例**：
```c
/* The first value of each array is accessible from the 0 address. */
mb_mapping = modbus_mapping_new(BITS_ADDRESS + BITS_NB,
                                INPUT_BITS_ADDRESS + INPUT_BITS_NB,
                                REGISTERS_ADDRESS + REGISTERS_NB,
                                INPUT_REGISTERS_ADDRESS + INPUT_REGISTERS_NB);
if (mb_mapping == NULL) {
    fprintf(stderr, "Failed to allocate the mapping: %s\n",
            modbus_strerror(errno));
    modbus_free(ctx);
    return -1;
}
```
```c
void modbus_mapping_free(modbus_mapping_t *mb_mapping);
```

**功能**：释放modbus_mapping_t 结构体
**描述**：释放 mb_mapping_t 结构的4个数组， 和最终由mb_mapping引用的 mb_mapping_t。
**返回值**：无


### 3.5.2 接收
```c
int modbus_receive(modbus_t *ctx, uint8_t *req);
```

**功能**：接收指示请求
**描述**：
* 从环境ctx的socket接收指示请求,该函数由modbus服务端（从机）接收并分析客户端（主机）发送的指示请求。
* 如果需要使用其他套接字或文件描述符, 而不是在ctx环境中定义的, 请参阅函数modbus_set_socket (3).


**返回值**：将指示请求存储到req中，并返回请求长度。如果指示请求被忽略，则返回的请求长度可以为零（例如，在RTU模式下对另一个从设备的查询）。否则返回-1并设置errno。

### 3.5.3 响应
```c
int modbus_reply(modbus_t *ctx, const uint8_t *req, int req_length, modbus_mapping_t *mb_mapping);
```

**功能**：为收到的请求发送一个响应
**描述**：
* 对收到的请求进行响应。分析在参数req给定的请求，然后使用modbus环境ctx的信息建立一个响应并发送。
* 如果请求的指示读取或写入一个值，则会根据操作的数据类型在modbus映射mb_mapping中执行。
* 如果发生错误, 将发送异常响应。
* 此函数是为 "服务器(从机)" 设计的。


**返回值**：成功则返回响应的长度。否则返回-1并设置errno
**错误**：EMBMDATA，发送已失败。此外请参阅用于发送响应的系统调用返回的错误值(例如send 或 write)。



```c
int modbus_reply_exception(modbus_t *ctx, const uint8_t *req, unsigned int exception_code);
```

**功能**：发送一个异常响应
**描述**：发送一个基于exception_code的异常响应。libmodbus 提供了以下异常代码:

* MODBUS_EXCEPTION_ILLEGAL_FUNCTION (1)
* MODBUS_EXCEPTION_ILLEGAL_DATA_ADDRESS (2)
* MODBUS_EXCEPTION_ILLEGAL_DATA_VALUE (3)
* MODBUS_EXCEPTION_SLAVE_OR_SERVER_FAILURE (4)
* MODBUS_EXCEPTION_ACKNOWLEDGE (5)
* MODBUS_EXCEPTION_SLAVE_OR_SERVER_BUSY (6)
* MODBUS_EXCEPTION_NEGATIVE_ACKNOWLEDGE (7)
* MODBUS_EXCEPTION_MEMORY_PARITY (8)
* MODBUS_EXCEPTION_NOT_DEFINED (9)
* MODBUS_EXCEPTION_GATEWAY_PATH (10)
* MODBUS_EXCEPTION_GATEWAY_TARGET (11)

初始请求req是构建有效响应所必需的。

**返回值**：如果成功，函数将返回发送的响应的长度。否则将返回-1并设置errno。
**错误**：EINVAL，异常代码非法。

# 4 错误处理
libmodbus函数使用POSIX系统上的标准约定处理错误。通常，一旦发生故障，libmodbus函数将返回一个NULL值（如果返回一个指针）或一个负值（如果返回一个整数），并且实际的错误代码将被存储在errno变量中。
```c
const char *modbus_strerror(int errnum);
```

**功能**：返回错误消息
**描述**：返回一个指向与errnum参数指定的错误号对应的错误消息字符串的指针。由于 libmodbus 定义了超出操作系统定义的其他错误号, 因此应用程序应该使用modbus_strerror ()而不是标准strerror ()函数来发送错误消息。
**返回值**：返回一个指向错误消息字符串的指针。



# 5 杂项
LIBMODBUS_VERSION_STRING常量表示程序编译时使用的libmodbus版本。

变量libmodbus_version_major、libmodbus_version_minor、libmodbus_version_micro给出程序链接的libmodbus版本。


# 6 作者

libmodbus 文档是由Stéphane Raimbault<stephane.raimbault@gmail.com>编写。

# 7 资源

主页:<http://www.libmodbus.org/>

提交问题<http://github.com/stephane/libmodbus/issues.>

# 8 版权
根据GNU通用公共许可证（LGPLv2.1 +）的条款，免费使用此软件。有关详细信息，请参阅包含在libmodbus发行版中的COPYING.LESSER 。


------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《libmodbus官方文档》](https://libmodbus.org/docs/v3.0.6/modbus_new_tcp.html)。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
