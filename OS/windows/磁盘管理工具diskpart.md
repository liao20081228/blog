---
title: 磁盘管理工具diskpart
tags: windows
---

------

***<font color=blue>版权声明</font>：<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

# 1 简介
&emsp;&emsp;diskpart可实现对硬盘的分区管理，包括创建分区、删除分区、合并（扩展）分区，完全可取代分区魔术师等第三方工具软件，它还有分区魔术师无法实现的功能，如设置动态磁盘、镜像卷等，而且设置分区后不用重启电脑也能生效。只不过它运行于命令提示符下。

# 2 运行环境
&emsp;&emsp;Diskpart是Windows环境下的一个命令，正常运行该命令时需要系统服务的支持，这几个服务是：Logical Disk Manager Administrative Service(dmadmin)、Logical Disk Manager(dmserver)、Plug and Play(PlugPlay)、Remote Procedure Call (RPC) (RPCss)。而这四个服务的依存关系为：dmserver依赖于PlugPlay和RPCss，dmadmin/依赖于dmserver。
&emsp;&emsp;如果这四个服务没有运行，那么是不可以成功运行Diskpart的，所以在纯DOS下面是不能够运行这个命令的,但在WINPE下是可以运行DISKPART的，现在很多ODM厂商在安装操作系统时都采用WINPE环境，就是因为在WINPE下可方便的对硬盘进行操作(WINPE环境本身启动在内存中)
# 3 启动
&emsp;&emsp;在命令提示符窗口，再输入“Diskpart”即可启动它，此时屏上显示为“Diskpart>”，不像普通的命令提示符那样是一个分区或目录后跟一个“>”，而且普通的DOS命令也无法在此运行，退出它只有输入“Exit”命令。从这三种迹象表明，它是一个集成的环境，只有特定的命令可在其下执行。输入“Help”命令，屏上会列出所有的可执行命令及各命令的简要说明。

# 4 命令详解
* ACTIVE
	* 在格式化主启动记录(MBR)磁盘的磁盘上，将带焦点的分区标记为活动分区。将一个值写入启动时由基本输入输出系统(BIOS)读取的磁盘。该值指定分区是有效的系统分区。必须选择一个分区才能成功执行此操作。
	* 警告: DiskPart 仅验证该分区是否能够包含操作系统启动文件。 DiskPart 不会检查分区中的内容。如果错误地将某个分区标记为活动分区，而该分区不包含 操作系统启动文件，则计算机可能无法启动。

-----
* ADD DISK=N [ALIGN=N] [WAIT] [NOERR]——将带焦点的简单卷镜像到指定的磁盘。
	* DISK=N   指定一个含有现有简单卷的磁盘以外的磁盘来包含镜像。 可以仅镜像简单卷。指定的磁盘必须具有不低于要镜像的简单卷大小的未分配空间。
	* ALIGN=N   通常与硬件 RAID 逻辑单元号(LUN)阵列一起使用以提高性能。 将所有卷范围与最近的对齐边界对齐。范围偏移将为 N 的倍数。
	*  WAIT返回前，等待卷完成与添加的磁盘的同步。如果不使用 wait参数，DiskPart 会在创建镜像卷后返回，而不等待同步完成。
	* NOERR       仅用于脚本。遇到错误时，DiskPart 会继续处理命令，如同没有出现错误一样。如果不使用 NOERR 参数，错误会导致 DiskPart 退出并返回 错误代码。
	* 必须选择一个卷才能成功执行此操作。Windows Vista 的任何版本都不支持容错卷(如 RAID-5)和镜像卷。

-----
* ASSIGN [LETTER=D | MOUNT=PATH] [NOERR]——给所选卷分配一个驱动器号或装载点
	* LETTER=D  分配给卷的驱动器号。
	* MOUNT=PATH分配给卷的装入文件夹的路径名称。
	* NOERR       仅用于脚本。遇到错误时，DiskPart 会继续处理命令，如同没有出现错误一样。如果不使用 NOERR 参数，错误会导致 DiskPart 退出并返回错误代码。
	* 如果没有指定驱动器号或装入的文件夹，则会分配下一个可用的驱动器号。如果驱动器号或装入的文件夹已经在使用，则会生成一个错误。通过使用 ASSIGN 命令，你可以更改与可移动驱动器关联的 驱动器号。
   * 你无法为启动卷或含有页面文件的卷分配驱动器号。此外，你无法为原始设备制造商(OEM)分区(除非启动到 Windows PE)或除基本数据分区以外的任何 GUID 分区表(GPT)分区、ESP 分区或恢复分区分配驱动器号。必须选择一个卷才能成功执行此操作。

