# Website

[Document map](README.md) · [C++ design](CppDesign.md) · [Workflow](Workflow.md) · [Build and test](BuildAndTest.md)

**Working draft, updated October 20, 2026.** This document records intended
organization, maintenance guidance, and the publication status verified below.
Later edits must distinguish current observations from future plans.

## Purpose and audience

The [FmtColor Sites website](https://fmtcolor.ardentokapi66.chatgpt.site) explains
the project and its changes. Its intended audience, in priority order, is:

1. Terry and the AI assistants working with him.
2. The {fmt} developers evaluating the extensions.
3. Terry's technical friends.
4. Terry's non-technical family.

Provide detailed technical documentation and rationale, with short plain-language
introductions that help readers understand the purpose before the details.
The website is not a chronological progress blog. Organize technical material
around accepted integration commits and selected proposed branches.

The website is separate from the library repository and this coordination
repository. Its implementation does not belong in the {fmt} fork.

## Current publication

Public Site **version 16** was successfully published on October 20, 2026,
before this coordination update. The native deployment succeeded and the live
accepted-fix section was checked in the browser. Public access was preserved.

- [Integrated changes](https://fmtcolor.ardentokapi66.chatgpt.site/integrated-changes.html#githubci)
  links accepted GitHubCI squash `c89d6dc1`, the matching candidate tree,
  all five passing workflows and 19 Linux jobs, local validation, and closed
  issue #1. Separate linked sections explain conversion fix `2ac6e0bd` and
  the targeted Clang 3.6 workaround `723a9c18`.
- The stale unresolved-failure note now points to the accepted fixes. The
  explanation distinguishes passed compiler workflows from PR-only lint,
  documents the non-template text-style underlying overload, and separates
  historical tablet evidence from validation of the accepted commit.
- The homepage summarizes the resolution and links GitHub Issues. Work in
  progress records the accepted fixes and current baseline without changing
  remaining proposal priorities or claiming ConstColor has been rebased.
- The established design, demonstration, assets, public audience, and existing
  accepted explanations are preserved. No source or coordination files were
  edited as part of that website publication.

All three pages passed link/anchor, duplicate-ID, and whitespace checks;
66 local links/anchors were checked. The browser preview and live page showed
the accepted commit, closed issue, and 19 passing Linux jobs. No fresh library
tests were run solely for publication; the page links existing test evidence.

Publication provenance:

- Site source commit: `9aac96fbe9956733b0f4869204eb115540ce1e81`.
- Saved version 16:
  `appgprj_6ad129c3e8408191b873a4e06e601c63~appgver_0ae87c4cbf408191b3fe121de09f4f26`.
- Successful deployment: `appgdep_6ad845bd6bdc81918279f26213f5def8`.

The subsequent mandatory pre-squash CI policy is recorded in Workflow.md by
this coordination update. Version 16 describes candidate CI before integration;
it does not claim that every later policy detail has already been published.

### Historical version 15

The public Site was successfully published on October 20, 2026, as **version 15**.
The native deployment returned success and the live TermColorValue section
was verified in the browser. Existing public access was preserved.

- [Integrated changes](https://fmtcolor.ardentokapi66.chatgpt.site/integrated-changes.html#termcolorvalue)
  now explains accepted TermColorValue `81f79d53`: indices, output mapping,
  specified black, numeric/binary/persistence migration, exact accepted files,
  regression tests, local checks, user-reported platform results, and the first
  Clang 3.6 CI failure. Amended CleanUp links now target `9f399a23` and cover
  sentinels and underlying support.
- The homepage identifies TermColorValue as integrated. Work in progress
  removes it from the candidate table and retains its old anchor as a pointer
  to the accepted explanation. ConstColor remains pending on its historical
  `8026a7d3` baseline; a rebase onto the current accepted history is not claimed.
- Fluent, VarColor, and Palette retain their planned order. VarColor now builds
  on the accepted sentinel rather than proposing it as new. Crazy remains
  excluded. The design, demonstration, styling, and assets were preserved.

Validation checked three pages, 58 local links/anchors, unique IDs, HTTP
responses, and whitespace. A local browser preview inspected the updated
section and remaining proposal table; the published accepted commit was then
verified in the browser. No new library runtime tests were part of publication.

Publication provenance:

- Site source commit: `b444aea162875166b432515ab269f24d606ece6e`.
- Saved Site version: 15,
  `appgprj_6ad129c3e8408191b873a4e06e601c63~appgver_09641c029a648191b28ae9926a4371c7`.
- Successful deployment: `appgdep_6ad7f38b072c8191a3379810bc46eb19`.

Publication preceded this coordination update. Version 15's CI failure note
describes `81f79d53`; the later pushed Clang fix `723a9c18` and subsequent
GCC conversion-warning failure are recorded in BuildAndTest.md. They have
not yet been published on the Site. This coordination-only request does not
authorize an additional website change.

### Historical version 14

The public Site was successfully published on October 20, 2026, as **version 14**,
before this coordination update. Native deployment confirmed success and the
live CleanUp explanation was checked in the browser. Existing public access
was verified and preserved (policy revision 2).

- [Integrated changes](https://fmtcolor.ardentokapi66.chatgpt.site/integrated-changes.html#cleanup)
  now explains accepted CleanUp squash `8026a7d3`: packed RGB conversion and OR,
  empty-component getters, clearer ANSI generation, both accepted source breaks
  and migration, unchanged padded output and composition, eight-job builds,
  git-cliff generation, and PC versus reported-tablet validation.
- [Work in progress](https://fmtcolor.ardentokapi66.chatgpt.site/work-in-progress.html#constcolor)
  links rebased ConstColor `ed21ec1f` and TermColorValue `db224fb2`, both above
  `8026a7d3`, and refreshes the terminal boundary-test link to `e834fbd8`.
  Terry's October 20 all-tests-pass and `--force-with-lease` reports are distinct
  from historical checks. Explicit packed conversions now belong to accepted
  CleanUp; ConstColor's remaining break replaces the named-color enum.
- The homepage summarizes today's accepted integration and remaining candidates.
  VarColor's wording reflects already-relaxed getter preconditions and the
  already-explicit private packed constructor. The remaining Fluent, VarColor,
  Palette priorities, demonstration, design, and assets are preserved.

At version 14, both candidates remained unintegrated. Crazy was an experiment and was not
added to the approved public candidate list. No new library tests, CI checks,
or tablet-layout checks were run as part of this website update.

Validation checked all local HTML links and anchors, unique IDs, dates, and
whitespace. A local browser preview and screenshot inspected CleanUp's layout;
page text confirmed both candidate records and links. The published CleanUp
section was then verified in the browser. Public source tips were checked with
the GitHub connector after command-line SSH lacked a usable key; the connector
was available and worked. No Git/SSH configuration was changed.

Publication provenance:

- Site source commit: `4c1e191948de91bdc7a555e4a1d3bb6932186434`.
- Saved Site version: 14,
  `appgprj_6ad129c3e8408191b873a4e06e601c63~appgver_e581a90487c0819196f2ab2d7e561f59`.
- Successful deployment: `appgdep_6ad73867b4bc819195c212f3b88d693d`.

### Version 13

The public Site was successfully published on October 19, 2026, as **version 13**,
before this coordination update. The deployment tool confirmed success and the
public URL. The existing public audience was verified before publication and
preserved.

The [Work in progress](https://fmtcolor.ardentokapi66.chatgpt.site/work-in-progress.html)
page now records both rebased candidates above accepted baseline `f913e54c`:

- [ConstColor](https://fmtcolor.ardentokapi66.chatgpt.site/work-in-progress.html#constcolor)
  links verified documentation tip `4cf51e13`, describes namespace constants,
  explicit packed conversions, migration, and reference/swatch generation,
  and links its committed branch guide.
- [TermColorValue](https://fmtcolor.ardentokapi66.chatgpt.site/work-in-progress.html#termcolorvalue)
  links verified documentation tip `7c54b595`, describes the numeric/SGR reference
  and migration guidance, refreshes the rebased boundary-test link, and links
  its committed branch guide. The branch name is singular, `TermColorValue`.

Both entries distinguish Terry's newly reported successful build, tests, and
documentation generation from earlier PC/tablet reports and focused assistant
checks. The old C++11 check is explicitly historical. User-facing documentation
is no longer listed as pending; changelog preparation, compatibility acceptance,
and final integration review remain pending. Neither candidate is integrated.

Validation checked local HTML links, anchors, unique IDs, and whitespace.
A local browser preview and screenshot inspected the updated ConstColor layout;
the browser's page text confirmed both revised sections and link destinations.
No new library tests or tablet-layout checks were performed during publication.
The existing integration plan, other proposal sections, homepage, Integrated
changes page, demonstration, and assets were preserved. The successful native
deployment result confirms publication; no separate live-content browser check
is recorded for version 13.

Publication provenance:

- Site source commit: `f2c34b248dfd9c360c6169bea9bc70482d2e42d1`.
- Saved Site version: 13,
  `appgprj_6ad129c3e8408191b873a4e06e601c63~appgver_0c9633664fb8819198efb88b07ffa89d`.
- Successful deployment: `appgdep_6ad659cdb1008191848fd63422c58b62`.

### Version 12

The public Site was successfully published on October 19, 2026, as **version 12**.
The deployment operation confirmed success and the live homepage was checked.
Public access was preserved. A prominent **Read the user documentation** button
appears beside the homepage introduction; a highlighted **Documentation** link
appears in all three pages' navigation. Both lead to the
[color documentation](https://tjgolubi.github.io/fmt/color/index.html).
The homepage's planned-documentation paragraph now describes the published guide.

Validation included a local browser preview, inspection of the published
homepage, documentation-link destinations, and `git diff --check`. Existing
design explanations and demo assets were preserved. This website update did
not run library tests or add an Integrated changes section for `doc/BaseColor`.

Publication provenance:

- Site source commit: `d01a2b18edf224614c84815539ad8068a338934a`.
- Saved Site version: 12,
  `appgprj_6ad129c3e8408191b873a4e06e601c63~appgver_e040aacd06108191a1569db29bddbd34`.
- Successful deployment: `appgdep_6ad5bdc4a464819181bad64f6b382338`.

### Version 10

The public Site was successfully published on October 18, 2026, as **version 10**
before this coordination update. The deployment operation confirmed success.
Public access was verified before publication and preserved.

Version 10 adds `feat/ConvertColors` to the overview and Work in progress plan,
with a dedicated section for the existing explicit RGB-promotion design.
The section reproduces all 16 specified Xterm `terminal_palette` entries
unchanged. Nearest-palette demotion remains a possible later extension.
This update names a feature branch for existing design work; it does not
expand conversion scope, settle API signatures, or assign a new integration
position. The initial order remains Fluent, VarColor, Palette.

Validation checked local links, anchors, unique IDs, and exact agreement with
the design's palette. A local browser preview confirmed the new section.
Existing implementation-status and validation records, Integrated changes,
styles, and demo assets were preserved. No library tests were rerun and no
ConvertColors Git branch or library implementation was created.

Publication provenance:

- Site source commit: `5181d6a04e8a4f233cd7f83015972dc4db8423bf`.
- Saved Site version: 10,
  `appgprj_6ad129c3e8408191b873a4e06e601c63~appgver_ec85999679c881919c9459cd74371270`.
- Successful deployment: `appgdep_6ad5790873708191b5c5fe79ab64cc33`.

### Version 9

The public Site was successfully published on October 18, 2026, as **version 9**
before this coordination update. The deployment operation confirmed success.
Public access was verified before publication and preserved.

Version 9 updates the overview and Work in progress with the agreed order
**Fluent, VarColor, Palette**. Fluent covers emphasis only; VarColor provides
the color variant, sentinel, constructor, fluent color operations, presence
aliases, and clearing. Palette subsequently extends VarColor, keeping the base
variant usable independently of palette enhancements. The website distinguishes
agreed API directions from unresolved overload details and implementation status.

The ConstColor section records the strict C++11 header syntax check at
`875850e5` with MSYS GCC 15.3.0, including its limits. It does not claim an
oldest working compiler release or a new runtime-suite/tablet/CI result.
See BuildAndTest.md for the exact command. No library candidate was integrated.

Validation for this content update checked all local links and anchors and
duplicate IDs. A local browser preview confirmed the revised content and the
Work in progress page layout. Integrated changes, demonstration assets, and
existing styles were preserved. No new library test run or device-layout check
was performed during publication.

Publication provenance:

- Site source commit: `b77dec75e16e47465aef4bae5da972327ffa5427`.
- Saved Site version: 9,
  `appgprj_6ad129c3e8408191b873a4e06e601c63~appgver_92e04a7a71908191bb380af87ad80dd1`.
- Successful deployment: `appgdep_6ad5750c24c881919b84b0642dc79468`.

### Version 8 and earlier

The public Site was previously published on October 18, 2026, as
**version 8**. The deployment operation confirmed success. Public access was
verified before publication and preserved; no audience change was requested.

Version 8 corrects the overview's "Terry's projects" link to
[https://terry.golubiewski.com/prj/](https://terry.golubiewski.com/prj/).
The old `/projects` destination returned 404. The live homepage links to
`prj/`, and the corrected destination returned HTTP 200 with the Projects
page title. This publication changed only that external URL; candidate and
integrated-change explanations and demonstration assets remain unchanged.

- [Overview and demonstration](https://fmtcolor.ardentokapi66.chatgpt.site/):
  distinguish current RGB-OR behavior from the proposed breaking Blend feature;
  record independent bold/faint bits, the deliverables' conceal/flash difference,
  and StyledText ownership of absolute and lazy nested blending.
- [Work in progress](https://fmtcolor.ardentokapi66.chatgpt.site/work-in-progress.html):
  all six approved proposals, Fluent then Palette first, later breaking order
  open, and StyledText dependencies under review. Planned work is distinguished
  from implemented candidates. ConstColor and TermColorValue link observed source
  tips and distinguish earlier reported PC/tablet passes from the later verified
  MSYS boundary-test run for TermColorValue. The other proposal entries are
  unchanged by version 7.
- [Integrated changes](https://fmtcolor.ardentokapi66.chatgpt.site/integrated-changes.html):
  unchanged explanations for the accepted BuildBash and Termux squash commits.

The accepted commits remain
[`f303af51`](https://github.com/tjgolubi/fmt/commit/f303af515b4a5295e67333d7697dd1c797b390d0)
and
[`1c61e624`](https://github.com/tjgolubi/fmt/commit/1c61e624ebe67a9ca6a2f1b6465d0ac4203f1c98).
No candidate was integrated or moved into Integrated changes during this update.
Version 5 previously published the accepted Termux explanation; the later
public-access change left that content version unchanged.

Version 7 changes only the TermColorValue WIP section: link pushed test commit
`4b8ae4eb`, describe the four numeric/rendering boundaries and black presence
checks, and record the verified MSYS color-test pass. A tablet run of that test
commit has not been reported. The candidate remains unintegrated and its
priority is unchanged. No new candidate or design decision was introduced.

Local HTML links and anchors were checked. The overview, Integrated changes,
other proposal sections, demonstration source, and expected output were
preserved in version 7. That publication did not rerun library tests or perform browser,
CI, example-compilation, or actual tablet-layout testing. Its PC test result
comes from the earlier focused test run, recorded in BuildAndTest.md.

Version 6 previously published the revised six-proposal development plan.

Publication provenance:

- Site source commit: `4f0076d6efb61dcf720f59ed2a02b633d53938df`.
- Saved Site version: 8,
  `appgprj_6ad129c3e8408191b873a4e06e601c63~appgver_5a7e1afa8d3c8191b0415e8ced2bcdcf`.
- Successful deployment: `appgdep_6ad563e89a14819189221a1771067add`.

Future APIs remain planned work. Approval to describe proposals does not
authorize their implementation, integration, or library pushes.

## Integrated changes

Provide detailed documentation and rationale for each project change commit on
our `FmtColor` development branch. Give each commit its own section, or a
separate page when lengthy. Link the exact accepted commit and explain:

- The problem and motivation.
- Design rationale, alternatives, and tradeoffs.
- Implementation details needed to understand the change.
- Examples and expected visible behavior.
- Compatibility effects and migration requirements.
- Relevant tests, platform results, and limitations.

The accepted squash commit is the natural unit. Upstream commits remain
upstream context rather than being presented as our extensions. If integration
history is rebased, refresh affected commit links.

This detailed explanation supplements the brief entry in
`FmtColorChangeLog.md`. Do not present an unaccepted branch as an integrated
change or substitute its development commits for the final accepted commit.

## Work in progress and branch approval

Include a development branch on the website only when Terry explicitly approves
that branch for inclusion. Approval to describe a branch does not accept its
implementation or authorize its integration. Experimental or "play" branches
must not appear merely because they exist or have been discussed.

Use conventional slash prefixes for development branches, such as `feat/`,
`fix/`, `doc/`, and `test/`, with other appropriate prefixes allowed. A prefix
describes purpose; it never grants website inclusion automatically.

For each approved proposal, explain its goals, current design, implementation
and testing status, compatibility effects, and known dependencies. For existing
remote branches, link the branch and identify the exact reviewed commit because
tips can advance. Label planned work without inventing source links or commits.

List the expected integration order as it becomes known. Label it provisional,
distinguish planning priority from technical dependencies, and leave undecided
positions explicit. Update the order when review or dependencies change.
Integration is conditional: a branch may change or never be accepted. Do not
present the order as fixed or promise release dates.

Terry approved Termux and ConstColor for inclusion on October 17. Termux is
now integrated. On October 18 he approved updating the website with all six
proposals from the planning discussion. The table below now reflects the revised
coordination plan, including ConvertColors, published as Site version 10 on
October 18, with accepted CleanUp and candidate status refreshed in
version 16 on October 20. TermColorValue and GitHubCI are now integrated:

| Priority | Proposal | Source / status |
| --- | --- | --- |
| 1 | `feat/Fluent` | Planned emphasis-only setters/getters. Local branch exists; its current implementation status requires source review. |
| 2 | `feat/VarColor` | Planned color variant, constructor, fluent color operations, presence aliases, and clearing; builds on the accepted sentinel. C++17+ variant APIs; existing older-mode APIs preserved. |
| 3 | `feat/Palette` | Planned cube/grayscale extension of VarColor. Preserve current operators and review discriminator `10` assumptions. |
| Order open | `feat/ConvertColors` | Planned branch for the existing explicit RGB-promotion design and specified Xterm terminal palette. Exact APIs and dependencies remain open; scope unchanged. |
| Later; order open | `feat/ConstColor` | Verified GitHub tip `ed21ec1fb95142263587d28d68d049187ed44366`, above historical `8026a7d3`; not yet rebased onto current `c89d6dc1`; namespace-constant breaking candidate with committed migration documentation and October 20 reported test pass. |
| Later; order open | `feat/Blend` | Planned explicit blending plus breaking strict composition for both `|` and `|=`. |
| After required foundations | `feat/StyledText` | Planned nested text, `absolute`, and lazy blending. Exact dependencies remain open. |

Implement Fluent first, VarColor on accepted Fluent, and Palette as an extension
of VarColor. This keeps VarColor usable independently of palette support.
ConvertColors names existing design work; its integration position remains open.
None
requires ConstColor or terminal-color renumbering. The
remaining ConstColor and Blend breaking branches have no agreed relative order. Do not infer acceptance
from the table's row order or claim StyledText requires every preceding branch.
Create planned branches only when work begins from the then-accepted baseline.
Link existing remote candidates; planned names need only page anchors until
their branches exist.

Historical candidate evidence below predates TermColorValue acceptance.
Terry's earlier PC/tablet pass reports for both implemented candidates did not
identify exact tested SHAs or provide fresh logs. Separately, TermColorValue's
new boundary checks passed in the assistant's MSYS color-test run and were
committed as `4b8ae4eb`. A tablet run of that new test commit and CI results
have not been reported. See BuildAndTest.md for the evidence and limits.
The October 19 branch documentation now supplies migration guidance, and Terry
reported successful compilation, tests, and documentation generation after his
revisions. No new tablet or CI result was established. See BuildAndTest.md for
the newer evidence separately from the October 18 results. Changelog preparation,
compatibility acceptance, and final integration review remain pending.

On October 20, both candidates were again rebased, all tests passed according
to Terry, and rewritten tips were pushed with `--force-with-lease`. At that time, the
verified tips were those recorded in BuildAndTest.md. These do not establish a new assistant runtime
or CI run; exact tested SHAs and configurations remain unreported. CleanUp is
accepted and described in Integrated changes, not a remaining candidate.

When a branch is accepted, move its explanation to integrated changes, update
it for the final accepted behavior and validation, and link the resulting
commit. Keep the work-in-progress list current.

## Publication before coordination updates

For agreed changes to the website and coordination set, publish and confirm
the website first, then update coordination documents with the actual version,
source commit, deployment result, and any remaining verification limits. Preserve
public access. Distinguish local edits from committed or pushed coordination
changes; the publication result does not imply a coordination Git push.

## Relationship to other documentation

[CppDesign.md](CppDesign.md) records forward intentions and open questions.
The website does not supersede it or establish implementation status by itself.
Source and tests establish implemented behavior. Explicit later decisions from
Terry take precedence over older draft wording and should be recorded.

The deliverable `/doc/` remains the user documentation entry point. The
`/doc/color/` pages now document the existing `fmt/color.h` API: overview,
colors, emphasis, `text_style`, `styled()`, formatting, printing, and a separate
named-color swatch page. Doxygen comments in `include/fmt/color.h` supply the
reference material through the existing MkDocs C++ handler. `doc/api.md` links
readers to this detailed guide. These pages describe the baseline API rather
than the proposed extensions.

The accepted `doc/BaseColor` integration is
[`f913e54c`](https://github.com/tjgolubi/fmt/commit/f913e54cf41c8aafae8ce951817008e7969d4c59).
The [GitHub Pages deployment](https://github.com/tjgolubi/fmt/actions/runs/39211196126)
completed successfully. The public entry point is
[https://tjgolubi.github.io/fmt/color/index.html](https://tjgolubi.github.io/fmt/color/index.html).
The Sites website links to this guide rather than hosting a second copy.

`.github/workflows/fmtcolor-pages.yml` publishes after pushes to `FmtColor`;
feature branches and pull requests do not publish. `support/mkdocs-pages.yml`
inherits the normal configuration and supplies the fork's URL and links.
Generated HTML is uploaded as a Pages artifact, not committed to source.
GitHub Pages uses the GitHub Actions source and its `github-pages` environment
allows `FmtColor`. See `support/fmtcolor-pages.md` in the library repository
for setup and [BuildAndTest.md](BuildAndTest.md) for build details. The VPS
remains a backup hosting option; no VPS deployment was performed.

Sites should explain the technical rationale behind changes beyond the
user-focused documentation. Link to usage documentation rather than maintaining
competing copies. If adding developer material under `/doc/color/`, choose its
scope deliberately to avoid unnecessary duplication.

Upstream documentation contains links to upstream-hosted pages. Review links
introduced or changed for our extensions, but do not turn color work into a
general repair of upstream's website.

## Demonstration code and expected output

Keep the sample program and its expected styled output together. Clearly label
an example that uses planned APIs rather than the current implementation.
Retain downloadable code and output where useful.

The website example targets the second deliverable: the styling library built
on standard C++ formatting, without fmtlib's formatting engine or unrelated
extensions. It retains `#include <fmt/color.h>` and the project's styling API in
the `fmt` namespace, while using standard formatting output such as
`std::println`. The include path selects the second deliverable's installation;
the header spelling does not imply a dependency on fmtlib. Explain this choice
and keep proposed names consistent with the reviewed API. See
[CppDesign.md](CppDesign.md#source-compatibility-between-deliverables).

The same styling source is intended to work with either deliverable. An example
using `std::println` requires standard formatter support in both implementations;
the {fmt} formatter alone does not establish that support. Treat the example as
a future acceptance target until it is implemented and tested against both.

The existing demonstration includes nested emphasis/colors, absolute styles,
and `gray_color{12}`. Preserve those cases when revising its presentation.
Expected output can be a screenshot or a faithful styled HTML rendering;
palette assumptions should be stated where they affect appearance.
The output need only be visually equivalent: identical SGR sequences are not
required. Distinguish legacy implementation output from tests of the new code.

The updated overview describes borrowed text fragments, optional borrowed
subtrees, and owned child nodes. It distinguishes strict `text_style::operator|`
composition from fluent replacement and sequential ANSI SGR updates. Unspecified
channels retain surrounding values in ordinary rendering; an absolute style
starts its terminal SGR sequence with `ESC[0;...m` to reset before applying its
attributes. Strict composition for both `|` and `|=` is a planned breaking change in
`feat/Blend`, introduced together with explicit blending operations. Current
{fmt} still ORs overlapping RGB colors and rejects same-channel terminal-color
conflicts. Preserve this distinction in the overview. Bold and faint are
independent with no library precedence in both deliverables. {fmt} retains
conceal without flash; our C++23-compatible library design replaces conceal
with flash. `absolute` and lazy nested blending belong to StyledText. Exact
style-blending rules remain open.

## Maintenance from desktop, Android, or the web

When Terry specifically requests a website update, the assistant handles the
required source edits and publication following agreed content decisions.
Do not change the website merely because related project work suggests an
update; Terry clarified on October 19 that changes require a specific request.
Explain
the resulting content changes and provide usable links. Terry does not need to
manage the website's source files, build tools, or publishing mechanics.
Ask for missing content decisions when needed, and explain briefly if a system
permission prompt is unavoidable. Website authorization does not override the
separate approval requirements for library merges or pushes.

When continuing from another conversation or device:

1. Read the current `Website.md` and relevant linked coordination documents from
   `tjgolubi/FmtColorGpt`. Identify any unavailable source material rather than
   assuming an old chat attachment is current.
2. Locate the existing FmtColor Site using its URL above. Retrieve its latest
   source and current published version through the available Sites tools
   before editing. The Site has a shared source repository; do not overwrite it
   using an old local checkout, downloaded page, Library capture, or remembered
   conversation contents. Do not create a replacement Site for an update.
3. Apply the requested or agreed change, preserving the Site's identity, access
   settings, demonstration, and unrelated content. Verify branch and commit
   claims against current GitHub source. Include only approved candidate
   branches and distinguish proposed, implemented, tested, and integrated work.
4. Check the affected content, layout, and links with the tools actually
   available. Save and publish through Sites when authorized; report success
   only after the tool confirms deployment. Report validation limits honestly.
5. Return the affected page links and a concise description of what changed.
   Coordinate substantive decisions through these documents so another
   assistant can continue from the same current state.

Do not assume that the Android ChatGPT app has the same Sites tools or interface
as the desktop or web session. If the Android assistant can access and update
the existing Site, it may complete the work there without asking Terry to
install local development tools or transfer this PC's working files.

If the assistant cannot retrieve the current source, make the requested change,
or confirm publication with its available tools, it must say which step it
cannot complete. It is acceptable to direct Terry to
[Sites in the ChatGPT web interface](https://chatgpt.com/sites), signed into the
same account, while he is working remotely. Provide a short handoff containing
the existing Site URL and requested edit, and tell the next assistant to read
the latest coordination documents and retrieve the latest Site source first.
Do not claim that a discussion, generated file, or proposed edit updated the
live website. Do not promise that switching interfaces guarantees tool access.

Keep credentials out of documents, chat output, repository files, and handoffs.
The published Site remains available independently of this PC's local checkout
or preview server. Avoid concurrent edits from multiple sessions; retrieve the
latest source again when resuming after another assistant's update.

## Windows publishing environment

Use native Windows Git for the separate Sites checkout when running its Node
source helper on this PC. Use MSYS2 Git for Terry's `fmt` and `FmtColorGpt`
repositories, as required by Workflow.md. Keep the environments separate.

Version 13 exposed three distinct setup issues:

1. The Sites packager invokes `bash`; the initial process PATH lacked Bash.
   Source commit/push succeeded before archive creation failed. Preserve that
   committed source and retry packaging rather than creating a replacement Site.
2. Prepending MSYS2's `usr/bin` also selected MSYS2 Git, which returned a path
   such as `/c/Users/...`. The Windows Node helper could not use that path for
   checkout validation. Native Windows Git must remain ahead of MSYS2 tools.
3. GNU tar interpreted the colon in a Windows archive path as a remote-host
   separator. Process-local `TAR_OPTIONS=--force-local` fixed that archive step.

The successful PowerShell setup put the bundled native Git directory first,
then `C:\msys64\usr\bin`, while retaining the remaining PATH. It set
`$env:TAR_OPTIONS = '--force-local'` and ran the normal Sites source helper
with the existing Site ID, opening result, and archive destination. Native Git
was observed under Codex's runtime `dependencies/native/git/cmd`; locate the
current runtime path instead of assuming that installation path is permanent.
Use forward slashes in an absolute Windows archive path, such as `C:/.../site.tar.gz`.
Scope these environment changes to the helper process; do not change global
Git settings or the source repositories' Git identity/SSH setup.

Continue using the supported Sites helper to own source synchronization,
commit/push, and packaging. Pass its verified project ID, commit, and archive
to the native save/deploy operations. The successful retry reused source commit
`f2c34b24`, saved version 13, and deployed with public access unchanged.
Do not claim deployment until the native tool confirms success. A Git push or
a successful archive alone does not publish the website. Keep credentials in
memory and pass them through the helper's hidden stdin, never files or output.

## Access and external links

On October 18, 2026, Terry explicitly requested public access. The Sites access
operation confirmed `access_mode=public`, policy revision 2. Anyone with the
URL can visit the website. Routine maintenance must preserve this public
audience; changing it again requires Terry's explicit instruction. Recheck
current access when resuming work rather than relying on older private-site
observations. A link on Terry's projects page does not itself change access.

Terry intends to link from his personal website's projects page. Personal
website changes belong in their own work context; they are not library changes.

Do not include private chat material or unreviewed coordination details merely
because they appeared in discussion. Public explanations should stand on their
own and describe the project accurately.

## Ongoing maintenance

Review website impact when accepting a branch. Prepare the technical explanation
alongside the change, and identify its accepted commit after integration.
A small infrastructure/test workaround may need only a short explanation.
Do not update unrelated pages just to show activity.

If a proposal changes or is abandoned, update its page so readers do not mistake
it for the current plan. Preserve distinctions among proposed, implemented,
tested, and integrated behavior.

The content can later be migrated to MkDocs/Material. That is a migration task,
not an automatic conversion or a reason to copy the Sites implementation into
the library now.
