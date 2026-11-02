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

curl 不会解析或以其他方式“理解”它获取或作为输出写出的内容。它不进行任何编码或解码，除非通过专用命令行选项明确要求。
# 协议
curl支持众多协议，或者用 URL 术语来说：schemes（方案）。你当前的具体构建版本可能并不支持全部这些协议。

- DICT：允许你使用在线词典查询单词。
- FILE：读取或写入本地文件。curl 不支持远程访问 `file://` URL，但在 Microsoft Windows 上运行时，使用原生 UNC 方式可以工作。仅支持绝对路径。
- FTP(S) ：curl 支持文件传输协议，并提供大量调整项和控制手段。可使用也可不使用 TLS。
- GOPHER(S)：获取文件。
- HTTP(S)：curl 支持 HTTP，并提供大量选项和变体。根据构建选项和正确的命令行选项，它可以使用 HTTP 0.9、1.0、1.1、2 和 3 版本。
- IMAP(S)：使用邮件读取协议，curl 可以为你下载电子邮件。可使用也可不使用 TLS。
- LDAP(S)：curl 可以为你进行目录查询，可使用也可不使用 TLS。
- MQTT：curl 支持 MQTT 版本 3。通过 MQTT 下载等同于订阅某个主题，而上传/发布等同于在某个主题上发布。目前尚不支持基于 TLS 的 MQTT。
- POP3(S)：从 POP3 服务器下载意味着获取一封电子邮件。可使用也可不使用 TLS。
- RTMP(S)：实时消息协议主要用于提供流媒体，curl 可以下载它。
- RTSP：curl 支持 RTSP 1.0 下载。
- SCP：curl 支持 SSH 版本 2 的 scp 传输。
- SFTP：curl 支持通过 SSH 版本 2 实现的 SFTP（草案 5）。
- SMB(S)： curl 支持 SMB 版本 1 的上传和下载。
- SMTP(S)：将内容上传到 SMTP 服务器意味着发送一封电子邮件。可使用也可不使用 TLS。
- TELNET： 获取 telnet URL 会启动一个交互式会话，它会将标准输入读取到的内容发送出去，并输出服务器发送给它的内容。
- TFTP：curl 可以进行 TFTP 下载和上传。
- WS(S)：通过 HTTP/1 实现 WebSocket。WSS 表示它通过 HTTPS 工作。

# 进度指示器

curl 在操作期间通常会显示一个进度指示器，显示已传输的数据量、传输速度以及预计剩余时间等。进度指示器以字节每秒为单位显示传输速率。所使用的后缀（“k”表示千、“M”表示兆、“G”表示吉、“T”表示太、“P”表示拍、“E”表示艾）均基于 1024。例如，1k 是 1024 字节，1M 是 1048576 字节。严格来说，这使得单位成为 kibibyte 和 mebibyte 等。

curl 默认将此数据显示到终端，因此如果你调用 curl 执行某个操作，而它即将把数据写入终端，它会禁用进度指示器，否则会将进度指示器和响应数据混在一起，扰乱输出。

如果你希望为 HTTP POST 或 PUT 请求显示进度指示器，需要将响应输出重定向到文件，使用 shell 重定向（`>`）、`--output` 或类似方式。

这不适用于 FTP 上传，因为该操作不会向终端输出任何响应数据。

如果你更喜欢进度条而不是常规指示器，`--progress-bar` 正合你意。你也可以使用 `--silent` 选项完全禁用进度指示器。

# 版本

本手册页描述的是 curl 8.18.0。如果你使用的是更高版本，本手册页很可能没有完整记录它。如果你使用的是更低版本，本文档会尽量包含关于是哪个具体版本引入了变更的版本信息。

你始终可以通过运行以下命令来了解最新的 `curl` 版本：

`curl https://curl.se/info`

本手册页的在线版本始终展示最新版本：<https://curl.se/docs/manpage.html>

# 选项
选项以一个或两个短横线开头。许多选项需要在其后附带一个值。如果提供的文本不以短横线开头，则它会被推定并当作 URL 处理。

选项的短“单短横线”形式（例如 `-d`）与其值之间可以有空格，也可以没有空格，不过建议使用空格作为分隔符。长双短横线形式（例如 `--data`）则要求它与值之间有一个空格。

不需要任何附加值的短选项可以紧挨着连写，例如你可以将 `-O`、`-L` 和 `-v` 这三个选项一次性写成 `-OLv`。

一般来说，所有布尔选项都通过 `--option` 启用，并通过 `--no-option` 再次禁用。也就是说，你使用相同的选项名，但在前面加上“no-”。不过，在本列表中，我们大多只列出并展示它们的 `--option` 版本。

使用 `--next` 时，它会重置解析器状态，你将以干净的选项状态重新开始，但全局选项除外。全局选项即使在 `--next` 之后也会保留其值和含义。

如果长选项名以等号（`=`）结尾，则参数就是其右侧紧随的文本。（自 8.16.0 新增）

第一个恰好为两个短横线（`--`）的参数会标记选项的结束；选项结束之后的任何参数都会被解释为 URL 参数，即使它以短横线开头。

curl 对命令行参数内容的验证很少甚至不做验证。传入诸如换行符这样的“创意字节”可能会触发意外结果。

以下选项是全局选项：`--fail-early`、`--libcurl`、`--parallel-immediate`、`--parallel-max-host`、`--parallel-max`、`--parallel`、`--progress-bar`、`--rate`、`--show-error`、`--stderr`、`--styled-output`、`--trace-ascii`、`--trace-config`、`--trace-ids`、`--trace-time`、`--trace` 和 `--verbose`。

# 所有选项
- <font color=darkcyan>--abstract-unix-socket</font > \<path\>
  （HTTP）通过抽象 Unix 域套接字连接服务器，而不是使用网络。注意：netstat 显示抽象套接字路径时会加上 "@" 前缀，但 \<path\> 参数不应包含这个前导字符。

  如果多次提供 <u>--abstract-unix-socket</u>，则使用最后设置的值。

  示例：
  `curl --abstract-unix-socket socketpath https://example.com`

  另请参阅 <u>--unix-socket</u>。

