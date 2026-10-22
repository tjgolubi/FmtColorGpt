# Workflow

[Document map](README.md) · [C++ design](CppDesign.md) · [Build and test](BuildAndTest.md) · [Website](Website.md)

Updated October 21, 2026.

## Authorization and scope

Modify project files only when Terry instructs us in the imperative mood.
Questions and statements of intent are discussion, not edit authorization.
Read-only inspection, CI checks, and other actions needed to answer a prompt are
allowed. This reflects Terry's October 21 clarification of his earlier wording.

Explain proposed changes before editing. An instruction authorizes the work
needed within its scope; it does not authorize unrelated maintenance. Preserve
explicit stopping points such as "stop before committing." Discuss one change,
implement it, verify it, and review it before moving on.

Library and coordination commits, pushes, merges, branch switches, rebases,
deletions, and stashes require authorization for the relevant operation.
Terry normally performs the final commit and push. Do not infer that permission
from authorization to edit or prepare a squash. Website updates follow the
separate publishing procedure in [Website.md](Website.md).

Before edits or branch changes, inspect the actual branch, commit, working tree,
and affected source. Resolve tracked changes from unrelated work with Terry;
do not discard them or stash automatically. Preserve concurrent edits and
re-read a file if it changed while preparing an update.

Untracked convenience scripts, archives, and logs do not make the checkout
unclean. Preserve them; mention them only when they affect the task, may be
included accidentally, or could conflict with a branch switch. Do not include
them in commits or stashes automatically.

Keep edits minimal and follow the existing source style. Avoid unrelated
cleanup, declaration changes, or broad toolchain changes. Preserve each existing
file's line endings; create new text files with LF. Git uses
`core.autocrlf=false` and `core.eol=lf`; inspect attributes or overrides before
assuming a checkout follows those settings.

## Repositories and tools

| Repository | Role |
| --- | --- |
| `tjgolubi/fmt` | Library source, tests, user documentation, and fork changelog. Current development branch: `FmtColorDev`. |
| `tjgolubi/FmtColorGpt` | These coordination documents; default branch `master`. |
| Sites source repository | Separate website implementation, managed by the Sites helper. |