-----
* ATTRIBUTES——操纵卷或磁盘属性。
	* ATTRIBUTES DISK [SET | CLEAR] [READONLY] [NOERR]
		* SET         设置所选磁盘的指定属性。
		* CLEAR       清除所选磁盘的指定属性。
		* READONLY    指定磁盘为只读磁盘。
		* NOERR       仅用于脚本。遇到错误时，DiskPart会继续处理命令，如同没有出现错误一样。如果不使用 NOERR 参数，错误会导致 DiskPart 退出并返回错误代码。
		*  显示当前的磁盘标志。READONLY 当前是可以进行更改的唯一磁盘标志。BOOT DISK 属性表示用于启动的磁盘。对于动态镜像卷，为包含镜像启动卷的启动丛的磁盘显示 BOOT DISK 属性。
		*  不带参数显示所有属性
	* ATTRIBUTES VOLUME [SET | CLEAR] [HIDDEN | READONLY | NODEFAULTDRIVELETTER | SHADOWCOPY] [NOERR]
		*  SET 为所选卷设置指定属性(HIDDEN、READONLY 和NODEFAULTDRIVELETTER 或 SHADOWCOPY)。
		*  CLEAR       从所选卷清除指定属性(HIDDEN、READONLY、NODEFAULTDRIVELETTER 或 SHADOWCOPY)。
		*  HIDDEN      指定卷为隐藏卷。
		*  READONLY    指定卷为只读卷。
		*  NODEFAULTDRIVELETTER 指定默认情况下卷不接收驱动器号。
		*  SHADOWCOPY  指定卷为卷影副本卷。
		*  NOERR       仅用于脚本。出现错误时，DiskPart继续处理命令，如同没有出现错误一样。如果不使用 NOERR 参数，错误会导致 DiskPart 退出并返回错误代码。
		*  在基本主启动记录(MBR)磁盘上，HIDDEN、READONLY 和 NODEFAULTDRIVELETTER 属性适用于该磁盘上的所有卷。在基本 GUID 分区表(GPT)磁盘以及动态 MBR 和 GPT磁盘上，HIDDEN、READONLY 和 NODEFAULTDRIVELETTER 属性仅适用于所选卷。
		*  不带参数显示所有属性
-----
* ATTACH VDISK [NOERR] [READONLY] { [SD="SDDL string"] | [USEFILESD] }
	* NOERR       仅用于脚本。遇到错误时，DiskPart会继续处理命令，如同没有出现错误一样。如果不使用 NOERR 参数，错误会导致 DiskPart退出并返回错误代码。
	*  READONLY    以只读方式连接虚拟磁盘。任何写入操作都将返回输入/输出设备错误。
	*  SD="SDDL string" 以安全描述符定义语言(SDDL)格式指定安全描述符。 默认情况下，安全描述符号允许访问任何物理磁盘等内容。有关 SDDL 的详细信息，请参阅 CREATE VDISK 命令。
	* USEFILESD   指定应该在虚拟磁盘上使用虚拟文件自身上的安全描述符号。如果不指定，则磁盘将没有显式安全描述符，除非使用 SD=(SDDL string)指定。必须选择虚拟磁盘才能成功完成此操作。
---
*  AUTOMOUNT [ENABLE] [DISABLE] [SCRUB] [NOERR]——启用或禁用自动装载功能
	*  ENABLE      使 Windows 可以自动将驱动器号分配给已添加到系统的卷。
	*  DISABLE     防止 Windows 自动将驱动器号分配给已添加到系统的卷。
	*  SCRUB       删除不再位于系统中的卷的装入文件夹路径名称、驱动器号、装入文件夹目录和注册表设置。这可防止在将之前位于系统中的卷重新引入到系统时自动为这些卷分配它们以前的驱动器号和装入文件夹路径名称。
	*  NOERR       仅用于脚本。遇到错误时，DiskPart 会继续处理命令，如同没有出现错误一样。如果不使用 NOERR 参数，错误会导致 DiskPart退出并返回错误代码。
	*  启用自动装载功能(默认在 Windows Server 的同一版本上)时，Windows 会在将卷添加到系统时自动使该卷联机，并且为该卷分配一个驱动器号和卷 GUID 路径名称。在存储区域网络(SAN)配置中，禁用自动装载可防止 Windows 自动使该卷联机，并防止为系统中可见的任何新卷分配一个驱动器号和卷 GUID 路径名称。
	*  请注意，在 Windows Vista 之前发布的 Windows 版本上，自动装载功能只能用于基本磁盘卷。从 Windows Vista 开始，自动装载功能可用于基本磁盘卷和动态磁盘卷。
---
* BREAK DISK=N [NOKEEP] [NOERR]——将带焦点的镜像卷分为两个简单卷。
	* DISK=N    指定含有镜像卷一个副本的磁盘。命令完成后，此磁盘会获得焦点
但此磁盘上使用镜像卷范围创建的新卷不会保留已与该镜像卷关联的驱动器号、卷 GUID 路径名称或装入文件夹路径。如果将指定磁盘上的镜像卷用作系统或启动分区，则命令会失败。
	* NOKEEP      指定仅保留镜像卷的一个副本。磁盘 N 上的镜像卷范围会转换为可用空间。该镜像卷的其余副本和磁盘 N 上可用空间 都不会收到焦点。
	* NOERR       仅用于脚本。遇到错误时，DiskPart 会继续处理命令，如同没有出现错误一样。如果不使用 NOERR 参数，错误会导致 DiskPart退出并返回错误代码。
	* 仅适用于动态磁盘。将带焦点的镜像卷分为两个简单卷。 一个简单卷保留已与该镜像卷关联的驱动器号、卷 GUID 路径名称或装入文件夹路径。另一个简单卷接收焦点，因此你可以为它分配一个驱动器号(将为其自动分配一个卷 GUID 路径名称)。
	* 默认情况下，镜像这两个副本的内容都会保留。每个副本都成为一个简单卷。通过使用 NOKEEP 参数，你可以仅将镜像卷的一个副本保留为简单卷，将另一个副本转换为可用空间。任何卷不会接收焦点。必须选择一个镜像卷才能成功执行此操作。

-----
* CLEAN [ALL]——清除选定磁盘的所有数据
	* 指定将当前焦点的磁盘上的每个字节\扇区都设置为 0，这会完全删除该磁盘上包含的所有数据。在主启动记录(MBR)磁盘上，仅会覆盖 MBR 分区信息和隐藏的扇区信息。在 GUID 分区表(GPT)磁盘上，包括保护性 MBR 在内的 GPT 分区信息都会被覆盖。如果不使用 ALL 参数， 则磁盘的第一个 1MB 和最后一个 1MB 都将为零。这将擦除以前应用到该磁盘的任何格式化磁盘。清除该磁盘后的磁盘状态为 'UNINITIALIZED'。
