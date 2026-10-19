# C++ Design

[Document map](README.md) · [Workflow](Workflow.md) · [Build and test](BuildAndTest.md) · [Website](Website.md)

**Working draft, updated October 18, 2026.** Forward intentions, not an
implementation authorization or a claim that all features exist.

## Goals and boundaries

Provide composable text styles, nested styled text, terminal rendering, and
textual and binary style serialization. User-configurable themes and a future
Taskwarrior integration are motivating uses. Application restrictions belong
in applications rather than in the general library.

The ultimate target is a color/text-style extension to the C++26 standard
library. There are two sequential deliverables:

1. A focused color/text-style extension to {fmt}, offered to the upstream
   project. Its users can provide feedback on the design and implementation.
2. A refactor of our library onto standard C++26 formatting. This deliverable
   excludes {fmt}'s support for pre-C++23 compilers and its unrelated extensions.

These are successive stages, not parallel development paths. C++23 is our
current implementation baseline. We do not assume future language or library
features are available in today's toolchains.

### Source compatibility between deliverables

Both deliverables retain the public header path `<fmt/color.h>` and the
project's styling API in the `fmt` namespace. Users select the corresponding
installation through their include path (`INCLUDE_PATH`); changing deliverables
should not require changing their styling source code.

The second deliverable implements the styling facilities on standard C++
formatting, without depending on fmtlib's formatting engine, compatibility
machinery, or unrelated extensions. Keeping the header path and namespace does
not imply a fmtlib dependency. This source-compatibility intent covers the
project's styling API, not arbitrary fmtlib-specific application code. Exact
proposed APIs remain subject to review; no binary-compatibility guarantee is
established by this decision.

The shared-source target is styling code using symbolic color names and the
project's styling operations. Underlying numeric terminal-color values may
differ between deliverables; retaining the same header path and API does not
promise identical enum representations.

The emphasis vocabulary also has an explicit deliverable difference: conceal
remains in {fmt}, while our C++23-compatible library replaces it with flash.
Shared-source intent applies to common styling facilities; it does not make
those two attributes interchangeable.

During the {fmt} stage, preserve upstream headers, compatibility macros,
formatting machinery, tests, and build configuration except for agreed changes.
We are not replacing its formatting engine or broadly rewriting it in C++23.
Initially, keep production changes in `include/fmt/color.h` where practical;
affected tests and documentation may also need changes. Revisit that boundary
when a feature actually requires it. These preservation constraints do not
require carrying {fmt}'s compatibility machinery or unrelated extensions into
the later standard-library refactor.

Modified source follows the style already present in that source. The eventual
independent library follows standard C++ library naming and API style.

## Proposed branches and integration priorities

Agreed planning decisions, October 18, 2026: prioritize non-breaking additions
for the {fmt}-compatible deliverable. Passing tests does not by itself establish
source/API, binary/ABI, or stored-data compatibility. Existing behavior remains
until an explicitly reviewed breaking feature changes it.

| Priority | Proposed branch | Scope and compatibility |
| --- | --- | --- |
| 1 | `feat/Fluent` | Add emphasis-only fluent setters and boolean getters for all eight existing attributes. Preserve unrelated bits, colors, existing APIs, rendering, layout, and supported language modes. |
| 2 | `feat/VarColor` | Add `no_color_t`/`no_color`, C++17+ `var_color`, color constructors, fluent foreground/background setters and getters, presence aliases, and clearing. Build on accepted Fluent. |
| 3 | `feat/Palette` | Extend VarColor with `cube_color` and `gray_color`, including free and fluent foreground/background support. Use discriminator `10` while preserving existing encodings and composition behavior. Intended as a non-breaking addition. |
| Order open | `feat/ConvertColors` | Give the existing explicit color-to-RGB conversion design and specified Xterm `terminal_palette` a dedicated branch. Exact APIs and dependencies remain open; no new conversion scope. |
| Later; relative order open | `feat/ConstColor` | Replace the named-color enum with RGB constants and make packed RGB conversions explicit. Breaking candidate; implemented on its branch. |
| Later; relative order open | `refactor/TermColorValue` | Change terminal-color values from SGR numbers to 0–15 while preserving symbolic styling and emitted SGR. Breaking numeric-value contract; implemented on its branch. |
| Later; relative order open | `feat/Blend` | Introduce RGB and style blending together with breaking strict-composition semantics for both `operator|` and `operator|=`. Include migration guidance. |
| After required foundations | `feat/StyledText` | Structured nested text, `absolute`, and lazy blending during rendering. Exact dependencies and compatibility effects remain to be reviewed. |

