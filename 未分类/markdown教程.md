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

## 1.10 代码和代码块

**句内代码用 \`……\`表示**。例如\`int a=2\`，显示效果：`int a=2`。

**代码块的每一行用四个空格或一个tab开头**。例如：
```
	int a=10;
	int b=20;
```
显示效果：

	int a=10;
	int b=20;

## 1.11 HTML标签

只要支持GFM的markdown都兼容html标签


## 1.12 语法嵌套

<span id=‘test’>所有的行内语法和几乎所有的多行语法都可以在其他语法中套用，比如加粗可在标题中使用。</span>
# 2 高级语法
|markdown版本|cmd|csdn|xiaoshujiang|
|:--|:--|:--|:--|
|任务列表|支持|支持|支持|
|数学公式|支持|支持|支持|
|加强代码块|支持|支持|支持|
|代码块设置行号|||支持|
|目录|支持|支持|支持|
|思维导图目录|||支持|
|脚注|支持|支持|支持|
|缩进|支持|支持|支持|
|下划线|不支持|支持|支持|
|文字高亮|不支持|支持|支持|
|页内跳转|支持|||
|锚点跳转||支持|支持|
|标签|支持||支持|
|字体、颜色、大小||支持|支持|
|视频|||支持|
|音频|||支持|
|附件|||支持|
|段代码文字格式|||支持|
|流程图|支持|支持|支持|
|序列图|支持|支持|支持|
|统计图|||支持|
|mermaid 流程图 序列图 甘特图|支持|支持|支持|
|html|支持|支持|支持|
|思维导图|||支持|

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

**支持参数depth, 表示显示到第几层**`[toc?depth=2]`，显示如下：

![图1](https://gitee.com/liao20081228/blog_pictures/raw/master/markdown教程/1.jpg#pic_center)

**支持显示成思维脑图大纲**`[toc!]`，显示如下：


**思维脑图大纲支持其它一些参数**：`[toc!?depth=n&direction=v|h|lr|rl|tb|bl&]colors=颜色值[,颜色值1][,颜色值2][,颜色值3]&theme=gray|colorful]`
* depth，表示显示到第几层
* direction参数设置布局，取值可以是：
	* v: 根结点在中间，分支均匀分布在垂直上下两侧
	* h: 根结点在中间，分支均匀分布在水平左右两侧
	* lr: 根结点在所有分支的左侧
	* rl: 根结点在所有分支的右侧
	* tb: 根结点在所有分支的上侧
	* bt: 根结点在所有分支的下侧
* colors参数设置分支颜色，必须是合法的十六进制颜色值，注意去掉`#`符号。
* theme参数设置主题，取值可以为：
	* colorful: 系统默认的彩色分支。如果用户指定了 colors 参数值， 则 theme 里面的颜色就会失效。
	* gray: 黑白效果的分支。

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
**高级代码块支持是否显示行号，默认会显示**：
````markdown
```语言关键字?linenums[=true|false]
... ...
... ...

```
````

**高级代码块支持设置起始行号**：
````markdown
```语言关键字?linenums=起始行号
... ...
... ...

```
````
**高级代码块支持单独高亮某些行**：
````markdown
```语言关键字?linenums&fancy=行号列表
... ...
... ...

```
````
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
|mathjax!|数学公式|
|flow或flow!|流程图|
|plot!|统计图|
|sequence或sequence!|序列图|
|mermaid!|mermaid流程图、序列图、甘特图
|mindmap!|思维导图


# 3 扩展语法
------

***<font color=blue>版权声明</font>：本文章参考了<font color=blue >《小书匠markdown 官方教程》。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处！！！</font>***

------
