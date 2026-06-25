# OSAT — User-Space Installation Specification

Version: 0.2.0
Status: Draft
Style Guide: style-guide--technical-documentation-for-technologists-v0.2.0

## Abstract

This document defines the target filesystem layout for all OS-Agnostic Tools (OSATs). It establishes platform-native paths for binaries, wrappers, configuration, credentials, state, and logs across Linux, macOS, and Windows. It also defines which software installation archetypes OSAT covers, which it deliberately does not, and why. It supersedes the earlier `~/bin/` convention used in hugo-tool, rclone-tool, marp-tool, and restic-tool.

## 1. Scope and archetype coverage

OSAT covers one installation archetype: **User XDG / Manual User Install** (archetype 5 in the companion archetypes document). This archetype has four defining properties: the install requires no elevation, the tool lives entirely within the invoking user's home directory, the tool is self-contained or manages its own isolation, and there is no dependency on an external package manager.

The six archetypes and their relationship to OSAT are as follows.

**Archetype 1 — System Package.** Installed by the OS package manager into system-wide paths. Requires root or admin. Outside OSAT scope — OSAT does not install to system paths and does not require elevation.

**Archetype 2 — System Local.** Installed by an admin into `/usr/local/` or equivalent. Requires elevation. Outside OSAT scope for the same reason.

**Archetype 3 — Service Deployment.** Installed to run as a background daemon under a dedicated service account. Requires elevation to configure. Outside OSAT scope. The companion `restic-server-paths-spec` covers this archetype for restic specifically.

**Archetype 4 — Package Manager (User-Scoped).** Managed by Homebrew, Scoop, winget, or similar. No elevation required, but depends on the package manager being present. Outside OSAT scope — OSAT has no external dependencies beyond Python 3.8 and network access. A user who already has Homebrew or Scoop may prefer those; OSAT does not conflict with them.

**Archetype 5 — User XDG / Manual User Install.** Self-contained binary, no elevation, no package manager, lives in the user's home directory. This is OSAT's archetype. All tools in the collection fit here: hugo-tool, rclone-tool, marp-tool, restic-tool, osat-tool.

**Archetype 6 — Isolated User (Runtime-Dependent).** Used when the tool has a language runtime dependency — Python, Node, Ruby. The tool and its dependencies are sandboxed into an isolated environment. OSAT supports a variant of this archetype for Python tools via an embedded virtual environment managed by the installer. The tool appears on PATH as a plain binary; the venv is an implementation detail. `tool-python-slugify` is the reference implementation of this variant.

## 2. Design principles

Every path decision in this specification follows from six principles, in order of priority.

**No elevation required.** The installer never calls sudo, never writes to system paths, and refuses to run as root. Any path that requires elevation is the wrong path.

**Principle of least privilege.** Every path in the OSAT layout is granted the minimum permissions required for the current, known use case — no more. User-space tools are installed for and operated by a single user; no group or world access is granted by default. If a future use case genuinely requires broader access — a group for automation, a shared credential — that requirement drives a specific, reasoned, documented change to that specific tool. It does not change the default for everything else, and it is never granted speculatively.

**Platform-native paths.** Each platform has established conventions for where user-space executables, configuration, state, and credentials live. We follow those conventions rather than imposing a single cross-platform path that is native to none of them.

**XDG compliance on Linux and macOS.** The XDG Base Directory Specification is the authoritative standard for user-space file locations on Linux. macOS CLI tools have converged on the same paths. We follow XDG defaults and respect overrides via the standard environment variables.

**Intentional deviation for credentials.** No OS standard defines a secure credential location for user-space CLI tools. We use `~/.private/<tool>/` as a deliberate non-standard convention. It is less likely to be synced by cloud tools than `~/.config/` and signals clearly that its contents are sensitive.

**Single PATH entry per platform.** All OSAT wrappers on a given platform install to the same directory. The user adds one directory to PATH; all OSAT tools become available. On Linux and macOS that directory is `~/.local/bin/`. On Windows it is `%LOCALAPPDATA%\Programs\`.

**Environment configuration travels with the tool.** The wrapper sources a per-tool env file at runtime rather than requiring the user to set variables in their shell profile. This works in non-interactive contexts — systemd user timers, launchd agents, Task Scheduler — without additional configuration.

## 3. Target layout

### 3.1 Linux and macOS

```
~
├── .local/
│   ├── bin/
│   │   └── <tool-name>                    ← wrapper (rendered by installer)
│   ├── share/
│   │   └── <tool-name>/
│   │       └── <version>/
│   │           └── <binary>               ← versioned binary (git-ignored)
│   └── state/
│       └── <tool-name>/                   ← persistent state and logs
│
├── .config/
│   └── <tool-name>/
│       └── env                            ← sourced by wrapper at runtime
│
└── .private/
    └── <tool-name>/                       ← credentials (chmod 700)