The agreed additive order is Fluent, VarColor, then Palette. Implement VarColor
on the accepted Fluent baseline; Palette then extends its color alternatives
and operations. VarColor remains usable without Palette, so users adopting the
color variant alone do not need to remove palette-specific handling. None of
these features requires ConstColor or renumbered terminal colors. Preserve
existing operator behavior until Blend deliberately changes it. StyledText
uses style semantics and color support; its precise branch dependencies remain
open. Binary serialization and the later standard-format refactor are not
prerequisites for these additions.

`feat/ConvertColors` names the existing conversion design, with no newly agreed
integration position. Conversions involving palette types require those types
when implemented; exact branch dependencies remain subject to review. It is
separate from the color types and from Blend operations using RGB promotion.

The revised eight-proposal plan was approved for website inclusion and published
as public Site version 10, not accepted for integration. Create branches
when implementation begins from the accepted baseline at that time; do not
create speculative branches merely to populate the plan. A local Fluent branch
was created early at the unchanged baseline; that does not mean implementation
has begun. ConstColor and TermColorValue have Terry-reported PC and tablet
passes, recorded in [BuildAndTest.md](BuildAndTest.md), and still need acceptance
review. Their successful tests do not move them ahead of non-breaking work.

## Colors

The breaking `feat/ConstColor` candidate replaces `enum class color` with
`namespace color` containing named RGB
constants. Use decimal `rgb{red, green, blue}` initializers, preserving the
existing names and values. Keep the constants in `color.h` for this change.
The additions support C++11: `FMT_INLINE_VARIABLE` becomes empty before C++17,
leaving namespace-scope constexpr RGB constants. A strict C++11 header syntax
check with MSYS GCC 15.3.0 passed at `875850e5`; this does not establish the
oldest working compiler release. See BuildAndTest.md for the command and limits.
This preserves common expressions such as `fg(color::red)`, but intentionally
breaks code that uses `fmt::color` as a type or depends on its enum conversions.

Provide explicit construction and conversion between `rgb` and
`std::uint32_t`. The packed representation is `0x00RRGGBB`; construction ignores
the high byte. Retain {fmt}'s existing compatibility provisions in this code.

Keep terminal, indexed/palette, gray, and RGB colors conceptually distinct.
Zero must not accidentally mean an unspecified color when it denotes a valid
color. The separate `feat/Palette` proposal adds `cube_color` (6×6×6 cube) and
`gray_color` (24-level grayscale ramp) for indexed-palette output. Preserve
existing `terminal_color` values and encoding in this additive feature. The
current `text_style` discriminator `10` is unused; the proposed palette payload
identifies the indexed color. Presence checks, classification, getters, OR
overflow arithmetic, and SGR generation currently rely on the old discriminator
invariants and must be reviewed together. Test foreground/background
discriminator combinations while preserving existing user-visible behavior.
Range validation, exact constructors, and RGB promotion APIs remain open.
Keeping object layout unchanged alone does not prove mixed-version ABI safety.

The separate breaking `refactor/TermColorValue` candidate changes numeric values
from 30–37 and 90–97 to 0–15 and maps them back to SGR during rendering. Code
using numeric conversions or persisted values needs migration; symbolic styling
remains unchanged. This representation change is negotiable for
the {fmt} contribution. If upstream requires the existing numeric SGR values,
such as `std::to_underlying(terminal_color::black) == 30`, retain those values in
the {fmt}-based deliverable and defer the representation change to the
independent C++26-only deliverable. Terry accepts that compatibility requirement
if upstream requests it; the representation change is not a prerequisite for
offering the other styling extensions.

