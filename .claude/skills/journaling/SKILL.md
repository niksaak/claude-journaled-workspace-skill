---
name: journaling
description: >-
  Maintain and consolidate this project's two-tier engineering journal
  under JOURNAL/. Load this when wrapping up or merging a branch — to
  distill its per-branch journal into one new persistent toplevel entry
  and remove the branch folder — when initializing the journal in a repo
  that doesn't have it yet, or when you want the full entry-writing
  procedure, templates, and edge-case handling. The always-on
  read-before / write-after rules live in CONVENTION.md (imported via
  CLAUDE.md); this skill holds the detailed procedures behind them.
---

# Journaling — procedures

The day-to-day rules — where the journal lives, consult-before /
write-after, the entry format, and the supersession rule — are in
`CONVENTION.md` next to this file and are already in context every
session (it is `@`-imported from `CLAUDE.md`). This skill is the
**detailed procedures** the convention points at. If you don't have
the convention handy, read `CONVENTION.md` first.

## Layout recap

```
JOURNAL/
  toplevel/                  persistent; survives in the trunk across all branches
    00_<feature>.md          one per merged feature (+ work done directly on the trunk)
    01_<feature>.md
  feature/
    <branch-slug>/           ephemeral; one folder per active branch
      00_<topic>.md          numbered locally to this branch (no cross-branch clashes)
      01_<topic>.md
```

`<branch-slug>` = current branch with `/` and other non-kebab
characters turned into `-` (`feature/auth-tokens` →
`feature-auth-tokens`).

## Writing a good entry

The format is in `CONVENTION.md`; this is the texture that makes an
entry worth having.

- **Lead with the relationship.** First sentence says what the entry
  covers and how it connects to prior ones — "follow-up to 4",
  "implements the plan in 7", "supersedes 2's approach". A reader
  scanning the log should be able to follow the thread by openers
  alone.
- **Plan → shipped pairs.** When you file a plan before doing the
  work, mark it as a plan. When it lands, write a *separate* shipped
  entry that cites the plan and calls out any *Deviation from plan*
  and why. Don't edit the old plan in place — the divergence is
  signal.
- **Open threads are a first-class section.** Deferred work, known
  follow-ups, and tuning questions go under *Open threads*. These
  are what get carried forward at consolidation, so write them so
  they still make sense out of context.
- **Gotchas verbatim.** The single highest-value content. If a
  reserved keyword, a silent default, an index-convention off-by-one,
  or a header-parsing quirk cost you an hour, quote the exact
  symptom and the exact cause.
- **Cite reality.** File paths, symbol names, test names, real
  numbers, pasted output. "Faster" is worthless; "92ms → 31ms"
  isn't.

See `reference/feature-entry-example.md` for a per-branch entry and
`reference/toplevel-entry-example.md` for a consolidated one.

## Consolidating a branch into the toplevel journal

This is the core procedure. Run it when a branch is wrapping up
(proactively offer it) or whenever asked. The goal: the persistent
`toplevel/` should tell the whole story of what the branch
delivered, so the per-branch detail can be retired.

1. **Scope it.** Default to the current branch
   (`git branch --show-current`); confirm with the user if it's
   ambiguous. Compute `<branch-slug>`. Check that
   `JOURNAL/feature/<branch-slug>/` exists — if it doesn't, there's
   nothing to distill (the work may already be in `toplevel/`, or
   the branch never journaled); say so and handle per *Edge cases*.

2. **Gather the material.**
   - Read **every** entry in `JOURNAL/feature/<branch-slug>/`, in
     order.
   - Skim the last few `JOURNAL/toplevel/` entries for continuity
     and to avoid repeating context the persistent journal already
     has.
   - Ground the summary in what *actually shipped*, not just what got
     journaled. Diff the branch against its merge base —
     `git diff $(git merge-base <trunk> HEAD)...HEAD --stat` for the
     shape, then look closer wherever the journal and the diff
     disagree. Mid-branch plans that were later reversed should be
     described at their **final** state, not as the journal left
     them.

