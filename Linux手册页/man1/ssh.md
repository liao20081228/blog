---
title: ssh
tags: 一般命令,man1
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《ssh manpages》。</font>ssh的版本为9.6，手册更新时间为2023-10。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------
# 名称
ssh——OpenSSH 远程登录客户端

# 概览

**ssh** \[-46AaCfGgKkMNnqsTtVvXxYy] \[-B <u>bind_interface</u>] \[-b <u>bind_address</u>] \[-c <u>cipher_spec</u>] \[-D \[<u>bind_address</u>:]<u>port</u>] \[-E <u>log_file</u>] \[-e <u>escape_char</u>] \[-F <u>configfile</u>] \[-I <u>pkcs11</u>] \[-i <u>identity_file</u>] \[-J <u>destination</u>] \[-L <u>address</u>] \[-l <u>login_name</u>] \[-m <u>mac_spec</u>] \[-O <u>ctl_cmd</u>] \[-o <u>option</u>] \[-P <u>tag</u>] \[-p <u>port</u>] \[-R <u>address</u>] \[-S <u>ctl_path</u>] \[-W <u>host</u>:<u>port</u>] \[-w <u>local_tun</u>\[:<u>remote_tun</u>]] <u>destination</u> \[<u>command</u> \[<u>argument</u> <u>...</u>]]
		   
**ssh** \[-Q <u>query_option</u>]

# 描述
SSH（SSH客户端）是一个用于登录远程计算机并在远程计算机上执行命令的程序。它旨在通过不安全的网络在两个不可信的主机之间提供安全的加密通信。X11连接、任意TCP端口和Unix域套接字也可以通过安全通道进行转发。

ssh会连接到指定的<u>目标</u>并登录，目标可以指定为\[user@]hostname或ssh\://\[user@]hostname\[:port]形式的URI。用户必须使用以下几种方法之一向远程计算机证明自己的身份（见下文）。

如果指定了<u>命令</u>，则该命令将在远程主机上执行，而不是启动登录shell。<u>命令</u>可以指定为完整的命令行，也可以包含额外的参数。如果提供了这些参数，它们将被附加到命令后面，用空格分隔，然后一起发送给服务器执行。

选项如下：

- **-4**

# 作者


# 报告bugs

# 版权



# 另见




------


***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《ssh manpages》。</font>ssh的版本为9.6，手册更新时间为2023-10。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***


------