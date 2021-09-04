---
title: gcc与g++教程
tags: Linux项目管理
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《Linux man pages》](https://linux.die.net/man/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

<style>table{word-break:initial;}</style>


# 1 简介

* 对于 C 文件，可以采用 gcc 或 g++编译
* 对于C++文件，应该采用 g\+\+进行编译。 g\+\+是一个调用GCC并自动指定与C ++库链接的程序。除非使用-x选项，否则它会将.c，.h和.i文件视为C\+\+文件而不是C文件。

# 2 编译过程
所用工具：预处理器—>编译器 —>汇编器 as—>连接器 
编译过程：预编译—>编译-->汇编-->链接
* 预处理：预处理器将对源文件中的宏和头文件进行展开。
* 编译：检查代码规范性、语法错误等，在检查无误后把代码翻译成汇编语言
*  汇编：将汇编文件编译成机器码。
* 链接：将目标文件和外部符号进行连接，得到一个可执行二进制文件。



下面以一个很简单的 test.c 来探讨这个过程。
```c
#include <stdio.h>
#defineNUMBER (1+2)
int main()
{
	int x =NUMBER;
	return 0;
}
```
* 预处理：`gcc –E test.c -o test.i `，我们用 `cat` 查看 `test.i` 的内容如下：

```c
...头文件展开部分省略...
int main() 
{
    int x=(1+2); 
    return 0;
}
```
可以看到，文件中宏定义`NUMBER`出现的位置被(1+2)替换掉了，头文件也被展开，其它的内容保持不变。引用与`typedef`区别就在于typedef只是为已有类型取一个别名，使用别名和原名效果一样，而引用是变量的别名。

* 编译：`gcc –S test.i –o test.s ` ，通过 `cat test.s `查看 `test.s` 的内容为汇编代码。
* 汇编：`as test.s -o test.o`， 利用as将汇编文件编译成机器码。得到输出文件为 test.o。test.o 中为目标机器上的二进制文件. 用 `nm `查看文件中的符号：` nm test.o` 输出如下：`00000000 T main`。有的编译器上会显示：`00000000 b .bss  00000000 d.data 00000000 t .text U___main U__alloca 00000000 T _main `。既然已经是二进制目标文件了，能不能执行呢？试一下`./test.o`，提示`can not execute binary file`。原来`___main `前面的 `U` 表示这个符号的地址还没有定下来，`T `表示这个符号属于代码。
* 链接：`gcc test.o –o test.exe` ，将所有的.o 文件链接起来生产可执行程序。

# 3 所支持文件类型
|后缀	|含义|	后缀|	含义|
|:--|:--|:--|:--|
|.c	|C源文件|.i|经过预处理的C文件|
|.cc/.cp/.cxx/.cpp/.CPP/.c++/.C|C++源文件|.ii	|经过预处理的C++文件|
|.m	|obje-C源文件|.mi|经过预处理的obj-C文件|
|.mm/.M	|obj-C++源文件|.mii|经过预处理的obj-C++文件|
|.F/.FOR/.fpp/.FPP/.FTN|固定的Fortran源文件|.f/.for/.ftn|固定的经过预处理的Fortran文件
|.F90/.F95/.F03/.F08|自由的Fortran源文件|.f90/.f95/.f03/.f08|自由的经过预处理的Fortran文件|
|.go|Go 源文件|||
|.ads|Ada的规范源文件|.adb| Ada的主体源文件。
|.S/.sx|汇编源文件|.s|经过预处理的汇编文件|
|.o|未链接的二进制文件|.exe/|经过链接的二进制文件
| .hh/.H/.hp/.hxx/.hpp/.HPP/.h++/.tcc|C++头文件|.h|C/C++/Obj-C/Obj-C++头文件|

           
# 4 编译选项

|选项|含义|输入|输出|
|:--|:--|:--|:--|
|-E |只预处理，缺省输出到屏幕|源文件|经过预处理的文件|
|-S |只编译不汇编，缺省输出同名.s文件|源文件或经过预处理的文件|经过预处理的汇编文件|
|-c |只编译汇编不链接，缺省输出同名.o文件|源文件、经过预处理的文件 或汇编文件|未链接的二进制文件
|&emsp;&emsp;|

|选项|	含义|
|--|--|
|@file|编译选项由文件file读入
|-o file|指定输出文件|
|-v|打印出编译器内部编译各过程的命令行信息和编译器的版本
|-I dir|在头文件的搜索列表中添加dir 目录
|-D macro|传入宏定义
|-x language|指定源文件语言，如c，c++|
|-std=standard|指定标准，如c99，c++11|
|-M|生成依赖关系，包括库文件
|-MM|生成依赖关系，不包括库文件


```c
//示例：project1.c

#include <stdio.h>
#define MAX100
#define max(a,b) ((a)>(b)?(a):(b)) //宏定义，执行-E 之后被替换
int main()
{
		printf("MAX=%d\n",MAX);
		printf("max(3,4)=%d\n",max(3,4));
		return 0;
}

#法一：（理解原理用，四步:.c-->.i-->.s->.o->.exe）
gcc –E project1.c -o project1.i  //预编译，生成已预处理过的 *.i文件
gcc –S project1.i -o project1.s   //编译，生成*.s文件
as project1.s -o project1.o       //汇编，生成未链接的.o二进制文件
gcc project1.o -o project1.exe    //链接，生成可执行程序

#法二：（理解原理用，三步: .c-->.i-->.s-->.exe）
gcc –E project1.c -o project1.i  //预编译，生成已已预处理过的 *.i文件
gcc –S project1.i -o project1.s   //编译，生成汇编语言原始程序*.s
gcc project1.s  -o project1.exe  //汇编并链接，生成可执行程序

#法三：（常用,两步.c-->.o -->.exe）
gcc –c project1.c -o project1.o   //预处理，编译，汇编
gcc project1.o -o project1.exe    //链接

#法四：（常用，一步：.c-->.exe）
gcc project1.c -o project1.exe     //编译并链接

#法五：（常用，一步：.c-->.exe）
gcc project1.c                     //缺省生成a.out可执行文件
```

```c
//示列: gcc传递宏定义选项 –D 的作用，就是定义宏的另一种方式
#include <stdio.h>
main()
{
#ifdef cjy   //表示如果定义了 cjy，即命令行参数传了cjy，就执行下面的输出
	printf("cjy is defined!\n");
#else
	printf("cjy is not defined!\n");
#endif
	printf("main exit\n");
}

gcc –E project2.c –o project2.i –D cjy //条件编译，用-D 传递，如果没有传cjy则执行#else
gcc–S project2.i –o project2.s
gcc–o project2.exe project2.s

或：

gcc project2.c –o project2  –D cjy//条件编译，用-D 传递，如果没有传cjy则执行#else
```
# 5 gcc 库选项
|选项|	含义|
|:--|:--|
|-static|	链接静态库，禁止使用动态库.用于最后链接阶段|
|-shared|	1.可以生成动态库文件 <br />2.进行动态编译，尽可能地链接动态库，只有没有动态库时才会连接同名静态库（默认选项，可省略）
|-L dir	|在库文件的搜索路径列表中添加dir目录
|-lname	|链接称为libname.a（静态库）或者 libname.so（动态库）的库文件若两个库都存在则根据编译方式时-static还是-shared来进行链接
|-fPIC或-fpic|	生成使用相对地址的与位置无关的目标代码。通常使用gcc的-shared选项从该PIC目标文件生成动态库文件

## 5.1 -fPIC 功能
&emsp;&emsp;-fPIC 作用于编译阶段，告诉编译器产生与位置无关代码(Position-Independent Code)，则产生的代码中，没有绝对地址，全部使用相对地址，故而代码可以被加载器加载到内存的任意位置，都可以正确的执行。这正是共享库所要求的，共享库被加载时，在内存的位置不是固定的。
&emsp;&emsp;`gcc -shared -fPIC -o 1.so 1.c`。这里有一个-fPIC参数，PIC就是position independent code。PIC使.so文件的代码段变为真正意义上的共享，如果不加-fPIC,则加载.so文件的代码段时,代码段引用的数据对象需要重定位, 重定位会修改代码段的内容,这就造成每个使用这个.so文件代码段的进程在内核里都会生成这个.so文件代码段的副本.每个副本都不一样,取决于这个.so文件代码段和数据段内存映射的位置.不加fPIC编译出来的so,是要再加载时根据加载到的位置再次重定位的.(因为它里面的代码并不是位置无关代码)，如果被多个应用程序共同使用,那么它们必须每个程序维护一份so的代码副本了.(因为so被每个程序加载的位置都不同,显然这些重定位后的代码也不同,当然不能共享)，我们总是用fPIC来生成动态库文件,从来不用fPIC来生成静态库文件.fPIC与动态链接可以说基本没有关系,libc.so一样可以不用fPIC编译,只是这样的so必须要在加载到用户程序的地址空间时重定向所有表目.因此,不用fPIC编译so并不总是不好.如果你满足以下4个需求/条件:
&emsp;&emsp;（1）该库可能需要经常更新
&emsp;&emsp;（2）该库需要非常高的效率(尤其是有很多全局量的使用时)
&emsp;&emsp;（3）该库并不很大.
&emsp;&emsp;（4）该库基本不需要被多个应用程序共享
就可以不适用-fpic来编译so文件。如果用没有加这个参数的编译后的共享库，也可以使用的话，可能是两个原因：
&emsp;&emsp;（1）gcc默认开启-fPIC选项
&emsp;&emsp;（2）loader使你的代码位置无关
从GCC来看，shared应该是包含fPIC选项的，但似乎不是所以系统都支持，所以最好显式加上fPIC选项。

## 5.2 静态库和动态库
&emsp;&emsp;静态库是目标文件.a的归档文件（格式为libname.a）。如果在编译某个程序时链接静态库，则链接器将会搜索静态库并直接拷贝到该程序的可执行二进制文件到当前文件中；
&emsp;&emsp;动态库（格式为libname.so[.主版本号.次版本号.发行号]）在程序编译时并不会被链接到目标代码中，而是在程序运行时才被加载器载入。跳转到动态库所在的地址。


### 5.2.1  创建静态库
```shell	
$ gcc -c add.c #编译 add.c 源文件默认生成 add.o 目标文件

$ ar rcsv libadd.a add.o #对目标文件*.o 进行归档，生成 lib*.a,此处 lib 要写

$ gcc -o main main.c –L ./ –ladd –I ./  #不要忘记-L 后面的那个. （即在库文件的搜索	路径中添加当前路径， -ladd 表示链接库文件libadd.a/.so， -I./表示包含在当前目录中的头文件）

$./main  #因为是静态编译，生成的执行文件可以独立于.a文件运行。
```

**命令**：ar rcsv  libxxx.a xx1.o
**描述**：建立、删除、管理静态库文件。
|常用参数|意义|
|--|--|
|r|在库中插入模块(替换)。当插入的模块名已经在库中存在，则替换同名的模块。如果若干模块中有一个模块在库中不存在，ar显示一个错误消息，并不替换其他同名模块。默认的情况下，新的成员增加在库的结尾处，可以使用其他任选项来改变增加的位置。
|c|创建一个库。不管库是否存在，都将创建。
|s|创建目标文件索引，这在创建较大的库时能加快时间。（补充：如果不需要创建索引，可改成大写S参数；如果.a文件缺少索引，可以使用ranlib命令添加）
|v|显示生成结果
|t|显示库目录|
|&emsp;&emsp;&emsp;&emsp;|



**命令**：nm -s libxxx.a
**描述**：显示库文件中的索引表。

**命令**：ranlib libxxx.a
**描述**：为库文件创建索引表。

### 5.2.2 创建动态库
```shell
#分三步：
$ gcc -fPIC -Wall -c add.c      #编译 add.c 源文件生成 add.o 目标文件
$ gcc -shared -o libadd.so add.o  #对目标文件*.o 进行归档，生成 lib*.so,此lib 要写
$ gcc -o main main.c –L . –ladd –I . #.和./等效

#分两步：
gcc -fPIC  -Wall  -shared  add.c  -o  libadd.so  #生成动态库文件
gcc -o main main.c –L ./ –ladd                   #编译时链接动态库
```
&emsp;&emsp;创建动态链接库之后，以后就可以使用该动态链接库了。例如在 test.c 里面调用了原来库中的函数，则编译时执行 `gcc–o test test.c –L ./ –ladd `就可以了。
&emsp;&emsp;注意：在运行 main 前，需要注册动态库的路径。方法有 3 种：
&emsp;&emsp;（1）修改/etc/ld.so.conf 
&emsp;&emsp;（2）修改LD_LIBRARY_PATH 环境变量,LD_LIBRARY_PATH=. ./main
&emsp;&emsp;（3）将库文件拷贝到/lib 或者/usr/lib 下（系统默认搜索库路径）。
```shell
$cp libadd.so /lib  #通常采用的方法， cp lib*.so /lib或者cp libadd.so  /usr/lib
$./main     #运行main
```
&emsp;&emsp;如果不拷贝，生成.so之后还有种方法：采用dlopen,dlclose,dlerror,dlsym加载动态链接库。
&emsp;&emsp;为了使程序方便扩展，具备通用性，可以采用插件形式。采用异步事件驱动模型，保证主程序逻辑不变，将各个业务已动态链接库的形式加载进来，这就是所谓的插件。linux提供了加载和处理动态链接库的系统调用，非常方便。
```c
#include <dlfcn.h>
void *dlopen(const char *filename, int flag);
char *dlerror(void);
void *dlsym(void *handle, const char *symbol);
int dlclose(void *handle);
```
* dlopen以指定模式打开指定的动态连接库文件，并返回一个句柄（指针）给调用进程，打开模式如下：
 * RTLD_LAZY  暂缓决定，等有需要时再解出符号 
 * RTLD_NOW  立即决定，返回前解除所有未决定的符号。
* dlerror返回出现的错误，
* dlsym通过句柄和连接符名称获取函数名或者变量名，
* dlclose卸载打开的库。
```c
//例：将如下程序编译为动态链接库libcaculate.so，程序如下：

int add(int a,int b)
{
    return (a + b);
}
int sub(int a, int b)
{
    return (a - b);
}

int mul(int a, int b)
{
    return (a * b);
}

int div(int a, int b)
{
    return (a / b);
}

//编译如下： gcc -fPIC -shared caculate.c -o libcaculate.so
//采用上面生成的libcaculate.so，写个测试程序如下：

#include <stdio.h>
#include <stdlib.h>
#include <dlfcn.h>

//动态链接库路径
 #define LIB_CACULATE_PATH "./libcaculate.so"
 
//函数指针
typedef int (*CAC_FUNC)(int, int);
 
int main()
{
    void *handle;
    char *error;
    CAC_FUNC cac_func = NULL;

   //打开动态链接库
    handle = dlopen(LIB_CACULATE_PATH, RTLD_LAZY);
    if (!handle) 
    {
        fprintf(stderr, "%s\n", dlerror());
        exit(EXIT_FAILURE);
    }

    //清除之前存在的错误
    dlerror();

    //获取一个函数
    *(void **) (&cac_func) = dlsym(handle, "add");
    if ((error = dlerror()) != NULL)  
    {
        fprintf(stderr, "%s\n", error);
        exit(EXIT_FAILURE);
    }
    printf("add: %d\n", (*cac_func)(2,7));

    cac_func = (CAC_FUNC)dlsym(handle, "sub");
    printf("sub: %d\n", cac_func(9,2));

    cac_func = (CAC_FUNC)dlsym(handle, "mul");
    printf("mul: %d\n", cac_func(3,2));

    cac_func = (CAC_FUNC)dlsym(handle, "div");
    printf("div: %d\n", cac_func(8,2));

    //关闭动态链接库
    dlclose(handle);
    exit(EXIT_SUCCESS);
}

//编译选项如下：gcc -rdynamic -o main main.c –ldl
```
## 5.3 静态库与动态库的比较
&emsp;&emsp;动态库只在执行时才被链接使用，不是直接编译为可执行文件，并且一个动态库可以被多个程序使用故可称为共享库。
&emsp;&emsp;静态库将会整合到程序中，在程序执行时不用加载静态库。 因此，静态库会使你的程序臃肿并且难以升级，但比较容易部署。而动态库会使你的程序轻便易于升级但难以部署。


# 6 常用警告选项
|选项|含义|
|:--|:--|
|-ansi |生成标准语法（ANSI C 标准）所要求的警告信息（并不列出所有警告）
|-pedantic |列出 ANSI C 标准的全部警告信息。
|-Wall|列出所有的警告信息（常用）|
|–Werror|要求gcc将所有的警告都当成错误来处理

# 7 优化选项
&emsp;&emsp;gcc 对代码进行优化通过选项“-On”来控制优化级别（不是零时大写的o，n 是整数）。不同的优化级别对应不同的优化处理工作。如使用优化选项：

|常用选项|描述|
|:--|:--|
|-O1|主要进行线程跳转和延迟退栈两种优化。
|-O2|除了完成所有“-O1”级别的优化之外，还要进行一些额外调整工作，如处理其指令调度等。 
|-O3|则还包括循环展开或其他一些与处理器特性相关的优化工作。

&emsp;&emsp;虽然优化选项可以加速代码的运行速度，但对于调试而言将是一个很大的挑战。因为代码在经过优化之后，原先在源程序中声明和使用的变量很可能不再使用，控制流也可能会突然跳转到意外的地方，循环语句也有可能因为循环展开而变得到处都有，所有这些对调试来讲都是不好的。所以在调试的时候最好不要使用任何的优化选项，只有当程序在最终发行的时候才考虑对其进行优化。通常用的是-O2。

# 7 调试选项
|选项|描述|
|:--|:--|
|-g|在可执行程序中包含标准调试信息
|-g0|等于不加-g。即不包含任何信息
|-g1|只包含最小信息，一般来说只有你不需要debug，只需要backtrace信息，并且真的很在意程序大小，或者有其他保密/特殊需求时才会使用-g1。
|–g2|gdb默认等级，包含绝大多数你需要的信息。|
|–g3|包含一些额外信息，例如包含宏定义信息。当你需要调试宏定义时，请使用-g3|
|&emsp;&emsp;|




------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《Linux man pages》](https://linux.die.net/man/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
