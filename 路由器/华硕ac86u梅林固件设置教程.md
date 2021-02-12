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
* Greter：大于，信号强于。用于触发2.4→G5G。
* Less：小于，信号弱于。用于触发5G→2.4G。

**注意事项**
* Smart Connect功能并不能完美，因为无线终端设备在某些时候会自主选择2.4G或5G频段。
* 设置Smart Connect规则时，不要参考ASUS固件中【System Log】→【Wireless Log】中的RSSI数据和Rx/Tx速率数据


**准备工作**：开启Smart Connect功能之前，你需要根据你自己家里环境，测出适合你能用的Smart Connect规则RSSI切换阈值。
1. 关闭Smart Connet，并且为2.4G和5G分别设置不同的SSID。
2. 用一个高通CPU的双频手机，在手机电话拨号界面输入【*#*#4636#*#*】，依次进入Sta【Wi-Fi information】、【Wi-Fi status】，然后一边远离路由器一边点击【Refresh stats】。当【NetWork State】由已连接变为未连接时，记录下RSSI值。这个值会变化，但是浮动不大，多刷几次取个均值，例如-80dbm，这就是5G向2.4G切换的阈值。要注意的是实际上此处的5G信号已经极差了，数据传输也即将中段，因此应当让切换提前发生，通常在这个值之上加5-10，例如-73dbm。
3. 原地不动，将Wi-Fi手动切换到2.4频段，点击【Refresh stats】，重新记录RSSI。和上面一样，去一个均值，例如-60 dBm，这就是2.4G向5G切换的阈值。要注意的是实际上此处5G信号其实极差，因此即使切到5G信号也不好，因此应当让切换延后发生，通常在这个值之上加10~20，例如-50dbm。


**设置智能切换规则**：点击【Wireless】→【General】→【Smart Connect Rule】或者点击【Network Tools】→【Smart Connect Rule】。Smart Connect切换规则由四部分构成，当且仅当这四个部分都满足的时候，才发生切换，只要有一个不满足都停留在当前频段。
1. 【steering Trigger Condition】：控制触发条件，当且仅当以下条件都为真时才触发切换程序。
	* 【band】：当前频段。
	* 【Enable Load Balance】：负载均衡。开启后该频段的所有终端网速均分（这个不是条件）。
	* 【Bandwidth Utilization】	：带宽使用率。该频段所有连接的终端设备带宽使用量之和超过该值时条件为真。通常2.4G默认为0，5G默认80%，如果无线设备不多时可以都设置为0。
	* RSSI：接受信号强度。
	* PHY Rate Less：终端最大网速小于设定值为真。
	* PHY Rate Greater：终端最大网速大于设定值为真。
	* VHT：超高吞吐量。

* STA Selection（终端选择）
* Interface Select and Qualify Procedures：接口选择与质量审核
* Bounce detect：来回切换检查

实际上是一个且运算表达式。









**开启Smart Connect**：点击【Wireless】→【General】→【Enable Smart Connect】设置为【On】。

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
