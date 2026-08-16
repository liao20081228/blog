---
title: Linux常用的管道命令
tags:   Linux使用教程
---

------

<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。网络转载请注明出处！！！</font>


&emsp;&emsp;***<font color=blue>版权声明</font>***：*本文章参考了<font color=blue >**[《鸟哥的Linux私房菜》](<http://linux.vbird.org "点击跳转")、[《Linux**命令**手册》](<http://linux.51yip.com "点击跳转")、[《Linux**命令**大全》](<http://man.linuxde.net "点击跳转")以及[《Linux man pages》](<https://linux.die.net/man/ "点击跳转")。**</font><font color=red>未经作者允许，**<font color=blue>严禁用于商业出版</font>**，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！*</font>

---


&emsp;&emsp;**管道命令**：就是可以使用管道`|`的**命令**。在管道**命令**当中，常常会使用到前一个指令的 stdout 作为这次的 stdin ， 某些指令需要用到文件名 (例如 tar) 来进行处理时，该 stdin 与 stdout 可以利用减号 "-"来替代。


#1 双向重定向——`tee`
&emsp;&emsp;**命令**：tee [-ai] FILE... 
&emsp;&emsp;**描述**：将标准输入数据重定向到文件FILE和标准输出。

>|常用选项|	作用|
|--|--|
|-a	|以追加的方式重定向到文件
|-i	|忽略终端信号
|更多信息|	<<http://linux.51yip.com/search/tee> 和 man 手册
|&emsp;&emsp;&emsp;&emsp;|	<<http://man.linuxde.net/tee>


#2 生成参数——`xargs`
&emsp;&emsp;**命令**：xargs [-0epn] [cmd] 
&emsp;&emsp;**描述**：从输入数据中生成指定参数传递给cmd，并执行cmd，cmd缺省为echo。

>|常用选项|	作用|
|--|--|
|-0	|将特殊字符作为普通字符，以NULL代替空格键来结束输入
|-e str|	当 xargs 分析到字符串str就会停止继续工作
|-p	|在执行每个指令的 argument 时，都会询问使用者的意思
|-n	|后面接次数，每次 command 指令执行时，要使用几个参数的意思。
|更多信息|	<http://linux.51yip.com/search/xargs> 和 man 手册
|&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;|	<http://man.linuxde.net/xargs>




---

&emsp;&emsp;***<font color=blue>版权声明</font>***：*本文章参考了<font color=blue >**[《鸟哥的Linux私房菜》](<http://linux.vbird.org "点击跳转")、[《Linux**命令**手册》](<http://linux.51yip.com "点击跳转")、[《Linux**命令**大全》](<http://man.linuxde.net "点击跳转")以及[《Linux man pages》](<https://linux.die.net/man/ "点击跳转")。**</font><font color=red>未经作者允许，**<font color=blue>严禁用于商业出版</font>**，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！*</font>

---




------

<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。网络转载请注明出处！！！</font>

------
