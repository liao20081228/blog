---
title: printf系列函数
tags: Linux编程手册
---


------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>[《Linux man-pages 4.15》](https://www.kernel.org/doc/man-pages/)。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

# 1 名字
printf, fprintf, dprintf, sprintf, snprintf, vprintf, vfprintf, vdprintf, vsprintf, vsnprintf——格式化输出转换
# 2 概述
c
#include <stdio.h>

int printf(const char *format, ...);
int fprintf(FILE *stream, const char *format, ...);
int dprintf(int fd, const char *format, ...);
int sprintf(char *str, const char *format, ...);
int snprintf(char *str, size_t size, const char *format, ...);

#include <stdarg.h>

int vprintf(const char *format, va_list ap);
int vfprintf(FILE *stream, const char *format, va_list ap);
int vdprintf(int fd, const char *format, va_list ap);
int vsprintf(char *str, const char *format, va_list ap);
int vsnprintf(char *str, size_t size, const char *format, va_list ap);


glibc的功能测试宏要求（参见feature_test_macros(7)）

snprintf(), vsnprintf():
&emsp;&emsp;_XOPEN_SOURCE >= 500 || _ISOC99_SOURCE || || /* Glibc versions <= 2.19: */ _BSD_SOURCE

dprintf(), vdprintf():
&emsp;&emsp;Since glibc 2.10:
&emsp;&emsp;&emsp;&emsp;_POSIX_C_SOURCE >= 200809L
&emsp;&emsp;Before glibc 2.10:
&emsp;&emsp;&emsp;&emsp;_GNU_SOURCE

# 3 说明
**printf**()系列中的函数根据如下所述的格式<u>format</u>产生输出。函数**printf**()和**vprintf**()将输出写入标准输出流<u>stdout</u>。 **fprintf**()和**vfprintf**()将输出写入给定的输出流<u>stream</u>； **sprintf**()，**snprintf**()，**vsprintf**()和**vsnprintf**()写入字符串<u>str</u>。

 函数**dprintf**()与**fprintf**()相同，只不过它输出到文件描述符<u>fd</u>而不是标标准I/O流。

函数**snprintf**()和**vsnprintf**()最多将<u>size</u>大小的字节（包括终止的空字节（'\0'））写入<u>str</u>。

函数**vprintf**()，**vfprintf**()，**vdprintf**()，**vsprintf**()，**vsnprintf**()分别等效于函数**printf**()，**fprintf**()，**dprintf**()，**sprintf**()，**snprintf**()，除了使用<u>va_list</u>而不是可变数量的参数来调用它们之外。 这些函数不调用<u>va_end</u>宏。 因为它们调用<u>va_arg</u>宏，所以在调用这些函数之后，<u>ap</u>的值是不确定的。 参见stdarg（3）。

所有这些函数都在格式字符串的控制下进行输出，该格式字符串指定如何转换后续参数（或通过stdarg(3)的可变长度参数工具访问的参数）以输出。

C99和POSIX.1-2001指定：如果对**sprintf**()，**snprintf**()，**vsprintf**()或**vsnprintf**()的调用将导致在重叠的对象之间发生复制（例如，目标字符串数组和提供的输入参数之一指向相同的缓冲区），则结果未定义。 请参阅注意事项。
 
## 3.1 格式字符串的格式
 
格式字符串是一个字符串，以其初始移位状态（如果有）开始和结束。格式字符串由零个或多个指令组成：普通字符（非 <b>％</b>），原样复制到输出流；和转换规范，每个规范都会导致获取零个或多个后续参数。每个转换说明都以字符％开头，并以转换说明符结尾。在两者之间可能有（按顺序）零个或多个<u>标志</u>，一个可选的最小<u>字段宽度</u>，一个可选的<u>精度</u>和一个可选的<u>长度修饰符</u>。

参数必须与转换说明符正确对应（在类型提升之后）。默认情况下，参数以给定的顺序使用，其中每个'\*'（请参见下面的<u>字段宽度</u>和<u>精度</u>）和每个转换说明符都要求下一个参数（如果给出的参数太多，则会出错）。还可以通过写“％m\$”而不是“％”和“\*m\$”而不是“\*”来明确指定在需要参数的每个位置采用哪个参数，其中十进制整数<u>m</u>表示所需参数在参数列表中的位置，索引从1开始。因此，
&emsp;&emsp;printf("%*d", width, num);
和
&emsp;&emsp;printf("%2$*1$d", width, num);

是等价的。第二种样式允许重复引用相同的参数。 C99标准不包含使用'\$'的样式，该样式来自单独的UNIX规范。 如果使用使用'\$'的样式，则必须在所有带参数的转换以及所有宽度和精度参数中使用该样式，但是它可以与不占用参数的“%%”格式混合使用。 使用'\$'指定的参数数量可能没有空隙；例如，如果指定了参数1和3，则还必须在格式字符串的某处指定参数2。
 
对于某些数值转换，会使用基数字符（“小数点”）或数千分字符。 实际使用的字符取决于语言环境的**LC_NUMERIC**部分。 （请参阅**setlocale**(3)）POSIX语言环境使用“.”作为基数字符，并且没有分组字符。从而，
&emsp;&emsp;printf（“％.2f”，1234567.89）;
会在POSIX语言环境中生成“1234567.89”，在nl_NL语言环境中生成“1234567,89”，在da_DK语言环境中生成“1.234.567,89”。
 
## 3.2 标志字符
 字符％后跟零个或多个以下标志：
 
 |标志|说明|
 |:--|:--|
 |#|该值应转换为“备用格式”。对于**o**转换，将输出字符串的第一个字符设置为零（如果不为零，则将其前缀为0）。对于**x**和**X**转换，非零结果前面带有字符串“0x”（对于X转换，则为“0X”）。对于**a，A，e，E，f，F，g**和**G**转换，即使没有数字跟随，结果也始终包含小数点（通常，只有在后面有数字时，这些转换的结果中才会出现小数点）。对于g和G转换，不会从结果中删除尾随的零，否则将移除。对于其他转换，结果是不确定的。|
|0|该值应用零填充。对于**d，i，o，u，x，X，a，A，e，E，f，F，g**和**G**转换，转换后的值在左侧填充零而不是空白。 如果同时出现0和-标志，则忽略标志0。 如果通过数字转换（**d，i，o，u，x**和**X**）给出精度，则忽略标志0。 对于其他转换，行为是不确定的。
|-|转换后的值在字段边界上向左调整。（默认值为右对齐。）转换后的值在右侧用空格填充，而不是在左侧用空格或零填充。 如果同时给出-，则-会覆盖0|
|空格|在带符号的转换产生的正数（或空字符串）之前应留一个空格。
| +|应始终将符号（+或-）放在由带符号转换产生的数字之前。 默认情况下，符号仅用于负数。 如果同时使用+和空格，则+会覆盖空格。
|&emsp;&emsp;||

上面的五个标志字符在C99标准中定义。 单独的UNIX规范还指定了另一个标志字符。

 |标志|说明|
 |:--|:--|
 |'|对于十进制转换（**i，d，u，f，F，g，G**），如果语言环境信息指示任何输出，则输出将用数千分组字符分组。（请参阅setlocale(3)。）请注意，许多版本的gcc(1)无法解析此选项，并且会发出警告。 （SUSv2不包含<u>％'F</u>，但是SUSv3添加了它。）
|&emsp;&emsp;||
 glibc 2.2添加了另一个标志字符。
 
|标志|说明|
|:--|:--|
|I|对于十进制整数转换（**i，d，u**），输出使用语言环境的替代输出数字（如果有）。 例如，自从glibc 2.2.3，这将在波斯语（“fa_IR”）语言环境中给出阿拉伯语-印度数字。
 |&emsp;&emsp;| |
 
## 3.3 字段宽度
 
 一个可选的十进制数字字符串（第一位非零）指定最小字段宽度。如果转换后的值的字符数少于字段宽度，它将在左边（或右边，如果已给出左调整标志）处填充空格。代替十进制数字字符串，可以写“\*”或“ \*m\$”（对于某些十进制整数m）来指定字段宽度分别在下一个参数或第m个参数中给出，必须属于<u>int</u>类型。负的宽度被认为为“-”标志后跟正的字段宽度。在任何情况下，不存在或较小的字段宽度都不会导致字段被截断；如果转换结果比字段宽度宽，则将字段扩展为包含转换结果。

## 3.4 精度
可选精度，以句点（'.'）形式，后跟可选的十进制数字字符串。代替十进制数字字符串，可以写“\*”或“\* m\$”（对于某些十进制整数m），以指定精度分别在下一个参数或第m个参数中给出。类型为<u>int</u>。如果给定的精度仅为“.”，则精度为零。负精度就像忽略精度一样。这给出了**d，i，o，u，x**和**X**转换出现的最小位数，**a，A，e，E，f**和**F**转换的基数字符后出现的位数，**g**和**G**转换的最大有效位数，或**s**和**S**转换的字符串中要打印的最大字符数。
 
## 3.5 长度修饰符

在此，“整数转换”代表d，i，o，u，x或X转换。

|修饰符|说明|
|:--|:--|
|hh|后面的整数转换对应于<u>signed char</u>或<u>unsigned char</u>参数，或者后面的**n**转换对应于<u>signed char</u>参数的指针。|
|h|后面的整数转换对应于<u>shortr int</u>或<u>unsigned short int</u>参数，或者后面的**n**转换对应于<u>short int</u>参数的指针。|
|l|后面的整数转换对应于<u>long int</u>或<u>unsigned long int</u>参数，或者随后的**n**转换对应于<u>long int</u>参数的指针，或者随后的c转换对应于<u>wint_t</u>参数，或者后面的**s**转换对应于<u>wchar_t</u>参数的指针。
|ll|后面的整数转换对应于<u>long long int</u>或<u>unsigned long long int</u>参数，或者后面的**n**转换对应于<u>long long int</u>参数的指针。|
|q|ll的同义词。这是源自BSD的非标准扩展；避免在新代码中使用它。|
|L|跟随**a，A，e，E，f，F，g**或**G**转换之后的是一个长<u>double</u>参数。 （C99允许％LF，但SUSv2不允许。）|
|j|后面的整数转换对应于**intmax_t**或**uintmax_t**参数，或者后面的**n**转换对应于指向<u>intmax_t</u>参数的指针。|
|z|后面的整数转换对应于<u>size_t</u>或<u>ssize_t</u>参数，或者后续的**n**转换对应于指向<u>size_t</u>参数的指针。
|Z|**z**的非标准同义词，早于**z**的出现。 不要在新代码中使用。|
|t|后面的整数转换对应于<u>ptrdiff_t</u>参数，或者后面的**n**转换对应于指向<u>ptrdiff_t</u>参数的指针。|
|&emsp;&emsp;&emsp;| |
 
SUSv3规定了以上所有内容，但明确指出是非标准扩展的那些修饰符除外。 SUSv2仅指定长度修饰符**h**（在**hd，hi，ho，hx，hX，hn**中）和**l**（·在**ld，li，lo，lx，lX，ln，lc，ls**中）和**L**（在**Le，LE，Lf，Lg，LG**中）。

作为非标准扩展，GNU实现将**ll**和**L**视为同义词，因此可以例如写**llg**（作为符合标准的Lg的同义词）和**Ld**（作为符合标准的**lld**的同义词）。 这种用法是不可移植的。
 
## 3.6 转换说明符

一个字符，指定要应用的转换类型。 转换说明符及其含义是：
|说明符|说明|
|:--|:--|
|d，i |<u>int</u>参数将转换为带符号的十进制表示法。精度（如果有的话）给出了必须出现的最小位数。如果转换后的值需要较少的数字，则在左边用零填充。默认精度为1。当以显式精度0打印0时，输出为空。|
|o，u，x，X|<u>unsigned int</u>参数将转换为无符号八进制（**o**），无符号十进制（**u**）或无符号十六进制（**x**和**X**）表示法。字母**abcdef**用于**x**转换；字母**ABCDEF**用于**X**转换。精度（如果有的话）给出了必须出现的最小位数。如果转换后的值需要较少的数字，则在左边用零填充。默认精度为1。当以显式精度0打印0时，输出为空。|
|e，E|将<u>double</u>参数四舍五入并转换为\[\-\]d.ddde±dd样式，其中小数点字符前有一位数字，小数点的位数等于精度。如果精度丢失，则取为6；如果精度为零，则不显示小数点字符。 E转换使用字母E（而不是e）来引入指数。指数始终至少包含两位数字；如果值为零，则指数为00。|
|f，F|将<u>double</u>参数四舍五入并转换为\[\-\]ddd.ddd格式的十进制表示法，其中小数点字符后的位数等于精度规格。如果缺少精度，则取值为6；否则，取值为0。如果精度明确为零，则不显示小数点字符。如果出现小数点，则在其前面至少出现一位数字。<br /> （SUSv2没有规定**F**，并说可以提供无穷大和NaN的字符串表示形式。SUSv3添加了**F**的规范。C99标准规定了，在**f**转换的情况下，以“\[-]inf”或“\[-]infinity”表示infinity，以“nan”开头的字符串表示NaN；在**F**转换的情况下，以“\[-]INTF”或“\[-]INFINITY”表示INFINITY，以“NAN”表示NaN。)）|
|g，G|<u>double</u>参数将转换为f或**e**样式（对于**G**转换，则转换为**F**或**E**）。精度指定有效位数。如果缺少精度，则给出6位数字。如果精度为零，则将其视为1。如果转换后的指数小于-4或大于或等于精度，则使用样式**e**。尾随零从结果的小数部分中删除；只有在小数点后跟至少一位数字时，小数点才会出现。|
|a，A|（C99；不在SUSv2中，而是在SUSv3中添加）进行转换时，将<u>double</u>参数转换为\[-]**0x**h.hhhhp±样式的十六进制表示法（使用字母abcdef）；对于**A**转换，使用前缀**0X**，字母ABCDEF和指数分隔符**P**。 小数点前有一个十六进制数字，其后的位数等于精度。 如果以2为基数的精确表示形式存在，则默认精度足以满足该值的精确表示形式；否则，该默认精度足够大以区分<u>double</u>类型的值。 对于未规范化的数字，未指定小数点前的数字；对于规范化数字没有指定但非0。
|c|如果不存在l修饰符，则将int参数转换为无符号字符，并写入转换后的字符。如果存在l修饰符，则通过调用wcrtomb(3)函数将wint_t(宽字符)参数转换为多字节序列，转换状态从初始状态开始，并写入转换后的多字节字符串。|
|s|如果不存在l修饰符：const char\*参数应为指向字符数组的指针（指向字符串的指针）。数组中的字符被写入（但不包括）终止空字节（'\ 0'）；如果指定了精度，则最多写入指定个数的字符。如果给出精度，则不需要空字节。如果未指定精度，或者精度大于数组的大小，则该数组必须包含一个终止的空字节。<br />如果存在l修饰符：const wchar_t\* 参数应为指向宽字符数组的指针。数组中的宽字符被转换为多字节字符（每个宽字符通过调用wcrtomb(3)函数，转换状态从第一个宽字符之前的初始状态开始），直至并包括一个终止的空宽字符。生成的多字节字符将写入（但不包括）终止空字节。如果指定了精度，则写入的字节数不超过指定的数字，但不写入部分多字节字符。请注意，精度决定写入的字节数，而不是宽字符或屏幕位置的数目。除非给出精度且精度太小以至于在到达数组末尾之前写入的字节数超过了精度，否则数组必须包含一个终止的null宽字符，|
|C|（不在C99或C11中，但在SUSv2，SUSv3和SUSv4中。）lc的同义词。 不要用|
|S|（不在C99或C11中，但在SUSv2，SUSv3和SUSv4中。）ls的同义词。 不要用|
|p| void\* 指针参数以十六进制打印（就像用％＃x或％＃lx一样）|
|n|到目前为止写入的字符数被存储在相应参数所指向的整数中。 该参数应为int \*或变体，其大小与（可选）提供的整数长度修饰符相匹配。没有参数被转换。 （仿生海狸的C库不支持此说明符。）如果转换规范包括任何标志，字段宽度或精度，则行为未定义。|
|m|（Glibc扩展；由uClibc和musl支持。）打印strerror(errno)的输出。 不需要参数。|
|%|写入“％”。 没有参数被转换。 完整的转换规范为'%%'。|
|&emsp;&emsp;&emsp;&emsp;| |


