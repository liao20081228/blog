---
title: getopt系列函数
tags: Linux程序员手册
---

------

***<font color=blue>版权声明</font>：本文参考<font color=blue>[《Linux man-pages 4.15》](https://www.kernel.org/doc/man-pages/)。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

# 1 名字

getopt，getopt_long，getopt_long_only，optarg，optind，opterr，optopt——解析命令行参数

# 2 概述
**原型**：
```cpp
#include <unistd.h>

int getopt(int argc, char * const argv[], const char *optstring);

extern char *optarg;
extern int optind, opterr, optopt;

#include <getopt.h>

int getopt_long(int argc, char * const argv[], const char *optstring, 
				const struct option *longopts, int *longindex);

int getopt_long_only(int argc, char * const argv[], const char *optstring,
					const struct option *longopts, int *longindex);
					
struct option 
{
	const char	*name;
	int			has_arg;
	int			*flag;
	int			val;
};
```

glibc的特性测试宏要求（参见feature_test_macros(7)）：

**getopt**():
&emsp;&emsp;\_POSIX_C_SOURCE >= 2 || \_XOPEN_SOURCE
**getopt_long**(), **getopt_long_only**():
&emsp;&emsp;\_GNU_SOURCE


# 2 说明
## 2.1 getopt
**getopt**()解析命令行参数。它的参数<u>argc</u>和<u>argv</u>是在程序调用时传递给main()函数的参数计数和数组。

>参数<u>argc</u>它是程序启动时传递给main()函数的参数。<u>argc</u>为传入程序的参数的个数，默认为1，代表程序自身。
>
>参数<u>argv</u>是程序启动时传递给main()函数的参数。<u>argv</u>是一个指针数组，用来存放命令行参数。传入的命令行参数以空白符（包括空格符，制表符，换行符）分界，并以字符串的形式存放在指针数组<u>argv</u>中，<u>argv</u>数组的每个元素存放一个字符串指针，指向一个传入的命令行参数。

以`-`（而不是`_`或`--`）开头的<u>argv</u>元素就是一个**选项元素**。`-`叫做选项标识符。选项标识符后面可以紧跟一个字符，这个字符叫做**选项字符**。选项字符后面可以紧跟一个或多个字符，这些字符叫做**选项参数**。

>也就是说一个选项元素包括选项标识符、选项字符、选项参数三部分，其中选项字符和选项参数是可选的，如果有选项字符则必须紧跟在标识符后，选项参数根据规则不同可以紧跟在标识符后面，也可以与选项字符用空白符分开。如：命令行`"app.exe -abc -d"`，此时<u>argc</u>为3，argv[0]为“app.exe”，argv[1] 为 “-abc”，argv[2] 为 “-d”。其中“a”是第一个选项字符，“bc”为选项参数，“d”是第二选项字符，没有选项参数。

如果重复调用**getopt**()，它依次返回每个选项元素的选项字符。在前面的例子中第一次返回“a”，第二次返回“d”。

默认情况下，**getopt**()在每次调用时会**重新排列**<u>argv</u>数组中除了argv[0]之外的元素，最终这些元素中所有既不是选项（无论合不合法）也不是选项参数的元素都会排到<u>argv</u>数组末尾。这样<u>argv</u>数组中除argv[0]之外的元素分为与选项有关和与选项无关两类。这两类元素的内部相对顺序并不会变。例如，<u>opstring</u>为`“a:b:cd::e:”`，命令行为`"app.exe test0 -a test1  test2 -b test3" `，那么<u>argv</u>的最后会变成`"-a test1 -b test3 test0 test2"`。

每次调用**getopt**时，**getopt**会依次扫描<u>argv</u>及其以后的元素，期待找到一个选项元素：
* 如果**getopt**()找到一个由<u>opstring</u>规定的合法选项元素，则返回它的选项字符，置<u>optopt</u>为0，并设置外部变量<u>optind</u>为下一个要扫描的元素下标，同时更新静态变量<u>nextchar</u>，以便下次调用时能够继续扫描后面的选项字符或元素。
  * 如果要求参数且选项参数紧跟选项字符，则为下一个元素；
  * 如果要求参数且选项参数与选项字符用空白符分开，则为下下个元素。
* 如果找不到选项元素(无论其选项字符是否合法，只要有，就算有选项元素)，getopt()返回-1，并将<u>optarg</u>置为null。在这种情况下：
  * 如果<u>argv</u>中，除了argv[0]之外，没有与选项无关的元素，则<u>optind</u>就置为<u>argc</u>。
  * 否则，则将<u>optind</u>置为第一个与选项无关的元素的下标。

特殊元素“--”强制结束选项扫描，无论扫描模式如何。

参数<u>optstring</u>是合法选项的字符串，合法的选项字符是指任何可见的单字节 ASCII（7）字符（即 isgraph(3) 函数返回非零值的字符），并且该字符不能是 `-`、`:` 或 `;`。比如：`"a:b:cd::e"`，这就是一个选项字符串。对应到命令行就是-a ,-b ,-c ,-d, -e 。规则如下：

* 如果选项字符后跟**一个冒号`:`**，就表示这个选项后面必须带有参数（没有带选项参数会报错，然后将<u>argv</u>置为null，<u>optopt</u>置为该选项字符，且返回’?’），**getopt**将指向同一个选项元素中的选项参数或下一个<u>argv</u>元素的指针放入<u>optarg</u>中，也就是说这个参数可以和选项连在一起写（此时<u>optarg</u>指向同一个选项元素中的选项参数），也可以用空白符隔开（此时<u>optarg</u>指向下一个<u>argv</u>元素），比如`app.exe -a123 `和`app.exe -a 123`（中间有空格）都表示123是-a的参数。特别地，如果漏写选项参数，且它不是最后一个<u>argv</u>元素，则无论下一个元素是什么都将视为该选项字符的选项参数。例如<u>opstring</u>为`“a:b:cd::e:”`，命令行为`app.exe -a -b`，`-b`会被作为`a`的参数。

* 如果选项字符后跟**两个冒号`::`**，就表示这个选项可以有参数，也可以没有参数，但要注意有参数时，**选项字符与选项参数之间不能有空格**（若有空格，则认为该选项不带参数, <u>optarg</u>置为NULL），这一点和一个冒号时是有区别的。这是一个GNU才有的扩展功能。

* **不带冒号**的表示这个选项没有参数。如果命令行参数中此选项字符之后紧跟其他字符，则会被作为选项字符处理。
 
* 如果<u>optstring</u>的第一个字符为“+”或设置了环境变量**POSIXLY_CORRECT**，则一旦遇到非选项也非选项参数的元素，就认为没有更多选项存在了,即立即停止。

* 如果<u>optstring</u>的第一个字符为“-”，那么每个非选项也非选项参数的<u>argv</u>元素都将被处理为ASCII为1的选项字符的参数。（这将被那些期待以任何顺序使用选项"-1"和其他<u>argv</u>元素的程序使用，或者被并且关心两者顺序的程序使用）。

* 如果<u>optstring</u>包含**W**且后跟一个分号`;`，则<b>-W foo</b>被视为长选项<b>--foo</b>。（-W选项由POSIX.2保留，用于实现扩展。）此行为是GNU才有的扩展功能，在glibc 2之前的库中不可用。


外部变量<u>optind</u>是<u>argv</u>数组中将要被扫描的**下一个元素**的下标。系统将该值初始化为1，因为索引0就是程序的名，调用者可以将其重置为1，以重新扫描<u>argv</u>数组，或扫描新的参数向量。
* 如果选项字符与参数写在一起，则<u>optind</u>为下一个元素的下标（即使该元素不存在，或为与选项无关的元素）。
* 如果选项字符与参数用空白符号分开，则<u>optind</u>为选项参数的下一个元素的下标（即使该元素不存在，或为与选项无关的元素）。

外部变量<u>optarg</u>是用来保存选项的参数的；初始值为NULL。如果选项(不论是否合法)无参数，则置为NULL。

外部变量<u>oprerr</u>表示是否将错误信息输出到stderr，为0时表示不输出，非0输出。

外部变量<u>optopt</u>表示不在选项字符串<u>optstring</u>中的 或者 缺少必需选项参数的选项字符，初始值为63，即'?'。

## 2.2 getopt_long和getopt_long_only

**getopt_long**()函数的作用类似于**getopt**()，但它可以接收接受长选项，以`--`开头。如果程序仅接受长选项，那么<u>optstring</u>应该被指定为一个空字符串（“”），而不是NULL。如果缩写是唯一的或者可以精确的匹配一些定义的选项，则可以缩写长选项名称。长选项可能会采用`--arg=param`或`--arg param`形式的参数。

**getopt_long_only**()类似于**getopt_long**()，但`-`以及`--`都表示长选项。如果以“-”（而不是“--”）开头的选项与长选项不匹配，但与短选项匹配，则会将其解析为短选项。

参数<u>argc</u>、<u>argv</u>、<u>optstring</u>和**getopt**函数中的一样。 

参数<u>longopts</u>是指向getopt.h中声明的<u>struct option</u>数组的第一个元素的指针，通常情况下是一个数组。**注意：struct option数组的最后一个元素必须用{0,0,0,0}填充。**。

结构体<u>struct option</u>描述如下：
* <u>name</u>——长选项的名称。
* <u>has_arg</u>——选项是否带参数。
  * no_argument（或0），该选项没有参数。
  * required_argument（或1），该选项需要参数。可以采用`--arg=param`或`--arg param`的形式，
  * optional_argument（或2），该选项具有可选参数。如果有参数则只能采用`--arg=param`的形式。
* <u>flag</u>——指定当遇到长选项时，如何返回结果。
  * 如果为NULL，那么**getopt_long**()返回成员<u>val</u>（**通常，设flag为NULL，将val设置为等效的短选项字符）**。
   * 如果不为NULL，**getopt_long**()返回0，并且**flag**指向一个变量（该变量必须是全局或者静态变量)，则将该变量值设置为<u>val</u>，但如果没有指定该选项则<u>flag</u>指向的变量不做改变。
