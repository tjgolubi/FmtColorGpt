# FmtColor coordination

Updated October 21, 2026.

FmtColor develops color and text-style extensions for C++. We plan two
sequential deliverables:

1. A focused extension to [{fmt}](https://github.com/fmtlib/fmt), offered
   upstream while preserving its formatting engine and compatibility machinery.
2. An independent styling library built on standard C++26 formatting, without
   {fmt}'s unrelated extensions or older-compiler compatibility machinery.

Development currently targets C++23. Both deliverables retain `<fmt/color.h>`
and the styling API in namespace `fmt`; the include path selects the
installation. Shared-source intent covers the common styling API, not arbitrary
fmtlib-specific application code or identical binary representations.

## Documents

| Document                           | Purpose |
| ---------------------------------- | ------- |
| [Workflow.md](Workflow.md)         | Authorization, Git, integration, and release rules. |
| [CppDesign.md](CppDesign.md)       | Implemented foundation, future design, and open decisions. |
| [BuildAndTest.md](BuildAndTest.md) | Build commands, compatibility checks, and current validation. |
| [Website.md](Website.md)           | Website content, publication, and maintenance. |

These files belong to the separate
[FmtColorGpt repository](https://github.com/tjgolubi/FmtColorGpt), not the library
deliverable. They record coordination and intentions. Source and tests establish
implemented behavior; newer explicit instructions from Terry override older
wording. Proposals are not implementation authorization. Earlier development
records remain available in Git history.

## Current baseline

The development branch is
[`FmtColorDev`](https://github.com/tjgolubi/fmt/tree/FmtColorDev). Its chosen
upstream baseline is
[`0fc31633`](https://github.com/fmtlib/fmt/commit/0fc31633e7d53ac895ae0a688b83b7d44f0b0c1f).
The fork's `main` tracks upstream; styling development belongs on `FmtColorDev`.

| Commit | Purpose |
| --- | --- |
| [`0c7dad5c`](https://github.com/tjgolubi/fmt/commit/0c7dad5c53635c474761f933a4bab5508f9bfa53) | One consolidated feature commit directly above the upstream baseline: named RGB constants, style construction, comparisons, hashing, RGB averaging, documentation, tests, and tooling. |
| [`60eed619`](https://github.com/tjgolubi/fmt/commit/60eed6190eb5fb8c8f2d79036e78442da93a57eb) | Publish the generated user guide from `FmtColorDev`, with matching edit links and instructions. |

Thus the current branch has one feature commit and one documentation-publication
fix above its upstream baseline.  No release tag or fork version has been
assigned. The contribution has not been merged into `fmtlib/fmt`.

All five applicable workflows passed for `60eed619`: 18 Linux jobs, 13 Windows
jobs, nine macOS jobs, one documentation job, and two guide-publication jobs.
See [BuildAndTest.md](BuildAndTest.md#current-validation) for evidence and limits.
GCC 16/C++26 reflection is temporarily excluded and matrix fail-fast is disabled;
[issue #2](https://github.com/tjgolubi/fmt/issues/2) tracks restoration.

## Public documentation and next work

- [Project website](https://fmtcolor.ardentokapi66.chatgpt.site): the contribution,
  design rationale, demonstration, and future work. Current public version: 18.
- [Generated user guide](https://tjgolubi.github.io/fmt/color/index.html): API
  references, examples, changed-behavior notes, and named-color swatches.
- [Issues](https://github.com/tjgolubi/fmt/issues): bugs and follow-up work.

The provisional next integration order is **Fluent, VarColor, Palette**.
ConvertColors, whole-style blending/strict composition, and StyledText remain
proposals with open dependencies. RGB averaging is already implemented.
See [CppDesign.md](CppDesign.md#planned-work).
