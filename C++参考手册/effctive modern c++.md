---
title: effctive modern c++
tags: C++
---

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>《Effective Modern C++》。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

<style>table{word-break:initial;}</style>

# 1 模板类型推导
```c++
template<typeanme T>
void f(paramType  param)//paraType 可能为：T，T&，T*，const T&，const T*，T&&

int x27；
const int cx = x;
const int& rx = x;
const int* p1=&x;
const int* const p2=&x;
f(x);
f(cx);
f(rx);
f(&x);
f(p1);
f(p2);
f(27);
```
类型T的推导结果并不一定与param的类型相同，它不仅取决于param本身类型，还取决于paramType的形式

* paramType 是一个指针或引用，但不是万能引用
	* 如果实参是引用，则忽略引用或指针，然后对实参类型和paramType执行模式匹配，以决定T的类型
* paramType 是万能引用
	* 实参是左值，T和paraType都被推导为一个引用
	* 实参是右值，按照规则1进行推导
* paramType 不是指针也不是引用
	* 实参是引用，则忽略引用
	* 实参具有const或volatile 属性，则忽略
* 实参是数组或函数
	* 按值传递
	* 按引用传递

如果实参是数组或函数
* 按值传递：形参被推导为函数或数组指针
* 按引用传递：形参被推导为函数或数组引用

# 2 auto类型推导
一般情况下，auto类型推导与模板类型推导一样，auto就相当于模板中的T。但是auto类型推导会假定初始化列表初始化的变量的类型std::initializer_list，而模板推导不会。

在函数返回值（C++14）或者 lambda的形参中使用auto，意思是使用模板推导而不是auto推导.。因此不能返回或者传入初始化列表。

>auto声明变量必须要有初始化表达式

#  3  decltype类型推导
C++11中decltype主要作用：
* 根据表达式推测其类型。
* 与auto联用以根据函数形参类型来推测函数返回值类型。

在C++14中，可以只用auto。但是要注意，auto会忽略引用，因此返回引用时要使用decltype(auto)。

一般情况，decltype会得出变量或者表达式的类型而不做任何修改，与期望的相同。

对于类型为T的左值表达式，除非该表达式只有名字，否则，decltype总是推导出类型T&。

C++14中还支持另一种decltype(auto)，它和auto一样会从初始化表达式来推测类型，但是使用decltype规则

(x)在c++是一个表达式，但和x不一样

# 4  查看类型推导结果
编写代码阶段： 使用IDE

编译期间：编辑器诊断信息，主要是使用这些类型来产生编译错误。

运行期间：
* 使用typeid().name()来查看类型有时是不可靠的。因为typeid处理类型就像是向函数模板按值传递形参，会按照模板类型推导的规则进行。
* 使用boost/type_index.hpp中的type_id_with_cvr<>().pretty_name()是可靠的。


# 5 优先使用auto，而非显式声明

# 7 在创建对象时要注意区分()和{}
指定初始化值的方式有四种：=、（）、{}、={}；C++11加入了初始化列表{}。
 * ()和=不能用来为容器指定初始化值，{}可以。
 * =和{}都可以用来为非静态成员指定默认初始值，但是()不可以。
 * ()和{}可以为不可复制对象指定初始值，但=不可以。
 * {}初始化禁止隐式缩窄类型转换。()和=不会禁止缩窄转换。
 
 C++规定任何能解析为声明的都要解析为声明，因此，使用()调用无参数默认构造函数来定义一个对象时会造成歧义，使得编译器误认为该定义是一个函数声明。

在构造函数被调用时，只要构造函数具有std::initializer_list的形参，编译器只要有任何可能把一个采用了{}的构造调用语句解读为代用std::initializer_list的类型形参的构造函数，编译器就回选用这种解释。

甚至当初始化列表中的实参转为std:;initializer_list中的类型出错时，仍旧会选择这种解释。

只有找不到任何办法进行这种转换时，编译器才会去检查其它重载。
# 8 优先使用nullptr代替0和NULL
C++基本观点0是int，而不是指针。

使用0和NULL在涉及到指针和整形之间的重载时会产生二义性。

C++中不要在指针和整形之间做重载。如果做了重载，则在调用时应使用nullptr而不是0或者NULL。

nullptr的类型是`std::nullptr_t`，`std::nullptr_t`既不整型，也不是指针。在头文件《cstddef》中，定义如下：`typedef decltype(nullptr) nullptr_t;`，`std::nullptr_t`可以隐式转为任何类型的裸指针。

模板类型推导和auto类型推导会将0和NULL推测为整型，而不是指针。

------

&emsp;&emsp;<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>《Effective Modern C++》。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
