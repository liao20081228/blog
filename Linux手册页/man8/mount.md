---
title: mount 
tags: man8,系统管理
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《mount manpages》。</font>当前版本为2.36.1，手册更新时间为2015-08。<font color=red>本文与原文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------

# 名称
mount——挂载一个文件系统。

# 总览
**mount** [**-h**|**-V**]
**mount** [**-l**] [**-t** <u>fstype</u>]
**mount** **-a** [**-fFnrsvw**] [**-t** <u>fstype</u>] [**-O** optlist]
**mount** [**-fnrsvw**] [**-o** <u>device</u>] <u>device</u>|<u>mountpoint</u>
**mount** [**-fnrsvw**] [**-t** <u>fstype</u>] [**-o** <u>device</u>] <u>device</u> <u>mountpoint</u>
**mount** **--bind**|**--rbind**|**--move** <u>olddir</u> <u>newdir</u>
**mount** **--make-**{**shared**|**slave**|**private**|**unbindable**|**rshared**|**rslave**|**rprivate**|**runbindable**} <u>mountpoint</u>
# 说明
在 Unix 系统中可访问的所有文件都排列在一棵大树中，即文件层次结构，以 / 为根。 这些文件可以分布在多个设备上。 **mount** 命令用于将在某个设备上找到的文件系统附加到大文件树上。 相反，**umount**(8) 命令则会将它分离。 文件系统用于控制数据如何存储在设备上或通过网络或其他服务以虚拟方式提供。

**mount** 命令的标准格式是：

&emsp;&emsp;**mount** **-t** <u>type</u> <u>device</u> <u>dir</u>

这告诉内核把在<u>device</u>（它的文件系统类型为<u>type</u>）上找到的文件系统附加到目录<u>dir</u>上。 选项 **-t** <u>type</u>是可选的。 **mount** 命令通常能够检测文件系统。 默认情况下，root 权限是挂载文件系统所必需的。 有关详细信息，请参阅下面的“[非超级用户挂载](#Nonsuperusermounts)”部分。 <u>dir</u>先前的内容（如果有）以及所有者和模式变得不可见，只要这个文件系统保持挂载，路径名 <u>dir</u> 就指向<u>device</u>上的文件系统的根目录。

如果只给出目录或设备，例如：

&emsp;&emsp;**mount /dir**

**mount**会在<u>/etc/fstab</u>文件中查找挂载点（如果未找到则查找设备）。 可以使用 **--target** 或 **--source** 选项来避免对给定参数的模棱两可的解释。 例如：

&emsp;&emsp;**mount --target /mountpoint**

同一个文件系统可能会被多次挂载，并且在某些情况下（例如，网络文件系统），同一个文件系统可能会被多次挂载在同一个挂载点上。 **mount** 命令不实施任何策略来控制此行为。 所有行为都由内核控制，并且通常特定于文件系统驱动程序。 例外情况是 **--all**，在这种情况下，已安装的文件系统将被忽略（有关详细信息，请参阅下面的 **[--all](#all)**）。

## 列出挂载目录

维护列表模式只是为了向后兼容。

要获得更强大和可定制的输出，请使用 **findmnt**(8)，**尤其是在您的脚本中**。 请注意，挂载点名称中的控制字符将替换为“?”。

以下命令列出所有已挂载的文件系统（类型为<u>type</u>）：

**mount** [**-l**] [**-t** <u>type</u>]


选项 **-l** 将标签添加到此列表中。 见下文。

## 指示设备和文件系统

大多数设备由（块特殊设备的）文件名表示，例如 <u>/dev/sda1</u>，但还有其他可能。 例如，在 NFS 挂载的情况下，<u>device</u>可能看起来像 <u>knuth.cwi.nl:/dir</u>。

磁盘分区的设备名是不稳定的； 硬件重新配置以及添加或删除设备可能会导致名称更改。 这就是强烈建议使用文件系统或分区标识符（如 UUID 或 LABEL）的原因。 当前支持的标识符（标签）：

&emsp;&emsp;LABEL=<u>label</u>
&emsp;&emsp;&emsp;&emsp;人类可读的文件系统标识符。另见**-L**。

&emsp;&emsp;UUID=uuid
&emsp;&emsp;&emsp;&emsp;文件系统通用唯一标识符。 UUID 的格式通常是一系列由连字符分隔的十六进制数字。另见**-U**。
&emsp;&emsp;&emsp;&emsp;请注意 **mount**(8) 使用 UUID 作为字符串。来自命令行或 **fstab**(5) 的 UUID 不会转换为内部二进制表示。 UUID 的字符串表示应基于小写字符。

&emsp;&emsp;PARTLABEL=<u>label</u>
&emsp;&emsp;&emsp;&emsp;人类可读的分区标识符。此标识符独立于文件系统，不会因 mkfs 或 mkswap 操作而更改。例如，支持 GUID 分区表 (GPT)。

              PARTUUID=uuid
                     分区通用唯一标识符。此标识符独立于文件系统，不会因 mkfs 或 mkswap 操作而更改。例如，支持 GUID 分区表 (GPT)。

              ID=id udevd 生成的硬件块设备 ID。此标识符通常基于 WWN（唯一存储标识符）并由硬件制造商分配。
                     有关详细信息，请参阅 ls /dev/disk/by-id，此目录和运行 udevd 是必需的。不建议将此标识符用于一般用途，因为该标识符没有严格定义，它取决于 udev、udev 规则和硬件。
					 
------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《mount manpages》。</font>当前版本为8.30，手册更新时间为2015-08。<font color=red>本文与原文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------

