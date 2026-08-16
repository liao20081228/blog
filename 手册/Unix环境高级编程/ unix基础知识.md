---
title: unix基础知识 
tags: unix环境高级编程
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>《Unix环境高级编程》。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

<style>table{word-break:initial;}</style>



# 1 引言
# 2 Unix体系结构
&emsp;&emsp;**操作系统**就是一种控制计算机硬件资源，提供程序运行环境的软件。
&emsp;&emsp;**系统调用**：内核的接口。
&emsp;&emsp;**Unix的体系结构**由内到外为：内核、系统调用、shell和函数库、用户程序。
# 3 登录
&emsp;&emsp;登录时，操作系统依据用户名查看口令文件/etc/password。
&emsp;&emsp;**口令文件**由登录项组成。每个登录项由7个以冒号分隔的字段组成。格式如：`用户名:加密口令：数值用户ID：数值组ID：注释：起始目录：shell程序`。
&emsp;&emsp;shell程序是命令行解释器，读取用户输入，执行命令。其输入可以是键盘输入或者shell脚本。 一般位于/bin/*sh。Linux 常用的是bourne-again shell。
&emsp;&emsp;用户登录后系统依据登录项中的最后一字段来启用哪种shell程序。
# 4 文件和目录
&emsp;&emsp;**文件系统**：Unix采用目录和文件的层次结构。起点为根目录。名字为“/”。
&emsp;&emsp;**目录**：目录是包含目录项的文件。每个目录项包含文件名和文件属性。每个目录中都自动创建两个文件“.”和“..”，分别指向当前目录和其父目录。
&emsp;&emsp;**文件**：数据的集合，包含文件名和数据。文件名不能包含字符/和字符NUL。
&emsp;&emsp;**路径名**：由/分隔或开头的多个文件名组成的序列。以根目录开始的称为绝对路径，否则，称为相对路径。
&emsp;&emsp;**工作目录**：当前正在使用的目录称为（当前）工作目录，相对路径由当前工作目录开始解释。
&emsp;&emsp;**起始目录**：登录时的当前目录。由口令文件的登录项中设置。
# 5 输入和输出
&emsp;&emsp;**文件描述符**：。进程启动时，shell自动为其打开标准输入（STDIN_FILENO）、标准输出（STDOUT_FILENO)和标准出错(STDERR_FILENO)3个文件描述符。
&emsp;&emsp;**不带缓冲的IO**：使用文件描述符进行IO操作。
&emsp;&emsp;**标准IO**：标准IO为不带缓冲的IO函数提供了带缓冲的接口。
# 6 程序与进程
&emsp;&emsp;**程序**：在外存中的可执行文件。内核使用exec函数将程序载入内存并执行。
&emsp;&emsp;**进程**：程序执行的实体，是分配资源的基本单位。**进程ID**：系统用于唯一标识进程的非负整数。
&emsp;&emsp;**进程控制**：fork、exec族、popen、system、exit、wait和waitpid。
&emsp;&emsp;**线程**：线程是轻量级进程，同一进程内的线程共享地址空间、文件描述符等资源。但具有独立的函数栈。**线程ID**：用于进程内唯一标识某线程的非负整数。
# 7 出错处理
&emsp;&emsp;头文件<errno.h>定义errrno及其可以赋值的常量，以E开头。
&emsp;&emsp;两条规则：（1）如果不出错，errno的值不会清除，（2）不要把errno设为0.
&emsp;&emsp;出错分为：致命错误和非致命错误。致命错误不可恢复，非致命错误可恢复，典型办法是稍后重试。非致命错误包括：EAGAIN、ENFILE、ENOBUFS、ENOLCK、ENOSPC、EWOULDBLOCK、ENOMEM、EBUSY、EINTR。
```c
#include<string.h>

char* strerror(int errnum)
```
&emsp;&emsp;**strerror函数**返回errnum错误对应的出错消息。
```c
#include<stdio.h>

void perror(const char *msg)
```
&emsp;&emsp;**perror函数**输出msg所指的字符串，然后输出冒号、空格、errno对应的出错消息和换行符。

# 8 用户标识
&emsp;&emsp;**用户ID**：OS用于唯一标识用户的非负整数，存放在口令文件登录项。用户不能更改自己用户ID。UID=0的用户为超级用户，登录名为root。
&emsp;&emsp;**组ID**：OS用于唯一标识用户组的非负整数。方便组内成员资源共享。组文件/etc/group将组名映射为组ID。
&emsp;&emsp;**附属组ID**：一个用户可以属于多个组。其它组的组ID的集合叫做附属组ID。
# 9 信号
&emsp;&emsp;**信号**：用于通知进程发生了某种状况。
&emsp;&emsp;进程收到信号后的反应：忽略、默认处理、自定义处理。
&emsp;&emsp;终端产生信号的办法：CTRL+C发出中断信号，CTRL+\发出退出信号并产生core文件。
# 10 时间值
&emsp;&emsp;Unix使用日历时间和进程时间。

&emsp;&emsp;**日历时间**：自1970年1月1日0时0分0秒以来经过的秒数累计值。用系统基本数据类型time_t存放。
&emsp;&emsp;**进程时间**：以CPU的时钟滴答计算。用系统基本数据类型clock_t存放。

&emsp;&emsp;Unix为一个进程维护三个时间：时钟时间、用户CPU时间、系统CPU时间。
&emsp;&emsp;同时有三个计时器：真实计时器（记录时钟时间）、虚拟计时器（记录用户CPU时间）、实用计时器（记录用户CPU时间+系统CPU时间）。
# 11 系统调用与库函数
&emsp;&emsp;系统调用是内核的入口。和库函数有本质区别。但用户并不关心。
&emsp;&emsp;区别：
&emsp;&emsp;&emsp;&emsp;(1)本质不同。
&emsp;&emsp;&emsp;&emsp;(2)职责不同。
&emsp;&emsp;&emsp;&emsp;(3)用户可以调用库函数和系统调用，很多库函数则会调用系统调用。
&emsp;&emsp;&emsp;&emsp;(4)系统调用功能单一，库函数功能复杂。


------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>《Unix环境高级编程》。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
