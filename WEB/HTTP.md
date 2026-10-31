---
title: HTTP
tags: https

---

# HTTP 消息结构

请求
- **请求行**（Request Line）：
  - **方法**：如 GET、POST、PUT、DELETE等，指定要执行的操作。
  - 请求 URI**（统一资源标识符）：请求的资源路径，通常包括主机名、端口号（如果非默认）、路径和查询字符串。
  - **HTTP 版本**：如 HTTP/1.1 或 HTTP/2。
- **请求头**（Request Headers）：
  - 包含了客户端环境信息、请求体的大小（如果有）、客户端支持的压缩类型等。
  - 常见的请求头包括`Host`、`User-Agent`、`Accept`、`Accept-Encoding`、`Content-Length`等。
- 空行**：请求头和请求体之间的分隔符，表示请求头的结束。
- **请求体**（可选）：在某些类型的HTTP请求（如 POST 和 PUT）中，请求体包含要发送给服务器的数据。
"

响应
- **响应行/状态行**（Status Line）：
  - **HTTP 版本**：与请求消息中的版本相匹配。
  -**状态码**：三位数，表示请求的处理结果，如 200 表示成功，404 表示未找到资源。
  - **状态信息**：状态码的简短描述。
- **响应头**（Response Headers）：
  - 包含了服务器环境信息、响应体的大小、服务器支持的压缩类型等。
  - 常见的响应头包括`Content-Type`、`Content-Length`、`Server`、`Set-Cookie`等
- **空行**：响应头和响应体之间的分隔符，表示响应头的结束。
- **响应体**（可选）：包含服务器返回的数据，如请求的网页内容、图片、JSON数据等。

# 方法

| 方法    | 描述                                                                                                                                               |
| ------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| GET     | 从服务器获取资源。用于请求数据而不对数据进行更改。<br/>例如，从服务器获取网页、图片等。                                                            |
| POST    | 向服务器发送数据以创建新资源。常用于提交表单数据或上传文件。<br/>发送的数据包含在请求体中。                                                        |
| PUT     | 向服务器发送数据以更新现有资源。如果资源不存在，则创建新的资源。<br/>与 POST 不同，PUT 通常是幂等的，即多次执行相同的 PUT 请求不会产生不同的结果。 |
| DELETE  | 从服务器删除指定的资源。请求中包含要删除的资源标识符。                                                                                             |
| PATCH   | 对资源进行部分修改。与 PUT 类似，但 PATCH 只更改部分数据而不是替换整个资源。                                                                       |
| HEAD    | 类似于 GET，但服务器只返回响应的头部，不返回实际数据。<br/>用于检查资源的元数据（例如，检查资源是否存在，查看响应的头部信息）。                    |
| OPTIONS | 返回服务器支持的 HTTP 方法。用于检查服务器支持哪些请求方法，<br/>通常用于跨域资源共享（CORS）的预检请求。                                          |
| TRACE   | 回显服务器收到的请求，主要用于诊断。客户端可以查看请求在服务器中的处理路径。                                                                       |
| CONNECT | 建立一个到服务器的隧道，通常用于 HTTPS 连接。<br/>客户端可以通过该隧道发送加密的数据。                                                             |
# 头部

