---
title: MySQL简明教程
tags: 数据库
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文翻译自<font color=blue>[《MYSQ参考手册5.7》](https://dev.mysql.com/doc/refman/5.7/en/)。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

<style>table{word-break:initial;}</style>

# 1  mysql数据库简介
MySQL是一个关系型数据库管理系统，由瑞典MySQL AB公司开发，目前属于Oracle公司。MySQL是一种关联数据库管理系统，关联数据库将数据保存在不同的表中，而不是将所有数据放在一个大仓库内，这样就增加了速度并提高了灵活性。

Mysql是开源的，所以你不需要支付额外的费用。

Mysql支持大型的数据库。可以处理拥有上千万条记录的大型数据库。

MySQL使用标准的SQL数据语言形式。

Mysql可以允许于多个系统上，并且支持多种语言。这些编程语言包括C、C++、Python、Java、Perl、PHP、Eiffel、Ruby和Tcl等。

Mysql对PHP有很好的支持，PHP是目前最流行的Web开发语言。

MySQL支持大型数据库，支持5000万条记录的数据仓库，32位系统表文件最大可支持4GB，64位系统支持最大的表文件为8TB。

Mysql是可以定制的，采用了GPL协议，你可以修改源码来开发自己的Mysql系统。
 
 # 2 安装mysql
## 2.1 ubuntu/linux 
安装：
```
sudo apt install  mysql-server 	//安装服务器端

sudo apt install  mysql-client        //安装命令行客户端

sudo apt install  mysql-workbench      //安图形化客户端界面

sudo apt install  libmysqlclient-dev    //安装c语言开发环境

```
卸载：
```
sudo apt  autoremove --purge mysql*
sudo rm -rf /etc/mysql/  /var/lib/mysql
dpkg -l |grep ^rc|awk '{print $2}' |sudo xargs dpkg -P  
```


## 2.2 Window上安装Mysql
Window上安装Mysql相对来说会较为简单，你只需要载 MySQL 下载中下载window版本的mysql安装包，并解压安装包。双击 setup.exe 文件，接下来你只需要安装默认的配置点击"next"即可，默认情况下安装信息会在C:\mysql目录中。

接下来你可以通过"开始" =》在搜索框中输入 " cmd" 命令 =》 在命令提示符上切换到 C:\mysql\bin 目录，并输入一下命令：
```shell
mysqld.exe --console
```
如果安装成功以上命令将输出一些mysql启动及InnoDB信息。 


# 3 msyql程序
## 3.1 mysql客户程序
### 3.1.1 mysqladmin

**命令**：mysqladmin [options] command [command-options] [command [command-options]] ...
**描述**：mysqladmin是执行管理操作的客户端。 您可以使用它来检查服务器的配置和当前状态，创建和删除数据库等。所有命令可以只写出唯一前缀。

|命令|描述|
|:--|:--|
|**create dbname**|创建数据库|
|debug|告诉服务器将调试信息写入错误日志。连接的用户必须具有SUPER权限。此信息的格式和内容可能会有所变化。这包括有关事件调度​​程序的信息
|**drop dbname**|删除名为db_name的数据库及其所有表。
|extended-status|显示服务器状态变量及其值。|
|flush-hosts|刷新主机缓存中的所有信息。|
|flush-logs [log_type ...]|刷新所有日志。日志类型：binary，engine，error，general，relay，slow。这些对应于可以为FLUSH LOGS SQL语句指定的日志类型。
|**flush-privileges**| 重新加载授权表（与重新加载相同）。|
|flush-status|清除状态变量。|
|flush-tables|刷新所有表		|
|flush-threads|刷新所有线程缓存|
|kill id,id,...| 杀死服务器线程。 如果有多个线程ID，则不能有空格。 要终止其他用户的线程，该用户必须具有SUPER权限|
|old-password new_password|这与password命令类似，但使用旧的（4.1之前的）密码散列格式存储密码。5.7.5删除|
|**password new_password**|&emsp;&emsp;设置新密码。这会将您使用mysqladmin连接到服务器的帐户的密码更改为new_password。因此，下次使用同一帐户调用mysqladmin（或任何其他客户端程序）时，您需要指定新密码。<br />&emsp;&emsp;警告使用mysqladmin设置密码应该被认为是不安全的。在某些系统上，您的密码对系统状态程序（例如ps）可见，其他用户可以调用它来显示命令行。 MySQL客户端通常在初始化序列期间用零覆盖命令行密码参数。但是，仍然存在一个短暂的间隔，在该间隔期间值是可见的。此外，在某些系统上，此覆盖策略无效，并且ps仍然可以看到密码。 （SystemV Unix系统以及其他系统可能会遇到此问题。）<br />&emsp;&emsp;如果new_password值包含空格或其他对命令解释程序特殊的字符，则需要将其括在引号内。在Windows上，请务必使用双引号而不是单引号;单引号不会从密码中删除，而是被解释为密码的一部分。<br /> &emsp;&emsp;在MySQL 5.7中，密码命令后可以省略新密码。在这种情况下，mysqladmin会提示输入密码值，这样可以避免在命令行上指定密码。仅当密码是mysqladmin命令行上的最终命令时，才应省略密码值。否则，将下一个参数作为密码。<br />&emsp;&emsp;警告 如果使用--skip-grant-tables选项启动服务器，请不要使用此命令。不会应用密码更改。即使您在同一命令行上使用flush命令在password命令之前重新启用授权表，也是如此，因为在连接之后会发生刷新操作。但是，您可以使用mysqladmin flush-privileges重新启用授权表，然后使用单独的mysqladmin password命令更改密码。|
|ping|检查服务器是否可用。 如果服务器正在运行，则mysqladmin的返回状态为0，否则为1。 即使出现拒绝访问等错误，这也是0，因为这意味着服务器正在运行但拒绝连接，这与服务器未运行不同。
| processlist|显示活动的服务器线程列表。 这类似于SHOW PROCESSLIST语句的输出。 如果给出--verbose选项，则输出类似于SHOW FULL PROCESSLIST。|
|**reload**| 重新加载授权表。|
|refresh| 刷新所有表并关闭和打开日志文件。|
|**shutdown**| 停止服务器。|
|start-slave| 在从属服务器上启动复制。|
|status|  显示简短的服务器状态消息。|
|stop-slave|停止从服务器上的复制。|
| variables|显示服务器系统变量及其值。|
| version|  显示服务器的版本信息。
|&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;|  |


|status命令结果|描述|
|:--|:--|
|Uptime|MySQL服务器运行的秒数。|
|Threads|  活动线程数（客户端）。|
|Questions| 自服务器启动以来客户端的问题（查询）数。|
|Slow queries|  超过long_query_time秒的查询数。|
|Opens| 服务器已打开的表数。|
| Flush tables|服务器已执行的flush- *，refresh和reload命令的数量。
|Open tables|当前打开的表的数量。|



|短选项|长选项|描述|
|:--|:--|:--|
|-？|--help|显示帮助消息并退出。|
||**--bind-address=ip**| 在具有多个网络接口的计算机上，使用此选项可选择用于连接MySQL服务器的接口。|
||--character-sets-dir=dir|安装字符集的目录。|
| -C|--compress|如果客户端和服务器都支持压缩，压缩所有通信信息。|
|-c N|--count=N|如果给出--sleep选项，则N是重复执行命令的迭代次数。|
|-# [ops]|--debug[=ops] |根据调试选项编写调试日志。如 d:t:o,file_name.默认为d:t:o,/tmp/mysqladmin.trace.
| |--debug-check|  程序退出时打印一些调试信息。|
| |--debug-info|程序退出时打印调试信息，内存和CPU使用情况统计信息。
|| --default-auth=plugin|   有关要使用的客户端身份验证插件的提示。 |
|| --default-character-set=charset_name|使用charset_name作为默认字符集。|
|| --defaults-extra-file=file_name|在全局选项文件之后但在用户选项文件之前读取此选项文件。如果文件不存在或无法访问，则会发生错误。|
||  --defaults-file=file_name|  仅使用给定的选项文件。如果文件不存在或无法访问，则会发生错误。即使使用--defaults-file，客户端程序也会读取.mylogin.cnf。|
||--defaults-group-suffix=str|不仅要读取常用选项组，还要读取通常名称和str后缀的组。如果给出了--defaults-group-suffix =\ _other选项，mysql也会读取[client\_other]和[mysql\_other]组。|
|| --enable-cleartext-plugin|启用mysql_clear_password明文身份验证插件。|
|-f|--force|请勿确认drop db_name命令。 使用多个命令，即使发生错误也会继续|。
| |--get-server-public-key|  从服务器请求基于RSA密钥对的密码交换所需的公钥。此选项适用于使用caching_sha2_password身份验证插件进行身份验证的客户端。对于该插件，除非有请求，否则服务器不会发送公钥。对于未使用该插件进行身份验证的帐户，将忽略此选项。如果不使用基于RSA的密码交换，也会忽略它，例如客户端使用安全连接连接到服务器的情况。如果给出了--server-public-key-path = file_name并指定了有效的公钥文件，则它优先于--get-server-public-key。
| **-h host_name**|--host=host_name| 连接到给定主机上的MySQL服务器。
| |--login-path=name|从.mylogin.cnf登录路径文件中的指定登录路径中读取选项。 “登录路径”是一个选项组，其中包含指定要连接到哪个MySQL服务器以及要进行身份验证的帐户的选项。|
| -b|--no-beep|    连接失败等错误时不发出的警告蜂鸣声|
||--no-defaults|  不要读任何选项文件。 如果由于从选项文件中读取未知选项而导致程序启动失败，则可以使用--no-defaults来防止它们被读取。例外情况是.mylogin.cnf文件（如果存在）在所有情况下都会被读取。|
| **-p[passwd]**|--password[=passwd]| 连接服务器时使用的密码。如果使用短选项表单（-p），则选项和密码之间不能有空格。在命令行上指定密码应该被认为是不安全的。请参见第6.1.2.1节“密码安全的最终用户指南”。您可以使用选项文件避免在命令行上输入密码。|
|-W|--pipe|在Windows上，使用命名管道连接到服务器。仅当服务器支持命名管道连接时，此选项才适用。
|| --plugin-dir=dir_name|查找插件的目录。如果使用--default-auth选项指定身份验证插件但mysql找不到它，请指定此选项。请参见第6.3.9节“可插入验证”。|
| **-P port_num**|--port=port_num|   用于连接的TCP / IP端口号。|
||--print-defaults|打印程序名称以及从选项文件中获取的所有选项。  有关此选项和其他选项文件选项的其他信息，请参见第4.2.7节“影响选项文件处理的命令行选项”。|
|| --protocol={protocol}|   用于连接服务器的连接协议。TCP、SOCKET、PIPE、MEMORY，有关允许值的详细信息，请参见第4.2.2节“连接到MySQL服务器”。|
|-r|--relative|与--sleep选项一起使用时，显示当前值和先前值之间的差异。 此选项仅适用于extended-status命令。
||  --show-warnings|显示执行发送到服务器的语句导致的警告。|
||--secure-auth|不要以旧的（4.1之前的）格式向服务器发送密码。这可以防止除使用较新密码格式的服务器之外的连接。 从MySQL 5.7.5开始，该选项已弃用，将在未来的MySQL版本中删除。它始终处于启用状态并尝试禁用它（ -- skip-secure-auth，--secure-auth = 0）产生错误。在MySQL 5.7.5之前，默认情况下启用此选项，但可以禁用此选项。  注意：使用4.1之前的哈希方法的密码不如使用本机密码哈希方法的密码安全，应该避免使用。不推荐使用4.1之前的密码，并且在MySQL 5.7.5中删除了对它们的支持。有关帐户升级说明，请参见第6.5.1.3节“迁移远离4.1之前的密码散列和mysql_old_password插件”。
|| --server-public-key-path=file_name|包含服务器为基于RSA密钥对的密码交换所需的公钥的客户端副本的文件的路径名。该文件必须采用PEM格式。这个选项适用于使用sha256_password或caching_sha2_password身份验证插件进行身份验证的客户端。对于未使用其中一个插件进行身份验证的帐户，将忽略此选项。如果不使用基于RSA的密码交换，也会忽略它，例如客户端使用安全连接连接到服务器的情况。  如果给出了--server-public-key-path = file_name并指定了有效的公钥文件，则它优先于--get-server-public-key。对于sha256_password，此选项仅适用于使用OpenSSL构建MySQL的情况。
||--shared-memory-base-name=name| 在Windows上，要使用的共享内存名称，用于使用共享内存连接到本地服务器。 默认值为MYSQL。 共享内存名称区分大小写。必须使用--shared-memory选项启动服务器以启用共享内存连接。|
| -s|--silent|    如果无法建立与服务器的连接，则以静默方式退出。|
|  -i delay|--sleep=delay| 重复执行命令，在两者之间休眠delay秒。 --count选项确定迭代次数。如果没有给出--count，mysqladmin会无限期地执行命令直到被中断。|
|-S path|   --socket=path| 用于连接到localhost，要使用的Unix套接字文件，或者在Windows上，要使用的命名管道的名称。|
|--ssl * |  以--ssl开头的选项指定是否使用SSL连接到服务器并指示在何处查找SSL密钥和证书|
||  --tls-version=protocol_list         |  客户端允许的加密连接协议。该值是以逗号分隔的列表，其中包含一个或多个协议名称。可以为此选项命名的协议取决于用于编译MySQL的SSL库。
|**-u user_name**| --user=user_name| 连接到服务器时使用的MySQL用户名。|
|-v|--verbose|详细模式。打印有关程序功能的更多信息。|
|-V|--version|显示版本信息并退出。
|-E |--vertical|  垂直打印输出。这类似于--relative，但是垂直打印输出。|
|  -w[count]|--wait[=count]|    如果无法建立连接，请等待并重试，而不是中止。 如果给出计数值，则表示重试的次数。 默认是一次。
||--connect_timeout= value|连接超时前的最大秒数。 默认值为43200（12小时）。
||--shutdown_timeout= value|等待服务器关闭的最大秒数。 默认值为3600（1小时）。
 
### 3.1.2  mysql

**命令**：mysql [options] db_name
**描述**：
&emsp;&emsp; mysql是一个带有输入行编辑功能的简单SQL shell。它支持交互式和非交互式使用。交互使用时，查询结果以ASCII表格式显示。当非交互式使用时（例如，作为过滤器），结果以制表符分隔格式显示。可以使用命令选项更改输出格式。

&emsp;&emsp;如果由于大型结果集的内存不足而出现问题，请使用--quick选项。这迫使mysql一次一行地从服务器检索结果，而不是检索整个结果集并在显示它之前在内存中缓冲它。这是通过使用客户端/服务器库中的mysql_use_result（）C API函数而不是mysql_store_result（）返回结果集来完成的。
&emsp;&emsp;注意：MySQL Shell提供对X DevAPI的访问。有关详细信息，请参阅MySQL Shell 8.0（MySQL 8.0的一部分）。
&emsp;&emsp;使用mysql非常容易。从命令解释器的提示符调用它，如下所示：
&emsp;&emsp;&emsp;&emsp;shell> mysql db_name
    或者
&emsp;&emsp;&emsp;&emsp;shell> mysql --user = user_name --password db_name
&emsp;&emsp;输入密码：your_password
&emsp;&emsp;然后键入一个SQL语句，以;，\ g或\ G结束它，然后按Enter键。
&emsp;&emsp;如果存在任何语句，则键入Control + C将中断当前语句，否则将取消任何部分输入行。
&emsp;&emsp;您可以在脚本文件（批处理文件）中执行SQL语句，如下所示：
&emsp;&emsp;&emsp;&emsp;shell> mysql db_name <script.sql> output.tab
&emsp;&emsp;在Unix上，mysql客户端将以交互方式执行的语句记录到历史文件中。请参阅“MYSQL客户端日志记录”一节。

&emsp;&emsp;mysql将您发出的每个SQL语句发送到要执行的服务器。 还有一组mysql自己解释的命令。 


 |短选项|长选项|描述|
|:--|:--|:--|
|-？|--help|显示帮助消息并退出。|
| |**--auto-rehash**| 启用自动重新散列。默认情况下，此选项处于启用状态，可以使数据库，表和列名称按TAB补全。使用--disable-auto-rehash禁用重新散列。这会导致mysql启动得更快，但如果要使用名称补全，则必须发出rehash或\\#命令。注意： 此功能需要使用readline库编译的MySQL客户端。通常，Windows上不提供readline库。
||**--auto-vertical-output**|           如果结果集对于当前窗口来说太宽，则会导致垂直显示结果集，否则使用普通的表格格式。 （这适用于以;或\ G终止的语句）|
 | -B|--batch|使用制表符作为列分隔符打印结果，每行都在新行上。 使用此选项，mysql不使用历史记录文件。批处理模式导致非常规输出格式和特殊字符的转义。 可以使用原始模式禁用转义; 请参阅--raw选项的说明。|
 | |--binary-as-hex| 当给出此选项时，mysql使用十六进制表示法（0xvalue）显示二进制数据。 无论整体输出显示格式是表格格式，垂直格式，HTML格式还是XML格式，都会出现这种情况。|
 | | --binary-mode| 此选项在处理可能包含BLOB值的mysqlbinlog输出时有帮助。默认情况下，mysql将语句字符串中的\ r \ n转换为\ n，并将\ 0解释为语句终止符。 --binary-mode禁用这两个功能。它还在非交互模式下禁用除charset和delimiter之外的所有mysql命令（用于输入管道到mysql或使用source命令加载）。
 ||**--bind-address=ip**|在具有多个网络接口的计算机上，使用此选项可选择用于连接MySQL服务器的接口。|
||--character-sets-dir=dir_name| 安装字符集的目录。|
||  **--column-names**|在结果中写入列名称。|
||  --column-type-info| 显示结果集元数据。|
|-c|--comments|是否在发送到服务器的语句中删除或保留注释。默认值为--skip-comments（跳过注释），要启用使用--comments（保留注释）。
| -C|--compress|如果客户端和服务器都支持压缩，压缩所有通信信息。|
||--connect-expired-password |  如果用于连接的帐户具有过期密码，则向服务器指示客户端可以处理沙盒模式。这对于mysql的非交互式调用很有用，因为通常服务器会断开尝试使用具有过期密码的帐户进行连接的非交互式客户端。|
| **-D db_name**| --database=db_name| 要使用的数据库。这主要用在选项文件中|
|-# [ops]|--debug[=ops] |根据调试选项编写调试日志。如 d:t:o,file_name.默认为d:t:o,/tmp/mysqladmin.trace. 只有在使用WITH_DEBUG构建MySQL时，此选项才可用。|
| |--debug-check| 程序退出时打印一些调试信息。|
|-T|-debug-info|     程序退出时打印调试信息，内存和CPU使用情况统计信息。|
|| --default-auth=plugin|有关要使用的客户端身份验证插件的提示。 |
|| --default-character-set=charset_name|   使用charset_name作为客户端和连接的默认字符集。   如果操作系统使用一个字符集并且默认情况下mysql客户端使用另一个字符集，则此选项很有用。因为 在这种情况下，输出可能格式不正确。|
||--defaults-extra-file=file_name|
|| --defaults-extra-file=file_name|在全局选项文件之后但在用户选项文件之前读取此选项文件。如果文件不存在或无法访问，则会发生错误。|
||  --defaults-file=file_name|  仅使用给定的选项文件。如果文件不存在或无法访问，则会发生错误。即使使用--defaults-file，客户端程序也会读取.mylogin.cnf。|
||--defaults-group-suffix=str|不仅要读取常用选项组，还要读取通常名称和str后缀的组。例如，mysql通常会读取[client]和[mysql]组。如果给出了--defaults-group-suffix =\_other选项，mysql也会读取[client\_other]和[mysql\_other]组。|
||**--delimiter=str**| 设置语句分隔符。 默认值为分号（;）。|
|| --disable-named-commands |禁用命名命令。 仅使用\ * 表单，或仅在以分号（;）结尾的行的开头使用命名命令。 mysql默认启用此选项。 但是，即使使用此选项，长格式命令仍然可以从第一行开始工作。|
|| --enable-cleartext-plugin|  启用mysql_clear_password明文身份验证插件。|
| -e statement| --execute=statement|执行该语句并退出。默认输出格式类似于--batch生成的格式。|
| -f|--force|   即使发生SQL错误也继续。
| |--get-server-public-key|  从服务器请求基于RSA密钥对的密码交换所需的公钥。此选项适用于使用caching_sha2_password身份验证插件进行身份验证的客户端。对于该插件，除非有请求，否则服务器不会发送公钥。对于未使用该插件进行身份验证的帐户，将忽略此选项。如果不使用基于RSA的密码交换，也会忽略它，例如客户端使用安全连接连接到服务器的情况。如果给出了--server-public-key-path = file_name并指定了有效的公钥文件，则它优先于--get-server-public-key。
 || --histignore|  由冒号分隔的一个或多个模式的列表，指定要为日志忽略的语句。 这些模式将添加到默认模式列表（“\* IDENTIFIED\*：\* PASSWORD \*”）中。 为此选项指定的值会影响写入历史文件的语句的记录，如果给出--syslog选项，则会影响syslog。|
 | **-h host_name**|--host=host_name| 连接到给定主机上的MySQL服务器。|
 | -H|--html|   生成HTML输出。|
 |-i|--ignore-spaces|  忽略函数名后的空格。在IGNORE_SPACE SQL模式的讨论中描述了这种效果（请参见第5.1.10节“服务器SQL模式”）。|
 || --init-command=str|连接到服务器后要执行的SQL语句。如果启用了自动重新连接，则在重新连接后再次执行该语句。|
 ||--line-numbers | 写错误的行号。使用--skip-line-numbers禁用它。|
 || --local-infile[=={0|1}]| 启用或禁用LOAD DATA INFILE的LOCAL功能。对于mysql，默认情况下禁用此功能。没有值，该选项启用LOCAL。该选项可以作为--local-infile = 0或--local-infile = 1给出，以显式禁用或启用LOCAL。启用本地数据加载还需要服务器允许它;请参见第6.1.6节“LOAD DATA LOCAL的安全问题”
||--login-path=name|从.mylogin.cnf登录路径文件中的指定登录路径中读取选项。 “登录路径”是一个选项组，其中包含指定要连接到哪个MySQL服务器以及要进行身份验证的帐户的选项。要创建或修改登录路径文件，请使用mysql_config_editor实用程序。请参阅mysql_config_editor（1）。有关此选项和其他选项文件选项的其他信息，请参见第4.2.7节“影响选项文件处理的命令行选项”。|
|-G|--named-commands|启用命名的mysql命令。 允许使用长格式命令，而不仅仅是短格式命令。 例如，quit和\ q都被识别。 使用--skip-named-commands禁用命名命令。 请参阅“MYSQL客户端命令”一节。
|-A|--no-auto-rehash| 这与--skip-auto-rehash具有相同的效果。 请参阅--auto-rehash的说明。
|-b|--no-beep|发生错误时不要发出蜂鸣声。
 ||--no-defaults|  不要读任何选项文件。 如果由于从选项文件中读取未知选项而导致程序启动失败，则可以使用--no-defaults来防止它们被读取。例外情况是.mylogin.cnf文件（如果存在）在所有情况下都会被读取。|
 |-o|--one-database|忽略除默认数据库是命令行上的命名数据库时发生的语句。此选项很简陋，应谨慎使用。语句过滤仅基于USE语句。最初，mysql在输入中执行语句，因为在命令行上指定数据库db_name等同于在输入的开头插入USE db_name。然后，对于遇到的每个USE语句，mysql接受或拒绝以下语句，具体取决于命名数据库是否为命令行中的数据库。陈述的内容并不重要。假设调用mysql来处理这组语句：<br />DELETE FROM db2.t2;<br /> USE db2;<br /> DROP TABLE db1.t1;<br />CREATE TABLE db1.t1 (i INT);<br />  USE db1;<br /> INSERT INTO t1 (i) VALUES(1);<br />    CREATE TABLE db2.t1 (j INT);<br /> 如果命令行是mysql --force --one-database db1，mysql将按如下方式处理输入：执行DELETE语句是因为默认数据库是db1，即使该语句在不同的数据库中命名一个表。DROP TABLE和CREATE TABLE语句未执行，因为默认数据库不是db1，即使这些语句在db1中命名了一个表。执行INSERT和CREATE TABLE语句是因为缺省数据库是db1，即使CREATE TABLE语句在另一个数据库中命名一个表也是如此。
 || --pager[=command]|使用给定命令进行分页查询输出。如果省略该命令，则默认分页器是PAGER环境变量的值。有效的分页器less，more，cat [> filename]等等。此选项仅适用于Unix，仅适用于交互模式。要禁用分页，请使用--skip-pager。名为“MYSQL CLIENT COMMANDS”的部分进一步讨论了输出分页。
| **-p[passwd]**|--password[=passwd]| 连接服务器时使用的密码。如果使用短选项表单（-p），则选项和密码之间不能有空格。在命令行上指定密码应该被认为是不安全的。请参见第6.1.2.1节“密码安全的最终用户指南”。您可以使用选项文件避免在命令行上输入密码。|
|-W|--pipe|在Windows上，使用命名管道连接到服务器。仅当服务器支持命名管道连接时，此选项才适用。
|| --plugin-dir=dir_name|查找插件的目录。如果使用--default-auth选项指定身份验证插件但mysql找不到它，请指定此选项。请参见第6.3.9节“可插入验证”。|
| **-P port_num**|--port=port_num|   用于连接的TCP / IP端口号。|
||--print-defaults|打印程序名称以及从选项文件中获取的所有选项。  有关此选项和其他选项文件选项的其他信息，请参见第4.2.7节“影响选项文件处理的命令行选项”。|
 || --prompt=format_str|  将提示符设置为指定的格式。 默认为mysql>。 提示可以包含的特殊序列在“MYSQL客户端命令”一节中描述。|
 || --protocol={protocol}|   用于连接服务器的连接协议。TCP、SOCKET、PIPE、MEMORY，有关允许值的详细信息，请参见第4.2.2节“连接到MySQL服务器”。|
 |-q |--quick|  不要缓存每个查询结果，在收到时打印每一行。如果输出暂停，这可能会降低服务器的速度。使用此选项，mysql不使用历史记录文件。|
 | --r|--raw|对于表格输出，列周围的“方框”可以将一个列值与另一个列值区分开来。 对于非表格输出（例如以批处理模式或在给出--batch或--silent选项时生成），特殊字符将在输出中转义，以便可以轻松识别它们。 换行符，制表符，NUL和反斜杠被写为\ n，\ t，\ 0和\\。 --raw选项禁用此字符转义。|
|  |--reconnect|    如果与服务器的连接丢失，则自动尝试重新连接。每次连接丢失时都会进行一次重新连接尝试。要禁止重新连接行为，请使用--skip-reconnect。|
| -U|--safe-updates,--i-am-a-dummy| 如果启用此选项，UPDATE和DELETE语句的WHERE子句或LIMIT子句中的不适用键时会产生错误。此外，对产生（或估计产生）非常大的结果集的SELECT语句施加了限制。如果在选项文件中设置了此选项，则可以在命令行上使用--skip-safe-updates来覆盖它。有关此选项的详细信息，请参阅“使用安全更新模式（--safe-updates）”一节。
||--secure-auth|不要以旧的（4.1之前的）格式向服务器发送密码。这可以防止除使用较新密码格式的服务器之外的连接。 从MySQL 5.7.5开始，该选项已弃用，将在未来的MySQL版本中删除。它始终处于启用状态并尝试禁用它（ -- skip-secure-auth，--secure-auth = 0）产生错误。在MySQL 5.7.5之前，默认情况下启用此选项，但可以禁用此选项。  注意：使用4.1之前的哈希方法的密码不如使用本机密码哈希方法的密码安全，应该避免使用。不推荐使用4.1之前的密码，并且在MySQL 5.7.5中删除了对它们的支持。有关帐户升级说明，请参见第6.5.1.3节“迁移远离4.1之前的密码散列和mysql_old_password插件”。|
||--server-public-key-path = file_name|包含服务器为基于RSA密钥对的密码交换所需的公钥的客户端副本的文件的路径名。 该文件必须采用PEM格式。 此选项适用于使用sha256_password身份验证插件进行身份验证的客户端。 对于未使用其中一个插件进行身份验证的帐户，将忽略此选项。 如果不使用基于RSA的密码交换，也会忽略它，例如客户端使用安全连接连接到服务器的情况。 只有在使用OpenSSL构建MySQL时，此选项才可用。 有关sha256_password和caching_sha2_password插件的信息，请参见第6.5.1.4节“SHA-256可插入认证”和第6.5.1.5节“高速缓存SHA-2可插入认证”。
||--shared-memory-base-name=name| 在Windows上，要使用的共享内存名称，用于使用共享内存连接到本地服务器。 默认值为MYSQL。 共享内存名称区分大小写。必须使用--shared-memory选项启动服务器以启用共享内存连接。|
||--show-warnings| 如果有任何语句，则会在每个语句后显示警告。此选项适用于交互式和批处理模式。|
||-sigint-ignore|  忽略SIGINT信号（通常是键入Control + C的结果）。|
|-s|--silent| 静音模式。 产量减少。 可以多次给出此选项以产生越来越少的输出。  此选项会导致非常规输出格式和特殊字符的转义。 可以使用原始模式禁用转义; 请参阅--raw选项的说明。|
| -N| --skip-column-names|    不要在结果中写入列名。|
|-L| --skip-line-numbers| 不要为错误写行号。 当您想要比较包含错误消息的结果文件时很有用。|
|-S path|   --socket=path| 用于连接到localhost，要使用的Unix套接字文件，或者在Windows上，要使用的命名管道的名称。|
||--ssl *| 以 --ssl开头的选项指定是否使用SSL连接到服务器并指示在何处查找SSL密钥和证书。 请参见第6.4.2节“加密连接的命令选项”。
|-j|--syslog| 此选项使mysql将交互式语句发送到系统日志记录工具。在Unix上，这是syslog;在Windows上，它是Windows事件日志。记录消息的出现位置取决于系统。在Linux上，目标通常是/ var / log / messages文件。 有关更多信息，请参阅“MYSQL客户端日志记录”一节。
|-t|--table| 以表格格式显示输出。这是交互式使用的默认设置，但可用于以批处理模式生成表输出。|
||--tee = file_name|  将输出的副本附加到给定文件。 此选项仅在交互模式下有效。 名为“MYSQL CLIENT COMMANDS”的部分，进一步讨论了tee文件。|
| |--tls-version=protocol_list|   客户端允许的加密连接协议。 该值是以逗号分隔的列表，其中包含一个或多个协议名称。 可以为此选项命名的协议取决于用于编译MySQL的SSL库。 有关详细信息，请参见第6.4.6节“加密连接协议和密码”。|
|-n|--unbuffered|每次查询后刷新缓冲区。|
|**-u user_name**| --user=user_name| 连接到服务器时使用的MySQL用户名。|
|-v|--verbose|详细模式。打印有关程序功能的更多信息。|
|-V|--version|显示版本信息并退出。
|-E |--vertical|  垂直打印输出。这类似于--relative，但是垂直打印输出。|
|-w|--wait|如果无法建立连接，请等待并重试，而不是中止。
|-X|--xml|生成XML输出。
||--connect_timeout=value|  连接超时前的秒数。 （默认值为0.）|
||--max_allowed_packet=value| 客户端/服务器通信的最大缓冲区大小。 默认值为16MB，最大值为1GB。
||--max_join_size=value|使用--safe-updates时连接中行的自动限制。 （默认值为1,000,000。）
||--net_buffer_length=value|TCP / IP和套接字通信的缓冲区大小。 （默认值为16KB。）
||--select_limit=value|   使用--safe-updates时SELECT语句的自动限制。 （默认值为1,000。）
 
 
|长命令|短命令|描述|
|:--|:--|:--|
|help [arg]|\\h [arg], \? [arg], ? [arg]|显示列出可用mysql命令的帮助消息。如果为help命令提供参数，mysql将其用作搜索字符串，以从MySQL参考手册的内容访问服务器端帮助。有关详细信息，请参阅“MYSQL客户端服务器端帮助”一节。|
|charset charset_name,|\C charset_name|   更改默认字符集并发出SET NAMES语句。 如果在启用自动重新连接的情况下运行mysql（不建议这样做），这使字符集在客户端和服务器上保持同步，因为指定的字符集用于重新连接。}
| clear|**\\c**|    清除当前输入。 如果您改变主意执行要输入的语句，请使用此选项。|
| connect [db_name host_name]]| \r [db_name host_name]]| 重新连接到服务器。 可以给出可选的数据库名称和主机名参数，以指定默认数据库或运行服务器的主机。 如果省略，则使用当前值。|
| delimiter str|**\\d str**| 更改mysql解释为SQL语句之间的分隔符的字符串。默认值为分号（;）。|可以在分隔符命令行上将分隔符字符串指定为不带引号或带引号的参数。引用可以使用单引号（'），双引号（“）或反引号（`）字符。要在引用的字符串中包含引号，请使用不同的引号字符引用字符串或使用反斜杠转义引号（\）字符。应该避免在带引号的字符串之外使用反斜杠，因为它是MySQL的转义符。对于不带引号的参数，分隔符被读取到第一个空格或行尾。对于带引号的参数，读取分隔符直到线上的匹配报价。除了在带引号的字符串中，mysql将分隔符字符串的实例解释为语句分隔符。注意定义可能出现在其他单词中的分隔符。例如，如果将分隔符定义为X，则无法在语句中使用INDEX一词。 mysql将此解释为INDE，后跟分隔符X.当mysql识别的分隔符设置为默认值;时，该字符的实例将被发送到服务器而不进行解释。但是，服务器本身仍然会解释;作为语句分隔符并相应地处理语句。服务器端的这种行为可用于多语句执行（参见第27.8.16节“C API多语句执行支持”），以及解析存储过程和函数体，触发器和事件（参见第23.1节） ，“定义存储程序”）。
| edit| \\e|编辑当前输入语句。 mysql检查EDITOR和VISUAL环境变量的值以确定要使用的编辑器。如果两个变量都未设置，则默认编辑器为vi。编辑命令仅在Unix中有效。
| ego|**\\G**| 将当前语句发送到要执行的服务器，并使用垂直格式显示结果。|
|exit|\\q| 退出mysql。
|go |\\g|   将当前语句发送到要执行的服务器。
|nopager|\\n|  禁用输出分页。请参阅分页器的说明。   nopager命令仅适用于Unix。|
| notee| 	\\t|  禁用输出复制到tee文件。请参阅tee的说明。
|  nowarning| \\w|在每个语句后禁用警告显示。|
 |pager [command]|\\P [command]|启用输出分页。通过在调用mysql时使用--pager选项，可以使用Unix程序（如less，more或任何其他类似程序）以交互模式浏览或搜索查询结果。如果没有为该选项指定值，mysql将检查PAGER环境变量的值并将寻呼机设置为该值。寻呼机功能仅在交互模式下有效。可以使用pager命令以交互方式启用输出分页，并使用nopager禁用输出分页。该命令采用可选参数;如果给定，则将寻呼程序设置为该。如果没有参数，则将分页器设置为在命令行上设置的分页器，如果未指定分页器，则设置为stdout。输出分页仅在Unix中有效，因为它使用了popen（）函数，该函数在Windows上不存在。对于Windows，可以使用tee选项来保存查询输出，尽管在某些情况下它不像分页器那样方便浏览输出。
