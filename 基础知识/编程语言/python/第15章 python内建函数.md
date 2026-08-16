---
title: 第15章 python内建函数
tags: python
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《菜鸟教程》](http://www.runoob.com/ "点击跳转")、[《易百教程》](https://www.yiibai.com/ "点击跳转")、[《python3 官方文档》](https://www.python.org/doc/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

&emsp;&emsp;*注意：此教程基于python3.5*

------

<style>table{word-break:initial;}</style>



# 1 内建函数列表
|函数|简介|python2|python3|
|--|--|--|--|
|**[abs](#abs)**(x)|取绝对值
|**[all](#all)**(iterable)|判断 iterable 中的所有元素是否都为 True|2.5|
|**[any](#any)**(iterable)|判断 iterable 中是否含有元素为True|2.5|
|**[ascii](#ascii)**(object)|返回一个表示对象的字符串|不支持|
|**[basestring](#basestring)**()|str与unicode的父类|不支持|2.3|
|**[bin](#bin)**(x)|将整数转换为前缀为“0b”的二进制字符串。|
|**[breakpoint](#breakpoint)**(\*args, \*\*kws)|此函数会在调用时将你陷入调试器中。|不支持|3.7
|**[bool](#bool)**([x])|将x转为布尔类型|
|**[bytearray](#bytearray)**([source[, encoding[, errors]]])|返回一个新字节数组|2.7|
|**[bytes](#bytes)**([source[, encoding[, errors]]])|bytearray 的不可变版本|不支持|
|**[callable](#callable)**(object)|检测object是否可调用
|**[chr](#chr)**(i)|将i转为字符
|**[classmethod](#classmethod)**()|将一个函数转为类方法
|**[compile](#compile)**(source, filename, mode, flags=0, dont_inherit=False, optimize=-1)|将一个字符串编译为字节代码
|**[complex](#complex)**([real[, imag]])|创建一个复数|
|**[delattr](#delattr)**(object, name)|删除属性|
|**[dict](#dict)**(\*\*kwarg)<br />**[dict](#dict)**(mapping, \*\*kwarg)<br />**[dict](#dict)**(iterable, \*\*kwarg)|创建一个字典|
|**[dir](#dir)**(obejct)|返回object或当前范围内的名称列表|
|**[divmod](#divmod)**(a, b)|返回a除以b的商和余数的元组
|**[enumerate](#enumerate)**(iterable, start=0)|返回一个枚举对象|
|**[eval](#eval)**(expression, globals=None, locals=None)|执行一个表达式并返回值|
|**[exec](#exec)**(object[, globals[, locals]])|执行储存在字符串或文件中的 Python 语句|
|**[filter](#filter)**(function, iterable)|过滤掉不符合条件的元素，返回一个迭代器对象|
|**[float](#float)**() |函数用于将整数和字符串转换成浮点数。
|**[format](#format)**(value[, format_spec])|将 value 转换为 format_spec 控制的“格式化”表示|
|**[frozenset](#frozenset)**([iterable])|返回一个新的 frozenset 对象|
|**[getattr](#getattr)**(object, name[, default])|返回对象命名属性的值。|
|**[globals](#globals)**()|以字典类型返回当前位置的全部全局变量。|
|**[hasattr](#hasattr)**(object, name)|判断对象是否包含对应的属性。|
|**[hash](#hash)**(object)|用于获取取一个对象（字符串或者数值等）的哈希值。
|**[help](#help)**([object])|函数用于查看函数或模块用途的详细说明|
|**[hex](#hex)**(x)|将整数转换为以“0x”为前缀的小写十六进制字符串。|
|**[id](#id)**(object)|返回对象的“标识值”。|
|**[input](#input)**([prompt])|提示并返回标准输入字符串。
|**[int](#int)**([x])<br /> **[int](#int)**(x, base=10)|将一个字符串或数字转换为整型|
|**[isinstance](#isinstance)**(object, classinfo)|判断一个对象是否是一个已知的类的实例。
|**[issubclass](#issubclass)**(class, classinfo)|判断 class 是否是 classinfo 的子类
|**[iter](#iter)**(object[, sentinel])|生成迭代器|
|**[len](#len)**(s)|返回对象（字符、列表、元组等）长度或项目个数|
|**[list](#list)**([iterable])|将元组或字符串转换为列表
|**[locals](#locals)**()|以字典类型返回当前位置的全部局部变量|
|**[map](#mao)**(function, iterable, …)|提供的函数对指定序列做映射
|**[max](#max)**(iterable, \*[, key, default])<br />**[max](#max)**(arg1, arg2, \*args[, key])|返回给定参数的最大值|
|**[memoryview](#memoryview)**(obj)|返回给定参数的内存查看对象|
|**[min](#min)**(iterable, \*[, key, default])<br />**[min](#min)**(arg1, arg2, \*args[, key])|返回给定参数的最小值|
|**[next](#next)**(iterator[, default])|返回迭代器的下一个项目|
|**[object](#object)**()|返回一个object|
|**[oct](#oct)**(x)|将一个整数转换成8进制字符串|
|**[open](#open)**(file, mode='r', buffering=-1, encoding=None, errors=None, newline=None, closefd=True, opener=None)|打开一个文件，并返回文件对象
|**[ord](#ord)**(char)|返回对应的 ASCII 数值|
|**[pow](#pow)**(x, y[, z])|返回 x 的 y 次幂；如果 z 存在，则对 z 取余|
|**[print](#print)**(\*objects, sep=' ', end='\n'<br>, file=sys.stdout, flush=False)|打印输出|
|**[property](#property)**(fget=None, fset=None, fdel=None, doc=None)|在新式类中返回属性值|
|**[range](#range)**(stop)<br />**[range](#range)**(start, stop[, step])|返回一个可迭代对象|
|**[repr](#repr)**(object)| 函数将对象转化为供解释器读取的形式|
|**[reversed](#reversed)**(seq)|返回一个反转的序列|
|**[round](#round)**(number[, ndigits]) |返回浮点数x的四舍五入值|
|**[set](#set)**([iterable])|返回一个新的 set 对象|
|**[setattr](#setattr)**(object, name, value)|设置属性值|
|**[slice](#slice)**(stop) <br />**[slice](#)**(start, stop[, step])|切片对象|
|**[sorted](#sorted)**(iterable, \*, key=None, reverse=False)|对所有可迭代的对象进行排序操作|
|**[staticmethod](#staticmethod)**()|返回函数的静态方法|
|**[str](#str)**(object='')<br />**[str](#str)**(object=b'', encoding='utf-8', errors='strict')|将对象转化为适于人阅读的形式|
|**[sum](#sum)**(iterable[, start])|对可迭代对象进行求和计算|
|**[super](#super)**([type[, object-or-type]])|调用父类](#)**(超类)的一个方法|
|**[tuple](#tuple)**([iterable])|转换为元组。|
|**[type](#type)**(object)<br />**[type](#type)**(name, bases, dict)|返回对象类型|
|**[vars](#vars)**([object])|返回具有 dict 属性的对象的 dict 属性。|
|**[zip](#zip)**(\*iterables)|将可迭代的对象中的元素打包成一个个元组
|**[\_\_import\_\_](#import)**(name, globals=None, locals=None, fromlist=(), level=0)|动态加载类和函数 。



# 2 内建普通函数 
* <span id='abs'>`abs(x)`</span>
	* 返回一个数的绝对值。实参可以是整数或浮点数。如果实参是一个复数，返回它的模

```python
#实例
a = 10
b = -3.14
c = 3 + 4j

print("a=%d, b=%d, c=%d" % (abs(a), abs(b), abs(c)))

#以上代码输出结果为：

a=10, b=3, c=5
```
* <span id='all'>`all(iterable)`</span>
	* 如果 iterable 的所有元素为真（或迭代器为空），返回 True ，否则返回false

```python
#实例
def all(iterable):
    for element in iterable:
        if not element:
            return False
    return True


def main():
    all01 = all([1, 2, 3, 4, 5])
    all02 = all(range(0, 10))
    print("all01返回结果为：%s" % all01)
    print("all02返回结果为：%s" % all02)


if __name__ == '__main__':
    main()

#以上代码输出结果为：

all01返回结果为：True
all02返回结果为：False
```

* <span id="any">`any(iterable)`
	* 如果iterable的任一元素为真则返回True。否则返回False，如果迭代器为空，也返回False。

```python
#实例
def any(iterable):
    for element in iterable:
        if not element:
            return True
    return False


def main():
    any01 = all([0, 1, 2, 3, 4, 5])
    any02 = all(range(1, 10))
    print("any01返回结果为：%s" % any01)
    print("any02返回结果为：%s" % any02)


if __name__ == '__main__':
    main()

#以上代码输出结果为：

any01返回结果为：False
any02返回结果为：True
```
	
	
* <span id="ascii">`ascii(object)`</span>
	* 类似 repr() 函数, 返回一个表示对象的字符串, 但是对于字符串中的非 ASCII 字符则返回通过 repr() 函数使用 \x, \u 或 \U 编码的字符。 生成字符串类似 Python2 版本中 repr() 函数的返回值。

```python
#实例
print(ascii("i love you 我爱你"))

#输出

'i love you \u6211\u7231\u4f60'
```


* `basestring()`
	* str和unicode的抽象类，他不能被调用或实例化，但可以用于测试一个对象是否是str或Unicode的实例。 `isinstance(obj, basestring)`等效于`isinstance(obj, (str, unicode))`

```python
#实例
a="str"
print isinstance(a, basestring)

#输出

True

```


* `bin(x)`
	* 将整数转换为前缀为“0b”的二进制字符串。结果是一个有效的Python表达式。如果x不是Python int对象，它必须定义一个返回整数的`__index__()`方法。

```python
#实例
a = bin(3)
b = bin(-10)

print("a=%s b=%s" % (a, b))

#以上代码输出结果为：

a=0b11 b=-0b1010
```

* `breakpoint(*args, **kws)`
	* 此函数会在调用时将你陷入调试器中。具体来说，它调用 sys.breakpointhook() ，直接传递 args 和 kws 。
	* 默认情况下， sys.breakpointhook() 调用 pdb.set_trace() 且没有参数。在这种情况下，它纯粹是一个便利函数，因此您不必显式导入 pdb 且键入尽可能少的代码即可进入调试器。但是， sys.breakpointhook() 可以设置为其他一些函数并被 breakpoint() 自动调用，以允许进入你想用的调试器。

*  `callable(object)`
	* 如果实参 object 是可调用的，返回 True，否则返回 False。如果返回真，调用仍可能会失败；但如果返回假，则调用 object 肯定会失败。注意类是可调用的（调用类会返回一个新的实例）。如果实例的类有 `__call __() `方法，则它是可调用。

```python
#实例：

a = callable(0)
b = callable("runoob")


def add(m, n):
    return m + n


class A(object):
    def method(self):
        return 0


class B(object):
    def __call__(self):
        return 0


def main():
    print("a返回：%s" % a)
    print("b返回：%s" % b)

    print("函数add返回：%s" % callable(add))  # 函数返回True

    print("类A返回：%s" % callable(A))  # 类返回True

    a01 = A()
    print("没有实现__call__()，返回:%s" % callable(a01))

    print("类B返回：%s" % callable(B))

    b01 = B()
    print("实现__call__()，返回:%s" % callable(b01))


if __name__ == '__main__':
    main()
#以上代码输出结果为：

a返回：False
b返回：False
函数add返回：True
类A返回：True
没有实现__call__()，返回:False
类B返回：True
实现__call__()，返回:True
```
* `@classmetheod`
	* 用在方法前面，把一个方法封装成类方法。一个类方法把类自己作为第一个实参，就像一个实例方法把实例自己作为第一个实参。
	* 类方法的调用可以在类上进行 (例如 C.f()) 也可以在实例上进行 (例如 C().f())。 其所属类以外的类实例会被忽略。 如果类方法在其所属类的派生类上调用，则该派生类对象会被作为隐含的第一个参数被传入。
```python
实例：

class A(object):
    bar = 1

    def func1(self):
        print("foo")

    @classmethod
    def func2(cls):
        print("func2")
        print(cls.bar)
        cls().func1()  # 调用foo方法


def main():
    A.func2()  # 不需要实例化


if __name__ == '__main__':
    main()
以上代码输出结果为：

func2
1
foo
```
* `compile(source, filename, mode, flags=0, dont_inherit=False, optimize=-1)`
	* 将 source 编译成代码或 AST 对象。代码对象可以被 exec() 或 eval() 执行。source 可以是常规的字符串、字节字符串，或者 AST 对象。参见 ast 模块的文档了解如何使用 AST 对象。
	* filename 实参需要是代码读取的文件名；如果代码不需要从文件中读取，可以传入一些可辨识的值（经常会使用 ‘\<string\>‘）。
	* mode 实参指定了编译代码必须用的模式。如果 source 是语句序列，可以是 ‘exec’；如果是单一表达式，可以是 ‘eval’；如果是单个交互式语句，可以是 ‘single’。（在最后一种情况下，如果表达式执行结果不是 None 将会被打印出来。）
	* 可选参数 flags 和 dont_inherit 控制在编译 source 时要用到哪个 future 语句。 如果两者都未提供（或都为零）则会使用调用 compile() 的代码中有效的 future 语句来编译代码。 如果给出了 flags 参数但没有 dont_inherit (或是为零) 则 flags 参数所指定的 以及那些无论如何都有效的 future 语句会被使用。 如果 dont_inherit 为一个非零整数，则只使用 flags 参数 — 在调用外围有效的 future 语句将被忽略。Future 语句使用比特位来指定，多个语句可以通过按位或来指定。具体特性的比特位可以通过 future 模块中的 _Feature 类的实例的 compiler_flag 属性来获得。
	* optimize 实参指定编译器的优化级别；默认值 -1 选择与解释器的 -O 选项相同的优化级别。显式级别为 0 （没有优化；debug 为真）、1 （断言被删除， debug 为假）或 2 （文档字符串也被删除）
	* 如果编译的源码不合法，此函数会触发 SyntaxError 异常；如果源码包含 null 字节，则会触发 ValueError 异常。
	* 如果您想分析 Python 代码的 AST 表示，请参阅 ast.parse()。
	* 注解: 在 'single' 或 'eval' 模式编译多行代码字符串时，输入必须以至少一个换行符结尾。 这使 code 模块更容易检测语句的完整性
	* 警告:
		* 在将足够大或者足够复杂的字符串编译成 AST 对象时，Python 解释器有可以因为 Python AST 编译器的栈深度限制而崩溃。
		* 在 3.2 版更改: 允许使用 Windows 和 Mac 的换行符。在 ‘exec’ 模式不再需要以换行符结尾。增加了 optimize 形参。
		* 在 3.5 版更改: 之前 source 中包含 null 字节的话会触发 TypeError 异常。
	* 参数：
		* source — 字符串或者AST（Abstract Syntax Trees）对象。
		* filename — 代码文件名称，如果不是从文件读取代码则传递一些可辨认的值。
		* mode — 指定编译代码的种类。可以指定为 exec, eval, single。
		* flags — 变量作用域，局部命名空间，如果被提供，可以是任何映射对象。
		* flags和dont_inherit是用来控制编译源码时的标志。

```python
实例：

str = "for i in range(0, 10):print(i)"
a = compile(str, "", "exec")
print(a)
print(exec(a))

str = "3 * 4 + 5"
b = compile(str, "", "eval")
print(b)
print(eval(b))
以上代码输出结果为：

<code object <module> at 0x0000021F3AFC40C0, file "", line 1>
0
1
2
3
4
5
6
7
8
9
None
<code object <module> at 0x0000021F3AFC4300, file "", line 1>
17
```

* `delattr(object, name)`
	* setattr()相关的函数。实参是一个对象和一个字符串。该字符串必须是对象的某个属性。如果对象允许，该函数将删除指定的属性。例如 delattr(x, ‘foobar’) 等价于 del x.foobar 。
	* 参数：
		* object — 对象。
		* name — 必须是对象的属性。
```python
实例：

class Coordinate(object):
    x = 10
    y = -5
    z = 0


def main():
    point01 = Coordinate()

    print("x=%d" % point01.x)
    print("y=%d" % point01.y)
    print("z=%d" % point01.z)

    delattr(Coordinate, "z")

    print("---删除z属性后---")
    print("x=%d" % point01.x)
    print("y=%d" % point01.y)
    print("z=%d"%point01.z)  # 删除之后触发会报错


if __name__ == '__main__':
    main()
以上代码输出结果为：

x=10
y=-5
z=0
---删除z属性后---
x=10
y=-5
AttributeError: 'Coordinate' object has no attribute 'z'
```

* `dir([object])`
	* 如果没有实参，则返回当前本地作用域中的名称列表。如果有实参，它会尝试返回该对象的有效属性列表。
	* 如果对象有一个名为` __dir__() `的方法，那么该方法将被调用，并且必须返回一个属性列表。这允许实现自定义 `__getattr__()` 或 `__getattribute__()` 函数的对象能够自定义 dir() 来报告它们的属性。
	* 如果对象不提供 ` __dir__() `，这个函数会尝试从对象已定义的 dict 属性和类型对象收集信息。结果列表并不总是完整的，如果对象有自定义 `__getattr__()`，那结果可能不准确。
		* 默认的 dir() 机制对不同类型的对象行为不同，它会试图返回最相关而不是最全的信息：
		* 如果对象是模块对象，则列表包含模块的属性名称。
		* 如果对象是类型或类对象，则列表包含它们的属性名称，并且递归查找所有基类的属性。
		* 否则，列表包含对象的属性名称，它的类属性名称，并且递归查找它的类的所有基类的属性。
	* 返回的列表按字母表排序。

```python
import struct

list01 = dir(struct)

print(list01)
以上代码输出结果为：

['Struct', '__all__', '__builtins__', '__cached__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__', '_clearcache', 'calcsize', 'error', 'iter_unpack', 'pack', 'pack_into', 'unpack', 'unpack_from']

```
* `divmod(a, b)`
	* 它将两个（非复数）数字作为实参，并在执行整数除法时返回一对商和余数。对于混合操作数类型，适用双目算术运算符的规则。对于整数，结果和 (a // b, a % b) 一致。对于浮点数，结果是 (q, a % b) ，q 通常是 math.floor(a / b) 但可能会比 1 小。在任何情况下， q * b + a % b 和 a 基本相等；如果 a % b 非零，它的符号和 b 一样，并且 0 \<= abs(a % b) \< abs(b) 。

```python
实例：

a = divmod(7, 2)
b = divmod(8, 2)
c = divmod(8, -2)
d = divmod(3, -3.14)

print("a=%s, b=%s, c=%s, d=%s" % (a, b, c, d))
以上代码输出结果为：

a=(3, 1), b=(4, 0), c=(-4, 0), d=(-1.0, -0.14000000000000012)
```
* `eval(expression, globals=None, locals=None)`
	* 实参是一个字符串，以及可选的 globals 和 locals。globals 实参必须是一个字典。locals 可以是任何映射对象。
	* expression 参数会作为一个 Python 表达式（从技术上说是一个条件列表）被解析并求值，使用 globals 和 locals 字典作为全局和局部命名空间。 如果 globals 字典存在且不包含以` __builtins __ `为键的值，则会在解析 expression 之前插入以此为键的对内置模块 builtins 的字典的引用。 这意味着 expression 通常具有对标准 builtins 模块的完全访问权限且受限的环境会被传播。 如果省略 locals 字典则其默认值为 globals 字典。 如果两个字典同时省略，表达式会在 eval() 被调用的环境中执行。 返回值为表达式求值的结果。 语法错误将作为异常被报告。
	* 这个函数也可以用来执行任何代码对象（如 compile() 创建的）。这种情况下，参数是代码对象，而不是字符串。如果编译该对象时的 mode 实参是 ‘exec’ 那么 eval() 返回值为 None 。
	* 提示： exec() 函数支持动态执行语句。 globals() 和 locals() 函数各自返回当前的全局和本地字典，因此您可以将它们传递给 eval() 或 exec() 来使用。
	* 参数：
		* expression — 表达式。
		* globals — 变量作用域，全局命名空间，如果被提供，则必须是一个字典对象。
		* locals — 变量作用域，局部命名空间，如果被提供，可以是任何映射对象。

```python
实例：

x = 7

print(eval("3 * x"))
print(eval("pow(2,2)"))
print(eval("2 + 2"))
print(eval("x + 4"))
以上代码输出结果为：

21
4
4
11
```
* `exec(object[, globals[, locals]])`
	* 这个函数支持动态执行 Python 代码。object 必须是字符串或者代码对象。如果是字符串，那么该字符串将被解析为一系列 Python 语句并执行（除非发生语法错误）。如果是代码对象，它将被直接执行。在任何情况下，被执行的代码都需要和文件输入一样是有效的（见参考手册中关于文件输入的章节）。请注意即使在传递给 exec() 函数的代码的上下文中，return 和 yield 语句也不能在函数定义之外使用。该函数返回值是 None 。
	* 无论哪种情况，如果省略了可选参数，代码将在当前范围内执行。如果提供了 globals 参数，就必须是字典类型，而且会被用作全局和本地变量。如果同时提供了 globals 和 locals 参数，它们分别被用作全局和本地变量。如果提供了 locals 参数，则它可以是任何映射型的对象。请记住在模块层级，全局和本地变量是相同的字典。如果 exec 有两个不同的 globals 和 locals 对象，代码就像嵌入在类定义中一样执行。
	* 如果 globals 字典不包含 builtins键值，则将为该键插入对内建 builtins 模块字典的引用。因此，在将执行的代码传递给 exec() 之前，可以通过将自己的 builtins 字典插入到 globals 中来控制可以使用哪些内置代码。
	* 注解
		* 内置 globals() 和 locals() 函数各自返回当前的全局和本地字典，因此可以将它们传递给 exec() 的第二个和第三个实参。
		* 默认情况下，locals 的行为如下面 locals() 函数描述的一样：不要试图改变默认的 locals 字典。如果您想在 exec() 函数返回时知道代码对 locals 的变动，请明确地传递 locals 字典
	* 参数：
		* object：必选参数，表示需要被指定的Python代码。它必须是字符串或code对象。如果object是一个字符串，该字符串会先被解析为一组Python语句，然后在执行（除非发生语法错误）。如果object是一个code对象，那么它只是被简单的执行。
		* globals：可选参数，表示全局命名空间（存放全局变量），如果被提供，则必须是一个字典对象。
		* locals：可选参数，表示当前局部命名空间（存放局部变量），如果被提供，可以是任何映射对象。如果该参数被忽略，那么它将会取与globals相同的值。

```python
实例：

# 单行语句字符串
exec('print("hello world!")')

# 多行语句字符串
exec("""for i in range(5):
    print("iter time:%d" % i)
    """)
以上代码输出结果为：

hello world!
iter time:0
iter time:1
iter time:2
iter time:3
iter time:4
实例：

x = 10
expr = """
z = 30
sum = x + y + z
print(sum)
"""


def func():
    y = 20
    exec(expr)
    exec(expr, {"x": 1, "y": 2})
    exec(expr, {"x": 1, "y": 2}, {"y": 3, "z": 4})


def main():
    func()


if __name__ == '__main__':
    main()
以上代码输出结果为：

60
33
34
```

* `filter(function, iterable)`
	* 从functio返回true的iterable元素构造一个迭代器。iterable可以是序列、支持迭代的容器或迭代器。如果函数为None，则假定恒等函数，也就是说，iterable中所有为false的元素都被删除。
	* 请注意， filter(function, iterable) 相当于一个生成器表达式，当 function 不是 None 的时候为 (item for item in iterable if function(item))；function 是 None 的时候为 (item for item in iterable if item) 。
	* 参数：
		* function — 判断函数。
		* iterable — 可迭代对象。
```python
实例：

# 过滤出列表中所有的奇数
def is_odd(n):
    return n % 2 == 1


def main():
    templist = filter(is_odd, [1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
    newlist = list(templist)
    print(newlist)


if __name__ == '__main__':
    main()
以上代码输出结果为：

[1, 3, 5, 7, 9]
实例：

import math


# 过滤出1~100中平方根是整数的数：
def is_sqr(x):
    return math.sqrt(x) % 1 == 0


def main():
    templist = filter(is_sqr, range(1, 101))
    newlist = list(templist)
    print(newlist)


if __name__ == '__main__':
    main()
以上代码输出结果为：

[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]

```

* `format(value[, format_spec])`
	* 将 value 转换为 format_spec 控制的“格式化”表示。format_spec 的解释取决于 value 实参的类型，但是大多数内置类型使用标准格式化语法：Format Specification Mini-Language。
	* 默认的 format_spec 是一个空字符串，它通常和调用 str(value) 的结果相同。
	* 调用 format(value, formatspec) 会转换成 type(value).\_\_format\_\_(value, formatspec) ，所以实例字典中的 \_\_format\_\_() 方法将不会调用。如果搜索到 object 有这个方法但 format_spec 不为空，format_spec 或返回值不是字符串，会触发 TypeError 异常。
		* 基本语法是通过 {位置} 和 : 来代替以前的 % 。format 函数可以接受不限个参数，位置可以不按顺序。
		* ^, 、<, 、> 分别是居中、左对齐、右对齐，后面带宽度， : 号后面带填充的字符，只能是一个字符，不指定则默认是用空格填充。
		* +表示在正数前显示 +，负数前显示 -；  （空格）表示在正数前加空格
		* b、d、o、x 分别是二进制、十进制、八进制、十六进制。
		* 此外我们可以使用大括号 {} 来转义大括号

```python



```





# 3 类型转换函数

* <span id='bool'>`class bool([x])`</span>
	* 返回一个布尔值，True 或者 False。 x 使用标准的真值测试过程来转换。如果 x 是假的或者被省略，返回 False；其他情况返回 True。bool 类是 int 的子类。其他类不能继承自它。它只有 False 和 True 两个实例。
```python
#实例
a = bool(0)
b = bool("abc")
c = bool(" ")  # 参数是一个空格，非空。
d = bool("")  # 参数为空。
e = bool([])
f = bool()
g = bool(None)

print("a=%s,b=%s,c=%s,d=%s,e=%s,f=%s,g=%s" % (a, b, c, d, e, f, g))
#以上代码输出结果为：

a=False,b=True,c=True,d=False,e=False,f=False,g=False
```
* <span id='bytearray'>`class bytearray([source[, encoding[, errors]]])`</span>
	* 返回一个新字节数组。 bytearray 类是一个可变序列，包含范围为 0 <= x < 256 的整数。它有可变序列和bytes大部分常见的方法。
	* 可选形参 source 可以用不同的方式来初始化数组：
		* 如果是一个 string，您必须提供 encoding 参数（errors 参数仍是可选的）；bytearray() 会使用 str.encode() 方法来将 string 转变成 bytes。
		* 如果是一个 integer，会初始化大小为该数字的数组，并使用 null 字节填充。
		* 如果是一个符合 buffer 接口的对象，该对象的只读 buffer 会用来初始化字节数组。
		* 如果是一个 iterable 可迭代对象，它的元素的范围必须是 0 <= x < 256 的整数，它会被用作数组的初始内容。
	* 参数 
		* 如果 source 为整数，则返回一个长度为 source 的初始化数组；
		* 如果 source 为字符串，则按照指定的 encoding 将字符串转换为字节序列；
		* 如果 source 为可迭代类型，则元素必须为[0 ,255] 中的整数；
		* 如果 source 为与 buffer 接口一致的对象，则此对象也可以被用于初始化 bytearray。
		* 如果没有输入任何参数，默认就是初始化数组为0个元素。
```python
#实例
a = bytearray()
b = type(a)
c = bytearray([1, 2, 3])
d = type(c)
e = bytearray("runoob", "utf-8")
f = type(e)

print(a, b)
print(c, d)
print(e, f)

#以上代码输出结果为：

bytearray(b'') <class 'bytearray'>
bytearray(b'\x01\x02\x03') <class 'bytearray'>
bytearray(b'runoob') <class 'bytearray'>
```


* <span id='bytes'>`class bytes([source[, encoding[, errors]]])`</span>
	* bytearray的不变版本，此外也具有bytearray的所有非修改方法
```python
#实例
a = bytes([1, 2, 3, 4, 5])
b = type(a)
c = bytes("hello", "ascii")
d = type(c)

print(a, b)
print(c, d)

#以上代码输出结果为：

b'\x01\x02\x03\x04\x05' <class 'bytes'>
b'hello' <class 'bytes'>
```

* `chr(i)`
	* 返回 Unicode 编码为整数 i 的字符的字符串格式。例如，chr(97) 返回字符串 ‘a’，chr(8364) 返回字符串 ‘€’。这是 ord() 的逆函数。
	* 实参的合法范围是 0 到 1,114,111（16 进制表示是 0x10FFFF）。如果 i 超过这个范围，会触发 ValueError 异常。
	* chr() 用一个范围在 0～255整数作参数，返回一个对应的字符。

```python
实例：

a = chr(0x30)
b = chr(0x31)
c = chr(0x61)
d = chr(48)
e = chr(49)
f = chr(97)

print("a=%s, b=%s, c=%s, d=%s, e=%s, f=%s" % (a, b, c, d, e, f))
以上代码输出结果为：

a=0, b=1, c=a, d=0, e=1, f=a
```

* `class complex([real[, imag]])`
	* 返回值为 real + imag*1j 的复数，或将字符串或数字转换为复数。如果第一个形参是字符串，则它被解释为一个复数，并且函数调用时必须没有第二个形参。第二个形参不能是字符串。每个实参都可以是任意的数值类型（包括复数）。如果省略了 imag，则默认值为零，构造函数会像 int 和 float 一样进行数值转换。如果两个实参都省略，则返回 0j。
	* 注解：当从字符串转换时，字符串在 + 或 - 的周围必须不能有空格。例如 complex('1+2j') 是合法的，但 complex('1 + 2j') 会触发 ValueError 异常。
	* 数字类型 —-int, float, complex 描述了复数类型。
	* 参数：
		* real — int, long, float或字符串；
		* imag — int, long, float；
```python
实例：

a = complex(1, 2)
b = complex(1)
c = complex("1")
d = complex("1+2j")

print("a=%s, b=%s, c=%s, d=%s" % (a, b, c, d))
以上代码输出结果为：

a=(1+2j), b=(1+0j), c=(1+0j), d=(1+2j)
```
* `class dict(kwarg)`
`class dict(mapping, kwarg)`
`class dict(iterable, **kwarg)`
	* 创建一个新字典。dict对象是dictionary类。有关此类的文档，请参阅dict和映射类型dict。

```python
#实例：
dict(a=1,b=2,c=3)
dict(zip(['a','b','c'],[1,2,3]))
dict([('a',1),('b',2),('c',3)])

#以上代码输出结果为：
{'a': 1, 'b': 2, 'c': 3}
{'a': 1, 'b': 2, 'c': 3}
{'a': 1, 'b': 2, 'c': 3}

```
* `enumerate(iterable, start=0)`
	* 返回一个枚举对象。iterable 必须是一个序列，或 iterator，或其他支持迭代的对象。 enumerate() 返回的迭代器的 `__next __() `方法返回一个元组，里面包含一个计数值（从 start 开始，默认为 0）和通过迭代 iterable 获得的值。

```python
实例：

seasons = ["C", "C++", "Java", "Python", "Go"]
print(list(enumerate(seasons)))

print(list(enumerate(seasons, start=1)))  # start=1：小标从1开始
以上代码输出结果为：

[(0, 'C'), (1, 'C++'), (2, 'Java'), (3, 'Python'), (4, 'Go')]
[(1, 'C'), (2, 'C++'), (3, 'Java'), (4, 'Python'), (5, 'Go')]
for循环实例：

seasons = ["C", "C++", "Java", "Python", "Go"]

for i, element in enumerate(seasons):
    print(i, seasons[i])
以上代码输出结果为：

0 C
1 C++
2 Java
3 Python
4 Go
````

* `class float([x])`
	* 返回从数字或字符串 x 生成的浮点数。
	* 如果实参是字符串，则它必须是包含十进制数字的字符串，字符串前面可以有符号，之前也可以有空格。可选的符号有 ‘+’ 和 ‘-‘ ； ‘+’ 对创建的值没有影响。实参也可以是 NaN（非数字）、正负无穷大的字符串。确切地说，除去首尾的空格后，输入必须遵循以下语法：
&emsp;&emsp;sign           ::=  "+" | "-"
&emsp;&emsp;infinity       ::=  "Infinity" | "inf"
&emsp;&emsp;nan            ::=  "nan"
&emsp;&emsp;numeric_value  ::=  floatnumber | infinity | nan
&emsp;&emsp;numeric_string ::=  [sign] numeric_value
这里， floatnumber 是 Python 浮点数的字符串形式，详见 浮点数字面值。字母大小写都可以，例如，“inf”、“Inf”、“INFINITY”、“iNfINity” 都可以表示正无穷大。
	* 另一方面，如果实参是整数或浮点数，则返回具有相同值（在 Python 浮点精度范围内）的浮点数。如果实参在 Python 浮点精度范围外，则会触发 OverflowError。
	* 对于一般的 Python 对象 x ， float(x) 指派给 x.float() 。
	* 如果没有实参，则返回 0.0 。

```python
实例：

print(float(1))
print(float(112))
print(float(-3.14))
print(float("123"))  # 字符串
以上代码输出结果为：

1.0
112.0
-3.14
123.0

```
------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《菜鸟教程》](http://www.runoob.com/ "点击跳转")、[《易百教程》](https://www.yiibai.com/ "点击跳转")、[《python3 官方文档》](https://www.python.org/doc/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

&emsp;&emsp;*注意：此教程基于python3.5*

------