| 响应头信息（英文）        | 响应头信息（中文） | 描述                                                                                                                                                                 |
| ------------------------- | ------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Date                      | 日期               | 响应生成的日期和时间。例如：Wed, 18 Apr 2024 12:00:00 GMT                                                                                                            |
| Server                    | 服务器             | 服务器软件的名称和版本。例如：Apache/2.4.1 (Unix)                                                                                                                    |
| Content-Type              | 内容类型           | 响应体的媒体类型（MIME类型），如`text/html; charset=UTF-8`, `application/json`等。                                                                                   |
| Content-Length            | 内容长度           | 响应体的大小，单位是字节。例如：3145                                                                                                                                 |
| Content-Encoding          | 内容编码           | 响应体的压缩编码，如 `gzip`, `deflate`等。                                                                                                                           |
| Content-Language          | 内容语言           | 响应体的语言。例如：zh-CN                                                                                                                                            |
| Content-Location          | 内容位置           | 响应体的 URI。例如：/index.html                                                                                                                                      |
| Content-Range             | 内容范围           | 响应体的字节范围，用于分块传输。例如：bytes 0-999/8000                                                                                                               |
| Cache-Control             | 缓存控制           | 控制响应的缓存行为, 如 no-cache 表示必须重新请求。                                                                                                                   |
| Connection                | 连接               | 管理连接的选项，如`keep-alive`或`close`，keep-alive 表示连接不会在传输后关闭。。                                                                                     |
| Set-Cookie                | 设置 Cookie        | 设置客户端的 cookie。例如：sessionId=abc123; Path=/; Secure                                                                                                          |
| Expires                   | 过期时间           | 响应体的过期日期和时间。例如：Thu, 18 Apr 2024 12:00:00 GMT                                                                                                          |
| Last-Modified             | 最后修改时间       | 资源最后被修改的日期和时间。例如：Wed, 18 Apr 2024 11:00:00 GMT                                                                                                      |
| ETag                      | 实体标签           | 资源的特定版本的标识符。例如："33a64df551425fcc55e6"                                                                                                                 |
| Location                  | 位置               | 用于重定向的 URI。例如：/newresource                                                                                                                                 |
| Pragma                    | 实现特定的指令     | 包含实现特定的指令，如 `no-cache`。                                                                                                                                  |
| WWW-Authenticate          | 认证信息           | 认证信息，通常用于HTTP认证。例如：Basic realm="Access to the site"                                                                                                   |
| Accept-Ranges             | 接受范围           | 指定可接受的请求范围类型。例如：bytes                                                                                                                                |
| Age                       | 经过时间           | 响应生成后经过的秒数，从原始服务器生成到代理服务器。例如：24                                                                                                         |
| Allow                     | 允许方法           | 列出资源允许的 HTTP 方法 。例如：GET, POST，HEAD等                                                                                                                   |
| Vary                      | 变化               | 告诉下游代理如何使用响应头信息来确定响应是否可以从缓存中获取。例如：Accept                                                                                           |
| Strict-Transport-Security | 严格传输安全       | 指示浏览器仅通过 HTTPS 与服务器通信。例如：max-age=31536000; includeSubDomains                                                                                       |
| X-Frame-Options           | 框架选项           | 控制页面是否允许在框架中显示，防止点击劫持攻击。例如：SAMEORIGIN                                                                                                     |
| X-Content-Type-Options    | 内容类型选项       | 指示浏览器不要尝试猜测资源的 MIME 类型。例如：nosniff                                                                                                                |
| X-XSS-Protection          | XSS保护            | 控制浏览器的 XSS 过滤和阻断。例如：1; mode=block                                                                                                                     |
| Public-Key-Pins           | 公钥固定           | HTTP 头信息，用于HTTP公共密钥固定（HPKP），一种安全机制，用于防止中间人攻击。例如：pin-sha256="base64+primarykey"; pin-sha256="base64+backupkey"; max-age=expireTime |
## Content-Type