This is a contingency for upstream review, not a claim that upstream has
already requested or accepted either representation. Code using symbolic
terminal-color names and styling operations should work with either encoding.

### Color conversions: `feat/ConvertColors`

The planned `feat/ConvertColors` branch owns the existing conversion design
below, including the specified terminal palette. Assigning the branch name
adds no conversion scope or API decisions. Explicit RGB promotion is planned;
nearest-palette demotion remains a possible later extension.

Plan explicit promotion from every supported color type to RGB using the
following `terminal_palette`.  `terminal_palette` was selected because Xterm is
a commonly-used standard terminal.  At compile-time, we can't know user's actual
terminal theme. No implicit promotion is required for blending.  Explicit
demotion to the nearest palette color is a possible later feature using
euclidian distance to the nearest color in the three-dimensional RGB color
space.

```cpp
// This terminal color palette is based on the Xterm default.
inline constexpr std::array<rgb, 16> terminal_palette = {{
  // 0–7 normal
  rgb{  0,   0,   0},  // black
  rgb{205,   0,   0},  // red
  rgb{  0, 205,   0},  // green
  rgb{205, 205,   0},  // yellow
  rgb{  0,   0, 238},  // blue
  rgb{205,   0, 205},  // magenta
  rgb{  0, 205, 205},  // cyan
  rgb{229, 229, 229},  // white

  // 8–15 bright
  rgb{127, 127, 127},  // bright black
  rgb{255,   0,   0},  // bright red
  rgb{  0, 255,   0},  // bright green
  rgb{255, 255,   0},  // bright yellow
  rgb{ 92,  92, 255},  // bright blue
  rgb{255,   0, 255},  // bright magenta
  rgb{  0, 255, 255},  // bright cyan
  rgb{255, 255, 255}   // bright white
}}; // terminal_palette
```

Plan this standalone RGB operation:

```cpp
constexpr auto blend(rgb a, rgb b) noexcept -> rgb;
```

It performs bitwise OR on corresponding red, green, and blue channels, not
averaging or alpha compositing. Color blending operates on RGB values; other
supported color categories can be promoted for explicit style blending. This
operation belongs to `feat/Blend` together with the operator changes below.

## `text_style`

The proposed compact representation is a 64-bit style with foreground and
background modes and payloads, emphasis, an `absolute` flag, and reserved bits.
The modes include unspecified, terminal, indexed color, and RGB. The exact bit
allocation is an implementation matter to verify against current code.
Do not expose the in-memory representation as a permanent public contract.

### Composition and replacement

The additive work is split into emphasis-only `feat/Fluent` and color-focused
`feat/VarColor`, followed by `feat/Palette`. Existing getters, free `fg()`/`bg()`,
constructors, operators, rendering, and object layout retain their behavior.
Keep `constexpr` and upstream language-compatibility provisions where feasible.

Fluent adds `bold(bool enabled = true) -> text_style&` and
`is_bold() const -> bool`, with the same pattern for `faint`, `italic`,
`underline`, `blink`, `reverse`, `conceal`, and `strikethrough`. Setters mutate
the object, return `*this`, and change only their own bit. Boolean getters
report false for unset bits, including an empty style. These operations are
`noexcept`, retain supported older-language modes, and use the appropriate
upstream constexpr macros. Fluent adds no foreground/background operations.

### Variant color API

VarColor introduces public `fmt::no_color_t` and `fmt::no_color` across supported
language modes, using upstream compatibility provisions for the constant.
For C++17 and later, provide:

```cpp
using var_color = std::variant<no_color_t, terminal_color, rgb>;
```

The first alternative ensures `var_color{}` means unspecified. No color is
distinct from RGB black or terminal black; do not use `std::monostate` as the
public sentinel. Guard the variant and variant-based APIs for C++17+, including
the required standard header. Existing color APIs remain available in older modes.

The agreed constructor direction is:

```cpp
text_style(var_color fgc, var_color bgc = no_color,
           emphasis emp = emphasis{});
```

