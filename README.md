# Amplifier PowerShell Tool Module

PowerShell command execution for Amplifier agents.

## Quick Start (Windows)

Get up and running in under 5 minutes:

### 1. Install Prerequisites

Open PowerShell as Administrator and run:

```powershell
# Install PowerShell Core (if not already installed)
winget install Microsoft.PowerShell

# Install UV (fast Python package manager)
irm https://astral.sh/uv/install.ps1 | iex

# Install Amplifier CLI
uv tool install git+https://github.com/microsoft/amplifier-app-cli
```

Close and reopen PowerShell for PATH changes to take effect.

### 2. Configure Amplifier

```powershell
# Set up your AI provider (choose one)
amplifier provider install anthropic
# OR
amplifier provider install openai
```

### 3. Add This Module

Create or edit your settings file at `~/.amplifier/settings.yaml`:

```yaml
modules:
  sources:
    tool-pwsh: git+https://github.com/Anokye-Labs/amplifier-module-tool-pwsh
```

Or use the CLI:

```powershell
amplifier module source set tool-pwsh git+https://github.com/Anokye-Labs/amplifier-module-tool-pwsh --scope global
```

### 4. Create a PowerShell-Enabled Bundle

Create a file at `~/.amplifier/bundles/pwsh-dev.md`:

```markdown
---
bundle:
  name: pwsh-dev
  version: 1.0.0
  includes:
    - foundation:base
  modules:
    tools:
      - tool-pwsh
---

You have access to PowerShell for Windows system administration and scripting.
Use the `pwsh` tool for all shell operations.
```

### 5. Try It Out!

```powershell
# Start Amplifier with your new bundle
amplifier run --bundle pwsh-dev

# Then ask it to do something with PowerShell:
# "List all running services"
# "Show me disk usage"
# "Find large files in my Downloads folder"
```

## Prerequisites

- **Python 3.11+**
- **[UV](https://github.com/astral-sh/uv)** - Fast Python package manager
- **PowerShell Core (pwsh)** - Recommended for cross-platform use

### Installing UV

```bash
# macOS/Linux/WSL
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"
```

### Installing PowerShell Core

PowerShell Core (`pwsh`) is cross-platform and recommended over Windows PowerShell:

```powershell
# Windows (winget)
winget install Microsoft.PowerShell

# macOS (Homebrew)
brew install powershell/tap/powershell

# Linux (Ubuntu/Debian)
sudo apt-get install -y powershell
```

See: https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell

## Purpose

Enables agents to execute PowerShell commands in a controlled environment. This is the **preferred shell tool on Windows** and works cross-platform where PowerShell Core is installed.

**Platform Behavior:**
- **Windows**: Uses `pwsh` (PowerShell Core) or falls back to `powershell` (Windows PowerShell)
- **Linux/macOS**: Uses `pwsh` if installed (requires explicit installation)

## Contract

**Module Type:** Tool  
**Mount Point:** `tools`  
**Entry Point:** `amplifier_module_tool_pwsh:mount`

## Tools Provided

### `pwsh`

Execute a PowerShell command.

**Input:**

- `command` (string): The PowerShell command to execute
- `run_in_background` (boolean, optional): Run in background, return immediately with PID (default: false)

**Output:**

- `stdout`: Standard output from command
- `stderr`: Standard error from command  
- `returncode`: Exit code (0 = success)

**PowerShell Features:**
- Pipelines: `Get-Process | Where-Object CPU -gt 100`
- Cmdlets: `Get-ChildItem`, `Set-Location`, `Invoke-WebRequest`
- Variables: `$env:PATH`, `$HOME`
- Object-oriented output (converted to text)
- Cross-platform paths: `Join-Path $HOME ".config"`

## Example Prompts

Once configured, try these prompts with Amplifier:

| Prompt | What It Does |
|--------|--------------|
| "List all running services" | `Get-Service \| Where-Object Status -eq Running` |
| "Show system information" | `Get-ComputerInfo` |
| "Find large files in Downloads" | `Get-ChildItem ~/Downloads -Recurse \| Sort-Object Length -Descending \| Select-Object -First 10` |
| "Check disk space" | `Get-PSDrive -PSProvider FileSystem` |
| "List installed programs" | `Get-Package` or `winget list` |
| "Show network configuration" | `Get-NetIPConfiguration` |

## Configuration

In your bundle or settings:

```yaml
modules:
  tools:
    - id: tool-pwsh
      config:
        timeout: 30           # Command timeout in seconds
        require_approval: false
        allowed_commands: []  # Empty = all non-dangerous allowed
```

## Security

**IMPORTANT**: PowerShell execution can be dangerous. Use with caution:

- Set `require_approval: true` for production
- Use `allowed_commands` to whitelist safe commands
- Run in isolated/containerized environments
- Never execute untrusted user input

**Blocked patterns include:**
- `Remove-Item -Recurse -Force` on system paths
- `Format-Volume`, `Clear-Disk`
- `Stop-Computer`, `Restart-Computer`
- Registry modifications to critical hives

## Usage Example

```python
# Agent uses pwsh tool
result = await session.call_tool("pwsh", {
    "command": "Get-ChildItem -Path . -Recurse | Measure-Object"
})
```

## Troubleshooting

### "PowerShell not found"

Install PowerShell Core:
```powershell
winget install Microsoft.PowerShell
```

### "Module not found" 

Ensure the module source is configured:
```powershell
amplifier module source set tool-pwsh git+https://github.com/Anokye-Labs/amplifier-module-tool-pwsh --scope global
```

### Commands timing out

Increase the timeout in your bundle config:
```yaml
modules:
  tools:
    - id: tool-pwsh
      config:
        timeout: 120  # 2 minutes
```

## Dependencies

- `amplifier-core>=1.0.0`

## Contributing

> [!NOTE]
> This project is not currently accepting external contributions, but we're actively working toward opening this up.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).

## Trademarks

This project may contain trademarks or logos for projects, products, or services. Authorized use of Microsoft trademarks or logos is subject to and must follow [Microsoft's Trademark & Brand Guidelines](https://www.microsoft.com/legal/intellectualproperty/trademarks/usage/general).