- <font color=darkcyan>--alt-svc</font > \<filename\>
  （HTTPS）启用 alt-svc 解析器。如果该文件名指向一个已存在的 alt-svc 缓存文件，则使用该文件。传输完成后，如果缓存文件已被修改，则再次将缓存保存到该文件名。

  指定空文件名（零长度）可避免加载/保存，并让 <u>curl</u> 仅在内存中处理缓存。

  你可能希望限制自己的 umask，以防止同一系统上的其他用户访问所创建的文件。

  如果多次使用此选项，<u>curl</u> 会从所有文件加载内容，但最后一个文件用于保存。

  <u>--alt-svc</u> 可以在命令行中多次使用。

  示例：
  `curl --alt-svc svc.txt https://example.com`

  另请参阅 <u>--resolve</u> 和 <u>--connect-to</u>。

- <font color=darkcyan>--anyauth</font >
  （HTTP）自动探测认证方法，并使用远程站点声称支持的最安全的方法。这是通过先发起请求并检查响应头来完成的，因此可能引发额外的网络往返。此选项用于代替设置特定认证方法，后者可通过 <u>--basic</u>、<u>--digest</u>、<u>--ntlm</u> 和 <u>--negotiate</u> 完成。

  如果从标准输入上传，不建议使用 <u>--anyauth</u>，因为它可能要求数据发送两次，且客户端必须能够回退。如果在从标准输入上传时出现这种需求，上传操作会失败。

  与 <u>--user</u> 一起使用。

  示例：
  `curl --anyauth --user me:pwd https://example.com`

  另请参阅 <u>--proxy-anyauth</u>、<u>--basic</u> 和 <u>--digest</u>。

- <font color=darkcyan>-a, --append</font >
  （FTP SFTP）在上传中使用时，此选项使 <u>curl</u> 追加到目标文件，而不是覆盖它。如果远程文件不存在，则会创建它。注意，某些 SFTP 服务器（包括 OpenSSH）会忽略此标志。

  多次提供 <u>--append</u> 没有额外效果。可使用 <u>--no-append</u> 再次禁用它。

  示例：
  `curl --upload-file local --append ftp://example.com/`

  另请参阅 <u>--range</u> 和 <u>--continue-at</u>。

- <font color=darkcyan>--aws-sigv4</font > \<provider1\[:prvdr2\[:reg\[:srv\]\]\]\>
  （HTTP）在传输中使用 AWS V4 签名认证。

  provider 参数是一个字符串，算法在创建传出认证头时会使用它。

  region 参数是一个字符串，当端点中省略区域名称时，它指向资源集合的地理区域（区域代码）。

  service 参数是一个字符串，当端点中省略服务名称时，它指向由云提供的功能（服务代码）。

  如果多次提供 <u>--aws-sigv4</u>，则使用最后设置的值。

  示例：
  `curl --aws-sigv4 "aws:amz:us-east-2:es" --user "key:secret" https://example.com`

  在 7.75.0 中添加。另请参阅 <u>--basic</u> 和 <u>--user</u>。

- <font color=darkcyan>--basic</font >
  （HTTP）使用 HTTP Basic 认证与远程主机通信。此方法是默认的，因此该选项通常没有意义，除非你使用它来覆盖先前设置的指定不同认证方法的选项（例如 <u>--ntlm</u>、<u>--digest</u> 或 <u>--negotiate</u>）。

  与 <u>--user</u> 配合使用。

  多次提供 <u>--basic</u> 没有额外效果。可使用 <u>--no-basic</u> 再次禁用它。

  示例：
  `curl -u name:password --basic https://example.com`

  另请参阅 <u>--proxy-basic</u>。

- <font color=darkcyan>--ca-native</font >
  （TLS）使用操作系统的原生 CA 存储进行证书验证。

  此选项独立于运行时或构建时设置的其他 CA 证书位置。除了原生 CA 存储之外，还会搜索那些位置。

  此选项适用于 Windows（在 7.71.0 中添加）上的 OpenSSL 及其分支（LibreSSL、BoringSSL 等），以及当 libcurl 在启用 Apple SecTrust 的情况下构建时的 Apple OS（在 8.17.0 中添加）。

  此选项在 Windows、Linux（Debian、Ubuntu、Gentoo、Fedora、RHEL）、macOS、Android 和 iOS 上适用于 wolfSSL。（在 8.3.0 中添加）

  此选项适用于 GnuTLS（在 8.5.0 中添加），并且当 libcurl 使用它构建时，也会使用 Apple SecTrust。（在 8.17.0 中添加）

  此选项在 Windows、macOS、Android 和 iOS 上适用于 Rustls。在 Linux 上，它等同于使用 Mozilla CA 证书包。与 Rustls 一起使用时，_仅_ 查询原生 CA 存储，而不是运行时或构建时设置的其他位置。（在 8.13.0 中添加）

  此选项目前对 Schannel 无效。这是来自 Microsoft 的原生 TLS 库，除非被 CA 证书位置设置覆盖，否则默认使用原生 CA 存储进行验证。

  多次提供 <u>--ca-native</u> 没有额外效果。可使用 <u>--no-ca-native</u> 再次禁用它。

  示例：
  `curl --ca-native https://example.com`

  在 8.2.0 中添加。另请参阅 <u>--cacert</u>、<u>--capath</u>、<u>--dump-ca-embed</u>、<u>--insecure</u> 和 <u>--proxy-ca-native</u>。

- <font color=darkcyan>--cacert</font > \<file\>
  （TLS）使用指定的证书文件来验证对端。该文件可以包含多个 CA 证书。证书必须为 PEM 格式。通常 curl 构建时会默认使用一个文件，因此此选项通常用于更改该默认文件。

  curl 识别名为 'CURL_CA_BUNDLE' 的环境变量（如果已设置且 TLS 后端不是 Schannel），并将给定的路径用作 CA 证书包的路径。此选项会覆盖该变量。

  （Windows）curl 会自动查找名为 'curl-ca-bundle.crt' 的 CA 证书文件，位置可以是与 curl.exe 相同的目录、当前工作目录，或者 PATH 中的任何文件夹。

  curl 8.11.0 添加了一个构建时选项来禁用此搜索行为，以及另一个将搜索限制在应用程序目录的选项。

  （Schannel）在 Windows 7 或更高版本中，Schannel 支持此选项（在 7.60.0 中添加）。此选项支持与其他 SSL 引擎的向后兼容性；相反，建议使用 Windows 的根证书存储（Schannel 的默认设置）。

  如果多次提供 <u>--cacert</u>，则使用最后设置的值。

  示例：
  `curl --cacert CA-file.txt https://example.com`

  另请参阅 <u>--capath</u>、<u>--dump-ca-embed</u> 和 <u>--insecure</u>。

