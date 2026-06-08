# JOURNAL

This project's two-tier engineering journal.

- **`toplevel/`** — the persistent history: numbered entries, one per
  merged feature (and any work done directly on the trunk). Survives in
  the trunk across every branch.
- **`feature/<branch-slug>/`** — the working journal for an in-progress
  branch, numbered locally. Distilled into a single `toplevel/` entry
  and removed when the branch merges.

The conventions for reading and writing the journal live in
`.claude/skills/journaling/` and are loaded into every Claude Code
session via `CLAUDE.md`. Consult before non-trivial work; append after.
