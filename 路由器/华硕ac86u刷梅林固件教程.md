---
title: 华硕ac86u刷梅林固件教程
tags: 路由器,meilin,梅林
---

------

***<font color=blue>版权声明</font>：本文参考了<font color=blue>[《koolshare固件发布》](https://koolshare.cn/thread-127878-1-1.html)。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------

# 1 梅林固件是什么

**[官方固件](https://www.asus.com.cn/ASUSWRT/)**：就是在华硕官网支持页面下载的RT-AC86U固件。

**[梅林固件](https://www.asuswrt-merlin.net)**：为加拿大独立开发者Eric Sauvageau在华硕RT-AC86U官方开源的源码上二次开发的第三方固件，相较华硕官方固件。与官方固件的主要区别是修复了一些bug，增加了[一些功能](https://www.asuswrt-merlin.net/features "新增功能特性")。

**[官方改版固件](https://koolshare.cn/thread-139965-1-1.html)**：为koolshare开发组基于官方源码修改后编译发布的固件。与官方固件的主要区别就是加入了软件中心。

**[梅林改版固件](https://koolshare.cn/thread-127878-1-1.html)**：koolshare开发组基于梅林固件源码修改后编译发布的固件，与梅林固件的主要区别就是加入了软件中心。

>P.S.
>1. 更多有关路由器固件的知识请参阅《主流路由器固件发展史》
>2. 本文用改版梅林固件代替梅林固件


# 2 刷机教程

刷机步骤：

1. 下载固件。
2. 登录路由器固件升级页面：[【系统设置】→【固件升级】](http://router.asus.com/Advanced_FirmwareUpgrade_Content.asp)。
3. 上传名为`xxx.w`固件，等待重启完成。
4. 根据需要选择是否恢复出厂设置（也叫双清操作）:[【系统设置】→【恢复/导出/上传设置】](http://router.asus.com/Advanced_SettingBackup_Content.asp)。
	* 以下情况如无特殊说明，刷机完成后不用双清，当然双清后更好。
		* 官方固件 → 梅林改版固件
		* 梅林固件 → 梅林改版固件
		* 梅林改版固件 → 梅林改版固件
	* 以下情况刷机完成后建议双清
		* 官方改版固件 → 梅林改版固件
		* 梅林改版固件 → 官方固件
		* 梅林改版固件 → 官方改版固件
		* 梅林改版固件 → 梅林固件
5. 根据需要选择是否要格式化或启用软件中心
	* 以下情况需要格式化并启用软件中心：在【系统管理】–【系统设置】内勾选Format JFFS partition at next boot 和 Enable JFFS custom scripts and configs 然后点击应用本页面设置，成功后点击顶部【重启按钮】重启路由器。
		* 官方固件 → 梅林改版固件
		* 梅林固件 → 梅林改版固件
		* 官方改版固件 → 梅林改版固件，并双清
		* 梅林改版固件 → 官方改版固件，并双清
	* 以下情况只需要启用软件中心：在【系统管理】–【系统设置】内勾选Enable JFFS custom scripts and configs 然后点击应用本页面设置。
		* 梅林改版固件 → 梅林改版固件
		* 官方改版固件 → 梅林改版固件，并不双清
		* 梅林改版固件 → 官方改版固件，并不双清

>注意事项：
>
> * 刷机建议使用chrome浏览器，或者chromium内核的浏览器进行刷机操作；
> * 刷机后如提示手动重启路由器，请等待几分钟后再手动开关电源进行重启；
> * 跨大版本刷机，如384 → 386或386 → 384，建议刷机完成后做一次固件双清操作；
> * 刷机后如果界面显示不正常，请使用组合键ctrl + F5强制清空浏览器缓存后重试；
> * 单纯的恢复出厂设置并不能完全清除jffs分区的软件中心文件，需要在恢复出厂设置的时候同时勾选恢复按钮旁边的选择框。

------

***<font color=blue>版权声明</font>：本文参考了<font color=blue>[《koolshare固件发布》](https://koolshare.cn/thread-127878-1-1.html)。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
