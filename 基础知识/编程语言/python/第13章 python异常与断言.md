---
title: 第13章 python异常与断言
tags: python
---

------
&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《菜鸟教程》](http://www.runoob.com/ "点击跳转")、[《易百教程》](https://www.yiibai.com/ "点击跳转")、[《python3 官方文档》](https://www.python.org/doc/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

&emsp;&emsp;*注意：此教程基于python3.5*
 

# 1 异常的概念
程序中的错误，按照其产生的原因和引起的后果，通常可以分为三种类型：即语法错误、编译错误、运行错误和逻辑错误。

* 语法错误（编译错误）：违反规定的语法规则，程序不能生成可执行程序。
* 运行错误：程序执行时报错，也叫异常。如果不处理则程序会终止执行。
* 逻辑错误：程序成功执行，但结果并非预期。


# 2 异常处理
```python
try:                                        #要捕获异常的代码
	... ...         
except exception1 [as e1]：                  #python3 捕获异常1
	... ...                
except (exception2,exception3,...) [as e2]:  # python3捕获异常2,3....
	... ... 
except :                         # 捕获其余异常...
	... ...                      #如果引发了异常，获得附加的数据
[else:]                          #如果没有异常则执行，else后语句执行
	... ...       
	
[finally:]                       #无论有无异常,finally后语句都要执行
	... ...                

```

try的工作原理是，当开始一个try语句后，python就在当前程序的上下文中作标记，这样当异常出现时就可以回到这里，try子句先执行，接下来会发生什么依赖于执行时是否出现异常。

* 一但出现异常，后面的代码则不会再执行

* 如果当try后的语句执行时发生异常，python就跳回到try并执行第一个匹配该异常的except子句，异常处理完毕，控制流就通过整个try语句（除非在处理异常时又引发新的异常）。

* 如果在try后的语句里发生了异常，却没有匹配的except子句，异常将被递交到上层的try，或者到程序的最上层（这样将结束程序，并打印缺省的出错信息）。

* 如果在try子句执行时没有发生异常，python将执行else语句后的语句（如果有else的话），然后控制流通过整个try语句。

* 无论是否出现异常，finally后的代码都要执行

* 一个异常处理语句中，可以有多个except语句，但只有一个会执行

* 一个except语句中，可以有多个异常组成的元组，表示可以捕获其中的任意一种异常

* except语句后如果不接具体的异常名，则表示捕获所有异常

* `except ... ... as e`：e是对应异常类的一个实例对象。在python2中把`as`换为逗号`,`



# 3 触发异常
语法格式：
&emsp;&emsp;`raise [Exception_name[(arg1,agr2,...,traceback)]] [from exception_source]`
* 单独的raise
	* 引发当前上下文中捕获的异常（当处于在 except 块中）
	* 默认引发 RuntimeError 异常。
* raise 后可以跟随匿名或者命名异常实例对象：
	* Exception——异常的类型(例如，NameError)，实际是一个类
	* arg1,arg2——异常类的构造参数。 参数是可选的；如果没有提供，则为None。
	* traceback——异常类的构造参数，可选的(在实践中很少使用)，如果存在，则是用于异常的追溯对象。
* raise语句后还可以跟随from语句
	* from 是可选的，为异常实例对象设置 `__cause__ `属性表明异常是由谁直接引起的。处理异常时发生了新的异常，在不使用 from 时更倾向于新异常与正在处理的异常没有关联。而 from 则是能指出新异常是因旧异常直接引起的。这样的异常之间的关联有助于后续对异常的分析和排查。from 语法会有个限制，就是第二个表达式必须是另一个异常类或实例。如果在异常处理程序或 finally 块中引发异常，默认情况下，异常机制会隐式工作会将先前的异常附加为新异常的 `__context__ `属性。当然，也可以通过 with_traceback() 方法为异常设置上下文 `__context__ `属性，这也能在 traceback 更好的显示异常信息。
	* from 还有个特别的用法：`raise ... from None` ，它通过设置` __suppress_context__` 属性指定来明确禁止异常关联。


# 4 内建异常
* BaseException —— 所有异常的基类
	* SystemExit  —— 解释器请求退出，由sys.exit()函数引发
	* KeyboardInterrupt  —— 用户中断执行(通常是输入^C)
	* GeneratorExit  —— 生成器(generator)发生异常来通知退出
	* Exception  —— 常规异常的基类
		* StopIteration  —— 迭代器没有更多的值, next()方法不指向任何对象时
		* StopAsyncIteration  —— 必须通过异步迭代器对象的`__anext__()`方法引发以停止迭代
		* ArithmeticError  —— 各种算术错误引发的内置异常的基类
			* FloatingPointError  —— 浮点计算错误
			* OverflowError  —— 数值运算结果太大无法表示
			* ZeroDivisionError  —— 除(或取模)零 (所有数据类型)
		* AssertionError  —— 当assert语句失败时引发
		* AttributeError  —— 属性引用或赋值失败
		* BufferError  —— 无法执行与缓冲区相关的操作时引发
		* EOFError  —— 当当raw_input()或input()函数在没有读取任何数据的情况下达到文件结束条件(EOF)时引发
		* ImportError  —— 导入模块/对象失败
			* ModuleNotFoundError  —— 无法找到模块或在在sys.modules中找到None
		* LookupError  —— 映射或序列上使用的键或索引无效时引发的异常的基类
			* IndexError  —— 序列中没有此索引(index)
			* KeyError  —— 映射中没有这个键
		* MemoryError  —— 内存溢出错误(对于Python 解释器不是致命的)
		* NameError  —— 未声明/初始化对象 (没有属性)
			* UnboundLocalError  —— 访问未初始化的本地变量
		* OSError  —— 操作系统错误，EnvironmentError，IOError，WindowsError，socket.error，select.error和mmap.error已合并到OSError中，构造函数可能返回子类
			* BlockingIOError  —— 操作将阻塞对象(e.g. socket)设置为非阻塞操作
			* ChildProcessError  —— 在子进程上的操作失败
			* ConnectionError  —— 与连接相关的异常的基类
				* BrokenPipeError  —— 另一端关闭时尝试写入管道或试图在已关闭写入的套接字上写入
				* ConnectionAbortedError  —— 连接尝试被对等方中止
				* ConnectionRefusedError  —— 连接尝试被对等方拒绝
				* ConnectionResetError    —— 连接由对等方重置
			* FileExistsError  —— 创建已存在的文件或目录
			* FileNotFoundError  —— 请求不存在的文件或目录
			* InterruptedError  —— 系统调用被输入信号中断
			* IsADirectoryError  —— 在目录上请求文件操作(例如 os.remove())
			* NotADirectoryError  —— 在不是目录的事物上请求目录操作(例如 os.listdir())
			* PermissionError  —— 尝试在没有足够访问权限的情况下运行操作
			* ProcessLookupError  —— 给定进程不存在
			* TimeoutError  —— 系统函数在系统级别超时
		* ReferenceError  —— weakref.proxy()函数创建的弱引用试图访问已经垃圾回收了的对象
		* RuntimeError  —— 在检测到不属于任何其他类别的错误时触发
			* NotImplementedError  —— 在用户定义的基类中，抽象方法要求派生类重写该方法或者正在开发的类指示仍然需要添加实际实现
			* RecursionError  —— 解释器检测到超出最大递归深度
		* SyntaxError  —— Python 语法错误
			* IndentationError  —— 缩进错误
				* TabError  —— Tab和空格混用
		* SystemError  —— 解释器发现内部错误
		* TypeError  —— 操作或函数应用于不适当类型的对象
		* ValueError  —— 操作或函数接收到具有正确类型但值不合适的参数
			* UnicodeError  —— 发生与Unicode相关的编码或解码错误
				* UnicodeDecodeError  —— Unicode解码错误
				* UnicodeEncodeError  —— Unicode编码错误
				* UnicodeTranslateError  —— Unicode转码错误
		* Warning  —— 警告的基类
			* DeprecationWarning  —— 有关已弃用功能的警告的基类
			* PendingDeprecationWarning  —— 有关不推荐使用功能的警告的基类
			* RuntimeWarning  —— 有关可疑的运行时行为的警告的基类
			* SyntaxWarning  —— 关于可疑语法警告的基类
			* UserWarning  —— 用户代码生成警告的基类
			* FutureWarning  —— 有关已弃用功能的警告的基类
			* ImportWarning  —— 关于模块导入时可能出错的警告的基类
			* UnicodeWarning  —— 与Unicode相关的警告的基类
			* BytesWarning  —— 与bytes和bytearray相关的警告的基类
			* ResourceWarning  —— 与资源使用相关的警告的基类。被默认警告过滤器忽略。





# 5 自定义异常
用户自定义异常实际和标准异常一样，它是任意一个内建异常的派生类即可。

```python
class myException(内建异常):
	... ...
```
# 6 断言
python assert（断言）用于判断一个表达式，在表达式条件为 false 的时候触发异常。语法：
&emsp;&emsp;`assert expression, (arg1，arg2...)`
断言失败时，使用arg1,arg2,...作为构造参数触发AssertionError异常。可用try-except捕获。









------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《菜鸟教程》](http://www.runoob.com/ "点击跳转")、[《易百教程》](https://www.yiibai.com/ "点击跳转")、[《python3 官方文档》](https://www.python.org/doc/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

&emsp;&emsp;*注意：此教程基于python3.5*

------
