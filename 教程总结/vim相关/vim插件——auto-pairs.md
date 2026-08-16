---
title: vim插件——auto-pairs
tags: vim插件
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《auto-pairs 官方文档》](https://github.com/jiangmiao/auto-pairs "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

<style>table{word-break:initial;}</style>

# 1 简介
* **插件介绍**：在输入/删除左括号时，能自动补上/删除右括号。
* **仓库地址**：<https://github.com/jiangmiao/auto-pairs>


# 2 安装
*  打开`~/.vimrc`
* 在`call vundle#begin()`和`call vundle#end()`之间添加`Plugin 'jiangmiao/auto-pairs'`
* 保存并退出。
* 打开vim
* 在命令行中输入`PluginInsttall`
* 显示done之后，退出vim


# 3 使用
<table border=1>
<caption>字符'|'表示光标所在位置</caption>
    <tr>
        <th>功能</th>
        <th>支持</th>
        <th>原文本</th>
        <th>按键</th>
        <th>新文本</th>
    </tr>
    <tr>
        <td>成对插入</td>
        <td>{}，[]，()，""，''，``</td>
        <td></td>
        <td>[</td>
        <td>[|]</td>
    </tr>
    <tr>
        <td>成对删除</td>
        <td>{}，[]，()，""，''，``</td>
        <td>foo[|]</td>
        <td>BACKSPACE</td>
        <td>foo|</td>
    </tr>
    <tr>
        <td>换行并自动缩进</td>
        <td>{}，[]，()</td>
        <td>node{|}</td>
        <td>ENTER</td>
        <td>node{<br>&emsp;&emsp;|<br>}</td>
    </tr>
    <tr>
        <td>在括号内两侧各插入空格</td>
        <td>{}，[]，()</td>
        <td>foo{|}</td>
        <td>SPACE</td>
        <td>foo{ | }</td>
    </tr>
    <tr>
        <td>词后单引号不成对插入</td>
        <td>'</td>
        <td>foo|</td>
        <td>'</td>
        <td>foo'|</td>
    </tr>
    <tr>
        <td>跳过右括号</td>
        <td>{}，[]，()</td>
        <td>[ foo| ]</td>
        <td>]</td>
        <td>[ foo ]|</td>
    </tr>
    <tr>
        <td>在转义符\后禁用插件</td>
        <td>{}，[]，()，""，''，``</td>
        <td>foo\|</td>
        <td>{</td>
        <td>foo\{|</td>
    </tr>
    <tr>
        <td>对字符串加小括号</td>
        <td>C风格字符串</td>
        <td>|'foo'</td>
        <td>ALT+e</td>
        <td>('foo')|</td>
    </tr>
    <tr>
    <td>删除重复成对符号</td>
        <td>{}，[]，()，''，""，``</td>
        <td>foo'''|'''</td>
        <td>BACKSPACE</td>
        <td>foo|</td>
    </tr>
    <tr>
     <td>飞行模式，跳出括号对而不插入</td>
        <td>{}，[]，()</td>
        <td>if(a[3|])</td>
        <td>)</td>
        <td>if(a[3])|</td>
    </tr>
     <tr>
     <td>撤销飞行模式，插入而不是跳出括号对</td>
        <td>{}，[]，()</td>
        <td>if(a[3])|</td>
        <td>ALT+b</td>
        <td>if(a[3])|</td>
    </tr>
</table>

# 4 选项
* `let g:AutoPairs = {'(':')', '[':']', '{':'}',"'":"'",'"':'"'}`
  设置要自动配对的符号
  
* `let g:AutoPairs['<']='>'`
  添加要自动配对的符号\<>
  
* `let b:AutoPairs = g:AutoParis`
 设置要自动配对的符号，默认为`g:AutoPairs`，可以通过自动命令来对不同文件类型设置不同自动匹配对符号。
 
* `let g:AutoPairsShortcutToggle = '<M-p>'`
  设置插件打开/关闭的快捷键，默认为ALT+p。
  
* `let g:AutoPairsShortcutFastWrap = '<M-e>'`
   设置自动为文本添加圆括号的快捷键，默认为ALT+e。
   
* `let g:AutoPairsShortcutJump = '<M-n>'`
   设置调到下一层括号对的快捷键，默认为ALT+n。
   
* `let g:AutoPairsShortcutBackInsert = '<M-b>'`
 设置撤销飞行模式的快捷键，默认为ALT+b。
 
* `let g:AutoPairsMapBS = 1`
 把BACKSPACE键映射为删除括号对和引号，默认为1。
 
* `let g:AutoPairsMapCh = 1`
 把ctrl+h键映射为删除括号对和引号，默认为1。
 
* `let g:AutoPairsMapCR = 1`
 把ENTER键映射为换行并缩进，默认为1。
 
* `let g:AutoPairsCenterLine = 1 `
 当`g:AutoPairsMapCR`为1时，且文本位于窗口底部时，自动移到窗口中间。
 
* `let g:AutoPairsMapSpace = 1`
 把SPACE键映射为在括号两侧添加空格，默认为1。
 
* `let g:AutoPairsFlyMode = 0`
 启用飞行模式，默认为0。
 
* `let g:AutoPairsMultilineClose = 1`
 启用跳出多行括号对，默认为1，为0则只能跳出同一行的括号。



------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《auto-pairs 官方文档》](https://github.com/jiangmiao/auto-pairs "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
