---
title: C与C++基本数据类型大小
tags: C_and_C++
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：<font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

<style>table{word-break:initial;}</style>



# 1 VC_64位
|基本类型|有符号| 无符号类型|大小|
|--|--|--|--|
|布尔型  |_Bool或bool|_Bool或bool|1B
|字符型|char singed char| unsigned char |1B|
|短整型|short=short int=signed short=signed short int	|unsigned short=unsigned short int|	2B	|	
|整型|int=signed int	|unsigned int|4B	|
|长整型|**long**=long int=signed long=signed long int|	**unsigned long**= unsigned long int|	4B	|
|超长整型|long long =long long int= signed long long =signed long long int| unsigned long long =unsigned long long int|8B
|单精度浮点型|float|	float|	4B
|双精度浮点型|double|double	|8B|
|长双精度浮点型|**long double**|long double|	8B|
|指针和sizeof|32位4B，64位8B


# 2 GCC_64位


|基本类型|有符号| 无符号类型|大小|
|--|--|--|--|
|布尔型  |_Bool或bool|_Bool或bool|1B
|字符型|char singed char| unsigned char |1B|
|短整型|short=short int=signed short=signed short int	|unsigned short=unsigned short int|	2B	|	
|整型|int=signed int	|unsigned int|4B	|
|长整型|**long**=long int=signed long=signed long int|	**unsigned long**= unsigned long int|	8B	|
|超长整型|long long =long long int= signed long long =signed long long int| unsigned long long =unsigned long long int|8B
|单精度浮点型|float|	float|	4B
|双精度浮点型|double|double	|8B|
|长双精度浮点型|**long double**|long double|	16B|
|指针和sizeof|32位4B，64位8B|






------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：<font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------