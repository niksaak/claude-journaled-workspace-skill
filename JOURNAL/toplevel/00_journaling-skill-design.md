# 00 — journaling skill: design & bootstrap

First entry. Records why this repo exists and the decisions behind the
journaling skill it ships, so the reasoning survives even though the
design conversation won't. Written on the trunk (`main`) because the
skill was built directly here, not on a feature branch — per the
convention, trunk-direct work goes straight to `toplevel/`.

## What's here

The repo *is* the distributable skill. Layout:

- `.claude/skills/journaling/CONVENTION.md` — the always-on core
  (read-before / write-after, where entries go, entry format, the
  supersession rule). `@`-imported from `CLAUDE.md`, so it's in context
  at every session start.
- `.claude/skills/journaling/SKILL.md` — on-demand procedures: the
  merge-time consolidation ritual, journal init, edge cases, handy
  commands.
- `.claude/skills/journaling/reference/{feature,toplevel}-entry-example.md`
  — worked examples of each entry kind.
- `README.md` — what it is + two-step install. `program.md` — the
  original brief.
- `JOURNAL/` — this repo dogfooding its own skill.

## Design decisions

- **Split into always-on `CONVENTION.md` + lazy `SKILL.md`.** The brief
  required the workflow to be "read in full at session start," but
  Claude Code skills are progressive-disclosure — only a skill's
  `name`+`description` load at startup, the body loads on demand. A pure
  skill therefore *can't* satisfy that requirement. The split maps the
  constraint onto a frequency-of-use boundary: the terse rules needed
  every message are always-on; the once-per-branch ritual stays lazy.
  `CONVENTION.md` is both the imported memory and a resource the skill
  cites — one source of truth.
- **`@import` over a SessionStart hook** for the always-on half. Both
  are committable and need no per-user config, but `@import` has no
  execution dependency, no trust-prompt, and survives `/compact`. It's
  also the mechanism the lineage project `east-indus-company` already
  proved.
- **Two tiers (`toplevel/` + `feature/<branch-slug>/`).** The
  predecessor projects (`gpu-neat`, `east-indus-company`) used one flat
  numbered log. That collides on the next index across concurrent
  branches and accretes blow-by-blow detail forever. The
  persistent/ephemeral split fixes both: feature journals are numbered
  *locally* (collision-proof) and distilled away at merge; `toplevel/`
  stays curated.
- **Distill, don't concatenate, at merge.** The consolidation step
  compresses a branch's entries into one high-level-but-detailed
  `toplevel/` entry — keep decisions, gotchas, open threads; drop the
  mid-branch back-and-forth — and grounds the summary in
  `git diff merge-base...HEAD` so it reflects what actually shipped, not
  plans later reversed. Generalizes the organic "open-threads-inventory"
  pass that `east-indus-company` did by hand in its journal 17.
- **Consolidate on the branch, before merge**, so the merge atomically
  carries the new `toplevel/` entry and the feature-folder deletion into
  the trunk.
- **Index collisions resolve, never overwrite.** Per-merge numbering can
  clash when two branches merge close together; the second branch in
  renumbers to the next free index — explicit and logged, never a silent
  overwrite.
- **No project-type assumptions.** The predecessor instructions were
  Rust/cargo-specific; all of it was stripped. Examples use
  extension-agnostic paths (`cache/store.*`) and are marked
  illustrative. Git is the one assumed tool, and even it degrades: no
  branch / non-git → entries go straight to `toplevel/`.

## Gotcha worth keeping

**Claude Code skills do not auto-load in full.** At session start only a
skill's frontmatter `name`+`description` enter context; the body is
pulled in lazily on invocation, and there is no frontmatter flag
(`eager`/`alwaysLoad`/…) to force a full load. Anything that must be in
force from the first message has to ride in via `CLAUDE.md` `@import` (or
a SessionStart hook) instead. This is the constraint that shaped the
whole design — re-check it before assuming a skill body is ever "always
on."

## Open threads / assumptions to revisit

Defaults taken without explicit confirmation; each is cheap to change:

- **`toplevel/` also absorbs trunk-direct work**, sharing the per-merge
  number sequence — a slight broadening of "numbered per merge" so trunk
  work leaves no silent gaps. (This very entry is an instance.) Revisit
  if `toplevel/` should be strictly merge-only.
- **Names:** journal root `JOURNAL/`; skill `journaling` (`/journaling`).
  Both easy to rename.
- **Branch slug** flattens `/` → `-` (`feature/auth` → `feature-auth`).
- **Trunk detected by convention** (main/master/trunk + judgment); no
  default-branch name is baked in.
- **Near-zero settings footprint:** feature journals are committed and
  auto-load is `@import`, so no `.claude/settings.json` is needed. The
  only `.gitignore` covers editor/OS scratch, not the workflow itself —
  an editor in this environment auto-created an Obsidian vault
  (`.obsidian/`, `{}` configs + a live `workspace.json`) inside the
  freshly-made skill folder the moment it was created. It's now deleted,
  ignored, and kept out of the distributable; re-check `git status` for
  it if this repo is opened in Obsidian again.

## Next

The first feature branch to use this repo should journal under
`JOURNAL/feature/<branch-slug>/` and, at merge, consolidate into
`toplevel/01_*` following `SKILL.md`.