-----
* COMPACT VDISK——压缩虚拟磁盘文件以减少文件的物理大小。只能对已分离的 EXPANDABLE 虚拟磁盘或以只读 方式连接的 EXPANDABLE 虚拟磁盘执行压缩操作。 必须选择虚拟磁盘才能成功完成此操作。

-----
* CONVERT BASIC|DYNAMIC|MBR|GPT——在不同的磁盘格式之间转换。
	* CONVERT BASIC [NOERR] ——将空的动态磁盘转换为基本磁盘。
		* NOERR       仅用于脚本。遇到错误时，DiskPart 会继续处理命令，如同没有出现错误一样。如果不使用 NOERR 参数，错误会导致DiskPart 退出并返回错误代码。
		* 必须选择一个动态磁盘才能成功执行此操作。 磁盘必须为空磁盘才能转换为动态磁盘。请在转换磁盘前备份数据，然后删除所有分区或卷。
	*  CONVERT DYNAMIC [NOERR]——将基本磁盘转换为动态磁盘。
		*    基本磁盘上的任何现有分区都会变为简单卷。必须选择一个基本磁盘才能成功执行此操作。
	* CONVERT MBR [NOERR]——将具有 GUID 分区表(GPT)分区形式的空基本磁盘转换为具有主启动记录(MBR)分区形式的基本磁盘。
		* 必须选择一个基本 GPT 磁盘才能成功执行此操作。若要将磁盘转换为 MBR 磁盘，该磁盘必须为空。请在转换磁盘前备份数据，然后删除所有分区或卷。
	* CONVERT GPT [NOERR]——将具有主启动记录(MBR)分区形式的空基本磁盘转换为具有 GUID 分区表(GPT)分区形式的基本磁盘。
		* 必须选择一个基本 MBR 磁盘才能成功执行此操作。 转换为 GPT 所需的最小磁盘大小是 128MB。若要将磁盘转换为 GPT 磁盘，该磁盘必须为空。请在转换磁盘前备份数据，然后删除所有分区或卷。

