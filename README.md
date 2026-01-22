# CSMStand-Lite

[English](./README.md) | [中文](./README(zh-cn).md)

## Introduction

CSMStand-Lite is a script execution engine based on the Communicable State Machine (CSM) framework, designed to run flexible CSM test scripts for automated testing. It also serves to demonstrate the capabilities of the CSM framework. Its design is inspired by NI TestStand, which is the origin of its name.

This project includes:

1. **CSMStand-Lite Library**: A lightweight CSM script execution engine, itself implemented as a CSM-based module.
   - **Engine**: The core execution engine responsible for parsing and executing CSM scripts, managing test states and results.
   - **UI**: The user interface that provides script management, execution control, and result viewing.
2. **CSMStandApp**: A CSM UI module that calls the CSMStand-Lite engine, demonstrating how to integrate and use CSMStand-Lite.
3. **Example Project**: Showcases how to use CSMStand-Lite in conjunction with other CSM modules to achieve script execution and automated testing.

![CSMStandApp](.github/csmstand%20ui.png)

## CSMStand-Lite Library Feature Overview

### Basic Feature: Script Execution

Supports all CSM commands, including synchronous messages, asynchronous messages, broadcast subscriptions, and more. For detailed information, please refer to the CSM framework documentation.

### Return Value Storage and Passing

Using the `=> variable_name` syntax, the return value of a command is stored in the script's temporary variable space for use in subsequent commands. For example:

```c
message1 >> arguments -@ module1 => returnValueVar
message2 >> ${returnValueVar} -@ module2
```

> [!NOTE]
> This feature is also supported in CSM - Run Script.vi, and it functions identically here.

### Extended Commands

CSMStand includes some extended commands not provided by CSM, used to implement basic functionalities such as jump instructions.

The format is similar to CSM commands: `command >> parameters`, and command names are case-insensitive.

| Command Set Name                   | Command                  | Description                                                                 |
|------------------------------------|--------------------------|-----------------------------------------------------------------------------|
| Jump Command                       | GOTO                     | Jump to the specified `<anchor>`                                            |
| Enable/Disable Auto Error Handling | AUTO_ERROR_HANDLE_ENABLE | Enable or disable automatic error handling                                  |
| Set Auto Error Jump Anchor         | AUTO_ERROR_HANDLE_ANCHOR | Set the anchor for automatic error jumps, default is `<cleanup>`            |
| Wait Command 1                     | WAIT, Sleep              | Wait for a specified time, supports minute(min), second(s), millisecond(ms) |
| Wait Command 2                     | WAIT(s), Sleep(s)        | Wait for a specified time in seconds, parameter is Float type               |
| Wait Command 3                     | WAIT(ms), Sleep(ms)      | Wait for a specified time in milliseconds, parameter is NUMERIC type        |

Examples:

```c
message1 >> arguments -@ csm
wait >> 1min 20s 500ms  // Wait for 1 minute 20 seconds 500 milliseconds

message1 >> arguments -@ csm
wait(ms) >> 100  // Wait for 100 milliseconds

message1 >> arguments -@ csm
wait(s) >> 1.5  // Wait for 1.5 seconds
```

### Anchor Jumping

Supports defining anchors in scripts and jumping to specified anchors for continued execution based on error jumps or variable logic judgments.

- **Anchor Definition Format**: `<anchor_name>`, e.g., `<setup>`, `<main>`, `<error_handler>`, `<cleanup>`, etc.
- **Anchor Jump Format**: `?? goto >> <anchor_name>` syntax, jumps to the specified anchor when a command execution error occurs. The default expression indicates jumping on command execution error.

The `AUTO_ERROR_HANDLE_ANCHOR` command can set the anchor for automatic error jumps, defaulting to `<cleanup>`.

The `AUTO_ERROR_HANDLE_ENABLE` command can enable automatic error handling, automatically jumping to the preset anchor when a command execution error occurs.

For example:

```c
// Enable automatic error handling
AUTO_ERROR_HANDLE_ENABLE >> TRUE
// Set the automatic error jump anchor to error_handler, no longer using cleanup as the default anchor
AUTO_ERROR_HANDLE_ANCHOR >> error_handler

<setup>  // ----- setup anchor ----

// Logically, if initialization fails, there's no need to execute the stop command,
// so here we explicitly add a jump to the cleanup anchor on error.
// If this command fails, it will jump to the cleanup anchor
initialize >> daq1 -@ ai ?? goto >> <cleanup>

<main>  // ----- main anchor ----

// All subsequent commands, if they fail, will jump to the error_handler anchor
configure >> Onboard Clock;10,-10,RSE -@ ai

start -@ ai

acquire >> Channel:ch0;Num:1000 -@ ai

<error_handler>  // ----- error handler anchor ----
stop -@ ai

<cleanup> //----- cleanup anchor ----
close -@ ai

```

> [!NOTE]
> By default, automatic error handling is disabled and needs to be enabled via the `AUTO_ERROR_HANDLE_ENABLE` command.
> When an error occurs, it will continue executing the subsequent commands.

> [!NOTE]
> The most common anchors are used to define script execution stages, for example, you can define anchors similar to TestStand sequences such as `<setup>`, `<main>`, `<cleanup>`, etc.
