---
title: How to configure Soft-RoCE
tags: RDMA
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文翻译<font color=blue>《[How to Configure Soft-RoCE](https://community.mellanox.com/s/article/howto-configure-soft-roce)》。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

**重要说明**：有关全面的RoCE介绍和部署文章，请参阅[RDMA/RoCE解决方案](https://community.mellanox.com/s/article/rdma-roce-solutions)。

Soft-RoCE是RoCE的软件实现，无论是否提供硬件加速，它都允许RoCE在任何以太网适配器上运行。

Soft-RoCE作为上游内核4.8（或更高版本）的一部分发布。 可以使用Mellanox OFED 4.0或上游驱动程序。 如果安装MLNX_OFED 4.0，则会自动获得Soft-RoCE内核模块和用户空间库。

这篇文章演示了如何安装和设置上游Soft-RoCE（又名RXE），适用于希望通过任何第三方适配器以软件的形式测试RDMA的IT管理员和开发人员。

# 1 参考文献
* [Ethernet Just Got a Big Performance Boost with Release of Soft-RoCE | Mellanox Technologies Blog](https://www.mellanox.com/blog/2015/06/ethernet-just-got-big-performance-boost-with-release-soft-roce/)
* <https://www.openfabrics.org/images/eventpresos/workshops2015/DevWorkshop/Tuesday/tuesday_14.pdf>
* [GitHub - linux-rdma/rdma-core: RDMA core userspace libraries and daemons](https://github.com/linux-rdma/rdma-core)

# 2 概述
Soft-RoCE是RDMA传输的软件实现。 它是一个由Github社区项目开发的项目，主要由IBM，Mellanox和System Fabric Works贡献。 现在，Soft-RoCE可以用于Linux上游提交了。 Soft-RoCE利用与RoCE相同的效率特性，在任何NIC上提供完整的RDMA堆栈实现。

**Soft-RoCE是如何工作的**：Soft-RoCE驱动程序通过Linux网络堆栈实现InfiniBand RDMA传输。 它使具有标准以太网适配器的系统可以与具有硬件RoCE适配器或运行Soft-RoCE的另一个系统进行互操作。

Soft-RoCE模拟Mellanox mlx4硬件RoCE适配器工作，它具有librxe用户空间库（与libmlx4用户空间库相同）和ib_rxe内核模块（与mlx4_ib内核模块相同）。

**现在如何测试Soft-RoCE**：遵循此配置过程 和/或 Github wiki页面。

# 3 架构

![图1](https://raw.githubusercontent.com/liao20081228/blog/master/图片/How_to_configure_Soft-RoCE/图1.jpg)

# 4 设置
CentOS 7/Ubuntu14.04通过以太网交换机或背对背连接两台服务器。

**注意**：不应在服务器上安装MLNX_OFED或任何其他OFED驱动程序。 如果安装了OFED，则应在测试之前将其删除。


# 5 安装
要获得Soft-RoCE功能，您需要在两台服务器上安装内核和用户空间库:

## 5.1 内核安装

1.克隆内核（4.9.0）。运行：
```shell
# cd /usr/src
# git clone git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git linux
```
2.编译内核。运行：
```shell
# cd linux
# cp /boot/config-`uname –r` .config
# make menuconfig
```
将打开下面的窗口:

![图2](https://raw.githubusercontent.com/liao20081228/blog/master/图片/How_to_configure_Soft-RoCE/图2.jpg)


3.要启用Soft-RoCE，请执行以下步骤:
* 按“/”:这将为您打开一个文本字段(搜索)。
* 输入“rxe”和“OK”。
* 按“1”进入“Software RDMA over Ethernet (ROCE) driver”。
* 按“Space”启用“Software RDMA over Ethernet (ROCE) driver”。在这之前应该查看一下“\M\>”。
* 使用键盘右箭头键进入“Save”，然后按“Enter”键。
* 按“Enter”将文件储存为 “.config” 文件。
* 按“Enter”退出。
* 反复使用右箭头进入“Exit”，按“Enter”，直到退出向导。
*
4.编译。运行：
```shell
# make –j $(nproc)
...

# make –j $(nproc) modules_install
...

# make –j $(nproc) install
...
```
**注意**：如果输出包含以下消息“严重致命错误：无法找到合适的模板”，则意味着我们需要编辑/boot/grub/grub.conf文件，并将内核详细信息手动添加到该文件中。

下面是grub.conf文件的一个示例:

```text
# Mellanox Technologies Multiboot GRUB Configuration
default 1
timeout 10
title CentOSreleasex86_64-3.10.0-123.el7.x86_64
root (hd0,0)
kernel /vmlinuz-CentOSreleasex86_64-3.10.0-123.el7.x86_64 root=/dev/sda2 console=tty0 console=ttyS0,115200n8 rhgb
initrd /initramfs-CentOSreleasex86_64-3.10.0-123.el7.x86_64.img
title CentOSreleasex86_64-4.9.0-rxe
root (hd0,0)
kernel /vmlinuz-4.9.0 root=/dev/sda2 console=tty0 console=ttyS0,115200n8 rhgb
initrd /initramfs-4.9.0.img
```
5. 重新启动服务器并再次登录。
6. 确保安装了新的内核和rdma_rxe模块。运行:
```shell
# uname -r
4.9.0
# modinfo rdma_rxe
filename: /lib/modules/4.9.0/kernel/drivers/infiniband/sw/rxe/rdma_rxe.ko
version: 0.2
license: Dual BSD/GPL
description: Soft RDMA transport
author: Bob Pearson, Frank Zago, John Groves, Kamal Heib
srcversion: D0E124BDAB1CF1F18E7DC56
depends: ib_core,ip6_udp_tunnel,udp_tunnel
vermagic: 4.9.0 SMP mod_unload modversions
parm: add:Create RXE device over network interface
parm: remove:Remove RXE device over network interface
```
## 5.2 用户空间库安装
支持Soft-RoCE的用户空间库还没有发布。这意味着您必须获取源代码并手动构建和安装它们。
1.克隆用户空间库。运行:
```shell
# git clone https://github.com/linux-rdma/rdma-core.git
```
2.编译和安装用户空间库。运行:
```shell
# cd rdma_core
# bash build.sh
...
# cd build ; sudo make install
```
# 6 设置RXE

rxe_cfg脚本负责启动，停止和配置RXE。

它加载并删除RXE内核模块（rxe_rdma），并将其与首选的以太网接口配对。

rxe_cfg位于../rdma-core/providers/rxe(当前脚本没有将它们添加到路径中)。

1.获取状态。运行：
```shell
# rxe_cfg status

rxe modules not loaded
Name Link Driver Speed NMTU IPv4_addr RDEV RMTU
enp5s0 yes e1000e
enp6s0 no e1000e
```
我们可以看到，RXE接口可以与两个潜在的以太网接口配对：**enp5s0**和**enp6s0**。

**注意**:此时还没有加载RXE内核模块。

在本例中，e1000e是一个Intel驱动程序，enp5s0是服务器上的1GbE端口。

要查看此命令的其他选项，请运行:
```shell
# rxe_cfg help

Usage:
rxe_cfg [options] start|stop|status|persistent
rxe_cfg debug on|off|<num>
rxe_cfg [-n] add <ndev>
rxe_cfg [-n] remove <ndev>|<rdev>
<ndev> = network device e.g. eth3
<rdev> = rdma device e.g. rxe1

Options:
-h: print this usage information
-n: do not make the configuration action persistent
-v: print additional debug output
-l: show status for interfaces with link up
-p <num>: (start command only) - set ethertype
```
2.启动RXE。运行：
```shell
# sudo rxe_cfg start
Name Link Driver Speed NMTU IPv4_addr RDEV RMTU
enp5s0 yes e1000e
enp6s0 no e1000e
```
要验证RXE内核模块是否已加载，请运行:
```shell
# lsmod |grep rdma_rxe

rdma_rxe 114688 0
ip6_udp_tunnel 16384 1 rdma_rxe
udp_tunnel 16384 1 rdma_rxe
ib_core 208896 9 rdma_rxe,ib_cm,rdma_cm,ib_umad,ib_uverbs,ib_ipoib,iw_cm,ib_ucm,rdma_ucm
```
3.通过将一个RXE与一个以太网接口配对来创建一个新的RXE设备/接口。运行:在本例中，我们使用**enp5s0**。
```shell
# sudo rxe_cfg add enp5s0
```
4.检查rxe_cfg的状态，确保在RDEV (rxe设备)下添加了rxe0。还可以检查ibv_devices命令。
```shell# rxe_cfg
Name Link Driver Speed NMTU IPv4_addr RDEV RMTU
enp5s0 yes e1000e rxe0
enp6s0 no e1000e

# ibv_devices
device node GUID

------ ----------------
rxe0 022590fffe8321de
```

5.测试连接。

* On the server:
```shell
# ibv_rc_pingpong -d rxe0 -g 0
```
* On the client:
```shell
# ibv_rc_pingpong -d rxe0 -g 0 <server_management_ip>
```








------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文翻译<font color=blue>《[How to Configure Soft-RoCE](https://community.mellanox.com/s/article/howto-configure-soft-roce)》。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
