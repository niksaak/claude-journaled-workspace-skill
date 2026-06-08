# 02 — retrofit Assisted-by trailer onto the root commit
> Origin: trunk-direct

Trunk-direct history fix that applies the attribution convention from
01 retroactively to the one commit that predated it: the bootstrap
commit (the repo root, `Bootstrap journaling skill and dogfood it`).
Implements 01's `Assisted-by:` rule end-to-end so the *entire* trunk
history conforms, not just commits written after the rule landed.

## What changed

The root commit still carried the harness-default trailer:

    Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>

It was rewritten to the Linux-kernel form mandated by `CLAUDE.md`:

    Assisted-by: Claude Code:claude-opus-4-8

Nothing else moved — message body, tree, author, and author-date are
all identical; only the trailer line and (necessarily) the commit
hashes changed. Before → after:

- root: `2dbbc5f` → `b9a0f8f`  (Bootstrap journaling skill and dogfood it)
- tip:  `120b576` → `3371f2b`  (Add Assisted-by commit attribution …)

## How — rewording a *root* commit without `rebase -i`

`git commit --amend` only rewords `HEAD`, and the target is the root,
two commits down. The textbook tool is `git rebase -i --root` (mark the
root `reword`), but interactive rebase isn't available in this
environment. The non-interactive equivalent used here:

    git checkout 2dbbc5f                       # detach at the root
    git commit --amend -F msg.txt              # reword it -> b9a0f8f
    git rebase --onto b9a0f8f 2dbbc5f main     # replay the rest onto it

`--onto` replays `2dbbc5f..main` (just the second commit) onto the new
root and lands `main` on the result. Because the new root's tree is
byte-identical to the old one, the replay applies with no conflicts.

## Why

Consistency. After 01 every *new* commit used `Assisted-by:`, but the
bootstrap commit was grandfathered in with `Co-Authored-By:`. Rewriting
it makes the whole trunk uniform and drops the "co-author" framing that
the kernel rationale (cited in 01) rejects for an assisting tool.

## Verification

- `git diff 120b576 main` → empty. Proves the rewrite is content-only:
  identical trees, only messages/hashes differ.
- `diff` of old-root vs new-root message → exactly one line changed (the
  trailer); body untouched.
- `git log --format='%(trailers:key=Assisted-by,valueonly)'` → both
  commits now resolve a real `Assisted-by: Claude Code:claude-opus-4-8`
  trailer. The string "Co-Authored-By" still appears in `3371f2b`'s
  *body* as prose describing the convention — intentional, left as-is.

## Gotchas worth keeping

- **Reword the root with detach + `rebase --onto`, not `--amend`
  alone.** `--amend` can't reach a non-HEAD commit; `rebase -i --root`
  is the usual way but needs an interactive editor. Detaching to the
  root, amending, then `rebase --onto <newroot> <oldroot> main` is the
  scriptable form and leaves the descendants' content intact.
- **History rewrite → force-push required.** Every hash from the root
  down changed, so `main` diverged from `origin/main`
  (`[ahead 2, behind 2]`). Publishing needs `git push --force`. Left to
  the user by request; the agent did **not** push.
- **Trailer drops the `[1m]` suffix** — `claude-opus-4-8`, not
  `claude-opus-4-8[1m]` — because `[...]` is reserved for analysis
  tools. Same rule as 01's gotcha; re-stated because it's the easy thing
  to get wrong when hand-writing the trailer.
- **The `feature/toplevel-workflows-and-attribution` branch was left
  untouched** and still descends from the *old* root `2dbbc5f`, so its
  copy of the bootstrap commit keeps the old `Co-Authored-By` trailer.
  That branch is retained deliberately (squash-merged branches are kept
  here because toplevel `Origin` markers reference them). If it's ever
  revived or rebased onto the rewritten trunk, it needs the same root
  fix.

## Open threads

- **Force-push pending.** Until the user force-pushes, `origin/main`
  still points at the old `120b576` line. No follow-up needed from the
  agent.
- **Old root object survives** via the feature branch (and the reflog),
  so `2dbbc5f` is not orphaned/GC-able yet — by design.