```

XDG environment variable overrides are respected. Scripts always fall back to XDG defaults explicitly:

```sh
DATA_HOME="${XDG_DATA_HOME:-$HOME/.local/share}"
BIN_HOME="${XDG_BIN_HOME:-$HOME/.local/bin}"
CONFIG_HOME="${XDG_CONFIG_HOME:-$HOME/.config}"
STATE_HOME="${XDG_STATE_HOME:-$HOME/.local/state}"
```

### 3.2 Windows

```
%USERPROFILE%
├── AppData\
│   ├── Local\
│   │   ├── Programs\
│   │   │   └── <tool-name>.cmd            ← cmd.exe wrapper (rendered by installer)
│   │   │   └── <tool-name>.ps1            ← PowerShell wrapper (rendered by installer)
│   │   └── <tool-name>\
│   │       ├── <version>\
│   │       │   └── <binary>.exe           ← versioned binary
│   │       └── logs\                      ← persistent state and logs
│   └── Roaming\
│       └── <tool-name>\
│           └── env.ps1                    ← sourced by wrapper at runtime
└── .private\
    └── <tool-name>\                       ← credentials (ACL: owner only)
```

The installer resolves Windows paths from environment variables with explicit fallbacks:

```python
local   = Path(os.environ.get("LOCALAPPDATA",
               Path.home() / "AppData" / "Local"))
appdata = Path(os.environ.get("APPDATA",
               Path.home() / "AppData" / "Roaming"))
