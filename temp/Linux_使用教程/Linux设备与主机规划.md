---
title:  Linux设备与主机规划
tags: Linux使用教程
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《鸟哥的Linux私房菜》](http://linux.vbird.org "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------



# 1 设备与文件名的关系
&emsp;&emsp;各组件或设备在Linux都是抽象成一个文件。

&emsp;&emsp;各设备在Linux中的文件名：
|设备|文件名|
|:--|:--|
|SCSI/SATA/USB硬盘| /dev/sd[a-p]
|U盘 |/dev/sd[a-p] (与 SATA 相同)
|VirtI/O 接口| /dev/vd[a-p] (用于虚拟机内)
|软盘驱动器| /dev/fd[0-7]
|打印机|/dev/lp[0-2] (25针)、/dev/usb/lp[0-15] (USB)
|鼠标|/dev/input/mouse[0-15] (通用)、/dev/psaux (PS/2)、/dev/mouse(当前鼠标)
|CD/DVD ROM|/dev/scd[0-1] (通用)、、/dev/sr[0-1] (通用，CentOS 较常见)、/dev/cdrom (当前 CDROM)
|磁带机|/dev/ht0 (IDE)、/dev/st0 (SATA/SCSI)、/dev/tape (当前磁带)
|IDE|硬盘机/dev/hd[a-d] (旧式系统才有）
&emsp;&emsp;由于 IDE界面的磁盘驱动器几乎已经被淘汰，太少见了！因此现在连 IDE 界面的磁盘文件名也都被仿真成/dev/sd[a-p]了！此外，如果你的机器使用的是跟因特网供货商(ISP)申请使用的云端机器，这时可能会得到的是虚拟机。为了加速，虚拟机内的磁盘是使用仿真器产生， 该仿真器产生的磁盘文件名为 /dev/vd[a-p] 系列的文件名。  　

# 2 主机规划
* 服务规划
   * NAT（网络地址转换）：将专用网中内部网址与转为因特网上的网址。
   * SAMBA ：加入windows 的网上邻居
   * MAil
   * web
   * DHCP
   * Proxy
   * FTp
*  磁盘规划
	* 自定义安装
		* 初次接触 Linux：只要分区/ 及swap即可。
		* 根据每个目录的用途来挂载不同大小的分区，预留一个备用的磁盘容量。
	* 选择 Linux  安装程序提供的默认硬盘分区方式
 


------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《鸟哥的Linux私房菜》](http://linux.vbird.org "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
