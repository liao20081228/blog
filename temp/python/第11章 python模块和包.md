---
title:  第11章 python模块和包
tags: python
---

------


&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《菜鸟教程》](http://www.runoob.com/ "点击跳转")、[《易百教程》](https://www.yiibai.com/ "点击跳转")、[《python3 官方文档》](https://www.python.org/doc/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

&emsp;&emsp;*注意：此教程基于python3.5*

------



# 1 模块
## 1.1 模块的概念
* Python 模块(Module)，是一个 Python 文件，以 .py 结尾，也是一个Python对象，可以绑定和引用。
* 模块能够有逻辑地组织 Python 代码段。
* 把相关的代码分配到一个模块里能让你的代码更好用，更易懂。
* 模块能定义函数，类和变量，模块里也能包含可执行的代码。

## 2.2 模块的引入
* 引入模块，导入整个模块，含私有对象和内部对象
	* `import module1[, module2[,... moduleN] `
	* 通过`module.name`来使用。
	* 一个模块只会被导入一次，不管你执行了多少次import。
* 引入模块的部分，导入指定对象，可以是私有对象（\_\_var）和内部对象（\_var）
	* `from modname import name1[, name2[, ... nameN]]` 或 `import module.name`
	* 直接使用name
* 引入模块的所有，但不包含私有对象（双下划线开头）和内部对象（下划线开头）
	* `from modname import *`
* 引入模块并重命名
	* `import module as alias  `
	* `from modname import name as  alias`


## 2.3 当前执行脚本
* 在模块中，模块的名称(作为字符串)可用作全局变量`__name__`的值。当其值是`__main__`时，表明该模块自身在运行，否则是被引入。
* 执行模块时，\_\_name_\_设置为“\_\_main\_\_”。
* 如果我们想在模块被引入时，模块中的某一程序块不执行，我们可以用__name__属性来使该程序块仅在该模块自身运行时执行。

```python
#!/usr/bin/python3

def fun(x):
"阶乘"
   if x==0:
        return 1
   else:
        return x * fun(x-1)

if __name__ == "__main__":
   f = fun(100)
   print(f)
   
``` 
* 模块内置属性：


## 2.4 搜索路径
* 当你导入一个模块，Python 解析器对模块位置的搜索顺序是：
	1. 当前目录
	 2. 如果不在当前目录，Python 则搜索在 shell 变量 PYTHONPATH 下的每个目录。
	 3. 如果都找不到，Python会察看默认路径。UNIX下，默认路径一般为/usr/local/lib/python/。
* 模块搜索路径存储在 system 模块的 sys.path变量中。变量里包含当前目录、PYTHONPATH、由安装过程决定的默认目录。

## 2.5 dir()函数
* dir() 函数返回一个排好序的字符串列表，内容是一个模块里定义过模块，变量和函数的名字。
* 特殊字符串变量：
	* `__name__`：模块名字
	* `__doc__`：模块文档
	* `__package__`：模块所在包
	* `__file__`：模块文件名
	*  `__loader__`：是由加载器在导入的模块上设置的属性,访问它时将会返回加载器对象本身。
	*   `__cached__`：表示缓存文件 python内部的优化机制，当导入别的模块时，会直接将生成的字节码文件导入，省略了python源文件到字节码文件的转换
	*  `__spec__`：模块规格说明
## 2.6 reload() 函数
* 当一个模块被导入到一个脚本，模块顶层部分的代码只会被导入一次。
* 因此，如果你想重新导入模块里顶层部分的代码，可以用 reload() 函数。
* 语法： `reload(module_name)`，module_name要直接放模块的名字，而不是一个字符串形式。


# 4 包
* 包是一个分层次的文件目录结构，它定义了一个由模块及子包，和子包下的子包等组成的 Python 的应用环境。
* 简单来说，包就是文件夹，但该文件夹下必须存在 \_\_init\_\_.py 文件,该文件的内容可以为空。\_\_int\_\_.py用于标识当前文件夹是一个包。
* 导入一个包的时候，Python 会根据 sys.path 中的目录来寻找这个包中包含的子目录。
* 引入包中的模块
	* 导入包中指定模块：
		* `from package import module` 
		* `import package.module`
	* 导入包中所有模块：	 `from package import *` 
	* 导入包中指定模块的指定对象：
		* `from package.module import name`
		* `from package.module import *`
* \_\_init\_\_.py文件中的两个属性
	* `__all__`：这是一个模块列表，指定当时用`from package import *`要导入的模块。
	* `__apth__`。这是一个目录列表，里面每一个包含的目录都有为这个包服务的\_\_init\_\_.py，你得在其他\_\_init\_\_.py被执行前定义哦。可以修改这个变量，用来影响包含在包里面的模块和子包。


```python
sound/                          顶层包
      __init__.py               初始化 sound 包
      formats/                  文件格式转换子包
              __init__.py
              wavread.py
              wavwrite.py
              aiffread.py
              aiffwrite.py
              auread.py
              auwrite.py
              ...
      effects/                  声音效果子包
              __init__.py
              echo.py
              surround.py
              reverse.py
              ...
      filters/                  filters 子包
              __init__.py
              equalizer.py
              vocoder.py
              karaoke.py
              ...
```

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《菜鸟教程》](http://www.runoob.com/ "点击跳转")、[《易百教程》](https://www.yiibai.com/ "点击跳转")、[《python3 官方文档》](https://www.python.org/doc/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

&emsp;&emsp;*注意：此教程基于python3.5*

------
