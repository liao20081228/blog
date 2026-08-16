---
title: ftp协议及Linux下搭建ftp服务器
tags: Linux使用教程
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>《vsftp官方文档》。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------




# 1 FTP协议解析
文件传输协议有基于TCP的FTP和基于UDP的简单文件传输协议TFTP，它们都是文件共享协议中的一大类，即复制整个文件，其特点是：若要存取一个文件，就必须先获得一个本地的文件副本。如果要修改文件，只能对文件的副本进行修改，然后再将修改后的文件传回到原节点。

文件传输协议FTP(File Transfer Protocol)是因特网中使用最广泛的**应用层**文件传输协议。FTP使用交互式的访问，允许客户指定文件的类型和格式(如指明是否使用ASCII码)，并允许文件具有存取权限(如访问文件的用户必须经过授权，并输入有效的口令)。


## 1.1 术语
**FTP**（FILE TRANSFER PROTOCOL）：文件传输协议。

**PI**（protocol interpreter）：协议解析器。用户和服务器用其来解析协议，它们的具体实现分别称为用户 PI （USER-PI）和服务器PI（SERVER-PI）。

 **用户PI**（user-PI）：用户协议解析器用 U 端口建立到服务器 FTP 过程的控制连接，并在文件传输时管理用户 DTP。
 
**服务器PI**（server-PI）：服务器 PI 在 L 端口“监听”用户协议解析器的连接请求并建立控制连接。它从用户 PI接收标准的 FTP 命令，发送响应，并管理服务器 DTP。

**服务器DTP**（server-DTP）：数据传输过程，在通常的“主动”状态下是用“监听”的数据端口建立数据连接。它建立传输和存储参数，并在服务器端 PI 的命令下传输数据。服务器端 DTP 也可以用于“被动”模式，而不是主动在数据端口建立连接。

**用户DTP**（user-DTP）：数据传输过程在数据端口“监听”服务器 FTP 过程的连接。

**控制连接**：用户PI 与服务器PI 用来交换命令和响应的信息传输通道。

**数据连接**：通过控制连接协商的模式和类型进行数据传输。
## 1.2 基本原理

  FTP（File Transfer Protocol）协议，文件传输协议。提供交互式的访问，对传输文件的格式和类型有分类，允许文件具有存取权限。屏蔽了各计算机系统的细节，适合异构网络任意计算机的传送。FTP有以下基本功能：

1. 提供不同种类主机系统（硬，软件体系等都可以不同）之间的文件传输能力。
2. 以用户权限管理的方式提供用户对远程FTP服务器上的文件管理能力。
3. 以匿名FTP的方式提供公用文件共享的能力。

FTP采用C/S架构，使用TCP可靠的传输服务。一个FTP服务器进程可同时为多个客户进程提供服务，FTP服务器检查有两大部分组成：一个主进程，负责接收新的请求，另外有若干的从属进程，负责处理单个请求，工作步骤如下：

1. 打开熟知端口21（控制端口），使客户进程能够连接上，
2. 等待客户进程发链接请求。
3. 启动从属进程来处理客户进程发来的请求。主进程与从进程并发执行，从属进程对客户进程的请求处理完毕后即终止。
4. 回到等待状态，继续接收其他客户进程的请求。

