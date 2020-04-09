---
title: Linux内核的make目标详解
tags: Linux内核构建系统
---


------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：<font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

# 1 清理目标
* `clean`—— 删除大多数产生的文件，但保留配置和足够的构建支持以构建外部模块
* `mrproper`—— 删除所有产生的文件+ config +各种备份文件
* `distclean`—— `$make mrproper`+删除编辑器备份文件和补丁文件

# 2 配置目标
 * 手动配置
	 * `config`——基于文本的配置界面
	 * `nconfig`——基于ncurses的配置界面
	 * `menuconfig`——基于菜单程序的配置界面
	 * `xconfig`——使用基于Qt的配置界面
	 * `gconfig`——使用基于GTK+的配置界面
 * 快速配置
	 * `oldconfig`——使用之前配置好的.config作为基础更新当前配置
	 * `localmodconfig `——更新当前配置，禁用当前内核未加载的模块
	 * `localyesconfig`——更新当前配置，并将模块转为内核内建
	 * `defconfig`——使用ARCH提供的defconfig默认配置
	 *  `xxx_defconfig`——使用xxx的默认配置
	 *  `silentoldconfig `——类似于oldconfig，只是会静默的更新依赖关系
	 * `savedefconfig `——将当前系统的内核配置保存为./defconfig（最小配置）
	 * `allnoconfig`——新配置，其中所有选项均设置为no
	 * `allyesconfig`——新配置，其中所有选项均设置为yes
	 * `allmodconfig`——新配置，其中所有选项均尽可能设置为module
	 * `alldefconfig`——新配置，其中所有选项均设置为默认值
	 * `randconfig`——新配置，其中所有选项均设置为随机值
	 * `listnewconfig`——列出新选项
	 * `olddefconfig`——与oldconfig相同，但在没有提示的情况下将新符号设置为其默认值
	 * `kvmconfig`——为kvm来宾内核支持启用其他选项
	 * `xenconfig`——为xen dom0和来宾内核支持启用其他选项
	 * `tinyconfig`——配置最小的内核
	 * `testconfig`——运行 Kconfig 单元测试 (要求 安装python3 和 pytest)
	
# 3 其它通用目标
* 
* `all`——编译本文中所有加粗的目标
* **`vmlinux`**—— 构建vmlinux格式的内核
* **`modules`**——构建所有的模块
*  `modules_install`——安装所有的module到`INSTALL_MOD_PATH (default: /)`
*  `firmware_install`——安装所有的firmware到 `INSTALL_FW_PATH(default:$(INSTALL_MOD_PATH)/lib/firmware)`
*  `dir/` —— 构建dir目录下的所有的文件
* `dir/file.[ois]`——编译构建指定目标
* `dir/file.ll`——构建LLVM程序集文件（需要编译器支持LLVM程序集生成）
* `dir/file.lst `—— 仅构建指定的混合源/程序集目标（需要最近的binutils和最近的构建（System.map））
* `dir/file.ko`—— 构建包括最终链接的模块
* ` modules_prepare` —— 设置用于构建外部模块
*  `tags/TAGS` ——为编辑器生成tags文件
*  ` cscope`—— 生成cscope 索引
*  ` gtags`——生成 GNU GLOBAL索引
*  ` kernelrelease`—— 显示发行版本 (与 make——s 一起使用)
*  ` kernelversion`—— 输出存放在makefile中的版本号 ((与 make -s 一起使用))
*  ` image_name`——输出镜像名称(与 make -s 一起使用))
*  ` headers_install`——将清洁的的内核头文件安装到INSTALL_HDR_PATH（默认值：./ usr）
# 4 静态分析器
*   `checkstack ` —— 生成栈列表
*   ` namespacecheck ` ——分析名称空间
*   ` versioncheck `   —— 对version.h 用法的完整性检查
*   `includecheck`    —— 检查重复引入头文件
*   `export_report `  —— 列出所有导出符号的用法
*  ` headers_check `  —— 对导出的头文件进行完整性检查 
*  ` headerdep `—— 检查是否有头文件循环引用  
*   `coccicheck `——使用coccicheck进行检查
# 5 内核自测

*  `kselftest `——构建并运行内核自检（以root身份运行）,在运行kselftest之前构建，安装和启动内核
*  `kselftest-clean `—— 删除所有生成的kselftest文件
*  `kselftest-merge` —— 将kselftest的所有配置依赖项合并到现有的.config中。

