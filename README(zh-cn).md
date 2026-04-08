# CSMScript-Lite

[English](./README.md) | [中文](./README(zh-cn).md)

[![GitHub all releases](https://img.shields.io/github/downloads/NEVSTOP-LAB/CSMScript-Lite/total)](https://github.com/NEVSTOP-LAB/CSMScript-Lite/releases)

CSMScript-Lite 是一款基于 [可通信状态机（CSM）](https://github.com/NEVSTOP-LAB/Communicable-State-Machine) 框架的轻量级脚本执行引擎，用于执行灵活的 CSM 测试脚本，实现自动化测试工作流。同时，它也是展示 CSM 框架能力的实践示例。其设计理念类似于 NI TestStand。

此项目包括：

- **CSMScript-Lite Library** — 轻量级 CSM 脚本执行引擎，其本身也是一个基于 CSM 实现的模块。
  - **[Engine](docs/Engine(CSM).md)**：核心执行引擎，负责解析并运行 CSM 脚本，管理测试状态与结果。
  - **[UI（ExecutionView）](docs/ExecutionView(CSM).md)**：用户界面，提供脚本管理、执行控制和结果查看功能。
  - **[App](docs/App.md)**：示例应用程序，展示如何使用 CSMScript 库执行脚本。
- **实例工程** — 展示如何将 CSMScript-Lite 与其他 CSM 模块结合，实现脚本驱动的自动化测试。

![CSMScriptApp](.github/CSMScript%20UI.png)

## 依赖

- LabVIEW 2020 及以上版本
- [Communicable State Machine Framework](https://github.com/topics/labview-csm)
  - [Communicable State Machine (CSM)](https://github.com/NEVSTOP-LAB/Communicable-State-Machine)
  - [CSM API String Arguments Support](https://github.com/NEVSTOP-LAB/CSM-API-String-Arguments-Support)
  - [CSM MassData Parameter Support](https://github.com/NEVSTOP-LAB/CSM-MassData-Parameter-Support)
  - [CSM INI Static Variable Support](https://github.com/NEVSTOP-LAB/CSM-INI-Static-Variable-Support)

## 功能说明

### 脚本执行

支持全部 CSM 命令，包括同步消息、异步消息、广播订阅等。完整语法请参考 [CSM 框架文档](https://github.com/NEVSTOP-LAB/Communicable-State-Machine)。

### 返回值捕获

通过 `=> 变量名` 语法，将指令的返回值保存到脚本临时变量空间中，供后续指令使用：

```c
message1 >> arguments -@ module1 => returnValueVar
message2 >> ${returnValueVar} -@ module2
```

> [!NOTE]
> `CSM - Run Script.vi` 中同样支持此语法，行为完全一致。

### 扩充指令

CSMScript 内置了一套超出标准 CSM 命令集的扩充指令，语法与 CSM 指令相同：`指令 >> 参数`，指令名称大小写不敏感。

| 类别 | 指令 | 说明 |
|---|---|---|
| 跳转 | `GOTO` | 跳转到指定的 `<anchor>` |
| 自动错误处理 | `AUTO_ERROR_HANDLE_ENABLE` | 开启或关闭自动错误处理 |
| 自动错误锚点 | `AUTO_ERROR_HANDLE_ANCHOR` | 设置错误跳转锚点（默认：`<cleanup>`） |
| 等待 | `WAIT`、`Sleep` | 等待指定时长，支持在一个表达式中混合使用 `min`、`s`、`ms` |
| 等待（秒） | `WAIT(s)`、`Sleep(s)` | 等待指定秒数，参数为浮点类型 |
| 等待（毫秒） | `WAIT(ms)`、`Sleep(ms)` | 等待指定毫秒数，参数为整数类型 |

示例：

```c
message1 >> arguments -@ csm
wait >> 1min 20s 500ms  // 等待 1 分 20 秒 500 毫秒

message1 >> arguments -@ csm
wait(ms) >> 100          // 等待 100 毫秒

message1 >> arguments -@ csm
wait(s) >> 1.5           // 等待 1.5 秒
```

### 锚点与跳转

脚本中可定义命名锚点，并通过错误跳转或显式 `GOTO` 跳转到指定锚点继续执行。

- **锚点定义**：`<anchor_name>`，例如 `<setup>`、`<main>`、`<error_handler>`、`<cleanup>`，大小写不敏感。
- **条件跳转**：`?? goto >> <anchor_name>`，在前一条指令出错时跳转到指定锚点。省略条件表达式等同于"任意错误时跳转"。

`AUTO_ERROR_HANDLE_ANCHOR` 指令设置默认的错误跳转锚点（默认为 `<cleanup>`）。
`AUTO_ERROR_HANDLE_ENABLE` 指令开启自动错误处理，任何指令执行失败时，自动跳转到预设锚点。

示例：

```c
// 开启自动错误处理
AUTO_ERROR_HANDLE_ENABLE >> TRUE
// 将默认错误锚点改为 error_handler
AUTO_ERROR_HANDLE_ANCHOR >> error_handler

<setup>  // ----- setup anchor ----

// 初始化失败不需要执行 stop，因此显式指定跳转到 cleanup
initialize >> daq1 -@ ai ?? goto >> <cleanup>

<main>  // ----- main anchor ----

// 之后的所有指令，执行失败时将跳转到 error_handler
configure >> Onboard Clock;10,-10,RSE -@ ai
start -@ ai
acquire >> Channel:ch0;Num:1000 -@ ai

<error_handler>  // ----- error handler anchor ----
stop -@ ai

<cleanup>  // ----- cleanup anchor ----
close -@ ai
```

> [!NOTE]
> 自动错误处理**默认关闭**，需通过 `AUTO_ERROR_HANDLE_ENABLE >> TRUE` 显式开启。未开启时，指令执行出错后将继续执行下一条指令。

> [!NOTE]
> 锚点最常见的用途是划分脚本执行阶段，类似于 NI TestStand 序列中的 `<setup>`、`<main>`、`<cleanup>` 等阶段。

> [!NOTE]
> 锚点名称中的 `<>` 符号可省略，`GOTO >> cleanup` 与 `GOTO >> <cleanup>` 效果相同。
