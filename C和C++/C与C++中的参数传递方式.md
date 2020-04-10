---
title: C与C++中的参数传递方式
tags: C和C++
---

------

***<font color=blue>版权声明</font>：<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------


# 1 传值

C/C++默认的采用传值，即在函数被调用的时候，给形参申请一个空间，再将实参的值传递给形参，**对形参的任何改变不会影响实参数的值**：

```c++
#include<iostream>
using namespace std;
int add(int x)
{
	cout << "变化前形参地址:" << &x << "   " << "形参值:" << x << endl;
	x++;
	cout << "变化后形参地址:" << &x << "   " << "形参值:" << x << endl;
	return 0;
}
int main()
{
	int a = 1;
	cout << "调用前实数地址:" << &a << "   " << "实参值:" << a << endl;
	add(a);
	cout << "调用后实参地址:" << &a << "   " << "实参值:" << a << endl;
	return 0;
}
```

输出结果如下：

![图1](https://gitee.com/liao20081228/blog_pictures/raw/master/C与C++中的参数传递方式/1586500022533.jpg#pic_center)

结论：
&emsp;&emsp;可以看到形参实参的地址不一样，说明新申请了内存空间。
&emsp;&emsp;可以看到对形参的操作不影响实参。
&emsp;&emsp;注：x是在函数内部定义的变量因而其作用范围是局部的（仅在该函数范围内能访问），生命周期是动态的（调用该函数时候创建，函数调用完成后就消除。）

# 2 传址
## 2.1 传指针
所谓传址又叫做传指针，即在函数被调用的时候，给形参开辟一个空间用来存放传递过来的地址，将实参的值传递给形参，**对形参的任何改变也不会影响实参所指的内容，但是对形参所指的内容的改变将会影响到实参所指的内容**（因为这两个指针都指向同一个内存空间）

```c++
#include<iostream>
using namespace std;
int fun(int *x,int *y)
{
	cout << "变化前,指针x所在的地址:" << &x <<"  指针y所在地址:"<< &y <<endl;
	cout << "变化前,指针x的值      :" << x << "  指针y的值    :" << y<<endl;
	cout << "变化前,指针x所指的值  :" << *x<< "             " <<"  指针y所指的值:"<< *y<<endl;
	x++;
	(*y)++;
	cout << "变化后,指针x所在的地址:" << &x <<"  指针y所在地址:"<<&y<<endl;
	cout << "变化后,指针x的值      :" << x  <<"  指针y的值    :" << y<<endl;
	cout << "变化后,指针x所指的值  :" << *x<< "             " <<"  指针y所指的值:"<< *y<<endl;
	return 0;

}
int main()
{
	int n1 = 1,n2=1;
	int *a = &n1;
	int *b = &n2;
	cout << "调用前,指针a所在的地址:" << &a <<"  指针b所在地址:"<<&b<<endl;
	cout << "调用前,指针a的值      :" << a  <<"  指针b的值    :" << b<<endl;
	cout << "调用前,指针a所指的值  :" << *a<< "             " <<"  指针b所指的值:"<< *b<<endl;
	fun(a,b);
	cout << "调用后,指针a所在的地址:" << &a <<"  指针b所å¨地址:"<<&b<<endl;
	cout << "调用后,指针a的值      :" << a  <<"  指针b的值    :" << b<<endl;
	cout << "调用后,指针a所指的值  :" << *a<< "             " <<"  指针b所指的值:"<< *b<<endl;
	return 0;
}   


  
```

运行下结果如下：

![图2](https://gitee.com/liao20081228/blog_pictures/raw/master/C与C++中的参数传递方式/1586500022542.jpg#pic_center)

结论：
&emsp;&emsp;指针a和指针x所在地址不同说明新申请内存空间用以存放传来的地址。
&emsp;&emsp;对指针x的值的改变不会改变指针a的值，也不会改变指针a所指的值，所以输出结果为1。
&emsp;&emsp;对指针y所指的值的改变会改变指针b所指的值。所以输出结果为2。

## 2.2 传数组
传数组是传址的另一方式，数组作为形参传递，实质传递的数组的首地址，一维数组形参 :`数据类型 数组名[]`，二维数组形参：`数据类型 数组名[][上界]` 。

>**注意**：数组在传递时会退化为指针，因而要传入数组长度。因此在C++中推荐使用容器来代替数组。


# 3 传引用
## 3.1 传引用
引用是什么？C++中的引用可以理解为类似为typedef的作用（C中是没有此用法，typedef是对类型取别名），引用相当于是变量的别名，**对形参的任何操作，就是对实参的操作**，函数调用时不会再内存中新开辟空间。

```c++
#include<iostream>
using namespace std;
int add(int &x)
{
	cout << "变化前,形式参数所在地址:" << &x << "   " << "形式参数值:" << x << endl;
	x++;
	cout << "变化后,形式参数所在地址:" << &x << "   " << "形式参数值:" << x << endl;
	return 0;
}
int main()
{
	int a = 1;
	cout << "调用前,形式参数所在地址:" << &a << "   " << "形式参数值:" << a << endl;
	add(a);
	cout << "调用后,形式参数所在地址:" << &a << "   " << "形式参数值:" << a << endl;
	return 0;
}
```
运行结果如下：

![图3](https://gitee.com/liao20081228/blog_pictures/raw/master/C与C++中的参数传递方式/1586500022415.jpg#pic_center)

结论：
&emsp;&emsp;可以看到实参和形参的地址是相同的，也就是说并没有开辟一个新的空间。对引用的操作就是对实参的操作。


## 3.2 传指针的引用
&emsp;&emsp;其形式为`function（数据类型* &指针名）`，这里的指针名就是引用，就是传来的指针的别名。对它的操作就是对实参指针的操作（类似于C中的二级指针。），对它所指的值的操作，就是对实参指针所指的值的操作。

```c++
#include<iostream>
using namespace std;
#define ok 0
int add(int* &x)
{
	cout << "变化前指针x所在的地址: " << &x << endl;
	cout << "变化前指针x的值      : " << x << endl;
	cout << "变化前指针x所指的值  : " << *x << endl;
	x++;
    cout << "变化后指针x所在的地址: " << &x << endl;
	cout << "变化后指针x的值      : " << x << endl;
	cout << "变化后指针x所指的值  : " << *x << endl;
	return ok;
}
int main()
{
	int c = 1;
	int *a = &c;
	cout << "调用前指针a所在的地址:" << &a << endl;
	cout << "调用前指针a的值      :" << a << endl;
	cout << "调用前指针a所指的值  :" << *a << endl;
	add(a);
	cout << "调用后指针a所在的地址:" << &a << endl;
	cout << "调用后指针a的值      :" << a << endl;
	cout << "调用后指针a所指的值  :" << *a << endl;
	return ok;
}
```

运行结果如下：

![图4](https://gitee.com/liao20081228/blog_pictures/raw/master/C与C++中的参数传递方式/1586500022534.jpg#pic_center)

结论：
&emsp;&emsp;可以看到调用时没有在内存中开辟空间存储传递来的地址。对形参的改变就是对实参的改变。

# 4 总结

一般时候对传入的实参如果要做改变，或传入的数据非常大时候，C\+\+中优先考虑传引用（C/C\+\+都可以传址，然后对指针所指内容做改变），这样调用函数时可以节约拷贝的时间和空间；如果对传入的实参不做改变可以传值、传引用、传址。



------

***<font color=blue>版权声明</font>：<font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。转载请注明出处！！！</font>***

------
