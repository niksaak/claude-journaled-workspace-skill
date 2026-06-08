# Engineering journal — convention

> This file is loaded into context at the start of **every** session
> (it is `@`-imported from `CLAUDE.md`). It is the always-on core of
> the journaling workflow. The full procedures — especially
> merge-time consolidation — and the entry templates live in the
> `journaling` skill next to this file; load that skill when you need
> the detail.

This project keeps a two-tier engineering journal under `JOURNAL/`.
Treat it as institutional memory: **consult it before non-trivial
work, append to it after non-trivial work.** It records the *why*
behind decisions, gotchas that bit someone, and the live state of
in-flight work — the things a diff and the commit log don't capture.

## The two tiers

- **`JOURNAL/toplevel/`** — the persistent journal. It lives in the
  trunk and survives across every branch. Numbered entries
  (`00_<slug>.md`, `01_<slug>.md`, …), one per merged feature (plus
  any work done directly on the trunk). Each entry carries an
  `> Origin:` marker under its heading — `trunk-direct` or
  `branch <name>` — recording where it came from; consecutive related
  trunk-direct work folds into the last such entry instead of adding a
  new one (see below). This is the long-term, high-level-but-detailed
  history of the project.
- **`JOURNAL/feature/<branch-slug>/`** — the working journal for the
  current branch. Same numbered-entry format, but scoped to one
  branch and numbered *locally* so concurrent branches never fight
  over the same index. When the branch is done, its entries are
  **distilled into a single new `toplevel/` entry and the branch
  folder is removed** (see *Wrapping up a branch* below).

`<branch-slug>` is the current branch name with `/` and other
non-kebab characters replaced by `-` (e.g. `feature/auth-tokens` →
`feature-auth-tokens`).

**Where does a new entry go?** If you're on a feature/topic branch,
write to `JOURNAL/feature/<branch-slug>/`. If you're on the trunk
(commonly `main` / `master` / `trunk`), or the project isn't using
branches — or isn't a git repo at all — write straight to
`JOURNAL/toplevel/`.

**Trunk-direct work folds into the last entry when it fits.** Work
done straight on the trunk (a hotfix, a quick pre-deadline feature)
doesn't always deserve its own number. Before adding a new
`toplevel/` entry, read the highest-numbered one's `> Origin:` marker:
if it's `trunk-direct` **and** the new work is *related*, update that
entry in place — fold the new work into its narrative so it still
reads as one coherent account — instead of creating a new file. Start
a fresh entry (marked `> Origin: trunk-direct`) only when the last
entry is a branch distillation, the new work is clearly unrelated, or
`toplevel/` is empty. Branch distillations always get their own fresh
entry. Full procedure in the skill.

## Consult — before non-trivial work

- List `JOURNAL/toplevel/` and, if on a branch,
  `JOURNAL/feature/<branch-slug>/`. Filenames are descriptive, so a
  scan of the listing usually surfaces relevant entries.
- Read the entries whose names look related, plus any plan/shipped
  pairs they reference.
- If filenames aren't enough, `grep -rin <keyword> JOURNAL/` for a
  topic buried under an unrelated title (a symbol name, an error
  string, a config key).
- For a current state-of-the-world snapshot rather than a topic
  lookup, read the last few entries — recent ones reflect the live
  design; older ones may have been superseded.

**Supersession rule.** Later-numbered entries supersede earlier
ones. Earlier entries describe decisions that may since have been
refined or reversed; treat them as historical record, not current
state. When entries conflict, trust the latest — and verify against
the actual code when in doubt.

## Write — after non-trivial work

Append an entry before finishing any non-trivial task: an
architectural change, a new module/interface, a bug fix with a
non-obvious root cause, a decision that closed off alternatives, an
investigation that didn't ship code, or a plan for upcoming work
(mark it as a plan; follow up with a "shipped" entry when it lands).
Skip trivial diffs — typos, renames, formatting, routine dependency
bumps — and anything already captured in a commit message or a code
comment.

- **File:** `<NN>_<descriptive-slug>.md` in the directory chosen
  above. `<NN>` is the next zero-padded index after the highest
  existing entry *in that directory*. The slug is kebab-case, short
  and specific (`retry-backoff-on-429`, not `fixes`). *(Exception:
  related trunk-direct work updates the last `toplevel/` entry instead
  — see "Where does a new entry go?" above.)*
- **Heading:** `# NN — <topic>` on one line.
- **Origin marker (`toplevel/` entries only):** the next line is
  `> Origin: trunk-direct` or `> Origin: branch <name>`, recording the
  entry's provenance. Feature-branch entries omit it.
- **Opening:** one or two sentences on what the entry covers and how
  it relates to prior entries (cite them by number — "follow-up to
  4", "implements the plan in 7").
- **Body:** sections as needed. Useful ones: *What changed* (with
  file paths), *Why / alternatives considered*, *Tests* (names +
  final pass/fail count), *Behaviour / smoke* (paste real output),
  *Open threads* (deferred work, known follow-ups).

**Style.**

- Be specific. Cite file paths, symbol/test names, concrete numbers.
- Explain *why*, not just *what* — the diff already shows what.
- Record gotchas verbatim. If a subtle thing bit you (a reserved
  keyword, a silent truncation, an off-by-one in an index
  convention), write it down — that is the highest-value content in
  the journal.
- Past tense for what shipped; present/future for plans and open
  threads.

## Wrapping up a branch — consolidate into the toplevel journal

When a branch's work is done — you're about to merge, you're opening
a PR, or you're otherwise winding it down — the per-branch journal
should be distilled into the persistent one. **Proactively offer to
do this whenever a branch is wrapping up**; it can also be requested
at any time.

The distillation writes **one** new
`JOURNAL/toplevel/<NN>_<feature>.md` — a high-level but detailed
summary of everything the branch did (what shipped, the load-bearing
decisions, gotchas worth keeping, open threads carried forward) —
and then **removes the `JOURNAL/feature/<branch-slug>/` folder**. Do
it as commits on the branch *before merging*, so the merge brings
both the new toplevel entry and the folder removal into the trunk in
one go.

The step-by-step procedure, the distillation templates, and the
edge cases (no git, multiple authors on one branch, index
collisions) are in the `journaling` skill — load it and follow it.

## Scratch files

Loose experiment artifacts at the project root (throwaway scripts,
log dumps, plot data, sample outputs) are not part of the project's
build and shouldn't be wired into it. Leave them where they fall;
the journal references them by name when relevant.