![图1](https://www.github.com/liao20081228/blog/raw/master/图片/ftp协议及Linux下搭建ftp服务器/图1.png)

首先由客户进程用随机非特权端口N向服务器端口21发送连接请求，控制连接就建立了。FTP控制连接在整个会话期间都保持打开，只用来发送连接/传送控制命令或请求。如果需要传输数据，则需要建立数据连接，建立方式分为主动模式和被动模式。主动模式是服务器端主动用20端口与客户端的N+1端口建立连接，而被动模式则由客户端以N+1端口主动连接到服务端随机非特权端口M。FTP使用了2个不同的端口号，所以数据连接和控制连接不会混乱。



## 1.2 数据表示
FTP协议规定了控制协议传送与存储的多种选择，在以下4几个方面必须做出一个选择。

* 文件类型
	* ASCII码文件(默认的)
	* 图像文件类型(二进制的)
	* 本地文件类型(用于在具有不同字节大小主机间传送二进制数据)
* 格式控制：
	* 该选项针对ASCII类型文件适用，非打印(默认选择，文件中不包含垂直格式信息)
	* 远程登录格式控制
* 传输方式：
	* 流方式(模式选择，文件以字节流方式传输，对于文件结构，发方在文件尾提示关闭数据连接，对于记录结构，有专用的两字节序列码记录结束和文件结束)
		* 文件结构(默认选择，文件被认为是一个连续的字节流，不存在内部的文件结构)
		* 记录结构(用于文本文件)
	* 块方式(文件以一系列块来传送，每块前面有一个或多个首部字节，头字节包括16位计数域和8位描述子代码)
	* 压缩方式：压缩模式中，因为数据是压缩过的，对于增加带宽有很多好处。

## 1.3 支持模式
FTP支持两种模式：Standard (PORT方式，主动方式)，Passive (PASV，被动方式)。

* Port模式：FTP 客户端首先用一个随机非特权端口N和服务器的TCP 21端口建立连接，用来发送命令（一般是一些简单的控制命令，如rm、ls等），客户端需要接收数据的时候在这个通道上发送PORT命令。PORT命令包含了客户端用什么端口接收数据（一般为N+1）。在传送数据的时候，服务器端通过自己的TCP 20端口连接至客户端的指定端口（N+1）发送数据。FTP server必须和客户端建立一个新的连接用来传送数据。
	* 工作过程
		* 客户端以随机非特权端口N，就是大于1024的端口，对server端21端口发起连接
		* 客户端开始监听 N+1端口；
		* 服务端会主动以20端口连接到客户端的N+1端口。
	* 优点：服务端配置简单，利于服务器安全管理，服务器只需要开放21端口
	* 缺点：如果客户端开启了防火墙，或客户端处于内网（NAT网关之后）， 那么服务器对客户端端口发起的连接可能会失败
* Passive模式：建立控制通道和PORT模式类似，但建立连接后发送Pasv命令。服务器收到Pasv命令后，打开一个临时端口（M，端口号大于1023小于65535）并且通知客户端在这个端口上传送数据，客户端打开一个随机非特权端口（一般为N+1）连接FTP服务器端口M，然后FTP服务器将通过端口M传送数据。
	* 工作过程
		* 客户端以随机非特权端口连接服务端的21端口
		* 服务端开启一个非特权端口为被动端口，并返回给客户端
		* 客户端以非特权端口+1的端口主动连接服务端的被动端口
	* 缺点：服务器配置管理稍显复杂，不利于安全，服务器需要开放随机高位端口以便客户端可以连接，因此大多数FTP服务软件都可以手动配置被动端口的范围
	* 优点：对客户端网络环境没有要求

很多防火墙在设置的时候都是不允许接受外部发起的连接的，所以许多位于防火墙后或内网的FTP服务器不支持PASV模式，因为客户端无法穿过防火墙打开FTP服务器的高端端口；而许多内网的客户端不能用PORT模式登陆FTP服务器，因为从服务器的TCP 20无法和内部网络的客户端建立一个新的连接，造成无法工作。
## 1.4 命令与响应

FTP命令主要用于控制连接，可以直接采用Telnet协议实现，所以FTP命令同Telnet命令，包括中断进程、Telnet的同步信号、查询服务器、带选项的Telnet命令等。命令以NVT ASCII[^1]码形式传送，要求在每行结尾都要CR、LF对。
![tu2](https://www.github.com/liao20081228/blog/raw/master/图片/ftp协议及Linux下搭建ftp服务器/图2.png)
![tu3](https://www.github.com/liao20081228/blog/raw/master/图片/ftp协议及Linux下搭建ftp服务器/图3.png)
 
 FTP响应都是ASCII码形式的3位数字，响应也是以NVT ASCII码形式传送，要求在每行结尾都要返回CR、LF对（也就是\r\n）。响应码用三位数字编码表示：

 ![tu4](https://www.github.com/liao20081228/blog/raw/master/图片/ftp协议及Linux下搭建ftp服务器/图4.png)
 
 <style>table{word-break:initial;}</style>

>网络虚拟终端ASCII码（NVT ASCII）
>NVT是一种通用的字符终端，叫网络虚拟终端。客户和服务器用它来建立数据表示和解释的一致性。而NVT ASCII码则是用来替代ASCII码在客户和服务器间传输的编码形式。
>NVT ASCII码的构成：
>NVT ASCII代表7位的ASCII字符集，网间协议族都使用NVT ASCII ,每个7位的字符都以8位格式发送，最高位为0。行结束符以两个字符CR（回车）和紧接着的LF（换行）这样的序列表示，以\r\n表示。单独的一个CR也是以两个字符序列来表示，它们是CR和紧接着的NUL（字节0），表示为\r\0。


# 2 搭建ftp服务器
## 2.1 用户分类

在组建FTP服务器的时候，我们就需要根据用户的类型，对用户进行归类。默认情况下，Vsftpd服务器会把建立的所有帐户都归属为Real用户。但是，这往往不符合企业安全的需要。因为这类用户不仅可以访问自己的主目录，而且，还可以访问其他用户的目录。这就给其他用户所在的空间带来一定的安全隐患。所以，企业要根据实际情况，修改用户所在的类别。

* Real（系统）帐户：也叫本地账户，就是真实的Linux用户，用户/etc/passwd,密码/etc/shadow。这类用户是指在FTP服务器上拥有帐号。当这类用户登录FTP服务器的时候，其默认的主目录就是其号命名的目录。但是，其还可以变更到其他目录中去。如系统的主目录等等。

* Guest（虚拟）用户：特定服务的专用用户，可以有两种方式：独立的用户名/密码文件或者用户数据库。在FTP服务器中，我们往往会给不同的部门或者某个特定的用户设置一个帐户。但是，这个账户有个特点，就是其只能够访问自己的主目录。服务器通过这种方式来保障FTP服务上其他文件的安全性。这类帐户，在Vsftpd软件中就叫做Guest用户。拥有这类用户的帐户，只能够访问其主目录下的目录，而不得访问主目录以外的文件。

* Anonymous（匿名）用户：这也是我们通常所说的匿名访问。这类用户是指在FTP服务器中没有指定帐户，但是其仍然可以进行匿名访问某些公开的资源。登录时使用默认的用户名，一般是ftp或anonymous。
 

## 2.2 安装ftp服务器
1. 更新源 `sudo apt update`

2. 查看是否安装 ftp服务器`vsftpd -v`

3. 安装ftp服务器`sudo apt install vsftpd`，如果安装失败或者配置出现问题，可以卸载 ftp服务器`sudo apt purge vsftpd`
4. 

## 2.3 vsfpd命令详解
**命令**：vsftpd [configuration file and / or options]
**描述**:  vsftpd是“非常安全的文件传输协议”守护程序。可以通过“超级服务器”（如inetd（8）或xinetd（8））启动服务器。另外，vsftpd可以以独立模式启动，在这种情况下，vsftpd本身将在网络上侦听。后一种模式更易于使用，也建议使用。通过在`/etc/vsftpd.conf`中设置listen = YES可以激活它。直接执行vsftpd二进制文件将启动FTP服务，以准备立即进行客户端连接。

**选项**：命令行上可以提供一个或多个可选配置文件。如果以root用户身份运行，则这些文件必须以root用户身份拥有。任何不以“-”字符开头的命令行选项都将被视为将要加载的配置文件。请注意，配置文件的加载严格按照在命令行上遇到的顺序进行。如果未指定配置文件，则在处理所有其他命令行选项后，将加载/etc/vsftpd.conf的默认配置文件。

|短选项|长选项|描述|
|:--|:--|:--|
|-v||打印版本信息并退出，即使遇到其他选项也是如此。
||-ooption=value|根据配置文件中的格式设置选项-值对。支持多个-o选项，严格按照它们在命令行中的出现顺序应用，包括与配置文件的加载混合在一起。

**示例**：vsftpd -olisten=NO /etc/vsftpd.conf -oftpd_banner=blah。 该示例覆盖了vsftpd的“ listen”选项的内置默认值NO，但随后加载了/etc/vsftpd.conf，它可能会覆盖该设置。 最后，“ftpd_banner”设置为“ blah”，它将覆盖所有默认的vsftpd设置以及配置文件中的所有相同设置。

## 2.4 配置ftp服务器

5 创建共享文件夹`sudo mkdir ~/ftp;sudo chmod 777 ~/ftp`
6 修改 vsftpd.conf 配置文件 `sudo vim /etc/vsftpd.conf`
```
# 阻止 vsftpd 在独立模式下运行
listen=NO                 
# vsftpd 将监听 ipv6 而不是 IPv4
listen_ipv6=YES           
# 关闭匿名登录
anonymous_enable=NO       
# 允许本地用户登录
local_enable=YES          
# 启用可以修改文件的 FTP 命令
write_enable=YES          
# 本地用户新增档案时的umask 值
local_umask=022           
# 当用户第一次进入新目录时显示提示消息 
dirmessage_enable=YES     
# 显示在您的本地时区的时间目录列表
use_localtime=YES         
# 一个存有详细的上传和下载信息的日志文件
xferlog_enable=YES        
# 在服务器上针对 PORT 类型的连接使用端口 20（FTP 数据）
connect_from_port_20=YES  
# 不单独建立ftp用户，直接使用Ubuntu桌面用户就可以登陆
chroot_local_user=YES
chroot_list_enable=NO
# 能够登录的用户名单
chroot_list_file=/etc/vsftpd.chroot_list   
# 使用uft8文件系统
utf8_filesystem=YES

# 锁定一个共享目录
local_root=/home/eagle/ftp
# 给共享目录添加写权限
allow_writeable_chroot=YES
```
7 创建 vsftpd.chroot_list 文件 `sudo vim /etc/vsftpd.chroot_list`，添加能够登录ftp服务器的用户名（就是主机的用户），一行一个

8 重启 ftp 服务器 `sudo service vsftpd restart`

9 浏览器登录ftp服务器`ftp://192.168.1.107` , 输入用户名和密码，登录成功，可以看到共享的文件


>详细配置项目请参考 `man 5 vsftpd.conf`

## 2.5 vsftpd.conf详解
**描述**：vsftpd.conf可用于控制vsftpd行为的各个方面。 默认情况下，vsftpd在`/etc/vsftpd.conf`中查找此文件。 但是，您可以通过为vsftpd指定命令行参数来覆盖它。 命令行参数是vsftpd的配置文件的路径名。 此行为很有用，因为您可能希望使用xinetd之类的高级inetd在每个虚拟主机上使用不同的配置文件启动vsftpd。
**格式**：
* vsftpd.conf的格式非常简单。 每行都是注释或指令。 注释行以＃开头，将被忽略。 指令行的格式为：**选项=值**
* 注意，在选项，等号和值之间放置任何空格是错误的。
* 每个设置都有默认的已编译的值，可以在配置文件中进行修改。

**布尔选项**：这些选项的值是YES或NO

|选项|默认值|描述|
|:--|:--|:--|
|allow_anon_ssl|NO|仅在ssl_enable处于激活状态时适用。如果设置为“是”，将允许匿名用户使用安全的SSL连接。|
|anon_mkdir_write_enable|NO|如果设置为YES，将允许匿名用户在某些条件下创建新目录。为此，必须激活选项write_enable，并且匿名ftp用户必须在父目录上具有写权限。
|anon_other_write_enable|NO|如果设置为YES，则将允许匿名用户执行除上传和创建目录之外的写操作，例如删除和重命名。通常不建议这样做，但出于完整性考虑，将其包括在内。|
|anon_upload_enable|NO|如果设置为YES，将允许匿名用户在某些条件下上传文件。为此，必须激活选项write_enable，并且匿名ftp用户必须对所需的上传位置具有写权限。虚拟用户也需要此设置。默认情况下，虚拟用户将获得匿名（即最大限制）特权。
|anon_world_readable_only|YES|启用后，将仅允许匿名用户下载其可读的文件。这是承认ftp用户可能拥有文件，尤其是在存在上传文件的情况下。|
|anonymous_enable|NO|控制是否允许匿名登录。 如果启用，则将具名用户和匿名用户都识别为匿名登录。
|ascii_download_enable|NO|启用后，下载时接受ASCII模式的数据传输。
|ascii_upload_enable|NO|启用后，上传时接受ASCII模式的数据传输。
|async_abor_enable|NO|启用后，将启用称为“async ABOR”的特殊FTP命令。 只有不建议使用的FTP客户端会使用此功能。 此外，此功能很难处理，因此默认情况下处于禁用状态。 不幸的是，除非有此功能，否则某些FTP客户端在取消传输时将挂起，因此您可能希望启用它。|
|background|NO|启用后，vsftpd以“listen”模式启动，vsftpd将后台运行监听程序。即立即返回到启动vsftpd的shell中。
|check_shell|YES|注意！此选项仅对vsftpd的非PAM版本有效。如果禁用，则vsftpd将不会在/etc/shells中检查本地登录的合法用户shell。
|chmod_enable|YES|启用后，允许使用SITE CHMOD命令。注意！这仅适用于本地用户。匿名用户永远不能使用SITE CHMOD。|
|chown_uploads|NO|如果启用，所有匿名上传的文件的所有权都将更改为chown_username中指定的用户。从管理（也许是安全性）的角度来看，这很有用。|
|chroot_list_enable|NO|如果设为YES，则可以提供本地用户列表，这些用户在登录时将被限制置在家目录的chroot()中(chroot一种技术，可以组织)。如果chroot_local_user设置为YES，则含义略有不同，这些用户在登录时将不会被限制chroot()中。默认情况下，此列表的文件是/etc/vsftpd.chroot_list，但是您可以使用chroot_list_file设来覆盖它。
|chroot_local_user|NO|如果设置为YES，则本地用户（默认情况下）将在登录后（默认情况下）将被限制置在chroot。警告：此选项具有安全隐患，尤其是在用户具有上传权限或shell程序访问权限的情况下。仅当您知道自己在做什么时才启用。请注意，这些安全隐患不是特定于vsftpd的。它们适用于所有将本地用户限制在主目录下的FTP守护程序。|
|connect_from_port_20|NO|这控制PORT样式的数据连接是否使用服务器计算机上的端口20（ftp-data）。 出于安全原因，某些客户可能会坚持认为是这种情况。 相反，禁用此选项将使vsftpd可以以较少的特权运行。|
|debug_ssl|NO |如果为true，则将OpenSSL连接诊断信息转储到vsftpd日志文件中。 （在v2.0.6中添加）。|
|delete_failed_uploads|NO|如果为true，则删除任何失败的上传文件。 （在v2.0.7中添加）。|
|deny_email_enable|NO|如果被激活，匿名电子邮件密码在该列表中的匿名用户会被拒绝登录（该列表中的电子邮件不能作为匿名登录密码）。 默认情况下，包含此列表的文件是/etc/vsftpd.banned_emails，但是您可以使用banned_email_file设置覆盖它。|
|dirlist_enable|YES|如果设置为NO，则所有目录列出命令（如ls）都将被拒绝。|
|dirmessage_enable|NO|如果启用，则在首次进入新目录时，将向FTP服务器的用户显示消息。默认情况下，将在目录中扫描以寻找文件.message，但是可以使用message_file设置覆盖它。|
|download_enable|YES| 如果设置为NO，则所有下载请求都将被拒绝。|
|dual_log_enable|NO| 如果启用，则将并行生成两个日志文件，默认为/var/log/xferlog和/var/log/vsftpd.log。前者是wu-ftpd样式的传输日志，可通过标准工具进行解析。后者是vsftpd自己的样式日志。
|force_dot_files|NO|如果激活，则以.开头文件和目录也会被显示在目录列表中，即使客户端未使用“ a”标志。此替代排除“.”和“..”条目。|
|force_anon_data_ssl|NO|仅在激活ssl_enable时适用。如果激活，则所有匿名登录都将被迫使用安全的SSL连接，以便在数据连接上发送和接收数据。|
|force_anon_logins_ssl|NO|仅在激活ssl_enable时适用。如果激活，则所有匿名登录都将被迫使用安全的SSL连接以便发送密码。|
|force_local_data_ssl|YES|仅在激活ssl_enable时适用。如果激活，则所有非匿名登录都将被迫使用安全的SSL连接，以便通过数据连接发送和接收数据。
|force_local_logins_ssl|YES|仅在激活ssl_enable时适用。如果激活，则所有非匿名登录都将被迫使用安全的SSL连接以便发送密码。|
|guest_enable|NO|如果启用，则所有非匿名登录均被分类为“访客”登录。 来宾登录名将重新映射到guest_username设置中指定的用户。|
|hide_ids|NO|如果启用，目录列表中的所有用户和组信息将显示为“ ftp”。|
|implicit_ssl|NO|如果启用，则SSL握手是所有连接（FTPS协议）上的第一件事。为了也支持显式SSL或纯文本，应运行单独的vsftpd侦听器进程。|
|listen|NO| 如果启用，则vsftpd将以独立模式运行。这意味着绝对不能从某种inetd运行vsftpd。相反，vsftpd可执行文件将直接运行一次。 vsftpd本身将负责侦听和处理传入的连接。|
|listen_ipv6|NO|就像listen参数一样，不同的是，vsftpd会在IPv6套接字上监听而不是在IPv4套接字上监听。请注意，默认情况下，侦听IPv6“任意”地址（::)的套接字将同时接受IPv6和IPv4连接。此参数和listen参数是互斥的。|
|local_enable|NO|控制是否允许本地登录。如果启用，则可以使用/etc/passwd（或您的PAM配置引用所在的任何地方）中的普通用户帐户登录。必须启用此选项才能使任何非匿名登录（包括虚拟用户）正常工作。|
|lock_upload_files|YES|启用后，所有上传将对上传文件进行写锁定。所有下载将对下载文件进行共享的读锁定。警告！在启用此功能之前，请注意，恶意读者可能会饿死想要这样做的写者。例如，附加文件。
|log_ftp_protocol|NO|启用后，如果未启用选项xferlog_std_format，则记录所有FTP请求和响应。对于调试很有用。
|ls_recurse_enable|NO|启用后，此设置将允许使用“ ls -R”。这是一个较小的安全风险，因为在大型站点的顶层使用ls -R可能会消耗大量资源。|
|mdtm_write|YES| 启用后，此设置将允许MDTM设置文件修改时间（取决于通常的访问检查）。|
|no_anon_password|NO|启用后，这将阻止vsftpd询问匿名密码，匿名用户将直接登录。|
|no_log_lock|NO|启用后，这可以防止vsftpd在写入日志文件时获取文件锁定。通常不应启用此选项。它可以解决操作系统错误，例如Solaris / Veritas文件系统组合（已被发现有时锁定日志文件时会挂起）。|
|one_process_model|NO|如果您具有Linux 2.4内核，则可以使用一个不同的安全模型，该模型每个连接仅使用一个进程。 它不是一个纯粹的安全模型，但是可以提高性能。 您真的不会想启用此功能除非您知道自己在做什么，并且您的站点支持大量同时连接的用户。|
|passwd_chroot_enable|NO| 如果启用了chroot_local_user，则可以基于每个用户指定主目录。每个用户的主目录都是从/etc/passwd中的主目录字符串派生的。主目录字符串中/和./的出现表示该主目录位于路径中的特定位置。|
|pasv_addr_resolve|NO|如果要在pasv_address选项中使用主机名（而不是IP地址），则设置为YES。
|pasv_enable|YES|如果要禁止使用PASV方法获取数据连接，请设置为NO|
|pasv_promiscuous|NO| 设置为“YES”禁用PASV安全检查，这个安全检查以确保数据连接源自与控制连接相同的IP地址。仅当您知道自己在做什么时才启用！唯一合法的用途是以某种形式的安全隧道方案，或者可能是为了促进FXP支持。|
|port_enable|YES|如果要禁止使用PORT方法获取数据连接，请设置为NO。|
|port_promiscuous|NO|设置为YES禁用PORT安全检查，这个安全检查确保传出数据连接只能连接到客户端。仅当您知道自己在做什么时才启用！|
|require_cert|NO|如果设置为yes，则需要所有SSL客户端连接提供客户端证书。此证书的合法程度由validate_cert（在v2.0.6中添加）控制。
|require_ssl_reuse|YES|如果设置为yes，则所有SSL数据连接都必须具有SSL会话重用性（这证明它们知道与控制通道相同的主密钥）。尽管这是一个安全的默认设置，但它可能会破坏许多FTP客户端，因此您可能要禁用它。有关后果的讨论，请参见http://scarybeastsecurity.blogspot.com/2009/02/vsftpd-210released.html（在v2.1.0中添加）。|
|run_as_launching_user|NO|如果要让vsftpd以启动vsftpd的用户身份运行，请设置为YES。 在没有根访问权限的情况下，这很有用。 严重警告！ 除非您完全知道自己在做什么，否则不要启用此选项，因为天真地使用此选项可能会造成严重的安全问题。 特别是，设置了此选项后，vsftpd不会也不能使用chroot技术来限制文件访问（即使由root启动）。 较差的替代方法可能是使用诸如{/\*,\*..\*}之类的deny_file设置，但是这种方法的可靠性无法与chroot相提并论，因此不应依赖。 如果使用此选项，则对其他选项有许多限制。 例如，要求特权的选项（例如非匿名登录，更改上传所有权，从端口20连接以及监听小于1024的端口）不起作用。 其他选项可能会受到影响。|
|secure_email_list_enable|NO|如果只希望允许匿名登录密码在列表中的用户登录，则设置为YES。这是一种在不需要虚拟用户的情况下限制对低安全性内容的访问的简便方法。启用后，除非在email_password_file设置指定的文件中列出了提供的密码，否则将阻止匿名登录。文件格式是每行一个密码，没有多余的空格。默认文件名是/etc/vsftpd.email_passwords。|
|session_support|NO|这可以控制vsftpd是否尝试维护登录会话。如果vsftpd正在维护会话，它将尝试更新utmp和wtmp。如果使用PAM进行身份验证，它还将打开pam_session，并且仅在注销时关闭它。如果不需要会话日志记录，则可能希望禁用此功能，并且希望给vsftpd更多的机会以更少的进程和/或更少的特权运行。注意-utmp和wtmp支持仅在启用PAM的版本中提供。|
|setproctitle_enable|NO|如果启用，vsftpd将尝试在系统进程列表中显示会话状态信息。换句话说，报告的进程名称将更改以反映vsftpd会话正在执行的操作（空闲，下载等）。为了安全起见，您可能希望将此设置保留下来。|
|ssl_enable|NO|如果启用，并且vsftpd是针对OpenSSL编译的，则vsftpd将支持通过SSL的安全连接。这适用于控制连接（包括登录）以及数据连接。您还需要具有SSL支持的客户端。注意！！当心启用此选项。仅在需要时启用它。 vsftpd无法保证OpenSSL库的安全性。通过启用此选项，您声明您相信已安装的OpenSSL库的安全性。
|ssl_request_cert|YES|如果启用，vsftpd将在传入的SSL连接上请求（但不一定需要；请参阅require_cert）一个证书。通常这根本不会造成任何麻烦，但是IBM zOS似乎有问题。 （v2.0.7中的新功能）。|
 |ssl_sslv2|NO| 仅在激活ssl_enable时适用。如果启用，此选项将允许SSL v2协议连接。首选TLS v1连接。|
 |ssl_sslv3|NO|仅在激活ssl_enable时适用。如果启用，此选项将允许SSL v3协议连接。首选TLS v1连接。|
 |ssl_tlsv1|YES|仅在激活ssl_enable时适用。如果启用，此选项将允许TLS v1协议连接。首选TLS v1连接。|
 |strict_ssl_read_eof|NO|如果启用，则需要通过SSL（而不是套接字上的EOF）终止SSL数据上传。 必须使用此选项，以确保攻击者不会使用伪造的TCP FIN提前终止上传。 不幸的是，默认情况下未启用它，因为很少有客户端正确使用它。 （v2.0.7中的新功能）。|
|strict_ssl_write_shutdown|NO|如果启用，则需要通过SSL（而不是套接字上的EOF）终止SSL数据下载。 默认情况下此功能处于关闭状态，因为我无法找到执行此操作的单个FTP客户端。 这是次要的。 它所影响的只是我们判断客户是否已确认完整接收文件的能力。 即使没有此选项，客户端也可以检查下载的完整性。 （v2.0.7中的新功能）。
|syslog_enable|NO|如果启用，则将发送到/var/log/vsftpd.log的所有日志输出将转至系统日志。日志是在FTPD工具下完成的。
|tcp_wrappers|NO|如果启用，并且vsftpd编译时支持tcp_wrappers，则传入的连接将通过tcp_wrappers访问控制进行馈送。此外，还有一种用于基于IP的配置的机制。如果tcp_wrappers设置了VSFTPD_LOAD_CONF环境变量，则vsftpd会话将尝试加载此变量中指定的vsftpd配置文件。|
|text_userdb_names|NO|默认情况下，数字ID显示在目录列表的用户和组字段中。您可以通过启用此参数来获取文本名称。由于性能原因，默认情况下处于禁用状态。|
|tilde_user_enable|NO|如果启用，vsftpd将尝试解析诸如〜chris/pics之类的路径名，即波浪号后跟用户名。请注意，vsftpd将始终解析路径名〜和〜/something（在这种情况下，〜解析为初始登录目录）。请注意，只有在当前主目录下可以找到文件/etc/passwd时，〜user路径才会解析。
|use_localtime|NO|如果启用，vsftpd显示目录列表将显示当地时区中的时间。默认为显示GMT。 MDTM FTP命令返回的时间也受此选项的影响。|
|use_sendfile|YES|一个内部设置，用于测试在平台上使用sendfile（）系统调用的相对优势。
|userlist_deny|YES|如果userlist_enable已激活，则检查此选项。如果将此设置设置为NO，则用户将被拒绝登录，除非用户在userlist_file指定的文件中明确列出。如果拒绝登录，则会在要求用户输入密码之前发出拒绝信息。|
|userlist_enable|NO|如果启用，则vsftpd将从userlist_file给出的文件名中加载用户名列表。如果用户尝试使用此文件中的名称登录，则会在要求其输入密码之前被拒绝。这对于防止传输明文密码可能很有用。另请参阅userlist_deny。
|validate_cert|NO|如果设置为“YES”，则收到的所有SSL客户端证书都必须验证“OK”。自签名证书不构成OK验证。 （v2.0.6中的新功能）。
|virtual_use_local_privs|NO|如果启用，虚拟用户将使用与本地用户相同的特权。默认情况下，虚拟用户将使用与匿名用户相同的特权，这通常会受到更严格的限制（尤其是在写访问权限方面）。
 |write_enable|NO|这控制是否允许任何更改文件系统的FTP命令。 这些命令是：STOR，DELE，RNFR，RNTO，MKD，RMD，APPE和SITE。
 |xferlog_enable|NO|如果启用，将维护一个日志文件，详细说明了上传和下载。 默认情况下，此文件将放置在/var/log/vsftpd.log中，但是可以使用配置设置vsftpd_log_file覆盖此位置。
 |xferlog_std_format|NO|如果启用，则传输日志文件将以wu-ftpd使用的标准xferlog格式写入。 这很有用，因为您可以重用现有的传输统计信息生成器。 但是，默认格式更具可读性。 这种样式的日志文件的默认位置是/var/log/xferlog，但是您可以使用xferlog_file设置对其进行更改。
 
 **数值选项**：以下是数字选项列表。 数字选项必须设置为非负整数。 支持八进制数字，以方便使用umask选项。 要指定一个八进制数字，请使用0作为数字的第一位。
 
|选项|默认值|描述|
|:--|:--|:--|
|accept_timeout|60|远程客户端与PASV样式数据连接建立连接的超时时间（以秒为单位）。==|==
|anon_max_rate|0|匿名客户端允许的最大数据传输速率，以每秒字节数为单位。0表示不限制|
|anon_umask|077|匿名用户的用于文件创建的umask值。注意！如果要指定八进制值，请记住前缀“ 0”，否则该值将被视为以10为底的整数！|
|chown_upload_mode|0600|匿名上传时，使用chown强制改变的的文件模式。 （在v2.0.6中添加）。
|connect_timeout|60|远程客户端响应我们的PORT样式数据连接的超时时间（以秒为单位）。
|data_connection_timeout|300|我们允许数据传输暂停而不进行任何操作的最长时间（以秒为单位）。如果触发超时，则断开远程客户端。|
|delay_failed_login|1|报告失败的登录之前暂停的秒数。
|delay_successful_login|0|报告登录成功之前暂停的秒数。
|file_open_mode|0666|创建上传文件的权限。 Umask应用于此值的顶部。如果希望上载的文件可执行，则可能希望更改为0777。|
|ftp_data_port|20| PORT样式连接所源自的端口（只要启用了connect_from_port_20）|
|idle_session_timeout|300|远程客户端在FTP命令之间花费的最长时间，以秒为单位。如果触发超时，则关闭远程客户端。|
|listen_port|21|如果vsftpd处于独立模式，则该端口将侦听传入的FTP连接。|
|local_max_rate|0|本地认证用户允许的最大数据传输速率，以每秒字节数为单位。0表示不限制|
|local_umask|077|用于本地用户的文件创建umask值。注意！如果要指定八进制值，请记住前缀“ 0”，否则该值将被视为以10为底的整数|
|max_clients|0|如果vsftpd处于独立模式，则这是可以连接的最大客户端数。额外连接的所有其他客户端将收到错误消息。0表示无限制|
|max_login_fails|3|在多次登录失败后，会话被终止。|
|max_per_ip|0|如果vsftpd处于独立模式，则这是可以从同一源Internet地址连接的最大客户端数。如果客户端超过此限制，则会收到错误消息。0表示不限制|
|pasv_max_port|0|分配给PASV样式数据连接的最大端口。 可用于指定狭窄的端口范围以辅助防火墙。0表示任意端口|
|pasv_min_port|0|分配给PASV样式数据连接的最小端口。 可用于指定狭窄的端口范围以辅助防火墙。0表示任意端口|
|trans_chunk_size|0|您可能不想更改此设置，但请尝试将其设置为8192之类的值，以使带宽限制器更加平滑。 "0"表示让 vsftpd 自己选择一个合理的值。|

**字符串选项**：下面是字符串选项列表

|选项|默认值|描述|
|:--|:--|:--|
|anon_root|空|此选项代表匿名登录后vsftpd将尝试切换到的目录。失败被默默忽略。|
|banned_email_file|/etc/vsftpd.banned_emails|此选项是包含不允许的匿名电子邮件密码列表的文件的名称。如果启用了 deny_email_enable选项则查询该文件。|
|banner_file|空|此选项是文件的名称，该文件包含当有人连接到服务器时要显示的文本。如果设置，它将覆盖ftpd_banner选项提供的标志字符串。|
|ca_certs_file|空|此选项是要加载数字证书的文件名，目的是验证客户端证书。还将将加载的证书发布给客户端，以适应TLSv1.0客户端（例如z / OS FTP客户端）。遗憾的是，由于vsftpd使用了受限制的文件系统空间（chroot），因此未使用默认的SSL CA证书路径。 （在v2.0.6中添加）。
|chown_username|root|这是匿名上载文件的所有者的用户名。仅当设置了另一个选项chown_uploads时，此选项才相关。|
|chroot_list_file|/etc/vsftpd.chroot_list|该选项是包含本地用户列表的文件名，该列表的用户被限制只能在其chroot活动。仅当启用选项chroot_list_enable时，此选项才有效。如果启用了chroot_local_user选项，则列表中的用户将不会被限制|
|cmds_allowed|空|此选项指定允许的FTP命令列表，以逗号分隔（登录后。登录前始终允许USER，PASS和QUIT以及其他命令）。 其他命令被拒绝。 这是真正锁定FTP服务器的强大方法。 示例：cmds_allowed = PASV，RETR，QUIT|
|cmds_denied|空|此选项指定拒绝的FTP命令列表，以逗号分隔（登录后。登录前始终允许USER，PASS，QUIT以及其他命令）。 如果在此命令和cmds_allowed上均显示命令，则拒绝优先。 （在v2.1.0中添加）。
|deny_file|空|此选项可用于设置不应以任何方式访问的文件名（和目录名等）的匹配模式。受影响的项目不会被隐藏，但是对它们执行任何操作（下载，更改目录，影响目录中的内容等）的任何尝试都将被拒绝。此选项非常简单，不应用于严格的访问控制，而应优先使用文件系统的权限。但是，此选项在某些虚拟用户设置中可能有用。特别要注意的是，如果文件名可以通过各种名称访问（可能是由于符号链接或硬链接），则必须注意拒绝所有名称的访问。如果项目名称包含hide_file中给定的字符串，或者它们与hide_file指定的正则表达式匹配，则将拒绝对项目的访问。请注意，vsftpd的正则表达式匹配代码是一个简单的实现，是完整正则表达式功能的子集。因此，您将需要仔细详尽地测试此选项的任何应用。建议您对任何重要的安全策略使用文件系统权限，因为它们具有更高的可靠性。支持的正则表达式语法为\* 、?和未嵌套的{，}运算符。正则表达式匹配仅在路径的最后一个组件上受支持，例如a/b/？支持，但不支持a/?/c。例如：deny_file ={\*.mp3,\*.mov,.private}|
|download_file|空|可以设置此选项以将下载限制为名称与指定模式匹配的文件。如果文件名也匹配deny_file模式，则拒绝优先。有关用法和模式的详细信息，请参见deny_file选项。|
|dsa_cert_file|空|此选项指定用于SSL加密连接的DSA证书的位置。|
|dsa_private_key_file|空|此选项指定用于SSL加密连接的DSA私钥的位置。如果未设置此选项，则私钥应与证书位于同一文件中。|
|email_password_file|/etc/vsftpd.email_passwords|此选项可用于提供备用文件，以供secure_email_list_enable设置使用。|
|ftp_username|ftp|这是我们用于处理匿名FTP的用户名。该用户的主目录是匿名FTP区域的根。|
|ftpd_banner|空|此字符串选项使您可以覆盖首次建立连接时vsftpd显示的问候语。
|guest_username|ftp|有关构成访客登录的说明，请参见布尔设置guest_enable。此设置是访客用户映射到的真实用户名。|
|hide_file|空|此选项可用于设置目录列表中要隐藏的文件名（和目录名等）的匹配模式，该模式应从目录列表中隐藏。 尽管文件 /目录等被隐藏了，但知道实际使用什么名称的客户端仍可以完全访问它们。 如果项目名称包含hide_file给定的字符串，或者它们与hide_file指定的正则表达式匹配，则这些项目将被隐藏。 请注意，vsftpd的正则表达式匹配代码是一个简单的实现，是完整正则表达式功能的子集。 有关确切支持的正则表达式语法的详细信息，请参见deny_file。|
|listen_address|空|如果vsftpd处于独立模式，则此设置可能会覆盖（所有本地接口的）默认侦听地址。 提供一个数字IP地址。|
|listen_address6|空|与listen_address类似，但是为IPv6侦听器指定默认侦听地址（如果设置了listen_ipv6，则使用该地址）。格式是标准的IPv6地址格式。|
|local_root|空|此选项代表在本地（即非匿名）登录后vsftpd将尝试切换到的目录。失败被默认忽略。|
|message_file|.message|此选项是进入新目录时要查找的文件的名称。内容显示给远程用户。仅当启用了dirmessage_enable选项时，此选项才有效。|
|nopriv_user|nobody|这是vsftpd想要完全没有特权时使用的用户名。请注意，这应该是一个专用用户，而不是没有用户。在大多数机器上，nobody被用来做很多重要的事情。|
|pam_service_name|vsftpd|该字符串是vsftpd将使用的PAM服务的名称。|
|pasv_address|空|使用此选项可以覆盖vsftpd将响应PASV命令而通告的IP地址。请提供数字IP地址，除非启用了pasv_addr_resolve，在这种情况下，您可以提供一个主机名，该主机名将在启动时为您解析DNS。|
|rsa_cert_file|/usr/share/ssl/certs/vsftpd.pem| 此选项指定用于SSL加密连接的RSA证书的位置。
|rsa_private_key_file|空|此选项指定用于SSL加密连接的RSA私钥的位置。如果未设置此选项，则私钥应与证书位于同一文件中。|
|secure_chroot_dir|/var/run/vsftpd/empty|此选项应该是空目录的名称。 另外，该目录对ftp用户不应是可写的。 在vsftpd不需要文件系统访问的情况下，此目录用作安全的chroot()目录。|
|ssl_ciphers|DES-CBC3-SHA|此选项可用于选择哪些SSL密码被vsftpd允许用来加密SSL连接。 有关更多详细信息，请参见密码手册页。 请注意，严格的密码可能是有用的安全预防措施，因为它可以防止恶意远程方强制使用他们发现问题的密码。|
|upload_file|空|可以设置此选项以限制上传名称与指定模式匹配的文件。 如果文件名也匹配deny_file模式，则拒绝优先。 有关用法和模式的详细信息，请参见deny_file选项。|
|user_config_dir|空|这个功能强大的选项允许基于每个用户对手册页中指定的任何配置选项进行覆盖。用法很简单，最好用一个例子说明。如果将user_config_dir设置为/etc/vsftpd_user_conf，然后以“ chris”用户身份登录，则vsftpd将在会话期间应用文件/etc/vsftpd_user_conf/chris中的设置。该文件的格式在本手册页中有详细说明！请注意，并非所有设置对每个用户都有效。例如，许多设置仅在启动用户会话之前。不会影响每个用户行为的设置示例包括listen_address，banner_file，max_per_ip，max_clients，xferlog_file等。|
|user_sub_token|空|与虚拟用户一起使用时，此选项很有用。它用于基于模板为每个虚拟用户自动生成主目录。例如，如果通过guest_username指定的真实用户的主目录是/home/virtual/\$USER，并且user_sub_token设置为\$USER，则当虚拟用户fred登录时，他将最终（通常是chroot()）在/ home / virtual / fred目录中。如果local_root包含user_sub_token，则此选项也会生效。
|userlist_file|/etc/vsftpd.user_list|此选项是当userlist_enable选项处于激活状态时加载的文件的名称。|
|vsftpd_log_file|/var/log/vsftpd.log|此选项是我们向其中写入vsftpd样式日志文件的文件的名称。 仅当设置了选项xferlog_enable且未设置xferlog_std_format时或者设置了选项dual_log_enable才写入此日志。 进一步的复杂化-如果您已设置syslog_enable，则不会写入此文件，而是将输出发送到系统日志。|
|xferlog_file|/var/log/xferlog|此选项是我们写入wu-ftpd样式传输日志的文件的名称。 仅当设置了选项xferlog_enable以及xferlog_std_format或者设置了选项dual_log_enable时，才会写入传输日志。 |





 :


# 3 ftp客户程序
1 浏览器输入 ftp://ipaddress:port
2 使用专门的ftp-gui客户端如：X-ftp
3 使用ftp客户端命令行：ftp

## 3.1 ftp命令详解

**命令**：ftp [-46pinegvd] [host [port]]
&emsp;&emsp;&emsp;ftp [-46pinegvd] [host [port]]

**描述**：Ftp是Internet标准文件传输协议的用户界面。 该程序允许用户在远程网络站点之间传输文件。

**选项**：选项可以在命令行或者命令解释器中指定。

|选项|描述|
|:--|:--|
|-4|使用IPv4|
|-6|使用IPv6
|-p|使用被动模式进行数据传输。允许在防火墙阻止外界连接回客户端计算机的环境中使用ftp。要求ftp服务器支持PASV命令。如果作为pftp调用，则为默认设置。|
|-i  |在多个文件传输期间关闭交互式提示|
|-n |禁止ftp在初始连接时尝试“自动登录”。如果启用了自动登录，则ftp将检查用户主目录中的.netrc（请参阅netrc（5））文件是否存在，该文件描述了远程计算机上帐户的条目。如果不存在任何条目，则ftp将提示输入远程计算机的登录名（默认为本地计算机上的用户身份），并在必要时提示输入密码和用于登录的帐户。
|-e|如果-e已编译到ftp可执行文件中，则禁用命令编辑和历史记录支持。否则，什么都不做。|
|-g|禁用文件名锁定。|
| -v  |Verbose选项强制ftp显示来自远程服务器的所有响应以及报告数据传输统计信息
|-d|启用调试。|

**子命令**：可以在命令行上指定客户端主机和ftp与之通信的可选端口号。 如果这样做，ftp将立即尝试在该主机上建立与FTP服务器的连接。 否则，ftp将进入其命令解释器并等待用户的指示。 当ftp正在等待来自用户的命令时，会向用户提供提示“ ftp>”。 ftp可以识别以下命令：
|名令|描述|
|:--|:--|
|! [command [args]]|在本地计算机上调用交互式shell程序。如果有参数，则将第一个视为直接执行的命令，其余参数作为其参数。
|\$ macro-name [args]| 执行使用macdef命令定义的宏macro-name。参数被传递给宏unglobbed。
|account [passwd]| 成功完成登录后，请提供远程系统所需的补充密码，以访问资源。如果不包含任何参数，则将在非回显输入模式下提示用户输入帐户密码。|
|append local-file [remote-file]|将本地文件追加到远程计算机上的文件。如果未指定remote-file，则在通过任何ntrans或nmap设置对其进行更改之后，将使用本地文件名来命名远程文件。文件传输使用当前设置的类型，格式，模式和结构。
|ascii    |将文件传输类型设置为网络ASCII。这是默认类型。|
| bell  |安排在每个文件传输命令完成后响起铃声。|
| binary |设置文件传输类型以支持二进制图像传输。
|bye  |终止与远程服务器的FTP会话并退出ftp。文件结尾也会终止会话并退出。
|  case    |在mget命令期间开/关远程计算机文件名的大小写映射。 启用case（默认为关闭）时，远程计算机文件名写入本地目录时会把大写字母映射为小写字母。
|cd remote-directory|将远程计算机上的工作目录更改为remote-directory|
|cdup |将远程计算机工作目录更改为当前远程计算机工作目录的父目录|
|chmod mode file-name|将远程系统上的文件file-name的权限模式更改为mode。|
|close  |终止与远程服务器的FTP会话，然后返回命令解释器。任何定义的宏都将被擦除。|
|cr |在ASCII类型的文件取回期间，切换为回车分离。在ASCII类型的文件传输过程中，记录由回车/换行顺序表示。当cr启用时（缺省值），回车符将从该序列中剥离，以与UNIX单个换行记录分隔符一致。非UNIX远程系统上的记录可能包含单个换行符。当进行ascii类型的传输时，仅当cr为off时，才可以将这些换行符与记录分隔符区分开。|
|qc|切换ASCII类型命令输出中控制字符的打印方式。启用此选项后，如果输出文件是标准输出，则控制字符将替换为问号。当标准输出是tty时，这是默认值。|
|delete remote-file|删除远程计算机上的文件remote-file。|
|debug [debug-value]|开/关调试模式。如果指定了可选的调试值，则是用于设置调试级别。调试打开时，ftp打印发送到远程计算机的每个命令，并在其前面加上字符串“->”|
|dir [remote-directory] [local-file]|打印远程目录中的内容的列表，并可以把输出放置在本地文件中。如果启用了交互式提示，则ftp将提示用户确认最后一个参数确实是用于接收dir输出的目标本地文件。如果未指定远程目录，则使用远程计算机上的当前工作目录。如果未指定本地文件，或者本地文件为-，则输出到终端。|
|disconnect  |close的同义词|
|form format|将文件传输格式设置为format。 默认格式为“file”。|
|get remote-file [local-file]|取回远程文件并将其存储在本地计算机上。 如果未指定本地文件名，则使用与远程计算机上相同的名称，该名称可能会因当前的大小写，ntrans和nmap设置而有所更改。 传输文件时会使用当前设置类型，格式，文件模式，结构。|
|glob  |mdelete，mget和mput的文件名扩展。如果使用glob关闭了通配符，则文件名参数将按字面意义解析，并且不会扩展。mput的通配符和csh一样。对于mdelete和mget，每个远程文件名在远程计算机上分别展开，并且列表不合并。目录名称的扩展可能与普通文件名称的扩展不同：确切的结果取决于外部操作系统和ftp服务器，并且可以通过执行“ mls remote-files-”进行预览。注意：mget和mput并不意味着传输文件的整个目录子树，这可以通过传输子树的tar（1）归档文件（以二进制模式）来完成。|
|  hash [increment]| 仅在没有参数的情况下为每个传输的数据块开/关哈希符号（\`\`＃''）打印方式。默认情况下，数据块的大小设置为1024字节，但可以通过参数increment更改，该参数还可以带后缀，'k'和'K'表示千字节，'m'和'M'表示兆字节，“ g”和“ G”表示千兆字节。设置大小将无条件激活哈希打印。|
|help [command]| 打印有关命令含义的信息。如果未提供任何参数，则ftp打印已知命令的列表。|
|  idle [seconds]| 将远程服务器上的不活动计时器设置为秒到秒。如果省略了seconds，则会打印当前的不活动计时器。
|   ipany|允许地址解析器返回任何地址族的ip。|
|    ipv4 |限制地址解析器仅查找IPv4地址。|
|    ipv6    |限制地址解析器仅查找IPv6地址。|
| lcd [directory]|更改本地计算机上的工作目录。 如果未指定目录，则使用用户的主目录。|
|  ls [remote-directory] [local-file]|打印远程计算机上目录的内容列表。 该列表包括服务器选择包含的任何与系统有关的信息； 例如，大多数UNIX系统将从命令“ ls -l”产生输出。 （另请参见nlist。）如果未指定remote-directory，则使用当前工作目录。 如果启用了交互式提示，则ftp将提示用户确认最后一个参数确实是用于接收ls输出的本地文件。 如果未指定本地文件，或者本地文件为“-”，则输出将发送到终端。|
|macdef macro-name| 定义一个宏。随后的行作为宏macro-name存储；空行（终端的连续换行符或者回车换行符）终止宏输入模式。所有定义的宏中最多可以包含16个宏，总共4096个字符。宏将保持定义状态，直到执行close命令为止。宏处理器将“ \$”和“ \”解释为特殊字符。\$后面跟一个数字（或多个数字）将被宏调用命令行上的相应参数替换。 \$后面跟一个i表示该宏处理器正在执行的宏将被循环。在第一次，\$i被宏调用命令行上的第一个参数替换，在第二次，\$i被第二个参数替换，依此类推。后面跟有任何字符的“ \”将替换为该字符。使用“ \”可防止对“$”进行特殊处理。|
|mdelete [remote-files]|删除远程计算机上的文件remote-files。|
|mdir remote-files local-file|类似于dir，但可以指定多个远程文件。如果启用了交互式提示，则ftp将提示用户确认最后一个参数确实是用于接收mdir输出的本地文件。|
|mget remote-files| 在远程机器上展开远程文件，并获取由此产生的每个文件。有关文件名扩展的详细信息，请参见glob。将根据大小写，ntrans和nmap设置处理生成的文件名。文件被传输到本地工作目录，本地目录可以通过“ lcd directory”进行更改；可以使用'!mkdir directory”创建新的本地目录。
| mkdir directory-name|在远程计算机上创建目录。|
| mls remote-files local-file|与nlist一样，除了可以指定多个远程文件之外，还必须指定本地文件。如果启用了交互式提示，则ftp将提示用户确认最后一个参数确实是用于接收mls输出的目标本地文件。|
| mode [mode-name]|将文件传输模式设置为mode-name。默认模式是“stream”模式。|
|modtime file-name|  显示远程计算机上文件的最后修改时间。|
|mput local-files|展开在作为参数给出的本地文件列表中的通配符，并对结果列表中的每个文件进行put。有关文件名扩展的详细信息，请参见glob。将根据ntrans和nmap设置处理生成的文件名。|
|newer file-name [local-file]|  仅当远程文件的修改时间比当前系统上的文件更新时间更加晚时，才获取文件。 如果该文件在当前系统上不存在，则认为该远程文件较新。 否则，此命令与get相同。|
|nlist [remote-directory] [local-file]| 在远程计算机上的目录中打印文件列表。 如果未指定remote-directory，则使用当前工作目录。 如果启用了交互式提示，则ftp将提示用户确认最后一个参数确实是用于接收nlist输出的目标本地文件。 如果未指定本地文件，或者本地文件为-，则将输出发送到终端。|
| nmap [inpattern outpattern]|设置或取消设置文件名映射机制。如果未指定任何参数，则取消设置文件名映射机制。如果指定了参数，则在发出mput和put命令且没有指定远程目标文件名期间映射远程文件名。如果指定了参数，则在发出mget和get命令且没有指定本地目标文件名期间映射本地文件名。当使用不同的文件命名约定或惯例连接到非UNIX远程计算机时，此命令很有用。映射遵循由inpattern和outpattern设置的模式。 Inpattern是传入文件名的模板（可能已经根据ntrans和case设置进行了处理）。通过在inpattern中包含序列\$1，\$2，...，\$9来完成变量模板化。使用“ \”可以防止对“\$”字符进行特殊处理。所有其他字符均按字面意义处理，并用于确定nmap inpattern变量值。例如，给定模式\$1.\$2和远程文件名“ mydata.data”，\$1将具有值“ mydata”，而\$2将具有值“ data”。outpattern确定生成的映射文件名。序列“ \$1”，“ \$2”，....，“ \$9”被替换为inpattern模板产生的任何值。序列“$0”被原始文件名替换。此外，如果seq1不是空字符串，则序列[seq1，seq2]被[seq1]替换；否则，将其替换为seq2。例如，命令`nmap $1.$2.$3 [$1,$2].[$2,file]` 将为输入文件名“ myfile.data”和“ myfile.data.old”生成输出文件名“ myfile.data”，为输入文件名“ myfile”生成“ myfile.file”，为输入文件名生成“ myfile.myfile” “ .myfile”。outpattern可以包含空格，例如`nmap $1 sed "s/  *$//" > $1'`。使用'\\'字符来防止对' \$'、'\['、'\['和'\,'字符进行特殊处理。
|  ntrans [inchars [outchars]]|设置或取消设置文件名字符转换机制。如果未指定任何参数，则未设置文件名字符转换机制。如果指定了参数，则在发出mput和put命令且没有指定远程目标文件名期间转换远程文件名中的字符。如果指定了参数，则在发出mget和get命令且没有指定本地目标文件名期间转换本地文件名中的字符。当使用不同的文件命名约定或惯例连接到非UNIX远程计算机时，此命令很有用。文件名中与inchars中的字符匹配的字符将替换为outchars中的相应字符。如果字符在inchars中的位置长于outchars的长度，则将从文件名中删除该字符。
| open host [port]|    建立与指定主机FTP服务器的连接。 可能会提供一个可选的端口号，在这种情况下，ftp将尝试在该端口与FTP服务器联系。 如果自动登录选项处于启用状态（默认），则ftp还将尝试自动将用户登录到FTP服务器（请参见下文）。
|prompt  |开/关交互式提示。 交互式提示在多个文件传输期间发生，以允许用户有选择地取回或存储文件。 如果关闭提示（默认设置为on），则任何mget或mput都将传输所有文件，而任何mdelete都将删除所有文件。|
| proxy ftp-command|在辅助控制连接上执行ftp命令。此命令允许同时连接到两个远程ftp服务器，以在两个服务器之间传输文件。第一个代理命令应该是open，以建立辅助控制连接。输入命令“ proxy？”查看辅助连接上可执行的其他ftp命令。以下命令在以proxy开头时会有所不同：open不会在自动登录过程中定义新的宏，close不会擦除现有的宏定义，get和mget从主控制连接上的主机传输文件到辅助控制连接上的主机，mput、put、append从辅助控制连接的主机传输文件到主要控制连接的主机。第三方文件传输取决于辅助控制连接上服务器对ftp协议PASV命令的支持。|
| put local-file [remote-file]|将本地文件存储在远程计算机上。如果未指定remote-file，则在命名远程文件使用根据任何ntrans或nmap设置进行处理后的本地文件名。文件传输使用当前设置的type, format, mode, 和 structure。|
|  pwd|在远程计算机上打印当前工作目录的名称。|
|quit     | bye的同义词。|
|quote arg1 arg2 ...|指定的参数将逐字发送到远程FTP服务器。
|recv remote-file [local-file]|get的同义词。|
|  reget remote-file [local-file]|   Reget的行为与get相似，不同之处在于，如果存在本地文件并且该文件小于远程文件，则假定本地文件是远程文件的部分传输副本，并且从明显的故障点开始继续传输。 如果本地文件不存在，则ftp不会获取该文件。 通过易于断开连接的网络传输非常大的文件时，此命令很有用。|
| remotehelp [command-name]|    向远程FTP服务器请求帮助。 如果指定了命令名，它也将提供给服务器。|
|remotestatus [file-name]| 不带参数时，显示远程计算机的状态。 如果指定了文件名，则显示远程计算机上的文件状态。|
|rename [from] [to]| 将远程计算机的文件from重命名为to。|
| reset  |清除回复队列。此命令与远程ftp服务器重新同步命令/答复顺序。远程服务器违反ftp协议后，可能需要重新同步。|
| restart marker|在指定的marker处重新启动get或put。在UNIX系统上， marker 通常是文件中的字节偏移量。|
| rmdir directory-name|  删除远程计算机上的目录。|
|runique  |开/关具有唯一文件名的本地系统上文件存储。如果已经存在一个名称，该名称等于get或mget命令的目标本地文件名，则该名称后会附加一个“ .1”。如果结果名称与另一个现有文件匹配，则在原始名称后附加一个“ .2”。如果此过程继续进行到“ .99”，将显示一条错误消息，并且不会进行传输。生成的唯一文件名将被报告。请注意，runique不会影响从shell命令生成的本地文件（请参见下文）。默认值为off。|
|send local-file [remote-file]| put的同义词。|
| site arg1 arg2 ...|指定的参数作为SITE命令逐字发送到远程FTP服务器。|
| size file-name| 返回远程计算机上文件名的大小。|
|status |显示ftp的当前状态。|
|struct [struct-name]  |将文件传输结构设置为struct-name。默认情况下，使用“stream”结构。|
|sunique  |开/光 远程机器上的唯一文件名存储。远程ftp服务器必须支持ftp protocol STOU命令才能成功完成。远程服务器将报告唯一名称。默认值是关闭的。|
| system    |显示在远程计算机上运行的操作系统的类型。|
| tenex | 将文件传输类型设置为与TENEX机器对话所需的文件传输类型。
| trace |开/关数据包跟踪。|
|type [type-name]|将文件传输类型设置为type-name。如果未指定类型，则打印当前类型。默认类型是网络ASCII。|
|umask [newmask]| 将远程服务器上的默认umask设置为newmask。如果省略了newmask，则打印当前的umask。|
|user user-name [password] [account]| 向远程FTP服务器标识自己。如果未指定password并且服务器需要密码，则ftp将提示用户输入密码（在禁用本地回显之后）。如果未指定account字段，而FTP服务器需要，则将提示用户输入。如果指定了帐户字段且远程服务器不需要帐户命令进行登录，则在完成登录之后，会将帐户命令传输到远程服务器。除非在禁用“auto-login”的情况下调用ftp，否则此过程为初始连接到FTP服务器时自动完成。|
| verbose   |  开光详细模式，在详细模式下，来自FTP服务器的所有响应都显示给用户。此外，如果启用详细信息，则在文件传输完成时，将报告有关传输效率的统计信息。默认情况下，详细信息处于启用状态。|
|  ? [command]|help 的同义词。|

注意：带有空格的命令参数可以用引号““”引起来。

**终止文件传输**：
* 要中止文件传输，请使用终端中断键（通常是Ctrl-C）。 发送传输将立即停止。 接受传输将停止通过向远程服务器发送ftp协议ABOR命令，并丢弃接收到的任何其他数据。 完成此操作的速度取决于远程服务器对ABOR处理的支持。 如果远程服务器不支持ABOR命令，则在远程服务器完成发送请求的文件之前，不会出现“ ftp>”提示。
* 当ftp完成任何本地处理并正在等待远程服务器的答复时，将忽略终端中断键序列。 此模式下的长时间延迟可能是由于上述ABOR处理或由于远程服务器的意外行为（包括违反ftp协议）引起的。 如果延迟是由意外的远程服务器行为导致的，则必须手动终止本地ftp程序。
  
 **文件命名约定**:指定为ftp命令参数的文件将根据以下规则进行处理。
 1. 如果指定文件名“-”，则使用stdin（用于读取）或stdout（用于写入）。
 2. 如果文件名的第一个字符是“ |”，则参数的其余部分将解释为shell命令。 Ftp然后使用带有提供的参数的popen（3）派生一个shell，并从stdout（stdin）读取（写入）。如果shell命令包含空格，则必须用引号将其引起来；例如““ ls -lt””。这种机制的一个特别有用的例子是：“dir more”。
 3. 没有通过上述检查，如果启用了“ globbing”，则根据csh（1）中使用的规则扩展本地文件名； c.f. glob命令。如果ftp命令需要单个本地文件（例如put），则仅使用“ globbing”操作生成的第一个文件名。
 4. 对于具有未指定本地文件名的mget命令和get命令，本地文件名是远程文件名，可以通过case，ntrans或nmap设置进行更改。如果runique打开，则也可以更改结果文件名。|
 5. 对于带有未指定远程文件名的mput命令和put命令，远程文件名是本地文件名，可以通过ntrans或nmap设置来更改。 如果打开了sunique，则可以由远程服务器更改结果文件名。|
 
 **文件传输参数**：
  
* FTP规范指定了许多可能影响文件传输的参数。 type可以是“ ascii”，“ image”（二进制），“ ebcdic”和“ local byte size”（主要用于PDP-10和PDP-20）之一。 Ftp支持文件传输的ascii和图像类型，以及用于tenex模式传输的本地字节大小8。
* Ftp仅支持其余文件传输参数的默认值：mode，form和struct。
  
**环境变量**： Ftp利用以下环境变量。
* HOME：.netrc文件的默认位置(如果存在)。
* SHELL： 默认shell。


  
------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