-----
*  CREAT PARTITION|VOLUME|VDISK——创建指定对象
	*  CREAT PARTITION ——创建分区
		* CREAT PARTITION PRIMARY [SIZE=N] [OFFSET=N] [ID={BYTE|GUID}] [ALIGN=N] [NOERR]——创建主分区
			*  SIZE=N    分区大小，以兆字节(MB)为单位。如果 未给定大小，则会继续创建分区，直到当前区域中没有 更多未分配的空间。
			*  OFFSET=N 创建分区所处的偏移量，以千字节(KB)为单位。如果未给定偏移量，则将分区放置在有足够空间容纳它的第一个磁盘盘区中。
			*  ID={BYTE | GUID}指定分区类型。仅适合原始设备制造商(OEM)使用。对于主启动记录(MBR)磁盘，可以为分区指定十六进制形式的 分区类型字节。如果未为 MBR 磁盘指定此参数，该命令将创建类型为 0x06 (指定不安装任何文件系统)的分区。
				*   LDM数据分区:0x42
				*   恢复分区:0x27
				*   识别的 OEM ID: 0x12、 0x84、0xDE、 0xFE、  0xA0
				*   对于 GUID 分区表(GPT)磁盘，可以为要创建的分区指定分区类型 GUID。已识别的 GUID 包括:
					*   EFI 系统分区: c12a7328-f81f-11d2-ba4b-00a0c93ec93b
					*   Microsoft 保留分区: e3c9e316-0b5c-4db8-817d-f92df00215ae
					*   基本数据分区:ebd0a0a2-b9e5-4433-87c0-68b6b72699c7
					*   动态磁盘上的 LDM 元数据分区:5808c8aa-7e8f-42e0-85d2-e1e90434cfb3
					*   动态磁盘上的 LDM 数据分区:af9b60a0-1431-4f62-bc68-3311714a69ad
					*   恢复分区:de94bba4-06d1-4d40-a16a-bfd50179d6ac
				*   如果未对 GPT 磁盘指定此参数，该命令将创建基本数据分区。
				*   可以使用此参数指定任何分区类型字节或 GUID。DiskPart 不会检查分区类型是否有效，只是确保此分区类型为十六进制形式字节或 GUID。
				*   警告: 使用此参数创建分区可能导致 计算机发生故障或无法启动。除非你是 OEM 或熟悉 GPT 磁盘的IT 专业人员，否则不要使用此参数在 GPT 磁盘上创建分区。而是应该在 GPT 磁盘上使用 CREATE PARTITION EFI 命令创建 EFI 系统分区，使用 CREATE PARTITION MSR命令创建 Microsoft 保留分区，使用不带此参数的 CREATE PARTITION PRIMARY 命令创建主分区。
			*   ALIGN=N   通常与硬件 RAID 逻辑单元号(LUN)阵列一同使用来提高性能。分区偏移将是 N 的倍数。如果指定了 OFFSET 参数，则将四舍五入为最接近的 N的倍数。
			*   NOERR 仅用于脚本。遇到错误时，DiskPart会继续处理命令，如同没有出现错误一样。如果不使用 NOERR 参数，错误会导致 DiskPart 退出并返回错误代码。
			*   注意：创建分区之后，焦点会自动移动到新分区。该分区未接收驱动器号。必须使用 assign 命令为该分区分配一个驱动器号。必须选择一个基本磁盘才能成功执行此操作。 如果未指定分区类型，磁盘未初始化且磁盘大小大于 2TB，则该磁盘将初始化为 GPT。	
		* CREAT PARTITION EFI   [size=n] [offset=n]  [NOERR]——创建EFI系统分区。创建分区之后，焦点会自动移动到新分区，只用于GPT磁盘。
		* CREAT PARTITION EXTENDED [size=n] [offset=n]  [ALIGN=n] [NOERR]——创建扩展分区。创建分区之后，焦点会自动移动到新分区，只用于MBR磁盘,每个磁盘只能有一个扩展分区。
		* CREAT PARTITION LOGICAL [size=n] [offset=n]  [NOERR] —— 创建逻辑驱动器。创建分区之后，焦点会自动移动到新分区，只用于MBR磁盘。
		* CREAT PARTITION MSR      [size=n] [offset=n]  [NOERR] —— 创建 Microsoft 保留分区。使用 create partition msr 命令时要小心。GPT 磁盘要求特定的分区布局，因此创建 Microsoft 保留分区可能导致磁盘不可读。在用于启动 Windows XP 64 位版本或 Windows Server 2003 家族 64 位版本的 GPT 磁盘上，EFI 系统分区是磁盘上的第一个分区，随后是 Microsoft 保留分区。仅用来储存数据的 GPT 磁盘没有 EFI 系统分区。Microsoft 保留分区是第一个分区。 
	* CREAT VOLUME——创建卷。
		* CREAT VOLUME RAID  [SIZE=N] DISK=N,N [ALIGN=N] [NOERR]——创建软件 RAID-5 卷集。
			* SIZE=N该卷在每个磁盘上占用的磁盘空间量， 单位为兆字节(MB)。如果未给定大小，新卷将占据较小磁盘上的剩余可用空间，并在其他磁盘上占据与此相同的空间量。
			* DISK=N,N 在其上创建镜像卷的动态磁盘。需要两个动态磁盘才能创建镜像卷。每个磁盘上分配等于 SIZE=N中指定的大小的空间量。
			* ALIGN=N   通常与硬件 RAID 逻辑单元号(LUN)阵列一起使用以提高性能。将所有的卷扩展到与最近的对齐边界对齐。扩展偏移将是 N 的倍数。
			* NOERR 仅用于脚本。遇到错误时，DiskPart会继续处理命令，如同没有出现错误一样。如果不使用 NOERR 参数，错误会导致 DiskPart 退出并返回错误代码。
		* CREAT VOLUME SIMPLE    [SIZE=N] DISK=N,N [ALIGN=N] [NOERR] ——创建简单卷。
		* CREAT VOLUME STRIPE    [SIZE=N] DISK=N,N [ALIGN=N] [NOERR]——创建带区卷集。
		* CREAT VOLUME MIRROR [SIZE=N] DISK=N,N [ALIGN=N] [NOERR]——创建镜像卷集。			
	* CREAT VDISK  FILE="filename" MAXIMUM=N [TYPE={FIXED|EXPANDABLE}] [SD="SDDL string"] [PARENT="filename"] [SOURCE="filename"] [NOERR]——创建虚拟磁盘
		* FILE="filename" 指定虚拟磁盘文件的完整路径和文件名。该文件可以位于网络共享上。
		*  MAXIMUM=N 虚拟磁盘公开的最大空间量，以兆字节(MB)为单位。
		*  TYPE={FIXED|EXPANDABLE}
			*  FIXED 指定固定大小的虚拟磁盘文件。
			*  EXPANDABLE指定重新调整大小以适应所分配数据的 虚拟磁盘文件。默认值为 FIXED。
		*   [SD="SDDL string"]指定安全描述符定义语言(SDDL)格式的安全描述符。默认情况下，安全描述符是从父目录中获取的。SDDL 字符串可能比较复杂，但很灵活。 其最简单的格式是保护访问的安全描述符，称为自定义访问控制列表(DACL)。其格式为:`D:<DACL_FLAGS>(<STRING_ACE>)(<STRING_ACE>)...(<STRING_ACE>)`
			*  DACL_FLAGS 为:
				*   "P" - DACL 不应被来自父容器的任何 ACL 所覆盖(保护)。VHD 或 VHDX 文件的容器就是 其目录。
				*   "AI"- DACL 应该从父容器自动继承。
			*  STRING_ACE 的格式为 `<ACE_TYPE>;;<RIGHTS>;;;<ACCOUNT_ID>`
				* 常用的 ACE_TYPE 为:
					*   "A" - 允许访问。
					* "D" - 拒绝访问。
				* 常用的 RIGHTS 为:
					* "GA" - 所有访问权限。
					* "GR" - 读取访问权限。
					* "GW' - 写入访问权限。
				* 常用的 ACCOUNT_ID 为:
					* "BA" - 内置管理员
					*  "AU" - 通过身份验证的用户。
					*  "CO" - 创建者所有者。
					*  "WD" - 任何人。
		*  [PARENT="filename"] 用于创建差异磁盘的现有父虚拟磁盘文件的路径。 对于 PARENT 参数，不应指定 MAXIMUM， 因为差异磁盘从其父目录获取大小。 而且，也不应指定 TYPE，因为只能创建 EXPANDABLE 差异磁盘。
		*   [SOURCE="filename"]用于预填充新虚拟磁盘文件的现有虚拟磁盘文件的路径。 当指定 SOURCE 时，输入虚拟磁盘文件中的数据 将从输入虚拟磁盘文件逐块复制到创建的 虚拟磁盘文件。但不建立父子关系。

