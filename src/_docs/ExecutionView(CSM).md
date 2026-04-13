# `ExecutionView(CSM)` — CSM 模块接口文档

## 功能简述

`ExecutionView(CSM)` 是 CSMScript-Lite 的 UI 显示模块，用于展示脚本执行的实时状态与结果。

该模块通过 `TS: Link to Engine` 与 `Engine(CSM)` 模块建立关联，自动订阅引擎广播的执行事件，并将步骤列表、执行进度、步骤结果等信息实时呈现在前面板上。

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

| 依赖                                                                                                | 类型                 |
| --------------------------------------------------------------------------------------------------- | -------------------- |
| [Communicable-State-Machine](https://github.com/NEVSTOP-LAB/Communicable-State-Machine)             | 必须                 |
| [CSM-API-String-Arguments-Support](https://github.com/NEVSTOP-LAB/CSM-API-String-Arguments-Support) | 必须                 |
| Engine(CSM)                                                                                         | 必须（作为关联引擎） |

---

## API 接口（消息接口）

以下是外部调用者可以发送给本模块的消息。

### `TS: Link to Engine`

将本 UI 模块关联到指定的 `Engine(CSM)` 实例。关联成功后，UI 模块自动订阅该引擎的执行广播事件，以更新显示状态。

- **参数**：`API String` — `String`：目标引擎模块的名称（如 `Engine`）
- **响应**：N/A

### `TS: Load Sequence`

将已加载的脚本序列的步骤信息更新到 UI 显示列表中，通常在引擎触发 `SequenceLoaded Event` 后由上层模块调用。

- **参数**：`API String` | `MassData` | `HexStr` — `String`：脚本文件的内容
- **响应**：N/A

### `UI: Front Panel State`

控制本模块前面板的显示状态。

- **参数**：`API String` — `Enum`：`Open`、`Close` 或 `Minimize`
- **响应**：N/A

### `UI: Cursor Set`

设置前面板光标样式。

- **参数**：`API String` — `Enum`：光标类型名称（如 `Busy`、`Default`）
- **响应**：N/A

### 参数类型说明

| 类型        | 说明                                                                            |
| ----------- | ------------------------------------------------------------------------------- |
| `HexStr`    | 将 LabVIEW Variant 序列化为十六进制字符串，内置支持                             |
| `SafeStr`   | 将特殊字符编码为 `%[HEXCODE]`，内置支持                                         |
| `ErrStr`    | 将错误信息编码为字符串，内置支持                                                |
| `APIString` | 支持嵌套键值对的纯文本字符串，需要 CSM API String Arguments Support 插件        |
| `MassData`  | 内存映射缓冲区，传递 `Start:N,Size:M`，需要 CSM MassData Parameter Support 插件 |
| 用户自定义  | 由模块自行解析的字符串，无需额外插件，但是要说明具体的解析规则和格式            |

> **注意**：接口文档中对 `String` 类型数据统一使用 `APIString` 标注（不直接写 `SafeStr`），因为 `SafeStr` 正是 `APIString` 针对 `String` 类型的内部实现。

---

## 内部处理接口

以下状态由本模块内部使用，或在与引擎关联后通过广播订阅自动触发，**不应**由外部调用者直接发送。

| 状态                        | 触发来源                         | 说明                   |
| --------------------------- | -------------------------------- | ---------------------- |
| `TS: Test Started Handler`  | `SequenceStarted Event@Engine`   | 序列开始时更新 UI 状态 |
| `TS: Step Start Handler`    | `StepStart Event@Engine`         | 步骤开始时高亮当前行   |
| `TS: Step Complete Handler` | `StepComplete Event@Engine`      | 步骤完成时显示结果     |
| `TS: Test Complete Handler` | `SequenceCompleted Event@Engine` | 序列完成时更新 UI 汇总 |

---

## 状态广播接口

本模块不主动向外发出业务状态广播。

标准 CSM 广播：

### `Error Occurred`

**默认广播类型**：`Status`

当 UI 模块发生未处理的错误时发出。

- **参数**：`ErrStr` — `Error Cluster`：错误信息

---

## 调用限制与注意事项

> [!IMPORTANT]
>
> - `TS: Load Sequence` 通常由上层模块（如 `App.vi`）在收到引擎 `SequenceLoaded Event` 广播后触发，无需手动调用步骤更新逻辑。

---

## 使用示例

### 启动并关联引擎

```csm
// 先启动 Engine 模块
// ...（在代码中使用 CSM - Async Start Group of CSMs.vi 启动引擎和 UI）

// 将 UI 关联到引擎
TS: Link to Engine >> Engine -@ ExecutionView

// 打开 UI 前面板
UI: Front Panel State >> Open -@ ExecutionView
```

### 加载脚本并更新视图

```csm
// 加载脚本文件到引擎
TS: Load Sequence >> C:\scripts\test.csmscript -@ Engine

// 加载完成后更新 UI 视图（通常由 App 模块在订阅事件后触发）
TS: Load Sequence -@ ExecutionView
```

---

## 备注

- UI 显示风格（步骤列表样式、列宽等）由 `Sequence-DefaultStyle.vi` 提供的默认样式初始化。

---

- _完整 CSM 语法参考：<https://github.com/NEVSTOP-LAB/Communicable-State-Machine/blob/main/.doc/Syntax.md>_
- _CSM Wiki：<https://nevstop-lab.github.io/CSM-Wiki/>_
