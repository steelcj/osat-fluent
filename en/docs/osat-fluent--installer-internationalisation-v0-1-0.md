# osat-fluent -- Installer Internationalisation

Version: 0.1.0
Status: Draft
Style Guide: style-guide--versioned-documents-in-unrendered-markdown-v0-1-0

## Abstract

This document captures the open questions and design considerations for adding multi-language support to OSAT Fluent tool installers. It is a working document, not a specification. Its purpose is to preserve the thinking accumulated so far and frame the decisions that must be made before implementation begins. When those decisions are settled, this document will be superseded by a specification and an implementation guide.

## 1. Motivation

OSAT Fluent tools are intended to be accessible to normal people on normal computers. The Accessible principle requires that the tools work for users regardless of their language. The installer is the highest-value target for translation, it is what the user sees when something goes wrong, and it is the first thing a new user encounters.

The goal is not just to support multiple languages but to invite the community to contribute translations. A tool that ships with English and makes it easy to add French, Arabic, or Japanese is more aligned with the OSAT Fluent values than one that requires a core contributor to do all translation work.

## 2. Constraints

Any internationalisation approach must satisfy the following constraints before it qualifies as an OSAT Fluent solution.

**Sovereign.** No external dependencies. The translation mechanism must use only the Python standard library. `gettext` is the natural candidate, it is stdlib and purpose-built for this problem.

**Accessible.** A missing or incomplete translation must never prevent the installer from running. English is the guaranteed fallback at every level, for a missing language file, for a missing string within a language file, and for a corrupted translation file.

**Fluent.** The installer detects the user's locale automatically and uses it without being asked. A `--lang` flag overrides detection for testing and for users who prefer a language other than their system default.

**Transparent.** The fallback behaviour is documented. Contributors know exactly what happens when they add a string without updating all translations.

## 3. Open questions

### 3.1 Detection

How does the installer determine which language to use?

Automatic detection via `$LANG`, `$LANGUAGE`, or the locale API is the fluent approach. The tool speaks the user's language without being asked. A `--lang` flag allows override. The two are not mutually exclusive — automatic detection with flag override is probably the right answer, but the interaction between them needs to be defined.

What happens on Windows where `$LANG` is not conventionally set? The Windows locale API (`locale.getdefaultlocale()`, deprecated in Python 3.11, or `locale.getlocale()`) returns the system locale but the mapping to a language code is not always clean.

### 3.2 Scope

What gets translated?

The installer `log()` and `fail()` messages are the highest priority. These are what the user sees during install and when something goes wrong.

The docstring at the top of the installer is lower priority, it is seen only by contributors reading the source.

The README, ROADMAP, and wrapper templates are a separate question. These are documents, not code, and their translation belongs to the document internationalisation system rather than the installer internationalisation system. They are out of scope for this document.

### 3.3 Translation file format

`gettext` uses `.po` source files compiled to binary `.mo` files. This is the established standard and is supported by widely used tools like Poedit and Weblate. The `.pot` template file is generated from the source with `pygettext` or `xgettext`.

The barrier to entry for `.po` files is higher than for a plain key-value format. A contributor who wants to add a French translation needs to understand `gettext` tooling. An alternative is a plain JSON or YAML file of key-value pairs that the installer converts to a `gettext`-compatible structure at runtime. This lowers the contribution barrier significantly, which aligns with Accessible.

The tradeoff is that a custom format adds complexity to the installer and departs from the established standard. This needs a decision.

### 3.4 Compiled vs source

If `gettext` `.po`/`.mo` files are used, should compiled `.mo` files be committed to the repository or generated at install time?

Committing `.mo` files means the installer works immediately after clone without a build step. Not committing them means the repository contains only source files and contributors cannot accidentally commit a stale compiled file, but requires a build step before the installer can use translations.

### 3.5 Maintenance and completeness

When a new string is added to the installer, existing translation files become incomplete. How does a contributor know their translation needs updating?

Options include a CI check that compares the `.pot` template against all translation files and reports missing strings, a note in `CONTRIBUTING.md` that lists the strings requiring translation on each release, or a runtime warning in the installer when it detects it is using a translation file that predates the current installer version.

### 3.6 The invitation to translate

Where does the invitation to contribute a translation live?

A `TRANSLATING.md` in the osat-fluent repository makes the invitation collection-wide rather than per-tool. Each tool repository links to it. This is cleaner than duplicating the invitation in every tool README.

The invitation should explain what to translate, how to submit it, what tools to use, and what the fallback behaviour is so contributors understand that a partial translation is welcomed and useful.

## 4. Proposed approach — provisional

This section records the approach that seems most promising at this stage. It is not a decision and will change as the open questions are resolved.

Use Python `gettext` with `.po` source files and compiled `.mo` files committed to the repository. Detect locale automatically via `$LANG` with a `--lang` flag override. Fall back to English at every level via `fallback=True`. Scope translation to `log()` and `fail()` messages only in the first iteration. Place the invitation to translate in `osat-fluent/TRANSLATING.md` and link from each tool README.

The JSON/YAML alternative should be reconsidered if contributor feedback indicates that `.po` tooling is a barrier to translation contributions.

## 5. Next steps

Before this document can become a specification the following must be resolved:

- Decision on translation file format, `gettext` `.po`/`.mo` or plain key-value
- Decision on compiled file handling, committed or generated
- Decision on Windows locale detection approach
- Prototype of the `gettext` integration in one installer to validate the approach
- Draft of `TRANSLATING.md` including the contributor invitation

## License

This document, *osat-fluent -- Installer Internationalisation*, by **Christopher Steel**, with AI assistance from **Claude Sonnet 4.6 (Anthropic)**, is licensed under the [GNU General Public License v3.0 or later](https://www.gnu.org/licenses/gpl-3.0.html).

## Changelog

| Version | Status | Notes |
|---------|--------|-------|
| 0.1.0 | Draft | Initial stub — open questions and constraints captured; no decisions made |
