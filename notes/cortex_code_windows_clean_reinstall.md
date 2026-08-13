# Cortex Code CLI on Windows

## Clean uninstall and fresh reinstall guide

**Purpose:** Remove stale Unix-style paths such as `C:\Users\kevin\.local\bin\cortex`, reinstall the native Windows CLI, and configure the Snowflake VS Code extension.


## Before you begin

Close Visual Studio Code and all PowerShell or Command Prompt windows running Cortex. These steps remove the CLI installation folders but preserve Snowflake connection files under your Snowflake configuration directory.

## 1. Stop any running Cortex process

Run in PowerShell:

```powershell
Get-Process cortex -ErrorAction SilentlyContinue | Stop-Process -Force
```

## 2. Remove old CLI installations

The first path is the native Windows installation location. The second removes a stale Unix/WSL-style shim if one exists.

```powershell
Remove-Item -Recurse -Force "$env:LOCALAPPDATA\cortex" -ErrorAction SilentlyContinue
Remove-Item -Force "$env:USERPROFILE\.local\bin\cortex*" -ErrorAction SilentlyContinue
```

## 3. Clear the stale CLI override

Remove the user-level environment variable that may continue pointing VS Code or an SDK at the old executable:

```powershell
[Environment]::SetEnvironmentVariable("CORTEX_CODE_CLI_PATH", $null, "User")
```

In VS Code, open **Preferences: Open User Settings (JSON)**. Remove the old `snowflake.coco.cliPath` entry for now, or replace it after reinstalling.

## 4. Install Cortex Code CLI for native Windows

Open a new PowerShell window and run:

```powershell
irm https://ai.snowflake.com/static/cc-scripts/install.ps1 | iex
```

The native Windows installer places the executable under `%LOCALAPPDATA%\cortex` and adds the installation directory to `PATH`.

## 5. Verify the installation

Open another new PowerShell window and run:

```powershell
where.exe cortex
cortex --version
Get-ChildItem "$env:LOCALAPPDATA\cortex" -Filter cortex.exe -Recurse
```

If the first command returns a path under `.local\bin`, close that PowerShell window, open a new one, and recheck your environment variables and `PATH`.

## 6. Configure the Snowflake VS Code extension

Use the exact native `cortex.exe` path returned in step 5. For example:

```json
{
  "snowflake.coco.cliPath": "C:\\Users\\kevin\\AppData\\Local\\cortex\\<version>\\cortex.exe"
}
```

Save the file, then run **Developer: Reload Window** from the VS Code Command Palette.

> **Important:** Do not configure the extension with `C:\Users\kevin\.local\bin\cortex` or with a `.cmd` wrapper. The extension requires the native Windows executable.

## Optional: Diagnose any remaining stale path

```powershell
$env:CORTEX_CODE_CLI_PATH
[Environment]::GetEnvironmentVariable("CORTEX_CODE_CLI_PATH", "User")
[Environment]::GetEnvironmentVariable("CORTEX_CODE_CLI_PATH", "Machine")
```

If any command returns the old malformed path, remove or correct that environment variable and restart VS Code.

## What this does not remove

The commands above do not remove `%USERPROFILE%\.snowflake\cortex`. That directory may contain Cortex settings, conversations, permissions, and MCP configuration. Delete it only if you intentionally want a complete configuration reset.

## Sources

* [Snowflake CoCo CLI documentation](https://docs.snowflake.com/en/user-guide/cortex-code/cortex-code-cli) — Windows installation and PATH behavior.
* [Snowflake support guidance](https://snowforce.lightning.force.com/lightning/r/KnowledgeArticle/kA9VI000000DGyD0AW/view) — Windows installation cleanup and troubleshooting. This article is internal only.
