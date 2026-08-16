---
title: 第4章 python基本数据类型
tags: python
---

------


&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《菜鸟教程》](http://www.runoob.com/ "点击跳转")、[《易百教程》](https://www.yiibai.com/ "点击跳转")、[《python3 官方文档》](https://www.python.org/doc/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

&emsp;&emsp;*注意：此教程基于python3.5*

------

<style>table{word-break:initial;}</style>


# 1 内置类型概述 

下面几节描述内置在解释器中的标准类型。

主要的内置类型是**数字，序列（列表、元组、范围、字符串），集合，映射，类，实例和异常**。


一些集合类（collection class）是可变的。 在适当的位置添加，减去或重新排列其成员且不返回特定项的方法永远不会返回集合实例本身，而是返回None。

&emsp;&emsp;一些操作由几种对象类型都支持，实际上**所有对象都可以进行比较，测试真值，判断身份，并转换为字符串**（使用repr（）函数或略有不同的str（）函数）。当print（）函数输出对象时，后一个函数会隐式使用。

有些操作由多种对象类型支持;特别地，几乎所有对象都可以比较、测试真值，并转换为字符串(使用repr()函数或略有不同的str()函数)。 后一个函数是在对象由 print() 函数输出时被隐式地调用的。

# 2 逻辑值测试与布尔类型
* 布尔类型有两个常量：True 和 False
 * 测试为True：
     * 常量:True、NotImplemented
     * 非零数字
     * 非空序列和集合
     * 默认情况下，一个对象被认为是真的，除非它的类定义了返回False的\__bool __（）方法或者返回零的\__len __（）方法。

 * 测试为False：
     * 常量：None和False。
     * 数值零：0，0.0，0j， Decimal(0), Fraction(0, 1)
     * 空序列和空集合：''，（），[]，{}，set（），range（0）
     * 除非另有说明，否则具有布尔结果的操作和内置函数始终返回0或False表示false，1或True表示true。 

# 3 数值类型
* int（有符号整型）
	* long（长整型）——Python使用L来显示长整型。python3.x用int代替该类型。
* float（浮点型）——可用小数或者科学计数法
* complex（复数）——用 a + bj，或者 complex(a,b) 表示， 复数的实部 a 和虚部 b 都是浮点型。



* 数字类型是不可变，对数字类型的变量的修改将会创建一个新的对象，然后将地址赋值给原来的变量
* 数学运算常用的函数基本都在 math 、cmath 、random模块中。
  * math 模块提供了许多对浮点数的数学运算函数。
  * cmath 模块包含了一些用于复数运算的函数。
  * random模块是产生随机数。
* 数值类型（除复数外）都支持算术运算符、比较运算符、身份运算符，逻辑运算符，整数支持位运算符

## 3.1 内建函数
|内建函数|	返回值 ( 描述 )|
|--|--|
|`abs(x)`	|返回数字的绝对值。
|`cmp(x, y)`|	如果 x < y 返回 -1, 如果 x == y 返回 0, 如果 x > y 返回 1，
|`max(x1, x2,...)`|返回给定参数的最大值。参数可以为序列。
|`min(x1, x2,...)`|返回给定参数的最小值。参数可以为序列。
|`pow(x, y[, z])`|返回x的y次方，如果z存在，则再对结果进行取模，其结果等效于pow(x,y) %z，把参数作为整型
|`round(x [,n])`|	返回浮点数x的四舍五入值，如给出n值，则代表舍入到小数点后n位。
|`int(x)`	|将x转为整数
|`float(x)`|将x转为浮点数
|`complex(re, im)`|创建一个复数real+imag（j）
|`divmod(x, y)`|	返回元组(x // y, x % y)	
|`pow(x, y)`|	x ** y
|`hex（x）`|将x转为16进制
|`oct(x)`|将x转为8进制
##3.2 内建方法
|实例方法| 描述|版本支持
|--|--|
|`int_var.bit_length()`|返回int的二进制的位数，不包括符号和前导零：|3.1
|`int_var.to_bytes(length, byteorder, *, signed=False)`|返回长length字节的整数，字节序为byteorder，signed补码标志|3.2
|`float.as_integer_ratio()`|返回一对整数，其比值与浮点数相等，分母为正整数
|`float.is_integer()`|小数部分等于0返回True，否则返回 False
|`float.hex()`|返回16进制字符串，p表示底数为10，指数用十进制表示
|**类方法**|**描述**|**版本支持**
|` int.from_bytes(bytes, byteorder,*,signed=False)`|将字节序为byteorder的bytes转为整数，signed补码标志|3.2
|` float.fromhex(s)`|把16进制字符串转为浮点数，

##1.2 List
* 列表用[ \]标识。 元素之间用逗号隔开。
* 列表是一种序列。
* 列表支持字符，数字，字符串，列表嵌套。
* 列表中的元素不一定是同类型。
* `[]`——表示空列表，不占内存空间。
* `[None]`——表示空列表，占内存空间。
*  `List_var[index]` ——访问某个元素。
 * 从左到右索引默认0开始的，最大范围是字符串长度少1
 * 从右到左索引默认-1开始的，最大范围是字符串开头
* `List_var[头下标:尾下标:步长]` ——截取子列表，包括头下标，不包括尾下标
 * 从左到右索引默认 0 开始，从右到左索引默认 -1 开始。
 * 头下标缺省为0，尾下标缺省列表长加1，头下标与尾下标符号相同。
 * 步长缺省为1，为正数从左到右提取元素，为负数，从右到左提取元素。
* `list_var[index]=val` 或 ` list_var[头下标:尾下标:步长]=list1`——列表二次赋值。
* `del list_var[index]` 或 `del list_var[头下标:尾下标:步长]`——删除列表元素。
* 列表支持成员运算符`in`和 `not in`，
* 列表支持身份运算符 `is`和 `not is`
* 列表支持连接运算符 `+`，如`list1+list2`
* 列表支持重复运算符 `*`，如`list * n`
* 列表比较规则
  * 如果比较的元素是同类型的，则比较其值，返回结果。
  * 如果两个元素不是同一种类型，则检查他们是否是数字。
      * 如果是数字，执行必要的数字强制类型转换，然后比较。
      * 如果有一方的元素是数字，则另一方的元素“大”(数字是“最小的”)。
      * 否则，通过类型名字的字母顺序进行比较。
  * 如果有一个列表首先到达末尾，则另一个长一点的列表“大”。
  * 如果我们用尽了两个列表的元素而且所有的元素都是相等的，那么结果就是相等，返回一个0。
###1.2.1 内建函数和方法
|函数|描述|
|--|--|
|cmp(list1, list2)|比较两个列表的元素，小于返回负数，等于返回0，大于返回正数
|len(list)|列表元素个数
|max(list)|返回列表元素最大值
|min(list)|返回列表元素最小值

|方法| 描述|
|--|--|
|list.append(obj)|在列表末尾添加新的对象
|list.count(obj)|统计某个元素在列表中出现的次数
|list.extend(seq)|在列表末尾一次性追加另一个序列中的多个值（用新列表扩展原来的列表）
|list.index(obj)|从列表中找出某个值第一个匹配项的索引位置
|list.insert(index, obj)|将对象插入列表的index位置
|list.pop(obj=list[-1])|移除列表中的一个元素（默认最后一个元素），并返回该元素
|list.remove(obj)|移除列表中某个值的第一个匹配项
|list.reverse()|反向列表中元素
|list.sort(cmpfunc=cmp,key=None,reverse=False)|对原列表进行排序,cmp比较函数，key是带一个参数的函数，用于从每个元素中提取一个用于比较的关键字，reverse是否逆序



##1.3 Tuple
 * 元组是只读列表，区别在于：
   * 元组用（）标识
   * 元组没有append、extend、insert、pop、remove、remove、sort方法
   * 元组不支持del、二次赋值
     
 
 

##1.4 String
* string与元组类似。区别在于
 * string用单引号，双引号（推荐），三引号表示。
     * 三引号忽略转义字符、特殊字符、控制字符
     * 三引号位于行首时表示注释
     * 如果要使用Unicode字符串，需要在引号前加上u。python3默认Unicode，不需要此操作。
 * string的元素不用逗号隔开
 * string的元素都是字符
 * string增加了一些独有的方法
 * string支持r/R运算符

###1.4.1 转义字符
|转义字符|	描述|转义字符|	描述|
|--|--|--|--|
|\ |(在行尾时)	续行符|\\\	|反斜杠符号|
|\'	|单引号|\"	|双引号
|\a	|响铃|\b	|退格(Backspace)
|\e|	转义|\000|	空
|\n	|换行|\v	|纵向制表符
|\t	|横向制表符|\r	|回车
|\f|换页|\oyy|	八进制数，yy代表的字符，
|\xyy|	十六进制数，yy代表的字符，|\other|其它的字符以普通格式输出

###1.4.2 运算符
|运算符|	描述|
|--|--|--|
|+	|字符串连接	|
|*	|重复字符串 |
|[index]	|通过索引获取字符串中字符|
|[start:stop:step ]|截取字符串中的一部分|
|in	| 如果字符串中包含给定的字符返回 True	|
|not in|如果字符串中不包含给定的字符返回 True	|
|is	| 两个字符串引自同一个字符串，返回 True	|
|not is|两个字符串引自不同字符串，返回 True	|
|r或R|忽略字符串中的转义字符、特殊字符、不能打印字符。 |
|%|格式字符串

###1.2.3 格式化输出
####1.2.3.1 C风格格式化
|符号|	描述|  符号|	描述|
|--|--| --|--|
| %c|	 字符及其ASCII码 |   %s|	 字符串
| %d|	整数 |    %u|	 无符号整型
| %o|	无符号八进制数 |    %x|	 无符号十六进制数
| %X|	 无符号十六进制数（大写）|    %f|	浮点数字，可指定小数点后的精度
| %e|	科学计数法|    %E|	 作用同%e，科学计数法
| %g|	 %f和%e的简写 |    %G|	 %f 和 %E 的简写
| %p|	 十六进制表示的变量地址

&emsp;&emsp;格式化操作符辅助指令:

|符号|	功能|
|--|--| --|--|
|*	|定义宽度或者小数点精度|
|-	|用做左对齐|
|+	|在正数前面显示加号( + )
|`<sp>`|	在正数前面显示空格
|#	|在八进制数前面显示零('0')，在十六进制前面显示'0x'或者'0X'(取决于用的是'x'还是'X')
|0|	显示的数字前面填充'0'而不是默认的空格
|%	|'%%'输出一个单一的'%'
|(var)|	映射变量(字典参数)
|m.n|	m 是显示的最小总宽度,n 是小数点后的位数(如果可用的话)

####1.2.3.2 format风格格式化

&emsp;&emsp;用{ }表示C风格的%，用冒号分隔参数和修饰格式
```python
"{arg[:format_spec]} ".format(var...)
```
* 参数设置
 * 不带参数，按默认顺序
`"{} {}".format(var1,var2)`
 * 带位置参数，按指定顺序
 `"{1}{0}{1}".format(var1,var2)`
  * 设置普通参数
`"{arg1}{arg2}".format(arg1=var1,arg2=var2)`
  * 通过列表索引设置参数
`"{0[index1]}{0[index2]}".format(list_var)`
  * 通过字典设置参数
`"{arg1}{arg2}".format(**dict_var)`
  * 通过对象设置参数
`"{0.attr1}{0.attr2}".format(obj)`

* 修饰格式：`[[fill]align][sign][#][0][width][grouping_option][.precision][type]`
 * fill——填充字符，可以是任何字符
 * align——对齐方式。可以是 "<" | ">"| "=" | "^"
 * sign—— 显示正负号。 可以是  "+" | "-" | " "
 * width——宽度，是一个整数。
 * grouping_option——分组显示方式。可以是  "_" | ","
 * precision——精度，是一个整数。
 * type—— 类型。可以是"b" | "c" | "d" | "e" | "E" | "f" | "F" | "g" | "G" | "n" | "o" | "s" | "x" | "X" | "%"

|修饰字符|作用|修饰字符|作用|
|--|--|--|--|
|<|左对齐|b|二进制|
|>|右对齐|d|十进制，整数|
|^|居中对齐|o|八进制|
|=|让填充字符显示在符号和数字中间|x|十六进制小写
|+|显示正负号|X|十六进制大写
|-|显示符号|f|浮点数，精度为6
|空格|正数显示空格，负数显示负号|F|浮点数，精度为6
|_|以下划线分隔数字|e|科学计数法，精度为6
|,|以逗号分隔数字|E|科学计数法，精度为6
|#|在八进制数前面显示零('0o')，在十六进制前面显示'0x'或者'0X'|g|自动选择f或e
|0|当没有指定填充字符，将填充字符设为0|G|自动选择F或E
|||n|等效于g或者d
|||c|字符
|||s|字符串
|||%|末尾显示百分号
|||None|等效于s或g或d





###1.2.4 内建函数和方法
|内建函数|	描述|
|--|--|
|cmp(str1, str2)|比较两个字符串 ，小于返回负数，等于返回0，大于返回正数
|len(str)|返回字符串 str 长度
|max(str)|返回字符串 str 中最大的字母。
|min(str)|返回字符串 str 中最小的字母。

|内建方法|描述|
|--|--|
|str.format()|格式化字符串
|str1.count(str2, beg=0,end=len(str1))|返回str2在str1的指定范围内出现的次数，beg缺省为0，end缺省为str1长度
|str.expandtabs(tabsize=8)|把字符串 str中的 tab 符号转为空格，tab符号默认的空格数是 8。
|str.join(seq)|以str作为连接符，将 seq 中所有的元素合并为一个新的字符串
|**成员判断**||
|str.isalnum()|如果 str至少有一个字符并且所有字符都是字母或数字则返回 True,否则返回 False
|str.isalpha()|如果 str 至少有一个字符并且所有字符都是字母则返回 True,否则返回 False
|str.isdigit()|如果 str 只包含数字则返回 True 否则返回 False.
|str.islower()|如果 str中包含至少一个区分大小写的字符，并且这些字符都是小写，则返回 True，否则返回 False
|str.isnumeric()|如果 str中只包含数字字符，则返回 True，否则返回 False
|str.isspace()|如果 str中只包含空格，则返回 True，否则返回 False.
|str.istitle()|如果 str是标题化的(见 title())则返回 True，否则返回 False
|str.isupper()|如果 str中包含至少一个区分大小写的字符，并且这些字符都是大写，则返回 True，否则返回 False
|str.isdecimal()|检查str是否只包含十进制字符。这种方法只存在于unicode对象。如果是返回 True，否则返回 False.
|**编码解码**||
|str.encode(encoding='UTF-8', errors='strict')|以 encoding 指定的编码格式编码 str。encoding缺省为utf-8， errors设置不同错误的处理方案。默认为 'strict',意为编码错误引起一个ValueError 异常，其他可能值有 'ignore', 'replace', 'xmlcharrefreplace', 'backslashreplace' 以及通过 codecs.register_error() 注册的任何值。
| str.decode(encoding='UTF-8', errors='strict')|以 encoding 指定的编码格式解码 str。encoding缺省为utf-8， errors设置不同错误的处理方案。默认为 'strict',意为编码错误引起一个UnicodeError异常，其他可能值有 'ignore', 'replace', 'xmlcharrefreplace', 'backslashreplace' 以及通过 codecs.register_error() 注册的任何值。
|**对齐填充**||
|str.ljust(width，fillchar=' ')|返回一个原字符串左对齐,并使用fillchar填充至长度 width 的新字符串，fillchar缺省为空格
|str.rjust(width，fillchar=' ')|返回一个原字符串右对齐,并使用fillchar填充至长度 width 的新字符串，fillchar缺省为空格
|str.center(width, fillchar=' ')|返回一个原字符串居中,并用fillchar填充至长度 width 的新字符串。fillchar缺省为空格。
|str.zfill(width)|返回长度为width的字符串，原字符串右对齐，前面填充0
|**大小写转换**||
|str.title()|返回"标题化"的str,就是说所有单词都是以大写开始，其余字母均为小写
|str.capitalize()|把字符串的第一个字符大写,其他字母变小写
|str.lower()|转换 str中所有大写字符为小写.
|str.upper()|转换 str中的小写字母为大写
|str.swapcase()|反转 str 中的大小写
|**查找**||
|str1.find(str2, [beg=0, end=len(string))|检测str2是否包含在str1指定范围中，如果是返回开始的索引值，否则返回-1。beg缺省为0，end缺省为str1长度。
|str1.rfind(str2, beg=0,end=len(str1) )|类似于 find()函数，不过是从右边开始查找.
|str1.index(str2, beg=0, end=len(string))|跟find()方法一样，只不过如果str2不在 str1中会报一个异常.
|str1.rindex( str2, beg=0,end=len(str1))|类似于 index()，不过是从右边开始.
|**删除边界字符**||
|str.strip(char=" ")|删除str 左边和右边 的字符char，缺省为空格
|str.lstrip(char=" ")|删除 str 左边的字符char，缺省为空格
|str.rstrip(char=" ")|删除str字符串末尾的字符char，缺省为空格.
|**分割**||
|str1.split(str2="", num=str1.count(str2))|以str为分隔符，分割 str1，如果num有指定值，则仅分隔 num 次，str2缺省为空。
|str.splitlines(keepends=False)|按照('\r', '\r\n', \n')分割str，参数 keepends 缺省为 False，不保留分隔符，如果为 True，则保留分隔符。
|str2.partition(str1)|从str1出现的第一个位置起,把字符串str2分 成 一 个 3 元 素 的 元 组 (str2_pre_str1,str1,str2_post_str1),如果str2 中不包含str1则 str2_pre_str1 == str2.
|str2.rpartition(str1)|类似于 partition()函数,不过是从右边开始查找.
|**判断边界字符**||
|string.startswith(obj, beg=0,end=len(string))|检查指定范围的字符串是否以obj 结束，如果是，返回 True,否则返回 False,。beg缺省为0，end缺省为str长度。
|str.endswith(obj, beg=0, end=len(str))|检查指定范围的字符串是否以obj 结束，如果是，返回 True,否则返回 False,。beg缺省为0，end缺省为str长度。
|**映射转化**||
|string.maketrans(intab, outtab])|创建字符映射的转换表，第一个参数是需要转换的字符，第二个参数是字符串表示转换的目标，需要一样长。
|str.translate(tranttab, del="")|根据tranttab给出的表转换 str 的字符,要过滤掉的字符放到 del 参数中，缺省为空
|**替换**|
|str.replace(str1, str2,  num=str.count(str1))|把 str 中的 str1 替换成 str2,如果 num 指定，则替换不超过num次.

###1.2.5 string模块
|字符常量|名称|	描述|
|--|--|--|
|whitespace|空白字符| ' \t\n\r\v\f'
|lowercase |小写字母| 'abcdefghijklmnopqrstuvwxyz'
|uppercase |大写字母|'ABCDEFGHIJKLMNOPQRSTUVWXYZ'
|letters |字母| lowercase + uppercase
|ascii_lowercase |ascii小写字母| lowercase
|ascii_uppercase |asdii大写字母| uppercase
|ascii_letters |ascii字母|ascii_lowercase + ascii_uppercase
|digits | 数字|'0123456789'
|hexdigits | 十进制数字|digits + 'abcdef' + 'ABCDEF'
|octdigits | 八进制数字|'01234567'
|punctuation | 标点|"""!"#$%&'()*+,-./:;<=>?@[\]^_`{|}~"""
|printable | 可打印字符|digits + letters + punctuation + whitespace

|方法|	描述 python3.x版本中已经没有这些函数了|
|--|--|
|expandtabs(s,tabsize=8)|把字符串 s中的 tab 符号转为空格，tab符号默认的空格数是 8。
|join(words,seq=' ')|将words用seq连接起来，缺省为空格
|count(str1，str2, beg=0,end=len(str1))|返回str2在str1的指定范围内出现的次数，beg缺省为0，end缺省为str1长度
|**字符转数值**|
|atoi(s,base=10)|字符串s转换成整型,base为进制，默认10
|atof(s|字符串s转换成浮点型
|atol(s,base=10)|字符串s转换成长整型,base为进制，默认10
|**大小写转换**||
|lower(s)|大写字母转为小写
|upper(s)|小写字母转为大写
|swapcase(s)|反转s中的大小写字母
|capwords(s,sep=None)|用split按sep分割s，用capitalize将各子串首字母大写，再用join以sep连接各子串
|capitalize(s)|把字符串的第一个字符大写,其他字母变小写
|**删除边界字符**||
|strip(s,char=None)|删除str 左边和右边 的字符char，缺省为空格
|lstrip(s,char=None)|删除 str 左边的字符char，缺省为空格
|rstrip(s,char=None)|删除str字符串末尾的字符char，缺省为空格.
|**分割**|
|split(s,seq=None,maxsplit=-1)|以seq为分隔符，分割 s，如果maxsplit有指定值，则仅分隔maxsplit 次，seq缺省为空。
|rsplit(s,seq=None,maxsplit=-1)|和split一样，只是从右边开始
|**查找**|
|find(s,sub,[beg=0, end=len(string))|检测sub是否包含在s指定范围中，如果是返回开始的索引值，否则返回-1。beg缺省为0，end缺省为str1长度。
|rfind(s,sub, beg=0,end=len(str1) )|类似于 find()函数，不过是从右边开始查找.
|index(s,sub, beg=0, end=len(string))|跟find()方法一样，只不过如果sub不在 s中会报一个异常.
|rindex( s,sub,beg=0,end=len(str1))|类似于 index()，不过是从右边开始.
|**对齐填充**||
|str.ljust(s，width，fillchar=' ')|返回一个原字符串左对齐,并使用fillchar填充至长度 width 的新字符串，fillchar缺省为空格
|str.rjust(s，width，fillchar=' ')|返回一个原字符串右对齐,并使用fillchar填充至长度 width 的新字符串，fillchar缺省为空格
|str.center(s，width, fillchar=' ')|返回一个原字符串居中,并用fillchar填充至长度 width 的新字符串。fillchar缺省为空格。
|str.zfill(s，width)|返回长度为width的字符串，原字符串右对齐，前面填充0
|**映射转换**||
|maketrans(fromstr,tostr）|创建字符映射的转换表，第一个参数是需要转换的字符，第二个参数是字符串表示转换的目标，需要一样长。
|translate(s,table,deletions="")|根据table给出的表转换s,要过滤掉的字符放到 del 参数中，缺省为空
|**替换**|
|replace(s,old,new,maxreplace=-1)|把 s中的 old替换成new,如果 maxreplace指定，则替换不超过maxreplace次.

##1.5 Dictionary
* 字典是无序对象的集合。
* 字典当中的元素是一个键值（key:value）对，键与值之间用冒号:分开。
* 字典用{ }标识。
* **键必须是唯一**的，但值则不必。
* 值可以取任何数据类型，但**键必须是不可变**的，如字符串，数字或元组。
* 支持集合运算符。
*  `dict_var[key]`——访问某个元素。
*  `dict_var[key]=value`——更改已有元素的值
*  `del dict_var[key]`——删除已有键/值对

###1.5.1 内置函数和方法

|内置函数|描述|
|--|--|
|	cmp(dict1, dict2)|比较两个字典元素。
|len(dict)|计算字典元素个数，即键的总数。

|**内置方法**|描述|
|--|--|
|dict.clear()|删除字典内所有元素
|dict.copy()|返回一个字典的浅复制
|dict.items()|返回可遍历的(键, 值) 元组的列表
|dict.iteritems()|返回返回字典的键值对迭代器。
|dict.keys()|以列表返回一个字典所有的键
|dict.iterkeys()|返回字典的基于键的迭代器。 
|dict.iter()|返回字典的基于键的迭代器。 这是iterkeys（）的别名
|dict.values()|以列表返回字典中的所有值
|dict.itervalues()|返回字典的基于值的迭代器。 
|dict.fromkeys(seq[, val])|创建一个新字典，以序列 seq 中元素做字典的键，val 为字典所有键对应的初始值
|dict.get(key, default=None)|返回指定键的值，如果键不在字典中返回default值
|dict.has_key(key)|如果键在字典dict里返回true，否则返回false
|dict.setdefault(key, default=None)|和get()类似,但如果键不存在于字典中，将会添加键并将值设为default
|dict.update(dict2)|把字典dict2的键值对添加到dict里，如果有相同键，则更新值
|pop(key[,default])|删除键key所对应的值，返回被删除的值。key值必须给出。 否则，返回default值。
|popitem()|返回随机删除的字典中的一对键值。




##1.6 数据类型转换
|函数|	描述|
|--|--|
|int(x ,base=10)|将x转整数，当x为字符串时需要指出基数，x为字符串或数字，base为进制，默认10
|long(x ,base=10)|将x转换长整数，当x为字符串时需要指出基数，x为字符串或数字，base为进制，默认10
|float(x)|将x转为浮点数，x为字符串或数
|complex([real [,imag]])|创建一个复数real+imag（j），real为int, long, float或字符串；imag为int, long, float；
|str(x='')|将对象 x 转换为适合人阅读的字符串
|repr(x)|将对象 x 转换为适合解释器读取的表达式字符串
|eval(expression[, globals[, locals]])|计算在字符串中的有效表达式,并返回一个对象。globals 全局命名空间，如果被提供，则必须是一个字典对象。locals局部命名空间，如果被提供，可以是任何映射对象。
|tuple(s)|将序列 s 转换为一个元组
|list(s)|将序列 s 转换为一个列表
|set([s])|用可迭代对象s创建一个无序无重复元素集
|dict([d[,kwarg])|创建一个字典。d 必须是一个映射或者可迭代对象
|frozenset(s)|将s转换为不可变集合，不能再添加或删除任何元素。
|chr(x)|将一个整数转换为一个字符，x是可迭代的对象，比如列表、字典、元组等等。
|unichr(x)|将一个整数转换为Unicode字符
|ord(x)|将一个字符转为它对应的 ASCII 数值，或者 Unicode 数值
|hex(x)|将一个整数转换为一个十六进制字符串
|oct(x)|将一个整数转换为一个八进制字符串


------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《菜鸟教程》](http://www.runoob.com/ "点击跳转")、[《易百教程》](https://www.yiibai.com/ "点击跳转")、[《python3 官方文档》](https://www.python.org/doc/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

&emsp;&emsp;*注意：此教程基于python3.5*

------
