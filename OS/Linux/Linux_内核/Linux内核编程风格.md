---
title: Linux内核编程风格
tags: Linux内核
---


------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《Documentation/process/coding-style.rst》](https://www.kernel.org/doc/html/latest/translations/zh_CN/coding-style.html)。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------


# 0 前言		

&emsp;&emsp;这是一个简短的文档，描述了linux内核的首选编码风格。 编码风格每个有所不同，我不会强迫每个人都接受我的观点。... ... 请至少考虑一下这里提出的观点。


# 1 缩进
* 水平制表符为8个字符
* switch语句中，switch与case或对齐
```cpp
switch (suffix) {
case 'G':
case 'g':
	mem <<= 30;
		break;
default:
	break;
}
```
* 不要在一行中写多个语句或多个赋值语句，除非需要隐藏什么。
```cpp

if (condition) do_this;
	do_something_everytime;
```
* 除了注释、文档、kconfig之外，不要使用空格进行缩进
* 不要在行尾添加空格

# 2 括号

* 对于非函数（如结构体、联合体、if、while、for、do、switch）中使用花括号，`{`不另起一行，`}`要另起一行；对于函数，`{`要另起一行，`}`另起一行
* 如果if、for、do 后面只有一行，不加花括号
* 如果if与else if 混用时，else不另起一行，如：

```cpp
if (condition)
	action();

if (condition)
	do_this();
else
	do_that();

if (condition) {
	do_this();
	do_that();
} else {
	otherwise();
}

```
# 3 空格 
* 大多数关键字后需要加一个空格，但` sizeof, typeof, alignof, and __attribute__ defined`不加
* 双目运算符，两侧各加一个空格，如`a + b`;
	* 算术运算符：` +、  -、   * 、 / 、 %`和` +=、  -=、   * =、 / =、 %=`
	* 位运算符：`  | 、 &、  ^ `和`  | =、 &=、  ^ =`
	* 比较运算符`：<、  >、 <= 、 >= 、 == 、 !=`
	* 移位运算符号：`<< 、>>`和`<<= 、>>=`
	* 关系运算符：`|| 、&& `
	*  条件运算符：`? : `
* 单目运算符,不加空格：`&（取地址）、  *（取内容） 、 +（正） 、 -（负）、 ~(按位取反) 、 !（非） 、 ++、 -- 、.（取成员）、->（取成员）[]（数组索引）`
* 逗号运算符，右侧加一个空格，左侧不加
* 定义指针时，类型名和指针符号`*`要加一个空格。如`char *ptr;`
* 中括号和小括号内侧不加空格



# 4 命名
* 标识符 应该以 **字母、数字、下划线**命名，并且不能以数字开头。在用户空间使用时，避免用下划线开头，因为大多时候是系统内置标识符。
* Linux不采用匈牙利命名法（也叫驼峰命名法、大小写混合命名法），而是尽量使用**小写+下划线**来命名。
* 对于全局变量和全局函数，需要具有描述性名称，其它标识符在保证意义的情况下可以适当缩写
* 不要在名称中添加变量或函数类型相关的信息。
* 对于宏，大多数时候全部大写，对于宏函数有时也会使用小写。
* 对于局部的循环变量，可使用单字符命名

# 5 类型定义

* 不要对结构体和指针使用typedef 来产生一个莫名其妙的新类型
* typedef一般用于下列情况
	*  完全不透明，使用typedef进行隐藏
	*  清楚的表达整数的类型
	*  在某些特殊情况下，与标准C99类型相同的新类型。
	*  在用户空间中安全使用。

# 6 函数
* 函数应该简短，只做一件事。（宽度为80 长度24）
	* 如果函数逻辑简单，那么也可以更长一些
	* 如果函数复杂，那么应该分拆函数。
* 函数的局部变量的数量。他们不应该超过5-10，否则，需要将其拆分成更小的部分。
* 在源文件中，使用一个空行分隔函数。如果要导出该函数，它的EXPORT 宏应该在关闭函数括号行之后立即执行。例如。：
```cpp
int system_is_up（void）
{
	return system_state == SYSTEM_RUNNING;
}
EXPORT_SYMBOL（system_is_up）;
```
* 在函数原型中，包含形参的参数名称及其数据类型。尽管C语言不需要这样做。

# 7 使用goto集中退出

* 编译器经常以无条件跳转指令的形式使用等效的goto语句。

* 当函数从多个位置退出并且必须完成一些常见工作（如清理）时，goto语句会派上用场。如果不需要，则直接返回。

* 标签名称应说明goto的作用或goto存在的原因。如果goto释放“buffer”，那么好的名称可以是“out_buffer：”。避免使用GW-BASIC名称，如“err1：”和“err2：”。也不要在goto位置之后命名它们，例如“err_kmalloc_failed：”

* 使用gotos的基本原理是：
	* 无条件陈述更容易理解和遵循
	*  减少嵌套
	*   防止在进行修改时不更新单个出口点的错误
	*   节省编译器工作以优化冗余代码;


		

# 8 注释
* 注释使用C89的`/* */`，而不是C99的`//`
* 注释只需说明做什么，不需说明怎么做
* 在注释内核API函数时，请使用**内核文档格式**，详见`Documentation/kernel-doc-nano-HOWTO.txt`和` scripts/kernel-doc`
* 长行注释风格：
```cpp
	/*
	 * This is the preferred style for multi-line
	 * comments in the Linux kernel source code.
	 * Please use it consistently.
	 *
	 * Description:  A column of asterisks on the left side,
	 * with beginning and ending almost-blank lines.
	 */
```
* 但在` net/ `和 `drivers/net/` 长行注释风格为：
```cpp
	/* The preferred comment style for files in net/ and drivers/net
	 * looks like this.
	 *
	 * It is nearly the same as the generally preferred comment style,
	 * but there is no initial almost-blank line.
	 */
```
# 9 数据结构

* 在创建和销毁该数据结构的线程之外具有可见性的数据结构应始终具有引用计数。在内核中，垃圾收集不存在，这意味着必须引用计数所有的用途。

* 引用计数意味着您可以避免锁定，并允许多个用户并行访问数据结构。请注意，锁定不是引用计数的替代。锁定用于保持数据结构的一致性，而引用计数是一种内存管理技术。通常两者都是必需的，不要相互混淆。

* 当存在不同“级”的用户时，许多数据结构确实可以具有两个级别的引用计数。子类计数计算子类用户的数量，并在子类计数变为零时减少一次全局计数。这种“多级引用计数”的示例可以在内存管理（“struct mm_struct”：mm_users和mm_count）和文件系统代码（“struct super_block”：s_count和s_active）中找到。

# 10 宏、枚举
* 宏常量 必须大写，但函数宏可以使用小写
* 定义几个连续的常量时应使用枚举
* 通常，内联函数优于类似的函数宏。
* 具有多个语句的宏应该包含在do-while块中：
```cpp
#define macrofun(a, b, c) 			\
	do {					\
		if (a == 5)			\
			do_this(b, c);		\
	} while (0)
```
> 使用宏时要避免的事情：
>* 影响控制流程的宏：
>```cpp
>#define FOO(x)					\
>	do {					\
>		if (blah(x) < 0)		\
>			return -EBUGGERED;	\
>	} while(0)
>```
>* 依赖于具有魔术名称的局部变量的宏：`#define FOO(val)  bar(index，val）`
>* 带参数的左值宏：`FOO（x）= y;`
>* 忘记优先级：使用表达式定义常量的宏必须将表达式括在括号中。`#define CONSTEXP（CONSTANT | 3）`

# 11 打印内核消息

* 不要使用像“dont”这样的残缺话语;使用“do not”或“don't”代替。使消息简洁，清晰，明确。
* 内核消息不必以句点终止。
* 应该避免。在括号中打印数字：`（％d）`
* 应该使用`<linux/device.h>`中的许多驱动程序模型诊断宏来确保消息与正确的设备和驱动程序匹配，并使用正确的级别标记：`dev_err（），dev_warn（）， dev_info（）`等等。对于与特定设备无关的消息，`<linux/printk.h>`定义`pr_notice（），pr_info（），pr_warn（），pr_err（）`等。
* 打印调试消息的处理方式与打印其他非调试消息的方式不同。当其他pr_XXX（）函数无条件打印时，`pr_debug（）`不会；默认情况下会编译它，除非定义了`DEBUG`或设置了`CONFIG_DYNAMIC_DEBUG`。对于`dev_dbg（）`也是如此，并且相关约定使用`VERBOSE_DEBUG`将`dev_vdbg（）`消息添加到已由`DEBUG`启用的消息中。
* 许多子系统都有Kconfig调试选项，可以在相应的Makefile中打开`-DDEBUG`;在其他情况下特定文件使用`#define DEBUG`。有时调试消息应该被无条件打印时，例如，如果它已经在调试相关的`#ifdef`部分中，则可以使用`printk（KERN_DEBUG ...）`。
# 12 分配内存

* 内核提供以下通用内存分配器：`kmalloc（），kzalloc（），kmalloc_array（），kcalloc（），vmalloc（）和vzalloc（）`。有关它们的更多信息，请参阅API文档。
* 传入结构大小的首选形式如下：`p = kmalloc（sizeof（* p），...）`;
* 强制转换一个void指针是多余的。 C编程语言保证从void指针到任何其他指针类型的转换。
* 分配数组的首选形式如下：`p = kmalloc_array（n，sizeof（...），...）;`
* 分配归零数组的首选形式如下：`p = kcalloc（n，sizeof（...），...）;`
* 两种形式都检查分配大小`n * sizeof（...）`上的溢出，如果发生则返回NULL。
		
# 13 内联函数
* 不要滥用内联函数
* 不要将超过3行的函数作为内联函数，除非已知参数是编译时常量的情况，而且因为这个常量你确定编译器在 编译时能优化掉你的函数的大部分代码。
# 14 函数返回值

* 函数可以返回许多不同类型的值，其中一个最常见的是指示函数是成功还是失败的值。
	* 如果函数的名称是操作或命令性命令，则该函数应返回错误代码整数（-Exxx =失败，0 =成功）。
	* 如果名称是谓词，则该函数应返回布尔值 (0 = 失败, non-zero = 成功).。
* 所有EXPORTed函数都必须遵守此约定，所有公共函数也应如此。私有（静态）函数不需要，但建议他们这样做。
* 返回值是计算的实际结果而不是指示是否成功的函数不受此规则的约束。通常它们通过返回一些超出范围的结果来指示失败。典型的例子是返回指针的函数;他们使用NULL或ERR_PTR机制来报告失败。

# 15 内联汇编
* 当C可以完成工作时，不要使用内联汇编。尽可能的情况下从C中与硬件交互。

* 应该编写简单的辅助函数来包装内联汇编，而不是重复编写它们。请记住，内联汇编可以使用C参数。

* 大型的汇编函数应该放在.S文件中，并在C头文件中定义相应的C原型。汇编函数的C原型应该使用“asmlinkage”。

* 您可能需要将您的asm语句标记为volatile，以防止GCC在其发现任何副作用时将其删除。但是，并不总是需要这样做，并且不必要地执行此操作会限制优化。

* 在编写包含多个指令的单个内联汇编语句时，将每个指令放在单独的带引号的字符串中，并单独成行，并使用\n\t结束除最后一个字符串以外的每个字符串：
```cpp
asm ("magic %reg1, #42\n\t"
	"more_magic %reg2, %reg3"
	: /* outputs */ : /* inputs */ : /* clobbers */);
```
		
# 16 条件编译 
* 尽可能不要在.c文件中使用预处理器条件（＃if，＃ifdef）;这样做会使代码更难阅读，逻辑更难以遵循。相反，在头文件中使用。
* 不要将#ifdef放在表达式中，而是将部分或全部表达式分解为单独的辅助函数，并该函数置于条件编译中。
* 如果您有一个可能在特定配置中未使用的函数或变量，并且编译器会警告其定义未使用，请将该定义标记为`__maybe_unused`，而不是将其包装在预处理器条件编译中。 （如果函数或变量始终未使用，请将其删除。）
* 在代码中，尽可能使用`IS_ENABLED`宏将Kconfig符号转换为C布尔表达式，并在正常的C条件中使用它。编译器将对此进行常量折叠，并且像#ifdef一样包含或排除代码块，因此这不会增加任何运行时开销。 但是，这种方法仍然允许C编译器查看块内的代码，并检查它的正确性（语法，类型，符号引用等）。 因此，如果块内的代码引用了不存在的符号，则仍然必须使用#ifdef。
```c
if（IS_ENABLED（CONFIG_SOMETHING））{
...
}
```

* 如果#if或#ifdef块（超过几行）较长，则要对应的的#endif之后放置注释 例如：
```c
#ifdef CONFIG_SOMETHING
...
#endif /* CONFIG_SOMETHING */
```
# 17 其它
* 不要重新发明内核宏
	* 头文件`include/linux/kernel.h`包含许多你应该使用的宏，而不是自己重新定义它们的一些变体。 例如，
		* 如果需要计算数组的长度，请利用宏`#define ARRAY_SIZE(x) (sizeof(x) / sizeof((x)[0]))`；
		* 如果需要计算某些结构成员的大小，请使用`#define FIELD_SIZEOF(t，f) (sizeof(((t *(0)->f))`

* 不要在源文件中包含编辑器的配置信息
------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>《内核官方文档》。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>


------