- <font color=darkcyan>--capath</font > \<dir\>
  （TLS）使用指定的证书目录来验证对端。如果 curl 是针对 OpenSSL 构建的，可以通过使用适当的平台特定分隔符来提供多个路径（例如在 Unix 风格平台上为 "path1:path2:path3"，在 Windows 上为 "path1;path2;path3"）。

  证书必须为 PEM 格式，并且如果 curl 是针对 OpenSSL 构建的，该目录必须已使用 OpenSSL 提供的 c_rehash 实用程序处理过。如果 <u>--cacert</u> 文件包含许多 CA 证书，使用 <u>--capath</u> 可以让由 OpenSSL 驱动的 curl 比使用 <u>--cacert</u> 更高效地建立 SSL 连接。

  如果设置了此选项，则忽略默认的 capath 值。

  如果多次提供 <u>--capath</u>，则使用最后设置的值。

  示例：
  `curl --capath /local/directory https://example.com`

  另请参阅 <u>--cacert</u>、<u>--dump-ca-embed</u> 和 <u>--insecure</u>。

- <font color=darkcyan>-E, --cert</font > \<certificate\[:password\]\>
  （TLS）在使用 HTTPS、FTPS 或其他基于 SSL 的协议获取文件时，使用指定的客户端证书文件。证书必须为 PEM 格式。如果未指定可选密码，则会在终端上提示输入。请注意，此选项假定证书文件是私钥和客户端证书的连接。参见 <u>--cert</u> 和 <u>--key</u> 以分别指定它们。

  在参数的 \<certificate\> 部分中，您必须将字符 ":" 转义为 "\\:"，以免它被识别为密码分隔符。同样，您必须将双引号字符转义为 \\"，以免它被识别为转义字符。

  如果 curl 是针对 OpenSSL 构建的，并且 engine pkcs11 或 pkcs11 provider 可用，则可以使用 PKCS#11 URI（RFC 7512）来指定位于 PKCS#11 设备中的证书。以 "pkcs11:" 开头的字符串被解释为 PKCS#11 URI。如果提供了 PKCS#11 URI，则在未提供的情况下，将 <u>--engine</u> 选项设置为 "pkcs11"，并且在未提供的情况下，将 <u>--cert-type</u> 选项设置为 "ENG" 或 "PROV"（取决于 OpenSSL 版本）。

  如果 curl 是针对 GnuTLS 构建的，则可以使用 PKCS#11 URI 来指定位于 PKCS#11 设备中的证书。以 "pkcs11:" 开头的字符串被解释为 PKCS#11 URI。

  （Schannel）必须通过指向证书存储的路径表达式来指定客户端证书。（不支持加载 PFX；您可以先将其导入到存储中）。您可以使用 "\<store location\>\\\<store name\>\\\<thumbprint\>" 来引用系统证书存储中的证书，例如 "CurrentUser\MY\934a7ac6f8a5d579285a74fa61e19f23ddfe8d7a"。指纹通常是 SHA-1 十六进制字符串，您可以在证书详细信息中看到。支持以下存储位置：CurrentUser、LocalMachine、CurrentService、Services、CurrentUserGroupPolicy、LocalMachineGroupPolicy 和 LocalMachineEnterprise。

  如果多次提供 <u>--cert</u>，则使用最后设置的值。

  示例：
  `curl --cert certfile --key keyfile https://example.com`

  另请参阅 <u>--cert-type</u>、<u>--key</u> 和 <u>--key-type</u>。

- <font color=darkcyan>--cert-status</font >
  （TLS）通过使用证书状态请求（又名 OCSP 装订）TLS 扩展来验证服务器证书的状态。

  如果启用了此选项，并且服务器发送了无效的（例如过期的）响应，如果响应表明服务器证书已被撤销，或者完全没有收到响应，则验证失败。

  目前仅在 OpenSSL 和 GnuTLS 后端中实现了此支持。

  多次提供 <u>--cert-status</u> 没有额外效果。可使用 <u>--no-cert-status</u> 再次禁用它。

  示例：
  `curl --cert-status https://example.com`

  另请参阅 <u>--pinnedpubkey</u>。

- <font color=darkcyan>--cert-type</font > \<type\>
  （TLS）设置所提供客户端证书的类型。识别的类型有 PEM、DER、ENG、PROV 和 P12。

  默认类型取决于 TLS 后端，通常是 PEM。对于 Schannel，它是 P12。如果 <u>--cert</u> 是 pkcs11: URI，那么默认类型是 ENG 或 PROV（取决于 OpenSSL 版本）。

  如果多次提供 <u>--cert-type</u>，则使用最后设置的值。

  示例：
  `curl --cert-type PEM --cert file https://example.com`

  另请参阅 <u>--cert</u>、<u>--key</u> 和 <u>--key-type</u>。

- <font color=darkcyan>--ciphers</font > \<list\>
  （TLS）如果连接协商使用 TLS 1.2 (1.1, 1.0)，则指定要使用的密码套件。密码套件列表必须指定有效的密码。有关密码套件的详细信息，请阅读此 URL：

  https://curl.se/docs/ssl-ciphers.html

  如果多次提供 <u>--ciphers</u>，则使用最后设置的值。

  示例：
  `curl --ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256 https://example.com`

  另请参阅 <u>--tls13-ciphers</u>、<u>--proxy-ciphers</u> 和 <u>--curves</u>。

- <font color=darkcyan>--compressed</font >
  （HTTP）使用 <u>curl</u> 支持的算法之一请求压缩响应，并自动解压内容。

  保存时不会修改响应头，因此如果之后再次单独“解释”这些响应头，它们可能看起来仍表示内容是（仍然）压缩的；而实际上它已经被解压了。

  如果使用了此选项而服务器发送了不支持的编码，<u>curl</u> 会报告错误。这是一个请求，而不是命令；服务器可能提供也可能不提供压缩数据。

  警告：解压数据时，即使极小的传输也可能被扩展并生成大量的字节。

  多次提供 <u>--compressed</u> 没有额外效果。可使用 <u>--no-compressed</u> 再次禁用它。

  示例：
  `curl --compressed https://example.com`

  另请参阅 <u>--compressed-ssh</u>。

