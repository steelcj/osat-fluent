---
status: Proposed
date: 2026-08-06
version: 0.1.0
---

# ADR-001: install- vs manage- naming for fluent tool CLIs

## Context

The tool creation pattern (osat--tool-creation-pattern-v0-1-0.md, section 4) documents one CLI shape: an idempotent installer that fetches the latest release, verifies it, and exits. Section 10's creation steps and every worked example (`install-hugo.py`, `install-rclone.py`) assume this shape. The naming convention `install-<tool>.py` follows directly from it — the script installs, nothing more.

`install-sat.py` already does more than this. Its own `--help` text describes it as managing "user-space installations of SAT Tools," and its interface bears that out:

```text
usage: install-sat.py [-h] [--install [VERSION]] [--switch VERSION]
                       [--status] [--remove VERSION] [--version]
```

Multiple versions coexist on disk. `--switch` repoints the active version without a reinstall. `--status` reports what's installed and what's active. `--remove` deletes a specific version. This is version-lifecycle management, not a single idempotent fetch — the filename says `install-`, the tool's own description and behaviour say `manage-`.

The mismatch surfaced while scoping a new tool for Cboard (osat-fluent-cboard-tool). Cboard is distributed as a pinned, checksummed build artifact rather than a binary (see the Cboard feasibility discussion), but it shares SAT's actual requirement: an operator needs to keep more than one built version on disk and switch between them — for rollback without rebuilding, and for comparing versions during development. A tool built to that requirement is shape 2 from the start, not shape 1 with lifecycle flags bolted on later. Naming it `install-cboard.py` would repeat the same inaccuracy `install-sat.py` already carries.

## Decision

Two CLI shapes are recognised, each with its own naming rule.

**Shape 1 — install-only.** Idempotent: installs the latest release if not already active, otherwise does nothing. No version switching, no removal, no status beyond what installation itself reports. This is the shape section 4 of the tool creation pattern already documents in full. Filename: `install-<tool>.py`. Examples: `install-hugo.py`, `install-rclone.py`.

**Shape 2 — version-managed.** Multiple versions coexist on disk by design. The CLI exposes explicit lifecycle operations:

```text
--install [VERSION]   install a version (default: latest release)
--switch VERSION       point the active pointer at an already-installed version
--status                show installed versions and the active one
--remove VERSION       remove an installed version
--version               show this manager's version
```

Filename: `manage-<tool>.py`. `install-sat.py` is shape 2 in behaviour today; osat-fluent-cboard-tool is designed as shape 2 from the outset.

**The test for which shape a new tool needs:** does an operator ever need to hold two versions side by side and choose between them without re-fetching — for rollback, comparison, or staged rollout? If yes, shape 2, name accordingly. If the tool only ever needs "the latest, verified, or nothing changes," shape 1 is simpler and the honest name for it.

This ADR does not resolve whether `install-sat.py` itself is renamed. That is a separate decision affecting a tool already in use, left open here.

## Alternatives Considered

**Keep `install-<tool>.py` for every tool regardless of shape.** Rejected — already inaccurate for `install-sat.py`, and would compound the same inaccuracy in every future shape-2 tool, including cboard-tool.

**Use `manage-<tool>.py` universally, even for shape-1 tools.** Rejected — overstates what shape-1 tools do. `install-hugo.py` cannot switch or remove versions; naming it as a manager would promise a capability it doesn't have.

**Name by artefact instead of behaviour** (e.g. `cboard-instance`, `ephemeral-cboard.py`, considered during the Cboard scoping discussion). Rejected — these name the *running thing* (a server process, an instance in use), which collides with the separate, already-settled naming of the runtime wrapper (`cboard`) itself. The installer/manager script and the thing it installs need distinct names; conflating them was the actual problem with `cboard-instance`, not a shape-1/shape-2 question.

## Consequences

- osat-fluent-cboard-tool's manager script is named `manage-cboard.py` from creation, not `install-cboard.py`.
- `install-sat.py` remains a known, named mismatch between filename and behaviour. Renaming it is out of scope here and left as a follow-on decision for the SAT project.
- osat--tool-creation-pattern-v0-1-0.md section 4 documents shape 1 only. A shape-2 companion section (the `--install`/`--switch`/`--status`/`--remove` contract, side-by-side version storage, active-pointer mechanics) is needed before other tool authors can follow shape 2 consistently. Not written here.
- This is a naming and CLI-shape distinction, not an archetype distinction. Both shapes can apply within archetype 5 (self-contained binary) or the pinned-source-artefact variant scoped for Cboard; the archetype governs what gets fetched and how it's verified, this ADR governs what the manager script is called and what its interface promises.

## References

- osat--tool-creation-pattern-v0-1-0.md — section 4 (shape 1, as currently documented)
- osat-fluent--archetype-5--self-contained-binary-v0-1-1.md
- install-sat.py — reference implementation of shape 2 in current use
