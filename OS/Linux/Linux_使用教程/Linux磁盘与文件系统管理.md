---
title:  Linux磁盘与文件系统管理
tags: Linux使用教程
---

------

<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《鸟哥的Linux私房菜》](http://linux.vbird.org "点击跳转")、[《Linux命令手册》](http://linux.51yip.com "点击跳转")、[《Linux命令大全》](http://man.linuxde.net "点击跳转")以及[《Linux man pages》](https://linux.die.net/man/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

<style>table{word-break:initial;}</style>


# 1   Linux文件系统
## 1.1 Ext2文件系统
Linux的标准文件系统为Ext2。是一种索引式文件系统。

* superblock——记录此 filesystem 的整体信息，包括 inode/block 的总量、使用量、剩余量， 以及文件系统的格式与相关信息等；
* inode——记录文件的权限、属性，一个文件占用一个 inode，同时还记录此文件的数据所在的 block 号码，如果block太多则使用多级节点；
* block——实际记录文件的内容，若文件太大时，会占用多个 block 。

文件系统一开始就将inode与block规划好了，除非重新格式化(或者利用resize2fs 等指令变更文件系统大小)，否则 inode 与 block 固定后就不再变动。

Ext2 文件系统在格式化的时候分为多个区块群组 (block group) 的，每个区块群组都有独立的inode/block/superblock系统。每一个区块群组(block group)的六个主要内容：

* data block
  * 原则上，block 的大小与数量在格式化完就不能够再改变了(除非重新格式化)；
  * 每个 block 内最多只能够放置一个文件的数据； 如果文件大于 block 的大小，则一个文件会占用多个 block 数量； 若文件小于 block，则该block的剩余容量就不能够再被使用了(磁盘空间会浪费)。
  * Block大小 1KB、2KB 、4KB，最大单一文件限制依次为16GB、256GB、2TB，最大文件系统总容量依次为2TB、8TB、16TB。
* inode table
  * inode 记录的文件数据至少有底下这些
     * 该文件的存取模式(read/write/excute)；
     * 该文件的拥有者与群组(owner/group)；
     * 该文件的大小；
     * 该文件建立或状态改变的时间(ctime)；
     * 最近一次的读取时间(atime)；
     * 最近修改的时间(mtime)；
     * 定义文件特性的旗标(flag)，如 SetUID...；
     * 该文件真正内容的指向 (pointer)；
  * inode 的数量与大小也是在格式化时就已经固定了
  * 每个inode大小均固定为128 bytes (新的ext4与xfs可设定到 256 bytes)，只能记录12个直接block号，1个间接，1个二级间接，1个三级间接。每个block号码占用4B。
  * 每个文件都仅会占用一个 inode 而已；
  * 因此文件系统能够建立的文件数量与 inode 的数量有关；
  * 系统读取文件时需要先找到inode，并分析inode所记录的权限与用户是否符合，若符合才能够开始实际读取 block 的内容。
  * 当文件较大使得block号较多时，可以使用多级间接记录block号。 
* Superblock ，记录的信息主要有：
  * block 与 inode 的总量；
  *  未使用与已使用的 inode / block 数量；
  * block 与 inode 的大小 (block 为 1, 2, 4K，inode 为 128bytes 或 256bytes)；
  * filesystem 的挂载时间、最近一次写入数据的时间、最近一次检验磁盘 (fsck) 的时间等文件系统的相关信息；
  * 一个有效位，若此文件系统已被挂载，则为 0 ，若未被挂载，则为1 。
* Filesystem Description——描述每个 block group 的开始与结束的 block 号码，以及说明每个区段 (superblock,bitmap, inodemap, data block) 分别介于哪一个 block 号码之间。
* block bitmap——记录block的占用情况
* inode bitmap——记录inode编号的使用情况


## 1.2 与目录树的关系
 目录——新建目录时，文件系统会分配一个 inode 与至少一块 block 给该目录。其中，inode 记录该目录的相关权限与属性，并可记录分配到的那块 block 号码； 而block则是记录在这个目录下的目录项，即文件名与每个文件占用的inode号码。 

文件—— 新建文件时， ext2 会分配一个 inode 与相对于该文件大小的若干个block 给该文件。

目录树读取——要读取某个文件，就务必经过目录的 inode 与 block ，才能根据文件名找到目录项，并得到那个待读取文件的 inode 号码，最终才会读到正确的文件的 block 内的数据。

filesystem大小与磁盘读取效能——数据离散分布，寻道时间增加。

## 1.3 日志文件系统
 inode table 与 data block称为**数据存放区域**，至于其他例如 superblock、 block bitmap 与 inode bitmap 等区段就被称为 **metadata (中介资料)** 。

新建文件的过程：

* 先确定用户对于欲新增文件的目录是否具有w与x的权限，若有的话才能新增；
* 根据 inode bitmap找到没有使用的inode号码，并将新文件的权限/属性写入；
* 根据 block bitmap 找到没有使用中的 block 号码，并将实际的数据写入 block 中，且更新 inode 的 block指向数据；
* 将刚刚写入的 inode 与 block 数据同步更新 inode bitmap 与 block bitmap，并更新 superblock 的内容。

**数据的不一致 (Inconsistent) 状态**：在修改过的块写回之前，系统崩溃，甚至导致metadata 的内容与实际数据存放区产生不一致。**解决办法**：

* 在 Ext2 文件系统中，如果发生这个问题， 那么系统在重新启动的时候，就会藉由 Superblock 当中记录的 valid bit (是否有挂载) 与 filesystem state (clean与否)等状态来判断是否强制进行数据一致性的检查！若有需要检查时则以e2fsck程序来进行的。**缺点**：这样的检查真的是很费时，因为要针对 metadata 区域与实际数据存放区来进行比对。
* ext3/ext4的**日志文件系统**：filesystem当中规划出一个区块，该区块专门在记录写入或修订文件时的步骤，万一数据的纪录过程当中发生了问题，系统只要去检查日志记录区块，就可以知道哪个文件发生了问题，针对该问题来做一致性的检查即可，而不必针对整块 filesystem 去检查， 这样就可以达到快速修复 filesystem ！也就是说：

  1. 预备：当系统要写入一个文件时，会先在日志记录区块中纪录某个文件准备要写入的信息；
  2. 实际写入：开始写入文件的权限与数据；开始更新 metadata 的数据；
  3. 结束：完成数据与metadata的更新后，在日志记录区块当中完成该文件的纪录。
## 1.4 Linux文件系统的运作
为了避免CPU等待磁盘I/O，Linux使用**异步处理** (asynchronously)的方式。所谓的异步处理是这样的：磁盘数据在内存中如果没有修改过则设置为干净(clean)的。否则设定为脏的(Dirty)。但不立即写入到磁盘中！ 系统会不定时的将内存中设定为Dirty的数据写回磁盘，以保持磁盘与内存数据的一致性。 也可以利sync 指令来手动强迫写入磁盘。

* 系统会将常用的文件数据放置到主存储器的缓冲区，以加速文件系统的读/写；
* 因此 Linux 的物理内存最后都会被用光！这是正常的情况！可加速系统效能；
* 你可以手动使用 sync 来强迫内存中设定为 Dirty 的文件回写到磁盘中；
* 若正常关机时，关机指令会主动呼叫 sync 来将内存的数据回写入磁盘内；
* 但若不正常关机(如跳电、当机或其他不明原因)，由于数据尚未回写到磁盘内， 因此重新启动后可能会花很多时间在进行磁盘检验，甚至可能导致文件系统的损毁(非磁盘损毁)。

## 1.5 挂载点的意义
挂载点一定是目录，该目录为进入该文件系统的入口。 文件系统必须要挂载到目录树的某个目录后，才能够使用该文件系统。

## 1.6 Linux支持的文件系统
想要知道你的Linux支持的文件系统有哪些，可以察看`/lib/modules/$(uname -r)/kernel/fs`这个目录；想要知道系统目前已加载到内存中支持的文件系统可以查看`/proc/filesystems`这个文件。

Linux支持的文件系统：

* 传统文件系统：ext2 / minix / MS-DOS / FAT (用 vfat 模块) / iso9660 (光盘)等等；
* 日志式文件系统： ext3（ext2增加日志功能） /ext4 （ext3升级）/ ReiserFS（适合小型文件） / Windows' NTFS / IBM's JFS / SGI's XFS / ZFS
*  网络文件系统： NFS / SMBFS

## 1.7 XFS文件系统

EXT系列文件系统的**缺点**：支持度最广，但格式化超慢，**原因**是预先规划出所有的inode/block/metadata等数据，随着磁盘容量越来越大，格式化越来越费时间。

xfs 文件系统**优点**：较适合大容量磁盘与巨型文件（如虚拟磁盘）。

xfs文件系统在资料的分布上，主要规划为三个部份，一个数据 区 (data section)、一个文件系统活动登录区 (log section)以及一个实时运作区 (realtime section)。 这三个区域的数据内容如下：

* **数据区**(data section):数据区就跟我们之前谈到的 ext文件系统类似，包括 inode/data block/superblock 等数据，都放置在这个区块，也是分为多个储存区群组来分别放置文件系统所需要的数据。 每个储存区群组都包含了整个文件系统的 superblock、 剩余空间的管理机制、 inode 的分配与追踪。此外，inode与block都是系统需要用到时才动态配置产生，所以格式化动作超级快！另外，与ext家族**不同的是**，xfs的block与inode有多种不同的容量可供设定，block 容量可由 512bytes~64K调配，不过，Linux 的环境下，由于内存控制的关系(页面文件pagesize的容量之故)，因此最高可以使用的 block 大小为 4K 而已，至于 inode 容量可由 256bytes 到 2M 这么大！

* **文件系统活动登录区** (log section)：在登录区有点像是日志区,主要用来纪录文件系统的变化，文件的变化会在这里纪录下来，直到该变化完整的写入到数据区后，该笔纪录才会被终结。如果文件系统因为某些缘故(例如最常见的停电)而损毁时，系统会拿这个登录区块来进行检验，看看系统挂掉之前，文件系统正在运作些啥动作，藉以快速的修复文件系统。因为系统所有动作的时候都会在这个区块做个纪录，因此这个区块的磁盘活动是相当频繁的！你可以指定外部的磁盘来作为xfs文件系统的日志区块！例如，将 SSD磁盘作为xfs的登录区。

* **实时运作区** (realtime section)：当有文件要被建立时，xfs 会在这个区段里面找一个到数个的extent区块，将文件放置在这个区块内，等到inode 与 block分配完毕后，再写入到 data section 的 inode 与 block ！ 这个extent区块的大小要在格式化的时候就先指定，最小值是 4K，最大可到 1G。一般非磁盘阵列的磁盘默认为 64K容量，而具有类似磁盘阵列的stripe 情况下，则建议 extent 设定为与 stripe 一样大较佳。这个extent 最好不要乱动，因为可能会影响到实体磁盘的效能喔。

## 1.8 虚拟文件系统VFS
windows使用盘符来处理不同的文件系统，所谓不需要将不同的文件系统整合，而大多数UNIX操作系统都使用虚拟文件系统概念尝试将多种文件系统统一成一个有序的框架。 

**关键思想**:抽象出所有文件系统的共有部分，并且将这部分代码放在单独的一层，该层调用底层的实际文件系统来具体管理数据。 

![1](https://www.github.com/liao20081228/blog/raw/master/图片/Linux磁盘与文件系统管理/1.JPG)

VFS有对用户进程的上层接口（POSIX接口），对实际文件系统的下层接口。只要实际文件系统提供VFS所需的功能，VFS就不需要知道或者关心数据具体存放在什么地方或者底层实际文件系统是什么的样子。 

实际文件系统在使用前必须先注册，即提供一个包含VFS所需函数的地址的列表。 
设计实际文件系统时只需要先获得VFS期待的功能，然后再实现这些功能即可，如果文件系统已经存在，则只需提供VFS所需功能即可。

## 1.9 硬链接与软链接
**硬链接**：多个文件名映射到同一文件的inode。hard link只是在某个目录的data block中新增一个文件名链接到某inode号码的关联记录。特点：
* 如果文件存在硬链接，则删除一个目录项后，文件仍不会删除，因为链接计数大于0。
* 通过任意目录项都可以读写文件。
* 一般不会改变磁盘空间和inode数量，除非正好填满目录的data block。
* 不能跨文件系统。
* 不能对目录硬链接。

**软链接（符号链接）**：建立一个独立的文件，该文件的block记录的目标文件的路径。类似于windows的快捷方式。特点：
* 原文件删除后，符号链接无法访问。
* 占用inode 与 block 
* 能跨文件系统。
* 能对目录软链接。

**目录的链接计数**：新建一个子目录时，该子目录的链接计数为2，而当前目录链接计数加1，因为自目录中有.和..两个目录。

# 2 文件系统管理
## 2.1 查看文件系统信息
### 2.1.1 查看EXT文件系统信息——`dumpe2fs`
**命令**：dumpe2fs [ -bfhixV ] [ -o superblock=superblock ] [ -o blocksize=blocksize ] device
**描述**：显示ext系列文件系统的超级块和区块群组信息。
|常用选项|作用|
|:--|:--|
|-b|列出保留为坏轨的部分|
|-h|仅列出 superblock 的数据，不会列出其他的区段内容！|
|**更多信息**|<http://linux.51yip.com/search/dumpe2fs> 和 man 手册|
||<http://man.linuxde.net/dumpe2fs>|

### 2.1.2 查看XFS文件系统信息——`xfs_info`
**命令**：xfs_info  挂载点 或 装置文件名
**描述**：显示xfs文件系统的超级块和区块群组信息。
|常用选项|作用|
|:----|:--|
|**更多信息**|<http://www.bubuko.com/infodetail-1878036.html> 和 man pages|  


## 2.2 管理硬链接和软链接
### 2.2.1 建立硬链接和软链接——`ln`
`ln`、`link` `unlink`
**命令**：ln [-sf]  来源文件 目标文件
**描述**：建立硬链接和软链接。
|常用选项|作用|
|:----|:--|
|-s |如果不加任何参数就进行链接，那就是 hard link，至于 -s 就是软链接
|-f |如果 目标文件 存在时，就主动的将目标文件直接移除后再建立！
|**更多信息**|<http://linux.51yip.com/search/ln> 和 man 手册|
||<http://man.linuxde.net/ln>|

### 2.2.2 建立硬链接——`link`
**命令**：link  来源文件 目标文件
**描述**：建立硬链接
|常用选项|无|
|:----|:--|
|**更多信息**|<http://linux.51yip.com/search/link> 和 man 手册|
||<http://man.linuxde.net/link>|

### 2.2.3 删除硬链接——`unlink`
**命令**：unlink  目标文件
**描述**：删除硬链接 
|常用选项|无|
|:----|:--|
|**更多信息**|<http://linux.51yip.com/search/unlink> 和 man 手册|
||<http://man.linuxde.net/unlink>|


## 2.3 文件系统检验与修复
### 2.3.1 xfs检验修复——`xfs_repair`
**命令**：xfs_repair [-fnd]  装置名 
**描述**：检验与修复xfs文件系统。修复时该文件系统不能被挂载！
|常用选项|作用|
|:--|:--|
|-f |后面的装置其实是个文件而不是实体装置
|-n |单纯检查并不修改文件系统的任何数据 (检查而已)
|-d |通常用在单人维护模式底下，针对根目录进行检查与修复！很危险！
|**更多信息**|<http://linux.51yip.com/search/xfs_repair> 和 man pages|

### 2.3.2 综合检验修复——`fsck`
**命令**：fsck [-lsAVRTMNP] [-r [fd]] [-C [fd]] [-t fstype] [filesystem...] [--] [fs-specific-options]
**描述**：根据指定的type调用相关工具来检验与修复文件系统。
|常用选项|作用|
|:--|:--|
|**更多信息**|<http://linux.51yip.com/search/fsck> 和 man 手册|
||<http://man.linuxde.net/fsck>|

### 2.3.3 ext4检验修复——`fsck.ext4`
**命令**：fsck.ext4 [-pf] [-b superblock]  装置名
**描述**：检验与修复ext4文件系统.
|常用选项|作用|
|:--|:--|
|-p |当文件系统在修复时，若有需要回复y的动作时，自动回复 y 来继续进行修复动作。
|-f|强制检查！一般来说，如果 fsck 没有发现任何 unclean的旗标，不会主动进入细部检查的，如果您想要强制 fsck进入细部检查，就得加上 -f 旗标啰！
|-D |针对文件系统下的目录进行优化配置。
|-b |后面接 superblock 的位置！一般来说这个选项用不到。但是如果你的 superblock 因故损毁时，透过这个参数即可利用文件系统内备份的 superblock 来尝试救援。一般来说，superblock 备份在：1K block 放在 8193, 2K block 放在 16384, 4K block 放在 32768
|**更多信息**|<http://linux.51yip.com/search/fsck.ext4> 和 man pages|
|&emsp;&emsp;&emsp;&emsp;||


## 2.4 文件系统的挂载与卸除
注意：
* 单一文件系统不应该被重复挂载在不同的挂载点(目录)中；
* 单一目录不应该重复挂载多个文件系统；
* 要作为挂载点的目录，理论上应该都是空目录才是。否则原目录下的内容会被隐藏。
### 2.4.1 文件系统挂载——`mount`
**命令**：mount [-l|-h|-V]
&emsp;&emsp;&emsp;mount -a [-fFnrsvw] [-t fstype] [-O optlist]
&emsp;&emsp;&emsp;mount [-fnrsvw] [-o options] device|dir
&emsp;&emsp;&emsp;mount [-fnrsvw] [-t fstype] [-o options] device dir
**描述**：挂载文件系统。
|常用选项|作用|
|:--|:--|
|-a |依照配置文件 /etc/fstab 的数据将所有未挂载的磁盘都挂载上来
|-l |单纯的输入mount会显示目前挂载的信息。加上 -l 可增列 Label 名称！
|-t |可以加上文件系统种类来指定欲挂载的类型。
|-n |在默认的情况下，系统会将实际挂载的情况实时写入 /etc/mtab 中，以利其他程序的运作。但在某些情况下(例如单人维护模式)为了避免问题会刻意不写入。此时就得要使用 -n 选项。
|-o |后面可以接一些挂载时额外加上的参数！async, sync: 此文件系统使用同步写入或异步的内存机制，预设为 async。atime,noatime: 是否修订文件的读取时间(atime)。ro, rw: 挂载文件系统成为只读 或可擦写。auto, noauto: 允许此 filesystem 被以 mount -a 自动挂载(auto)。dev, nodev: 是否允许此 filesystem 上，可建立装置文件？ dev 为可允许。suid, nosuid: 是否允许此 filesystem 含有 suid/sgid 的文件格式？exec, noexec: 是否允许此 filesystem 上拥有可执行 binary 文件？user, nouser: 是否允许此 filesystem 让任何使用者执行 mount ？一般来说，mount 仅有 root 可以进行，但下达 user 参数，则可让一般 user 也能够对此 partition 进行 mount 。remount:重新挂载，这在系统出错，或重新更新参数时，很有用！defaults：rw, suid, dev, exec, auto, nouser, and async
|**更多信息**|<http://linux.51yip.com/search/mount> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/mount>|

基本上，不需要加上-t这个选项，系统会自动的分析最恰当的文件系统来尝试挂载你需要的装置！由于文件系统几乎都有 superblock ， Linux 可以透过分析superblock 搭配Linux自己的驱动程序去测试挂载，如果成功的套和了，就立刻自动的使用该类型的文件系统挂载起来啊！那么系统有没有指定哪些类型的 filesystem 才需要进行上述的挂载测试呢？主要是参考底下这两个文件：

* /etc/filesystems：系统指定的测试挂载文件系统类型的优先级；
* /proc/filesystems：Linux 系统已经加载的文件系统类型。

那怎么知道Linux有没有相关文件系统类型的驱动程序呢？Linux支持的文件系统之驱动程序都写在`/lib/modules/$(uname -r)/kernel/fs/`目录找中，例如 ext4 的驱动程序就写在`/lib/modules/$(uname -r)/kernel/fs/ext4/`这个目录下啦！

光驱一挂载之后就无法退出光盘片了！除非你将他卸除才能够退出！ 
```bash
$ mount -o remount,rw,auto #将/重新挂载。尤其时根目录为只读时。

$ mount  --bind resource destination #将一个目录挂载到另一个目录。

$ mount  -o loop imagefile destination #挂载一个镜像文件，如.iso文件。
```
### 2.4.2 文件系统卸载——`umount`
**命令**：umount [-fn]  装置文件名或挂载点
**描述**：卸除文件系统或装置。

|常用选项|作用|
|:--|:--|
|-f |强制卸除！可用在类似网络文件系统 (NFS) 无法读取到的情况下；
|-l |立刻卸除文件系统，比 -f 还强！
|-n |不更新 /etc/mtab 情况下卸除。
|**更多信息**|<http://linux.51yip.com/search/umount> 和 man 手册|
||<http://man.linuxde.net/umount>|

### 2.4.3 设置开机挂载
在开机的时候就将文件系统挂载好，可以通过修改`/etc/fstab`,mount指令就是将所有的选项与参数写入到这个文件中。除此之外，`/etc/fstab `还加入了 dump 这个备份用指令的支持！`/etc/fstab`是开机时的配置文件，实际 filesystem 的挂载是记录到`/etc/mtab`与`/proc/mounts`。但当 `/etc/fstab`数据错误，导致无法开机而进入单人维护模式时，由于根目录是只读状态，无法修改 `/etc/fstab` ，也无法更新 `/etc/mtab `，只能使用`mount -n -o remount,rw /`来重新挂载根目录。

`/etc/fstab`记录的内容：

* 装置/UUID/LABEL等
* 挂载点
* 文件系统类型
* 文件系统参数：也就是mount的-o选项输入的参数，
* dump ：能否被dump备份指令使用。一般填0.
* fsck：是否以 fsck 检验扇区。xfs系统不需要，填0。

注意：

* 根目录 / 是必须先于其它 mount point 被挂载进来。
* 其它 mount point必须为已建立的目录﹐可任意指定﹐但一定要遵守FHS
* 所有 mount point 或 partition 在同一时间之内只能挂载一次。
* 如若进行卸除﹐您必须先将工作目录移到 mount point(及其子目录) 之外。

## 2.5 修改文件系统参数
### 2.5.1 修改xfs参数——`xfs_admin`
**命令**：xfs_admin [-lu] [-L label] [-U uuid] 装置文件名
**描述**：修改XFS文件系统的 UUID  与 Label name
|常用选项|作用|
|:--|:--|
|-l |列出这个装置的 label name
|-u |列出这个装置的 UUID
|-L |设定这个装置的 Label name
|-U |设定这个装置的 UUID 喔！
|**更多信息**|<http://linux.51yip.com/search/xfs_admin> 和 man pages|
|||

### 2.5.2 修改ext4参数——`tune2fs`
**命令**：tune2fs [-l] [-L Label] [-U uuid]  装置文件名
**描述**：修改ext4文件系统的 UUID  与 Label name
|常用选项|作用|
|:--|:--|
|-l |类似 dumpe2fs -h 的功能～将 superblock 内的数据读出来
|-L |修改 LABEL name
|-U |修改 UUID 啰！
|**更多信息**|<http://linux.51yip.com/search/tune2fs> 和 man 手册|
||<http://man.linuxde.net/tune2fs>|





# 3 磁盘管理
新增磁盘的步骤：

1. 对磁盘进行分区，以建立可用的 partition ；
2. 对该 partition 进行格式化 (format)，以建立系统可用的 filesystem；
3. 若想要仔细一点，则可对刚刚建立好的 filesystem 进行检验；
4. 在 Linux 系统上，需要建立挂载点 (亦即是目录)，并将他挂载上来

## 3.1 查看磁盘信息
### 3.1.1 列出块设备——`lsblk`
**命令**：lsblk [-dfimpt ] [device]
**描述**：列出系统上的所有磁盘列表。
|常用选项|作用|
|:----|:--|
|-d |仅列出磁盘本身，并不会列出该磁盘的分区数据
|-f |同时列出该磁盘内的文件系统名称
|-i |使用 ASCII 的线段输出，不要使用复杂的编码 (再某些环境下很有用)
|-m |同时输出该装置在 /dev 底下的权限数据 (rwx 的数据)
|-p |列出该装置的完整文件名！而不是仅列出最后的名字而已。
|-t |列出该磁盘装置的详细数据，包括磁盘队列机制、预读写数据量大小等
|**更多信息**|<http://linux.51yip.com/search/lsblk> 和 man 手册|
||<http://man.linuxde.net/lsblk>|

### 3.1.2 查看块设备信息——`blkid`
**命令**：blkid device
**描述**：可以查看块设备（包括交换分区）的文件系统类型、LABEL、UUID、挂载目录等信息。
|常用选项|无|
|:----|:--|
|**更多信息**|<http://linux.51yip.com/search/blkid> 和 man 手册|
||<http://man.linuxde.net/blkid>|

### 3.1.3 查看分区信息——`parted`
**命令**：parted device_name print
**描述**：列出磁盘的分区表类型与分区信息。
|常用选项|无|
|:----|:--|
|**更多选项**|<http://linux.51yip.com/search/parted> 和 man pages|


### 3.1.4 查看文件系统的整体磁盘使用情况——`df`
**命令**：df [-ahikHTm] [ 目录或文件名] 
**描述**：可以查看文件系统的整体磁盘使用情况。
|常用选项|作用|
|:----|:--|
|-a |列出所有的文件系统，包括系统特有的 /proc 等文件系统；
|-k |以 KBytes 的容量显示各文件系统；
|-m |以 MBytes 的容量显示各文件系统；
|-h |以人们较易阅读的 GBytes, MBytes, KBytes 等格式自行显示；
|-H |以 M=1000K 取代 M=1024K 的进位方式；
|-T |连同该 partition 的 filesystem 名称 (例如 xfs) 也列出；
|-i |不用磁盘容量，而以 inode 的数量来显示
|**更多信息**|<http://linux.51yip.com/search/df> 和 man 手册|
||<http://man.linuxde.net/df>|


### 3.1.5 评估文件系统的磁盘使用量——`du`
**命令**：du [-ahskm]  文件或目录 
**描述**：评估文件系统的磁盘使用量(常用在推估目录所占容量)。
|常用选项|作用|
|:----|:--|
|-a |列出所有的文件与目录容量，因为默认仅统计目录底下的文件量而已。
|-h |以人们较易读的容量格式 (G/M) 显示；
|-s |列出总量而已，而不列出每个各别的目录占用容量；
|-S |不包括子目录下的总计，与 -s 有点差别。
|-k |以 KBytes 列出容量显示；
|-m |以 MBytes 列出容量显示；
|**更多信息**|<http://linux.51yip.com/search/du> 和 man 手册|
||<http://man.linuxde.net/du>|





## 3.2 磁盘分区
### 3.2.1 综合分区——`parted`
**命令**：parted [装 置] [ 指令 [ 参数 ]]
**描述**：对GPT或者MBR磁盘建立分区
|常用指令|无|
|:----|:--|
|新增分区|mkpart [primary\logical\extended] [ext4\vfat\xfs] 开始 结束|
|显示分区|print
|删除分区|rm [partition]
|**更多信息**|<http://linux.51yip.com/search/parted> 和 man 手册|
||<http://man.linuxde.net/parted>|

### 3.2.2 gpt分区——`gdisk`
**命令**：gdisk  装置名
**描述**：对GPT格式磁盘分区。
|常用命令|无|
|:----|:--|
|d|	删除分区
|n|	新建分区
|p|	打印分区表
|q|	退出，并不保存操作
|w|	退出，保存操作
|?|	打印帮助信息
|**更多信息**|<http://linux.51yip.com/search/gdisk> 和 man 手册|

### 3.2.3 mbr分区——`fdisk`
**命令**：fdisk  装置名
**描述**：对MBR格式磁盘分区。
|常用命令|无|
|:----|:--|
|d|	删除分区
|n|	新建分区
|p|	打印分区表
|q|	退出，并不保存操作
|w|	退出，保存操作
|m|	打印帮助信息
|**更多信息**|<http://linux.51yip.com/search/fdisk> 和 man 手册|
||<http://man.linuxde.net/fdisk>|

### 3.2.4 重载分区表——`partprobe`
**命令**：partprobe [-s] 
**描述**：更新分区表信息。
|常用选项|无|
|:--|:--|
|**更多信息**|<http://linux.51yip.com/search/partprobe> 和 man 手册|
||<http://man.linuxde.net/partprobe>|

## 3.3 磁盘格式化
### 3.3.1 综合格式化——`mkfs`
**命令**：mkfs [-t type] [fs-options] device [size]
**描述**：根据指定的文件系统类型调用对应的格式化工具格式化磁盘。
|常用选项|无|
|:--|:--|
|**更多信息**|<http://linux.51yip.com/search/mkfs> 和 man 手册|
||<http://man.linuxde.net/mkfs>|

### 3.3.2 xfs格式化——`mkfs.xfs`
**命令**：mkfs.xfs [-b bsize] [-d parms] [-i parms] [-l parms] [-L label] [-f] [-r parms]  装置名
**描述**：将磁盘进行xfs格式化
|常用选项|无|
|:--|:--|
|**更多信息**|<http://linux.51yip.com/search/mkfs.xfs> 和 man 手册|

#### 3.3.3 ext4格式化——`mkfs.ext4`
**命令**：mkfs.ext4 [-b size] [-L label] 装置名
**描述**：将磁盘进行ext4格式化
|常用选项|无|
|:--|:--|
|**更多信息**|<http://linux.51yip.com/search/mkfs.ext4> 和 man pages|





## 3.4 设置设备代码——`mknod`
**命令**：mknod  装置文件名 [bcp] [Major] [Minor]
**描述**：设置装置的主次设备代码。
|常用选项|作用|
|:--|:--|
|b|设定装置名称成为一个周边储存设备文件，例如磁盘等；
|c|设定装置名称成为一个周边输入设备文件，例如鼠标/键盘等；
|p|设定装置名称成为一个 FIFO 文件；
|**更多信息**|<http://linux.51yip.com/search/mknod> 和 man 手册|
||<http://man.linuxde.net/mknod>|
常见的磁盘文件名 /dev/sda 与 /dev/loop0装置代码如下所示：
|磁盘文件名| Major |Minor|
|:--|:--|--
|/dev/sda |8 |0-15
|/dev/sdb |8 |16-31
|/dev/loop0 |7| 0
|/dev/loop1 |7 |1





# 4 建立swap分区
**步骤**：
1. 分区或建立一个文件——先磁盘中划分一个分区或建立一个文件。由于 Linux 的 gdisk 预设会将分区槽的ID设定为Linux的文件系统，所以可能还得要设定一下 system ID 。
2. 格式化——用`mkswap 装置/文件`将该分区或文件格式化为swap 格式。
3. 使用——用`swapon 装置文件名`启用该分区。
4. 观察——用` free `与 `swapon -s `这个指令来观察一下内存的用量吧！
5. 使用——用`swapoff 装置/文件名`关闭该分区。


## 4.1 swap格式化——`mkswap`
**命令**：mkswap [options] device [size]
**描述**：用于在一个文件或者设备上建立交换分区。
|常用选项|无|
|:--|:--|
|**更多信息**|<http://linux.51yip.com/search/mkswap> 和 man 手册|
||<http://man.linuxde.net/mkswap>|

## 4.2 挂载swap分区——`swapon`
**命令**：swapon [options] [specialfile...]
**描述**：用于在一个文件或者设备上建立交换分区。
|常用选项|无|
|:--|:--|
|**更多信息**|<http://linux.51yip.com/search/swapon> 和 man 手册|
||<http://man.linuxde.net/swapon>|

## 4.3 卸载swap分区——`swapoff` 
**命令**：swapoff [-va] [specialfile...]
**描述**：用于在一个文件或者设备上建立交换分区。
|常用选项|无|
|:--|:--|
|**更多信息**|<http://linux.51yip.com/search/swapoff> 和 man 手册|
||<http://man.linuxde.net/swapoff>|

# 5 查看内存使用量——`free`
**命令**：free [options]
**描述**：显示当前系统内存使用情况。
|常用选项|无|
|:--|:--|
|**更多信息**|<http://linux.51yip.com/search/free> 和 man 手册|
||<http://man.linuxde.net/free>|



------

<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《鸟哥的Linux私房菜》](http://linux.vbird.org "点击跳转")、[《Linux命令手册》](http://linux.51yip.com "点击跳转")、[《Linux命令大全》](http://man.linuxde.net "点击跳转")以及[《Linux man pages》](https://linux.die.net/man/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
