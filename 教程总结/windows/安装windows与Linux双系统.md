---
title: 安装windows与Linux双系统
tags: 其它
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：<font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------


# 1 先安装Windows10 
* 下载Windows ISO镜像、 UtraISO 9.7。
* 运行UtraISO，文件→打开，选择要安装的镜像文件。
* 启动→写入硬盘镜像。
* 写入方式为USB-HDD++，隐藏启动分区为无。
* 写入。完成后关机。
* 开机F2或者其它键进入BIOS。选择UEFI，并设置启动盘为USB。
* 退出BIOS，进入安装界面。
* 选择安装语言、时间格式、键盘输入法，点击下一步。
* 点击`现在安装`,如果是启动出问题，则可以点击`修复计算机`。
* 输入密钥或者跳过。
* 如果是Multiple版本，包含有2个版本，pro是专业版，home是家庭版，选择之后，点击下一步。
* 勾选`我接受许可条款`，点击下一步。
* 执行的安装类型，要选择`自定义：仅安装Windows(高级)`。
* 如果要装系统的的硬盘原本就是GPT格式
	* 如果安装新系统，则进入到分区界面，如果之前有系统，要先备份好数据，然后删除所有分区，只剩下一个未分配的空间，选择未分配的空间，点击`下一步`。或者点击`新建`进行手动分区后，选择一个分区，点击`下一步`。
	* 如果安装多个系统，则选择一个空分区，点击`下一步`。
*  如果要装系统的的硬盘原本就是MBR格式，就需要使用diskpart工具来转换磁盘格式。
	* shift+F10，进入CMD。
	* diskpart，回车。
	* list disk，回车——列出磁盘。如果GPT一栏下不带*，则该磁盘不是GPT格式。
	* select disk 磁盘编号，回车——选中要操作的磁盘。
	* clean，回车——该步骤会清除磁盘数据，注意备份。
	* convert GPT，回车——转为GPT，当然也转为MBR。
	* list partition，回车——列出所有分区。
	* create partition msr size=指定大小，回车——创建GPT的msr保护分区。
	* create partition efi size=指定大小，回车——创建efi分区。
	* create partition primary size=指定大小，回车——创建主分区。size:表示分区大小，单位M。在MSR格式下，如果这个位置是logical：表示逻辑分区，extended：表示扩展分区。
	* 如果创建错了分区
		* select partition 分区编号，回车。
		* delete partition，回车——删除分区。
 * exit，回车——退出。
 * 选择一个主分区，点击`下一步`。
 * 当然用diskpart命令将磁盘转为GPT后，也可以退出到安装界面进行分区。

# 2 后安装Centos 7
## 2.1 安装Centos7
* 运行windows的磁盘管理工具，选择要安装的磁盘，压缩出足够空间。
* 下载Centos7 ISO镜像、 UtraISO 9.7。
* 运行UtraISO，文件→打开，选择要安装的镜像文件。
* 启动→写入硬盘镜像。
* 写入方式为USB-HDD++，隐藏启动分区为无。
* 写入。完成后关机。
* 开机F2或者其它键进入BIOS。关闭Secure Boot，选择UEFI，并设置启动盘为USB。
* 退出BIOS，进入UEFI的安装界面。这种界面只会出现在以 UEFI模式 安装时，第一个是正常安装，第二个是检测安装，第三个是修复模式。上下键控制，选择第一个 lnstall CentOS 7 。按“e”进入编辑状态。

![32](https://www.github.com/liao20081228/blog/raw/master/图片/安装windows与Linux双系统/1.jpg)
>传统Leagcy 模式显示的是!
>
>![32](https://www.github.com/liao20081228/blog/raw/master/图片/安装windows与Linux双系统/2.jpg)

* 进入 “lnstall CentOS 7” 的编辑状态后  把 ` linuxefi /images/pxeboot/vmlinuz inst.stage2=hd:LABEL=CentOS\x207\x20x86_64 quiet ` 改为：`linuxefi /images/pxdboot/vmlinuz linux dd quiet` ”

![33](https://www.github.com/liao20081228/blog/raw/master/图片/安装windows与Linux双系统/3.jpg)
* 按下CTRL+x，过一会后，显示结果如下，很明显我的U盘是 sda4 这个U盘名称  不是固定的，sd 代表硬盘，a代表第几块硬盘，4代表的的第几个分区（理论上U盘应该是sdc1，没办法系统要读成这样）。知道U盘在CentOS下的设备文件名之后按 “ c ”  回车，返回到安装选项界面 （如果还没退出 按ctrl + alt + delete 键退出）。
* 返回到安装选项界面选择install centos7，按“e” 进入编辑 把：`linuxefi /images/pxeboot/vmlinuz inst.stage2=hd:LABEL=CentOS\x207\x20x86_64 quiet` 改为` linuxefi /images/pxdboot/vmlinuz inst.stage2=hd:/dev/sda4 quiet`, 按ctrl + x 键确认,进入图形安装界面,/dev/sda4是U盘所在的位置     /代表根,dev存放设备文件的目录 ,sda4 是U盘的设备文件名。（这里如果一开始选择 lnstall CentOS 7 按 “e” 查询到了U盘的设备名，这里就要到同一安装选项中输入设备路径，才可以进入图形安装界面，正常安装与检测安装中查出的所有设备名都不一致，这里不能乱，在哪里查就在哪里用，否则不能进入图形安装界面。）
* 选择安装程序的语言
* 设置日期时间、语言支持、键盘、要安装的软件。
* 设置网络，如果不知道，可以不设置。
* 设置安装位置。
	* 单硬盘，和window是安装到同一硬盘。
	* 多硬盘，安装到无系统的硬盘。
	*  选择要安装的磁盘。
*  点`+`，新建/、/home、/boot/efi、/var、swap的挂载点，大小按照需求设定。其中/、/boot/efi必须挂载，如果内存够大也可不挂在swap。
     *  分区类型有标准分区、lvm、简单lvm、btrfs。/boot采用标准分区。其他使用lvm。
     *  文件系统有ext2，ext3，ext4，vfat，xfs，efi，swap。除了/boot/efi使用efi，swap使用swap外，其他可以自由设置。
*  开始安装，等待重启即可。

## 2.2 补充
* 如果使用Leagcy BIOS+MBR ，则不是按E而是按TAB编辑安装磁盘。
* 如果使用Leagcy BIOS+GPT ，除了按TAB编辑安装磁盘，还需要在quiet后输入：空格inst.gpt来使用gpt分区。
* 如果使用BIOS+MBR的方式，可按下表挂载分区：

|所需目录/装置 |磁盘容量 |文件系统 |分区格式|
|--|--|--|--|
|BIOS boot| 2MB |系统自定义 |主分区|
|/boot |1GB |xfs |主分区|
|/ |10GB |xfs |LVM 方式|
|/home| 5GB| xfs |LVM 方式|
|swap| 1GB |swap| LVM 方式|



  



------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：<font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
