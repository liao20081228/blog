---
title: Linux内核的构建系统详解
tags: Linux内核构建系统
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>《Linux内核文档》。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

<style>table{word-break:initial;}</style>
# 1 准备知识
## 1.1 程序构建过程
&emsp;&emsp;**编译**：把高级语言书写的代码转换为机器可识别的机器指令。编译高级语言后生成的指令虽然可被机器识别，但是还不能被执行。编译时，编译器检查高级语言的语法、函数与变量的声明是否正确。只有所有的语法正确、相关变量定义正确编译器就可以编译出中间目标文件。通常，一个高级语言的源文件都可对应一个目标文件。目标文件在Linux中默认后缀为“.o”（如“hello.c”的目标文件为“hello.o”）。
&emsp;&emsp;**链接**：将多个.o文件，或者.o文件和库文件链接成为可被操作系统执行的可执行程序。链接器不检查函数所在的源文件，只检查所有.o文件中的定义的符号。将.o文件中使用的函数和其它.o或者库文件中的相关符号进行合并，最后生成一个可执行的程序。“ld”是GNU的链接器。
&emsp;&emsp;**静态库**：又称为文档文件（Archive File）。它是多个.o文件的集合。Linux中静态库文件的后缀为“.a”。静态库中的各个成员（.o文件）没有特殊的存在格式，仅仅是一个.o文件的集合。使用“ar”工具维护和管理静态库。
&emsp;&emsp;**共享库**：也是多个.o文件的集合，但是这些.o文件时有编译器按照一种特殊的方式生成。对象模块的各个成员的地址（变量引用和函数调用）都是相对地址。因此在程序运行时，可动态加载库文件和执行共享的模块（多个程序可以共享使用库中的某一个模块）。
&emsp;&emsp;一般来说，无论是C、C++、还是pas，首先要把源文件编译成中间代码文件，在Windows下也就是 .obj 文件，UNIX下是 .o 文件，即 Object File，这个动作叫做编译（compile）。然后再把大量的Object File合成执行文件，这个动作叫作链接（link）。

## 1.2 make简介
&emsp;&emsp;Make是GCC提供的一种半自动化的工程管理器，可以根据预定规则自动构建目标程序。所谓的**半自动化是指在使用工程管理器之前需要人工编写程序的编译规则**，所有的编译规则都保存在Makefile文件中，全自动化工程管理器会在编译程序前自动生成Makefile文件。
&emsp;&emsp;make通过比较对应文件（规则的目标和依赖）的最后修改时间，来决定哪些文件需要更新。对需要更新的文件make就执行数据库中所记录的相应命令（在make读取Makefile以后会建立一个编译过程的描述数据库。此数据库中记录了所有各个文件之间的相互关系，以及它们的关系描述）来重建它，对于不需要重建的文件make什么也不做。而且可以通过make的命令行选项来指定需要重新编译的文件。
&emsp;&emsp;执行make时，需要一个命名为Makefile的特殊文件来告诉make需要做什么（完成什么任务），该怎么做。当使用make工具进行编译时，工程中以下几种文件在执行make时将会被编译（重新编译）：
* 所有的源文件没有被编译过，则对各个C源文件进行编译并进行链接，生成最后的可执行程序；
* 每一个在上次执行make之后修改过的C源代码文件在本次执行make时将会被重新编译；
* 头文件在上一次执行make之后被修改。则所有包含此头文件的C源文件在本次执行make时将会被重新编译。

后两种情况make只将修改过的C源文件重新编译生成.o文件，对于没有修改的文件不进行任何工作。重新编译过程中，任何一个源文件的修改将产生新的对应的.o文件，新的.o文件将和以前的已经存在、此次没有重新编译的.o文件重新连接生成最后的可执行程序。
# 2 配置内核
## 2.1 配置方式
&emsp;&emsp;Linux内核源代码组织了一个配置系统，该配置系统可以生成内核配置菜单，方便内核配置。**配置系统主要包含Makefile、Kconfig 和 scripts**。其中，scripts/kconfig中存放了各种配置工具，配置界面是通过配置工具来生成的，配置工具是通过指定make目标，然后由make构建并执行，配置界面中的选项则是通过各级的Kconfig文件定义。
&emsp;&emsp;Linux内核配置系统也被移植到其它软件(如Busybox、glibc、uclibc等)中，来提供同样的配置界面以方便有选择性的配置。Linux内核配置命令主要有：make config、make menuconfig、make xconfig和make gconfig，分别是字符界面、ncurses光标菜单、QT图形窗口和GTK图形窗口的配置界面。




|工具 |Makefile目标 |源程序|依赖库|
|:--|:--|:--|:--|
|conf |config defconfig ... |conf.c、zconf.tab.c||
|mconf|menuconfig|mconf.c、zconf.tab.c|ncurses
|nconf|nconfig|nconf.c、zconf.tab.c|ncurses
|qconf|xconfig|qconf.c、kconfig_load.c、zconf.tab.c|QT图形库|
|gconf|gconfig|gconf.c、kconfig_load.c、zconf.tab.c|GTK图形库|

其中zconf.tab.c程序实现了解析Kconfig文件和内核配置主要函数。zconf.tab.c程序还直接包括了下列一些C程序，这样各种配置功能都包含在zconf.tab.o目标文件中了。
* lex.zconf.c实现lex语法解析器，
* util.c实现配置工具，
* confdata.c实现.config等相关数据文件保存，
* expr.c实现表达式函数，
* symbol.c实现变量处理函数，
* menu.c实现菜单控制函数。

## 2.2 配置过程
以make menuconfig为例

### 2.2.1 基本变量
首先需要知道一些顶层Makefile中定义的变量(内核版本号：4.18.7):

