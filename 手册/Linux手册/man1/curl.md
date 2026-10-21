---
title: curl
tags: man1
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《cp manpages》。</font>cp的版本为8.18.0，手册更新时间为2026-10。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------
# 名称
curl——传输一个URL

# 概览

**curl** \[OPTION / URLS]



# 描述
**curl** 是一个使用 URL 从服务器传输数据或向服务器传输数据的工具。它支持以下协议：DICT、FILE、FTP、FTPS、GOPHER、GOPHERS、HTTP、HTTPS、IMAP、IMAPS、LDAP、LDAPS、MQTT、POP3、POP3S、RTMP、RTMPS、RTSP、SCP、SFTP、SMB、SMBS、SMTP、SMTPS、TELNET、TFTP、WS 和 WSS。

**curl** 的所有传输相关功能均由 libcurl 提供支持。详情请参见 <u>libcurl(3)</u>。


# URL
URL 语法取决于协议。详细说明可在 RFC 3986 中找到。

如果你提供的 URL 没有开头的 `protocol:// scheme`，curl 会猜测你想要的协议。它默认使用 HTTP，但也会根据常用的主机名前缀推断其他协议。例如，对于以 "ftp." 开头的主机名，curl 会假定你想要使用 FTP。

你可以在命令行中指定任意数量的 URL。除非使用 <u>--parallel</u>，否则它们会按照指定顺序依次获取。你可以在命令行中以任意顺序混合指定命令行选项和 URL。

curl 在进行多次传输时会尝试复用连接，这样从同一服务器获取许多文件时就不会使用多次连接和设置握手。这可以提高速度。连接复用只能用于单次命令行调用中指定的 URL，不能在彼此独立的 curl 运行之间进行。

在 URL 中提供 IPv6 区域 ID 时，要使用转义的百分号。例如：`http://[fe80::3%25eth0]/`

命令行中提供的任何不是命令行选项或其参数的内容，curl 都会假定它是一个 URL，并按 URL 处理。

# 通配符
你可以在花括号中写列表，或在方括号中写范围，来指定多个 URL 或 URL 的多个部分。我们称之为“globbing”（通配展开）。

像这样提供一个包含三个不同名称的列表：

`https://fun.example/{one,two,three}.jpg`
`http://sftp://{one,two,three}.example/README`

使用 `[]` 来表示字母数字序列，如下所示：

`ftp://ftp.example.com/file[1-100].txt`

带前导零：

`ftp://ftp.example.com/file[001-100].txt`

使用字母表中的字母：

`ftp://ftp.example.com/file[a-z].txt`

不支持嵌套序列，但你可以将多个序列并排使用：

`https://example.com/archive[1996-1999]/vol[1-4]/part{a,b,c}.html`

你可以为范围指定步长计数器，以获取每第 N 个数字或字母：

`https://example.com/file[1-100:10].txt`

`https://example.com/file[a-z:2].txt`

在命令行提示符下使用 `[]` 或 `{}` 序列时，你可能需要将完整 URL 放在双引号内，以避免 shell 对其进行干扰。对于其他被视为特殊的字符也是如此，例如 `'&'`、`'?'` 和 `'*'`。

使用 `--globoff` 可以关闭 globbing。

# 变量
curl 支持命令行变量（自 8.3.0 起新增）。使用 `--variable name=content` 或 `--variable name@file` 设置变量（其中 `file` 若设为单个短横线 `-`，则可以是标准输入）。

如果选项名以 `--expand-` 为前缀，则可以在选项参数中使用 `{{name}}` 展开变量内容。这会插入变量 `name` 的内容；如果该名称不作为变量存在，则插入空白。要在字符串中原样插入 `{{`，请在其前面加反斜杠，例如 `\{{`。

要访问和展开环境变量，需先导入它们。你可以选择要求环境变量必须已设置，或者在其尚未设置时提供默认值。单纯的 `--variable %name` 会导入名为 `name` 的变量，但如果该环境变量尚未设置，则会以错误退出。若要在未设置时提供默认值，请使用 `--variable %name=content` 或 `--variable %name@content`。

示例：将 `USER` 环境变量放入 URL，如果 `USER` 未设置则失败：

`--variable '%USER'`
`--expand-url = "https://example.com/api/{{USER}}/method"`

展开变量时，curl 支持一组函数，可使变量内容更便于使用。它可以用 `trim` 去除首尾空白，用 `json` 将内容输出为 JSON 引号字符串，用 `url` 对字符串进行 URL 编码，用 `b64` 进行 base64 编码，用 `64dec` 进行 base64 解码。要对变量展开应用函数，请将它们以冒号分隔添加到变量右侧。变量内容若包含空字节且在展开时未编码，则会导致错误。

示例：将名为 `$HOME/.secret` 的文件内容读入名为 `fix` 的变量。确保内容在作为 POST 数据发送时已去除首尾空白并进行百分号编码：

`--variable %HOME`
`--expand-variable fix@{{HOME}}/.secret`
`--expand-data "{{fix:trim:url}}"`
`https://example.com/`

命令行变量和展开功能是在 8.3.0 中新增的。
# 输出
除非另有指示，curl 会将接收到的数据写入标准输出。可以使用 `--output` 或 `--remote-name` 选项指示它改为将数据保存到本地文件。如果在命令行上给 `curl` 指定多个要传输的 URL，它同样需要多个选项来指定各自的保存位置。

`curl` 不会解析或以其他方式“理解”它获取或作为输出写出的内容。它不进行任何编码或解码，除非通过专用命令行选项明确要求。

# 作者
Torbjorn Granlund、David MacKenzie、Jim Meyering.。

# 报告bugs
GNU coreutils联机帮助：<https://www.gnu.org/software/coreutils/>
将转换bugs报告给<https://translationproject.org/team/>

# 版权
版权所有©2022自由软件基金会公司。 许可证GPLv3+: GNU GPL第3版或更高版本<https://gnu.org/licenses/gpl.html>。

这是自由软件：您可以自由更改和重新分发它。在法律允许的范围内，不提供任何保证。

# 另见

完整文档位于：<https://www.gnu.org/software/coreutils/cp>或通过以下方式在本地获得：`info '(coreutils) cp invocation'`。


------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《cp manpages》。</font>cp的版本为9.1，手册更新时间为2023-01。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------