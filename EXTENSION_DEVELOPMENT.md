# MC4JLauncher — Extension Development Guide

**Official guide for creating extensions for the MC4J ecosystem**

Version: 1.0  
Language: Python  
Platform: Windows  
Compatible software: MC4JLauncher

---

## 1. Introduction

MC4JLauncher supports extensions that allow developers to expand the launcher's functionality without directly modifying its source code.

Extensions can interact with the launcher through the `ExtensionAPI`, an interface provided when the extension is executed.

The API provides functionality such as:

- Logging messages.
- Displaying notifications and errors.
- Opening files and directories.
- Selecting files through a dialog.
- Accessing game state information.
- Executing background tasks.
- Receiving game lifecycle events.

Each extension operates independently inside its own directory.

## 2. Extension structure

Every extension must be placed inside its own subdirectory within the `extensions/` directory.

The required structure is:

```text
extensions/
│
└── MyExtension/
    │
    ├── extension.json
    └── main.py
```

### Required files

| File | Description |
|---|---|
| `extension.json` | Contains extension metadata. |
| `main.py` | Contains the extension's Python implementation and entry point. |

Multiple extensions can coexist in the same directory.

Example:

```text
extensions/
│
├── MyExtension/
│   ├── extension.json
│   └── main.py
│
├── AnotherExtension/
│   ├── extension.json
│   └── main.py
│
└── ThirdExtension/
    ├── extension.json
    └── main.py
```

## 3. Extension metadata

The `extension.json` file contains information identifying the extension.

Example:

```json
{
    "id": "my_extension",
    "name": "My Extension",
    "version": "1.0.0",
    "description": "An example MC4JLauncher extension."
}
```

### Metadata fields

| Field | Description |
|---|---|
| `id` | Unique extension identifier. |
| `name` | Human-readable extension name. |
| `version` | Extension version. |
| `description` | Brief explanation of its functionality. |

**Important:** The `id` field is required by the currently documented `ExtensionAPI` implementation, which uses it for extension logging. Other fields are recommended metadata; additional requirements imposed by the extension loader should also be respected.

Use a unique identifier for each extension to avoid conflicts.

## 4. Creating the entry point

The `main.py` file is the main execution point of an extension.

Every extension must expose a function named `run(api)`.

Example:

```python
def run(api):
    api.log("My Extension has started!")

    api.notify(
        "My Extension",
        "Extension successfully loaded!"
    )
```

MC4JLauncher provides the API object when executing the extension.

The `api` parameter gives access to the supported launcher functionality.

Do not attempt to instantiate `ExtensionAPI` yourself.

## 5. Understanding the ExtensionAPI

The following methods are available in the supplied MC4JLauncher API implementation.

### 5.1. Accessing game information

```python
game = api.game
```

Returns a shallow copy of the current game state.

Example:

```python
def run(api):
    game = api.game

    api.log(f"Current game state: {game}")
```

The returned object is a dictionary.

Its available keys depend on the game state implementation.

The property is intended for reading information, not modifying the launcher's internal state.

### 5.2. Logging

```python
api.log("Hello from my extension!")
```

Sends a message to the launcher's logging system.

Messages are automatically identified using the extension ID.

Example output:

```text
[Extensão my_extension] Hello from my extension!
```

Use logging to report operations, debugging information and unexpected situations.

### 5.3. Notifications

```python
api.notify(
    "My Extension",
    "Operation completed successfully!"
)
```

Displays an informational message box.

Parameters:

- `title`: Notification title.
- `message`: Notification content.

The notification is scheduled on the launcher's main interface thread.

### 5.4. Displaying errors

```python
api.show_error(
    "Extension Error",
    "An unexpected error occurred."
)
```

Displays an error message box.

Use this method when an operation fails and the user needs to be informed.

### 5.5. Selecting files

```python
file_path = api.ask_open_file()
```

Opens the system file selection dialog.

You can customize its title and accepted file types.

Example:

```python
file_path = api.ask_open_file(
    title="Select a configuration file",
    filetypes=[
        ("JSON files", "*.json"),
        ("All files", "*.*")
    ]
)

if file_path:
    api.log(f"Selected file: {file_path}")
```

The method returns the selected file path or an empty string if the dialog is cancelled.

**Thread restriction:** This method must be called from Python's main thread.

### 5.6. Opening files and directories

```python
api.open_path("C:/MyFolder")
```

Opens a file or directory using the operating system's default application.

The method supports Windows and Linux.

It raises `FileNotFoundError` if the specified path does not exist.

Example:

```python
try:
    api.open_path("C:/MyFolder")

except FileNotFoundError:
    api.show_error(
        "File Not Found",
        "The specified directory does not exist."
    )
```

### 5.7. Background execution

Extensions can execute time-consuming operations without blocking the launcher's interface.

Use:

```python
api.run_in_background(function)
```

The method creates a daemon thread to execute the supplied function.

It also supports completion and error callbacks.

Full signature:

