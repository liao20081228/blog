---
title: ASN详解
tags: 未分类
---


------

***<font color=blue>版权声明</font>：本文参考了<font color=blue>。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

# 1 ASN简介 
ASN.1抽象语法标记（Abstract Syntax Notation One）是一种 ISO/ITU-T标准，描述了**一种对数据进行表示、编码、传输和解码的数据格式**。它提供了一整套正规的格式用于描述对象的结构，而不管语言上如何执行及这些数据的具体指代，也不用去管到底是什么样的应用程序。也就是说ASN.1**只包含信息结构，不处理具体业务数据，不是一个编程语言**。

在任何需要以数字方式发送信息的地方，ASN.1 都可以发送各种形式的信息（声频、视频、数据等等）。ASN.1 和特定的 ASN.1 编码规则推进了结构化数据的传输，尤其是网络中应用程序之间的结构化数据传输，它以一种独立于计算机架构和语言的方式来描述数据结构。

OSI应用层协议使用了ASN.1来描述它们所传输的PDU，包括：
* 用于传输电子邮件的X.400
* 用于目录服务的X.500
* 用于VoIP 的H.323 和SNMP
* 通用移动通信系统（UMTS）中的接入和非接入层

一种ASN.1数据类型对应的传输语法可以有多种，但只能使用其中的一种。

ASN.1 的描述可以容易地被映射成 C 或 C++ 或 Java 的数据结构，并可以被应用程序代码使用，并得到运行时程序库的支持。

ASN.1规范包括：ISO8824/ITU X.208（说明语法）和ISO8825/ITU X.209（说明基本编码规则）。

# 2 相关背景知识
## 2.1 基本概念：

**抽象语法**：描述了与任何表示数据的编码技术无关的通用数据结构（包括不同的数据类型），并且建立了和应用层对话所用的构架。
* 抽象语法使得人们能够定义数据类型，并指明这些类型的值。
* 抽象语法只描述数据的结构形式，与具体的编码格式无关，同时也不涉及这些数据结构在计算机内如何存放。

**实际语法**：本地的，并且定义本地系统的数据表示方法。如C和Java。

**传输语法**：定义两个系统间的表示层间交换数据的表示方法，是实际通讯系统间的码流。当数据在两个表示层实体之间传输时，这些数据的实际比特模式表示方法就是传输语法。

**编码与解码**：指将抽象语言法转换成实际通讯系统间比特流。解码则相反。

**编解码规则**：提供从实际语法到传输语法和其相反操作的方法。(从抽象语法到传输语法，由ASN.1编译器按照编解码规则实现)。

ASN.1标准化编码规则：
* 基本编码规则（BER） X.209 ：ASN.1最基本的编码方式。
* 规范编码规则（CER）：主要用于编解码数据量巨大的消息，它能够保证在所有数据还没有到达的时候就开始进行编码解码的工作。
* 识别名编码规则（DER）：主要用于对安全性要求比较高的应用程序。
* 压缩编码规则（PER）：采用了有效 算法，缩短了编解码的时间
* XML编码规则（XER）


这些编码规则描述了如何对 ASN.1 中定义的数值进行编码，以便用于传输，而不管计算机、编程语言或它在应用程序中如何表示等因素。ASN.1 的编码方法比许多与之相竞争的标记系统更先进，它支持可扩展信息快速可靠的传输。


BER 采用了最基本的TLV三元组结构对抽象数据进行编解码操作 ， 其编码规则简单，但编码后数据开销过大，增加了很多冗余数据。

CER和DER两种编码规则均 在BER 的 基础上增加 了 一定 的 约束发展而来 。 CER与DER的主要区别在于：CER使用不定长编码格式，满足了传输大量数据的 需要 ；DER使用定长编码格式为可靠数据的传输而设计。