版本号和名称变量：
```makefile?linenums=2
VERSION = 4          
PATCHLEVEL = 18       
SUBLEVEL = 7        
EXTRAVERSION =       
NAME = Merciless Moray  
```


定义伪目标，伪目标的先决条件_all，其没有先决条件和命令，被认为为最新的，因此始终需要更新该目标：
```makefile?linenums=15
PHONY := _all 
_all:  
```

定义MAKEFLAGS，-rR禁用内建变量和隐式规则，指定include引入的makefile文件目录为当前文件
```makefile?linenums=21
MAKEFLAGS += -rR --include-dir=$(CURDIR)
```

定义KBUILD_VERBOSE变量，其值由命令行参数v传入，如`make v=1`
```makefile?linenums=74
ifeq ("$(origin V)", "command line")
  KBUILD_VERBOSE = $(V) 
endif
ifndef KBUILD_VERBOSE
  KBUILD_VERBOSE = 0
endif
```

定义quiet和Q，根据KBUILD_VERBOSE决定是否要显示将执行的命令。
```makefile?linenums=81
ifeq ($(KBUILD_VERBOSE),1)
  quiet =
  Q =
else
  quiet=quiet_
  Q = @
endif
```

如果make使用了-s选项，则不显示执行的命令
```makefile?linenums=92
ifneq ($(findstring s,$(filter-out --%,$(MAKEFLAGS))),) 
  quiet=silent_
  tools_silent=s
endif

#将变量输出到子make中
export quiet Q KBUILD_VERBOSE
```

使用make O=dir 或 export KBUILD_OUTPUT=dir 来指定生成文件的存放目录
```makefile?linenums=116
ifeq ($(KBUILD_SRC),)
 
# OK, Make called in directory where kernel src resides
# Do we want to locate output files in a separate directory?
ifeq ("$(origin O)", "command line")
  KBUILD_OUTPUT := $(O)
endif
 
# Cancel implicit rules on top Makefile
$(CURDIR)/Makefile Makefile: ;
 
ifneq ($(words $(subst :, ,$(CURDIR))), 1)
  $(error main directory cannot contain spaces nor colons)
endif

ifneq ($(KBUILD_OUTPUT),)
# check that the output directory actually exists
saved-output := $(KBUILD_OUTPUT)
KBUILD_OUTPUT := $(shell mkdir -p $(KBUILD_OUTPUT) && cd $(KBUILD_OUTPUT) \ && pwd)

$(if $(KBUILD_OUTPUT),, $(error failed to create output directory "$(saved-output)"))
       
	   
PHONY += $(MAKECMDGOALS) sub-make
       
$(filter-out _all sub-make $(CURDIR)/Makefile, $(MAKECMDGOALS)) _all: sub-make
        @:
       
# Invoke a second make in the output directory, passing relevant variables
sub-make:
        $(Q)$(MAKE) -C $(KBUILD_OUTPUT) KBUILD_SRC=$(CURDIR) \
        -f $(CURDIR)/Makefile $(filter-out _all sub-make,$(MAKECMDGOALS))
 
# Leave processing to above invocation of make
skip-makefile := 1
endif #ifneq ($(KBUILD_OUTPUT),)
endif #ifeq ($(KBUILD_SRC),)
```
进入目录时不要显示
```makefile?linenums=160
MAKEFLAGS += --no-print-directory
```
使用make C=1 来检查重新编译的源码，使用make C=2 检查所有源码，缺省不检查。
```makefile?linenums=172
ifeq ("$(origin C)", "command line")
  KBUILD_CHECKSRC = $(C)
endif
ifndef KBUILD_CHECKSRC
  KBUILD_CHECKSRC = 0
endif
```
使用make M=dir 或者 make SUBDIRS=$PWD来设置模块存放目录
```makefile?linenums=182
ifdef SUBDIRS
  KBUILD_EXTMOD ?= $(SUBDIRS)
endif
 
ifeq ("$(origin M)", "command line")
  KBUILD_EXTMOD := $(M)
endif
```
定义了srctree、src、obj、objtree、VPATH
```makefile?linenums=190
ifeq ($(KBUILD_SRC),)
        # building in the source tree
        srctree := .
else
        ifeq ($(KBUILD_SRC)/,$(dir $(CURDIR)))                             
                # building in a subdirectory of the source tree
                srctree := ..
        else
                srctree := $(KBUILD_SRC)
        endif
endif
 
export KBUILD_CHECKSRC KBUILD_EXTMOD KBUILD_SRC
 
objtree         := .
src             := $(srctree)
obj             := $(objtree)
 
VPATH           := $(srctree)$(if $(KBUILD_EXTMOD),:$(KBUILD_EXTMOD))
 
export srctree objtree VPATH
```

定义内核版本相关的变量KERNELRELEASE，KERNELVERSION，并和前面的几个变量一起输出的到子make中。
```makefile?linenums=285
KERNELRELEASE = $(shell cat include/config/kernel.release 2> /dev/null)
KERNELVERSION = $(VERSION)$(if $(PATCHLEVEL),.$(PATCHLEVEL)$(if $(SUBLEVEL),.$(SUBLEVEL)))$(EXTRAVERSION)
export VERSION PATCHLEVEL SUBLEVEL KERNELRELEASE KERNELVERSION
```
定义要构建的架构变量SUBARCH

```makefile?linenums=295
SUBARCH := $(shell uname -m | sed -e s/i.86/x86/ -e s/x86_64/x86/ \
                                  -e s/sun4u/sparc64/ \
                                  -e s/arm.*/arm/ -e s/sa110/arm/ \
                                  -e s/s390x/s390/ -e s/parisc64/parisc/ \
                                  -e s/ppc.*/powerpc/ -e s/mips.*/mips/ \
                                  -e s/sh[234].*/sh/ -e s/aarch64.*/arm64/ \
                                  -e s/riscv.*/riscv/)
```