```

### 3.3 Platform path translation

| Concept | Linux | macOS | Windows |
|---------|-------|-------|---------|
| Wrapper | `~/.local/bin/<tool-name>` | `~/.local/bin/<tool-name>` | `%LOCALAPPDATA%\Programs\<tool-name>.cmd` |
| Versioned binary | `~/.local/share/<tool-name>/<version>/<binary>` | `~/.local/share/<tool-name>/<version>/<binary>` | `%LOCALAPPDATA%\<tool-name>\<version>\<binary>.exe` |
| Config / env | `~/.config/<tool-name>/env` | `~/.config/<tool-name>/env` | `%APPDATA%\<tool-name>\env.ps1` |
| Credentials | `~/.private/<tool-name>/` | `~/.private/<tool-name>/` | `%USERPROFILE%\.private\<tool-name>\` |
| State / logs | `~/.local/state/<tool-name>/` | `~/.local/state/<tool-name>/` | `%LOCALAPPDATA%\<tool-name>\logs\` |

## 4. Permissions

All permissions follow the principle of least privilege: owner-only throughout. No group or world access is granted at any path.

| Path | Linux / macOS | Windows |
|------|---------------|---------|
| `~/.local/bin/<tool-name>` | `700` | Owner only (ACL) |
| `~/.local/share/<tool-name>/` | `700` | Owner only (ACL) |
| `~/.local/share/<tool-name>/<version>/<binary>` | `700` | Owner only (ACL) |
| `~/.config/<tool-name>/` | `700` | Owner only (ACL) |
| `~/.config/<tool-name>/env` | `600` | Owner read only (ACL) |
| `~/.private/` | `700` | Owner only (ACL) |
| `~/.private/<tool-name>/` | `700` | Owner only (ACL) |
| `~/.private/<tool-name>/<credential>` | `600` | Owner read only (ACL) |
| `~/.local/state/<tool-name>/` | `700` | Owner only (ACL) |

The installer sets these permissions on creation and verifies them on every subsequent run. If any path is found with permissions broader than specified, the installer fails explicitly rather than proceeding silently.

## 5. The wrapper

The wrapper is a rendered shell script (nix) or `.cmd`/`.ps1` pair (Windows). It has two responsibilities: sourcing the env file if it exists, and exec-ing the versioned binary. It is rendered by the installer from a template in `scripts/nix/<tool-name>` or `scripts/windows/<tool-name>.cmd` and `.ps1`.

### 5.1 Nix wrapper template

```sh
#!/bin/sh
# <tool-name> — rendered by install-<tool-name>.py; do not edit by hand
_cfg="${XDG_CONFIG_HOME:-$HOME/.config}/<tool-name>/env"
[ -f "$_cfg" ] && . "$_cfg"
_bin="${XDG_DATA_HOME:-$HOME/.local/share}/<tool-name>/__TOOL_VERSION__/<binary>"
exec "$_bin" "$@"
```

### 5.2 Windows cmd wrapper template

```batch
@echo off
rem <tool-name>.cmd — rendered by install-<tool-name>.py; do not edit by hand
set "_cfg=%APPDATA%\<tool-name>\env.cmd"
if exist "%_cfg%" call "%_cfg%"
set "_bin=%LOCALAPPDATA%\<tool-name>\__TOOL_VERSION__\<binary>.exe"
"%_bin%" %*
```

### 5.3 Windows PowerShell wrapper template

```powershell
# <tool-name>.ps1 — rendered by install-<tool-name>.py; do not edit by hand
$_cfg = "$env:APPDATA\<tool-name>\env.ps1"
if (Test-Path $_cfg) { . $_cfg }
$_bin = "$env:LOCALAPPDATA\<tool-name>\__TOOL_VERSION__\<binary>.exe"
& $_bin @args
```

The placeholder `__TOOL_VERSION__` is replaced by the installer with the active version string at render time. The wrapper is idempotent: if the rendered content already matches the template with the current version substituted, the installer does not rewrite it.

## 6. The env file

The env file is not created by the installer. It is created by the user or by a separate configuration script. The wrapper sources it at runtime if it exists and silently skips it if it does not. This means a freshly installed tool works without configuration — the user adds the env file when they have values to set.

The env file format on nix is a POSIX sh fragment:

```sh
export RESTIC_REPOSITORY="sftp:user@host:/mnt/backup/restic/hostname"
export RESTIC_PASSWORD_FILE="$HOME/.private/restic-tool/password.txt"
```

On Windows the `.cmd` wrapper sources a `.cmd` env file and the `.ps1` wrapper sources a `.ps1` env file. Both live in `%APPDATA%\<tool-name>\`.

## 7. Archetype 6 variant — Python tools

Python tools in the OSAT collection embed a virtual environment managed by the installer. The venv lives in `~/.local/share/<tool-name>/venv/` on nix and `%LOCALAPPDATA%\<tool-name>\venv\` on Windows. The wrapper invokes the venv's Python interpreter directly rather than the system Python, so the tool is isolated from system packages and other venvs.

```sh
#!/bin/sh
# <tool-name> — rendered by install-<tool-name>.py; do not edit by hand
_cfg="${XDG_CONFIG_HOME:-$HOME/.config}/<tool-name>/env"
[ -f "$_cfg" ] && . "$_cfg"
_python="${XDG_DATA_HOME:-$HOME/.local/share}/<tool-name>/venv/bin/python"
exec "$_python" -m <tool_module> "$@"
```

The version tracked in the installer and reflected in idempotency checks is the package version within the venv, not the Python version.

## 8. PATH configuration

### 8.1 Linux

Modern Ubuntu, Debian, and Fedora installs add `~/.local/bin/` to PATH automatically via `~/.profile` if the directory exists. The installer creates the directory; the user's next login picks it up. The installer warns if `~/.local/bin/` is not in the current PATH.

### 8.2 macOS

macOS does not add `~/.local/bin/` to PATH automatically. The installer warns and prints the export line to add to `~/.zshrc` or `~/.bash_profile`:

```sh
export PATH="$HOME/.local/bin:$PATH"
```

### 8.3 Windows

`%LOCALAPPDATA%\Programs\` is not on PATH by default. The installer warns and prints instructions for adding it via System Properties or PowerShell:

```powershell
[Environment]::SetEnvironmentVariable(
    "PATH",
    "$env:LOCALAPPDATA\Programs;" + [Environment]::GetEnvironmentVariable("PATH","User"),
    "User"
)
```

This change takes effect in new shell sessions without requiring a reboot.

## 9. Repository layout

Every OSAT repository follows this layout. The layout is the same regardless of the tool or platform.

```
<tool-name>/
├── .gitignore              versioned binary dirs excluded automatically
├── README.md               install, upgrade, rollback, layout sections
├── ROADMAP.md              planned platform bringup and known gaps
├── create-repo-dirs.py     creates the repository directory skeleton
├── install-<tool-name>.py  the installer
└── scripts/
    ├── nix/
    │   └── <tool-name>     nix wrapper template
    └── windows/
        ├── <tool-name>.cmd Windows cmd wrapper template
        └── <tool-name>.ps1 Windows PowerShell wrapper template
