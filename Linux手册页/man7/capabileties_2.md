---
title: capabileties_2 
tags: 新建,模板,小书匠
category: /小书匠/日记/2025-02
grammar_cjkRuby: true
---

# 过去和当前的实现情况

能力的完整实现要求：

- 对于所有的特权操作，内核必须检查线程在其有效集中是否具有所需的能力。
- 内核必须提供允许更改和获取线程能力的系统调用。
- 文件系统必须支持将能力添加到可执行文件，以便进程在执行文件时获得这些能力。

在Linux 2.6.24之前，只满足这些要求中的前两个；从Linux 2.6.24开始，满足所有三个要求。

# 内核开发者须知
在添加新的内核功能时，应考虑以下几点。
 - 能力的目标是将超级用户的权力分成若干块，这样，如果具有一个或多个能力的程序被破坏，它对系统造成破坏的权力将小于以root特权运行的同一程序。
 - 您可以选择为您的新特性创建新能力，或将该特性与现有能力之一关联。为了将能力集保持在可管理的大小，后一种选择是更可取的，除非有令人信服的理由采取前一种选择。（还有一个技术限制：能力集的大小目前被限制在64位。）
 - 要确定哪种现有能力可能最适合与您的新特性相关联，请查看上面的能力列表，以便找到您的新特性最适合的“能力项”。采取的一种方法是确定是否有其他特性要求与新功能一起使用的能力。如果没有这些其他特性，新特性就没用了，则你应该使用与其他特性相同的能力。
 - 如果可以避免，不要选择CAP_SYS_ADMIN! 很大一部分现有的能力检查都与此能力相关联（请参阅上面的部分列表）。它可以被合理地称为“新root”，因为一方面，它赋予了广泛的权力，另一方面，它的广泛范围意味着这是许多特权程序所需要的能力。不要让问题变得更糟。唯一应与CAP_SYS_ADMIN关联的新功能是与该能力中的现有用途非常匹配的功能。
 - 如果您确定确实有必要为您的特性创建新能力，请不要将其命名为“一次使用”能力。因此，例如，添加高度具体的CAP_SYS_PACCT可能是一个错误。相反，尝试将您的新能力标识并命名为更广泛的能力项，其他相关的未来用例可能会融入其中。

# 线程能力集