Keep the first argument required so the existing default and emphasis
constructors remain unambiguous. Background-only construction uses
`text_style{no_color, terminal_color::blue}`. Before ConstColor is integrated,
named enum colors require explicit wrapping, for example `rgb{color::red}`;
do not add a direct named-enum alternative to the variant.

Provide fluent member `fg()`/`bg()` setters and const value getters, plus
`has_fg()`/`has_bg()` aliases for existing presence checks. Color setters replace
only their own channel and return `text_style&`; preserve the other channel and
all emphasis. The variant getters report `no_color_t` for unspecified channels
without requiring a presence check. Existing `get_foreground()` and
`get_background()` keep their existing preconditions and return types.
Passing `no_color` clears the selected channel to unspecified; it does not
store a terminal-reset instruction. The sentinel and clearing syntax should
also work in older modes without exposing `std::variant` there. Exact overloads,
including ambiguity avoidance for RGB/terminal arguments, remain subject to
implementation review. Separate `clear_fg()`/`clear_bg()` convenience methods
have not been settled.

Make the private packed-value constructor explicit with VarColor and adjust
affected internal construction. This related cleanup needs no separate branch.
Palette subsequently adds its alternatives and support to this color API.

Current {fmt} `operator|` and `operator|=` OR overlapping RGB bits. They reject
a terminal color overlapping another specified color in the same channel;
terminal foreground plus background or emphasis still composes normally.
Preserve this behavior through Fluent, VarColor, and Palette.

The breaking `feat/Blend` proposal introduces standalone RGB `blend()` and
fluent `text_style::blend(text_style)` together with strict composition for both
`operator|` and `operator|=`. Independent components such as
`fg(red) | bg(blue)` remain valid. Two specified colors in the same channel are
rejected, including identical colors. Intentional old RGB-OR blending migrates
to explicit blending operations in the same feature.

Style blending uses the RGB operation and can promote other supported color
categories to RGB. Its return type, unspecified-channel behavior, exact
promotion/conflict rules, emphasis handling, and interaction with `absolute`
remain to be specified. The public operators do not enable a hidden rendering
blend policy. Lazy nested-text blending belongs to StyledText.

### Emphasis

For the {fmt}-compatible deliverable, retain the existing emphasis set and
names, including `conceal`, and do not add `flash`. For our C++23-compatible
library design, replace conceal with flash; this does not change the eventual
standard C++26 target or authorize flash in the {fmt} deliverable.

Bold and faint are independent in both deliverables. A setter toggles its own
bit without clearing the other; getters report each bit. When both are set,
render both without library-imposed precedence. Do not promise a uniform
combined appearance across terminals. This supersedes the former exclusive
setter and bold-precedence policy.

Blink/flash pairing and possible emission of both codes remain design questions
only for our library, where flash is planned. They do not apply to the
{fmt}-compatible emphasis API.

### Absolute styles

`absolute` is part of `feat/StyledText`, not Fluent or a standalone feature.
It acquires meaning through nested rendering. An absolute style is protected
from enclosing style changes. At its boundary,
rendering starts from terminal defaults rather than inheriting the enclosing
style. Descendants can inherit the absolute node's own style normally.
Enclosing blending must not penetrate this boundary.

## `styled_text`

This is the planned `feat/StyledText` feature; upstream {fmt} does not provide
it. Its scope includes nested structure, `absolute`, and lazy blending during
rendering. Initial work modifies the existing `text_style` first; the exact
required feature dependencies remain to be confirmed.

The current design intention is a `text_style` plus three content alternatives:

```cpp
std::variant<
    std::string_view,
    const styled_text*,
    std::vector<styled_text>
>
```

- A string view borrows immutable text characters without copying them.
- A pointer borrows one existing subtree, allowing an enclosing style to be
  applied without copying that subtree.
- A vector owns child nodes for concatenation and parser-created structure.
  Those nodes can themselves contain any of the three alternatives.

Appending new styled fragments is the primary use case. Design this path to
build owned child vectors efficiently and move temporary nodes into them.
Copying an owned vector copies its child nodes recursively, but string views
still refer to the original characters. Copying a borrowed-subtree node copies
the pointer, not its target. Moving should transfer owned storage without
deep-copying the tree.

