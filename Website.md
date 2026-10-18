# Website

[Document map](README.md) · [C++ design](CppDesign.md) · [Workflow](Workflow.md) · [Build and test](BuildAndTest.md)

**Working draft, updated October 18, 2026.** This document records intended
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

The existing Site was updated on October 17, 2026. Its published pages are:

- [Overview and demonstration](https://fmtcolor.ardentokapi66.chatgpt.site/):
  the two sequential deliverables, their shared styling API and header path,
  the proposed example, and the styling and rendering design.
- [Integrated changes](https://fmtcolor.ardentokapi66.chatgpt.site/integrated-changes.html):
  the entry point for detailed documentation of accepted project commits.
- [Work in progress](https://fmtcolor.ardentokapi66.chatgpt.site/work-in-progress.html):
  the approved candidate branches, their rationale, status, dependencies, and
  provisional expected integration order.

At that review, `FmtColor` was at upstream baseline
`b4ffa39b9994c961418406f47358791d89a5aa48`, with no project extension commits yet
integrated. The integrated-changes page states this explicitly. The Site's
access was verified as owner-only. Recheck source history and access settings
when updating these observations.

The published example source and expected ANSI output were preserved. The
updated pages and local links were checked in a desktop browser and through
local validation; tablet layout and compilation of the proposed example were
not verified in that update.

### Publication pending after the first integration

On October 18, 2026, `build/BuildBash` was accepted as the first project squash
commit on `FmtColor`:
[`f303af51`](https://github.com/tjgolubi/fmt/commit/f303af515b4a5295e67333d7697dd1c797b390d0),
`build: add shared build script and changelog configuration`. It is pushed and
verified. It adds the shared MSYS/Termux build script, git-cliff configuration,
initial fork changelog, and Python bytecode ignore rule. PC build/test/docs
validation and PC/tablet git-cliff verification are recorded in BuildAndTest.md.

The website explanation is prepared but has not been published. The last
published pages described above therefore lag the repository. Next publication
should add the accepted commit's rationale and validation limits to integrated
changes, link its exact commit, and update the approved WIP status and order.
Do not describe this pending publication as already deployed. The accepted
BuildBash change belongs in integrated changes; it does not need a candidate
WIP entry. No new candidate-branch website approval is recorded here.

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

For each approved branch, explain its goals, current design, implementation and
testing status, compatibility effects, and known dependencies. Link the branch
and identify the exact reviewed commit because branch tips can advance.

List the expected integration order as it becomes known. Label it provisional,
distinguish planning priority from technical dependencies, and leave undecided
positions explicit. Update the order when review or dependencies change.
Integration is conditional: a branch may change or never be accepted. Do not
present the order as fixed or promise release dates.

As of October 17, 2026, Terry approved these two branches for inclusion:

| Branch | Reviewed tip | Expected order and dependencies |
| --- | --- | --- |
| `fix/Termux` | `671a9f357b0af64c0c5f3ed6733060dae7d8fc49` | Next candidate after BuildBash; rebased onto accepted `f303af51`. Validation after rebase is pending. |
| `feat/ConstColor` | `87b6cde3224fc368b63cd44545f569b49d7b027c` | Tentatively after Termux; that position remains to be confirmed. No dependency on Termux was found in the reviewed history. |

On October 18, the local and recorded remote `fix/Termux` tips matched the new
commit above; its merge base with `FmtColor` is accepted `f303af51`. The diff
still contains the same three test/build files and 36 insertions. The table
records current repository status for the next website update, not a claim
that the published page already has these links. `feat/ConstColor` retains its
previously reviewed baseline and tip until rechecked. No other candidate branch
has website inclusion approval recorded here.

When a branch is accepted, move its explanation to integrated changes, update
it for the final accepted behavior and validation, and link the resulting
commit. Keep the work-in-progress list current.

## Relationship to other documentation

[CppDesign.md](CppDesign.md) records forward intentions and open questions.
The website does not supersede it or establish implementation status by itself.
Source and tests establish implemented behavior. Explicit later decisions from
Terry take precedence over older draft wording and should be recorded.

The deliverable `/doc/` remains the user documentation entry point. Introduce
`/doc/color/` for user-facing guidance on the library extensions, with examples,
screenshots where useful, and developer material where appropriate. Use the
existing Doxygen/MkDocs/Material tooling. See [BuildAndTest.md](BuildAndTest.md).
Treat this folder as planned until it exists on the relevant branch; avoid
publishing links to nonexistent documentation.

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
attributes. Possible blending through `operator|=`, including automatic RGB
promotion, remains an exploratory idea rather than an agreed API. Keep those
distinctions when revising the explanations.

## Maintenance from desktop, Android, or the web

Terry authorizes the assistant to handle routine website maintenance and
publication during project work, following agreed content decisions. Explain
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

## Access and external links

The website was verified owner-only on October 17, 2026. Its current access
setting must be checked before describing it as public or usable by friends.
A link on Terry's projects page does not itself change Site permissions.
Routine authorized publication preserves the audience; changing access requires
Terry's explicit instruction.

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