-----
* DELETE DISK|PARTITION|VOLUME——删除指定对象
	* DISK——从磁盘列表删除遗失的磁盘。
	* PARTITION——删除所选分区。
	* VOLUME —— 删除所选卷。

-----
* DETAIL  DISK|PARTITION|VOLUME|VDISK ——显示指定对象的详细信息
	* DISK—— 显示选中的磁盘的属性。
	* PARTITION ——显示选中的分区的属性。
	* VOLUME ——显示选中的卷的属性。
	* VDISK —— 显示所选虚拟磁盘的属性。

------
* DETACH VDISK [NOERR]— 断开指定的虚拟磁盘。
	* NOERR       仅用于脚本。遇到错误时，DiskPart会继续处理命令，如同没有出现错误一样。如果不使用 NOERR 参数，错误会导致 DiskPart 退出并返回错误代码。
	* 必须选择虚拟磁盘才能成功完成此操作。

-----
* EXIT——退出 DiskPart。

-----
 * EXTEND [SIZE=N] [DISK=N] [NOERR] 
 * EXTEND FILESYSTEM [NOERR]——将带焦点的卷或分区及其文件系统扩展到磁盘上
    的可用(未分配)空间。
    * SIZE=N    指定要添加到当前卷或分区的空间量(以兆字节(MB)为单位)。如果未给定大小，则使用可在磁盘上获得的所有连续可用空间。
    * DISK=N   指定要在其上扩展该卷或分区的磁盘。如果未指定磁盘，则在当前磁盘上扩展该卷或分区。
    * FILESYSTEM  扩展带焦点的卷的文件系统以覆盖整个卷。仅在未使用该卷扩展文件系统的卷上使用。
    * NOERR       仅用于脚本。遇到错误时，DiskPart 会继续处理命令，如同没有出现错误一样。如果不使用NOERR 参数，错误会导致 DiskPart退出并返回错误代码。
    * 在基本磁盘上，可用空间与带焦点的卷或分区必须位于同一个磁盘上。可用空间还必须紧跟在带焦点的卷或分区之后(在下一个扇区偏移处开始)。
    * 在具有简单卷或跨区卷的动态磁盘上，可以将卷扩展到任何动态磁盘上的任何可用空间。使用该命令，可以将简单的动态卷转换成跨区的动态卷。不能扩展镜像卷、RAID-5卷和带区卷。
    * 如果先前已使用 NTFS 文件系统格式化分区，则会自动扩展该文件系统以填充更大分区。不会出现数据丢失。如果先前使用 NTFS 之外的其他文件系统格式化分区，则该命令将会失败，不会对分区进行任何更改。如果先前未使用文件系统格式化该分区，则仍然会 扩展该分区。
    * 必须选择一个卷或分区才能继续执行此操作。

-----
* EXPAND VDISK MAXIMUM=N——扩展虚拟磁盘上可用的最大大小。
	*  MAXIMUM=N    指示磁盘的新虚拟大小，单位为兆字节(MB)。
	*  必须选择虚拟磁盘才能成功完成此操作。 只能扩展已断开连接的磁盘。

-----
* FILESYSTEMS——显示有关选定卷的当前文件系统以及用于格式化该卷的受支持文件系统的信息。 必须选择一个卷才能成功执行此操作。

------
* FORMAT [[FS=fs] [REVISION=X.XX] | RECOMMENDED] [LABEL="label"] [UNIT=N] [QUICK] [COMPRESS] [OVERRIDE] [DUPLICATE] [NOWAIT] [NOERR]——格式化指定卷以便在 Windows 中使用。
	* FS=FS    指定文件系统的类型。如果未指定文件系统，则使用 FILESYSTEMS 命令显示的默认文件系统。
	* REVISION=X.XX 指定文件系统修订版(如果适用)。
	* RECOMMENDED 如果指定，则使用建议的文件系统和修订版(如果存在)，而不使用默认系统。可以通过FILESYSTEMS 命令显示建议的文件系统(如果存在)。
	* LABEL="label" 指定卷标。
	* UNIT=N   替代默认分配单元大小。对于一般用途，强烈建议使用默认设置。可以通过 FILESYSTEMS 命令显示特定文件系统的默认分配单元大小。对于大于 4096 的分配单元大小，不支持 NTFS压缩。
	* QUICK       执行快速格式化。
	* COMPRESS    仅限于 NTFS: 默认情况下压缩在新卷上创建的文件。
	* OVERRIDE    如果必要，可强制先卸载文件系统。该卷所有打开的句柄将不再有效。
	* DUPLICATE   仅限于 UDF: 该标志应用于 UDF 格式，版本 2.5 或更高。该标志指示格式化操作将文件系统元数据复制到磁盘上的另一扇区集。复制的元数据由操作系统使用，例如，用于修复或恢复应用程序。如果发现主元数据扇区已损坏，则将从复制的扇区中读取文件系统元数据。
	* NOWAIT      强制该命令在格式化正在进行的过程中立即返回。如果未指定 NOWAIT，DiskPart 将以百分比形式显示格式进度。
	* NOERR       仅用于脚本。遇到错误时，DiskPart 会继续处理命令，如同没有出现错误一样。如果不使用 NOERR 参数，错误会导致 DiskPart退出并返回错误代码。
	* 必须选择一个卷才能继续执行此操作。