- <font color=darkcyan>--compressed-ssh</font >
  （SCP SFTP）启用 SSH 压缩。这是一个请求，而不是命令；服务器可能会也可能不会执行。这允许数据在网络上压缩发送，并在接收端自动解压，从而节省带宽。

  多次提供 <u>--compressed-ssh</u> 没有额外效果。可使用 <u>--no-compressed-ssh</u> 再次禁用它。

  示例：
  `curl --compressed-ssh sftp://example.com/`

  另请参阅 <u>--compressed</u>。

- <font color=darkcyan>-K, --config</font > \<file\>
  指定一个文本文件以从中读取 curl 参数。文本文件中找到的命令行参数将被视同于在命令行上提供的一样使用。

  选项及其参数必须在文件中的同一行指定，用空白、冒号或等号分隔。长选项名在配置文件中可以可选地不带初始的双短横线，如果这样，则可以使用冒号或等号字符作为分隔符。如果选项指定了单短横线或双短横线，则选项与其参数之间不能有冒号或等号字符。

  如果参数包含空白，或以冒号 (:) 或等号 (=) 开头，则必须将其包含在双引号内（"像这样"）。在双引号内，可以使用以下转义序列：`\\`、`\"`、`\t`、`\n`、`\r` 和 `\v`。任何其他字母前的反斜杠将被忽略。

  如果配置行的第一个非空白列是 '#' 字符，则该行被视为注释。

  在配置文件中，每个物理行只能写一个选项。单行要求不超过 10 兆字节（自 8.2.0 起）。

  将 <u>--config</u> 的文件名指定为减号 "-"，以使 curl 从标准输入读取文件。

  请注意，为了能够在配置文件中指定 URL，你需要使用 <u>--url</u> 选项来指定它，而不是仅在其单独的行上写入 URL。因此，它看起来可能类似于以下内容：

  ```text
  url = "https://curl.se/docs/"
  ```

  ```text
  # --- Example file ---
  # this is a comment
  url = "example.com"
  output = "curlhere.html"
  user-agent = "superagent/1.0"
  
  # and fetch another URL too
  url = "example.com/docs/manpage.html"
  -O
  referer = "http://nowhereatall.example.com/"
  # --- End of example file ---
  ```

  调用 curl 时，它会（除非使用了 <u>--disable</u>）检查默认配置文件并在找到时使用它，即使使用了 <u>--config</u>。默认配置文件的检查顺序如下：

  1) "$CURL_HOME/.curlrc"

  2) "$XDG_CONFIG_HOME/curlrc" (在 7.73.0 中添加)

  3) "$HOME/.curlrc"

  4) Windows: "%USERPROFILE%\.curlrc"

  5) Windows: "%APPDATA%\.curlrc"

  6) Windows: "%USERPROFILE%\Application Data\.curlrc"

  7) 非 Windows：使用 getpwuid 查找主目录

  8) 在 Windows 上，如果在上述序列中未找到 .curlrc 文件，它会在放置 curl 可执行文件的同一目录中检查。

  在 Windows 上，每个位置检查两个文件名：.curlrc 和 _curlrc，优先使用前者。Windows 上的旧版本仅检查 _curlrc。

  <u>--config</u> 可以在命令行中多次使用。

  示例：
  `curl --config file.txt https://example.com`

  另请参阅 <u>--disable</u>。

- <font color=darkcyan>--connect-timeout</font > \<seconds\>
  允许 curl 连接所花费的最长时间，以秒为单位。这仅限制连接阶段，因此如果 curl 在给定时间内连接成功，它会继续；否则会退出。

  此选项接受十进制值。十进制值需要使用点 (.) 作为小数分隔符提供——而不是本地化的版本，即使它可能使用另一个分隔符。

  当 DNS 查找以及请求的 TCP、TLS 或 QUIC 握手完成时，连接阶段被视为完成。

  如果多次提供 <u>--connect-timeout</u>，则使用最后设置的值。

  示例：
  `curl --connect-timeout 20 https://example.com`
  `curl --connect-timeout 3.14 https://example.com`

  另请参阅 <u>--max-time</u>。

- <font color=darkcyan>--connect-to</font > \<HOST1:PORT1:HOST2:PORT2\>
  对于发往 "HOST1:PORT1" 对的请求，改为连接到 "HOST2:PORT2"。此选项仅用于建立网络连接。它不影响用于 TLS/SSL（例如 SNI、证书验证）或应用协议的主机名/端口号。

  "HOST1" 和 "PORT1" 可以为空字符串，表示任意主机或任意端口号。"HOST2" 和 "PORT2" 也可以为空字符串，表示使用请求的原始主机名和端口号。

  为此选项指定的主机名作为字符串进行比较，因此它需要与请求 URL 中使用的名称匹配。它可以是数字形式（如 "127.0.0.1"）或完整主机名（如 "example.org"）。

  示例：无论端口号如何，将来自 example.com 主机名的连接重定向到 127.0.0.1：
  `curl --connect-to example.com::127.0.0.1: https://example.com/`

  示例：无论端口号如何，将所有主机名的连接重定向到 127.0.0.1：
  `curl --connect-to ::127.0.0.1: http://example.com/`

  <u>--connect-to</u> 可以在命令行中多次使用。

  示例：
  `curl --connect-to example.com:443:example.net:8443 https://example.com`

  另请参阅 <u>--resolve</u> 和 <u>--header</u>。

