---
title: vim插件——UltiSnips中文手册
tags: vim插件
---



------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>[《UltiSnips官方文档-2019-7-29》](https://github.com/SirVer/ultisnips/blob/master/doc/UltiSnips.txt)。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------


# 1 描述
UltiSnips为Vim编辑器提供**片段**管理。 所谓**片段就是一个经常重复使用的或包含大量相同内容的一段文本**。UltiSnips使您只需少量的击键就可以插入片段。片段在结构化文本中很常见，例如源代码，但也可以用于一般编辑，例如在电子邮件中插入签名或在文本文件中的插入当前日期。

@SirVer 发布了几段简短的截图，对UltiSnips做了很好的介绍，说明了它的特性和用法。

<http://www.sirver.net/blog/2011/12/30/first-episode-of-ultisnips-screencast/>
<http://www.sirver.net/blog/2012/01/08/second-episode-of-ultisnips-screencast/>
<http://www.sirver.net/blog/2012/02/05/third-episode-of-ultisnips-screencast/>
<http://www.sirver.net/blog/2012/03/31/fourth-episode-of-ultisnips-screencast/>

同样优秀的[vimcast](http://vimcasts.org)为UltiSnips奉献了三集视频:

<http://vimcasts.org/episodes/meet-ultisnips/>
<http://vimcasts.org/episodes/ultisnips-python-interpolation/>
<http://vimcasts.org/episodes/ultisnips-visual-placeholder/>

## 1.1 要求
这个插件适用于Vim 7.4或更高版本。只有在没有设置“compatible”设置时，它才会工作。

这个插件在Vim 7.4和8中分别针对Python 2.7和3.6进行了测试。所有其他python版本和vim版本不支持，但可能会工作。

Python 2.x或Python 3.x接口必须可用。 换句话说，Vim必须使用 +python 或 +python3 特性进行编译。以下命令显示如何测试Vim是否使用了python特性进行编译。如果使用了，则打印 '1'，否则打印 '0'。

测试是否VIm使用了python2.x进行编译：
&emsp;&emsp;`:echo has("python")`
在vim中可以使用如下命令来显示python2的版本：
&emsp;&emsp;`:py import sys; print(sys.version)`

测试是否VIm使用了python3.x进行编译：
&emsp;&emsp;`:echo has("python3")`
在vim中可以使用如下命令来显示python3的版本：
&emsp;&emsp;`:py3 import sys; print(sys.version)`


注意，Vim可能没有使用您在系统中安装的python版本，所以一定要检查Vim内部的python版本。

UltiSnips 尝试自动检测Vim中编译时使用了哪个python版本。不幸的是，在Vim的某些版本中，这种检测不起作用。在这种情况下，您必须在`vimrc`中使用全局变量“UltiSnipsUsePythonVersion”显式地告诉UltiSnips使用哪个版本python。

要使用python 2.x：
&emsp;&emsp; `let g:UltiSnipsUsePythonVersion = 2`

要使用python 3.x：
&emsp;&emsp; `let g:UltiSnipsUsePythonVersion = 3`

## 1.2 致谢

UltiSnips的灵感来自TextMate(<http://macromates.com/>)的片段功能，TextMate是Mac OS x的GUI文本编辑器，vim中管理片段并不是什么新鲜事。我想感谢,snipMate的作者Michael Sanders，我从他的插件中借用的一些实现细节，感谢他允许我使用他的代码片段。

# 2  安装和更新

获得UltiSnips的推荐方法是跟踪github上的 SirVer/ UltiSnips 。主分支总是稳定的。

## 2.1 使用Pathogen

如果你是Pathogen使用者，你可以追踪github上UltiSnips 的官方镜像。

&emsp;&emsp;`$ cd ~/.vim/bundle && git clone git://github.com/SirVer/ultisnips.git`
 
如果你还想要默认的snippets文件（**该文件的作用是定义要片段，是一个或多个片段的集合**），也可以跟踪：

&emsp;&emsp;`$ cd ~/.vim/bundle && git clone git://github.com/honza/vim-snippets.git`

有关如何更新包的详细信息，请参阅Pathogen文档。
## 2.2 使用一个下载包

下载包并解压缩到您选择的目录中，然后把下面一行添加到`vimrc`文件中，目的是将这个目录添加到Vim运行时路径。

&emsp;&emsp; `set runtimepath+=~/.vim/ultisnips_rep`

UltiSnips还需要ftdetect/目录中的Vim文件。不幸的是，Vim只允许`.Vim`目录中的这个目录。因此，您必须链接或复制这个文件：
&emsp;&emsp;`mkdir -p ~/.vim/ftdetect/`
&emsp;&emsp;`ln -s ~/.vim/ultisnips_rep/ftdetect/* ~/.vim/ftdetect/`

重启Vim和UltiSnips应该可以工作。要获得帮助，请使用

&emsp;&emsp;`:helptags ~/.vim/ultisnips_rep/doc`
&emsp;&emsp;`:help UltiSnips`

UltiSnips没有默认snippets文件。可以在这里找到一组优秀的snippets文件:<https://github.com/honza/vim-snippets>

# 3 设置和命令
## 3.1 命令
### 3.1.1 UltiSnipsEdit

UltiSnipsEdit命令为当前文件类型打开一个私有的片段定义文件（**也就是snippets文件，其作用就是定义触发时要插入的片段**）。如果不存在，则新建。如果使用`UltiSnipsEdit!` ，所有公共的片段定义文件也被考虑在内。如果多个文件匹配搜索，用户可以选择文件。

有几个变量与UltiSnipsEdit命令相关联。

- `g:UltiSnipsEditSplit `       定义打开编辑窗口的方式. 可能值为：
	- ==normal==        默认，在当前窗口中打开。
	- ==tabdo==    在标签（tab）中打开窗口。
	- ==horizontal==     水平分割口
	- ==vertical==      垂直拆分窗扣。                            
	- ==context==       依赖于环境，垂直或水平拆分窗口
	


- `g:UltiSnipsSnippetsDir`   指定私有的片段定义文件的存放目录。例如，如果变量设置为“~/.vim /mydir/UltiSnips”，当前的“filetype”为“cpp”， 如果文件不为空则`:UltiSnipsEdit`将打开`~/.vim/mydir/UltiSnips/cpp.snippets`，如果为空：UltiSnipsEdit将在`g:UltiSnipsSnippetDirectories`查找文件，如果没有找到，`:UltiSnipsEdit`将在`~/.vim/mydir/UltiSnips`创建新的片段定义文件。注意，名为“snippets”的目录是为snipMate的 片段定义文件保留的，不能使用。此外，此设置不影响搜索片段定义文件的位置，因此可以将此设置更改为UltiSnips无法找到的打开文件。有关详细信息，请参见==UltiSnips片段定义文件如何被加载==。

- `g:UltiSnipsSnippetDirectories`    指定查找片段定义文件的目录。不要混淆这个变量与上面的变量。有关详细信息，请参见==UltiSnips片段定义文件如何被加载==。

- `g:UltiSnipsEnableSnipMate`   允许在`＆runtimepath`中查找SnipMate 的片段定义文件。 UltiSnips将只在名为“snippets”的目录搜索SnipMate 的片段定义文件。该变量默认值为“1”，所以UltiSnips会查找SnipMate 的片段定义文件。

### 3.1.2  UltiSnipsAddFiletypes

`UltiSnipsAddFiletypes` 命令允许显式合并其它文件类型的片段到当前缓冲区。 例如，如果您正编辑.rst文件，但也希望Lua的片段可用，您可以发出命令：

&emsp;&emsp;`:UltiSnipsAddFiletypes rst.lua`

使用点分语法。此列表中的第一个文件类型将是UltiSnipsEdit中使用的文件类型，因此顺序很重要，列表按优先级排序。例如您可以将其添加到您的`ftplugin/rails.vim`中：

&emsp;&emsp;`:UltiSnipsAddFiletypes rails.ruby`

我先提到rails是因为我想在使用`UltiSnipsEdit`时编辑rails的片段定义文件，这是因为rails的 片段覆盖了等价的ruby 片段。 优先级现在成了rails -> ruby -> all。 如果您有一些优先级低于您的ruby snippets的特殊programming 片段定义文件，你可以调用：

&emsp;&emsp;`:UltiSnipsAddFiletypes ruby.programming`

那么优先级就是rails --> ruby --> programming --> all。

## 3.2 触发器
### 3.2.1 触发器按键映射

你可以通过设置全局变量来定义用于触发UltiSnips操作的按键。定义按键的变量用于展开片段，在片段中向前跳转 、向后跳转，列出当前上下文中的所有可用片段。请注意，某些终端仿真器不会将`<c-tab>等其他组合键`发送到正在运行的程序。具有默认值的变量有：

&emsp;&emsp;`g:UltiSnipsExpandTrigger="<TAB>"  "展开代码片`
&emsp;&emsp;`g:UltiSnipsJumpForwardTrigger="<c-j>"     "后一个 `
&emsp;&emsp;`g:UltiSnipsJumpBackwardTrigger="<c-k>"  "前一个`
&emsp;&emsp;`g:UltiSnipsListSnippets="<c-tab>"         "打开代码片段列表`

UltiSnips只会在片段处于活动状态时映射跳转触发器，以尽可能少地与其他映射进行干扰。

`g：UltiSnipsJumpBackwardTrigger`的默认值会干扰内置的补全功能 ==i_CTRL-X_CTRL-K==。解决方法是将以下内容添加到您的vimrc文件中，或者切换到像Supertab或youcompleteme这样的插件。

&emsp;&emsp;`inoremap <c-x><c-k> <c-x><c-k>`

### 3.2.2 使用自己的触发函数
对于高级用户，有四个函数可以直接映射到按键，它们对应于前面定义的一些触发器变量:

&emsp;&emsp;`g:UltiSnipsExpandTrigger        <--> UltiSnips#ExpandSnippet`
&emsp;&emsp;`g:UltiSnipsJumpForwardTrigger   <--> UltiSnips#JumpForwards`
&emsp;&emsp;`g:UltiSnipsJumpBackwardTrigger  <--> UltiSnips#JumpBackwards`

如果你让`g：UltiSnipsExpandTrigger`和`g：UltiSnipsJumpForwardTrigger`设置相同值，那么你实际要使用的函数是`UltiSnips＃ExpandSnippetOrJump`

每次调用`UltiSnips#ExpandSnippet`、`UltiSnips#ExpandSnippetOrJump`、`UltiSnips# jumpforward`或`UltiSnips#JumpBackwards`函数时，都会设置一个全局变量，其中包含相应函数的返回值。

对应的变量和函数是：

&emsp;&emsp;`UltiSnips#ExpandSnippet       --> g:ulti_expand_res (0: fail, 1: success)`
&emsp;&emsp;`UltiSnips#ExpandSnippetOrJump --> g:ulti_expand_or_jump_res (0: fail,1: expand, 2: jump)`
&emsp;&emsp;`UltiSnips#JumpForwards        --> g:ulti_jump_forwards_res (0: fail, 1: success)`
&emsp;&emsp;`UltiSnips#JumpBackwards       --> g:ulti_jump_backwards_res (0: fail, 1: success)`

为了查看这些返回值如何派上用场，假设一个情景：您想要映射一个要展开或跳转的键，但是如果这些操作都没有成功，那么您想要调用另一个函数。UltiSnips已经为supertab自动做到了这一点，但这允许您对TAB键的使用进行单独的微调。

用法如下:定义一个函数
```vim
	let g:ulti_expand_or_jump_res = 0 "default value, just set once
	function! Ulti_ExpandOrJump_and_getRes()
		 call UltiSnips#ExpandSnippetOrJump()
		 return g:ulti_expand_or_jump_res
	endfunction
```
然后定义映射如下：

&emsp;&emsp;`inoremap <NL> <C-R>=(Ulti_ExpandOrJump_and_getRes() > 0)?"":IMAP_Jumpfunc('', 0)<CR>`

如果你不能从当前位置展开或跳转，那么另一个函数IMAP_Jumpfunc('', 0)将被调用。

### 3.2.3 自定义自动命令

注意，自动命令不能以任何方式更改缓冲区。如果添加、删除、更改了行，则可能会使UltiSnips迷茫从而打乱你的片段内容。

为了最大限度地与其他插件兼容，UltiSnips在片段开始展开时设置了一些特殊的状态，包括映射和自动命令，并在最后一个片段退出后将它们删除。为了能够覆盖这些“内部”设置，它将触发以下“User”自动命令：

&emsp;&emsp;`UltiSnipsEnterFirstSnippet`
&emsp;&emsp;`UltiSnipsExitLastSnippet`

例如，要调用一对自定义函数来响应这些事件，您可以这样做：
```vim
   autocmd! User UltiSnipsEnterFirstSnippet
   autocmd User UltiSnipsEnterFirstSnippet call CustomInnerKeyMapper()
   autocmd! User UltiSnipsExitLastSnippet
   autocmd User UltiSnipsExitLastSnippet call CustomInnerKeyUnmapper()
```

注意片段展开可能是嵌套的，在这种情况下，==UltiSnipsEnterFirstSnippet==只在第一个(最外层)片段输入时触发，而==ultisnipsenterlastsnippet==只在最后一个(最外层)代码片段退出时触发。

### 3.2.4 直接使用python API

对于更高级的用法，您可以使用Ultisnip的python模块直接编写python函数。这是一个内部的不稳定的API，不推荐。

下面是一个展开片段的小示例函数:

```vim
   function! s:Ulti_ExpandSnip()
   Python << EOF
   import sys, vim
   from UltiSnips import UltiSnips_Manager
   UltiSnips_Manager.expand()
   EOF
   return ""
   endfunction
```

### 3.2.5 关于select模式映射的警告
Vim对==mapmode-s==的帮助文档写到：

&emsp;&emsp;注意:在select模式下映射可打印字符可能会使用户感到困惑。对于可打印的字符，最好显式地使用`:xmap`和`:smap`。或定义映射后使用`:sunmap`。

然而，大多数Vim插件，包括一些默认的Vim插件，都不支持这一点。UltiSnips使用Select模式在片段中标记制表符，以便进行替换、覆盖。现有的visual+select模式映射将会干扰。因此，UltiSnips发出==sunmap==命令来删除每个可打印字符的select模式映射。不涉及其他映射，特别是，UltiSnips不会更改现有nomal、insert或visual模式的映射。

如果不需要这种行为，可以通过在vimrc文件中添加这一行来禁用它：
&emsp;&emsp;`let g:UltiSnipsRemoveSelectModeMappings = 0`

如果只对特定映射禁用此功能，请将它们添加到要忽略的映射列表中。例如，vimrc文件中的以下几行将取消映射所有select模式映射，除了那些包含字符串“somePlugin”或字符串“otherPlugin”的映射，这些映射的完整定义由==:smap==命令列出。

&emsp;&emsp;`let g:UltiSnipsRemoveSelectModeMappings = 1`
&emsp;&emsp;`let g:UltiSnipsMappingsToIgnore = [ "somePlugin", "otherPlugin" ]`

## 3.4  函数
UltiSnips 提供一些函数来扩展核心功能。

### 3.4.1  UltiSnips#AddSnippetWithPriority

第一个函数是 `UltiSnips#AddSnippetWithPriority(trigger, value, description,options, filetype, priority)`，它添加一个新的片段到当前的片段列表。有关大多数函数参数的详细信息，请参阅==UltiSnips编写片段定义文件==。优先级是一个数字，它定义应该优先选择哪个片段。请参阅==UltiSnips添加片段定义文件==中的priority关键字。

### 3.4.2 UltiSnips#Anon 
第二个函数是 `UltiSnips#Anon(value, ...)`，它展开了一个匿名的片段，匿名片段是当场定义的，展开后立即丢弃。匿片段不会被添加到全局片段文件列表中，因此除非再次调用该函数，否则不能再次展开它们该函数按顺序接受三个可选参数：trigger,description, options。参数与`UltiSnips#AddSnippetWithPriority`函数中的同名参数一致。参数trigger和options可以改变片段展开的方式。可以在片段定义中指定相同的options。在==UltiSnips片段选项==参见options的完整列表。description参数目前未使用。

一个用例可能是这行来自于一个reStructuredText插件文件的这一行：
```vim
`inoremap <silent> $$ $$<C-R>=UltiSnips#Anon(':latex:\`$1\`', '$$')<cr>`
```

这将在键入两个\$符号时展开片段。
注意：映射的右侧以'\$\$'触发器的迅速重复键入开始，并将'\$\$'作为触发参数传递给该函数。这是为了让UltiSnips能够访问输入的字符，以便确定触发器是否匹配。创建这样一个片段的更优雅的方法是使用==UltiSnips自动触发==。

### 3.4.3 UltiSnips#SnippetsInCurrentScope

UltiSnips#SnippetsInCurrentScope返回触发器与当前单词匹配的所有片段 的vim字典。如果需要当前缓冲区的所有片段信息，可以简单地将1(意味着all)作为该函数的第一个参数传递，并使用全局变量`g:current_ulti_dict_info`来获得结果(参见下面的示例)。

此函数不直接向UltiSnips添加任何新功能，但允许使用第三方插件集成当前可用的片段。例如，所有与UltiSnips集成的补全插件都使用这个函数。

关于如何使用这个函数的另一个例子是考虑下面的函数和映射定义：
```vim
function! ExpandPossibleShorterSnippet()
  if len(UltiSnips#SnippetsInCurrentScope()) == 1 "only one candidate...
    let curr_key = keys(UltiSnips#SnippetsInCurrentScope())[0]
    normal diw
    exe "normal a" . curr_key
    exe "normal a "
    return 1
  endif
  return 0
endfunction
inoremap <silent> <C-L> <C-R>=(ExpandPossibleShorterSnippet() == 0? '': UltiSnips#ExpandSnippet())<CR>
```
如果您的片段的触发器是lorem，您输入lor，并且没有其他片段的触发器与lor匹配，那么按`<C-L>`将展开为lorem会展开为的任何内容。

关于如何使用这个函数来提取当前缓冲区中所有可用的片段，还有一个例子：

```vim
function! GetAllSnippets()
  call UltiSnips#SnippetsInCurrentScope(1)
  let list = []
  for [key, info] in items(g:current_ulti_dict_info)
    let parts = split(info.location, ':')
    call add(list, {
      \"key": key,
      \"path": parts[0],
      \"linenr": parts[1],
      \"description": info.description,
      \})
  endfor
  return list
endfunction
```
## 3.5 关于缺少python支持的警告


当加载UltiSnips时，它将检查正在运行的Vim是否使用python支持j进行编译。如果没有检测到支持，将显示一个警告并跳过加载UltiSnips。

如果要禁止此警告消息，您可以将以下行添加到您的vimrc文件中。

&emsp;&emsp;`let g：UltiSnipsNoPythonWarning = 1`

如果您的Vim配置文件在多个系统中共享，其中一些系统的vim可能没有使用python支持进行编译，这可能会有用。

# 4 编写片段
## 4.1 基础
### 4.1.1 片段如何被加载
片段定义文件存储在片段目录中。在哪里搜索这些目录的主要控制变量是列表变量，默认设置为：

&emsp;&emsp;`let g:UltiSnipsSnippetDirectories=["UltiSnips"]`

注意“snippets”是为snipMate片段定义文件保留的，不能在这个列表中使用。

UltiSnips将按照`g:UltiSnipsSnippetDirectories`定义的目录的顺序，在每个“runtimepath”目录中进行搜索。例如，如果您将片段定义文件保存在`~/.vim/mycoolsnippets`中，还想使用其他插件附带的UltiSnips片段，请将以下代码添加到您的vimrc文件中。

&emsp;&emsp;`let g:UltiSnipsSnippetDirectories=["UltiSnips", "mycoolsnippets"]`

如果您不想使用插件附带的第三方片段，请相应地定义变量:

&emsp;&emsp;` let g:UltiSnipsSnippetDirectories=["mycoolsnippets"]`

如果这个列表变量只有一个是绝对路径的条目，UltiSnips将不会遍历`&runtimepath`，而是只在这个目录中查找片段定义文件。这可能导致显著的加速效果。这意味着您将错过第三方插件附带的片段定义文件。您需要手动将它们复制到这个目录中。

一个示例：

&emsp;&emsp;`let g:UltiSnipsSnippetDirectories=[$HOME.'/.vim/UltiSnips']`

还可以通过设置变量`b: ultisnipssnippetdirectory`，在缓冲区的基础上逐个重新定义搜索路径。这个变量优先于全局变量。

使用类似于Vim检测==ftplugins==的策略，UltiSnips遍历片段定义目录，查找具有以下名称的文件：`ft.snippets，ft_*或ft/*`，其中“ft”是“ filetype“和`“*”`是一个类似shell的通配符，匹配任何字符串包含空字符串。下表显示了一些典型的snippets文件名及其相关的文件类型。


| 片段定义文件名 |      文件类型 |片段定义文件名 |      文件类型 |
|:--:|-:|-:|-:|
| ruby.snippets    |        ruby| perl.snippets        |    perl|
| c.snippets          |     c | c_my.snippets       |     c
|   c/a             |         c | c/b.snippets            | c|
|all.snippets     |        all  | all/a.snippets           all|

“all”文件类型是唯一的，它表示可以在编辑任何文档时都可以使用的片段，无论文件类型是什么。例如，于在all.snippets文件中的一个日期插入片段就可以很好地适用。

UltiSnips理解Vim的点文件类型语法。例如，如果您为CUDA C框架定义了一个点文件类型，例如。”:set ft=cuda.cpp“。则ultisnips将搜索并激活cuda和cpp 的片段定义文件。

### 4.1.2 基本语法

片段定义文件的语法很简单。所有**以#字符开头的行都被认为是注释**。UltiSnips会忽略注释。使用它们来说明片段。

**以关键字'extends'开头的一行提供了组合片段定义文件的方法**。当“extends”指令包含在片段定义文件中时，它指示UltiSnips引入指定文件类型的所有片段定义文件。

语法像这样的：

&emsp;&emsp;`extends ft1, ft2, ft3`

例如，cpp.snippets 中的第一行代是这样的:

&emsp;&emsp;`extends c`

当UltiSnips为cpp文件激活片段定义文件时，它首先查找c的所有片段定义文件并激活它们。这是一种从更一般的片段定义文件创建专门的片段定义文件的便捷方法。片段定文件中允许多个“extends”行，它们可以出现在文件中的任何位置。


**以关键字“priority”开头的行设置当前文件中定义的所有片段的优先级**。文件的默认优先级总是0。当展开一个片段时，UltiSnips将从匹配触发器的所有源中收集所有片段定义，并只保留具有最高优先级的片段定义。例如，所有发布了的片段段都具有< 0的优先级，因此用户定义的片段段总是覆盖已发布的片段。


**以关键字“snippet”开头的行表示片段定义的开始，以关键字“endsnippet”开头的行表示片段定义的结束**。片段定义位于这两行之间。下面是Unix shell (sh)文件类型的“if”语句的片段定义。

```vim
    snippet if "if ... then (if)"
    if ${2:[[ ${1:condition} ]]}; then
            ${0:#statements}
    fi
    endsnippet
```
起始行采用以下形式:

&emsp;&emsp;`snippet trigger_word [ "description" [ options ] ]`

trigger_word是必需的，但是描述和选项是可选的。

“trigger_word”是用于触发片段的单词或字符串序列，即触发器。通常使用一个单词，但trigger_word也可以包含空格。如果希望包含空格，则必须将触发器括在引号对中。：

 &emsp;&emsp;`snippet "tab trigger" [ "description" [ options ] ]`
 
引号不是触发器的一部分。为了激活片段，键入tab_trigger，按下tab键就会展开片段。

从技术上讲，不是只能必要为含有空格的触发器加上引号，任何成对匹配的字符都可以。例如，这是一个有效的片段起始行：

 &emsp;&emsp;`snippet !tab trigger! [ "description" [ options ] ]`

通过将触发器封装到另一种成对字符中，可以将引号作为触发器的一部分。

 &emsp;&emsp;`snippet !"tab trigger"! [ "description" [ options ] ]`
 
 为了激活片段。，入“tab trigger”，按下tab键就会展开片段。
 
 “description”是描述触发器的字符串。它有助于说明片段，并将其与具有相同触发器的其他片段区分开来。当一个片段被激活并且多个片段的触发器匹配时，UltiSnips将显示所有匹配的片段及其描述。然后用户选择他们想要的片段。
 
 
 ### 4.1.3 片段选项
 
 “options”控制片段的行为。 “options”由单个字符表示。片段的“options”字符可以是多个options单字符组合 的不含空格的词。
 
 目前支持的options有:

* 缺省 ——只要触发器（**也就是触发片段展开的关键词**）位于行首或紧跟空白字符（空格、制表符、换行、回车、换页）后，触发器在一行的任意位置都可以展开片段。

* b  行首——只有触发器是一行的第一个非空白字符才可以展开片段。

* i  词中展开——除了默认的展开方式以外，触发器紧跟非空白可打印字符后也可以展开片段。

* w 词边界——除了默认的展开方式以外，如果触发器起始和结束部分都是词边界（词边界是词字符与非词字符的分界）也可以展开片段。词字符由`iskeyword=@,48-57,_,192-255` （其中@表示大小写字符）设置，其实就是大小写字母、下划线、数字。例如，如果触发器紧跟标点符号后则可以展开片段。

* r 正则表达式——使用此选项，则触发器是一个python正则表达式。 如果最近键入的字符与正则表达式匹配，则该片段将被展开。 注意：正则表达式必须位于双引号（或其他成对符号）内。匹配结果被传递给片段定义中的python代码作为本地变量“match”。

* t  不展开制表符——默认情况下如果片段包含制表符，UltiSnips会按照Vim 'shiftwidth'，'softtabstop'，'expandtab'和'tabstop'设置的制表符展开片段中的制表符。（例如，如果设置了'expandtab'，则制表符将被替换为空格。）如果设置了此选项，UltiSnips将忽略Vim设置并插入制表符。此选项对于与制表符分隔格式相关的片段非常有用。

* s ——在跳转到下一个制表符之前，删除行尾的空白字符。如果在一行的末尾有一个带有可选文本的定位符，这将非常有用。 

* m   ——删除片段的每行最后一个字符后的所有空白字符。当代码片中有空格组成的行时，删除空格用空来代替。                                               
* e   自定义上下文片段——使用此选项后，片段的展开不仅可以由以前的字符在行中进行控制，还可以通过给定的python表达式进行控制。 该选项可以与其他选项一起指定，如“b”。更多信息参见==UltiSnips自定义上下文片段==

* A   ——当条件匹配时，片段自动触发，不需按tab键   。更多详细信息参见==UltiSnips自动触发==。


 结束行是“endsnippet”关键字独立成行：
 
 &emsp;&emsp;`endsnippet`

当解析片段文件时，UltiSnips从“endsnippet”结束行中删除尾随的换行符。
 
 
 ### 4.1.4 字符转义
 
在片段定义中，字符“\`”、“{”、“$”和“\\”具有特殊的含义。如果您想按字面意思插入其中一个字符，请使用反斜杠'\\'转义它们。
 
 
 ## 4.2 纯文本片段
 
 为了解释纯文本片段，让我们从一个简单的示例开始。你可以自己尝试这些例子。用Vim编辑一个新文件。示例片段将被添加到“all.snippets”中，所以你将希望在Vim中打开它，以便在相同的Vim实例中进行编辑。您可以为此使用==UltiSnipsEdit==，但也可以直接运行
 
 &emsp;&emsp;` :tabedit ~/.vim/UltiSnips/all.snippets`
 
 把下面这片段添加到“all.snippets”中并保存。
 
 ```
snippet bye "My mail signature"
Good bye, Sir. Hope to talk to you soon.
- Arthur, King of Britain
endsnippet
 ```
 
 
 
 UltiSnips检测您将更改写入片段文件并自动激活更改。因此，在空缓冲区中，键入触发器'bye'，然后按TAB键。
 
 `bye<Tab>`输出如下：
 
 Good bye, Sir. Hope to talk to you soon.
- Arthur, King of Britain

单词“bye”将替换片段定义的文本。

>注： 纯文本片段是片段中显示的是什么，展开的就是什么，展开的内容不可以操作。相当于常量
 
## 4.3 视觉占位符

片段可以包含名为`${VISUAL}`的特殊占位符。 `${VISUAL}`变量将用触发片段展开前选择的文本进行展开。

为了明白`${VISUAL}`占位符如何工作，用这个占位符定义一个片段，在vim的visual模式下选择一些文本后，键入触发按键（参见==g:UltiSnipsExpandTrigger==）。选择的文本会被删除，然后进入插入模式。现在键入触发器，并按下触发键，当片段展开时，先前选择的文本将被输出以代替`${VISUAL}`占位符。

当片段在非visual模式下被触发时，`${VISUAL}`占位符可以包含默认文本。语法是:
 
 &emsp;&emsp;`${VISUAL:default text}`
 
 `${VISUAL}`占位符还可以定义转换（参见==转换==），语法如下：
  &emsp;&emsp;`${VISUAL:default/search/replace/option}`
 
 下面是一个简单的例子来解释视觉转换。片段接受选定的文本，将其中的每个“should”替换为“is”，并将结果包装在tags中。

```vim
snippet t
<tag>${VISUAL:inside text/should/is/g}</tag>
endsnippet
 ```
 从这一行文字开始:
 
 &emsp;&emsp;`this should be cool`
  
  将光标放在“should”上，按下键序：`viw`（visual mode -> select inner word）。然后按下TAB，键入t，再按TAB，结果如下：
 
&emsp;&emsp;`this <tag>is</tag> be cool`
  
 如果在非visual模式下展开此片段 (例如，在出入模式键入t，再按TAB），你将得到：
 

&emsp;&emsp; `<tag>inside text</tag>`
 
 
 
## 4.4 插值
### 4.4.1 shellcode
 
 片段可以引入shell代码。将shell命令放入片段中，当片段展开时，shell命令将被命令执行时的输出代替。语法很简单：将shell代码放在反引号\` 中。当展开一个片段时，UltiSnips通过先将其写入一个临时脚本然后执行，从而运行shell代码。shell代码将被其标准输出替代，任何可以作为脚本运行的东西都可以在shell代码中使用。包括一个shebang行，例如` #!/usr/bin/perl`，您的片段能够使用其他程序(例如perl)运行脚本。
 
 这里有一些例子。这个片段使用shell命令插入当前日期。

```vim
snippet today
Today is the `date +%d.%m.%y`.
endsnippet
```
键入`today<tab>`输出如下 ：

Today is the 15.07.09.
 
下面这个例子使用perl插入当前日期：
```vim
snippet today
Today is `#!/usr/bin/perl
@a = localtime(); print $a[3] . '.' . $a[4] . '.' . ($a[5]+1900);`.
endsnippet
```
键入`today<tab>`输出如下 ：
Today is 15.6.2009.

### 4.4.2 VimScript
您还可以在插值中使用Vim脚本(有时称为VimL)。语法类似于shellcode。将vim脚本的代码用反引号括起来，为了区分与shellcode区分，代码以“!v”开头。下面是一个计算当前行缩进的例子:

```vim
snippet indent
Indent is: `!v indent(".")`.
endsnippet
```
`(note the 4 spaces in front): indent<tab>` ，输出如下：
 (note the 4 spaces in front): Indent is: 4.


### 4.4.3 Python
 
Python插值是迄今为止最强大的。语法类似于Vimscripts，只是代码以'!p'开头。python脚本可以使用python shebang `#!/usr/bin/python`来运行，但是使用“!p”可以附带一些预定义的对象和变量，可以简化和缩短代码。例如，python代码中隐含一个“snip”对象实例。Python代码使用'!p'还有另一个不同之处。通常，当一个片段被展开时，代码的标准输出将替换该代码。在python代码中，“snip.rv”属性的值将替换代码，而标准输出将被忽略。
 
python代码中自动定义的变量是：

|变量|描述|
|:--|:--|
|fn|当前文件名|
|path|当前文件完整路径|
|t|占位符的值，t\[1]是\$\{1}的文本，等等|
|snip|UltiSnips.TextObjects.SnippetUtil 对象实例，具有简化缩进处理的方法，并拥有应该为该片段插入的字符串。|
|context|上下文条件的结果。参见==自定义上下文片段==|
|match|正则表达式触发的片段。返回匹配正则表达式的值。参见<http://docs.python.org/library/re.html#match-objects>|
 
snip对象提供下列方法：
 
|方法|描述|
|:--|:--|
|snip.mkline(line="", indent=None)|返回准备添加到结果中的行。如果缩进为None，那么mkline将添加适合当前“tabstop”和“expandtab”变量的空格或制表符。|
| snip.shift(amount=1)|将mkline使用的默认缩进级别右移由'shiftwidth'  X 'amount' 定义的空格数。|
|snip.unshift(amount=1)|将mkline使用的默认缩进级别左移由'shiftwidth' X 'amount' 定义的空格数。|
|snip.reset_indent()|将缩进级别重置为其初始值|
|snip.opt(var, default)|检查Vim变量“var”是否已设置。如果已设置，则返回变量的值;否则，它返回“default”|

snip对象提供下列属性：
|属性|描述|
|:--|:--|
| snip.rv|“rv”是返回值，该文本将替换片段定义中的python块。它初始化为空字符串。这将禁用“res”变量。
| snip.c|片段中当前位于python块位置的文本。一旦插值完成，它就被设置为空字符串。因此，您可以检查snip.c是否不等于 ""，以确保插值只执行一次。这将禁用“cur”变量。
| snip.v|与\${VISUAL}占位符相关的数据。有两个属性<br />snip.v.mode   ('v', 'V', '^V', 参见 ==visual-mode== ) 和 snip.v.text 选择的文本|
| snip.fn|当前文件名
| snip.basename|当前文件名，不含扩展名
| snip.ft|当前文件类型
| snip.p|上一次选定的占位符，将包含具有以下属性的占位符对象:'current_text' 选择时占位符中的文本;“start”—选择时起始占位符;“end”—选择时的结束占位符|


snip对象提供下列运算符：
|运算符|描述|
|:--|:--|
|snip>>amout|snip.shift(amount)||
|snip << amount|snip.unshift(amount)|
|snip += line|snip.rv += '\n' + snip.mkline(line)|
 
 在python块中定义的任何变量都可以在相同片段中的其他python块中使用。此外，python模块“vim”、“re”、“os”、“string”和“random”是在片段的范围内预先导入的。其他模块可以使用python 'import'命令导入。
 
 Python代码允许非常灵活的片段。例如，下面的片段镜像了同一行上的第一个站位符的值，但是右对齐，并且使用大写。
 ```vim
snippet wow
${1:Text}`!p snip.rv = (75-2*len(t[1]))*' '+t[1].upper()`
endsnippet
 ```
`wow<tab>Hello World `，输出：
Hello World   
 `                                                  ` Hello World 
 
 
下面的片段使用正则表达式选项，并使用python的match对象演示正则表达式分组。它显示了片段的展开可以依赖于用于定义片段的触发器，而触发器本身可以变化。
 
 ```
 snippet "be(gin)?( (\S+))?" "begin{} / end{}" br
\begin{${1:`!p
snip.rv = match.group(3) if match.group(2) is not None else "something"`}}
    ${2:${VISUAL}}
\end{$1}$0
endsnippet
``` 
 `be<tab>center<c-j>`，输出
\begin{center}

\end{center}
 
 
` be center<tab> `,输出
\begin{center}

\end{center}
 
第二种形式是第一种形式的变体；两者产生了相同的结果，但它说明了正则表达式分组是如何工作的。以这种方式使用正则表达式有一些缺点:
1. 如果您同时使用Tab键来展开片段和补全代码，那么如果您键入`“be form<Tab>”`，期望补全代码“be formatted”，那么您将得到上面的结果，而不是您想要的。
2. 片段难以理解

然而，最大的优势是，您可以创建考虑了“触发器”前面部分的文本的片段。通过这种方式，您可以使用它来创建后缀片段，这在一些ide中很流行。
 
 ```
 snippet "(\w+).par" "Parenthesis (postfix)" r
(`!p snip.rv = match.group(1)`$1)$0
endsnippet
```
`something.par<tab>`，输出：
(something)

```
snippet "([^\s].*)\.return" "Return (postfix)" r
return `!p snip.rv = match.group(1)`$0
endsnippet
```
`value.return<tab> `,输出：
return value


### 4.4.4 全局片段

全局片段提供了在多个片段中重用通用代码的方法。
目前，只支持python代码。全局片段的执行结果被放入片段文件中的每个python代码块的全局变量中。要创建全局片段，使用关键字“global”代替“snippet”，对于python代码，使用“!p“代表触发器。例如，下面的片段生成与上一个示例相同的输出。但是，使用这种语法，“upper_right” 片段可以被其他片段重用。


```vim
global !p
def upper_right(inp):
    return (75 - 2 * len(inp))*' ' + inp.upper()
endglobal

snippet wow
${1:Text}`!p snip.rv = upper_right(t[1])`
endsnippet
```
`wow<tab>Hello World `,输出：
Hello World  
`                               `HELLO WORLD 

Python全局函数可以存储在Python模块中，然后导入。
这使得全局函数很容易被所有片段文件访问。从Vim7.4开始，您可以将python文件放入 `~/.vim/pythonx`，并将它们直接导入到您的片段中。例如，使用`~/.vim/pythonx/my_snippets_helper.py`：

```vim
global !p
   from my_snippet_helpers import *
endglobal
```  

 ## 4.5 定位点和占位符
 
 片段被用于快速将重用的文本插入文档。文本通常具有固定的结构和可变的组件。定位点用于简化可变内容的修改。使用定位点，您可以轻松地将光标放在可变内容的位置，输入您想要的内容，然后跳转到下一个可变组件，输入该内容，然后继续，直到所有可变组件都修改完成为止。
 
定位点的语法是一个\$符号后跟一个数字，例如`"$1"`。定位点从1开始，按顺序排列。`"$0"`是一个特殊的定位点。不管定义了多少个定位点，它总是片段中的最后一个定位点。如果没有定义`'$0'`，那么将在片段段的末尾定义`'$0'`。

例如：

```vim
snippet letter
Dear $1,
$0
Yours sincerely,
$2
endsnippet
```

`letter<tab>Ben<c-j>Paul<c-j>Thanks for suggesting UltiSnips!`，输出

Dear Ben,
Thanks for suggesting UltiSnips!
Yours sincerely,
Paul

你可以使用`<C-j>`跳转到下一个定位点，`<C-k>`跳转到上一个定位点。`<Tab>`键不用于向前跳转，因为它通常用于补全。有关为动作定义不同按键的帮助信息请参见==UltiSnips触发器==。

为定位点设置一些默认文本通常很有用。默认文本可以是可变组件常用的值，也可以是提醒您可变组件需要什么的一个单词或短语。默认文本语法是`'${number:value}'`。

下面的示例演示了shell 'case'语句的片段。定位点使用默认值来提醒用户需要什么值。

```vim
snippet case
case ${1:word} in
    ${2:pattern} ) $0;;
esac
endsnippet
```
`case<tab>$option<c-j>-v<c-j>verbose=true`， 输出：
case $option in
    -v ) verbose=true;;
esac


有时候，在一个定位点中再加上一个定位点是很有用的。要做到这一点，只需将嵌套的定位点包含在默认文本中即可。考虑下面演示HTML锚代码片段的示例。
```vim
snippet a
<a href="${1:http://www.${2:example.com}}"</a>
    $0
</a>
endsnippet
```
当这个片段展开时，第一个定位点有默认值`'http://www.example.com'`。如果您想要`“http://”`模式，请跳转到下一个定位点。它的默认值是“example.com”，可以键入您想要的任何域名来替换。

`<tab><c-j>google.com<c-j>Google `，输出
`<a href="http://www.google.com">`
    `Google`
`</a>`

如果在第一个定位点中您想要一个不同的url模式，或者想要用一个命名的锚替换默认的url，例如，'#name'，只需输入您想要的值。

`a<tab>#top<c-j>Top`，输出 
`<a href="#top">`
    `Top`
`</a>`

在上一个示例中，在第一个定位点上键入任何文本都将用键入的文本替换默认值，包括第二个定位点。所以第二个定位点被删除了。当触发定位点跳转时，UltiSnips将移动到下一个剩余的定位点 '$0'。这个特性可以被有意地用作向用户提供可选定位点的便捷方法。下面是一个例子。
```vim
snippet a
<a href="$1"${2: class="${3:link}"}>
    $0
</a>
endsnippet
```

这里，`$1`表示第一个定位点，它假设您总是希望为'href'属性添加值。输入url后并按下`<c-j>`，片段将跳转到下一个定位点`$2`。这个定位点是可选的，默认值是' class="link"'。你可以按`<c-j>`来接受该定位点，然后片段将跳转到第三个定位点`$3`，你可以键入calss属性值。或在第二个定位点你可以按下退格键从而使用一个一个空字符串代替器默认值，实质是删除它。在这两种情况下，继续按`<c-j>`，片段将跳转到锚内的最后一个定位点。

`<tab>http://www.google.com<c-j><c-j>visited<c-j>Google`，输出 ：
`<a href="http://www.google.com" class="visited">`
   ` Google`
`</a>`

`a<tab>http://www.google.com<c-j><BS><c-j>Google `，输出 ：
`<a href="http://www.google.com">`
    `Google`
`</a>`

定位点的默认文本还可以包含镜像、转换或插值。

 ## 4.6 镜像
 镜像重复定位点的内容。在片段展开期间，当您输入一个定位点的值时，该定位点的所有镜像都将被替换为相同的值。要镜像一个定位点，只需再次使用\$后面跟着数字的语法插入定位点，例如“\$1”。
 
 一个定位点可以在一个片段中镜像多次，一个片段中也可以有多个定位点被镜像。镜像的定位点可以定义一个默认值。只有t定位点的第一个实例需要有默认值。镜像的定位点将自动接受默认值。
 
```vim
snippet env
\begin{${1:enumerate}}
    $0
\end{$1}
endsnippet
```
`env<tab>itemize` ，输出
`\begin{itemize}`

`\end{itemize}`

```vim
snippet ifndef
#ifndef ${1:SOME_DEFINE}
#define $1
$0
#endif /* $1 */
endsnippet
```

`ifndef<tab>WIN32` ,输出
#ifndef WIN32
#define WIN32

#endif /* WIN32 */
 
 
 

 ## 4.7 转换
 
注意：转换有点难掌握，所以本章分为两部分。第一个描述转换及其语法，第二个用实例演示转换。
 
转换类似于镜像，但与逐字复制原始定位点中的文本，而是用一个正则表达式去匹配引用的定位点的内容，然后将转换应用到匹配的模式。UltiSnips中转换的语法和功能与TextMate转换非常接近。
 
 转换语法如下: 
 &emsp;&emsp;`${tab_stop_no/regular_expression/replacement/options}`
 
 组件的定义如下：
 
 * tab_stop_no：引用的定位点序号
 * regular_expression：正则表达式。所引用的定位点内容要进行匹配的正则表达式。
 * replacement：替代字符串，详细解释见4.7.1。
 * options：正总表示选项
	 * g：全局替换。默认情况下，只有第一个匹配结果被替换。使用此选项，所有的匹配结果都将被替换。
	 * i：忽略大小写。默认情况下，正则表达式匹配区分大小写。使用此选项，匹配将不考虑大小写。
	 * m：多行。默认情况下，'^'和'\$'特殊字符只应用于整个字符串的开始和结束；因此，如果您选择多行，就会对将其作为一个完整的单行字符串进行转换。使用这个选项，'^'和'$'特殊字符匹配字符串中任何一行的开始或结束(由换行字符'\n'分隔)。
	 * a：ascii转换。默认情况下，转换是在原始utf-8字符串上进行的。使用此选项，将对对应的ASCII字符串进行匹配，例如“a”将变为“a”。这个选项需要python包“unidecode”。
 
 正则表达式的语法超出了本文档的范围。内部使用使用了python正则表达式，所以python 're'模块可以用作用户指南。见<http://docs.python.org/library/re.html>
 
 换字符串的语法是惟一的。下一段对它作了详细的描述。
 
 ### 4.7.1 替换字符串
 
 替换字符串可以包含\$no变量，例如\$1，它引用正则表达式中匹配的分组。\$0变量是特殊的，它生成整个匹配。替换字符串还可以包含特殊的转义序列
 
 * \u——大写下一个字母
 * \l—— 小写下一个字母
   \U——大写所有字母直到遇到 \E
   \L——小写所有字母直到遇到 \E
   \E——结束 \L or \U 开始的大写或小写
   \n ——新行
   \t ——水平制表符
   
最后，替换字符串可以包含使用条件替换语法 (?no:text:other text)。其含义如下：如果组\$no已匹配，则插入“text”，否则插入“other text”。"other text"是可选的，如果没有，则默认为空字符串，""。这个功能非常强大。它允许您将可选文本添加到片段中。

 
 ### 4.7.2 样例
 
 转换是非常强大的，但通常语法是复杂的。希望下面的演示能够帮助说明转换功能。
 
 示例：大写一个字符
 
 ```vim
 snippet title "Title transformation"
 ${1:a text}
 ${1/\w+\s*/\u$0/}
 endsnippet
 ```
 

`title<tab>big small`，输出 
big small
Big small


示例: 大写一个字符并全局替换
```vim
snippet title "Titlelize in the Transformation"
${1:a text}
${1/\w+\s*/\u$0/g}
endsnippet
```
`title<tab>this is a title`，输出
this is a title
This Is A Title


示例: ASCII转换
```vim
snippet ascii "Replace non ascii chars"
${1: an accentued text}
${1/.*/$0/a}
endsnippet
```
`ascii<tab>à la pêche aux moules`，输出
à la pêche aux moules
a la peche aux moules


示例: 正则表达式分组
     这是一个聪明的类似c的printf片段，只有当有一种格式字符(%)在第一个定位点中，第二个定位点才显示。

```vim
snippet printf
printf("${1:%s}\n"${1/([^%]|%%)*(%.)?.*/(?2:, :\);)/}$2${1/([^%]|%%)*(%.)?.*/(?2:\);)/}
endsnippet
```
`printf<tab>Hello<c-j> // End of line `，输出
printf("Hello\n"); // End of line

But
`printf<tab>A is: %s<c-j>A<c-j> // End of line `，输出
printf("A is: %s\n", A); // End of line


在vim-snippets 仓库中，有更多的例子可以说明转换可以做什么。
 
 
 ## 4.8 清除片段
 
 要删除当前文件类型的片段，请使用“clearsnippets”指令。
 
 
 ```vim
clearsnippets
 ```
 “clearsnippets”删除所有优先级低于当前片段的片段。例如，下面的片段将清除优先级<= 1的所有片段，即使示例片段是在“clearsnippets”之后定义的。
 ```vim
priority 1
clearsnippets

priority -1
snippet example "Cleared example"
	This will never be expanded.
endsnippet
```
要清除一个或多个特定片段，请将片段的触发器作为参数提供给“clearsnippets”命令。下面的示例将清除片段trigger1和trigger2。
```vim
clearsnippets trigger1 trigger2
```
 
## 4.9 自定义上下文片段
 可以在片段定义使用中的'e'选项来启用自定义上下文片段。

在这种情况下，片段应该使用以下语法定义:
 
 &emsp;&emsp;`nippet trigger_word "description" "expression" options`
 
上下文可以使用一个特殊的头定义:

 &emsp;&emsp;`context "python_expression"`
 &emsp;&emsp;` snippet trigger_word "description" options`

 “expression”可以是任何python表达式。如果'expression'的值为' true '，则此代码段将有资格展开。 expression必须用双引号括起来。
 
在'expression'被求值之前，以下python模块会自动导入范围:'re'、'os'、'vim'、'string'、'random'。
 
 全局变量“snip”将具有以下属性:
 
* snip.window——“vim.current.window”的别名
* snip.buffer——“vim.current.window.buffer“的别名
* snip.cursor——光标对象，其行为类似于'vim.current.window.cursor”，但零索引并具有以下附加方法:
	* preserve()——执行展开前、展开后、跳转动作的特殊方法
	* set(line, column)——将光标设置为指定的行和列
	* to_vim_cursor()——返回索引为1的光标，适合赋值给给“vim.current.window.cursor”;
* snip.line和snip.column——光标位置的别名(零索引);
* snip.visual_mode——('v', 'V', '^V', 参见==visual-mode==);
* snip.visual_text——上一次visual模式选择的文本
* snip.last_placeholder——前一个片段中的最后一个活动占位符，具有以下属性
	* 'current_text' ——选择时占位符中的文本;
	*  'start' ——选择时起始占位符;
	*   'end' ——选择时的结束占位符;

```vim
snippet r "return" "re.match('^\s+if err ', snip.buffer[snip.line-1])" be
return err
endsnippet
```
只有当前一行开始于“if err”前缀时，该代码段才会扩展为“return err”。

注意:自定义上下文片段优先于其他片段。如果没有上下文匹配，可以使用其他片段作为后备:

```vim
snippet i "if ..." b
if $1 {
    $2
}
endsnippet

snippet i "if err != nil" "re.match('^\s+[^=]*err\s*:?=', snip.buffer[snip.line-1])" be
if err != nil {
    $1
}
endsnippet

```
 
 如果前一行匹配“err:=”前缀，则该代码段将展开为“if err != nil”，否则将展开默认的“if”片段。
 
将上下文条件移动到单独的模块是一个好主意，这样其他UltiSnips用户就可以使用它。在这种情况下，应该使用'global'关键字导入模块，如下所示:

 ```vim
global !p
import my_utils
endglobal

snippet , "return ..., nil/err" "my_utils.is_return_argument(snip)" ie
, `!p if my_utils.is_in_err_condition():
    snip.rv = "err"
else:
    snip.rv = "nil"`
endsnippet
```
只有当光标位于return语句中时，该片段才会展开，然后根据它所在的if语句，它将展开为“err”或“nil”。'is return argument'和'is in err condition'是在本例中称为'my utils'的自定义python模块的一部分。

上下文条件可以返回能在python'if'语句中用作条件的任何值，如果它被认为是'True'，则片段将被展开。“条件”的评估值在片段变量“snip.context”中可用。

```vim
snippet + "var +=" "re.match('\s*(.*?)\s*:?=', snip.buffer[snip.line-1])" ie
`!p snip.rv = snip.context.group(1)` += $1
endsnippet
```

该片段将在行之后展开为'var1 +='，它从'var1:='开始。
 
您可以使用以下技巧从前面的片段中捕获占位符文本:
 
 ```vim
 snippet = "desc" "snip.last_placeholder" Ae
`!p snip.rv = snip.context.current_text` == nil
endsnippet
 ```
 
只有在替换其他片段中选定的定位点(例如，在“if \${1:var}”中)，并将后面的定位点值替换为“== nil”时，才会展开该代码段。 

 
 
## 4.10 片段动作

片段动作是任意的python代码，可以在片段展开的生命周期中的特定点执行。

动作类型有三种：
* 展开前——在触发器条件匹配之后调用，但在片段实际展开之前调用;
* 展开后——在片段展开和插入之后调用
* 跳转——在跳转到前一个或下一个占位符后调用

指定的代码将在上面定义的阶段进行计算，在==ultisnips自定义上下文片段==小节中陈述的相同的全局变量和模块都可用。

注意:所有缓冲区修改都应使用名为“snip.buffer”的特殊变量。 不是'vim.current.buffer'，也不是'vim.command（“...”）'，因为在这种情况下，UltiSnips将无法正取跟踪缓冲区中的变化。

“snip.buffer“具有与”vim.current.window.buffer“相同的接口。

### 4.10.1 展开前动作
展开前动作可用于在一个位置匹配片段，然后在不同的位置展开片段。一些有用的例子是：为片段纠正缩进；在初始函数之外移动展开点的另一个函数体中展开函数声明片段；通过在不同位置展开片段来执行extract方法重构。



展开前动作声明如下：
&emsp;&emsp;`pre_expand "python code here"`
&emsp;&emsp;`snippet ...`
&emsp;&emsp;`endsnippet`

缓冲区可以通过名为snip.buffer的变量在展开前动作代码中修改。片段展开位置将自动调整。

修改光标行(与触发器匹配的地方)可以通过传入期望的光标位置来调用特殊变量的方法'snip.cursor.set(line, column)' 。在这种情况下，UltiSnips不会删除任何匹配的触发器文本，它应该在动作代码中手动执行。

此外，上面定义的范围变量'snip.visual content'也将被声明，并包含片段展开之前选择的文本(类似于\$VISUAL占位符)。

无论在哪里触发的，下面的片段将在4个空格缩进级别展开，。

```vim
pre_expand "snip.buffer[snip.line] = ' '*4; snip.cursor.set(snip.line, 4)"
snippet d
def $1():
    $0
endsnippet
```
下面的片段将把选择的代码移动到文件的末尾，并为它创建一个新的方法定义:

```vim
pre_expand "del snip.buffer[snip.line]; snip.buffer.append(''); snip.cursor.set(len(snip.buffer)-1, 0)"
snippet x
def $1():
	${2:${VISUAL}}
endsnippet
```

### 4.10.2 展开后动作
展开后动作可用于执行一些基于展开的片段文本的操作。一些场景:代码风格格式化(例如，在方法声明之前和之后插入新行)，根据python插值结果应用动作。

展开后动作声明如下：
&emsp;&emsp;`post_expand "python code here"`
&emsp;&emsp;`snippet ...`
&emsp;&emsp;`endsnippet`

缓冲区可以通过名为snip.buffer的变量在展开后的动作代码中修改。片段展开位置将自动调整。

变量'snip.snippet_start' 和 'snip.snippet_end'将在action code作用域中定义，并相应地以“(line，colum)”的形式指向展开的片段的开始和结束的位置。

注意:如果展开后动作在展开前插入或删除行，'snip.snippet_start' 和 'snip.snippet_end'将自动调整到正确的位置。


下面的片段将展开为方法定义，并在片段结束后自动插入额外的换行。这对创建一个会在特定上下文中插入多个换行符的函数很有用。

```vim
post_expand "snip.buffer[snip.snippet_end[0]+1:snip.snippet_end[0]+1] = ['']"
snippet d "Description" b
def $1():
	$2
endsnippet
```

### 4.10.3 跳转后动作
跳转后动作可以根据用户在占位符中输入的内容触发执行一些代码。值得注意的场景：在跳转后展开另一个片段或在上次跳转后展开匿名片段(例如执行move方法重构，然后插入新方法调用);在最后一次跳转后将标题插入TOC。

跳转后动作声明如下：
&emsp;&emsp;`post_jump "python code here"`
&emsp;&emsp;`snippet ...`
&emsp;&emsp;`endsnippet`

缓冲区可以通过名为snip.buffer的变量在跳转后的动作代码中修改。片段展开位置将自动调整。

下面变量和方法也将在代码作用域中定义:

* snip.tabstop——跳转的定位点编号
* snip.jump_direction——跳转方向，1向前，-1向后。
* snip.tabstops——定位点对象列表
* snip.snippet_start——(line, column)展开片段段的起始部分
* snip.snippet_end——(line, column)展开片段段的结束部分
* snip.expand_anon()——"UltiSnips_Manager.expand_anon()"的别名

Tabstop对象有几个有用的属性:

* start——(line, column)，定位点的起始位置(也可以作为 'tabstop.line' 和 'tabstop.col'来访问)。
* end——(line, column)，定位点的结束位置
* current_text——定位点的文本内容

下面的片段将在vim-help文件的目录中插入章节:
```vim
post_jump "if snip.tabstop == 0: insert_toc_item(snip.tabstops[1], snip.buffer)"
snippet s "section" b
`!p insert_delimiter_0(snip, t)`$1`!p insert_section_title(snip, t)`
`!p insert_delimiter_1(snip, t)`
$0
endsnippet
```
'insert_toc_item'将在第一次跳转后调用，并将新的进入章节添加到当前文件的TOC中。

注意：还可以从跳转动作中触发片段展开。在这种情况下，应该调用'snip.cursor.preserve()'方法，以便ultisnips知道游光标已经位于所需位置。

下面的示例将在用户跳出方法声明片段后，在文件末尾插入方法调用。

```vim
global !p
def insert_method_call(name):
	vim.command('normal G')
	snip.expand_anon(name + '($1)\n')
endglobal

post_jump "if snip.tabstop == 0: insert_method_call(snip.tabstops[1].current_text)"
snippet d "method declaration" b
def $1():
	$2
endsnippet
```


## 4.11 自动触发
注意:vim必须比7.4.214更新才能支持该特性。

许多语言构造只能在特定的位置出现，因此不需要手动触发片段就可以使用它们。

片段可以通过在定义中指定“A”选项来标记为自动触发。

片段定义为自动触发后，片段触发条件将对每个键入的字符进行检查，如果触发条件匹配，则将触发片段展开。

**警告**：使用该特性可能会导致vim显著变慢。如果您发现了，请报告问题。

考虑以下有用的Go片段:：

```vim
snippet "^p" "package" rbA
package ${1:main}
endsnippet

snippet "^m" "func main" rbA
func main() {
	$1
}
endsnippet
```

当“p”字符出现在行首时，它将漂亮地扩展为“package main”。“m”也是一样。“m”后无需按触发键.

# 5 UltiSnips和其他插件
## 5.1 现有的集成

UltiSnips内置了对一些常见插件的支持，还有一些插件支持UltiSnips并使用它来改善用户体验。这是一个不完整的列表——如果你想在这里列出你的插件，只要发送pull请求。

snipMate——UltiSnips是snipMate的一个非正式的替代品。它有更多的特性，所以移植片段仍然是一个好主意，但是切换的阻力很小。UltiSnips正在努力真正地模拟snipMate，例如，在snipMate片段中不支持递归定位点(当然UltiSnips片段也不支持)。
YouCompleteMe——提供了对UltiSnips的开箱即用完整支持。它为片段提供了一个非常好的补全对话框。

neocomplete——UltiSnips附带了neocomplete的源代码，因此也提供了开箱即用的补全对话支持

deoplete——neocomplete的继任者也得到了支持

Supertab——UltiSnips内置了对Supertab的支持。只要这两个插件版本足够新，`<tab>`要么展开一个代码段，要么遵从Supertab进行扩展。

unite——UltiSnips是unite的一个来源。作为如何使用它的一个例子，添加以下函数和映射到您的vimrc:
```vim
function! UltiSnipsCallUnite()
    Unite -start-insert -winheight=100 -immediately -no-empty ultisnips
    return ''
  endfunction

  inoremap <silent> <F12> <C-R>=(pumvisible()? "\<LT>C-E>":"")<CR><C-R>=UltiSnipsCallUnite()<CR>
  nnoremap <silent> <F12> a<C-R>=(pumvisible()? "\<LT>C-E>":"")<CR><C-R>=UltiSnipsCallUnite()<CR>
  ```
在insert或normal模式下键入`<F12>`时，您将得到带有匹配片段的unite接口。按回车键将展开相应的片段。如果只有一个片段匹配，那么当您按下`<F12>`键时，光标前面的文本将被展开。

## 5.2 扩展UltiSnips   
UltiSnips允许其他插件动态添加新的片段。由于UltiSnipsis是用python编写的，所以集成也是基于python的。可以在' test/test_UltiSnipsFunc.py‘中找到一个小的例子，搜索AddNewSnippetSource。如果您将ultisnips与您的插件集成，请在github上与我们联系，以便将其列在文档中。

# 6 FAQ
Q：我必须调用UltiSnips#ExpandSnippet()来检查片段是否可展开吗?有没有一个替代neosnippet#expandable类似物
A:  有的，尝试

```vim
function UltiSnips#IsExpandable()
    return !empty(UltiSnips#SnippetsInCurrentScope())
  endfunction
  
```
考虑一下如果您在空格字符后面调用UltiSnips#SnippetsInCurrentScope()的话，它将返回所有的片段。如果您想让UltiSnips#IsExpandable()在空格字符后面调用时返回false，请使用这个稍微复杂一点的实现：
```vim
function UltiSnips#IsExpandable()
    return !(
      \ col('.') <= 1
      \ || !empty(matchstr(getline('.'), '\%' . (col('.') - 1) . 'c\s'))
      \ || empty(UltiSnips#SnippetsInCurrentScope())
      \ )
  endfunction
```

# 7 及时帮助

UltiSnips需要Vim社区的帮助才能不断改进。请考虑通过提供新特性或错误报告来加入这项工作。

* 在GitHub上克隆仓库(`git Clone git@github.com:SirVer/ultisnips.git`)，进行更改并在GitHub上发送pull request。
* 制作一个补丁，报告一个bug/特性请求(见下面)并将补丁附加到它上面。

您可以通过在我们的问题跟踪器中修复或报告bug来做出贡献:<https://github.com/sirver/ultisnips/issues>
# 8 贡献者

UltiSnips 由HolgerRapp (@SirVer, SirVer@gmx.de)于2009年6月至2015年12月发起并维护。截止2018年4月，该网站由StanislavSeletskiy (@seletskiy)维护。现在，它由越来越多的贡献者维护。

这是按时间顺序排列的git贡献者列表。完整的贡献者列表是git日志中的作者和这个下面列表的并集。

  JCEB - Jan Christoph Ebersbach
   Michael Henry
   Chris Chambers
   Ryan Wooden
   rupa - Rupa Deadwyler
   Timo Schmiade
   blueyed - Daniel Hahler
   expelledboy - Anthony Jackson
   allait - Alexey Bezhan
   peacech - Charles Gunawan
   guns - Sung Pae
   shlomif - Shlomi Fish
   pberndt - Phillip Berndt
   thanatermesis-elive - Thanatermesis
   rico-ambiescent - Rico Sta. Cruz
   Cody Frazer
   suy - Alejandro Exojo
   grota - Giuseppe Rota
   iiijjjii - Jim Karsten
   fgalassi - Federico Galassi
   lucapette
   Psycojoker - Laurent Peuch
   aschrab - Aaron Schrab
   stardiviner - NagatoPain
   skeept - Jorge Rodrigues
   buztard
   stephenmckinney - Steve McKinney
   Pedro Algarvio - s0undt3ch
   Eric Van Dewoestine - ervandew
   Matt Patterson - fidothe
   Mike Morearty - mmorearty
   Stanislav Golovanov - JazzCore
   David Briscoe - DavidBriscoe
   Keith Welch - paralogiki
   Zhao Cai - zhaocai
   John Szakmeister - jszakmeister
   Jonas Diemer - diemer
   Romain Giot - rgiot
   Sergey Alexandrov - taketwo
   Brian Mock - saikobee
   Gernot Höflechner - LFDM
   Marcelo D Montu - mMontu
   Karl Yngve Lervåg - lervag
   Pedro Ferrari - petobens
   Ches Martin - ches
   Christian - Oberon00
   Andrew Ruder - aeruder
   Mathias Fußenegger - mfussenegger
   Kevin Ballard - kballard
   Ahbong Chang - cwahbong
   Glenn Griffin - ggriffiniii
   Michael - Pyrohh
   Stanislav Seletskiy - seletskiy
   Pawel Palucki - ppalucki
   Dettorer - dettorer
   Zhao Jiarong - kawing-chiu
   Ye Ding - dyng
   Greg Hurrell - wincent






------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>[《UltiSnips官方文档-2019-7-29》](https://github.com/SirVer/ultisnips/blob/master/doc/UltiSnips.txt)。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