|print|\\p|  打印当前输入语句而不执行它。|
| prompt [str]|\\R [str]|   将mysql提示重新配置为给定的字符串。可在提示中使用的特殊字符序列将在本节后面介绍。如果指定不带参数的prompt命令，mysql会将提示重置为缺省值mysql>。
| quit|**\\q**|   退出mysql。
| rehash|\\#|   在输入语句时，重建完成哈希，以使数据库，表和列名称补全。 （请参阅--auto-rehash选项的说明。）|
| resetconnection| \\x| 重置连接以清除会话状态。|
|source file_name, |\\. file_name|  读取命名文件并执行其中包含的语句。在Windows上，您可以将路径名称分隔符指定为/或\\。引用字符作为文件名本身的一部分。为获得最佳效果，名称不应包含空格字符。
 |status |\\s| 提供有关您正在使用的连接和服务器的状态信息。如果您在启用了--safe-updates的情况下运行，则status还会打印影响查询的mysql变量的值。
 |system command |\\! command| 使用默认命令解释程序执行给定命令。system命令仅在Unix中有效。|
 | tee [file_name]| \\T [file_name]|通过在调用mysql时使用--tee选项，您可以记录语句及其输出。 屏幕上显示的所有数据都会附加到给定文件中。 这对于调试目的也非常有用。 mysql在每个语句之后将结果刷新到文件，就在它打印下一个提示之前。 Tee功能仅在交互模式下有效。 您可以使用tee命令以交互方式启用此功能。 如果没有参数，则使用先前的文件。 可以使用notee命令禁用tee文件。 再次执行tee重新启用日志记录。
 |**use db_name**|\\u db_name|    使用db_name作为默认数据库。|
 |warnings|\\W|  在每个语句后启用警告显示（如果有）。

