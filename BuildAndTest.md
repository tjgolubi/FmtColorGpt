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
of an existing build directory. Terry reported successful runs of the workaround
branch on both tablet and PC. This is reported validation, not a fresh check of
the current integration branch or its merge status.

## Documentation dependencies

The documentation uses Doxygen, MkDocs, Material, and the project's related
plugins and Python support. Follow the checked-out
`support/doc-requirements.txt` rather than installing an arbitrary latest
MkDocs or changing upstream requirements for convenience.

Install a suitable Doxygen and Python through the platform's package manager.
Use the project's virtual environment for the Python dependencies:

```bash
python -m venv venv
source venv/bin/activate
python -m pip install -r support/doc-requirements.txt
command -v python
python --version
command -v mkdocs
mkdocs --version
command -v doxygen
doxygen --version
```

The activation path above is the one used successfully with this project's
Termux/MSYS2 environments. Other Python distributions can use a different
virtual-environment layout; inspect the environment rather than mixing layouts.

## Build and view documentation

Activate the virtual environment first, then configure and build:

```bash
cmake -S . -B build -DFMT_DOC=ON
cmake --build build --target doc
```

If CMake previously cached a stale MkDocs executable, clear that entry while
configuring:

```bash
cmake -S . -B build -U MKDOCS -DFMT_DOC=ON
cmake --build build --target doc
```

The configuration is `support/mkdocs.yml`. The observed output directory is
`build/doc-html`, even though upstream getting-started text may mention a
different directory. Use the actual build output as the guide.

To view the built files locally:

```bash
python -m http.server 8000 --bind 127.0.0.1 --directory build/doc-html
```

Open `http://127.0.0.1:8000/` on that device. Stop the server with Ctrl-C.
The Termux documentation build succeeded with the project virtual environment.
Known upstream link/anchor issues were observed and are outside the current
color work. New documentation should still be reviewed for correct links.

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
