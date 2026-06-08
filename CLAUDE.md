# CLAUDE.md

This repository **is** a drop-in Claude Code skill: a two-tier
engineering-journal workflow that can be copied into any project.
See `README.md` for what it is and how to install it elsewhere.

## Repo layout

- `.claude/skills/journaling/` — the distributable skill bundle
  (`CONVENTION.md` always-on core, `SKILL.md` procedures, `reference/`
  worked examples).
- `JOURNAL/` — this repo dogfoods its own skill; the journal lives here.
- `README.md` — install guide. `program.md` — the original design brief.

## This repo follows its own journaling workflow

The line below imports the always-on journaling convention into every
session — the same mechanism a consumer project uses. Consult the
journal before non-trivial work; append after.

@.claude/skills/journaling/CONVENTION.md

## Commit attribution

Commits in this repo that a coding assistant helped write must carry an
`Assisted-by:` trailer in their footer, in the Linux Kernel
coding-assistant format ([guidelines][ca]):

    Assisted-by: AGENT_NAME:MODEL_VERSION [TOOL1] [TOOL2]

- `AGENT_NAME` — the assistant tool/framework (for us, `Claude Code`).
- `MODEL_VERSION` — the exact model ID (e.g. `claude-opus-4-8`).
- `[TOOL1] [TOOL2]` — optional specialized analysis tools (coccinelle,
  sparse, smatch, …). Basic tools (git, gcc, make, editors) are never
  listed.

This trailer **replaces** the `Co-Authored-By:` line — per the kernel's
rationale, an assisting tool is not a co-author. When Claude Code (this
agent) writes a commit, the trailer is:

    Assisted-by: Claude Code:claude-opus-4-8

Drop the model's bracketed context-window suffix (`claude-opus-4-8[1m]`
→ `claude-opus-4-8`): square brackets in this format are reserved for
analysis tools, so a trailing `[1m]` would be misread as one.

[ca]: https://raw.githubusercontent.com/torvalds/linux/refs/heads/master/Documentation/process/coding-assistants.rst