- <font color=darkcyan>-C, --continue-at</font > \<offset\>
  从给定的字节偏移量处继续先前的传输。给定的偏移量是精确的字节数，从源文件传输到目的地之前的开头计算起被跳过的确切字节数。如果与上传一起使用，curl 不会使用 FTP 服务器的 SIZE 命令。

  使用 `-C -` 指示 curl 自动找出从哪里/如何恢复传输。然后它会使用给定的输出/输入文件来推断出该信息。

  当将此选项用于使用 POST 或 PUT 的 HTTP 上传时，不保证功能正常。HTTP 协议没有标准的可互操作恢复上传机制，curl 为此使用了一组请求头，这组请求头曾经在某些服务器上被证明有效，并保留给那些觉得它有用的人。

  此命令行选项与 <u>--range</u> 互斥：单次传输只能使用其中之一。

  <u>--no-clobber</u> 和 <u>--remove-on-error</u> 选项不能与 <u>--continue-at</u> 一起使用。

  如果多次提供 <u>--continue-at</u>，则使用最后设置的值。

  示例：
  `curl -C - https://example.com`
  `curl -C 400 https://example.com`

  另请参阅 <u>--range</u>。

- <font color=darkcyan>-b, --cookie</font > \<data\|filename\>
  （HTTP）此选项有两个略有不同的 Cookie 发送功能。

  或者：在 Cookie 头中将要发送的精确数据传递给 HTTP 服务器。这应当是先前从服务器的 "Set-Cookie:" 行中接收到的数据。数据应采用 "NAME1=VALUE1; NAME2=VALUE2" 的格式。当给定一组特定的 Cookie 时，<u>curl</u> 会在所有传出请求中显式地将其 Cookie 头填充为此内容。如果由于认证、后续重定向或类似原因而执行多个请求，它们都会传递此 Cookie 头。

  或者：如果参数中未使用 "=" 符号，则将其视为读取先前存储的 Cookie 的文件名。此选项还会激活 Cookie 引擎，使 <u>curl</u> 记录传入的 Cookie。如果你将其与 <u>--location</u> 选项结合使用，或在同一次调用中执行多个 URL 传输，这可能会很方便。

  如果文件名是单个减号（"-"），<u>curl</u> 会从标准输入读取内容。如果文件名是空字符串（""）并且是唯一的 Cookie 输入，<u>curl</u> 会在没有任何 Cookie 的情况下激活 Cookie 引擎。

  读取 Cookie 的文件格式应为纯 HTTP 头（Set-Cookie 样式）或 Netscape/Mozilla Cookie 文件格式。

  使用 <u>--cookie</u> 指定的文件仅用作输入。不会将 Cookie 写入该文件。要存储 Cookie，请使用 <u>--cookie-jar</u> 选项。

  如果你使用 Set-Cookie 文件格式且未指定域名，则不会发送该 Cookie，因为域名永远无法匹配。要解决此问题，请在 Set-Cookie 行中设置域名（这样做会包含子域名），或者更好的做法是：使用 Netscape 格式。

  用户通常希望既从文件读取 Cookie，又将更新后的 Cookie 写回文件，因此在同一命令行中同时使用 <u>--cookie</u> 和 <u>--cookie-jar</u> 很常见。

  如果 <u>curl</u> 在构建时启用了 PSL（公共后缀列表）支持，它会检测并丢弃为不应拥有 Cookie 的此类后缀域名指定的 Cookie。如果 <u>curl</u> 构建时未启用 PSL 支持，则无法阻止超级 Cookie。

  <u>--cookie</u> 可以在命令行中多次使用。

  示例：
  `curl -b "" https://example.com`
  `curl -b cookiefile https://example.com`
  `curl -b cookiefile -c cookiefile https://example.com`
  `curl -b name=Jane https://example.com`

  另请参阅 <u>--cookie-jar</u> 和 <u>--junk-session-cookies</u>。

- <font color=darkcyan>-c, --cookie-jar</font > \<filename\>
  （HTTP）指定在操作完成后，你希望 <u>curl</u> 将所有 Cookie 写入哪个文件。<u>curl</u> 在操作结束时将其内存中的 Cookie 存储中的所有 Cookie 写入给定文件。即使已知没有 Cookie，也会创建一个文件，以从文件中移除任何以前存在的 Cookie。该文件使用 Netscape Cookie 文件格式。如果你将文件名设置为单个减号 "-"，则 Cookie 将写入标准输出。

  使用 <u>--cookie-jar</u> 指定的文件仅用于输出。不会从该文件读取 Cookie。要读取 Cookie，请使用 <u>--cookie</u> 选项。这两个选项可以指定同一个文件。

  此命令行选项激活 Cookie 引擎，使 <u>curl</u> 记录并使用 Cookie。<u>--cookie</u> 选项也会激活它。

  如果 Cookie 罐（cookie jar）无法创建或写入，整个 <u>curl</u> 操作不会失败，甚至不会清楚地报告错误。使用 <u>--verbose</u> 会显示一条警告，但这是你在这种可能致命的情况下获得的唯一可见反馈。

  你可能希望限制你的 umask，以防止同一系统上的其他用户访问所创建的文件。

  如果多次提供 <u>--cookie-jar</u>，则使用最后设置的值。

  示例：
  `curl -c store-here.txt https://example.com`
  `curl -c store-here.txt -b read-these https://example.com`

  另请参阅 <u>--cookie</u> 和 <u>--junk-session-cookies</u>。

- <font color=darkcyan>--create-dirs</font >
  与 <u>--output</u> 选项配合使用时，<u>curl</u> 会根据需要创建必要的本地目录层级。此选项会创建 <u>--output</u> 选项中提到的目录，并结合可能使用 <u>--output-dir</u> 设置的路径。如果组合后的输出文件名不包含目录，或者其提到的目录已经存在，则不会创建目录。

  在 Unix 风格的文件系统上，创建的目录权限模式为 0750。

  要在使用 FTP 或 SFTP 时创建远程目录，请尝试 <u>--ftp-create-dirs</u>。

  多次提供 <u>--create-dirs</u> 没有额外效果。可使用 <u>--no-create-dirs</u> 再次禁用它。

  示例：
  `curl --create-dirs --output local/dir/file https://example.com`

  另请参阅 <u>--ftp-create-dirs</u> 和 <u>--output-dir</u>。

- <font color=darkcyan>--create-file-mode</font > \<mode\>
  （SFTP SCP FILE）当使用 <u>curl</u> 通过支持的协议之一远程创建文件时，此选项允许用户在创建文件时设置文件的 'mode'，而不是默认的 0644。

  此选项接受一个八进制数作为参数。

  如果多次提供 <u>--create-file-mode</u>，则使用最后设置的值。

  示例：
  `curl --create-file-mode 0777 -T localfile sftp://example.com/new`

  在 7.75.0 中添加。另请参阅 <u>--ftp-create-dirs</u>。

