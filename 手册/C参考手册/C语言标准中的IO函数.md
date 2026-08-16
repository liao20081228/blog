---
title: C语言标准中的IO函数
tags: C和C++
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

<style>table{word-break:initial;}</style>


# 1 标准IO



# 1 文件IO
文件：程序和数据的集合就叫文件。

文件缓冲区：由于内存和I/O设备的的速度差异很大。因而在内存中设置输入、输出缓冲区。读取时，先从输入设备中读取一批数据到输入缓冲区，然后在依次送给变量。输出时先存放在输出缓冲区，待缓冲区满或程序结束或主动刷新时才输出到输出设备。

文件信息区：每个打开一个文件，都会在内存中开辟一个文件信息区用于存放文件的相关相关属性。这些信息存放在一个名为FILE的结构体中。

标准I/O流与文件流：标准输入流、标准输出流、标出出错输出流（键盘，屏幕，屏幕）如putchar，getchar，putc，getc，puts，gets，printf，scanf。文件流（文本文件与二进制文件）如fputc，fgetc，fputs，fgets，fprintf，fscanf。实质就是流指针的不同，前者流指针为stdin或stdout，后者则为FILE型指针。

文件的存储形式：
（1）	ASCII文件，又叫文本文件，内存数据（二进制）——>ASCII码——>硬盘
（2）	二进制文件，又叫映像文件，内存数据（二进制）——>硬盘


```c
#include <stdio.h>

       FILE *fopen(const char *path, const char *mode);
	   FILE *freopen(const char *path, const char *mode, FILE *stream);
```
**参数**：
- path
- mode
- stream 











------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