Applying an enclosing style preserves the enclosed styles. Composition records
structure and intent; the renderer resolves effective styles later. It does
not eagerly rewrite the children or store embedded SGR as the normal
representation.

This is a design intention to try in practice, not a commitment to retain the
representation regardless of implementation experience. If borrowing makes
implementation or usage too difficult, fall back to:

```cpp
std::variant<std::string_view, std::vector<styled_text>>
```

That fallback still borrows characters, but owns every child subtree. Wrapping
an lvalue would copy its nodes; transferring an rvalue could move them.

### Lifetimes and construction

Referenced characters must remain valid for every use of the structure,
including when child nodes are owned. A borrowed subtree must also outlive its
use and remain at a stable address. Moving, destroying, or relocating its target
can invalidate a pointer. A pointer to const prevents modification through that
pointer, but does not provide ownership or prevent modification through another
reference. Borrowed relationships must not form cycles.

The intended constructor directions are:

```cpp
constexpr styled_text() noexcept = default;
constexpr styled_text(const styled_text&) = default;
constexpr styled_text(styled_text&&) = default;
constexpr explicit styled_text(std::string_view, text_style = text_style{}) noexcept;
constexpr styled_text(const styled_text& child, text_style style);
constexpr styled_text(styled_text&& child, text_style style);
```

Wrapping an lvalue borrows its address; wrapping a temporary or transferred
rvalue must instead own the child, moving it into child storage. Never retain a
pointer to a constructor parameter or temporary that will be destroyed.
The required style argument keeps wrapping distinct from copy/move
construction. Exact overloads and exception specifications will be reviewed
during implementation.

### Appending and parsing

The proposed append behavior can be illustrated by:

```cpp
auto rval = styled_text{"plain1 "};
rval += styled_text{"red ", fg(terminal_color::red)};
rval += "plain2";
```

The first append changes the root from a text view into an owned child vector
containing the original plain fragment and the moved red fragment. The next
append adds a plain fragment. The result owns three child nodes, while their
characters remain borrowed. No borrowed-subtree pointer is required for this
example. Preserve the existing fragment styles when forming the enclosing
sequence.

The SGR parser likewise owns the nodes it creates. Its text fragments can be
string views into the original input, excluding the SGR sequences; keeping the
input valid preserves those characters. Owning the parser-created nodes avoids
pointers to parser-local objects that would dangle after return. If parsing ever
requires transformed character storage, that storage needs a separate lifetime
solution. Exact append overloads and parser APIs remain open.

Direct construction supplies the new styling API. Using `styled()` with a
`styled_text` in the usual formatting context remains supported.

### Lazy blending

As part of StyledText, plan this member operation:

```cpp
styled_text& blend() noexcept;
```

It enables blending of the current node's style with enclosed styles during
rendering and returns `*this`. It changes only the current node's blending
policy; it does not traverse or modify its children. Absolute boundaries remain
protected. Whether the policy is stored in a private style bit or in a private
`styled_text` member is an implementation detail.

For example:

```cpp
styled_text wrapper{child, style};
wrapper.blend();
```

In this lvalue-wrapping example, the child must outlive the wrapper's use;
the wrapper uses the borrowed-subtree pointer. Owning construction and appending
may allocate child storage. Enabling the policy through `.blend()` only changes
the current node and does not allocate or copy children.

Do not provide the previously proposed free `blend(styled_text, text_style)`
operation. Explicit RGB `blend(rgb, rgb)` and fluent
`text_style::blend(text_style)` remain planned.

The renderer converts other color types to RGB before blending. Terminal colors
use the fixed `terminal_palette` above; cube and gray colors use their conversion
formulas.

The motivating use is text containing individually emphasized or colored
phrases, then an enclosing paragraph background, and later an enclosing emphasis
style (flash only in our library design). Preserve the nested structure
throughout, with absolute text protected.

### Plain text and SGR input

Plain-text construction does not scan for or reject embedded SGR sequences.
Callers supply plain text; when input may contain SGR, they call the explicit
SGR parser first. No separate unchecked constructor is needed merely to bypass
an SGR validation scan.

