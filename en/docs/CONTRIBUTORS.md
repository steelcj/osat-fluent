# CONTRIBUTORS

## Authorship

*osat-fluent* is authored and maintained by **Christopher Steel**.

AI assistance provided by **Claude Sonnet 4.6 (Anthropic)**.

---

## How versioning works in this repository

This repository uses two distinct versioning systems that operate independently and serve different purposes. Understanding both is essential before making changes.

### Repository version

The repository version is a single semantic version string stored in the `VERSION` file at the root of the repository. It tracks the state of the repository as a whole — the collection of documents and tools it contains.

```
VERSION → 0.2.0
```

Semantic versioning rules for the repository:

- **PATCH** — corrections to existing documents (typos, clarifications, broken links), additions or corrections to `utils/`
- **MINOR** — new documents added, existing documents substantially revised, new tools added to `utils/`
- **MAJOR** — a fundamental change in scope, audience, or governing philosophy; removal of documents; restructuring of the repository layout

The repository version is bumped when a commit changes the published state of the repository in a meaningful way. It is not bumped for work in progress.

### Document versions

Every document in `en/docs/` is independently versioned using semantic versioning (`MAJOR.MINOR.PATCH`). The document version appears in the version block immediately below the H1 heading:

```
Version: 0.2.0
Status: Draft | Review | Stable
```

Document versioning rules:

- **PATCH** — spelling, grammar, or clarification of existing content with no change to meaning or intent
- **MINOR** — new sections added, existing sections meaningfully revised, new decisions documented
- **MAJOR** — a fundamental change in approach, scope, or audience; changes that require existing tool installers to be updated

Document versions are independent of the repository version. A single repository version bump may include changes to multiple documents at different version levels.

### The relationship between the two

The repository version does not aggregate or reflect document versions. They answer different questions:

- **Repository version** — what is the current published state of this repository as a whole?
- **Document version** — what is the current published state of this specific document?

A document at `v0.2.0` in a repository at `v0.1.0` is normal. The document was revised; the repository version reflects that a revision was published.

### Pre-release status

Documents at `v0.n.n` are pre-release. They may change substantially between versions. The filename carries the version suffix during this period:

```
osat--user-space-installation-specification-v0-2-0.md
```

At `v1.0.0` the version suffix is dropped from the filename:

```
osat--user-space-installation-specification.md
```

The repository itself follows the same convention — `v0.n.n` indicates the repository is pre-release and the governing documents are still being established.

### Status field

Every document carries a status in its version block:

- **Draft** — actively being written; content may change substantially
- **Review** — content is complete; open for review before stabilisation
- **Stable** — content is settled; changes require a new version

The repository does not carry a status field. The `VERSION` file version number is the signal.

### Making changes

Before changing any document, read its version block and changelog to understand its current state. After changing a document, bump its version, update its `dcterms:modified` date, and add a changelog entry. Then bump the repository `VERSION` to reflect that a change was published.

Document the rationale for every meaningful change. The changelog entry is not a commit message — it explains what changed and why, not just that something changed.