-----
 * GPT ATTRIBUTES=N——在基本 GUID 分区表(GPT)磁盘上，将 GPT 属性分配给带焦点的分区。
	 * ATTRIBUTES=N  表示可应用于带焦点的分区的属性的十六进制值。GPT 属性字段是一个 64 位字段，该字段包含两个子字段。高位字段只有在特定分区类型 GUID 的环境中才进行解释，而低位字段则是所有分区类型公用的字段。
	 * 可以在所有分区上设置以下属性:0x0000000000000001 将该分区标记为必需分区。这向所有的磁盘管理实用工具表明该分区不应被删除。EFI 系统分区仅包含启动操作系统所需的二进制数据。这样可以轻松地将原始设备制造商(OEM)或操作系统特定的二进制数据放置在其他分区中。
	 * 对于基本数据分区，定义了以下属性:
		 * 0x8000000000000000  GPT_BASIC_DATA_ATTRIBUTE_NO_DRIVE_LETTER  防止为分区自动分配驱动器号。默认情况下，会为每个分区分配一个新驱动器号和卷 GUID 路径名。驱动器号和卷 GUID 路径名都可用于打开使用 Win32 API 的卷。设置此属性可确保在将磁盘移动到新计算机上时，系统不会自动生成新的驱动器号。而用户可以手动分配驱动器号。
		 * 0x4000000000000000 GPT_BASIC_DATA_ATTRIBUTE_HIDDEN 将分区的卷定义为隐藏。设置此属性可指定不会为卷分配驱动器号或卷 GUID 路径名。 驱动器号和卷 GUID 路径名都可用于打开使用 Win32 API 的卷。Win32 API 不会报告隐藏分区的卷，例如 FindFirstVolume 和 FindNextVolume。
		 *  0x2000000000000000 GPT_BASIC_DATA_ATTRIBUTE_SHADOW_COPY将分区定义为卷快照服务卷影副本卷。文件系统筛选器使用此标志阻止筛选器附加到卷中。
		 *  0x1000000000000000  GPT_BASIC_DATA_ATTRIBUTE_READ_ONLY 阻止写入卷。
		 *   Microsoft 可能会随时添加其他属性。
		 *   GPT 分区属性提供有关该分区使用情况的附加信息。
		 *   必须选择一个基本 GPT 分区才能继续执行此操作。
		 *   更改 GPT 属性可能会导致无法为基本数据卷分配驱动器号，或阻止装载文件系统。除非你是原始设备制造商(OEM)或熟悉 GPT 磁盘的 IT 专业人士，否则请勿更改 GPT 属性。

------
* HELP [COMMAND]——显示帮助信息
	* COMMAND——  要显示其详细帮助的命令。 如果未指定命令，HELP 将显示所有可能的命令。示例:`HELP CREATE PARTITION PRIMARY`

------
* IMPORT [NOERR]——将外部磁盘组导入到本地计算机的联机磁盘组。
	* NOERR       仅用于脚本。遇到错误时，DiskPart 会继续处理命令，如同没有出现错误一样。如果不使用 NOERR 参数，错误会导致 DiskPart 退出并返回错误代码。
	*  此命令将位于同一组中的每个磁盘作为带焦点的磁盘导入。
	*  必须选择动态磁盘才能继续执行该操作。

-----
* INACTIVE——在格式为主启动记录(MBR)磁盘的磁盘上，将带焦点的分区标记为非活动分区。
	* 重新启动计算机时，计算机将从 BIOS 中指定的下一选项启动，例如 CD-ROM驱动器或基于预启动执行环境(PXE)的启动环境(例如远程安装服务(RIS))。
	* 必须选择一个分区才能成功执行此操作。
	* 如果没有活动分区，则计算机可能无法启动。请勿将系统分区或启动分区标记为非活动分区，除非你是对 Windows 存储管理有着透彻理解的经验丰富的用户。

-----
* LIST DISK|PARTITION|VOLUME|VDISK——显示对象列表
	* DISK —— 显示磁盘列表。例如，`LIST DISK`。
	* PARTITION——显示所选磁盘上的分区列表。例如，`LIST PARTITION`。
	* VOLUME ——显示卷列表。例如，`LIST VOLUME`。
	* VDISK——显示虚拟磁盘列表。

------
* MERGE VDISK DEPTH=N——将子磁盘与其父磁盘合并。
	* DEPTH=N   指示要一起合并的父磁盘数。例如，MERGE=1 指示要合并一个级别的差异链。
	* 必须选择虚拟磁盘才能成功完成此操作。

------
* ONLINE DISK|VOLUME [NOERR]——使当前标为脱机的对象联机。
使所选脱机磁盘处于联机状态。
	* DISK 此命令在单个磁盘上运行。必须选择一个磁盘才能继续执行此操作。此命令在处于 SAN 脱机模式的磁盘上运行。 如果为动态磁盘，且为只读状态、脱机，并且所需状态为读/写、联机， 则建议操作顺序为先将磁盘设为读/写，然后将磁盘联机。如果动态只读磁盘联机，其状态将为“错误”。原因为当动态磁盘联机时，卷管理器必须写入磁盘上的动态磁盘数据库，以便更新其状态。如果动态磁盘只读，这些写操作将会失败。
	* VOLUME  此命令在单个卷上运行。必须选择一个卷才能继续执行此操作。不能在 OEM、ESP 或恢复分区上运行此命令。
	* NOERR       仅用于脚本。遇到错误时，DiskPart 会继续处理命令，如同没有出现错误一样。如果不使用 NOERR 参数，错误会导致 DiskPart 退出并返回错误代码。

------
* OFFLINE  DISK|VOlUME [NOERR] ——使当前标记为联机的对象脱机。
	* DISK 使所选的联机磁盘处于脱机状态。此命令在单个磁盘上运行。必须选择一个磁盘才能继续执行此操作。此命令在处于 SAN 联机模式的磁盘上运行。不能在 OEM、ESP 或恢复分区上运行此命令。
	* VOLUME 使所选的联机卷处于脱机状态。 此命令在单个卷上运行。 必须选择一个卷才能使此操作成功。
	* NOERR       仅用于脚本。遇到错误时，DiskPart 会继续处理命令，如同没有出现错误一样。如果不使用 NOERR 参数，错误会导致 DiskPart 退出并返回错误代码。

