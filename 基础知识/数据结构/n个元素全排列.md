---
title: n个元素全排列
tags: 数据结构
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>**严蔚敏**的《数据结构》。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
# 1 算法描述
1. 先选择一个元素放在一个固定的位置，然后求剩余n-1个元素的全排列。
2. 接着选择第2个元素放在一个固定的位置，然后求剩余n-1个元素的全排列。n-1个元素的求法仍然按照（1）求。直到只剩下一个元素为止。

# 2 算法实现
```cpp
#include<iostream>
using namespace std;

void perm(char str[], int k, int n,int &m)//排列函数，
{
	int i, j;//m用于记录全排列的总数
	char temp;
	if (k == 0)
	{
		cout << str << '\n';
		m++;
	}
	else
	{
		for (j = 0; j <=k; j++)
		{
			temp = str[k];
			str[k] = str[j];
			str[j] = temp;
			perm(str, k - 1, n,m);
			temp = str[j];
			str[j] = str[k];
			str[k] = temp;		
		}
	}
}
int main()
{
	int m=0;
	char str[] = "abcd";
	perm(str, sizeof(str) - 2, sizeof(str) - 2, m);
	cout << "一共有" << m << "种全排列";
	return 0;
}

```
或者
```cpp
#include<iostream>
using namespace std;

void perm(char str[], int k, int n,int &m)//排列函数，
{
	int i, j;//m用于记录全排列的总数
	char temp;
	if (k == 1)
	{
		cout << str << '\n';
		m++;
	}
	else
	{
		for (j = 0; j <k; j++)
		{
			temp = str[k-1];
			str[k-1] = str[j];
			str[j] = temp;
			perm(str, k - 1, n,m);
			temp = str[j];
			str[j] = str[k-1];
			str[k-1] = temp;		
		}
	}
}
int  main()
{
	int m=0;
	char str[] = "abcd";
	perm(str, sizeof(str) - 1, sizeof(str) - 1, m);
	cout << "一共有" << m << "种全排列";
	return 0;
}
```
> 注意:第一种写法调用的时候是按数组下标的实际大小调用如求“abced”的全排列应该输入的4，第二种写法则是按照日常逻辑来调用输入的大小是5。


------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>**严蔚敏**的《数据结构》。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
