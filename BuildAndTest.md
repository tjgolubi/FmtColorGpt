# Build and test

[Document map](README.md) · [Workflow](Workflow.md) · [C++ design](CppDesign.md) · [Website](Website.md)

Updated October 23, 2026. Commands run from the library checkout root unless
stated otherwise. `/doc/` in project discussion means repository-relative.

## Platforms and tools

Primary development platforms are Windows/MSYS2 and Android/Termux; GitHub CI
adds Linux, Windows, and macOS coverage. New work targets C++23 while retaining
{fmt}'s supported older-language paths. C++26 is a future target, not a current
local-build prerequisite.

On the PC use MSYS `/usr/bin` GCC, CMake, Python, and Doxygen. UCRT64 was removed
at Terry's request; do not use or reinstall it. Use MSYS2 Git for both project
repositories. Before diagnosing tool failures, check:

```bash
command -v git python cmake g++ doxygen
```

The recorded PC toolchain is MSYS GCC 15.3.0, CMake 4.4.4, Python 3.12.15,
MkDocs 1.6.1, and Doxygen 1.18.0. These are observed versions, not instructions
to install arbitrary latest releases. Keep CMake caches and virtual environments
specific to the selected platform/toolchain. See
[Workflow.md](Workflow.md#repositories-and-tools) for Git, SSH, and PATH rules.

## Build and tests

The shared script configures Release/C++23 with tests and documentation,
builds with eight jobs, runs CTest serially, then builds the guide:

```bash
./build.bash
```

Run it from the checkout root; do not source it. It stops at a failing test
before building documentation. For separate control:

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_CXX_STANDARD=23 -DFMT_TEST=ON -DFMTCOLOR_SKIP=OFF
cmake --build build --parallel 8
ctest --test-dir build --output-on-failure
```

For a focused color change:

```bash
cmake --build build --target color-test --parallel 8
ctest --test-dir build -R '^color-test$' --output-on-failure
```

A focused pass does not establish a full-suite pass. Include wide-character,
compile-time-format, or other tests when affected; do not broaden or repeat
checks without a relevant change, failure, or unresolved concern.

### Termux descriptor skips

`FMTCOLOR_SKIP` is a test-only option, OFF by default. Seven tests deliberately
close descriptors owned by live C `FILE` objects and conflict with Android
`fdsan`: four in `gtest-extra-test`, three in `os-test`. ON produces explicit
GoogleTest skips; it does not disable fdsan or alter production code.
`build.bash` detects Termux and selects ON there, OFF elsewhere.

```bash
cmake -S . -B build -DCMAKE_CXX_STANDARD=23 \
  -DFMT_TEST=ON -DFMTCOLOR_SKIP=ON
cmake --build build --parallel 8
ctest --test-dir build --output-on-failure
ctest --test-dir build -R '^(gtest-extra-test|os-test)$' -V
```

Use OFF on ordinary PC/Linux runs; explicitly reset cached options when changing
purpose. An unused-option warning on an older branch means skips were not
enabled. Passing executables can contain individual skips, so record both.
Missing locales can also limit locale-dependent tests.

## Documentation

Use the checked-out, hash-pinned `support/doc-requirements.txt` and Doxygen.
Follow the virtual-environment technique in `support/release.py`, but do not run
that release script merely to build documentation. Reuse the environment unless
an interpreter/toolchain change makes it incompatible; preserve unrelated files.

```bash
(
  set -euo pipefail
  if [[ ! -d build/venv ]]; then
    python -m venv build/venv
  fi
  doc_bin="$(pwd)/build/venv/bin"
  "$doc_bin/python" -m pip install --require-hashes \
    -r support/doc-requirements.txt
  export PATH="$doc_bin:$PATH"
  cmake -S . -B build -U MKDOCS -DFMT_DOC=ON
  cmake --build build --target doc
)
```

`build.bash` uses this environment and installs its pinned dependencies when
creating it. Reinstall from the requirements if dependency inputs change.
MSYS and Termux environments use `bin/`; native Windows Python may differ.
`-U MKDOCS` clears an old cached launcher path. A stale Python launcher after an
upgrade may require recreating the environment, not replacing the whole toolchain.
Do not mix native Windows path conventions with incompatible MSYS tools.

The normal configuration is `support/mkdocs.yml`; output is `build/doc-html`.
The exact GitHub Pages build is:

```bash
(
  export PATH="$(pwd)/build/venv/bin:$PATH"
  PYTHONPATH=support/python python -m mkdocs build \
    --config-file support/mkdocs-pages.yml \
    --site-dir "$(pwd)/build/doc-html" --no-directory-urls
)
```

Preview locally, then stop with Ctrl-C:

```bash
python -m http.server 8000 --bind 127.0.0.1 --directory build/doc-html
```

Open `http://127.0.0.1:8000/` on that device. A known missing anchor in upstream
`syntax.md` and a Material notice about future MkDocs changes are nonfatal; keep
pinned dependencies and do not turn color work into an upstream-link cleanup.

### Reference rendering and publication

Doxygen comments in `include/fmt/color.h` supply shared API descriptions.
Markdown in `doc/color/` supplies organization and examples. The C++ handler at
`support/python/mkdocstrings_handlers/cxx/__init__.py` invokes Doxygen and renders
the reference, named RGB constants, explicit conversions, and swatches.
Public members need Doxygen comments or the handler omits their entries.
The text-style reference uses `exclude_members` to hide the six longer
component/presence accessors and the two former strike methods. Compatibility
notes describe the retained names. Check that `term_color`, `strike()`,
`is_strike()`, and the short getters, setters, and presence queries remain
rendered. The emphasis reference retains the documented enum alias.

The `::: name` blocks are **MkDocs directives**, not C++ or portable GitHub
Markdown. `!!! note "Changed behavior"` needs the configured `admonition`
extension. Check generated HTML for rendered API declarations and
`<div class="admonition note">`, rather than judging raw `.md` pages on GitHub.
The named-color page should contain all 141 unique swatches matching the source.

`.github/workflows/fmtcolor-pages.yml` builds and publishes after pushes to
`FmtColorDev`. The `github-pages` environment permits that branch. The separate
`doc` workflow verifies the build; it does not itself publish this guide.
See [Website.md](Website.md#user-guide) and `support/fmtcolor-pages.md` for setup.

## Compatibility checks

For color changes intended to preserve older modes, check relevant C++11,
C++14, C++17, C++20, and C++23 paths. For example:

```bash
g++ -std=c++11 -pedantic-errors -Wall -Wextra \
  -Wconversion -Wshadow -Werror -Iinclude \
  -fsyntax-only -x c++ include/fmt/color.h

g++ -D_GNU_SOURCE -std=c++11 -pedantic-errors -Wall -Wextra -Werror \
  -Iinclude -Itest -Itest/gtest -fsyntax-only test/color-test.cc
```

Repeat with affected language modes. MSYS GoogleTest needs `_GNU_SOURCE` for
POSIX declarations. The test-source command omits conversion/shadow warnings
for bundled GoogleTest; the header command and CI provide stricter coverage.
A modern compiler in C++11 mode is not evidence about the oldest compiler.

CI has verified old GCC/Clang releases as well as modern standard libraries.
When an exact release is unavailable locally, an authorized Compiler Explorer
check can isolate a diagnostic. Supply actual headers as separate files;
concatenating them can produce misleading errors. Include a negative check when
testing a rejection path and verify scoped warning suppression does not leak.

Existing tests cover construction, unspecified colors, terminal index/SGR
boundaries, style composition and failures, comparisons, hashing/containers,
and variadic RGB averaging. The pairwise RGB member and RGB OR operators do not
have separate direct unit tests; do not claim that variadic tests cover them.
Choose useful tests for actual risk rather than tests that mirror trivial code.

Fluent regression tests in `test/color-test.cc` cover all 256 emphasis
masks for all eight attributes, enable/disable and idempotence, reference
returns and chaining, independent bold/faint rendering, empty getters, color
preservation, component replacement/clearing, and constexpr/noexcept behavior.
Presence-query tests cover each emphasis bit, unspecified colors versus RGB
and terminal black, clearing, and agreement with compatibility names.

ShortNames regression coverage includes alias type identity, all 16 terminal
colors through both spellings, mixed old/new setters and queries, state
preservation, constant evaluation, and ANSI output. The fluent regression
helper exercises `strike()` across all 256 emphasis masks. Tests added since
`main` use the shorter names; the older {fmt} tests retain their original names.

VarColor regression tests cover empty sentinel traits and numeric helpers,
construction, all four reference categories of type/index access, pointer
identity, null and wrong-kind access, cross-alternative assignment and
self-assignment, public reserved-type visitor overloads, forwarded visitors,
mutation, reference/void results, exceptions, constant evaluation, and independent
style getter values. Compiler-specific test workarounds are gated to old GCC
and native MSVC; see [CppDesign.md](CppDesign.md#compiler-constraints).
Palette will need range-boundary, new-alternative, and mixed-kind
composition/renderer coverage.

## Current validation

Verified October 23, 2026 for `FmtColorDev` commit
[`1aaba9b5945dbf269a6a6b3f199e8f77e793be4f`](https://github.com/tjgolubi/fmt/commit/1aaba9b5945dbf269a6a6b3f199e8f77e793be4f):

| Workflow | Successful jobs | Evidence |
| --- | --- | --- |
| Linux | 18 | [Run](https://github.com/tjgolubi/fmt/actions/runs/39903520987) |
| Windows | 13 | [Run](https://github.com/tjgolubi/fmt/actions/runs/39903520980) |
| macOS | 9 | [Run](https://github.com/tjgolubi/fmt/actions/runs/39903520976) |
| Documentation build | 1 | [Run](https://github.com/tjgolubi/fmt/actions/runs/39903520981) |
| Guide publication | 2, including deployment | [Run](https://github.com/tjgolubi/fmt/actions/runs/39903520990) |

All **43 jobs** passed for this exact final amended SHA, including VarColor,
public `fmt::rsvd_color`, and the generated changelog. Local and GitHub SHAs
matched. The runs and every job were independently checked, with no queued,
failed, cancelled, or skipped jobs in the final successful results. Windows
needed a rerun after an MSYS2 mirror download timed out during setup; the
successful final attempt includes both MinGW jobs. The guide is published.

Local validation of the final public reserved-type change included strict
C++11/C++23 header and test-source compilation, the focused Release/C++23
`color-test` build and CTest pass, and the GitHub Pages documentation build.
Earlier VarColor checks covered C++11/14/17/20/23 compilation and a C++11 color
test executable; those do not replace exact-final-SHA CI. This is not a claim
of a fresh local full-suite or tablet/Termux run. The CI matrix supplies old
compiler and broader platform coverage.

Website version 23 was separately validated on all three pages: 45 local
links/anchors, unique IDs, unchanged demo downloads and agreement with the
displayed C++, and two new VarColor examples compiled in C++11 and C++23.
Browser inspection and native Sites publication succeeded. Website checks do
not establish library runtime coverage; see [Website.md](Website.md#current-publication).

### Coverage limits and follow-up

GCC 16/C++26 reflection is temporarily commented out of the Linux matrix because
its package service was unavailable. `fail-fast: false` lets independent jobs
finish. [Issue #2](https://github.com/tjgolubi/fmt/issues/2) tracks restoring both
previous settings and obtaining fresh results. Disabled coverage is not a pass.
Upstream package caching is retained; an empty cache still needs downloads.

PR-only `lint` and `fuzz` are not covered by these push results. Intentional
individual test skips remain distinct from job success. Old Termux logs do not
identify the current tested commit and are not current-SHA evidence.

Record branch/full SHA, platform/compiler, language mode, relevant CMake options,
commands, results, and skips for new validation. Distinguish observed runs from
Terry-reported results. For complete script logs:

```bash
set -o pipefail
./build.bash 2>&1 | tee build.log
```

`build/Testing/Temporary/LastTest.log` contains CTest output, not compiler warnings.
Earlier debugging logs and superseded results remain in Git history. Follow
[Workflow.md](Workflow.md#ci-gate-and-acceptance) for the exact-SHA CI gate.
