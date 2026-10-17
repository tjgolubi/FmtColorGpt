# Website

[Document map](README.md) · [C++ design](CppDesign.md) · [Workflow](Workflow.md) · [Build and test](BuildAndTest.md)

**Initial working draft: October 16, 2026.** This document describes intended
website organization. It does not claim the current Site already has these pages.

## Purpose

The [FmtColor Sites website](https://fmtcolor.ardentokapi66.chatgpt.site) explains
the project and its changes to interested readers, including friends following
the work. It is separate from the library repository and this coordination
repository. Its implementation does not belong in the {fmt} fork.

The website is not a chronological progress blog. Organize technical material
around accepted integration commits and proposed branches.

## Integrated changes

Provide a page devoted to commits on our `FmtColor` development branch. Give
each project change commit its own section, or a separate page when lengthy.
Link the exact commit and explain:

- The problem and motivation.
- Design rationale, alternatives, and tradeoffs.
- Implementation details needed to understand the change.
- Examples and expected visible behavior.
- Compatibility effects and migration requirements.
- Relevant tests, platform results, and limitations.

The accepted squash commit is the natural unit. Upstream commits remain
upstream context rather than being presented as our extensions. If integration
history is rebased, refresh affected commit links.

## Proposed branches

Provide another page for branches not yet integrated into `FmtColor`. Explain
their goals, current design, relevant implementation/testing status, and known
dependencies. Link the branch and distinguish a proposal from accepted behavior.

Integration is conditional: a branch may change or never be accepted. Do not
imply a fixed integration order or promise release dates. State dependencies
explicitly when one branch requires another.

## Relationship to other documentation

[CppDesign.md](CppDesign.md) records forward intentions and open questions.
The website does not supersede it or establish implementation status by itself.
Source and tests establish implemented behavior.

The deliverable `/doc/` remains the user documentation entry point. Extend it
under `/doc/color/` with detailed explanations of the library extensions,
examples, screenshots where useful, and developer material where appropriate.
Use the existing Doxygen/MkDocs/Material tooling. See
[BuildAndTest.md](BuildAndTest.md).

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

The existing demonstration includes nested emphasis/colors, absolute styles,
and `gray_color{12}`. Preserve those cases when revising its presentation.
Expected output can be a screenshot or a faithful styled HTML rendering;
palette assumptions should be stated where they affect appearance.
The output need only be visually equivalent: identical SGR sequences are not
required. Distinguish legacy implementation output from tests of the new code.

## Access and external links

The website has previously been owner-only. Its current access setting must be
checked before describing it as public or usable by friends. A link on Terry's
projects page does not itself change Site permissions. Publishing or changing
access is a separate explicit action.

Terry intends to link from his personal website's projects page. Personal
website changes belong in their own work context; they are not library changes.

Do not include private chat material or unreviewed coordination details merely
because they appeared in discussion. Public explanations should stand on their
own and describe the project accurately.

## Maintenance

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