* <u>val</u>——要返回的值，或者加载到由<u>flag</u>指向的变量中的值。

参数<u>longindex</u>，如果不为NULL，则它指向一个变量。如果长选项合法，则函数返回时将该变量设置为长选项在<u>longopts</u>所指数组的索引。如果长选项非法或者没有更多的长选项，则保持上一次的值不变。
  

# 3 返回值
## 3.1 getopt
如果**getopt**()在<u>argv</u>中找到一个合法的选项，则返回该选项字符。如果有参数，则<u>optarg</u>指向其参数，否则<u>optarg</u>为NULL。optopt保持不变。

如果**getopt**()在<u>argv</u>中找到一个非法的选项，则会根据<u>opterr</u>设置决定是否输出错误消息。

* 如果选项字符未包含在<u>optstring</u>中，则将该字符存储在<u>optopt</u>中，并返回'？'。
* 如果选项缺少参数，则将外部变量<u>optopt</u>设置为实际选项字符。如<u>optstring</u>的第一个字符（如果是'+'或'-'则看第二个字符）是冒号（':'），那么**getopt**()返回':'，否则返回'?'。

如果没有更多命令行选项，**getopt**()返回-1。

## 3.2 getopt_long和getopt_long_only

当处理短选项时，**getopt_long**()和**getopt_long_only**()和**getopt**()进行相同的处理。 

