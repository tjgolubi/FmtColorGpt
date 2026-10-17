# Workflow

[Document map](README.md) · [C++ design](CppDesign.md) · [Build and test](BuildAndTest.md) · [Website](Website.md)

**Initial working draft: October 16, 2026.**

## Repository roles

`tjgolubi/fmt` contains the deliverable source, tests, library documentation,
and our change log. Upstream is `fmtlib/fmt`; our `FmtColor` branch is the
integration/development branch above upstream `main`.

`tjgolubi/FmtColorGpt` contains coordination documents that apply across source
branches. Its default branch is `master`. Keep these documents separate from
the deliverable and from the Sites implementation.

The Sites project is maintained separately. See [Website.md](Website.md).

## Agree, implement, verify

Discuss and agree on one change, implement it on a focused branch, test it,
review it, and accept it before moving to the next change. A design document
records intentions; it is not blanket implementation authorization.

Questions and tentative proposals are discussion. Explicit instructions
authorize the requested action. Do not treat a possible future feature as an
approved current task.

Before editing, inspect the actual branch, commit, working tree, source, and
tests. Preserve unrelated work. Attached headers and old handoffs may be stale;
they do not establish the current source baseline.

Keep changes minimal and in the existing source style. Change only affected
test lines; do not perform an unrelated cleanup or convert working declarations
to `auto`. Prefer `auto` where an affected declaration needs adjustment.

## Branches and commits

Create `fix/` and `feat/` branches from the agreed `FmtColor` baseline. Use
PascalCase after the prefix, for example `fix/Termux` or `feat/ConstColor`.
Branches may have dependencies; record those rather than assuming a fixed
integration order.

Use Conventional Commit messages compatible with planned git-cliff processing:

```text
feat(color): named RGB constants
fix(test): optional skips for FILE descriptor tests
test(color): explicit packed RGB conversion coverage
doc(color): RGB conversion examples
doc(changelog): FmtColor change log update
```

Mark intentional breaking changes with `!` and explain them in a
`BREAKING CHANGE:` footer. For example, replacing the `fmt::color` type with a
namespace changes code that names the enum type even when common color
expressions remain valid.

Intermediate commits can help development and review. Accepted source changes
are **squash-merged into `FmtColor`**, so the integration branch does not expose
all development subcommits. Review the final squash message as part of
acceptance. Do not substitute a fast-forward merge.

Do not assume branch status from these documents. Verify current Git history
before merging, rebasing, deleting branches, or recovering a merge.

## Change log

Maintain `FmtColorChangeLog.md` separately from upstream `ChangeLog.md`.
git-cliff is the intended tool; its configuration and initial adoption still
need completion. For now, maintain correct commit style.

When configuring git-cliff, remember that parser rules use the first matching
rule. A rule skipping `doc(changelog)` must precede a broad documentation rule.
Review generated entries for user meaning, compatibility effects, and accurate
scope. Prefer descriptive change-log prose.

The separate coordination repository uses its own Git history for document
revisions; those revisions are not library release entries by default.

## Acceptance checklist

Before accepting a feature or fix into `FmtColor`:

- Confirm the baseline, intended scope, dependency branches, and final diff.
- Review public behavior, compatibility breaks, and unresolved design decisions.
- Run relevant tests and required platform checks; record configurations,
  results, skips, and anything not verified. See [BuildAndTest.md](BuildAndTest.md).
- Review deliverable documentation under `/doc/`, especially `/doc/color/`, and
  build it if affected. A review may reasonably conclude that no edit is needed.
- Review `FmtColorChangeLog.md` and prepare the applicable entry.
- Review the coordination documents for changed intentions or instructions.
- Review Sites coverage and prepare the relevant integrated-commit explanation;
  avoid presenting an unmerged proposal as accepted. See [Website.md](Website.md).
- Review the squash message and staged result, then squash-merge and verify the
  resulting commit before pushing the accepted integration branch.

This is a review checklist, not a requirement to change every document for
every fix. For the Termux workaround, the CMake option and warning-bearing test
skips were considered sufficient explanation without an upstream `/doc/` edit.

## Upstream updates

Fetch and inspect upstream changes before incorporating them. Our intended
strategy is to rebase `FmtColor` onto upstream `main`, keeping the fork's
changes small and understandable.

Because rebasing rewrites history, coordinate it with anyone using the branch.
Record the old tip, resolve conflicts deliberately, and rerun relevant tests.
If a rewritten published branch must be updated, use an agreed
`--force-with-lease` operation rather than an unguarded force push. Update
dependent feature branches deliberately after the integration baseline is
validated. Do not rebase unrelated work merely because an upstream update exists.

Commit hashes used by Sites may change during rebases. Refresh links while
preserving the explanation of the accepted change.

## Coordination document maintenance

After review and adoption, this repository is the authoritative coordination
set. Commit substantive decision changes, keep the documents linked, and mark
proposals and open questions clearly. Avoid maintaining competing authoritative
copies in project attachments. Historical copies can remain references, but
should not override newer decisions.
