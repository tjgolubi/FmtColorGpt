# Build and Test

[Document map](README.md) · [C++ design](CppDesign.md) · [Workflow](Workflow.md) · [Website](Website.md)

**Initial working draft: October 16, 2026.** Commands below run from the root
of the `tjgolubi/fmt` checkout unless stated otherwise. A leading `/doc/` in
project discussion means repository-relative, not a filesystem-root path.

## Platforms and configuration

The primary user test platforms are Android/Termux and the Windows PC using
MSYS2. UCRT64 builds have passed on the PC, but UCRT64 is not the project's
required general-purpose environment. Terry prefers POSIX tooling and uses
native Windows tooling when needed. Linux testing is also useful.

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

## Candidate validation reported October 18, 2026

Terry reported that both `feat/ConstColor` and `refactor/TermColorValue` pass
on his PC and tablet. Current GitHub branch tips observed during this update:

| Candidate | Observed GitHub tip | Source status |
| --- | --- | --- |
| `feat/ConstColor` | `875850e5a31c5811041fdd14c28675361f919d54` | Named RGB constants and explicit packed conversions; source/tests inspected. |
| `refactor/TermColorValue` | `0d104dfd55cd71d42d9048c43890207aa769cf53` | Terminal-color indices 0–15 mapped to existing SGR output; source inspected. |

Both branch histories include integrated baseline `1c61e624`. The report does
not identify the exact tested commits, compiler versions, configurations,
commands, counts, or individual skips. Do not attribute the passes to a precise
SHA without that evidence. No new assistant test run or GitHub Actions result
was established during this documentation update. Both candidates remain
unintegrated and require acceptance review; passes do not prove API, ABI, or
stored-data compatibility.

The inherited GitHub Actions workflows provide additional compiler/platform
coverage. Whether Actions is enabled and whether these tips have passing runs
must be checked separately; PC/tablet passes are not CI passes.

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

## Recording validation

Record the source branch/commit, platform, compiler, language mode, relevant
CMake options, commands, results, and skips. Distinguish a source change tested
locally from a user-reported platform result. Historical successful runs do not
replace verification after new changes. See the acceptance checklist in
[Workflow.md](Workflow.md).
