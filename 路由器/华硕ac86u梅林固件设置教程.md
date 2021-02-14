---
title: 华硕ac86u梅林固件设置教程
tags: 路由器,meilin,梅林
---


------

***<font color=blue>版权声明</font>：本文参考了<font color=blue>。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------


# 1 基本设置
进入[设置页面](router.asus.com/)。语言选择英文。

1. 点击【Quick Internet Setup】并选择【Create New NetWork】。
2. 设置WIFI名和密码。如果需要分别2.4G和5G的WIFI名和密码，请勾选【Separate 2.4GHz and 5GHz】进行设置。
3. 输入路由器管理员用户名和密码。

# 2 设置访客网络

访客网络就是只能访问WAN网络，而不能访问同一局域网设备的网络。如果经常有陌生设备需要连接WIFI则则建议开启访客网络，一般家庭网络无需开启。

1. 点击【Guest Network】→【Enable】。
2. 设置无线参数。
	* 【Hide SSID】：隐藏WIFI名。通常选择【No】。
	* 【Network Name（SSID）】：WIFI名
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
2. 设置设置无线WIFI的基本信息。
	* 【Hide SSID】：隐藏WIFI名，通常选择【No】。
	* 【Network Name（SSID）】：设置WIFI名
	* 【Authentic Method】：连接WIFI时是否需要密码及如何加密。通常选择【WPA2-Personal】
	* 【WPA Encryption】：加密方式。通常选择【AES】。
	* 【WPA Pre-shared key】：WIFI密码
	* 【Wireless Mode】：无线模式。设置为【Auto】
	* 【Protected Management Frames】：启用保护帧，可以防止窃听和伪造。通常设置【Disable】


>如果终端不支持AES加密，则【Authentic Method】需要选择【WPA2-auto-Personal】，【WPA Encryption】需要选择【TKIP+AES】，AES比TKIP更安全，且速度更快。
>
>如果希望每个用户都有自己的用户名和密码，则【Authentic Method】需要选择【WPA2-Enterprise】或【WPA-auto-Enterprise】版本，这需要设置radius服务器的地址和端口及连接密码，这通常用于企业，同样你也需要搭建一个radus服务器。

## 3.2 设置智能连接

Smart Connect能将多个频段用一个SSID，并根据预先设定的规则进行频段间的自动切换。

为什么需要开启Smart Connect？因为2.4G WIFI速度慢，但穿透能力强，覆盖范围大；而5G WIFI速度更快，穿透能力更弱，覆盖范围更小。因此当终端由5G覆盖域移动到2.4G覆盖域时，需要将WIFI由2.4G切换为5G，反之则需要切回来。之前这个过程是由用户手动完成的，开启Smart Connect后，这个过程将由路由器自动完成。

**注意事项**：
* Smart Connect功能并不能完美，因为无线终端设备在某些时候会自主选择2.4G或5G频段。
* 设置Smart Connect规则时，不要参考ASUS固件中【System Log】→【Wireless Log】中的RSSI数据和Rx/Tx速率数据，而应该参考终端设备侧的值。

**准备工作**：开启Smart Connect功能之前，你需要根据你自己家里环境，测出适合你能用的Smart Connect规则RSSI切换阈值。
1. 关闭Smart Connet，并且为2.4G和5G分别设置不同的SSID。
2. 用一个高通CPU的双频手机，在手机电话拨号界面输入【*#*#4636#*#*】，依次进入Sta【Wi-Fi information】、【Wi-Fi status】，然后一边远离路由器一边点击【Refresh stats】。当【NetWork State】由已连接变为未连接时，记录下RSSI值。这个值会变化，但是浮动不大，多刷几次取个均值，例如-80dbm，这就是5G向2.4G切换的阈值。要注意的是实际上此处的5G信号已经极差了，数据传输也即将中段，因此应当让切换提前发生，通常在这个值之上加5-10，例如-73dbm。
3. 原地不动，将Wi-Fi手动切换到2.4频段，点击【Refresh stats】，重新记录RSSI。和上面一样，去一个均值，例如-60 dBm，这就是2.4G向5G切换的阈值。要注意的是实际上此处5G信号其实极差，因此即使切到5G信号也不好，因此应当让切换延后发生，通常在这个值之上加10~20，例如-50dbm。

**理解切换流程**：Smart Connect切换流程由三部分构成：触发切换流程、选择待切换终端、向目标频段切换。这三步想要完成则需要满足各自的约束条件，且不处于冷却状态。这就涉及四组规则：
1. 【Steering Trigger Condition】：控制触发条件。***当且仅当以下所有规则都满足时***，才会触发切换流程。
	* 【Load Balance】：启用负载均衡，所有终端均分当前频段带宽（不是条件，但会影响终端获得的带宽）。通常设置为【No】。
	* 【Bandwidth Utilization】：本频段带宽使用率。要求该频段所有连接的终端设备带宽使用量之和超过设定值。通常设置为【0】。
	* 【RSSI】：终端接收信号强度。
		* 【Greater】：要求RSSI强于设定值。通常2.4G设置为【-50】。
		* 【Less】：要求RSSI弱于设定值。通常5G设置为【-70】。
	* 【PHY Rate Less】：要求终端与路由器协商的带宽小于设定值。（有人说这个设计带宽或实际速率，都是错的）。通常设置【0】表示禁用该规则。
	* 【PHY Rate Greater】：要求终端与路由器协商的带宽大于设定值。通常设置【0】表示禁用该规则。
	* 【VHT】：超高吞吐量，要求终端是否支持802.11ac。通常设置为【AC-only】或【all】。
		* 设置为【all】：不要求终端支持。
		* 设置为【AC-only】：要求终端支持。
		* 设置为【not-allowed】：要求终端不支持。
2. 【STA Selection Policy】：终端选择策略，用于选择将要切换的终端，***当且仅当终端满足以下所有规则时***，才能选出待切换终端。通常与1中设置相同。
	* 【RSSI】
	* 【PHY Rate Less】
	* 【PHY Rate Greater】
	* 【VHT】
3. 【Interface Select and Qualify Procedures】：接口选择与质量审核。***当且仅当目标频段和终端满足以下所有规则时***，终端才会被切换到目标频段。
	* 【Bandwidth Utilization】：目标频段的带宽使用率。要求目标频段所有连接的终端设备带宽使用量之和小于设定值。通常设置为【80%】.
	* 【VHT】：通常和1设置相同。
4. 【Bounce detect】：来回切换检查。要求**设定的连续时间内，切换不超过设定次数**，否则终端处于冷却指定时间。
	* 【Windos Time】：窗口时间。通常设置为【60】。
	* 【Count】：最大切换次数。通常设置为【2】。
	* 【Dewell Time】：冷却时间。通常设置为【180】。

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
