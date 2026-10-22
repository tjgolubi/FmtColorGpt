# Website

[Document map](README.md) · [Workflow](Workflow.md) · [C++ design](CppDesign.md) · [Build and test](BuildAndTest.md)

Updated October 22, 2026.

## Purpose and content

The public [FmtColor website](https://fmtcolor.ardentokapi66.chatgpt.site)
explains the contribution, design rationale, demonstration, and future work.
Its audience, in priority order, is Terry and collaborating assistants, the
{fmt} developers, technical friends, and nontechnical family. Use brief
plain-language introductions followed by useful technical detail.

The site is not a progress blog. Present the current contribution as a coherent
change, without narrating development struggles, intermediate commits, or
superseded fixes. Explain motivation, behavior, examples, tradeoffs,
compatibility, and validation. Keep implemented APIs separate from proposals,
and fork integration separate from upstream acceptance.

The current structure is:

- [Overview](https://fmtcolor.ardentokapi66.chatgpt.site/): current features, the
  two deliverables, design, and demonstration.
- [The contribution](https://fmtcolor.ardentokapi66.chatgpt.site/integrated-changes.html):
  the foundation, Fluent API, and ShortNames at `8e8b8132`, the upstream baseline, examples,
  compatibility changes, and passing CI.
- [Future work](https://fmtcolor.ardentokapi66.chatgpt.site/work-in-progress.html):
  VarColor, Palette, ConvertColors, whole-style blending, and StyledText.

The next provisional order is VarColor, then Palette. Other dependencies
remain open. Named RGB constants, RGB averaging, and Fluent are implemented;
style blending is not. Branch inclusion needs Terry's approval, not merely a
conventional prefix or existence. Experimental/play branches such as Crazy stay
off the public list unless explicitly approved. Approval to describe a proposal
does not authorize implementation or acceptance.

Current `FmtColorDev` is `8e8b8132`: three feature commits and the separate
publication fix above upstream `0fc31633`. Fluent includes component
getters/setters, `has_fg()`/`has_bg()`/`has_emph()`, and all eight attribute
controls. ShortNames adds `term_color`, `emphasis::strike`, `strike()`, and
`is_strike()` with source-compatible old names. VarColor concerns the future public variant API, not the introduction
of color replacement or short presence queries. The website
does not claim fmtlib has merged the contribution. Check exact refs before
changing links; source and tests establish implementation.

## User guide

The Sites website explains rationale; the library's generated
[user guide](https://tjgolubi.github.io/fmt/color/index.html) explains use.
Do not maintain a competing guide in Sites or put the Sites implementation into
the library repository.

Documentation buttons and user-guide links must point to the **generated HTML**,
not GitHub's raw/rendered Markdown. The source contains `:::` MkDocs reference
directives and `!!! note` admonitions that GitHub Markdown does not process.
Source links remain appropriate when explicitly labeled as source.

The guide covers colors, emphasis, text styles, styled arguments, formatting,
printing, and a separate named-color swatch page. Doxygen comments supply shared
reference text; Markdown provides organization and examples. Keep RGB concepts
before named RGB constants and numeric representation details after ordinary
usage. Keep emphasis discussion on its own page. Normal enum-underlying
facilities do not need generic explanation in this guide.

Keep component accessors, presence queries, attribute queries, and attribute
setters grouped on the Text styles page. Use `fg()`, `bg()`, `emph()`,
`has_fg()`, `has_bg()`, and `has_emph()` throughout examples and prose. Mention
the six older accessor names only in one backward-compatibility note; omit
their separate generated entries with the reference handler's `exclude_members`
option. Keep Doxygen comments on public members, including compatibility
wrappers; undocumented members are omitted by the handler. Use `term_color`,
`emphasis::strike`, `strike()`, and `is_strike()` in examples. The former strike
methods are also excluded from generated member entries and explained in a
compatibility note.

Use `!!! note "Changed behavior"` boxes for changed behavior. Describe the
change without telling users how to fix their code. The MkDocs configuration
enables admonitions; judge the generated HTML rather than source Markdown.

`.github/workflows/fmtcolor-pages.yml` publishes after pushes to `FmtColorDev`.
`support/mkdocs-pages.yml` supplies the fork URL/edit links and disables the
version selector. The `github-pages` environment now permits `FmtColorDev`;
existing `FmtColor` and `main` rules were retained. It uses GitHub Actions as the
Pages source and uploads generated HTML as an artifact, not committed output.
The ordinary `doc` workflow builds but does not publish this guide.
See `support/fmtcolor-pages.md` and [BuildAndTest.md](BuildAndTest.md#documentation).

Guide build and deployment succeeded for `8e8b8132`. The guide includes
the grouped Fluent API, presence queries, and shorter color/emphasis names,
with compatibility notes for retained older names.
The VPS remains an unused alternative host, not a second deployment completed
by this work.

## Demonstration and future design

Keep the downloadable `FmtColorDemo.cpp` and `AnsiRenderDemo.txt`, the displayed
sample, and its styled output together. The demo is a **future acceptance
target**, not implemented/tested functionality of the current contribution.
Preserve nested emphasis/colors, absolute underline, the enclosing background,
and `gray_color{12}`. The HTML output reproduces Terry's older AnsiRender output
with a stated representative terminal palette; exact SGR bytes need not match.

The sample targets the later standard-formatting deliverable using
`std::println`, retaining `<fmt/color.h>` and namespace `fmt` through the selected
installation. Both the displayed and downloadable samples use `term_color`
and `strike()`, retaining `no_color` and
`emphasis::reverse`. The sample shows color-first, background-only,
emphasis-only, emphasis-first, and default-plus-fluent construction. Keep
its downloadable and displayed C++ identical. Shared-source compatibility
also requires standard formatter
support in the {fmt}-based implementation. Do not imply that retaining the
header name introduces a dependency on fmtlib in the independent library.

Preserve these design distinctions:

- Current RGB and style composition ORs overlapping RGB bits and rejects
  terminal conflicts. RGB `blend()` averages channels. Strict style composition
  and whole-style blending remain proposals.
- Fluent setters replace state; sequential ANSI SGR commands update terminal
  state. Neither is the proposed strict composition rule.
- Bold/faint are independent. {fmt} retains conceal and omits flash; the
  independent library replaces conceal with flash.
- Absolute boundaries reset terminal state and block enclosing blending in the
  planned StyledText renderer. Lazy blending, subtree lifetimes, parsing,
  width/precision, and serialization remain design work.

Use [CppDesign.md](CppDesign.md) for the complete design and open decisions.
A documentation or website request does not authorize implementing proposals.

## Sites maintenance and publication

Only update the website when requested. An authorized website update includes
the source edits and normal publication; Terry need not manage its checkout or
build tools. It does not authorize library/coordination commits or pushes.
Follow the currently installed Sites skill and native tool access checks.

1. Identify the existing site by its URL and retained project ID. Read its
   latest metadata, access, and shared source before editing. Do not overwrite
   from an old checkout, downloaded page, captured Library copy, or chat summary,
   and do not create a replacement site.
2. Preserve the site identity, public audience, design, demonstration, assets,
   and unrelated content. Verify source, commit, and CI claims against GitHub.
   Keep credentials in memory and pass them only through the helper's hidden
   stdin, never shell arguments, repository files, logs, or handoffs.
3. Use the Sites source helper to open/synchronize the checkout, own source
   commit/push, run remaining checks, and package the exact pushed state.
   Validate affected copy, local links/anchors, unique IDs, examples, and layout
   with available tools. State meaningful verification limits honestly.
4. Save the exact source/archive as a Sites version and deploy through native
   Sites tools. Reuse successful source/package/version results after a later
   failure. A source push or archive alone is not publication; report success
   only after deployment is confirmed.
5. Return the affected live links. For a request covering both website and
   coordination, publish first, then record confirmed provenance here. Do not
   claim that local coordination edits have been committed or pushed.

Every Sites deployment URL is production. Preserve public access; do not
change it to deploy. Poll only a nonterminal deployment, stop on success/failure,
and do not resave or redeploy an already successful result unnecessarily.
Avoid concurrent edits; resynchronize after another session's update.

If tools cannot retrieve source, edit, or confirm publication, identify the
blocked step. Do not promise that switching apps or devices guarantees access.
A handoff to Sites in the ChatGPT web interface should include the existing URL,
requested change, and direction to read these documents and retrieve fresh source.
Personal-website links belong to Terry's separate personal-website context.

### Windows publishing

The Sites Node helper needs **native Windows Git** ahead of
`C:\msys64\usr\bin` on its process-local PATH. This is an exception for the
separate Sites checkout; MSYS2 Git remains required for `fmt` and `FmtColorGpt`.
Locate the current native Git/runtime rather than assuming an old plugin path.

Bash must also be available for packaging. Set process-local
`TAR_OPTIONS=--force-local` so GNU tar does not interpret the colon in an absolute
Windows archive path as a remote host. Use forward slashes in archive arguments.
Do not alter global Git, SSH, or library-toolchain settings to repair Sites.
Follow the skill's helper procedure and preserve the opening result, source SHA,
and exact archive returned for the selected site.

## Current publication

**Public Site version 22** is live at
[fmtcolor.ardentokapi66.chatgpt.site](https://fmtcolor.ardentokapi66.chatgpt.site).
It describes the integrated foundation, Fluent API, and shorter names, links
to `8e8b8132` and its 43 passing CI jobs, and points to the generated guide.
Examples throughout the site and the downloadable demo use the new names;
former names appear only in the backward-compatibility explanation. VarColor
and Palette remain outside the integrated baseline. The demo remains a future
acceptance target.

| Item | Current value |
| --- | --- |
| Site project ID | `appgprj_6ad129c3e8408191b873a4e06e601c63` |
| Source commit | `0afa422dcd8e6632bf732bb3f711d6fd51b3aec0` |
| Saved version ID | `appgprj_6ad129c3e8408191b873a4e06e601c63~appgver_af9a77cf39bc8191b0c7418d83bf22e6` |
| Successful deployment ID | `appgdep_6ada82bf153c81918f9f98929a45ca5e` |
| Audience | Public |

Website validation covers all three pages, 44 local links/anchors, unique
IDs, whitespace, naming, current source/CI links, and agreement between the
displayed and downloadable demo. The new section was inspected locally and on
the live site in the browser, including its shorter-name example, compatibility
wording, and current SHA. Native deployment returned `succeeded`. The demo
section, styled output, and both downloads are preserved. This publication did
not recompile the website example blocks; website checks are not library
runtime or tablet validation.

Known content drift: the future-work page still describes C++17-gated
`std::variant` APIs. The newer C++11-compatible public variant requirement for
the {fmt} deliverable in [CppDesign.md](CppDesign.md#varcolor-and-palette) is
authoritative. Version 22 updated the names there without changing that design
text; correct it in the next authorized VarColor website update.

Future migration to MkDocs/Material would be a separate task, not an automatic
consequence of maintaining this site.