1994年ASN.1引入了PER编码规则。与BER规则相比，PER编码后数据 占用的空间 能获得40%-50%的改进，因此被广泛应用于VoIP、视频电话、多媒体及第三代移动通信系统等高速数据传输领域。与BER中递归使用三元组TLV不同，PER的格式为：[P][L][V]  （ optional Preamble, optional Length, optional Value ） ，PLV中每个域不是八位组串 ， 而是比特串。在PER规则中，由于Length可以省略（有些时候Value也可以省略），因而不能从编码中得知边界，×××必须知道抽象描述才能正确解码。PER编码中没有Tag域，因此 , PER不再缺省支持扩展，必须明确在描述中添加扩展符。只有当长度没有被固定或者数据长度很重要的情况下，才对Length进行编码。对SEQUENCE或SET类型的值编码时，使用一个bitmap来标识可选成员是否出现；同样，在编码CHOICE的被选择成员前， 也 会增加一个序号指示其位置。和BER相比，PER使编××× 的 的处理时间相对少些，传输速度更快。PER编码规则可以分为基本的和规范的两类，每一类又可以分为对齐和不对齐两种。

XER编码规则主要被用来将数据转换成XML格式数据，该编码规则在1999年才被引入到ASN.1标准中。
## 2.2 应用层

应用层向表示层发送数据时，同时告知表示层自己的ASN.1名字(即对象标识符)，ASN.1名字标记了一个ASN.1语法——用以解释数据中各字段的含义。通过参考ASN.1定义，表示层可以得知数据单元的类型和长度，以及传输时应当采用的编码方法。

## 2.3 表示层

两个系统在传输数据前需要协商共用的编码方式，事实上编码方式在应用层发出的数据中已经确定，应用数据中包含抽象语法/传输语法的组合关系，告诉表示层数据的结构、含义以及传输语法规则。表示层参考抽象语法，将应用数据转换为传输语法定义的比特流。

## 2.4 边界对齐

同样一条消息，在计算机内存中是以Byte为单位存储的，在物理层则是以bit为单位传送的。

如果一个信元的第一个bit也恰好是Byte流中某Byte的开始bit时，我们称之为开始于边界对齐的；如果信元的最后一个bit也恰好是Byte流中某Byte的最后一个bit时，则可以称之为结束于边界对齐的。

对于不是结束于边界对齐的情况，一般要进行补位。有两种方式：
* 对每个信元的结束立即进行补位，保证下一个信元是开始于边界对齐的；
* 从信元结束的位置开始新的信元，到消息结束时再进行一次补位操作。(用于无线空口中)

## 2.5 大小端

(1).大端方式，也叫网络序，从左往右，第一个8位表示高位，例如0X0102,用比特流表示是0000000100000010。

(2).小端方式，也叫主机序，与大端方式相反，数字0X0102用比特流表示则是0000001000000001，低8位在前，高8位在后。

Motorola的PPC系列、IP协议中使用大端方式;VAX计算机、Intel的x86系列中使用小端方式。

# 2 基本语法

1. ASN.1使用巴科斯范式(BNF):
	* 在双引号中的字`"word"`代表着这些字符本身。而double_quote用来代表双引号。
	* 在双引号外的字（有可能有下划线）代表着语法部分。
	* 尖括号`< >`内包含的为必选项。
	* 方括号`[ ]`内包含的为可选项。
	* 大括号`{ }`内包含的为可重复0至无数次的项。
	* 竖线`|`表示在其左右两边任选一项，相当于"OR"的意思。
	* `::=`是“被定义为”的意思。

	下面是用BNF来定义的Java语言中的For语句的实例：
	```
	  FOR_STATEMENT::=
	  
	  "for""(" ( variable_declaration |
	  
	  (expression ";" ) | ";" )
	  
	  [expression ] ";"
	  
	  [expression ]
	  
	  ")"statement
	```
2. 在ASN.1中，符号的定义没有先后次序：只要能够找到该符号的定义即可。

3. 所有的标识符、参考、关键字都要以一个字母开头，后接字母(大、小写都可以)、数字或者连字符“-”(但不能以连字符“-”结尾，也不能连续出现两个连字符)，不能出现下划线“_”。

4. 关键字一般都是全部大写。

5. 在标识符中，只有类型和模块名字是以大写字母开头的，其它标识符都是以小写字母开头。

6. ASN.1中实数实际定义为三个整数：尾数、基数和指数。没有小数表示方式。

