---
title:  Linux配置教程
tags: Linux使用教程
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：<font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

<style>table{word-break:initial;}</style>


# 1 设置超级用户密码
```bash
$ sudo passwd [root]
```
# 2 添加/删除用户。
```bash
#添加一个普通用户,并创建用户主目录。指定shell脚本解析器是/bin/bash。
$sudo useradd –m 用户名 –s /bin/bash 

#删除用户
$sudo userdel –r 用户名
```
# 3 设置密码
```bash
#设置或修改用户名的密码。
$passwd 用户名
```

# 4 安装vmwaretools
* 点击VMware菜单栏的虚拟机
* 点击安装vmwaretools
* 复制vmware-tools到home目录下新建的temp
```bash
$cd temp 
$tar xzvf  vmware-tools.tar.gz
$cd vmware-tools-distrib
$sudo ./vmware-install.pl
```
* 一路回车，遇到no就输入yes。
* 出现enjoy则安装成功
* 重启ubuntu
```shell
$ sodu reboot
```

# 5 更改软件源
* 自动修改: 桌面-->设置->software&update->选择一个源 
* 手动修改
```
#备份
$sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak 
$sudo gedit /etc/apt/sources.list

# 加入如下三个源的一个

#中科大的：
deb http://mirrors.ustc.edu.cn/ubuntu/ precise-updates main restricted
deb-src http://mirrors.ustc.edu.cn/ubuntu/ precise-updates main restricted
deb http://mirrors.ustc.edu.cn/ubuntu/ precise universe
deb-src http://mirrors.ustc.edu.cn/ubuntu/ precise universe
deb http://mirrors.ustc.edu.cn/ubuntu/ precise-updates universe
deb-src http://mirrors.ustc.edu.cn/ubuntu/ precise-updates universe
deb http://mirrors.ustc.edu.cn/ubuntu/ precise multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ precise multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ precise-updates multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ precise-updates multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ precise-backports main restricted universe multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ precise-backports main restricted universe multiverse
deb http://security.ubuntu.com/ubuntu precise-security main restricted
deb-src http://security.ubuntu.com/ubuntu precise-security main restricted
deb http://security.ubuntu.com/ubuntu precise-security universe
deb-src http://security.ubuntu.com/ubuntu precise-security universe
deb http://security.ubuntu.com/ubuntu precise-security multiverse
deb-src http://security.ubuntu.com/ubuntu precise-security multiverse

#搜狐源
deb http://mirrors.sohu.com/ubuntu/ precise-updates main restricted
deb-src http://mirrors.sohu.com/ubuntu/ precise-updates main restricted
deb http://mirrors.sohu.com/ubuntu/ precise universe
deb-src http://mirrors.sohu.com/ubuntu/ precise universe
deb http://mirrors.sohu.com/ubuntu/ precise-updates universe
deb-src http://mirrors.sohu.com/ubuntu/ precise-updates universe
deb http://mirrors.sohu.com/ubuntu/ precise multiverse
deb-src http://mirrors.sohu.com/ubuntu/ precise multiverse
deb http://mirrors.sohu.com/ubuntu/ precise-updates multiverse
deb-src http://mirrors.sohu.com/ubuntu/ precise-updates multiverse
deb http://mirrors.sohu.com/ubuntu/ precise-backports main restricted universe multiverse
deb-src http://mirrors.sohu.com/ubuntu/ precise-backports main restricted universe multiverse

#网易源：
deb http://mirrors.163.com/ubuntu/ precise-updates main restricted
deb-src http://mirrors.163.com/ubuntu/ precise-updates main restricted
deb http://mirrors.163.com/ubuntu/ precise universe
deb-src http://mirrors.163.com/ubuntu/ precise universe
deb http://mirrors.163.com/ubuntu/ precise-updates universe
deb-src http://mirrors.163.com/ubuntu/ precise-updates universe
deb http://mirrors.163.com/ubuntu/ precise multiverse
deb-src http://mirrors.163.com/ubuntu/ precise multiverse
deb http://mirrors.163.com/ubuntu/ precise-updates multiverse
deb-src http://mirrors.163.com/ubuntu/ precise-updates multiverse
deb http://mirrors.163.com/ubuntu/ precise-backports main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ precise-backports main restricted universe multiverse
```