-----
* RECOVER [NOERR]  刷新所选包中所有磁盘的状态，尝试恢复无效包中的磁盘，并重新同步具有过时丛或奇偶校验数据的镜像卷和 RAID-5 卷。
	* NOERR       仅用于脚本。遇到错误时，DiskPart 会继续处理命令，如同没有出现错误一样。如果不使用 NOERR 参数，错误会导致 DiskPart 退出并返回错误代码。
	* 此命令对包执行操作。 必须选择属于该包的一个磁盘才能继续执行此操作。
	* 此命令仅适用于动态磁盘。如果在基本磁盘上调用，则该命令会成功，但不执行任何操作。

------
 * REM comments——提供向脚本添加注释的方法。

-----
* REMOVE [LETTER=D | MOUNT=Path | ALL] [DISMOUNT] [NOERR]——从带焦点的卷中删除驱动器号或装入的文件夹路径名。
	* LETTER=D  要删除的驱动器号。
	*  MOUNT=Path 要删除的装入文件夹的路径。
	*  ALL         删除当前的所有驱动器号和装入的文件夹路径。
	*  DISMOUNT    以下情况下会使用此参数: 1) 已从卷中删除所有的驱动器号及装入的文件夹路径，2) 指定了 ALL 参数。该参数指定即将卸载文件系统并使卷脱机。如果其他进程正在使用该卷，DiskPart将在卸载文件系统并使卷脱机之前关闭任何打开的句柄。通过为卷分配驱动器号或创建到卷的装入文件夹路径，或者使用 ONLINE 命令，可以使卷联机。如果使用的卷上具有任何剩余的驱动器号或装入文件夹路径，则 DISMOUNT 将会失败。对于脚本，建议使用 REMOVE ALL DISMOUNT。
	* NOERR       仅用于脚本。遇到错误时，DiskPart 会继续处理命令，如同没有出现错误一样。如果不使用 NOERR 参数，错误会导致 DiskPart 退出并返回错误代码。
    * 如果未指定驱动器号或装入文件夹路径，DiskPart 将删除遇到的第一个驱动器号或装入文件夹路径。如果使用了 ALL 参数，则将删除当前的所有驱动器号和装入文件夹路径。如果使用了 DISMOUNT 参数，DiskPart 将关闭所有已打开的指向该卷的句柄，然后卸载该卷并使其脱机。
    *  REMOVE 命令可用于更改与可移动驱动器相关联的驱动器号。无法删除启动卷或页面卷上的驱动器号。必须选择一个卷才能继续执行此操作。

-----
* REPAIR DISK=N [ALIGN=N] [NOERR]——通过使用指定的动态磁盘替换出现故障的 RAID-5 成员修复带焦点的 RAID-5 卷。
	* DISK=N   指定将替换出现故障的 RAID-5 成员的动态磁盘。指定磁盘的可用空间必须等于或大于出现故障的 RAID-5 成员的总大小。
	* ALIGN=N>   通常与硬件 RAID 逻辑单元号(LUN)阵列一起使用以便提高性能。将所有卷范围与最近的对齐边界对齐。范围偏移将为 N 的倍数。
	* NOERR       仅用于脚本。遇到错误时，DiskPart 会继续处理命令，如同没有出现错误一样。如果不使用 NOERR 参数，错误会导致 DiskPart 退出并返回错误代码。
	* 指定的动态磁盘可用空间必须大于或等于出现故障的 RAID-5 成员的大小。
	* 必须选择一个 RAID-5 卷才能继续执行此操作。

------
* RESCAN——重新扫描计算机，查找磁盘和卷。

------
* RETAIN——在一个简单卷下放置一个保留分区。 准备要用作启动卷或系统卷的现有动态简单卷。
	* 该命令为带焦点的动态简单卷创建分区项。
	* 必须选择一个动态简单卷才能成功执行此操作。
	
-------	
* SAN [POLICY={OnlineAll | OfflineAll | OfflineShared | OfflineInternal}] [NOERR]	——显示或设置用于操作系统的 SAN 策略。
	* 如果没有为该命令提供任何参数，将显示当前的SAN 策略。
	*  POLICY=value  设置当前启动的操作系统的 SAN 策略。
	* NOERR       仅用于脚本。遇到错误时，DiskPart 会继续处理命令，如同没有出现错误一样。如果不使用 NOERR 参数，错误会导致 DiskPart 退出并返回错误代码。
	* 此命令允许用户显示或更改当前启动的操作系统的 SAN 策略。Windows Advanced Server 和 Windows Data Center 的默认策略是 OfflineShared。在这种情况下， 将使启动磁盘联机，就像使不位于共享总线(如 SCSI、iSCSI、SAS 等)上的所有磁盘联机一样。默认情况下，脱机磁盘将为只读磁盘。在所有其他版本的 Windows 上，默认情况下将使 所有磁盘联机。在这种情况下，磁盘将处于联机和读/写状态。第三方策略值为 OfflineAll。对于这种情况，默认情况下，除启动磁盘外的所有磁盘都处于脱机和只读状态。前一策略值为 OfflineInternal，默认情况下将新发现的内部 磁盘保持为脱机和只读状态。

