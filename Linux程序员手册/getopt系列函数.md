---
title: getopt系列函数
tags: Linux程序员手册
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《Linux man pages》](https://linux.die.net/man/ "点击跳转")。。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

<style>table{word-break:initial;}</style>

# 1 getopt
**原型**：
```cpp
#include <unistd.h>
int getopt(int argc, char * const argv[],const char *optstring);

extern char *optarg;
extern int optind, opterr, optopt;
```

**描述**：

getopt解析命令行参数。它的参数argc和argv是在程序调用时传递给main（）函数的参数计数和数组。

参数**argc**它是程序启动时传递给main（）函数的参数。argc为传入程序的参数的个数，默认为1，代表程序自身。
 
参数**argv**是程序启动时传递给main（）函数的参数。argv是一个指针数组，用来存放命令行参数。传入的命令行参数以空白符（包括空格符，制表符，换行符）分界，并以字符串的形式存放在指针数组argv中，argv数组的每个元素存放一个字符串指针，指向一个传入的命令行参数。

以`-`（而不是"\_"或"--"）开头的argv元素就是一个**选项元素**。“-”叫做选项标识符。选项标识符后面可以紧跟一个字符，这个字符叫做**选项字符**。选项字符后面可以紧跟一个或多个字符，这些字符叫做**选项参数**。

也就是说一个选项元素包括选项标识符、选项字符、选项参数三部分，其中选项字符和选项的参数是可选的，如果有选项字符则必须紧跟在标识符后，选项参数根据规则不同可以紧跟在标识符后面，也可以与选项字符用空白符分开。如：命令行`"app.exe -abc -d"`，此时argc为3，argv[0]为“app.exe”，argv[1] 为 “-abc”，argv[2] 为 “-d”。其中“a”是第一个选项字符，“bc”为选项参数，“d”是第二选项字符，没有选项参数。如果重复调用getopt（），它依次返回每个选项元素的选项字符。在前面的例子中第一次返回“a”，第二次返回“d”。

默认情况下，getopt（）在每次调用时会**重新排列**argv数组中除了argv[0]之外的元素，最终这些元素中所有既不是选项（无论合不合法）也不是选项参数的元素都会排到argv数组末尾。这样argv数组中除argv[0]之外的元素分为与选项有关和与选项无关两类。但类内元素的相对顺序并不会变。例如，opstring为“a:b:cd::e:”，命令行为`"app.exe test0 -a test1  test2 -b test3" `，那么argv的最后会变成`"-a test1 -b test3 test0 test2"`。

调用getopt时，getopt会依次扫描optind及其以后的元素，期待找到一个选项元素：
* 如果getopt（）找到一个由opstring规定的合法选项元素，则返回它的选项字符，置optopt为0，并设置外部变量optind为下一个要扫描的元素下标，同时更新静态变量nextchar，以便下次调用时能够继续扫描后面的选项字符或元素。
* 如果找不到选项元素(无论其选项字符是否合法，只要有，就算有选项元素)，getopt（）返回-1，并将optarg置为null。在这种情况下：
  * 如果argv中，除了argv[0]之外，没有与选项无关的元素，则optind就置为argc。
  * 否则，则将optind置为第一个与选项无关的元素的下标。

特殊元素“--”强制结束选项扫描，无论扫描模式如何。

参数**optstring**是合法选项的字符串，比如："a:b:cd::e"，这就是一个选项字符串。对应到命令行就是-a ,-b ,-c ,-d, -e 。规则如下：
* **如果选项字符后跟一个冒号**，就表示这个选项后面必须带有参数（没有带选项参数会报错，然后将optarg置为null，optopt置为该选项字符，且返回’?’），getopt将指向同一个选项元素中的选项参数或下一个argv元素的指针放入**optarg**中，也就是说这个参数可以和选项连在一起写（此时optarg指向同一个同一个选项元素中的选项参数），也可以用空白符隔开（此时optarg指向下一个argv元素），比如`app.exe -a123 `和`app.exe -a 123`（中间有空格）都表示123是-a的参数。特别地，如果漏写选项参数，且它不是最后一个argv元素，则无论下一个元素是什么都将视为该选项字符的选项参数。例如opstring为“a:b:cd::e:”，命令行为`app.exe -a -b`，`-b`会被作为`a`的参数。

* **如果选项字符后跟两个冒号**，就表示这个选项可以有参数，也可以没有参数，但要注意有参数时，**选项字符与选项参数之间不能有空格**（若有空格，则认为该选项不带参数），这一点和一个冒号时是有区别的。这是一个GNU才有的扩展功能。

* **不带冒号**的表示这个选项没有参数。如果命令行参数中此选项字符之后紧跟其他字符，则会被作为选项字符处理。
 
* 如果optstring的第一个字符为“+”或设置了环境变量POSIXLY_CORRECT，则一旦遇到非选项也非选项参数的元素，就认为没有更多选项存在了。

* 如果optstring的第一个字符为'-'，那么每个非选项也非选项参数的argv元素都将被处理为ASCII为1的选项字符的参数。（这被用于编写可以以任何顺序使用选项"-1"和其他argv元素，并且关心两者的顺序的程序。）。

* 如果optstring包含`W;`，则`-W foo`被视为长选项`--foo`。（-W选项由POSIX.2保留，用于实现扩展。）此行为是GNU才有的扩展功能，在glibc 2之前的库中不可用。


外部变量**optind**是要argv数组中将要被扫描的**下一个元素**的下标。系统将该值初始化为1，因为索引0就是程序的名，调用者可以将其重置为1，以重新扫描argv数组，或扫描新的参数向量。
* 如果选项字符与参数写在一起，则optind为下一个元素的下标（即使该元素不存在，或为与选项无关的元素）。
* 如果选项字符与参数用空白符号分开，则optind为选项参数的下一个元素的下标（即使该元素不存在，或为与选项无关的元素）。

外部变量**optarg** 是用来保存选项的参数的；初始值为NULL。如果选项(不论是否合法)无参数，则置为NULL。

外部变量**oprerr** 表示是否将错误信息输出到stderr，为0时表示不输出，非0输出。

外部变量**optopt** 初始值为63，即'?'。表示不在选项字符串optstring中的 或者 缺少必需选项参数的选项字符。

 
**返回值**：

*  如果getopt（）在argv中找到一个合法的选项，则返回该选项字符。如果有参数，则optarg指向其参数，否则optarg为NULL。optopt保持不变。

*  如果getopt（）在argv中找到一个非法的选项，则会根据opterr设置决定是否输出错误消息。
   * 如果选项字符未包含在optstring中的选，择将该字符存储在optopt中，并返回'？'。
   * 如果选项缺少参数，则将外部变量optopt设置为实际选项字符。如optstring的第一个字符（如果是'+'或'-'则看第二个字符）是冒号（':'），那么getopt（）返回':'，否则返回'?'。

**注意**：

一个程序
* 扫描多个参数数组，或者不止一次重新扫描同一个参数数组
* 想要使用GNU扩展功能，例如opstring开始的“+”和“-”，或者在两个扫描之间更改POSIXLY_CORRECT的值

都必须将optind重新初始化为0，而不是1。因为置为0可以强制调用内部初始化函数，这个函数会重新检查POSIXLY_CORRECT和optstring中的GNU扩展功能。
# 2 getopt_long和getopt_long_only
**原型**：
```c
extern char *optarg;
extern int optind, opterr, optopt;

#include <getopt.h>
int getopt_long(int argc, char * const argv[],const char* optstring,
                  const struct option *longopts, int *longindex);
				  
int getopt_long_only(int argc, char * const argv[],const char* optstring,        
                  const struct option *longopts, int *longindex);


struct option 
{
      const char *name;
      int      has_arg;
      int        *flag;
      int         val;
};


```
**描述**：
  
 getopt_long（）函数的作用类似于getopt（），但它可以接收接受长选项，以`--`开头。如果程序仅接受长选项，那么optstring应该被指定为一个空字符串（“”），而不是NULL。如果缩写是唯一的或者可以精确的匹配一些定义的选项，则可以缩写长选项名称。长选项可能会采用`--arg=param`或`--arg param`形式的参数。

getopt_long_only（）类似于getopt_long（），但' - '以及“ -- ”都表示长选项。如果以“-”（而不是“--”）开头的选项与长选项不匹配，但与短选项匹配，则会将其解析为短选项。

参数**argc、argv、optstring**和getopt函数中的一样。 

参数**longopts**是指向getopt.h中声明的`struct option`数组的第一个元素的指针，通常情况下是一个数组。**注意：struct option数组的最后一个元素必须用{0,0,0,0}填充。**。

结构体**struct option**描述如下：
* **name**——长选项的名称。
* **has_arg**——选项是否带参数。
  * no_argument（或0），该选项没有参数。
  * required_argument（或1），该选项需要参数。可以采用`--arg=param`或`--arg param`的形式，
  * optional_argument（或2），该选项具有可选参数。如果有参数则只能采用`--arg=param`的形式。
* **flag**——指定当遇到长选项时，如何返回结果。
  * 如果flag为NULL，那么getopt_long（）返回成员**val**。 （**通常，设flag为NULL，将val设置为等效的短选项字符）**，
   * 如果flag不为NULL ，getopt_long（）返回0，并且**flag**指向一个变量（该变量必须是全局或者静态变量)，则将该变量值设置为val，但如果没有指定该选项则**flag**指向的变量不做改变。
