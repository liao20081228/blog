---
title: C与C++中的main函数
tags: C和C++
---

------

***<font color=blue>版权声明</font>：<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

# 1 main函数的返回值
main函数的返回值，用于说明程序的退出状态。如果返回0，则代表程序正常退出；返回其它数字的含义则由系统决定。通常，返回非零代表程序异常退出。

很多人甚至市面上的一些书籍，都使用了`void main( ) `，其实这是**错误的**。C/C++ 中从来没有定义过`void main( )`。

# 2 C中的main
 在 C89 中，main()是可以接受的。Brian W. Kernighan 和 Dennis M. Ritchie 的经典巨著 The C programming Language 2e（《C 程序设计语言第二版》）用的就是 main( )。不过在最新的 C99 标准中，只有以下两种定义方式是正确的： 
```cpp
int main( void )					//不接受从命令行中获取参数
int main()							//对命令行中参数保持沉默
int main( int argc, char *argv[] )	//可从命令行中获取参数

//argc存放传入的命令行参数个数，包括程序自身，最小值为1
//argv存放传入的命令行参数，以字符串的方式存放，argv[0]表示程序自身
```
如果不接受从命令行参数，请用`int main(void)` ；否则请用`int main( int argc, char *argv[] )`。而`int main()` 对命令行中参数保持沉默。main 函数的**返回值类型必须是 int** ，这样返回值才能传递给程序的激活者（如操作系统）。

如果 main 函数的最后没有写 return 语句的话，C99 规定编译器要自动在生成的目标文件中（如 exe 文件）加入`return 0;`，表示程序正常退出。不过，建议最好在main函数的最后加上return语句，虽然没有这个必要，但这是一个好的习惯。注意，vc6不会在目标文件中加入`return 0;` ，大概是因为 vc6 是 98年的产品，所以才不支持这个特性。现在明白为什么建议最好加上 return 语句了吧！但gcc3.2（Linux 下的 C编译器）会在生成的目标文件中加入 `return 0;` 。 

# 3 C++中的main
C++98 中定义了如下两种 main 函数的定义方式：
```cpp
int main()										//隐式
int main( void)									//显式
int main( int argc, char *argv[] ) 

//argc存放传入的命令行参数个数，包括程序自身，最小值为1
//argv存放传入的命令行参数，以字符串的方式存放，argv[0]表示程序自身
```
`int main()` 等同于 C99 中的 `int main( void )` ，表示函数不接受任何参数；`int main( int argc, char*argv[] )` 的用法也和C99 中定义的一样。同样，main函数的返回值类型也必须是int。如果main函数的末尾没写return语句，C++ 98规定编译器要自动在生成的目标文件中加入 `return 0;` 。同样，vc++ 6也不支持这个特性，但是 g++ 3.2（Linux 下的 C++编译器）支持。 






------

***<font color=blue>版权声明</font>：<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------