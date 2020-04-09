---
title: **getaddrinfo**系列函数
tags: Linux编程手册
---

------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>[《Linux man-pages 4.15》](https://www.kernel.org/doc/man-pages/)。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

# 1 名字
getaddrinfo, freeaddrinfo,gai_strerror——网络地址和服务转换

# 2 概述
```cpp
#include <sys/types.h> 
#include <sys/socket.h>
#include <netdb.h>

int getaddrinfo(const char *node, const char *service, const struct addrinfo *hints,struct addrinfo **res);

void freeaddrinfo(struct addrinfo *res);

const char *gai_strerror(int errcode);
```


glibc的函数测试宏要求(请参阅feature_test_macros(7))

**getaddrinfo**()，**freeaddrinfo**()，**gai_strerror**()：
* 从glibc 2.22开始：\_POSIX_C_SOURCE> = 200112L
* Glibc 2.21和更早版本：\_POSIX_C_SOURCE

# 3 说明 

给定用于标识Internet主机和服务的<u>node</u>和<u>service</u>，**getaddrinfo**()返回一个或多个<u>addrinfo</u>结构体，每个结构体都包含一个可以在**bind**(2)和**connect**(2)中指定的Internet地址。 **getaddrinfo**()函数将**gethostbyname**(3)和**getservbyname**(3)函数提供的功能组合到一个接口中，但是与后面的函数不同，**getaddrinfo**()是可重入的，并允许程序消除IPv4-vs-IPv6依赖性。

**getaddrinfo**()使用的<u>addrinfo</u>结构体包含以下字段：

```cpp
struct addrinfo
{
	int 				ai_flags;
	int 				ai_family;
	int 				ai_socktype;
	int 				ai_protocol;
	socklen_t			ai_addrlen;
	struct sockaddr		*ai_addr;
	char 				*ai_canonname;
	struct addrinfo		*ai_next;
};
```
<u>hints<u>参数指向一个addrinfo结构体，该结构体指定用于选择<u>res<u>指向的列表中返回的套接字地址结构体的条件。 如果<u>hints<u>不是NULL，则指向一个<u>addrinfo<u>结构体，它的<u>ai_family<u>，<u>ai_socktype<u>和<u>ai_protocol<u>指定了限制**getaddrinfo**()返回的套接字地址集的条件，如下所示：

<u>ai_family<u> 该字段为返回的地址指定所需的地址族。 该字段的有效值包括**AF_INET**和**AF_INET6**。 值**AF_UNSPEC**指示**getaddrinfo**()应该返回可与<u>node<u>和<u>service<u>一起使用的任何地址系列(例如，IPv4或IPv6)的套接字地址。


<u>ai_socktype<u> 该字段指定首选的套接字类型，例如**SOCK_STREAM**或**SOCK_DGRAM**。 在该字段中指定0表示**getaddrinfo**()可以返回任何类型的套接字地址。

<u>ai_protocol<u> 该字段指定返回的套接字地址的协议。 在此字段中指定0表示**getaddrinfo**()可以返回任何协议的套接字地址。

<u>ai_flags<u> 此字段指定附加选项，如下所述。 通过对它们进行按位或运算来指定多个标志。

<u>hints<u>所指向的结构体中的所有其他字段必须包含0或空指针(视情况而定)。

将<u>hints<u>指定为NULL等效于将<u>ai_socktype<u>和<u>ai_protocol<u>设置为0； <u>ai_family<u>设置为**AF_UNSPEC**; <u>ai_flags<u>设置为(**AI_V4MAPPED | AI_ADDRCONFIG**)。(POSIX为<u>ai_flags<u>指定了不同的默认值；请参见“注意”。)<u>node<u>指定了数字网络地址(对于IPv4，由inet_aton支持的点分十进制格式；对于IPv6，由inet_pton(3)支持的冒分十六进制格式 )或可以查找并解析其网络地址的网络主机名。 如果<u>hints.ai_flags<u>包含**AI_NUMERICHOST**标志，则<u>node<u>必须是数字网络地址。 **AI_NUMERICHOST**标志禁止任何可能冗长的网络主机地址查找。

如果在<u>hints.ai_flags<u>中指定了**AI_PASSIVE**标志，并且<u>node<u>为NULL，则返回的套接字地址将适用于**bind**(2) 和 **accpet**(2)。 返回的套接字地址将包含“通配符地址”(对于IPv4地址为**INADDR_ANY**，对于IPv6地址为**IN6ADDR_ANY_INIT**)。 通配符地址由打算接受任何主机网络地址上的连接的应用程序(通常是服务器)使用。 如果<u>node<u>不为NULL，则**AI_PASSIVE**标志将被忽略。


如果在<u>hints.ai_flags中<u>未设置**AI_PASSIVE**标志，则返回的套接字地址将适用于**connect**(2)，**sendto**(2)或**sendmsg**(2)。 如果<u>node<u>为NULL，则将网络地址设置为回送接口地址(对于IPv4地址为**INADDR_LOOPBACK**，对于IPv6地址为**IN6ADDR_LOOPBACK_INIT**)； 打算与在同一主机上运行的对等方进行通信的应用程序使用此地址。

<u>service<u>在每个返回的地址结构体中设置端口。 如果此参数是服务名称(请参阅**services**(5))，则将其转换为相应的端口号。 也可以将此参数指定为十进制数，它被简单地转换为二进制数。 如果<u>service<u>为NULL，则返回的套接字地址的端口号将保持未初始化状态。 如果在<u>hints.ai_flags<u>中指定了**AI_NUMERICSERV**，并且<u>service<u>不为NULL，则<u>service<u>必须指向包含数字端口号的字符串。 该标志用于在已知不需要名称解析服务的情况下禁止其调用。

<u>node<u>或<u>service<u>都可以为NULL，但不能同时为NULL。

**getaddrinfo**()函数分配并初始化一个addrinfo结构体的链接列表(其中每个与<u>node<u>和<u>service<u>匹配的网络地址都受<u>hints<u>的施加的任何限制)，然后返回指向列表开头的指针<u>res<u>。 链接列表中的项目通过<u>ai_next<u>字段链接。
  
 链表可能具有多个<u>addrinfo<u>结构体的原因有很多，其中包括：网络主机是地址族的，可以通过多种协议访问(例如，**AF_INET**和**AF_INET6**)； 或者可以从多种套接字类型(例如，一个**SOCK_STREAM**地址和另一个**SOCK_DGRAM**地址)获得相同的服务。 通常，应用程序应尝试按返回地址的顺序使用地址。 **getaddrinfo**()中使用的排序函数在RFC 3484中定义； 可以通过编辑`/etc/gai.conf`(自glibc 2.5起可用)来调整特定系统的顺序。
 
如果<u>hints.ai_flags<u>包含**AI_CANONNAME**标志，则将返回列表中第一个<u>addrinfo<u>结构体的<u>ai_canonname<u>字段设置为指向主机的正式名称。

每个返回的<u>addrinfo<u>结构体的其余字段初始化如下：

* <u>ai_family<u>，<u>ai_socktype<u>和<u>ai_protocol<u>字段返回套接字创建参数(即，这些字段与**socket**(2)的相应参数含义相同)。 例如，<u>ai_family<u>可能返回**AF_INET**或**AF_INET6**。 <u>ai_socktype<u>可能返回**SOCK_DGRAM**或**SOCK_STREAM**； <u>ai_protocol<u>返回套接字的协议。

* 套接字地址的指针放在<u>ai_addr<u>字段中，套接字地址的长度(以字节为单位)放在<u>ai_addrlen<u>字段中。

如果<u>hints.ai_flags<u>包含**AI_ADDRCONFIG**标志，则仅当本地系统配置了至少一个IPv4地址时，才在<u>res<u>指向的列表中返回IPv4地址；仅当本地系统配置了至少一个IPv6地址时，才返回IPv6地址。在这种情况下，回送地址不视为已配置的有效地址。例如，该标志在仅IPv4的系统上很有用，以确保**getaddrinfo**()不会返回在**connect**(2)或**bind**(2)中始终失败的IPv6套接字地址。

如果<u>hints.ai_flags<u>指定**AI_V4MAPPED**标志，并且<u>hints.ai_family<u>被指定为**AF_INET6**，并且找不到匹配的IPv6地址，则在<u>res<u>指向的列表中返回IPv4映射的IPv6地址。如果在<u>hints.ai_flags<u>中同时指定了**AI_V4MAPPED**和**AI_ALL**，则在<u>res<u>指向的列表中返回IPv6地址和IPv4映射的IPv6地址。如果未同时指定**AI_V4MAPPED**，则将忽略**AI_ALL**。

**freeaddrinfo**()函数释放为动态分配的链表res分配的内存。
	   
## 3.1 国际化域名的**getaddrinfo**扩展
从glibc 2.3.4开始，对**getaddrinfo**()进行了扩展，以选择性地允许传入和传出的主机名与国际化域名(IDN)格式进行透明转换(请参阅RFC 3490，应用程序中的国际化域名(IDNA)) 。 定义了四个新标志：
 
 **AI_IND** 
 如果指定了此标志，则在必要时将<u>node<u>中给定的节点名转换为IDN格式。源编码是当前语言环境的编码。

如果输入名称包含非ASCII字符，则使用IDN编码。包含非ASCII字符的节点名称的那些部分(由点分隔)在传递给名称解析函数之前使用ASCII兼容编码(ACE)进行编码。

**AI_CANONIDN**
成功查找名称后，如果指定了**AI_CANONNAME**标志，则**getaddrinfo**()将返回与传递回的<u>addrinfo<u>结构体相对应的节点的正式名称。 返回值是名称解析函数返回的值的准确副本。

如果名称是使用ACE编码的，则名称的一个或多个组成部分将包含xn--前缀。 要将这些组件转换为可读形式，除了**AI_CANONNAME**之外，还可以传递**AI_CANONIDN**标志。 使用当前语言环境的编码对结果字符串进行编码。

**AI_IDN_ALLOW_UNASSIGNED, AI_IDN_USE_STD3_ASCII_RULES**

设置这些标志将启用分别用于IDNA处理的IDNA_ALLOW_UNASSIGNED(允许未分配的Unicode代码点) 和IDNA_USE_STD3_ASCII_RULES(检查输出以确保它是符合STD3的主机名)标志。
 
# 4 返回值
**getaddrinfo** 成功返回0，失败返回错误码。如下：

**EAI_ADDRFAMILY** 指定的网络主机在请求的地址族中没有任何网络地址。

**EAI_AGAIN**名称服务器返回了临时故障指示。 稍后再试。

**EAI_BADFLAGS**  <u>hints.ai_flags<u>包含无效标志； 或者，<u>hints.ai_flags<u>包含**AI_CANONNAME**且<u>name<u>为NULL。

**EAI_FAIL** 名称服务器返回永久故障指示。

**EAI_FAMILY** 不支持请求的地址族。

**EAI_MEMORY**  内存不足。

**EAI_NODATA** 指定的网络主机存在，但没有定义任何网络地址。

**EAI_NONAME** <u>node<u>或<u>service<u>是未知的； 或<u>node<u>和<u>service<u>均为NULL；或在<u>hints.ai_flags<u>中指定了**AI_NUMERICSERV**且<u>service<u>不是数字端口号字符串。

**EAI_SERVICE** 所请求的<u>service<u>对于所请求的套接字类型是不可用的。 对通过另一种套接字类型，它可能是可用的。 例如，如果<u>service<u>为“ shell”(仅在流套接字上可用的服务)，并且<u>hints.ai_protocol<u>为**IPPROTO_UDP**或<u>hints.ai_socktype<u>为**SOCK_DGRAM**，则可能发生此错误。 或者，如果<u>service<u>不为NULL，并且<u>hints.ai_socktype<u>为**SOCK_RAW**(不支持服务概念的套接字类型)，则可能发生错误。

**EAI_SOCKTYPE** 不支持请求的套接字类型。 例如，如果<u>hints.ai_socktype<u>和<u>hints.ai_protocol<u>不一致(例如，分别为**SOCK_DGRAM**和**IPPROTO_TCP**)，就会发生这种情况。

**EAI_SYSTEM**  其他系统错误，请检查errno以获取详细信息。


**gai_strerror**()英语再将错误码转为字符串，适用于错误报告。

# 5 文件
`/etc/gai.conf`

# 6 属性
对本节使用的术语，请看**attributes**(7)。

|接口|属性|值|
|:--|:--|:--|
|**getaddrinfo**()|线程安全|MT_Safe  env local|
|**freeaddrinfo**()<br />**gai_strerror**|线程安全|MT-Safe|

# 7 遵守标准
POSIX.1-2001，POSIX.1-2008。**getaddrinfo**()函数记录在RFC 2553中。

# 8 注意
**getaddrinfo**()支持用于指定IPv6作用域ID的<u>address％scope-id<u>表示法。

自glibc 2.3.3起，**AI_ADDRCONFIG，AI_ALL和AI_V4MAPPED**可用。 自glibc 2.3.4起**AI_NUMERICSERV**可用。

根据POSIX.1，将<u>hints<u>指定为NULL应该会使<u>ai_flags<u>假定为0。在这种情况下，GNU C库假定值为(**AI_V4MAPPED | AI_ADDRCONFIG**)，因为该值被认为是对规范的改进。


# 9 示例
以下程序演示了**getaddrinfo**()，**gai_strerror**()，**freeaddrinfo**()和getnameinfo(3)的用法。 这个程序是UDP数据报的回显服务器和客户端。

服务端
``` cpp
	   #include <sys/types.h>
       #include <stdio.h>
       #include <stdlib.h>
       #include <unistd.h>
       #include <string.h>
       #include <sys/socket.h>
       #include <netdb.h>

       #define BUF_SIZE 500

       int
       main(int argc, char *argv[])
       {
           struct addrinfo hints;
           struct addrinfo *result, *rp;
           int sfd, s;
           struct sockaddr_storage peer_addr;
           socklen_t peer_addr_len;
           ssize_t nread;
           char buf[BUF_SIZE];

           if (argc != 2) {
               fprintf(stderr, "Usage: %s port\n", argv[0]);
               exit(EXIT_FAILURE);
           }

           memset(&hints, 0, sizeof(struct addrinfo));
           hints.ai_family = AF_UNSPEC;    /* Allow IPv4 or IPv6 */
           hints.ai_socktype = SOCK_DGRAM; /* Datagram socket */
           hints.ai_flags = AI_PASSIVE;    /* For wildcard IP address */
           hints.ai_protocol = 0;          /* Any protocol */
           hints.ai_canonname = NULL;
           hints.ai_addr = NULL;
           hints.ai_next = NULL;

           s = **getaddrinfo**(NULL, argv[1], &hints, &result);
           if (s != 0) {
               fprintf(stderr, "**getaddrinfo**: %s\n", **gai_strerror**(s));
               exit(EXIT_FAILURE);
           }

           /* **getaddrinfo**() returns a list of address structures.
              Try each address until we successfully bind(2).
              If socket(2) (or bind(2)) fails, we (close the socket
              and) try the next address. */

           for (rp = result; rp != NULL; rp = rp->ai_next) {
               sfd = socket(rp->ai_family, rp->ai_socktype,
                       rp->ai_protocol);
               if (sfd == -1)
                   continue;
				     if (bind(sfd, rp->ai_addr, rp->ai_addrlen) == 0)
                   break;                  /* Success */

               close(sfd);
           }

           if (rp == NULL) {               /* No address succeeded */
               fprintf(stderr, "Could not bind\n");
               exit(EXIT_FAILURE);
           }

           **freeaddrinfo**(result);           /* No longer needed */

           /* Read datagrams and echo them back to sender */

           for (;;) {
               peer_addr_len = sizeof(struct sockaddr_storage);
               nread = recvfrom(sfd, buf, BUF_SIZE, 0,
                       (struct sockaddr *) &peer_addr, &peer_addr_len);
               if (nread == -1)
                   continue;               /* Ignore failed request */

               char host[NI_MAXHOST], service[NI_MAXSERV];

               s = getnameinfo((struct sockaddr *) &peer_addr,
                               peer_addr_len, host, NI_MAXHOST,
                               service, NI_MAXSERV, NI_NUMERICSERV);
               if (s == 0)
                   printf("Received %zd bytes from %s:%s\n",
                           nread, host, service);
               else
                   fprintf(stderr, "getnameinfo: %s\n", **gai_strerror**(s));

               if (sendto(sfd, buf, nread, 0,
                           (struct sockaddr *) &peer_addr,
                           peer_addr_len) != nread)
                   fprintf(stderr, "Error sending response\n");
           }
       }
```
客户端

``` cpp
       #include <sys/types.h>
       #include <sys/socket.h>
       #include <netdb.h>
       #include <stdio.h>
       #include <stdlib.h>
       #include <unistd.h>
       #include <string.h>

       #define BUF_SIZE 500

       int
       main(int argc, char *argv[])
       {
           struct addrinfo hints;
           struct addrinfo *result, *rp;
           int sfd, s, j;
           size_t len;
           ssize_t nread;
           char buf[BUF_SIZE];

           if (argc < 3) {
               fprintf(stderr, "Usage: %s host port msg...\n", argv[0]);
               exit(EXIT_FAILURE);
           }

           /* Obtain address(es) matching host/port */

           memset(&hints, 0, sizeof(struct addrinfo));
           hints.ai_family = AF_UNSPEC;    /* Allow IPv4 or IPv6 */
           hints.ai_socktype = SOCK_DGRAM; /* Datagram socket */
           hints.ai_flags = 0;
           hints.ai_protocol = 0;          /* Any protocol */

           s = **getaddrinfo**(argv[1], argv[2], &hints, &result);
           if (s != 0) {
               fprintf(stderr, "**getaddrinfo**: %s\n", **gai_strerror**(s));
               exit(EXIT_FAILURE);
           }

           /* **getaddrinfo**() returns a list of address structures.
              Try each address until we successfully connect(2).
              If socket(2) (or connect(2)) fails, we (close the socket
              and) try the next address. */

           for (rp = result; rp != NULL; rp = rp->ai_next) {
               sfd = socket(rp->ai_family, rp->ai_socktype,
                            rp->ai_protocol);
               if (sfd == -1)
                   continue;

               if (connect(sfd, rp->ai_addr, rp->ai_addrlen) != -1)
                   break;                  /* Success */

               close(sfd);
           }

           if (rp == NULL) {               /* No address succeeded */
               fprintf(stderr, "Could not connect\n");
               exit(EXIT_FAILURE);
           }

           **freeaddrinfo**(result);           /* No longer needed */

           /* Send remaining command-line arguments as separate
              datagrams, and read responses from server */

           for (j = 3; j < argc; j<u>) {
               len = strlen(argv[j]) + 1;
                       /* +1 for terminating null byte */

               if (len + 1 > BUF_SIZE) {
                   fprintf(stderr,
                           "Ignoring long message in argument %d\n", j);
                   continue;
               }

               if (write(sfd, argv[j], len) != len) {
                   fprintf(stderr, "partial/failed write\n");
                   exit(EXIT_FAILURE);
               }

               nread = read(sfd, buf, BUF_SIZE);
               if (nread == -1) {
                   perror("read");
                   exit(EXIT_FAILURE);
               }

               printf("Received %zd bytes: %s\n", nread, buf);
           }

           exit(EXIT_SUCCESS);
       }
```

# 10 另请参阅
**getaddrinfo**_a**(3), **gethostbyname**(3), **getnameinfo**(3), **inet**(3), **gai.conf**(5), **hostname**(7), **ip**(7)

# 11 版权

该页面是Linux手册页项目4.15版的一部分。 可以在<https://www.kernel.org/doc/man-pages/>上找到该项目的描述，有关报告错误的信息以及此页面的最新版本。



------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>[《Linux man-pages 4.15》](https://www.kernel.org/doc/man-pages/)。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

