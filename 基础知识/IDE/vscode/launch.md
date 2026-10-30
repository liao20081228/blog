---
title: launch
tags: vscode
---

[TOC]
# 配置架构
```json
{
    // 使用 IntelliSense 了解相关属性。
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "compounds": [
        {
            "name": "组合名字"
            "configurations": ["configurations中的name的值"]
        }
    ],
    "inputs": [],
    "configurations": []
}
```


| 配置项         | 取值示例 | 作用                                                                              |
| :------------- | :------- | :-------------------------------------------------------------------------------- |
| version        | 0.2.0    | 版本信息                                                                          |
| compounds      | \[]      | 组合调试。如果你有前端和后端，<br/>可以在这里配置“一键同时启动并调试多个进程”。 |
| inputs         | \[]      | 输入变量                                                                          |
| configurations | \[]      | 启动配置项                                                                        |
# 配置列表
## 公共配置
- **name**：调试配置名称，显示在 VS Code 调试下拉列表中。
- **type**：调试器类型；`debugpy` 表示使用 VS Code Python Debugger。
- **request**：请求类型；`launch` 表示启动新进程并调试。
- **cwd**：程序运行时工作目录；`${fileDirname}` 表示当前文件所在目录。
- **program**：要调试的 Python 入口文件；`${file}` 表示当前打开文件。
- **args**：传给被调试 Python 程序的命令行参数；这里传文件名和版本号两个参数。
- **preLaunchTask**：调试启动前运行的 VS Code 任务；空表示不运行。
- **postDebugTask**：调试结束后运行的任务；空表示不运行。
- **env/environment**：额外环境变量对象；空表示不添加。
- **envFile**：从文件加载环境变量；这里从工作区 `.env` 加载。
- **stopOnEntry/stopAtEntry**：启动后是否立即暂停在入口；`false` 表示不自动暂停。。
- **presentation**：该配置在调试下拉列表中的展示方式。
  - **hidden**：是否隐藏；`false` 表示显示。
  - **group**：分组名；这里为 `pyhton`，拼写有误但不影响运行。
  - **order**：同组内排序；`1` 表示靠前。
- **windows**：Windows 平台专属覆盖配置；空对象表示无覆盖。
- **linux**：Linux 平台专属覆盖配置；空对象表示无覆盖。
- **osx**：macOS 平台专属覆盖配置；空对象表示无覆盖。
- **internalConsoleOptions**：内部调试控制台打开策略；`neverOpen` 表示不自动打开。
- **suppressMultipleSessionWarning**：是否抑制多调试会话警告；`true` 表示抑制。

## python
```json
{
    "name": "Python Debug",
    "type": "debugpy",
    "request": "launch",
    "cwd": "${fileDirname}",
    "program": "${file}",
    "console": "integratedTerminal",
    "args": ["12_OMUSERVICE_CHECK_GET_NET_TOSCA_NAME.txt",  "v100r900"],
    "pythonArgs": [],
    "python": "${command:python.interpreterPath}",
    "preLaunchTask": "",
    "postDebugTask": "",
    "showReturnValue": true,
    "redirectOutput": false,
    "env": {},
    "consoleName": "Python Debug Console",
    "autoReload": {
        "enable": true,
        "include": [],
        "exclude": []
    },
    "stopOnEntry": false,
    "pyramid": false,
    "justMyCode": true,
    "pathMappings": [],
    "sudo": false,
    "presentation": {
        "hidden": false,
        "group": "pyhton",
        "order": 1,
    },
    "windows": {},
    "linux": {},
    "osx": {},
    "autoStartBrowser": false,
    "clientOS": "windows",
    "debugAdapterPath": "",
    "debugServer": 4711,
    "django": false,
    "envFile": "${workspaceFolder}/.env",
    "gevent": false,
    "guiEventLoop": "matplotlib",
    "internalConsoleOptions": "neverOpen",
    "jinja": false,
    "logToFile": false,
    "module": "",
    "pathMappings": [],
    "purpose": [],
    "python": "${command:python.interpreterPath}",
    "serverReadyAction":{
        "action": "debugWithEdge",
        "pattern": "listening on port ([0-9]+)",
        "uriFormat": "http://localhost:%s",
        "webRoot": "${workspaceFolder}",
        "killOnServerStop": false,

    },
    "subProcess": false,
    "suppressMultipleSessionWarning": true,
    "terminalQuoteCharacter": "'"
}
```

