---
title: 华硕ac86u梅林固件设置教程
tags: 路由器,meilin,梅林
---


------

***<font color=blue>版权声明</font>：本文参考了<font color=blue>。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------


# 1 基本设置
进入[设置页面](router.asus.com/)。

1. 点击【Quick Internet Setup】并选择【Create New NetWork】。
![1](https://gitee.com/liao20081228/blog_pictures/raw/master/华硕ac86u梅林固件设置教程/1.jpg#pic_center)
2. 设置WIFI名和密码。如果需要分别2.4G和5G的WIFI名和密码，请勾选[Separate 2.4GHz and 5GHz]进行设置。
![2](https://gitee.com/liao20081228/blog_pictures/raw/master/华硕ac86u梅林固件设置教程/2.jpg#pic_center)

3. 输入路由器管理员用户名和密码。

# 2 设置访客网络

访客网络就是只能访问WAN网络，而不能访问同一局域网设备的网络。如果经常有陌生设备需要连接WIFI则则建议开启访客网络，一般家庭网络无需开启。

1. 点击【Guest Network】→【Enable】。
![4](https://gitee.com/liao20081228/blog_pictures/raw/master/华硕ac86u梅林固件设置教程/4.jpg#pic_center)
2. 设置无线参数。
![5](https://gitee.com/liao20081228/blog_pictures/raw/master/华硕ac86u梅林固件设置教程/5.jpg#pic_center)
	* 【Hide SSID】：隐藏WIFI名，通常选择【No】。
	* 【Network Name（SSID）】：设置WIFI名
	* 【Authentic Method】：连接WIFI时是否需要密码及如何加密。通常选择【WPA-Personal】和【AES】，如果终端不支持AES加密，则选择【WPA2-auto-Personal】和【TKIP+AES】，AES比TKIP更安全，且速度更快。
	* 【Access time】：允许上网多久。通常【Unlimited Acess】。
	* 【Bandwidth limiter】：是否限制带宽。通常选择【Disable】。
	* 【Access Intranet】：是否可以访问内部网络。通常选择【否】。
	* 【Sync to AiMesh Node】：是否同步到AiMesh节点。通常选择【Router Only】，不同步。
	* 【Enable MAC Filter】：根据MAC地址设置允许连接或不允许连接终端。通常选择【Disable】

# 3 设置无线网络

## 3.1 设置无线基本信息

除了【Quick Internet Setup】，该【Wireless】界面也可以设置无线WIFI的基本信息。

1. 点击【Wireless】→【General】
![6](https://gitee.com/liao20081228/blog_pictures/raw/master/华硕ac86u梅林固件设置教程/6.jpg#pic_center)
2. 设置设置无线WIFI的基本信息。
	* 【Hide SSID】：隐藏WIFI名，通常选择【No】。
	* 【Network Name（SSID）】：设置WIFI名
	* 【Authentic Method】：连接WIFI时是否需要密码及如何加密。通常选择【WPA2-Personal】
	* 【WPA Encryption】：加密方式。通常选择【AES】
	* 【WPA Pre-shared key】：WIFI密码

>如果终端不支持AES加密，则【Authentic Method】需要选择【WPA2-auto-Personal】，【WPA Encryption】需要选择【TKIP+AES】，AES比TKIP更安全，且速度更快。
>
>如果希望每个用户都有自己的用户名和密码，则【Authentic Method】需要选择【WPA2-Enterprise】或【WPA-auto-Enterprise】版本，这需要设置radius服务器的地址和端口及连接密码，这通常用于企业，同样你也需要搭建一个radus服务器。

## 3.2 设置智能连接

Smart Connect能将多个频段用一个SSID，并根据预先设定的规则进行频段间的自动切换。其他路由器厂商也有类似功能，名字不一样，比如：小米路由器叫“双频合一”，TP-LINK路由器叫“频谱导航”。

为什么需要开启Smart Connect？因为2.4G WIFI速度慢，但穿透能力强，覆盖范围大；而5G WIFI速度更快，穿透能力更弱，覆盖范围更小。因此当终端由5G覆盖域移动到2.4G覆盖域时，需要将WIFI由2.4G切换为5G，反之则需要切回来。之前这个过程是由用户手动完成的，开启Smart Connect后，这个过程将由路由器自动完成。

**基本术语**：
* RSSI（Received SignalStrength Indicator）接收信号强度指示。RSSI是一个负数，越大信号越强。
* PHY（Physical）：物理（网卡）。
* STA（Station）：站点、客户端（无线终端设备））
* VHT（Very High Throughput）：超高吞吐量，802.11ac就属于VHT技术

**注意事项**
* Smart Connect功能并不能完美，因为无线终端设备在某些时候会自主选择2.4G或5G频段。
* 设置Smart Connect规则时，不要参考ASUS固件中【System Log】→【Wireless Log】中的RSSI数据和Rx/Tx速率数据！


**准备工作**：开启Smart Connect功能之前，你需要根据你自己家里环境，测出适合你能用的Smart Connect规则RSSI切换阈值。
1. 用一个高通CPU的双频手机，移动到Wi-Fi切换为2.4G断开连接了，要5G Wi-Fi勉强还能用），在手机电话拨号界面输入【*#*#4636#*#*】，依次进入【Wi-Fi information】、【Wi-Fi status】，点击【Refresh stats】刷新当前Wi-Fi连接状态，记录手机屏幕上显示的RSSI值。刷新时RSSI数值也会波动，一般上下波动5dBm左右，记录信号最强的那个值。比如我自己的是-83 dBm，为了稳妥在这个基础上，再加5dBm，就可以设置成你的5G切换阈值，所以我的设置成 -78 dBm；

③还站在原地不动，将安卓手机Wi-Fi断开5G频段，连接步骤①设置好的2.4频段，重新在电话拨号界面输入【*#*#4636#*#*】，进入【Wi-Fi information】、【Wi-Fi status】，点击【Refresh stats】刷新当前Wi-Fi连接状态，记录安卓手机屏幕上显示的RSSI值。和上面一样，记录最强的值，比如我的是-59 dBm，为了稳妥在这个基础上，再加15 dBm，就可以设置成你的2.4G切换阈值，所以我的设置成 -44 dBm。
（为了减少切换次数，也可以将2.4G的RSSI切换阈值设定的更强一些，比如在测量结果基础上加15 dBm，甚至加20 dBm）



ASUS固件ASUS固件在路由器管理界面【系统记录】、【无线用户】中，看到的RSSI的值和Rx/Tx很容易让人进入误区，里面显示的RSSI值因计算方式不同，不能用于Smart Connect规则设置中。
大家可以参照我上述安卓手机测试方法与【无线用户】显示的RSSI值进行对比，分别设置在Smart Connect规则中，看哪个RSSI值效果更好，用实践来验证。我自己的验证结果，安卓手机测出来的RSSI值效果远远好于路由器端显示的RSSI值。
华硕RT-AC88U原厂固件ASUSWRT中没有【无线用户】这个界面，不存在这个问题。

1. 点击【Wireless】→【General】→【Enable Smart Connect】设置为【On】。
2. 点击【Wireless】→【General】→【Smart Connect Rule】或者点击【Network Tools】→【Smart Connect Rule】
3. 设置智能切换规则。






## 3.2 关闭WPS

## 3.3. 设置无线MAC过滤

## 3.4 设置漫游过滤

# 4 DDSN设置

# 5 虚拟内存

# 6 超大JFFS设置

# 7 双拨设置

#  DNS设置

------

***<font color=blue>版权声明</font>：本文参考了<font color=blue>。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
