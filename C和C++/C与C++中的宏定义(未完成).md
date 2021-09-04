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
SINGLESHARP(singlesharp) //==> "singlesharp"
```
# 2 \#@
`#@`在宏被展开后为后面的宏形参添加单引号。
```c
#define SINGLESHARPWITHAT(arg) #@arg
SINGLESHARPWITHAT(singlesharp) //==> 'singlesharp'
```
# 3 \##
`##`用于连接前后的两个宏形参，同时会将其前后空格移除。
```c
#define A1(name, type)   type name_##type##_type
A1(a1, int) //==>  int name_int_type

#define A2(name, type)   type name##_##type##_type
A1(a1, int) //==>int a1_int_type;

#define A1(name, type)   type name_   ##type ##_type 
A1(a1, int) //==>  int name_int_type

#define A2(name, type)   type name ##_ ##type ##_type
A1(a1, int) //==>int a1_int_type;
```
当宏参数是另一个宏的时候   需要注意的是凡宏定义里有用`#`或`##`的地方宏参数是不会再展开。

```c
#define STR(s)      #s  
printf(STR(vck));            // 输出字符串"vck" 

#define CONS(a,b)   int(a##e##b)
printf("%d\n", CONS(2,3));        // 2e3 输出:2000 

#define TOW   (2) 
#define MUL(a,b) (a*b)
printf("%d*%d=%d\n", TOW, TOW, MUL(TOW,TOW));  //printf("%d*%d=%d\n", (2), (2), ((2)*(2))); 

//INT_MAX和A都不会再被展开
#define A    (2)  
#define STR(s)   #s 
#define CONS(a,b)   int(a##e##b)

printf("int max: %s\n", STR(INT_MAX));  // INT_MAX  这行会被展开为：printf("int max: %s\n", "INT_MAX");
printf("%s\n", CONS(A, A));           // 这一行则是：   printf("%s\n", int(AeA));
```

然而解决这个问题的方法是加多一层中间转换宏。加这层宏的用意是把所有宏的参数在这层里全部展开, 那么在转换宏里的那一个宏就能得到正确的宏参数。

```c
#define A   (2)  
#define _STR(s) #s   
#define STR(s)  _STR(s)           // 转换宏   
#define _CONS(a,b)   int(a##e##b)  
#define CONS(a,b)    _CONS(a,b)        // 转换宏 

printf("int max: %s\n", STR(INT_MAX)); // INT_MAX,int型的最大值，为一个变量。输出为: int max: 0x7fffffff
                                       //STR(INT_MAX) -->  _STR(0x7fffffff) 
printf("%d\n", CONS(A, A));           //输出为：200  , CONS(A, A)  --> _CONS((2), (2))  --> int((2)e(2)) 
```
# 4 经典用法  
## 4.1 合并匿名变量名  
```c
#define   ___ANONYMOUS1(type, var, line)   type   var##line  
#define   __ANONYMOUS0(type,  line)   ___ANONYMOUS1(type, _anonymous, line)  
#define   ANONYMOUS(type)   __ANONYMOUS0(type, __LINE__)   
```
例：`ANONYMOUS(static int);`   即: `static int _anonymous70;`   70表示该行行号；
第一层：`ANONYMOUS(static int);`   ==>   `__ANONYMOUS0(static int, __LINE__);`  
第二 层：`__ANONYMOUS0(static int, __LINE__);`   ==>   `___ANONYMOUS1(static int, _anonymous, 70);` 
第三层：`___ANONYMOUS1(static int, _anonymous, 70);`   ==>   `static int   _anonymous70;`

每次只能解开当前层的宏，所以__LINE__在第二层才能被解开；


## 4.2 记录文件名   
```c
#define   _GET_FILE_NAME(f)    #f   
#define   GET_FILE_NAME(f)  _GET_FILE_NAME(f)   
static char  FILE_NAME[] = GET_FILE_NAME(__FILE__); 
```
其中2用得比较多，很方便。





------

***<font color=blue>版权声明</font>：<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