Use **MSYS2 Git** for `fmt` and `FmtColorGpt`: `/usr/bin/git` in MSYS, or
`C:\msys64\usr\bin\git.exe` from PowerShell. Verify executable provenance;
bare `git` in PowerShell may select Codex's bundled native Git. The Sites Node
helper instead needs native Windows Git; see
[Website.md](Website.md#windows-publishing).

Terry removed UCRT64. Do not use or reinstall it for this project. The MSYS PATH
is `/home/tjgolubi/.cargo/bin:/usr/bin:/c/Windows/System32:/c/Windows`.
Git-cliff 2.14.2 is the MSYS/Cygwin build at
`/home/tjgolubi/.cargo/bin/git-cliff.exe`, built without default features for
`x86_64-pc-cygwin`; it does not require UCRT64.

For SSH in a nonlogin MSYS shell, source `~/.ssh/agent.env`. Reuse the existing
agent, keys, Git identity, and SSH configuration; do not replace them to solve
an environment problem. Use the GitHub connector for remote inspection when
appropriate. Keep credentials out of files, output, and handoffs.

The library remotes are `origin` → `tjgolubi/fmt` and `upstream` → `fmtlib/fmt`.
Keep fork changes on development branches and `main` aligned with upstream.

## Branches and integration

The current baseline is recorded in [README.md](README.md#current-baseline).
Do not continue from an older branch merely because it appears in historical
documents. Verify refs and ancestry before changing history.

Create focused branches when implementation begins from the agreed baseline.
Record the proposed name first. Use lowercase slash prefixes and PascalCase
subjects, such as `feat/Fluent`, `fix/Termux`, or `doc/ColorGuide`; other suitable
prefixes are allowed. A name or design entry neither creates a branch nor grants
approval to implement it or include it on the website.

Each accepted feature integration is one squash commit containing applicable
source, tests, documentation, and changelog changes. Do not substitute a
fast-forward or ordinary merge. The current consolidated feature commit and
separate publication fix are recorded as they actually exist; this policy does
not authorize rewriting them.

### CI gate and acceptance

1. Confirm scope, baseline, dependencies, and preservation of local work.
2. Review the complete candidate. Assess source/API, binary/ABI, and stored-data
   compatibility separately. Unchanged size does not prove ABI compatibility.
3. Finish applicable source, tests, user documentation, and changelog. Run the
   relevant local checks and prepare the proposed squash message.
4. Terry pushes the candidate, or explicitly authorizes the assistant to push.
   Verify local and GitHub full SHAs match. **Every applicable workflow and
   matrix job must pass for that exact SHA before squash preparation.** Queued,
   running, failed, cancelled, or missing expected jobs are not passes.
5. Obtain authorization for the squash onto the agreed development baseline.
   Prepare it, verify the resulting tree and relevant integration checks, and
   stop uncommitted for Terry's review unless instructed otherwise.
6. Terry normally commits and pushes. Verify the intended commit structure and
   all applicable CI for the resulting SHA, including documentation publication
   on `FmtColorDev`. Candidate results do not replace final integration checks.
7. Publish an authorized website update before recording its confirmed
   publication in coordination documents. Update or close an issue only when
   instructed; a successful fix does not by itself authorize closure.

Any amendment, new commit, rebase, changed baseline, or changed squash tree needs
fresh CI for the resulting SHA. Never rely on earlier validation for new code.
The four ordinary push workflows are `linux`, `windows`, `macos`, and `doc`.
Pull-request-only checks such as `lint` and `fuzz` must pass when applicable;
push results do not establish those passes. Do not trigger unrelated release,
security, or publication workflows merely to manufacture a broader result.

Record intentional skips and exclusions explicitly. The current temporary
GCC 16 exclusion and `fail-fast: false` are tracked in
[issue #2](https://github.com/tjgolubi/fmt/issues/2), not treated as GCC 16 coverage.

## Commit messages and changelog

Use Conventional Commits compatible with `cliff.toml`, for example
`feat(color): named RGB constants` or `fix(docs): publish the user guide`.
Mark deliberate breaks with `!` and a `BREAKING CHANGE:` footer. Describe actual
effects without promising stable numeric or binary representations.

Maintain `FmtColorChangeLog.md` separately from upstream `ChangeLog.md`.
Keep entries concise; detailed rationale belongs on the website. Coordination
revisions are not library release entries. Preserve explicit decisions to omit
an entry for a particular fix rather than adding one later without instruction.

Use git-cliff **2.14.2** and an explicit accepted-history range after the chosen
upstream baseline. Exclude upstream history and intermediate feature commits.
For the current baseline, a preview is:

```bash
git cliff --config cliff.toml --offline --no-exec \
  0fc31633e7d53ac895ae0a688b83b7d44f0b0c1f..FmtColorDev
```

Before the final squash, preview its message with `--with-commit` so the entry
can be included in that commit. Inspect output before replacing the changelog.
`cliff.toml`'s baseline comment may lag the current branch; verify the range.
Do not invent a prospective hash or blindly regenerate unrelated entries.

Keep bookkeeping exclusions before general parsers, protect breaking commits,
retain deterministic groups, and fail on unexpected nonconventional commits.
The `fmtcolor-` tag namespace separates fork tags from imported upstream tags;
tag filtering alone does not select the correct history.

Do not automatically bump inherited `FMT_VERSION` or assign a fork release
number. Fork versioning remains undecided. Record the exact upstream baseline;
a development header's version is not proof of a tagged release. Push only
authorized tags; published release tags stay fixed.

## Upstream and cross-machine updates

Continue on the agreed upstream baseline. Inspect upstream changes before
incorporating them. Update the development baseline when a relevant upstream
change warrants it, and before offering the completed extension upstream;
do not rebase unrelated work merely because upstream advanced.

A fast-forward update of `main` is separate from rebasing development branches:

```bash
git fetch upstream
git switch main
git merge --ff-only upstream/main
git push origin main
```

These commands are instructions for an authorized update, not standing
permission to run them. Check local changes and coordinate branch switches.

Before rewriting published history, record the old tip and coordinate with
other checkouts. Resolve conflicts deliberately, compare content, rerun checks,
and use an authorized `--force-with-lease` rather than unguarded force.
A lease protects the expected remote ref, not the intended content. Inspect
refs and reflogs before synchronizing a second machine; never force-push older
history simply to make a rejected push succeed. Refresh website commit links
after an authorized rewrite.

## Coordination maintenance

Read these documents before new work and state when required material is
unavailable or stale. Update affected decisions without maintaining competing
authoritative copies in attachments. Keep proposals, implemented behavior,
tested results, and acceptance distinct.

When asked to update both website and coordination, publish and verify the
website first. Then record the actual version, source, and deployment result.
Report whether coordination edits are local, committed, or pushed; another
machine cannot read local changes from GitHub until they are pushed.
