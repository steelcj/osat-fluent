# osat-fluent

The governing repository for the OSAT (Operating System Sovereign Autonomous Tools) collection.

## What OSAT means

**Operating System** — the OS is central, not abstracted. Each platform is addressed on its own terms.

**Sovereign** — no external dependencies, no package manager, no proprietary services required. Answers to no higher authority. Self-contained and portable across providers and environments.

**Autonomous** — works offline, archive-first resolution, does not depend on network availability or upstream services to function.

**Tools** — a collection built to a common pattern, governed by a shared specification.

## The defining principle

**Fluent** — platform-native in every dimension. Paths, permissions, wrappers, and conventions follow what each platform expects from user-space software. We do not impose a false universal — we speak each platform's language.

## Design principles

**Accessible** — no elevation required, no package manager prerequisite, no proprietary toolchain. Works on Windows, macOS, and Linux including older hardware and unreliable networks. Every decision is documented so the work remains legible to future maintainers. The tools remain reachable even when external services are not.

**Secured** — owner-only permissions throughout, credential isolation, checksum verification on every download, root guard, no speculative access. Security is not assumed — it is applied deliberately at every path decision, every permission, and every installer behaviour.

**Transparent** — every decision is documented with its rationale and the alternatives considered. Nothing is hidden. The work remains legible to a future maintainer who was not present when the decisions were made.

## Repository layout

```
osat-fluent/
├── README.md
├── CONTRIBUTORS.md
├── VERSION
└── en/
    ├── docs/
    │   ├── osat--initial-commit-push-and-versioning-workflow-v0-1-0.md
    │   ├── osat--tool-creation-pattern-v0-1-0.md
    │   ├── osat--user-space-installation-specification-v0-2-0.md
    │   ├── restic-cross-platform-filesystem-layout-specification-v0-1-0.md
    │   ├── restic-platform-paths-spec-v0-1-0.md
    │   ├── restic-server-paths-spec-v0-1-0.md
    │   ├── software-installation-archetypes-v0-1-0.md
    │   └── style-guide--versioned-documents-in-unrendered-markdown-v0-1-0.md
    └── utils/
        └── create-repo-dirs.py
```

## The collection

Each tool in the OSAT collection is a separate repository. Eventually they will all implements this specification and be renamed:

These older tool style repos still function well on nix environments.

- [hugo-tool](https://github.com/steelcj/hugo-tool) — Hugo static site generator
- [rclone-tool](https://github.com/steelcj/rclone-tool) — rclone cloud storage sync
- [marp-tool](https://github.com/steelcj/marp-tool) — Marp CLI presentation tool
- [restic-tool](https://github.com/steelcj/restic-tool) — restic backup tool
- [osat-tool](https://github.com/steelcj/osat-tool) — OSAT pattern reference implementation

## Starting a new tool

```
python3 en/utils/create-repo-dirs.py <tool-name>
```

Then follow the ten-step checklist in `en/docs/osat--tool-creation-pattern`.

## Languages

- [English](README.md)

## License

This repository, *osat-fluent*, by **Christopher Steel**, with AI assistance from **Claude Sonnet 4.6 (Anthropic)**, is licensed under the [GNU General Public License v3.0 or later](https://www.gnu.org/licenses/gpl-3.0.html).