# 4 mysql服务器管理
* 启动
使用 service 启动：`service mysqld start`
使用 mysqld 启动：`/etc/inint.d/mysqld start `

*  停止
使用 service 启动：`service mysqld stop`
使用 mysqld 脚本启动：`/etc/inint.d/mysqld stop`

*  重启
使用 service 启动：service mysqld restart
使用 mysqld  脚本启动：/etc/inint.d/mysqld restart

*  查看状态
使用 service 启动：service mysqld status
使用 mysqld  脚本启动：/etc/inint.d/mysqld status


# 5 基本数据类型

|数值类型|	说明|
|:--|:--|
|BIT(M)|位类型。M指定位数，默认值1，范围1-64
|BOOL，BOOLEAN| 使用0表示假，非0表示真
|{TINYINT \| SMALLINT \|  MEDIUMINT \| INT \| BIGINT } [(M)] [UNSIGNED] [ZEROFILL] |带符号的范围是-128到127。无符号0到255。
|DECIMAL \| DEC \| NUMERIC \| FIXED [(M[,D])] [UNSIGNED] [ZEROFILL]|精确的固定小数数字
|{FLOAT \| DOUBLE \|  REAL} [(M,D)] [UNSIGNED] [ZEROFILL] |	浮点数，M表示有效位数，D表示小数位数
|**文本、二进制**|**说明**|
|CHAR(M) \|  VARCHAR(M) |固定/可变长度字符串
|BINARY (M) \| VARBINARY(M)|固定/可变长度二进制字符串
|TINYTEXT \| TEXT[(M)] \|  MEDIUMTEXT \|  LONGTEXT |文本	
|TINYBLOB \| BLOB[(M)] \|  MEDIUMBLOB \|  LONGBLOB |二进制
|ENUM|枚举
|SET|集合
|**时间日期**|**说明**|
|YEAR|年
|DATE|日期
|TIME|时间
|DATETIME|日期时间
|TimeStamp	|时间戳，它可用于自动记录操作时间


