---
title: Linux登陆文件分析
tags: Linux使用教程
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

# 1 登陆文件简介
&emsp;&emsp;登陆文件：记录系统在什么时候由哪个程序做了什么样的行为时，发生了何种的事件等等。
* 登陆文件的重要性
	* 解决系统方面的错误
	* 解决网络服务的问题
	* 记录过往事件
* Linux  常见的登录文件
	* /var/log/boot.log：开机的时候系统内核会去侦测与启动硬件，接下来开始各种内核支持的功能启动等。这些流程都会记录在/var/log/boot.log 里面！ 不过这个文件只会存在这次开机启动的信息，前次开机的信息并不会被保留下来！
	*  /var/log/cron：crontab 定时任务没有实际被进行？ 进行过程有没有发生错误？你的 /etc/crontab 是否撰写正确？在这个登录文件内查询看看。
	*  /var/log/dmesg：记录系统在开机的时候内核侦测过程所产生的各项信息。由于 CentOS 默认将开机时内核的硬件侦测过程取消显示， 因此额外将数据记录一份在这个文件中；
	*   /var/log/lastlog：可以记录系统上面所有的账号最近一次登入系统时的相关信息。第十三章讲到的 lastlog 指令就是利用这个文件的记录信息来显示的。
	*  /var/log/maillog 或 /var/log/mail/\*：记录邮件的往来信息，其实主要是记录 postfix (SMTP 协议提供者) 与 dovecot (POP3 协议提供者) 所产生的讯息啦。 SMTP 是发信所使用的通讯协议， POP3 则是收信使用的通讯协议。 postfix 与 dovecot 则分别是两套达成通讯协议的软件。
	*  /var/log/messages：这个文件相当的重要，几乎系统发生的错误讯息 (或者是重要的信息) 都会记录在这个文件中； 如果系统发生莫名的错误时，这个文件是一定要查阅的登录文件之一。
	*  /var/log/secure：基本上，只要牵涉到『需要输入账号密码』的软件，那么当登入时 (不管登入正确或错误) 都会被记录在此文件中。 包括系统的 login 程序、图形接口登入所使用的 gdm 程序、 su, sudo 等程序、还有网络联机的ssh, telnet 等程序， 登入信息都会被记载在这里；
	*  /var/log/wtmp, /var/log/faillog：这两个文件可以记录正确登入系统者的帐户信息 (wtmp) 与错误登入时所使用的帐户信息 (faillog) ！ 我们在第十章谈到的 last 就是读取 wtmp 来显示的， 这对于追踪一般账号者的使用行为很有帮助！
	*  /var/log/httpd/\*, /var/log/samba/\*：不同的网络服务会使用它们自己的登录文件来记载它们自己产生的各项讯息！上述的目录内则是个别服务所制订的登录文件。


------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
