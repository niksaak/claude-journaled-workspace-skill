In ../gpu-neat/ take a look at AGENTS_STUFF, journaling instructions in
CLAUDE.md, then at JOURNAL_INSTRUCTIONS.md which was written up based on the
workflow that organically grown in that project. Then take a look at the
very similar workflow in ../east-indus-company/ which was the second iteration.

The aim of this project is to create a full-fledged CLAUDE skill that could
be dropped in any project and implement a similar workflow even in a repo
co-maintained by several people.

Write up this skill, ensure that:
* It is set up so that it is automatically read / enabled in full when claude
  session starts up.
* It doesn't leak any assumptions about what kind of project it is going to be
  used in.

In addition: this workflow is intended to be per-branch or per-feature. When
the development of a particular feature or bugfix is done, or it is time to
merge, this particular per-feature journal needs to be replaced with a
**high-level but detailed** summary of the changes and go into
the toplevel journal (as opposed to feature journal) that persists in the main
branch between all the branched work.

Ask clarifying questions during the design and implementation. Be terse when
communicating, but do not omit details.
