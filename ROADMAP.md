# ROADMAP

## 2026-07-07 - Inconsistency to resolve

Two things to flag before that settles, both small but worth catching now.

First, the bump size. By the spec's own versioning rules, PATCH is reserved for spelling, grammar, or clarification with no change to intent, while a new rule added is MINOR. Section 10.7 is a new rule, so this is a `v0.3.0` change, not `v0.2.1`. If you haven't committed or tagged yet, it's a rename of the file to `...-v0-3-0.md` and a version-line edit; if you have, the correction can simply ride the next bump, with a changelog note that 0.2.1 should have been 0.3.0.

Second, the naming rule is now homeless, and one of my documents points at the vacancy. The rclone collision rule, management directories use `<tool-name>-tool`, which we generalized this session into the ownership convention across config, share, and state, was recorded as destined for 10.7 but evidently never landed in the spec. With language layer placement now occupying 10.7, the naming and ownership rule should land as **10.8** in the same bump, it's arguably the more foundational of the two since three tool repos already depend on it. And in the scaffold I gave you, `docs/en/README.md` says the sat-tool layout "generalizes the config-collision rule of specification section 10.7", which is now pointing at the wrong section. Once the naming rule lands as 10.8, that line should read 10.8 instead, a one-word edit worth making before the scaffold's first push so the reference is never wrong in history.

So the tidy end state: spec at `v0.3.0` containing 10.7 language layer placement and 10.8 management identifier and ownership, changelog recording both, and the scaffold's doc reference updated to 10.8. Want me to draft the 10.8 section text now so both land in one commit?