7. ASN.1不对空格、制表符、换行符和注释做翻译。但是在定义符号（或者分配符号Assignment）“::=”中不能有分隔符。

# 4 类型
类型是一个非空的值的集合，可以被编码后传输。相比与高级语言中复杂的数据结构，ASN.1中的类型主要是为了数据的传输。

## 4.1 内建类型


|基本类型|含义|
|:--|:--|
|NULL|只包含一个值NULL，用于传送一个报告或者作为CHOICE类型中某些值
|INTEGER|全部整数（包括正数和负数）
|REAL|实数，表示浮点数
|ENUMERATED|标识符的枚举（实例状态机的状态）
|BITSTRING|比特串
|OCTETSTRING|字节串
|OBJECT IDENTIFIER,RELATIVE-OID|一个实体的标识符，它在一个全世界范围树状结构中注册
|EXTERNAL,EMBEDDED PDV|表示层上下文交换类型
|…String(除了BITSTRING、OCTETSTRING外)|各种字符串，有NumericString、PrintableString、VisibleStirng、<br />ISO64String、IA5String、TeletexStirng、T61String、VideotexString、GraphicString、GeneralString、UniversalString、BMPString和UTF8String|
|CHARACTERSTRING|允许为字符串协商一个明确的字符表
|UTCTime,GeneralizedTime|日期|
|**组合类型**|**含义**|
|CHOICE|在类型中选择(相当于C中的联合)|
|SEQUENCE|由不同类型的值组成一个有序的结构(相当于C中的结构体)|
|SET|由不同类型的值组成一个无序的结构|
|SEQUENCEOF|由相同类型的值组成一个有序的结构(相当于C中的数组)|
|SETOF|由相同类型的值组成一个无序的结构|

## 4.2 类型定义

<新类型的名字>::= <类型描述>

例：
```
Married ::= BOOLEAN

Age ::= INTEGER

Picture ::= BIT STRING

Form ::= SEQUENCE

{

	name PrintableString,

	age Age,

	married Married,

	marriage-certificate PictureOPTIONAL

}
```
Married类型是一个基本类型BOOLEAN，Form类型是一组基本类型的有序序列

注意：在SEQUENCE和SET等(好像应该是所有组合类型的)定义中，最后一个成员结尾没有逗号“,”。

为了接收方能正确解码，发送方为每个值的类型附加一个数，称为tag，在描述中以“[]”标识。缺省情况下，编码器会使用universal的tag。在给合类型中，为了明确各个成员，有必要指明每个成员的Tag：
```
Coordinates ::= SET

{

	x [1] INTEGER, //这证明好像也可以用类来直接声明变量

	y [2] INTEGER,

	z [3] INTEGER OPTIONAL

}
```
Tag会在传输规则使用到，用于在比特流中指明数据的具体类型。

为了准确描述一个类型，我们需要对值的集合进行一定的限制。这用到子类型约束，在类型之后用圆括号进行标识。如：
```
Lottery-number::= INTERGER(1..49) //表示取1-49任一一个值

Lottery-draw ::=SEQUENCESIZE(6) OF Lottery-number //指定了该SEQUENCE类型由6个Lottery-number类型有序组成。

Upper-case-words::= IA5String (FROM(“A”..”Z”)) //表示按ASCII取A-Z中任一一个，IA5String是ASCII字符串类型
```
为了方便在新的版本中往现有类型中添加新成员，可用“…”来标记可能以后是其它类型的地方：
```
Type ::= SEQUENCE
{
	component1 INTERGER,
	component2 BOOLEAN,
	…
}
```
以后新的版本中，描述可能为：
```
Type ::= SEQUENCE
{
	component1 INTERGER,
	component2 BOOLEAN,
	…,
	[[component3REAL]], -- version 2
	…
}
```
注意：新加入的类型成员要嵌套在“[[]]”中,--version 2指定新版本号

## 4.3 值定义

<新的值的名字> <该值的类型> ::=<值描述>

其中：

<新的值的名字>是以小写字母开头的标识符；

<该值的类型>可以是一个类型的名字，也可以是类型描述；

<值描述>是基于整数、字符串、标识符的组合。

