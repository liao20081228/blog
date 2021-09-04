---
title: cscope教程
tags: vim插件
---

------

***<font color=blue>版权声明</font>：本文参考了<font color=blue>《cscope man文档 v15.9》。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------



# 1 简介
&emsp;&emsp;cscope 是一个 C语言的浏览工具，通过这个工具可以很方便地找到某个函数或变量的定义位置、被调用的位置等信息。目前支持 C 和 C++。cscope 自身带一个基于文本的用户界面，不过 gvim 提供了cscope接口，因此可以在 gvim 中调用 cscope，方便快捷地浏览源代码。

&emsp;&emsp;cscope是一个类似ctags的工具。 你可以把它想作是超过频的ctags，因为它功能比ctags强大很多。 在 Vim里，通过cscope查询结果来跳转就象跳转到其他的标签完全一样；它被保存在标签栈里。这样你就可以象使用tags一样在函数等等之间便捷的跳转。在VIM中使用cscope非常简单，首先调用`cscope add cscope.out`命令添加一个cscope数据库，然后就可以调用`cscope find`命令进行查找了。VIM支持8种cscope的查询功能，然后列出目标出现的所有位置。我们还可以进行字符串查找，它会在双引号或单引号括起来的内容中查找。还可以输入一个正则表达式，这类似于egrep程序的功能。

# 2 安装
* 手动编译安装
	* 从<http://cscope.sourceforge.net/>下载源码
	* 解压后进入源码根目录
	* 配置：`./configure --with-flex ` (注：如果平台是Linux,最好带上 --with-flex选项)
	* 编译：`make ` (注：我没有遇到错误)
	* 安装：`make install `
* 下载已编译的二进制包
	* `sudo apt install cscope` 

>**注意**：cscope即可以单独使用，也可以在作为vim的一个插件来使用，但要求vim编译时配置了`--enable-cscepe`特性。可以使用`$ vim --version |grep cscepe`来查看是否支持。

# 3 命令详解
## 3.1  概要和描述
**命令** ：cscope [-bCcdehkLlqRTUuVvX] [-Fsymfile] [-freffile] [-Iincdir] [-inamefile] [-0123456789pattern] [-pn] [-sdir] [files]

**描述**：
cscope是一个交互式的，面向屏幕的工具，允许用户浏览C源文件以获取指定的代码元素。

默认情况下，cscope检查当前目录中的C（.c和.h），lex（.l）和yacc（.y）源文件。可以在命令行上指明源文件。cscope会搜索头文件目录。 cscope使用交叉引用符号（默认情况下称为cscope.out）来定位文件中的函数，函数调用，宏，变量和预处理器符号。

cscope将在首次浏览程序源文件时构建交叉引用符号数据库。在后续调用中，仅当源文件已更改或源文件列表不同时，cscope才会重建交叉引用。重建交叉引用时，将从旧的交叉引用复制未更改文件的数据数据库，这使得重建速度比初始构建更快。

## 3.2  选项
* -I，-c，-k，-p，-q和-T选项也可以位于cscope.files文件中。

|短选项|描述
|:--|:--|
|**-b**|  只创建数据库，然后退出。否则进入查找界面
| -C |   搜索时忽略字母大小写。
|-c|在数据库中只使用ASCII字符，即不压缩数据。
|-d|不更新交叉索引数据库
|-e|禁止顺序编辑文件时，切换文件之间的[Ctrl]-e命令提示符。
|-F symfile|从symfile读取符号引用行。 （符号引用文件由>和>>创建，也可以使用\<命令读取，如下文“发出后续请求”所述）。）
|-f reffile|使用reffile作为交叉引用文件名，而不使用默认的“ cscope.out”。
|-I incdir|在incdir中查找（在\$INCDIR查找之前，\$INCDIR是头文件的标准位置，通常为/usr/include），以查找名称不是以“/''开头且未在命令行或在下面的namefile文件中指定的任何#include文件。（#include文件可以用双引号或尖括号指定。）除了当前目录（首先搜索）和标准列表（最后搜索）之外，还搜索incdir目录。如果出现多个-I出现，则按照目录在命令行中出现的顺序搜索目录。|
|-i namefile|浏览namefile中列出名称的所有源文件（文件名之间用空格，制表符或换行符分隔），而不浏览默认名称列表文件cscope.files。如果指定了此选项，则cscope会忽略命令行上出现的所有文件名。可以将参数namefile设置为”-''以接受来自标准输入的文件列表。namefile中包含空格的文件名必须用“双引号”引起来。在此类带引号的文件名中，任何双引号和反斜杠字符都必须由反斜杠转义。|
|**-k**|"Kernel Mode''，在创建数据库时禁用默认的头文件目录（/usr/ include）
| -L|与-n pattern选项一起使用时进行单次搜索，然后使用命令行模式进行输出
|-l|命令行模式
|**-n pattern**|进入指定的搜索模式，并搜索pattern。n为[0-9]
|**-P path**|在预先建立数据库中的相对文件名前加上路径path，这样不必更改工作目录到数据库所在目录。仅与-d联用有效
|-p n|显示最后n个文件路径而不是最后1个。0不显示文件名。
|**-q**|通过反向索引启用快速符号查找。除普通数据库外，cscope会创建2个文件（默认为`cscope.in.out`和`cscope.po.out`）。
|**-R**|在搜索源文件期间递归子目录。
|-s dir|在dir中查找其他源文件。 如果在命令行上提供源文件，则忽略此选项。
|-T|仅使用前八个字符匹配C符号。除`.`之外的长度大于8的正则表达式不会匹配任何标识符
|-U|检查文件时间戳。 即使没有更改文件，此选项也将更新数据库上的时间戳。
|-u|无条件地构建交叉引用数据库（假设所有文件都已更改）。
|-v|建立数据库时，可见详细过程|
|**-X**| 退出时删除cscope数据库和逆序数据库|
| files|要操作的文件名列表。
|-h|  帮助|
|-V| 查看版本号
|**长选项**|**描述**
|--help|等效于-h
|--version| 等效于-V




