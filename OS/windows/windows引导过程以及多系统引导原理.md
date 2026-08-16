---
title: windows引导过程以及多系统引导原理
tags: windows
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：<font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------



# 1 Windows的引导方式
&emsp;&emsp;目前主要的系统引导方式也有两种：**传统的Legacy  BIOS和新型的UEFI BIOS**。
&emsp;&emsp;Legacy BIOS无法识别GPT分区表格式；UEFI BIOS可同时识别MBR分区和GPT分区，所以UEFI下，MBR和GPT磁盘都可用于启动操作系统。不过由于微软限制，UEFI下使用Windows安装程序安装操作系统是只能将系统安装在GPT磁盘中。
&emsp;&emsp;UEFI是一个微型操作系统，放在固件中，能够识别FAT文件系统，运行efi程序。

---

>**Boot Loader**：
>
>* 对于PC平台来说Boot Loader=BIOS+启动管理器+启动文件+载入程序（IPL+SPL），但是一般指启动管理器及启动设置文件。
>* 功能：
>     * 启动管理器+启动设置文件：提供启动选项，定位、加载并执行SPL。
>     * SPL：加载OS内核，SPL只认识自己分区的内核文件。




## 1.1 Legacy BIOS+MBR引导原理
&emsp;&emsp;**引导过程**：上电-->Legacy BIOS-->MBR-->DPT-->PBR--> Bootmgr（vista开始）/NTLDR-->BCD（vista开始）/boot.ini-->Winload.exe-->内核加载 -->windows vista/windows xp

>* 上电，执行Legacy BIOS
>* Legacy BIOS先要对CPU初始化，然后跳转到BIOS启动处进行POST自检，此过程如有严重错误，则电脑会用不同的报警声音提醒，接下来采用读中断的方式加载各种硬件，完成硬件初始化。
>* 读入MBR中的IPL。
>* IPL确定分区表中的活动主分区（也叫激活主分区），并找到该分区的引导扇区（分区的第一个扇区）中的分区引导记录（PBR），载入PBR中的启动管理器（bootmgr.exe或NTLDR）。
>* 启动管理器读取该分区根目录下boot文件夹里的启动设置文件（BCD或boot.ini），然后根据启动设置文件定位并加载二级引导载入程序winload.exe（位置：C:\Windows\system32\winload.exe）来加载OS内核。如果有多个系统，启动设置文件就会有多个启动项，相应的bootmgr也会提供选择菜单，然后则根据用户选择来决定加载哪个loader（**多系统引导原理1**）。

&emsp;&emsp;启动管理器bootmgr和启动设置文件BCD合称为启动文件，IPL以及SPL（winload.exe）合称为载入程序。启动文件与载入程序合在一起叫做boot loader。

&emsp;&emsp;MBR磁盘分区格式下，只允许有一个分区是活动的，因此启动文件（bootmgr、BCD）必须存放在活动的主分区内，这样才能找到系统载入程序。

&emsp;&emsp;如果使用Windows的官方安装程序，会自动写入MBR，此外还会建一个隐藏的活动主分区，写入启动管理器和启动设置文件，并将自身的以及可识别的所有二级引导载入程序加入启动设置文件中（**多系统引导原理2**）。以后装入其它windows系统时，则会重写MBR和该分区。当然也可以不使用这个隐藏分区，而将bootmgr、BCD放在系统所在分区的PBR（PBR容量远远大于MBR）中，并设置系统分区为活动分区。但要注意的是：虽然可以在多个分区中安装系统，但只有一个分区是活动分区，能够引导系统，删除该分区，其他分区的系统也无法启动。



## 1.2 UEFI BIOS+GPT 引导原理
&emsp;&emsp;**引导过程**：上电-->UEFI-->GPT分区表-->EFI分区-->\efi\Microsoft\boot\bootmgfw.efi-->efi\Microsoft\BCD→\Windows\system32\winload.efi。
>* 开机，执行UEFI BIOS；
>* UEFI BIOS运行预加载环境先直接初始化CPU和内存，CPU和内存若有问题则直接黑屏，其后启动PXE采用枚举方式搜索各种硬件并加载驱动，完成硬件初始化。
>* 从FAT格式的EFI分区找到、读入、执行启动管理器（\efi\Microsoft\boot\bootmgfw.efi）。
>* 启动管理器导入EFI分区BCD文件（efi\Microsoft\BCD），然后根据其配置内容加载引导加载程序winload.efi（位置：C:\Windows\system32\winload.efi）来加载OS内核。如果BCD中有多个配置项，则会让用户选择要启动的系统。如果有多个系统，启动设置文件就会有多个启动项，相应的bootmgr也会提供选择菜单，然后则根据用户选择来决定加载哪个loader（**多系统引导原理1**）。。

&emsp;&emsp;需要注意的是：GPT磁盘格式下，windows系统的启动文件（bootmgfw.efi、BCD）是存放在一个FAT格式的分区里的，有些出厂预装win8系统的电脑下将该FAT分区称之为ESP分区或EFI分区。
&emsp;&emsp;可见，UEFI+GPT模式引导windows系统时，并不需要mbr主引导记录，也不需要活动分区，只需要你一个存放了引导启动文件的fat格式分区就可以了，这个fat分区当然也可以是U盘等外接USB设备了。所以我们在安装第一个系统时，需要由系统自动或手动建立一个EFI分区，用来存放启动文件，之后在其它分区安装系统时，会覆盖之前的启动文件（不是系统载入程序）。

# 2 多系统引导原理
&emsp;&emsp;每个分区都有一个boot sector，如果该分区有操作系统，操作系统会将份启动文件 写入到各自所在分区的boot sector中，也可以选择写入MBR中。



&emsp;&emsp;在 Linux 系统安装时，你可以选择将 boot loader 安装到 MBR 去，也可以选择不安装。 如果选择安装到 MBR 的话，那理论上你在 MBR 与 boot sector 都会保有一份boot loader 程序的。 至于 Windows 安装时，预设主动的将 MBR 与 boot sector 都装上一份boot loader！

&emsp;&emsp;因此安装多个系统时，MBR、活动分区会不断的被覆盖。但是启动管理程序能够识别其他分区引导扇区中的启动管理器，因此可以根据用户选择将控制权交给相应的启动管理器从而实现多重引导。但如果无法识别，则不能实现多系统。

&emsp;&emsp;Linux的启动管理器能够识别Windows的，但Windows的启动管理器则无法识别Linux的。又由于Windows在装系统时，默认覆盖MBR和活动分区。所以先装Linux再装windows，则无法引导Linux。而Linux则可以选择是否覆盖MBR分区和活动分区。如果不覆盖，则需要原来的MBR支持。

&emsp;&emsp;对GPT来说也类似，只不过覆盖的是EFI分区。



------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：<font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------