# 6  访问管理与权限控制
## 6.1 两步验证
* 第一步：根据用户（包括用户名和主机）和密码，查询mysql.user，若匹配则允许连接。
* 第二步：根据用户指令和作用对象，查询user、db、tables_priv、columns_priv、procs_priv、proxies_priv表，如果匹配则执行用户指令。



## 6.2 mysql提供的权限


|权限|作用|级别|
|:--|:--|:--|
|ALL|在指定访问级别的所有权限，除了 GRANT OPTION 和PROXY|
|ALTER|允许使用ALTER TABLE。|全局、数据库、表|
|ALTER ROUTINE|允许更改或删除已存储的程序。|全局、数据库、程序|
|CREATE|启用创建数据库和表。|全局、数据库、表|
|CREATE ROUTINE|启用创建已存储的程序。|全局，数据库|
|CREATE TABLESPACE|允许创建、修改或删除表空间和日志文件组。|全局|
|CREATE TEMPORARY TABLES|	允许使用CREATE TEMPORARY TABLE. |全局，数据库|
|CREATE USER|	允许使用CREATE USER, DROP USER, RENAME USER, and REVOKE ALL PRIVILEGES. |全局|
|CREATE VIEW|	允许视图被创建和修改 |全局、数据库、表|
|DELETE|	允许使用 DELETE.|全局、数据库、表|
|DROP|	允许删除数据库，表和视图|全局、数据库、表|
|EVENT	|允许时间调度器使用事件|全局，数据库|
|EXECUTE|允许用户执行已存储的程序|全局、数据库、程序|
|FILE|	允许用户让服务器输出或输入文件|全局|
|GRANT OPTION	|允许为其他账户授予或移除权限| 全局、数据库、表、程序、代理|
|INDEX|允许删除和创建索引 | 全局、数据库、表|
|INSERT|允许使用INSERT|	 全局、数据库、表、列|
|LOCK TABLES	|允许在具有SELECt权限的表上使用 LOCK TABLES|全局、数据库|
|PROCESS|	允许使用SHOW PROCESSLIST| 全局|
|PROXY|	允许用户代理|用户到用户|
|REFERENCES	|允许创建外键| 全局、数据库、表、列|
|RELOAD|	允许使用FLUSH 操作|全局|
|REPLICATION CLIENT|允许用户询问主或从服务器在哪里|全局|
|REPLICATION SLAVE|	允许复制从机从主服务器读取二进制日志的事件。|全局|
|SELECT	|允许使用SELECT|全局、数据库、表、列|
|SHOW DATABASES	|允许使用SHOW DATABASES|全局|
|SHOW VIEW	|允许使用SHOW CREATE VIEW.|全局、数据库、表
|SHUTDOWN	|允许使用mysqladmin shutdown| 全局|
|SUPER	|允许使用奇谭管理操作，如 CHANGE MASTER TO, KILL, PURGE BINARY LOGS, SET GLOBA,和 mysqladmin debug 命令|全局
|TRIGGER|允许触发器操作|	全局、数据库、表|
|UPDATE|	允许使用UPDATE| 全局、数据库、表、列|
|USAGE|没有任何权限|



