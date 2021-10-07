---
title:  scp
tags: BSD普通命令, 目录文件管理
---


------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>《scp manpages》</font>， scp当前版本为version，手册更新时间为2019-11-30。<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

# 名称

**scp**——OpenSSH安全文件传输。

# 总览

``` bash
scp [-346BCpqrTv] [-c cipher] [-F ssh_config] [-i identity_file] [-J destination] [-l limit] [-o ssh_option] [-P port] [-S program] source ... target
```
# 描述

**scp** 在网络上的主机之间复制文件。 它使用 ssh(1) 进行数据传输，并使用与 ssh(1) 相同的身份验证，并提供相同的安全性。 如果身份验证需要，**scp** 将要求输入密码或密码短语。

>译者注：password 密码， passcode纯数字密码。 passphrase长的密码。

源<u>source</u>和目标<u>target</u>可以指定为本地路径名、 具有可选路径的远程主机`[user@]host:[path]`或 URI`scp://[user@]host[:port][/path]`。 本地文件名可以使用绝对或相对路径名显示指定，以避免 **scp** 将包含“:”的文件名视为主机说明符。

在两个远程主机之间进行拷贝时，如果使用 URI 格式，如果使用 -3 选项，则只能在目标<u>target</u>上指定端口<u>port</u>。

选项如下：

|选项|说明|
|:--|:--|
|**-3**|两个远程主机之间的副本是通过本地主机传输。如果没有此选项，数据将直接在两个远程主机之间复制。请注意，此选项会禁用进度表。|
|**-4**|强制 **scp** 仅使用 IPv4 地址。|
|**-6**|强制 **scp** 仅使用 IPv6 地址。|
|**-B**|选择批处理模式（防止要求输入密码或密码）。|
|**-C**|允许压缩。将 **-C** 标志传递给 ssh(1) 以启用压缩。|
|**-c** <u>cipher</u>|选择用于加密数据传输的密码。此选项直接传递给 ssh(1)。|
|**-F** <u>ssh_config</u>|为 **ssh** 指定替代的每用户配置文件。此选项直接传递给 ssh(1)。|
|**-i** <u>identity_file</u>|指定 从中读取用于公钥认证的身份（私钥）的文件。此选项直接传递给 ssh(1)。|
|**-J** <u>destination</u>|通过如下方式连接到目标主机：首先与<u>destnation</u>描述的跳转主机建立一个**scp**连接，然后从那里建立到最终目标主机的TCP转发。 可以用逗号分隔来指定多个跳转。 这是指定 **ProxyJump** 配置指令的快捷方式。 此选项直接传递给 ssh(1)。|
|**-l** <u>limit</u>|限制使用的带宽，以 Kbit/s 为单位。|
|**-o** <u>ssh_option</u>|可用于以 ssh_config(5) 中使用的格式将选项传递给 **ssh**。 主要用于指定没有单独 scp 命令行标志的选项。 有关下面列出的选项及其可能值的完整详细信息，请参阅 ssh_config(5)。<br /> <br />AddressFamily<br />BatchMode<br />BindAddress<br />BindInterface<br />CanonicalDomains<br />CanonicalizeFallbackLocal<br />CanonicalizeHostname<br />CanonicalizeMaxDots<br />CanonicalizePermittedCNAMEs<br />CASignatureAlgorithms<br />CertificateFile<br />ChallengeResponseAuthentication<br />CheckHostIP<br />Ciphers<br />Compression<br />ConnectionAttempts<br />ConnectTimeout<br />ControlMaster<br />ControlPath<br />ControlPersist<br />GlobalKnownHostsFile<br />GSSAPIAuthentication<br />GSSAPIDelegateCredentials<br />HashKnownHosts<br />Host<br />HostbasedAuthentication<br />HostbasedKeyTypes<br />HostKeyAlgorithms<br />HostKeyAlias<br />Hostname<br />IdentitiesOnly<br />IdentityAgent<br />IdentityFile<br />IPQoS<br />KbdInteractiveAuthentication<br />KbdInteractiveDevices<br />KexAlgorithms<br />LogLevel<br />MACs<br />NoHostAuthenticationForLocalhost<br />NumberOfPasswordPrompts<br />PasswordAuthentication<br />PKCS11Provider<br />Port<br />PreferredAuthentications<br />ProxyCommand<br />ProxyJump<br />PubkeyAcceptedKeyTypes<br />PubkeyAuthentication<br />RekeyLimit<br />SendEnv<br />ServerAliveInterval<br />ServerAliveCountMax<br />SetEnv<br />StrictHostKeyChecking<br />TCPKeepAlive<br />UpdateHostKeys<br />User<br />UserKnownHostsFile<br />VerifyHostKeyDNS|
|**-P** <u>port</u>|指定要连接到的远程主机上的端口。请注意，此选项使用大写的“P”编写，因为 **-p** 已被保留用于保留文件的时间和模式。|
|**-p**|保留原始文件的修改时间、访问时间和模式。|
|**-q**| 安静模式：禁用进度表以及来自 ssh(1) 的警告和诊断消息。|
|**-r** |递归复制整个目录。请注意， **scp** 跟随目录树遍历中遇到的符号链接。（访问原文件而不是符号链接本身）|
|**-S** <u>program</u>|用于加密连接的程序名称。该程序必须了解 ssh(1) 选项。|
|**-T**|禁用严格的文件名检查。默认情况下，将文件从远程主机复制到本地目录时，**scp** 会检查接收到的文件名是否与命令行上请求的文件名匹配，以防止远程端发送不期望或不想要的文件。由于各种操作系统和 shell 解释文件名通配符的方式不同，这些检查可能会导致想要的文件被拒绝。此选项禁用这些检查，但代价是完全信任服务器不会发送不期望的文件。|
|**-v** |详细模式。使 **scp** 和 ssh(1) 打印有关其进度的调试消息。这有助于调试连接、身份验证和配置问题。|
|&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;||

# 退出状态
**scp** 实用程序在成功时退出状态 0，如果发生错误，则退出状态 >0。

# 另见
sftp(1)、ssh(1)、ssh-add(1)、ssh-agent(1)、ssh-keygen(1)、ssh_config(5)、sshd(8)

# 历史
**scp** 基于加州大学 Regents 的 BSD 源代码中的 rcp 程序。

# 作者
Timo Rinne <tri@iki.fi>
Tatu Ylonen <ylo@cs.hut.fi>

------


***<font color=blue>版权声明</font>：本文翻译自<font color=blue>《scp manpages》</font>， scp当前版本为version，手册更新时间为2019-11-30。<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
