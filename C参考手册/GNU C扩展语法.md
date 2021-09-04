---
title: GNU C扩展语法
tags: C_and_C++
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《GCC Manual chapter 6》](https://gcc.gnu.org/onlinedocs/gcc/index.html)。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

# 1 简介

&emsp;&emsp;GNU C提供了ISO C中没有的几种语言特性。（如果使用任何这些特性了，--pedantic选项指示GCC在使用了这些特性时打印警告消息）要在条件编译中测试这些特性是否可用，请检查预定义宏`__GNUC__`，在GCC中总是被预定义。

&emsp;&emsp;扩展语法在C和Objective-C中可以使用，其中绝大部分特性也可以在C++中更实用。

&emsp;&emsp;GCC在C90和C\+\+中也接受一些在ISO C99中但不是ISO C90或ISO C\+\+的特性作为扩展语法。


# 2 语句表达式
GNU C把圆括号中的复合语句（将多个语句用花括号括起来）看成是语句表达式。
* 可以在其中使用开关，循环，局部变量。
* 复合语句的最后一个语句后要跟一个分号；
* 最后一个语句应该是一个表达式，它的值为整个语句表达式的值。
* 常量表达式不孕去使用语句表达式，例如枚举常量的值，位域的宽度或静态变量的初始值。
* 临时变量在语句表达式结束时被销毁。
* 不允许在语句表达式外部使用goto 或者switch中的case或default跳转到语句表达式内部，但可以在语句表达式内部使用goto 或者switch中的case或default跳转到语句表达式外部。

语句表达式式主要用于安全的定义宏：
```c
#define min(x,y) ((x) < (y) ? (x) : (y))
min(++a,++b);/*参数计算两次*/

#define min(x,y) ({int __x; int __y; __x < __y ? __ x :__y;})
min(++a,++b);/*参数计算一次*/
```
# 3 局部标签
GCC允许您在任何块作用域中声明局部标签。
* 局部标签就像普通标签一样，但只能在声明它的块中引用它（使用goto语句或获取其地址）。
* 局部标签的声明必须在块的开始，在任何普通标签和语句之前。
* 局部标签只需声明，而定义和普通标签一样。
* 局部标签对块内的嵌套函数可见。

局部标签声明如下所示：
```c
__label__ label;

//or

__label__ label1, label2, /* … */;

/*for example*/
     __label__ found;
     int a=1;
     int b=1;          
     goto found;
found:  
   printf("%d\n",a+b);
   return 0;

```



------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[Using the GNU Compiler Collection (GCC)](https://gcc.gnu.org/onlinedocs/gcc/index.html)。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