当处理长选项时：

* 对于合法长选项，如果<u>flag</u>为NULL，则返回<u>val</u>，否则返回0。 
* 对于非法的长选项
  * 如果无法识别，则返回"?"。
  * 如果缺少参数，根据<u>optstring</u>做出与**getopt**()一样的处理
* 出错和返回-1的情况与**getopt**()相同，加上'？'用于模糊匹配或无关的参数。

# 4 环境变量
**POSIXLY_CORRECT**
&emsp;&emsp;如果设置，一旦遇到既非选项元素又非选项参数的元素时停止处理。

**\_\<PID>\_GNU_nonoption_argv_flags_**
&emsp;&emsp;这个变量被**bash**(1)2.0使用，用于向glibc传达哪些参数是通配符展开的结果，所以不应该被视为选项。 这个行为在**bash**(1) 2.01版本中被删除了，但在glibc中仍然支持。

# 5 属性
对于本节中使用的术语的解释，请参阅**attribute**(7)。

|接口|属性|值|
|:--|:--|:--|
|**getopt**(), **getopt_long**(), <br />**getopt_long_only**()|Thread safety | MT-Unsafe race:getopt env|

# 6 遵守标准
**getopt**()：

&emsp;&emsp;如果设置了环境变量POSIXLY_CORRECT，则POSIX.1-2001，POSIX.1-2008和POSIX.2。 否则，<u>argv</u>的元素并不是真正的<u>const</u>，因为我们对其进行了置换。 我们假装它们在原型中是<u>const</u>，以便与其他系统兼容。

&emsp;&emsp;在<u>optstring</u>中使用'+'和'-'是GNU扩展。

