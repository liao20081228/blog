---
title: C与C++中的宏定义(未完成)
tags: C_and_C++
---

------

***<font color=blue>版权声明</font>：<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

# 1 \#
`#`在宏被展开后为后面的宏形参添加双引号。
```c
#define SINGLESHARP(arg) #arg
SINGLESHARP(singlesharp) ==> "singlesharp"
```
# 2 \#@
`#@`在宏被展开后为后面的宏形参添加单引号。
```c
#define SINGLESHARPWITHAT(arg) #@arg
SINGLESHARPWITHAT(singlesharp) ==> 'singlesharp'
```
# 3 \##
`##`用于连接前后的两个宏形参。

具体的过程（为方便认知，主观认为如此，实际过程并非如此）：
1）用空格替代##，则可以把宏分成几段，每段和宏形参比较，如果是宏形参，就用相应的实参替换；
2）去掉所有空格，连接这些段。
例：
#define A1(name, type)   type name_##type##_type

A1(a1, int);
步骤1，变为
type、name_、type、_type 四段，用实参替换，得到：int、name_、int、_type（注意name_和type_和前面面的形参并不符，所以没有替换）；
步骤2，得到
int name_int_type;

-------------------------------------------------------------------------------

#define A2(name, type)   type name##_##type##_type
A2(a1, int);
步骤1，变为
type、name、_ 、type 、_type 五段，用实参替换，得到：int、a1、_、int、_type（注意和上面的例子对比）；
步骤2，得到
int a1_int_type;

-------------------------------------------------------------------------------

验证##的去空格连接特性：
<##前后随意加上一些空格>
#define A1(name, type)   type name_   ##type ##_type 
#define A2(name, type)   type name ##_ ##type ##_type
结果是## 会把前后的空格去掉完成强连接，得到和上面结果相同的宏定义。


一、一般用法   我们使用#把宏参数变为一个字符串,用##把两个宏参数贴合在一起.

#define STR(s)      #s  

#define CONS(a,b)   int(a##e##b)

printf(STR(vck));            // 输出字符串"vck"       
printf("%d\n", CONS(2,3));   // 2e3 输出:2000 



二、当宏参数是另一个宏的时候   需要注意的是凡宏定义里有用'#'或'##'的地方宏参数是不会再展开.

1, 

1, 非'#'和'##'的情况 
#define TOW   (2) 
#define MUL(a,b) (a*b)
printf("%d*%d=%d\n", TOW, TOW, MUL(TOW,TOW));  

这行的宏会被展开为：   printf("%d*%d=%d\n", (2), (2), ((2)*(2)));  MUL里的参数TOW会被展开为(2).

2, 当有'#'或'##'的时候 
#define A    (2)  
#define STR(s)   #s   
#define CONS(a,b)   int(a##e##b)

printf("int max: %s\n", STR(INT_MAX));  // INT_MAX  这行会被展开为：printf("int max: %s\n", "INT_MAX");    
printf("%s\n", CONS(A, A));             // 这一行则是：   printf("%s\n", int(AeA));

INT_MAX和A都不会再被展开, 然而解决这个问题的方法很简单.加多一层中间转换宏.
加这层宏的用意是把所有宏的参数在这层里全部展开, 那么在转换宏里的那一个宏(_STR)就能得到正确的宏参数.    
#define A   (2)  
#define _STR(s) #s   
#define STR(s)  _STR(s)           // 转换宏   
#define _CONS(a,b)   int(a##e##b)  
#define CONS(a,b)    _CONS(a,b)        // 转换宏 

printf("int max: %s\n", STR(INT_MAX));     // INT_MAX,int型的最大值，为一个变量。输出为: int max: 0x7fffffff   STR(INT_MAX) -->   _STR(0x7fffffff) 然后再转换成字符串；   
printf("%d\n", CONS(A, A));  输出为：200   CONS(A, A)   -->   _CONS((2), (2))   --> int((2)e(2)) 



三、'#'和'##'的一些应用特例  
1、合并匿名变量名  
#define   ___ANONYMOUS1(type, var, line)   type   var##line  
#define   __ANONYMOUS0(type,  line)   ___ANONYMOUS1(type, _anonymous, line)  
#define   ANONYMOUS(type)   __ANONYMOUS0(type, __LINE__)   
例：ANONYMOUS(static int);   即: static int _anonymous70;   70表示该行行号；
第一层：ANONYMOUS(static int);   -->   __ANONYMOUS0(static int, __LINE__);  
第二 层：__ANONYMOUS0(static int, __LINE__);   -->   ___ANONYMOUS1(static int, _anonymous, 70); 
第三层： ___ANONYMOUS1(static int, _anonymous, 70);    -->   static int   _anonymous70;  
即每次只能解开当前层的宏，所以__LINE__在第二层才能被解开；


2、记录文件名   #define   _GET_FILE_NAME(f)    #f   
#define   GET_FILE_NAME(f)  _GET_FILE_NAME(f)   
static char  FILE_NAME[] = GET_FILE_NAME(__FILE__); 

其中2用得比较多，很方便。
————————————————
版权声明：本文为CSDN博主「peter_teng」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/peter_teng/article/details/9730831




------

***<font color=blue>版权声明</font>：<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
