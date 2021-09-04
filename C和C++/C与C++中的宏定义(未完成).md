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
当宏参数是另一个宏的时候，需要注意的是凡宏定义里有用`#`或`##`的地方宏参数是不会再展开。

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

然而解决这个问题的方法是加多一层中间转换宏。加这层宏的用意是把所有宏的参数在这层里全部展开，那么在转换宏里的那一个宏就能得到正确的宏参数。

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

## 4 可变参数宏
在GNU C中，宏可以接受可变数目的参数，就可变参函数一样，例如:
```c
//可变参数宏
#define pr_debug(fmt,arg...)  printk(KERN_DEBUG fmt, ##arg)

//可变参数函数
void printf(const char* format, ...);
```
但可变参数宏不被ANSI/ISO C++ 所正式支持。直到C99才允许定义可变参数宏(variadic macros)，可变参数宏就像下面这个样子:
```c
#define debug(...) printf(__VA_ARGS__)

Debug("Y = %d\n", y); //处理器会把宏的调用替换成:printf("Y = %d\n", y);

```
省略号代表一个可以变化的参数表。使用保留名 `__VA_ARGS__` 把参数传递给宏。当宏的调用展开时，实际的参数就传递给 `printf()`了。

gcc的预处理提供的可变参数宏定义
```c
#ifdef DEBUG 
	#define dbgprint(format,args...) fprintf(stderr, format, ##args)
#else 
	#define dbgprint(format,args...) 
#endif
```
如此定义之后，代码中就可以用dbgprint了，例如dbgprint(“%s”, __FILE__);
下面是C99的方法:
1
2
#define dgbmsg(fmt,...)     
printf(fmt,__VA_ARGS__)
新的C99规范支持了可变参数的宏。
　　具体使用如下:
以下内容为程序代码:
1
#include <stdarg.h> #include <stdio.h> #define LOGSTRINGS(fm, ...) printf(fm,__VA_ARGS__)  int main() {     LOGSTRINGS("hello, %d ", 10);     return 0; }
但当前只有gcc才支持。
可变参数的宏里的’##’操作说明带有可变参数的宏(Macros with a Variable Number of Arguments)在1999年版本的ISO C 标准中，宏可以象函数一样，定义时可以带有可变参数。宏的语法和函数的语法类似。下面有个例子:
1
#define debug(format, ...) fprintf (stderr, format, __VA_ARGS__)
这里，’…’指可变参数。这类宏在被调用时，它(这里指’…’)被表示成零个或多个符号，包括里面的逗号，一直到到右括弧结束为止。当被调用时，在宏体(macro body)中，那些符号序列集合将代替里面的__VA_ARGS__标识符。更多的信息可以参考CPP手册。
　　GCC始终支持复杂的宏，它使用一种不同的语法从而可以使你可以给可变参数一个名字，如同其它参数一样。例如下面的例子:
1
#define debug(format, args...) fprintf (stderr, format, args)
这和上面举的那个ISO C定义的宏例子是完全一样的，但是这么写可读性更强并且更容易进行描述。
GNU CPP还有两种更复杂的宏扩展，支持上面两种格式的定义格式。
　　在标准C里，你不能省略可变参数，但是你却可以给它传递一个空的参数。例如，下面的宏调用在ISO C里是非法的，因为字符串后面没有逗号:
　　debug (“A message”)
　　GNU CPP在这种情况下可以让你完全的忽略可变参数。在上面的例子中，编译器仍然会有问题(complain)，因为宏展开后，里面的字符串后面会有个多余的逗号。
##__VA_ARGS__
1
#define debug(format, ...) fprintf (stderr, format, ## __VA_ARGS__)
这里，如果可变参数被忽略或为空，’##’操作将使预处理器(preprocessor)去除掉它前面的那个逗号。如果你在宏调用时，确实提供了一些可变参数，GNU CPP也会工作正常，它会把这些可变参数放到逗号的后面。象其它的pasted macro参数一样，这些参数不是宏的扩展。
##还可以起到替换作用，如:
1
#define FUN(IName)  IName##_ptr
这里将会把IName变成实际数据.
怎样写参数个数可变的宏：
　　一种流行的技巧是用一个单独的用括弧括起来的的 “参数” 定义和调用宏, 参数在 宏扩展的时候成为类似 printf() 那样的函数的整个参数列表。
1
#define DEBUG(args) (printf("DEBUG: "), printf args)  if (n != 0) DEBUG(("n is %d\n", n));
明显的缺陷是调用者必须记住使用一对额外的括弧。
gcc 有一个扩展可以让函数式的宏接受可变个数的参数。 但这不是标准。另一种 可能的解决方案是根据参数个数使用多个宏 (DEBUG1, DEBUG2, 等等), 或者用逗号玩个这样的花招:
1
#define DEBUG(args) (printf("DEBUG: "), printf(args))  #define _ , DEBUG("i = %d" _ i);
C99 引入了对参数个数可变的函数式宏的正式支持。在宏 “原型” 的末尾加上符号 … (就像在参数可变的函数定义中), 宏定义中的伪宏 __VA_ARGS__ 就会在调用是 替换成可变参数。
最后, 你总是可以使用真实的函数, 接受明确定义的可变参数。
如果需要替换宏, 使用一个 函数和一个非函数式宏, 如 #define printf myprintf。 [2] 
# 5 经典用法  
## 5.1 合并匿名变量名  
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


## 5.2 记录文件名   
```c
#define   _GET_FILE_NAME(f)    #f   
#define   GET_FILE_NAME(f)  _GET_FILE_NAME(f)   
static char  FILE_NAME[] = GET_FILE_NAME(__FILE__); 
```
其中2用得比较多，很方便。





------

***<font color=blue>版权声明</font>：<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
