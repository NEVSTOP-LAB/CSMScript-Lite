# `App` — CSM 模块接口文档

## 功能简述

`App` 是 CSMScript-Lite 的顶层应用模块，将 `Engine(CSM)` 和 `ExecutionView(CSM)` 整合为一个完整的脚本执行应用程序。

该模块负责协调引擎与 UI 的启动顺序、关联与生命周期管理，并对外暴露简洁的脚本文件加载与设置 API，可作为将 CSMScript-Lite 嵌入更大 CSM 系统的参考实现。

---

## 模块信息

| 属性 | 值 |
| --- | --- |
| LabVIEW 版本 | ≥ 2020 |
| 支持的操作系统 | Windows |
| 支持 RT | ❌ 不支持 |
| 支持 64-bit | ✅ 支持 |
| 所属模块组 | CSMScript-Engine(Lite).lvlib |

---

## 依赖项

| 依赖 | 类型 |
| --- | --- |
| [Communicable-State-Machine](https://github.com/NEVSTOP-LAB/Communicable-State-Machine) | 必须 |
| [CSM-API-String-Arguments-Support](https://github.com/NEVSTOP-LAB/CSM-API-String-Arguments-Support) | 必须 |
| [CSM-MassData-Parameter-Support](https://github.com/NEVSTOP-LAB/CSM-MassData-Parameter-Support) | 必须 |
| [CSM-INI-Static-Variable-Support](https://github.com/NEVSTOP-LAB/CSM-INI-Static-Variable-Support) | 必须 |
| Engine(CSM) | 必须 |
| ExecutionView(CSM) | 必须 |

---

## API 接口（消息接口）

以下是外部调用者可以发送给本模块的消息。

### `API: Load Script File`

加载指定的 `.csmscript` 脚本文件。内部将先调用 `TS: Load Sequence` 加载到引擎，并在加载完成后更新 ExecutionView 的步骤列表。

- **参数**：用户自定义 — `String`：脚本文件的绝对路径（如 `C:\scripts\test.csmscript`）
- **响应**：N/A

### `API: Unload Sequence File`

卸载当前已加载的序列文件，清空引擎内容并重置 UI 显示。

- **参数**：N/A
- **响应**：N/A

### `API: Information`

显示关于本应用的版本与信息对话框。

- **参数**：N/A
- **响应**：N/A

### `API: Settings`

打开设置对话框，允许用户修改应用配置。

- **参数**：N/A
- **响应**：N/A

### `UI: Front Panel State`

控制本模块（主窗口）前面板的显示状态。

- **参数**：用户自定义 — `String`：`Open`、`Close` 或 `Minimize`
- **响应**：N/A

---

## 内部操作接口

以下状态由模块内部使用，**不应**由外部调用者直接发送。

| 状态 | 说明 |
| --- | --- |
| `UI: Insert CSMScript OI` | 将 ExecutionView 嵌入到主应用前面板 |
| `UI: Start CSMScript Engine Behind` | 在后台启动 Engine 和 ExecutionView 并完成关联 |
| `UI: Show Version Dialog` | 在内部触发显示版本对话框 |
| `UI: Update Title` | 更新主窗口标题（通常在加载/卸载脚本后触发） |

---

## 状态广播接口

本模块不主动向外发出业务状态广播。

标准 CSM 广播：

### `Error Occurred`

**广播类型**：`Status`

当应用模块发生未处理的错误时发出。

- **参数**：`HexStr` — `Error Cluster`：错误信息

---

## 调用限制与注意事项

- [ ] 本模块为**单例**——同一时间不可运行多个实例。
- [ ] 本模块在 `Macro: Initialize` 阶段会自动启动内部的 `Engine(CSM)` 和 `ExecutionView(CSM)` 子模块，无需外部手动启动这两个模块。
- [ ] `API: Load Script File` 与 `API: Unload Sequence File` 为互斥操作，应避免并发调用。

---

## 使用示例

### 基本生命周期

```text
// 打开主窗口
UI: Front Panel State >> Open -> App

// 加载脚本文件
API: Load Script File >> C:\scripts\test.csmscript -> App

// 卸载脚本
API: Unload Sequence File -> App

// 查看版本信息
API: Information -> App
```

### 嵌入到更大系统中

```text
// 在系统初始化时启动 App 模块（通过 CSM - Async Start Group of CSMs.vi 启动）
// 然后直接发送 API 消息控制脚本加载与执行：
API: Load Script File >> ${scriptPath} -> App
```

---

## 备注

- `App.vi` 是 CSMScript-Lite 的示例应用，可参考其实现了解如何将 `Engine(CSM)` 和 `ExecutionView(CSM)` 整合进自己的应用程序。
- 如需自定义界面或集成到现有系统，建议直接使用 `Engine(CSM)` 和 `ExecutionView(CSM)` 模块，而非直接复用 `App.vi`。

---

*完整 CSM 语法参考：<https://github.com/NEVSTOP-LAB/Communicable-State-Machine/blob/main/.doc/Syntax.md>*  
*CSM Wiki：<https://nevstop-lab.github.io/CSM-Wiki/>*