3. **Write one toplevel entry.** New file
   `JOURNAL/toplevel/<NN>_<feature>.md`, where `<NN>` is the highest
   existing `toplevel/` index + 1, zero-padded to the same width.
   Make it **high-level but detailed** — someone reading only
   `toplevel/` should understand what the branch delivered and why,
   without ever opening the per-branch entries. **Distill, don't
   concatenate:**
   - Open with what the branch set out to do and what landed.
   - The load-bearing decisions and the alternatives they closed off.
   - Gotchas worth preserving, verbatim.
   - Open threads carried forward — the per-branch *Open threads*
     that didn't get resolved become this entry's *Open threads*.
   - Describe the final state; mark anything still in flight.

   Use `reference/toplevel-entry-example.md` as the shape.

4. **Remove the branch folder.** Delete
   `JOURNAL/feature/<branch-slug>/`. The raw per-branch entries stay
   recoverable in git history if anyone ever needs them; the
   distilled entry is now the canonical record.

5. **Sequence it for the merge.** Make steps 3–4 commits *on the
   feature branch, before merging*, so the merge carries the new
   toplevel entry in and removes the branch folder together. Stage
   the changes and show them; **commit/push only if the user asks**.

6. **Index collisions are expected — resolve, don't overwrite.** Two
   branches merging close together can both pick the same
   `toplevel/<NN>`. This is inherent to per-merge numbering. If the
   merge conflicts on the filename, the second branch in **renumbers
   its entry to the next free index** — a mechanical fix, no content
   lost. Never silently overwrite an existing toplevel entry; if
   `<NN>` already exists, bump to the next free index.

## Initializing the journal in a repo that lacks it

1. Create `JOURNAL/toplevel/` and `JOURNAL/feature/`.
2. (Optional) Add a one-paragraph `JOURNAL/README.md` for humans:
   what the two tiers are and a pointer to this skill.
3. Ensure the repo-root `CLAUDE.md` imports the convention — add the
   line `@.claude/skills/journaling/CONVENTION.md` to it. Create
   `CLAUDE.md` with that line if it doesn't exist.
4. Write the first entry — `toplevel/00_<slug>.md` if on the trunk,
   else `feature/<branch-slug>/00_<slug>.md`.

## Edge cases

- **No git repo / no branches.** Everything is effectively the
  trunk: write to `toplevel/` directly, name entries for the feature
  in hand, and skip the consolidation step (there's no branch folder
  to retire).
- **On the trunk (main/master/trunk).** Same — entries go straight
  to `toplevel/`.
- **Detached HEAD or an unclear branch.** Don't guess a branch name.
  Ask, or derive a slug from the topic and confirm it.
- **Multiple authors on one branch.** Entries interleave by `<NN>`
  inside the shared `feature/<branch-slug>/` folder. Pull before
  writing; if two people grabbed the same index, renumber on
  conflict. Keep entries small and append-only to minimize the
  window.
- **Branch with no journal entries at merge.** Don't leave a silent
  gap in the persistent journal for a merged feature. Write a brief
  toplevel entry reconstructed from the branch diff and commit
  messages instead.
- **Renamed branch.** The folder is keyed by the old slug. Rename
  `JOURNAL/feature/<old-slug>/` → `<new-slug>/` to match, or just
  consolidate under whichever folder exists.

## Handy commands

Language- and build-agnostic; substitute your trunk's name for
`<trunk>`:

```bash
ls JOURNAL/toplevel/ | sort | tail -1                 # highest toplevel index
git branch --show-current | tr '/ ' '-'               # current branch slug
grep -rin <keyword> JOURNAL/                          # search the whole journal
git diff $(git merge-base <trunk> HEAD)...HEAD --stat # branch change shape
```
