---
title: Linux内核的配置、编译、安装、卸载
tags: Linux内核构建系统
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>《Linux内核官方文档》。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------



# 1 准备源码

## 1.1 获取并解压内核源码
```
$tar -xvJf linux-x.x.x.tar.xz
$cd linux-x.x.x.tar.xz
```
## 1.2 检查硬件环境
```
$cat /proc/cpuinfo
$ls pci
```
## 1.3 获取干净的源码
* `$make clean`—— 删除大多数生成的文件，但保留配置和足够的构建支持以构建外部模块
* `$make mrproper` —— 删除所有生成的文件+ config +各种备份文件
* `$make distclean` —— `$make mrproper` +删除编辑器备份文件和补丁文件


# 2 配置内核
* `make [ARCH=xxx] xxxconfig`，配置内核，每个选项要么是二选一yes或no，要么是三选一yes、no、module。yes表示把代码编译进内核映像，module表示以模块方式编译。
* 常见配置目标
	* 手动配置 
		* `$ make [ARCH=xxx] config`——基于文本的配置界面
		* `$ make [ARCH=xxx] nconfig`——基于ncurses的配置界面
		* `$ make [ARCH=xxx] menuconfig`——基于菜单程序的配置界面，
			* 常用按键：
				* `?`：帮助
				* `enter`： 进入
				* `y`：内建
				* `n`：禁用
				* `m`：模块化
				* `space`：循环选择,Y->N->M->Y
				* `ESC ESC`：退出
				* `/`：搜索
			* 符号意义
				* `[]`：内建或移除
				* `<>`：内建移除模块化
				* `{}`：模块化移除
				* `--`：被其他功能选中
				* `--->`：有子菜单
				* `----`: 子菜单为空
		* `$ make [ARCH=xxx] xconfig`——使用基于Qt的配置界面
		* `$ make [ARCH=xxx] gconfig`——使用基于GTK+的配置界面
	* 快速配置
		* `$ make [ARCH=xxx] oldconfig`——使用之前配置好的.config作为基础更新当前配置
		* `$ make [ARCH=xxx] defconfig`——使用ARCH提供的defconfig默认配置
		* `$ make [ARCH=xxx] tinyconfig`——配置最小的内核

>补充：
> * 内核配置好后，配置项存放在源代码内核源码根目录的`.config`文件中，可以直接修改该文件，修改后应该更新用`$ make oldconfig`更新配置。
>* 选项`CONFIG_IKCONFIG_PROC`把压缩过的内核配置文件放在`/proc/config.gz`中，如果当前内核启用了该选项，则可以直接克隆该配置。
>* 如果不想在命令行输入ARCH=XXX，则可以直接在顶层Makefile中进行修改。

# 3 编译内核与模块 

* `$make [ARCH=xxx] [编译目标][编译选项与参数]`，编译内核，2.6版本之前每次编译需要运行`$make dep`。 
	* 如果想减少编译时的垃圾信息可以使用
		* `$make XXXX > 文件名` ： 把编译细节、警告、错误输出到指定文件。
		*  `$make XXXX > /dev/null`：不记录任何编译信息
	*  如果不想在命令行输入ARCH=XXX和CROSS_COMPILE=XXX，则可以直接在顶层Makefile中进行修改。
	*  如果想编译快一些，可以使用-jn选项，n为cpu个数
* 常见编译目标
	* all
	* vmlinux 
	* Image
	* zImage/bzImage
	* modules
	
>更多目标请参考《Linux内核的make目标》

# 3 安装内核
* 自动安装
	* `$make modules_install`，安装模块
	*  `$make install`，安装内核
	*  `$sudo update-initramfs -c -k 4.17-rc2`,启用内核
	*  `$sudo update-grub`,更新grub
*  手动安装
	*  拷贝bzImage为`/boot/vmlinuz-VERSION-RELEASE`
	*  建立并拷贝initrd.img为`/boot/initrd-KERNEL-VERSION`
	*   拷贝system.map为`/boot/System.map-KERNEL-VERSION`
	*   拷贝.config文件为`/boot/config-KERNEL-VERSION`
	*   更新/boot/grub的配置文件，增加内核启动列表



# 4 卸载内核 
1. 删除`/boot/vmlinuz-KERNEL-VERSION`
2. 删除`/boot/initrd-KERNEL-VERSION*`
3. 删除`/boot/System-map-KERNEL-VERSION`
4. 删除`/boot/config-KERNEL-VERSION`
5. 更新`/boot/grub`的配置文件，删除不需要的内核启动列表





-----

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>《Linux内核官方文档》。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