-----
* SELECT DISK|PARTITION|VOLUME|VDISK n —— 选中对象
	* DISK——将焦点移动到磁盘。例如，`SELECT DISK`。
	* PARTITION——将焦点移动到分区。例如，`SELECT PARTITION`。
	* VOLUME——将焦点移动到卷。例如，`SELECT VOLUME`。
	* VDISK——将焦点转移到虚拟磁盘。例如，`SELECT VDISK`。

>partition与volume区别：分区是基本磁盘上的概念，也叫基本卷；卷是动态磁盘的概念，也叫动态卷。

-----
* SET ID={BYTE | GUID} [OVERRIDE] [NOERR]——为带焦点的分区更改分区类型字段。
	*  ID={BYTE | GUID} 指定新分区类型。
		*  对于主启动记录(MBR)磁盘，可以采用十六进制形式为分区指定分区类型字节。可以使用此参数指定任何分区类型字节，类型 0x42 (LDM 分区)除外。请注意，指定十六进制分区类型时，会忽略前导的 "0x"。
		*  对于 GUID 分区表(GPT)磁盘，可以为分区指定分区类型 GUID。已识别的 GUID 包括:
			*  EFI 系统分区:c12a7328-f81f-11d2-ba4b-00a0c93ec93b
			*  基本数据分区: ebd0a0a2-b9e5-4433-87c0-68b6b72699c7
			*  可以使用此参数指定任何分区类型 GUID，以下几种类型除外:
				*   Microsoft 保留分区:e3c9e316-0b5c-4db8-817d-f92df00215ae
				*   动态磁盘上的 LDM 元数据分区:5808c8aa-7e8f-42e0-85d2-e1e90434cfb3
				*   动态磁盘上的 LDM 数据分区:af9b60a0-1431-4f62-bc68-3311714a69ad
				*    群集元数据分区: db97dba9-0840-4bae-97f0-ffb9a327c7e1
			*   除提及的限制之外，DiskPart 不会检查分区类型是否有效，只是确保该分区类型是十六进制形式的字节或 GUID。
   *  OVERRIDE    启用 DiskPart 能够强制首先卸载卷上的文件系统， 然后再更改分区类型。更改分区类型时，DiskPart将尝试锁定和卸载卷上的文件系统。如果未指定此参数，并且锁定文件系统的调用失败(因为某些 其他应用程序具有卷的打开句柄)，则整个操作将失败。指定此参数时，将强制执行卸载，即使锁定文件系统的调用失败也是如此。卸载文件系统后，卷的所有打开句柄都将无效。
   *  NOERR       仅用于脚本。遇到错误时，DiskPart 会继续处理命令，如同没有出现错误一样。如果不使用 NOERR 参数，错误会导致 DiskPart退出，并返回错误代码。
   *  仅供原始设备制造商(OEM)使用。
   *  必须选择一个分区才能成功执行此操作。
   *  使用此参数更改分区类型字段可能会导致计算机发生故障或无法启动。除非你是 OEM 或熟悉 GPT 磁盘的 IT 专业人员，否则不要使用此参数更改 GPT 磁盘上的分区类型字段。不过，始终可以在 GPT 磁盘上使用CREATE PARTITION EFI 命令创建 EFI 系统分区，使用CREATE PARTITION MSR 命令创建 Microsoft 保留分区以及使用不带 ID 参数的 CREATE PARTITION PRIMARY命令在 GPT 磁盘上创建主分区。
   * 此命令在动态磁盘或 Microsoft 保留分区上无法使用。

-----
* SHRINK [DESIRED=N] [MINIMUM=N] [NOWAIT] [NOERR]
* SHRINK QUERYMAX [NOERR]——使带焦点的卷的大小减少指定的量。从该卷末尾的未使用空间创建可用磁盘空间。
	* DESIRED=N 指定要该卷减少的空间大小，单位为兆字节(MB)。 如果未指定所需量，则该卷减少的量为该卷上最大可用空间量。
	* MINIMUM=N 指定要该卷减少的最小空间量，单位为 MB。
	*  QUERYMAX    返回该卷可以减少的最大字节数(该卷上的可用空间)。 如果应用程序当前正在访问该卷，则该值可能会发生变化。
	*   NOWAIT      强制该命令在收缩过程仍在进行时立即返回。
	*   NOERR       仅用于脚本。出现错误时，DiskPart 会继续处理命令， 如同未出现错误一样。如果未使用 NOERR 参数，错误会导致 DiskPart 退出并返回错误代码。
	* 如果未指定 MINIMUM 量，则该卷将减少 DESIRED 量(如果已指定)或该卷上可用空间的最大量。如果已指定 MINIMUM 量，但没有足够的可用空间可用，则该命令将会失败。
	* 此命令在基本卷以及简单卷或跨区动态卷上有效。只有该卷使用 NTFS 文件系统格式化或该卷上没有文件系统时，才可以减少卷的大小。
	* 必须选择一个卷才能继续执行此操作。不能在 OEM、ESP 或恢复分区上执行此命令。

-----
 * UNIQUEID DISK [ID={DWORD | GUID}]  [NOERR]——显示或设置带焦点的磁盘的 GUID 分区表(GPT)标识符或主启动记录(MBR)签名。
	 * ID={DWORD | GUID}  对于 MBR 磁盘，可为签名以十六进制形式指定一个四字节(DWORD)值。对于 GPT 磁盘，指定 GUID 作为标识符。
	 * NOERR       仅用于脚本。遇到错误时，DiskPart会继续处理命令，如同没有出现错误一样。如果不使用 NOERR 参数，错误会导致 DiskPart 退出 并返回错误代码。
	 * 必须选择一个磁盘才能使此操作成功。此命令在基本和动态磁盘上运行。
------

***<font color=blue>版权声明</font>：<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------