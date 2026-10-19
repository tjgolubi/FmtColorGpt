# FmtColorGpt

Working documents for FmtColor development.

The ultimate target is a color/text-style extension to the C++26 standard
library. We plan two sequential deliverables:

1. A focused extension to [{fmt}](https://github.com/fmtlib/fmt), which we will
   offer to the upstream project while preserving its compatibility machinery.
2. A refactored color/text-style library built on standard C++26 formatting,
   excluding {fmt}'s support for pre-C++23 compilers and its unrelated extensions.

We develop against C++23 now. The {fmt} extension is the intermediate step;
the standard-library refactor follows it rather than being developed in parallel.

Both deliverables retain `<fmt/color.h>` and the project's styling API in the
`fmt` namespace. Users select the implementation through their include path,
pointing to the corresponding installation. The intent is to keep user styling
code unchanged between deliverables. The second deliverable uses standard C++
formatting rather than fmtlib's formatting engine or unrelated extensions.
See [CppDesign.md](CppDesign.md) for the compatibility scope.

These coordination documents apply across source branches. They are maintained
separately from the library so they remain available when switching branches.
They are not part of the FmtColor library deliverable.

## Documents

| Document | Purpose |
| --- | --- |
| [CppDesign.md](CppDesign.md) | Forward C++ design, agreed intentions, and open questions. |
| [Workflow.md](Workflow.md) | Git policies, development practices, review, and acceptance checks. |
| [BuildAndTest.md](BuildAndTest.md) | Platform notes, tests, and documentation builds. |
| [Website.md](Website.md) | Sites content and its relationship to library documentation. |

## Repositories and website

- [FmtColor source](https://github.com/tjgolubi/fmt/tree/FmtColor): the `FmtColor`
  integration branch of Terry's {fmt} fork.
- [FmtColorGpt](https://github.com/tjgolubi/FmtColorGpt): this coordination repository.
- [Upstream {fmt}](https://github.com/fmtlib/fmt): the source baseline.
- [FmtColor website](https://fmtcolor.ardentokapi66.chatgpt.site): design and
  implementation explanations; public access.
- [User documentation](https://tjgolubi.github.io/fmt/color/index.html): usage
  examples and reference for the existing `fmt/color.h` API, published through
  GitHub Pages from `FmtColor`.

## Current integration status

On October 18, 2026, the first project change was integrated and pushed to
`FmtColor` as one squash commit:
[`f303af51`](https://github.com/tjgolubi/fmt/commit/f303af515b4a5295e67333d7697dd1c797b390d0),
adding the shared build script, git-cliff configuration, initial fork changelog,
and Python bytecode ignore rule. Git-cliff 2.14.2 works on the PC and tablet.
No release tag or version bump was created. See BuildAndTest.md for validation
and its limits, and Website.md for the confirmed website publication.
The Termux workaround was then accepted and pushed as one further squash commit,
[`1c61e624`](https://github.com/tjgolubi/fmt/commit/1c61e624ebe67a9ca6a2f1b6465d0ac4203f1c98).
Terry reported successful PC and Termux runs of the rebased branch. The website
documents both accepted changes. ConstColor and TermColorValue remain
unintegrated candidates with Terry-reported PC and tablet passes.

On October 19, 2026, `doc/BaseColor` was integrated and pushed as one squash
commit, [`f913e54c`](https://github.com/tjgolubi/fmt/commit/f913e54cf41c8aafae8ce951817008e7969d4c59),
`doc(color): document existing color and text styling APIs`. It adds Doxygen
comments, the `/doc/color/` guide and reference pages, and GitHub Pages
publication. The documentation describes the existing API; it does not
implement the proposed styling extensions. The Pages build and deployment
succeeded. Public Site version 12 now links prominently to the documentation
from its homepage and every page's navigation. See [Website.md](Website.md)
for publication provenance and [BuildAndTest.md](BuildAndTest.md) for validation.
No release tag or version bump was created for this integration.

## Current development plan

Prioritize non-breaking work for the {fmt}-compatible release: `feat/Fluent`,
then `feat/VarColor`, then `feat/Palette`. Fluent covers emphasis only; VarColor
covers color values, constructors, and fluent color operations. Palette extends
VarColor with cube/grayscale alternatives. Later breaking candidates are
`feat/ConstColor`, `refactor/TermColorValue`, and `feat/Blend`; their relative
order is open. `feat/ConvertColors` names the existing explicit RGB-promotion
work using the specified Xterm palette; its integration position remains open.
Blend combines explicit blending with strict composition for `|` and `|=`.
`feat/StyledText` includes `absolute` and lazy nested blending after its required
foundations; exact dependencies remain open. See
[CppDesign.md](CppDesign.md#proposed-branches-and-integration-priorities).

Branch names can describe planned work without a created branch. Create them
when implementation starts to avoid repeated rebases. The revised eight-proposal
plan, including ConvertColors, was published as public Site version 10 on
October 18, 2026, before this coordination update. Website inclusion does not accept an
implementation or authorize integration. See [Website.md](Website.md).

Both deliverables allow independent bold and faint. Retain conceal and omit
flash for {fmt}; our C++23-compatible library design replaces conceal with flash.

## Status and authority

**Initial working draft: October 16, 2026.** These documents consolidate the
project discussion, the former `design.md`, and the coordination handoff. They
need review before being adopted as the authoritative set.

After review and adoption, the documents in this repository are authoritative
for coordination and forward intentions. Chat attachments and exported copies
may become stale. Actual source and tests establish what is implemented;
`CppDesign.md` describes what we intend to build. A design statement does not
mean that a feature is implemented, tested, or merged.

The former `design.md` is replaced by [CppDesign.md](CppDesign.md) after adoption.
There is no separate `CppStyle.md`: modified source follows its existing style;
the eventual independent library follows standard C++ library style.

## Maintaining these files

Use Git history to track changes. Update the relevant document when a decision
changes, distinguish proposals from agreements, and keep unresolved questions
explicit. Link between these files with relative Markdown links so they work
both on GitHub and in a local checkout. See [Workflow.md](Workflow.md).
