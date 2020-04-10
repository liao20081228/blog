---
title: limits.h 
tags: C
---


------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>[《C参考手册》](https://en.cppreference.com/w/c/header)。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------


limis.h规定了整数的极限值，一个字符的宽度，多字的最大宽度，由编译器厂商提供。定义了一系列宏常量。

|宏常量|描述|
|:--|:--|:--|
|CHAR_BIT|字节位数 |
|MB_LEN_MAX|多字节字符的最大字节数 |
|CHAR_MIN|char 的最小值 |-128
|CHAR_MAX|char 的最大值 |127
|SCHAR_MIN<br />SHRT_MIN<br />INT_MIN<br />LONG_MIN<br />LLONG_MIN(C99)|signed char <br /> short <br /> int <br /> long <br /> long long的最小值 |-128<br />-32768<br />-2147483648
|SCHAR_MAX<br />SHRT_MAX<br />INT_MAX<br />LONG_MAX<br />LLONG_MAX(C99)|signed char <br /> short <br /> int <br /> long <br />long long的最大值 
|UCHAR_MAX<br />USHRT_MAX<br />UINT_MAX<br />ULONG_MAX<br />ULLONG_MAX(C99)|unsigned char <br /> unsigned short <br /> unsigned int <br /> unsigned long <br /> unsigned long long 的最大值


------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>[《C参考手册》](https://en.cppreference.com/w/c/header)。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