定义交叉编译相关的设置
```makefile?linenums=311								  
# 当执行交叉编译时，需要将ARCH设置为目标架构，默认ARCH为主机的架构。
# 第一种设置方法是通过命令行设置，如：make ARCH=ia64
# 第二种设置方法是在环境变量中设置
# 
# CROSS_COMPILE 用于指定交叉编译工具的前缀，默认为空。 
# 第一种设置方法是通过命令行设置，如：make CROSS_COMPILE=ia64-linux-
# 第二种设置方法是在环境变量中设置
#
# 注意: 有些CROSS_COMPILE的设置是在对应的arch/*/Makefile中
								  
ARCH            ?= $(SUBARCH)
        
# Architecture as present in compile.h
UTS_MACHINE     := $(ARCH)
SRCARCH         := $(ARCH)
        
# Additional ARCH settings for x86
ifeq ($(ARCH),i386)
        SRCARCH := x86
endif   
ifeq ($(ARCH),x86_64)
        SRCARCH := x86
endif   
        
# Additional ARCH settings for sparc
ifeq ($(ARCH),sparc32)
       SRCARCH := sparc
endif   
ifeq ($(ARCH),sparc64)
       SRCARCH := sparc
endif   
        
# Additional ARCH settings for sh
ifeq ($(ARCH),sh64)
       SRCARCH := sh
endif   
 
# 配置文件名
KCONFIG_CONFIG  ?= .config
export KCONFIG_CONFIG

# 用于kbuild的shell
CONFIG_SHELL := $(shell if [ -x "$$BASH" ]; then echo $$BASH; \
          else if [ -x /bin/bash ]; then echo /bin/bash; \
          else echo sh; fi ; fi)
       
HOST_LFS_CFLAGS := $(shell getconf LFS_CFLAGS 2>/dev/null)
HOST_LFS_LDFLAGS := $(shell getconf LFS_LDFLAGS 2>/dev/null)
HOST_LFS_LIBS := $(shell getconf LFS_LIBS 2>/dev/null)
       
HOSTCC       = gcc
HOSTCXX      = g++
HOSTCFLAGS   := -Wall -Wmissing-prototypes -Wstrict-prototypes -O2 \
                -fomit-frame-pointer -std=gnu89 $(HOST_LFS_CFLAGS)
HOSTCXXFLAGS := -O2 $(HOST_LFS_CFLAGS)
HOSTLDFLAGS  := $(HOST_LFS_LDFLAGS)
HOST_LOADLIBES := $(HOST_LFS_LIBS)

# Make 变量(CC, etc...)                                        
AS              = $(CROSS_COMPILE)as
LD              = $(CROSS_COMPILE)ld
CC              = $(CROSS_COMPILE)gcc
CPP             = $(CC) -E
AR              = $(CROSS_COMPILE)ar
NM              = $(CROSS_COMPILE)nm
STRIP           = $(CROSS_COMPILE)strip
OBJCOPY         = $(CROSS_COMPILE)objcopy
OBJDUMP         = $(CROSS_COMPILE)objdump
LEX             = flex
YACC            = bison
AWK             = awk
GENKSYMS        = scripts/genksyms/genksyms
INSTALLKERNEL  := installkernel
DEPMOD          = /sbin/depmod
PERL            = perl
PYTHON          = python
PYTHON2         = python2
PYTHON3         = python3
CHECK           = sparse

CHECKFLAGS     := -D__linux__ -Dlinux -D__STDC__ -Dunix -D__unix__ \
                  -Wbitwise -Wno-return-void -Wno-unknown-attribute $(CF)
NOSTDINC_FLAGS  =
CFLAGS_MODULE   =
AFLAGS_MODULE   =
LDFLAGS_MODULE  =
CFLAGS_KERNEL   =
AFLAGS_KERNEL   =
LDFLAGS_vmlinux =

# Use USERINCLUDE when you must reference the UAPI directories only.
USERINCLUDE    := \
                -I$(srctree)/arch/$(SRCARCH)/include/uapi \
                -I$(objtree)/arch/$(SRCARCH)/include/generated/uapi \
                -I$(srctree)/include/uapi \
                -I$(objtree)/include/generated/uapi \
                -include $(srctree)/include/linux/kconfig.h
       
# Use LINUXINCLUDE when you must reference the include/ directory.
# Needed to be compatible with the O= option
LINUXINCLUDE    := \
                -I$(srctree)/arch/$(SRCARCH)/include \
                -I$(objtree)/arch/$(SRCARCH)/include/generated \
                $(if $(KBUILD_SRC), -I$(srctree)/include) \
                -I$(objtree)/include \
                $(USERINCLUDE)
       
KBUILD_AFLAGS   := -D__ASSEMBLY__
KBUILD_CFLAGS   := -Wall -Wundef -Wstrict-prototypes -Wno-trigraphs \
                   -fno-strict-aliasing -fno-common -fshort-wchar \
                   -Werror-implicit-function-declaration \
                   -Wno-format-security \
                   -std=gnu89
KBUILD_CPPFLAGS := -D__KERNEL__
KBUILD_AFLAGS_KERNEL :=
KBUILD_CFLAGS_KERNEL :=
KBUILD_AFLAGS_MODULE  := -DMODULE
KBUILD_CFLAGS_MODULE  := -DMODULE
KBUILD_LDFLAGS_MODULE := -T $(srctree)/scripts/module-common.lds
LDFLAGS :=
GCC_PLUGINS_CFLAGS :=

export ARCH SRCARCH CONFIG_SHELL HOSTCC HOSTCFLAGS CROSS_COMPILE AS LD CC
export CPP AR NM STRIP OBJCOPY OBJDUMP HOSTLDFLAGS HOST_LOADLIBES
export MAKE LEX YACC AWK GENKSYMS INSTALLKERNEL PERL PYTHON PYTHON2 PYTHON3 UTS_MACHINE
export HOSTCXX HOSTCXXFLAGS LDFLAGS_MODULE CHECK CHECKFLAGS
        
export KBUILD_CPPFLAGS NOSTDINC_FLAGS LINUXINCLUDE OBJCOPYFLAGS LDFLAGS
export KBUILD_CFLAGS CFLAGS_KERNEL CFLAGS_MODULE
export CFLAGS_KASAN CFLAGS_KASAN_NOSANITIZE CFLAGS_UBSAN
export KBUILD_AFLAGS AFLAGS_KERNEL AFLAGS_MODULE
export KBUILD_AFLAGS_MODULE KBUILD_CFLAGS_MODULE KBUILD_LDFLAGS_MODULE
export KBUILD_AFLAGS_KERNEL KBUILD_CFLAGS_KERNEL
export KBUILD_ARFLAGS
        
# When compiling out-of-tree modules, put MODVERDIR in the module
# tree rather than in the kernel tree. The kernel tree might
# even be read-only.
export MODVERDIR := $(if $(KBUILD_EXTMOD),$(firstword $(KBUILD_EXTMOD))/).tmp_versions
        
# Files to ignore in find ... statements
        
export RCS_FIND_IGNORE := \( -name SCCS -o -name BitKeeper -o -name .svn -o    \
                          -name CVS -o -name .pc -o -name .hg -o -name .git \) \
                          -prune -o
export RCS_TAR_IGNORE := --exclude SCCS --exclude BitKeeper --exclude .svn \
                         --exclude CVS --exclude .pc --exclude .hg --exclude .git
```

