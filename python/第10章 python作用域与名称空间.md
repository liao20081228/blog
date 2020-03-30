---
title: 第10章 python作用域与名称空间
tags: python
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《菜鸟教程》](http://www.runoob.com/ "点击跳转")、[《易百教程》](https://www.yiibai.com/ "点击跳转")、[《python3 官方文档》](https://www.python.org/doc/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

&emsp;&emsp;*注意：此教程基于python3.5*

------



# 1 变量作用域
* 作用域
	* 全局作用域——在函数体外部定义的变量具有全局作用域。可以在整个程序范围内访问
	* 局部作用域——在函数体内定义的变量具有局部作用域。只能在其被声明的函数内部访问

# 2 命名空间
 * 变量是映射到对象的名称(标识符)，命名空间是变量名(键)及其对应对象(值)的字典。
 * 一个 Python 表达式可以访问局部命名空间和全局命名空间里的变量。如果一个局部变量和一个全局变量重名，则局部变量会覆盖全局变量。
 * 每个函数都有自己的命名空间。类的方法的作用域规则和普通函数的一样。
 * Python 会智能地猜测一个变量是局部的还是全局的，它假设任何在函数内赋值的变量都是局部的。因此，一个函数可以随意读取全局变量，但是要修改全局变量的时候有两种方法:
     * 使用`global VarName` 声明全局变量 ，`global VarName` 表达式会告诉 Python， VarName 是一个全局变量，这样 Python 就不会在局部命名空间里寻找这个变量了。
     * 全局变量是可变类型数据的时候可以修改


## 2.1 globals() 和 locals() 函数
* `globals()`和`locals()`函数可用于返回全局和局部命名空间中的名称，具体取决于它们被调用的位置。

 * 如果在函数内部调用 locals()，返回的是所有能在该函数里访问的命名。
 * 如果在函数内部调用 globals()，返回的是所有在该函数里能访问的全局名字。
* 这两个函数的返回类型是字典。 因此，可以使用keys()函数提取名称。

Python中通过提供 namespace 来实现重名函数/方法、变量等信息的识别，其一共有三种 namespace，分别为：

local namespace: 作用范围为当前函数或者类方法
global namespace: 作用范围为当前模块
build-in namespace: 作用范围为所有模块
当函数/方法、变量等信息发生重名时，Python会按照 “local namespace -> global namespace -> build-in namespace”的顺序搜索用户所需元素，并且以第一个找到此元素的 namespace 为准。

同时，Python中的内建函数locals()和globals()可以用来查看不同namespace中定义的元素。






------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《菜鸟教程》](http://www.runoob.com/ "点击跳转")、[《易百教程》](https://www.yiibai.com/ "点击跳转")、[《python3 官方文档》](https://www.python.org/doc/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

&emsp;&emsp;*注意：此教程基于python3.5*

------