## 3.3  初始搜索请求
* 交叉索引数据库准备就绪后，cscope将显示以下菜单：
	* Find this C symbol——找到这个C符号
	* Find this global definition——找到此定义：
	* Find functions called by this function——查找此函数调用的函数：
	* Find functions calling this function——查找调用此函数的函数：
	* Find this text string——找到这个文字字符串： 
	* Change this text string——更改此文本字符串：
	* Find this egrep pattern——找到这个egrep pattern：
	* Find this file——找到这个文件：
	* Find files #including this file——查找文件#include文件
	* Find assignments to this symbol——查找此符号的赋值
* 反复按`<up>`或`<down>`键移动到所需的输入字段，键入要搜索的文本，然后按`<ENTER>`键搜索。如果搜索成功，则可以使用以下任何单字符命令：

## 3.4 发出后续请求
**搜索成功后可以使用下例命令**

|按键|描述|
|:--|:--|
| 0-9A-ZA-Z|编辑给定行号的文件。
|`<space>`|显示下一组匹配行。
|`<TAB>`|在菜单和匹配行列表之间交替显示
|`<up>`|移动到上一个菜单项（光标在菜单中）或移动到上一个匹配行（光标在匹配的行列表中。）
|`<Down>`|移动到下一个菜单项（光标在菜单中）或移动到下一个匹配的行（光标在匹配的行列表中。）
|+|显示下一组匹配。
|`-或CTRL+V`| 显示上一组匹配行。
|ctrl+e |按顺序编辑显示的文件。
| > |将显示的行列表写入文件。
|\>>|将显示的行列表添加到文件中。
|<|从由>或>>创建的数据库文件中读取行，就像-F选项一样。
|^|通过shell命令过滤所有行并显示结果，替换已存在的行。
|\| |将所有行传递给shell命令并原封不动的显示它们。


**在任何时候，可以使用这些命令**：

|按键|描述|
|:--|:--|
|`<ENTER>`|移至下一个输入字段。
|ctrl+n|移动到下一个输入字段。
|ctrl+p|移至上一个输入字段。
|ctrl+y/a|使用键入的最后一个文本进行搜索。
|ctrl+b|移至上一次输入的字段和搜索模式。
|ctrl+f|移动到下一次输入的字段和搜索模式。
|ctrl+c|在搜索时切换忽略/使用字母大小写。 
|ctrl+r|重建交叉引用数据库
|！|启动交互式shell（键入ctrl+d以返回到cscope）。
|ctrl+l|重绘屏幕。
|？|提供有关cscope命令的帮助信息。
|ctrl+d|退出cscope。


>注意：如果要搜索的文本的第一个字符与上述命令之一匹配，请先键入（反斜杠）以将其转义。

**用新文本替换旧文本**：

键入要被改变的文本后，cscope将提示输入新文本，然后显示包含旧文本的行。 使用这些单字符命令选择要更改的行

|按键|描述|
|:--|:--|
|0-9A-ZA-Z|标记或取消标记要更改的行。
|*|标记或取消标记要更改的所有显示出的行。
|`<space>`|显示下一组。
| +|显示下一组。
|- |显示上一组。
|ctrl+a|标记或取消标记要更改的所有行（包括还未显示的）。
|ctrl+d|更改标记的行并退出。
|`<Esc>`|退出而不更改标记的行。
|！|启动交互式shell（键入^d以返回到cscope）。
|ctrl+l|重绘屏幕。
|?|提供有关cscope命令的帮助信息。


> 特殊键：如果终端具有在vi中工作的箭头键，则可以使用它们在输入字段中移动。 向上箭头键可用于移动到上一个输入字段，而不是重复使用`<Tab>`键。如果您有`<CLEAR>`，`<NEXT>`或`<PREV>`键，它们将分别充当^l，+和 - 命令。



