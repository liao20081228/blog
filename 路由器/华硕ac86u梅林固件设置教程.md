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
* PHY（Physical）：物理层。
* STA（Station）：站点、客户端（无线终端设备））
* VHT（Very High Throughput）：超高吞吐量，802.11ac就属于VHT技术
* Greater：大于，信号强于。用于触发2.4G→5G。
* Less：小于，信号弱于。用于触发5G→2.4G。

**注意事项**：
* Smart Connect功能并不能完美，因为无线终端设备在某些时候会自主选择2.4G或5G频段。
* 设置Smart Connect规则时，不要参考ASUS固件中【System Log】→【Wireless Log】中的RSSI数据和Rx/Tx速率数据

**准备工作**：开启Smart Connect功能之前，你需要根据你自己家里环境，测出适合你能用的Smart Connect规则RSSI切换阈值。
1. 关闭Smart Connet，并且为2.4G和5G分别设置不同的SSID。
2. 用一个高通CPU的双频手机，在手机电话拨号界面输入【*#*#4636#*#*】，依次进入Sta【Wi-Fi information】、【Wi-Fi status】，然后一边远离路由器一边点击【Refresh stats】。当【NetWork State】由已连接变为未连接时，记录下RSSI值。这个值会变化，但是浮动不大，多刷几次取个均值，例如-80dbm，这就是5G向2.4G切换的阈值。要注意的是实际上此处的5G信号已经极差了，数据传输也即将中段，因此应当让切换提前发生，通常在这个值之上加5-10，例如-73dbm。
3. 原地不动，将Wi-Fi手动切换到2.4频段，点击【Refresh stats】，重新记录RSSI。和上面一样，去一个均值，例如-60 dBm，这就是2.4G向5G切换的阈值。要注意的是实际上此处5G信号其实极差，因此即使切到5G信号也不好，因此应当让切换延后发生，通常在这个值之上加10~20，例如-50dbm。


**理解切换规则**：Smart Connect切换规则由四部分构成，***当且仅当这四个部分都为真*** 时才会成功切换，只要有一个不满足终端都停留在当前频段。
1. 【Trigger Conditions】：触发条件集，用于触发切换流程。***当且仅当所有触发条件都为真时，【Trigger Conditions】才为真***，才会触发切换流程。
	* 【Load Balance】：启用负载均衡，所有终端均分当前频段带宽（不是触发条件，但会影响终端获得的带宽）。
	* 【Bandwidth Utilization】：本频段的带宽使用率。当该频段所有连接的终端设备带宽使用量之和超过设定值时为真。
	* 【RSSI】：终端接收信号强度。
		* 【Greater】：RSSI强于设定值时为真。用于2.4G→5G。
		* 【Less】：RSSI弱于设定值时为真。用于5G→2.4G。
	* 【PHY Rate Less】：终端与路由器协商的带宽小于设定值为真。（有人说这个设计带宽或实际速率，都是错的），这需要了解无线通信的知识，在终端的WIFI状态可以看到：，随着信号的衰减，协商的带宽也在变小，简单说来终端协商的带宽往往大于实际速率。0表示禁用，该条件始终为真。
	* 【PHY Rate Greater】：终端与路由器协商的带宽大于设定值为真。0表示禁用，该条件始终为真。
	* 【VHT】：超高吞吐量，802.11ac才有的技术。
		* 设置为【all】：不对终端做VHT要求，该条件始终为真。
		* 设置为【AC-only】：只有终端支持802.11ac时该条件为真。
		* 设置为【not-allowed】：只有终端不支持VHT时该条件为真。
2. 【STA Selection Policy】：终端选择策略，用于选择将要切换的终端，***当且仅当终端满足以下所有条件都为真时，【STA Selection Policy】才为真***，该终端才能被选中。通常与1中设置相同。
	* 【RSSI】
	* 【PHY Rate Less】
	* 【PHY Rate Greater】
	* 【VHT】
3. 【Interface Select and Qualify Procedures】：接口选择与质量审核。当且 ***当且仅当以下所有条件都为真***。
	* 【Bandwidth Utilization】：目标频段的带宽使用率。当目标频段所有连接的终端设备带宽使用量之和小于设定值时为真。
	* 【VHT】：超高吞吐量，802.11ac才有的技术。
		* 设置为【all】：不对终端做VHT要求，该条件始终为真。
		* 设置为【AC-only】：只有终端支持802.11ac时该条件为真。
		* 设置为【not-allowed】：只有终端不支持VHT时该条件为真。