- <font color=darkcyan>--crlf</font >
  （FTP SMTP）在上传时将换行符转换为回车加换行符。对 MVS (OS/390) 很有用。

  多次提供 <u>--crlf</u> 没有额外效果。可使用 <u>--no-crlf</u> 再次禁用它。

  示例：
  `curl --crlf -T file ftp://example.com/`

  另请参阅 <u>--use-ascii</u>。

- <font color=darkcyan>--crlfile</font > \<file\>
  （TLS）提供使用 PEM 格式的文件，其中包含证书吊销列表，该列表可以指定应被视为已吊销的对端证书。

  如果多次提供 <u>--crlfile</u>，则使用最后设置的值。

  示例：
  `curl --crlfile rejects.txt https://example.com`

  另请参阅 <u>--cacert</u> 和 <u>--capath</u>。

- <font color=darkcyan>--curves</font > \<list\>
  （TLS）根据 RFC 8422, 5.1，设置在 SSL 会话建立期间使用的特定曲线。可以通过用 ":" 分隔来提供多种算法（例如 "X25519:P-521"）。该参数在 OpenSSL 的 "s_client" 和 "s_server" 实用程序中同样可用。

  <u>--curves</u> 允许由 OpenSSL 驱动的 <u>curl</u> 使用客户端请求的确切（EC）曲线建立 SSL 连接，从而避免不透明的客户端/服务器协商。

  如果设置了此选项，将忽略 OpenSSL 内置的默认曲线列表。

  如果多次提供 <u>--curves</u>，则使用最后设置的值。

  示例：
  `curl --curves X25519 https://example.com`

  在 7.73.0 中添加。另请参阅 <u>--ciphers</u>。

- <font color=darkcyan>-d, --data</font > \<data\>
  （HTTP MQTT）以 POST 请求将指定数据发送到 HTTP 服务器，其方式与用户填写 HTML 表单并按下提交按钮时浏览器所做的一样。此选项使 <u>curl</u> 使用 content-type application/x-www-form-urlencoded 将数据传递给服务器。与 <u>--form</u> 相比。

  <u>--data-raw</u> 几乎相同，但不对 @ 字符进行特殊解释。要发布纯二进制数据，你应该改用 <u>--data-binary</u> 选项。要对表单字段的值进行 URL 编码，你可以使用 <u>--data-urlencode</u>。

  如果在同一命令行中多次使用这些选项中的任何一个，则指定的数据片段将使用分隔符 `&` 合并。因此，使用 `-d name=daniel -d skill=lousy` 将生成类似 `name=daniel&skill=lousy` 的 post 数据块。

  如果数据以字母 @ 开头，其余部分应该是从中读取数据的文件名，或者如果希望 <u>curl</u> 从标准输入读取数据，则使用 `-`。因此，从名为 'foobar' 的文件发布数据将使用 `--data @foobar`。当告知 <u>--data</u> 从这样的文件读取时，回车符、换行符和空字节将被剥离。如果你不希望 @ 字符具有特殊解释，请改用 <u>--data-raw</u>。

  此选项的数据完全按照命令行中提供的内容传递给服务器。<u>curl</u> 不会转换、更改或改进它。用户有责任以正确的格式提供数据。

  <u>--data</u> 可以在命令行中多次使用。

  示例：
  `curl -d "name=curl" https://example.com`
  `curl -d "name=curl" -d "tool=cmdline" https://example.com`
  `curl -d @filename https://example.com`

  此选项与 <u>--form</u>、<u>--head</u> 和 <u>--upload-file</u> 互斥。另请参阅 <u>--data-binary</u>、<u>--data-urlencode</u> 和 <u>--data-raw</u>。

- <font color=darkcyan>--data-ascii</font > \<data\>
  （HTTP）此选项只是 <u>--data</u> 的别名。

  <u>--data-ascii</u> 可以在命令行中多次使用。

  示例：
  `curl --data-ascii @file https://example.com`

  另请参阅 <u>--data-binary</u>、<u>--data-raw</u> 和 <u>--data-urlencode</u>。

- <font color=darkcyan>--data-binary</font > \<data\>
  （HTTP）完全按照指定的内容发布数据，不进行任何额外处理。

  如果数据以字母 @ 开头，则其余部分应为文件名。"@-" 使 <u>curl</u> 从标准输入读取数据。数据发布方式与 <u>--data</u> 类似，不同之处在于保留换行符和回车符，且从不进行转换。

  与 <u>--data</u> 一样，发送到服务器的默认 content-type 是 application/x-www-form-urlencoded。如果你希望服务器将数据视为任意二进制数据，则将 content-type 设置为 octet-stream：`-H "Content-Type: application/octet-stream"`。

  如果多次使用此选项，第一个选项之后的选项将按照 <u>--data</u> 中的描述追加数据。

  <u>--data-binary</u> 可以在命令行中多次使用。

  示例：
  `curl --data-binary @filename https://example.com`

  另请参阅 <u>--data-ascii</u>。

- <font color=darkcyan>--data-raw</font > \<data\>
  （HTTP）发布数据，类似于 <u>--data</u>，但不对 @ 字符进行特殊解释。

  <u>--data-raw</u> 可以在命令行中多次使用。

  示例：
  `curl --data-raw "hello" https://example.com`
  `curl --data-raw "@at@at@" https://example.com`

  另请参阅 <u>--data</u>。

