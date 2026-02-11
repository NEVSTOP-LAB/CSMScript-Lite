# CSMStand-Lite

## 简介

CSMStand-Lite 是一款基于 Communicable State Machine (CSM) 框架的脚本执行引擎，能够运行灵活的 CSM 测试脚本，实现自动化测试。同时，它也用于展示 CSM 框架的功能。其设计类似于 NI TestStand，这也是其名称的来源。

此项目包括：

1. **CSMStand-Lite Library**：一个轻量级的 CSM 脚本执行引擎，它本身也是基于 CSM 实现的模块。
    - Engine：核心执行引擎，负责解析和执行 CSM 脚本，管理测试状态和结果。
    - UI：用户界面，提供脚本管理、执行控制和结果查看功能。
    - App：一个示例应用程序，展示如何使用 CSMStand 库进行脚本执行。
3. **实例工程**：展示如何使用 CSMStand-Lite 配合其他 CSM 模块，实现脚本执行和自动化测试。

![CSMStandApp](.github/csmstand%20ui.png)

## 依赖关系

- LabVIEW 2020 及以上版本
- NEVSTOP-LAB Palette
- Communicable State Machine (CSM) Framework
  - Communicable State Machine(CSM)
  - CSM API String Arguments Support
  - CSM MassData Parameter Support
  - CSM INI Static Variable Support
- NEVSTOP-Programming-Palette

## CSMStand-Lite Library 功能说明

### 基础功能：脚本执行

支持 CSM 的全部命令，包括同步消息、异步消息、广播订阅等功能。详细的信息，请参考 CSM 框架的文档。

### 返回值保存传递 (Return Value)

通过 `=> 变量名` 语法，将指令的返回值保存到脚本的临时变量空间中，供后续指令使用。例如：

```c
message1 >> arguments -@ module1 => returnValueVar
message2 >> ${returnValueVar} -@ module2
```

> [!NOTE]
> CSM - Run Script.vi 中也支持此功能，此处的功能与之相同。

### 扩充指令 (Extended Commands)

CSMStand 内置了一些非 CSM 提供的扩充指令，用于实现基本功能，如跳转指令等。

格式与 CSM 指令类似：`指令 >> 参数`，指令名称大小写不敏感。

| 指令集名称            | 指令                     | 说明                                                         |
| --------------------- | ------------------------ | ------------------------------------------------------------ |
| 跳转指令              | GOTO                     | 跳转到指定的 `<anchor>`                                      |
| 开启/关闭自动错误处理 | AUTO_ERROR_HANDLE_ENABLE | 开启或关闭自动错误处理功能                                   |
| 设置自动错误跳转锚点  | AUTO_ERROR_HANDLE_ANCHOR | 设置自动错误跳转的锚点，默认为 `<cleanup>`                   |
| 等待指令1             | WAIT, Sleep              | 等待指定的时间，支持 minute(min)、second(s)、microsecond(ms) |
| 等待指令2             | WAIT(s), Sleep(s)        | 等待指定的时间，单位为秒，参数为 Float 类型                  |
| 等待指令3             | WAIT(ms), Sleep(ms)      | 等待指定的时间，单位为毫秒，参数为 NUMERIC 类型              |

举例：

```c
message1 >> arguments -@ csm
wait >> 1min 20s 500ms  // 等待 1 分 20 秒 500 毫秒

message1 >> arguments -@ csm
wait(ms) >> 100  // 等待 100 毫秒

message1 >> arguments -@ csm
wait(s) >> 1.5  // 等待 1.5 秒
```

### 锚点跳转 (Anchor)

支持在脚本中定义锚点，并通过错误跳转或变量逻辑判断跳转到指定锚点继续执行脚本。

- **锚点定义格式**：`<anchor_name>`，例如 `<setup>`、`<main>`、`<error_handler>`、`<cleanup>` 等，大小写不敏感。
- **锚点跳转格式**：`?? goto >> <anchor_name>` 语法，在指令执行错误时跳转到指定锚点继续执行脚本。缺省 expression 表示指令执行错误时跳转。

`AUTO_ERROR_HANDLE_ANCHOR` 指令可以设置自动错误跳转的锚点，缺省为 `<cleanup>`。

`AUTO_ERROR_HANDLE_ENABLE` 指令可以开启自动错误处理功能，当指令执行错误时，自动跳转到预设的锚点继续执行脚本。

例如：

```c
// 开启自动错误处理
AUTO_ERROR_HANDLE_ENABLE >> TRUE
// 设置自动错误跳转锚点为 error_handler，不再使用 cleanup 作为默认锚点
AUTO_ERROR_HANDLE_ANCHOR >> error_handler

<setup>  // ----- setup anchor ----

// 逻辑上，初始化失败，无需执行 stop 指令，
// 所以这里显式添加了错误时跳转到 cleanup 锚点，
// 当此指令执行失败时，将跳转到 cleanup 锚点
initialize >> daq1 -@ ai ?? goto >> <cleanup>

<main>  // ----- main anchor ----

// 之后的所有指令，如果执行失败，都将跳转到 error_handler 锚点
configure >> Onboard Clock;10,-10,RSE -@ ai

start -@ ai

acquire >> Channel:ch0;Num:1000 -@ ai

<error_handler>  // ----- error handler anchor ----
stop -@ ai

<cleanup> //----- cleanup anchor ----
close -@ ai

```

> [!NOTE]
> 默认的情况下，自动错误处理的是不打开的，需要通过 `AUTO_ERROR_HANDLE_ENABLE` 指令开启。
> 此时当发生错误时，将继续执行之后的指令。

> [!NOTE]
> 最常见的锚点主要用于定义脚本运行阶段，例如可以定义类似 TestStand 序列的 `<setup>`, `<main>`, `<cleanup>` 等锚点。

> [!NOTE]
> GOTO 指令和 AUTO_ERROR_HANDLE_ANCHOR 指令参数中可以携带 <> 符号，也可以不携带，例如 `GOTO >> cleanup` 和 `GOTO >> <cleanup>` 效果相同。
