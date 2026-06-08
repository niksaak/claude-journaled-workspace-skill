# Journaled Workspace — a drop-in journaling skill for Claude Code

A self-contained Claude Code skill that gives any project a **two-tier
engineering journal**: a persistent history that lives in the trunk,
plus an ephemeral per-branch working journal that is distilled back
into the persistent one when a branch merges. Built to work in a repo
co-maintained by several people, and to make no assumptions about the
language, build system, or domain of the project it's dropped into.

It is the generalized, productized form of a journaling workflow that
grew organically across two projects (`gpu-neat`, then
`east-indus-company`).

## The two tiers

```
JOURNAL/
  toplevel/                  persistent — survives in the trunk across every branch
    00_<feature>.md          one numbered entry per merged feature (+ direct trunk work)
    01_<feature>.md
  feature/
    <branch-slug>/           ephemeral — one folder per active branch
      00_<topic>.md          numbered locally to the branch, so concurrent
      01_<topic>.md          branches never collide on an index
```

- While you work on a branch, entries accumulate under
  `JOURNAL/feature/<branch-slug>/`.
- When the branch wraps up, the skill **distills** those entries into
  a single new `JOURNAL/toplevel/<NN>_<feature>.md` — high-level but
  detailed — and **removes the branch folder**. Done before merging,
  so the merge carries both changes into the trunk.

The discipline is: **consult the journal before non-trivial work,
append to it after.** It captures the *why*, the gotchas, and the
live state of in-flight work — the things a diff and the commit log
don't.

## Why a skill *and* a CLAUDE.md import

A Claude Code skill is only surfaced by its name + description at
session start; its body loads on demand. That's wrong for a
convention that must be in force from the first message. So the
bundle splits in two:

- **`CONVENTION.md`** — the always-on core (read-before / write-after,
  where entries go, the entry format). It is `@`-imported from
  `CLAUDE.md`, so Claude Code expands it into context at **every**
  session start. This is the most robust drop-in mechanism: committed
  to the repo, no per-user/machine config, survives `/compact`.
- **`SKILL.md`** — the on-demand procedures the convention points at:
  the merge-time consolidation ritual, the templates, and the edge
  cases. Loaded when you're actually doing that work.

Single source of truth, always-on where it must be, lazy where it can
be.

## Install (two steps)

From the target repo's root:

1. **Copy the skill in.**

   ```bash
   mkdir -p .claude/skills
   cp -r /path/to/this/.claude/skills/journaling .claude/skills/
   ```

2. **Import the convention from `CLAUDE.md`.** Add this line to the
   repo-root `CLAUDE.md` (create the file with just this line if it
   doesn't exist):

   ```markdown
   @.claude/skills/journaling/CONVENTION.md
   ```

That's everything. The `JOURNAL/` directory is created on first use —
or ask Claude to "initialize the journal," and the skill scaffolds
`JOURNAL/toplevel/`, `JOURNAL/feature/`, and the first entry.

Commit `.claude/skills/journaling/`, the `CLAUDE.md` line, and the
`JOURNAL/` tree so every collaborator's session picks them up.

## How it behaves once installed

- **Every session:** Claude has the convention in context and follows
  it — checks the journal before non-trivial work, writes an entry
  after.
- **On a feature branch:** entries go to `JOURNAL/feature/<branch-slug>/`.
- **Wrapping up / merging:** Claude proactively offers to consolidate
  the branch journal into a new `toplevel/` entry and remove the
  branch folder. You can also ask for it at any time ("consolidate
  the journal").
- **On the trunk, or in a non-git project:** entries go straight to
  `JOURNAL/toplevel/`; there's no per-branch folder to retire.

## Multi-maintainer notes

- Per-branch folders are numbered **locally**, so two branches never
  fight over the same `NN_` index while in flight.
- `toplevel/` entries are numbered per merge. Two branches merging
  close together can land on the same `toplevel/<NN>` — an expected,
  mechanical conflict: the second branch in **renumbers** to the next
  free index (the skill never silently overwrites an existing entry).
- The per-branch journal is committed, so it's visible in PRs and
  shared among everyone on the branch; it disappears from the trunk
  once distilled, but stays recoverable in git history.

## What's in the bundle

```
.claude/skills/journaling/
  CONVENTION.md                       always-on core (@-imported from CLAUDE.md)
  SKILL.md                            on-demand procedures (consolidation, init, edge cases)
  reference/
    feature-entry-example.md          worked example of a per-branch entry
    toplevel-entry-example.md         worked example of a consolidated entry
```