- **name**：调试配置名称，显示在 VS Code 调试下拉列表中。
- **type**：调试器类型；`debugpy` 表示使用 VS Code Python Debugger。
- **request**：请求类型；`launch` 表示启动新进程并调试。
- **cwd**：程序运行时工作目录；`${fileDirname}` 表示当前文件所在目录。
- **program**：要调试的 Python 入口文件；`${file}` 表示当前打开文件。
- **console**：程序输入输出使用的控制台；`integratedTerminal` 表示 VS Code 集成终端。
- **args**：传给被调试 Python 程序的命令行参数；这里传文件名和版本号两个参数。
- **pythonArgs**：传给 Python 解释器本身的参数；空数组表示不额外传。
- **python**：指定 Python 解释器；`${command:python.interpreterPath}` 解析为当前选择的解释器。
- **preLaunchTask**：调试启动前运行的 VS Code 任务；空表示不运行。
- **postDebugTask**：调试结束后运行的任务；空表示不运行。
- **showReturnValue**：是否显示函数返回值；`true` 表示显示。
- **redirectOutput**：是否把输出重定向到调试控制台；`false` 表示通常仍输出到集成终端。
- **env**：额外环境变量对象；空表示不添加。
- **consoleName**：集成终端名称；这里为 `Python Debug Console`。
- **autoReload**：自动重载相关配置。
  - **enable**：是否启用自动重载；`true` 表示启用。
  - **include**：自动重载包含的文件/模式；空数组表示未额外指定。
  - **exclude**：自动重载排除的文件/模式；空数组表示未额外指定。
- **stopOnEntry**：启动后是否立即暂停在入口；`false` 表示不自动暂停。
- **pyramid**：是否启用 Pyramid 框架调试支持；`false` 表示不启用。
- **justMyCode**：是否只调试用户代码、跳过库内部；`true` 表示只调试自己的代码。
- **pathMappings**：本地路径与远程/容器路径映射；空数组表示不做映射。
- **sudo**：是否以管理员/`sudo` 权限运行；`false` 表示否。
- **presentation**：该配置在调试下拉列表中的展示方式。
  - **hidden**：是否隐藏；`false` 表示显示。
  - **group**：分组名；这里为 `pyhton`，拼写有误但不影响运行。
  - **order**：同组内排序；`1` 表示靠前。
- **windows**：Windows 平台专属覆盖配置；空对象表示无覆盖。
- **linux**：Linux 平台专属覆盖配置；空对象表示无覆盖。
- **osx**：macOS 平台专属覆盖配置；空对象表示无覆盖。
- **autoStartBrowser**：是否自动启动浏览器；`false` 表示不自动启动。
- **clientOS**：客户端操作系统标识；`windows` 表示按 Windows 客户端处理。
- **debugAdapterPath**：自定义调试适配器路径；空表示使用默认 debugpy 适配器。
- **debugServer**：调试服务器端口；这里为 `4711`，普通本地 `launch` 通常不一定需要。
- **django**：是否启用 Django 调试支持；`false` 表示不启用。
- **envFile**：从文件加载环境变量；这里从工作区 `.env` 加载。
- **gevent**：是否启用 gevent 支持；`false` 表示不启用。
- **guiEventLoop**：GUI 事件循环集成类型；`matplotlib` 表示针对 matplotlib 交互窗口处理事件循环。
- **internalConsoleOptions**：内部调试控制台打开策略；`neverOpen` 表示不自动打开。
- **jinja**：是否启用 Jinja 模板调试；`false` 表示不启用。
- **logToFile**：是否把调试器日志写入文件；`false` 表示不写。
- **module**：是否以 `python -m 模块名` 方式运行；空表示不使用，改用 `program` 文件。
- **purpose**：调试配置用途标记；空数组表示无特殊用途。
- **serverReadyAction**：服务器就绪后自动执行动作。
  - **action**：动作类型；`debugWithEdge` 表示用 Edge 打开并调试。
  - **pattern**：从输出中匹配端口的正则；如 `listening on port ([0-9]+)`。
  - **uriFormat**：用匹配端口生成 URL；如 `http://localhost:%s`。
  - **webRoot**：Web 根目录；这里为工作区根目录。
  - **killOnServerStop**：服务器停止时是否关闭浏览器；`false` 表示不关闭。
- **subProcess**：是否自动调试子进程；`false` 表示否。
- **suppressMultipleSessionWarning**：是否抑制多调试会话警告；`true` 表示抑制。
- **terminalQuoteCharacter**：终端中给路径/参数加引号使用的字符；这里为单引号 `'`。




# cppdbg

# gdb
# lldb