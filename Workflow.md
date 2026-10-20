# Workflow

[Document map](README.md) · [C++ design](CppDesign.md) · [Build and test](BuildAndTest.md) · [Website](Website.md)

**Working draft, updated October 20, 2026.** Review and adoption are in progress.

## Repository roles

`tjgolubi/fmt` contains the deliverable source, tests, library documentation,
and our change log. Upstream is `fmtlib/fmt`; our `FmtColor` branch is the
integration/development branch above upstream `main`.

`tjgolubi/FmtColorGpt` contains coordination documents that apply across source
branches. Its default branch is `master`. Keep these documents separate from
the deliverable and from the Sites implementation.

The Sites project is maintained separately. See [Website.md](Website.md).

## Git environment on the Windows PC

Use MSYS2 Git for both the `fmt` and `FmtColorGpt` repositories, including
status, diffs, commits, fetches, and pushes. In an MSYS shell, use `/usr/bin/git`.
When invoking Git from PowerShell, explicitly select
`C:\msys64\usr\bin\git.exe` or run it through the MSYS shell. Check executable
provenance before Git operations; a bare `git` in PowerShell may resolve to
Codex's bundled Windows Git. Do not use that bundled Git for these repositories.
Use the existing MSYS2 Git identity and SSH configuration for commits and
network operations rather than replacing them with Windows-specific settings.

