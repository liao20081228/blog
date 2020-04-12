---
title: markdown教程
tags: 未分类, 未完成
---

------

***<font color=blue>版权声明</font>：本文章参考了<font color=blue >《小书匠markdown 官方教程》。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处！！！</font>***

------

# 1 标准语法
## 1.1 转义

右斜线`\`表示转义，markdown中标记字符，例如`[`、 `\`、`*`、`\`、`$`、`_`、`<`、\`都需要加上转义符号才能使用原字符。

## 1.2 标题

### 1.2.1 markdown
**第一种，`# 标题`或`# 标题 #`，·闭合的`#`可与开始的`#`数目不同**。例如：

```
# head1 表示一级标题 
## head2 表示二级标题 ##
### head3 表示三级标题 ###
#### head4 表示四级标题 #
##### head5 表示五级标题 #
###### head6  表示六级标题
```
显示效果：

![图0](https://gitee.com/liao20081228/blog_pictures/raw/master/markdown教程/0.jpg#pic_center)

	>**注意**：只有当`#`位于一行开头，或引用文字中的一行开头的才会起作用。
	>**注意**：#与标题之间要有空格，在标准的markdown规范中必须要有空格才会起作用。

**第二种，使用`=`与`-`作为标题的底线**。例如：

```markdown
head1
==
head2
---
```

### 1.2.2 html
```html
<h1>head1</h1>
<h2>head2</h2>
<h3>head3</h3>
<h4>head4</h4>
<h5>head5</h5>
<h6>head6</h6>
```

## 1.3 段落
### 1.3.1 markdown

Markdown 的段落是由一个或多个连续的文本行组成，它的前后要有一个以上的空行（空行的定义是显示上看起来像是空的，便会被视为空行。比方说，若某一行只包含空格和制表符，则该行也会被视为空行）。普通段落不该用空格或制表符来缩进。

### 1.3.2 html

一个段落：`<p>段落文字</p>`。

强迫换行：`<br />`。

## 1.4 加粗与斜体

### 1.4.1 markdown

一个`*`或`_`环绕表示斜体，例如`*斜体*`， 显示效果：*斜体*。

两个`*`或`_`环绕表示加粗，例如`**加粗**` ，显示效果：**加粗**。

三个`*`或`_`环绕或混用`*`和`_`表示粗斜体，例如`***粗斜体***`，显示效果：***粗斜体***。

### 1.4.2 html

加粗：`<b>加粗</b>`。

斜体：`<i>斜体</i>`。

粗斜体：`<i><b>粗斜体</b></i>`。

## 1.5 删除线与水平线

### 1.5.1 markdown

* 两个  `~`环绕表示删除线，例如`~~删除线~~`，显示效果：~~删除线~~
* 三个及以上的`*` 或  `-`表示一条横线，例如`***`和`---`。

### 1.5.2 html
* 删除线：`<del>删除线</del>`。
* 水平线：`<hr />`。

## 1.6 引用
### 1.6.1 markdown

**在一行的开始用`>`表示引用**。例如：

```
>这是引用
```
显示效果:

>这是引用

**对一段话引用只需在开头使用一次,也可以每一行都使用**`>`。例如

```
> This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet, 
> consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus. 
> Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus. 

> This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet, 
consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus. 
Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus. 
```

显示效果：

> This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet, 
> consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus. 
> Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus. 
>  

> This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet, 
consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus. 
Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus. 

**连继多个 `>` 符号，就可以实现多级引用**。例如：

```
> 这是一个多级引用 
> 一级引用 
>> 二级引用 
>>> 三级引用 
```

显示效果：

> 这是一个多级引用 
> 一级引用 
>> 二级引用 
>>> 三级引用 

**在引用里可以使用其他 markdown 语法，例如：标题、列表、粗体、斜体、图片、链接、表格、代码、代码块、高级代码块等，对于需要多行的语法每一行都要添加`<`标记**。


### 1.6.2 html

引用：`<blockquote>引用文字</blockquote>`

## 1.7 无序列表

### 1.7.1 markdown

**在每一个条目前面添加一个`*`或`+`或`-`，再接一个空格就可以表示无序列表的一个列表项，同一个列表的同一级列表项应使用同一种标记符号**。 例如：

```
+ 无序条目 
+ 无序条目 
+ 无序条目
```

显示效果：

+ 无序条目 
+ 无序条目 
+ 无序条目


**无序列表可以有多级，每一级用一个缩进表示：两个空格或一个制表符以上，同级连续列表项应使用相同数量的同一缩进符号**。例如：

```
+ 无序条目 
  + 子无序条目 
  + 子无序条目 
		- 孙无序条目 
		- 孙无序条目 
+ 无序条目 
	* 子无序条目 
	* 子无序条目 
+ 无序条目 
```

显示效果：

+ 无序条目 
  + 子无序条目 
  + 子无序条目 
		- 孙无序条目 
		- 孙无序条目 
+ 无序条目 
	* 子无序条目 
	* 子无序条目 
+ 无序条目 


**无序列表可以有多层，可以是有序列表或无需列表**。例如：

```
* 一级一层无序条目
* * 一级二层无序条目
	* * + 三层三级无序条目
	* * 1. 三层三级有序条目
```
显示效果：

* 一级一层无序条目
* * 一级二层无序条目
	* * + 三层三级无序条目
	* * 1. 三层三级有序条目

**无序列表可与有序列表嵌套**。例如：

```
* 无序条目
* 无条条目
	1. 有序条目
	2. 有序条目
* 无序条目
```
显示效果：

* 无序条目
* 无条条目
	1. 有序条目
	2. 有序条目
* 无序条目

**单个列表项可以有多行，每行开头可以添加空格或制表符，或者不添加，但是不能有空行，否则视为下一段**。例如：

```
* Lorem ipsum dolor sit amet, consectetuer adipiscing elit.
Aliquam hendrerit mi posuere lectus. Vestibulum enim wisi,
  viverra nec, fringilla in, laoreet vitae, risus. 
  * Donec sit amet nisl. Aliquam semper ipsum sit amet velit.
	Suspendisse id sem consectetuer libero luctus adipiscing.
```

显示效果：

* Lorem ipsum dolor sit amet, consectetuer adipiscing elit.
Aliquam hendrerit mi posuere lectus. Vestibulum enim wisi,
  viverra nec, fringilla in, laoreet vitae, risus. 
  * Donec sit amet nisl. Aliquam semper ipsum sit amet velit.
	Suspendisse id sem consectetuer libero luctus adipiscing.

**单个列表项可以有多段，每段之间用空行隔开，每段第一行或每一行必须在列表项目的缩进基础上再加一个缩进：两个空格或一个制表符以上**。例如：

```
* This is a list item with two paragraphs. Lorem ipsum dolor 
sit amet, consectetuer adipiscing elit. Aliquam hendrerit 
mi posuere lectus. 

  Vestibulum enim wisi, viverra nec, fringilla in, laoreet 
  vitae, risus. Donec sit amet nisl. Aliquam semper ipsum 
  sit amet velit. 

	* This is a list item with two paragraphs. Lorem ipsum dolor 
sit amet, consectetuer adipiscing elit. Aliquam hendrerit 
mi posuere lectus. 

		Vestibulum enim wisi, viverra nec, fringilla in, laoreet 
	  vitae, risus. Donec sit amet nisl. Aliquam semper ipsum 
	  sit amet velit. 
```
显示效果：

* This is a list item with two paragraphs. Lorem ipsum dolor 
sit amet, consectetuer adipiscing elit. Aliquam hendrerit 
mi posuere lectus. 

  Vestibulum enim wisi, viverra nec, fringilla in, laoreet 
  vitae, risus. Donec sit amet nisl. Aliquam semper ipsum 
  sit amet velit. 

	* This is a list item with two paragraphs. Lorem ipsum dolor 
sit amet, consectetuer adipiscing elit. Aliquam hendrerit 
mi posuere lectus. 

		Vestibulum enim wisi, viverra nec, fringilla in, laoreet 
	  vitae, risus. Donec sit amet nisl. Aliquam semper ipsum 
	  sit amet velit. 

**一个列表项可以嵌套引用，如果引用作为列表项内容的一段，则应该在列表项的基础上上加上一个缩进，即两个空格以上或一个制表符**。例如：

```
* 这是一个列表项
* >引用作为一个列表项
* 这是一个列表项
	>引用作为一个列表项的一部分
	* 这是一个列表项

		>引用作为一个列表项的一部分
		>引用作为一个列表项的一部分
```

显示效果：

* 这是一个列表项
* >引用作为一个列表项
* 这是一个列表项
	>引用作为一个列表项的一部分
	* 这是一个列表项

		>引用作为一个列表项的一部分
		>引用作为一个列表项的一部分

**一个列表项可以嵌套代码块，该区块每一行需要在列表项目的缩进基础上再加一个制表符或4个空格，如果列表项标记后紧跟代码块，则可以省略首行的缩进**。例如：

```
* 这是一个无序列表项

		嵌套代码块作为列表项的一部分（两个制表符）
		嵌套代码块作为列表项的一部分（两个制表符）

	* 这是一个无序列表项

			嵌套代码块作为列表项的一部分（三个制表符，1个表示二级列表+1个表示属于列表项+1个表示代码块）
			嵌套代码块作为列表项的一部分（三个制表符，1+1+1）

*     嵌套代码块作为列表项（1空格+4个空格）
	  嵌套代码块作为列表项（6个空格，2个表示属于列表项+4个表示代码块）
		嵌套代码块作为列表项（8个空格，4个表示属于列表项+4个表示代码块）
		嵌套代码块作为列表项（两个制表符，1个表示属于列表项+1个表示代码块）
		嵌套代码块作为列表项（两个制表符）
```
显示效果：

* 这是一个无序列表项

		嵌套代码块作为列表项的一部分（两个制表符）
		嵌套代码块作为列表项的一部分（两个制表符）

	* 这是一个无序列表项

			嵌套代码块作为列表项的一部分（三个制表符，1个表示二级列表+1个表示属于列表项+1个表示代码块）
			嵌套代码块作为列表项的一部分（三个制表符，1+1+1）

*     嵌套代码块作为列表项（1空格+4个空格）
	  嵌套代码块作为列表项（6个空格，2个表示属于列表项+4个表示代码块）
		嵌套代码块作为列表项（8个空格，4个表示属于列表项+4个表示代码块）
		嵌套代码块作为列表项（两个制表符，1个表示属于列表项+1个表示代码块）
		嵌套代码块作为列表项（两个制表符）

**一个列表项可以嵌套高级代码块，和普通块类似，只不过用\`\`\`代替四个空格或制表符**。

**一个列表项还可以嵌套图片、链接、粗体、表格等语法**。

### 1.7.2 html

```html
<ul>
<li>列表项</li>
<li>列表项</li>
</ul
```

## 1.8 有序列表
### 1.8.1 markdown

有序列表用**序号.加空格**表示，有序列表与无序列表类似。

有序列表每一级都会重新开始排序

有序列表显示的是实际的序号与输入的序号无关，而是以该层列表第一项的编号作为起始编号。


>**注意**：大部分情况下，一个TAB或两个或三个空格都能算作一次缩进。建议使用TAB。

### 1.8.2 html
```html
<ol>
<li>列表项</li>
<li>列表项</li>
</ol
```

## 1.9 链接与图片
### 1.9.1 markdown

**链接有三种方式**：

1. 行内式：`[描述文字](链接地址 “悬停提示")` 
2. 参考式：`[描述文字][id]` ，在任何地方使用 `[id](链接地址 “悬停提示")`，可以省略id，省略id表示id为描述文字
3. 直接链接：`<完整链接地址>`，如：<https://www.baidu.com>

**图片的语法和链接前两种语法一致，只是在开头需要加上一个英文的感叹号** `!`，表示这是图片。如：`![这是行内式连接](www.baidu.com   "百度")`，可以在图片链接尾部加上`#pic_ctenter`，以把图片居中显示。

### 1.9.2 html

链接：`<a href="URL" title="链接的标题">链接文字</a>`

图片：`<img src="url" alt="无法载入时的替换文本" title="载入时的标题" />`


## 1.10 页内跳转
### 1.10.1 markdown

`[页内跳转](#页内文字)`，例如：`[跳到开头](#基本语法)`，显示效果：[跳到开头](#基本语法)

### 1.10.2 html

用`<span id='idname'>跳转文字</span>`设置锚点，在其他地方就可以使用`[跳转处](#idname)` 进行跳转。例如：[跳转到1.11](#test)。

## 1.11 代码和代码块

**句内代码用 \`……\`表示**。例如\`int a=2\`，显示效果：`int a=2`。

**代码块的每一行用四个空格或一个tab开头**。例如：
```
	int a=10;
	int b=20;
```
显示效果：

	int a=10;
	int b=20;

## 1.12 空格

`&emsp;`表示一个中文字符空格；
`&ensp;`表示一个英文字符空格。

## 1.13 HTML标签

<span id="test">只要支持GFM的markdown都兼容html标签</span>


## 1.14 语法嵌套

<span id=‘test’>所有的行内语法和几乎所有的多行语法都可以在其他语法中套用，比如加粗可在标题中使用。</span>
# 2 高级语法
|markdown版本|csdn|xiaoshujiang|
|:--|:--|:--|:--|
|任务列表|支持|支持|
|数学公式|支持|支持|
|加强代码块|支持|支持|
|代码块设置行号||支持|
|目录|支持|支持|
|思维导图目录||支持|
|脚注|支持|支持|
|缩进|支持|支持|
|下划线|支持|支持|
|文字高亮|支持|支持|
|页内跳转|||
|锚点跳转|支持|支持|
|标签||支持|
|字体、颜色、大小|支持|支持|
|视频||支持|
|音频||支持|
|附件||支持|
|段代码文字格式||支持|
|流程图|支持|支持|
|序列图|支持|支持|
|统计图||支持|
|mermaid 流程图 序列图 甘特图支持|支持|支持|
|html|支持|支持|
|思维导图||支持|

## 2.1 表格
### 2.1.1 基本表格
冒号表示对齐方式，分别是左对齐，居中，右对齐。可以在最后一行使用`&emsp;`或`&ensp`来控制列宽。

```markdown
|姓名|年龄|体重|
|:--|:--:|--:| 
|张三|100|20|    
```
显示效果：

 |姓名|年龄|体重|
 |:--|:--:|--:| 
 |张三|100|20|   



### 2.1.2 扩展表格

```markdown
 姓名|年龄|体重
 :--|:--:|--:
 张三|100|20  
```
显示效果：

 姓名|年龄|体重
 :--|:--:|--:
 张三|100|20

### 2.1.3 增强表格

提供增强型表格语法扩展功能：

**行合并**：如果单元格的右侧内容为空时，系统自动向右合并单元格。系统也支持在前一个单元格输入 `>`，来把当前的单元格合并到下一个单元格里。

**列合并**：系统也支持在前一个单元格输入 `^`，来把当前的单元格合并到上一个单元格里。

**表格标题**：在表格定义的结尾添加 `[标题内容]`

**多表头**：普通表格只能在 ---- 分界线的上方添加一个表格头，增强版的表格允许添加多个表格头

**多表格身**：当表格内容部份有空行时，系统会生成多表格身效果。在 html 上就是生成多个 tbody

```markdown
|First Header  | Second Header ||
|First Header  | Second Header | Third Header|
|------------- | -------------|-------------|
表身1Content Cell  | Merge Content Cell||
Content Cell  | Content Cell| Content Cell|

表身2Content Cell  | Merge Content Cell||
Content Cell  | Content Cell| Content Cell|
[表格标题]
```

显示效果：

![8](https://www.github.com/liao20081228/blog/raw/master/图片/markdown教程/8.jpg)

### 2.1.4 html表格

**定义表格**：`<table>...</table>` 

* 定义表格标题：`<caption>...</caption>`
* 定义行：`<tr>...</tr>`
	* 定义表头单元格：`<th>...</th>`
	* 定义单元格：`<td>...</td>`

**表格属性**：
* align：水平对齐方式
	* left 左对齐
	* right 右对齐
	* center 中心对齐
* valign：定义垂直对齐方式
* border：定义线宽

**单元格属性**：
* colspan：定义跨列
* bgcolor：定义背景色
* background：定义背景图
* rowspan：定义跨行
* align和valign：定义对齐方式
	* left
	* right
	* center

**示例**：
```html
<table>
  <caption>跨行和跨列，带背景色和前景色</caption> 
  <tr>
    <td>&emsp;</td>
    <th colspan='2'><font color="red">跨2列</font></th>
    <th><font color="red">第3列</font></th>
  </tr>
    <th rowspan='2'><font color="blue">跨2行</font></th>
    <td bgcolor="red">1</td>
    <td align="center">2</td>
    <td align="right">3</td>
  <tr>
    <td bgcolor="red" >1</td>
    <td align="center">2</td>
    <td align="right">3</td>
  </tr>
  <tr>
    <th><font color="blue">第3行</font></th>
    <td bgcolor="red">1</td>
    <td align="center">2</td>
    <td align="right">3</td>
  </tr>
</table>
```

显示效果：

<table border='1' align="center">
  <caption>跨行和跨列，带背景色和前景色</caption> 
  <tr>
    <td>&emsp;</td>
    <th colspan='2'><font color="red">跨2列</font></th>
    <th><font color="red">第3列</font></th>
  </tr>
    <th rowspan='2'><font color="blue">跨2行</font></th>
    <td bgcolor="red">1</td>
    <td align="center">2</td>
    <td align="right">3</td>
  <tr>
    <td bgcolor="red" >1</td>
    <td align="center">2</td>
    <td align="right">3</td>
  </tr>
  <tr>
    <th><font color="blue">第3行</font></th>
    <td bgcolor="red">1</td>
    <td align="center">2</td>
    <td align="right">3</td>
  </tr>
</table>

## 2.2 大纲或目录

**可以在新行用`[toc]`生成大纲**。显示效果：

[toc]

## 2.3 上标和下标
### 2.3.1 markdown
`文字^上角^`，显示如：文字^上角^。
`文字~下角~`，显示如：文字~下角~。

### 2.3.2 html

`文字<sup>上角</sup>`，显示如：文字<sup>上角</sup>

`文字<sub>下角</sub>`，显示如：文字<sub>下角</sub>


## 2.5 定义列表
### 2.5.1 markdown
```markdown
苹果
: 	一种水果
: 	一种品牌，计算机，手持设备
桔子
: 一种水果
```
显示效果：


苹果
:	一种水果
:	一种品牌，计算机，手持设备

桔子
:	一种水果

### 2.5.2 html

<dl> 
<dt>苹果</dt> 
<dd>一种水果</dd> 
<dd>一种品牌，计算机，手持设备</dd> 
<dt>桔子</dt> 
<dd>一种水果</dd> 
</dl> 

## 2.6 任务列表或TODO


任务列表,`- [] 任务`表示未完成`- [X] 任务`表示已完成， 任务列表也支持多级使用。例如：

```markdown
-  [ ] 未完成
-  [x] 已完成
-  [X] 已完成
```
显示效果：

-  [ ] 未完成
-  [x] 已完成
-  [X] 已完成


## 2.7 高级代码块

**在行首使用 \` \` \`…… \` \` \`表示高级代码块，在第一行的三个\` \` \` 后面可以添加对应语言关键字来实现语法高亮，此时语言关键字后面不需要加感叹号`！`，如**：
````markdown
```cpp 
int a = 10 ;
int b = 20; 

```
````
显示效果：
```cpp
int a = 10 ;
int b = 20;
```

支持的语言包括：

|语言|关键字|语言|关键字|
|--|--|--|--
|AppleScript|	applescript|普通文本|text , plain
|ActionScript 3.0|	actionscript3 , as3|Python|	py , python|
|Shell	|bash , shell|Ruby|	ruby , rails , ror , rb
|ColdFusion	|coldfusion , cf|SASS&SCSS|	sass , scss
|C|	cpp , c|Scala|	scala
|C#	|c# , c-sharp , csharp|SQL|	sql
|CSS|	css|Visual Basic|	vb , vbnet
|Delphi|	delphi , pascal , pas|XML|	xml , xhtml , xslt , html
|diff&patch|	diff patch|Objective C|	objc , obj-c
|Erlang|	erl , erlang|F#	|f# f-sharp , fsharp
|Groovy|	groovy|R|	r , s , splus
|Java|	java|matlab|	matlab
|JavaFX|	jfx , javafx|swift|	swift
|JavaScript	|js , jscript , javascript|GO	|go , golang
|Perl|	perl , pl , Perl
|PHP|	php

**增强代码块还支持根据特定的命令关键字实现特定的功能**。

|命令关键字|作用|
|--|--|
|flow|流程图|
|sequence|序列图|
|mermaid|mermaid流程图、序列图、甘特图

## 2.8 脚注

**内联脚注**: `内容[^id] ` ，然后在任意行首使用`[^id]:脚注内容`。显示效果：内容[^id]

[^id]:脚注内容

**引用脚注**：`正文内容^[脚注定义内容]`，显示效果：正文内容^[脚注定义内容]

**多行脚注**：允许在脚注定义里，进行多行定义，每行需要一个缩进，即至少两个空格或者一个制表符。只有引用式脚注支持多行脚注内容，内联式脚注法实现多行脚注内容定义。例如：

```
这是一个使用了多行脚注内容的文章.[^longnote] 
 
[^longnote]: 多行脚注定义 
 
    注意前面要有空格 
	在脚注里面定义数学公式 `!$x = {-b \pm \sqrt{b^2-4ac} \over 2a}.$` 
```
显示效果：

这是一个使用了多行脚注内容的文章[^longnote] 。
 
[^longnote]: 多行脚注定义 
 
    注意前面要有空格 
	在脚注里面定义数学公式 `!$x = {-b \pm \sqrt{b^2-4ac} \over 2a}.$` 
	

## 2.9 LaTeX数学公式
### 2.9.1 基本语法

行内数学公式：`$数学公式$`，如` $a^+b^2 $`，显示效果：$a^2+b^2$；
块级数学公式：`$$数学公式$$`，如`$$a^2+b^2 =c^2$$`，显示效果：$$a^2+b^2 =c^2$$


### 2.9.2 公式对齐

`\begin{align} ... \end{align}`，**使用&表示对齐位置，\\\表示换行，\tag{n}标签序号**。

```markdown
$$
\begin{align}
h(x) =& \frac{1}{\int_xt(x)\mathrm{d}x} \tag{1} \\
=& \frac{1}{\int_x\eta(x)\mathrm{d}x}g(x)\tag{2}
\end{align}
$$
```

显示效果：

![图2](https://gitee.com/liao20081228/blog_pictures/raw/master/markdown教程/2.jpg#pic_center)

`\begin{eqnarray} ...\end{eqnarray}`，**使用&表示对齐位置**：

```markdown
$$
\begin{eqnarray}
a & = & b + c \\
& = & d + e + f + g + h + i + j + k + l\\
&& +\: m + n + o \\
& = & p + q + r + s
\end{eqnarray}
$$
```
显示如下：

![图3](https://gitee.com/liao20081228/blog_pictures/raw/master/markdown教程/3.jpg#pic_center)

### 2.9.3 矩阵

**简单矩阵**：`$$\begin{matrix}…\end{matrix}$$`来生成矩阵，其中... 表示的是LaTeX 的矩阵命令，矩阵命令中每一行以  \   \\ 结束，矩阵的元素之间用&来分隔开。例如：
```markdown
 $$
  \begin{matrix}
   1 & 2 & 3 \\
   4 & 5 & 6 \\
   7 & 8 & 9
  \end{matrix} \tag{1}
$$
```
显示效果：

 $$
  \begin{matrix}
   1 & 2 & 3 \\
   4 & 5 & 6 \\
   7 & 8 & 9
  \end{matrix} \tag{5}
$$



**带括号的矩阵**：可以给矩阵加上括号，分为两种：使用`\left矩阵左括号 ... \right矩阵右括号 `(其中花括号要进行转义)或者把公式命令中的matrix 改成 `pmatrix（圆括号）、bmatrix（方括号）、Bmatrix（花括号）、vmatrix（行列式）、Vmatrix（双竖线）`等。例如：
```markdown
$$
 \left\{
 \begin{matrix}
   1 & 2 & 3 \\
   4 & 5 & 6 \\
   7 & 8 & 9
  \end{matrix}
  \right\} \tag{2}
$$
```

显示效果：

$$
 \left\{
 \begin{matrix}
   1 & 2 & 3 \\
   4 & 5 & 6 \\
   7 & 8 & 9
  \end{matrix}
  \right\} \tag{2}
$$

```markdown
$$
 \begin{pmatrix}
   1 & 2 & 3 \\
   4 & 5 & 6 \\
   7 & 8 & 9
  \end{pmatrix} \tag{4}
$$
```

显示效果：

$$
 \begin{pmatrix}
   1 & 2 & 3 \\
   4 & 5 & 6 \\
   7 & 8 & 9
  \end{pmatrix} \tag{4}
$$

```markdown
$$
 \begin{bmatrix}
   1 & 2 & 3 \\
   4 & 5 & 6 \\
   7 & 8 & 9
  \end{bmatrix} \tag{5}
$$
```
显示效果：

$$
 \begin{bmatrix}
   1 & 2 & 3 \\
   4 & 5 & 6 \\
   7 & 8 & 9
  \end{bmatrix} \tag{5}
$$

```markdown
$$
 \begin{Bmatrix}
   1 & 2 & 3 \\
   4 & 5 & 6 \\
   7 & 8 & 9
  \end{Bmatrix} \tag{5}
$$
```
显示效果：

$$
 \begin{Bmatrix}
   1 & 2 & 3 \\
   4 & 5 & 6 \\
   7 & 8 & 9
  \end{Bmatrix} \tag{5}
$$

```markdown
$$
 \begin{vmatrix}
   1 & 2 & 3 \\
   4 & 5 & 6 \\
   7 & 8 & 9
  \end{vmatrix} \tag{5}
$$
```
显示效果：

$$
 \begin{vmatrix}
   1 & 2 & 3 \\
   4 & 5 & 6 \\
   7 & 8 & 9
  \end{vmatrix} \tag{5}
$$

```markdown
$$
 \begin{Vmatrix}
   1 & 2 & 3 \\
   4 & 5 & 6 \\
   7 & 8 & 9
  \end{Vmatrix} \tag{5}
$$
```
显示效果：
$$
 \begin{Vmatrix}
   1 & 2 & 3 \\
   4 & 5 & 6 \\
   7 & 8 & 9
  \end{Vmatrix} \tag{5}
$$

**带省略符号的矩阵**：如果矩阵元素太多，可以使用\cdots \ddots \vdots 等省略符号来定义矩阵。例如：

```markdown
$$
\left[
    \begin{matrix}
        1      & 2      & \cdots & 4      \\
        7      & 6      & \cdots & 5      \\
        \vdots & \vdots & \ddots & \vdots \\
        8      & 9      & \cdots & 0      \\
    \end{matrix}
\right]
$$
```
显示效果：

$$
\left[
\begin{matrix}
 1      & 2      & \cdots & 4      \\
 7      & 6      & \cdots & 5      \\
 \vdots & \vdots & \ddots & \vdots \\
 8      & 9      & \cdots & 0
\end{matrix}
\right]
$$

**带参数的矩阵**：比如写增广矩阵，可能需要最右边一列单独考虑。可以用array命令来处理,其中\\begin{array}{cc|c}中的c表示居中对齐元素,|用来作为分割列的符号**。例如：

```markdown
$$ 
\left[
    \begin{array}{cc|c}
        1 & 2 & 3 \\
        4 & 5 & 6
    \end{array}
\right] \tag{7}
$$
```
显示效果：

$$ 
\left[
    \begin{array}{cc|c}
      1 & 2 & 3 \\
      4 & 5 & 6
    \end{array}
\right] \tag{7}
$$

**行间矩阵**：可以使用`\bigl(\begin{smallmatrix} ... \end{smallmatrix}\\bigr)`，例如：

```markdown
 $\bigl( \begin{smallmatrix} a & b \\ c & d \end{smallmatrix} \bigr)$
```
&emsp;&emsp;显示如下：![3](https://www.github.com/liao20081228/blog/raw/master/图片/markdown教程/3.jpg)

### 2.9.4 数学符号
**希腊字母**：如果使用大写的希腊字母，把命令的首字母变成大写即可，例如 \Gamma 输出的是 $\Gamma$。如果使用斜体大写希腊字母，再在大写希腊字母的LaTeX命令前加上var，例如\varGamma 生成$\varGamma$。

|命令|	显示|命令|	显示|命令|	显示|命令|	显示|
|--|--|--|--|--|--|--|--|
|\alpha|$\alpha$|\beta|$\beta$|\gamma|$\gamma$|\delta|$\delta$|
|\epsilon|	$\epsilon$|	\zeta|$\zeta$	|\eta	|$\eta$	|	\theta|	$\theta$
|\iota	|$\iota$	|	\kappa|$\kappa$	|\lambda	|$\lambda$	|	\mu|$\mu$
|\xi|	$\xi$	|	\nu	|$\nu$|\pi|	$\pi$	|	\rho|	$\rho$
|\sigma|$\sigma$	|		\tau|$\tau$|\upsilon|	$\upsilon$|		\phi|$\phi$
|\chi|	$\chi$	|	\psi|	$\psi$|\omega|	$\omega$|	\ell|$\ell$|		
	

**和号、积号、积分号、集合、极限**：

|命令|	显示	|	命令|	显示
|--|--|--|--|
|\sum|	$\sum$	|	\int|	$\int$
|\sum_{i=1}^{N}|$\sum_{i=1}^{N}	$|\int_{a}^{b}|	$\int_{a}^{b}$|
|\prod|	$\prod$|\iint|$\iint$
|\prod_{i=1}^{N}|$\prod_{i=1}^{N}$|	\iint_{a}^{b}|	$\iint_{D}$|
|\bigcup|$\bigcup$|		\iiint|	$\iiint$
|\bigcup_{i=1}^{N}|$\bigcup_{i=1}^{N}$|	\iiint_{a}^{b}|	$\iiint_{\bigcap}$|
|\bigcap|$\bigcap$|\lim|$\lim$|
|\bigcap_{i=1}^{N}|$\bigcap_{i=1}^{N}$|\lim_{x \to 0}	|$\lim_{x \to 0}$|


**三角函数和双曲函数**：

|命令|	显示	|	命令|	显示|
|--|--|--|--|--|--|--|--|
|\sin x|$\sin x$|\cos x|$\cos x$|
|\tan x|$\tan x$|\cot x|$\cot x$|
|\sec x|$\sec x$|\csc x|$\csc x$
|\arcsin x|$\arcsin x$|\arccos x|$\arccos x$|
|\arctan x|$\arctan x$|
|\sinh x|$\sinh x$|\cosh x|$\cosh x$|
|\tanh x|$\tanh x$|\coth x|$\coth x$|


**根号、上标、下标、分数、括号、空格**：

|命令|	显示|		命令|	显示|
|--|--|--|--|
|\sqrt[3]{2}|$\sqrt[3]{2}$|X^{3}	|$x^{3}$|
|\sqrt{2}|$\sqrt{2}$|x_{3}|	$x_3$|
|\frac{1}{2}	|$\frac{1}{2}$	|


**特殊数学符号**：

|命令|	显示|		命令|	显示|
|--|--|--|--|
|\dot x|$\dot x$|\ddot x|$\ddot x$|
|\hat x|$\hat x$|\widehat {XYZ}|$\widehat {xyz}$|
|\bar x|$\bar x$|\overline {xyz}|$\overline {xyz}$|
|\vec x|$\vec x$|\overrightarrow {xyz}|$\overrightarrow {xyz}$|
|||\overleftrightarrow {xyz}|$\overleftrightarrow {xyz}$|


**特殊符号**：

|命令|	显示|		命令|	显示|
|--|--|--|--|
|( )|$()$|[ ]|$[]$|
|\\{\ \}|$\{\}$|
|a\,b|$a\,b$|a\;b|$a\;b$|
|a\quadb|$a\quad b$|a\qquadb|$a\qquad b$|
|`\$`|$\$ $|\\_|$\_$|
|\backslash|$\backslash$|a\\\b|$a\\b$|
|{n+1 \choose 2k}|${n+1 \choose 2k}$|\binom{n+1}{2k}|$\binom{n+1}{2k}$|
|a\equiv b\pmod n|$a\equiv b\pmod n$|\tag x|$\tag x$|


**其他符号**：

|命令|	显示|命令|	显示|命令|	显示|		命令|	显示|命令|	显示|
|--|--|--|--|--|--|--|--|--|--|
|\lt|$\lt$|\gt|$\gt$|\le|$\le$|\ge|$\ge$|\neq|$\neq$|
|\not\lt|$\not\lt$|\not\gt|$\not\gt$|\not\le|$\not\le$|\not\ge|$\not\ge$|
|\times|$\times$| \div|$\div$| \pm|$\pm$| \mp |$\mp$|\cdot{xy}|$\cdot$| 
|\cup|$\cup$| \cap| $\cap$|\setminus|$\setminus$| 
|\subset|$\subset$|\subseteq |$\subseteq$|\subsetneq |$\subsetneq$|
|\supset|$\supset$|\supseteq |$\supseteq$|\supsetneq |$\supsetneq$|
|\in| $\in$|\notin|$\notin$ |\emptyset |$\emptyset$|\varnothing|$\varnothing$
|\to |$\to$|\mapsto| $\mapsto$|
|\rightarrow| $\rightarrow$|\leftarrow|$\leftarrow$|\Rightarrow|$\Rightarrow$|\Leftarrow|$\Leftarrow$| 
|\land |$\land$|\lor|$\lor$| \lnot |$\lnot$|\forall |$\forall$|\exists |$\exists$|
|\top |$\top$|\bot|$\bot$| \vdash |$\vdash$|\vDash |$\vDash$|
|\star|$\star$| \ast|$\ast$| \oplus|$ \oplus$| \circ|$\circ$| \bullet |$\bullet$|
|\approx |$\approx $|\sim |$\sim$|\simeq|$\simeq$|\cong|$\cong$|\equiv|$\equiv$
|\prec|$\prec$| \lhd|$\lhd$| \rhd|$\rhd$|\Re|$\Re$ |\Im|$\Im$ |
|\infty|$\infty$|\aleph_0|$\aleph_0$|\nabla |$\nabla$|\partial|$\partial$| 
|\ldots|$\ldots$|\cdots|$\cdots$ |\ddots |$\ddots$|\vdots|$\vdots$|


* 字体

|命令|字体名|显示|
|--|--|--|
|\mathbb 或 \Bbb | blackboard bold |$\mathbb  {ABCDEFGH}$|
|\mathbf |boldface|$\mathbf {ABCDEFGH}$|
|\mathtt |typewriter|$\mathtt {ABCDEFGH}$|
|\mathrm |roman|$\mathrm {ABCDEFGH}$
|\mathsf | sans-serif |$\mathsf {ABCDEFGH}$
|\mathcal| calligraphic|$\mathcal {ABCDEFGH}$
|\mathscr| script letters|$\mathscr {ABCDEFGH}$
|\mathfrak| Fraktur|$\mathfrak {ABCDEFGH}$


**分组**：同一个元素如果有个多个普通字符，使用大括号括起来表示同组：`{a+b}`

## 2.10 UML图

### 2.10.1 流程图
**定义流程图**：
````markdown
```flow!
... ...
... ...
```
````
**定义元素**：` tag=>type: text_content:>url[blank]`
* tag就是一个标签，在第二段连接元素时用
* type是流程图的基本类型
	* start：开始节点
	* end：结束节点
	* operation：处理节点
	* subroutine：子路径节点
	* condition：判断节点
	* inputoutput：输入输出节点
	* parallel：并行节点
* `:>url`：可点击的节点，可以省略。
* `[blank]`：在新窗口打开，默认当前窗口。

**连接元素**：`tag1(方向)->tag2(方向)->tag3....`

* 使用 `->` 来连接两个元素
* right,left：表示箭头在当前模块上的起点(默认箭头从下端开始)
* 对于condition类型，有yes和no两个分支，如示例中的cond(yes)和cond(no)
 
**示例**:  
````
```flow
st=>start: START:>www.baidu.com
e=>end: END
in=>inputoutput: input
out=>inputoutput: output
op=>operation: My Operation
sub=>subroutine: My subroute
cond=>condition: Yes or No?
io=>inputoutput: catch something

st->in->op->cond
cond(yes)->out->e
cond(no)->sub
sub(right)->op
```
````
显示效果：

```flow
st=>start: START:>www.baidu.com
e=>end: END
in=>inputoutput: input
out=>inputoutput: output
op=>operation: My Operation
sub=>subroutine: My subroute
cond=>condition: Yes or No?
io=>inputoutput: catch something

st->in->op->cond
cond(yes)->out->e
cond(no)->sub
sub(right)->op
```


### 2.10.2 序列图

````
```sequence
... ...
... ...

```
````
* 定义标题：`title: 标题`
* 定义对象（可以省略）：`participant 对象 [as 别名]`
* 定义注释：`note [left of|right of|over]  对象：注释 `
	* left of, 表示当前对象的左侧
	* right of, 表示当前对象的右侧
	* over, 表示覆盖在当前对象（们）的上面
* 定义动作：`对象1 箭头 对象2:内容`
	* -> 不带箭头的实线
	* --> 不带箭头的虚线
   * ->> 带箭头的实线
   * -->>	带箭头的虚线
   * -x 带x带箭头实线 (异步)
   * --x	带x带箭头虚线(异步)
* 激活/不激活：
	* 第一种：`activate|deactivate  对象 `
	* 第二种：定义动作时在对象2之前加`+`表示激活，加`-`表示不激活
* 循环：
```shell
loop Loop-text
... statements ...
end
```
* 替代路径alt或opt：
```shell
#如果有两条替代路径
alt Describing text
... statements ...
else Describing text
... statements ...
end

#如果只有一条替代路径
opt Describing text
... statements ...
end
```
* 示例
````markdown
```mermaid!
sequenceDiagram
title: 序列图sequence(示例)
participant A
participant B
participant C
participant D as test

note left of A: A左侧说明
note over B,C: 覆盖B,C的说明
note right of C: C右侧说明

A->A:自己到自己
A->B:实线不带箭头
A-->B:虚线不带箭头
A->>B:实线带箭头
A-->>B:虚线带箭头
A-xB:带x带箭头实线（异步）
A--xB:带x带箭头虚线（异步）
C->D:激活
activate C
C->D:不激活
deactivate C

C->+D:激活
C->+D:激活
C->-D:不激活
C->-D:不激活

loop 循环
B->C:测试循环
B->>C:测试循环
B-->C:测试循环
B-->>C:测试循环
end
A->>D:首选路径
alt 替代路径1
A->>C: 
C->>D: 
else 替代路径2
A->>B: 
B->>D: 
end
A-->>C: 首选路径
opt 一种替代路径
A-->>B: 
B-->>D: 
end
```
````

```mermaid!
sequenceDiagram
title: 序列图sequence(示例)
participant A
participant B
participant C
participant D as test

note left of A: A左侧说明
note over B,C: 覆盖B,C的说明
note right of C: C右侧说明

A->A:自己到自己
A->B:实线不带箭头
A-->B:虚线不带箭头
A->>B:实线带箭头
A-->>B:虚线带箭头
A-xB:带x带箭头实线（异步）
A--xB:带x带箭头虚线（异步）
C->D:激活
activate C
C->D:不激活
deactivate C

C->+D:激活
C->+D:激活
C->-D:不激活
C->-D:不激活

loop 循环
B->C:测试循环
B->>C:测试循环
B-->C:测试循环
B-->>C:测试循环
end
A->>D:首选路径
alt 替代路径1
A->>C: 
C->>D: 
else 替代路径2
A->>B: 
B->>D: 
end
A-->>C: 首选路径
opt 一种替代路径
A-->>B: 
B-->>D: 
end
```

 

# 3 扩展语法


## 3.6 Mermaid流程图

## 2.12 统计图
语法格式为：

````markdown
```plot!
{
"data": [ [[0, 0], [1, 1]] ],
"options": { "yaxis": { "max": 1 } }
}
```
````
显示如图：

![7](https://www.github.com/liao20081228/blog/raw/master/图片/markdown教程/7.jpg)


## 2.14 甘特图

* 定义甘特图： 
````markdown
```mermaid! 
gantt
... ...
... ...

```
````
* 定义时间格式：```dateFormat  时间格式 ```

|格式字符|举列	|描述|
|--|--|--|
|YYYY|	2014|4位数字的年|
|YY	|14	|2位数字的年|
|Y|	-25	|Year with any number of digits and sign|
|Q|	1,2,3,4|季度|
|M 或 MM|	1..12|数字月份|
|MMM 或 MMMM|Jan..December|英文名月份|
|D| DD|	1..31|该月的几号|
|Do|	1st..31st|一月的第几天|
|DDD 或 DDDD|	1..365|	一年的第几天|
|X|	1410715640.579|	Unix 时间戳，单位为秒|
|x|	1410715640579|	Unix 时间戳，单位为毫秒|
|gggg|	2014|当地的4位数字的年
|gg	|14|当地的2位数字的年
|w 或 ww|1..53|	一年的第几周|
|e|	0..6|一周的星期几（数字）|
|ddd 或 dddd|	Mon...Sunday|	一周的星期几（英文）|
|GGGG|	2014|	ISO 4位数字的年
|GG	|14	|ISO 2位数字的年
|W 或 WW	|1..53|	ISO 一年的第几周|
|E|	1..7|	ISO 一周的星期几（数字）|
|H 或 HH|0..23|24小时制的小时（0-23）
|h 或 hh|1..12|12小时制的小时（）
|k 或 kk|1..24|24小时制的小时，（1-24）
|a 或 A|am pm|	上午或下午
|m 或 mm|	0..59|	分
|s 或 ss|	0..59|	秒
|S 或 SS 或 SSS|	0..999|	小数秒
|Z 或 ZZ|+12:00	|与 UTC 的偏移量，如 +-HH:mm, +-HHmm, or Z


* 定义部门：`section section_name`
* 定义任务：`taskname :[crit][,active|done][,task_id][,start_time][,end_time]`
	 * done ：已完成
	 * active ：未完成
	 * crit：重要的，高亮显示
	 * id：唯一标识任务
	 * start：开始时间
		 * 缺省：前一个任务的结束时间
		 * 绝对时间：任务开始时间
		 * 相对时间：after task_id 表示某任务的结束时间
	 * end：结束时间
		 * 绝对时间：该任务的截止时间
		 * 持续时间：该任务需要多久完成
* 示例
````markdown
```mermaid!
gantt
    dateFormat YYYY-MM-DD
    title 项目开发流程
    section 项目确定
        需求分析       :done,a1, 2016-06-22, 14d
        可行性报告     :after a1, 5d
        概念验证       :crit,5d
    section 项目实施
        概要设计      :2016-07-05, 5d
        详细设计      :2016-07-08, 10d
        编码          :2016-07-15, 10d
        测试          :2016-07-22, 5d
    section 发布验收
        发布: 2d
        验收: 3d
```
````

```mermaid!
gantt
        dateFormat YYYY-MM-DD
        title 项目开发流程
    section 项目确定
        需求分析       :done,a1, 2016-06-22, 3d
        可行性报告     :after a1, 5d
        概念验证       :crit,5d
		
    section 项目实施
        概要设计      :2016-07-05, 5d
        详细设计      :2016-07-08, 10d
        编码          :2016-07-15, 10d
        测试          :2016-07-22, 5d
    section 发布验收
        发布: 2d
        验收: 3d
```
## 2.13 流程图


### 3.13.2 graph

* 定义流程图： 
````markdown
```mermaid! 
graph [方向]
... ...
... ...

```
````
* 方向:
	 * TD或TB（ top bottom）表示从上到下(缺省)
	 * BT（bottom top）表示从下到上
	 * RL（right left）表示从右到左
	 * LR（left right）表示从左到右

* 定义节点：```node[text1] ```
	 * 默认节点： A
	 * 矩形节点： B[bname]
	 * 圆角矩形节点： C(cname)
	 * 圆形节点： D((dname))
	 * 非对称节点： E>ename]
	 * 菱形节点： F{fname}

* 连接
     * A-->B 实线带箭头
     * A---B 实线不带箭头
     * A--text-->B或A-->|text|B 实线带箭头带标签
     * A--text---B或A---|text|B 实线不带箭头带标签
     * A-.->B 虚线带箭头
     * A-.-B 虚线不带箭头
     * A-.text.->B或A-.->|text|B  虚线带箭头带标签
     * A-.text.-B或A-.-|text|B  虚线不带箭头带标签

* 定义子流程图
```markdown
subgraph title
    graph definition
end
```
* 定义交互
	 * js回调：`click node callback "js函数名"`
	 * 打开连接：` click node "URL" "注释"`

* 示例
````markdown
```mermaid!
graph BT
A
B[矩形]
C(圆角矩形)
D{菱形}
E((圆形))
F>非对称]

A-->B 
A---B 
A--text-->B
A-->|text|B 
C--text---D
C---|text|D 
C-.->D
C-.-D
E-.text.->F
E-.->|text|F 
E-.text.-F
E-.-|text|F

G-->H
click H "http://www.baidu.com" "This is a link"
```
````

```mermaid!
graph TB
A
B[矩形]
C(圆角矩形)
D{菱形}
E((圆形))
F>非对称]

A-->B 
A---B 
A--text-->B
A-->|text|B 
C--text---D
C---|text|D 
C-.->D
C-.-D
E-.text.->F
E-.->|text|F 
E-.text.-F
E-.-|text|F

G-->H
click H "http://www.baidu.com" "This is a link"
```

### 2.10.2 序列图

* 定义序列图:
````markdown
```mermaid!
sequenceDiagram
... ...
... ...

```
	或







------

***<font color=blue>版权声明</font>：本文章参考了<font color=blue >《小书匠markdown 官方教程》。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处！！！</font>***

------