* 更新软件列表
```shell
$ sudo apt update  
```
* 检查软件版本并更新软件（不升级软件则不用做）
```shell
$ sudo apt upgrade
```

&emsp;&emsp;
**补充：ubuntu软件管理**

* linux在线管理命令apt：

| 命令|命令的功能|
|:--|:--|
|apt install 包名|	安装软件
|apt remove 包名|		移除软件，保留配置
|apt autoremove|自动卸载软件
|apt purge 包名	|	移除软件包及配置文件
|apt search 包名|	搜索应用程序
|apt show 包名|	显示软件包详细信息
|apt update	|刷新存储库索引
|apt upgrade|升级所有可升级的软件包
|apt full-upgrade|	升级软件和依赖包
|apt list –installed|	列出已安装的软件
|apt list –upgradable|		列出可升级的软件
|apt edit-sources|	编辑源文件
|apt list 包名|列出软件


* linux本地管理命令dpkg:

| 命令|命令的功能|
|:--|:--|
 |sudo dpkg -i |安装|
| sudo dpkg -P|  干净卸载|
 |sudo dpkg -r | 卸载|
| sudo dpkg -l|  列出安装的软件|

# 6 安装基本软件
## 6.1 安装ssh
```bash
$ ps -elf|grep ssh     查看ssh否安装，有sshD则安装成功
$ sudo apt install ssh #安装ssh

```

## 6.2 安装git
```bash
$ sudo apt install git                                //安装git和配置用户
$ git config --global user.name "Your Name"          //设置用户名
$ git config --global user.email "email@example.com" //"设置用户邮件地址"
```

在用户主目录下，看看有没有.ssh目录，如果有，再看看这个目录下有没有id_rsa和id_rsa.pub这两个文件，如果已经有了，可直接跳到下一步。如果没有，s使用如下命令创建SSH 密钥： 
```
$ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```
  生成密钥用于创建远程仓库：登录github 账户——>设置 添加SSH key.pub的内容，新建仓库。
