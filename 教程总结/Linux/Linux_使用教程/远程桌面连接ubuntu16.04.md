---
title: 远程连接ubuntu16.04 
tags: Linux教程
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：<font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

<style>table{word-break:initial;}</style>

# 1 图形界面
# 1.1 使用 XDMCP+xmanager+lightdm
第一步：开启XDMCP server
```bash
vim /etc/lightdm/lightdm.conf

#添加
[Seat:*]
Xserver-allow-tcp=true
[XDMCPserver]
enabled=true
```

第二步：如果启动了ufw，则打开 177端口，默认情况下，ubuntu桌面版没有开启ufw


```
sudo ufw allow 177
```

第三步：安装xubuntu，因为unit有bug
```
sudo apt install xubuntu-desktop
```
第四步：重启lightdm
```
sudo service lightdm restart
```

第五步：使用xmanager连接即可

# 1.2 使用 mate+xstart
```

sudo apt install ssh mate-desktop-evironment mate-desktop

```
然后用xstart建立会话，会话命令使用：`/usr/bin/mate-session --display $DISPLAY`
# 1.3 xshell+ssh+xmanager
```
sudo apt install ssh
```
xshell转发x11连接到xmanager

# 1.4  使用Xrdp+VNC

```
sudo apt install tightvncserver

echo unity > ~/.xsesstion

sudo apt install xrdp

sudo service xrdp restart

gsetttings set org.gnome.desktop.remote-access.requre-encryption false

```
然后在桌面上搜索 `Desktop sharing`,打开远程桌面，设置好密码。

* windows可用远程桌面
* 也可用VNC viewer 连接

# 2 终端连接

# 2. 1 xshell+ssh
正常建立即可，要注意的是：
* 小键盘设置为普通模式
* backspace和delete设置为backspace（ctrl+h）
# 2.2  xstart+ssh
xstar 运行命令 选择`gnome-terminal`



------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：<font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------