| 文件扩展名 | Content-Type(Mime-Type) | 文件扩展名 | Content-Type(Mime-Type) |
| --- | --- | --- | --- |
| .*（ 二进制流，不知道下载文件类型） | application/octet-stream | .tif | image/tiff |
| .001 | application/x-001 | .301 | application/x-301 |
| .323 | text/h323 | .906 | application/x-906 |
| .907 | drawing/907 | .a11 | application/x-a11 |
| .acp | audio/x-mei-aac | .ai | application/postscript |
| .aif | audio/aiff | .aifc | audio/aiff |
| .aiff | audio/aiff | .anv | application/x-anv |
| .asa | text/asa | .asf | video/x-ms-asf |
| .asp | text/asp | .asx | video/x-ms-asf |
| .au | audio/basic | .avi | video/avi |
| .awf | application/vnd.adobe.workflow | .biz | text/xml |
| .bmp | application/x-bmp | .bot | application/x-bot |
| .c4t | application/x-c4t | .c90 | application/x-c90 |
| .cal | application/x-cals | .cat | application/vnd.ms-pki.seccat |
| .cdf | application/x-netcdf | .cdr | application/x-cdr |
| .cel | application/x-cel | .cer | application/x-x509-ca-cert |
| .cg4 | application/x-g4 | .cgm | application/x-cgm |
| .cit | application/x-cit | .class | java/* |
| .cml | text/xml | .cmp | application/x-cmp |
| .cmx | application/x-cmx | .cot | application/x-cot |
| .crl | application/pkix-crl | .crt | application/x-x509-ca-cert |
| .csi | application/x-csi | .css | text/css |
| .cut | application/x-cut | .dbf | application/x-dbf |
| .dbm | application/x-dbm | .dbx | application/x-dbx |
| .dcd | text/xml | .dcx | application/x-dcx |
| .der | application/x-x509-ca-cert | .dgn | application/x-dgn |
| .dib | application/x-dib | .dll | application/x-msdownload |
| .doc | application/msword | .dot | application/msword |
| .drw | application/x-drw | .dtd | text/xml |
| .dwf | Model/vnd.dwf | .dwf | application/x-dwf |
| .dwg | application/x-dwg | .dxb | application/x-dxb |
| .dxf | application/x-dxf | .edn | application/vnd.adobe.edn |
| .emf | application/x-emf | .eml | message/rfc822 |
| .ent | text/xml | .epi | application/x-epi |
| .eps | application/x-ps | .eps | application/postscript |
| .etd | application/x-ebx | .exe | application/x-msdownload |
| .fax | image/fax | .fdf | application/vnd.fdf |
| .fif | application/fractals | .fo | text/xml |
| .frm | application/x-frm | .g4 | application/x-g4 |
| .gbr | application/x-gbr | . | application/x- |
| .gif | image/gif | .gl2 | application/x-gl2 |
| .gp4 | application/x-gp4 | .hgl | application/x-hgl |
| .hmr | application/x-hmr | .hpg | application/x-hpgl |
| .hpl | application/x-hpl | .hqx | application/mac-binhex40 |
| .hrf | application/x-hrf | .hta | application/hta |
| .htc | text/x-component | .htm | text/html |
| .html | text/html | .htt | text/webviewhtml |
| .htx | text/html | .icb | application/x-icb |
| .ico | image/x-icon | .ico | application/x-ico |
| .iff | application/x-iff | .ig4 | application/x-g4 |
| .igs | application/x-igs | .iii | application/x-iphone |
| .img | application/x-img | .ins | application/x-internet-signup |
| .isp | application/x-internet-signup | .IVF | video/x-ivf |
| .java | java/* | .jfif | image/jpeg |
| .jpe | image/jpeg | .jpe | application/x-jpe |
| .jpeg | image/jpeg | .jpg | image/jpeg |
| .jpg | application/x-jpg | .js | application/x-javascript |
| .jsp | text/html | .la1 | audio/x-liquid-file |
| .lar | application/x-laplayer-reg | .latex | application/x-latex |
| .lavs | audio/x-liquid-secure | .lbm | application/x-lbm |
| .lmsff | audio/x-la-lms | .ls | application/x-javascript |
| .ltr | application/x-ltr | .m1v | video/x-mpeg |
| .m2v | video/x-mpeg | .m3u | audio/mpegurl |
| .m4e | video/mpeg4 | .mac | application/x-mac |
| .man | application/x-troff-man | .math | text/xml |
| .mdb | application/msaccess | .mdb | application/x-mdb |
| .mfp | application/x-shockwave-flash | .mht | message/rfc822 |
| .mhtml | message/rfc822 | .mi | application/x-mi |
| .mid | audio/mid | .midi | audio/mid |
| .mil | application/x-mil | .mml | text/xml |
| .mnd | audio/x-musicnet-download | .mns | audio/x-musicnet-stream |
| .mocha | application/x-javascript | .movie | video/x-sgi-movie |
| .mp1 | audio/mp1 | .mp2 | audio/mp2 |
| .mp2v | video/mpeg | .mp3 | audio/mp3 |
| .mp4 | video/mpeg4 | .mpa | video/x-mpg |
| .mpd | application/vnd.ms-project | .mpe | video/x-mpeg |
| .mpeg | video/mpg | .mpg | video/mpg |
| .mpga | audio/rn-mpeg | .mpp | application/vnd.ms-project |
| .mps | video/x-mpeg | .mpt | application/vnd.ms-project |
| .mpv | video/mpg | .mpv2 | video/mpeg |
| .mpw | application/vnd.ms-project | .mpx | application/vnd.ms-project |
| .mtx | text/xml | .mxp | application/x-mmxp |
| .net | image/pnetvue | .nrf | application/x-nrf |
| .nws | message/rfc822 | .odc | text/x-ms-odc |
| .out | application/x-out | .p10 | application/pkcs10 |
| .p12 | application/x-pkcs12 | .p7b | application/x-pkcs7-certificates |
| .p7c | application/pkcs7-mime | .p7m | application/pkcs7-mime |
| .p7r | application/x-pkcs7-certreqresp | .p7s | application/pkcs7-signature |
| .pc5 | application/x-pc5 | .pci | application/x-pci |
| .pcl | application/x-pcl | .pcx | application/x-pcx |
| .pdf | application/pdf | .pdf | application/pdf |
| .pdx | application/vnd.adobe.pdx | .pfx | application/x-pkcs12 |
| .pgl | application/x-pgl | .pic | application/x-pic |
| .pko | application/vnd.ms-pki.pko | .pl | application/x-perl |
| .plg | text/html | .pls | audio/scpls |
| .plt | application/x-plt | .png | image/png |
| .png | application/x-png | .pot | application/vnd.ms-powerpoint |
| .ppa | application/vnd.ms-powerpoint | .ppm | application/x-ppm |
| .pps | application/vnd.ms-powerpoint | .ppt | application/vnd.ms-powerpoint |
| .ppt | application/x-ppt | .pr | application/x-pr |
| .prf | application/pics-rules | .prn | application/x-prn |
| .prt | application/x-prt | .ps | application/x-ps |
| .ps | application/postscript | .ptn | application/x-ptn |
| .pwz | application/vnd.ms-powerpoint | .r3t | text/vnd.rn-realtext3d |
| .ra | audio/vnd.rn-realaudio | .ram | audio/x-pn-realaudio |
| .ras | application/x-ras | .rat | application/rat-file |
| .rdf | text/xml | .rec | application/vnd.rn-recording |
| .red | application/x-red | .rgb | application/x-rgb |
| .rjs | application/vnd.rn-realsystem-rjs | .rjt | application/vnd.rn-realsystem-rjt |
| .rlc | application/x-rlc | .rle | application/x-rle |
| .rm | application/vnd.rn-realmedia | .rmf | application/vnd.adobe.rmf |
| .rmi | audio/mid | .rmj | application/vnd.rn-realsystem-rmj |
| .rmm | audio/x-pn-realaudio | .rmp | application/vnd.rn-rn_music_package |
| .rms | application/vnd.rn-realmedia-secure | .rmvb | application/vnd.rn-realmedia-vbr |
| .rmx | application/vnd.rn-realsystem-rmx | .rnx | application/vnd.rn-realplayer |
| .rp | image/vnd.rn-realpix | .rpm | audio/x-pn-realaudio-plugin |
| .rsml | application/vnd.rn-rsml | .rt | text/vnd.rn-realtext |
| .rtf | application/msword | .rtf | application/x-rtf |
| .rv | video/vnd.rn-realvideo | .sam | application/x-sam |
| .sat | application/x-sat | .sdp | application/sdp |
| .sdw | application/x-sdw | .sit | application/x-stuffit |
| .slb | application/x-slb | .sld | application/x-sld |
| .slk | drawing/x-slk | .smi | application/smil |
| .smil | application/smil | .smk | application/x-smk |
| .snd | audio/basic | .sol | text/plain |
| .sor | text/plain | .spc | application/x-pkcs7-certificates |
| .spl | application/futuresplash | .spp | text/xml |
| .ssm | application/streamingmedia | .sst | application/vnd.ms-pki.certstore |
| .stl | application/vnd.ms-pki.stl | .stm | text/html |
| .sty | application/x-sty | .svg | text/xml |
| .swf | application/x-shockwave-flash | .tdf | application/x-tdf |
| .tg4 | application/x-tg4 | .tga | application/x-tga |
| .tif | image/tiff | .tif | application/x-tif |
| .tiff | image/tiff | .tld | text/xml |
| .top | drawing/x-top | .torrent | application/x-bittorrent |
| .tsd | text/xml | .txt | text/plain |
| .uin | application/x-icq | .uls | text/iuls |
| .vcf | text/x-vcard | .vda | application/x-vda |
| .vdx | application/vnd.visio | .vml | text/xml |
| .vpg | application/x-vpeg005 | .vsd | application/vnd.visio |
| .vsd | application/x-vsd | .vss | application/vnd.visio |
| .vst | application/vnd.visio | .vst | application/x-vst |
| .vsw | application/vnd.visio | .vsx | application/vnd.visio |
| .vtx | application/vnd.visio | .vxml | text/xml |
| .wav | audio/wav | .wax | audio/x-ms-wax |
| .wb1 | application/x-wb1 | .wb2 | application/x-wb2 |
| .wb3 | application/x-wb3 | .wbmp | image/vnd.wap.wbmp |
| .wiz | application/msword | .wk3 | application/x-wk3 |
| .wk4 | application/x-wk4 | .wkq | application/x-wkq |
| .wks | application/x-wks | .wm | video/x-ms-wm |
| .wma | audio/x-ms-wma | .wmd | application/x-ms-wmd |
| .wmf | application/x-wmf | .wml | text/vnd.wap.wml |
| .wmv | video/x-ms-wmv | .wmx | video/x-ms-wmx |
| .wmz | application/x-ms-wmz | .wp6 | application/x-wp6 |
| .wpd | application/x-wpd | .wpg | application/x-wpg |
| .wpl | application/vnd.ms-wpl | .wq1 | application/x-wq1 |
| .wr1 | application/x-wr1 | .wri | application/x-wri |
| .wrk | application/x-wrk | .ws | application/x-ws |
| .ws2 | application/x-ws | .wsc | text/scriptlet |
| .wsdl | text/xml | .wvx | video/x-ms-wvx |
| .xdp | application/vnd.adobe.xdp | .xdr | text/xml |
| .xfd | application/vnd.adobe.xfd | .xfdf | application/vnd.adobe.xfdf |
| .xhtml | text/html | .xls | application/vnd.ms-excel |
| .xls | application/x-xls | .xlw | application/x-xlw |
| .xml | text/xml | .xpl | audio/scpls |
| .xq | text/xml | .xql | text/xml |
| .xquery | text/xml | .xsd | text/xml |
| .xsl | text/xml | .xslt | text/xml |
| .xwd | application/x-xwd | .x_b | application/x-x_b |
| .sis | application/vnd.symbian.install | .sisx | application/vnd.symbian.install |
| .x_t | application/x-x_t | .ipa | application/vnd.iphone |
| .apk | application/vnd.android.package-archive | .xap | application/x-silverlight-app |


# 状态码

| 分类  | 分类描述                                       |
| ----- | ---------------------------------------------- |
| 1\*\* | 信息，服务器收到请求，需要请求者继续执行操作   |
| 2\*\* | 成功，操作被成功接收并处理                     |
| 3\*\* | 重定向，需要进一步的操作以完成请求             |
| 4\*\* | 客户端错误，请求包含语法错误或无法完成请求     |
| 5\*\* | 服务器错误，服务器在处理请求的过程中发生了错误 |

| 状态码 | 状态码英文名称 | 中文描述 |
| --- | --- | --- |
| 100 | Continue | 继续。客户端应继续其请求 |
| 101 | Switching Protocols | 切换协议。服务器根据客户端的请求切换协议。只能切换到更高级的协议，例如，切换到HTTP的新版本协议 |
|  |
| 200 | OK | 请求成功。一般用于GET与POST请求 |
| 201 | Created | 已创建。成功请求并创建了新的资源 |
| 202 | Accepted | 已接受。已经接受请求，但未处理完成 |
| 203 | Non-Authoritative Information | 非授权信息。请求成功。但返回的meta信息不在原始的服务器，而是一个副本 |
| 204 | No Content | 无内容。服务器成功处理，但未返回内容。在未更新网页的情况下，可确保浏览器继续显示当前文档 |
| 205 | Reset Content | 重置内容。服务器处理成功，用户终端（例如：浏览器）应重置文档视图。可通过此返回码清除浏览器的表单域 |
| 206 | Partial Content | 部分内容。服务器成功处理了部分GET请求 |
|  |
| 300 | Multiple Choices | 多种选择。请求的资源可包括多个位置，相应可返回一个资源特征与地址的列表用于用户终端（例如：浏览器）选择 |
| 301 | Moved Permanently | 永久移动。请求的资源已被永久的移动到新URI，返回信息会包括新的URI，浏览器会自动定向到新URI。今后任何新的请求都应使用新的URI代替 |
| 302 | Found | 临时移动。与301类似。但资源只是临时被移动。客户端应继续使用原有URI |
| 303 | See Other | 查看其它地址。与301类似。使用GET和POST请求查看 |
| 304 | Not Modified | 未修改。所请求的资源未修改，服务器返回此状态码时，不会返回任何资源。客户端通常会缓存访问过的资源，通过提供一个头信息指出客户端希望只返回在指定日期之后修改的资源 |
| 305 | Use Proxy | 使用代理。所请求的资源必须通过代理访问 |
| 306 | Unused | 已经被废弃的HTTP状态码 |
| 307 | Temporary Redirect | 临时重定向。与302类似。使用GET请求重定向 |
|  |
| 400 | Bad Request | 客户端请求的语法错误，服务器无法理解 |
| 401 | Unauthorized | 请求要求用户的身份认证 |
| 402 | Payment Required | 保留，将来使用 |
| 403 | Forbidden | 服务器理解请求客户端的请求，但是拒绝执行此请求 |
| 404 | Not Found | 服务器无法根据客户端的请求找到资源（网页）。通过此代码，网站设计人员可设置"您所请求的资源无法找到"的个性页面 |
| 405 | Method Not Allowed | 客户端请求中的方法被禁止 |
| 406 | Not Acceptable | 服务器无法根据客户端请求的内容特性完成请求 |
| 407 | Proxy Authentication Required | 请求要求代理的身份认证，与401类似，但请求者应当使用代理进行授权 |
| 408 | Request Time-out | 服务器等待客户端发送的请求时间过长，超时 |
| 409 | Conflict | 服务器完成客户端的 PUT 请求时可能返回此代码，服务器处理请求时发生了冲突 |
| 410 | Gone | 客户端请求的资源已经不存在。410不同于404，如果资源以前有现在被永久删除了可使用410代码，网站设计人员可通过301代码指定资源的新位置 |
| 411 | Length Required | 服务器无法处理客户端发送的不带Content-Length的请求信息 |
| 412 | Precondition Failed | 客户端请求信息的先决条件错误 |
| 413 | Request Entity Too Large | 由于请求的实体过大，服务器无法处理，因此拒绝请求。为防止客户端的连续请求，服务器可能会关闭连接。如果只是服务器暂时无法处理，则会包含一个Retry-After的响应信息 |
| 414 | Request-URI Too Large | 请求的URI过长（URI通常为网址），服务器无法处理 |
| 415 | Unsupported Media Type | 服务器无法处理请求附带的媒体格式 |
| 416 | Requested range not satisfiable | 客户端请求的范围无效 |
| 417 | Expectation Failed（预期失败） | 服务器无法满足请求头中 Expect 字段指定的预期行为。 |
| 418 | I'm a teapot | 状态码 418 实际上是一个愚人节玩笑。它在 RFC 2324 中定义，该 RFC 是一个关于超文本咖啡壶控制协议（HTCPCP）的笑话文件。在这个笑话中，418 状态码是作为一个玩笑加入到 HTTP 协议中的。 |
|  |
| 500 | Internal Server Error | 服务器内部错误，无法完成请求 |
| 501 | Not Implemented | 服务器不支持请求的功能，无法完成请求 |
| 502 | Bad Gateway | 作为网关或者代理工作的服务器尝试执行请求时，从远程服务器接收到了一个无效的响应 |
| 503 | Service Unavailable | 由于超载或系统维护，服务器暂时的无法处理客户端的请求。延时的长度可包含在服务器的Retry-After头信息中 |
| 504 | Gateway Time-out | 充当网关或代理的服务器，未及时从远端服务器获取请求 |
| 505 | HTTP Version not supported | 服务器不支持请求的HTTP协议的版本，无法完成处理 |

# URL与URI
```
URL：协议://IP:port + URI
URI：path?para=val
```

# 转义编码
## URL 与 URI 场景（HTTP 协议）

**规则：只要你的数据出现在 URL 的“参数值（Value）”里，且包含了 URL 的“保留字符”，就必须进行 URL 编码（百分号编码）。**


* **URL 保留字符**作为数据**出现在参数值里时需要编码**
* **URL 保留字符**表示其URL含义时不需要编码

### 通用分隔符（Gen-delims）
这些字符用来划分 URL 的各个主要组成部分（协议、主机、路径、参数等）。

| 字符    | 编码  | 在 URL 中的特殊含义（为什么被保留）                               |
| :------ | :---- | :---------------------------------------------------------------- |
| **`:`** | `%3A` | 分隔协议和主机（如 `https:`），或分隔主机和端口（如 `:8080`）     |
| **`/`** | `%2F` | 分隔路径的层级（如 `/api/v1/users`）                              |
| **`?`** | `%3F` | 标记路径的结束，查询参数（Query String）的开始                    |
| **`#`** | `%23` | 标记网页内的锚点/片段（Fragment），`#` 后面的内容不会发送给服务器 |
| **`[`** | `%5B` | 用于 IPv6 地址的包裹（如 `http://[::1]:8080`）                    |
| **`]`** | `%5D` | 用于 IPv6 地址的包裹                                              |
| **`@`** | `%40` | 用于分隔用户信息和主机名（如 `ftp://user:pass@host`）             |
### 子分隔符（Sub-delims）
这些字符用来划分更细粒度的数据，比如查询参数里的键值对。

| 字符 | 编码 | 在 URL 中的特殊含义（为什么被保留） |
| :--- | :--- | :--- |
| **`&`** | `%26` | 分隔多个查询参数（如 `?a=1&b=2`） |
| **`=`** | `%3D` | 分隔查询参数的键和值（如 `key=value`） |
| **`+`** | `%2B` | 在查询参数中通常代表空格（历史遗留） |
| **`;`** | `%3B` | 某些旧框架中用于分隔参数 |
| **`,`** | `%2C` | 用于分隔某些类型的值 |
| **`!`** | `%21` | 特殊用途 |
| **`$`** | `%24` | 特殊用途 |
| **`'`** | `%27` | 特殊用途 |
| **`(`** | `%28` | 特殊用途 |
| **`)`** | `%29` | 特殊用途 |
| **`*`** | `%2A` | 特殊用途 |

## HTML / XML 场景（标记语言）

**规则：当你要在文本或属性值里，原样显示 XML/HTML 的“控制字符”时，必须转义为实体字符。**
*   **什么时候需要转义？**
    *   文本里包含 `<` 或 `>`。如果不转义，解析器会认为你要开启/关闭一个标签。必须转成 `&lt;` 和 `&gt;`。
    *   文本里包含 `&`。因为 `&` 在 XML 里代表“实体的开始”，必须转成 `&amp;`。
    *   属性值（双引号内）包含双引号 `"`，必须转成 `&quot;`。
*   **什么时候不需要转义？**
    *   普通的字母、汉字、数字。
    *   单引号不需要转义（除非你的属性是用单引号包裹的）