4. 【Bounce detect】：来回切换检查。当且仅当在设定的连续时间内，切换次数不超过设定的次数，且不处于冷却状态时，条件为真。
	* 【Windos Time】：窗口时间，计算切换次数的连续时间。
	* 【Count】：在设定的连续时间内，最大切换次数。
	* 【Dewell Time】：冷却时间。在【Windos Time】设定的时间内，如果切换超过【【Count】设定的次数，则在接下来在【Dewell Time】设定的时间内处于冷却状态。

![7](https://gitee.com/liao20081228/blog_pictures/raw/master/华硕ac86u梅林固件设置教程/7..JPG#pic_center)

**设置智能连接规则**：点击【Wireless】→【General】→【Smart Connect Rule】或者点击【Network Tools】→【Smart Connect Rule】。
1. 【Steering Trigger Conditions】：
	* 【Enable Load Balance】：设为【No】。
	* 【Bandwidth Utilization】：2.4G设置为0，5G设置为80%。
	* 【RSSI】：2.5G设置为【Greater】-50dbm，5G设置为【Less】-73dbm。
	* 【PHY Rate Less】：2.4G设为【Disable】，5G设为。
	* 【PHY Rate Greater】：2.4G设为150Mbps，5G设为【Disable】
	* 【VHT】：【all】【AC-only】【not-allowed】
2. 【STA Selection Policy】：
	* 【RSSI】：2.5G设置为【Greater】-50dbm，5G设置为【Less】-73dbm。
	* 【PHY Rate Less】：2.4G设为【Disable】，5G设为。
	* 【PHY Rate Greater】：2.4G设为150Mbps，5G设为【Disable】
	* 【VHT】：【all】【AC-only】【not-allowed】
3. 【Interface Select and Qualify Procedures】：
	* 【Bandwidth Utilization】：目标频段2.4G设为80%，目标频段5G设为80%
	* 【VHT】：【all】【AC-only】【not-allowed】
4. 【Bounce detect】：
	* 【Windos Time】：设为60秒
	* 【Count】：设为2次。
	* 【Dewell Time】：设为180秒。

**【PHY Rate】和【VHT】究竟应该怎么设置？**

现行无线标准主要有802.11a、802.11b、802.11g、802.11n、802.11ac、802.11ax。其特性如下表所示：

|标准版本|802.11a|802.11b|802.11g|802.11n(HTW)|802.11ac(VHTW)|802.11ax（HEW）|
|:--|:--|:--|:--|:--|:--|:--|
|**发布时间**|1999|1999|2003|2009|2012(波1)<br />2016(波2)|2018|
|**工作频段（GHz）**|5|2.4|2.4|2.4、5|5|2.4、5|
|**信道宽度（MHz）**|20|22|20|20、20+20/40|20、40、80、160/80+80|20、40、80、160/80+80|
|**单流最大带宽（Mbps）**|54|11|54|20MHz：72.2<br />40MHz：150|20MHz：86.7<br />40MHz：200<br />80MHz：433.3<br />160MHz：866|20MHz：143.4<br />40MHz：286.8<br />80MHz：600.4<br />160MHz：1201|
|**最大空间流数**|1|1|1|4x4|8x8|8x8|

>P.S. 
>802.11ac还衍生出了TurboQAM和NitroQAM两种非标技术。
>1. TurboQAM技术主要用在2.4GHz频段上，可以让IEEE 802.11n标准改用IEEE 802.11ac的256-QAM调制模式，与64-QAM相比，在20MHz可以提升20%带宽，在40MHz可以提升33%带宽。
>2. NitroQAM则相当于TurboQAM二代，可以让IEEE 802.11n/ac使用1024-QAM调制模式，与TurboQAM相比，在20MHz可以提升20%带宽，在40MHz及以上可以提升25%带宽。
>
>虽然TurboQAM和NitroQAM可以提升终端2.4G带宽，但这要求终端和WIFI都要支持这些技术；此外TurboQAM和NitroQAM需要使用到802.11ac的调制模式，也就是说但凡支持TurboQAM技术的终端都必然支持IEEE 802.11ac标准。因此在设置Smart Connect时不需要考虑这个问题的。

问题就出在802.11n上，由于802.11n设备可以工作在2.4G和5G频段，并且有些设备两个频段都支持，有些只支持其中一个。如果不设置限制条件，路由器将只支持2.4G 802.11n的终端在2.4G切向5G时，首先将终端从2.4G下线，之后连接5G频段。但是由于终端不支持5G 802.11n，因此该设备将无网络可用。同理，只支持5G 802.11n的终端在5G切向2.4G时也无网络可用。因此我们通常通过【PHY Rate】和【VHT】组合使用来这样的特殊设备。此外，由于MIMO的出现，这就导致这些设备的带宽不是固定的，因此没有统一值，只有适合自己环境的最优值。

先看看802.11n的最大带宽：

|天线数|带宽（20Hz）|带宽（40Hz）|
|:--|:--|:--|
|1|72.2Mbps|150Mbps|
|2|144.4Mbps|300Mbps|
|3|216.7Mbps|450Mbps|
|4|288.9Mbps|600Mbps|

考虑2.4G向5G切换时拥有不同天线数的各设备的情况：

|驻留2.4G的终端|PHY为55Mbps|PHY为75Mbps|PHY为150Mbps|PHY为225Mbps|PHY为300Mbps|
|:--|:--|:--|:--|:--|:--|
|802.11ac|1/2/3/4天线切换|1天线不切换，2/3/4天线切换|1/2天线不切换，3/4天线切换|1/2/3天线不切换，4天线切换|1/2/3/4天线不切换|
|802.11n 2.4|1/2/3/4天线切换但无网络|1天线不切换，2/3/4天线切换但无网络|1/2天线不切换，3/4天线切换但无网络|1/2/3天线不切换，4天线切换|1/2/3/4天线不切换但无网络|
|802.11n 2.4/5|1/2/3/4天线切换|1天线不切换，2/3/4天线切换|1/2天线不切换，3/4天线切换|1/2/3天线不切换，4天线切换|1/2/3/4天线不切换|

建议如下：
* 如果没有仅支持802.11n 2.4G的设备时，则只需把【PHY Rate Greater】、【PHY Rate less】设置为【Disable】，【VHT】设为【all】即可。
* 如果有仅支持802.11n 2.4G的设备时，有两种办法：
	* 1. 把【PHY Rate Greater】、【PHY Rate less】设置为【Disable】，【VHT】设为【AC only】即可。但这样会造成802.11n 2.4/5G终端无法切换到5G，当然即使有这种设备驻留也没有太大影响，毕竟需要的带宽不高。
	* 2. 把【PHY Rate Greater】设置为这些802.11n 2.4G终端设计带宽的最大值即可，【PHY Rate less】设置为【Disable】，【VHT】设为【all】。但同样可能会导致支持802.11ac和802.11n 2.4/5G一直驻留。

考虑5G向2.4G切换的情况：

|终端支持无线标准|PHY为Mbps|PHY为75Mbps|PHY为150Mbps|PHY为225Mbps|PHY为300Mbps|
|:--|:--|:--|:--|:--|:--|
|802.11ac/n2.4/n5|1/2/3/4天线切换|1天线不切换，2/3/4天线切换|1/2天线不切换，3/4天线切换|1/2/3天线不切换，4天线切换|1/2/3/4天线不切换|
|802.11ac/n5|1/2/3/4天线切换|1天线不切换，2/3/4天线切换|1/2天线不切换，3/4天线切换|1/2/3天线不切换，4天线切换|1/2/3/4天线不切换|
|802.11n5|1/2/3/4天线切换但无网络|1天线不切换，2/3/4天线切换但无网络|1/2天线不切换，3/4天线切换但无网络|1/2/3天线不切换，4天线切换|1/2/3/4天线不切换但无网络|
|802.11n 2.4/5|1/2/3/4天线切换|1天线不切换，2/3/4天线切换|1/2天线不切换，3/4天线切换|1/2/3天线不切换，4天线切换|1/2/3/4天线不切换|


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
