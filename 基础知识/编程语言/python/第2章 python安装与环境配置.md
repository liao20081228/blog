---
title:  第2章 python安装与环境配置
tags: python
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《菜鸟教程》](http://www.runoob.com/ "点击跳转")、[《易百教程》](https://www.yiibai.com/ "点击跳转")、[《python3 官方文档》](https://www.python.org/doc/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

&emsp;&emsp;*注意：此教程基于python3.5*

------

# 1 python相关网站
* Python官网：<https://www.python.org/>

* Python文档下载地址：<https://www.python.org/doc/>

* Python安装包下载地址:<https://www.python.org/downloads/>

* Python源码下载地址：<https://www.python.org/downloads/source/>


# 2 安装python
python支持很多平台，包括
* Unix (Solaris, Linux, FreeBSD, AIX, HP/UX, SunOS, IRIX, 等等。)
* Win 9x/NT/2000
* Macintosh (Intel, PPC, 68K)
* OS/2
* DOS (多个DOS版本)
* PalmOS
* Nokia 移动手机
* Windows CE
* Acorn/RISC OS
* BeOS
* Amiga
* VMS/OpenVMS
* QNX
* VxWorks
* Psion
* Python 同样可以移植到 Java 和 .NET 虚拟机上。

Python已经被移植在许多平台上（经过改动使它能够工作在不同平台上）。您需要下载适用于您使用平台的二进制代码，然后安装Python。

如果您平台的二进制代码是不可用的，你需要使用C编译器手动编译源代码。手编译的源代码，功能上有更多的选择性， 为python安装提供了更多的灵活性。

## 2.1 windows平台

*  下载python安装包，[Windows x86-64 executable installer](https://www.python.org/downloads/windows/)
*  双击 python-版本号-amd64.exe 可执行文件。
*  勾选“install launcher for all users”
*  勾选“add python to PATH”，否则安装完成后要设置环境变量。
*  选择“Cusomize installation”。
*  勾选要安装的功能，建议全选。
*  点击“next”，勾选需要的高级选项，设置安装路径。建议不要装在系统目录下。
*  点击“install”，开始安装安装完毕后，点击“close”。
*  在cmd窗口中输入python，测试是否安装成功。

## 2.2 Linux平台（Ubuntu）
### 2.2.1 从在线仓库安装
```
$sudo apt update
$sudo apt install python
$sudo apt install python3 
```
### 2.2.2 从源码安装
```sh
#安装依赖项
$sudo apt install build-essential checkinstall
$sudo apt install libreadline-gplv2-dev libncursesw5-dev libssl-dev libsqlite3-dev tk-dev libgdbm-dev libc6-dev libbz2-dev

#下载源码，也可使用浏览器下载
$wget http://www.python.org/ftp/python/3.6.0/Python-3.6.0.tar.xz

#解压缩
$tar xvf Python-3.6.0.tar.xz

#现在cd进入源目录
$cd Python-3.6.0/

#设置编译选项
$./configure

#编译，安装。有两种方式：
#第一种，不覆盖系统自带python
$sudo make altinstall  #altinstall选项跳过创建符号链接，所以/usr/bin/python仍然指向旧版本的Python，保证Ubuntu系统将不会中断。
#第二种，覆盖原来的python
$make
$make install

```
## 2.3 MAC平台
&emsp;&emsp;最近的Macs系统都自带有Python环境，你也可以在 <https://www.python.org/downloads/mac-osx/>下载最新版安装。


# 3 设置环境变量

&emsp;&emsp;环境变量中是由操作系统维护的一个命名的字符串。这些变量包含可用的命令行解释器和其他程序的信息。Unix或Windows中路径变量为PATH（UNIX区分大小写，Windows不区分大小写）。当可执行程序不在PATH中时，就需要设置环境变量PATH。

## 3.1.1 Unix/Linux平台
```sh
#在 csh shell
setenv PATH "$PATH:/usr/local/bin/python" 

#在 bash shell (Linux):
export PATH="$PATH:/usr/local/bin/python" 

#在 sh 或者 ksh shell: 
PATH="$PATH:/usr/local/bin/python" 

#注意: /usr/local/bin/python 是 Python 的安装目录。
```
## 3.1.2 windows平台

* 第一种方式：在命令行输入`path=%path%;python安装路径`
* 第二种方式：在GUI中设置
 * 右键点击"计算机"，然后点击"属性"
 * 然后点击"高级系统设置"
 * 选择"系统变量"窗口下面的"Path",双击即可！
 * 然后在"Path"行，添加python安装路径，路径直接用分号";"隔开
 



# 4 运行python

## 4.1 交互命令或脚本
```sh
$ python  [opntion]      #Unix/Linux 
>>>command

#或者 

$ python [opntion]  script.py   #Unix/Linux 

#或者
./script.py

#注意：在直接执行脚本时，请检查脚本是否有可执行权限。
```
## 4.2 集成开发环境
* PyCharm——PyCharm 是由 JetBrains 打造的一款 Python IDE，支持 macOS、 Windows、 Linux 系统。PyCharm 功能 : 调试、语法高亮、Project管理、代码跳转、智能提示、自动完成、单元测试、版本控制……PyCharm 下载地址 : <https://www.jetbrains.com/pycharm/download/>


* IDLE——IDLE是第一个用于Python的Unix IDE。

* PythonWin——PythonWin是Python的第一个Windows图形用户界面，是具有GUI的IDE。

* Macintosh——PythonWin是Mac版本的Python的IDLE。





# 5 man手册
## 5.1 概述
```sh
python [ -B ] [ -b ] [ -d ] [ -E ] [ -h ] [ -i ] [ -I ]
              [ -m module-name ] [ -q ] [ -O ] [ -OO ] [ -s ] [ -S ] [ -u ]
              [ -v ] [ -V ] [ -W argument ] [ -x ] [ [ -X option ] -?  ]
              [ -c command | script | - ] [ arguments ]
```

Python是一种解释性的，交互式的，面向对象的编程语言，它将非凡的功能与非常清晰的语法相结合 有关Python编程的介绍，请参阅Python教程。 Python Library Reference记录了内置和标准类型，常量，函数和模块。 最后，Python参考手册描述了核心语言的语法和语义（可能也是如此）。 （这些文件可以通过下面的INTERNET资源找到;它们也可以安装在您的系统上。）

Python的基本功能可以使用您自己的用C或C ++编写的模块进行扩展。 在大多数系统上，这些模块可以动态加载。 Python也可以作为现有应用程序的扩展语言。 有关提示，请参阅内部文档。

 可以通过运行pydoc程序查看已安装的Python模块和软件包的文档。

## 5.1 命令行选项
|选项|作用|
|--|--|
|-B|不要在导入时写入 .py \[co\]文件。另见PYTHONDONTWRITEBYTECODE。|
|-b|发出有关str（bytes_instance），str（bytearray_instance）以及将bytes / bytearray与str进行比较的警告。（-bb：问题错误）|
|-c command|指定要执行的命令（请参阅下一节）。这将终止选项列表（后续的选项作为参数传递给命令）。
|-d	|在解析时显示调试信息（仅适用于向导，具体取决于编译选项）。
|-E|忽略修改解释器行为的环境变量，如PYTHONPATH和PYTHONHOME。
| -h,-?,--help| 打印帮助并退出。|
|-i|当脚本作为第一个参数传递或使用-c选项时，请在执行脚本或命令后进入交互模式。它不会读取$PYTHONSTARTUP文件。当脚本引发异常时，这对于检查全局变量或堆栈跟踪非常有用。|
|-I|以隔离模式运行Python。这也意味着-E和-s。在隔离模式下，sys.path既不包含脚本的目录，也不包含用户的site-packages目录。所有PYTHON* 环境变量也会被忽略。可以施加进一步的限制以防止用户注入恶意代码。
|-m module-name|在sys.path中搜索指定的模块，并将相应的.py文件作为脚本运行
|-O	|启用基本优化。这会将已编译（字节码）文件的文件扩展名从.pyc更改为.pyo。给定两次，导致文档字符串被丢弃。
|-OO|除了-O优化之外，还丢弃文档字符串。|
|-q|不打印版本和版权信息。 在非交互模式下也会抑制这些消息。
|-R|打开“哈希随机化”，以便str，bytes和datetime对象的hash（）值被“加盐”，并带有不可预测的伪随机值。尽管它们在单个Python进程中保持不变，但在重复调用Python之间无法预测它们。
|-s|    不要将用户站点目录添加到sys.path。|
|-S	|禁用模块站点的导入以及它所需的sys.path的依赖于站点的操作。如果稍后显式导入站点，也要禁用这些操作。
|-u|强制stdout和stderr的二进制I/O层无缓冲。 stdin总是被缓冲。文本I/O层仍将是行缓冲的。
|-v|每次初始化模块时打印一条消息，显示加载模块的位置（文件名或内置模块）。给出两次后，为搜索模块时检查的每个文件打印一条消息。还提供退出时模块清理的信息。
|-V|	输出Python版本号
|-W argument|&emsp;&emsp;警告控制。 Python有时会向sys.stderr输出警告消息。典型的警告消息具有以下形式：file：line：category：message。默认情况下，每个警告都会针对发生它的每个源代码行打印一次。此选项控制警告的打印频率。可以给出多个-W选项;当警告与多个选项匹配时，将执行最后一个匹配选项的操作。忽略无效的-W选项（在发出第一个警告时打印有关无效选项的警告消息）。也可以使用警告模块在Python程序中控制警告。<br>  &emsp;&emsp;最简单的参数形式是以下操作字符串之一（或唯一的缩写）：ignore忽略所有警告;default 显式请求默认行为（每个源代码行打印一次警告）; all 每次发生时都会打印一个警告（如果针对同一源重复触发警告，这可能会生成许多消息，例如在循环内）;module仅在每个模块中第一次出现时发出警告;once仅在程序第一次出现时打印每个警告; error引发异常而不是打印警告消息。<br>  &emsp;&emsp;完整形式的参数是action：message：category：module：line。action如上所诉，但只适用于与剩余字段匹配的消息。空字段匹配所有值。尾随的空字段可以省略。message字段与打印的警告消息的开头匹配，不区分大小写。category字段与警告类别匹配，它测试消息的实际警告类别是否是指定警告类别的子类，这必须是一个完整的类名称。module与（完全限定的）模块名称匹配，区分大小写。  line字段与行号匹配，其中0匹配所有行号，等同于省略行号。
|-X option|设置实现特定选项。
|-X|跳过源的第一行。 这仅适用于DOS特定的hack。 警告：错误消息中的行号将被关闭！



## 5.2 解释器接口
**解释器接口类似于UNIX shell**：当使用连接到tty设备的标准输入调用时，它会提示输入命令并执行它们直到读取EOF;当使用文件名参数或文件作为标准输入调用时，它从该文件读取并执行脚本;当使用-c命令调用时，它执行作为命令给出的Python语句。这里的命令可能包含由换行符分隔的多个语句。在Python语句中，前导空格是微不足道的！在非交互模式下，整个输入在执行之前进行解析。

如果可用，脚本名称和之后的其他参数将传递给脚本中的Python变量sys.argv中，该变量是一个字符串列表（必须首先导入sys才能访问它）。如果没有给出脚本名称，则sys.argv [0]为空字符串;如果使用-c，则sys.argv [0]包含字符串'-c'。请注意，**Python解释器本身的选项不会放在sys.argv中**。

在交互模式下，主要提示符是“>>>”；次要提示符（在命令未完成时出现）是“...”。可以通过分配给sys.ps1或sys.ps2来更改提示。解释器在提示后读取到EOF时退出。发生未处理的异常时，将打印堆栈跟踪并且控制权返回到主要提示符;在非交互模式下，解释器在打印堆栈跟踪后退出。中断信号引发键盘中断异常；其他UNIX信号不会被捕获（除了有时会忽略SIGPIPE，支持IOError异常）。错误消息将写入stderr。


## 5.3 文件和目录

根据本地的安装惯例，这些可能会有所不同; ${prefix} 和 ${exec_prefix}是依赖于安装的，应该解释为GNU软件;它们可能是一样的。在Debian GNU / {Hurd，Linux}上，两者的默认值是/usr。
```sh
${exec_prefix}/bin/python  #解释器的推荐位置。
	   
${prefix}/lib/python<version>
${exec_prefix}/lib/python<version> #包含标准模块的目录的推荐位置。

${prefix}/include/python<version>
${exec_prefix}/include/python<version> #包含开发Python扩展和嵌入解释器所需的包含文件的目录的推荐位置。

```
## 5.4  环境变量
|环境变量|作用|
|--|--|
|PYTHONHOME|标准Python库的位置。当$ PYTHONHOME设置为单个目录时，其值将替换$ {prefix}和$ {exec_prefix}。要指定不同的值，请将$ PYTHONHOME设置为$ {prefix}：$ {exec_prefix}。|
|PYTHONPATH| 模块文件的默认搜索路径。格式与shell的$ PATH相同：一个或多个以冒号分隔的目录路径名。默认忽略不存在的目录。默认搜索路径取决于安装，但通常以$ {prefix} / lib / python < version>开头（参见上面的PYTHONHOME）。默认搜索路径始终附加到$ PYTHONPATH。如果给出了脚本参数，则包含该脚本的目录将插入$ PYTHONPATH前面的路径中。搜索路径可以在Python程序中作为变量sys.path进行操作。|
|PYTHONSTARTUP|	如果这是可读文件的名称，则在以交互模式显示第一个提示之前，将执行该文件中的Python命令。 该文件在执行交互式命令的同一名称空间中执行，以便在交互式会话中无需限定地使用在该文件中定义或导入的对象。 您还可以更改此文件中的提示sys.ps1和sys.ps2。
|PYTHONOPTIMIZE|如果将其设置为非空字符串，则等同于指定-O选项。如果设置为整数，则相当于多次指定-O。
|PYTHONDEBUG| 如果将其设置为非空字符串，则等同于指定-d选项。 如果设置为整数，则相当于多次指定-d。|
|PYTHONDONTWRITEBYTECODE| 如果将其设置为非空字符串，则相当于指定-B选项（不要尝试编写.py [co]文件）。|
|  PYTHONINSPECT| 如果将其设置为非空字符串，则等同于指定-i选项。|
|PYTHONIOENCODING|  如果在运行解释器之前设置了它，它将覆盖用于stdin / stdout / stderr的编码，语法为encodingname：errorhandler。errorhandler是可选的，与str.encode具有相同的含义。对于stderr，错误处理程序部分被忽略，处理程序将始终是'反斜杠替换'。
|PYTHONNOUSERSITE|如果将其设置为非空字符串，则等效于指定-s选项（不要将用户站点目录添加到sys.path）。
| PYTHONUNBUFFERED|如果将其设置为非空字符串，则等同于指定-u选项。
|PYTHONVERBOSE| 如果将其设置为非空字符串，则等同于指定-v选项。 如果设置为整数，则相当于多次指定-v。
|PYTHONWARNINGS| 如果将其设置为以逗号分隔的字符串，则等效于为每个单独的值指定-W选项。
|PYTHONHASHSEED| 如果此变量设置为“random”，则使用随机值为str，bytes和datetime对象的哈希值设定种子。如果将PYTHONHASHSEED设置为整数值，则将其用作固定种子，其目的是允许可重复的散列，例如用于解释器本身的自我测试，或允许一组python进程共享散列值，整数必须是[0,4294967295]范围内的十进制数。 指定值0将禁用散列随机化。
|PYTHONCASEOK|	加入PYTHONCASEOK的环境变量, 就会使python导入模块的时候不区分大小写.



------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《菜鸟教程》](http://www.runoob.com/ "点击跳转")、[《易百教程》](https://www.yiibai.com/ "点击跳转")、[《python3 官方文档》](https://www.python.org/doc/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

&emsp;&emsp;*注意：此教程基于python3.5*

------