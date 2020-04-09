---
title: API文档生成工具——Doxygen
tags: C_and_C++
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《Doxygen详细介绍》](http://blog.51cto.com/ticktick/188671 "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

<style>table{word-break:initial;}</style>



# 1 简介
&emsp;&emsp;为代码写注释一直是大多数程序员有些困扰的事情。更头痛的是写文档，以及维护文档的问题，而doxygen就能把遵守某种格式的注释自动转化为对应的文档。
&emsp;&emsp;Doxygen是基于GPL的开源项目，是一个非常优秀的文档系统，当前支持在大多数unix（包括linux），windows家族，Mac系统上运行，完全支持C++, C, Java, IDL（Corba和Microsoft 家族）语言，部分支持PHP和C#语言，输出格式包括HTML、latex、RTF、ps、PDF、压缩的HTML和unix manpage。有很多开源项目（如log4cpp和CppUnit）都使用了doxygen文档系统。  

# 2 Doxygen配置文件
## 2.1   生成Doxygen配置文件
&emsp;&emsp;运行Doxywizard创建配置文件。可以直接点“Save...”按钮，将保存默认的配置文件，名为Doxyfile，内容是Doxygen的默认设置。Doxyfile是普通文本文件，可以直接打开手动编辑。不过在Doxywizard的界面上填写也很方便，每个参数都有详细提示。建议用Doxywizard完成第一次设置。以后如果需要调整个别参数，可以直接编Doxyfile。
&emsp;&emsp;上述Doxywizard界面中提供了生成Doxygen文档的4个步骤，按照上述步骤一步步执行就可以生成漂亮的文档了。
&emsp;&emsp;第一步是生成配置文件，提供三种方式：

* Wizard方式指简约方式，在其中只提供一些重要的参数设置，其余的均为默认值；
* Expert方式为详细设置方式，通过该选项可以详细地配置Doxygen的各个配置项；  
* Load方式，用于导入以前生成的Doxygen配置文件，导入后可以再点击Expert进行修改。

## 2.2    配置选项含义详解

|选项|含义|
|:----|:----|
DOXYFILE_ENCODING|Doxygen文件的编码方式，默认为UTF-8，若希望支持中文，最好设置为 GB2312
PROJECT_NAME|Project 的名字，以一个单词为主，多个单词请使用双引号括住。
PROJECT_VERSION|Project的版本号码。
OUTPUT_DIRECTORY|输出路径。产生的文件会放在这个路径之下。如果没有填这个路径，将会以目前所在路径作为输出路径。
OUTPUT_LANGUAGE|输出语言, 默认为English 。
EXTRACT_ALL|为NO，只解释有doxygen格式注释的代码；为YES，解析所有代码，即使没有注释
EXTRACT_PRIVATE|是否解析类的私有成员
EXTRACT_STATIC|是否解析静态项
EXTRACT_LOCAL_CLASSES|是否解析源文件（cpp文件）中定义的类
INPUT|指定加载或找寻要处理的程序代码文件路径。这边是一个表列式的型态。并且可指定档案及路径。
FILE_PATTERNS|如果您的INPUT Tag 中指定了目录。您可以透过这个Tag来要求Doxygen在处理时，只针对特定的档案进行动作。例如：您希望对目录下的扩展名为.c, .cpp及.h的档案作处理。您可设定FILE_PATTERNS = *.c, *.cpp, *.h。    
RECURSIVE|这是一个布尔值的Tag，只接受YES或NO。当设定为YES时，INPUT所指定目录的所有子目录都会被处理.
EXCLUDE|如果您有某几个特定档案或是目录，不希望经过Doxygen处理。您可在这个Tag中指定。   
EXCLUDE_PATTERNS|类似于FILE_PATTERNS的用法，只是这个Tag是供EXCLUDE所使用。
SOURCE_BROWSER|如果设定为YES，则Doxygen会产生出源文件的列表，以供查阅。
INLINE_SOURCES|如果设定为YES ，则函数和类的实现代码被包含在文档中
ALPHABETICAL_INDEX|如果设定为YES，则一个依照字母排序的列表会加入在产生的文件中。（有很多类、结构等项时建议设为YES）
GENERATE_HTML|若设定为YES ，就会产生HTML版本的说明文件。HTML文件是Doxygen预设产生的格式之一。
HTML_OUTPUT|HTML文件的输出目录。这是一个相对路径，所以实际的路径为OUTPUT_DIRECTORY加上HTML_OUTPUT。这个设定预设为html。      
GENERATE_HTMLHELP|是否生成压缩HTML格式文档（.chm）
HTML_FILE_EXTENSION|HTML文件的扩展名。预设为.html。
HTML_HEADER|要使用在每一页HTML文件中的Header。如果没有指定，Doxygen会使用自己预设的Header。
HTML_FOOTER|要使用在每一页HTML文件中的Footer。如果没有指定，Doxygen会使用自己预设的Footer。
HTML_STYLESHEET|您可给定一个CSS 的设定，让HTML的输出结果更完美。
GENERATE_HTMLHELP|如设定为YES，Doxygen会产生一个索引文件。这个索引文件在您需要制作windows 上的HTML格式的HELP档案时会用的上。
GENERATE_TREEVIEW|若设定为YES，Doxygen会帮您产生一个树状结构，在画面左侧。这个树状结构是以JavaScript所写成。所以需要新版的Browser才能正确显示。
TREEVIEW_WIDTH|用来设定树状结构在画面上的宽度。
GENERATE_LATEX|设定为YES 时，会产生LaTeX 的文件。不过您的系统必需要有安装LaTeX 的相关工具。   
LATEX_OUTPUT|LaTeX文件的输出目录，与HTML_OUTPUT用法相同，一样是指在OUTPUT_DIRECTORY之下的路径。预设为latex。                        
LATEX_CMD_NAME|LaTeX程序的命令名称及档案所在。预设为latex。
GENERATE_RTF|若设定为YES ，则会产生RTF 格式的说明档。
RTF_OUTPUT|与HTML_OUTPUT 用法相同，用来指定RTF 输出档案路径。预设为rtf。
GENERATE_MAN|若设定为YES ，则会产生Unix Man Page 格式的说明文件。
MAN_OUTPUT|与HTML_OUTPUT 用法相同，用来指定Man Page的输出目录。预设为man。
GENERATE_XML|若设定为YES ，则会产生XML 格式的说明文件。
ENABLE_PREPROCESSING|若设定为YES ，则Doxygen 会激活C 的前置处理器来处理原始档。               
PREDEFINED|可以让您自行定义一些宏。类似于gcc 中的-D选项。
CLASS_DIAGRAMS|这个标记用来生成类继承层次结构图。要想生成更好的视图，可以从 Graphviz 下载站点 下载 dot 工具。Doxyfile 中的以下标记用来生成图表：
HAVE_DOT|如果这个标记设置为 Yes，doxygen 就使用 dot 工具生成更强大的图形，比如帮助理解类成员及其数据结构的协作图。注意，如果这个标记设置为 Yes，<CLASS_DIAGRAMS> 标记就无效了
CLASS_GRAPH|如果 <HAVE_DOT> 标记和这个标记同时设置为 Yes，就使用 dot 生成继承层次结构图
GRAPHICAL_HIERARCHY|设置为YES时，将会绘制一个图形表示的类图结构

# 3 Doxygen的注释风格
## 3.1 综述
&emsp;&emsp;在每个代码项中都可以有两类描述, 这两类描述将在文档中格式化在一起: brief描述和detailed。 两种都是可选的，但不能同时没有。,**简述(brief)就是在一行内简述地描述。而详细描述(detailed description)则提供更长, 更详细的文档。**
&emsp;&emsp;Doxygen支持c风格注释、c++风格注释以及javaDoc风格注释等，下面将分别予以介绍。
若需要通过Doxygen生成漂亮的文档，一般有如下几个地方需要使用Doxygen支持的风格进行注释：

* 文件头（包括.h和.cpp）
        主要用于声明版权，描述本文件实现的功能，以及文件版本信息等
* 类的定义
        主要用于描述该类的功能，同时也可以包含使用方法或者注意事项的简要描述
* 类的成员变量定义
        在类的成员变量上方，对该成员变量进行简要地描述
* 类的成员函数定义
        在类定义的成员函数上方，对该成员函数功能进行简要描述
* 函数的实现在函数的实现代码处，详细描述函数的功能、参数的含义、返回值的含义、使用本函数需要注意的问题、本函数使用其他类或函数的说明等
 
## 3.2 Doxygen支持的指令
&emsp;&emsp;可以在注释中加一些Doxygen支持的指令，主要作用是控制输出文档的排版格式，使用这些指令时**需要在前面加上“\”或者“@”（JavaDoc风格）符号**，告诉Doxygen这些是一些特殊的指令，通过加入这些指令以及配备相应的文字，可以生成更加丰富的文档，下面对比较常用的指令做一下简单介绍。
|指令|说明
---|---
@file|档案的批注说明。
|@author|作者的信息
|@brief|用于class 或function的简易说明eg：@brief 本函数负责打印错误信息串
|@param|主要用于函数说明中，后面接参数的名字，然后再接关于该参数的说明
|@return|描述该函数的返回值情况eg:@return 本函数返回执行结果，若成功则返回TRUE，否则返回FLASE
|@retval|描述返回值类型，eg:@retval NULL 空字符串。
|@note|注解
|@attention|注意
|@warning|警告信息
|@enum|引用了某个枚举，Doxygen会在该枚举处产生一个链接，eg：@enum CTest::MyEnum
|@var|引用了某个变量，Doxygen会在该枚举处产生一个链接，eg：@var CTest::m_FileKey
|@class |引用某个类，格式：`@class <name> [<header-file>] [<header-name> ]，eg:@class CTest "inc/class.h"`
@exception|可能产生的异常描述,eg:@exception 本函数执行可能会产生超出范围的异常
 
## 3.3  JavaDoc风格的注释
### 3.3.1 概述
&emsp;&emsp;JavaDoc风格的注释风格主要使用下面这种样式：即在注释块开始使用两个星号‘*‘
```cpp 
/**   description      
 *    description      
 *    description      
 */ 
```
### 3.3.2    简述与详述的方式
&emsp;&emsp;Doxygen支持的块（类、函数、结构体等）的注释描述分为两种：简述 和 详述。一般注释的描述由简述开始，经过特殊分隔方式后，后面紧跟详述的内容，javaDoc风格可以使用的分隔方法有以下两种：

* 使用 \brief 参数标识，空行作为简述和详述的间隔标识
```
/*!    @brief  Brief description.  
 *    description continued.  
 *  
 *    Detailed description starts here.  
 */ 
```
*  直接使用javaDoc风格，javaDoc风格自动以简述开头，以空行（或者小数点加空格）作为简述与详述的分割
```cpp
/**    Brief description  
 *    description continued  
 *  
 *    Detailed description starts here.  
 */ 
 
/**        Brief description  
 *         description continued . (注意:这里有一个小数点,加上一个空格)  
 *         Detailed description starts here.  
 */ 
```
### 3.3.3   文件头注释示例

### 3.3.4   类定义注释示例
```cpp
/**    本类的功能：打印错误信息  
 *   
 *     本类是一个单件  
 *     在程序中需要进行错误信息打印的地方  
 */ 
class CPrintError  
{  
            ……  
} 
```
### 3.3.5   类成员变量定义示例
* 在成员变量上面加注释的格式
```
/** 成员变量描述 */ 
int  m_Var; 
```
* 在成员变量后面加注释的格式
```
int  m_color;     /**< 颜色变量 */ 
```    
### 3.3.6   成员函数的注释示例
```
/** 下面是一个含有两个参数的函数的注释说明（简述）  
 *  
 *     这里写该函数的详述信息  
 *     @param a 被测试的变量（param描述参数）  
 *     @param s 指向描述测试信息的字符串  
 *     @return    测试结果 （return描述返回值）  
 *     @see    Test()    （本函数参考其它的相关的函数，这里作一个链接）  
 *     @note    (note描述需要注意的问题)  
 */ 
int testMe(int a,const char *s); 
```
### 3.3.7     枚举变量的注释示例
```
/**    颜色的枚举定义  
  *   
  *    该枚举定义了系统中需要用到的颜色\n  
  *    可以使用该枚举作为系统中颜色的标识  
  */ 
enum TEnum   
{   
     RED,           /**< 枚举，标识红色    */      
     BLUE,          /**< 枚举，标志蓝色    */      
     YELLOW         /**< 枚举，标志黄色. */      
}enumVar;    
```
## 3.4      C++风格的注释
### 3.4.1    概述
&emsp;&emsp;C ++ 的注释风格主要使用下面这种样式：即在注释块开始使用三个反斜杠‘/’其他地方其实与JavaDoc的风格类似，只是C ++风格不用 “*” 罢了。
### 3.4.2       简述与详述
&emsp;&emsp;C ++风格的简述与详述方式与javaDoc类似。
&emsp;&emsp;一般注释的描述由简述开始，经过特殊分隔方式后，后面紧跟详述的内容，C ++ 风格可以使用的分隔方法有以下两种：
* 使用 \brief 参数标识，空行作为简述和详述的间隔标识
``` 
///    \brief  Brief description.  
///    description continued.  
///  
///    Detailed description starts here.  
/// 
```
* 使用以空行作为简述与详述的分割
``` 
///   Brief description  
///   description continued.  
///     
///   Detailed description starts here. 
```
* 以小数点加空格作为分隔如下：
```
///         Brief description  
///         description continued . （注意:这里有一个小数点,加上一个空格）  
///         Detailed description starts here.  
/// 
```
### 3.4.3    注释风格约定
* 一个代码块（类、函数、结构等）的概述采用单行的”///”加一个空格开头的注释，并写在该代码块声明的前面；
* 一个代码块的详述采用至少两行的”///”加一个空格开头的注释，若不足两行第二行的开头也要写出来，并且放在代码块定义的前面；如果某代码没有声明只有定义或者相反，则在定义或者声明前面写上单行的概述+一个空行+多行的详述；
* 枚举值列表的各项、结构域的各项等采用在本行最后添加”///<”加一个空格开头的注释；
* 对变量的定义采用在变量上面加单行”///”加一个空格开头的注释（相当于是给改变量一个概述）；
* 函数的参数用”/// @param”+一个空格开头的行描述在函数的详述里面；
* 函数的返回值用”/// @return”+一个空格开头的行描述在函数的详述里面；
* 函数之间的参考用”/// @see”+一个空格开头的行描述在函数的详述里面；
* 文件头的版权以及文件描述的注释参见例代码。
### 3.4.4       文件头注释示例
![这里写图片描述](http://img.blog.csdn.net/20170817132958871?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGlhbzIwMDgxMjI4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
### 3.4.5       类定义注释示例
```
///    本类的功能：打印错误信息  
///  
///    本类是一个单件  
///    在程序中需要进行错误信息打印的地方  
class CPrintError  
{  
          ……  
} 
``` 
###3.4.6       类成员变量定义示例
*在成员变量上面加注释的格式
```
/// 成员变量描述  
int  m_Var; 
```
*在成员变量后面加注释的格式
```
int  m_color;     /// 颜色变量    
```
### 3.4.7       成员函数的注释示例
```
/// 下面是一个含有两个参数的函数的注释说明（简述）   
///    
///     这里写该函数的详述信息    
///     @param a 被测试的变量（param描述参数）    
///     @param s 指向描述测试信息的字符串    
///     @return    测试结果 （return描述返回值）   
///     @see    Test()    （本函数参考其它的相关的函数，这里作一个链接）  
///     @note    (note描述需要注意的问题)    
int testMe(int a,const char *s);  
```
### 3.4.8       枚举变量的注释示例
``` 
///    颜色的枚举定义  
///   
///    该枚举定义了系统中需要用到的颜色\n  
///    可以使用该枚举作为系统中颜色的标识  
enum TEnum   
{   
    RED,            ///< 枚举，标识红色       
    BLUE,           ///< 枚举，标志蓝色       
    YELLOW          ///< 枚举，标志黄色.       
}enumVar;  
```   
## 3.5     需要注意的问题
* Doxygen会忽略你在注释中加入的多余的换行，如果你希望保留两行注释之间的空行，那么，在该行加入 \n
 

# 4   常见问题
## 4.1 中文问题：中文注释在文档中是乱码
&emsp;&emsp;解决：在expert中的INPUT选项页的INPUT_ENCODEING中填入“GB2312”，这样基于GB的文本编辑器生成的代码就可以正常使用了。
## 4.2    图形问题：无法绘制类图协作图等图形。
&emsp;&emsp;解决：首先确保安装了graphviz for win，注意不是wingraphviz，后者是一个graphviz的com封装，但是doxygen并不是基于它开发的，所以装了也没用。然后在expert的DOT_PATH中填入graphviz的安装路径。接着在wizard的diagram中选择需要生成的图形类别就可以了。
如果出现无法包含.map文件的错误，可以将工作目录设置成html，并将html中所有文件都清除再试。这个问题的原因还不太确定。
## 4.3    输出chm的问题：如何输出.chm文件
&emsp;&emsp;配置时注意expert中的HTML页：选中“GENERATE_HTMLHELP”，然后在CHM_FILE中填上想要的chm文件名。
&emsp;&emsp;HHC_LOCATION中输入hhc.exe文件的路径。hhc.exe可以通过安装HTML Help Workshop获得。
&emsp;&emsp;或者使用HTML Help Workshop来编译Doxygen生成的html文件夹中的.hhp文件，编译完成后即可在该html文件夹中找到对应的chm文件
## 4.4     Doxygen无法为DLL中定义的类 导出文档
```
例如：
class __declspec(dllexport) CClassName:public CObject
{
}
目前发现Doxygen无法识别出DLL中定义的类。
```


------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《Doxygen详细介绍》](http://blog.51cto.com/ticktick/188671 "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