构建内化部分，还是只构建模块，或者两者都构建
```makefile?linenums=556 
KBUILD_MODULES :=
KBUILD_BUILTIN := 1
 
# If we have only "make modules", don't compile built-in objects.
# When we're building modules with modversions, we need to consider
# the built-in objects during the descend as well, in order to
# make sure the checksums are up to date before we record them.
 
ifeq ($(MAKECMDGOALS),modules)
  KBUILD_BUILTIN := $(if $(CONFIG_MODVERSIONS),1)
endif
 
# If we have "make <whatever> modules", compile modules
# in addition to whatever we do anyway.
# Just "make" or "make all" shall build modules as well
 
ifneq ($(filter all _all modules,$(MAKECMDGOALS)),)
  KBUILD_MODULES := 1
endif
 
ifeq ($(MAKECMDGOALS),)
  KBUILD_MODULES := 1
endif
 
export KBUILD_MODULES KBUILD_BUILTIN
```

### 2.2.2 Makefile之间的关系
![1](https://www.github.com/liao20081228/blog/raw/master/图片/Linux内核的构建系统/1.JPG)

### 2.2.3 配置过程分析

当我们使用`make menuconfig`这个命令时（其它配置命令类似），在顶层makefile第535行有:
```makefile?linenums=535
%config: scripts_basic outputmakefile FORCE
	$(Q)$(MAKE) $(build)=scripts/kconfig $@
```
menuconfig与目标%config匹配，而这个规则有三个依赖：scripts_basic、outputmakefile、FORCE。这三个先决条件如下：

------

（1）FORCE
&emsp;定义在顶级makefile的第1717、1718行：

```makefile?linenums=1717
PHONY += FORCE
FORCE: 
```
PHONY中的项目被称为伪目标，伪目标作为另外一个目标的依赖时，因为总被认为被更新过的，所以那个规则的中定义的命令总会被执行。在执行此规则时，目标FORCE总会被认为是最新的。这样当它作为其它规则的依赖时，因为总被认为被更新过的，所以那个规则的中定义的命令总会被执行。 



（2）scripts_basic 
&emsp;顶级makefile的第462、463、464、465行：
  
```makefile?linenums=462
PHONY += scripts_basic
scripts_basic: 
       $(Q)$(MAKE) $(build)=scripts/basic 
	   $(Q)rm -f .tmp_quiet_recordmcount
```  
build这个变量定义在`scripts/kbuild.include`第188行，该头文件由顶级Makefile在282行引入：

```makefile?linenums=188
build := -f $(srctree)/scripts/Makefile.build obj 
``` 
由前面2.2.1可知srctree值为`.`，Q的值为@，MAKE默认为make，所以上面的规则可写成如下形式： 
```makefile?linenums=463
scripts_basic: 
      @make -f ./scripts/Makefile.build obj=scripts/basic 
	  @rm -f .tmp_quiet_recordmcount
``` 
这个规则的命令是启动子make，并读取scripts/Makefile.build文件，并传递命令行变量obj=scripts/basic。然后强制删除 .tmp_quiet_recordmcount文件。在scripts/Makefile.build的第6行： 
```makefile?linenums=6
src := $(obj) #这就把传递进来的obj=scripts/basic赋给了src,这个src并非顶级Makefile中的src
```
scripts/Makefile.build的第43，44，45行为：
```makefile?linenums=43
kbuild-dir := $(if $(filter /%,$(src)),$(src),$(srctree)/$(src))
kbuild-file := $(if $(wildcard $(kbuild-dir)/Kbuild),$(kbuild-dir)/Kbuild,$(kbuild-dir)/Makefile)
include $(kbuild-file) 
``` 
src值为scripts/basic ,而srctree 有顶级Makefile输出，其值为.。作用就是把scripts/basic目录下的Makefile包含进来（如果有Kbuild则包含Kbuild）,但是并没有引入到顶级Makefile中，只在scripts/Makefile.build这一级。在scripts/Makefile.build的第87行，重新定义了目标__build的规则：
```makefile?linenums=87  
__build: $(if $(KBUILD_BUILTIN),$(builtin-target) $(lib-target) $(extra-y)) \ 
        $(if $(KBUILD_MODULES),$(obj-m)) \ 
        $(subdir-ym) $(always) 
       @: 
```
由2.2.1节可知KBUILD_BUILTIN在顶层Makefile的第556行定义 ,如果执行”make modules”，会在564行开始对该变量进行一些处理 ,所以我们这里 `KBUILD_BUILTIN为1` ；KBUILD_MODULES在顶层Makefile的第555行定义: 如果执行”make all”、”make \_all”、”make modules”、”make”中任一个命令或者省略了目标，则在572行开始会对该变量进行处理,因此，我们这里KBUILD_MODULES 为空。分析了这两个变量后，上面的规则可重新写为 ：
```makefile?linenums=87 
__build: $(builtin-target) $(lib-target) $(extra-y)) $(subdir-ym) $(always) 
       @: 
```
该规则中的依赖由几个变量\$(builtin-target) \$(lib-target) \$(extra-y)) \$(subdir-ym) \$(always)表示。规则的命令是一个冒号命令”:”，冒号(:)命令是bash的内建命令，通常把它看作true命令。没有作用，该命令是空操作，退出状态总是0，表示成功）。这里主要是构建$(always)变量指定的目标,在scripts/basic/Makefile（该文件在scripts/Makefile.build的第45行引入）中有:
```makefile?linenums=11
hostprogs-y     := fixdep
hostprogs-$(CONFIG_BUILD_BIN2C)     += bin2c
always          := $(hostprogs-y)

#fixdep 用于生成依赖信息
```
所以always的值为fixdep，其他变量在scripts/basic/Makefile中并没有定义。scripts/Makefile.build第64行