# 4 返回值

成功返回后，这些函数将返回打印的字符数（不包括用于结束输出到字符串的空字节）。

函数**snprintf**()和**vsnprintf**()写入的字节数不超过size字节（包括终止空字节（'\0'））。 如果由于该限制而输出被截断，则返回值是如果有足够的空间，将被写入最终字符串的字符数（不包括终止的空字节）。 因此，返回的size或更大的值表示输出被截断。 （另请参见下面的注意事项。）

如果遇到输出错误，则返回负值。

# 5 属性
对于本节中使用的术语的解释，请参阅attribute(7)。

|接口|属性|值|
|:--|:--|:--|
|printf(), fprintf(),sprintf(), snprintf() ，<br />vprintf(), vfprintf(), vsprintf(), vsnprintf()|Thread safety|MT-Safe locale|

# 6 遵守标准
fprintf(), printf(), sprintf(), vprintf(), vfprintf(), vsprintf()：POSIX.1-2001, POSIX.1-2008, C89, C99。

snprintf(), vsnprintf(): POSIX.1-2001, POSIX.1-2008, C99。

dprintf（）和vdprintf（）函数最初是GNU扩展，后来在POSIX.1-2008中进行了标准化。

关于snprintf()的返回值，SUSv2和C99彼此矛盾：当使用size = 0调用snprintf()时，SUSv2规定了未指定的返回值，小于1，而在这种情况下C99允许str为NULL，并给出返回值（一如既往）是在输出字符串足够大的情况下要写入的字符数。 POSIX.1-2001及更高版本将其snprintf（）的规范与C99对齐。