如：

counter Lottery-number ::= 45

sextuple Lottery-draw ::= { 7, 12, 23, 31, 33, 41 }

## 4.4 信息对象类和信息对象

<信息对象类>::= CLASS <类描述>

WITHSYNTAX <信息描述>

用于表达比注释更为正式的一些信息

## 4.5 模块定义
```
<模块名字> DEFINITIONS <缺省Tag>::=

BEGIN

	EXPORTS <导出描述>

	IMPORTS <导入描述>

	<模块体描述>

END
```
一般协议由一个或者多个模块组成，模块用来收集数据结构定义。

模块名字必须以大写字母开头。模块能以一种“全局指针”(UniversalPointer)的方式来引用，称为对象标识符（ObjectIdentifier），用花括号标识在名字之后。

如：
```
Module2 { isomember-body(2) f(250) type-org(1) ft(16)

asn1-book(9)chapter5(0) module2(1) }

DEFINITIONS AUTOMATICTAGS ::=

BEGIN

	EXPORTS Type2;

	IMPORTS Type1, value FROM Module1 {iso member-body(2)

	f(250) type-org(1)ft(16) asn1-book(9) chapter5(0) module1(0)};

	Type2 ::= SEQUENCE OFChoice

	Choice ::= CHOICE

	{

		a INTEGER (0..value),

		b Type1

	}`

END
```
(1).AUTOMATICTAGS是指缺省Tag，说明不关注模块的Tag。

(2).IMPORTS声明在其它模块定义但在本模块会用到的类型或者值。

EXPORT声明在本模块之外可以访问的类型或者值。

IMPORTS的语法为：

IMPORTS <名字>,value FROM <其它模块的ObjectIdentifier >;

EXPORTS的语法为：

EXPORTS<名字>;

(3).对象标识符(OBJECTIDENTIFIER,OID)类型用层次的形式来表示标准规范。标识符树通过一个点分的十进制符号来定义，这个符号以组织，子部分然后是标准的类型和各自的子标识符开始．

例如：MD5的OID是1.2.840.113549.2.5表示为"iso(1)member-body (2) US (840) rsadsi(113549) digestAlgorithm (2) md5 (5)",所以当解码程序看到这个OID时,就知道是MD5散列。

OID在公钥算法标准中很流行,它指出证书绑定了哪种散列算法。

OID在传输时编码规则:

前两部分如果定义为x.y,那么它们将合成一个字40*x+ y,其余部分单独作为一个字节进行编码。

每个字首先被分割为最少数量的没有头零数字的7位数字.这些数字以big-endian格式进行组织,并且一个接一个地组合成字节.除了编码的最后一个字节外,其他所有字节的最高位(位8)都为1。

MD5OID的编码:

<1>.将1.2.840.113549.2.5转换成字数组{42,840, 113549, 2, 5}。

<2>.然后将每个字分割为带有最高位的7位数字，{{0x},{0x86,0x48},{0x86,0xF7,0x0D},{0x02},{0x05}}。

<3>.最后完整的编码为0x0608 86 48 7 0D 02 05。　

6.模块和分配(Assignment)

(1).定义一个新类型

TypeReference ::=CHOICE

{

integer INTEGER,

boolean BOOLEAN

}

(2).给类型赋值

在ASN.1中给类型赋的值不会被编码(ASN.1语法只是一种语法规则用来描述业务数据，而不会被当成业务数据)，这种值常用作DEFAULT，上下界或者信息对象中。

value-referenceTypeReference ::= integer:12

如果两个类型在语法上是完全一样的，则这两种类型的值可以相互赋值。如：

Pair::= SEQUENCE

{

x INTEGER,

y INTEGER

}

Couple::= SEQUENCE

{

x INTEGER,

y INTEGER

}

pairPair ::= {x 5, y 13}

coupleCouple ::= pair

(3).值集合

在语义上，一个值集合相当于一个添加约束后的类型。如：

PrimeNumbers INTEGER ::= {2 | 3 | 5 | 7 | 11 | 13}


------

***<font color=blue>版权声明</font>：本文参考了<font color=blue>。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
