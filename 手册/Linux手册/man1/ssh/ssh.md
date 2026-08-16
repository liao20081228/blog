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

ssh会连接到指定的<u>destination</u>并登录，目标可以指定为\[user@]hostname或ssh\://\[user@]hostname\[:port]形式的URI。用户必须使用以下几种方法之一向远程计算机证明自己的身份（见下文）。

如果指定了<u>command</u>，则该命令将在远程主机上执行，而不是启动登录shell。<u>命令</u>可以指定为完整的命令行，也可以包含额外的参数。如果提供了这些参数，它们将被附加到命令后面，用空格分隔，然后一起发送给服务器执行。

选项如下：

- **-4**
强制SSH仅使用IPv4地址。 

- **-6**
强制SSH仅使用IPv6地址。 

- **-A**
启用从认证代理（如<u>ssh-agent</u>(1)）转发连接的功能。此选项也可以在配置文件中针对每个主机单独指定。 

	启用代理转发时应谨慎。如果用户有能力绕过远程主机上（用于代理Unix域套接字）的文件权限限制，他们可以通过转发连接访问本地代理。虽然攻击者无法从代理中获取密钥材料，但他们可以对密钥进行操作，从而利用加载到代理中的身份进行认证。更安全的选择可能是使用跳板主机（参见 **-J** 选项）。 

- **-a** 
禁用认证代理连接的转发功能。

- **-B** <u>bind_interface</u>
在尝试连接到目标主机之前，绑定到<u>bind_interface</u>>的地址。此操作仅在拥有多个地址的系统上才有用。

- **-b** <u>bind_address</u>
使用本地机器上的<u>bind_address</u>作为连接的源地址。仅在拥有多个地址的系统上才有用。

- **-C**
请求对所有数据（包括标准输入、标准输出、标准错误以及转发X11、TCP和Unix域连接的数据）进行压缩。所使用的压缩算法与<u>gzip</u>(1)相同。 在调制解调器线路和其他慢速连接上，压缩是有益的；但在快速网络上，压缩只会降低速度。默认值可以在配置文件中按主机逐个设置；请参阅<u>ssh_config</u>(5)中的Compression选项。 

- **-c** <u>cipher_spec</u>
选择用于加密会话的加密算法规范。<u>cipher_spec </u>是一个以逗号分隔的加密算法列表，这些算法按优先顺序排列。有关更多信息，请参阅<u>ssh_config</u>(5)中的Ciphers关键字。

- **-D** \[<u>bind_address</u>:]<u>port</u>
指定本地“动态”应用层端口转发。其工作原理是分配一个套接字来监听本地<u>port</u>，可以选择性地绑定到指定的<u>bind_address</u>。每当有连接尝试连接到该端口时，连接将通过安全通道转发，然后应用程序协议用于确定从远程机器连接到何处。目前支持SOCKS4和SOCKS5协议，ssh将充当SOCKS服务器。只有root用户才能转发特权端口。动态端口转发也可以在配置文件中指定。

IPv6地址可以通过将地址用方括号括起来来指定。只有超级用户才能转发特权端口。默认情况下，本地端口的绑定方式遵循GatewayPorts设置。然而，也可以使用显式的绑定地址将连接绑定到特定地址。绑定地址为“localhost”表示该监听端口仅供本地使用，而空地址或“*”则表示该端口应从所有网络接口均可访问。







# 作者


# 报告bugs

# 版权



# 另见




------


***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《ssh manpages》。</font>ssh的版本为9.6，手册更新时间为2023-10。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***


------