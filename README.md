# MC4JEditor — Packaging Guide

A packaging tool for Minecraft Legacy Console Edition projects within the MC4J ecosystem.

## 1. Overview

MC4JEditor allows users to package Minecraft Legacy projects into `.mc4j` archives, along with optional auxiliary files required by MC4JLauncher.

The packaging process separates project source files from launcher-related metadata, icons, extensions, and executable binaries.

**MC4JEditor does not include or distribute Minecraft Legacy Console Edition source code.** Users must independently obtain their own copy of the source code for the specific Minecraft Legacy version they want to package.

Downloading MC4JEditor does not provide the Minecraft source code, compiled game binaries, or a ready-to-play game installation.

## 2. Getting Started

### Requirements

- Windows
- MC4JEditor executable
- A valid Minecraft Legacy project containing the source files you intend to package

**No Python installation is required to run MC4JEditor.** It is distributed as a standalone executable.

### Opening MC4JEditor

1. Download the MC4JEditor executable.
2. Place it in a convenient location.
3. Run `MC4JEditor.exe`.

## 3. Prepare Your Project

Before starting the packaging process:

1. Obtain your own copy of the source code for the Minecraft Legacy version you want to package.
2. Open MC4JEditor.
3. Load your Minecraft Legacy project from the **Home** page.
4. Verify that your project contains all the source files you wish to package.
5. Ensure the required `x64/` binary directory is present in your project root.

### How Packaging Works

MC4JEditor recursively collects project source files and bundles them inside the `.mc4j` archive.

Certain root-level items are deliberately kept outside the archive so they remain accessible to MC4JLauncher:

- `installation.txt` — Generated automatically.
- `icon.png` — Optional package icon.
- `main.py` — Optional entry-point script.
- `extensions/` — Optional launcher extensions.
- `x64/` — Compiled game binaries.

## 4. Open the Packaging Page

Select **Packaging** from the left navigation menu.

The interface is divided into three main sections:

- **Package Metadata:** Project naming and versioning.
- **Auxiliary Files:** Optional scripts, icons, and extensions.
- **Package Output:** Destination settings.

## 5. Configure Package Metadata

The following fields are required:

- **Name:** The name of your Minecraft Legacy project.
- **Description:** A short summary of what your project includes.
- **Version:** The current release version (e.g., `1.0.0`).

## 6. Select Auxiliary Files

Auxiliary files are optional assets copied directly alongside the final `.mc4j` archive.

### 6.1. Icon (`icon.png`)

- Select a PNG image to serve as the package icon.
- The file will be saved in the output directory as `icon.png`.
- If left blank, MC4JEditor automatically looks for an `icon.png` in your project root.

### 6.2. Main Script (`main.py`)

- Select the main Python script used as the launcher's entry point.
- The selected file must be named exactly `main.py`.
- If left blank, MC4JEditor automatically looks for a `main.py` in your project root.

### 6.3. Extensions (`extensions/`)

- Select the directory containing your MC4JLauncher extensions.
- Every subfolder represents a separate extension and must contain:
  - `extension.json`
  - `main.py`

Example structure:

```text
extensions/
├── ExtensionA/
│   ├── extension.json
│   └── main.py
└── ExtensionB/
    ├── extension.json
    └── main.py
```

The original directory structure is preserved. Extensions remain outside the compressed `.mc4j` archive.

If no directory is selected, MC4JEditor attempts to automatically detect an `extensions/` folder in the project root.

## 7. Choose the Output Location

1. In the **Package Output** section, click **Choose Destination**.
2. Select your target directory and enter a filename.
3. Ensure the filename ends with the `.mc4j` extension.

**Important:** The output destination must be outside your source project directory to prevent recursive packaging loops.

## 8. Create the Package

Once the configuration is complete:

1. Double-check your metadata fields.
2. Verify the paths for your auxiliary files.
3. Ensure the output destination is correct.
4. Click **Create .mc4j Package**.

MC4JEditor performs a validation before packaging.

If conflicting files already exist in the target directory, a confirmation prompt will ask whether you want to overwrite them.

## 9. Final Output Structure

A successfully generated distribution folder follows this layout:

```text
Minecraft Legacy/
│
├── Minecraft.mc4j          # Compressed source files
├── installation.txt        # Automatically generated metadata
├── icon.png                # Optional package icon
├── main.py                 # Optional entry-point script
│
├── x64/                    # Compiled game binaries
│   └── ...
│
└── extensions/             # Launcher extensions
    ├── ExtensionA/
    │   ├── extension.json
    │   └── main.py
    └── ExtensionB/
        ├── extension.json
        └── main.py
```

### File Responsibilities

| File or Directory | Purpose |
|---|---|
| `Minecraft.mc4j` | Contains the collected project source files. |
| `installation.txt` | Contains package metadata required by the launcher. |
| `icon.png` | Optional package icon. |
| `main.py` | Optional entry-point script. |
| `x64/` | Contains compiled game binaries. |
| `extensions/` | Contains external extensions for MC4JLauncher. |

### installation.txt Format

This file is generated automatically by MC4JEditor using the metadata provided:

```text
name=Project Name
description=Project Description
version=1.0.0
```

## 10. Extension Requirements

Extensions are handled at runtime by MC4JLauncher.

Each extension must:

- Be placed in its own subfolder inside `extensions/`.
- Include an `extension.json` file.
- Include a `main.py` entry-point file.

The Python entry point must follow this structure:

```python
def run(api):
    # Your extension logic here
    pass
```

## 11. Bundled Third-Party Software

MC4JEditor includes the **VSCodium CLI** as a third-party component.

VSCodium is an independent, community-driven distribution of Visual Studio Code. It is not developed or maintained by the MC4J project.

The VSCodium project is distributed under the **MIT License**. Its license permits redistribution, provided that the required copyright notice and license text are retained.

### VSCodium License and Copyright

The bundled VSCodium component is subject to its own license, independently of MC4JEditor's licensing terms.

Copyright notices associated with the VSCodium project include:

- Copyright (c) 2018-present The VSCodium contributors
- Copyright (c) 2018-present Peter Squicciarini
- Copyright (c) 2015-present Microsoft Corporation

The original MIT License must accompany the bundled VSCodium component in the MC4JEditor distribution.

Any applicable third-party notices supplied with the bundled VSCodium distribution must also be preserved and made available.

### Third-Party License Information

- VSCodium repository: https://github.com/VSCodium/vscodium
- VSCodium license: https://github.com/VSCodium/vscodium/blob/master/LICENSE
- VSCodium website: https://vscodium.com/

MC4JEditor does not claim ownership of VSCodium or its third-party components.

The inclusion of VSCodium CLI does not imply endorsement, sponsorship, or official affiliation between the VSCodium project and MC4J.

## 12. Troubleshooting

| Error Message | Solution |
|---|---|
| **No source code has been loaded** | Return to the Home page and open a valid project before packaging. |
| **Missing package metadata** | Ensure Name, Description, and Version fields are not blank. |
| **Extensions directory contains no extensions** | Select a directory containing at least one valid extension folder. |
| **Missing extension.json / main.py** | Add the missing required files inside the extension folder. |
| **x64 directory not found** | Ensure an `x64/` directory exists in your project root. |
| **Output directory is inside source** | Select a destination outside your source project directory. |
| **Files already exist** | Confirm the overwrite prompt or choose another filename or destination. |

---

**MC4JEditor — Packaging Minecraft Legacy projects for the MC4J ecosystem.**
