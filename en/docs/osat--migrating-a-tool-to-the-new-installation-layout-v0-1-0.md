# OSAT -- Migrating a Tool to the New Installation Layout

Version: 0.1.0
Status: Draft
Style Guide: style-guide--versioned-documents-in-unrendered-markdown-v0-1-0

## Abstract

This document describes how to migrate an existing OSAT tool from the original `~/bin/` convention to the XDG-compliant layout defined in `osat--user-space-installation-specification-v0-2-0`. It covers the changes required to the installer, the wrapper templates, and the README, and explains what existing users of the tool must do after the updated installer is published.

## 1. What changes and what does not

The migration touches paths and permissions only. The installer logic, including version detection, checksum verification, archive extraction, binary health check, and idempotency, does not change. The wrapper template structure does not change. The repository layout does not change. Only where things are installed and how permissions are set is different.

**What changes:**

- Versioned binary location, from `~/bin/<tool-name>/<version>/` to `~/.local/share/<tool-name>/<version>/` on nix, and to `%LOCALAPPDATA%\<tool-name>\<version>\` on Windows
- Wrapper location, from `~/bin/<tool-name>` to `~/.local/bin/<tool-name>` on nix, and to `%LOCALAPPDATA%\Programs\<tool-name>.cmd` and `.ps1` on Windows
- Permissions, from `755` on directories and executables to `700` throughout, with `600` on all files
- Wrapper template, gains an env file sourcing stanza
- PATH advisory, updated to reference `~/.local/bin/` instead of `~/bin/`

**What does not change:**

- The installer filename, `install-<tool-name>.py`
- The wrapper template filename and placeholder convention
- The `.gitignore` pattern
- The idempotency, root guard, and health check logic
- The checksum verification approach
- The repository structure

## 2. Installer changes

### 2.1 Replace path constants with platform_paths()

The old layout used module-level constants for all paths. Replace them with a single `platform_paths()` function that returns a dictionary of resolved paths for the current platform. This function is called once in `main()` and all subsequent path references use its output.

```python
def platform_paths() -> dict:
    home = Path.home()
    system = platform.system()

    if system == "Windows":
        local = Path(os.environ.get(
            "LOCALAPPDATA", home / "AppData" / "Local"))
        appdata = Path(os.environ.get(
            "APPDATA", home / "AppData" / "Roaming"))
        return {
            "tool_dir":    local / "<tool-name>",
            "wrapper_dir": local / "Programs",
            "wrapper":     local / "Programs" / "<tool-name>.cmd",
            "config_dir":  appdata / "<tool-name>",
            "state_dir":   local / "<tool-name>" / "logs",
            "private_dir": home / ".private" / "<tool-name>",
        }
    else:
        data  = Path(os.environ.get(
            "XDG_DATA_HOME",  home / ".local" / "share"))
        bin_  = Path(os.environ.get(
            "XDG_BIN_HOME",   home / ".local" / "bin"))
        cfg   = Path(os.environ.get(
            "XDG_CONFIG_HOME", home / ".config"))
        state = Path(os.environ.get(
            "XDG_STATE_HOME", home / ".local" / "state"))
        return {
            "tool_dir":    data  / "<tool-name>",
            "wrapper_dir": bin_,
            "wrapper":     bin_  / "<tool-name>",
            "config_dir":  cfg   / "<tool-name>",
            "state_dir":   state / "<tool-name>",
            "private_dir": home  / ".private" / "<tool-name>",
        }
```

Replace `<tool-name>` with the actual tool name throughout. The `private_dir` and `state_dir` entries are included for completeness even if the tool does not currently use them, so the function is consistent across the collection.

### 2.2 Update permissions

Replace all `stat.S_IRWXU | stat.S_IRGRP | stat.S_IXGRP | stat.S_IROTH | stat.S_IXOTH` permission sets (`755`) with owner-only permissions following the principle of least privilege.

For directories and executables:

```python
path.chmod(stat.S_IRWXU)  # 700 — owner read, write, execute only
```

For files:

```python
path.chmod(stat.S_IRUSR | stat.S_IWUSR)  # 600 — owner read, write only
```

Apply this to every `chmod` call in the installer, including the wrapper, the binary, and any directories created.

### 2.3 Add permission verification

Add a check in `main()` that verifies the wrapper and binary have the expected permissions on subsequent runs. If a path is found with permissions broader than specified, fail explicitly.

```python
def check_permissions(path: Path, expected_mode: int) -> None:
    actual_mode = path.stat().st_mode & 0o777
    if actual_mode != expected_mode:
        fail(
            f"{path} has permissions {oct(actual_mode)}, "
            f"expected {oct(expected_mode)}; "
            f"correct with: chmod {oct(expected_mode)[2:]} {path}"
        )