&emsp;&emsp;在某些较旧的实现中，在<u>\<stdio.h\></u>中声明了**getopt**()。 SUSv1允许该声明出现在<u>\<unistd.h\></u>或<u>\<stdio.h\></u>中。 POSIX.1-1996将<u>\<stdio.h\></u>的使用标记为LEGACY。 POSIX.1-2001不需要声明出现在<u>\<stdio.h\></u>中。
  
**getopt_long**()和**getopt_long_only**()：

&emsp;&emsp;这些函数是GNU扩展。

# 7 注意事项

一个程序扫描多个参数数组，或者不止一次重新扫描同一个参数数组，想要使用GNU扩展功能，例如<u>opstring</u>开始的“+”和“-”，或者在两个扫描之间更改**POSIXLY_CORRECT**的值，都必须将<u>optind</u>重新初始化为0，而不是1。因为置为0可以强制调用内部初始化函数，这个函数会重新检查**POSIXLY_CORRECT**和<u>optstring</u>中的GNU扩展功能。

# 8 示例
## 8.1 getopt
以下简单示例程序使用**getopt**()处理两个程序选项：<u>-n</u>，没有关联值； <u>-t</u> <u>val</u>，它需要一个关联的值。

```cpp
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>

int main(int argc, char *argv[])
{
	int flags, opt;
	int nsecs, tfnd;

	nsecs = 0;
	tfnd = 0;
	flags = 0;
	while ((opt = getopt(argc, argv, "nt:")) != -1)
	{
		switch (opt)
		{
		case 'n':
			flags = 1;
			break;
		case 't':
			nsecs = atoi(optarg);
			tfnd = 1;
			break;
		default: /* '?' */
			fprintf(stderr, "Usage: %s [-t nsecs] [-n] name\n",argv[0]);
			exit(EXIT_FAILURE);
		}
	}

	printf("flags=%d; tfnd=%d; nsecs=%d; optind=%d\n", flags, tfnd, nsecs, optind);

	if (optind >= argc)
	{
		fprintf(stderr, "Expected argument after options\n");
		exit(EXIT_FAILURE);
	}

	printf("name argument = %s\n", argv[optind]);

	/* Other code omitted */

	exit(EXIT_SUCCESS);
}
```

## 8.2 getopt_long

以下示例程序说明了**getopt_long**()及其大多数特性的用法。

```cpp
#include <stdio.h> /* for printf */
#include <stdlib.h>/* for exit */
#include <getopt.h>

int main(int argc, char **argv) 
{
	int c;
	int digit_optind = 0;

	while (1)
	{
		int this_option_optind = optind ? optind : 1;
		int option_index = 0;
		static struct option long_options[] =
		{
			{"add", required_argument, 0,  0 },
			{"append",  no_argument,0,  0 },
			{"delete",  required_argument, 0,  0 },
			{"verbose", no_argument,0,  0 },
			{"create",  required_argument, 0, 'c'},
			{"file",required_argument, 0,  0 },
			{0,  0,   0,  0 }
		};

		c = getopt_long(argc, argv, "abc:d:012", long_options, &option_index);
		if (c == -1)
			 break;

		switch (c)
		{
			case 0:
				printf("option %s", long_options[option_index].name);
				if (optarg)
					printf(" with arg %s", optarg);
				printf("\n");
				break;

			case '0':
			case '1':
			case '2':
				if (digit_optind != 0 && digit_optind != this_option_optind)
					printf("digits occur in two different argv-elements.\n");
				digit_optind = this_option_optind;
				printf("option %c\n", c);
				break;

			case 'a':
				printf("option a\n");
				break;

			case 'b':
				printf("option b\n");
				break;

			case 'c':
				printf("option c with value '%s'\n", optarg);
				break;

			case 'd':
				printf("option d with value '%s'\n", optarg);
				break;

			case '?':
				break;

			default:
				printf("?? getopt returned character code 0%o ??\n", c);
		 }
	}

	if (optind < argc)
	{
		printf("non-option ARGV-elements: ");
		while (optind < argc)
		printf("%s ", argv[optind++]);
		printf("\n");
	}

	exit(EXIT_SUCCESS);
}
```

# 9 另起参阅

**getopt**(1)，**getsubopt**(3)

# 10 版权
该页面是Linux<u>手册页</u>项目4.15版的一部分。 该项目的描述，有关报告错误的信息以及此页面的最新版本,可以在<https://www.kernel.org/doc/man-pages/>上找到。











------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>[《Linux man-pages 4.15》](https://www.kernel.org/doc/man-pages/)。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
