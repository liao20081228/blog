---
title:  GNU make与Makefile教程
tags: Linux项目管理
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>**陈皓**先生的[《跟我一起写makefile》](https://blog.csdn.net/haoel/article/details/2886)，并根据最新的[《GNU make手册》](http://www.gnu.org/software/make/manual/make.html)（截止2018年5月），以及[《Linux man pages》](https://linux.die.net/man/ "点击跳转")做了修改，增添了一部分内容。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------


# 1 工程管理器make
## 1.1 make简介
* make是什么？Make是GCC提供的一种半自动化的工程管理器。所谓的**半自动化是指在使用工程管理器之前需要人工编写程序的编译规则**，所有的编译规则都保存在Makefile文件中，全自动化工程管理器会在编译程序前自动生成Makefile文件。

* 为什么要有工程管理器Make？因为在实际的开发过程中，仅仅通过gcc命令对程序进行编译时非常低效的，原因主要有两点：
     * 程序往往是由多个源文件组成的，源文件的个数越多，那么gcc的命令行就会越长。此外，各种编译规则也会加大gcc命令行的复杂度，所以在开发调试的过程中，通过输入gcc命令行来编译程序是很麻烦的。
     *  在程序的整个开发过程中，调试的工作量占到了整体工作量的70％以上。在调试程序的过程中，每次调试一般只会修改部分源文件。而在使用gcc命令行编译程序时，gcc会把那些没有被修改的源文件一起编译，这样就会影响变异的总体效率。

* 工程管理器Make有哪些优越性？
	* **使用方便**。通过命令“make”就可以启动MAKE工程管理器对程序进行编译，所以不再需要每次都输入GCC命令行。MAKE启动后会根据Makefile文件中的编译规则命令自动对源文件进行编译和链接，最终生成可执行文件。
	* **调试效率高**。为了提高编译程序的效率，Make会检查每个源文件的修改时间（时间戳）。只有在上次编译之后被修改的源文件才会在接下来的编译过程中被编译和链接，这样就能避免多余的编译工作量。为了保证源文件具有正确的时间戳，必须保证操作系统时间的准确性。

## 1.2 make的退出码
<style>table{word-break:initial;}</style>

|退出码|描述|
|--|--|
|0|表示成功执行。|
|1|使用make的“-q”选项时，如果指定目标需要更新，make退出码1，否则为0。|
|2| 如果make运行时出现任何错误|



## 1.3 make命令详解
**命令**：make [options] [target] ...
**描述**：根据Makefile自动编译。默认执行第一个目标。


|短选项|长选项|描述(版本4.1)|
|:--|:--|:--|
|-b, -m||为了兼容性，忽略这些选项|
|-B|--always-make|无条件重建所有目标，即使目标是最新的|
|**-C dir**|--directory=dir|切换到dir目录，读取并执行makefile文件。默认当前目录。使用多个-C命令行选项时，后一个指定的目录是前一个指定的目录的子目录。|
|-d||打印大量调试信息。调试信息指出正在考虑重新创建哪些文件，正在比较哪些文件时间以及结果如何，哪些文件真的需要重编译，哪些隐式规则会被考虑并应用|
||--debug[=FLAGS]|打印各种调试信息。如果FLAGS被省略，则与-d相同。FLAGS为b适用于基本调试，v适用于更详细的基本调试，i适用于显示隐式规则，j适用于调用命令的详细信息，m适用于调试时重新生成文件。n禁用所有以前的调试标志。|
|-e|--environment-overrides|系统环境变量将覆盖 makefile 中定义的同名变量|
|**-f FILE**| --file=file, <br />--makefile=FILE|读取FILE作为一个makefile。默认读取GNUmakefile、makefile、Makefile
|-h||打印帮助消息。
|**-i**|--ignore-errors|执行某个命令时如果发生错误则忽略并继续往下执行，不指定该选项则停止
|**-I dir**|--include-dir=dir|指定makefile中引入的其它makefile文件的搜索目录，使用多个-I 选项时，make 将按照顺序依次在 dir目录下搜索
|**-j [N]**|--jobs[=jobs] |同时允许N个任务；无参数表明允许无限个任务。如果多个-j选项，则最后一个有效
|**-k**|--keep-going|当某些目标无法创建时尽可能多的继续。
|-l [N]|--load-average[=load]|不开始新的作业除非系统负载低于N。省略N则删除之前ed负载限制
|-L|--check-symlink-times|在符号链接和目标之间使用最新的mtime。
|**-n**|--just-print, <br />--dry-run, --recon|仅显示要执行的命令，但不执行命令。
|-o FILE|--old-file=file,<br /> --assume-old=file| 强制将FILE认作非常老，不要重新 make 它。
|-O[type]|--output-sync[=type]|当与-j并行运行多个作业时，确保每个作业的输出都被收集在一起，而不是散布在其他作业的输出中。如果type被省略或是target，则将每个目标的输出组在一起。如果type是line，每个命令的输出将被组在一起。如果type是recurse，则整个递归make的输出被组在一起。如果type是none，则禁用输出同步。
|-p|--print-data-base|打印读取makefile所产生的数据库（规则和变量值）
|-q|--question|不运行命令；退出状态为0则目标为最新，否则目标过期。
|-r|--no-builtin-rules|禁用内置隐含规则。  同时清除后缀规则的默认后缀列表。
|-R|--no-builtin-variables|禁用内置变量设置。
|**-s**|--silent, --quiet|执行时不打印执行的命令。
|-S|--no-keep-going, --stop|关闭 -k。除非在递归make中-k可能通过MAKEFLAGS从顶层make继承，或者在环境中的MAKEFLAGS中设置-k，否则该选项是不需要的。
|-t|--touch |使用touch将目标的时间标记为最新，而不重新构建它们。
||--trace|打印每个目标的处置信息（为什么要重建目标以及运行哪些命令来重建目标）。
|-v| --version|打印 make 的版本号并退出。
|-w|--print-directory |在其它处理前后打印工作目录。
||--no-print-directory|即使 -w 隐式开启，也要关闭 -w。
|-W FILE|--what-if=file, --new-file=file,<br /> --assume-new=file|make将把file的最近修改时间视为当前时间，与-n联用时可以看到当文件修改后会发生的事情
||--warn-undefined-variables|当引用未定义的变量时打印警告信息。
|&emsp;&emsp;&emsp;&emsp;|&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;||



# 2 Makefile简介
&emsp;&emsp;Makefile存储着工程管理器Make进行工作所需的编译规则命令。

## 2.1 make的工作过程
&emsp;&emsp;例如，有 Makefile 文件，内容如下：
```makefile
main.exe:main.o func.o //有头文件时要加入头文件
	g++ -o main.exe main.o func.o 
main.o:main.cpp 
	g++ -c main.cpp 
func.o:func.cpp 
	g++ -c func.cpp
```
对于该 Makefile 文件,程序 make 处理过程如下:

&emsp;&emsp;1. make程序首先读到第1行的目标文件`main.exe`和它的两个依赖文件`main.o`和`func.o`;然后比较文件`main.exe`和`main.o`、`func.o`的产生时间，如果 `main.exe` 比 `main.o`或`func.o` 旧的话，则执行第2行命令，以产生目标文件`main.exe`。
&emsp;&emsp;2. 在执行第2行的命令前,它首先会查看makefile中的其他定义，看有没有以第1行` main.o `和 `func.o`为目标文件的依赖文件，如果有的话，继续按照1、2的方式匹配下去。
&emsp;&emsp;3. 根据2的匹配过程，make 程序发现第3行有目标文件` main.o` 依赖于`main.cpp`，则比较目` main.o `与它的依赖文件 `main.cpp` 的文件新旧,如果`main.o`比`main.cpp`旧,则执行第4行的命令以产生目标文件`main.o`。
&emsp;&emsp;4. 在执行第 4 条命令时，`main.cpp`在文件makefile不再有依赖文件的定义，make程序不再继续往下匹配，而是执行第4条命令，产生目标文件`main.o`。目标文件 `func.o`按照上面的同样方式判断产生。
&emsp;&emsp;5. 执行3、4产生完` main.o `和 `func.o `以后，则第 2 行的命令可以顺利地执行了，最终产生了第 1 行的目标文件` main.exe`。

## 2.2 Makefile组成

* 规则
   * **显式规则**。显式规则说明了，如何生成一个或多个目标文件。这是由Makefile的书写者明显指出，要生成的文件，文件的依赖文件，生成的命令。
   * **隐式规则**。由于我们的make有自动推导的功能，所以隐晦的规则可以让我们比较粗糙地简略地书写Makefile，这是由make所支持的。
* **变量定义**。在Makefile中我们要定义一系列的变量，变量一般都是字符串，当Makefile被执行时，其中的变量都会被扩展到相应的引用位置上。
* **文件指令**。其包括了三个部分，一个是在一个Makefile中引用另一个Makefile，就像C语言中的include一样；另一个是指根据某些情况指定Makefile中的有效部分，就像C语言中的预编译#if一样；还有就是定义一个多行的命令。
* **注释**。Makefile中只有行注释，和UNIX的Shell脚本一样，其注释是用“#”字符，如果你要在你的Makefile中使用“#”字符，可以用反斜线进行转义，如：“\#”。

## 2.3 Makefile长行分割和注释
* Makefile使用`\`来进行长行分割。
* Makefile使用`#`进行注释。

## 2.4 Makefile文件名
&emsp;&emsp;默认的情况下，make命令会在当前目录下按顺序找寻文件名为“GNUmakefile”、“gnumakefile”，“makefile”、“Makefile”的文件。
&emsp;&emsp;当然，你可以使用别的文件名来书写Makefile，比如：“xxxx”，但要用make的“-f FILE”选项指定该文件为Makefile文件，如：`make -f xxxx`。

## 2.5 引入其它的Makefile

&emsp;&emsp;在Makefile使用include关键字可以把别的Makefile包含进来，这很像C语言的`#include<xxx.h>`，被包含的文件会原模原样的放在当前文件的包含位置。include的语法是：
```makefile
include filename1 filename2... #filename可以保含路径和shell通配符
```
&emsp;&emsp;make处理include指令时，会找寻所指出的其它Makefile，并把其内容安置在当前的位置。就好像C/C++的`#include`一样。如果文件都没有指定绝对路径或是相对路径的话，make会在当前目录下首先寻找，如果当前目录下没有找到，那么，make还会在下面的几个目录下找：

* 如果make执行时，有`-I`或`--include-dir`参数，那么make就会在这个参数所指定的目录下去寻找。
* 如果目录`<prefix>/include`（一般是：`/usr/local/bin`或`/usr/include`）存在的话，make也会去找。

&emsp;&emsp;如果有文件没有找到的话，make会生成一条警告信息，但不会马上出现致命错误。它会继续载入其它的文件，一旦完成makefile的读取，make会再重试这些没有找到，或是不能读取的文件，如果还是不行，make才会出现一条致命信息。如果你想让make不理那些无法读取的文件，而继续执行，你可以在include前加一个减号“-”，表示无论include过程中出现什么错误，都不要报错继续执行，如 `-include <filename>`。

## 2.6 环境变量MAKEFILES

&emsp;&emsp;如果你的makefile中定义了环境变量MAKEFILES，那么，make会把这个变量中的值做一个类似于include指令的动作。这个变量中的值是其它的Makefile，用空格分隔。它和include不同的是，从这个环境变中引入的Makefile的“目标”不会成为默认目标(但是可以手动指定），如果MAKEFILES中列出的文件未发现，make也不认为其是一个error。





## 2.7 覆盖其它Makefile的部分
* 对于`inlclude`引入的其它makefile，相同的规则（目标与依赖都相同），在后面展开的的会覆盖在前面展开的（）
* GNUmakefile，gnumakefile，makefile，Makefile。优先级由高到低，因此前面的会覆盖后面的文件。



## 2.8 如何读取makefile

* GNU的make工作时的执行步骤入下：
	* 读入阶段
		1. 读入所有的Makefile。
		2. 读入被include的其它Makefile。
		3. 内化文件中的变量及值，隐式和显式规则
		4. 为所有的目标及其先决条件文件创建依赖关系链。
	* 目标更新阶段：
	  5. 根据依赖关系，决定哪些目标要重新生成。
	  6. 执行命令。

* 展开：将变量、函数的值放在对应的位置。分为：
	* 立即展开：在第一阶段中发生的展开。
	* 延后展开：直到构建上下文出现或者第二阶段才发生的展开。

* 变量赋值
```makefile
immediate = deferred
immediate ?= deferred
immediate := immediate
immediate ::= immediate
immediate += deferred or immediate //右值是用:=或::=定义的则立即展开，否则延后展开
immediate != immediate

define immediate
  deferred
endef

define immediate =
  deferred
endef

define immediate ?=
  deferred
endef

define immediate :=
  immediate
endef

define immediate ::=
  immediate
endef

define immediate +=
  deferred or immediate
endef

define immediate !=
  immediate
endef
```
* 条件指令：立即展开
* 规则：目标和先决条件总是立即展开，静态模式和命令总是延后展开
```makefile
immediate : immediate ; deferred
    deferred
```

## 2.9 再次展开 

&emsp;&emsp;第一步变量的展开中，有时候我们并不希望某些变量在规则中的目标、先决条件确定之前或者其他变量赋值之前就展开，那样就无法得到我们想要的值，这个时候就需要对这个Makefile进行二次展开，就是说make程序在第一次解析完Makefile后，再次对Makefile进行解析，对一些推迟展开的变量进行展开。比如：默认情况下，自动变量是只能在命令中使用的，因为第一次展开之前make是无法知道这些自动变量的值的，等到第一次展开之后，所有的自动变量的值都已经确定，这时候就可以让make进行二次展开，从而在命令之外使用自动变量。

&emsp;&emsp;如果要让make知道需要对某个Makefile进行二次展开的话，那么就需要在Makefile中定义`.SECONDEXPANSION`这个特殊的目标，一般情况下只需要写明这个目标，然后prerequisite和recipe都可以放空。如果默认延后展开，则可以省略`.SECONDEXPANSION`。此外还需要在变量引用的时候再多加一个\$符号，就是说变量名前面有两个连续的\$符号，所以如果在二次展开使能的情况下，如果你想要在prerequisite中使用一个$符号的时候，这时候你就需要写连续的四个\$符号，即\$\$，因为在Makefile中都是以\$符号作为变量引用的开始，在每一次解析中，需要连续两个\$符号，make才会知道那是一个\$字符。
```makefile
#例1

dirs=a b c
all:
	for d in $(dirs); do echo $$d; done;

#例2

.SECONDEXPANSION:
AVAR = top
onefile: $(AVAR)
twofile: $$(AVAR)
AVAR = bottom
```
# 3 显式规则

```makefile
target: dependency  #目标项:先决条件
    command1           
    command2  #必须以Tab键开头，command为更新目标的命令，支持shell脚本

#或

target: dependency_files; command1   
    command2
```
规则规定了两件事：

* 如何判断目标项是否过期：如果目标不存在或者先决条件新于目标项
* 怎样更新目标项。
* **依赖关系中最好加入头文件**。原因在于头文件改变，实现文件未改变时，不会重新编译目标。

## 3.1 目标项和先决条件
### 3.1.1 先决条件的类型
* **一般先决条件**：当普通先决条件新于目标项时，默认情况下目标项要更新。
* **顺序先决条件**：用`普通先决条件|顺序先决条件`表示，其仅表示一个先执行的操作，当顺序先决条件新于目标项时，目标项不需要更新。例如：将目标文件放在单独的目录中，因此需要先创建目录，但是任何对目录项的操作都会引起目录修改时间更新，如果目录是普通先决条件，则会导致目标需要更新而重新构建。
```makefile
 OBJDIR := objdir              
 OBJS := $(OBJDIR)/add.o $(OBJDIR)/main.o

$(OBJDIR)/main.exe : $(OBJS)
	gcc -o $@ $^
$(OBJDIR)/%.o : %.c |$(OBJDIR)
	gcc -c $< -o $@
$(OBJDIR) :
	mkdir $(OBJDIR)

```
### 3.1.2 使用通配符
|符号|含义|
|:--|:--|
|*|匹配0个或多个字符|
|?|匹配1个字符|
|[abc]|匹配abc中的任意1个字符|
|[!abc]或[^abc]|匹配除abc之外的任意1个字符|
|{*.c,ab}|匹配以.c结尾的字符串和字符ab|


* 通配符可用于规则中，但是如果在第一阶段展开时，没有文件与之匹配时就解析为字符串
* 用通配符对变量赋值时解析为字符串
* 通配符展开时，是把所有匹配的结果都展开。

### 3.1.3 设置先决条件的搜索路径
* 使用make环境变量`VPATH=dir1:dir2`
* 使用`vpath`关键字（注意，它是全小写的），它可以指定不同的文件在不同的搜索目录中。它的使用方法有三种：
	 * `vpath <pattern> <directories>`
为符合模式<pattern>的文件指定搜索目录<directories>。
	 * `vpath <pattern>`
清除符合模式<pattern>的文件的搜索目录。   
	 * `vpath`
清除所有已被设置好了的文件搜索目录。
 * 我们可以连续地使用vpath语句，以指定不同搜索策略。如果连续的vpath语句中出现了语义重复的`<pattern>`，那么，make会按照vpath语句的先后顺序来执行搜索。
* 搜索路径也适用于隐式规则
* 如果先决条件是库`-lxxx`，则先搜索`-libxxx.so`再搜索`-libxxx.a`。先在指定路径搜索，再到系统默认路径中搜索。

### 3.1.4 伪目标
* `.PHONY`关键字表示它后面列表中的先决条件均为伪目标。
* **伪目标可以有先决条件**。
* 伪目标和相同的普通目标区别在于：
   * 伪目标的执行需要显式指定（除非只有伪目标），而make总是默认执行第一个普通目标。
   * 当存在同名文件时，后者会认为目标始终最新而不执行命令，伪目标则会执行命令。
   * 伪目标即使是最新的也要执行命令
```makefile
#例main.c中使用了fun.c中的函数，Makefile如下：

main.exe:main.o func.o 
	gcc  main.o func.o -o main.exe
main.o:main.c       
	gcc -c main.c -o main.o       #可以写作gcc -c main.c
func.o:func.c        
	gcc -c func.c    
.PHONY:rebuild clean              #表示后面的是伪目标
rebuild: clean main.exe           #先执行清理，在执行重新编译main.exe
clean: 
	rm –rf main.o func.o main.exe #最后删除.o 和.exe 的文件


$make         //直接 make,即从默认文件（Makefile）的第一个普通目标开始执行
$make clean   //表示执行目标 clean: 开始的命令段
$make func.o  //表示执行目标 func.o: 开始的命令段
$make rebuild //则先执行目标rebuild，先清除clean，再重新编译连接main.exe
```
### 3.1.5 不带先决条件或命令的规则
* 规则可以没有先决条件
* 规则可以没有先决条件和命令，如果一个规则不带先决条件或命令，并且规则的目标不是一个真正存在的文件，则make会而认为该目标更新过。 这意味着所有依赖于此目标的规则将始终运行其命令。
* 如果有先决条件，则先决条件必须有生成规则（除非使用了.DEFAULT) 

### 3.1.6 特殊目标
|名字|描述|先决条件|
|:--|:--|:--:|
|.PHONY|先决条件被视为伪目标，伪目标总是无条件执行|有|
|.SUFFIXES|其先决条件被用于定义隐式规则所需的后缀|有|
|.DEFAULT|为所有没有显式和隐式规则的目标指定生成命令，如果其命令为空则清除之前的设置|有|
|.PRECIOUS|如果在其先决条件的命令执行过程中make被杀死或中断，其先决条件不会被删除。如果其先决条件是中介文件,当其不需要时也不会删除|有|
|.INTERMEDIATE|他的先决条件被视为中间文件（比如.o文件），当其不再需要时会自动删除，如果没有先决条件则无效|有|
|.SECONDARY|他的先决条件被视为中间文件，但不自动删除，如果先决条件为空则将所有的目标都视为中介文件不自动删除|有|
|.SECONDEXPANSION|将其后的所有规则的先决条件及命令再次展开|无|
|.DELETE_ON_ERROR|如果某规则的命令非0退出（出错）则删除该规则的目标|无|
|.IGNORE|如果其先决条件的命令非0退出（出错）则忽略，与命令前加`-`作用相同|有
|.LOW_RESOLUTION_TIME|由先决条件的命令创建的文件使用低精度时间戳|有|
|.SILENT|先决条件的命令执行时不显示，与命令前加`@`作用相同|有|
|.EXPORT_ALL_VARIABLES|将所有变量输出给下一层make|无|
|.NOTPARALLEL|即使make指定-j选项，仍串行执行|无|
|.ONESHELL|将一个规则的所有命令在同一个shell中执行，而不是每行命令调用一次shell|无|
|.POSIX|makefile将以符合POSIX的模式进行解析和运行|有|
|||&emsp;&emsp;&emsp;&emsp;|


### 3.1.7 一个规则多个目标

```makefile
target1 tarhet2 : dependency
    command
    
#等价于

target1 : dependency
    command
tarhet2 : dependency
    command
```
### 3.1.8 一个目标多个规则
&emsp;&emsp;一个目标可有可以有多个规则，所有的先决条件都合并为先决条件列表，但是如果有多个命令则只有最后一个命令会执行。
```makefile
target1 : dependency
    command
    
target1 : dependency
    command
```
### 3.1.9 静态模式
```makefile
targets...: target-pattern: prereq-patterns
   command
```
*  设置匹配target-pattern的目标的先决条件为 prereq-patterns
 * targets定义了一系列的目标文件，可以有通配符。是目标的一个集合。
 * target-parrtern是指明了targets的模式，也就是的目标集模式。
 * prereq-parrterns是目标的依赖模式，它对target-parrtern形成的模式再进行一次依赖目标的定义。
*  与隐式规则的区别在于： 隐式规则适用于任何与目标模式匹配的目标，但要求有先决条件且没有命令。
 

### 3.1.10 双冒号规则
```makefile
target::dependency
   command
```
* 双冒号规则是在目标名称之后用'::'而不是'：'写的显式规则。
* 当同一个目标出现在多个规则中时
 * 普通规则只执行最后出现的规则的命令
 * 双冒号规则则会执行每个规则的命令，即使目标已经存在。

### 3.1.11 自动生成先决条件
* gcc/g++会自动生成依赖关系
   * -M：生成依赖关系包括库文件
   * -MM：生成依赖关系不包括库文件


## 3.2 命令
```makefile
target: dependency  //目标项:先决条件
    command1           
    command2  //必须以Tab键开头，command为更新目标的命令，支持shell脚本

#或

target: dependency_files; command1   
    command2
```
* 基本语法
	* 必须以TAB键开头
	* 连续多行一TAB键开头的命令为同一规则的命令
	* 命令的基本语法和shell脚本一样
		* 注释用#
		* 长行分割用\
	* 命令中可以使用makefile变量
* 命令显示
	* 命令以@开头表示只执行但不显示命令
	* make的-n选项表示只显示而不执行命令
	* -s选项表示执行时禁止显示所有命令
* 命令执行
	* 当先决条件新于目标项时（时间上新于或者先决条件不存在），就执行命令
	* make调用子shell执行命令
	* **每行命令调用一个新的shell**（除非使用了`.ONESHELL`特殊目标）
	* 如果你要让上一条命令的结果应用在下一条命令，要将命令放在同一行，用分号分隔
	* 可以使用预定义变量SHELL来指定shell，用.SHELLFLAGS指定shell选项
	* 并行执行
		* 正常情况，每次执行一条命令
		* make的-j n选项可以设置一次执行的命令数，-l n 选项可以设置最大负载
		* 特殊目标.NOTPARALLEL禁止并行执行命令
		* 并行执行时，结果一产生就输出，因此可能变得混乱，可以通过make 的`-O[type]`选项执行输出方式
			* 如果type被省略或type是target，则一但一个规则的所有命令执行完毕，则输出
			* 如果type是line，则一行的所有命令执行完毕，则输出
			* 如果type是recurse，则一但递归调用的所有命令执行完毕，则输出
			* 如果不指定`-O`选项或type是none，则禁用输出同步，每个命令的结果一产生就输出。
* 命令出错
	* 每当命令运行完后，make会检测每个命令的返回码，如果命令返回成功，那么make会执行下一条命令，当规则中所有的命令成功返回后，这个规则就算是成功完成了。如果一个规则中的某个命令出错了（命令退出码非零），那么make就会终止执行当前规则，这将有可能终止所有规则的执行。
	* 命令以-开头表示忽略命令执行过程中的错误
	* make选项`-i`，表示忽略所有命令执行过程中的错误
	* make选项`-k`，表示如果某规则中的命令出错了，那么就终目该规则的执行，但继续执行其它规则。
	* 特殊目标`.IGNORE`，表示忽略其先决条件所在规则的命令执行错误
* 递归执行make
	* 在一些大的工程中，我们会把我们不同模块或是不同功能的源文件放在不同的目录中，我们可以在每个目录中都书写一个该目录的Makefile，然后在工程目录上设置一个总控makefile
	* make调用
		* 递归使用make意味着使用make作为makefile中的命令，但不是直接使用make而是使用预定义变量`MAKE`
		* “-t”,“-n”,和“-q”会失效。
	* 变量传递
		* 总控Makefile的变量可以传递到下级的Makefile中，但是不会覆盖下层的Makefile中所定义的变量，除非指定了“-e”参数。
		* `export <variable...>`：传递变量到子make程序中。
		* `unexport <variable...>`：禁止变量传递到下级Makefile
		* `export`：传递所有的变量到下级Makefile
	* make选项传递
		* 通过预定义变量`MAKEFLAGS`传递
		* 选项“-C”,“-f”,“-h”“-o”和“-W”不会往下层传递
		* 选项“-w”输出执行前后的目录。
		* 选项“-C”来指定make下层Makefile，且“-w”会被自动打开的。
		* 选项“-s”或“--no-print-directory”自动关闭“-w”。
	* 多行命令
```makefile
define cmd_lable[=|+=|:=|?=]
    cmd1
    cmd2
    ...
endef
```

# 4 特殊字符
|字符|描述|
|:--|:--|
|\	|换行符或者转义符号|
|#|#	注释符,一定要另起一行不然容易出错|
|@|	如果命令以@开头则不显示当前执行的命令|
|%|	匹配1个或多个字符，匹配结果可以引用
|~|用户主目录或者宿主目录|
|-|命令以-打头，在执行时如果发生错误时，不中断make过程
|+|命令以+打头，无论make命令后面是否跟着-n,-t,-q三个参数，命令都会被执行。
|$$	|$字符
|$$$$	|使用了二阶段展开时的$字符
|$(var)|	取变量var的值

# 5 变量
&emsp;&emsp;在Makefile中的定义的变量，就像是C/C++语言中的宏一样，他代表了一个文本字串，在Makefile中执行的时候其会自动原模原样地展开在所使用的地方。在Makefile中，变量可以使用在“目标”，“依赖目标”，“命令”或是Makefile的其它部分中。

&emsp;&emsp;变量的命名字可以包含字符、数字，下划线（可以是数字开头），但不应该含有“:”、“#”、“=”或是空字符（空格、回车等）。推荐使用大小写搭配的变量名,可以避免和系统变量冲突。


## 5.1 变量定义和引用

* 定义变量和赋值
	* `变量名=值`：递规变量展开(几个变量共享一个值)，如`a=b；b=c；c=10；`，则执行时a就会与c相同
	* `变量名:=值`或 `变量名::=值`：简单变量展开。
	* `变量名?=值`：如果变量之前定义过，则什么也不做，否则，定义变量并赋值
	* `变量名!=shell脚本`：将shell脚本的输出赋值给变量
* 使用变量
	* `$(变量名)`或`${变量名}`  ：取某个变量的值
* 变量值替换，不会改变原来变量的值
	* `$(var:.a=.b)`或`${var:.a=.b}`，变量“var”中所有以“.a”结尾的字符串中的“.a”替换成“.b”。
	* 使用模式规则：`$(var:%.a=%.b)`，变量“var”中所有和%.a匹配的字符串替换为和%.b匹配的字符串。
* 把变量的值再当成变量：` var1 := $($(var2))`
* 变量追加：`var+=value`或`var:=$(var) value`
     * 如果变量之前没有定义过，那么，“+=”会自动变成“=”，
     * 如果前面有变量定义，那么“+=”会继承于前次操作的赋值符。
     * 如果前一次的是“:=”，那么“+=”会以“:=”作为其赋值符。
* 覆盖make命令行传入的参数变量：在变量定义前加上关键字`override`
* 多行变量
```makefile
define var_lable[=|+=|:=|?=]
var1
var2
...
endef
```
* 取消变量定义：`undefine var`
* 禁止继承变量：
	* `target:private 变量赋值`禁止先决条件继承目标变量，
	* `private 变量赋值`禁止任何规则和目标继承全局变量


## 5.2 变量分类
* 普通变量：在makefile中使用`variable-assignment`定义的变量，一般为全局变量
* 特定目标变量：只在指定目标的规则中有效的变量。`target:[override] variable-assign`
* 特定模式变量：就是将模式规则用于目标变量，`pattern: [override] variable-assignment`
* 命令行变量：变量由make命令行参数传入，使用`$make arg_var=value`定义
* 自动变量：指在使用的时候，自动用特定的值替换。常用的有：

|变量|	说明|
|:--|:--|
|@|当前规则的目标文件。在多目标的模式规则中，它代表的是触发规则被执行的目标文件名
|* |不包含扩展名的目标文件名称
|%|规则的目标文件是一个库文件时，代表库的第一个成员名，如果目标不是函数库文件，其值为空。
|<|当前规则的第一个依赖文件,如果是隐含规则，则它代表通过目标指定的第一个依赖文件
|^|当前规则的所有依赖文件，以空格分隔。如果目标是静态库文件名，它所代表的只能是所有库成员（.o文件）名。变量“$^”会去掉规则中重复的依赖文件。
|?|当前规则中日期新于目标文件的所有依赖文件列表，空格分隔，如果目标是静态库文件名，代表的是库成员（.o文件）的更新情况。
|+ |类似“\$^”，但是它保留了依赖文件中重复出现的文件。主要用在程序链接时，库的交叉引用场合。
|\| |所有的顺序先决条件，以空格分隔
|@D|目标文件的目录部分。如果`$@`不存在斜杠，其值就是`.`（当前目录）。等价于函数`$(dir $@)`
|@F|目标文件的文件名。`$(@F)`等价于函数`$(notdir $@)`
|\*D|不包含扩展名的目标文件名称的目录部分。如果`$@`不存在斜杠，其值就是`.`（当前目录）。
|\*F|不包含扩展名的目标文件名称的文件名部分。
|%D|当以如`archive(member)`形式静态库为目标时，分别表示库文件成员`member`名中的目录部分
|%F|当以如`archive(member)`形式静态库为目标时，分别表示库文件成员`member`名中的文件名部分。
|<D|规则中第一个依赖文件的目录部分
|<F|规则中第一个依赖文件的文件名部分
|^D|所有依赖文件的目录部分（不存在同一文件）
|^F|所有依赖文件的文件名部分（不存在同一文件）。
|+D|所有依赖文件的目录部分（可存在重复文件）。
|+F |所有依赖文件的文件名部分（可存在重复文件）。
|?D|当前规则中日期新于目标文件的所有依赖文件的目录部分。
|?F|当前规则中日期新于目标文件的所有依赖文件的文件名部分。
|&emsp;&emsp;||

* 预定义（默认）变量：内部事先定义好的变量，但是它的值是固定的，并且有些的值是为空的。

|默认变量|说明|普通变量|	说明|
|:--|:--|:--|:--|
|AR|函数库打包程序，默认“ar”。|ARFLAGS|库打包程序参数，默认是“rv”。
|AS|编译程序，默认是“as”|ASFLAGS|汇编程序参数
|CC|c编译器，默认cc	|CFLAGS|c编译器参数
|CXX|cPP编译器，默认为 g++|	CXXFLAGS|cPP编译器参数
|CPP|c预编译器，默认为$(CC) –E	|CPPFLAGS|c预编译器参数
|CO |从RCS文件中扩展文件程序。默认为co。|COFLAGS|RCS命令参数
|FC |Fortran 和 Ratfor的编译器和预处理器。默认是f77|FFLAGS|Fortran编译器参数|
|GET | 从SCCS文件中扩展文件的程序。默认是get| GFLAGS|SCCS “get”程序参数
|LEX |C或Ratfor的Lex方法分析器程序。默认是lex|LFLAGS|Lex文法分析器参数
|PC |Pascal语言编译程序。默认命令是pc|PFLAGS |Pascal编译器参数
|YACC |C程序的Yacc文法分析器。默认是yacc|YFLAGS |Yacc文法分析器参数
|YACCR |Ratfor程序的Yacc文法分析器。默认是yacc –r
|MAKEINFO |转换Texinfo源文件到Info文件程序。默认是makeinfo
|TEX |从TeX源文件创建TeX DVI文件的程序。默认是tex
|TEXI2DVI |从Texinfo源文件创建TeX DVI文件的程序。默认是texi2dvi
|WEAVE |转换Web到TeX的程序。默认是weave
|CWEAVE| 转换C Web 到 TeX的程序。默认是“cweave
|TANGLE| 转换Web到Pascal语言的程序。默认是tangle
|CTANGLE| 转换C Web 到 C。默认是ctangle
|RM |删除文件命令。默认是rm –f
|LDLIBS|库文件|LDFLAGS|链接程序参数。
|||RFLAGS|Ratfor的Fortran编译器参数
|||LINTFLAGS|lint参数|
|MAKE|默认为make|MAKEFLAGS|make参数
|SHELL|shell程序,默认为/bin/sh|SHELLFLAGS|shell参数

* 环境变量：设置make运行环境的变量，包括系统环境变量。注意：这与origin函数的结果可能不同。

|变量|	说明|
|:--|:--|
|VPATH|先决条件查找路径
|MAKEFILES|引入的makefile所在路径
|CURDIR|当前目录


* 特殊变量

|变量|	说明|
|:--|:--|
|MAKEFILE_LIST|make解析的makefile文件名列表|
|MAKE_RESTARTS|make重新开始的次数
|MAKE_TERMOUT|标准输出设备|
|MAKE_TERMERR|标准出错输出设备|
|.DEFAULT_GOAL|make的默认目标，缺省为第一个目标|
|.VARIABLES|该makefile中的全局变量,不包括特定目标变量和特定模式变量
|.FEATURES|该版本make支持的功能|
|.RECIPEPREFIX|指令行前缀，表示以此开头的行为指令，默认为TAB|
|.INCLUDE_DIRS|设置make搜索引入的其它makefile文件的路径|







# 6 条件语句

```makefile
conditional-directive
text-if-one-is-true
endif

//或

conditional-directive
text-if-true
else
text-if-false
endif

//或

conditional-directive-one
text-if-one-is-true
else conditional-directive-two
text-if-two-is-true
else
text-if-one-and-two-are-false
endif
```
* 条件指令
	 * `ifeq|ifneq (arg1, arg2)`
	 * `ifeq|ifneq 'arg1' 'arg2'`
	 * `ifeq|ifneq "arg1" "arg2"`
	 * `ifeq|ifneq "arg1" 'arg2'`
	 * `ifeq|ifneq 'arg1' "arg2"`
	 * `ifdef|ifndef variable-name`

# 7 函数
## 7.1 函数定义与调用

```makefile
#自定义函数
define fun
... ...
... ...
endef

#调用自定义函数

$(call function,arg1，arg2，...) #或
${call function，arg1,arg2...}

#调用内建函数

$(function arg1,arg2,...) #或
${function arg1,arg2,...}
```
在函数中形参用\$(1)、\$(2)、\$(3)……表示，$(0)表示函数本身。
## 7.2 字符串处函数
* `$(subst <from>,<to>,<text> )`
	 * 名称：字符串替换函数——subst。
	 * 功能：把字串`<text>`中的`<from>`字符串替换成`<to>`。
	 * 返回：函数返回被替换过后的字符串。
	 * 示例：`$(subst ee,EE,feet on the street)`，把“feet on the street”中的“ee”替换成“EE”，返回结果是“fEEt on the strEEt”。
* `$(patsubst <pattern>,<replacement>,<text> )`
	 * 名称：模式字符串替换函数——patsubst。
	 * 功能：查找`<text>`中的单词（单词以“空格”、“Tab”或“回车”“换行”分隔）是否符合模式`<pattern>`，如果匹配的话，则以`<replacement>`替换。这里，`<pattern>`可以包括通配符“%”，表示任意长度的字串。如果`<replacement>`中也包含“%”，那么，`<replacement>`中的这个“%”将是`<pattern>`中的那个“%”所代表的字串。（可以用“\”来转义，以“\%”来表示真实含义的“%”字符）
	 * 返回：函数返回被替换过后的字符串。
	 * 示例：`$(patsubst %.c,%.o,x.c.c bar.c)`，把字串“x.c.c bar.c”符合模式[%.c]的单词替换成[%.o]，返回结果是“x.c.o bar.o”
* `$(strip <string>)`
	* 名称：去空格函数——strip。
	* 功能：去掉`<string>`字串中开头和结尾的空字符。
	* 返回：返回被去掉空格的字符串值。
	* 示例：`$(strip a b c )`,把字串“a b c ”去到开头和结尾的空格，结果是“a b c”。
* `$(findstring <find>,<in> )`
	 * 名称：查找字符串函数——findstring。
	 * 功能：在字串`<in>`中查找`<find>`字串。
	 * 返回：如果找到，那么返回`<find>`，否则返回空字符串。
	 * 示例：` $(findstring a,a b c)`返回“a”字符串。
* `$(filter <pattern...>,<text> )`
	 * 名称：过滤函数——filter。
	 * 功能：以`<pattern>`模式过滤`<text>`字符串中的单词，保留符合模式`<pattern>`的单词。可以有多个模式。
	 * 返回：返回符合模式`<pattern>`的字串。
	 * 示例：`$(filter %.c %.s,foo.c bar.c baz.s ugh.h)` 返回“foo.c bar.c baz.s”。
* `$(filter-out <pattern...>,<text> )`
	* 名称：反过滤函数——filter-out。
	* 功能：以`<pattern>`模式过滤`<text>`字符串中的单词，去除符合模式`<pattern>`的单词。可以有多个模式。
	* 返回：返回不符合模式`<pattern>`的字串。
	* 示例：`$(filter-out %.c %.s,foo.c bar.c baz.s ugh.h)` 返回“ugh.h”。
* `$(sort <list>)`
	 * 名称：排序函数——sort。
	 * 功能：给字符串`<list>`中的单词排序（升序）。
	 * 返回：返回排序后的字符串。
	 * 示例：`$(sort foo bar lose)`返回“bar foo lose” 。
	 * 备注：sort函数会去掉`<list>`中相同的单词。
* `$(word <n>,<text> )`
	 * 名称：取单词函数——word。
	 * 功能：取字符串`<text>`中第`<n>`个单词。（从一开始）
	 * 返回：返回字符串`<text>`中第`<n>`个单词。如果`<n>`比`<text>`中的单词数要大，那么返回空字符串。
	 * 示例：`$(word 2, foo bar baz)`返回值是“bar”。
* `$(wordlist <s>,<e>,<text> )`
	 * 名称：取单词串函数——wordlist。
	 * 功能：从字符串`<text>`中取从`<s>`开始到`<e>`的单词串。`<s>`和`<e>`是一个数字。
	 * 返回：返回字符串`<text>`中从`<s>`到`<e>`的单词字串。如果`<s>`比`<text>`中的单词数要大，那么返回空字符串。如果`<e>`大于`<text>`的单词数，那么返回从`<s>`开始，到`<text>`结束的单词串。
	 * 示例：` $(wordlist 2, 3, foo bar baz)`返回值是“bar baz”。
* `$(words <text> )`
	 * 名称：单词个数统计函数——words。
	 * 功能：统计`<text>`中字符串中的单词个数。
	 * 返回：返回`<text>`中的单词数。
	 * 示例：`$(words, foo bar baz)`返回值是“3”。
	 * 备注：如果我们要取`<text>`中最后的一个单词，我们可以这样：`$(word 
	 $(words<text>),<text> )`。
* `$(firstword <text>)`
	 * 名称：首单词函数——firstword。
	 * 功能：取字符串`<text>`中的第一个单词。
	 * 返回：返回字符串`<text>`的第一个单词。
	 * 示例：`$(firstword foo bar)`返回值是“foo”。
	 * 备注：这个函数可以用word函数来实现：`$(word 1,<text> )`。
* `$(lastword <text>)`
	 * 名称：末单词函数——lastword。
	 * 功能：取字符串`<text>`中的最后一个单词。
	 * 返回：返回字符串`<text>`的最后一个单词。
	 * 示例：`$(lastword foo bar)`返回值是“bar”。
	 * 备注：这个函数可以用word函数来实现：`$(word $(words <text>),<text> )`。

## 7.3 文件名操作函数
* `$(dir <names...> )`
	 * 名称：取目录函数——dir。
	 * 功能：从文件名序列`<names>`中取出目录部分。目录部分是指最后一个反斜杠（“/”）之前的部分。如果没有反斜杠，那么返回“./”。
	 * 返回：返回文件名序列`<names>`的目录部分。
	 * 示例： `$(dir src/foo.c hacks)`返回值是“src/ ./”。
* `$(notdir <names...> )`
	 * 名称：取文件函数——notdir。
	 * 功能：从文件名序列`<names>`中取出非目录部分。非目录部分是指最后一个反斜杠（“/”）之后的部分。
	 * 返回：返回文件名序列`<names>`的非目录部分。
	 * 示例： `$(notdir src/foo.c hacks)`返回值是“foo.c hacks”。
* `$(suffix <names...> )`
	 * 名称：取后缀函数——suffix。
	 * 功能：从文件名序列`<names>`中取出各个文件名的后缀。
	 * 返回：返回文件名序列`<names>`的后缀序列，如果文件没有后缀，则返回空字串。
	 * 示例：`$(suffix src/foo.c src-1.0/bar.c hacks)`返回值是“.c .c”。
* `$(basename <names...> )`
	 * 名称：取前缀函数——basename。
	 * 功能：从文件名序列`<names>`中取出各个文件名的前缀部分,包括目录。
	 * 返回：返回文件名序列`<names>`的前缀序列，如果文件没有前缀，则返回空字串。
	 * 示例：`$(basename src/foo.c src-1.0/bar.c hacks)`返回值是“src/foo src-1.0/bar”。
* `$(addsuffix <suffix>,<names...> )`
	 * 名称：加后缀函数——addsuffix。
	 * 功能：把后缀`<suffix>`加到`<names>`中的每个单词后面。
	 * 返回：返回加过后缀的文件名序列。
	 * 示例：`$(addsuffix .c,foo bar)`返回值是“foo.c bar.c”。
* `$(addprefix <prefix>,<names...> )`
	 * 名称：加前缀函数——addprefix。
	 * 功能：把前缀`<prefix>`加到`<names>`中的每个单词前面。
	 * 返回：返回加过前缀的文件名序列。
	 * 示例：`$(addprefix src/,foo bar)`返回值是“src/foo src/bar”。
 * `$(join <list1>,<list2> )`
	 * 名称：连接函数——join。
	 * 功能：把`<list2>`中的单词**对应**地加到`<list1>`的单词后面。如果`<list1>`的单词个数要比`<list2>`的多，那么，`<list1>`中的多出来的单词将保持原样。如果`<list2>`的单词个数要比<list1>多，那么，`<list2>`多出来的单词将被复制到`<list2>`中。
	 * 返回：返回连接过后的字符串。
	 * 示例：`$(join aaa bbb , 111 222 333)`返回值是“aaa111 bbb222 333”。
* `$(wildcard <pattern>)`
	 * 名称：当前目录文件搜索函数——wildcard。
	 * 功能：使用通配符搜索当前目录下匹配`<pattern>`的文件，文件名以空格间隔。
	 * 返回：返回匹配`<pattern>`的文件名。
	 * 示例：假设当前目录下有`main.c add.c `,`$(wildcard m*.c)`返回值是“main.c”。
* `$(realpath <name>)`
	 * 名称：真实路径函数——realpath。
	 * 功能：计算`<name>`的真实路径（绝对路径且跟随符号链接），以空格间隔。`<name>`支持通配符。
	 * 返回：返回`<name>`的真实路径，以空格间隔。
	 * 示例：假设用户主目录下有`main.c`链接到该目录下的`test.c` , `$(realpath main.c)`返回值是“/home/usrname/test.c”。
* `$(abspath <name>)`
	 * 名称：绝对路径函数——abspath。
	 * 功能：计算`<name>`的绝对路径（不跟随符号链接），以空格间隔。`<name>`支持通配符。
	 * 返回：返回`<name>`的绝对路径，以空格间隔。
	 * 示例：假设用户主目录下有`main.c`链接到该目录下的`test.c` , `$(realpath main.c)`返回值是“/home/usrname/main.c”。
 

## 7.4 条件函数

* `$(if <condition>,<then-part>[,<else-part>])`
	 * 名称：如果函数——if。
	 * 功能：if函数可以包含“else-part”部分，或是不含。`<condition>`参数是if的表达式，如果其返回的为非空字符串，那么这个表达式就相当于返回真，于是，`<then-part>`会被计算，否则`<else-part>` 会被计算。
	 * 返回：如果`<condition>`为真（非空字符串），那个`<then- part>`会是整个函数的返回值，如果`<condition>`为假（空字符串），那么`<else-part>`会是整个函数的返回值，此时如果`<else-part>`没有被定义，那么，整个函数返回空字串。
	 * 示例：`$(if a,b,c)`返回"b"。
* `$(or condition1[,condition2[,condition3…]])`
	 * 名称：或函数——or。
	 * 功能：对每个进行逻辑或操作，可短路。
	 * 返回：如果某个`<condition>`为非空则返回该`<condition>`，否则返回空字符串
	 * 示例：`$(or a,b,c)`返回"a"。
* `$(and condition1[,condition2[,condition3…]])`
	 * 名称：且函数——and。
	 * 功能：对每个进行逻辑且操作，可短路。
	 * 返回：如果某个`<condition>`为空则返回空字符串，否则返回最后一个`<condition>`。
	 * 示例：`$(and a,b,c)`返回"c"。

## 7.5 foreach函数

* `$(foreach <var>,<list>,<text> )`
	 * 名称：遍历函数——foreach。
	 * 功能：把参数`<list>`中的单词逐一取出放到参数`<var>`所指定的变量中，然后再执行`<text>`所包含的表达式。每一次`<text>`会返回一个字符串，循环过程中，`<text>`的所返回的每个字符串会以空格分隔，最后当整个循环结束时，`<text>`所返回的每个字符串所组成的整个字符串（以空格分隔）将会是foreach函数的返回值。所以，`<var>`最好是一个变量名，`<list>`可以是一个表达式，而`<text>`中一般会使用`<var>`这个参数来依次枚举`<list>`中的单词。
	 * 返回：`<text>`每次返回的值得组合，以空格隔开。
	 * 示例：` $(foreach n,a b c d,$(n).o)`返回“a.o b.o c.o d.o”。
	 * 备注：foreach中的<var>参数是一个临时的局部变量，foreach函数执行完后，参数<var>的变量将不在作用，其作用域只在foreach函数当中。

## 7.6 file函数
* `$(file op filename[,text])`
	* 名称：文件读写函数——file。
	* 功能：进行文件读写。
		* op为<，读文件filename，text应省略；
		* op为>，将text写入到filename，且filename中的内容将清除
		* op为>>，将text追加到filename。
	* 返回：读取时，返回文件内容，如果filename不存在，返回空字符串；写入时返回空字符串。
	* 示例：` $(file >>,main.c,int main(void))`返回空字符串。
 




## 7.7 call函数

* `$(call <expression>,<parm1>,<parm2>,<parm3>...)`
	 * 名称：参数化函数——call。
	 * 功能：用来创建新的参数化的函数。当 make执行这个函数时，`<expression>`参数中的变量，`$(1)，$(2)，$(3)`等，会被参数`<parm1>`，`<parm2>`，`<parm3>`依次取代。
	 * 返回：`<expression>`被`<parm>`取代后的值。expression可以是函数或变量或多行变量。
	 * 示例：`ret=$(1) $(2);$(call ret,love,you)`返回"love you"。

## 7.8 取值函数
* `$(value varname)`
	 * 名称：取值函数——value。
	 * 功能：获取varname的值。`$(value varname)`等价于`$(varname)`。
	 * 返回：varname的值。
	 * 示例：`$(value CXX)`返回"g++"。

## 7.9 eval函数
* `$(eval string)`
	 * 名称：——eval。
	 * 功能:将string作为Makefile语法进行解析并执行。“eval”函数执行时会对它的参数进行两次展开。第一次展开过程发是由函数本身完成的，第二次是函数展开后的结果被作为Makefile内容时由make解析时展开的。明确这一过程对于使用“eval”函数非常重要。理解了函数“eval”二次展开的过程后。实际使用时，如果在函数的展开结果中存在引用（格式为：$(x)），那么在函数的参数中应该使用“$$”来代替“$”。
	 * 返回：空，也可以说没有返回值。
	 * 示例：`$(eval main:$$(OBJ);gcc -o main $$(OBJ))`。



## 7.10 origin函数

* `$(origin <variable> )`
	 * 名称：原始函数——origin。
	 * 功能：获取变量的来源。
	 * 返回：
		 * “undefined”：未定义变量。
		 * “default”：预定义变量。
		 * “environment”：仅指系统环境变量。这与前面说的环境变量有所不同。
		 * "environment override"，被系统环境变量覆盖的同名变量
		 * “file”,定义在Makefile中的变量。
		 * “command line”,变量是被命令行定义的。
		 * “override”,变量被override定义的。
		 * “automatic”,自动化变量。
	* 示例：`$(origin CXX)`返回。
	* 备注：`<variable>`是变量的名字，不应该是引用。

## 7.11 falvor函数
* `$(flavor <variable> )`
	* 名称：类型函数——flavor。
	* 功能:获取变量的类型。
	* 返回：
		* “undefined”：未定义变量。
		* “recursive”：递归展开的变量。
		* “simple”：简单变量。
    * 示例：`$(flavor CXX)`返回“recursive”。
	* 备注：`<variable>`是变量的名字，不应该是引用。
 

## 7.12 make控制函数
make提供了一些函数来控制make的运行。通常，你需要检测一些运行Makefile时的运行时信息，并且根据这些信息来决定，你是让make继续执行，还是停止。

* `$(error <text ...> )`
	 * 名称：错误函数——error。
	 * 功能：产生一个错误
* `$(warning <text ...> )`
	 * 名称：警告函数——warning。
	 * 功能：这个函数很像error函数，只是它并不会让make退出，只是输出一段警告信息，而make继续执行。
* `$(info <text ...>)`
	 * 名称：通知函数——info。
	 * 功能：这个函数很像error函数，只是它并不会让make退出，只是输出一段通知信息，而make继续执行。

## 7.13 shell函数

* `$(shell shell_cmd)`
	 * 名称：shell函数——shell。
	 * 功能: 生成一个shell来执行shell_cmd，等效于命令替换。
	 * 返回：shell_cmd的输出
	 * 示例：`$(shell date)`返回“018年 05月 09日 星期三 23:01:42 CST"

## 7.14 guile函数
* `$(guile guile_expr)`
	 * 名称：guile函数——guile。
	 * 功能: 执行GNU guile表达式。
	 * 返回：执行结果
	 * 备注：需要make支持才行  


# 8 隐式规则
* 隐式规则即不需要显示指出的规则。
* 隐含规则链
	* 有些时候，一个目标可能被一系列的隐含规则所作用。例如，一个[.o]的文件生成，可能会是先被Yacc的[.y]文件先成[.c]，然后再被C的编译器生成。我们把这一系列的隐含规则叫做“隐含规则链”。
	* 在默认情况下，对于中间目标，它和一般的目标有两个地方所不同：第一个不同是除非中间的目标不存在，才会引发中间规则。第二个不同的是，只要目标成功产生，那么，产生最终目标过程中，所产生的中间目标文件会被以“rm -f”删除。
	* 通常，一个被makefile指定成目标或是依赖目标的文件不能被当作中介。然而，你可以明显地说明一个文件或是目标是中介目标，你可以使用特殊目标“.INTERMEDIATE”来强制声明。（如：.INTERMEDIATE ： mid ）

	* 你也可以阻止make自动删除中间目标，要做到这一点，你可以使用特殊目标“.SECONDARY”来强制声明（如：.SECONDARY:sec）。你还可以把你的目标，以模式的方式来指定（如：%.o）成特殊目标“.PRECIOUS”的依赖目标，以保存被隐含规则所生成的中间文件。

	* 在“隐含规则链”中，禁止同一个目标出现两次或两次以上，这样一来，就可防止在make自动推导时出现无限递归的情况。

	* Make会优化一些特殊的隐含规则，而不生成中间文件。如，从文件“foo.c”生成目标程序“foo”，按道理，make会编译生成中间文件“foo.o”，然后链接成“foo”，但在实际情况下，这一动作可以被一条“cc”的命令完成（cc –o foo foo.c），于是优化过的规则就不会生成中间文件。



* 编译C程序的隐含规则：
	 * name.o自动依赖name.c
	 * 生成命令：`$(CC) –c $(CPPFLAGS) $(CFLAGS)`
* 编译C++程序的隐含规则:
	 * name.o目标自动依赖name.cc、name.cpp、name.C
	 * 生成命令：`$(CXX) –c $(CPPFLAGS) $(CXXFLAGS)`。
	 * 建议使用.cc作为C++源文件的后缀，而不是.C
* 编译Pascal程序的隐含规则:
	 * name.o目标自动依赖name.p 
	 * 生成命令：`$(PC) $(PFLAGS) -c`
* 编译Fortran/Ratfor程序的隐含规则:
	* name.o目标自动依赖name.r、name.F、name.f
	* 生成命令
		 * “.f”:  `$(FC) –c  $(FFLAGS)`
		 * “.F”: `$(FC) –c  $(FFLAGS) $(CPPFLAGS)`
		 * “.f”: `$(FC) –c  $(FFLAGS) $(RFLAGS)`
* 预处理Fortran/Ratfor程序的隐含规则：
	 * name.f目标自动依赖name.r 、name.F
	 * 生成命令  
		 * ‘.F’：`$(FC) $(CPPFLAGS) $(FFLAGS) -F`
		 * ‘.r’：`$(FC) $(FFLAGS) $(RFLAGS) -F`

* 编译Modula-2程序的隐含规则：
	 * name.sym目标自动依赖name.def，
	 * 生成命令：`$(M2C) $(M2FLAGS) $(DEFFLAGS)`。
	 * name.o目标自动依赖name.mod
	 * 生成命令是`$(M2C) $(M2FLAGS) $(MODFLAGS)`

* 汇编和汇编预处理的隐含规则:
	 * name.o目标自动依赖name.s，默认使用编译器“as”
	 * 生成命令：`$(AS) $(ASFLAGS)`。
	 * name.s目标自动依赖name.S，默认使用C预编译器“cpp”
	 * 生成命令：`$(CPP) $(CPPFLAGS)`。

* 链接Object文件的隐含规则:
	 * name目标自动依赖name.o，通过运行C的编译器来运行链接程序（一般是“ld”）来生成目标
	 * 生成命令：`$(CC) $(LDFLAGS) <n>.o $(LOADLIBES) $(LDLIBS)`。
* Yacc C程序时的隐含规则:
	 * name.c目标自动依赖name.y（yacc生成的文件）
	 * 生成命令：`$(YACC) $(YFALGS)`
* Lex C程序时的隐含规则:
	 * name.c目标自动依赖name.l（Lex生成的文件）
	 * 生成命令：`$(LEX) $(LFALGS)`

* Lex Ratfor程序时的隐含规则:
	 * name.r目标自动依赖name.l（Lex生成的文件）
	 * 生成命令：`$(LEX) $(LFALGS)`
* 从C程序、Yacc文件或Lex文件创建Lint库的隐含规则:
	 * name.ln（lint生成的文件）目标自动依赖name.c
	 * 生成命令是：`$(LINT) $(LINTFALGS) $(CPPFLAGS) -i`。
	 * 对于“<n>.y”和“<n>.l”也是同样的规则。
* TeX and Web
	 * name.dvi目标自动依赖name.tex
	 * 生成命令：`$(TEX)`
	 * name.tex目标自动依赖name.web、name.ch
	 * 生成命令：
		 * '.web': `$(WEAVE)`
		 * '.ch': `$(CWEAVE)`
	 * name.p目标自动依赖name.web
	 * 生成命令：`$(TANGL)`
	 * name.c目标自动依赖name.w、name.ch
	 * 生成命令：`$(CTANGL)`
* Texinfo and Info
	 * name.dvi目标自动依赖name.texinfo、name.texi、name.txinfo
	 * 生成命令：`$(TEXI2DVI) $(TEXI2DVI_FLAGS)`
	 * name.info目标自动依赖name.texinfo, name.texi, name.txinfo
	 * 生成命令：`$(MAKEINFO) $(MAKEINFO_FLAGS)`



# 9 模式规则
模式规则：在规则的目标定义中包含"%"的规则。
 
* 目标中的"%"定义表示对文件名的匹配，"%"表示长度任意的非空字符串。
* 如果"%"定义在目标中，那么，目标中的"%"的值决定了先决条件中的"%"的值
* "%"的展开发生在变量和函数的展开之后，变量和函数的展开发生在make载入Makefile时，而模式规则中的"%"则发生在运行时。

# 10 后缀规则

* 后缀规则是一个比较老式的定义隐含规则的方法。
	 * 双后缀规则定义了一对后缀：先决条件后缀和目标后缀。如".c.o"相当于"%o : %c"。
	 * 单后缀规则只定义一个后缀，也就是源文件的后缀。如".c"相当于"% : %.c"。
* 后缀规则中所定义的后缀应该是make所认识的，如果不认识可以使用特殊目标".SUFFIXES"来定义或是删除
* 后缀规则不允许任何的依赖文件，如果有依赖文件的话，那就不是后缀规则，那些后缀统统被认为是文件名。
    

# 11 重载隐式规则
* 使用模式规则来重载隐式规则。如：
```makefile
%.o : %.c
            $(CC) -c $(CPPFLAGS) $(CFLAGS) -D$(date)
```
* 使用后缀规则来重载隐式规则。如：
```makefile
.SUFFIXES:.c.o     #自定义后缀
.c.o:
    $(CC) -c $(CPPFLAGS) $(CFLAGS) -D$(date)

```
* 取消内建的隐含规则，只要不在后面写命令就行。如：
```makefile
%.o : %.s
   
   # 或

.c.o:
```

# 12 更新库文件
```makefile
libname1.a(name2.o name3.o):name.o
    command
```
* \$@为libname1.a
* \$%为name2.o
* 使用ar打包静态库文件时要用"-s"选项或者打包完成后使用`ranlib archivefile`更新文件符号表。

# 13 makefile约定
* 每个makefile应该包括`SHELL=/bin/sh`
* makefile的命令应该运行在sh中
* 为了构建和安装程序的configure脚本和Makefile规则 不应直接使用除以下程序：
```makfile
awk cat cmp cp diff echo egrep expr false grep install-info ln ls
mkdir mv printf pwd rm rmdir sed sleep sort tar test touch tr true`
```
* 压缩程序（如gzip）可用于dist规则。
* 尽可能使用通用的选项
* 不要在makefile中创建符号链接
* 编译程序及选项不应该直接使用，而应该通过预定义变量
* 分阶段安装
	* 在实际安装路径前使用变量 DESTDIR，这样可以安装到其它目录
	* 不要在makefile文件中设置DESTDIR的值，而应由用户在make命令行用绝对路径指定
	* DESTDIR应该只在install和uninstall目标中有效
	* 安装目录变量
		* 安装根目录
			* prefix：数据文件根目录，默认为/usr/local/
			* exec_prefix：可执行文件根目录，默认为/usr/local/
        * 可执行文件目录
            * bindir：安装普通用户可执行程序，默认为` $(exec_prefix)/bin`
            * sbindir：安装非正常状态时可执行程序，默认为` $(exec_prefix)/sbin`
            * libexecdir：安装由程序而不是用户调用的可执行文件，默认为`$(exec_prefix)/libexec`
       * 数据文件目录
			* datarootdir：安装与架构无关的只读数据，默认为`$(prefix)/share`
			* datadir：安装与架构无关的只读数据，默认为`$(datarootdir)`
			* sysconfdir：安装只读配置文件，该目录的文件应为文本文件，默认为`$(prefix)/etc`
			* sharedstatedir：安装与架构无关且运行时可修改数据，默认为`$(prefix)/com`
			* localstatedir：安装与特定机器有关的运行时可修改数据，`$(prefix)/var`
			* runstatedir:安装与特定机器有关且不需要持久保存的运行时可修改数据，默认为`$(localstatedir)/run`
			* includedir：安装GCC的C头文件，默认为`$(prefix)/include`
		    * oldincludedir：安装非GCC的头文件，默认为`/usr/include`
			* docdir：安装文档，默认为`$(datarootdir)/doc/yourpkg`
			* infodir：安装Info文件，默认为`$(datarootdir)/info`
			* htmldir、dvidir、pdfdir、psdir：安装特殊格式的文档，默认为`$(docdir)`
			* libdir：安装库文件，默认为`$(exec_prefix)/lib`
			* lispdir：安装Emacs Lisp 文件，默认为`$(datarootdir)/emacs/site-lisp`
		* man文档目录
			* mandir：man手册顶层目录， 默认为`$(datarootdir)/man`
			* man1dir： 默认为`$(mandir)/man1`
			* man2dir： 默认为`$(mandir)/man2`
			* manext：man文件名扩展，默认为.1
			* man1ext：man 1文件名扩展
			* man2ext：man 2文件名扩展
		 * 源文件目录：srcdir
* 标准目标
	 * all：编译所有目标
	 * install：安装程序
	 * install-html、install-dvi、install-pdf、install-ps：以指定格式安装文档
	 * uninstall：卸载程序
	 * install-strip：安装程序，但是跳过可执行文件
	 * clean：清除构建程序是创建的文件
	 * distclean：清除配置或构建程序创建的文件
	 * mostlyclean：类似clean,但是保留一些不想删除的文件 
	 * maintainer-clean：删除所有可以用该Makefile构建的文件
	 * TAGS：更新标签列表
	 * info：生成需要的info文件
	 * dvi、html、pdf、ps：生成指定格式的文档
	 * dist：创建程序的发行版本
	 * check：执行测试
	 * installcheck：执行安装测试
	 * installdirs：添加名为installdirs的目标来创建安装目录
* 安装命令分类
	 * 普通命令：移动文件到合适位置，并设置mode。
	 * 安装前命令
	 * 安装后命令

# 14 示例
## 14.1 普通makefile文件         
```makefile
test:main.o add.o sub.o mul.o div.o
    gcc $^  -o $@    
%.o:%.c
    gcc $^  -o $@  
.PHONY:clean
clean:
        rm -f *.o test
```
 
## 14.2 另一种makefile文件  

* 目录分类：
	 1. 功能目录，有几个创建几个，之下建立src,src之中放.c文件，在每个功能目录下都建一个makefile
	 2. include目录放头文件
	 3. scripts目录放脚本文件  在它之下也要放makefile 
	 5. bin 目录放可执行文件
	 6. lib 目录放库文件
	 7. doc 目录放文档信息   readme  使用说明
	 8. 顶层目录下放一个总控makefile文件


* 总控makefile文件
```makefile
include scripts/Makefile

modules_make = $(MAKE) -C $(1);
modules_clean = $(MAKE) clean -C $(1);

.PHONY: all mm mc clean

all: $(Target)
mm:
    @$(foreach n,$(Modules),$(call modules_make,$(n)))
mc:
    @$(foreach n,$(Modules),$(call modules_clean,$(n)))

$(Target) : mm
    $(CC) $(CFLAGS) -o $(Target) $(AllObjs) $(Libs)
    @echo $(Target) make done!

clean : mc
    rm -rf $(Target)
    @echo clean done!                                            '
```    
* scripts下的makefile文件      
```makefile
CC := gcc
CFLAGS := -Wall -O3
Libs = -lpthread
Target := client
Source := $(wildcard src/*.c)
Objs := $(patsubst %.c,%.o,$(Source))
Modules += check_putin pack_message main
AllObjs := $(addsuffix /src/*.o,$(Modules))
```

* 使用时，主控makefile复制到自己的工程目录下，根据自己的工程目 录修改scripts下的makefile文件中的`Modules += check_putin pack_message main`这一行中的变量名。并复制到相应位置，子目录下的makefile也通过复制完成。






------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>**陈皓**先生的[《跟我一起写makefile》](https://blog.csdn.net/haoel/article/details/2886)，并根据最新的[《GNU make手册》](http://www.gnu.org/software/make/manual/make.html)（截止2018年5月），以及[《Linux man pages》](https://linux.die.net/man/ "点击跳转")做了修改，增添了一部分内容。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