```python
api.run_in_background(
    function,
    *args,
    on_done=None,
    on_error=None,
    **kwargs
)
```

Example:

```python
import time


def background_task():
    time.sleep(3)
    return "Task completed!"


def on_complete(result):
    api.notify("Background Task", result)


def on_error(error):
    api.show_error("Task Failed", str(error))


def run(api):
    api.run_in_background(
        background_task,
        on_done=on_complete,
        on_error=on_error
    )
```

The callbacks are scheduled on the launcher's main interface thread.

Use background execution for:

- File processing.
- Network requests.
- Data analysis.
- Large calculations.
- Other operations that could temporarily freeze the interface.

Avoid directly manipulating Tkinter widgets from background threads.

## 6. Extension events

MC4JLauncher supports events associated with the game's lifecycle.

Currently documented events include:

| Event | Description |
|---|---|
| `game_started` | Indicates that the game has started. |
| `game_stopped` | Indicates that the game has stopped. |

The API exposes the event bus through:

```python
api.events
```

Extensions can use this object to integrate with the launcher's event system.

## 7. Complete example

Below is a complete example of a simple extension.

Directory:

```text
extensions/
└── SystemUtility/
    ├── extension.json
    └── main.py
```

### extension.json

```json
{
    "id": "system_utility",
    "name": "System Utility",
    "version": "1.0.0",
    "description": "A simple utility extension for MC4JLauncher."
}
```

### main.py

```python
import os
import time


def scan_directory():
    time.sleep(2)

    return os.getcwd()


def on_scan_complete(result):
    api.notify(
        "System Utility",
        f"Current directory: {result}"
    )


def on_scan_error(error):
    api.show_error(
        "System Utility",
        str(error)
    )


def run(extension_api):
    global api
    api = extension_api

    api.log("System Utility initialized.")

    api.notify(
        "System Utility",
        "The extension has been loaded!"
    )

    api.run_in_background(
        scan_directory,
        on_done=on_scan_complete,
        on_error=on_scan_error
    )
```

This example demonstrates:

- Extension initialization.
- Logging.
- Notifications.
- Background execution.
- Completion callbacks.
- Error handling.

## 8. Packaging extensions

MC4JEditor supports including extensions in Minecraft Legacy project distributions.

To include an extension:

1. Open MC4JEditor.
2. Load your Minecraft Legacy project.
3. Navigate to the Packaging page.
4. Locate the Auxiliary Files section.
5. Select your `extensions/` directory.
6. Complete the remaining package configuration.
7. Create the `.mc4j` package.

The extension directory is copied alongside the generated archive.

It is not compressed inside the `.mc4j` file.

Expected distribution:

```text
Minecraft Legacy/
│
├── Minecraft.mc4j
├── installation.txt
├── main.py
│
├── x64/
│
└── extensions/
    │
    └── SystemUtility/
        ├── extension.json
        └── main.py
```

MC4JLauncher can then access the distributed extension files through its extension management system.

## 9. Development recommendations

When developing extensions, follow these recommendations:

**Use the provided API**

Prefer `api.notify()`, `api.log()` and other supported methods instead of accessing launcher internals directly.

**Keep operations asynchronous**

Use `api.run_in_background()` for time-consuming operations.

**Handle exceptions**

Unexpected errors should be caught and reported appropriately.

**Respect thread restrictions**

File dialogs and interface operations must follow the main-thread requirements.

**Use unique identifiers**

Each extension should have its own unique metadata ID.

**Keep extensions independent**

Avoid requiring modifications to MC4JLauncher itself just to install an extension.

## 10. Security considerations

MC4JLauncher extensions execute Python code.

An extension should therefore be treated as executable software, not as a passive configuration file.

Only install extensions from sources you trust.

Developers should:

- Avoid executing untrusted downloaded files.
- Validate user-provided paths.
- Avoid exposing sensitive information.
- Handle network responses and external data safely.
- Avoid destructive operations without user confirmation.

The `ExtensionAPI` provides a supported interface, but it should not be assumed to be a security sandbox.

## 11. Troubleshooting

| Problem | Possible solution |
|---|---|
| Extension is not detected | Verify its directory is inside `extensions/`. |
| Missing `extension.json` | Create the required metadata file. |
| Missing `main.py` | Add the extension entry point. |
| Missing extension ID | Verify the `id` field in `extension.json`. |
| Extension fails during initialization | Check the launcher's logs and Python exceptions. |
| Interface becomes unresponsive | Move lengthy operations into background tasks. |
| File dialog raises an exception | Ensure it is called from the main thread. |
| Background task fails | Implement an `on_error` callback. |

---

## 12. Additional resources

- **MC4JEditor:** https://github.com/Project-MC4J/MC4JEditor
- **MC4JLauncher:** https://github.com/Project-MC4J
- **Python Documentation:** https://docs.python.org/3/

---

**MC4J Project — Extension Development Guide**

Extend the MC4JLauncher experience through independently developed Python extensions.