glibc 2.1添加了长度修饰符hh，j，t和z以及转换字符a和A。

glibc 2.2添加具有C99语义的转换字符F和标志字符I。

# 7 注意事项

一些程序不谨慎地依赖于以下代码
&emsp;&emsp;sprintf(buf, "%s some further text", buf);
将文本追加到buf。但是，标准明确指出，如果在调用sprintf()，snprintf()，vsprintf()和vsnprintf()时源缓冲区和目标缓冲区重叠，则结果不确定。根据所用gcc(1)的版本以及所用的编译器选项，上述调用不会产生预期的结果。

从glibc 2.1版开始，函数snprintf()和vsnprintf()的glibc实现符合C99标准，即，其行为如上所述。 在glibc 2.0.6之前，输出将截断时它们将返回-1。

# 8 bugs

因为sprintf()和vsprintf()假定使用任意长的字符串，所以调用者必须注意不要溢出实际空间。 这通常是无法保证的。 请注意，生成的字符串的长度与语言环境有关，并且难以预测。 请改用snprintf()和vsnprintf()（或asprintf(3)和vasprintf(3)）。

诸如printf（foo);的代码 通常会指示bug，因为foo可能包含％字符。 如果foo来自不受信任的用户输入，则它可能包含％n，从而导致printf()调用写入内存并创建安全漏洞。