```makefile?linenums=64
ifneq ($(hostprogs-y)$(hostprogs-m)$(hostlibs-y)$(hostlibs-m)$(hostcxxlibs-y)$(hostcxxlibs-m),)
	include scripts/Makefile.host
 endif
```
由上面的分析可知hostprogs-y的值为fixdep，hostprogs-m值为空，因此scripts/Makefile.host一定会被引入。在scripts/Makefile.host的第24行：
```makefile?linenums=24
__hostprogs := $(sort $(hostprogs-y) $(hostprogs-m))
```
由上面分析知道__hostprogs值为fixdep。在scripts/Makefile.host的第52行：
```makefile?linenums=52
host-csingle    := $(addprefix $(obj)/,$(host-csingle))
```
由上面分析知道obj值为scripts/basic，因此，host-csingle的值为scirpt/basic/fixdep。在scripts/Makefile.host的第65行：
```makefile?linenums=65
_hostc_flags   = $(HOSTCFLAGS)   $(HOST_EXTRACFLAGS)   \   
                 $(HOSTCFLAGS_$(basetarget).o)
_hostcxx_flags = $(HOSTCXXFLAGS) $(HOST_EXTRACXXFLAGS) \
                 $(HOSTCXXFLAGS_$(basetarget).o)
                                   
ifeq ($(KBUILD_SRC),)              
__hostc_flags   = $(_hostc_flags)
__hostcxx_flags = $(_hostcxx_flags)
else                               
__hostc_flags   = -I$(obj) $(call flags,_hostc_flags)
__hostcxx_flags = -I$(obj) $(call flags,_hostcxx_flags)
endif                              
 
hostc_flags    = -Wp,-MD,$(depfile) $(__hostc_flags)
hostcxx_flags  = -Wp,-MD,$(depfile) $(__hostcxx_flags)
                                
#####                           
# Compile programs on the host  
                                
# Create executable from a single .c file
# host-csingle -> Executable    
quiet_cmd_host-csingle  = HOSTCC  $@
	cmd_host-csingle  = $(HOSTCC) $(hostc_flags) $(HOSTLDFLAGS) -o $@ $< \ 
					$(HOST_LOADLIBES) $(HOSTLOADLIBES_$(@F))
$(host-csingle): $(obj)/%: $(src)/%.c FORCE
         $(call if_changed_dep,host-csingle)
```		 
由2.2.1可知：

HOSTCC定义在顶级Makefile第361行，值为gcc；
HOSTCFLAGS定义在顶级Makefile第362行，值为`-Wall -Wmissing-prototypes -Wstrict-prototypes -O2 -fomit-frame-pointer -std=gnu89`；
HOST_EXTRACFLAGS值为空；
basetarget定义在scripts/kbuild.include的第24行，`basetarget = $(basename $(notdir $@))`，值为fixdep；
depfile定义在scripts/kbuild.include的第20行`depfile = $(subst $(comma),_,$(dot-target).d)`,值为scripts/basic/.fixdep.d。
KBUILD_SRC为空；
HOSTLDFLAGS 为空；
HOST_LOADLIBES 为空； 
HOSTLOADLIBES_fixdep 为空。

所以cmd_host-csingle展开为：
```makefile?linenums=87
cmd_host-csingle=gcc -Wp,-MD,scripts/basic/.fixdep.d -Wall -Wmissing-prototypes -Wstrict-prototypes \
		-O2 -fomit-frame-pointer -std=gnu89 -o scripts/basic/fixdep scripts/basic/fixdep.c
scirpt/basic/fixdep : scirpt/basic/%:scirpt/basic/%.c FORCE
		$(call if_changed_dep,host-csingle)
```
该规则主要用于编译生成fixdep。if_changed_dep定义在scripts/Kbuild.include中。

