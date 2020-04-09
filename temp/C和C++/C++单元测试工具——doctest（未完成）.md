---
title: C++单元测试工具——doctest（未完成）
tags: C++
---

------

<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《doctest项目的README》](<https://github.com/onqtam/doctest>)。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------

<style>table{word-break:initial;}</style>


# 1 项目地址
<https://github.com/onqtam/doctest>
# 2 doctest介绍
<b>
<table>
    <tr>
        <td>
            master branch
        </td>
        <td>
            Linux/OSX <a href="https://travis-ci.org/onqtam/doctest"><img src="https://travis-ci.org/onqtam/doctest.svg?branch=master"></a>
        </td>
        <td>
            Windows <a href="https://ci.appveyor.com/project/onqtam/doctest/branch/master"><img src="https://ci.appveyor.com/api/projects/status/j89qxtahyw1dp4gd/branch/master?svg=true"></a>
        </td>
        <td>
            <a href="https://coveralls.io/github/onqtam/doctest?branch=master"><img src="https://coveralls.io/repos/github/onqtam/doctest/badge.svg?branch=master"></a>
        </td>
        <td>
            <a href="https://scan.coverity.com/projects/onqtam-doctest"><img src="https://scan.coverity.com/projects/7865/badge.svg"></a>
        </td>
    </tr>
    <tr>
        <td>
            dev branch
        </td>
        <td>
            Linux/OSX <a href="https://travis-ci.org/onqtam/doctest"><img src="https://travis-ci.org/onqtam/doctest.svg?branch=dev"></a>
        </td>
        <td>
            Windows <a href="https://ci.appveyor.com/project/onqtam/doctest/branch/dev"><img src="https://ci.appveyor.com/api/projects/status/j89qxtahyw1dp4gd/branch/dev?svg=true"></a>
        </td>
        <td>
            <a href="https://coveralls.io/github/onqtam/doctest?branch=dev"><img src="https://coveralls.io/repos/github/onqtam/doctest/badge.svg?branch=dev"></a>
        </td>
        <td>
        </td>
    </tr>
</table>
</b>

doctest用于单元测试和TDD的功能最丰富的C98 / C11单个头文件测试框架。

doctest是一个新的C++测试框架，但与其他功能丰富的替代方法相比，它在编译时间（[**按数量级**][1]）和运行时间上都是最快的。通过提供一个快速，透明和灵活的测试器和一个干净的接口，它使[**D**](https://dlang.org/spec/unittest.html) / [**Rust**](https://doc.rust-lang.org/book/testing.html) / [**Nim**](https://nim-lang.org/docs/unittest.html) 等编译语言能够直接在生产代码中进行测试。

该框架是和将保持自由，但需要你的支持，以维持其发展。有很多[**新功能特性**](https://github.com/onqtam/doctest/blob/master/doc/markdown/roadmap.md)和维护要做。如果您使用doctest为公司工作，或者有办法这样做，请考虑财务支持。每月通过Patreon捐款和通过PayPal的一次性捐款。[![Patreon](https://cloud.githubusercontent.com/assets/8225057/5990484/70413560-a9ab-11e4-8942-1a63607c0b00.png)](http://www.patreon.com/onqtam)[![PayPal](https://www.paypalobjects.com/en_US/i/btn/btn_donate_LG.gif)](https://www.paypal.me/onqtam/10)

一个编译为可执行文件的自注册测试的完整示例如下所示：

![此处输入图片的描述][2]

有很多C++测试框架，如[Catch](https://github.com/catchorg/Catch2)，[Boost.Test](http://www.boost.org/doc/libs/1_64_0/libs/test/doc/html/index.html)，[UnitTest](https://github.com/unittest-cpp/unittest-cpp)，[cpputest](https://github.com/cpputest/cpputest)，[googletest](https://github.com/google/googletest)以及[其它一些](https://en.wikipedia.org/wiki/List_of_unit_testing_frameworks#C.2B.2B)。

doctest和其他测试框架之间的主要区别在于它很轻而且没有侵入性:
                         
* 就[包括头文件](https://github.com/onqtam/doctest/blob/master/doc/markdown/benchmarks.md#cost-of-including-the-header)和编写[数千个断言](https://github.com/onqtam/doctest/blob/master/doc/markdown/benchmarks.md#cost-of-an-assertion-macro)方面，编译时间都非常短。
* 即使在MSVC / GCC /Clang的[最严格](https://github.com/onqtam/doctest/blob/master/scripts/cmake/common.cmake#L84)的警告级别也不会产生任何警告。
* 提供了一种使用[DOCTEST_CONFIG_DISABLE](https://github.com/onqtam/doctest/blob/master/doc/markdown/configuration.md#doctest_config_disable)标识符从二进制文件中删除所有与测试相关的代码的方法。
* 不污染全局命名空间（一切都在命名空间doctest），并不拖动任何标题。
* 完全[兼容](https://github.com/onqtam/doctest/blob/master/doc/markdown/features.md#extremely-portable)C++98——每次提交之前，都在CI上通过超过300个案例（静态分析，杀毒）![此处输入图片的描述][3]

这允许框架有比其他测试工具更多的使用方式—— 测试可以直接写在生产代码中！

Mantra说：*测试可以被认为是一种文档形式，应该能够靠近他们测试的产品代码。*

* 这使得写测试的障碍非常低。
  * 你不必制作一个单独的源文件。
  * 不用包括一堆东西在里面。
  * 不用将其添加到构建系统。
  * 不用将其添加到源代码管理 。
  * 您可以在其源代码文件的底部,甚至头文件中编写一个类或一个功能的测试！
* 生产代码中的测试可以被认为是文档或最新的评论 - 显示如何使用API。
* 不通过公共API和头文件暴露测试内部细节，不再是一个令人费解的问题。
* [测试驱动开发](https://en.wikipedia.org/wiki/Test-driven_development)从未如此简单。

如果您不想/需要混合生产代码和测试，可以像使用其他框架一样使用该框架——[查看该功能](https://github.com/onqtam/doctest/blob/master/doc/markdown/features.md)。

doctest是在Catch之后建模，代码的某些部分已经被直接使用 ——[请检查它们之间的差异](https://github.com/onqtam/doctest/blob/master/doc/markdown/faq.md#how-is-doctest-different-from-catch)。
[这个表](https://github.com/martinmoene/catch-lest-other-comparison)比较doctest / [catch](https://github.com/catchorg/Catch2) / [lest](https://github.com/martinmoene/lest)，非常相似。
[![Standard](https://img.shields.io/badge/c%2B%2B-98/11/14/17-blue.svg)](https://en.wikipedia.org/wiki/C%2B%2B#Standardization) [![License](https://img.shields.io/badge/license-MIT-blue.svg)](https://opensource.org/licenses/MIT) [![Version](https://badge.fury.io/gh/onqtam%2Fdoctest.svg)](https://github.com/onqtam/doctest/releases) [![download](https://img.shields.io/badge/download%20%20-latest-blue.svg)](https://raw.githubusercontent.com/onqtam/doctest/master/doctest/doctest.h) [![Join the chat at https://gitter.im/onqtam/doctest](https://badges.gitter.im/onqtam/doctest.svg)](https://gitter.im/onqtam/doctest?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge) [![Try it online](https://img.shields.io/badge/try%20it-online-orange.svg)](https://wandbox.org/permlink/Jcmj4Eqb7IeFgIZr) [![Language](https://img.shields.io/badge/language-C++-blue.svg)](https://isocpp.org/) [![documentation](https://img.shields.io/badge/documentation%20%20-online-blue.svg)](https://github.com/onqtam/doctest/blob/master/doc/markdown/readme.md#reference)

查看YouTube上的[CppCon 2017演讲](https://cppcon2017.sched.com/event/BgsI/mix-tests-and-production-code-with-doctest-implementing-and-using-the-fastest-modern-c-testing-framework) ，以更好地了解框架的工作原理，并阅读[ACCU Overload 2017二月版文章](https://accu.org/var/uploads/journals/Overload137.pdf))中的使用方法！








# 4 简明教程
要开始使用doctest，您只需要下载其最新版本，并将其包含在源文件中（或者将此版本库作为git子模块添加）——doctest仅仅为一个头文件。
本教程假定您可以直接使用头文件：`#include"doctest.h"`,它与测试源文件在同一个文件夹中，或者您已经在构建系统中正确设置了包含路径。
[驱动测试开发TDD](https://en.wikipedia.org/wiki/Test-driven_development)不是本教程的讨论范围。
## 4.1 简单例子
假设你要测试一个函数factorial：
```cpp
int factorial(int number) 
{ 
	return number <= 1 ? number : factorial(number - 1) * number; 
}
```

 使用doctest提供的默认main函数的测试程序如下所示：
```cpp
#define DOCTEST_CONFIG_IMPLEMENT_WITH_MAIN
#include "doctest.h"

int factorial(int number)
{ 
   return number <= 1 ? number : factorial(number - 1) * number; 
}

TEST_CASE("testing the factorial function") {
    CHECK(factorial(1) == 1);
    CHECK(factorial(2) == 2);
    CHECK(factorial(3) == 6);
    CHECK(factorial(10) == 3628800);
}
```
这将编译成一个完整的可以响应命令行参数的可执行文件。如果没有任何命令行参数，它将执行所有的测试用例（在这个测试中 只有一个测试用例），报告每一个失败，生成关于有多少测试通过和失败的摘要。成功返回0，如果有任何失败返回1（如果你指向知道测试是否通过，这会很有用）。
如果你运行这个测试它会完全通过。似乎一切都很好，对吗？但这里还有一个bug。我们没有测试factorial（0）== 1，让我们添加这个测试用例：
```
TEST_CASE("testing the factorial function") 
{
    CHECK(factorial(0) == 1);
    CHECK(factorial(1) == 1);
    CHECK(factorial(2) == 2);
    CHECK(factorial(3) == 6);
    CHECK(factorial(10) == 3628800);
}
```
再次编译运行，结果显示我们没有完全通过测试：
```
test.cpp(7) FAILED!
  CHECK( factorial(0) == 1 )
with expansion:
  CHECK( 0 == 1 )
```
注意`CHECK( 0 == 1 )`：我们获得了factorial(0)的真正返回值0，在表达式中又使用了`==`。这是我们意识到了问题所在，我们修改这个函数：
```
int factorial(int number) { return number > 1 ? factorial(number - 1) * number : 1; }
```
现在测试通过了。
当然还有更多的问题需要考虑。例如，当返回值超过int的范围时，我们会遇到问题。使用阶乘可以很快出现这样的结果，您可能需要为这些用例添加测试，并决定如何处理这。在这里就不做这些测试了。

## 4.2 我们做了什么？
虽然这是一个简单的测试，但是它足以展示如何使用doctest。
我们需要做的就是**定义一个宏，然后引入一个头文件**——这样我们就获得所需的一切，包括响应命令行参数的main函数。只能在一个源文件中定义宏，但可以在多个源文件中引入doctest.h。建议在一个专门的实现文件中`#define DOCTEST_CONFIG_IMPLEMENT_WITH_MAIN `和`#include "doctest.h"`。当然也可以自己提供main函数——参考[提供自定义的主函数](https://github.com/onqtam/doctest/blob/master/doc/markdown/main.md)。
我们**用TEST_CASE宏引入测试用例**。它需要一个参数，该参数是 一个自由格式的测试名称（更多信息参见[Test cases and subcases](https://github.com/onqtam/doctest/blob/master/doc/markdown/testcases.md)）。测试名称不必是唯一的,您可以通过指定通配的测试名称或标签来运行指定测试集。有关运行测试的更多信息，请参阅[命令行文档](https://github.com/onqtam/doctest/blob/master/doc/markdown/commandline.md)。
测试名称只是一个字符串,可以为空。我们不必声明一个函数或方法，或者在任何地方明确注册测试用例。在后台，一个带有名称的函数被定义，并且使用静态注册表自动注册。通过将函数名称抽象出来，我们可以在不受标识符名称限制的情况下命名我们的测试。
我们使**用CHECK（）编写我们的各个用例的测试断言**。我们用C++的语法自然的表述条件，而不是针对每种类型的条件表达式（等于，小于，大于等）单独用一种，在后台，用一个简单的表达式模板来捕获表达式的左侧值和右侧值，所以我们可以在我们的测试报告中显示这些值。还有[其他的断言宏](https://github.com/onqtam/doctest/blob/master/doc/markdown/assertions.md)在本教程中没有涉及,但由于这种技术，他们的数量大幅减少。
## 4.3 测试案例和子案例
大多数测试框架都有一个基于类的夹具机制。测试用例映射到类的方法，并且可在`setup（）`和`teardown（）`函数（或者在C++这样的语言的构造/析构函数）中执行相同的setup和teardown。
尽管doctest完全支持这种工作方式，但这种方法存在一些问题。特别是你的代码必须被拆分，拆分的粒度可能会导致问题。你只能在一组方法中有一个setup / teardown对，但是有时你需要在每种方法中稍微不同的setup或者你可能需要几个级别的setup（这个概念我们稍后会在本教程中进行阐述）。像[这样的问题]就导致了James Newkirk领导建立了NUnit的团队，从头开始重新构建了[xUnit](http://jamesnewkirk.typepad.com/posts/2007/09/announcing-xuni.html)。
doctest采取了一种与NUnit和xUnit不同的方法，这更适合于C++和C一类的语言。
最好通过一个例子来解释：
```
TEST_CASE("vectors can be sized and resized") {
    std::vector<int> v(5);

    REQUIRE(v.size() == 5);
    REQUIRE(v.capacity() >= 5);

    SUBCASE("adding to the vector increases it's size") {
        v.push_back(1);

        CHECK(v.size() == 6);
        CHECK(v.capacity() >= 6);
    }
    SUBCASE("reserving increases just the capacity") {
        v.reserve(6);

        CHECK(v.size() == 5);
        CHECK(v.capacity() >= 6);
    }
}
```
**对于每个SUBCASE（），TEST_CASE（）都从头开始执行**,所以当执行每个子案例时，我们知道大小是5，容量至少是5。我们用顶级的REQUIRE（）宏来执行这些要求，因为我们对此有信心。如果CHECK（）失败，这个测试用例被标记为失败，但会执行继续，但是如果REQUIRE（）失败，则停止并退出测试。
这是因为SUBCASE（）宏包含一个if语句，该语句将回调到doctest中以查看是否应该执行该子用例。TEST_CASE（）每次运行一个子用例，其他子用例将被跳过。下一次执行下一个子用例，直到没有遇到新的子用例。
到现在为止，一直都还不错——这已经是setup / teardown方法的一个改进，因为现在我们看得到setup代码，并使用了栈。当我们在下面的例子中开始嵌套时，SUBCASE的实力就显示出来了：
```
#define DOCTEST_CONFIG_IMPLEMENT_WITH_MAIN
#include "doctest.h"

#include <iostream>
using namespace std;

TEST_CASE("lots of nested subcases") 
{
    cout << endl << "root" << endl;
    SUBCASE("") 
    {
        cout << "1" << endl;
        SUBCASE("") { cout << "1.1" << endl; }
    }
    SUBCASE("") 
    {   
        cout << "2" << endl;
        SUBCASE("") { cout << "2.1" << endl; }
        SUBCASE("") 
        {
            cout << "2.2" << endl;
            SUBCASE("")
            {
                cout << "2.2.1" << endl;
                SUBCASE("") { cout << "2.2.1.1" << endl; }
                SUBCASE("") { cout << "2.2.1.2" << endl; }
            }
        }
        SUBCASE("") { cout << "2.3" << endl; }
        SUBCASE("") { cout << "2.4" << endl; }
    }
}
```
输出结果：
```
root
1
1.1

root
2
2.1

root
2
2.2
2.2.1
2.2.1.1

root
2
2.2
2.2.1
2.2.1.2

root
2
2.3

root
2
2.4
```
subcase可以嵌套到任意深度（仅受限于堆栈大小）。叶子用例（不包含subcase 的用例）将按一个独立于其它叶子用例的路径恰好执行一次，所以一个叶子用例不会干扰另一个叶子用例。而父用例中的致命故障将阻止子用例运行，就是这样设计的。
## 4.4 扩展
为了简化教程，我们把所有的代码放在一个文件中。这是对启动有好处，使得进入doctest更加容易和快速。当你开始编写更多真实的测试时，这并不是最好的方法。
要求是下面的代码块([或等价的代码块](https://github.com/onqtam/doctest/blob/master/doc/markdown/main.md)):
```
#define DOCTEST_CONFIG_IMPLEMENT_WITH_MAIN
#include "doctest.h"
```
恰好出现在一个翻译单元（源文件）中。根据需要为测试使用尽可能多的额外的源文件——分区对于你的工作来说是最有意义的。每个额外的文件只需要`#include"doctest"`,但不要重复`#define`。
实际上把上述的`#define`语句放在自己的源文件会更好。
## 4.5 下一步
这是一个为了让你快速开始和运行doctest的简单介绍，指出了doctest和其他你可能已经熟悉的框架之间的一些主要区别。这会让你走得很远，你现在可以沉浸其中并写一些测试。
当然你也可以学习后续的内容。


# 5 断言
## 5.1 普通断言
普通断言宏和C中的宏assert(expression)类似，带有一个参数，该参数是一个一元或者二元的条件表达式，为真或为假，产生对应动作 。

* doctest中肯定断言有三个级别：
     * `REQUIRE(expression)`—— 如果表达式为假，将立即退出测试用例，并将测试用例标记为失败。
     * `CHECK(expression)`——如果表达式为假，将测试用例标记为失败，但将继续测试用例。对于用有多用例的测试是很适用的。  
     * `WARN(expression)`—— 如果表达式为假，将仅打印一条消息，但不会将测试用例标记为失败。
* doctest中否定断言有三个级别：
      * `REQUIRE_FALSE(expression)`—— 如果表达式为真，将立即退出测试用例，并将测试用例标记为失败。
      * `CHECK_FALSE(expression)`—— 如果表达式为真，将测试用例标记为失败，但将继续测试用例。  
      * `WARN_FALSE(expression)`—— 如果表达式为真，将仅打印一条消息，但不会将测试用例标记为失败。


* 带消息的断言
   * 这类似于C++中的static_assert(expression, string)  
   * 带消息的断言就是在之后加上_MESSAGE，如REQUIRE_MESSAGE（expression, "string"）
   * 它基本上是一个代码块{}，其范围为[INFO（）](https://github.com/onqtam/doctest/blob/master/doc/markdown/logging.md#info)日志记录宏以及CHECK宏—— 这样消息将仅与该断言相关。更多信息参见[INFO宏与日志](https://github.com/onqtam/doctest/blob/master/doc/markdown/logging.md)。
  * 只有这里的6个断言有*_MESSAGE以及异常断言有这种形式，二元、一元、快速断言都没有这种形式
```cpp
INFO("this is relevant to all asserts, and here is some var: " << local);

CHECK_MESSAGE(a < b, "relevant only to this assert " << other_local << "more text!");

CHECK(b < c); // here only the first INFO() will be relevant
```
    
所有的断言只评估一次表达式，如果失败，则表达式的值会被[字符串化](https://github.com/onqtam/doctest/blob/master/doc/markdown/stringification.md)。
只要抛出的异常，就会被捕获，报告，并算作失败，除非当前的断言级别是`WARN`。
**请注意**，REQUIRE级别的断言使用异常结束当前的测试用例。在用户定义的类的析构函数中使用此级别的断言可能是危险的——如果一个析构函数在异常引起堆栈展开期间被调用，并且REQUIRE断言失败，则程序将终止。此外，从C++11开始，除非另有说明，否则所有析构函数都将默认为noexcept（true），因此这样的断言将导致`std :: terminate（）`被调用。



##9.2 二元和一元断言
 这些断言不使用模板来分解条件表达式以捕获左侧和右侧值。所以**编译速度快快25％-45％。**
 
 `<LEVEL>`是3种可能之一：REQUIRE / CHECK / WARN。
 
```cpp
<LEVEL>_EQ(left, right)   等价  <LEVEL>(left == right)
<LEVEL>_NE(left, right)   等价  <LEVEL>(left != right)
<LEVEL>_GT(left, right)   等价  <LEVEL>(left > right)
<LEVEL>_LT(left, right)   等价  <LEVEL>(left < right)
<LEVEL>_GE(left, right)   等价  <LEVEL>(left >= right)
<LEVEL>_LE(left, right)   等价  <LEVEL>(left <= right)
<LEVEL>_UNARY(expr)       等价  <LEVEL>(expr)
<LEVEL>_UNARY_FALSE(expr) 等价  <LEVEL>_FALSE(expr)
```
  
##9.3 快速断言
 快速断言是二元和一元断言的更快版本 ，快60-80％。区别在于 如果表达式抛出异常则整个测试结束，不会评估try / catch块中的表达式。
  如果定义了`DOCTEST_CONFIG_SUPER_FAST_ASSERTS`，可以更快50-80％！
  
   `<LEVEL>`是3种可能之一：REQUIRE / CHECK / WARN。
```cpp
FAST_<LEVEL>_EQ(left, right) 等价 <LEVEL>(left == right)
FAST_<LEVEL>_NE(left, right) 等价 <LEVEL>(left != right)
FAST_<LEVEL>_GT(left, right) 等价 <LEVEL>(left > right)
FAST_<LEVEL>_LT(left, right) 等价 <LEVEL>(left < right)
FAST_<LEVEL>_GE(left, right) 等价 <LEVEL>(left >= right)
FAST_<LEVEL>_LE(left, right) 等价 <LEVEL>(left <= right)
FAST_<LEVEL>_UNARY(expr) 等价 <LEVEL>(expr)
FAST_<LEVEL>_UNARY_FALSE(expr) 等价 <LEVEL>_FALSE(expr)
```  
##9.4 异常断言
  `<LEVEL>`是3种可能之一：REQUIRE / CHECK / WARN。

* `<LEVEL>_THROWS(expression)`
  断言在表达式的测试期间会抛出异常（任何类型）。
  
* `<LEVEL>_THROWS_AS(expression, exception_type)`
断言在表达式的测试期间抛会出指定类型的异常。 
**注意：异常类型原本添加了const或&，用户只需要指定类型即可，因为C++中标准做法是按值抛出，用引用捕捉。**
* `<LEVEL>_NOTHROW(expression)`
断言在表达式的测试期间不会抛出异常。

* **注意： 异常断言也有带消息的形式，也是在后面加_MESSAGE**
##9.5 浮点数比较

当比较浮点数时，特别是至少有一个浮点数已经被计算出来时， 必须非常小心，以允许舍入误差和不精确的表示。
doctest提供了一种名为`doctest::Approx()`的包装类 来实现浮点数的含有误差的比较。 `doctest::Approx()`可以用在比较表达式的两边。它重载了比较运算符以允许一定的误差。下面是一个简单的例子：
```
   REQUIRE(performComputation() == doctest::Approx(2.1));
```
默认情况下，使用使用微小的量（这是相对百分比来说），这能满足很多简单的舍入误差情况。当这个微小的量不能满足需要时，可以通过调用`doctest::Approx()`的`epsilon()`方法来指定误差值。如：
```
REQUIRE(22.0/7 == doctest::Approx(3.141).epsilon(0.01)); // allow for a 1% error
```
当处理非常大或非常小的数字时，指定一个比例会比较有用的，可以通过调用`doctest::Approx()`的`scale（）`方法来实现。

不要将断言宏用在在try / catch中因为REQUIRE宏会抛出异常来结束测试用例！
查看这个[例子](https://github.com/onqtam/doctest/blob/master/examples/all_features/assertion_macros.cpp)来了解断言宏的使用。

#10 测试用例与测试套件
在对头文件进行预处理和包含之后，测试对每个处理过的源文件从顶部到底部进行注册，但在源文件之间没有顺序。
##10.1 经典测试用例
尽管doctest完全支持传统的xUnit风格的基于类的测试用例写法，但这不是首选样式。doctest提供了在测试用例中嵌套子句的强大机制。测试用例和子用例：

* `TEST_CASE（test name）`
* `SUBCASE（subcase name）`  

test name 和subcase name 可以是空，引用，字符串。测试名称在doctest可执行文件中不必是唯一的。它们也应该是字符串。

经典测试用例[参加教程](https://github.com/onqtam/doctest/blob/master/doc/markdown/tutorial.md)。
测试用例也可以参数化——[参见文档](https://github.com/onqtam/doctest/blob/master/doc/markdown/parameterized-tests.md)。
可以通过使用命令行过滤测试用例和子用例——参见[命令行](https://github.com/onqtam/doctest/blob/master/doc/markdown/commandline.md)。
更多示例请参考[SUBCASE 和 BDD 用法示例](https://github.com/onqtam/doctest/blob/master/examples/all_features/subcases.cpp)。

##10.2 BDD风格的测试用例
doctest除了支持经典风格的测试用例之外，doctest还支持一种替代语法，该语法允许将测试写为“可执行规范”（[行为驱动开发](http://dannorth.net/introducing-bdd/)的早期目标之一）。这组宏映射到TEST_CASE s 和SUBCASE s，带有一些内部支持以使它们更使用更顺畅。

* `SCENARIO( scenario name )`——该宏映射到`TEST_CASE`并以相同的方式工作，但测试用例名称将以“Scenario:”为前缀。
* `SCENARIO_TEMPLATE( scenario name, type, list of types )`——该宏映射到`TEST_CASE_TEMPLATE`，并以相同的方式工作，但测试用例名称将以“Scenario:”为前缀。
* `SCENARIO_TEMPLATE_DEFINE( scenario name, type, id )`——该宏映射到`TEST_CASE_TEMPLATE_DEFINE`，并以相同的方式工作，但测试用例名称将以“Scenario:”为前缀。

* `GIVEN( something )`、`WHEN( something )`、 `THEN( something )`——这些宏映射到SUBCASE s上，但子用例的名称是前缀为“given：”，“when：”或“then：”的_something_s。
* `AND_WHEN( something )`、`AND_THEN( something )`——与WHEN和THEN类似，但前缀是以“and”开头。这些用于将WHEN s和THEN s链接在一起。

当使用这些宏中的任何一个时，控制台报告器会识别它们并格式化测试用例头文件，以使Givens，Whens和Thens对齐来提高可读性。
除了附加前缀和控制台报告器中的格式外，这些宏与TEST_CASE和SUBCASE完全相同。因此，没有执行这些宏的正确排序——这取决于程序员！

更多示例请参考[SUBCASE 和 BDD 用法示例](https://github.com/onqtam/doctest/blob/master/examples/all_features/subcases.cpp)。

##10.3 test fixtures
虽然doctest允许您将测试集合在一起作为一个测试用例中的子用例，但是有时候，使用更传统的Test fixtures将它们分组更方便。 doctest也完全支持这种方式。您将测试夹具定义为简单结构：
```cpp
class UniqueTestsFixture 
{
  private:
	  static int uniqueID;
  protected:
	  DBConnection conn;
  public:
	  UniqueTestsFixture() : conn(DBConnection::createConnection("myDB")) 
	   {
	   }
  protected:
	   int getID() 
	   {
	     return ++uniqueID;
	   }
 };

 int UniqueTestsFixture::uniqueID = 0;

 TEST_CASE_FIXTURE(UniqueTestsFixture, "Create Employee/No Name") 
 {
   REQUIRE_THROWS(conn.executeSQL("INSERT INTO employee (id, name) VALUES (?, ?)", getID(), ""));
 }
 TEST_CASE_FIXTURE(UniqueTestsFixture, "Create Employee/Normal") 
 {
   REQUIRE(conn.executeSQL("INSERT INTO employee (id, name) VALUES (?, ?)", getID(), "Joe Bloggs"));
 }
```
这两个测试用例将创建唯一命名的UniqueTestsFixture的派生类，因此可以访问getID（）保护方法和conn成员变量。这样可以确保两个测试用例能够使用相同的方法（DRY原则）创建一个DBConnection，并且创建的任何ID是唯一的，这使得执行测试的顺序并不重要。
##10.4 测试套件
测试用例可以分组成测试套件。这是通过TEST_SUITE（）或TEST_SUITE_BEGIN（）/ TEST_SUITE_END（）完成的。例如：
```cpp
TEST_CASE("") {} // not part of any test suite

TEST_SUITE("math") {
    TEST_CASE("") {} // part of the math test suite
    TEST_CASE("") {} // part of the math test suite
}

TEST_SUITE_BEGIN("utils");

TEST_CASE("") {} // part of the utils test suite

TEST_SUITE_END();

TEST_CASE("") {} // not part of any test suite
```
那么指定测试套件的测试用例可以通过指定过滤器的来执行——参见[命令行](https://github.com/onqtam/doctest/blob/master/doc/markdown/commandline.md)。

有关测试套件的更多例子请参看[断言宏示例](https://github.com/onqtam/doctest/blob/master/examples/all_features/assertion_macros.cpp)。
##10.5 修饰
测试用例可以使用如下附加属性进行修饰：
```cpp
TEST_CASE("name"
          * doctest::description("shouldn't take more than 500ms")
          * doctest::timeout(0.5)) {
    // asserts
}
```
可以同时使用多个修饰器。下面是目前支持的修饰器：  

* `skip（bool = true）`——标记要从执行中跳过的测试用例，除非使用--no-skip命令行选项。
* `may_fail（bool = true）`——如果任何给定的断言失败，测试不会失败（但仍然报告）—— 这可以用于标记正在进行的工作，或您不想立即修复但仍想在测试中追踪的已知问题。
* `should_fail（“text”）` ——类似may_fail（），但如果通过测试，也算作失败——如果要通知意外或第三方修复，他可能会很有用。
* `expected_failures（int）`——定义在测试用例中预计失败的断言数—— 当失败的断言数与声明的预期失败次数不同时，该测试用例算作失败。  
* `timeout（double）`——如果测试用例的执行超过此限制（以秒为单位），该测试用例就算做失败——但不会终止测试用例——这需要子进程支持 。
* `test_suite（“name”）` ——可以在测试用例中用来重载其所在的测试套件。  
* `description（“text”）`——描述测试用例。

修饰器采用的值在注册测试用例时（在全局初始化期间）计算，是在进入main（）之前，而不是在运行之前。
修饰器也可以应用于测试套件块，并且该块中的所有测试用例都继承这些属性。

```cpp
TEST_SUITE("some TS" * doctest::description("all tests will have this")) {
    TEST_CASE("has a description from the surrounding test suite") {
        // asserts
    }
}
TEST_SUITE("some TS") {
    TEST_CASE("no description even though in the same test suite as the one above") {
        // asserts
    }
}
```
测试用例可以重载他们从父测试套件继承的装饰器：

```cpp
TEST_SUITE("not longer than 500ms" * doctest::timeout(0.5))
{
    TEST_CASE("500ms limit")
    {
        // asserts
    }
    TEST_CASE("200ms limit" * doctest::timeout(0.2)) 
    {
        // asserts
    }
}




```
#11 参数化测试用例
测试用例可以通过类型和值进行参数化。

##11.1 值参数化测试用例
目前在doctest中有两种做数据驱动测试的方法，在未来会有更好的得支持。

* 在辅助函数中断言，并使用用户构造的数据数组来调用这个辅助函数：
```cpp
void doChecks(int data) 
{
    // do asserts with data
}

TEST_CASE("test name") 
{
    std::vector<int> data {1, 2, 3, 4, 5, 6};
    
    for(auto& i : data) 
    {
        CAPTURE(i); // log the current input data
        doChecks(i);
    }
}
```
缺点：
（1）在异常情况下（或REQUIRE断言失败），整个测试用例将结束，对剩余的输入数据不再测试。
（2）用户必须通过调用CAPTURE（）（或INFO（））来手动记录数据。  
（3）更多的样板 ——doctest应该提供用于生成数据的原语，但目前没有—— 所以用户必须编写自己的数据生成器。  

* 使用子用例来将数据设为不同值：
```cpp
 TEST_CASE("test name") 
 {
    int data;
    SUBCASE("") { data = 1; }
    SUBCASE("") { data = 2; }
    
    CAPTURE(data);
    
    // do asserts with data
}
```
缺点：
(1)不能很好地扩展——  为多个不同的输入编写这样的代码是不切实际的。
(2)用户必须通过调用CAPTURE（）（或INFO（））来手动记录数据。
然而有一个简单的方法可以把它封装成一个宏（为简单起见用C++11编写）：
```cpp
#include <algorithm>
#include <vector>
#include <string>
#define DOCTEST_VALUE_PARAMETERIZED_DATA(data, data_array)            \
static std::vector<std::string> _doctest_subcases = [&data_array]()   \
{                                                                     \
    std::vector<std::string> out;                                     \
    while(out.size() != data_array.size())                            \
        out.push_back(std::string(#data_array "[") +std::to_string(out.size() + 1) + "]");                                                    \
    return out;                                                       \
}();                                                                  \
    int _doctest_subcase_idx = 0;                                     \
    std::for_each(data_array.begin(), data_array.end(), [&](const auto& in) {                   \
        DOCTEST_SUBCASE(_doctest_subcases[_doctest_subcase_idx++].c_str()) { data = in; }       \
    })
```
现在可以这样使用：
```
TEST_CASE("test name")
{
    int data;
    std::list<int> data_array = {1, 2, 3, 4}; 
    // must be iterable—— std::vector<> would work as well

    DOCTEST_VALUE_PARAMETERIZED_DATA(data, data_array);
    
    printf("%d\n", data);
}
```
通过重复进入测试用例3次（在第一次进入之后）来打印4个数字——就像子用例那样。
这种方法的一个很大的局限性是宏不能和其他子类在同一个代码块{}缩进级别下使用（会很奇怪） - 它只能在子用例内使用。
`static std::vector<std::string>`是必要的，因为SUBCASE（）宏接受const char *参数，并不复制字符串，而是在内部保持指针——这就是为什么我们需要构建字符串的持久版。这可能会在将来版本中改变（接受一个字符串类），以方便使用……
## 5.2 模板测试用例 - 按类型参数化
假设您有多个相同的接口的实现，并希望确保它们都满足一些共同的要求。或者，您可能已经定义了符合“相同概念”的几种类型，并且您想要验证它们。在这两种情况下，您想针对不同类型重复相同的测试逻辑。
尽管您可以为每个要测试的类型编写一个TEST_CASE（并且您甚至可以将测试逻辑分解成从测试用例中调用的函数模板），但是它是乏味且不通用的：如果您需要M次测试超过N类型，你最终将编写M * N测试。
模板测试允许您在一个类型的列表上重复相同的测试逻辑。你只需要写一次测试逻辑。有两种方法可以做到：

* 直接将类型列表传递给模板测试用例
```cpp
typedef doctest::Types<char, short, int, long long int> the_types;

TEST_CASE_TEMPLATE("signed integers stuff", T, the_types) 
{
    T var = T();
    --var;
    CHECK(var == -1);
}
```
* 使用特定的唯一名称（标识符）定义模板测试用例，以供稍后实例化
```cpp
TEST_CASE_TEMPLATE_DEFINE("signed integer stuff", T, test_id) 
{
    T var = T();
    --var;
    CHECK(var == -1);
}

typedef doctest::Types<char, short, int, long long int> the_types;
TEST_CASE_TEMPLATE_INSTANTIATE(test_id, the_types);

typedef doctest::Types<float, double> the_types_2;
TEST_CASE_TEMPLATE_INSTANTIATE(test_id, the_types_2);
```
如果您正在设计一个接口或概念，您可以定义一组类型参数化测试来验证接口/概念的任何有效实现应具有的属性。然后，每个实现的作者都可以用他的类型实例化测试套件，以验证它是否符合要求，而不必重复编写类似的测试。
一个名为signed integers stuff的测试用例实例化为int类型，将产生以下测试用例名称：
```cpp
signed integers stuff<int>
```
默认情况下，所有基本类型（ int，bool，float ...）都具有由库提供的字符串。对于所有其他类型，用户将不得不使用TYPE_TO_STRING（类型）宏，如下所示：
```cpp
TYPE_TO_STRING(std::vector<int>);
```
`TYPE_TO_STRING`宏仅在当前源文件中有效果，因此如果在分开的源文件中使用相同类型的模板化测试用例，则需要在某些头文件中使用该宏。
其他测试框架使用头文件`<typeinfo>`来自动获取类型对应字符串，但是doctest不能在头文件的前向声明部分（公共头文件）中包含任何头文件—— 所以用户必须告知框架每种类型，这是为了实现最优的[编译时性能](https://github.com/onqtam/doctest/blob/master/doc/markdown/benchmarks.md)。

* **注意：**
 * 类型不会因唯一性而被过滤—— 相同的模板测试用例可以针对同一类型进行多次实例化——阻止的工作留给用户来做。
 * 当它仅是测试用例的名称的一部分时 ，您不需要为每个类型提供字符串—— 默认值为<>——测试仍然可以工作并且被区分。
 * `doctest :: Types <>`模板最多可以接受60个参数。
 * 如果启用了可变参数宏（请参阅[`DOCTEST_CONFIG_WITH_VARIADIC_MACROS`](https://github.com/onqtam/doctest/blob/master/doc/markdown/configuration.md#doctest_config_with_variadic_macros)），可以跳过typedef，并且可以直接在宏中构造类型列表，否则编译器会认为类型列表中的每个逗号都引入了一个新的宏参数。通过可变参数宏支持，TYPE_TO_STRING宏也可以和`std :: pair <int，float>`这样的类型一起使用。  
 * 如果您需要在1种以上的类型上进行参数化，则可以将多个类型打包为一个类型，如下所示：
```cpp
template <typename first, typename second>
struct TypePair
{
    typedef first  A;
    typedef second B;
};

typedef Types<
    TypePair<int, char>,
    TypePair<char, int>
> pairs;

TEST_CASE_TEMPLATE("multiple types", T, pairs) {
    typedef typename T::A T1;
    typedef typename T::B T2;
    // use T1 and T2 types
}
```
更多示例请参考[模板测试用例](https://github.com/onqtam/doctest/blob/master/examples/all_features/templated_test_cases.cpp)。
## 5.3 命令行
doctest没有任何命令行选项也工作相当不错，但是有一堆控制选项可用。分为如下几类：

* **Query flags**—打印测试结果后，程序退出而不执行任何测试用例（如果框架集成到提供自己的main（）入口点的客户端代码库中——在doctest :: Context对象上调用run（）后，程序应检查shouldExit（）方法的结果，然后退出 ——这留给用户完成）。
* **Int/String选项** ——它们需要在=符号之后的给出一个值—— 之间没有空格！例如：`--order-by = rand`。
* **Bool选项**——在=符号之后期待1 / yes / on / true或0 / no / off / false这样的值——但它们也可以像flags一样使用，“=值”部分可以省略—— 这样就默认为真。

过滤器使用通配符来进行匹配—— 其中`*`表示“匹配任何序列”，`？`意思是“匹配任何一个字符”。模式串应放在引号内，比如：`--test-case="*no sound*,vaguely named test number ?"`
如果用户提供main（）函数，所有选项也可以使用代码设置（使用默认值或者重新设置）。

|Query Flags|Description|
|:------|:---|
|`-?`、`--help`、`-h`|打印一个帮助消息,会列出所有这些标志/选项|
|`-v`、`--version`|打印doctest框架的版本|
|`-c`、`--count`|打印与当前过滤器匹配的测试用例数（见下文）|
|`-ltc`、`--list-test-cases`|按名字列出与当前过滤器匹配的所有测试用例（见下文）|
|`-lts`、`--list-test-suites`|按名字列出所有测试套件，每个测试套件至少有一个与当前过滤器匹配的测试用例（见下文）|



|Int/String Options|Description|
|--|--|
|`-tc`、`--test-case=<filters>`|根据测试名称过滤测试用例。默认情况下，所有测试用例都匹配，但是如果给这个过滤器赋予一个值，如`--test-case = * math *,* sound *`，则只有与模式串列表中的任意一个或多个匹配的测试用例才会得到执行/计算/列出|
|`-tce `、`--test-case-exclude=<filters>`	|类似`-test-case = <filters>`选项，只是会跳过与模式串匹配的测试用例|
|`-sf` 、  `--source-file=<filters>`	|与`--test-case = <filters>`类似，但是是基于文件名过滤|
|`-sfe`、 `--source-file-exclude=<filters>`	|与`--test-case-exclude = <filters>`类似，但是基于文件名过滤|
|`-ts`  、 `--test-suite=<filters>`	|与`--test-case = <filters>`类似，但是基测试套件名过滤|
|`-tse` 、`--test-suite-exclude=<filters>`	|与`--test-case-exclude = <filters>`类似，但是基于测试套件名过滤|
|`-sc`  、 `--subcase=<filters>`	|与`--test-case = <filters>`类似，但根据子用例名过滤 subcase|
|`-sce` 、`--subcase-exclude=<filters>`	|与`--test-case-exclude = <filters>`类似，但基于子用例名过滤|
|`-ob` 、  `--order-by=<string>`|测试用例在被执行之前将按照所在文件名/所在测试套件名/测试用例名/随机 进行排序， string的可能值是file、suit、name、rand，默认为file。
|`-rs `、`--rand-seed=<int>`|随机排序的种子|
|`-f ` 、`--first=<int>`	|通过当前过滤器执行的第一个测试用例——用于基于范围的执行——请[参见示例](https://github.com/onqtam/doctest/blob/master/examples/range_based_execution.py)|
|`-l` 、`--last=<int>`|通过当前过滤器执行的最后一个测试用例——用于基于范围的执行——请[参见示例](https://github.com/onqtam/doctest/blob/master/examples/range_based_execution.py)|
|`-aa`、`--abort-after=<int>`|测试框架在多次断言失败之后将停止测试。默认值为0，表示不会停止。请注意，框架使用异常来停止当前的测试用例，而不管断言（CHECK / REQUIRE）的级别如何——因此要在析构函数中断言要非常小心|
|`-scfl` 、`--subcase-filter-levels=<int>`|仅在前<int>级子用例中使用子用例过滤器，也只执行深度为<int>的子用例，默认值是非常高的数字，意味着过滤所有的子用例|
|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;||

|Bool Options|Description|
|--|--|
|`-s` 、`--success=<bool>`	|在输出结果中包含成功的断言|
|`-cs` 、`--case-sensitive=<bool>`	|过滤器要区分大小写|
|`-e`  、`--exit=<bool>`|测试结束后退出—— 只有当客户端[提供main（）入口点](https://github.com/onqtam/doctest/blob/master/doc/markdown/main.md)时，这才有意义——程序应该在`doctest :: Context`对象调用`run（）`之后检查`shouldExit（）`方法，然后退出—— 这留给用户来完成。这个想法是只有测试在客户端程序中才能执行 ，而自身并不能继续执行|
|`-d`、 `--duration=<bool>`	|以秒为单位打印每个测试所用时间|
|`-nt `、  `--no-throw=<bool>`|	跳过与[异常相关的断言](https://github.com/onqtam/doctest/blob/master/doc/markdown/assertions.md#exceptions)|
|`ne `、  `--no-exitcode=<bool`>	|始终返回成功的退出代码—— 即使测试失败|
|`-nr`  、 `--no-run=<bool>`|	跳过所有运行时的doctest操作（发生在程序进入main（）之前的测试注册除外）。如果将测试框架集成到提供了main（）入口点的客户端代码库并且用户想要跳过执行测试而只是使用该程序，这将非常有用|
|`-nv`、  `--no-version=<bool>`	|输出中的省略框架版本|
|`-nc` 、 `--no-colors=<bool>`|	禁用彩色输出|
|`-fc` 、 `--force-colors=<bool>`	|强制使用彩色输出，即使无法检测到tty|
|`-nb` 、  `--no-breaks=<bool>`	|当断言失败时，禁用调试器中的断点|
|`-ns`  、 `--no-skip=<bool>`|	不要跳过用装饰器标记为skip的测试用例|
|`-npf` 、`--no-path-filenames=<bool>`|	当打印文件名时，将路径从输出中删除—— 用于要在不同环境中得到来自测试框架的相同输出|
|`-nln`、 `--no-line-numbers=<bool>`|当打印源位置时，输出中的行号将被替换为0 ——用于即使源文件中的测试位置发生变化，也想获得相同的输出|
|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;||


* 所有的标志/选项都有一个前缀版本（前面带有--dt-）—— 例如`--version`也可以用`--dt-version`或`--dt-v`代替。
* 可以使用`DOCTEST_CONFIG_NO_UNPREFIXED_OPTIONS`来禁用所有无前缀的标志/选项。
* 这是为了在测试框架集成在客户端代码库中时，与客户端命令行选项互操作方便——所有doctest相关的标志/选项都可以作为前缀，因此没有冲突，因此用户可以从他们的选项解析中排除以--dt开头的所有内容。
* 如果没有排除以--dt-开头的一切flag/option的选项，那么dt_removed辅助类可能有助于过滤掉它们：
```cpp
#define DOCTEST_CONFIG_NO_UNPREFIXED_OPTIONS
#define DOCTEST_CONFIG_IMPLEMENT
#include "doctest.h"

class dt_removed 
{
    std::vector<const char*> vec;
public:
    dt_removed(const char** argv_in) 
    {
        for(; *argv_in; ++argv_in)
            if(strncmp(*argv_in, "--dt-", strlen("--dt-")) != 0)
                vec.push_back(*argv_in);
        vec.push_back(NULL);
    }

    int  argc() 
    { 
        return static_cast<int>(vec.size()) - 1; 
    }
    const char** argv() 
    { 
        return &vec[0]; 
    } // Note: non-const char **:
};

int program(int argc, const char** argv);

int main(int argc, const char** argv)
{
    doctest::Context context(argc, argv);
    int test_result = context.run();
    // run queries, or run tests unless --no-run

    if(context.shouldExit()) // honor query flags and --exit
        return test_result;

    dt_removed args(argv);
    int app_result = program(args.argc(), args.argv());

    return test_result + app_result; // combine the 2 results
}

int program(int argc, const char** argv) 
{
    printf("Program: %d arguments received:\n", argc - 1);
    while(*++argv)
        printf("'%s'\n", *argv);
    return EXIT_SUCCESS;
}
```
当我们按下面所示运行时：
```cpp
program.exe --dt-test-case=math* --my-option -s --dt-no-breaks
```
输出结果为：
```cpp
Program: 2 arguments received:
'--my-option'
'-s'
```
#12 日志宏
在测试用例中可以记录额外消息。
##12.1 INFO()
INFO（）宏允许以和std :: ostream，std :: cout等相同的方式使用插入运算符（<<）。
```cpp
INFO("The number is " << i);
```
此消息将与当前作用域中位于info之后或嵌套在当前作用域中的所有断言相关，并且只有在断言失败时才会打印。
注意，没有初始<< ——而是将插入序列放在括号中。
该消息不是立即构建的，而是仅在需要时才被延迟进行字符串化。这意味着rvalues（临时值）不能传递给INFO（）宏。所有文字常量都被视为标准的右值引用——除了C字符串文字（“like this one”）。这意味着即使正常的整数文字也不能直接使用—— 在传递给INFO（）之前，它们需要存储在变量/常量中。如果使用C ++14（或Visual Studio 2017），doctest提供`TO_LVALUE（）`宏来帮助这方面 ——它将任何文字常量或constexpr值转换为左值，可以像这样使用：
```cpp
constexpr int foo() 
{ 
	return 42; 
}
TEST_CASE("playing with literals and constexpr values") 
{
    INFO(TO_LVALUE(6) << " and this is a C string literal " << TO_LVALUE(foo()));
    CHECK(false);
}
```
TO\_LVALUE（）也可以在想要避面静态constexpr成员称为ODR-used的上下中起作用，请看[DOCTEST_CONFIG_ASSERTION_PARAMETERS_BY_VALUE](https://github.com/onqtam/doctest/blob/master/doc/markdown/configuration.md#doctest_config_assertion_parameters_by_value)。

>**一些注意事项**：
>
>* 延迟字符串化意味着值被字符串化发生在一个断言失败而不是在捕获时—— 所以该值可能已经改变。
* 如果库使用C++ 11右引用构建（请参阅[`DOCTEST_CONFIG_WITH_RVALUE_REFERENCES`](https://github.com/onqtam/doctest/blob/master/doc/markdown/configuration.md#doctest_config_with_rvalue_references)），则提供删除重载以禁止在INFO（）调用中捕获右值——因为延迟字符串化实际上缓存了指向对象的指针。对于C ++98临时值将再次无效，但会有可怕的编译错误。
* [`DOCTEST_CONFIG_NUM_CAPTURES_ON_STACK`](https://github.com/onqtam/doctest/blob/master/doc/markdown/configuration.md#doctest_config_num_captures_on_stack)配置标识符可用于控制使用多少栈空间来以避免为stream macros 分配堆空间。
* 可以使用流操纵器（来自<iomanip>），但需要创建为局部变量并作为左值使用。  
* 有关如何教导doctest对您的类型进行字符串的信息，请参阅[字符串化页面](https://github.com/onqtam/doctest/blob/master/doc/markdown/stringification.md)  。

延迟字符串和栈的使用意味着在常见的情况下，当无断言失败时，代码运行速度非常快。这使得它在循环中也适用——也许要记录迭代器。

还有一个CAPTURE（）宏，它是对INFO（）的一个更方便的包装：
```cpp
CAPTURE(some_variable)
```
这将为你处理变量名称的字符串（实际上它适用于任何表达式，而不仅仅是变量）。这将记录如下：
```cpp
  some_variable := 42
```

## 5.4 可选的测试失败时消息

  还有一些其他宏用于记录信息：

* `MESSAGE(message)`
* `FAIL_CHECK(message)`
* `FAIL(message)`
 
`FAIL（）`类似于REQUIRE断言 ——使测试用例失败并退出。 `FAIL_CHECK（）`的作用就像一个CHECK断言 ——使测试用例失败，但继续执行。 `MESSAGE（）`只打印一条消息。

在这些所有宏中，消息再次使用“流操作符<<， 如下所示：
```cpp
 FAIL("This is not supposed to happen! some var: " << var);
```
此外，这里没有延迟字符串化——字符串总是构造和打印，因此对记录的值没有限制——临时和值可被接受 ——与`INFO（）`宏不同。
还有一些打算供第三方库使用的消息宏，如mocking frameworks:
```
ADD_MESSAGE_AT(file, line, message)
ADD_FAIL_CHECK_AT(file, line, message)
ADD_FAIL_AT(file, line, message)
```
当将来自不同框架的断言与doctest集成时，它们将非常有用。

更多示例请参考[日志示例](https://github.com/onqtam/doctest/blob/master/examples/all_features/logging.cpp)。
# 13 main函数进入点
## 13.1 默认和自定义main函数
在C++中编写测试的通常方法是将源文件与它们测试的代码分开，形成只包含测试的可执行文件。在这种情况下，由doctest提供的默认main（）通常就足够了：
```cpp
#define DOCTEST_CONFIG_IMPLEMENT_WITH_MAIN
#include "doctest.h"
```
这应该在一个源文件中完成，甚至在一个单独没有其他的东西的文件中。
但是，如果您需要更多的控制——想要用代码设置执行上下文，或者希望将框架集成到自己的生产代码中，那么默认的main（）就无法胜任此工作了。在这种情况下请使用[`DOCTEST_CONFIG_IMPLEMENT.`](https://github.com/onqtam/doctest/blob/master/doc/markdown/configuration.md#doctest_config_implement)。
所有的命令行选项都可以这样设置（falg则不能，因为没有意义）。只能使用`doctest::Context`对象的`addFilter（）`或`clearFilters（）`方法附加或清除过滤器 ——用户无法使用代码删除特定过滤器。
```cpp
#define DOCTEST_CONFIG_IMPLEMENT
#include "doctest.h"

int main(int argc, char** argv) {
    doctest::Context context;

    // !!! THIS IS JUST AN EXAMPLE SHOWING HOW DEFAULTS/OVERRIDES ARE SET !!!

    // defaults
    context.addFilter("test-case-exclude", "*math*"); // exclude test cases with "math" in their name
    context.setOption("abort-after", 5);              // stop test execution after 5 failed assertions
    context.setOption("order-by", "name");            // sort the test cases by their name

    context.applyCommandLine(argc, argv);

    // overrides
    context.setOption("no-breaks", true);             // don't break in the debugger when assertions fail

    int res = context.run(); // run

    if(context.shouldExit()) // important - query flags (and --exit) rely on the user doing this
        return res;          // propagate the result of the tests
    
    int client_stuff_return_code = 0;
    // your program - if the testing framework is integrated in your production code
    
    return res + client_stuff_return_code; // the result from doctest is propagated here as well
}
```
注意在上下文对象中调用`.shouldExit（）` ——这非常重要—— 当使用查询标志（或--no-run选项设置为true）时将会设置它，并且以正常方式退出应用程序是用户有责任。

##13.2 处理共享对象（DLL）
框架可以在二进制文件（可执行文件/共享对象）中单独使用，每个都有自己的测试运行器——这样甚至可以使用不同版本doctest——但是没有简单的方法可以从加载的全部二进制文件中执行测试，并让结果汇总。
还有一个选项可以让测试运行器（实现）内置在二进制文件中，并通过导出它的公共符号与其他程序共享（因此有一个单独的测试注册表），（是用户编写测试所需的是框架的所有的前向声明）。
更多信息请看[DOCTEST_CONFIG_IMPLEMENTATION_IN_DLL](https://github.com/onqtam/doctest/blob/master/doc/markdown/configuration.md#doctest_config_implementation_in_dll)配置标识符和[示例](https://github.com/onqtam/doctest/blob/master/examples/executable_dll_and_plugin)。

# 14 配置
doctest旨在尽可能地“高效工作”。它还允许使用一组标识符配置它的构建方式。
标识符应在包含框架头文件之前定义。
全局定义的东西意味着每个源文件的二进制（可执行/共享对象）都共享配置。

* [DOCTEST_CONFIG_IMPLEMENT_WITH_MAIN](#1)
* [DOCTEST_CONFIG_IMPLEMENT](#2)
* [DOCTEST_CONFIG_DISABLE](#3)
* [DOCTEST_CONFIG_IMPLEMENTATION_IN_DLL](#4)
* [DOCTEST_CONFIG_NO_SHORT_MACRO_NAMES](#5)
* [DOCTEST_CONFIG_NUM_CAPTURES_ON_STACK](#6)
* [DOCTEST_CONFIG_TREAT_CHAR_STAR_AS_STRING](#7)
* [DOCTEST_CONFIG_SUPER_FAST_ASSERTS](#8)
* [DOCTEST_CONFIG_USE_IOSFWD](#9)
* [DOCTEST_CONFIG_NO_COMPARISON_WARNING_SUPPRESSION](#10)
* [DOCTEST_CONFIG_NO_UNPREFIXED_OPTIONS](#11)
* [DOCTEST_CONFIG_NO_TRY_CATCH_IN_ASSERTS](#12)
* [DOCTEST_CONFIG_NO_EXCEPTIONS](#13)
* [DOCTEST_CONFIG_NO_EXCEPTIONS_BUT_WITH_ALL_ASSERTS](#14)
* [DOCTEST_CONFIG_ASSERTION_PARAMETERS_BY_VALUE](#15)
* [DOCTEST_CONFIG_COLORS_NONE](#40)
* [DOCTEST_CONFIG_COLORS_WINDOWS](#16)
* [DOCTEST_CONFIG_COLORS_ANSI](#17)
* [DOCTEST_CONFIG_WINDOWS_SEH](#18)
* [DOCTEST_CONFIG_NO_WINDOWS_SEH](#19)
* [DOCTEST_CONFIG_POSIX_SIGNALS](#20)
* [DOCTEST_CONFIG_NO_POSIX_SIGNALS](#21)
* [DOCTEST_CONFIG_INCLUDE_TYPE_TRAITS](#34)
检查C++的新特性：
* [DOCTEST_CONFIG_WITH_DELETED_FUNCTIONS](#22)
* [DOCTEST_CONFIG_WITH_RVALUE_REFERENCES](#23)
* [DOCTEST_CONFIG_WITH_VARIADIC_MACROS](#24)
* [DOCTEST_CONFIG_WITH_NULLPTR](#25)
* [DOCTEST_CONFIG_WITH_LONG_LONG](#26)
* [DOCTEST_CONFIG_WITH_STATIC_ASSERT](#27)
* [DOCTEST_CONFIG_NO_DELETED_FUNCTIONS](#28)
* [DOCTEST_CONFIG_NO_RVALUE_REFERENCES](#29)
* [DOCTEST_CONFIG_NO_VARIADIC_MACROS](#30)
* [DOCTEST_CONFIG_NO_NULLPTR](#31)
* [DOCTEST_CONFIG_NO_LONG_LONG](#32)
* [DOCTEST_CONFIG_NO_STATIC_ASSERT](#33)

对于大多数人来说，唯一所需的配置是告诉doctest哪个源文件应该托管所有的实现代码.

* <span id='1'>**`DOCTEST_CONFIG_IMPLEMENT_WITH_MAIN`**</spa>

```cpp
#define DOCTEST_CONFIG_IMPLEMENT_WITH_MAIN
#include "doctest.h"

```

这只能在库实现的源文件中定义。

* <span id='2'>**`DOCTEST_CONFIG_IMPLEMENT`**</span>

如果客户端想要提供main（）函数（要么从代码中设置一些选项的值或将框架集成到其他现有的项目代码库中），那么就应该使用这个标识符。

* <span id='3'>**`DOCTEST_CONFIG_DISABLE`**</span>

最重要的配置选项之一所有与测试相关的配置选项都从二进制文件中删除——包括框架实现和写在任何地方测试用例！这是doctest最独特的功能之一。
这应该在全局定义。

* <span id='4'>**`DOCTEST_CONFIG_IMPLEMENTATION_IN_DLL`**</span>

这个宏将影响doctest的公共接口——所有编写测试所需的前向声明将被转换为导入的符号。这样，测试运行器不必在二进制（可执行/共享对象）中实现，并且可以从构建和导出它的另一个二进制文件重用。
要从二进制文件导出测试运行程序，只需在测试实现测试运行器的任何源文件中一起使用[`DOCTEST_CONFIG_IMPLEMENTATION_IN_DLL`](#4)与[`DOCTEST_CONFIG_IMPLEMENT`](#2)（或[`DOCTEST_CONFIG_IMPLEMENT_WITH_MAIN`](#1)，但此时其他二进制文件必须链接到可执行文件）。请注意，不应在导出doctest测试运行器的二进制文件的其他源文件中定义此标识符，否则将导致链接器冲突——在同一个二进制文件中具有相同的导入和导出符号。
查看这个[示例](https://github.com/onqtam/doctest/blob/master/examples/executable_dll_and_plugin)——它展示了如何在dll中实现测试运行器（甚至在一个动态加载的插件中测试）。
这应该在导入符号的二进制文件中全局定义。
应该仅在导出测试运行器的二进制文件的库的源文件中定义。

* <span id='5'>**`DOCTEST_CONFIG_NO_SHORT_MACRO_NAMES`**</span>

这将删除doctest中没有DOCTEST\_前缀的所有宏，如`CHECK`，`TEST_CASE`和`SUBCASE`。然后只有完整的宏名称可用——如`DOCTEST_CHECK`，`DOCTEST_TEST_CASE`和`DOCTEST_SUBCASE`。用户可以自己制作这些宏的简短版本——参见[示例](https://github.com/onqtam/doctest/blob/master/examples/all_features/alternative_macros.cpp)。
他可以在全局或仅在特定的源文件中进行定义。

* <span id='6'>**`DOCTEST_CONFIG_NUM_CAPTURES_ON_STACK`**</span>

使用此标识符，用户可以通过INFO（）日志记录宏来配置栈上的捕获次数（[请阅读更多信息](https://github.com/onqtam/doctest/blob/master/doc/markdown/logging.md)）。默认值为5——这意味着对于这样的调用：`INFO（var1 <<“la la”<< var2）`；所有3个被记录的变量将在栈上被捕获（支持2个以上——没有堆分配，除非在同一作用域内的断言失败）——栈中总共`5*（sizeof（void *）*2））`字节用于捕获。随后调用的INFO（）将拥有自己的栈空间。请注意，0是无效值。例子：。
```cpp
#define DOCTEST_CONFIG_NUM_CAPTURES_ON_STACK 10
#include <doctest.h>
```
或通过命令行：`-DDOCTEST_CONFIG_NUM_CAPTURES_ON_STACK = 10`
应当在全局定义。

* <span id='7'>**`DOCTEST_CONFIG_TREAT_CHAR_STAR_AS_STRING`**</span>

默认情况下，char *被视为指针。使用此选项比较char *指针将切换到为用strcmp（）进行比较，并且当字符串化时，将打印字符串而不是指针值。
应当在全局定义。

* <span id='8'>**`DOCTEST_CONFIG_SUPER_FAST_ASSERTS`**</span>

这使得快速断言宏（FAST_CHECK_EQ（a，b）——具有FAST_前缀）[编译更快](https://github.com/onqtam/doctest/blob/master/doc/markdown/benchmarks.md#cost-of-an-assertion-macro)！（50-80％）
每个快速断言变成一个单一的函数调用——唯一的缺点是：如果一个断言失败并且调试器被附加——当中断时，它将变成一个内部函数——用户必须在1级调用栈才能看到实际的断言。
这可以在全局和仅在特定的源文件中进行定义。

* <span id='9'>**`DOCTEST_CONFIG_USE_IOSFWD`**</span>

默认情况下，库没有提供`std::ostream`的前向声明来支持运算符<<的字符串化机制。这是被标准禁止的（即使它在所有测试的编译器上都可以使用）。但如果用户希望100％与标准兼容，则可以使用此配置选项强制包含`<iosfwd>`头文件。
另外，可能一些使用niche的编译器的STL实现定义了不同的用法—— 那么在STL头中会出现编译错误，使用这个选项可以解决该问题。
应该在全局定义。

* <span id='10'>**`DOCTEST_CONFIG_NO_COMPARISON_WARNING_SUPPRESSION`**</span>

默认情况下，库禁止关于比较有符号和无符号类型时的警告。

> * g++/clang -Wsign-conversion
* g++/clang -Wsign-compare
* msvc C4389 'operator' : signed/unsigned mismatch
* msvc C4018 'expression' : signed/unsigned mismatch

您可以查看[该问题](https://github.com/onqtam/doctest/issues/16#issuecomment-246803303)，以更好地了解为什么默认禁止这些警告。
这可以在全局和仅在特定的源文件中进行定义。

* <span id='11'>**`DOCTEST_CONFIG_NO_UNPREFIXED_OPTIONS`**</span>

这将禁用[命令行选项](https://github.com/onqtam/doctest/blob/master/doc/markdown/commandline.md)的简短版本，只有具有--dt-前缀的版本才会由doctest解析——在测试框架集成到客户端代码库中时，与客户端命令行选项的处理会更容易交互——没有冲突，所以用户可以从他们的选项解析中排除以--dt-开头的所有内容。
这应该仅在库实现的源文件中定义（仅在那里）。

* <span id='12'>**`DOCTEST_CONFIG_NO_TRY_CATCH_IN_ASSERTS`**</span>

这将从下面的情况中删除所有try / catch部分，：
 
 >* [正常断言](https://github.com/onqtam/doctest/blob/master/doc/markdown/assertions.md#expression-decomposing-asserts)
* [二元和一元断言](https://github.com/onqtam/doctest/blob/master/doc/markdown/assertions.md#binary-and-unary-asserts)——使其功能与快速断言相同（但不包括编译速度）

因此在断言中评估一个表达式时，若抛出异常，将终止当前的测试用例。
这可以在全局和仅在特定的源文件中进行定义。

* <span id='13'>**`DOCTEST_CONFIG_NO_EXCEPTIONS`**</span>

这将从框架中删除使用异常的所有内容——如果使用`-fno-exceptions`禁用异常，也可以为某些编译器（GCC，Clang）自动检测到。对于MSVC `_HAS_EXCEPTIONS`不能用于自动检测，因为它在系统头文件中定义，而不是项目中定义，并且doctest不会包含这个头文件。

什么变了：

>* 在这样的上下文中，在try / catch部分中的测试表达式的断言不再进行测试。
* `REQUIRE`宏已经消失（未定义）
* [`异常宏消失`](https://github.com/onqtam/doctest/blob/master/doc/markdown/assertions.md#exceptions)（未定义）
* `abort-after`选项将不能完全正常的工作，因为异常用于终止测试用例。

当测试用例失败时，REQUIRE系列的断言使用异常终止当前测试用例。使用异常而不是简单的返回；因为断言不仅可以在测试用例中使用，还可以在测试用例的函数调用中使用。
还有一些像REQUIRE断言（终止测试用例）的日志记录宏——就像FAIL（）——开始不一样的工作——FAIL_CHECK（）。

[`DOCTEST_CONFIG_NO_EXCEPTIONS`](#13)意味着[`DOCTEST_CONFIG_NO_TRY_CATCH_IN_ASSERTS`](#12)。
如果您希望使用可以处理异常的断言，但有时构建时又不用异常——请检查[`DOCTEST_CONFIG_NO_EXCEPTIONS_BUT_WITH_ALL_ASSERTS`](#14)配置选项。

这应该在全局定义。

* <span id='14'>**`DOCTEST_CONFIG_NO_EXCEPTIONS_BUT_WITH_ALL_ASSERTS`**</span>

当不用异常构建测试时(参考[`DOCTEST_CONFIG_NO_EXCEPTIONS`](https://github.com/onqtam/doctest/blob/master/doc/markdown/configuration.md#doctest_config_no_exceptions))，RQUIRE宏和处理异常的宏都将消失（未定义）。
然而，如果您希望可以使用处理异常的断言，但有时构建时又不用异常——就可以使用这个配置项，影响如下：
>* REQUIRE断言并没有消失——但是他们的行为就像CHECK宏——当其中一个测试失时败，整个测试用例将被标记为失败，但不会立即退出。
* [处理异常的断言](https://github.com/onqtam/doctest/blob/master/doc/markdown/assertions.md#exceptions)变为空操作（而不是完全未定义）。

这可以在全局和仅在特定的源文件中进行定义。

* <span id='15'>**`DOCTEST_CONFIG_ASSERTION_PARAMETERS_BY_VALUE`**</span>
此选项强制所有doctest断言按值复制给定的表达式，而不是将其绑定到const引用。这有助于避免静态常量的ODR-usage（可能导致g++ / clang的链接器错误）：
```cpp
template<typename T> 
struct type_traits { static const bool value = false; };

// unless DOCTEST_CONFIG_ASSERTION_PARAMETERS_BY_VALUE is defined the following assertion
// will lead to a linker error if type_traits<int>::value isn't defined in a translation unit
CHECK(type_traits<int>::value == false);
```
这可以在全局和仅在特定的源文件中进行定义。

* <span id='40'>**`DOCTEST_CONFIG_COLORS_NONE`**</span>

这将消除在框架的控制台输出中对颜色的支持。 
这应该仅在库实现的源文件中定义（仅在那里）。

* <span id='16'>**`DOCTEST_CONFIG_COLORS_WINDOWS`**</span>

这将强制对控制台输出颜色的支持，以使用Windows API和头文件。
 这应该仅在库实现的源文件中定义（仅在那里）。

* <span id='17'>**`DOCTEST_CONFIG_COLORS_ANSI`**</span>

这将在控制台强制支持输出颜色，以使用ANSI转义代码。
这应该仅在库实现的源文件中定义（仅在那里）。

* <span id='18'>**`DOCTEST_CONFIG_WINDOWS_SEH`**</span>

这将在Windows上启用SEH处理。目前仅在使用MSVC编译时启用，因为某些版本的MinGW没有必要的Win32 API支持。用户可以选择明确启用该功能—— 已知可以使用在MinGW-w64项目中。
这应该仅在库实现的源文件中定义（仅在那里）。

* <span id='19'>**`DOCTEST_CONFIG_NO_WINDOWS_SEH`**</span>

当`DOCTEST_CONFIG_WINDOWS_SEH`被默认开启时，用于禁用它。
这应该仅在库实现的源文件中定义（仅在那里）。

* <span id='20'>**`DOCTEST_CONFIG_POSIX_SIGNALS`**</span>

这将允许在UNIX下使用信号来处理崩溃。默认开启，
这应该仅在库实现的源文件中定义（仅在那里）。

* <span id='21'>**`DOCTEST_CONFIG_NO_POSIX_SIGNALS`**</span>

当`DOCTEST_CONFIG_POSIX_SIGNALS`被默认开启时，用于禁用`DOCTEST_CONFIG_POSIX_SIGNALS`。
这应该仅在库实现的源文件中定义（仅在那里）。

*  <span id='34'>**`DOCTEST_CONFIG_INCLUDE_TYPE_TRAITS`**</span>

这被用于强制引入C++11的`<type_traits>`头文件。——这使得`Approx`辅助类可以使用`double`这样的强类型定义，查看[这个](https://github.com/onqtam/doctest/issues/62)或[这个](https://github.com/onqtam/doctest/issues/85)问题的以获得更多细节。
这可以在全局和仅在特定的源文件中进行定义。

* <span id='22'>**`DOCTEST_CONFIG_WITH_DELETED_FUNCTIONS`**</span>

doctest尝试检测C++ 11已被删除的函数是否可用，如果没有检查到，用户可以定义这个宏。
这应该在全球定义。

* <span id='23'>**`DOCTEST_CONFIG_WITH_RVALUE_REFERENCES`**</span>

doctest尝试检测C++ 11右值引用是否可用，如果没有检查到，用户可以定义这个宏。
这应该在全球定义。

* <span id='24'>**`DOCTEST_CONFIG_WITH_VARIADIC_MACROS`**</span>

doctest尝试检测C++ 11 可变参数宏 是否可用，如果没有检查到，用户可以定义这个宏。
这可以在全局和仅在特定的源文件中进行定义。

* <span id='25'>**`DOCTEST_CONFIG_WITH_NULLPTR`**</span>

doctest尝试检测C++ 11 `nullptr`是否可用，如果没有检查到，用户可以定义这个宏。
这应该在全球定义。

* <span id='26'>**`DOCTEST_CONFIG_WITH_LONG_LONG`**</span>

doctest尝试检测C++ 11 `long long` 的功能是否可用，如果没有检查到，用户可以定义这个宏。
这应该在全球定义。

* <span id='27'>**`DOCTEST_CONFIG_WITH_STATIC_ASSERT`**</span>

doctest尝试检测C++ 11 `static_assert`是否可用，如果没有检查到，用户可以定义这个宏。
这应该在全球定义。



* <span id='28'>**`DOCTEST_CONFIG_NO_DELETED_FUNCTIONS`**</span>

如果doctest检测到支持C++11 已删除函数，并且用户很了解，这可以用来禁用该功能。
这应该在全球定义。

* <span id='29'>**`DOCTEST_CONFIG_NO_RVALUE_REFERENCES`**</span>

如果doctest检测到支持C++11 右值引用，并且用户很了解，这可以用来禁用该功能。
这应该在全球定义。

* <span id='30'>**`DOCTEST_CONFIG_NO_VARIADIC_MACROS`**</span>

如果doctest检测到支持C++11 可变参数宏，并且用户很了解，这可以用来禁用该功能。
这可以在全局和仅在特定的源文件中进行定义。

* <span id='31'>**`DOCTEST_CONFIG_NO_NULLPTR`**</span>

如果doctest检测到支持C++11 `nullptr`，并且用户很了解，这可以用来禁用该功能。
这应该在全球定义。

* <span id='32'>**`DOCTEST_CONFIG_NO_LONG_LONG`**</span>

如果doctest检测到支持C++11 `long long`，并且用户很了解，这可以用来禁用该功能。
这应该在全球定义。

* <span id='33'>**`DOCTEST_CONFIG_NO_STATIC_ASSERT`**</span>

如果doctest检测到支持C++11 `static_assert（）`，并且用户很了解，这可以用来禁用该功能。
这应该在全球定义。

# 15 字符串
##15.１ 字符串转换
doctest要能够转换在断言中使用的类型，并将表达式记录到字符串中（用于记录和报告）。大多数内置类型都支持开箱即用，但有三种方法可以告诉doctest如何将自己的类型（或其他第三方类型）转换为字符串。

* 重载运算符`<< `

这是在C++中提供字符串转换的标准方式，您可能已经为自己的目标提供了这种方式。如果你不熟悉这种用法，和写一个自由函数类似：
```cpp
std::ostream& operator<< (std::ostream& os, const T& value) 
{
    os << convertMyTypeToString(value);
    return os;
}
```
（其中T是您的类型，convertMyTypeToString是您将编写的使您的类型可打印的任何代码——它不必在另一个函数中）。
您应该将此函数放在与您的类型相同的命名空间中。
或者你可能更喜欢把它写成一个成员函数：
```cpp
std::ostream& T::operator<<(std::ostream& os) const 
{
    os << convertMyTypeToString(*this);
    return os;
}
```

* 重载`doctest::toString` 

如果你不想重载运算符<<，或者你想为测试目的转换不同类型，你可以重载`toString（）`，其返回`doctest::String`。
```cpp
namespace user 
{
    struct udt {};
    
    doctest::String toString(const udt& value) 
    {
        return convertMyTypeToString(value);
    }
}
```
请注意，该函数必须在与您的类型相同的命名空间中。如果类型不在任何命名空间中，那么重载也应在全局命名空间中。convertMyTypeToString是您编写以使您的类型可打印的任何代码。

* `doctest::StringMaker<T>`特例化
在某些情况下，重载`toString`不能按预期工作。特例化`StringMaker <T>`为您提供更精确和可靠的控制——但代价和复杂性稍微增加：
```cpp
namespace doctest 
{
    template<> 
    struct StringMaker<T> 
    {
        static String convert(const T& value)   
        {
            return convertMyTypeToString(value);
        }
    };
}
``` 

##15.2 异常转义
默认情况下，所有从`std::exception`排派生的异常将通过调用·what（）·方法转换为字符串。对于不是从`std::exception`派生的异常类型或者`what（）`不返回合适的字符串，使用`REGISTER_EXCEPTION_TRANSLATOR`。这定义了一个函数，它接收你的异常类型并返回一个`doctest ::String`。它可以出现在代码的任何地方——它不必在同一个转化单元中。例如：
```cpp
REGISTER_EXCEPTION_TRANSLATOR(MyType& ex) 
{
    return doctest::String(ex.message());
}
```
请注意，异常可被接受而无需引用，但在C++中被认为是不良做法。
注册异常转换器的另一种方法是在执行任何测试之前执行以下函数：
```cpp
 // adding a lambda - the signature required is `doctest::String(exception_type)`
    doctest::registerExceptionTranslator<int>([](int in){ return doctest::toString(in); });
```
可以控制注册异常翻译器的顺序,只需按照所需的顺序显式的调用 函数，或者在单个翻译单元（源文件）中以顶端到底部的方式列出异常转换器的宏,对单个源文件来说，doctest中自动注册都以顶端到底部的方式工作。

查看[这个示例](https://github.com/onqtam/doctest/blob/master/examples/all_features/stringification.cpp)来看如何对`std::vector<T>`和其他类型/异常进行字符串化。
注意：当特例化`StringMaker<T> `或者重载`toString()`时，type string 将被使用——doctest也使用type string。`std::string` 不是候选项，因为 doctest没有引入头文`<string>`。
为了支持运算符<<、（、std::ostream&...字符串库没有提供std::ostream的前向声明，虽然在库中就是这样，但在标准中是被禁止的。目前在所有测试编译器中都可以工作，但如果用户希望与标准百分百兼容——标识符`DOCTEST_CONFIG_USE_IOSFWD `应被使用以强制引入头文件`<iosfwd>`。默认不引入该头文件的原因是因为在MSVC中，它会移入一大堆东西，预处理器完成后，转化文件已经增长到42000行的C++代码，而Clang和libc则实现得非常好，引入头文件`<iosfwd>`只是不到400行代码。





  [1]:https://github.com/onqtam/doctest/blob/master/doc/markdown/benchmarks.md
  [2]:https://github.com/onqtam/doctest/raw/master/scripts/data/using_doctest_888px_wide.gif
  [3]:https://github.com/onqtam/doctest/raw/master/scripts/data/benchmarks/header.png
  








------

<font color=blue>**_版权声明_**</font>：本文参考了<font color=blue>[《doctest项目的README》](<https://github.com/onqtam/doctest>)。</font><font color=red>未经作者允许，<font color=blue>严禁用于商业出版</font>，否则追究法律责任。网络转载请注明出处，这是对原创者的起码的尊重！！！</font>

------