>* 全局级：\*.\*，全局权限， 存放在mysql.user
>* 数据库级：db_name.\* 或 \*  ,会被全局级覆盖, 存放在mysql.db。
>* 表级：db_name.table_name，能被全局和数据库级覆盖, 存放在mysql.tables_priv
>* 列级：privileges(i)，能被全局、数据库、表级覆盖, 存放在mysql.columns_priv
>* 程序级：主要针对函数和过程，能被全局，数据库级，表级覆盖, 存放在mysql.procs_priv
>* 代理级： 针对代理, 存放在mysql.proxies_priv



|全局|数据库|表|列|程序|代理|用户对用户|
|:--|:--|:--|:--|:--|:--|:--|
|ALTER|ALTER|ALTER|||
|ALTER ROUTINE|ALTER ROUTINE|||ALTER ROUTINE||
|CREATE|CREATE|CREATE||||
|CREATE ROUTINE|CREATE ROUTINE|||||
|CREATE TABLESPACE||||||
|CREATE TEMPORARY TABLES|CREATE TEMPORARY TABLES|||||
|CREATE USER||||||
|CREATE VIEW|CREATE VIEW|CREATE VIEW||||
|DELETE|DELETE|DELETE||||
|DROP|DROP|DROP||||
|EVENT|EVENT|||||
|EXECUTE|EXECUTE|||EXECUTE||
|FILE||||||
|GRANT OPTION|GRANT OPTION|GRANT OPTION||GRANT OPTION|GRANT OPTION|
|INDEX|INDEX|INDEX||||
|INSERT|INSERT|INSERT|INSERT|||
|LOCK TABLES|LOCK TABLES|||||
|PROCESS||||||
|||||||PROXY|
|REFERENCES|REFERENCES|REFERENCES|REFERENCES|||
|RELOAD||||||
|REPLICATION CLIENT||||||
|REPLICATION SLAVE||||||
|SELECT|SELECT|SELECT|SELECT|||
|SHOW DATABASES||||||
|SHOW VIEW|SHOW VIEW|SHOW VIEW||||
|SHUTDOWN||||||
|SUPER||||||
|TRIGGER|TRIGGER|TRIGGER||||
|UPDATE|UPDATE|UPDATE|UPDATE|||