```

## 10. Decisions and rationale

### 10.1 Why move from ~/bin/ to ~/.local/bin/

`~/bin/` is a longstanding Unix convention and works on Linux and macOS. It has no Windows equivalent. `~/.local/bin/` is XDG-compliant, is added to PATH automatically by modern Linux distributions, is used by the macOS CLI tool ecosystem, and maps cleanly to `%LOCALAPPDATA%\Programs\` on Windows — both being the platform-native location for user-installed executables that require no elevation. Adopting `~/.local/bin/` eliminates the only deviation from the XDG standard in the previous layout and gives Windows a proper native equivalent.

We considered keeping `~/bin/` as a deliberate cross-platform root. We rejected it because the cost of false universality is higher than the cost of platform-native paths handled by the installer. The `platform_paths()` function in each installer absorbs the platform difference once; every other part of the tool is unaffected.

### 10.2 Why ~/.local/share/ for versioned binaries

The versioned binary store — `~/.local/share/<tool-name>/<version>/` — is application data under XDG. It is not an executable on PATH, not configuration, not state. `$XDG_DATA_HOME` (`~/.local/share/`) is the correct location. On Windows, `%LOCALAPPDATA%` is the machine-local equivalent.

We considered placing versioned binaries under `~/.local/bin/<tool-name>/` to keep all tool files in one subtree. We rejected it because executables in subdirectories of `~/.local/bin/` are not on PATH and the directory would accumulate non-wrapper content, blurring the single-purpose role of `~/.local/bin/` as the wrapper directory.

### 10.3 Why ~/.local/state/ for logs

XDG added `$XDG_STATE_HOME` (`~/.local/state/`) specifically for persistent state that is not important enough to live in `$XDG_DATA_HOME` — logs, history, application state across restarts. Logs are state, not data. The earlier layout used `~/.local/share/` for logs, which was wrong per the standard.

### 10.4 Why ~/.private/ for credentials

No OS standard defines a secure credential location for user-space CLI tools. `~/.config/` is the XDG config home but is routinely synced by dotfile managers and cloud tools. `~/.private/` is intentionally non-standard. It is unlikely to be touched by sync tools, clearly named, and consistently placed across all three platforms. The installer creates it with `700` permissions and checks those permissions on subsequent runs.

### 10.5 Why the wrapper sources the env file rather than the shell profile

Sourcing environment variables from `~/.bashrc` or `~/.zshrc` works only in interactive login shells. Scheduled jobs — systemd user timers, launchd agents, Task Scheduler tasks — do not source shell profiles. The wrapper sources the env file unconditionally at every invocation, making configuration consistent across interactive and non-interactive contexts.

### 10.6 Why owner-only permissions throughout

User-space tools are installed for and operated by a single user. No other user, group, or process on the machine has a legitimate need to read, write, or execute files in the OSAT layout. Granting group or world access — even read-only — creates unnecessary exposure: a misconfigured or compromised process running as another user in the same group could read configuration, credentials, or invoke binaries it has no business touching.

The principle of least privilege requires that we grant the minimum access required for the current, known use case. The current, known use case is a single user invoking their own tools. `700` for directories and executables, `600` for files, no group or world bits set anywhere.

If a future use case genuinely requires group access — an automation account in the same group, a shared credential for a team workflow — that requirement deserves its own explicit, documented decision scoped to the specific tool and path that needs it. It does not justify opening the default permissions for everything else. Speculative permissions granted "just in case" are the opposite of least privilege.

This document, *OSAT — User-Space Installation Specification*, by **Christopher Steel**, with AI assistance from **Claude Sonnet 4.6 (Anthropic)**, is licensed under the [GNU General Public License v3.0 or later](https://www.gnu.org/licenses/gpl-3.0.html).

## Changelog

| Version | Status | Notes |
|---------|--------|-------|
| 0.2.0 | Draft | Applied principle of least privilege to all permissions — 700/600 owner-only throughout; added principle of least privilege to design principles (section 2); added rationale section 10.6 |
| 0.1.0 | Draft | Initial draft — supersedes ~/bin/ convention; establishes XDG-compliant layout across Linux, macOS, and Windows; covers all six installation archetypes |
