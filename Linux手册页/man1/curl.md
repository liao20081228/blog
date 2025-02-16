---
title: curl
tags: http,man1
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[curl manpages](https://curl.se/docs/manpage.html)》。</font>curl的版本为8.12.0，手册更新时间为2025-02。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------

# 名字

curl--转换一个URL

# 总览

**curl \[options / URLs]**

# 说明

**curl** 是一种使用 URL 从服务器传输数据或向服务器传输数据的工具。它支持以下协议：DICT、FILE、FTP、FTPS、GOPHER、GOPHERS、HTTP、HTTPS、IMAP、IMAPS、LDAP、LDAPS、MQTT、POP3、POP3S、RTMP、RTMPS、RTSP、SCP、SFTP、SMB、SMBS、SMTP、SMTPS、TELNET、TFTP、WS 和 WSS。

curl 由 libcurl 支持所有与传输相关的功能。有关详细信息，请参阅 <u>libcurl</u>(3)。

# URL

URL 语法与协议相关。您可以在 RFC 3986 中找到详细说明。

如果您提供的 URL 没有以 **protocol://** 开头，curl 会猜测您想要的协议。然后它会默认为 HTTP，但会根据常用的主机名前缀假设其他协议。例如，对于以“ftp.”开头的主机名，curl 会假设您想要 FTP。

您可以在命令行上指定任意数量的 URL。除非您使用<u> -Z、--parallel</u>，否则它们会按指定的顺序依次获取。您可以在命令行上以任意顺序混合指定命令行选项和 URL。

curl 会在进行多次传输时尝试重用连接，以便从同一服务器获取许多文件时不会使用多次连接和建立握手。这可以提高速度。连接重用只能针对为单个命令行调用指定的 URL 进行，并且不能在分开的 curl 运行之间执行。

在 URL 中提供带有转义百分号的 IPv6 区域 ID。例如
```
“http://[fe80::3%25eth0]/”
```
命令行中提供的所有内容（不是命令行选项或其参数）均被 curl 假定为 URL 并按 URL 进行处理。

# 通配符
您可以通过在括号内写列表或在方括号内写范围来指定多个 URL 或 URL 的部分内容。我们称之为“通配符”。

提供一个包含三个不同名称的列表，如下所示：
```
“http://site.{one,two,three}.com”
```
使用 \[] 执行字母数字序列，如下所示：
```
“ftp://ftp.example.com/file[1-100].txt”
```
带前导零：
```
“ftp://ftp.example.com/file[001-100].txt”
```
带字母表字母：
```
“ftp://ftp.example.com/file[a-z].txt”
```
不支持嵌套序列，但可以使用多个相邻的序列：
```
“http://example.com/archive[1996-1999]/vol[1-4]/part{a,b,c}.html”
```
可以为范围指定步进计数器以获取每第 N 个数字或字母：
```
“http://example.com/file[1-100:10].txt”

“http://example.com/file[a-z:2].txt”
```

从命令行提示符调用时使用 \[] 或 \{} 序列时，您可能必须将完整 URL 放在双引号内，以避免 shell 干扰它。这也适用于其他特殊处理的字符，例如“&”、“？”和“\*”。

使用 <u>-g、--globoff</u> 关闭通配符。

# 变量

curl 支持命令行变量（在 8.3.0 中添加）。使用 <u>--variable</u> name=content 或 <u>--variable </u>name@file 设置变量（其中“file”如果设置为单个破折号 (-)，则可以是 stdin）。

如果选项名称以“--expand-”为前缀，则可以使用“{{name}}”在选项参数中展开变量内容。这将插入变量“name”的内容，如果name不存在，则为空白。通过在字符串前面加上反斜杠，如“\{{，来逐字插入“{{”，”。

您可以通过首先导入环境变量来访问和展开它们。您可以选择要求设置环境变量，或者在尚未设置的情况下提供默认值。空白的“<u>--variable</u> %name”会导入名为“name”的变量，但如果尚未设置该环境变量，则会退出并出现错误。要提供默认值（如果未设置），请使用“<u>--variable</u> %name=content”或“<u>--variable</u> %name@content”。

示例。将 USER 环境变量放入 URL 中，如果未设置 USER，则失败：

--variable '%USER'
--expand-url = "https://example.com/api/{{USER}}/method"

扩展变量时，curl 支持一组函数，可使变量内容更易于使用。它可以使用“trim”修剪前导和尾随空格，可以使用“json”将内容输出为 JSON 引号字符串，使用“url”对字符串进行 URL 编码或使用“b64”对其进行 base64 编码。要将函数应用于变量扩展，请将它们以冒号分隔添加到变量的右侧。变量内容包含扩展时未编码的空字节会导致错误。

示例：将名为 $HOME/.secret 的文件的内容放入名为“fix”的变量中。确保在以 POST 数据形式发送时内容经过修剪和百分比编码：

--variable %HOME
--expand-variable fix@{{HOME}}/.secret
--expand-data "{{fix:trim:url}}"
https://example.com/
命令行变量和扩展已在 8.3.0 中添加。
------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[curl manpages](https://curl.se/docs/manpage.html)》。</font>curl的版本为8.12.0，手册更新时间为2025-02。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------