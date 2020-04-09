---
title: vivado与zynq开发教程
tags: FPJA与数字电路
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

<style>table{word-break:initial;}</style>


# 1 简介

Vivado设计套件，是FPGA厂商赛灵思公司2012年发布的集成设计环境。包括高度集成的设计环境和新一代从系统到IC级的工具，这些均建立在共享的可扩展数据模型和通用调试环境基础上。这也是一个基于AMBA AXI4 互联规范、IP-XACT IP封装元数据、工具命令语言(TCL)、Synopsys 系统约束(SDC) 以及其它有助于根据客户需求量身定制设计流程并符合业界标准的开放式环境。赛灵思构建的的Vivado 工具把各类可编程技术结合在一起，能够扩展多达1 亿个等效ASIC 门的设计。

# 2 PL开发流程

1. 新建一个工程， 选定开发板
2. 添加或编写源文件
3. 添加或编写约束文件
5. 综合（synthesisi）：配置实际的门电路
6. 布线（implementation）：连线
7. 生成（generate bitstream）：生成配置FPJA的二进制流
	* 无需固化：（progam device）：将bit文件写入开发板
	* 需要固化：generate bitstream，选项添加-bin_file,然后add configure memory device
 

如果需要仿真：

1. 添加或编写仿真文件
2. 综合前仿真（run behavioral simulation）：验证代码的逻辑性，不加任何的时延信息。
3. 综合后仿真（post synthesisi simulation）：
4. 布线后仿真（postimplementation simulation）：





# 3 PL-PS开发流程

![图1](https://www.github.com/liao20081228/blog/raw/master/图片/vivado教程/图1.jpg)


# 4 zynq7020供电系统




------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