>**补充：git更多用法**——[git教程](http://blog.csdn.net/liao20081228/article/details/76861995)。
 
## 6.3 安装数据库mysql
```
$sudo apt install mysql-server mysql-client libmysqlclient-dev
```
## 6.4 安装vim
### 6.4.1 安装vim
* 安装vim7.4-python3版本
```
sudo apt install vim
```

* 安装vim7.4-python版本,
```
sudo apt install vim-nox-py2
```

* 切换版本
```
sudo update-alternatives --config vim
```

* 自己编译安装vim8.0
```
//安装依赖库:如果您不需要对Python 3、Lua、Ruby的支持的话，可以选择不安装相应的依赖或者编译Vim时不添加支持。

$ sudo apt install libncurses5-dev libgnome2-dev libgnomeui-dev libgtk2.0-dev libatk1.0-dev libbonoboui2-dev

$ sudo apt install libcairo2-dev libx11-dev libxpm-dev libxt-dev python-dev python3-dev ruby-dev lua5.1 lua5.1-dev git


//删除原有vim——首先查询系统中有哪些与vim相关的软件，我的是vim、vim-common和vim-run，然后彻底删除他们：

$dpkg -l | grep vim
$sudo dpkg -P vim vim-common vim-run


//下载vim源码

$ git clone https://github.com/vim/vim.git
$ cd vim

//配置编译选项:安装前先获取Python的配置路径，比如我的是/usr/lib/python2.7/config-x86_64-Linux-gnu。
$ ./configure --with-features=huge --enable-multibyte --enable-rubyinterp --enable-pythoninterp --with-python-config-dir=/usr/lib/python2.7/config-x86_64-linux-gnu --enable-python3interp --with-python3-config-dir=/usr/lib/python3.5/config-3.5m-x86_64-linux-gnu --enable-perlinterp --enable-luainterp --enable-gui=gtk2 --enable-cscope --prefix=/usr 

//编译
$ make VIMRUNTIMEDIR=/usr/share/vim/vim80

//安装
$ sudo make install

//可以安装checkinstall工具将从源码安装的软件变得像用deb包安装的一样，方便以后可以直接用sudo dpkg -P vim删除vim：
$ sudo apt-get install checkinstall
$ cd vim
$ sudo checkinstall

//最后，删除vim源码包，执行vim 
```
* 查看vim的版本号、补丁号以及是否成功开启了Python的支持（包含+python）。
```
$ vim --version | grep python
```
### 6.4.2 安装vim插件管理器
* 下载代码仓库
```
$cd ~
$git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
$vim ~/.vimrc
```

* 添加以下内容
```
set nocompatible 
filetype off 
set rtp+=~/.vim/bundle/Vundle.vim 
call vundle#begin() 
Plugin 'VundleVim/Vundle.vim'
call vundle#end() 
filetype plugin indent on 
```
* 保存后退出
* 安装插件
```
$vim 
ESC
:PluginInstall
```
* 显示done则成功

**补充1：快速安装vim插件**
&emsp;&emsp;可以先用git clone克隆源码到 ~/.vim/bundle 文件夹， 或者直接将别人下载好的仓库拷贝到~/.vim/bundle。
&emsp;&emsp;然后在~/.vimrc文件中加上下面的语句，执行PluginInstall即可快速安装。
```
set nocompatible              "不与vi兼容
filetype off       
set runtimep+=~/.vim/bundle/Vundle.vim 
call vundle#begin() 
Plugin 'VundleVim/Vundle.vim'
plugin '用户名/插件仓库名'          "GitHub上的插件
Plugin 'vim-scripts/插件仓库名'     "来自vim-scripts.org的插件
Plugin 'git clone 后面的地址'        "由Git支持但不再github上的插件仓库
Plugin 'file:///+本地插件仓库绝对路径'  "本地的Git仓库(例如自己的插件)
Plugin 'rstacruz/sparkup', {'rtp':'vim/'} "插件在仓库的子目录中.
call vundle#end() 
filetype plugin indent on           "打开文件类型检测，并加载相关插件，根据文件类型设置缩进距离
```
**补充2：卸载vundle插件**
&emsp;&emsp;先在.vimrc中删除相应插件，然后输入vim,再输入PluginClean
**补充3：vundle常用命令**
```
PluginList         -列举列表(也就是.vimrc)中配置的所有插件  
PluginInstall      -安装列表中的全部插件  
PluginInstall!     -更新列表中的全部插件  
PluginSearch foo   -查找foo插件  
PluginSearch! foo  -刷新foo插件缓存  
PluginClean        -清除列表中没有的插件  
PluginClean!       -清除列表中有的插件
```
### 6.4.3 安装vim自动补全插件

* 查看vim是否支持python,显示+则支持，否则安装支持python的vim-nox-py2
```
$vim --version | grep python  
$sudo apt install vim-nox-py2
```
* 安装编译环境
```
$ sudo apt install gcc g++
$ sudo apt install python-dev python3-dev      //安装python
$ sudo apt install cmake build-essential         //安装cmake
$ sudo apt install clang-4.0                       //安装clang
$ sudo apt install libclang-4.0-dev                //安装libcang
$ sudo apt install ctags                       //安装ctags
```
* 下载youcompleteme源码
```
$ git clone https://github.com/Valloric/YouCompleteMe.git ~/.vim/bundle/YouCompleteMe
$ cd ~/.vim/bundle/YouCompleteMe
$ git submodule update --init --recursive
```
* 也可以跳过上面clone youcompleteme源码步骤，直接拷贝他人装好YCM的的对应Youcompleteme文件夹 ，进行下面的编译YouCompleteMe操作
```
$ cd ~/.vim/bundle/YouCompleteMe 
$ ./install.py --clang-completer 
//如果自己系统自带的clang版本高于7.0，则可以使用自身clang库，节约时间
$./install.py --clang-completer --system-libclang
```
* 编译完成后
```
$cd ~
$vim ~/.vimrc
```
* 将原内容替换为
```
set nocompatible 
filetype off 
set rtp+=~/.vim/bundle/Vundle.vim 
call vundle#begin() 
Plugin 'VundleVim/Vundle.vim'
Plugin 'Valloric/YouCompleteMe'
call vundle#end() 
filetype plugin indent on 
```
* 保存后退出，注册YCM
```
$vim 
ESC
:PluginInstall
```
* 显示done则成功
* 添加C/C++支持
```
#打开YCM配置文件
$vim ~/.vim/bundle/YouCompleteMe/third_party/ycmd/cpp/ycm/.ycm_extra_conf.py

#在flags的括号中添加头文件路径
'-isystem',
'C/C++ 头文件绝对路径',
... ...


#如果不想因为c++11新特性发出警告就添加'-Wc++11-compat'，并删除'-Wc++98-compat'（如果有的话）

#注释调下面的语句（在每行前面加#）
try:
final_flags.remove( '-stdlib=libc++' )
except ValueError:
pass
```


### 6.4.4 Vim配置
&emsp;&emsp;直接将仓库里的vimrc文件拷贝为主目录下的.vimrc, 或者`gedit ~/.vimrc`替换以下内容到.vimrc中

```vim
""""""""""""""""""""""""""""""""""选项设置方式说明""""""""""""""""""""""""""
"选项有三种类型布尔型、数值型、字符串
":set                  显示所有不同于缺省值的选项
":set all              显示除终端设置以外的所有选项
":set termcap          显示所有的终端设置选项。
":set {option}?        显示option的值        
":set {option}         bool选项: 打开,数值选项:显示其值。字符串选项:显示其值    
":set no{option}       bool选项: 关闭
":set {option}!        bool选项: 反转其值。Vi无此功能
":set {option}&        设置选项为缺省值
":set {option}&vi      设置选项为 Vi 的缺省值,Vi 无此功能                
":set {option}&vim     设置选项为 Vim 的缺省值,Vi 无此功能
":set all&             设置除了终端设置以外的所有选项为其缺省值
":set {option}={value} 设置字符串或数值选项的值为 value
":set {option}:{value} 设置字符串或数值选项的值为 value
":set {option}+={value} 把value加到数值选项里，或者附加到字符串选项之后。如果选项是逗号分隔的列表，除非原来的值为空，会加上一个逗号。如果选项是标志位的列表，删除多余的标志位。如果加入已经存在的标志位，选项值不变
":set {option}^={value} 把value乘到数值选项里，或者附加到字符串选项之前。如果选项是逗号分隔的列表，除非原来的值为空，会加上一个逗号。
":set {option}-={value} 把value从数值选项里减去，或者从字符串选项里删除，如果该值原来存在的话，如果不存在，不会有错误或者警告。如果选项是逗号分隔的列表，除非新值为空，删除一个逗号。如果选项是标志位的列表，{value} 必须和选项里出现的顺序 完全相同。一个一个地分别删除标志位可以解决这个问题。



""""""""""""""""""""""""""""""""""添加vim插件""""""""""""""""""""""""""""""""""""
set nocompatible                               "设置为不兼容vi
filetype off                                   "关闭文件类型检测
set runtimepath+=~/.vim/bundle/Vundle.vim      "添加运行时的路径
call vundle#begin() 
Plugin 'VundleVim/Vundle.vim'                  "安装vundle插件管理器,安装PunginInstall，卸载BundleClean
Plugin 'sjl/gundo.vim'                         "撤销与重做
Plugin 'luochen1990/rainbow'                   "彩虹括号
Plugin 'jiangmiao/auto-pairs'                  "自动输入括号对,括号快速跳转
Plugin 'tomasr/molokai'                        "主题molokai，切换主题：colorscheme 主题名
Plugin 'Valloric/YouCompleteMe'                "安装youcompleteme自动补全
Plugin 'bling/vim-airline'                     "状态栏插件,代替powerline
Plugin 'vim-airline/vim-airline-themes'        "状态栏插件主题
Plugin 'SirVer/ultisnips'                      "安装ultisnips块输入引擎,代替c.vim
Plugin 'honza/vim-snippets'                    "安装ultisnips的代码块
Plugin 'scrooloose/nerdcommenter'              "快速注释反注释,相比Tcomment来说键盘映射简单,但不能用于插入模式,代替c.vim
call vundle#end()
filetype plugin indent on                      "打开文件文件类型检测，并载入插件，根据文件类型设置缩进



"""""""""""""""""""""""本配置文件使用的快捷键"""""""""""""""""""""""""
"F1                                  vim帮助 

"F11                                 快速跳过括号对
"F9                                  注释/反注释  
"F10                                 插入模式下添加注释符号
"\cm                                 注释多行只用一对符号注释，之前的注释都是每行独立注释 
"\cs                                 使用好看的注释方式


"tab                                 插入模式下ultisnip代码块插入
"ctrl +j                             插入模式下ultisnip代码块补全后，跳至下一个或光标位置
"ctrl +k                             插入模式下ultisnip代码块补全后，跳至上一个或光标位置


"ctrl+空格                           插入模式下YCM搜索更多补全候选项
"\gc                                 跳转到声明
"\gf                                 跳转到定义
"\gh                                 跳转到头文件
"\gr                                 跳转到引用
"\gi                                 跳转到实现
"\gt                                 跳转到类型定义
"\st                                 显示类型
"\sp                                 显示父类
"\sd                                 显示文档
"\d                                  显示详细的错误诊断信息

":help 插件名                        查看该插件的帮助



"""""""""""""""""""""""""""""theme"""""""""""""""""""""""""""""""""""
":colorscheme molokai 切换主题
":colorscheme 查看当前主题
":colorscheme 空格 tab键 列出所有主题


if has("gui_runing")              "如果是在图形用户界面
	set background=light          "设置背景色为亮色
else                              "否则
	set t_Co=256                  "就开启256色支持，默认为8色                         
	set background=dark           "设置背景色为暗色
endif

"设置主题色为molokai
colorscheme molokai 



"""""""""""""""""""""""""""rainbow""""""""""""""""""""""""""""""""""
"自动启动插件
let g:rainbow_active=1   



"""""""""""""""""""""""""""auto-pairs"""""""""""""""""""""""""""""""
"设置F11为快速跳到下一个括号对的快捷键
let g:AutoPairsShortcutJump='<f11>' 



""""""""""""""""""""""""""molokai主题"""""""""""""""""""""""""""""""
"使用原始配色
"let g:molokai_original=1

"使用接近图形用户界面的配色
"let g:rehash256=1  "设置为1则更加亮眼



""""""""""""""""""""""""""youcompleteme"""""""""""""""""""""""""""""
" 设置触发标识符补全的最小字符数，设置为99或更大的数字，则等效于关闭标识符补全功能，但保留语义补全功能
let g:ycm_min_num_of_chars_for_completion=1

"设置要在标示符补全列表中候选的项的最小字符数，也可设置为其他值，设置为99或更大的数字，则等效于关闭补全功能，对语义补全无影响
let g:ycm_min_num_identifier_candidate_chars=1

"补全功能在注释中同样有效，1表示打开
let g:ycm_complete_in_comments=1

"用vim的语法标识符来建立标识符数据库
let g:ycm_seed_identifiers_with_syntax=1

"从ctags中收集标识符
let g:ycm_collect_identifiers_from_tags_files=0

"设置用于选择补全列表中的第一个选项以及进入补全列表后向下选择的快捷键，可以添加<Enter>键，默认为tab键和方向下键                          
let g:ycm_key_list_select_completion=['<Down>']

"设置用于向上选择补全列表中的选项的快捷键，默认为shift+tab，和方向上键
let g:ycm_key_list_previous_completion=['<Up>']

"配置ycm的全局路径，避免每次都复制到当前目录.若为空则每次都需赋值文件到当前目录                                                          
let g:ycm_global_ycm_extra_conf='~/.vim/bundle/YouCompleteMe/third_party/ycmd/.ycm_extra_conf_c.py' 
"let g:ycm_global_ycm_extra_conf='~/.vim/bundle/YouCompleteMe/third_party/ycmd/.ycm_extra_conf_cpp.py' 

" 允许自动加载.ycm_extra_conf.py，不再提示，设置为1，则每次都提示用于确认该文件是否安全                        
let g:ycm_confirm_extra_conf=0 

"设置运行jedi库的python解释器
let g:ycm_python_binary_path='/usr/bin/python3' 

"设置ycm服务的python解释器
let g:ycm_server_python_interpreter='/usr/bin/python3'

"跳转到声明的快捷键为<leader>gc
nnoremap <leader>gc :YcmCompleter GoToDeclaration<CR>                                                                                    

"跳转到定义的快捷键为<leader>gf
nnoremap <leader>gf :YcmCompleter GoToDefinition<CR>               

"跳转到头文件的快捷键为<leader>gh
nnoremap <leader>gh :YcmCompleter GoToInclude<CR>

"跳转到引用的快捷键为<leader>gr
nnoremap <leader>gr :YcmCompleter GoToReferences<CR>

"跳转到实现的快捷键为<leader>gi
nnoremap <leader>gi :YcmCompleter GoToImplementation<CR>

"跳转到类型的快捷键为<leader>gt
nnoremap <leader>gt :YcmCompleter GoToType<CR>

"显示类型的快捷键为<leader>st
nnoremap <leader>st :YcmCompleter GetType<CR>  

"显示语义父项的快捷键为<leader>sp
nnoremap <leader>sp :YcmCompleter GetParent<CR>

"显示文档的快捷键为<leader>sd
nnoremap <leader>sd :YcmCompleter GetDoc<CR>



"""""""""""""""""""""""""""""""""nerdcommit""""""""""""""""""""""""""""""""""""
"可在命令前加入数字，表示对连续n行执行相同操作
"\cc                                 注释当前选中文本，如果选中的是整行则在每行首添加//，如果选中一行的部分内容则在选中部分前后添加分别 /*xxx */；
"\cu                                 取消选中文本块的注释。                                                             
"\c<space>                           有注释取消，无注释加注释            
"\cm                                 注释多行只用一对符号注释，之前的注释都是每行独立注释 
"<leader>ci                          单独的行单独切换所选行的注释状态              
"\cs                                 使用好看的注释方式
"\cy                                 与cc一样只是在注释掉之前先复制原内容                           
"\c$                                 注释掉从光标到行尾的内容               
"\cA                                 在行位插入注释符号，并进入插入模式                    
"\ca                                 在/**/和//之间切换                              
"\cl                                 与cc相同，只是左对齐                                    
"\cb                                 与cc相同，只是两端对齐                   

"在c++中强制使用/**/注释,默认为//
let g:nerdaltdelims_cpp=1 

"按F9智能注释
nmap <f9> <plug>NERDCommenterToggle
vmap <f9> <plug>NERDCommenterToggle

"按F10在插入模式中注释
imap <f10> <plug>NERDCommenterInsert 



"""""""""""""""""""""""""""""""""""""airline"""""""""""""""""""""""""""""""""
"启动显示状态行1,总是显示状态行2
set laststatus=2

"命令行（在状态行下）的高度，默认为1，这里是2
set cmdheight=2

"设置状态栏主题
"let g:airline_theme='simple'



"""""""""""""""""""""""""""""""""""""ultisnips""""""""""""""""""""""""""""""""
"设置触发键
let g:ultisnipsexpandtrigger="<tab>"      

"后一个
let g:ultisnipsjumpforwardtrigger="<c-j>"  

"前一个
let g:ultisnipsjumpbackwardtrigger="<c-k>"  

"打开补全列表
let g:ultisnipslistsnippets="<c-tab>" 

"指定snippets文件的目录，可以有多个且该目录runtimepath 中的某一项之下
"let g:ultisnipssnippetsdir='~/.vim/Ultisnips' 

"指定snippets文件的搜索路径
"let g:ultisnipssnippetdirectories=['ultisnips'] 

"设置窗口打开的方式：normal当前窗口 horizontal水平vertical垂直context智能 
let g:ultisnipseditsplit="normal" 


"指定ultisnip使用python3
let g:ultisnipsusepythonversion=3 



""""""""""""""""""""""""""""""""""vim缩进"""""""""""""""""""""""""""""""""""""""""""
"不要自动多行显示
set nowrap

"显示制表符TAB
set list 

"使用|代替制表符，方便对齐显示，可以改为自己想要的符号,第一个字符显示一次，第二字符重复显示"
""set listchars=tab:\ \ 
set nolist

"是否将<tab>替换为空格
set noexpandtab


"已经存在文件里的 <Tab>的宽度 
set tabstop=8

"插入时<Tab>和<BS>的宽度，感觉上就像使用单个 <Tab> 一样，而实际上使用的是<Tab>+空格的混合。
"如果为零，关闭此特性。如果为负，使用 'shiftwidth'的值。如果置位 'paste'选项，'softtabstop' 被设为0。'paste' 复位时恢复本选项。
set softtabstop=8 

"自动缩进(换行或者选中行后使用>>或者<<进行缩进)每一级使用的空白数目.如果为零，使用 'tabstop' 的值。如果 expandtab 开启是插入的是Space
"如果noexpandtab 且 softtabstop 与 tabstop 一致时，是 TAB 符号。
set shiftwidth=8 

"打开后，在行首插入或删除<Tab> 根据 'shiftwidth' 插入或删除空白。而'tabstop' 或 'softtabstop' 用在别的地方。<backspace>删除行首 'shiftwidth' 那么多的空白
"如果关闭，<Tab> 总是根据 'tabstop' 或 'softtabstop' 决定插入空白的数目。'shiftwidth' 只用于文本左移或右移。 
set smarttab 


"设置允许退格的位置，0等同于set backspace=(Vi 兼容);1等同于set backspace=indent,eol;2等同于set backspace=indent,eol,start;
"其中indent 允许在缩进上退格，eol允许在换行符(行尾)上退格,start 允许在插入位置上退格
set backspace=2 

"autoindent自动对齐（继承前一行的缩进方式,智能缩进smartindent，c风格缩进cindent，自定义缩进indentexpr,后一种会覆盖前一种
if &filetype == 'c'
	set cindent "使用C样式的缩进  
elseif &filetype == 'cpp'
	set cindent "使用C样式的缩进
elseif &filetype == 'java' 
	set cindent "使用C样式的缩进  
else
	set smartindent "智能自动缩进（以c程序的方式）  
endif

"""""""""""""""""""""""""""""""""""vim 搜索/替换""""""""""""""""""""""""""""""""""""""""

"开启高亮显示结果   
set hlsearch 

"搜索到文件末不回到文件首 
set nowrapscan 

"输入搜索内容时实时显示结果，关闭则回车后显示 
set incsearch 

""""""""""""""""""""""""""""""""""vim编码方式"""""""""""""""""""""""""""""""""""""""""""

"设置 Vim 内部使用的字符编码。
set encoding=utf-8 

"设置当前编辑的文件的字符编码 
set fileencoding=utf-8 

"Vim 启动时会按照它所列出的字符编码方式逐一探测即将打开的文件的字符编码方式，并且将 fileencoding 设置为最终探测到的字符编码方式。
set fileencodings=utf-8,gbk,cp936,latin-1,ucs-bom,GB18030 "这是一个字符编码的列表，开始编辑已存在的文件时，参考此选项。

"""""""""""""""""""""""""""""""""""""""""""""""""vim高亮""""""""""""""""""""""""""""""""""""""""""""""""""
"开启语法高亮
syntax on 

"显示行号
set number 

"开启高亮光标行
set cursorline
highlight CursorLine  ctermbg=black ctermfg=NONE

"开启高亮光标列
set cursorcolumn
highlight CursorColumn ctermbg=black  ctermfg=darkyellow

"Vim识别三种不同的终端：term，黑白终端；cterm，彩色终端；gui，gvim窗口。
"term,cterm,gui可以定义其字体显示为：bold、underline、reverse、italic或standout。
"ctermfg设置前景色；ctermbg设置背景色。

"高亮显示匹配的括号,类似当输入一个左括号时会匹配相应的那个右括号
set showmatch 

"匹配括号高亮的时间（单位是十分之一秒）
set matchtime=4 

"""""""""""""""""""""""""""""""""""vim 其它设置""""""""""""""""""""""""""""""""""""""""

"代码补全功能，可以选择preview以在顶配生成一个选项预览窗口，menu选项以弹出菜单出现，
"或者longest，在输入位置只显示最长的匹配，menuone仅有一个选项也弹出菜单
set completeopt=longest,menu 

"去掉输入错误的提示声音
set noerrorbells

"将当前工作目录切换至所打开的文件目录
set autochdir

"设置tags文件的搜索路径
set tags+=./tags

```
### 6.4.6 安装其它插件
```
$vim
esc
:PluginInstall
```
等待显示done，完成。
### 6.4.7 配置ultisnip
```
$ mkdir ~/.vim/Ultisnips
$ cp c_linux.snippets ~/.vim/Ultisnips/c.snippets
$ cp cpp_linux.snippets ~/.vim/Ultisnips/cpp.snippets
```
## 6.5 安装帮助信息
&emsp;&emsp;一般只装最后一个
```
$ sudo apt-get install manpages  //包含 GNU/Linux 的基本操作
$ sudo apt-get install manpages-dev //包含 GNU/Linux 的基本操作API
$ sudo apt-get install manpages-posix //包含 POSIX 所定义公用程序的方法
$ sudo apt-get install manpages-posix-dev //包含 POSIX 的 header files 和 library calls 的用法
```

# 7 Host SMBus controller not enabled

* 查明装入模块的确切名字，显示输出的结果是模块的确切名字：i2c_piix4
* 将该模块列入不装入名单。
```
$ sudo vim /etc/modprobe.d/blacklist.conf
```
* 在末尾加入blacklist i2c-piix4
* 保存退出
* 重启
 
# 8 显示设置
```
$cd ~
$vim ~/.bashrc
```
* 改LINUX用户命令行提示为为彩色显示,在末尾添加：
```
export PS1="\[\033[1;32m\][\[\033[0;36m\]\u\[\033[1;35m\]@\[\033[0;33m\]\h:\[\033[0;34m\]\W\[\033[1;32m\]]\[\033[0;31m\]\$ \[\033[0m\]"
```
* 让linux支持256色的终端 ,在末尾添加：
```
if [ -e /lib/terminfo/x/xterm-256color ]
then
	export TERM='xterm-256color'
else
	export TERM='xterm-color'
fi
```
* 查看服务器颜色位数
```
$tput colors 
```
* 显示当前支持的终端的类型
```
$ echo $TERM 
```
* 如果你使用putty为终端，改变putty显示背景,使用系统颜色配置不打勾。
* 终端颜色配置，推荐护眼颜色为前景色131，148，150，背景色为0,43,53,颜色方案也适用于xshell。

# 9 防止rm误删除
* 将remove 文件夹的文件拷贝到/usr/local/bin
* 在.bashrc文件后添加
```
alias rm='/usr/local/bin/rm.sh'
alias lrm='/usr/local/bin/lrm.sh'
alias urm='/usr/local/bin/urm.sh'
```

# 10 安装第三方库

## 10.1 安装 log4 for cpp
 
* 下载log4cpp-1.1.1.tar.gz           
* 安装：以root权限打开终端，
* 拷贝到/usr/local
```
# cd /usr/local
# tar zxvf log4cpp-1.1.1.tar.gz 
# cd /usr/local/log4cpp/
# ./configure
# make
# make check
# make install
```
* 这里已经安装成功.
默认lib库路径是 ： /usr/local/lib/
默认头文件的位置： /usr/local/include/log4cpp                    
* 使用:
 * 编译使用log4cpp库的CPP文件时，要加上库文件，才能顺利的编译通过。如下示例：`# g++ log4test.cpp -llog4cpp -lpthread`
 * 运行时，如若提示缺少log4cpp库文件，表示找不到log4cpp的动态库，需要进行以下设置以管理员身份登录终端，然后执行以下操作：
     * `# vim /etc/ld.so.conf`
     * 在打开的文件末尾添加动态库log4cpp的路径(这里是/usr/local/lib)，然后保存退出；执行命令ldconfig使设置生效即可。
     * `# ldcondfig`
* log4cpp学习:
	http://blog.csdn.net/liuhong135541/article/category/1496383
	http://log4cpp.sourceforge.net/
	http://en.cppreference.com



## 10.2 安装boost库
* 下载好了Boost库，下面开始编译(假设下载完后的，代码解压在了BOOST_ROOT目录)
* 进入到BOOST_ROOT/libs/regex/build目录
  * 如果要使用静态库，请执行make -fgcc.mak
  * 如果要使用动态库，请执行make -fgcc-shared.mak
* 执行完上面三步后的，在BOOST_ROOT/libs/regex/build/下会生成一个gcc目录 ，进入该目录 ，可以看到生成了下面四个文件:
 * libboost_regex-gcc-1_42.a , 这是release版的静态库
 * libboost_regex-gcc-1_42.so, 这是release版的动态库（共享库）
 * libboost_regex-gcc-d-1_42.a, 这是debug版的静态库
 * libboost_regex-gcc-d-1_42.so, 这里debug版的动态库（共享库）
* 在使用之前你需要把Boost的安装目录加入到系统的Path中(当然也可以在编译时直接指定）
     * 第一种办法：复制编译好的库文件到`/usr/local/lib`，复制头文件 `boost/regex.hpp`到`/usr/local/include`
     * 第二种办法：打开~/.bashrc 添加以下内容后，执行`sudo ldconfig`
     
```
export CPLUS_INCLUDE_PATH=$BOOST_ROOT:$CPLUS_INCLUDE_PATH
exoprt LIBRARY_PATH=$BOOST_LIB_PATH:$LIBRARY_PATH
export LD_LIBRARY_PATH=$BOOST_SO_LIB_PATH:$LD_LIBRARY_PATH
```
* 编译要指定链接的库文件，如果名字太长，可以使用`sudo ln -s 源文件 目标文件` 来创建一个短的库文件的链接










------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：<font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
