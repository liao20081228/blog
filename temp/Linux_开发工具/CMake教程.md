---
title: CMake教程
tags: cmake
---

------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>《[CMake tutorial v3.16](https://cmake.org/cmake/help/latest/guide/tutorial/index.html)》。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。网络转载请注明出处！！！</font>***

------

CMake教程提供了逐步指南，涵盖了CMake可以帮助解决的常见构建系统问题。 了解示例项目中各个主题如何协同工作将非常有帮助。 **示例的教程文档和源代码可在CMake源代码树的`Help/guide/tutorial`目录中找到**。 每个步骤都有其自己的子目录，其中包含可以用作起点的代码。 教程示例是渐进式的，因此每个步骤都为上一步提供了完整的解决方案。

# 1 基本起点（第1步）
最基本的项目是从源代码文件构建一个可执行文件。 对于简单的项目，只需三行`CMakeLists.txt`文件。 这是本教程的起点。 在Step1目录中创建一个`CMakeLists.txt`文件，如下所示：
```cmake
cmake_minimum_required(VERSION 3.10)

# set the project name
project(Tutorial)

# add the executable
add_executable(Tutorial tutorial.cxx)
```
请注意，此示例在`CMakeLists.txt`文件中使用小写的命令。 CMake支持大写，小写和大小写混合的命令。 Step1目录中提供了`tutorial.cxx`的源代码，可用于计算数字的平方根。


## 1.1 添加版本号和配置头文件
我们将添加的第一个功能是为我们的可执行文件和项目提供版本号。 虽然我们可以仅在源代码中执行此操作，但是使用`CMakeLists.txt`可以提供更大的灵活性。

首先，修改`CMakeLists.txt`文件来设置版本号。
```cmake
cmake_minimum_required(VERSION 3.10)

# set the project name and version
project(Tutorial VERSION 1.0)


###早期版本的写法
###		project(Tutorial）
###		set (Tutorial_VERSION_MAJOR 1)
###		set (Tutorial_VERSION_MINOR 0)
```
然后，配置一个头文件，将版本号传递给源代码:
```cmake
configure_file(TutorialConfig.h.in TutorialConfig.h)

###早期版本的写法
###		configure_file ("${PROJECT_SOURCE_DIR}/TutorialConfig.h.in" "${PROJECT_BINARY_DIR}/TutorialConfig.h")
```
由于配置的文件将被写入二进制树中，所以我们必须将该目录添加到搜索include文件的路径列表中。在`CMakeLists.txt`文件的末尾添加以下行:

```cmake
#必须在add_excutable之后
target_include_directories(Tutorial PUBLIC  "${PROJECT_BINARY_DIR}")


###早期版本的写法:
###可以位于任意位置，一般放在add_excutable之前
###		include_directories("${PROJECT_BINARY_DIR}")
```
使用您喜欢的编辑器在源码目录中创建`TutorialConfig.h.in`，内容如下:
```c++
// the configured options and settings for Tutorial
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```
当CMake配置这个头文件时，`@Tutorial_VERSION_MAJOR@`和`@Tutorial_VERSION_MINOR@`的值将被替换。

接下来，修改`tutorial.cxx`以包括配置的头文件TutorialConfig.h。

最后，通过更新`tutorial.cxx`来打印出版本号，如下所示：

```c++
  if (argc < 2) {
    // report version
    std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << "."
              << Tutorial_VERSION_MINOR << std::endl;
    std::cout << "Usage: " << argv[0] << " number" << std::endl;
    return 1;
  }
  ```

完整的`CMakeLists.txt`如下：
```cmake
cmake_minimum_required(VERSION 3.10)

#set project name and version
project(Tutorial VERSION 1.0)

configure_file(TutorialConfig.h.in TutorialConfig.h)

#add the executable
add_executable(Tutorial tutorial.cxx)

target_include_directories(Tutorial PUBLIC  "${PROJECT_BINARY_DIR}"  )
```

## 1.2 指定c++标准

接下来，通过在`tutorial.cxx`中用`std::stod`替换atof，将一些C ++ 11功能添加到我们的项目中。 同时，删除`#include <cstdlib>`。
```c++
const double inputValue = std::stod(argv[1]);
```
我们需要在CMake代码中明确声明应使用正确的标志。 在CMake中启用对特定C ++标准的支持的最简单方法是使用`CMAKE_CXX_STANDARD`变量。 对于本教程，请将`CMakeLists.txt`文件中的`CMAKE_CXX_STANDARD`变量设置为11，并将`CMAKE_CXX_STANDARD_REQUIRED`设置为True：

```cmake
cmake_minimum_required(VERSION 3.10)

# set the project name and version
project(Tutorial VERSION 1.0)

# specify the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
```
## 1.3 构建和测试
运行cmake或cmake-gui以配置项目，然后使用所选的构建工具进行构建。

例如，从命令行我们可以导航到CMake源代码树的`Help /guide/tutorial`目录并运行以下命令：
```shell
mkdir Step1_build
cd Step1_build
cmake ../Step1
cmake --build .
```

导航到构建教程的目录（可能是make目录或Debug或Release构建配置子目录），然后运行以下命令：
```shell
Tutorial 4294967296
Tutorial 10
Tutorial
```

# 2 添加库（第2步）
现在，我们将添加一个库到我们的项目中。 该库是我们自己的实现的用于计算数字的平方根的库。 可执行文件可以使用此库，而不是使用编译器提供的标准平方根函数。

在本教程中，我们将库放入名为`MathFunctions`的子目录中。 该目录已包含头文件`MathFunctions.h`和源文件`mysqrt.cxx`。 源文件具有一个称为`mysqrt`的函数，该函数提供与编译器的`sqrt`函数类似的功能。

将以下一行`CMakeLists.txt`文件添加到`MathFunctions`目录中：
```cmake
add_library(MathFunctions mysqrt.cxx)
```
为了使用新的库，我们将在顶层`CMakeLists.txt`文件中添加`add_subdirectory`调用，以便构建该库。 我们将新的库添加到可执行文件，并将`MathFunctions`添加为include目录，以便可以找到`mqsqrt.h`头文件。 顶级`CMakeLists.txt`文件的最后几行现在应如下所示：
```cmake
# add the MathFunctions library
add_subdirectory(MathFunctions)

# add the executable
add_executable(Tutorial tutorial.cxx)

#必须位于add_excutable之后
target_link_libraries(Tutorial PUBLIC MathFunctions)

###早期版本的写法
###		target_link_libraries(Tutorial MathFunctions)

#add the binary tree to the search path for include files so that we will find TutorialConfig.h
target_include_directories(Tutorial PUBLIC "${PROJECT_BINARY_DIR}" "${PROJECT_SOURCE_DIR}/MathFunctions")
```

现在让我们将MathFunctions库设为可选。 虽然对于本教程而言确实不需要这样做，但是对于大型项目来说，这是很常见的。 第一步是向顶层`CMakeLists.txt`文件添加一个选项。
```cmake
option(USE_MYMATH "Use tutorial provided math implementation" ON)

# configure a header file to pass some of the CMake settings to the source code
configure_file(TutorialConfig.h.in TutorialConfig.h)
```
此选项将显示在CMake GUI和ccmake中，默认值ON，可由用户更改。 此设置将存储在缓存中，因此用户不必每次在构建目录上运行CMake时设置该值。

下一个更改是使构建和链接MathFunctions库成为布尔选项。 为此，我们将顶层`CMakeLists.txt`文件的结尾更改为如下所示：
```cmake
if(USE_MYMATH)
  add_subdirectory(MathFunctions)
  list(APPEND EXTRA_LIBS MathFunctions)
  list(APPEND EXTRA_INCLUDES "${PROJECT_SOURCE_DIR}/MathFunctions")
endif()

# add the executable
add_executable(Tutorial tutorial.cxx)

#必须位于add_executable之后
target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})

# add the binary tree to the search path for include files so that we will find TutorialConfig.h
target_include_directories(Tutorial PUBLIC "${PROJECT_BINARY_DIR}" ${EXTRA_INCLUDES})

###早期版本的写法
###		if(USE_MYMATH)
###		  	include_directories ("${PROJECT_SOURCE_DIR}/MathFunctions")
###	  		add_subdirectory (MathFunctions)
###	  		set(EXTRA_LIBS ${EXTRA_LIBS} MathFunctions)
###		endif(USE_MYMATH)
###		include_directories("${PROJECT_BINARY_DIR}")
###		add_executable(Tutorial tutorial.cxx)
###		target_link_libraries(Tutorial ${EXTRA_LIBS})
```
请注意，使用变量`EXTRA_LIBS`来收集任意可选库，以供以后链接到可执行文件中。 变量`EXTRA_INCLUDES`类似地用于可选的头文件。 当处理许多可选组件时，这是一种经典方法，我们将在下一步中介绍现代方法。

对源代码的相应更改非常简单。 首先，如果需要，在`tutorial.cxx`中包含`MathFunctions.h`头文件：

```c++
#ifdef USE_MYMATH
#  include "MathFunctions.h"
#endif
```
然后，在同一文件中，使USE_MYMATH控制使用哪个平方根函数：

```c++
#ifdef USE_MYMATH
  const double outputValue = mysqrt(inputValue);
#else
  const double outputValue = sqrt(inputValue);
#endif
```
由于源代码现在需要`USE_MYMATH`，因此可以使用以下行将其添加到`TutorialConfig.h.in`中：
```cmake.in
#cmakedefine USE_MYMATH
```
**练习**：为什么在`USE_MYMATH`选项之后配置TutorialConfig.h.in如此重要？ 如果我们将两者倒置会怎样？

运行cmake或cmake-gui以配置项目，然后使用所选的构建工具进行构建。 然后运行构建的Tutorial可执行文件。

使用ccmake或CMake GUI更新`USE_MYMATH`的值。 重新生成并再次运行本教程。 sqrt或mysqrt哪个函数可提供更好的结果？

完整的`CMakeLists.txt`文件如下：
```cmake
cmake_minimum_required(VERSION 3.5)                                                                                  
# set the project name and version
project(Tutorial VERSION 1.0)
 
# specify the C++ standard
set(CMAKE_CXX_STANDARD 11) 
set(CMAKE_CXX_STANDARD_REQUIRED True)
 
option(USE_MYMATH "Use tutorial provided math implementation" ON) 
 
# configure a header file to pass some of the CMake settings to the source code
configure_file(TutorialConfig.h.in TutorialConfig.h)
 
if(USE_MYMATH)
	add_subdirectory(MathFunctions)
	list(APPEND EXTRA_LIBS MathFunctions)
	list(APPEND EXTRA_INCLUDES "${PROJECT_SOURCE_DIR}/MathFunctions")
endif()
 
# add the executable
add_executable(Tutorial tutorial.cxx)

#必须位于add_executable之后
target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})  

# add the binary tree to the search path for include files so that we will find TutorialConfig.h
target_include_directories(Tutorial PUBLIC  "${PROJECT_BINARY_DIR}" ${EXTRA_INCLUDES}) 
```

# 3 添加库的使用要求（第3步）
使用要求可以更好地控制库或可执行文件的链接和include行，同时还可以更好地控制CMake内部目标的传递属性。 利用使用要求的主要命令是：
* target_compile_definitions
* target_compile_options
* target_include_directories
* target_link_libraries

让我们从第2步中重构代码，以利用现代的CMake方法编写使用要求。 我们首先声明，链接到MathFunctions的任何东西都需要包括当前源码目录，而MathFunctions本身不需要。 因此，这可以成为`INTERFACE`使用要求。

请记住，`INTERFACE`是指消费者需要的，而生产者不需要东西。 将以下行添加到`MathFunctions/CMakeLists.txt`的末尾：
```cmake
target_include_directories(MathFunctions INTERFACE ${CMAKE_CURRENT_SOURCE_DIR)
```
现在，我们已经指定了MathFunction的使用要求，我们可以安全地从顶级`CMakeLists.txt`中删除对`EXTRA_INCLUDES`变量的使用：
```cmake
if(USE_MYMATH)
  add_subdirectory(MathFunctions)
  list(APPEND EXTRA_LIBS MathFunctions)
endif()
... ...
... ...

target_include_directories(Tutorial PUBLIC "${PROJECT_BINARY_DIR}")
```

# 4 安装与测试（第4步）

现在，我们可以开始向项目添加安装规则和测试支持。

## 4.1 安装规则
安装规则非常简单：对于MathFunctions，我们要安装库和头文件，对于应用程序，我们要安装可执行文件和配置的头文件。

因此，在`MathFunctions/CMakeLists.txt`的末尾添加：
```cmake
install(TARGETS MathFunctions DESTINATION lib)
install(FILES MathFunctions.h DESTINATION include)
```
然后在顶级cmakelt .txt的末尾添加

```cmake
install(TARGETS Tutorial DESTINATION bin)
install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h" DESTINATION include)
  ```
这就是创建本教程的基本本地安装所需的全部工作。

运行cmake或cmake-gui以配置项目，然后使用所选的构建工具进行构建。 从命令行键入`cmake --install`进行安装（自3.15中引入，较早版本的CMake必须使用make install），或从IDE构建`INSTALL`目标。 这将安装适当的头文件，库和可执行文件。

CMake变量`CMAKE_INSTALL_PREFIX`用于确定文件的安装根目录。 如果使用`cmake --install`，则可以通过`--prefix`参数指定自定义安装目录。 对于多配置工具，请使用`--config`参数指定配置。

验证已安装的Tutorial可以运行。
## 4.2 测试支持

接下来，测试我们的应用程序。 在顶级`CMakeLists.txt`文件的末尾，我们可以启用测试，然后添加一些基本测试以验证应用程序是否正常运行。

```cmake
enable_testing()

# does the application run
add_test(NAME Runs COMMAND Tutorial 25)

# does it sqrt of 25
add_test (NAME Comp25 COMMAND Tutorial 25)
set_tests_properties (Comp25 PROPERTIES PASS_REGULAR_EXPRESSION "25 is 5")

# does the usage message work?
add_test(NAME Usage COMMAND Tutorial)
set_tests_properties(Usage  PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number")

# define a function to simplify adding tests
function(do_test target arg result)
  add_test(NAME Comp${arg} COMMAND ${target} ${arg})
  set_tests_properties(Comp${arg}  PROPERTIES PASS_REGULAR_EXPRESSION ${result} )
endfunction(do_test)

# do a bunch of result based tests
do_test(Tutorial 4 "4 is 2")
do_test(Tutorial 9 "9 is 3")
do_test(Tutorial 5 "5 is 2.236")
do_test(Tutorial 7 "7 is 2.645")
do_test(Tutorial 25 "25 is 5")
do_test(Tutorial -25 "-25 is [-nan|nan|0]")
do_test(Tutorial 0.0001 "0.0001 is 0.01")

###早期版本的写法
###		include(CTest)
###		add_test (TutorialRuns Tutorial 25)
###
###		add_test (TutorialComp25 Tutorial 25)
###		set_tests_properties (TutorialComp25 PROPERTIES PASS_REGULAR_EXPRESSION "25 is 5")
###
###		add_test (TutorialUsage Tutorial)
###		set_tests_properties (TutorialUsage PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number")
###
###		#define a macro to simplify adding tests, then use it
###		macro (do_test arg result)
###		  add_test (TutorialComp${arg} Tutorial ${arg})
###		  set_tests_properties (TutorialComp${arg} PROPERTIES PASS_REGULAR_EXPRESSION ${result})
###		endmacro (do_test)
###
###		do_test(4 "4 is 2")
###		do_test(9 "9 is 3")
###		do_test(5 "5 is 2.236")
###		do_test(7 "7 is 2.645")
###		do_test(25 "25 is 5")
###		do_test(-25 "-25 is [-nan|nan|0]")
###		do_test(0.0001 "0.0001 is 0.01")

```
第一个测试只是验证应用程序你能否运行，没有段错误或其他崩溃，并且返回值为零。 这是CTest测试的基本形式。

下一个测试使用`PASS_REGULAR_EXPRESSION`测试属性来验证测试的输出是否包含某些字符串。 在这种情况下，验证在提供了错误数量的参数时是否打印了用法消息。

最后，我们有一个名为`do_test`的函数，该函数运行应用程序并验证所计算的平方根对于给定输入是否正确。 对于`do_test`的每次调用，都会基于传递的参数将另一个测试添加到项目中，该测试具有名称，输入和预期结果。

重新构建应用程序，然后cd到二进制目录并运行`ctest -N`和`ctest -VV`。 对于多配置生成器（例如Visual Studio），必须指定配置类型。 例如，要在“调试”模式下运行测试，请从构建目录（而不是“调试”子目录！）中使用`ctest -C Debug -VV`。 或者，从IDE构建`RUN_TESTS`目标。


早期版本的另一种写法是：
```cmake

```
# 5 添加系统自检（第5步）

让我们考虑向我们的项目中添加一些代码，这些代码取决于目标平台可能不具备的功能。 对于此示例，我们将添加一些代码，具体取决于目标平台是否具有`log`和`exp`函数。 当然，几乎每个平台都具有这些函数，但对于本教程而言，假设它们并不常见。

如果平台具有`log`和`exp`，那么我们将使用它们来计算`mysqrt`函数中的平方根。 我们首先使用顶级`CMakeLists.txt`中的`CheckSymbolExists`模块测试这些函数的可用性。 我们将在`TutorialConfig.h.in`中使用新定义，因此请确保在配置该文件之前进行设置。
```cmake
include(CheckSymbolExists)
set(CMAKE_REQUIRED_LIBRARIES "m")
check_symbol_exists(log "math.h" HAVE_LOG)
check_symbol_exists(exp "math.h" HAVE_EXP)

###早期版本的写法
###		include (CheckFunctionExists)
###		check_function_exists (log HAVE_LOG)
###		check_function_exists (exp HAVE_EXP)

```
现在，将这些定义添加到`TutorialConfig.h.in`中，以便我们可以从`mysqrt.cxx中`使用它们：

```cmake
// does the platform provide exp and log functions?
#cmakedefine HAVE_LOG
#cmakedefine HAVE_EXP
```
修改`mysqrt.cxx`以包括cmath。 接下来，在`mysqrt`函数的同一文件中，我们可以使用以下代码提供基于`log`和`exp`(如果在系统上可用）的替代实现（在`return result; `前不要忘记`#endif`！）：

```c++
#if defined(HAVE_LOG) && defined(HAVE_EXP)
  double result = exp(log(x) * 0.5);
  std::cout << "Computing sqrt of " << x << " to be " << result
            << " using log and exp" << std::endl;
#else
  double result = x;
```
运行cmake或cmake-gui来配置项目，然后使用所选的构建工具进行构建并运行Tutorial可执行文件。

您会注意到，我们也没有使用`log`和`exp`，即使我们认为它们应该是可用。 我们应该很快意识到，我们忘记在`mysqrt.cxx`中包含`TutorialConfig.h`。

我们还需要更新`MathFunctions/CMakeLists.txt`，以便`mysqrt.cxx`知道此文件的位置：
```cmake
target_include_directories(MathFunctions INTERFACE ${CMAKE_CURRENT_SOURCE_DIR} PRIVATE ${CMAKE_BINARY_DIR})
```
完成此更新后，继续并再次构建项目，然后运行构建的Tutorial可执行文件。 如果仍未使用`log`和`exp`，请从构建目录中打开生成的TutorialConfig.h文件。 也许它们在当前系统上不可用？

哪个函数现在可以提供更好的结果,`sqrt`或`mysqrt`？


















## 5.1 指定编译定义
除了在`TutorialConfig.h`中保存`HAVE_LOG`和`HAVE_EXP`值，对我们来说还有更好的地方吗？ 让我们尝试使用`target_compile_definitions`。

首先，从`TutorialConfig.h.in`中删除定义。 我们不再需要包含`mysqrt.cxx`中的`TutorialConfig.h`或`MathFunctions/CMakeLists.txt`中的其他包含内容。

接下来，我们可以将`HAVE_LOG`和`HAVE_EXP的`检查移至`MathFunctions/CMakeLists.txt`，然后将这些值指定为`PRIVATE`编译定义。
```cmake
include(CheckSymbolExists)
set(CMAKE_REQUIRED_LIBRARIES "m")
check_symbol_exists(log "math.h" HAVE_LOG)
check_symbol_exists(exp "math.h" HAVE_EXP)

if(HAVE_LOG AND HAVE_EXP)
  target_compile_definitions(MathFunctions
                             PRIVATE "HAVE_LOG" "HAVE_EXP")
endif()
```
完成这些更新后，继续并重新构建项目。 运行内置的Tutorial可执行文件，并验证结果与本步骤前面的内容相同。

# 6 添加自定义命令和生成的文件(第6步)

出于本教程的目的,假设我们决定不再使用平台`log`和`exp`函数，而是希望生成一个可在`mysqrt`函数中使用的预计算值表。 在本节中，我们将在构建过程中创建表，然后将该表编译到我们的应用程序中。

首先，让我们删除`MathFunctions/CMakeLists.txt`中对`log`和`exp`函数的检查。 然后从mysqrt.cxx中删除对`HAVE_LOG`和`HAVE_EXP`的检查。 同时，我们可以删除`#include <cmath>`。


在`MathFunctions`子目录中，提供了一个名为`MakeTable.cxx`的新的源文件以生成表。

查看完文件后，我们可以看到该表是作为有效的C\+\+代码生成的，并且输出文件名作为参数传入。

下一步是将适当的命令添加到`MathFunctions/CMakeLists.txt`文件中，以构建MakeTable可执行文件，然后在构建过程中运行它。 需要一些命令来完成此操作。

首先，在`MathFunctions/CMakeLists.txt`的顶部，添加`MakeTable`的可执行文件，就像添加任何其他可执行文件一样。
```cmake
add_executable(MakeTable MakeTable.cxx)
```
然后，我们添加一个自定义命令，该命令指定如何通过运行MakeTable生成`Table.h`。

```cmake
add_custom_command(
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Table.h
  COMMAND MakeTable ${CMAKE_CURRENT_BINARY_DIR}/Table.h
  DEPENDS MakeTable
  )
```
接下来，我们必须让CMake知道`mysqrt.cxx`依赖于生成的文件`Table.h`。 这是通过将生成的`Table.h`添加到库MathFunctions的源列表中来完成的。

```cmake
add_library(MathFunctions mysqrt.cxx ${CMAKE_CURRENT_BINARY_DIR}/Table.h )
```
我们还必须将当前的二进制目录添加到include目录列表中，以便`mysqrt.cxx`可以找到并包含`Table.h`。
```cmake
target_include_directories(MathFunctions
          INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
          PRIVATE ${CMAKE_CURRENT_BINARY_DIR}
          )
```
现在，我们来使用生成的表。 首先，修改`mysqrt.cxx`以包含`Table.h`。 接下来，我们可以重写`mysqrt`函数以使用该表：
```cpp
double mysqrt(double x)
{
  if (x <= 0) {
    return 0;
  }

  // use the table to help find an initial value
  double result = x;
  if (x >= 1 && x < 10) {
    std::cout << "Use the table to help find an initial value " << std::endl;
    result = sqrtTable[static_cast<int>(x)];
  }

  // do ten iterations
  for (int i = 0; i < 10; ++i) {
    if (result <= 0) {
      result = 0.1;
    }
    double delta = x - (result * result);
    result = result + 0.5 * delta / result;
    std::cout << "Computing sqrt of " << x << " to be " << result << std::endl;
  }

  return result;
}
```
运行cmake或cmake-gui以配置项目，然后使用所选的构建工具进行构建。

构建此项目时，它将首先构建`MakeTable`可执行文件。 然后它将运行`MakeTable`来生成`Table.h`。 最后，它将编译包括了`Table.h`的`mysqrt.cxx`，以生成MathFunctions库。

运行Tutorial可执行文件，并验证它是否正在使用该表。

:wq# 7 构建一个安装程序（第7步）

接下来，假设我们想将项目分发给其他人，以便他们可以使用它。 我们希望在各种平台上提供二进制和源代码。 这与我们之前在“安装和测试”（第4步）中进行的安装有些不同，在“安装和测试”中，我们是安装根据源代码构建的二进制文件。 在此示例中，我们将构建支持二进制安装和包管理功能的安装程序包。 为此，我们将使用CPack创建平台特定的安装程序。 具体来说，我们需要在顶级`CMakeLists.txt `文件的底部添加几行。
```cmake
include(InstallRequiredSystemLibraries)
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/License.txt")
set(CPACK_PACKAGE_VERSION_MAJOR "${Tutorial_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${Tutorial_VERSION_MINOR}")
include(CPack)
```
这就是全部需要做的事。 我们首先包含`InstallRequiredSystemLibraries`。 该模块将包括项目当前平台所需的任何运行时库。 接下来，我们将一些CPack变量设置为存储该项目的许可证和版本信息的位置。 版本信息是在本教程的前面设置的，并且`license.txt`已包含在此步骤的顶级源目录中。

最后，我们包含CPack模块，该模块将使用这些变量和当前系统的其他一些属性来设置安装程序。

下一步是以常规方式构建项目，然后在其上运行CPack。 要构建二进制发行版，请从二进制目录运行：
```shell
cpack
```
要指定生成器，请使用-G选项。 对于多配置构建，请使用-C指定配置。 例如：
```shell
cpack -G ZIP -C Debug
```
要创建源码分发，您可以输入：
```shell
cpack --config CPackSourceConfig.cmake
```
或者，运行`make package`或在IDE中右键单击`Package`目标和`Build Project`。

运行在二进制目录中找到的安装程序。 然后运行已安装的可执行文件，并验证其是否有效。

# 8 添加Dashboard支持（第8步）
添加支持以将测试结果提交到Dashboard非常容易。 我们已经在“测试支持”中为我们的项目定义了许多测试。 现在，我们只需要运行这些测试并将其提交到Dashboard即可。 为了包含对Dashboard的支持，我们在顶层`CMakeLists.txt`中包含了CTest模块。

用
```cmake
# enable dashboard scripting
include(CTest)
```
替换
```cmake
# enable testing
enable_testing()
```
CTest模块将自动调用`enable_testing（）`，因此我们可以将其从CMake文件中删除。

我们还需要在顶级目录中创建一个`CTestConfig.cmake`文件，在该目录中我们可以指定项目的名称以及提交Dashboard的位置。
```cmake
set(CTEST_PROJECT_NAME "CMakeTutorial")
set(CTEST_NIGHTLY_START_TIME "00:00:00 EST")

set(CTEST_DROP_METHOD "http")
set(CTEST_DROP_SITE "my.cdash.org")
set(CTEST_DROP_LOCATION "/submit.php?project=CMakeTutorial")
set(CTEST_DROP_SITE_CDASH TRUE)
```
CTest将在运行时读入该文件。 要创建一个简单的Dashboard，您可以运行cmake或cmake-gui来配置项目，但不构建它。 而是，将目录更改为二进制树，然后运行：
```shell
ctest [-VV] -D Experimental
```
请记住，对于多配置生成器（例如Visual Studio），必须指定配置类型：
```shell
ctest [-VV] -C Debug -D Experimental
```
或者，从IDE中构建`Experimental`目标。

`ctest`将构建和测试项目，并将结果提交给Kitware公共仪表板Dashboard。 Dashboard的结果将被上传到Kitware的公共Dashboard：<https://my.cdash.org/index.php?project=CMakeTutorial>。

# 9 混合静态和动态库（第9步）

在本节中，我们将展示如何使用`BUILD_SHARED_LIBS`变量来控制`add_library`的默认行为，并允许控制如何构建没有显式类型（`STATIC，SHARED，MODULE或OBJECT`）的库。

为此，我们需要将`BUILD_SHARED_LIBS`添加到顶级`CMakeLists.txt`。 我们使用`option`命令，因为它允许用户可以选择该值是On还是Off。

接下来，我们将重构MathFunctions使其成为使用mysqrt或sqrt封装的真实库，而不是要求调用代码执行此逻辑。 这也意味着`USE_MYMATH`将不会控制构建MathFuctions，而是将控制此库的行为。

第一步是将顶级`CMakeLists.txt`的开始部分更新为：
```cmake
cmake_minimum_required(VERSION 3.10)

# set the project name and version
project(Tutorial VERSION 1.0)

# specify the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# control where the static and shared libraries are built so that on windows
# we don't need to tinker with the path to run the executable
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")

option(BUILD_SHARED_LIBS "Build using shared libraries" ON)

# configure a header file to pass the version number only
configure_file(TutorialConfig.h.in TutorialConfig.h)

# add the MathFunctions library
add_subdirectory(MathFunctions)

# add the executable
add_executable(Tutorial tutorial.cxx)
target_link_libraries(Tutorial PUBLIC MathFunctions)
```
既然我们已经使MathFunctions始终被使用，我们将需要更新该库的逻辑。 因此，在`MathFunctions/CMakeLists.txt`中，我们需要创建一个`SqrtLibrary`，当启用`USE_MYMATH`时有条件地对其进行构建。 现在，由于这是一个教程，我们将明确要求SqrtLibrary是静态构建的。

最终结果是`MathFunctions/CMakeLists.txt`应该如下所示：
```cmake
# add the library that runs
add_library(MathFunctions MathFunctions.cxx)

# state that anybody linking to us needs to include the current source dir
# to find MathFunctions.h, while we don't.
target_include_directories(MathFunctions
                           INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
                           )

# should we use our own math functions
option(USE_MYMATH "Use tutorial provided math implementation" ON)
if(USE_MYMATH)

  target_compile_definitions(MathFunctions PRIVATE "USE_MYMATH")

  # first we add the executable that generates the table
  add_executable(MakeTable MakeTable.cxx)

  # add the command to generate the source code
  add_custom_command(
    OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Table.h
    COMMAND MakeTable ${CMAKE_CURRENT_BINARY_DIR}/Table.h
    DEPENDS MakeTable
    )

  # library that just does sqrt
  add_library(SqrtLibrary STATIC
              mysqrt.cxx
              ${CMAKE_CURRENT_BINARY_DIR}/Table.h
              )

  # state that we depend on our binary dir to find Table.h
  target_include_directories(SqrtLibrary PRIVATE
                             ${CMAKE_CURRENT_BINARY_DIR}
                             )

  target_link_libraries(MathFunctions PRIVATE SqrtLibrary)
endif()

# define the symbol stating we are using the declspec(dllexport) when
# building on windows
target_compile_definitions(MathFunctions PRIVATE "EXPORTING_MYMATH")

# install rules
install(TARGETS MathFunctions DESTINATION lib)
install(FILES MathFunctions.h DESTINATION include)
```
接下来，更新`MathFunctions/mysqrt.cxx`以使用`mathfunctions`和`detail`命名空间：

```cpp
#include <iostream>

#include "MathFunctions.h"

// include the generated table
#include "Table.h"

namespace mathfunctions
{
namespace detail 
{
// a hack square root calculation using simple operations
double mysqrt(double x)
{
  if (x <= 0) 
    return 0;

  // use the table to help find an initial value
  double result = x;
  if (x >= 1 && x < 10) 
  {
    std::cout << "Use the table to help find an initial value " << std::endl;
    result = sqrtTable[static_cast<int>(x)];
  }

  // do ten iterations
  for (int i = 0; i < 10; ++i)
  {
    if (result <= 0) 
      result = 0.1;
    double delta = x - (result * result);
    result = result + 0.5 * delta / result;
    std::cout << "Computing sqrt of " << x << " to be " << result << std::endl;
  }

  return result;
}
}
}
```
我们还需要在`tutorial.cxx`中进行一些更改，以使其不再使用`USE_MYMATH`：
1. 始终包含` MathFunctions.h`
2. 始终使用`mathfunctions::sqrt`
3. 不要包含 `cmath`

最后，更新 `MathFunctions/MathFunctions.h`以使用dll导出定义:
```c++
#if defined(_WIN32)
#  if defined(EXPORTING_MYMATH)
#    define DECLSPEC __declspec(dllexport)
#  else
#    define DECLSPEC __declspec(dllimport)
#  endif
#else // non windows
#  define DECLSPEC
#endif

namespace mathfunctions {
double DECLSPEC sqrt(double x);
}
```
此时，如果您构建了所有内容，则会注意到链接失败，因为我们将没有位置独立代码的静态库与具有位置独立代码的库组合在一起。 解决方案是无论构建类型如何，都将SqrtLibrary的`POSITION_INDEPENDENT_CODE`目标属性显式设置为True。

```cmake
 # state that SqrtLibrary need PIC when the default is shared libraries
  set_target_properties(SqrtLibrary PROPERTIES
                        POSITION_INDEPENDENT_CODE ${BUILD_SHARED_LIBS}
                        )

  target_link_libraries(MathFunctions PRIVATE SqrtLibrary)
 ```
 练习：我们修改了`MathFunctions.h`以使用dll导出定义。 使用CMake文档，您可以找到一个帮助器模块来简化此过程吗？
 
 # 10 添加生成器表达式（第10步）
在构建系统生成期间会评估生成器表达式，以生成特定于每个构建配置的信息。

在许多目标属性（例如`LINK_LIBRARIES，INCLUDE_DIRECTORIES，COMPLIE_DEFINITIONS`等）的上下文中允许生成器表达式。 在使用命令填充这些属性（例如`target_link_libraries（），target_include_directories（） ,target_compile_definitions（）`等）时，也可以使用它们。

生成器表达式可用于启用条件链接，编译时使用的条件定义，条件包含目录等。 条件可以基于构建配置，目标属性，平台信息或任何其他可查询信息。

生成器表达式有不同类型，包括逻辑，信息和输出表达式。

逻辑表达式用于创建条件输出。 基本表达式是0和1表达式。` $<0:...>`导致空字符串，而`<1:...>`导致内容“…”。 它们也可以嵌套。

生成器表达式的常见用法是有条件地添加编译器标志，例如用于语言级别或警告的标志。 一个不错的模式是将该信息与一个`INTERFACE`目标相关联，以允许该信息传播。 让我们从构造一个`INTERFACE`目标并指定所需的`C++`标准级别`11`开始，而不是使用`CMAKE_CXX_STANDARD`。 

所以，下面的代码：
```cmake
# specify the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
```
将被替换为：
```cmake
add_library(tutorial_compiler_flags INTERFACE)
target_compile_features(tutorial_compiler_flags INTERFACE cxx_std_11)
```
接下来，我们为项目添加所需的编译器警告标志。 由于警告标志根据编译器的不同而不同，因此我们使用`COMPILE_LANG_AND_ID`生成器表达式来控制在给定一种语言和一组编译器ID的情况下应应用的标志，如下所示：
 
 ```cmake
 set(gcc_like_cxx "$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU>")
set(msvc_cxx "$<COMPILE_LANG_AND_ID:CXX,MSVC>")
target_compile_options(tutorial_compiler_flags INTERFACE
  "$<${gcc_like_cxx}:$<BUILD_INTERFACE:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>>"
  "$<${msvc_cxx}:$<BUILD_INTERFACE:-W3>>"
)
 ```
查看此内容，我们看到警告标志封装在`BUILD_INTERFACE`条件内。 这样做是为了使我们已安装项目的使用者不会继承我们的警告标志。

练习：修改`MathFunctions/CMakeLists.txt`，以便所有目标都具有对`tutorial_compiler_flags` `target_link_libraries()`调用。

# 11 增加输出配置（第11步）

在本教程的“安装和测试”（第4步）中，我们添加了CMake的功能，以安装项目的库和头文件。 在"构建安装程序"（第7步）期间，我们添加了打包此资料的功能，以便可以将其分发给其他人。

下一步是添加必要的信息，以便其他CMake项目可以使用我们的项目，无论是从构建目录，本地安装还是打包的文件。


第一步是更新我们的`install（TARGETS）`命令，不仅要指定`DESTINATION`，还要指定`EXPORT`。 `EXPORT`关键字生成并安装一个CMake文件，该文件包含用于从安装树中导入install命令中列出的所有目标的代码。 因此，让我们继续，通过更新`MathFunctions/CMakeLists.txt`中的`install`命令显式`EXPORT`MathFunctions库，如下所示：
```cmake
install(TARGETS MathFunctions tutorial_compiler_flags
        DESTINATION lib
        EXPORT MathFunctionsTargets)
install(FILES MathFunctions.h DESTINATION include)
```
现在我们已经导出了MathFunctions，我们还需要显式安装生成的`MathFunctionsTargets.cmake`文件。 这是通过将以下内容添加到顶级`CMakeLists.txt`的底部来完成的：

```cmake
install(EXPORT MathFunctionsTargets
  FILE MathFunctionsTargets.cmake
  DESTINATION lib/cmake/MathFunctions
)
```
此时，您应该尝试运行CMake。 如果一切设置正确，您将看到CMake将生成如下错误：

```text
Target "MathFunctions" INTERFACE_INCLUDE_DIRECTORIES property contains
path:

  "/Users/robert/Documents/CMakeClass/Tutorial/Step11/MathFunctions"

which is prefixed in the source directory.
```
CMake试图说的是，在生成导出信息的过程中，它将导出与当前机器固有联系的路径，并且在其他机器上无效。 解决方案是更新MathFunctions `target_include_directories`，以了解从构建目录和install/包中使用它时需要不同的`INTERFACE`位置。 这意味着将MathFunctions的`target_include_directories`调用转换为：
```cmake
target_include_directories(MathFunctions
                           INTERFACE
                            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
                            $<INSTALL_INTERFACE:include>
                           )
```
更新后，我们可以重新运行CMake并确认它不再发出警告。					   

至此，我们已经正确地打包了CMake所需的目标信息，但仍然需要生成`MathFunctionsConfig.cmake`，以便CMake `find_package`命令可以找到我们的项目。 因此，我们继续将名为`Config.cmake.in`新文件添加到项目顶层项目的顶层目录，其内容如下：			

```cmake
@PACKAGE_INIT@

include ( "${CMAKE_CURRENT_LIST_DIR}/MathFunctionsTargets.cmake" )
```
然后，要正确配置和安装该文件，请将以下内容添加到顶级`CMakeLists.txt`的底部：
```cmake
install(EXPORT MathFunctionsTargets
  FILE MathFunctionsTargets.cmake
  DESTINATION lib/cmake/MathFunctions
)

include(CMakePackageConfigHelpers)
# generate the config file that is includes the exports
configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/Config.cmake.in
  "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfig.cmake"
  INSTALL_DESTINATION "lib/cmake/example"
  NO_SET_AND_CHECK_MACRO
  NO_CHECK_REQUIRED_COMPONENTS_MACRO
  )
# generate the version file for the config file
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfigVersion.cmake"
  VERSION "${Tutorial_VERSION_MAJOR}.${Tutorial_VERSION_MINOR}"
  COMPATIBILITY AnyNewerVersion
)

# install the configuration file
install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfig.cmake
  DESTINATION lib/cmake/MathFunctions
  )
```
至此，我们为项目生成了可重定位的CMake配置，可以在安装或打包项目后使用它。 如果我们也希望从构建目录中使用我们的项目，则只需将以下内容添加到顶级`CMakeLists.txt`的底部：
```cmake
export(EXPORT MathFunctionsTargets
  FILE "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsTargets.cmake"
)
```
通过此导出调用，我们现在生成一个`Targets.cmake`，允许在构建目录中配置的`MathFunctionsConfig.cmake`由其他项目使用，而无需安装它。

# 12 导入一个CMake项目（消费者）

本示例说明项目如何查找生成`Config.cmake`文件的其他CMake软件包。

它还显示了在生成`Config.cmake`时如何声明项目的外部依赖关系。

# 13 打包调试和发布(多个包)
默认情况下，CMake的模型是一个构建目录仅包含一个配置，可以是Debug，Release，MinSizeRel或RelWithDebInfo。

但是可以将CPack设置为同时捆绑多个构建目录，以构建一个包含同一项目的多个配置的软件包。

首先，我们需要构建一个名为`multi_config`的目录，该目录将包含我们要打包在一起的所有构建。

其次，在`multi_config`下创建一个`debug`和`release`目录。 最后，您应该具有如下布局：

```
─ multi_config
    ├── debug
    └── release
```
现在，我们需要设置调试和发布版本，这大致需要以下内容：

```shell
cmake -DCMAKE_BUILD_TYPE=Debug ../../../MultiPackage/
cmake --build .
cd ../release
cmake -DCMAKE_BUILD_TYPE=Release ../../../MultiPackage/
cmake --build .
cd ..
```
既然调试和发行版本均已完成，我们就可以使用自定义的`MultiCPackConfig.cmake`文件将两个版本打包到一个发行版中。

```shell
cpack --config ../../MultiPackage/MultiCPackConfig.cmake
```




------

***<font color=blue>版权声明</font>：本文翻译自<font color=blue>《[CMake tutorial v3.16](https://cmake.org/cmake/help/latest/guide/tutorial/index.html)》。</font><font color=red>未经作者允许</font>，<font color=blue>严禁用于商业出版</font>，<font color=red>否则追究法律责任。网络转载请注明出处！！！</font>***

------