# 9 示例
要将Pi打印到小数点后五个位：

 c
#include <math.h>
#include <stdio.h>
fprintf(stdout, "pi = %.5f\n", 4 * atan(1.0));

以格式"Sunday, July 3, 10:02"打印日期和时间，其中工作日和月份是指向字符串的指针：
c
#include <stdio.h>
fprintf(stdout, "%s, %s %d, %.2d:%.2d\n", weekday, month, day, hour, min);

许多国家/地区使用日-月-年顺序。 因此，国际化版本必须能够按照format指定的顺序打印参数：
c
#include <stdio.h>
fprintf(stdout, format, weekday, month, day, hour, min);

格式取决于语言环境，并且可能置换参数。 带有值：
&emsp;&emsp;"%1$s, %3$d. %2$s, %4$d:%5$.2d\n"
一个人可能会获得"Sonntag, 3. Juli, 10:02"。

要分配足够大的字符串并打印到其中（对于glibc 2.0和glibc 2.1都正确的代码）：
c
#include <stdio.h>
#include <stdlib.h>
#include <stdarg.h>

char *
make_message(const char *fmt, ...)
{
	int size = 0;
	char *p = NULL;
	va_list ap;

	/* Determine required size */

	va_start(ap, fmt);
	size = vsnprintf(p, size, fmt, ap);
	va_end(ap);

	if (size < 0)
		return NULL;

	size++;	  /* For '\0' */
	p = malloc(size);
	if (p == NULL)
		return NULL;

	va_start(ap, fmt);
	size = vsnprintf(p, size, fmt, ap);
	va_end(ap);

	if (size < 0) {
		free(p);
		return NULL;
	}
	if (size < 0)
		return NULL;

	size++;	  /* For '\0' */
	p = malloc(size);
	if (p == NULL)
		return NULL;

	va_start(ap, fmt);
	size = vsnprintf(p, size, fmt, ap);
	va_end(ap);

	if (size < 0) {
		free(p);
		return NULL;
	}

	return p;
}


如果在2.0.6之前的glibc版本中发生截断，则将其视为错误，而不是进行适当处理。

# 10 另请参阅

printf(1), asprintf(3), puts(3), scanf(3), setlocale(3), strfromd(3), wcrtomb(3), wprintf(3), locale(5)


# 11 版权
该页面是Linux手册页项目4.15版的一部分。 该项目的描述，有关报告错误的信息以及此页面的最新版本,可以在<https://www.kernel.org/doc/man-pages/>上找到。


------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>[《Linux man-pages4.15》](https://www.kernel.org/doc/man-pages/)。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