# 6 用户空间工具
* 用户工具是单独的makefile控制的
	* `make tools/help`或  `cd tools; make help`——查看帮助
	* `make -C tools/ <tool>_install` 或`cd tools; make <tool>_install`——构建并安装指定工具
	*  `make tools/all`或 `cd tools; make all`——构建所有工具
	*  `make tools/install`或  `cd tools; make install`——安装所有工具
	*  ` make tools/<tool>_clean`或  `cd tools; make <tool>_clean`——清除指定工具生成的文件
	*  `make tools/clean`或  `cd tools; make clean`——清除所有工具生成的文件
* 用户工具
	* `acpi`—— ACPI 工具
	* `cgroup`—— cgroup 工具
	* `cpupower`—— 适用于所有x86 CPU电源的工具
	* `firewire`—— nosy的用户空间部分，IEEE-1394流量嗅探器
	*  `freefall`—— 用于磁盘保护的笔记本电脑加速计程序
	* ` gpio`—— GPIO工具
	* `hv`—— 在Hyper-V客户端中使用的工具
	* `iio`—— IIO 工具
	* `kvm_stat`—— 用于显示kvm统计信息的顶级实用程序
	* `leds`—— LEDs 工具
	* `liblockdep`——用于内核锁定验证程序的用户空间包装器
	* `bpf`—— misc BPF 工具
	* `perf`—— Linux性能测试和分析工具
	* `selftests`—— 各种内核自测工具
	* `spi`—— spi 工具
	* `objtool`—— 一个ELF object 分析工具
	* `tmon`—— 温度监控和调整工具
	* `turbostat`——
	* `usb`—— USB测试模块
	* `virtio`—— vhost测试模块
	* `vm`—— misc虚拟内存工具
	* `wmi`——WMI 接口示例
	* `x86_energy_perf_policy`—— 英特尔电源策略工具


# 7 内核打包
* `rpm-pkg`—— 构建包含源和二进制的rpm包
* `binrpm-pkg`——构建只含二进制的rpm包
* `deb-pkg`——构建包含源和二进制的deb包
* `bindeb-pkg`——构建只含二进制的deb包
* `snap-pkg` —— 构建只含二进制的snap包
* `tar-pkg `—— 将内核打包，不压缩
* `targz-pkg`—— 将内核打包，并用gzip压缩
* `tarbz2-pkg`—— 将内核打包，并用bzip2压缩
* `tarxz-pkg`—— 将内核打包，并用xz压缩
* `perf-tar-src-pkg`—— 构建perf-version.tar 源码包，未压缩
* `perf-targz-src-pkg`—— 构建 perf-version.tar.gz 源码压缩包，使用gzip压缩
* `perf-tarbz2-src-pkg`—— 构建 perf-version.tar.bz2 源码压缩包，使用把bzip2压缩
* `perf-tarxz-src-pkg`—— 构建 perf-version.tar.xz 源码压缩包，使用xz压缩

# 8 文档目标
* 指定内核文档的格式，默认输出位置为 Documentation/output
	* `htmldocs`        —— HTML
	*  `latexdocs`       —— LaTeX
	* `pdfdocs `        —— PDF
	* `epubdocs `       —— EPUB
	* `xmldocs `        —— XML
	* `linkcheckdocs`   —— 检查外部链接是否损坏（将连接到外部主机）
	* `refcheckdocs`    —— 检查是否有对Documentation目录下不存在文件的引用
	* `cleandocs`       ——清除生成的文件
* ` make SPHINXDIRS="s1 s2" [target]`—— 仅生成文件夹s1，s2的文档，SPHINXDIRS的有效值是：
	* core-api 
	* networking 
	* input sound 
	* userspace-api 
	* gpu 
	* media 
	* process 
	* maintainer 
	* doc-guide 
	* crypto 
	* vm 
	* sh 
	* dev-tools
	* driver-api 
	* filesystems 
	* kernel-hacking 
	* admin-guide
* ` make SPHINX_CONF={conf——file} [target]` ——使用额外的 sphinx-build配置。
  