```makefile
echo-cmd = $(if $($(quiet)cmd_$(1)),echo '  $(call escsq,$($(quiet)cmd_$(1)))$(echo-why)';)

any-prereq = $(filter-out $(PHONY),$?) $(filter-out $(PHONY) $(wildcard $^),$^)

arg-check = $(if $(strip $(cmd_$@)),,1)

if_changed_dep = $(if $(strip $(any-prereq) $(arg-check) ), \
        @set -e;                                                
		$(cmd_and_fixdep), @:)

cmd_and_fixdep =                                \
        $(echo-cmd) $(cmd_$(1));               \
        scripts/basic/fixdep $(depfile) $@ '$(make-cmd)' > $(dot-target).tmp;\
        rm -f $(depfile);                   \
        mv -f $(dot-target).tmp $(dot-target).cmd;
```
可以知道实if_changed_dep主要就是调用cmd_host-csingle生成fixdep。

（3）outputmakefile 
&emsp;定义在顶层Makefile中：
```makefile?linenums=470
PHONY += outputmakefile
outputmakefile:
ifneq ($(KBUILD_SRC),)
        $(Q)ln -fsn $(srctree) source
        $(Q)$(CONFIG_SHELL) $(srctree)/scripts/mkmakefile \
            $(srctree) $(objtree) $(VERSION) $(PATCHLEVEL)
endif
```

这个规则的命令运行一个shell脚本scripts/mkmakefile，并传递四个参数。这个脚本主要是在$(objtree)参数指定的目录中生成一个Makefile文件。由于这里KBUILD_SRC为空，所以这个脚本并不会被执行。

-----

至此，menuconfig的三个依赖项已经分析完毕，回到一开始的那个规则 ：
  
```makefile
%config: scripts_basic outputmakefile FORCE
	$(Q)$(MAKE) $(build)=scripts/kconfig $@
```
由前面的分析知道，改规则可展开为：
```makefile
menuconfig: scripts_basic outputmakefile FORCE
	@make -f ./scripts/Makefile.build obj=scripts/kconfig menuconfig
```
在三个依赖被处理完后，开始执行规则的命令。在scripts/Makefile.build的第6行： 
```makefile?linenums=6
src := $(obj) #这就把的scripts/kconfig赋给了src,这个src并非顶级Makefile中的src
```

在scripts/Makefile.build的第43，44，45行，把src(即scripts/kconfig)目录下的Makefile包含进来（如果有Kbuild则包含Kbuild）,但是并没有引入到顶级Makefile中：
```makefile?linenums=43
kbuild-dir := $(if $(filter /%,$(src)),$(src),$(srctree)/$(src))
kbuild-file := $(if $(wildcard $(kbuild-dir)/Kbuild),$(kbuild-dir)/Kbuild,$(kbuild-dir)/Makefile)
include $(kbuild-file) 
``` 
在scripts/kconfig/Makefile中
```makefile?linenums=28
menuconfig: $(obj)/mconf
         $< $(silent) $(Kconfig)  
```

silent变量是scripts/kconfig/Makefile中本地变量；而quite在顶层Makefile定义；KBUILD_KCONFIG由命令行参数传入，指向另一个Kconfig文件。
```makefile?linenums=9
ifdef KBUILD_KCONFIG
Kconfig := $(KBUILD_KCONFIG)
else       
Kconfig := Kconfig
endif   
           
ifeq ($(quiet),silent_)
silent := -s
endif   
```
解析完两个变量后，scripts/kconfig/Makefile中的 menuconfig规则可以展开为：
```makefile?linenums=28
menuconfig: scripts/kconfig/mconf
          scripts/kconfig/mconf -s Kconfig
```
mconf的构建是整个Kernel Makefile中最基础也是非常精彩的部分，linux中大部分子模块均是按照这样的方式组织的。 由scripts/kconfig/Makefile中的一些规则生成：
```makefile?linenums=179
hostprogs-y     += mconf
lxdialog        := checklist.o inputbox.o menubox.o textbox.o util.o yesno.o
mconf-objs      := mconf.o zconf.tab.o $(addprefix lxdialog/, $(lxdialog))

$(obj)/mconf.o: $(obj)/.mconf-cfg

$(obj)/.%conf-cfg: $(src)/%conf-cfg.sh FORCE
         $(call filechk,conf_cfg)	 
```
filechk定义在scripts/kbuild.include中。在scripts/Makefile.build第64行

```makefile?linenums=64
ifneq ($(hostprogs-y)$(hostprogs-m)$(hostlibs-y)$(hostlibs-m)$(hostcxxlibs-y)$(hostcxxlibs-m),)
	include scripts/Makefile.host
 endif
```
由上面的分析知道scripts/kconfig/Makefile是在Makefile.build的第45行引入，所以hostprogs-y的值至少含有mconf，因此scripts/Makefile.host一定会被引入。mconf主要由scripts/Makefile.host中的以下规则构建：
```makefile?linenums=24
__hostprogs := $(sort $(hostprogs-y) $(hostprogs-m))

host-cmulti     := $(foreach m,$(__hostprogs),\
                 $(if $($(m)-cxxobjs),,$(if $($(m)-objs),$(m)))) 
host-cmulti     := $(addprefix $(obj)/,$(host-cmulti))

host-cobjs      := $(addprefix $(obj)/,$(host-cobjs))				 
host-cobjs      := $(sort $(foreach m,$(__hostprogs),$($(m)-objs)))

quiet_cmd_host-cmulti   = HOSTLD  $@
       cmd_host-cmulti   = $(HOSTCC) $(HOSTLDFLAGS) -o $@ \
                           $(addprefix $(obj)/,$($(@F)-objs)) \
                           $(HOST_LOADLIBES) $(HOSTLOADLIBES_$(@F))
$(host-cmulti): FORCE
        $(call if_changed,host-cmulti)
$(call multi_depend, $(host-cmulti), , -objs)
quiet_cmd_host-cobjs    = HOSTCC  $@
       cmd_host-cobjs    = $(HOSTCC) $(hostc_flags) -c -o $@ $<			 
$(host-cobjs): $(obj)/%.o: $(src)/%.c FORCE
        $(call if_changed_dep,host-cobjs)

```
与上面fixdep构建过程类似，if_changed_dep主要调用 cmd_host-cobjs产生多个.o文件，然后if_changed调用cmd_host-cmulti生成mconf。mconf与.o文件的依赖关系是通过`multi_depend`建立，该函数定义在scripts.lib中。

