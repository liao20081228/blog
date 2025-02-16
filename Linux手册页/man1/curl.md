---
title: curl
tags: http,man1
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[curl manpages](https://curl.se/docs/manpage.html)》。</font>curl的版本为8.12.0，手册更新时间为2025-02。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------

# 名字

curl--传输一个URL

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
```
--variable '%USER'
--expand-url = "https://example.com/api/{{USER}}/method"
```
展开变量时，curl 支持一组函数，可使变量内容更易于使用。它可以使用“<u>trim</u>”修剪前导和尾随空格，可以使用<u>“json</u>”将内容输出为 JSON 引号字符串，使用“<u>url</u>”对字符串进行 URL 编码或使用“<u>b64</u>”对其进行 base64 编码。要将函数应用于变量展开，请将它们以冒号分隔添加到变量的右侧。展开时变量内容包含未编码的空字节会导致错误。

示例：将名为 \$HOME/.secret 的文件的内容放入名为“fix”的变量中。确保在以 POST 数据形式发送时内容经过修剪和百分比编码：
```
--variable %HOME
--expand-variable fix@{{HOME}}/.secret
--expand-data "{{fix:trim:url}}"
https://example.com/
```
命令行变量和展开已在 8.3.0 中添加。

# 输出

如果没有其他指示，curl 会将接收到的数据写入 stdout。可以使用 <u>-o、--output</u> 或<u> -O、--remote-name</u> 选项指示将数据保存到本地文件中。如果在命令行上为 curl 提供了多个要传输的 URL，它同样需要多个选项来指定保存位置。

curl 不会解析或以其他方式“理解”它获取或写入到输出的内容。它不进行编码或解码，除非使用专用命令行选项明确要求。

# 协议
curl 支持多种协议，或者用 URL 术语来说：方案。您的特定版本可能不支持所有协议。

DICT:
: 允许您使用在线词典查找单词。

FILE
: 读取或写入本地文件。curl 不支持远程访问 file:// URL，但在 Microsoft Windows 上运行时，使用本机 UNC 方法可以工作。

FTP(S)
: curl 支持文件传输协议，并进行了大量调整。使用或不使用 TLS 均可。

GOPHER(S)
: 检索文件。

HTTP(S)
: curl 支持 HTTP，并有大量选项和变体。它可以使用 HTTP 版本 0.9、1.0、1.1、2 和 3，具体取决于构建选项和正确的命令行选项。

IMAP(S)
: 使用邮件阅读协议，curl 可以为您下载电子邮件。使用或不使用 TLS 均可。

LDAP(S)
: curl 可以为您执行目录查找，使用或不使用 TLS 均可。

MQTT
: curl 支持 MQTT V3。通过 MQTT 下载相当于订阅主题，而上传/发布相当于发布主题。目前尚不支持 基于 TLS 的 MQTT。

POP3(S)
: 从 pop3 服务器下载意味着收到邮件。使用或不使用 TLS 均可。

RTMP(S)
: 实时消息协议主要用于提供流媒体，curl 可以下载。

RTSP
: curl 支持 RTSP 1.0 下载。

SCP

: curl 支持 SSH 2.0 scp 传输。

SFTP

: curl 支持通过 SSH 2.0 完成的 SFTP（草案 5）。

SMB(S)

: curl 支持 SMB 版本 1 进行上传和下载。

SMTP(S)

: 将内容上传到 SMTP 服务器意味着发送电子邮件。使用或不使用 TLS 均可。

TELNET

: 获取 telnet URL 会启动一个交互式会话，它会在 stdin 上发送读取的内容并输出服务器发送的内容。

TFTP

: curl 可以进行 TFTP 下载和上传。

WS(S)

: WebSocket 基于 HTTP/1。WSS 意味着它基于 HTTPS 工作。

# 进度计
curl 通常在操作期间显示进度计、条，指示传输的数据量、传输速度和估计剩余时间等。进度计显示每秒的传输速率（字节数）。后缀（k、M、G、T、P）以 1024 为基数。例如，1k 表示 1024 字节。1M 表示 1048576 字节。

curl 默认将此数据显示到终端，因此如果您调用 curl 执行操作并且即将将数据写入终端，它会禁用进度计，否则会将进度计和响应数据混合在一起，弄乱输出。

如果您想要 HTTP POST 或 PUT 请求的进度条，则需要使用 shell 重定向 (>)、<u>-o</u>、<u>--output</u> 或类似命令将响应输出重定向到文件。

这不适用于 FTP 上传，因为该操作不会向终端吐出任何响应数据。

如果您更喜欢进度条而不是常规仪表，<u>-#、--progress-bar</u> 是您的好帮手。您还可以使用 <u>-s, --silent</u> 选项完全禁用进度计。

# 版本
此手册页描述了 curl 8.12.2。如果您使用的是更高版本，则此手册页可能未完整记录它。如果您使用的是早期版本，本文档将尝试包含有关引入更改的特定版本的版本信息。

您始终可以通过运行以下命令了解最新的 curl 版本
```
curl https://curl.se/info
```
此手册页的在线版本始终显示最新版本：<https://curl.se/docs/manpage.html>


# 选项



------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[curl manpages](https://curl.se/docs/manpage.html)》。</font>curl的版本为8.12.0，手册更新时间为2025-02。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------