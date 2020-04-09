---
title: 第3章 python基础知识
tags: python
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《菜鸟教程》](http://www.runoob.com/ "点击跳转")、[《易百教程》](https://www.yiibai.com/ "点击跳转")、[《python3 官方文档》](https://www.python.org/doc/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

&emsp;&emsp;*注意：此教程基于python3.5*

------

<style>table{word-break:initial;}</style>


# 1 两种编程方式
* 交互式编程
```
$python [option]
>>>comd1
>>>comd2
...
```

* 脚本编程
	* 脚本后缀为 .py
	* 脚本第一行用`#!python路径`指明python解释器所在路径，如果使用默认路径也可省略。
	* 脚本第二行用` # -*- coding: 编码方式 -*-` 或者 `#coding=编码方式`指明编码方式。python2.x使用ascii编码，python3.x使用utf-8编码。当脚本中有默认编码方式不支持的字符时，就要设置，否则可省略。

```python
 #!/usr/bin/python
 # -*- coding: utf-8 -*-
 ...
 ...
```
# 2 标识符
* 在 Python2.x里，标识符由字母、数字、下划线组成，但不能以数字开头。在 Python3.x里标识符可以是非ascii字符。
* 表示符不能是保留关键字。
* Python 中的标识符是区分大小写的。
* 以下划线开头的标识符有特殊意义
	* 以单下划线开头表示仅限内部使用。
		* 对于类成员，不影响访问。
		* 对于模块，将导致不能用`from xxx import * `导入该标识符。
	* 以双下划线开头代表类的私有成员，导致解释器重写该标识符，外部不可以访问;
	* 以双下划线开头和结尾的 \_\_foo__ 代表 Python 里特殊方法或特殊属性，如 \_\_init__() 代表类的构造函数。
* 命名规范
	* 模块名和包名：小写字母，单词之间用下划线分割
	* 类名：每个单词首字母大写
	* 全局变量名：大写字母，单词之间用下划线分割
	* 普通变量、函数、方法：小写字母，单词之间用下划线分割
	* 内部变量、函数、方法、类：供内部使用。
	* 类的私有变量、函数、方法（外部访问会报错）：以\__开头（2个下划线），其他和普通变量一样
	* 专有变量、函数、方法：\__开头，\__结尾，一般为python的自带标识符，不要以这种方式命名

```python
class Test:
	def __init__(self):
		self.a=1
		self._a=2
		self.__a=3
		self.__a__=4
	def getval(self):
		return self.__a
	def _getval(self):
		return self.__a
	def __getval(self):
		return self.__a
	def __getval__(self):
		return self.__a
	
print Test().a    # 正常
print Test()._a   #正常,但不应该，表示内部使用
print Test().__a   # 错误
print Test().__a__ # 正常，但不应该，表示语言专用
print Test().getval() #正常
print Test()._getval() #正常
print Test().__getval() #错误
print Test().__getval__() #正常，但不应该，表示语言专用
```


# 3 保留字符
&emsp;&emsp;下面的列表显示了在Python中的保留字。这些保留字不能用作常量或变量，或任何其他标识符名称。
|标识符|意义|2.7支持|3.5支持|
|--|--|--|--|--|--|
|and|逻辑与
|as|[from...]import...as...，或 with .... as...，或except.... as...|不支持except...as...|
|assert|断言
|break|跳出循环
|class|定义类
|continue|下一循环
|def|定义函数
|del|删除变量、属性、对象
|elif|与if搭配
|else|与if，while，for，try搭配
|except|与try搭配
|	exec|||X|
|finally|与try搭配，有无异常都执行
|for|for循环
|from|from...import...，raise ... from ...
|	global|声明全局变量
|if|条件语句
|import|引入
|in|成员判断
|	is|身份判断
|lambda|匿名函数
|not|逻辑非
|or|逻辑或
|pass|空语句
|print|print||X|
|raise| 生成异常
|return|结束函数并返回
|try|捕获异常
|while|while循环
|with|
|yield|
|nonlocal|非当前局部变量|X|
|False|假|X|
|None| |X|
|True|真|X|


# 4 常量与变量
## 4.1 常量
* 数值常量：整数，浮点数，复数。如：123、0.12、a + bj
* 字符串常量：用单引号，双引号，三引号表示。‘str’、"str"、‘’‘str’‘’。其中三引号可以换行、较少使用。


## 4.2变量
### 4.2.1 变量定义与删除

&emsp;&emsp;在Python中，变量无需事先声明，也不需要指定变量类型，当向变量赋值时，Python会自动发出声明。 

&emsp;&emsp;del 用来删除变量或者从某些数据结构中删除成员。

### 4.2.2 变量赋值

```python
#赋值、定义
var = value 
 
#连续赋值
var1=var2=var3=value 

#分别赋值 
var1，var2，var3=value1，value2，value3 

#删除变量
del var1[，var2...] 
```
# 5 模块引入
* 内建函数、类、对象、常量可以直接调用。

* 非内建的需要从模块导入
  * 导入指定对象，可以是私有对象（\_\_var）和内部对象（\_var）：
	  *   `from  module import xxx1[,xxx2...]` 使用方法`xxx1`。
	  *   `from  module import xxx as alias]` 使用方法`alias`。
  * 导入整个模块，含私有对象和内部对象
	  * `import module1[,module2...]`，使用方法`模块.xxx`.
	  *  `import module as alias`，使用方法`alias.xxx`.
  * 导入模块中的所有对象，但不包含私有对象（双下划线开头）和内部对象（下划线开头）
	  * `from module import *`，使用方法`xxx`.

# 6 基本语法
## 6.1 行和缩进
&emsp;&emsp;Python 的代码块（类，函数，循环，判断）不使用大括号 {}， 而是使用缩进。

&emsp;&emsp;缩进的空白数量是可变的，可以是空格或者制表符，但是所有代码块缩进必须包含相同的空白数量。

>* IndentationError: unindent does not match any outer indentation level错误表明缩进方式不一致，有的是 tab 键缩进，有的是空格缩进，改为一致即可。
>* IndentationError: unexpected indent 错误, 则 可能是tab和空格没对齐的问题"，所有 python 对格式要求非常严格。


&emsp;&emsp;缩进相同的一组语句构成一个代码块，我们称之代码组。

&emsp;&emsp;像if、while、def和class这样的复合语句，首行以关键字开始，以冒号结束，该行之后的一行或多行代码构成代码组。我们将首行及后面的代码组称为一个子句(clause)。

## 6.2 一行多句与多行一句 
* Python可以在一行中写多个语句，语句之间使用分号隔开。
* Python可以将一个语句分为多行。
	* 使用斜杠\将一行分为多行。
	* 语句中包含` []，{} ，() ，或三引号（'''或"""）`，且需要续行时，无需使用续行符`\`。


## 6.3 注释
* 单行注释采用#开头，#可以在行首或者行末。

* 多行注释
	* 每行分开注释。
	* 采用三个引号'''...'''或者"""..."""，只能位于行首。(不推荐)


## 6.4 空行
* 函数之间或类的方法之间用空行分隔，表示一段新的代码的开始。
* 类和函数入口之间也用一行空行分隔，以突出函数入口的开始。

* 空行与代码缩进不同，**空行并不是Python语法的一部分**。书写时不插入空行，Python解释器运行也不会出错。但是空行的作用在于分隔两段不同功能或含义的代码，便于日后代码的维护或重构。

* 空行是程序代码的一部分。

# 7 标准输入与输出
* 输入
  * raw_input("提示") 
      * 将输入作为原始数据放入字符串中
  * input("提示")
      * 默认用户输入是合法的python表达式
      * input()=eval(raw_input())
* 输出
  * python2.x中print是内建，
      * 默认输出是换行的，如果要不换行需要在语句末尾加上逗号 （python3不支持）。
      * 输出内不需要加括号
      * 格式化输出 
          * print "C风格格式控制字符串" % (var1,var2...)
          * print "format风格格式控制字符串".format(var1,var2...)
  * python3.x中print是函数
      * 输出是换行的
      * 遇到`,`输出一个空格
      * 需要加括号
      * 格式化输出
          * print("C风格格式控制字符串" % (var1,var2...))
          * print("format风格格式控制字符串".format(var1,var2...))
     * 
         

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《菜鸟教程》](http://www.runoob.com/ "点击跳转")、[《易百教程》](https://www.yiibai.com/ "点击跳转")、[《python3 官方文档》](https://www.python.org/doc/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

&emsp;&emsp;*注意：此教程基于python3.5*

------