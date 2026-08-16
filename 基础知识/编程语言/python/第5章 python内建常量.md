---
title: 第5章 python内建常量与内建函数
tags: python
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《菜鸟教程》](http://www.runoob.com/ "点击跳转")、[《易百教程》](https://www.yiibai.com/ "点击跳转")、[《python3 官方文档》](https://www.python.org/doc/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

&emsp;&emsp;*注意：此教程基于python3.7*

------






# 1 True

* True是bool类型用来表示真值的常量。
* 对常量True进行任何赋值操作都会抛出语法错误。

# 2 False

* False是bool类型用来表示假值的常量。
* 对常量False进行任何赋值操作都会抛出语法错误。


# 3 None

* None表示无，它是NoneType的唯一值。
* 对常量None进行任何赋值操作都会抛出语法错误。
* 对于函数，如果没有return语句，或return不带表达式即相当于返回None。

# 4 NotImplemented

*  NotImplemented是NotImplementedType类型的常量。
* 使用bool()函数进行测试可以发现，NotImplemented是一个真值。
* NotImplemented不是一个绝对意义上的常量，因为他可以被赋值却不会抛出语法错误，我们也不应该去对其赋值，否则会影响程序的执行结果。
* NotImplemented多用于一些二元特殊方法(比如eq、lt等)中做为返回值，表明没有该实现方法，而Python在结果返回NotImplemented时会聪明的交换二个参数进行另外的尝试。
```python
class A(object):
  def init(self,name,value):
     self.name = name
     self.value = value
  def eq(self,other):
     return NotImplemented
 
a1 = A('Tom',1)
a2 = A('Jay',1)
a1 == a2

#执行结果
False
#当执行a1==a2(即调用eq(a1,a2))，返回NotImplemented时，Python会自动交换参数再次调用eq(a2,a1)。
```
# 5 Ellipsis

* Ellipsis是ellipsis类型的常量，它和`…`是等价的。
* 使用bool()函数进行测试可以发现，Ellipsis是一个真值。
* Ellipsis不是一个绝对意义上的常量，因为他可以被赋值却不会抛出语法错误，我们也不应该去对其赋值，否则会影响程序的执行结果。
* **Ellipsis多用于表示循环的数据结构**。
```
>>> a = [1,2,3,4]
>>> a.append(a)
>>> a
[1, 2, 3, 4, [...]]
>>> a
[1, 2, 3, 4, [...]]
>>> len(a)
5
>>> a[4]
[1, 2, 3, 4, [...]]

```
# 6 \__debug__

* \__debug__是一个bool类型的常量。
* 对常量\__debug__进行任何赋值操作都会抛出语法错误。
* 如果Python没有使用-O选项启动，此常量是真值，否则是假值。

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《菜鸟教程》](http://www.runoob.com/ "点击跳转")、[《易百教程》](https://www.yiibai.com/ "点击跳转")、[《python3 官方文档》](https://www.python.org/doc/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

&emsp;&emsp;*注意：此教程基于python3.7*

------
