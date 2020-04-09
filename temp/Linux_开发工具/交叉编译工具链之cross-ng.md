---
title: 交叉编译工具链之cross-ng
tags: Linux项目管理
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《crosstoll-ng官方文档》](http://crosstool-ng.github.io/docs/)。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

<style>table{word-break:initial;}</style>

# 1 介绍

crosstool-NG旨在构建工具链。工具链是软件开发项目中必不可少的组件。它将编译、汇编和链接正在开发的代码。工具链的某些部分最终会出现在最终的二进制文件中：静态库只是一个例子。

因此，工具链是一个非常敏感的软件，因为其中一个组件中的任何错误或配置不当都可能导致执行问题，包括性能不佳，应用程序意外结束，以及行为不端的软件（通常很难检测到），硬件损坏，甚至人类风险（这是令人遗憾的）。

工具链是由不同的软件组成的，每个软件都非常复杂，需要特别设计的选项才能无缝地构建和工作。这通常不是那么容易，即使在本地工具链的情况下也是如此。当涉及到交叉编译时，工作达到了更高的复杂性，在这种情况下，交叉编译可能成为一场噩梦……

互联网上存在一些交叉工具链，可用于一般开发，但它们有许多局限性：

* 它们可以是通用的，因为它们是为大多数配置的：没有针对您的特定目标进行优化。
* 它们可以针对特定目标做好准备，因此不易使用，也不优化甚至支持您的目标。
* 他们经常使用老化的组件（编译器，C库等），不支持闪亮的新处理器的特殊功能。

另一方面，这些工具链也有一些优势：
* 它们随时可用，并且非常易于安装和设置。
* 如果被广泛的社区使用，它们就被证明了。

但是，一旦您想要从特定硬件中获得所有的功能，您就需要构建自己的工具链。这就是使用crosstool-NG的原因。

还有一些工具可以为特定的需求构建工具链，但它们并不是真正可扩展的。例子有:

* [builidroot](http://buildroot.uclibc.org/):：其主要目的是构建根文件系统，因此得名。但是一旦您拥有了带有buildroot的工具链，它的一部分就会安装在将来的根目录中，所以如果您想构建一个全新的根目录，您就必须将现有的根目录保存为模板，稍后再进行恢复，或者重新启动。这并不方便。
* [ptxdist](http://www.ptxdist.org/)，其用途与buildroot非常相似。
* 其他项目(例如openembedded.org)，它们也被用来构建根文件系统。

crosstool-NG真正的目标是构建工具链，而且仅仅是工具链。然后由您决定如何使用它。

**历史**

crosstool最初由Dan Kegel构思，他将其作为一组脚本，一个补丁库以及一些预配置的通用设置文件提供给社区，用于配置crosstool。这可以在 kegel.com/crosstool 上找到，subversion存储库托管在[Google Code](http://code.google.com/p/crosstool/)上。

Yann E. Morin曾经设法增加对基于uClibc的工具链的支持，但它没有进入主线，主要是因为Yann没有时间将补丁移植到新版本，部分原因是它需要巨大的努力。

因此，Yann决定清理crosstool，重新整理所需的东西，为需要的东西添加适当的支持，如uClibc支持和菜单驱动的配置方式，重命名为crosstool-NG（表示下一代的crosstool），并将其提供给社区。

2014年底，Yann忙于buildroot和其他项目，所以Bryan Hundven成为crosstool-NG的新维护者。

**关于cross-NG**

该项目的长名称是crosstool-NG：

* 没有前导大写字母（句子中的第一个单词除外）
* 用连字符（破折号）分隔的crosstool和NG，
* NG大写。

Crosstool-NG也可以简称为CT-NG：

* 都是大写的，
* CT和NG用连字符分隔（破折号）。

长名称比短名称更受欢迎，但邮件主题除外，短名称更适合。

当引用特定版本的crosstool-NG时，请将版本号附加为:

* crosstool-NG X.Y.Z(长名称、空格和版本字符串)
* crosstool-ng-X.Y.Z(小写的长名称、连字符(破折号)和版本字符串)——用于命名发布tarballs
* crosstool-ng-X.Y.Z+git_id(小写的长名称、连字符、版本字符串和ct-ng version返回的Git ID)——用于区分版本和快照

crosstool-NG的前端命令是ct-ng：

* 全部用小写，
* ct和ng用连字符（破折号）分隔。


# 2  安装crosstool-NG

在安装crosstool-NG之前，您可能需要在主机操作系统上安装其他软件包。这里提供了几个支持的操作系统和发行版的具体说明。请注意，当前配置脚本不能检测到所有依赖项;缺少其中一些可能会导致ct-ng build失败。
## 2.1 获取源码
有两种方法可以获得crosstool-NG源码。
* 下载已发布的源码包
* 克隆仓库

### 2.1.1  下载源码包

首先，下载源码包
```bash
wget http://crosstool-ng.org/download/crosstool-ng/crosstool-ng-VERSION.tar.bz2

#或者

wget http://crosstool-ng.org/download/crosstool-ng/crosstool-ng-VERSION.tar.xz
```

从1.21.0开始，发行时用Bryan Hundven的PGP密钥签署。指纹是：
```bash?linenums=false
561E D9B6 2095 88ED 23C6 8329 CAD7 C8FC 35B8 71D1
```
从1.23.0开始，发行时用Alexey Neyman的PGP密钥签署。指纹是：
```bash?linenums=false
64AA FBF2 1475 8C63 4093 45F9 7848 649B 11D6 18A4
```
公钥可在 http://pgp.surfnet.nl/ 上找到。要验证发布的源码包，您需要从密钥服务器导入密钥并下载tarball的签名，然后在同一目录中验证：

```bash
gpg --keyserver http://pgp.surfnet.nl --recv-keys 35B871D1 11D618A4
wget http://crosstool-ng.org/download/crosstool-ng/crosstool-ng-VERSION.tar.bz2.sig
gpg --verify crosstool-ng-VERSION.tar.bz2.sig
```
Crosstool-NG 1.19.0及更早版本为tarball提供MD5 / SHA1 / SHA512摘要。使用md5sum / sha1sum / sha512sum命令验证源码包：

```bash
md5sum -c crosstool-ng-VERSION.tar.bz2.md5
sha1sum -c crosstool-ng-VERSION.tar.bz2.sha1
sha512sum -c crosstool-ng-VERSION.tar.bz2.sha512
```

### 2.1.2 克隆仓库


如果发布的版本不够用于您的目的，您可以尝试使用当前开发的版本进行构建。为此，克隆Git存储库：
```bash
git clone https://github.com/crosstool-ng/crosstool-ng
```
在运行configure之前，您需要运行bootstrap脚本:
```bash
./bootstrap
```

## 2.2 使用crosstool-NG
您还可以通过两种方式使用crosstool-NG：

* 构建并安装它 ，然后摆脱源码，就像像大多数程序那样；
* 或者只构建它，然后从源目录运行。



### 2.2.1 构建并安装

首先解压缩源码包并cd进入crosstool-ng-VERSION目录。
```
tar -xf 源码包
cd crosstool-ng-VERSION
```
>1.22.0版本由于发布脚本的一些bug，没有VERSION后缀

然后按照经典配置方式: `./configure` :

```bash
./configure --prefix=/some/place #配置
make                    #编译
make install               #安装
export PATH="${PATH}:/some/place/bin" #添加执行程序的搜索路径
```
>此处如果不指定–prefix 那么默认目录是/usr/local

然后你可以退出crosstool-NG源码目录。创建一个目录作为工作区，进入该目录并运行：

```bash
mkdir work-dir
cd work-dir
ct-ng help
```
>如果使用ct-ng --help，则会进入make（1），因为cn-ng实际是一个make脚本。

还安装了ct-ng实用程序的man手册页。您可以输入man ct-ng来获得一些简短的帮助。

### 2.2.2 构建不安装

```bash
./configure --enable-local
make
```
现在，不要删除crosstool-NG源码目录。他们需要用于运行crosstool-NG！留在该目录中，然后运行
```bash
./ct-ng help
```

## 2.3 打包准备 
如果您计划打包crosstool-NG，那么肯定不希望在根文件系统中安装它。crosstool-NG的安装过程支持DESTDIR变量:
```
./configure --prefix=/usr
make
make DESTDIR=/packaging/place install
```
## 2.4 shell补全
crosstool-NG附带了一个shell脚本片段，它定义了与bash兼容的补全。该shell片段目前没有自动安装。
要安装shell脚本片段，您有两个方式:

* 为所有用户安装：`cp ct-ng.comp  /etc/bash_completion.d/`
* 为特定用户安装：`cp ct-ng.comp ${HOME}/`然后，在`${HOME}/.bashrc`添加`source ${HOME}/ct-ng.comp`



## 2.5 贡献的代码

有些人提供的代码由于各种原因无法合并。此代码在contrib /子目录中以lzma压缩补丁的形式提供。在安装之前，使用以下方式将这些补丁应用于crosstool-NG的源：
```
lzcat contrib/foobar.patch.lzma | patch -p1
```

无法保证特定的补丁适用于当前版本的crosstool-ng，或者它将完全起作用。使用这些补丁需要您自担风险。

# 3 配置cross-NG





------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《crosstoll-ng官方文档》](http://crosstool-ng.github.io/docs/)。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
