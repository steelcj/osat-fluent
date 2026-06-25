# OSAT Fluent -- Initial Commit, Push, and Versioning Workflow

Version: 0.1.0
Status: Draft
Style Guide: web-ready-unrendered-markdown-using-apa-7-v0.2.2

## Abstract

This document walks through the initial commit and push of the osat-fluent repository to GitHub, and explains how versioning works in practice for both the repository and the documents it contains. It is written for contributors who are setting up this or some other similar repository for the first time or who need a reference for the versioning workflow on subsequent changes.

## Sources and Acknowledgements

Git conventions in this document follow <a name="apa-git-citation"></a>[Software Freedom Conservancy (2024)](#apa-git-reference). GitHub remote conventions follow <a name="apa-github-citation"></a>[GitHub (2024)](#apa-github-reference). Semantic versioning follows <a name="apa-semver-citation"></a>[Preston-Werner (2024)](#apa-semver-reference).

## 1. Prerequisites

Before the initial commit, confirm the following are in place on your local machine.

Git must be installed and configured with your identity:

```bash
git config --global user.name "FirstName LastName"
git config --global user.email "your@email.com"
```

A GitHub repository named `osat-fluent` must exist under your account at `github.com/steelcj/osat-fluent`. Create it with no README, no .gitignore, and no licence, so it is completely empty. The local repository will push its own initial state.

Your local `something-fluent` directory matches your target structure:

```text
osat-fluent/
├── .gitignore
├── CONTRIBUTORS.md
├── README.md
├── VERSION
└── en/
    ├── docs/
    │   ├── osat--tool-creation-pattern-v0-1-0.md
    │   ├── osat--user-space-installation-specification-v0-2-0.md
    │   ├── restic-cross-platform-filesystem-layout-specification-v0-1-0.md
    │   ├── restic-platform-paths-spec-v0-1-0.md
    │   ├── restic-server-paths-spec-v0-1-0.md
    │   └── software-installation-archetypes-v0-1-0.md
    └── utils/
        └── create-repo-dirs.py
```

## 2. Initialise the local repository

Navigate to your local `osat-fluent` directory and initialise git:

```bash
cd ~/Documents/areas/development/osat-fluent
git init
git branch -M main
git status
```

`git init` creates the `.git` directory. `git branch -M main` names the default branch `main` rather than the legacy `master`.

## 3. Stage and inspect

Stage all files:

```bash
git add .
```

Inspect what will be committed before committing:

```bash
git status
git diff --staged
```

Confirm the file list matches the target structure. Confirm `en/utils/create-repo-dirs.py`, all documents in `en/docs/`, `VERSION`, `README.md`, `CONTRIBUTORS.md`, and `.gitignore` are all staged. Confirm no unintended files are included.

## 4. The initial commit

```bash
git commit -m "Initial commit — osat-fluent v0.1.0

Establishes the osat-fluent governing repository for the OSAT collection.

Documents:
- osat--user-space-installation-specification-v0-2-0
- osat--tool-creation-pattern-v0-1-0
- software-installation-archetypes-v0-1-0
- restic-cross-platform-filesystem-layout-specification-v0-1-0
- restic-platform-paths-spec-v0-1-0
- restic-server-paths-spec-v0-1-0

Tools:
- create-repo-dirs.py

Repository version: 0.1.0"
```

The commit message structure is: a short subject line naming the version, a blank line, a body explaining what the commit contains. The subject line must be 72 characters or fewer. The body has no line length limit but each line should be readable as plain text.

## 5. Connect the remote and push

Add the GitHub remote:

```bash
git remote add origin git@github.com:steelcj/osat-fluent.git
```

Push the initial commit:

```bash
git push -u origin main
```

`-u` sets the upstream tracking reference so subsequent `git push` and `git pull` commands need no arguments.

Confirm the push succeeded by visiting `github.com/steelcj/osat-fluent` and verifying the file structure matches what was committed.

## 6. How repository versioning works in practice

The `VERSION` file at the repository root contains the current repository version as a single line:

```text
0.1.0
```

The version is bumped when a change is published, not when work begins. The rules are:

- **PATCH** — corrections to existing documents, additions or corrections to `utils/`
- **MINOR** — new documents added, existing documents substantially revised
- **MAJOR** — fundamental change in scope, philosophy, or structure; removal of documents

To bump the version, edit `VERSION`, commit the change, and push. The commit message names the new version in the subject line.

Example of a MINOR bump after adding a new document:

```bash
# Edit VERSION: change 0.1.0 to 0.2.0
git add VERSION en/docs/new-document-v0-1-0.md
git commit -m "Add new-document — bump repository to v0.2.0"
git push
```

## 7. How document versioning works in practice

TODO Update this. This is a poorly interpreted AI description...

Every document in `en/docs/` carries its own version in the version block immediately below its H1 heading for documents below version `1.0.0`, once a document reaches version 1.0.0 they are no longer versioned individually using the file name. Until then you do something like this in order to facilitate rapid development until our vesrion becomes major:

```text
Version: 0.2.0
Status: Draft
```

And in its filename:

```text
osat--user-space-installation-specification-v0-2-0.md
```

When a document changes, three things happen together: the version number in the version block is bumped, the `dcterms:modified` date is updated to today, and a changelog entry is added at the top of the changelog table. These three changes are always made together, never separately.

The filename changes to reflect the new version. The old filename is removed and the new filename is added in the same commit:

```bash
git rm en/docs/osat--user-space-installation-specification-v0-2-0.md
git add en/docs/osat--user-space-installation-specification-v0-3-0.md
git add VERSION
git commit -m "Revise installation specification to v0.3.0 — bump repository to v0.2.0"
git push
```

The commit message names both the document version and the repository version so the change is traceable in the git log without opening any files.

## 8. Document status transitions

A document moves through three statuses in its version block:

- **Draft** — actively being written; content may change substantially between versions
- **Review** — content is complete; open for review before stabilisation
- **Stable** — content is settled; changes require a new version with a documented rationale

A status transition is a MINOR version bump. Changing a document from Draft to Review, or Review to Stable, bumps the document version and the repository version.

At `v1.0.0` a document graduates from pre-release. The version suffix is dropped from the filename. The document `osat--user-space-installation-specification-v0-9-0.md` becomes `osat--user-space-installation-specification.md` in the commit that bumps it to `v1.0.0`. The git history preserves the full pre-release version progression.

## Resources

### Version control

- [Git documentation](#apa-git-reference)
- [GitHub remote repositories](#apa-github-reference)

### Versioning standard

- [Semantic Versioning 2.0.0](#apa-semver-reference)

## References

<a name="apa-git-reference"></a>Software Freedom Conservancy. (2024). *Git documentation*. https://git-scm.com/doc
[Return to citation](#apa-git-citation)

<a name="apa-github-reference"></a>GitHub. (2024). *GitHub documentation*. GitHub Inc. https://docs.github.com
[Return to citation](#apa-github-citation)

<a name="apa-semver-reference"></a>Preston-Werner, T. (2024). *Semantic versioning 2.0.0*. https://semver.org
[Return to citation](#apa-semver-citation)

## Changelog

| Version | Status | Notes |
|---------|--------|-------|
| 0.1.0 | Draft | Initial draft |
