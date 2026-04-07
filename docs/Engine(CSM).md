# `Engine(CSM)` — CSM 模块接口文档

## 功能简述

`Engine(CSM)` 是 CSMScript-Lite 的核心脚本执行引擎模块，负责解析并运行 `.csmscript` 脚本文件，管理测试流程状态与结果。

引擎采用多线程架构，支持脚本中的同步/异步 CSM 消息、广播订阅、锚点跳转与自动错误处理，并在每个关键执行节点通过广播通知 UI 模块。

---

## 模块信息

| 属性           | 值                           |
| -------------- | ---------------------------- |
| LabVIEW 版本   | ≥ 2020                       |
| 支持的操作系统 | Windows                      |
| 支持 RT        | ❌ 不支持                    |
| 支持 64-bit    | ✅ 支持                      |
| 所属模块组     | CSMScript-Engine(Lite).lvlib |

---

## 依赖项

| 依赖                                                                                                | 类型 |
| --------------------------------------------------------------------------------------------------- | ---- |
| [Communicable-State-Machine](https://github.com/NEVSTOP-LAB/Communicable-State-Machine)             | 必须 |
| [CSM-API-String-Arguments-Support](https://github.com/NEVSTOP-LAB/CSM-API-String-Arguments-Support) | 必须 |
| [CSM-MassData-Parameter-Support](https://github.com/NEVSTOP-LAB/CSM-MassData-Parameter-Support)     | 必须 |
| [CSM-INI-Static-Variable-Support](https://github.com/NEVSTOP-LAB/CSM-INI-Static-Variable-Support)   | 必须 |

---

## API 接口（消息接口）

以下是外部调用者可以发送给本模块的消息。

### `TS: Load Sequence`

加载指定的 `.csmscript` 脚本文件并解析为内部序列数据。加载完成后会广播 `SequenceLoaded Event`。

- **参数**：用户自定义 — `String`：脚本文件的绝对路径（如 `C:\scripts\test.csmscript`）
- **响应**：N/A

### `TS: Unload Sequence`

卸载当前已加载的脚本序列，释放相关内部资源。卸载完成后会广播 `SequenceUnloaded Event`。

- **参数**：N/A
- **响应**：N/A

### `TS: SinglePass`

以单次遍历模式执行已加载的脚本序列。执行期间依次广播 `SequenceStarted Event`、`StepStart Event`、`StepComplete Event` 及 `SequenceCompleted Event`。

- **参数**：N/A
- **响应**：N/A

### `TS: Terminate`

立即终止正在执行的脚本序列。

- **参数**：N/A
- **响应**：N/A

---

## 状态广播接口

以下是本模块**发出**的消息，用于通知订阅者内部状态变化。

### `SequenceLoaded Event`

**广播类型**：`Status`

当 `TS: Load Sequence` 成功加载脚本文件后发出。

- **参数**：N/A

### `SequenceUnloaded Event`

**广播类型**：`Status`

当 `TS: Unload Sequence` 成功卸载序列后发出。

- **参数**：N/A

### `SequenceStarted Event`

**广播类型**：`Status`

当脚本序列开始执行时发出。

- **参数**：N/A

### `StepStart Event`

**广播类型**：`Status`

当脚本中某一步骤开始执行时发出。

- **参数**：`HexStr` — `Def-Thread Data.ctl`：当前步骤的线程数据

### `StepComplete Event`

**广播类型**：`Status`

当脚本中某一步骤执行完成时发出。

- **参数**：`HexStr` — `Def-StepComplete Event Data.ctl`：步骤完成的状态数据（含结果与错误信息）

### `SequenceCompleted Event`

**广播类型**：`Status`

当脚本序列全部执行完毕时发出。

- **参数**：N/A

### `Error Occurred`

**广播类型**：`Status`

当引擎发生未处理的错误时发出（标准 CSM 错误广播）。

- **参数**：`HexStr` — `Error Cluster`：错误信息

---

## 调用限制与注意事项

- [ ] 必须先调用 `TS: Load Sequence` 成功加载脚本后，才能调用 `TS: SinglePass`。
- [ ] 本模块为**单例**——同一时间不可运行多个实例。
- [ ] 使用 `TS: Terminate` 可以在执行过程中强制终止序列，但不会等待当前正在执行的 CSM 消息返回。

---

## 使用示例

### 基本生命周期

```text
// 加载脚本文件
TS: Load Sequence >> C:\scripts\test.csmscript -> Engine

// 订阅加载完成事件后执行
SequenceLoaded Event@Engine >> TS: SinglePass@Engine -><register>

// 执行完毕后卸载
SequenceCompleted Event@Engine >> TS: Unload Sequence@Engine -><register>
```

### 订阅步骤状态广播

```text
// 将 Engine 的步骤完成事件路由到 UI 模块处理
StepComplete Event@Engine >> TS: Step Complete Handler@ExecutionView -><register>
StepStart Event@Engine >> TS: Step Start Handler@ExecutionView -><register>
```

---

## 备注

- 脚本语法说明请参阅 [README](../README.md) 或 [CSM 语法文档](https://github.com/NEVSTOP-LAB/Communicable-State-Machine/blob/main/.doc/Syntax.md)。
- 扩充指令（如 `WAIT`、`GOTO`、`AUTO_ERROR_HANDLE_ENABLE`）由引擎内部解析，无需额外依赖。

---

_完整 CSM 语法参考：<https://github.com/NEVSTOP-LAB/Communicable-State-Machine/blob/main/.doc/Syntax.md>_
_CSM Wiki：<https://nevstop-lab.github.io/CSM-Wiki/>_
