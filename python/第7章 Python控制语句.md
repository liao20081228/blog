---
title: 第7章 Python控制语句
tags: python
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《菜鸟教程》](http://www.runoob.com/ "点击跳转")、[《易百教程》](https://www.yiibai.com/ "点击跳转")、[《python3 官方文档》](https://www.python.org/doc/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

&emsp;&emsp;*注意：此教程基于python3.5*

------

# 1 python语句简介
## 1.1 简单语句
* 表达式
* assert语句——检测条件是否为真，若为假则引发AssertionError异常。
* 赋值语句——将值与变量绑定，可单独赋值、连续赋值、同时赋值。
* 增量赋值语句——将变量修改后再赋值给变量。
* pass语句——什么也不做。
* del语句——删除变量，移除数据结构（映射或序列）中的某部分（位置、切片、存储槽）
* print语句——标准输出，python3.x已经删除，转为函数
* return语句——终止函数，并返回值，默认返回None
* yield——暂时终止生成器的执行并生成一个值。
* raise——触发异常
* [from...]import...[as...]——(从....)导入....(重名为....)
* global——标记一个变量为全局变量
* nonlocal——标记一个局部变量不在本地命名空间中
* exec——执行包含python语句的字符串
* break——终止循环
* continue——执行下一次循环


## 1.2 复合语句
* if...elif...else...语句——条件语句，包括else语句和elif语句。
* while...break..continue...else...语句——条件为真时执行循环语句，为假时执行else语句
* for...in...break...continue...else...语句——用于对序列或可迭代对象执行循环，循环结束时执行else语句
* try...expect..else...finally...语句——捕获异常
* with语句——用上下文管理器对代码块进行包装，允许上下文管理器实现一些设置或清理操作。
* def语句——定义函数
* class语句——定义类

# 2 条件语句
```python
#其中"判断条件"成立时（非零），则执行后面的语句
if 判断条件：
    执行语句……

#其中"判断条件"成立时（非零），则执行后面的语句，否则执行else后面的语句。
if 判断条件：
    执行语句……
else：
    执行语句……

#其中"判断条件1"成立时（非零），则执行后面的语句，否则执行elif后面的语句。
if 判断条件1:  
    执行语句1……
elif 判断条件2:
    执行语句2……
elif 判断条件3:
    执行语句3……
else:
    执行语句4……
```
# 3 循环语句
## 3.1 while循环
```python

##当判断条件为true时执行语句，为假false时，循环结束。
while 判断条件：
    执行语句……

##当判断条件为true时执行语句，为假false时，执行else后面的语句。
while 判断条件：
    执行语句……
else:
    执行语句……
```
## 3.2 for循环
```python
#遍历序列sequence，序列可以是字符串、列表、元组、字典等
for iterating_var in sequence:
    执行语句……

#遍历完sequence后，执行else后面的语句。
for iterating_var in sequence:
    执行语句……
else：
    执行语句……
```
# 4 循环控制语句
```
#终止当前循环
break

#下一轮循环
continue
```




------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《菜鸟教程》](http://www.runoob.com/ "点击跳转")、[《易百教程》](https://www.yiibai.com/ "点击跳转")、[《python3 官方文档》](https://www.python.org/doc/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

&emsp;&emsp;*注意：此教程基于python3.5*

------
