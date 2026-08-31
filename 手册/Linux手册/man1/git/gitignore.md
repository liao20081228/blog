---
title: gitignore
tags: 一般命令,git,man1
---

------

***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[git manpages](https://git-scm.com/docs/gitignore)》。</font>ssh的版本为2.55，手册更新时间为2026-6。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***

------



## 名称

gitignore — 指定需要故意不予追踪、应当被 Git 忽略的文件

## 概要

`$XDG_CONFIG_HOME/git/ignore`、`$GIT_COMMON_DIR/info/exclude`、`.gitignore`

## 描述

gitignore 文件用于指定 Git 需要故意忽略的未追踪文件。**已经被 Git 追踪的文件不受该文件影响**；详细说明参见下文【备注】章节。

gitignore 文件的每一行书写一条匹配模式。当 Git 判断是否忽略某一路径时，会从多处来源读取忽略规则，优先级从高到低如下（同一优先级内，**最后一条匹配的模式决定最终结果**）：

1.  支持该参数的命令，从命令行传入的匹配模式。
2.  路径所在目录，或任意上层目录（直至工作树根目录）内的 `.gitignore` 文件；下层目录的规则会覆盖上层目录的规则。模式的匹配相对于该 `.gitignore` 文件自身所在位置。项目仓库内通常会存放这类 `.gitignore`，用来描述项目构建过程生成的各类文件。
3.  读取 `$GIT_COMMON_DIR/info/exclude` 文件内的模式。
4.  读取配置项 `core.excludesFile` 所指定文件内的模式。

应当把规则写在哪一个文件，取决于这条规则的使用场景：

*   需要纳入版本控制、可随仓库克隆分发（所有开发者都希望忽略的文件）：写入仓库内的 `.gitignore`。
*   仅针对当前仓库，不需要共享给其他关联仓库（例如存放在仓库内，但属于某用户个人工作流的辅助文件）：写入 `$GIT_COMMON_DIR/info/exclude`。
*   用户希望在全部 Git 仓库全局生效（例如编辑器生成的备份、临时文件）：写入用户 `~/.gitconfig` 中 `core.excludesFile` 指定的文件。默认路径为 `$XDG_CONFIG_HOME/git/ignore`；若未设置或为空 `$XDG_CONFIG_HOME`，则回退使用 `$HOME/.config/git/ignore`。

Git 底层工具（如 `git ls‑files`、`git read‑tree`）读取命令行选项给出的忽略规则；高层命令（如 `git status`、`git add`）会使用上面全部来源的忽略规则。

## 模式格式

*   空行：不匹配任何文件，仅作为分隔，提升可读性。
*   以 `#` 开头的行代表注释。如果模式本身需要以字面 `#` 开头，需要在 `#` 前加反斜杠 `\` 转义。
*   行尾空格默认被忽略；如果要保留尾部空格，需要用反斜杠 `\` 转义。
*   前缀 `!`：取反模式。被前面规则排除的匹配文件，会被重新纳入版本管理。**如果某文件的父目录已经被排除，则该文件无法通过取反重新纳入**。出于性能考量，Git 不会枚举被排除目录内部的内容，因此目录被忽略后，内部文件的任何规则都不再生效。 如果模式本身字面以 `!` 开头，需要写反斜杠转义，例如 `\!important!.txt`。
*   斜杠 `/` 用作目录分隔符，可以出现在模式开头、中间或末尾。
    *   模式开头/中间包含 `/`：该模式的匹配基准是该 `.gitignore` 文件所在目录。
    *   模式不含 `/`：可在 `.gitignore` 的任意下级目录进行匹配。
*   工作树外部的排除规则（`$GIT_COMMON_DIR/info/exclude`、`core.excludesFile`），匹配基准视为**工作树根目录**；即模式开头的 `/` 代表仓库根目录锚定。
*   模式末尾带 `/`：仅匹配目录；不带末尾斜杠，文件、目录均可匹配。

> 
> 
> 示例： `doc/frotz/` 只匹配 `doc/frotz` 目录，不会匹配 `a/doc/frotz`； `frotz/` 可以匹配 `frotz`、`a/frotz` 目录（全部路径均相对于 `.gitignore` 文件位置）。
> 
> 

*   星号 `*`：匹配除斜杠 `/` 以外任意字符。
*   问号 `?`：匹配除 `/` 以外**单个**任意字符。
*   范围写法 `[a‑zA‑Z]`：匹配括号范围内任意一个字符。

    > 
    > 
    > 底层行为参考 `fnmatch(3)` 与 `FNM_PATHNAME` 标志。
    > 
    > 

*   反斜杠 `\` 用于转义字符。例如 `\*` 匹配字面星号；模式末尾的反斜杠属于非法模式，永远不会匹配任何内容。

模式中的连续双星号 `**` 在完整路径匹配时有特殊语义：

1.  `**/` 开头：匹配所有目录层级。例如 `**/foo` 在任意层级匹配名为 `foo` 的文件/目录，等价于 `foo`；`**/foo/bar` 匹配任意 `foo` 的直接子项 `bar`。
2.  `/**` 结尾：匹配目录内部全部内容。例如 `abc/**`，相对于 `.gitignore`，递归匹配 `abc` 目录下无限深度所有内容。
3.  `/**/`：匹配零层或多层中间目录。例如 `a/**/b` 匹配 `a/b`、`a/x/b`、`a/x/y/b` 等。
4.  其他连续星号，按普通星号规则处理。

## 配置

配置项 `core.excludesFile` 指定一个外部文件路径，存放忽略规则，作用类似 `$GIT_COMMON_DIR/info/exclude`。该文件内规则会叠加在 `$GIT_COMMON_DIR/info/exclude` 规则之上共同生效。

## 备注

gitignore 的目的：保证**未被追踪的文件持续处于未追踪状态**。

> 
> 
> 如果某文件已经被追踪，想要停止追踪：使用 `git rm --cached` 将该文件从索引中移除，之后再把文件名写入 `.gitignore`，避免后续提交又把该文件加回来。
> 
> 

Git 读取工作树中的 `.gitignore` 文件时，不会跟随符号链接。保证从文件系统读取、从索引/树对象读取时行为保持一致。

## 示例

1.  `hello.*`：匹配名称以 `hello.` 开头的任意文件/目录。 若只想限定当前目录，不匹配子目录，前面加斜杠：`/hello.*`。会匹配 `hello.txt`、`hello.c`；**不会匹配**`a/hello.java`。
2.  `foo/`：匹配 `foo` 目录以及该目录下全部内容；**不会匹配**普通文件或符号链接 `foo`。该行为与 Git 的路径规范（pathspec）保持一致。
3.  `doc/frotz` 和 `/doc/frotz` 在同一个 `.gitignore` 效果完全相同。模式中间已经存在斜杠时，开头的斜杠无意义。
4.  `foo/*`：匹配 `foo/test.json`（普通文件）、`foo/bar`（目录）；**不匹配**`foo/bar/hello.c`，因为 `*` 不能匹配斜杠。

```
$ git status
[...]
# Untracked files:
[...]
#       Documentation/foo.html
#       Documentation/gitignore.html
#       file.o
#       lib.a
#       src/internal.o
[...]

$ cat .git/info/exclude
# ignore objects and archives, anywhere in the tree.
*.[oa]

$ cat Documentation/.gitignore
# ignore generated html files,
*.html
# except foo.html which is maintained by hand
!foo.html

$ git status
[...]
# Untracked files:
[...]
#       Documentation/foo.html
[...]

```

另一个示例：

```
$ cat .gitignore
vmlinux*

$ ls arch/foo/kernel/vm*
arch/foo/kernel/vmlinux.lds.S

$ echo '!/vmlinux*' >arch/foo/kernel/.gitignore

```

第二层 `.gitignore` 阻止 Git 忽略 `arch/foo/kernel/vmlinux.lds.S`。

示例：排除全部内容，只保留目录 `foo/bar`（注意 `/*`；如果去掉斜杠，通配符会把 `foo/bar` 内部全部内容也排除）

```
$ cat .gitignore
# exclude everything except directory foo/bar
/*
!/foo
/foo/*
!/foo/bar

```

## 参见

`git‑rm`、git 仓库布局、`git‑check‑ignore`

## GIT

本手册属于 Git 工具套件文档。



------
***<font color=blue>版权声明：</font>本文翻译自<font color=blue>《[git manpages](https://git-scm.com/docs/gitignore)》。</font>ssh的版本为2.55，手册更新时间为2026-6。<font color=red>本文与原始文档采用相同的版权许可。</font><font color=blue>转载请注明出处！！！</font>***


------