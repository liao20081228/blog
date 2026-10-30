---
title: launch
tags: vscode
category: /vscode
---

# pyhton
这份 `launch.json` 配置极其详尽，包含了 VS Code Python 调试器（`debugpy`）几乎所有可用的参数。其中很多是**默认生成的冗余配置**，或者针对特定框架（如 Django）的配置。

作为 C 程序员，你可以把它当成一个**“调试器结构体（Struct）的初始化列表”**。我将其按功能划分为 6 大类，为你逐一拆解：

---

### 一、 核心基础配置（最常用，决定怎么启动）

| 配置项 | 你的值 | 作用（C 程序员类比） |
| :--- | :--- | :--- |
| `name` | `"Python Debug"` | 调试配置的名称，会显示在 VS Code 调试下拉菜单里。 |
| `type` | `"debugpy"` | 指定使用的调试器类型（新版 Python 插件统一叫 `debugpy`）。 |
| `request` | `"launch"` | **`launch`** 表示启动并调试一个新进程；`attach` 表示附加到已经运行的程序上（类似 C 里的 `attach PID`）。 |
| `program` | `"${file}"` | 要运行的入口文件。`${file}` 表示当前编辑器里激活的那个文件。 |
| `cwd` | `"${fileDirname}"` | **当前工作目录**。极其重要！它决定了相对路径（比如你传的 `12_OMU...txt`）去哪里找。`${fileDirname}` 表示以当前脚本所在目录为准。 |
| `args` | `["12_OMU...", "v100r900"]` | 传给脚本的命令行参数。等同于 C 语言的 `argv[1]` 和 `argv[2]`。 |
| `console` | `"integratedTerminal"` | 输出和控制台放在哪。`integratedTerminal` 表示用 VS Code 底部的终端，支持输入交互。 |
| `python` | `"${command:python.interpreterPath}"` | 动态获取当前右下角选中的 Python 解释器路径（保证环境一致性）。 |

---

### 二、 调试行为控制（断点、步进与输出）

| 配置项 | 你的值 | 作用 |
| :--- | :--- | :--- |
| `stopOnEntry` | `false` | 设为 `true` 时，启动后会在第一行代码自动暂停（类似于在 `main` 函数第一行打断点）。 |
| `justMyCode` | `true` | **核心选项**。设为 `true` 时，按 `F11` 单步进入只会进你自己的代码，**直接跳过标准库和第三方库**（类似 C 里不步入 `libc` 源码）。 |
| `showReturnValue` | `true` | 在调试器的“变量”面板中，显示函数返回后的返回值。 |
| `redirectOutput` | `false` | 是否将程序的标准输出重定向到调试器内部。设为 `false` 表示直接输出到终端。 |
| `logToFile` | `false` | 若调试器本身出 Bug 闪退，设为 `true` 会生成 `debugpy` 的底层通信日志。 |
| `autoReload` | `enable: true` | 热重载功能。当你的代码文件被修改并保存时，调试器会自动重启。 |

---

### 三、 环境变量与生命周期钩子

| 配置项 | 你的值 | 作用 |
| :--- | :--- | :--- |
| `env` | `{}` | 在启动前注入的环境变量，格式为 `{"KEY": "VALUE"}`（类似 C 里的 `setenv`）。 |
| `envFile` | `"${workspaceFolder}/.env"` | 从 `.env` 文件中读取环境变量（通常用于存放密码、API Key）。 |
| `preLaunchTask` | `""` | 在启动调试**之前**自动执行的任务（比如先跑一遍编译、代码格式化）。 |
| `postDebugTask` | `""` | 在调试**结束/终止后**自动执行的任务（比如清理临时文件）。 |

---

### 四、 UI 展示与控制台设置

| 配置项 | 你的值 | 作用 |
| :--- | :--- | :--- |
| `consoleName` | `"Python Debug Console"` | 终端标签页的显示名称。 |
| `internalConsoleOptions` | `"neverOpen"` | 不自动打开内部的“调试控制台”，因为前面配置了使用 `integratedTerminal`。 |
| `presentation` | `{ hidden: false, group: "pyhton", order: 1 }` | 控制调试配置下拉菜单的显示顺序和分组。（注意你这里有个拼写错误 `pyhton`，应为 `python`，但不影响运行）。 |
| `terminalQuoteCharacter` | `"'"` | 在终端里拼接命令行时使用的引号类型。 |

---

### 五、 特定框架与库专用（这里全关了）

这些配置是给特定 Web 框架或异步库准备的，你这里都是 `false` 或空：

| 配置项 | 你的值 | 作用 |
| :--- | :--- | :--- |
| `django` | `false` | 如果调试 Django 项目，设为 `true` 会自动处理模板断点和路由。 |
| `jinja` | `false` | 支持 Jinja2 模板引擎的断点调试。 |
| `gevent` | `false` | 针对协程库 `gevent` 的猴子补丁（Monkey Patch）调试支持。 |
| `pyramid` | `false` | 针对 Pyramid Web 框架的支持。 |
| `guiEventLoop` | `"matplotlib"` | 如果你用 Matplotlib 画图，调试时它需要接管事件循环才能显示窗口。 |
| `module` | `""` | 如果你想以模块方式运行（如 `python -m pytest`），填模块名（此时不能填 `program`）。 |

---

### 六、 高级网络与远程附加

| 配置项 | 你的值 | 作用 |
| :--- | :--- | :--- |
| `debugServer` | `4711` | 调试器底层的监听端口（通常用于远程调试）。 |
| `pathMappings` | `[]` | 本地代码路径与远程服务器代码路径的映射（远程调试必填，类似 GDB 的 `set substitute-path`）。 |
| `subProcess` | `false` | 是否调试由主进程启动的子进程。 |
| `sudo` | `false` | 是否以 Root/管理员权限启动调试（在 Linux 下常用）。 |
| `serverReadyAction` | `{...}` | 当你的程序在终端打印出特定模式（比如 `listening on port 8080`）时，自动帮你启动 Edge 浏览器并打开对应网址。 |
| `compounds` | `[]` | 组合调试。如果你有前端和后端，可以在这里配置“一键同时启动并调试多个进程”。 |

---

### 💡 给你的“瘦身”建议

这份配置有 40 多行，但对于日常 Python 脚本调试，**90% 都是不需要的**。而且里面有一些隐性的冲突（比如 `pathMappings` 出现了两次，其中一次被注释了）。

如果你想让配置清爽一点，**其实只需要保留下面这几行，就完全足够你现在的调试需求了**：

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Python Debug",
            "type": "debugpy",
            "request": "launch",
            "program": "${file}",
            "cwd": "${fileDirname}",
            "console": "integratedTerminal",
            "args": ["12_OMUSERVICE_CHECK_GET_NET_TOSCA_NAME.txt", "v100r900"],
            "python": "${command:python.interpreterPath}",
            "justMyCode": true
        }
    ]
}
```
其他那些配置，等你有特殊需求（比如调试 Django、远程调试、或者需要热重载）时，再通过点击调试面板的齿轮图标逐个加上去即可。