This requirement applies to Terry's source and coordination repositories, not
the separate Sites source checkout. On this PC, the Sites Node helper needs
native Windows Git so that checkout paths use Windows syntax. Keep native Git
ahead of MSYS2 tools in the helper's process-local PATH. Do not change the
source repositories' Git configuration to accommodate Sites. See
[Website.md](Website.md#windows-publishing-environment) for Bash and tar setup.

### MSYS PATH, git-cliff, and SSH

Terry explicitly removed UCRT64 from the PC on October 20. Do not use or
reinstall it for this project. The MSYS command PATH is:

`/home/tjgolubi/.cargo/bin:/usr/bin:/c/Windows/System32:/c/Windows`

The reinstalled git-cliff 2.14.2 is the MSYS/Cygwin build at
`/home/tjgolubi/.cargo/bin/git-cliff.exe`, built with
`--no-default-features --target x86_64-pc-cygwin`. It depends on
`msys-2.0.dll`, not UCRT64; optional remote integrations were omitted.
Use it with the existing `cliff.toml`; no UCRT64 PATH is needed.

Command-line Git now uses the existing SSH agent successfully. In each
nonlogin MSYS command needing SSH, source `~/.ssh/agent.env` to import
`SSH_AUTH_SOCK` and the recorded agent state. Do not start replacement agents,
change keys, or replace Git identity to work around a missing environment.
The GitHub connector remains useful for remote inspection; local Git state,
squashes, and diffs use MSYS Git.

## Agree, implement, verify

Discuss and agree on one change, implement it on a focused branch, test it,
review it, and accept it before moving to the next change. A design document
records intentions; it is not blanket implementation authorization.

Questions and tentative proposals are discussion. Explicit instructions
authorize the requested action. Do not treat a possible future feature as an
approved current task. Explain proposed changes before making them. Obtain
Terry's approval before merging or pushing.

On October 19, Terry explicitly clarified: make no changes unless he
specifically requests them. A review, status report, or discussion does not
authorize edits, branch switches, commits, or pushes. Read-only inspection can
support the discussion. A requested documentation or website update authorizes
the edits needed for that update; website publication follows Website.md.
Do not extend that request into unrelated coordination or repository changes.
This clarification supersedes broader standing maintenance wording.

Terry further specified: change things only when he uses the
imperative mood, for example, "Please update the tests." Questions such as
"Can you update the tests?" and statements of intent remain discussion and
do not authorize changes. Read-only inspection can support an answer. This
applies to source, tests, documentation, Git operations that change state, and
website changes. An imperative request authorizes the necessary work within
its scope; honor any stopping point such as "stop before committing."

Before editing, inspect the actual branch, commit, working tree, source, and
tests. Preserve unrelated work. Attached headers and old handoffs may be stale;
they do not establish the current source baseline.

Before starting a task or switching branches, ensure there are no staged or
unstaged changes to tracked files left from other work. Commit those changes
or stash them in agreement with Terry before proceeding. Changes made as part
of the task in progress are expected; account for them before switching away.
Do not discard local work to make the checkout clean.

Untracked files do not make the workspace unclean. Preserve them and do not
repeatedly report them. Raise them only when they affect the task, might be
included accidentally in a commit, or could conflict with a branch switch.
Convenience scripts, archives, and debugging output may intentionally remain
untracked. Do not automatically include untracked files in a commit or stash.

Keep changes minimal and in the existing source style. Change only affected
test lines; do not perform an unrelated cleanup or convert working declarations
to `auto`. Prefer `auto` where an affected declaration needs adjustment.

### Line endings

Keep files as they are stored in the repository. Preserve existing CRLF files;
do not normalize them as part of unrelated edits. Create new text files with
LF line endings. Windows Git must not automatically convert checked-out files
to CRLF: use `core.autocrlf=false` and `core.eol=lf`. These values were set in
Terry's global Git configuration and locally in the `fmt` checkout on October
19, 2026. Inspect any repository attributes or overriding settings before
assuming another checkout follows that policy. Do not introduce a blanket
line-ending conversion or renormalization.

## Branches and commits

Create focused development branches from the agreed `FmtColor` baseline. Use
conventional lowercase slash prefixes, such as `feat/`, `fix/`, `doc/`, `test/`,
`build/`, and `refactor/`; other appropriate prefixes are allowed. Use PascalCase
after the prefix, for example `fix/Termux` or `feat/ConstColor`.
Record proposed branch names before creating their Git branches. Create a
branch when implementation begins from the accepted baseline then, avoiding
repeated rebases of speculative branches. An empty branch is not evidence of
implementation. Prioritize non-breaking additions for the {fmt}-compatible
release; keep known breaking work explicit and later. See CppDesign.md for
the provisional feature plan.

Branches may have dependencies; record those rather than assuming a fixed
integration order. Branch prefixes describe purpose, not acceptance or approval
to include a branch on the website. See [Website.md](Website.md) for that policy.

Use Conventional Commit messages compatible with git-cliff processing:

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

Review source/API compatibility, binary/ABI compatibility, and stored-data
compatibility separately. Passing tests supports acceptance but does not alone
establish compatibility. An unchanged object size does not guarantee an
unchanged binary representation. Distinguish changes to supported contracts
from private implementation details, and state actual effects accurately.
Keep changelog compatibility notes brief and specific, with migration guidance
where needed; put detailed rationale on the website.

Intermediate commits can help development and review. Each accepted integration
adds **exactly one squash commit to `FmtColor`**, including its applicable source,
tests, documentation, and changelog changes. The integration branch must not
expose the intermediate development commits or require a separate changelog
commit for the same integration. Do not substitute a fast-forward or an ordinary
merge commit for the squash commit.

Review the complete proposed result and final squash message with Terry before
merging. Obtain his approval before the merge and before pushing; one explicit
approval may cover both actions when their scope is clear. Verify the resulting
commit before pushing the accepted integration branch.

Do not assume branch status from these documents. Verify current Git history
before merging, rebasing, deleting branches, or recovering a merge.

### Terry's final commit and push

Terry clarified the integration stopping point on October 20: the assistant
normally prepares `git merge --squash`, leaves the result uncommitted for
his review, and provides a final commit message. Terry decides whether to
commit and push. Do not infer permission to commit or push from permission
to prepare the squash. Explicit later instructions can authorize a different
workflow. TermColorValue followed this process.

Terry separately chose to commit the small Clang compatibility fix directly
to `FmtColor` as `723a9c18`, without a feature-branch squash or changelog
edit. This is a recorded exception, not a replacement for the normal workflow.

## Change log

Maintain `FmtColorChangeLog.md` separately from upstream `ChangeLog.md`.
Keep entries concise and descriptive, with accurate scope and brief compatibility
or migration notes where needed. Detailed rationale belongs on the website.
Coordination-document revisions use this repository's own history and are not
library release entries by default.

### Tool version and adopted configuration

Use the same verified git-cliff version on the PC and tablet. Version 2.14.2 is
the current baseline and was verified on both platforms. On October 18, 2026,
`cliff.toml` and the initial `FmtColorChangeLog.md` were adopted in accepted
commit [`f303af51`](https://github.com/tjgolubi/fmt/commit/f303af515b4a5295e67333d7697dd1c797b390d0),
alongside the shared build script and Python bytecode ignore rule. That single
squash commit was pushed to `FmtColor`. The configuration is now tracked;
its fork release-label format remains undecided. See BuildAndTest.md for the
recorded platform results.

InsetXml's unfinished configuration is a useful reference, not an established
working release template. Verify the actual executable version and environment
on each machine before assuming a configuration or modernization works there.

### Commit selection and parser rules

Generate entries from accepted project commits on `FmtColor`, using an explicit
range after the agreed upstream baseline. Exclude upstream history and feature
branches' intermediate development commits. Imported upstream release tags do
not define FmtColor release sections. The configuration uses a separate `fmtcolor-` tag
namespace; its final release-label format remains undecided. Tag filtering alone
does not select the correct branch or exclude upstream commits.

Parser rules use the first matching rule. Bookkeeping exclusions such as
`doc(changelog)` must precede the broad documentation rule. Use
`protect_breaking_commits = true` so intentional breaking changes cannot be
silently excluded. Exclusion rules must also assign a group so a protected
breaking commit remains classified. Keep section order explicit and fail on
nonconventional accepted commits rather than silently hiding them.

The PC previews checked section ordering, ordinary fix entries, breaking-change
markers and compatibility descriptions, changelog-only exclusions, protection of
breaking commits matching exclusions, rejection of nonconventional messages,
and the empty accepted-project range. Representative tablet previews completed
before adoption, confirming that git-cliff works on the tablet. The platform
verification prerequisite was satisfied without changing the workflow policy.

### One commit per accepted integration

Review the final squash message and preview its entry with `--with-commit`
before creating the accepted commit. Include the applicable changelog update
in that same squash commit, rather than adding a separate bookkeeping commit
afterward. Do not invent a commit hash for a prospective entry. Review generated
output before replacing `FmtColorChangeLog.md`; avoid duplicate entries when
regenerating it.

Terry explicitly declined a changelog edit for the Clang 3.6 compatibility
fix `723a9c18`. Preserve that decision; do not add a compensating entry or
regenerate the changelog solely for this fix without a new instruction.

### Fork versions and releases

Do not automatically bump upstream's `FMT_VERSION` from our Conventional Commit
types. Keep the inherited version identifier separate from our fork release
identity. Decide FmtColor's release labels explicitly before enabling automatic
version bumping; the adopted configuration does not enable it.

Record the exact upstream baseline commit as well as the version declared in
its source. A development header's version number does not establish that the
baseline is a tagged upstream release. Upstream tags may remain local for
reference. Push only explicitly approved release tags, not every local tag.
Once published, release tags remain fixed even if development is later rebased.

## Acceptance checklist

Before accepting any proposed integration into `FmtColor`, including source,
documentation, tests, or build changes:

1. Confirm the starting state: the agreed baseline commit, branch dependencies,
   tracked-file cleanliness, and preservation of unrelated local work.
2. Review the complete proposed diff and confirm that it contains only the
   agreed scope, including applicable tests, documentation, and changelog edits.
3. Review source/API, binary/ABI, and stored-data compatibility separately.
   Identify intentional breaks, required migration, and unresolved design
   decisions; verify the Conventional Commit breaking-change metadata.
4. Run relevant tests and required platform checks. Record configurations,
   results, individual skips, and anything unverified. Distinguish fresh tests
   from previously reported results. See [BuildAndTest.md](BuildAndTest.md).
5. Review affected deliverable documentation under `/doc/`, including
   `/doc/color/` when present or needed, and build it when changed. Prepare and
   review the applicable `FmtColorChangeLog.md` entry for inclusion in the same
   squash commit. Record a conclusion that no documentation or changelog edit
   is needed when appropriate.
6. Review coordination documents for changed decisions or instructions. Record
   accepted decisions without presenting unresolved proposals as agreements.
7. Prepare the detailed website explanation. After integration, link the exact
   accepted commit and update any approved work-in-progress entry and expected
   integration order. Do not describe a pending change as integrated. See
   [Website.md](Website.md).
8. Review the complete proposed result and final squash message with Terry and
   obtain his approval before merging.
9. After the approved integration, verify that exactly one intended squash
   commit was added to `FmtColor` and inspect its contents. Push only with
   Terry's approval; one explicit approval may cover both merge and push when
   their scope is clear.

This is a review checklist, not a requirement to change every document for
every integration. The Termux workaround was accepted and pushed on October 18,
2026, as one squash commit,
[`1c61e624`](https://github.com/tjgolubi/fmt/commit/1c61e624ebe67a9ca6a2f1b6465d0ac4203f1c98),
including its changelog entry. The CMake option and warning-bearing test skips
sufficiently explain the change without an upstream `/doc/` edit. Its website
explanation was then published as Site version 5 before this coordination
update. See BuildAndTest.md for the reported platform results. Revisit the
user-documentation conclusion if the scope changes.

Rebasing is governed by the upstream-update policy below; it is not a
requirement for accepting each individual change.

## Upstream updates

Continue development on the agreed upstream baseline. Plan to rebase `FmtColor`
onto upstream `main` when the extension work is complete, before submitting it
upstream. Rebase earlier only when an upstream change directly affects the
work in progress.

Fetch and inspect upstream changes before incorporating them. Record the exact
baseline commit, and keep the fork's changes small and understandable.

Because rebasing rewrites history, coordinate it with anyone using the branch.
Record the old tip, resolve conflicts deliberately, and rerun relevant tests.
If a rewritten published branch must be updated, use an agreed
`--force-with-lease` operation rather than an unguarded force push. Update
dependent feature branches deliberately after the integration baseline is
validated. Do not rebase unrelated work merely because an upstream update exists.

Commit hashes used by Sites may change during rebases. Refresh links while
preserving the explanation of the accepted change.

## Historical initial CleanUp integration, October 20, 2026

Terry accepted the explicit packed RGB construction and emphasis-OR result-type
changes after discussing their source compatibility effects. He requested
CleanUp's squash preparation, stopping before a commit. The assistant staged
the feature and documentation with the changelog generated by git-cliff 2.14.2,
and prepared a message describing both breaks. It stopped with `FmtColor` HEAD
unchanged at `f913e54c`, as requested.

Terry subsequently committed and pushed the accepted result as
[`8026a7d3`](https://github.com/tjgolubi/fmt/commit/8026a7d33bf2a9ec902c8b2e842bcedd30336bf1),
verified locally and through the GitHub connector. This is one squash commit
above `f913e54c`, including source, tests, guide, parallel build invocation,
and changelog. It did not create a release tag or change the inherited version.
The remaining candidate priorities stay provisional. See BuildAndTest.md and
Website.md for validation and confirmed public Site version 14.

## Subsequent amendments and TermColorValue acceptance

Terry amended and force-pushed CleanUp several times. Its accepted replacement
is [`9f399a23`](https://github.com/tjgolubi/fmt/commit/9f399a23b11812e48ec47eb20cbfe80343788c22) above `f913e54c`; `8026a7d3` remains a historical
reference in earlier validation records. The amendments include sentinels,
minimal internal variant operations, underlying support, value-type naming,
tests, and user documentation. They were reviewed before the later integration.

Terry rebased TermColorValue and included the regenerated changelog. The
assistant prepared its squash without committing; Terry reviewed, committed,
and pushed [`81f79d53`](https://github.com/tjgolubi/fmt/commit/81f79d534e56ef24d9f08caefa1cb6dfa47c2936). It is one commit above `9f399a23`, touching
the header, color tests, color guide, and fork changelog. No tag or version
bump was made. Public Site version 15 was then successfully published.

The compiler fix [`723a9c18`](https://github.com/tjgolubi/fmt/commit/723a9c18beebfbda98b0c8e9fbf45528a4be065a) was committed and pushed by Terry afterward.
Local compatibility checks do not imply that the full GitHub matrix passed;
record current CI failures separately in BuildAndTest.md.

## Coordination document maintenance

When an agreed change updates both the website and coordination documents,
publish and verify the website first. Then record its confirmed publication
status in the coordination set. Do not claim a pending publication succeeded.
This ordering applies to design-plan updates as well as accepted integrations;
it does not authorize library integration or Git pushes.

After review and adoption, this repository is the authoritative coordination
set. Commit substantive decision changes, keep the documents linked, and mark
proposals and open questions clearly. Avoid maintaining competing authoritative
copies in project attachments. Historical copies can remain references, but
should not override newer decisions.

Read the latest available coordination documents before starting work. State
when a required document cannot be accessed or its currency is uncertain.
Verify source and website references rather than relying on old chat summaries
or attachments.

Preserve Terry's edits, including changes made during an editing session.
Inspect the current file before modifying it and keep edits limited to the
agreed scope. If the file changes while an edit is being prepared, inspect and
preserve the new content rather than overwrite it with an older copy.

Distinguish proposed wording, accepted decisions, and implemented behavior.
Accepting one revision does not automatically adopt every draft document.
Update affected documents when decisions change, while keeping one
authoritative coordination set in `FmtColorGpt`.

Report whether changes are local, committed, or pushed. Changes become
available to an assistant reading GitHub only after they are pushed; identify
pending local decisions in a cross-device handoff. Do not imply that an
accepted or locally committed edit has already reached GitHub. Follow Terry's
instructions for committing and obtain approval before pushing.
