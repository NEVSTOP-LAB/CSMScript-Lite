# CSMScript-Lite

[English](./README(us-en).md) | [中文](./README.md)

[![GitHub all releases](https://img.shields.io/github/downloads/NEVSTOP-LAB/CSMScript-Lite/total)](https://github.com/NEVSTOP-LAB/CSMScript-Lite/releases)

CSMScript-Lite is a lightweight script execution engine built on the [Communicable State Machine (CSM)](https://github.com/NEVSTOP-LAB/Communicable-State-Machine) framework. It executes flexible CSM-based test scripts to automate testing workflows, and also serves as a practical demonstration of CSM's capabilities. The design concept is similar to NI TestStand.

This project includes:

- **CSMScript-Lite Library** — A lightweight CSM script execution engine, itself implemented as a CSM module.
  - **[Engine](docs/Engine(CSM).md)**: Core execution engine that parses and runs CSM scripts, managing test state and results.
  - **[UI (ExecutionView)](docs/ExecutionView(CSM).md)**: User interface for script management, execution control, and result viewing.
  - **[App](docs/App.md)**: A sample application demonstrating how to use the CSMScript library.
- **Example Projects** — Demonstrate how to combine CSMScript-Lite with other CSM modules for script-driven automated testing.

![CSMScriptApp](.github/CSMScript%20UI.png)

## Dependencies

- LabVIEW 2020 or later
- [Communicable State Machine Framework](https://github.com/topics/labview-csm)
  - [Communicable State Machine (CSM)](https://github.com/NEVSTOP-LAB/Communicable-State-Machine)
  - [CSM API String Arguments Support](https://github.com/NEVSTOP-LAB/CSM-API-String-Arguments-Support)
  - [CSM MassData Parameter Support](https://github.com/NEVSTOP-LAB/CSM-MassData-Parameter-Support)
  - [CSM INI Static Variable Support](https://github.com/NEVSTOP-LAB/CSM-INI-Static-Variable-Support)

## Features

### Script Execution

Supports all CSM commands, including sync messages, async messages, and broadcast/subscription. Refer to the [CSM documentation](https://github.com/NEVSTOP-LAB/Communicable-State-Machine) for full syntax details.

### Return Value Capture

Use the `=> varName` syntax to save a command's return value into the script's temporary variable space for use in subsequent commands:

```c
message1 >> arguments -@ module1 => returnValueVar
message2 >> ${returnValueVar} -@ module2
```

> [!NOTE]
> This feature is also supported in `CSM - Run Script.vi` and behaves identically.

### Extended Commands

CSMScript provides built-in commands beyond the standard CSM command set. The syntax follows the same pattern as CSM: `command >> arguments`. Command names are case-insensitive.

| Category | Command | Description |
|---|---|---|
| Jump | `GOTO` | Jump to the specified `<anchor>` |
| Auto Error Handling | `AUTO_ERROR_HANDLE_ENABLE` | Enable or disable automatic error handling |
| Auto Error Anchor | `AUTO_ERROR_HANDLE_ANCHOR` | Set the error jump anchor (default: `<cleanup>`) |
| Wait | `WAIT`, `Sleep` | Wait for a specified duration; supports `min`, `s`, and `ms` units in one expression |
| Wait (seconds) | `WAIT(s)`, `Sleep(s)` | Wait for a float number of seconds |
| Wait (milliseconds) | `WAIT(ms)`, `Sleep(ms)` | Wait for an integer number of milliseconds |

Examples:

```c
message1 >> arguments -@ csm
wait >> 1min 20s 500ms  // wait 1 minute, 20 seconds, and 500 milliseconds

message1 >> arguments -@ csm
wait(ms) >> 100          // wait 100 milliseconds

message1 >> arguments -@ csm
wait(s) >> 1.5           // wait 1.5 seconds
```

### Anchors and Jumps

Scripts can define named anchors and jump to them on error or via explicit `GOTO` commands.

- **Anchor definition**: `<anchor_name>` — e.g., `<setup>`, `<main>`, `<error_handler>`, `<cleanup>`. Names are case-insensitive.
- **Conditional jump**: `?? goto >> <anchor_name>` — jumps to the anchor if the preceding command produced an error. Omitting the condition expression means "on any error."

`AUTO_ERROR_HANDLE_ANCHOR` sets the default error-jump anchor (default: `<cleanup>`).
`AUTO_ERROR_HANDLE_ENABLE` turns on automatic error handling so that any failing command automatically jumps to the preset anchor.

Example:

```c
// Enable automatic error handling
AUTO_ERROR_HANDLE_ENABLE >> TRUE
// Override default error anchor to error_handler
AUTO_ERROR_HANDLE_ANCHOR >> error_handler

<setup>  // ----- setup anchor ----

// Initialization failure should skip "stop", so jump directly to cleanup on error
initialize >> daq1 -@ ai ?? goto >> <cleanup>

<main>  // ----- main anchor ----

// All subsequent commands jump to error_handler on failure
configure >> Onboard Clock;10,-10,RSE -@ ai
start -@ ai
acquire >> Channel:ch0;Num:1000 -@ ai

<error_handler>  // ----- error handler anchor ----
stop -@ ai

<cleanup>  // ----- cleanup anchor ----
close -@ ai
```

> [!NOTE]
> Automatic error handling is **disabled by default**. Enable it with `AUTO_ERROR_HANDLE_ENABLE >> TRUE`. Without it, execution continues to the next command even after an error.

> [!NOTE]
> Anchors are most commonly used to define script phases, similar to the `<setup>`, `<main>`, and `<cleanup>` sequence stages in NI TestStand.

> [!NOTE]
> The `<>` brackets in anchor names are optional. `GOTO >> cleanup` and `GOTO >> <cleanup>` are equivalent.