```

### 2.4 Update the wrapper guard

The old installer checked whether `~/bin/<tool-name>` exists but is not a regular file. Update this check to reference the new wrapper path from `platform_paths()`.

## 3. Wrapper template changes

### 3.1 Nix wrapper

Add the env file sourcing stanza above the exec line. The full updated template:

```sh
#!/bin/sh
# <tool-name> — rendered by install-<tool-name>.py; do not edit by hand
_cfg="${XDG_CONFIG_HOME:-$HOME/.config}/<tool-name>/env"
[ -f "$_cfg" ] && . "$_cfg"
_bin="${XDG_DATA_HOME:-$HOME/.local/share}/<tool-name>/__TOOL_VERSION__/<binary>"
exec "$_bin" "$@"
```

Replace `<tool-name>`, `__TOOL_VERSION__`, and `<binary>` with the actual values for the tool.

### 3.2 Windows cmd wrapper

```batch
@echo off
rem <tool-name>.cmd — rendered by install-<tool-name>.py; do not edit by hand
set "_cfg=%APPDATA%\<tool-name>\env.cmd"
if exist "%_cfg%" call "%_cfg%"
set "_bin=%LOCALAPPDATA%\<tool-name>\__TOOL_VERSION__\<binary>.exe"
"%_bin%" %*
```

### 3.3 Windows PowerShell wrapper

```powershell
# <tool-name>.ps1 — rendered by install-<tool-name>.py; do not edit by hand
$_cfg = "$env:APPDATA\<tool-name>\env.ps1"
if (Test-Path $_cfg) { . $_cfg }
$_bin = "$env:LOCALAPPDATA\<tool-name>\__TOOL_VERSION__\<binary>.exe"
& $_bin @args
```

## 4. README changes

### 4.1 Install section

Update the install instructions to reference `~/.local/bin/` instead of `~/bin/`:

```markdown
If `<tool-name>` is not found after install, open a new terminal. On Ubuntu,
`~/.local/bin/` is added to PATH automatically at next login. On macOS, add
the following to `~/.zshrc` or `~/.bash_profile`:

    export PATH="$HOME/.local/bin:$PATH"
```

### 4.2 Layout section

Update the directory tree to reflect the new paths:

```markdown
~/.local/bin/<tool-name>                          wrapper (rendered by installer)
~/.local/share/<tool-name>/                       versioned binaries
~/.local/share/<tool-name>/<version>/<binary>     installed binary (not in repo)
~/.config/<tool-name>/env                         env file (not created by installer)
```

### 4.3 Migration note

Add a migration note for existing users who have the tool installed under the old `~/bin/` layout. Place it after the Upgrade section:

```markdown
## Migrating from the ~/bin/ layout

If you installed this tool before version X.Y.Z, your binary and wrapper are
under `~/bin/`. Running the updated installer places the new installation under
`~/.local/bin/` and `~/.local/share/<tool-name>/`. The old installation is not
removed automatically.

After running the updated installer:

1. Confirm the new wrapper works: `<tool-name> version`
2. Remove the old wrapper: `rm ~/bin/<tool-name>`
3. Remove the old versioned binaries: `rm -rf ~/bin/<tool-name>/`
4. Add `~/.local/bin/` to PATH if it is not already there
5. If `~/bin/` is now empty and you no longer need it: `rmdir ~/bin/`
```

Replace `X.Y.Z` with the version of the installer that introduces the migration.

## 5. What existing users must do

The updated installer does not touch the old `~/bin/` installation. It installs cleanly to the new paths and leaves the old paths in place. This means after running the updated installer, two versions of the tool coexist on the machine until the user cleans up.

The shell resolves whichever version of the tool appears first in PATH. If `~/bin/` precedes `~/.local/bin/` in PATH, the old wrapper at `~/bin/<tool-name>` will still be invoked. The installer will warn if this is the case, via the existing shadowing check in `post_install_notes()`.

The user must:

1. Run the updated installer
2. Verify the new version is working
3. Remove the old `~/bin/<tool-name>` wrapper manually
4. Remove the old `~/bin/<tool-name>/` versioned binary directories manually
5. Confirm PATH resolves to the new wrapper

The installer does not automate cleanup of old paths. Removing files the user may have deliberately retained is not the installer's responsibility.

## 6. Decisions and rationale

### 6.1 Why the installer does not clean up old paths

The old `~/bin/<tool-name>/` directory contains versioned binaries the user may have retained deliberately for rollback. The installer has no way to know which of those the user considers safe to remove. Automated cleanup of user data, even data the installer itself placed there, violates the principle of least surprise. The migration note in the README gives the user the information they need to clean up themselves.

### 6.2 Why the migration is not versioned separately

The migration is a change to the installer, not a new tool. It is released as a new version of the existing tool following the normal OSAT versioning convention. The README migration note names the version at which the change was introduced so users can identify whether they need to migrate.

## License

This document, *OSAT -- Migrating a Tool to the New Installation Layout*, by **Christopher Steel**, with AI assistance from **Claude Sonnet 4.6 (Anthropic)**, is licensed under the [GNU General Public License v3.0 or later](https://www.gnu.org/licenses/gpl-3.0.html).

## Changelog

| Version | Status | Notes |
|---------|--------|-------|
| 0.1.0 | Draft | Initial draft |