- <font color=darkcyan>--data-urlencode</font > \<data\>
  （HTTP）发布数据，类似于其他 <u>--data</u> 选项，区别在于此选项执行 URL 编码。

  为了符合 CGI 规范，\<data\> 部分应以名称开头，后跟分隔符和内容规范。\<data\> 部分可以使用以下语法之一传递给 <u>curl</u>：

    * content
      对内容进行 URL 编码并传递。只需注意内容不要包含任何 "=" 或 "@" 符号，因为这会使语法匹配下面的其他情况。

    * =content
      对内容进行 URL 编码并传递。前面的 "=" 符号不包含在数据中。

    * name=content
      对内容部分进行 URL 编码并传递。请注意，名称部分应为已进行 URL 编码的形式。

    * @filename
      从给定文件加载数据（包括任何换行符），对该数据进行 URL 编码，并在 POST 中传递。使用 "@-" 使 <u>curl</u> 从标准输入读取数据。

    * name@filename
      从给定文件加载数据（包括任何换行符），对该数据进行 URL 编码，并在 POST 中传递。名称部分会附加一个等号，结果为 name=urlencoded-file-content。请注意，名称应为已进行 URL 编码的形式。

  <u>--data-urlencode</u> 可以在命令行中多次使用。

  示例：
  `curl --data-urlencode name=val https://example.com`
  `curl --data-urlencode =encodethis https://example.com`
  `curl --data-urlencode name@file https://example.com`
  `curl --data-urlencode @fileonly https://example.com`

  另请参阅 <u>--data</u> 和 <u>--data-raw</u>。

- <font color=darkcyan>--delegation</font > \<LEVEL\>
  （GSS/kerberos）设置当涉及用户凭据时，允许 <u>curl</u> 进行何种级别的委派。

  none：不允许任何委派。

  policy：当且仅当 Kerberos 服务票据中设置了 OK-AS-DELEGATE 标志时才进行委派，这属于领域策略问题。

  always：无条件允许服务器进行委派。

  如果多次提供 <u>--delegation</u>，则使用最后设置的值。

  示例：
  `curl --delegation "none" https://example.com`

  另请参阅 <u>--insecure</u> 和 <u>--ssl</u>。

- <font color=darkcyan>--digest</font >
  （HTTP）启用 HTTP Digest 认证。此认证方案避免以明文形式在网络上发送密码。将其与常规的 <u>--user</u> 选项结合使用以设置用户名和密码。

  多次提供 <u>--digest</u> 没有额外效果。可使用 <u>--no-digest</u> 再次禁用它。

  示例：
  `curl -u name:password --digest https://example.com`

  另请参阅 <u>--user</u>、<u>--proxy-digest</u> 和 <u>--anyauth</u>。

- <font color=darkcyan>-q, --disable</font >
  如果作为命令行上的第一个参数使用，则不会读取或使用 curlrc 配置文件。有关默认配置文件搜索路径的详细信息，请参见 <u>--config</u>。

  多次提供 <u>--disable</u> 没有额外效果。可使用 <u>--no-disable</u> 再次禁用它。

  示例：
  `curl -q https://example.com`

  另请参阅 <u>--config</u>。

- <font color=darkcyan>--disable-eprt</font >
  （FTP）在进行主动 FTP 传输时，禁用 EPRT 和 LPRT 命令。通常 <u>curl</u> 会先尝试使用 EPRT，然后再使用 PORT，但使用此选项后，它会直接使用 PORT。EPRT 是原始 FTP 协议的扩展，并非在所有服务器上都能工作，但与传统的 PORT 命令相比，它能以更好的方式启用更多功能。

  <u>--eprt</u> 可用于显式重新启用 EPRT，而 <u>--no-eprt</u> 是 <u>--disable-eprt</u> 的别名。

  如果使用 IPv6 访问服务器，此选项无效，因为此时 EPRT 是必需的。

  禁用 EPRT 仅改变主动模式的行为。如果你想切换到被动模式，你需要不使用 <u>--ftp-port</u> 或使用 <u>--ftp-pasv</u> 强制执行。

  多次提供 <u>--disable-eprt</u> 没有额外效果。可使用 <u>--no-disable-eprt</u> 再次禁用它。

  示例：
  `curl --disable-eprt ftp://example.com/`

  另请参阅 <u>--disable-epsv</u> 和 <u>--ftp-port</u>。

- <font color=darkcyan>--disable-epsv</font >
  （FTP）在进行被动 FTP 传输时，禁用 EPSV 命令。通常 <u>curl</u> 会在 PASV 之前先尝试使用 EPSV，但使用此选项后，它不会尝试 EPSV。

  <u>--epsv</u> 可用于显式重新启用 EPSV，而 <u>--no-epsv</u> 是 <u>--disable-epsv</u> 的别名。

  如果服务器是 IPv6 主机，此选项无效，因为此时 EPSV 是必需的。

  禁用 EPSV 仅改变被动模式的行为。如果你想切换到主动模式，你需要使用 <u>--ftp-port</u>。

  多次提供 <u>--disable-epsv</u> 没有额外效果。可使用 <u>--no-disable-epsv</u> 再次禁用它。

  示例：
  `curl --disable-epsv ftp://example.com/`

  另请参阅 <u>--disable-eprt</u> 和 <u>--ftp-port</u>。

- <font color=darkcyan>--disallow-username-in-url</font >
  如果传入包含用户名的 URL，则报错退出。当 URL 是在运行时提供或类似情况下提供时，这可能最有用。

  在 URL 中接受并使用凭据通常被视为安全隐患，因为它们很容易通过这种方式泄露。

  多次提供 <u>--disallow-username-in-url</u> 没有额外效果。可使用 <u>--no-disallow-username-in-url</u> 再次禁用它。

  示例：
  `curl --disallow-username-in-url https://example.com`

  另请参阅 <u>--proto</u>。

- <font color=darkcyan>--dns-interface</font > \<interface\>
  （DNS）通过给定的网络接口发送传出的 DNS 请求。此选项是对 <u>--interface</u> 的补充（后者不影响 DNS）。提供的字符串必须是接口名称（而不是地址）。

  如果多次提供 <u>--dns-interface</u>，则使用最后设置的值。

  示例：
  `curl --dns-interface eth0 https://example.com`

  要使 <u>--dns-interface</u> 生效，需要底层 libcurl 在构建时支持 c-ares。另请参阅 <u>--dns-ipv4-addr</u> 和 <u>--dns-ipv6-addr</u>。

- <font color=darkcyan>--dns-ipv4-addr</font > \<address\>
  （DNS）在进行 IPv4 DNS 请求时绑定到特定的 IP 地址，以便 DNS 请求源自该地址。参数应为单个 IPv4 地址。

  如果多次提供 <u>--dns-ipv4-addr</u>，则使用最后设置的值。

  示例：
  `curl --dns-ipv4-addr 10.1.2.3 https://example.com`

  要使 <u>--dns-ipv4-addr</u> 生效，需要底层 libcurl 在构建时支持 c-ares。另请参阅 <u>--dns-interface</u> 和 <u>--dns-ipv6-addr</u>。

