---
title: Linux网络管理工具
tags: Linux使用教程, iproute2, net-tools, ifupdown, netplan
---


------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>《Linux man 手册页》</font>，当前版本为iproute2-ss190708，net-tools 2.10-alpha。<font color=red>未经作者允许</font><font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

# 1 持久配置
Linux网络管理工具按照作用效果可以分为两种
* 一种是临时的管理，重启后就会丢失。例如net-tools和iproute2。.
* 另一种就是持久配置，主要是通过修改或读取`/etc/下面的配置文件来完成，例如ifupdown，ifupdown2和netplan。

这些配置工具按照时间又可以分为过时的和新兴的

* 过时的：ifupdown，ifupdown2，net-tools
* 新兴的：netplan，iproute2
## 1.1 ifupdown和ifupdown2

## 1.2 Netplan

### 1.2.1 netplan命令
**概述**
&emsp;&emsp;`netplan [ COMMAND | help ]`

**说明**

从Ubuntu18.04开始，netplan替代ifupdown成为默认网络配置管理工具。Netplan是用于在Linux系统上轻松配置网络的实用程序。您只需为所需的网络接口以及每个接口应配置的功能创建一个YAML描述文件。Netplan将根据此描述为您选择的渲染器工具生成所有必要的配置。

Netplan读取网络配置`/etc/netplan/*.yaml`，这个文件可以被管理员，安装程序，云映像实例化或其他OS部署编写。在早期启动期间，Netplan在`/run`生成后端特定的配置文件，以将设备的控制权移交给特定的网络守护程序。

Netplan当前可与这些受支持的渲染器一起使用：NetworkManager和Systemd-networkd。

显然，如果没有配置，netplan将无法执行任何操作。最有用的配置代码片段（通过dhcp调出内容）如下：

```yaml
network:
    version: 2
    renderer: NetworkManager
```
这将使NetworkManager管理所有设备（默认情况下，一旦检测到运营商，任何以太网设备都将带有DHCP）。

将networkd用作渲染器不会使设备自动启用DHCP。每个接口都需要在`/etc/netplan`的文件中指定，以便编写其配置并在networkd中使用。

Netplan使用一组子命令来驱动其行为，主要包括：

|子命令|说明|
|:--|:--|
|generate|使用`/etc/netplan/*.yaml`生成渲染器所需的配置。|
|apply|为渲染器应用所有配置，并根据需要重新启动。|
|info :|显示当前netplan版本和可用特性|
|ip|从系统中检索IP信息|
|try|尝试通过自动回滚将新的netplan配置应用于正在运行的系统|
|help|获取帮助信息并退出|

**另请参阅**

netplan-generate(8),  netplan-apply(8),  netplan-try(8),   systemd-networkd(8), NetworkManager(8)

**作者**

Mathieu Trudel-Lapierre   (<cyphermox@ubuntu.com>); Martin Pitt  (<martin.pitt@ubuntu.com>).



### 1.2.2 netplan配置文件
#### 1.2.2.1 介绍

分发安装程序，云实例化，针对特定设备的映像构建，或任何其他部署操作系统的方式，都将其所需的网络配置放入YAML配置文件中。 在早期启动期间，netplan“网络渲染器”运行，该程序读取`/{lib,etc,run}/netplan/*.yaml`并将配置写入`/run`，以将设备的控制权移交给指定的网络守护程序。
* 默认情况下，配置的设备由systemd-networked处理，除非明确标记为由特定渲染（NetworkManager）管理
* 网络配置未涵盖的设备完全不会被管理。
* 在initramfs中可用（很少的依赖，且快速）
* 没有永久生成的配置，只有原始的YAML配置
* 解析器支持多个配置文件，以允许libvirt或lxd之类的应用程序打包预期的网络配置（virbr0，lxdbr0），或更改全局默认策略以对所有网络设备使用NetworkManager。
* 由于生成的配置是短暂的，因此保留了灵活性以后更改后端/策略，或删除NetworkManager。

#### 1.2.2.2 总体结构

netplan的配置文件使用YAML格式。 所有的`/{lib,etc,run}/netplan/*.yaml`都被认为是 netplan的配置文件。 从词法上讲，后面的文件（无论它们在哪个目录中）都会修改（存在新的映射的键）或覆盖（存在相同的映射的键）先前的文件。` /run/netplan`中的文件完全遮盖了`/etc/netplan`中具有相同名称的文件，而这些目录中的任何一个文件都遮盖了`/lib/netplan`中具有相同名称的文件。

netplan配置文件中的顶级节点是`network:`，包含`version：2`（curtin，MaaS等当前使用的YAML是版本1），之后是按照设备类型分组的设备定义，例如：`ethernets:`，`wifis:`或 `bridges:`。 这些是渲染器可以理解的并且受后端支持类型。

每个类型块均含有作为映射的设备定义，其中键（称为“配置ID”）的定义如下。

#### 1.2.2.3 设备配置ID

每个设备类型定义映射下的键名（如ethernets：）称为“ID”。 在整个配置文件集合中，它们必须是唯一的。 它们的主要目的是用作复合设备的锚点名称，例如枚举当前正在定义的网桥的成员。

设备定义在物理上/结构上有两种不同的类别，并且ID字段对每种设备都有不同的解释：

**物理设备**

（例如：ethernet，wifi）这些可以在重新启动之间甚至在运行时（热插拔）动态地变化。 在一般情况下，可以通过有关所需属性的`match:`规则进行选择，例如name/name pattern，MAC地址，驱动程序或设备路径。 通常，这些设备将与任何数量的设备匹配（除非它们引用的属性是唯一的，例如完整路径或MAC地址），因此，如果不进一步了解硬件，这些设备将始终被视为一个组。

完全不指定match规则是合法的，在这种情况下，ID字段只是要匹配的接口名称。 如果您想简化简单场景，并且对网络设备进行长时间的配置，这非常有用。

如果存在`match:`规则，则ID字段是纯不透明的名称，仅用于配置中复合设备定义的引用。





# 2 临时配置
## 2.1 net-tools
## 2.2 iproute2
显示或操作路由、网络设备、接口、隧道。
### 2.1 ip
**概述**
```
ip [ OPTIONS ] OBJECT { COMMAND | help }

ip [ -force ] -batch filename

OBJECT := { link | address | addrlabel | route | rule | neigh | ntable | tunnel | tuntap | maddress | mroute | mrule | monitor | xfrm | netns | l2tp | tcp_metrics | token | macsec }

OPTIONS := { -V[ersion] | -h[uman-readable] | -s[tatistics] | -d[etails] | -r[esolve] | -iec | -f[amily] { inet | inet6 | ipx | dnet | link } | -4 | -6 | -I | -D | -B | -0 | -l[oops] { maximum-addr-flush-attempts } | -o[neline] | -rc[vbuf] [size] | -t[imestamp] | -ts[hort] | -n[etns] name | -a[ll] | -c[olor] -br[ief] }
```
**说明**
|短选项|长选项|说明|
|:--|:--|:--|
|-V|--version|打印ip工具的版本并退出。|
|-h|-human,<br />-human-readable|输出统计信息，其后跟人类可读的值。|
| -b|-batch \<FILENAME>|从提供的文件或标准输入中读取命令并调用它们。第一次失败将导致ip工具终止。|
||-force| 不批处理模式下出错时不要终止IP工具。如果在执行命令期间出现任何错误，则应用程序返回码将为非零。|
|-s|-stats,<br /> -statistics|输出更多信息。如果该选项出现两次或更多次，则信息量会增加。通常，信息是统计信息或一些时间值。|
| -d|-details|输出更多详细信息。|
|-l| -loops \<COUNT>|指定在放弃之前，“ip address flush'”逻辑将尝试的最大循环次数。 默认值为10。零（0）表示循环，直到所有地址被删除。|
|-f|-family \<FAMILY>|指定要使用的协议族。 协议族标识符可以是inet，inet6，bridge，mpls或link中的一种。 如果不存在此选项，则从其他参数中猜测协议族。如果命令行的其余部分没有提供足够的信息来猜测该系列，则ip会退回到默认值，通常是inet或任何其他值。 link是一个特殊的协议族标识符，表示不涉及网络协议|
|-4||等于-family inet
|-6||等于-family inet6
|-B||等于-family bridge
|-M||等于-family mpls
|-0||等于-family link
| -o|-oneline|在单独的一行上输出每个记录，并用'\\'字符替换换行符。当您想使用wc(1)对记录进行计数或对输出进行grep(1)时，这非常方便。
|-r|-resolve|使用系统的域名解析来打印DNS域名而不是主机地址。|
|-n|-netns \<NETNS>|将ip切换到指定的网络名称空间NETNS。实际上，它只是简化了以下操作：<br />`ip netns exec NETNS ip [ OPTIONS ] OBJECT { COMMAND | help }`<br />到<br />`ip -n[etns] NETNS [ OPTIONS ] OBJECT { COMMAND | help }`
| -a|-all|在所有对象上执行指定的命令，这取决于命令是否支持此选项。|
|-c|-color[={always\|auto\|never}]|配置彩色输出。如果省略参数或为always，则不管标准输出状态如何，都启用彩色输出。如果参数为auto，则在启用颜色输出之前，stdout将检查终端。如果参数为never，则禁用彩色输出。如果多次指定，则最后一个优先。如果给出了-json，则忽略此标志。使用的调色板会受到COLORFGBG环境变量的影响（请参阅 ENVIRONMENT）。|
| -t|-timestamp|使用monitor选项时显示当前时间。|
|-ts|-tshort| 与-timestamp类似，但使用较短的格式。|
| -rc|-rcvbuf\<SIZE>|设置netlink套接字接收缓冲区的大小，默认为1MB。|
||-iec|以IEC单位打印人类可读的速率（例如1Ki = 1024）。|
|-br|-brief|仅以表格格式打印基本信息，以提高可读性。当前仅ip addr show和ip link show命令支持此选项。|
|-j| -json|以JavaScript对象表示法（JSON）输出结果。|
|-p |-pretty|默认的JSON格式紧凑且解析效率更高，但大多数用户难以阅读。该标志添加缩进以提高可读性。|
|&emsp;&emsp;&emsp;|

**IP-命令语法**

***OBJECT***

|OBJECT|说明|OBJECT|说明|
|:--|:--|:--|:--|
|address|设备上的协议（IP或IPv6）地址|addrlabel|用于协议地址选择的标签配置|
|l2tp|基于IP的隧道以太网（L2TPv3）|link |网络设备|
|maddress|多播地址|monitor|监视netlink消息|
|mroute|多播路由缓存条目|mrule|多播路由策略数据库中的规则
|neighbour|管理ARP或NDISC缓存条目|netns|管理网络名称空间|
|ntable|管理邻居缓存的操作|route |路由表条目|
| rule|路由策略数据库中的规则|tcp_metrics/tcpmetrics|管理TCP Metrics|
|token|管理令牌化的接口标识符|tunnel|基于IP的隧道|
|tuntap |管理TUN / TAP设备|xfrm|管理IPSec策略|

所有对象的名称都可以全写或缩写形式，例如address可以缩写为addr或a。

***COMMAND***

指定要在OBJECT上执行的操作。 可能的操作集取决于OBJECT类型。 通常，可以add，delete和show（或list）对象，但是某些对象不允许所有这些操作或具有一些其他命令。 help命令适用于所有对象。 它打印出可用命令和参数语法约定的清单。

如果未给出命令，则假定使用一些默认命令。 通常是list，或者，如果无法列出此类的对象，则为help。

**环境变量**：

COLORFGBG：如果设置，则该值用于检测背景是暗还是亮，并使用相反的颜色。COLORFGBG环境变量通常包含两个或三个用分号分隔的值； 无论哪种情况，我们都希望最后一个值。 如果此值为0-6或8，则选择适合深色背景的颜色：`COLORFGBG=";0" ip -c a`

**退出状态**：

如果命令成功，则退出状态为0，如果存在语法错误，则退出状态为1。 如果内核报告错误，则退出状态为2。

**示例**:

`ip addr`
&emsp;&emsp;显示分配给所有网络接口的地址。
`ip neigh`
&emsp;&emsp;显示内核中的当前邻居表。
`ip link set x up`
&emsp;&emsp;打开接口x
`ip link set x down`
&emsp;&emsp;关闭接口x
`ip route`
&emsp;&emsp;显示路由表

**历史**

ip由Alexey N. Kuznetsov编写，并在Linux 2.2中添加。

**另请参阅**

ip-address(8), ip-addrlabel(8), ip-l2tp(8), ip-link(8), ip-maddress(8), ip-monitor(8), ip-mroute(8), ip-neighbour(8), ip-netns(8), ip-ntable(8), ip-route(8), ip-rule(8), ip-tcp_metrics(8), ip-token(8), ip-tunnel(8), ip-xfrm(8)

IP Command reference ip-cref.ps

**报告错误**

将任何错误报告给Network Developers邮件列表<netdev@vger.kernel.org>，主要进行开发和维护。 您不必订阅列表即可在此处发送消息。

**作者**

Michail Litvak的原始联机帮助页<mci@owl.openwall.com>


## 2.2 ip address
**概述**
```
ip [ OPTIONS ] address  { COMMAND | help }

ip address { add | change | replace } IFADDR dev IFNAME [ LIFETIME ] [ CONFFLAG-LIST ]

ip address del IFADDR dev IFNAME [ mngtmpaddr ]

ip address { save | flush } [ dev IFNAME ] [ scope SCOPE-ID ] [ metric METRIC ] [ to PREFIX ] [ FLAG-LIST ] [ label PATTERN ] [ up ]

ip address [ show [ dev IFNAME ] [ scope SCOPE-ID ] [ to PREFIX ] [ FLAG-LIST ] [ label PATTERN ] [ master DEVICE ] [ type TYPE ] [ vrf NAME ] [ up ] ]

ip address { showdump | restore }

IFADDR := PREFIX | ADDR peer PREFIX [ broadcast ADDR ] [ anycast ADDR ] [ label LABEL ] [ scope SCOPE-ID ]

SCOPE-ID := [ host | link | global | NUMBER ]

FLAG-LIST := [ FLAG-LIST ] FLAG

FLAG := [ [-]permanent | [-]dynamic | [-]secondary | [-]primary | [-]tentative | [-]deprecated | [-]dadfailed | [-]temporary | CONFFLAG-LIST ]

CONFFLAG-LIST := [ CONFFLAG-LIST ] CONFFLAG

CONFFLAG := [ home | mngtmpaddr | nodad | noprefixroute | autojoin ]

LIFETIME := [ valid_lft LFT ] [ preferred_lft LFT ]

LFT := [ forever | SECONDS ]

TYPE := [ bridge | bridge_slave | bond | bond_slave | can | dummy | hsr | ifb | ipoib | macvlan | macvtap | vcan | veth | vlan | vxlan | ip6tnl | ipip | sit | gre | gretap |erspan | ip6gre | ip6gretap | ip6erspan | vti | vrf | nlmon | ipvlan | lowpan | geneve | macsec ]
```
**说明**

address是连接到网络设备的协议（IPv4或IPv6）地址。 每个设备必须至少具有一个地址才能使用相应的协议。 一台设备可以连接多个不同的地址。 这些地址没有区别，因此别名一词不太适合它们，我们在本文档中不使用它。

ip address命令显示地址及其属性，添加新地址并删除旧地址。

***ip address add——添加新的协议地址***

|参数|说明|
|:--|:--|
|dev IFNAME|要添加地址的设备的名称。|
| local ADDRESS (default)|接口的地址。地址的格式取决于协议。对于IP，它是一个点分十进制；对于IPv6，它是冒分十六进制。 ADDRESS后面可以跟一个斜线和一个十进制数字，用于编码网络前缀长度。|
| peer ADDRESS|点对点接口的远程端点的地址。同样，ADDRESS后面可以跟一个斜杠和一个十进制数，以编码网络前缀长度。如果指定了对等地址，则本地地址不能具有前缀长度。网络前缀与对等方而不是与本地地址相关联。|
| broadcast ADDRESS|接口上的广播地址。 可以使用特殊符号“ +”和“-”代替广播地址。 在这种情况下，广播地址是通过设置/重置接口前缀的主机位派生出的。|
|label LABEL|每个地址都可以用标签字符串标记。 为了保持与Linux-2.0网络别名的兼容性，此字符串必须与设备名称一致，或者必须在设备名称前加上冒号。 标签的最大总长度为15个字符。|
| scope SCOPE_VALUE|该地址有效的范围。可用范围在文件/etc/iproute2/rt_scopes中列出。预定义的范围值是：<br />global——地址是全局有效的<br />site——（仅IPv6，已弃用）该地址是站点本地地址，即在此站点内有效。<br />link——地址是本地链接，即仅在此设备上有效。<br /> host——地址仅在该主机内部有效|
| metric NUMBER|与地址关联的前缀路由的优先级|
| valid_lft LFT|该地址的有效期限；请参阅RFC 4862的5.5.4节。当它到期时，该地址将被内核删除。默认为永远。|
|preferred_lft LFT|该地址的首选生存时间；请参阅RFC 4862的5.5.4节。到期后，该地址将不再用于新的传出连接。默认为永远。|
|home|（仅IPv6）将此地址指定为RFC 6275中定义的“home address”。|
| mngtmpaddr|（仅IPv6）将此地址作为代表“隐私扩展”（RFC3041），使内核管理从此地址创建的临时地址。为了使其生效，必须将use_tempaddr sysctl设置为大于零的值。给定地址的前缀长度必须为64。此标志允许在手动配置的网络中使用隐私扩展，就像无状态自动配置处于激活状态一样。|
| nodad|（仅IPv6）添加此地址时，不执行重复地址检测（RFC 4862）。|
|noprefixroute|不要为添加的地址的网络前缀自动创建路由，并且不要在删除地址时搜索要删除的路由。更改地址以添加此标志，将删除自动添加的前缀路由，更改地址以删除此标志将自动创建前缀路由。|
|autojoin| 如果连接到执行IGMP侦听的以太网交换机，则无法通过ip maddr命令在以太网级别加入多播组，因为该交换机不会在没有针对多播地址的IGMP报告的端口上复制多播数据包。<br />通过ip link add vxlan创建的Linux VXLAN接口具有组选项，使它们能够执行所需的连接。<br />添加多播地址时使用自动加入标志可以为Openvswitch VXLAN接口以及需要接收多播流量的其他隧道机制启用类似功能。
|&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;|

***ip address delete——删除协议地址***

参数：与ip addr add的参数一致。 设备名称是必填参数。 其余的是可选的。 如果未提供任何参数，则删除第一个地址。

***ip address show——查看协议地址***

|参数|说明|
|:--|:--|
|dev IFNAME (default)|设备名称。|
| scope SCOPE_VAL|仅列出具有此范围的地址。|
|to PREFIX|仅列出与此前缀匹配的地址。|
|label PATTERN|仅列出标签与PATTERN匹配的地址。 PATTERN是通常的shell样式模式。|
|master DEVICE| 仅列出从属与主设备的接口。|
|vrf NAME|仅列出从属此vrf的接口。|
| type TYPE|只列出给定类型的接口。 请注意，不会根据支持的类型列表检查type名称-而是将其原样发送到内核。以后，如果内核尚未过滤它，则通过将其与相关属性进行比较来过滤返回的接口列表。因此，可以接受任何字符串，但可能会导致输出为空。|
|up|仅列出正在运行的接口。|
|dynamic|（仅IPv6）仅列出由于无状态地址配置而安装的地址。
| -dynamic|等于permanen
|permanen |（仅IPv6）仅列出永久（非动态）地址。|
| -permanent|等于dynamic
| tentative|（仅IPv6）仅列出尚未通过重复地址检测的地址。|
| -tentative|（仅IPv6）仅列出当前不在重复地址检测过程中的地址。|
| deprecated|（仅IPv6）仅列出弃用的地址。|
| -deprecated|（仅IPv6）仅列出未弃用的地址。|
| dadfailed|（仅IPv6）仅列出重复地址检测失败的地址。|
| -dadfailed|（仅IPv6）仅列出重复地址检测没有失败的地址。|
| temporary|仅列出临时IPv6。secondary的别名。Linux内核为此共享一个位，因此尽管它们的含义因地址系列而异，但它们实际上是彼此的别名。
|-temporary|primary的别名。
|secondary|仅列出辅助IPv4地址 temporary的别名。Linux内核为此共享一个位，因此尽管它们的含义因地址系列而异，但它们实际上是彼此的别名。
|-secondary|primary的别名
|primary|仅列出主要地址，在IPv6中不包括临时地址。此标志是temporary和 secondary相反符号。
|-primary|temporary和secondary的别名|
|&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;||

***ip address flush——刷新协议地址***

此命令刷新某些条件选择的协议地址。

该命令具有与show相同的参数，只是不支持type和master选择器。另一个区别是，当没有给出参数时，它不会运行。

警告：此命令和其他刷新命令不可逆。他们将残酷地清除所有地址。
  
使用-statistics选项，该命令将变得可见的。它打印出删除的地址数和刷新地址列表的回合数。如果两次给出此选项，则ip address flush也将以前面小节中描述的格式转储所有已删除的地址。

**示例**

`ip address show`
&emsp;&emsp;显示分配给所有网络接口的IPv4和IPv6地址。 'show'子命令可以省略。
`ip address show up`
&emsp;&emsp;除仅显示分配给活动网络接口的地址外，其余与上述相同。
`ip address show dev eth0`
&emsp;&emsp;显示分配给网络接口eth0的IPv4和IPv6地址。
`ip address add 2001:0db8:85a3::0370:7334/64 dev eth1`
&emsp;&emsp;将IPv6地址添加到网络接口eth1。
`ip address delete 2001:0db8:85a3::0370:7334/64 dev eth1`
&emsp;&emsp;删除上面添加的IPv6地址。
`ip address flush dev eth4 scope global`
&emsp;&emsp;从设备eth4删除所有全局IPv4和IPv6地址。 如果没有“ scope global”，它将删除所有地址，包括IPv6本地链接地址。

**另请参阅**

 ip(8)
 
 **作者**
 
 Michail Litvak的原始联机帮助页<mci@owl.openwall.com>
 









------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>《Linux man 手册页》</font>，当前版本为iproute2-ss190708，net-tools 2.10-alpha。<font color=red>未经作者允许</font><font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