>补充：
>* mysql中 % 表示任意个字符，_ 表示一个字符
>* 表示对数据库、表、列、程序等的通配。
# 7 基本SQL语句
## 7.1 数据库管理语句 
### 7.1.1 账户管理语句

|账户管理语句|描述|
|:--|:--|
**CREATE USER**|创建帐户及其身份验证，SSL/TLS，资源限制和密码管理属性，并控制帐户最初是锁定还是解锁。|
|**DROP USER**|删除一个或多个MySQL帐户及其特权。它从所有授予表中删除这些帐户的特权行。|
|ALTER USER|修改了帐户及其身份验证，SSL/TLS，资源限制和密码管理属性。它还可用于锁定和解锁帐户。|
|RENAME USER|重命名现有MySQL帐户。对于不存在的旧帐户或已经存在的新帐户，将发生错误。|
|**GRANT**|授予帐户权限，包括全局、数据库、表、列、路径、代理的权限。如果用户不存在则建立用户。|
|**REVOKE**|撤消MySQL帐户的权限。
|**SET PASSWORD** |为指定账户设置密码
### 7.1.2 表维护语句
|SQL语句|描述|
|:--|:--|
| ANALYZE TABLE |执行密钥分发分析，存储表分布。对 MYISA表，等效于myisamchk --analyze.。
|CHECK TABLE |检查一个或多个表是否有错误，视图是否有问题。
|  CHECKSUM TABLE|报告表的内容的校验和。
| OPTIMIZE TABLE|重新组织表数据的物理存储，重新关联索引，以节约存储和提升IO效率。
| REPAIR TABLE |修复可能损坏的表，仅用于某些存储引擎。|
### 7.1.3 插件和自定义函数语句
|SQL语句|描述|
|:--|:--|
|CREATE FUNCTION|创建自定义函数
|DROP FUNCTION|删除用户定义函数
|INSTALL PLUGIN|安装服务器插件
|UNINSTALL PLUGIN|删除已安装的服务器插件

