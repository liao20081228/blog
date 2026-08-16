---
title: Linux搭建nfs服务
tags: Linux使用教程
---

------

&emsp;&emsp;<font color=blue>**_版权声明_：**</font></font>本文章参考了[《鸟哥的Linux私房菜》](http://linux.vbird.org "点击跳转")、[《Linux命令手册》](http://linux.51yip.com "点击跳转")、[《Linux命令大全》](http://man.linuxde.net "点击跳转")以及[《Linux man pages》](https://linux.die.net/man/ "点击跳转")。<font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------


&emsp;&emsp;nfs服务（网络文件系统）是实现Linux和Linux之间的文件共享，nfs服务的搭建比较简单。

# 1 搭建服务端
* 安装nfs服务端
```shell
    sudo apt install nfs-kernel-server
```
* 修改配置文件`/etc/exports`，添加内容，格式：`共享目录  用户(权限)`
	* 用户   ：   指定哪些用户可以访问
		 * \* 所有可以ping同该主机的用户
		 * 192.168.1.*  指定网段，在该网段中的用户可以挂载
		 * 192.168.1.12 只有该用户能挂载
	 * 权限
		 * ro : 只读
		 * rw : 读写
		 * sync :  同步
		 * no_root_squash: 不降低root用户的权限
		 * 其他权限man 5 exports 查看
*  重启nfs服务
```shell
    sudo /etc/init.d/nfs-kernel-server restart
```


# 2 搭建客户端
* 安装NFS客户端
```shell
    sudo apt install nfs-common
```
* 检查客户端和服务端的网络是否连通
```shell
    ping +服务器IP
```
* 查看服务端的共享目录
```shell
    showmount -e + 服务器IP
```

* 将该共享目录挂载到本地
```shell
    mount 服务器ip:目录  本地挂载点 -t nfs
```
* 访问本地的挂载点，就可访问服务端共享的目录了。


* 开机自动挂载，修改`/etc/fstab`文件，格式如下：
```shell
 nfs-server-ip:共享目录   挂载点  文件类型 文件系统参数 允许dump指令   允许fsck指令
```

------

&emsp;&emsp;<font color=blue>**_版权声明_：**</font></font>本文章参考了[《鸟哥的Linux私房菜》](http://linux.vbird.org "点击跳转")、[《Linux命令手册》](http://linux.51yip.com "点击跳转")、[《Linux命令大全》](http://man.linuxde.net "点击跳转")以及[《Linux man pages》](https://linux.die.net/man/ "点击跳转")。<font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>


------

