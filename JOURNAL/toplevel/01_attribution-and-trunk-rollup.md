# 01 — commit attribution + trunk-direct roll-up
> Origin: branch feature/toplevel-workflows-and-attribution

Consolidates `feature/toplevel-workflows-and-attribution` (per-branch
entries 00–01, now distilled and removed). Two independent additions,
both building on the workflow bootstrapped in 00: a commit-attribution
convention scoped to this repo's `CLAUDE.md`, and a trunk-direct
roll-up rule that ships in the distributable `journaling` skill.

## What shipped

### 1. Commit attribution — `Assisted-by:` trailer (`CLAUDE.md`)

Assistant-written commits in this repo end with a Linux-kernel-format
trailer instead of `Co-Authored-By:`:

    Assisted-by: Claude Code:claude-opus-4-8

`CLAUDE.md` gains a **Commit attribution** section documenting the
`Assisted-by: AGENT_NAME:MODEL_VERSION [TOOL…]` grammar. Enforcement is
the CLAUDE.md instruction itself — it overrides the harness default
`Co-Authored-By` trailer — with no git hook. Both commits on this
branch use the trailer and `git` parses it as a real trailer.

### 2. Trunk-direct roll-up rule (`journaling` skill)

Work done straight on the trunk (hotfixes, late pre-deadline features)
no longer takes a fresh toplevel number every time. Each `toplevel/`
entry now carries a provenance marker on the line under its heading:

    > Origin: trunk-direct        # written directly on the trunk
    > Origin: branch <name>       # distilled from a merged branch

Before writing trunk-direct work, read the highest-numbered entry's
marker: if it's `trunk-direct` **and** the new work is *related*,
update that entry in place (merged narrative); otherwise a fresh entry.
Branch distillations always get their own entry. This refines 00's open
thread that trunk-direct work merely "shares the per-merge number
sequence" — it now rolls up instead of always taking a new number.
Changed: `CONVENTION.md` (rule + marker in the entry format),
`SKILL.md` (the 3-step procedure, marker stamping in consolidation and
init, a roll-up-check command), `reference/toplevel-entry-example.md`
(marker + note), and a backfilled `> Origin: trunk-direct` on
`toplevel/00`.

## Key decisions

- **Attribution: `AGENT_NAME` = `Claude Code`, and it replaces (not
  joins) `Co-Authored-By`.** Tool-name + model-id
  (`Claude Code:claude-opus-4-8`) is a more accurate split than the
  kernel example's bare `Claude:claude-3-opus`. Dropping
  `Co-Authored-By` follows the kernel's rationale that an assisting
  tool is not a co-author.
- **Roll-up detection = explicit marker.** Chosen over inferring from
  git history (squash/rebase/cherry-pick erase the merge-vs-direct
  signal — and note this very branch lands as a squash) or a reserved
  filename (couples detection to the slug). Self-contained and survives
  across sessions; cost is a one-line marker on every toplevel entry
  plus the 00 backfill.
- **Roll up only when *related***, not unconditionally — a burst of
  unrelated pre-deadline features would otherwise be buried in one
  entry. Tie-breaker: when unsure, new entry.
- **Merged narrative**, not stacked dated subsections — the entry is
  rewritten to read coherently at its current state, with a hard
  instruction to preserve every existing gotcha/open thread so the
  merge can't silently drop detail.

## Gotchas worth keeping

- **Don't put the model's context suffix in the trailer.** The exact ID
  here is `claude-opus-4-8[1m]` (1M-context variant); in the
  `Assisted-by` grammar square brackets delimit *analysis tools*, so a
  trailing `[1m]` parses as a tool named "1m". Use `claude-opus-4-8`.
- **In-place roll-up does not override two existing rules.** Editing the
  latest toplevel entry is consistent with *supersession* (it's still
  the latest), and it does **not** loosen *plan→shipped*: a plan and a
  later divergent outcome still get the divergence written down, not
  overwritten. Roll-up folds completed, *related* trunk work only —
  `SKILL.md` says so explicitly so the apparent contradiction isn't
  "fixed" away.

## Open threads (carried forward)

- **No mechanical enforcement** for either convention. The `Assisted-by`
  trailer and the `> Origin:` marker are both convention, not validated;
  a malformed/missing marker degrades safely to "treat as not
  trunk-direct → new entry". A `commit-msg` hook (trailer) and a marker
  lint are possible — declined for now in favour of documentation.
- **"Related" stays subjective** — no heuristic beyond same-subsystem /
  direct-follow-up and the unsure→new-entry tie-breaker.
- **Title/slug drift** on long roll-up chains: the rule says widen and
  rename when scope outgrows the title, but a chain of small fixes can
  still end under an underselling slug.
- **`[TOOL]` slots unused** — no coccinelle/sparse-style analysis tooling
  runs here, so the optional trailer tool tags stay empty in practice.