### 7.1.4 SET语句
|SQL语句|描述|
|:--|:--|
|  **SET**|变量赋值|
|  **SET CHARACTER SET**|设置字符集|
|  SET NAMES|设置字符集 |

### 7.1.5 SHOW语句
|SQL语句|描述|
|:--|:--|
|SHOW BINARY | MASTER LOGS|列出服务器上的二进制日志文件。|
| SHOW BINLOG EVENTS|显示二进制日志中的事件。|
| **SHOW CHARACTER SET** |显示所有可用的字符集。|
| **SHOW COLLATION**|列出了服务器支持的排序集。|
| **SHOW COLUMNS**|显示给定表的列信息|
|SHOW CREATE DATABASE|显示创建指定数据库的语句。|
|SHOW CREATE EVENT|显示创建给定事件的语句。|
| SHOW CREATE FUNCTION|显示创建已存储的函数的语句|
| SHOW CREATE PROCEDURE |显示创建已存储的过程的语句。|
|SHOW CREATE TABLE|显示创建指定数据表的语句。|
| SHOW CREATE TRIGGER |显示创建指定触发器的语句。|
|  SHOW CREATE USER|显示创建指定账户的语句。|
| SHOW CREATE VIEW |显示创建指定视图的语句。|
| **SHOW DATABASES** |显示所有的数据库。|
| SHOW ENGINE|显示有关存储引擎的操作信息|
|SHOW ENGINES|显示有关服务器存储引擎的状态信息
| SHOW ERRORS|显示错误信息|
|SHOW EVENTS|显示关于事件管理器事件的信息|
| SHOW FUNCTION CODE|显示已存储的函数的内部实现。|
| SHOW FUNCTION STATUS|显示已存储的函数的状态。|
|SHOW GRANTS|显示分配给MySQL用户帐户的特权|
|**SHOW INDEX**|显示表的的索引信息|
|SHOW MASTER STATUS |显示主机二进文件中的状态信息|
|SHOW OPEN TABLES|显示当前在表缓存中打开的非临时表。|
|SHOW PLUGINS|显示关于服务器插件的信息。|
|**SHOW PRIVILEGES**|显示MySQL服务器支持的系统特权列表。|
|SHOW PROCEDURE CODE|显示已存储的过程的内部实现。|
|SHOW PROCEDURE STATUS|显示已存储的过程的状态。|
|SHOW PROCESSLIST|显示正在运行的线程。|
|SHOW PROFILE |显示分析信息，指示在当前会话过程中执行的语句的资源使用情况。
|SHOW PROFILES |显示分析信息，指示在当前会话过程中执行的语句的资源使用情况。
|SHOW RELAYLOG EVENTS|显示复制从站的接替日志中的事件。
|SHOW SLAVE HOSTS|显示当前在主服务器上注册的复制从服务器的列表。
|SHOW SLAVE STATUS|显示关于从线程的关键参数的状态信息。
|SHOW STATUS|显示状态信息
|SHOW TABLE STATUS|类似于SHOW TABLES，但是提供了关于每个非临时表的大量信息。
|**SHOW TABLES**|列出给定数据库中的非临时表。
|SHOW TRIGGERS|列出数据库中的表定义的触发器。
|SHOW VARIABLES|显示MySQL系统变量的值
|SHOW WARNINGS|显示有关在当前会话中执行语句所产生的错误，警告和注释的信息。
### 7.1.6 其他管理语句
|SQL语句|描述|
|:--|:--|
|BINLOG|一个内部使用声明。它由mysqlbinlog 程序生成，作为二进制日志文件中某些事件的可打印表示。
|CACHE INDEX|CACHE INDEX语句将表索引分配给特定的键缓存。只用于MYISAM表。
|FLUSH|有几种变体形式，用于清除或重新加载各种内部高速缓存，刷新表或获取锁。|
|**KILL**|与mysqld的每个连接都在一个单独的线程中运行。你可以用语句杀死一个线程。|
|LOAD INDEX INTO CACHE|将表索引预加载到由CACHE INDEX语句分配给它的键缓存中，否则加载到默认键缓存中。|
|RESET |用于清除各种服务器操作的状态。|
|SHUTDOWN|停止MySQL服务器。|

