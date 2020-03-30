---
title: Linux软件管理(未完待续)
tags: Linux使用教程
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《Linux man pages》](https://linux.die.net/man/ "点击跳转")。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------



起初 GNU/Linux 的世界中只有.tar.gz。如果用户要使用一个软件，那就必须自己编译。当 Debian 诞生以后，一种能管理操作系统中已安装的软件包的系统显得很有必要，这个系统被命名为‘dpkg’。‘软件包’一词在此第一次出现在 GNU/Linux。不久之后，红帽公司创建了他们自己的包管理系统‘rpm’。

GNU/Linux 的创造者们很快又陷入了新的窘境。他们希望通过一种快捷、实用而且高效的方式来安装软件包，并能自动处理相互之间的依赖关系，还要在软件包升级过程中维护好配置文件。Debian 又一次充当了开路先锋的角色，首创了 APT（Advanced Packaging Tool，高级软件包管理工具）。这一工具后来被 Conectiva 移植到红帽公司的 rpm 包管理系统叫做yum。在其他一些发行版中，我们也能看到 APT 的身影。



|distribution | 本地软件管理机制 |本地管理指令 |在线管理机制|在线管理指令
|:--|:--|:--|:--|:--|
|Red Hat/Fedora| RPM |rpm|YUM |yum
|Debian/Ubuntu| DPKG |dpkg |APT |apt套件

# 1 在线管理机制

## 1.1 APT

### 1.1.1 重要的相关目录和文件
* `/etc/apt/sources.list`
存放的是软件源站点, 当你执行 `sudo apt-get update` 时，Ubuntu 就去这些站点下载软件包列表到`/var/lib/apt/lists/`目录中。
* `/etc/apt/sources.list.d/`
该文件夹下的文件是第三方软件的源，可以分别存放不同的第三源地址，只需“扩展名”为list即可，
* `/var/cache/apt/archives/`
目录是在用` apt-get install` 安装软件时，软件包的临时存放路径
*  `/var/cache/apt/archives/partial/`
传输中软件包。

* `/var/lib/apt/lists/`
使用`apt-get update`命令会从`/etc/apt/sources.list`的源站点中下载软件列表，并保存到该目录，执行`apt-get`安装或升级时，会访问该目录的软件列表文件，以获取包信息。
*  `/var/lib/apt/lists/partial/`
传输中的源的状态信息。
* `/etc/apt/apt.conf`
APT配置文件
* `/etc/apt/apt.conf.d/`
APT 配置文件,存放各个单独的配置
* `/etc/apt/preferences`
版本首选项   
* `/etc/apt/preferences.d/`
版本首选项， 存放各个单独的版本首选项 
* `/var/lib/dpkg/available`
文件的内容是软件包的描述信息, 该软件包括当前系统所使用的 Debian 安装源中的所有软件包,其中包括当前系统中已安装的和未安装的软件包.

      

### 1.1.2 工作原理
Ubuntu采用集中式的软件仓库机制，将各式各样的软件包分门别类地存放在软件仓库中，进行有效地组织和管理。然后，将软件仓库置于许许多多的镜像服务器中，并保持基本一致。这样，所有的Ubuntu用户随时都能获得最新版本的安装软件包。因此，对于用户，这些镜像服务器就是他们的软件源（Reposity）。

然而，由于每位用户所处的网络环境不同，不可能随意地访问各镜像站点。为了能够有选择地访问，在Ubuntu系统中，使用软件源配置文件`/etc/apt/sources.list`列出最合适访问的镜像站点地址。

无论用户使用哪些手段配置APT软件源，只是修改了配置文件`/etc/apt/sources.list`，目的只是告知软件源镜像站点的地址。但那些所指向的镜像站点所具有的软件资源并不清楚，若每安装一个软件包，就在服务器上寻找一遍，效率是很低的。因而，就有必要为这些软件资源列个清单（建立索引文件），以便本地主机查询。

索引文件的包信息主要有：Package、Architecture、Multi-Arch、Version、Priority、Section、Origin、Maintainer、Original-Maintainer、Bugs、Installed-Size、Provides、Depends、Breaks、Filename、Size、MD5sum、SHA1、SHA256、Homepage、Description、Task、Description-md5: 、Supported、conflict

用户可以使用`apt-get update`命令更新软件包列表。在Ubuntu 中，`apt-get update`命令会扫描每一个软件源服务器，并为该服务器所具有软件包资源建立索引文件，存放在本地的`/var/lib/apt/lists/`目录中。使用apt-get执行安装、更新操作时，都将依据这些索引文件，获取相应包的信息，然后向软件源服务器申请资源。同时，APT能够检查Linux系统中的软件包依赖关系，大大简化了Ubuntu用户安装和卸载软件包的过程。

使用`apt-get install`安装软件包大体分为4步：

1. 第一步，扫描本地存放的软件包更新列表（由`apt-get update`命令刷新更新列表，也就是`/var/lib/apt/lists/`），找到最新版本的软件包；
2. 第二步，进行软件包依赖关系检查，找到支持该软件正常运行的所有软件包；
3. 第三步，从软件源所指的镜像站点中，下载相关软件包；
4. 第四步，解压软件包，并自动完成应用程序的安装和配置。

不过，APT并不是指某个命令，而是一组命令：
|常用APT命令|作用
|--|--|
|apt-add-repository|添加或者删除源站点|
|**apt-cache**|查询软件包信息；
|apt-cdrom|将CD-ROM加入软件源配置文件。
|apt-config|用于读取APT配置文件的简单工具；
|aptd|
|aptdcon|
|apt-extracttemplates|提取一个或多个软件包的配置脚本和模板文件成
|apt-file|查到软件包所含的文件和安装的位置
|apt-ftparchive  |生成APT用来访问分发源的索引文件。
|**apt-get**|管理软件包，包括安装、卸载、升级等操作；
|apt-key|管理apt认证包时使用的密钥。
|apt-mark|作为一个统一的前端来设置一个包的各种设置
|apt-mirror|创建镜像站点
|apt-proxy|用于搭建APT代理服务器；
|apt-sortpkgs | 采用索引文件（源索引或包索引）对记录的包行排序
|**apt**|apt-get、apt-cache、apt-config常用命令的高级接口|
|apturl|通过点击URL安装软件|
|apt-url-gtk
|aptitude|整合dselect和apt-get的功能|
|aptoncd|是将安装的软件包备份到光盘或ISO映像的简单方法

### 1.1.3 设置APT源
&emsp;&emsp;在安装Linux时，系统会根据用户所选择的国家/时区，推荐合适的软件源地址。通常，用户使用默认的`/etc/apt/sources.list`文件就可以。不过，当用户发现更合适的软件源镜像站点时，可以重新设置APT源。sources.list 条目格式：
```
DebType  Repository_URL  Distribution  Component1  Component2……
```
* DebType表示Deb软件包类型
	*  deb表示二进制软件包
	*  deb-src表示源码包；
* Repository URL表示软件包所在仓库的地址。我们可以更换仓库来提高下载速度。
* Distribution表示发行版本，发行版有两种分类方法：
	* 第一类是发行版的具体代号，如 xenial（16.04 非洲地松鼠）, trusty（可靠的塔尔羊）, precise （精准的穿山甲）等；
	* 第二类则是发行版的发行类型，如 oldstable, stable, testing 和 unstable。
	* 另外，在发行版后还可能有进一步的指定，如 xenial-updates, xenial-security, xenial-backports 等。
* Component表示软件包组件类别
	* Debian
		 * main——包含符合 DFSG指导原则的自由软件包，而且这些软件包不依赖不符合该指导原则的软件包。这些软件包被视为 Debian 发型版的一部分。
		* contrib——包含符合 DFSG 指导原则的自由软件包，不过这些软件包依赖不在 main 分类中的软件包。
		* non-free——包含不符合 DFSG 指导原则的非自由软件包。
	* Ubuntu 
		* main——官方支持的自由软件。
		* restricted——官方支持的非自由的软件。
		* universe——非官方支持的自由软件。
		* multiverse——非官方支持的非自由软件。

### 1.1.4 添加删除仓库——`apt-add-repository`
&emsp;&emsp;**命令**：apt-add-repository [OPTIONS] REPOSITORY
&emsp;&emsp;**功能**：将一个APT仓库添加到`/etc/apt/sources.list`或`/etc/apt/sources.list.d/`中或删除已存在的存储库。
|常用选项|描述|
|--|--|
|-m|将大量调试信息打印到命令行中
|-r|删除指定的存储库
|-y|假设所有查询都是|
|-k|使用自定义密钥服务器URL而不是默认值
| -s|允许从存储库下载源包
|**更多信息**|<http://linux.51yip.com/search/apt-add-repository> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/apt-add-repository>|

###1.1.5 查询软件包信息——`apt-cache`
&emsp;&emsp;**命令**：apt-cache subcommand [options] [pkg_list]
&emsp;&emsp;**功能**：apt-cache在APT的包缓存上执行各种操作。 apt-cache不会改变系统状态，但会提供操作以从软件包元数据中生成感兴趣的结果。元数据是通过`apt-get update`命令获取和更新的。 


|常用子命令|描述|
|--|--|
|gencaches|创建APT的包缓存。
|showpkg pkg|获取二进制软件包的常规描述信息
|showsrc pkg|获取源码包的详细描述信息
|show pkg|获取二进制软件包的详细描述信息
|stats|获取软件源的基本统计信息
|dump|获取软件源中所有软件包的简要信息
|dumpavail|获取当前中已安装的所有软件包的描述信息
|unmet|获取所有未满足的依赖关系
|search|根据正则表达式检索软件包
|depends pkg|获取该软件包的依赖信息
|rdepends pkg|获取所有依赖于该软件包的软件包
|pkgnames|列出所有已安装软件包的名字
|policy|获取软件包当前的安装状态
|**常用选项**|**描述**|
|-p|软件包的缓存
|-s|源代码包的缓存
|-q|关闭进度获取
|-i|获取重要的依赖关系，仅与unmet命令一起使用
|-n|只搜索包名，不搜索描述
|-f|搜索时显示完整的包信息
|-c file|指定配置文件
|-o strings|设置配置选项
|**更多信息**|<http://linux.51yip.com/search/apt-cache> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/apt-cache>|

###1.1.5  将CD-ROM加入源—`apt-cdrom`
&emsp;&emsp;**命令**：apt-cdrom  subcommands  [-d|-r|-m|-f|-a|-h|-c config_name ]
&emsp;&emsp;**功能**：用于检查Ubuntu安装光盘，也可将安装光盘作为软件源添加到软件源配置文件中

|常用子命令|描述|
|--|--|
|ident|用于扫描Ubuntu安装光盘，
|add|用于向`/etc/apt/sources.list`文件添加CD-ROM配置项。
|**常用选项**|**描述**|
|-d|指定CD-ROOM的挂载点
|-r|为一个已经识别CD-ROM改名
|-m|声明CD-ROOM无挂载点
|-f|快速模式，不检查软件包文件
|-a|完整扫描模式
|-c file|指定配置文件
|-o strings|设置配置选项
|**更多信息**|<http://linux.51yip.com/search/apt-cdrom> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/apt-cdrom>|

### 1.1.6 软件管理——`apt`、`apt-get`
####1.1.6.1 apt与apt-get的关系
&emsp;&emsp;在基于 Debian 的 Linux 发行版中，有各种工具可以与 APT 进行交互，以方便用户安装、删除和管理的软件包。apt-get 便是其中一款广受欢迎的命令行工具，另外一款较为流行的是 Aptitude 这一命令行与 GUI 兼顾的小工具。

&emsp;&emsp;apt套件包含许多工具，如apt-cache、apt-config 等。这些命令都比较低级又包含众多功能，普通的 Linux 用户也许永远都不会使用到。换种说法来说，就是最常用的 Linux 包管理命令都被分散在了 apt-get、apt-cache 和 apt-config这三条命令当中。

&emsp;&emsp;apt 命令的引入就是为了解决命令过于分散的问题，它包括了 apt-get 命令出现以来使用最广泛的功能选项，以及 apt-cache 和 apt-config 命令中很少用到的功能。

&emsp;&emsp;在使用 apt 命令时，用户不必再由 apt-get 转到 apt-cache 或 apt-config，而且 apt更加结构化，并为用户提供了管理软件包所需的必要选项。

&emsp;&emsp;简单来说就是：apt = apt-get、apt-cache 和 apt-config 中最常用命令选项的集合。

####1.1.6.2 apt命令的特点
* 用户可以在同一地方集中得到所有必要的工具，不必再由 apt-get 转到 apt-cache 或 apt-config
* apt具有更精减但足够的命令选项，而且参数选项的组织方式更为有效。
* 命令安装或删除程序时有进度条。
* 在更新存储库数据库时提示用户可升级的软件包个数。

#### 1.1.6.3 软件管理命令——`apt-get`
&emsp;&emsp;**命令**：apt-get subconmmand [options]
&emsp;&emsp;**功能**：apt-get命令本身并不具有管理软件包功能，只是提供了一个软件包管理的命令行平台。在这个平台上使用更丰富的子命令，完成具体的管理任务。
|常用子命令|描述|
|--|--|
|update|下载更新软件包列表信息
|upgrade|将系统中所有软件包升级到最新的版本
|dist-upgrade|发布版升级
|dselect-upgrade|根据dselect的选择来进行软件包升级
|install|下载所需软件包并进行安装配置
|remove|卸载软件包，但保留配置文件
|purge|卸载软件包和配置文件
|autoremove|根据依赖关系卸载软件包和卸载不需要或不满足依赖关系的软件包
|source|下载源码包
|build-dep|为源码包构建所需的编译环境
|clean|删除缓存区中所有已下载的包文件
|autoclean|删除缓存区中老版本的已下载的包文件
|check|检查系统中依赖关系的完整性
|download|下载二进软件包到当前文件夹
|**常用选项**|**描述**|
|-d|仅下载软件包，而不安装或解压
|-f|修复系统中存在的软件包依赖性问题
|-m|当发现缺少关联软件包时，仍试图继续执行
|-q|将输出作为日志保留，不获取命令执行进度
|--purge|与remove子命令一起使用，完全卸载软件包，等效于purge子命令
|--reinstall|与install子命令一起使用，重新安装软件包
|-b|在下载完源码包后，编译生成相应的软件包
|-s|不做实际操作，只是模拟命令执行结果
|-y|对所有询问都作肯定的回答，apt-get不再进行任何提示
|-u|获取已升级的软件包列表
|-V|显示安装、升级过程
|-u|显示要升级的包
|**更多信息**|<http://linux.51yip.com/search/apt-get> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/apt-get>|

#### 1.1.6.4 软件管理高级命令——`apt`

&emsp;&emsp;**命令**：apt subconmmand [options] [pkg]
&emsp;&emsp;**功能**：apt命令本身并不具有管理软件包功能，只是提供了一个软件包管理的命令行平台。在这个平台上使用更丰富的子命令，完成具体的管理任务。
|常用命令|	描述|
|--|--|--|
|install | 下载所需软件包并进行安装配置|
|remove |移除软件包,但保留配置文件|
|autoremove	|根据依赖关系卸载软件包和卸载不需要或不满足依赖关系的软件包
|purge |移除软件包及配置文件|
|search |搜索应用程序
|show |显示软件包详细信息
|update|刷新存储库索引|
|upgrade|升级所有可升级的软件包|
|full-upgrade|在升级软件包时自动升级依赖包
|list |列出包，--installed，已安装；--upgradable可升级； --all-versions所有版本
|edit-sources|编辑源文件
|**更多信息**|<http://linux.51yip.com/search/apt> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/apt>|

#### 1.1.6.5 apt与apt-get对照表
|apt 命令|	取代的命令|	命令的功能|
|--|--|--|
|`apt install 包名`| `apt-get install 包名`|安装软件|
|`apt remove 包名`|	` apt-get remove 包名`|移除软件，保留配置|
|`apt autoremove`	| `apt-get autoremove`|自动卸载软件
|`apt purge 包名`|	`apt-get purge 包名`|移除软件包及配置文件|
|`apt search 包名`|`apt-cache search 包名`|搜索应用程序
|`apt show 包名`	|`apt-cache show 包名`|显示软件包详细信息
|`apt update`	| `apt-get update`|刷新存储库索引|
|`apt upgrade`	| `apt-get upgrade`|升级所有可升级的软件包|
|`apt full-upgrade`|	`apt-get dist-upgrade`|升级软件和依赖包
|`apt list –installed`| `dpkg -l`|列出已安装的软件
|`apt list –upgradable`|`apt-get -u upgrade –assume-no`|列出可升级的软件
|`apt edit-sources`|`vim /etc/apt/sources.list`|编辑源文件

### 1.1.7 配置APT——`apt-config`
&emsp;&emsp;**命令**：apt-config [--empty] [--format '%f "%v";%n'] [-o=config_string] [-c=config_file] {shell | dump | {-v | --version} | {-h | --help}}
&emsp;&emsp;**功能**：apt-config用于设置APT或读取APT配置文件
|常用子命令|描述|
|--|--|
|shell|从shell脚本中获取配置信息
|dump|显示配置信息
|**常用选项**|**描述**|
|--empty|输出中包含值为空的选项|
|--no-empty|输出中不包含值为空的选项
|--format '%f "%v";%n'|定义每个配置选项的输出。％t输出选项名，％f输出完整的层次结构名称，％v输出选项值。％n将被替换为换行符，而％N将被替换为制表符。 ％可以使用%%打印。
|-c file|指定配置文件
|-o|设置配置选项，语法是-o Foo::Bar=bar|
|**更多信息**|<http://linux.51yip.com/search/apt-config> 和 man 手册|
|&emsp;&emsp;&emsp;&emsp;|<http://man.linuxde.net/apt-config>|

## 1.2 YUM
&emsp;&emsp;redhot/centos使用`yum`来安装、卸载、搜索、查看应用程，会把所有的依赖包都一起安装。
```
sudo yum install：安装rpm软件包；
sudo yum update：更新rpm软件包；
sudo yum check-update：检查是否有可用的更新rpm软件包；
sudo yum remove：删除指定的rpm软件包；
sudo yum list：显示软件包的信息；
sudo yum search：检查软件包的信息；
sudo yum info：显示指定的rpm软件包的描述信息和概要信息；
sudo yum clean：清理yum过期的缓存；
sudo yum shell：进入yum的shell提示符；
sudo yum resolvedep：显示rpm软件包的依赖关系；
sudo yum sudo yum localinstall：安装本地的rpm软件包；
sudo yum localupdate：显示本地rpm软件包进行更新；
sudo yum deplist：显示rpm软件包的所有依赖关系。
```
#2 本地管理机制
##2.1 dpkg命令
dpkg安装deb包
Ubuntu软件包格式为deb，安装方法如下：

    sudo  dpkg  -i  package.deb

dpkg的详细使用方法，网上有很多，下面简单列了几个：

dpkg -i package.deb	安装包
dpkg -r package	删除包
dpkg -P package	删除包（包括配置文件）
dpkg -L package	列出与该包关联的文件
dpkg -l package	显示该包的版本
dpkg –unpack package.deb	解开 deb 包的内容
dpkg -S keyword	搜索所属的包内容
dpkg -l	列出当前已安装的包
dpkg -c package.deb	列出 deb 包的内容
dpkg –configure package	配置包
根据Ubuntu中文论坛上介绍，使用apt-get方法安装的软件，所有下载的deb包都缓存到了/var/cache/apt/archives目录下了，所以可以把常用的deb包备份出来，甚至做成ISO工具包、刻盘，以后安装Ubuntu时就可以在没有网络环境的情况下进行了。下面的命令是拷贝archives这个目录到/var/cache/apt/目录下，替换原有的archives

enadmin@ubuntu-server:~/ftp$ sudo cp -r archives/ /var/cache/apt/
##2.2 rpm

#3 源代码安装
&emsp;&emsp;如果要使用make安装的话，那么必须得安装build-essential这个依赖包。源码安装大致可以分为三步骤：

* `./configure`——配置编译选项，选项可通过 --help 查询。也有某些程序无需执行此步。
* `make`——编译：一旦配置通过，可即刻使用make指令来执行源代码的编译过程。
* 安装
 * `make install`——直接安装：如果编译没有问题，那么执行`make install` 就可以将程序安装到系统中了。
 * `check install`——生成deb/rpm包。

# 2 本地管理机制

---

&emsp;&emsp;***<font color=blue>版权声明</font>***：*本文章参考[《Linux man pages》](https://linux.die.net/man/ "点击跳转")做了修改，增添了一部分内容。<font color=red>未经作者允许，**<font color=blue>严禁用于商业出版</font>**，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>*

---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
