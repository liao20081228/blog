---
title: vim插件——vim-surround 
tags: vim插件
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《vim-surround 官方文档》](https://github.com/tpope/vim-surround "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

<style>table{word-break:initial;}</style>




# 1 简介
* **插件介绍**：删除，修改，插入成对符号。
* **仓库地址**：<https://github.com/tpope/vim-surround>

# 2 安装
* `$vim ~/.vimrc`
* 在`call vundle#begin()`和`call vundle#end()`之间添加`Plugin 'tpope/vim-surround'`
* `:wq`
* `$vim`
* `:PluginInsttall`
 

# 3 使用
<table border=1>
    <tr>
        <th>普通模式</th>
        <th>描述</th>
        <th>原文本</th>
        <th>命令</th>
        <th>新文本</th>
    </tr>
    <tr>
        <td>ds 目标符号</td>
        <td>删除成对符号</td>
        <td>hellow,(world)!</td>
        <td>ds(</td>
        <td>hellow,world!</td>
    </tr>
    <tr>
         <td>cs 目标符号或文本对象界定符 替代符号</td>
         <td>修改成对符号</td>
         <td>hellow),(world)!</td>
        <td>cs({</td>
        <td>hellow,{ world }!</td>
    </tr>
    <tr>
        <td>ys 文本对象 插入符号</td>
        <td>添加成对符号</td>
        <td>hellow,world!</td>
        <td>ysiw(</td>
        <td>hellow,(world)!</td>
    </tr>
     <tr>
        <td>yss 插入符号</td>
        <td>为整行添加成对符号</td>
        <td>hellow,world!</td>
        <td>yss(</td>
        <td>(hellow,world!)</td>
    </tr>
    <tr>
        <td>yS 文本对象 插入符号</td>
        <td>类似ys，但要换行和缩进</td>
        <td>hellow,world!</td>
        <td>ySiw(</td>
        <td>hellow,(<br>&emsp;&emsp;world<br>)!</td>
    </tr>
    <tr>
        <td>ySS 插入符号</td>
        <td>类似yss，但要换行和缩进</td>
        <td>hellow,world!</td>
        <td>ySS(</td>
        <td>(<br>&emsp;&emsp;hellow,world<br>)!</td>
    </tr>
    <tr>
        <th>可视模式</th>
         <th>描述</th>
        <th>原文本</th>
        <th>命令</th>
        <th>新文本</th>
    </tr>
    <tr>
        <td>S 插入符号</td>
        <td>类似ys</td>
        <td>hellow,world!</td>
        <td>S(</td>
        <td>hellow,(world)!)</td>
    </tr>
    <tr>
        <td>gS 插入符号</td>
        <td>添加成对符号，并换行和缩进</td>
        <td>hellow,world!</td>
        <td>gS(</td>
        <td>hellow,(<br>&emsp;&emsp;world<br>)!)</td>
    </tr>
    <tr>
        <th>插入模式</th>
         <th>描述</th>
        <th>原文本</th>
        <th>命令</th>
        <th>新文本</th>
    </tr>
    <tr>
        <td>CTRL-g s 插入符号</td>
        <td>添加成对符号</td>
        <td></td>
        <td>>CTRL+g sb</td>
        <td>()</td>
    </tr>
        <tr>
        <td>CTRL-g S 插入符号</td>
        <td>添加成对符号,并换行</td>
        <td></td>
        <td>>CTRL+g Sb</td>
        <td>(<br><br>)</td>
    </tr>
</table>

* 目标符号
  * 可以是任意的成对的英文符号中的一个
  * 可以是文本对象界定符b，B，r，a，t
* 插入符号和替代符号
  * 可以是任意中、英符号
  * 可以是s，表示只在左边插入空格
  * 可以是<或t，表示自定义html标签，要插入或替换为尖括号，请使用>或a
  * 可以是文本对象界定符b，B，r，a，t
  * 如果为{，[，(，则会在文本两边自动添加一个空格
 
* 文本对象
  * 文本对象由`对象范围+文本对象界定符`组成
  * 对象范围
     * a表示包括界定符号
     * i表示不包括界定符
 * 文本对象界定符
     * b，(，)表示圆括号
     * B，{，}表示花括号
     * r，[，]表示方括号
     * a，<，>表示尖括号
     * " 表示双引号
     * ' 表示单引号
     * ` 表示反引号
     * t 表示html标签
     * w 表示单词，单词间由不是字母，数字，下划线的其余字符分隔
     * W 表示字串，字串间由空白符（空格，制表，换行）分隔
     * s 表示句子
     * p 表示段落

|文本对象|选择区域|文本对象|选择区域|
|:--|:--|:--|:--|
|a)或ab或a(|一对圆括号|aw	|当前单词及一个空格
|i)或ib或i(|圆括号内部|iw	|当前单词
|a}或aB或a{	|一对花括号|aW	|当前字串及一个空格
|i}或iB或i{	|花括号内部|iW	|当前字串
|a]或ar或a[|一对中括号|as	|当前句子及一个空格
|i]或ir或i[|中括号内部|is	|当前句子
|a>或aa或a<	|一对尖括号|ap	|当前段落及一个空行
|i>或ia或i<	|尖括号内部|ip	|当前段落
|a"|一对双引号
|i"|双引号内部
|a'|一对单引号
|i'|单引号内部
|a`|一对反引号
|i`|反引号内部
|at	|一对XML标签
|it	|XML标签内部

# 4 选项
* `let g:surround_ascii码 = "要映射的字符串"`
  * 将ascii字符映射为特定的字符
  * `\r` 表示原文本
  * "、'、\需要进行转义
  * `\n提示字符\r正则表达式\r\n`表示等待键盘输入，并过滤掉与正则表达式匹配的内容，然后放在一对\n之间，n最大为7
     * 提示字符可以省略
     * 第一个`\r`表示正则表达式，可省略
     * 第二个`\r`表示过滤后的字符，可省略
  * 示列：`let g:surround_116 = "<\1please input:\r[^a-z].*\r\1>\r</\1\r[^a-z].*\r\1>"`
* `let g:surround_insert_tail = "字符串"`
  * 插入模式中，在插入指定符号后自动添加一个字符串



------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《vim-surround 官方文档》](https://github.com/tpope/vim-surround "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