## 7.2 数据定义语句
|SQL语句|描述|
|:--|:--|
|**{CREATE \| DROP \| ALTER} DATABASE**|创建 \| 删除 \| 更改 数据库|
|{CREATE \| DROP \| ALTER} EVENT |创建 \| 删除 \| 更改 事件|
|{CREATE \| DROP \| ALTER}  LOGFILE GROUP|创建 \| 删除 \| 更改 日志文件组|
|{CREATE \| DROP \| ALTER} SERVER|创建 \| 删除 \| 更改 用于FEDERATED存储引擎的服务器的定义|
|{CREATE \| DROP \| ALTER} TABLESPACE|创建 \| 删除 \| 更改 表空间|
|{CREATE \| DROP \| ALTER} VIEW|创建或替代 \| 删除 \| 更改 视图|
|{CREATE \| DROP \| ALTER}  FUNCTION|创建 \| 删除 \| 更改 已存储函数|
|**{CREATE \| DROP \| ALTER \| RENAME \| TRUNCATE}  TABLE**|创建 \| 删除 \| 更改 \| 重命名 \| 清空 表|
|**{CREATE \| DROP }  INDEX**|创建 \| 删除 索引|
|**{CREATE \| DROP }  TRIGGER**|创建 \| 删除 触发器|
|{CREATE \| DROP}  {PROCEDURE \| FUNCTION}|创建 \| 删除已存储过程|
|ALTER PROCEDURE|修改已存储过程|
|ALTER INSTANCE|旋转用于InnoDB表空间加密的主加密密钥|



## 7.3 数据操作语句
|SQL语句|描述|
|:--|:--|
|**INSERT**|新行插入现有表中|
|**DELETE**|从表中删除行并返回已删除行的数量|
|**UPDATE**|修改表中的行|
|**REPLACE**|替换表中的行|
|CALL|调用先前定义的已存储的过程|
|DO|行表达式但不返回任何结果。|
| HANDLER|提供对表存储引擎接口的直接访问。它适用于 InnoDB和MyISAM表。|
|LOAD DATA|以非常快的速度将文本文件中的行读入表|
|LOAD XML|将数据从XML文件读入表。|
|**SELECT** |查询表|

## 7.4 实用程序语句
* **DESCRIBE | DESC**
获取表结构

* EXPLAIN 
获取查询、执行计划的详细信息
* HELP
返回MySQL参考手册中的在线帮助信息
* **USE**
切换当前数据库





------


&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《MYSQ参考手册5.7》](https://dev.mysql.com/doc/refman/5.7/en/)。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>


------
