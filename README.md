# amplifier-module-tool-pwsh

PowerShell command execution for [Amplifier](https://github.com/microsoft/amplifier). The Windows-native equivalent of `tool-bash` — preferred over bash on Windows platforms.

## Installation

### As an app bundle (recommended for Windows users)

Installs the tool with Windows shell guidance, Unix-to-PowerShell equivalents context, and the full PowerShell skill:

```bash
amplifier bundle add git+https://github.com/colombod/amplifier-module-tool-pwsh@main --app
```

### As a composable behavior (add to existing bundle)

Layers PowerShell support onto any foundation-based bundle:

```bash
amplifier bundle add git+https://github.com/colombod/amplifier-module-tool-pwsh@main#subdirectory=behaviors/windows-shell.yaml --app
```

### As a raw module (advanced)

Add to your bundle's `tools:` section:

```yaml
tools:
  - module: tool-pwsh
    source: git+https://github.com/colombod/amplifier-module-tool-pwsh@main
    config:
      safety_profile: standard
```

## Prerequisites

- **PowerShell 7+** (`pwsh`) — [Install PowerShell](https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell)

```powershell
# Windows
winget install Microsoft.PowerShell

# macOS
brew install powershell/tap/powershell

# Linux (Ubuntu/Debian)
sudo apt-get install -y powershell
```

## What's Included

| Component | Purpose |
|-----------|---------|
| `amplifier_module_tool_pwsh/` | Python tool module (`tool-pwsh`) |
| `bundle.md` | Standalone bundle entry point |
| `behaviors/windows-shell.yaml` | Composable behavior (tool + context) |
| `context/windows-shell.md` | Always-injected Unix-to-PowerShell quick reference |
| `skills/powershell-windows-dev/` | On-demand comprehensive PowerShell skill |
| `tests/` | 53 tests (behavioral, structural, safety, execution) |

## Tool API

### `pwsh`

Execute a PowerShell command.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `command` | string | Yes | — | PowerShell command to execute |
| `timeout` | integer | No | 30 | Timeout in seconds. Increase for builds/tests. |
| `run_in_background` | boolean | No | false | Fire-and-forget mode, returns PID immediately |

**Output:**
```json
{
  "stdout": "...",
  "stderr": "...",
  "returncode": 0
}
```

Output is automatically truncated at ~100KB (40% head + 40% tail) with byte-level fallback for single-line output.

## Safety System

Four configurable profiles control what commands are allowed:

| Profile | Blocks | Allowlist Override |
|---------|--------|-------------------|
| **strict** (default) | Disk ops, shutdown, recursive system deletes, privilege escalation, registry hive ops, execution policy changes, fork bombs, pipe-to-Invoke-Expression | No |
| **standard** | Same as strict | Yes |
| **permissive** | Root directory deletion, fork bombs only | Yes |
| **unrestricted** | Nothing | Yes |

Command-position detection is PowerShell-aware — `Get-Help Format-Volume` is allowed (Format-Volume is an argument), but `Format-Volume -DriveLetter C` is blocked (command position).

Configure in your bundle:
```yaml
tools:
  - module: tool-pwsh
    source: git+https://github.com/colombod/amplifier-module-tool-pwsh@main
    config:
      safety_profile: standard      # strict | standard | permissive | unrestricted
      timeout: 30
      allowed_commands: []           # Allowlist patterns (wildcards supported)
      denied_commands: []            # Extra blocklist patterns
      safety_overrides:              # Fine-grained overrides
        allow: []
        block: []
```

## Platform Behavior

| Platform | Shell | Fallback |
|----------|-------|----------|
| Windows | `pwsh` (PowerShell Core) | `powershell` (Windows PowerShell 5.1) |
| Linux/macOS | `pwsh` (if installed) | Error with install instructions |

## Example Prompts

| Prompt | What the agent runs |
|--------|--------------------|
| "List all running services" | `Get-Service \| Where-Object Status -eq Running` |
| "Find large files in Downloads" | `Get-ChildItem ~/Downloads -Recurse \| Sort-Object Length -Descending \| Select-Object -First 10` |
| "Show disk space" | `Get-PSDrive -PSProvider FileSystem` |
| "Check network config" | `Get-NetIPConfiguration` |
| "Install Python via winget" | `winget install Python.Python.3.12` |

## Related Bundles

These bundles build on `tool-pwsh` for Windows development:

| Bundle | Purpose | Install |
|--------|---------|---------|
| [winget-ops](https://github.com/colombod/amplifier-bundle-winget-ops) | Windows Package Manager agent | `amplifier bundle add git+https://github.com/colombod/amplifier-bundle-winget-ops@main --app` |
| [dotnet-ops](https://github.com/colombod/amplifier-bundle-dotnet-ops) | Cross-platform .NET CLI agent | `amplifier bundle add git+https://github.com/colombod/amplifier-bundle-dotnet-ops@main --app` |

## Development

```bash
# Clone and set up
git clone https://github.com/colombod/amplifier-module-tool-pwsh.git
cd amplifier-module-tool-pwsh
uv venv && source .venv/bin/activate
uv pip install -e .
uv pip install pytest pytest-asyncio "amplifier-core @ git+https://github.com/microsoft/amplifier-core@main"

# Run tests (requires pwsh installed)
pytest tests/ -v
```

## License

MIT