## 3.5 命令行接口
* -l选项允许您在面向屏幕的界面不会有用的情况在使用cscope，例如，来自另一个面向屏幕的程序。
* 当cscope准备输入以字段编号（从0开始计数）后立即跟随搜索模式的输入行时，它将提示>>，例如，`lmain`找到主函数的定义。
* 如果您只想要单个搜索，请使用-L和-num模式选项，
* 而不是-l选项，并且不会获得>>提示。
* 对于-l，cscope输出参考行数 cscope：2行
* 对于每个找到的索引，cscope输出`文件名，函数名，行号，行文本`，用空格隔开
* 请注意，与面向屏幕的界面不同，不会调用编辑器来显示单个引用。
* 您可以使用c命令在搜索时忽略/使用字母大小写。 
* 您可以使用r命令重建数据库。
* 当cscope检测到文件结束时，或者当输入行的第一个字符是“^d”或“q”时，它将退出。

## 3.6 环境变量

|环境变量|描述|
|:--|:--|
|CSCOPE_EDITOR|覆盖EDITOR和VIEWER变量。在cscope中使用不同于EDITOR/VIEWER变量指定的编辑器
|CSCOPE_LINEFLAG|编辑器行号的格式。cscope默认通过`editor +N file`来调用编辑器，`N`是编辑器应跳转到的行号。emacs和vim都使用此格式。如果您的编辑器需要不同的方式，请在此变量中指定此项，并使用“％s”作为行号的占位符。例如：`editor -＃103 file`，应将此变量设置为` -＃％s`。
|CSCOPE_LINEFLAG<br />_AFTER_FILE|如果需要在要编辑的文件名后使用行号选项调用编辑器，请将此变量设置为“yes”。例如：`editor file +＃number`
|EDITOR|首选编辑器，默认为vi。
|HOME|主目录，自动设置为登录。
|INCLUDEDIRS|以冒号分隔的目录列表，用于搜索#include文件。
|SHELL|首选shell，默认为sh。
|SOURCEDIRS|以冒号分隔的目录列表，用于搜索其他源文件。
|TERM|终端类型，必须是屏幕终端。
|TERMINFO|终端信息目录的完整路径名。
|TMPDIR|临时文件目录，默认为/var/tmp。
|VIEWER|首选文件显示程序（如less），它会覆盖EDITOR（见上文）。
|VPATH|以冒号分隔的目录列表，每个目录下面都有相同的目录结构。 如果设置了VPATH，cscope将搜索指定目录中的源文件; 如果未设置，则cscope仅在当前目录中搜索。

## 3.7 相关文件
|文件|描述|
|:--|:--|
|cscope.files|包含-I，-p，-q和-T选项的默认文件以及源文件列表（由-i选项覆盖）。
|cscope.out|交叉索引数据库（由-f选项覆盖），如果无法在当前目录中创建，则放在主目录中。
|cscope.in.out、cscope.po.out|默认的反向索引数据库（由-q选项建立）。如果使用-f选项重命名数据库文件（cscope.out）， 则会在加上.in和.po后缀
|INCDIR |#include文件的标准目录（通常为`/usr/include`）。

## 3.8 注意
cscope识别如下形式的函数定义：
```
fname blank ( args ) white arg_decs white {
```
fname 是 函数名

blank 是0或多个空格，水平制表符，垂直制表符，换页或回车，不包括换行符

args是不包含双引号或换行符的任何字符串

white 零个或多个空格，制表符，垂直制表符，换页符，回车符或换行符

arg_decs 是零个或多个参数声明（arg_decs可能包含注释和空格）

函数声明不必从行的开头开始。返回类型可以在函数名称之前； cscope仍会识别该声明。偏离此格式的函数定义将不会被cscope识别。

菜单选项“查找此函数调用的函数”的搜索输出的“函数”列：输入域 将仅显示该行中调用的第一个函数，即此函数：
```
 e()
{
                return (f() + g());
 }
```
cscope搜索界面如下：
```
Functions called by this function: e
```
显示如下：
```
File  Function Line
a.c      f     3       return(f() + g());
```
有时，由于#if语句中的花括号，可能无法识别函数定义或调用。类似地，变量的使用可能被错误地识别为定义。

在预处理程序语句之前的typedef名称将被错误地识别为全局定义，例如，
```c
LDFILE  *
#if AR16WR
```
预处理程序语句还可能阻止识别全局定义，例如，
```c
 char flag
#ifdef ALLOCATE_STORAGE
             = -1
#endif
        ;
```
函数内部的函数声明被错误地识别为函数调用，例如，
```c
 f()
{
                void g();
}
```		
被错误地识别为对g的调用。

cscope通过查找class关键字识别C ++类，但不识别结构体也是一个类，因此它不识别定义在结构体中的内联成员函数。 它还不希望typedef中的class关键字，因此它错误地将X识别为定义：
```c
 typedef class X  *  Y;
```
 
 它还不能识别运算符函数定义

```cpp
 Bool Feature::operator==(const Feature & other)
{
          ...
}
``` 
 它也不能识别带函数指针参数的函数定义：
 ```cpp
ParseTable::Recognize(int startState, char *pattern, int finishState, void (*FinalAction)(char *))
{
          ...
 }
``` 
 
 
 
------

***<font color=blue>版权声明</font>：本文参考了<font color=blue>《cscope man文档 v15.9》。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

