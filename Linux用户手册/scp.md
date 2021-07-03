---
title:  scp
tags: BSD普通命令, 目录文件管理
---


------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>《scp manpages》</font>， scp当前版本为version，手册更新时间为2019-11-30。<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

# 1 名称

**scp**——OpenSSH安全文件传输。

# 2 概要

``` bash
scp [-346BCpqrTv] [-c cipher] [-F ssh_config] [-i identity_file] [-J destination] [-l limit] [-o ssh_option] [-P port] [-S program] source ... target
```
# 3 说明

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
|**-F** <u>ssh_config</u>|为 ssh 指定替代的每用户配置文件。此选项直接传递给 ssh(1)。|
|**-i** <u>identity_file</u>|选择从中读取用于公钥认证的身份（私钥）的文件。此选项直接传递给 ssh(1)。|
|**-J** <u>destination</u>|
|**-l** <u>limit</u>|
|**-o** <u>ssh_option</u>|
|**-P** port|
|**-p**|
|**-q**|
|**-r** |
|**-S** program|
|**-T**|
|**-v** |
|&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;||






------


***<font color=blue>版权声明</font>：本文翻译自<font color=blue>《scp manpages》</font>， scp当前版本为version，手册更新时间为2019-11-30。<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
