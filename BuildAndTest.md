# Build and Test

[Document map](README.md) · [C++ design](CppDesign.md) · [Workflow](Workflow.md) · [Website](Website.md)

**Working draft, updated October 20, 2026.** Commands below run from the root
of the `tjgolubi/fmt` checkout unless stated otherwise. A leading `/doc/` in
project discussion means repository-relative, not a filesystem-root path.

## Platforms and configuration

The primary user test platforms are Android/Termux and the Windows PC using
MSYS2. UCRT64 builds have passed on the PC, but UCRT64 is not the project's
required general-purpose environment. Terry prefers POSIX tooling and uses
native Windows tooling when needed. Linux testing is also useful.

Use MSYS2 Git for both project repositories on this PC. Explicitly select
its executable when working from PowerShell; see
[Workflow.md](Workflow.md#git-environment-on-the-windows-pc).

New library work targets C++23. Preserve upstream compatibility machinery and
test relevant older-language configurations when a change is intended to retain
that compatibility. C++26 is an eventual target, not a current prerequisite.

Use a build directory appropriate to the platform and toolchain. Do not reuse
a CMake cache across different machines or incompatible toolchains. The example
`build` directory assumes the intended local toolchain is already selected.

## Normal build and tests

```bash
cmake -S . -B build -DFMT_TEST=ON -DFMTCOLOR_SKIP=OFF -DCMAKE_CXX_STANDARD=23
cmake --build build
ctest --test-dir build --output-on-failure
```

`FMTCOLOR_SKIP` is our fork's opt-in workaround, available on branches containing
that change. It defaults to OFF. If CMake reports that the option is unused,
verify the branch rather than assuming tests were skipped.

Run focused color tests when useful:

```bash
ctest --test-dir build -R '^color-test$' --output-on-failure
```

Choose additional tests for the behavior changed, including extended-character
formatting when relevant. A focused pass does not establish a full-suite pass.

## Termux and FMTCOLOR_SKIP

Some tests deliberately close raw descriptors belonging to live C `FILE`
objects. Android's file-descriptor sanitizer (`fdsan`) can terminate those
processes. The affected test scenarios are unrelated to color rendering.

Our workaround makes the selected tests skip with an informative warning when
explicitly requested. It does not alter production descriptor handling or
disable fdsan. It is not automatically enabled by `__ANDROID__`.

On the affected tablet:

```bash
cmake -S . -B build -DFMT_TEST=ON -DFMTCOLOR_SKIP=ON -DCMAKE_CXX_STANDARD=23
cmake --build build
ctest --test-dir build --output-on-failure
ctest --test-dir build -R '^(gtest-extra-test|os-test)$' -V
```

Verbose output exposes GoogleTest skip warnings that an ordinary successful
CTest run may hide. Individual cases can be skipped inside an otherwise
successful test executable; report the skips explicitly.

The workaround currently covers four `output_redirect_test` cases in
`gtest-extra-test` and three cases in `os-test`: seven individual GoogleTest
cases across two test executables. The definition is scoped to those two
targets. The source is authoritative if that selection changes.

Keep the option OFF on the PC and ordinary Linux builds so those cases run.
Because CMake caches options, explicitly turn it OFF when changing the purpose
of an existing build directory. The workaround is now integrated in `FmtColor`
as `1c61e624`. Terry reported successful runs on the rebased branch on both
PC and Termux; see the accepted validation record below. These are reported
platform results, not a new assistant test run.

## Documentation dependencies

The documentation uses Doxygen, MkDocs, Material, and the project's related
plugins and Python support. Follow the checked-out
`support/doc-requirements.txt` rather than installing an arbitrary latest
MkDocs or changing upstream requirements for convenience.

Follow the environment-setup technique in upstream `support/release.py`:
create a Python virtual environment at `build/venv`, install the pinned
requirements with hash verification, and pass its executable directory to
CMake through PATH. This isolates the Python dependencies without requiring
manual activation in the calling shell. Do not run the release script merely
to build local documentation; it also performs release operations.

For FmtColor on the PC, use the MSYS shell with Python, CMake, GCC, and Doxygen
from `/usr/bin`. UCRT64 is reserved for work that needs that environment.
Install a suitable Doxygen and Python through the platform's package manager.
Do not reuse an environment created with UCRT64 Python in an MSYS build.

## Build and view documentation

From the checkout root in MSYS or Termux, the following subshell prepares the
Python tools and builds documentation without changing the parent shell's PATH:

```bash
(
  set -euo pipefail
  if [[ ! -d build/venv ]]; then
    python -m venv build/venv
  fi
  doc_bin="$(pwd)/build/venv/bin"
  "$doc_bin/python" -m pip install --require-hashes -r support/doc-requirements.txt
  export PATH="$doc_bin:$PATH"
  cmake -S . -B build -U MKDOCS -DFMT_DOC=ON
  cmake --build build --target doc
)
```

The executable `build.bash`, integrated into `FmtColor` on October 18, 2026,
uses this same technique, builds the library in Release/C++23 mode, runs its
tests, and generates documentation. Run it from the checkout root as
`./build.bash` or `bash build.bash`; do not source it into an interactive shell.
It selects `FMTCOLOR_SKIP=ON` when it detects Termux and OFF elsewhere. The
option's implementation was accepted in `1c61e624`; it is now present alongside
the script on `FmtColor`. Selecting ON on older branches without that
implementation cannot enable skips.
Since accepted CleanUp commit `8026a7d3`, the library build invocation uses
`--parallel 8`; test execution remains serial. No measured speedup is claimed.
The script stops on a failing test run, before the documentation-build step.
The commands above remain useful for building documentation separately.

Unlike the upstream release script, these local commands reuse an existing
`build/venv` rather than deleting it on every run. Recreate the environment
when changing toolchains or after an incompatible Python upgrade, preserving
any unrelated files first. Virtual environments are generated local files;
do not commit them. The existing root-level `venv/` may remain untouched.
If using that older environment interactively, `deactivate` exits it without
deleting it. Activation is unnecessary for the script above.

These MSYS/Termux environments use `bin/` inside the venv. Other Python
distributions may use a different layout. Verify executable provenance with
`command -v python cmake g++ doxygen` before setup, and inspect the venv's
Python and MkDocs versions when troubleshooting. `-U MKDOCS` clears any cached
MkDocs path so CMake finds the selected environment.

The configuration is `support/mkdocs.yml`. The observed output directory is
`build/doc-html`, even though upstream getting-started text may mention a
different directory. Use the actual build output as the guide.

To view the built files locally:

```bash
python -m http.server 8000 --bind 127.0.0.1 --directory build/doc-html
```

Open `http://127.0.0.1:8000/` on that device. Stop the server with Ctrl-C.
The Termux documentation build previously succeeded with the project virtual
environment. On October 17, 2026, Codex also verified an MSYS build of `FmtColor`
at `b4ffa39b9994c961418406f47358791d89a5aa48`: Release, C++23, `FMT_TEST=ON`,
`FMT_DOC=ON`, GCC 15.3.0, CMake 4.4.4, Python 3.12.15, MkDocs 1.6.1, and
Doxygen 1.18.0. All 22 CTest executables passed and documentation was generated.
The updated `build.bash` was verified end to end using `build/venv`:
library build, all 22 CTest executables, and documentation generation succeeded.
The baseline did not yet contain `FMTCOLOR_SKIP`; passing OFF produced an
unused-option warning and did not skip tests.

The successful documentation build reported a missing anchor in `syntax.md`
and a Material notice about future MkDocs changes. These did not prevent the
build. Preserve the pinned dependencies. Known upstream link/anchor issues
are outside the current color work; review new documentation for correct links.

### Accepted build tooling and validation, October 18, 2026

`build/BuildBash` was integrated into `FmtColor` as one squash commit,
[`f303af515b4a5295e67333d7697dd1c797b390d0`](https://github.com/tjgolubi/fmt/commit/f303af515b4a5295e67333d7697dd1c797b390d0),
and Terry pushed it to GitHub. The assistant fetched and verified that local
and remote `FmtColor` matched, exactly one commit was added above upstream
baseline `b4ffa39b9994c961418406f47358791d89a5aa48`, and the tree was clean.

The commit includes executable `build.bash`, `cliff.toml`, the initial
`FmtColorChangeLog.md` entry, and the `__pycache__/` ignore rule. No library
API, ABI, stored-data, release-tag, or version-number change was made. Existing
CMake/MkDocs documentation and script comments cover the tooling; no additional
user-facing `/doc/` change was needed. The script's C++23 build setting does
not change the library targets' public C++11 requirement.

The PC library build, all 22 CTest executables, and documentation generation
were verified as recorded above; Terry also reported a successful PC build.
Terry reported that Termux selected `FMTCOLOR_SKIP=ON`, built the library, and
encountered two expected failing test executables because `fix/Termux` was not
yet included. This establishes platform detection, not a passing full Termux
test suite or completed end-to-end script run: the script stops at test failure
before generating documentation. Those historical failures were resolved by
the subsequently accepted Termux workaround, recorded below.

Git-cliff 2.14.2 works on both the PC and Termux. Terry supplied tablet results
on October 18 after updating from 2.12.0 to 2.14.2. Captured output confirms
the intended front matter, scoped entries, section ordering, bookkeeping
exclusion, and protected breaking entry. The tablet prerequisite is complete;
additional testing of git-cliff itself is outside this integration's scope.
Use valid Conventional Commit messages with `--with-commit` to preview the
prospective squash entry before including it in that same commit.

The supplied tablet records were `GitCliffTabletTesting20261018.md`,
`dotest.bash`, `git-cliff.txt`, and `dotest.txt`. These are validation evidence,
not additional deliverable files. The accepted change was published on the
website on October 18 as Site version 4; see Website.md.

### Accepted Termux workaround and validation, October 18, 2026

Terry committed the staged squash as
[`1c61e624ebe67a9ca6a2f1b6465d0ac4203f1c98`](https://github.com/tjgolubi/fmt/commit/1c61e624ebe67a9ca6a2f1b6465d0ac4203f1c98),
`fix(test): allow optional skips for Android fdsan conflicts`, and pushed it.
The assistant verified exactly one commit above `f303af51`, the four intended
files, a tree matching the prepared feature branch, and clean whitespace
checks. A subsequent fetch confirmed the accepted GitHub tip.

Terry reported that `build.bash` ran successfully on the rebased `fix/Termux`
branch on the PC after deleting `build/`, recreating the build and Python
environment. He also reported success on Termux. The PC cache records Release,
C++23, `FMT_TEST=ON`, `FMT_DOC=ON`, and `FMTCOLOR_SKIP=OFF`. Earlier Termux
selection of ON was verified. Seven individual tablet skips are expected from
source; a new verbose tablet log and exact compiler version were not supplied.
These are user-reported results, not a fresh assistant build/test run.

The commit includes the test-only option and guards plus its changelog entry.
No library API, ABI, or stored-data representation changed. The CMake option
and warning-bearing skips explain the workaround; no additional user-facing
`/doc/` change was needed. Site version 5 publishes the accepted explanation
and removes Termux from the candidate list. No release tag or version bump
was created.

## Accepted baseline color documentation, October 19, 2026

`doc/BaseColor` was integrated as one squash commit,
[`f913e54c`](https://github.com/tjgolubi/fmt/commit/f913e54cf41c8aafae8ce951817008e7969d4c59),
on `FmtColor`. Validation during preparation passed all 22 CTest executables
with MSYS tools, Release/C++23, and `FMTCOLOR_SKIP=OFF`; documentation generation
also succeeded. This records that completed run, not a fresh run during this
coordination update. No new Termux validation is recorded for this integration.

The reference pages use Doxygen comments in `include/fmt/color.h`. The handler
in `support/python/mkdocstrings_handlers/cxx/__init__.py` invokes `doxygen -`
while MkDocs builds the pages. It also contains the rendering adjustments
needed by these reference pages. Update comments for shared API descriptions
and Markdown for page organization, examples, tables, and links.

The fork publication build uses:

```bash
PYTHONPATH=support/python python -m mkdocs build \
  --config-file support/mkdocs-pages.yml \
  --site-dir build/doc-html --no-directory-urls
```

Run with the pinned Python documentation dependencies and Doxygen available.
The `FmtColor documentation` GitHub Actions workflow builds on Ubuntu 24.04
with Python 3.12 and deploys the HTML artifact through GitHub Pages after a push
to `FmtColor`. The first
[build and deployment](https://github.com/tjgolubi/fmt/actions/runs/39211196126)
succeeded. The published overview, emphasis table, and named-color swatches
were checked in the browser. See [Website.md](Website.md) for the public links
and the separate Sites publication record.

## Candidate validation reported October 18, 2026

Terry reported that both `feat/ConstColor` and `refactor/TermColorValue` pass
on his PC and tablet. Current GitHub branch tips observed during this update:

| Candidate | Observed GitHub tip | Source status |
| --- | --- | --- |
| `feat/ConstColor` | `875850e5a31c5811041fdd14c28675361f919d54` | Named RGB constants and explicit packed conversions; source/tests inspected. |
| `refactor/TermColorValue` | `4b8ae4ebf2fa4640341845ec5b2983f9e69be9fb` | Terminal-color indices 0–15; source and four-color boundary regression tests inspected; MSYS color-test passed. |

Both branch histories include integrated baseline `1c61e624`. The report does
not identify the exact tested commits, compiler versions, configurations,
commands, counts, or individual skips. Do not attribute the passes to a precise
SHA without that evidence. During the earlier planning update, no assistant test run or GitHub Actions
result was established. The later boundary-test result is recorded below. Both candidates remain
unintegrated and require acceptance review; passes do not prove API, ABI, or
stored-data compatibility.

The inherited GitHub Actions workflows provide additional compiler/platform
coverage. Whether Actions is enabled and whether these tips have passing runs
must be checked separately; PC/tablet passes are not CI passes.

### Terminal-color boundary tests, October 18, 2026

Terry requested boundary checks rather than exhaustive tests of all terminal
colors. Commit
[`4b8ae4eb`](https://github.com/tjgolubi/fmt/commit/4b8ae4ebf2fa4640341845ec5b2983f9e69be9fb)
adds 28 lines to `test/color-test.cc` on `refactor/TermColorValue`, retaining
all existing tests. It checks `black`, `white`, `bright_black`, and
`bright_white` for numeric values 0, 7, 8, and 15; foreground SGR output
30, 37, 90, and 97; background output 40, 47, 100, and 107; and black's
foreground/background presence flags despite its zero numeric value.

The assistant used the MSYS toolchain to run:

```bash
cmake --build build --target color-test --parallel 4
ctest --test-dir build -R '^color-test$' --output-on-failure
```

The complete color-test executable passed with the added boundary checks and
existing tests. This is a focused PC result, not a new full-suite, tablet,
older-language, or CI run. Terry committed and pushed the test addition;
a fetch confirmed the exact remote tip above. Earlier PC/tablet pass reports
remain separate; a tablet run of this new test commit has not been reported.

No additional production-code change was made. The candidate remains breaking
and unintegrated, with Fluent and Palette still ahead of it in the plan.
Website version 7 records the updated tip and this validation evidence.

## ConstColor C++11 header check

On October 18, 2026, Codex verified the color header from
`feat/ConstColor` at `875850e5a31c5811041fdd14c28675361f919d54` using
MSYS GCC 15.3.0 in strict C++11 mode. The check ran from the fmt checkout at
`1c61e624`, without switching branches:

```bash
set -o pipefail
git show feat/ConstColor:include/fmt/color.h |
  g++ -std=c++11 -pedantic-errors -Iinclude/fmt -x c++ -fsyntax-only -
```

The command passed. Supporting headers came from the unchanged accepted
baseline. Namespace-scope constexpr constants and explicit RGB conversions
require no feature newer than C++11; `FMT_INLINE_VARIABLE` is empty before
C++17. This check uses a modern compiler in C++11 mode; the oldest working
GCC, Clang, or MSVC release has not been established. It is not a full library
build, runtime-suite rerun, tablet result, or CI result.

Public Site version 9 published this result before this coordination update.
Earlier reported PC/tablet passes remain separate from this focused check.

## Rebased candidate documentation, October 19, 2026

Both candidates now include accepted baseline `f913e54c`. The documentation
commits below were reviewed in this chat, committed and pushed by Terry, and
confirmed through GitHub during the subsequent website update:

| Branch | Verified GitHub tip | Documentation scope |
| --- | --- | --- |
| `feat/ConstColor` | [`4cf51e13`](https://github.com/tjgolubi/fmt/commit/4cf51e134222dd4b1692fa455fb5ffb5f6f01b5f) | Namespace RGB constants, explicit packed conversions, migration examples, named-color guide, and reference/swatch handler. |
| `refactor/TermColorValue` | [`7c54b595`](https://github.com/tjgolubi/fmt/commit/7c54b59558da188a390f7f1af68b5ac8e7b87902) | Zero-based palette indices, all 16 foreground/background SGR mappings, numeric migration, and binary/persistence guidance. |

ConstColor's rebased implementation and test commits are `97da1b1d` and
`f53a7767`. TermColorValue's are `26462caa` and `38a20f6d`; the latter retains
the four-color boundary and black-presence checks previously tested at
`4b8ae4eb`. These are candidate branch commits, not accepted integrations.

### Assistant checks during documentation preparation

Using the existing MSYS documentation environment, the assistant successfully
built the fork publication configuration for both branches:

```bash
export PATH="$(pwd)/build/venv/bin:/usr/bin:$PATH"
PYTHONPATH=support/python python -m mkdocs build \
  --config-file support/mkdocs-pages.yml \
  --site-dir "$(pwd)/build/doc-html" --no-directory-urls
```

For ConstColor, the assistant also built and ran the complete focused
`color-test` executable successfully, verified the generated declarations show
`explicit` for both packed conversions, inspected the rendered reference, and
checked that all 141 unique named-color swatches match the source RGB values.
These checks preceded Terry's final wording revisions. The assistant did not
rerun the full library suite or claim the historical C++11 header check for
the new tip. For TermColorValue, the assistant verified documentation generation
and whitespace during preparation; no fresh assistant runtime test run was
performed. The known upstream syntax-anchor and Material notices remained
nonfatal. Review of Terry's revisions found one comment typo, which he corrected.

### Terry's reported validation

After his ConstColor wording revisions, Terry reported successful compilation,
tests, and documentation generation before committing and pushing. For
TermColorValue, he first reported that the rebase passed all tests and built
documentation, then reported successful compilation, all tests, and documentation
generation again after his documentation revisions and typo correction.

These are user-reported results, not new assistant full-suite runs. Exact
commands, toolchain/configuration, counts, individual skips, and a final tested
SHA were not supplied. The subsequently verified GitHub tips are recorded
separately above. Earlier PC/tablet reports and October 18 focused checks remain
historical evidence; no fresh tablet or CI result is established by this update.

User-facing documentation and migration guidance are now present on both
branches. Changelog preparation, compatibility acceptance, and final integration
review remain pending. Neither branch was integrated into `FmtColor` during
this chat. Public Site version 13 publishes this status; see Website.md.

## Accepted CleanUp validation, October 20, 2026

CleanUp was accepted and pushed as
[`8026a7d3`](https://github.com/tjgolubi/fmt/commit/8026a7d33bf2a9ec902c8b2e842bcedd30336bf1),
one squash commit above `f913e54c`. It includes the guide, color tests,
`build.bash`'s eight-job build invocation, and the git-cliff-generated entry.
No tag or version bump was created. Source breaks were explicitly accepted;
tests alone are not the compatibility acceptance.

### Assistant's staged-squash PC run

Before Terry committed the final integration, the assistant ran `./build.bash`
on the staged CleanUp squash on `FmtColor`, with MSYS tools, GCC 15.3.0,
Release/C++23, `FMT_TEST=ON`, `FMT_DOC=ON`, and `FMTCOLOR_SKIP=OFF`.
The library build, all **22 CTest executables**, and documentation generation
passed. The pinned Python environment was reused. Known upstream syntax-anchor
and Material notices remained nonfatal. This records the actual pre-commit
run; it is not a claim that the final accepted SHA was rerun after Terry's
additional RGB-test work.

Strict header checks also passed in C++11, C++14, and C++23:

```bash
g++ -std=c++11 -pedantic-errors -Wall -Wextra -fsyntax-only -x c++ include/fmt/color.h
g++ -std=c++14 -pedantic-errors -Wall -Wextra -fsyntax-only -x c++ include/fmt/color.h
g++ -std=c++23 -pedantic-errors -Wall -Wextra -fsyntax-only -x c++ include/fmt/color.h
```

These use a modern compiler's language modes, not the oldest supported compiler
releases. The new color-type composition test covers the three supported kinds
(none, RGB, terminal), both operators, RGB payload OR, reference return, and
unchanged operands on failure. Reserved discriminator `10` is invalid and was
removed from the supported-input matrix when the overflow algorithm was chosen.
The earlier experiment's all-four-discriminator tests do not apply to CleanUp.

The user-guide build passed, and new examples were compiled in C++11 and
C++23. The generated descriptions and section anchors were checked. Explicit
packed conversions, emphasis-valued OR, and getters for empty components are
covered in the committed guide. CleanUp retains padded RGB output and does
not include Crazy's decimal table, unpadded output, or emphasis setter.

### Reported tablet validation and changelog

Terry reported `refactor/CleanUp` working on his tablet before acceptance.
Exact commands, final tested SHA, compiler version, test counts, and individual
skips were not supplied. This remains a user-reported platform result.

The assistant used **git-cliff 2.14.2** with the explicit range
`b4ffa39b9994c961418406f47358791d89a5aa48..FmtColor` and `--with-commit` for the
proposed squash message. The entry includes both accepted breaks and their
migration. It was staged with the change; no separate changelog commit was
created. The staged feature tree matched CleanUp apart from the generated
changelog, and whitespace checks passed. The assistant stopped before commit;
Terry's later accepted GitHub tip was verified through the GitHub connector.

## Candidate rebases reported October 20, 2026

Terry reported successfully rebasing both candidates onto `FmtColor`, passing
all tests, and force-pushing with `--force-with-lease`. GitHub confirmed:

| Candidate | Current tip | Rebased implementation / test commits |
| --- | --- | --- |
| `feat/ConstColor` | [`ed21ec1f`](https://github.com/tjgolubi/fmt/commit/ed21ec1fb95142263587d28d68d049187ed44366) | `aca29f5e` / `efc44c58` |
| `refactor/TermColorValue` | [`db224fb2`](https://github.com/tjgolubi/fmt/commit/db224fb291b46a9e7a511a36ee3cb981e9413e9d) | `6cfaac04` / `e834fbd8` |

Local ancestry checks confirmed accepted `8026a7d3` is an ancestor of both
matching branch tips. Exact tested SHAs, commands, platform configurations,
counts, and individual skips were not supplied; the October 20 report is not a
new assistant runtime, tablet-specific, or CI result. Earlier PC/tablet reports,
October 18 header/boundary checks, and October 19 documentation builds remain
separate historical evidence. Both candidates retain migration documentation
and remain unintegrated, with compatibility acceptance, changelog preparation,
and final integration review pending. Public Site version 14 publishes this
status; see Website.md.

### Experimental Crazy checks

Before CleanUp was selected, temporary compile/link smoke checks of the broader
experiment passed in C++11/14/23, including its lookup-table definition required
before C++17. Unpadded-output expectations passed the focused color, wide-
character, and compile-time-format executables after adjustment. Later Crazy
guide examples compiled and ran in C++11 and C++23, including emphasis replacement
and unpadded output; its documentation built successfully. These are scoped
historical checks, not fresh results for every rebased Crazy tip, an accepted
integration, or a measured speed comparison. CleanUp's padded output is separate.

## Planned Fluent, VarColor, and Palette checks

The agreed implementation order is emphasis-only Fluent, color-focused VarColor,
then Palette extending VarColor. No Fluent, VarColor, or Palette implementation
or test run is recorded by this planning update; earlier validation records
above remain separate.

For Fluent, test all eight emphasis setters/getters, default enabling, explicit
disabling, chaining and reference return behavior, and preservation of colors
and unrelated emphasis bits. Include independent bold/faint combinations and
empty-style queries. Verify relevant older-language and constexpr boundaries.

For VarColor, test the default no-color alternative, color replacement and
clearing, preservation of the other channel and emphasis, presence aliases,
constructors, and unchanged existing getter behavior. Check RGB and terminal
black as specified colors and named-enum use through explicit RGB wrapping.
Verify C++17+ variant APIs and older-mode sentinel/clearing compatibility.
Palette later extends these checks for its new alternatives and boundaries.

Run the shared build script with MSYS tools on the PC, keeping FMTCOLOR_SKIP
OFF, and relevant additional compatibility checks. Record Termux results only
when actually supplied or run, separately from PC results. Follow Workflow.md
for documentation, changelog, and acceptance review.

## Troubleshooting lessons

On Termux, a global MkDocs launcher still pointed to removed Python 3.12 after
Python was upgraded to 3.14. The project virtual environment resolved this.
Check the launcher and interpreter when a present executable reports a missing
file or bad interpreter. Recreate a virtual environment after an incompatible
Python upgrade rather than assuming it remains valid.

On the PC, native UCRT64 Python supplied Windows paths to an MSYS Doxygen,
and the output-directory check failed. Check the provenance and path conventions
of each interacting executable. A native Windows compiler does not require
every documentation or shell tool to be replaced with its UCRT64 counterpart.
Do not change the whole toolchain to fix one documentation dependency.

Sites publication uses a different Windows Git/path environment from the
MSYS library and documentation build. See
[Website.md](Website.md#windows-publishing-environment); do not change the
library toolchain to repair a Sites packaging problem.

## Recording validation

Record the source branch/commit, platform, compiler, language mode, relevant
CMake options, commands, results, and skips. Distinguish a source change tested
locally from a user-reported platform result. Historical successful runs do not
replace verification after new changes. See the acceptance checklist in
[Workflow.md](Workflow.md).