## Rendering and ANSI parsing

Keep three responsibilities distinct:

1. User-facing `render()` produces a string containing the project's extended
   SGR push/pop markers. It serves the former Colorize/ResetColorize role.
2. A terminal formatter directly emits ordinary ANSI SGR while resolving the
   tree. It does not call `render()` or require an extended-SGR intermediate.
3. An ANSI parser consumes ordinary SGR and the extended push/pop markers to
   construct a structured `styled_text`.

The legacy markers are `ESC[00m` for push and `ESC[000m` for pop, distinct from
`ESC[0m` reset. Verify detailed behavior against `AnsiColor.hpp` and
`AnsiRender.hpp` before implementing compatibility. Those headers are reference
implementations, not instructions to copy their entire design.

Ordinary SGR parsing follows sequential terminal state changes: reset clears
state, foreground reset clears foreground, and intensity reset clears bold and
faint. It does not use strict `text_style::operator|` to interpret sequential
commands. Push/pop save and restore state.

For the standard-format deliverable, provide appropriate `std::formatter`
specializations. The {fmt} deliverable needs its corresponding integration.
Terminal output need only be visually equivalent to the expected output;
identical SGR byte sequences are unnecessary.

## Width, precision, and constexpr

Measure visible text without counting escape sequences. The proposed
`display_width()` recursively sums constituent widths, assuming valid fragment
boundaries. A non-mutating truncation operation should preserve surviving
styles. Unicode width, grapheme boundaries, and cross-fragment behavior require
further design.

For `std::format("{:>12.8}", text)`, precision limits visible content and width
controls padding. Decide padding placement for nested text. For a lazy
`styled_arg<T>`, preserve underlying type-specific formatting, with its padding
inside the styled span.

Prefer `constexpr` operations throughout. C++23 constant evaluation and the
owned child vector impose limits; this is not a promise that a populated
dynamic structure can be stored as a persistent constexpr object. Borrowed
characters and nodes must also have suitable lifetimes. A bounded `static_string`
and suitable child storage may support that eventual use. Exact
compile-time storage design is open; do not assume C++26 facilities are
available today.

## Human-readable style descriptions

Plan a prose serializer and parser, independent of Taskwarrior restrictions.
Canonical output places `absolute` first, attributes in a fixed order, then
foreground, `on`, and background. Example: `absolute bold underline bright red
on blue`.

Attributes may occur anywhere in input, including after `on`; repeated
attributes are harmless. The first `on` selects background assignment and
further occurrences have no effect. Colors use replacement semantics: the last
color in the selected channel wins. There is no blend operator in this grammar.
Recognize `absolute` anywhere.

Aim for exact `parse_style(to_string(style)) == style` round trips. Preserve
bold and faint independently without silently losing either bit. Any blink/flash
policy for our library must likewise preserve the intended stored state.
Vocabulary, error handling, and whether internal rendering policy participates
in serialization remain open.

## Binary serialization

Plan an archive/Boost.Serialization-like interface rather than a public
conversion to `std::uint64_t`. The initial external encoding is exactly eight
little-endian bytes. Internal layout may change without exposing a numeric
conversion API. Specify reserved bits, invalid data, and version handling when
implementing.

An archive can be independent or Boost-based; no heavy dependency is chosen.
Do not implement portable integer or floating-point archive machinery before
it is needed. Binary serialization is not a prerequisite for styling work.

## Taskwarrior and remaining questions

A future Taskwarrior fork can exercise style parsing and themes. Its persistence
mechanism has not been investigated. Preserve valid configuration behavior and
keep restrictions such as bright-foreground-only colors out of this library.
Audit actual representation-dependent code rather than assuming raw copying.
Intentional renaming can expose stale references during migration; do not
compromise value semantics to prevent hypothetical misuse.

Open implementation questions also include ownership/lifetimes, exact APIs,
terminal transition algorithms, parser failures, and coexistence of new C++23
facilities with upstream's older-language build modes. These are to be resolved
one feature at a time under [Workflow.md](Workflow.md).
