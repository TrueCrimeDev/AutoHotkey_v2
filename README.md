# Featured Projects

## [RunAdmin](https://github.com/jasc2v8/AutoHotkey_v2/blob/main/Projects/~Lib/RunAdmin)

>Run a script.ahk or script.exe elevated without the UAC prompt.

## [RunLib](https://github.com/jasc2v8/AutoHotkey_v2/blob/main/Projects/~Lib/RunLib/)

>Library to run any command line without a command window and return StdOut & StdErr.

>CommandLine: Array, CSV, or String.

>Extensions: ahk, bat, cmd, exe, ps1, or none (built-in command e.g. dir).

>Handles spaces in the arguments as needed.

## [BackupTool](https://github.com/jasc2v8/AutoHotkey_v2/blob/main/Projects/~Tools/BackupTool/)

>Run SyncBackSE.exe as Admin without the UAC prompt.

## [PowerTool](https://github.com/jasc2v8/AutoHotkey_v2/blob/main/Projects/~Tools/PowerTool/)

>Push Button Gui for Sleep, Signout, Shutdown, Restart, and Display Off. 

---

# Unique & Advanced Library Features

## Inter-Process Communication (IPC)

### [SharedMemory.ahk](Lib/SharedMemory.ahk)
The most technically deep IPC mechanism in the library.  It wraps the Windows memory-mapping API (`CreateFileMapping` / `MapViewOfFile`) with kernel event objects so two scripts can exchange UTF-16 strings without polling.

Key highlights:
- **Security descriptor** built with `ConvertStringSecurityDescriptorToSecurityDescriptor` (SDDL `"D:(A;;GA;;;WD)"`) so a non-admin and an admin process can share the same named mapping.
- **Dual-role design** — the same class acts as either *server* or *client*; the `IsServer` flag controls which kernel event is signalled on `Write()` and waited on in `WaitRead()`.
- **Zero-on-read/write** — stale bytes are cleared with `RtlZeroMemory` before every write and after every read, preventing data leakage between messages.
- **`__Delete()` destructor** — automatically unmaps the view and closes all four handles (mapping, view, server event, client event) when the object goes out of scope.

### [NamedPipe.ahk](Lib/NamedPipe.ahk)
Full-duplex named-pipe communication with server/client lifecycle management.

Key highlights:
- Separate `Create()` (server) and `Wait()` (client) paths — the client side calls `WaitNamedPipe` to block until a server instance is available, then uses `CreateFile` to connect.
- Security attributes with an SDDL string are applied at pipe creation to allow cross-privilege connections.
- Error 231 (pipe busy) and Error 2 (pipe does not exist yet) are handled gracefully with automatic retry.
- Used internally by `RunAdmin` to relay elevated-process replies back to the caller.

### [Messenger.ahk](Lib/Messenger.ahk)
Lightweight IPC via the `WM_COPYDATA` (0x4A) window message — no shared memory or pipe handles required.

Key highlights:
- **UIPI bypass** — calls `ChangeWindowMessageFilterEx` with `MSGFLT_ALLOW` so a low-integrity (non-admin) sender can reach a high-integrity (admin) receiver, which would otherwise be silently blocked by Windows.
- **PassKey authentication** — `dwData` in `COPYDATASTRUCT` carries a user-supplied numeric key; mismatched keys are rejected before the user callback fires.
- **Reply support** — `Reply()` uses the `wParam` HWND captured during `_HandleIncoming()` to send a response back to the original sender without re-querying the window list.
- Fully handles hidden windows (`DetectHiddenWindows True`) and supports an optional send timeout.

---

## Object Inspection & Visualization

### [ObjTree.ahk](Lib/ObjTree.ahk)
An interactive GUI TreeView for inspecting any AHK object at runtime.

Key highlights:
- **Circular-reference detection** — a visited-object map prevents infinite recursion when the same object appears more than once in the tree.
- **Dark / Light theme toggle** — switches colours on the TreeView, Edit control, labels, and buttons, and calls `uxtheme\SetWindowTheme` + a `DwmSetWindowAttribute` helper (`SetDarkModeFrame`) to flip the non-client area as well.
- **Live filter** — typing in the filter box instantly rebuilds the tree, showing only nodes whose key or value contains the search text.
- **JSON-style string detection** — values that look like newline-delimited lists or CSV rows are rendered as child nodes instead of a flat string.
- **Context menu** (right-click) and double-click to copy a node's full line or its value alone.
- **Export to file** from the context menu.