----

概括起来：

* 首先由make编译生成`scripts/kconfig/mconf。（xconfig对应qconf，gconfig对应gconf，config对应conf）
* 然后执行`scripts/kconfig/mconf Kconfig`
* mconf程序读取内核根目录下的Kconfig文件，Kconfig载入了`arch/$(SRCARCH)/Kconfig`，`arch/$(SRCARCH)/Kconfig`又分别载入各目录下的Kconfig文件，以此递归下去，最后生成主配置界面以及各级配置菜单。`SRCARCH`是由顶层Makefile中定义的，它等于`$(ARCH)`，而`$ARCH`由Makefile或make的命令行参数指定。
* 在完成配置后，mconf会将配置保存在Linux内核源代码根目录下的`.config`文件中。.config文件实质是一个Makefile文件。

* 当我们使用`make defconfig`这个命令时：系统直接将`arch/$SRCARCH/configs`（**该目录存放内核的默认配置文件**）下的对应的默认配置文件拷贝到Linux内核源代码根目录下的`.config`文件。

## 2.3 Kconfig语法

&emsp;&emsp;linux在2.6版本以后将配置文件由原来的config.in改为Kconfig，对于Kconfig的语法在内核源代码`Documentation/kbuild/kconfig-language.txt`中做了详细的说明。
### 2.3.1 Kconfig格式
```
菜单入口 "菜单入口名"
   [依赖]
   [反向依赖]
   [引入其它Kconfig文件]
   ... ...
   [帮助]
    
    
配置项 symbol
   值类型
   [输入提示]
   [默认值]
   [依赖]
   [反向依赖]
   ... ...
   [帮助]