- <font color=darkcyan>--dns-ipv6-addr</font > \<address\>
  （DNS）在进行 IPv6 DNS 请求时绑定到特定的 IP 地址，以便 DNS 请求源自该地址。参数应为单个 IPv6 地址。

  如果多次提供 <u>--dns-ipv6-addr</u>，则使用最后设置的值。

  示例：
  `curl --dns-ipv6-addr 2a04:4e42::561 https://example.com`

  要使 <u>--dns-ipv6-addr</u> 生效，需要底层 libcurl 在构建时支持 c-ares。另请参阅 <u>--dns-interface</u> 和 <u>--dns-ipv4-addr</u>。

- <font color=darkcyan>--dns-servers</font > \<addresses\>
  （DNS）设置要使用的 DNS 服务器列表，以代替系统默认值。IP 地址列表应以逗号分隔。端口号也可以可选地给出，用冒号分隔附加在 IP 地址后面。

  如果多次提供 <u>--dns-servers</u>，则使用最后设置的值。

  示例：
  `curl --dns-servers 192.168.0.1,192.168.0.2 https://example.com`
  `curl --dns-servers 10.0.0.1:53 https://example.com`

  要使 <u>--dns-servers</u> 生效，需要底层 libcurl 在构建时支持 c-ares。另请参阅 <u>--dns-interface</u> 和 <u>--dns-ipv4-addr</u>。

- <font color=darkcyan>--doh-cert-status</font >
  （DNS）与 <u>--cert-status</u> 相同，但用于 DoH（DNS-over-HTTPS）。

  通过使用证书状态请求（又名 OCSP 装订）TLS 扩展来验证 DoH 服务器证书的状态。

  如果启用了此选项，并且 DoH 服务器发送了无效的（例如过期的）响应，如果响应表明服务器证书已被撤销，或者完全没有收到响应，则验证失败。

  目前仅在 OpenSSL 和 GnuTLS 后端中实现了此支持。

  多次提供 <u>--doh-cert-status</u> 没有额外效果。可使用 <u>--no-doh-cert-status</u> 再次禁用它。

  示例：
  `curl --doh-cert-status --doh-url https://doh.example https://example.com`

  在 7.76.0 中添加。另请参阅 <u>--doh-insecure</u>。

- <font color=darkcyan>--doh-insecure</font >
  （DNS）默认情况下，<u>curl</u> 与 DoH 服务器建立的每个连接在传输之前都会验证其安全性。此选项告诉 <u>curl</u> 跳过验证步骤并在不检查的情况下继续。

  警告：使用此选项会使 DoH 传输和名称解析变得不安全。

  此选项等同于 <u>--insecure</u> 和 <u>--proxy-insecure</u>，但仅用于 DoH（DNS-over-HTTPS）。

  多次提供 <u>--doh-insecure</u> 没有额外效果。可使用 <u>--no-doh-insecure</u> 再次禁用它。

  示例：
  `curl --doh-insecure --doh-url https://doh.example https://example.com`

  在 7.76.0 中添加。另请参阅 <u>--doh-url</u>、<u>--insecure</u> 和 <u>--proxy-insecure</u>。

- <font color=darkcyan>--doh-url</font > \<URL\>
  （DNS）指定使用哪个 DNS-over-HTTPS (DoH) 服务器来解析主机名，而不是使用默认的名称解析机制。该 URL 必须是 HTTPS。

  您为传输设置的一些 SSL 选项也适用于 DoH，因为名称查找是在 SSL 上进行的。但是，证书验证设置不会被继承，而是通过 <u>--doh-insecure</u> 和 <u>--doh-cert-status</u> 单独控制。

  默认情况下，在最初查找 DoH 服务器的 DNS 记录时会绕过 DoH。你可以使用 <u>--resolve</u> 指定 DoH 服务器的 IP 地址以避免这种情况。

  如果使用空字符串 "" 作为 URL，则取消设置此选项。（在 7.85.0 中添加）

  如果多次提供 <u>--doh-url</u>，则使用最后设置的值。

  示例：
  `curl --doh-url https://doh.example https://example.com`
  `curl --doh-url https://doh.example --resolve doh.example:443:192.0.2.1 https://example.com`

  另请参阅 <u>--doh-insecure</u>。

- <font color=darkcyan>--dump-ca-embed</font >
  （TLS）将嵌入在 <u>curl</u> 中的 CA 证书包写入标准输出，然后退出。

  如果 <u>curl</u> 构建时未嵌入默认的 CA 证书包，则输出为空。

  多次提供 <u>--dump-ca-embed</u> 没有额外效果。可使用 <u>--no-dump-ca-embed</u> 再次禁用它。

  示例：
  `curl --dump-ca-embed`

  在 8.10.0 中添加。另请参阅 <u>--ca-native</u>、<u>--cacert</u>、<u>--capath</u>、<u>--proxy-ca-native</u>、<u>--proxy-cacert</u> 和 <u>--proxy-capath</u>。

- <font color=darkcyan>-D, --dump-header</font > \<filename\>
  （HTTP FTP）将接收到的协议头写入指定文件。如果未接收到任何头信息，使用此选项将创建一个空文件。将文件名指定为 "-"（单个减号）可将其写入标准输出。

  从 <u>curl</u> 8.10.0 开始，将文件名指定为 "%"（单个百分号）会将输出写入标准错误。

  在 FTP 中使用时，FTP 服务器响应行被视为“头”，因此会保存在那里。

  从 <u>curl</u> 8.11.0 开始，使用 <u>--create-dirs</u> 选项还可以为 <u>--dump-header</u> 中提供的路径创建缺失的目录组件。

  在一组操作中包含多个传输（即在同一个 `--next` 子句中的 URL）时，会将它们追加到同一个文件中，并用空行分隔。

  如果多次提供 <u>--dump-header</u>，则使用最后设置的值。

  示例：
  `curl --dump-header store.txt https://example.com`
  `curl --dump-header - https://example.com -o save`

  另请参阅 <u>--output</u>。
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