使用**capset**(2)，线程可以操作自己的能力集；请参阅下面的[以编程方式调整能力集](#Programmatically_adjusting_capability_sets)。

从Linux 3.2开始，文件<u>/proc/sys/kernel/cap_last_cap</u>公开了运行内核支持的最高能力的数值；这可以用来确定能力集中可能设置的最高位。

通过**fork**(2)创建的子进程继承其父的能力集的副本。有关**execve**(2)如何影响能力的详细信息，请参阅下面的[execve()期间的能力转换](#Transformation_of_capabilities_during_execve)。

每个线程具有以下包含零个或多个上述能力的能力集：

## Permitted
这是线程可能具有的effective能力的限制超集。它也是 可由在其effective集中不具有**CAP_SETPCAP**能力的线程添加到inheritable集中的的限制超集。

如果一个线程从其permitted集合中删除了一个能力，那么它永远不能重新获得该能力（除非它**execve**(2）一个set-user-ID-root程序，或者一个关联文件能力授予了该能力的程序)。

## Inheritable
这是在**execve**(2)后可保留的能力集。在执行任何程序时Inheritable能力仍然是可继承的，且当执行一个在文件Inheritable集中设置了相应bit位的程序时，Inheritable集中的能力将被添加到Permitted集中。

由于在以非root用户身份运行时，通常不会在**execve**(2)后保留Inheritable能力，因此希望运行具有提升能力的助手程序的应用程序应考虑使用ambient能力，如下所述。

## Effective
这是内核用来对线程执行权限检查的能力集。

## Bounding

能力Bounding集是一种机制，可用于限制在**execve**(2)期间获得的能力。

从Linux 2.6.25开始每线程都有此能力集。在较旧的内核中，Bounding能力集是系统范围的属性，由系统上的所有线程共享。

有关更多详细信息，请参阅下面的[Bounding能力集](#Capability_bounding_set)。

## Ambient
从Linux 4.3开始每线程都有此能力集。

这是一组非特权程序通过**execve**(2)保留的能力。Ambient能力集遵循不变式，即如果不能同时被允许和继承，则任何能力都不能是Ambient能力。

Ambient能力集可以直接使用**prctl**(2)进行修改。如果相应的permitted或Inheritable能力中的任何一个被降低，则Ambient能力将自动降低。

执行因set-user-ID或set-group-ID位而更改UID或GID的程序，或设置了任何文件能力的程序，将清除Ambient集。当调用**execve**(2)时，将Ambient能力添加到Permitted集和Effective集。如果在**execve**(2)期间，Ambient能力导致进程的Permitted和Effective能力增加，这不会触发 ld.so(8)中描述的安全执行模式。

# 文件能力
从Linux 2.6.24开始，内核支持使用**setcap**(8)将能力集与可执行文件关联。文件能力集存储在一个名为<u>security.capability</u>的扩展属性（参见**setxattr**(2）和**xattr**(7))中，写入该扩展属性需要使用**CAP_SETFCAP**能力。文件能力集与线程的能力集一起决定了在执行execve(2)后的线程能力。

三个文件能力集分别为：

## Permitted（以前称为强制）
这些能力被自动添加到线程的Permitted集，而忽略线程的inheritable能力。

## Inheritable（以前称为允许）
该集合与线程的inheritable集合进行且运算，以确定在**execve**(2)之后线程的Permitted集中的哪些inheritable能力被允许。

## Effective
这不是一个集合，而只是单个bit位。如果设置了此位，则在**execve**(2)期间，线程的所有新的Permitted能力也会提升到Effective集。如果未设置此位，则在执行一次**execve**(2)之后，新的Permitted能力都不在新的Effective中。

启用文件Effective能力位意味着导致线程在**execve**(2)期间获取相应Permitted能力的任何文件Permitted或Inheritable能力（请参阅下面的[execve()过程中的能力转换](#Transformation_of_capabilities_during_execve)）也将获得其Effective集中能力。因此，在为文件分配能力时(**setcap**(8)、**cap_set_file**(3)、**cap_set_fd**(3))，如果我们将effective标志指定为对任何能力启用，则相应的permitted或inheritable标志为启用的effective标志也必须指定为启动。


 # execve()过程中的能力转换
 
 在**execve**(2)期间，<span id='Transformation_of_capabilities_during_execve'>内核</span>使用以下算法计算进程的新能力：
```
P'(ambient)    = (file is privileged) ? 0 : P(ambient)
P'(permitted)  = (P(inheritable) & F(inheritable)) | (F(permitted) & P(bounding)) | P'(ambient)
P'(effective)  = F(effective) ? P'(permitted) : P'(ambient)
P'(inheritable)= P(inheritable)    \[i.e., unchanged]
P'(bounding)   = P(bounding)       \[i.e., unchanged]
```
其中：
P()表示**execve**(2)之前的线程能力集的值
P'()表示**execve**(2)之后线程能力集的值
F()表示文件能力集

<u>注意</u>：与上述能力转换规则相关的下列详细信息：

- ambient能力集从Linux 4.3开始才出现。在**execve**(2)期间确定ambient集的转换时，特权文件是指具有能力或set-user-ID或set-group-ID bit位的文件。
- 在Linux 2.6.25之前，Bounding集是所有线程共享的系统范围属性。在**execve**(2)期间使用该系统范围的值来计算新的Permitted集，其方式与上面显式的用于<u>P(bounding)</u>的方式相同。

<u>注意</u>：在上述能力转换期间，文件能力可能被忽略（视为空），原因与忽略set-user-ID和set-group-ID位相同；参见**execve**(2)。如果内核是用<u>no_file_caps</u>选项引导的，文件能力也同样会被忽略。

<u>注意</u>：根据上面的规则，如果具有非零用户ID的进程执行**execve**(2)，那么在其permitted和effective集中存在的任何能力都将被清除。有关用户ID为零的进程执行**execve**(2)时的能力的处理，请参阅下面的<u>[root程序的执行和能力](#Capabilities_and_execution_of_programs_by_root)</u>。


# root程序的执行和能力
<span id='Capabilities_and_execution_of_programs_by_root'>为了</span>镜像传统的UNIX语义，当具有UID 0 (root)的进程执行程序和执行set-user-ID-root程序时，内核会对文件功能进行特殊处理。

在对由二进制文件的set-user-ID模式位触发的进程有效ID执行任何更改之后，例如，由于执行set-user-ID-root程序，将有效用户ID切换为0 (root)，内核按如下方式计算文件能力集：

（1）如果进程的真实或有效用户ID为0（root），则忽略文件inheritable和permitted集；替代的是它们在概念上被认为是全1（即启用了所有能力）。（此行为有一个例外，在下面<u>[具有文件能力的Set-user-ID-root程序](#Set-user-ID-root_programs_that_have_file_capabilities)</u>中描述。）

(2)如果进程的有效用户ID为0（root）或者文件effective位实际上是启用的，那么文件effective位在概念上被定义为1（启用）。

然后，如上所述，使用文件能力集的这些名义值来计算**execve**(2)过程中进程能力的转换。

因此，当具有非零UID的进程**execve**(2)没有附加能力集的set-user-ID-root程序，或者当实际和有效UID为零的进程**execve**(2)执行一个程序时，该进程的新permitted能力的计算简化为：
```
P'(permitted)   = P(inheritable) | P(bounding)

P'(effective)   = P'(permitted)
```
因此，进程将获得其permitted和effective能力集中的所有能力，但被bounding集屏蔽的能力除外。（在P'(permitted）的计算中，P'（ambient）项可以简化，因为根据定义它是P'（inheritable）的适当子集。)

本小节中描述的对用户ID 0 (root)的特殊处理可以使用下面描述的securebits机制禁用。

# 具有文件能力的Set-user-ID-root程序
在<span id='Set-user-ID-root_programs_that_have_file_capabilities'>上面</span>的<u>[root程序的执行和能力](#Capabilities_and_execution_of_programs_by_root)</u>中描述的行为有一个例外。

# 以编程方式调整能力集
<span id='Programmatically_adjusting_capability_sets'>editorSelectionText</span><u>editorSelectionText</u>