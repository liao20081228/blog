---
title: ubuntu显示管理器LightDM
tags: Linux教程
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文翻译自<font color=blue>[《ubuntu官方文档》](https://wiki.ubuntu.com/LightDM)。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------


# 1 LightDM是什么

什么是LightDM？在16.04 LTS版本及之前，LightDM是在Ubuntu中运行的显示管理器。虽然在后来的Ubuntu版本中它已被GDM取代，但默认情况下仍会在最近的几个版本的Ubuntu中使用LightDM。

LightDM启动X服务器，用户会话和欢迎界面（登录屏幕）。最高版本为16.04 LTS的Ubuntu中欢迎界面是Unity Greeter。



# 2 配置 

后来版本的lightdm(15.10及以上)用`[Seat:*]`替换了过时的`[SeatDefaults]`。

 
LightDM 配置文件包括: 

```
/usr/share/lightdm/lightdm.conf.d/*.conf 
/etc/lightdm/lightdm.conf.d/*.conf 
/etc/lightdm/lightdm.conf 

```

系统使用的配置参数保存在 `/usr/share/lightdm/lightdm.conf.d/*.conf` ,用户不能编辑。系统管理员可以在`/etc/lightdm/lightdm.conf.d/*.conf 和/etc/lightdm/lightdm.conf`中覆盖该配置。系统会依次读取前述的三个文件最后得到 LightDM 的配置信息。




例如,如果你想要重载系统默认的会话(默认会话保存在 /usr/share/lightdm/lightdm.conf.d/50-ubuntu.conf )，你应该新建文件`/etc/lightdm/lightdm.conf.d/50-myconfig.conf` ,其内容如下: 
```
[SeatDefaults] 
user-session=mysession 
```
**在 `/usr/share/doc/lightdm/lightdm.conf.gz` 文件中保存着所有可能的配置选项**。

还有一个配置文件：`/etc/lightdm/users.conf`,但是如果您的系统上运行了accountsservice，则会忽略此配置文件（如果您不确定，请使用命令ps -aef | grep accountsservice从shell进行检查）。



## 2.1 禁止访客登录 
LightDM 默认允许你以临时访客登录,禁止该功能:  
```
[SeatDefaults] 
allow-guest=false
```
## 2.2 隐藏用户列表 
Unity Greeter以及其他欢迎界面，默认显示一个所有可能的用户的列表。如果你想禁用该功能，可以使用以下配置。使用这个配置也能意味着你需要启动手动登录功能。 
```
[SeatDefaults] 
greeter-hide-users=true
```
## 2.3 允许手动登录 
Unity Greeter以及其他欢迎界面 默认不允许你输入用户名来登录。你可以使用以下配置启用该特性。 
```
[SeatDefaults] 
greeter-show-manual-login=true
```
## 2.4 设置自动登录 
设置 autologin-user来使系统启动时自动登录以前登陆过的某个帐户。设置autologin-user-timeout 限制用户在设定秒内如果没有自动登录则不能自动登录。 
```
[SeatDefaults] 
autologin-user=username 
autologin-user-timeout=delay
```

设置自动作为访客登陆。
```
autologin-guest=true
```

## 2.4 修改默认会话 
默认会话设置保存在 `/usr/share/lightdm/lightdm.conf.d/ `会话包中。 
```
[SeatDefaults] 
user-session=name 
```
其中 name 代表` /usr/share/xsessions/*.desktop` 中 name.desktop。

## 2.5 修改欢迎界面 
欢迎界面由 `/usr/share/lightdm/lightdm.conf.d/` 中欢迎界面包提供。你可以重载该设置。 
```
[SeatDefaults] 
greeter-session=name 
```
其中 name 代表` /usr/share/xgreeters/*.desktop` 中的 anme.desktop 文件。

## 2.6 添加系统钩子 
如果你想在 X servers 和用户会话启动/关停时自动做些事情,那么可以按照以下方式设置自动执行命令: 
```
[SeatDefaults] 
display-setup-script=command 
display-stopped-script=command (Not in Ubuntu 12.04 LTS) 
greeter-setup-script=command 
session-setup-script=command 
session-cleanup-script=command 
session-wrapper=command 
greeter-wrapper=command (Not in Ubuntu 12.04 LTS) 
guest-wrapper=command
xserver-command-command

```
**display-setup-script** 在 X server 启动后,欢迎界面启动之前运行。该命令由 root 运行,如果命令执行出现错误,X server 会停止运行。 

**display-stopped-script** 在 X server 退出后运行。该命令由 root 运行。

**greeter-setup-script** 在欢迎界面启动前运行。该命令由 root 运行。如果命令执行出现错误，欢迎界面将无法启动并导致 LightDM 退出。 

**session-setup-script** 用户会话启动之前运行，如果失败，用户会话将不启动，并返回欢迎界面。

**session-cleanup-script** 在欢迎界面或用户会话停止之后运行。由 root 运行。 

**session-wrapper** 该命令用于运行会话。该命令使用用户身份运行，需要执行传递来的参数以完成运行会话。默认值为 lightdm-session。因此如果需要覆盖此设置，您应该链接到它 

**greeter-wrapper** 该命令用于运行欢迎界面。等同于 session-wrapper。

**guest-wrapper** 该命令用于运行欢迎界面。等同于guest-wrapper。

**xserver-command** 设置xserver启动时要执行的命令。

## 2.7 修改墙纸 

LightDM 不配置欢迎界面外观。这由欢迎界面设置。
。 
ubuntu Unity Greeter 默认显示当前选中的用户的背景图案。为了设置默认背景，并停止背景切换，请编辑 `/usr/share/glib-2.0/schemas/10_unity_greeter_background.gschema.override`。

```
[com.canonical.unity-greeter] 
draw-user-backgrounds=false 
background='/foo/wallpaper.png’ 
```
然后运行 `sudo glib-compile-schemas /usr/share/glib-2.0/schemas/` 使配置生效。


如果使用的是 LightDM GTK+ 欢迎界面,编辑` /etc/lightdm/lightdm-gtk-greeter.conf `: 
```
background=/usr/share/lubuntu/wallpapers/lubuntu-default-wallpaper.png
```


>上面列举的文件中可能有些是不存在的，只要新建即可达到效果。比如入口文件目录：/usr/share/lightdm/lightdm.conf.d/
>其次是默认的文件，一定有这个文件：/usr/share/lightdm/lightdm.conf.d/50-ubuntu.conf，很多情况只需要编辑这个文件即可。

# 2.8 禁止日志备份
默认备份设置保存在 `/usr/share/lightdm/lightdm.conf.d/ `备份设置包中。 
```
backup-logs=false
```
# 3 LightDM 相关操作 
 
LightDM 日志:/var/log/lightdm。 

关停 LightDM: sudostoplightdm。

启动LightDM: sudo start lightdm。 

设置 LightDM 为默认显示管理器:$ sudo dpkg-recofigure lightdm。



路径技巧：

# 4  lightdm命令

# 5 dm-tool命令

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
