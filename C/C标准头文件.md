---
title: C标准头文件
tags: C
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

<style>table{word-break:initial;}</style>

# 1 标准头文件

|头文件|版本|描述|
|--|--|--|
|assert.h|	|条件编译宏，将参数与零比较
|complex.h| C99 起|	复数运算
|ctype.h||	用来确定包含于字符数据中的类型的函数
|errno.h|	|报告错误条件的宏
|fenv.h| C99 起|	浮点数环境
|float.h|	|浮点数类型的极限
|inttypes.h| C99 起|	整数类型的格式转换
|iso646.h| C95 起|	符号的替代写法
|limits.h|	|基本类型的大小
|locale.h|	|本地化工具
|math.h|	|常用数学函数
|setjmp.h|	|非局部跳转
|signal.h|	|信号处理
|stdalign.h| C11 起|	alignas 与 alignof 便利宏
|stdarg.h|	|可变参数
|stdatomic.h| C11 起|	原子类型
|stdbool.h| C99 起|	布尔类型
|stddef.h||常用宏定义
|stdint.h| C99 起|	定宽整数类型
|stdio.h|	|输入/输出
|stdlib.h|	|基础工具：内存管理<br />程序工具<br />字符串转换<br />随机数
|stdnoreturn.h| C11 起|	noreturn 便利宏
|string.h|	|字符串处理
|tgmath.h| C99 起|	泛型数学（包装 math.h 和 complex.h 的宏）
|threads.h| C11 起|	线程库
|time.h|	|时间/日期工具
|uchar.h| C11 起|	UTF-16 和 UTF-32 字符工具
|wchar.h| C95 起|	扩展多字节和宽字符工具
|wctype.h|C95 起|	用来确定包含于宽字符数据中的类型的函数


# 2 limits.h
limis.h规定了整数的极限值，一个字符的宽度，多字的最大宽度，由编译器厂商提供。定义了一系列宏常量。

|宏常量|描述|
|--|--|--|
|CHAR_BIT|字节位数 |
|MB_LEN_MAX|多字节字符的最大字节数 |
|CHAR_MIN|char 的最小值 |-128
|CHAR_MAX|char 的最大值 |127
|SCHAR_MIN<br />SHRT_MIN<br />INT_MIN<br />LONG_MIN<br />LLONG_MIN(C99)|signed char <br /> short <br /> int <br /> long <br /> long long的最小值 |-128<br />-32768<br />-2147483648
|SCHAR_MAX<br />SHRT_MAX<br />INT_MAX<br />LONG_MAX<br />LLONG_MAX(C99)|signed char <br /> short <br /> int <br /> long <br />long long的最大值 
|UCHAR_MAX<br />USHRT_MAX<br />UINT_MAX<br />ULONG_MAX<br />ULLONG_MAX(C99)|unsigned char <br /> unsigned short <br /> unsigned int <br /> unsigned long <br /> unsigned long long 的最大值



-----

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
