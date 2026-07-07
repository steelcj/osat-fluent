# OSAT Commit Typology

Version: 0.1.0
Status: Draft
Style Guide: style-guide--technical-documentation-for-technologists v0.2.0

## Abstract

This document defines the types of commits made across OSAT repositories, the content contract of each type, and which types may be automated across many repositories at once. It exists because batch tooling such as myrepos makes it trivial to run commit commands across every repository simultaneously, and exactly one commit type is safe to run that way. The typology is normative for all OSAT Fluent tool repositories, the governance repository, and SAT repositories.

## The four types

### Work commit

The everyday type, covering content, code, and documentation changes. The contract: stage everything with `git add .`, write a subject line describing the change, reuse the `git status` output verbatim as the commit body, push with plain `git push`. Its defining negative is that it contains no version machinery, no change to a `VERSION` file, no `Version:` line edits, no changelog rows. Work commits are safe to batch across repositories.

### Version bump commit

A deliberate, singular act that exercises judgment: choosing patch, minor, or major, and authoring a changelog entry. Version bumps are never batched, because judgment cannot be applied to twelve repositories in one keystroke. Bumps come in two scopes.

Repo scope covers the `VERSION` file, the repository changelog, an annotated tag, and the one-time tag push. Work lands first as work commits, then the bump commit contains only version machinery.

Document scope covers a single document's `Version:` line, its changelog table, and its filename rename while pre-release. The content change and the bump are one commit, because a versioned document's version line, modified date, and changelog entry are always made together. One document per bump commit; two documents needing bumps are two commits. Document-scope bumps carry no tag. Tags are exclusively repo scope, and this is the crisp mechanical distinction between the scopes.

### Initial commit

The birth of a fresh scaffold: everything, `git push -u origin main`, and the `v0.1.0` tag. It happens once per repository by definition, so batching is meaningless. It is governed by the initial commit and versioning workflow document.

### Parking commit

A ROADMAP entry recording a deferred decision or loose end. Mechanically a work commit, but named separately because it is often the only change and its subject should state what was parked, so repository history doubles as a decision log. Parking commits are batchable in principle and singular in practice.

## Boundary rules

The purity rule: a commit is its type entirely. A batch work-commit command must refuse to run when a `VERSION` file is modified, or when a tracked document whose filename carries a pre-release version suffix has been modified, renamed, or deleted, since either signals a bump of some scope in flight that must not be swept into a work batch. A new, untracked version-suffixed document is permitted, since committing an initial draft is work, not a bump.

The tag rule: tags belong exclusively to repo-scope version bumps and initial commits. No batch command ever tags.

The batch rule: only work commits and parking commits may be implemented as batch commands. Version bumps of either scope remain per-repository acts using each repository's own tooling.

## Decisions and rationale

We considered a fifth type for document-scope bumps in the governance repository, where documents version independently of the repository. We rejected it: a document bump has every defining property of the version bump commit, judgment, a permanent changelog claim, and unbatchability, so it is the same type at a different scope. Classifying it as a work commit would have placed a judgment-bearing act in the batchable column, which is the one boundary this typology exists to protect.

We considered defining version bumps as batch commands with per-invocation confirmation. We rejected this as well: a confirmation prompt repeated across repositories trains the operator to approve without reading, and the cost of an unconsidered tag is a permanent public claim.

## License

This document, *OSAT Commit Typology*, by **Christopher Steel**, with AI assistance from **Claude (Anthropic)**, is licensed under the [GNU General Public License v3.0 or later](https://www.gnu.org/licenses/gpl-3.0.html).

## Changelog

| Version | Status | Notes |
|---------|--------|-------|
| 0.1.0 | Draft | Initial draft: four commit types, scope axis for version bumps, purity, tag, and batch rules |