# 9 特定架构目标
* x86
	* **`bzImage`**——压缩内核镜像 (arch/x86/boot/bzImage)
    * `install` ——使用`~/bin/installkernel`或 ` (distribution) /sbin/installkernel `安装内核 或者安装到 `$(INSTALL_PATH)`并运行 lilo
	* `fdimage`      —— 创建 1.4MB软盘启动镜像 (arch/x86/boot/fdimage)
	* `fdimage144`—— 创建 1.4MB软盘启动镜像 (arch/x86/boot/fdimage)
	* `fdimage288`—— 创建 2.8MB软盘启动镜像 (arch/x86/boot/fdimage)
	* `isoimage`—— 创建 CD-ROM启动镜像 (arch/x86/boot/image.iso)， 可以跟随以下参数
		* FDARGS="..." ——引导内核的参数 
		* FDINITRD=file ——引导内核的initrd
	* 特定默认配置
		*  i386_defconfig——  i386默认配置
		* x86_64_defconfig—— x86_64默认配置
* arm
	* **`zImage`**——压缩内核镜像 (arch/arm/boot/zImage)
	*  `Image` ——未压缩内核镜像 (arch/arm/boot/Image)
	* **`xipImage`**——XIP 内核镜像(arch/arm/boot/xipImage)
	* `uImage`——U-Boot专用内核镜像
	* `bootpImage` ——包含zImage和 initial RAM disk的镜像 ( 通过 make variable INITRD=path来提供initrd image)
	* **`dtbs`** —— 建立设备树
	* `dtbs_install` —— 安装设备树到/boot/dtbs/
	* `install`——安装未压缩的内核镜像
	* `zinstall`——安装压缩的内核镜像
	* `uinstall`——安装uboot内核镜像，使用`~/bin/installkernel`或 ` (distribution) /sbin/installkernel `安装内核 或者安装到 `$(INSTALL_PATH)`并运行 lilo
	* `vdso_install` —— 安装unstripped vdso.so 到$(INSTALL_MOD_PATH)/vdso
	* 特定默认配置
		* acs5k_defconfig—— Build for acs5k
		* acs5k_tiny_defconfig—— Build for acs5k_tiny
		* am200epdkit_defconfig—— Build for am200epdkit
		* aspeed_g4_defconfig—— Build for aspeed_g4
		* aspeed_g5_defconfig—— Build for aspeed_g5
 		* assabet_defconfig—— Build for assabet
 		* at91_dt_defconfig—— Build for at91_dt
		* axm55xx_defconfig—— Build for axm55xx
		* badge4_defconfig—— Build for badge4
		* bcm2835_defconfig—— Build for bcm2835
		* cerfcube_defconfig—— Build for cerfcube
		* clps711x_defconfig—— Build for clps711x
		* cm_x2xx_defconfig—— Build for cm_x2xx
		* cm_x300_defconfig—— Build for cm_x300
		* cns3420vb_defconfig—— Build for cns3420vb
		* colibri_pxa270_defconfig—— Build for colibri_pxa270
		* colibri_pxa300_defconfig—— Build for colibri_pxa300
		* collie_defconfig—— Build for collie
		* corgi_defconfig—— Build for corgi
		* davinci_all_defconfig—— Build for davinci_all
		* dove_defconfig—— Build for dove
		* ebsa110_defconfig—— Build for ebsa110
		* efm32_defconfig—— Build for efm32
		* em_x270_defconfig—— Build for em_x270
		* ep93xx_defconfig—— Build for ep93xx
		* eseries_pxa_defconfig—— Build for eseries_pxa
		* exynos_defconfig—— Build for exynos
		* ezx_defconfig—— Build for ezx
		* footbridge_defconfig—— Build for footbridge
		* gemini_defconfig—— Build for gemini
		* h3600_defconfig—— Build for h3600
		* h5000_defconfig—— Build for h5000
		* hackkit_defconfig—— Build for hackkit
		* hisi_defconfig—— Build for hisi
		* imote2_defconfig—— Build for imote2
		* imx_v4_v5_defconfig—— Build for imx_v4_v5
		* imx_v6_v7_defconfig—— Build for imx_v6_v7
		* integrator_defconfig—— Build for integrator
		* iop13xx_defconfig—— Build for iop13xx
		* iop32x_defconfig—— Build for iop32x
		* iop33x_defconfig—— Build for iop33x
		* ixp4xx_defconfig—— Build for ixp4xx
		* jornada720_defconfig—— Build for jornada720
		* keystone_defconfig—— Build for keystone
		* ks8695_defconfig—— Build for ks8695
		* lart_defconfig—— Build for lart
		* lpc18xx_defconfig—— Build for lpc18xx
		* lpc32xx_defconfig—— Build for lpc32xx
		* lpd270_defconfig—— Build for lpd270
		* lubbock_defconfig—— Build for lubbock
		* magician_defconfig—— Build for magician
		* mainstone_defconfig—— Build for mainstone
		* mini2440_defconfig—— Build for mini2440
		* mmp2_defconfig—— Build for mmp2
		* moxart_defconfig—— Build for moxart
		* mps2_defconfig—— Build for mps2
		* multi_v4t_defconfig—— Build for multi_v4t
		* multi_v5_defconfig—— Build for multi_v5
		* multi_v7_defconfig Build for multi_v7
		* mv78xx0_defconfig—— Build for mv78xx0
		* mvebu_v5_defconfig—— Build for mvebu_v5
		* mvebu_v7_defconfig—— Build for mvebu_v7
		* mxs_defconfig—— Build for mxs
		* neponset_defconfig—— Build for neponset
		* netwinder_defconfig—— Build for netwinder
		* netx_defconfig—— Build for netx
		* nhk8815_defconfig—— Build for nhk8815
		* nuc910_defconfig—— Build for nuc910
		* nuc950_defconfig—— Build for nuc950
		* nuc960_defconfig—— Build for nuc960
		* omap1_defconfig—— Build for omap1
		* omap2plus_defconfig—— Build for omap2plus
		* orion5x_defconfig—— Build for orion5x
		* oxnas_v6_defconfig—— Build for oxnas_v6
		* palmz72_defconfig—— Build for palmz72
		* pcm027_defconfig—— Build for pcm027
		* pleb_defconfig—— Build for pleb
		* prima2_defconfig—— Build for prima2
		* pxa168_defconfig—— Build for pxa168
		* pxa255-idp_defconfig—— Build for pxa255-idp
		* pxa3xx_defconfig—— Build for pxa3xx
		* pxa910_defconfig—— Build for pxa910
		* pxa_defconfig—— Build for pxa
		* qcom_defconfig—— Build for qcom
		* raumfeld_defconfig—— Build for raumfeld
		* realview_defconfig—— Build for realview
		* rpc_defconfig—— Build for rpc
		* s3c2410_defconfig—— Build for s3c2410
		* s3c6400_defconfig—— Build for s3c6400
		* s5pv210_defconfig—— Build for s5pv210
		* sama5_defconfig—— Build for sama5
		* shannon_defconfig—— Build for shannon
		* shmobile_defconfig—— Build for shmobile
		* simpad_defconfig—— Build for simpad
		* socfpga_defconfig—— Build for socfpga
		* spear13xx_defconfig—— Build for spear13xx
		* spear3xx_defconfig—— Build for spear3xx
		* spear6xx_defconfig—— Build for spear6xx
		* spitz_defconfig—— Build for spitz
		* stm32_defconfig—— Build for stm32
		* sunxi_defconfig—— Build for sunxi
		* tango4_defconfig—— Build for tango4
		* tct_hammer_defconfig—— Build for tct_hammer
		* tegra_defconfig—— Build for tegra
		* trizeps4_defconfig—— Build for trizeps4
		* u300_defconfig—— Build for u300
		* u8500_defconfig—— Build for u8500
		* versatile_defconfig—— Build for versatile
		* vexpress_defconfig—— Build for vexpress
		* vf610m4_defconfig—— Build for vf610m4
		* viper_defconfig—— Build for viper
		* vt8500_v6_v7_defconfig—— Build for vt8500_v6_v7
		* xcep_defconfig—— Build for xcep
		* zeus_defconfig—— Build for zeus
		* zx_defconfig—— Build for zx


# 10 其它
* `make V=0|1|2 [targets]`
	* 0 ： 静默构建 (default), 
	* 1 ： 可见构建 build
	* 2 ： 给出重新构建目标的原因
* `make O=dir [targets]`——将所有的输出文件放在dir中包括.config
* `make C=1   [targets]`—— 使用$CHECK检查重新编译的c源文件（默认为sparse）
* `make C=2   [targets]`—— 强制使用$ CHECK检查所有的c源文件（默认为sparse）
* `make RECORDMCOUNT_WARN=1 [targets]`—— 对忽略的mcount部分发出警告
* `make W=n   [targets]`——允许额外的gcc检查，n=1,2,3 ：
	* 1：可能相关且不经常发生的警告
	* 2：经常发生但可能仍然相关的警告
	* 3：更加模糊的警告，很可能被忽略
	* 多个级别可以组合，如w=123



------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：<font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------