### [ObjView.ahk](Lib/ObjView.ahk)
A console-window object pretty-printer with more than 50 configurable display properties.

Key highlights:
- Tracks duplicate references to detect and annotate circular structures.
- Handles unset variable references without throwing.
- Supports a live GUI console with Pause / Continue / Reload / Exit hotkey buttons and auto-scroll.

---

## System & Process Utilities

### [RunAdmin.ahk](Lib/RunAdmin.ahk) / [RunAdminIPC.ahk](Lib/RunAdminIPC.ahk)
Elevates a script or executable **without a UAC prompt** by registering a Windows Task Scheduler task that already runs at the highest privilege level.

Key highlights:
- **Four operation modes**: direct run, NamedPipe-based IPC (recommended for `RunWait` with reply), shortcut target, and one-time setup.
- The calling (non-elevated) script sends the command via a named pipe to the already-elevated `RunAdmin` instance, which executes it and pipes the reply back.
- CSV parameter encoding is used to safely pass multi-argument command lines through the pipe.

### [ProcessMonitor.ahk](Lib/ProcessMonitor.ahk)
Non-blocking process lifecycle watcher built on a configurable polling timer.

Key highlights:
- `ObjBindMethod` is used to bind the timer callback to the class instance, avoiding global state.
- Raises `OnExit`, `OnStatus`, and `OnStop` callbacks with elapsed-time formatting.
- Supports lookup by process name or by PID.

### [RegSettings.ahk](Lib/RegSettings.ahk)
Thin registry persistence layer that auto-detects REG_SZ / REG_DWORD / REG_BINARY / REG_MULTI_SZ types and namespaces all keys under `HKCU\Software\AHK_Scripts\<ScriptName>`.

---

## Data Processing

### [JSON.ahk](Lib/JSON.ahk)
Recursive-descent JSON parser and serialiser.

Key highlights:
- Preserves integer vs. float distinction when round-tripping numbers.
- Optional `ComValue`-based true / false / null types for strict JSON boolean semantics.
- Dual output modes: native AHK `Map` or plain `Object`.

### [CSV.ahk](Lib/CSV.ahk)
RFC 4180-compliant CSV ↔ 2D-array converter.

Key highlights:
- Handles quoted fields, embedded commas, newlines inside quotes, and `""` escape sequences.
- Configurable delimiter (comma, tab, or custom character).

---

## GUI Utilities

### [GuiLayout.ahk](Lib/GuiLayout.ahk)
Anchor-based responsive layout engine for AHK GUIs — controls reposition and resize proportionally when the window is resized.

### [CustomMsgBox.ahk](Lib/CustomMsgBox.ahk) / [MsgBoxCustom.ahk](Lib/MsgBoxCustom.ahk)
Fully customisable message dialog: window size, background and text colours, fonts, button labels, icon, and optional sound.  A right-click context menu lets the user copy the message text.  Returns the name of the button that was pressed.

---

## Advanced Patterns Used Across the Library

| Pattern | Where |
|---|---|
| **Kernel event synchronisation** | `SharedMemory` — zero-polling IPC via `WaitForSingleObject` |
| **SDDL security descriptors** | `SharedMemory`, `NamedPipe` — cross-privilege access without hardcoded SIDs |
| **UIPI message filter bypass** | `Messenger` — admin ↔ non-admin window messaging |
| **Dual-role server/client class** | `SharedMemory`, `NamedPipe` — one class, two roles, controlled by a flag |
| **Circular-reference detection** | `ObjTree`, `ObjView` — safe recursion over arbitrary object graphs |
| **`ObjBindMethod` timer callbacks** | `ProcessMonitor` — instance methods as timer targets without globals |
| **`__Delete()` resource cleanup** | `SharedMemory` — deterministic handle/memory release via destructor |
| **Task Scheduler elevation** | `RunAdmin` — UAC bypass via a pre-registered scheduled task |

---

## Note

**This repository is for learning purposes.**  Many scripts in this are from different sources and have not been fully tested.  They may be incomplete and may not work as-is, but are a good starting point to expand and enhance their functionality.