```

### 2.3.2 属性
* `bool/tristate/int/hex/string`
值类型，只有配置选项有值类型。包括： bool——值为y或n、 tristate——值为y或m或n、string——值为字符串、int——值为十进制整数、 hex——值为十六进制整数

* `prompt "提示字符串" [if <expr>]`
输入提示：每个菜单入口最多只能有一个展示给用户看的输入提示，可以使用“if”来表示输入提示的依赖性，这个依赖性是可选的。

* `default <expr> [if <expr>]`
默认值：一个配置选项可以有任意多个默认值，但只有第一个有效的。当输入提示是可见时，才能看到默认值，并且可以输入一个值将默认值覆盖。与输入提示一样，可以使用“if”来表示默认值的依赖性，这个依赖性是可选的。

* `def_bool/def_tristate <expr> ["if" <expr>]`
值类型+默认值：说明值类型时同时说明默认值。可以使用“if”来表示默认值的依赖性，这个依赖性是可选的。

* `requires(或depends on）<expr>`，
依赖: 依赖项给菜单入口或配置项定义了一个依赖规则，只有当expr为真或被选中时，该菜单入口或配置项才有效，依赖对菜单入口或配置项中的其它属性都有效。

* `select <symbol> ["if" <expr>]`
反向依赖：当前配置项被选中时，选中symbol，忽略依赖项和手动设置值。**只有boolean和tristate类型的symbol可以使用反向依赖**。

* `imply <symbol> [if <expr>]`
弱反向依赖：当前配置项被选中时，选中symbol，但是symbol仍然可由依赖或者手动配置为n。

* `range <value1> <value2> ["if"<expr>]`，
值范围：**限定int和hex类型symbol的输入值**。用户只能输入一个大于等于第一个symbol的值，并且小于等于第二个symbol的值。

* `visible if <expr>`
只能用于menu/endmenu中，当expr为真时菜单可见。

* `---help---/help`
帮助:定义了一个帮助文本。帮助文本的结尾是根据缩进级别来决定的，这就意味着如果帮助文本中某一行相对于第一行有更小的缩进，那么这一行就是帮助文档的最后一行。"---help---"和“help”在使用功能上没有区别，"---help---"是用来作为对开发者的一种提示，它显式区别于文档中的配置选项“help”。

* `option <symbol>[=<value>]`
其它属性:各种不常见的选项的通过这个选项来定义，比如修改菜单入口的行为和配置symbol。下面这些配置当前是允许的：
 
	 * `defconfig_list`
 定义了一系列默认入口，当使用默认配置时可以从这里寻找（当主.config文件不存在时会使用默认配置）
	 * `modules`
 声明了一个symbol将被当做MODULES symbol，MODULES symbol是所有配置symbol的第三种模块化状态。
	 * `env=<value>`，
 导入了一个环境变量到Kconfig中。环境变量就像是Kconfig中的一个默认值，但是它是从外部环境中导入的。正因为它从外部环境导入，所以赋值的这个时候它相对于正常的默认值来说是没有定义的。这个symbol当前没有导出到构建环境中（如果想要这样的话，可以通过另一个symbol导出）
	 * `allnoconfig_y`
 声明symol默认值为y。

>**补充**：
>
>* 值类型后可以紧跟一个输入提示（也可以单独使用一个提示属性），所以下面的这两个例子是等价的：
>```
>bool "Networking support"
>#和
>bool
>prompt "Networking support"
>```
>* 依赖对菜单入口中的其它属性都有效，下面两种写法是等价的
>```
>bool"foo" if BAR
>default y if BAR
>#和
>depends on BAR
>bool "foo"
>default y
>```
>* 并非每个菜单入口或者配置项都具有所有的属性。
>* 限制一个配置项只能编译为模块或不选择。
>```
>config FOO
>       dependson m
>```
>* 限制一个配置项只能编译或模块化
>```
>config FOO
>   tristate "foo"
>   select FOO if m
>   default m
>```
> * 如果一个配置项只有值类型属性，则它的选中只能通过其他配置项的select属性


### 2.3.3 表达式expr
```
<expr> ::= <symbol>                             (1)
           <symbol> '=' <symbol>                (2)
           <symbol> '!=' <symbol>               (3)
           <symbol1> '<' <symbol2>              (4)
           <symbol1> '>' <symbol2>              (4)
           <symbol1> '<=' <symbol2>             (4)
           <symbol1> '>=' <symbol2>             (4)
           '(' <expr> ')'                       (5)
           '!' <expr>                           (6)
           <expr> '&&' <expr>                   (7)
           <expr> '||' <expr>                   (8)


``` 

* 表达式以降序的顺序排列在下面。
	 1. 将一个symbol转换成表达式。Bool和tristate symbol简单地转换成相应的表达式值。其它类型的symbol就转换成‘n’。
	 2. 如果两个symbol的值相等，就返回‘y’，否则返回‘n’。
	 3. 如果两个symbol的值不相等，就返回‘n’，否则返回‘y’。
	 4. 如果symbol1的值小于、大于、小于等于、大于等于symbol2的值返回y，否则返回n。
	 5. 返回表达式的值。用来覆盖之前的值
	 5. 返回（2-表达式的值）
	 6. 返回min（expr,expr2）
	 7. 返回max（expr,expr2）

* 一个表达式的值可以是'n','m'或'y'（或者相对于0,1,2）。当表达式的值是m或者y的时候，菜单入口就是可见的。

* 存在两种类型的symbol：常数symbol和非常数symbol。非常数symbol是最常见的一类symbol，定义的时候使用‘config’来声明。非常数symbol由字母和下划线组成。

* 常数symbol只是表达式的一部分。常数symbol通常被单引号或者双引号包围着。在引号中，任何字母都是允许的，并且可以使用‘\’进行转义。

 
### 2.3.4 菜单入口

* 主菜单——最顶层的菜单
```
mainmenu "主菜单名字"
    
```
* 多选菜单——带配置项但本身不可配置。**它的属性只能是依赖项和可见性**。
```
menu "string"
    ... ...
endmenu 
```
* 可选菜单——带配置项且本身是配置项。配置关键字前面添加CONFIG_后就构成了“.config”文件中的配置项名字。
```
menuconfig 配置关键字
    ... ...
```
* config——配置项，配置关键字前面添加CONFIG_后就构成了“.config”文件中的配置项名字，但不是配置界面显示的字符，配置界面显示的是提示字符。
```
config 配置关键字
    ... ...
```

* 单选菜单，**单选菜单只能是bool类型或tristate类型** ，并且布尔选择只允许一个单一的配置项被选中，三态选择还允许任何配置项被设置为“M”。这可以用在下面的情况：如果一个硬件存在多个驱动程序，并且只有一个驱动程序可以编译/加载到内核中，但所有的驱动程序可以编译成模块。
```
 choice
    ... ...
 endchoice
```

* 注释，这定义了一条在用户配置过程中显示的注释，同时会写入导出文件。**它的属性只有依赖项**。
```
 comment
```
* 条件——当expr为真或选中时，中间的内容才有效。
```
if <expr>
 ... ...
endif
```

* 引入其他Kconfig文件，方便菜单嵌套。
```
source "...dir/Kconfig"
```


### 2.3.5 菜单结构
* 一种是使用了菜单入口明确指定，如下中所有位于“menu”…和“endmenu”之间的入口都是"Network device support"的一个子菜单入口。所有的子入口都继承了菜单入口的依赖项，例如，依赖项”NET”就会被加入到子菜单”NETDEVICESx”的依赖项列表中。
```
menu "Network device support"
    depends on NET
    config NETDEVICES1
       ...
    config NETDEVICES2
       ... 
    menuconfig NETDEVICES3
       ...
        config NETDEVICES1
            ...
        config NETDEVICES2
            ... 
endmenu
```

* 另外一种生成菜单结构的方法是通过分析依赖项。如果一个菜单入口依赖依赖于前一个入口，那么它就是前一个入口的一个子菜单。首先，之前的（父）symbol一定位于子入口的依赖列表中，其次，下面两个条件中有一个必须是真的：
 * 子入口必须是不可见的，当父symbol被设置成’n’
 * 子入口必须是可见的，当父菜单是可见的
```
config MODULES
    bool"Enable loadable module support"
config MODVERSIONS
    bool"Set version information on all module symbols"
    dependson MODULES
comment "module support disabled"
    dependson !MODULES
```



# 3  内核编译过程
* 在输入编译命令后，make首先读取`.config`文件（实际是一个Makefile），并根据配置项载入对应头文件到`include/config/`，并将一些配置项写入`include/config/auto.conf`。
* 脚本程序将`include/config/auto.conf`中的配置项`CONFIG_XXXX=y|m|xxx`翻译为宏定义`#define CONFIG_XXXX[_MODULE] 1|xxx`，并写入`include/generate/autoconf.h`中。
* autoconf.h就是`include/config/auto.conf`中的配置项的内容的C语言写法，以便在以后使用的时候作为宏定义出现，以实现条件编译。
* make根据Makefile执行编译。

# 4 在内核中添加程序
* 将源代码拷贝到内核源码的相应目录
* 修改对应目录下的Kconfig文件，按照Kconfig语法增加对应的选项；
* 修改对应目录下的Makefile文件完成编译选项的添加`obj-$(CONFIG_symbol)+= filename.o`；


------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>《Linux内核文档》。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