* **val**——要返回的值，或者加载到由flag指向的变量中的值。

参数**longindex**，如果不为NULL，则它指向一个变量。如果长选项合法，则函数返回时将该变量设置为长选项在longopts所指数组的索引。如果长选项非法或者没有更多的长选项，则保持上一次的值不变。
  
**返回值**：

* 当处理短选项时，getopt_long（）和getopt_long_only（）和getopt()进行相同的处理。 
* 当处理长选项时：
  * 对于合法长选项，如果flag为NULL，则返回val，否则返回0。 
  * 对于非法的长选项
    * 如果无法识别，则返回"?"。
    * 如果缺少参数，根据optstring做出与getopt一样的处理
 * 出错和返回-1的情况与getopt（）相同，加上'？'用于模糊匹配或无关的参数。

# 3 示例
```c
#include<stdlib.h>
#include<stdio.h>
#include<unistd.h>
#include <getopt.h>
#include<string.h>
void ShowHelpInfo()
{
	printf("Usage: cmd [-i file] [-o file] [-h] [-v]\n");
	printf("  -h, --help	    display this help\n");
	printf("  -v, --version       show version\n");
	printf("  -i, --input 	      input file\n");
	printf("  -o, --output        output file\n");
	printf("  -t,                 test short option \n");
	printf("       --test1        test long option \n");
	printf("       --test2        test long option \n");
	printf("\n");
}


void ShowVersionInfo()
{
	printf("version 1.1 \n");

}


int main(int argc , char * argv[])
{

	
	char input_file_name[256],
		  output_file_name[256];
 
	int ret = 0;
	int option_index = 0;
	static int flag=0;
	
 	static struct option arg_options[] =
	{
		{"help", 0, 0, 'h'},
		{"version", 0, 0, 'v'},
		{"input", 1, 0, 'i'},
		{"ouput", 1, 0, 'o'},
		{"test1",0,&flag, 520},
		{"test2",0,&flag, 1314},
		{0, 0, 0, 0}
	};
 

	while ((ret = getopt_long(argc, argv, ":hvi:o:t", arg_options, &option_index)) != -1) 
	{
		switch (ret) 
		{
		case 'h':
			ShowHelpInfo();
			return 0;
		case 'v':
			ShowVersionInfo();
			return 0;
		case 'i':
			strncpy(input_file_name, optarg, sizeof(input_file_name));
			printf("input file name is :%s\n",input_file_name);
			break;
		case 'o':
			strncpy(output_file_name, optarg, sizeof(output_file_name));
			printf("output file name is :%s\n",output_file_name);
			break;
		case 't':
			printf("独立短选项\n");
			break;
		case ':':
			printf("miss argument\n");
			return -1;
		case '?':
			printf("unrecognizable argument\n");
			return -1;
		case 0://longindex参数的作用
			if (0 == strcmp(arg_options[option_index].name, "test1"))
			{
				printf("i love you ,%d,\n",flag);
			}
			else if (0 == strcmp(arg_options[option_index].name, "test2"))
			{
				printf("forever !%d,\n",flag);
			}
			break;
		default:
			return -1;
		}
	}
}

```












------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《Linux man pages》](https://linux.die.net/man/ "点击跳转")。。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>
    
------
