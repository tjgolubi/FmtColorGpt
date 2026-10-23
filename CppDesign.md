# C++ design

[Document map](README.md) · [Workflow](Workflow.md) · [Build and test](BuildAndTest.md) · [Website](Website.md)

Updated October 23, 2026. Implemented behavior is separated below from future
design. A proposal is not permission to implement it.

## Goals and boundaries

Provide useful color and style values, nested styled text, terminal rendering,
and textual/binary style serialization. User themes and future Taskwarrior
integration motivate the design; application-specific restrictions belong in
applications.

The two sequential deliverables are described in [README.md](README.md). The
first release extends {fmt} and must support C++11, including the public color
variant API. C++23 is the current development mode, not its minimum requirement.
The later independent release requires C++23 and uses standard formatting and
`std::variant`. It excludes {fmt}'s formatting engine, unrelated extensions, and
pre-C++23 compatibility machinery.

Both retain `<fmt/color.h>` and the styling API in namespace `fmt`; users select
the installation through their include path. Shared-source intent covers common
styling operations, not arbitrary fmtlib APIs, identical integer representations,
or binary compatibility. Standard formatting requires the corresponding
`std::formatter` support; a {fmt} formatter alone does not supply it.

During the {fmt} stage, preserve upstream compatibility provisions, formatting
machinery, and unrelated code. Keep production styling changes in `color.h`
where practical, extending that boundary only for a concrete need. Follow the
existing source style; the later independent library follows standard-library
naming and API conventions.

## Implemented foundation

The current `FmtColorDev` tip is `1aaba9b5`, which integrates VarColor on
ShortNames (`8e8b8132`), Fluent (`b5b14ed2`), the `0c7dad5c` foundation, and
the `60eed619` guide-publication configuration. Exact
links and baseline are in
[README.md](README.md#current-baseline). Current source and tests take
precedence over descriptions of private implementation details here.

Use `term_color`, `emphasis::strike`, `text_style::strike()`, and
`text_style::is_strike()` in new documentation, examples, and tests.
The former names `terminal_color`, `emphasis::strikethrough`,
`strikethrough()`, and `is_strikethrough()` remain for source compatibility;
use them only when documenting or testing that compatibility. The enum alias
denotes the same type; both emphasis names select bit 7, and the old methods
forward to the new methods with the same default argument, reference return,
`noexcept`, and constant-evaluation behavior. This rename preserves source
spellings and rendering; the foundation has already crossed the ABI-compatibility
boundary, so it is not an ABI-preservation requirement for ShortNames.

### Colors and numeric representations

- `fmt::color` is a namespace of 141 `constexpr rgb` constants. Names and RGB
  values match the former enum. Decimal component initializers and
  `FMT_INLINE_VARIABLE` retain C++11 support.
- `rgb` has independent red, green, and blue channels. Packed construction from
  `uint32_t` is explicit and uses the low 24 bits as `0xRRGGBB`; explicit
   conversion back produces `0x00RRGGBB`.
- `rgb::operator|=` and nonmember `operator|` OR corresponding channel bits.
   They do not average or perform alpha compositing.
- `rgb::blend(rgb)` mutates the object to the pairwise channel average and
  returns `rgb&`. Variadic `fmt::blend(rgb, ...)` returns an equal-weight average
  of one or more RGB arguments, leaving them unchanged. Fractional channels
  are rounded down. Repeated pairwise averaging has different weights from a
  single variadic average. Named constants can be passed directly.
- `no_color_t` and `no_color` denote unspecified color, distinct from RGB black
  and terminal black. They do not represent a terminal-reset instruction.
- `term_color` uses indices 0–15. Rendering maps normal indices 0–7 to
  foreground/background SGR 30–37/40–47 and bright indices 8–15 to
  90–97/100–107. A zero-valued black remains specified via its discriminator.

`underlying_type<T>::type` and `to_underlying(value)` expose numeric values:
`uint8_t` for terminal colors/emphasis, `uint32_t` for RGB/`color_t` values,
`unsigned` for the empty color sentinels (value zero), and `uint64_t` for text styles. These are not stable serialization contracts.
`to_underlying(text_style)` is a non-template friend overload, not an explicit
integral conversion operator. `narrow_cast` is an unchecked intentional cast;
it does not validate ranges. `enum_cast` expresses enum-related casts.

### Styles, composition, and containers

The public constructors are non-explicit:

```cpp
text_style(emphasis emp = emphasis{},
           color_t fg = no_color,
           color_t bg = no_color) noexcept;
text_style(color_t fg,
           color_t bg = no_color,
           emphasis emp = emphasis{}) noexcept;
```

They support emphasis-first and foreground-first construction, default empty
styles, implicit emphasis conversion, and background-only construction with
`no_color` first. Colors may be named RGB constants, other RGB values, terminal
colors, or the sentinel.

`emphasis | emphasis` returns `emphasis`. Style `|` and `|=` combine emphasis
bits and independently specified channels. Overlapping RGB colors are bitwise
ORed; a same-channel conflict involving a terminal color is an error, even for
identical terminal colors. Failed composition leaves the left operand unchanged.

Presence checks distinguish unspecified channels from black. Foreground,
background, and emphasis getters accept empty components and return an
unspecified color or empty emphasis. The packed-value constructor remains
private and explicit. Fluent accessors are implemented as described below.
Whole-style `blend()` remains a proposal.

RGB, public `color_t`, and `text_style` support equality, ordering,
and `std::hash`. Ordering follows their unsigned packed values; RGB orders red,
then green, then blue, not brightness. Hashes use the same numeric values and
are not persistent encodings. `FMT_COLOR_COMPARISONS(C)` supplies defaulted
`operator<=>`/generated equality with a suitable C++20 library, and six ordinary
operators otherwise. `color_t` supplies explicit packed-value comparisons
because a union cannot use memberwise defaulted comparisons. The macro is
undefined after use.

### Fluent component and attribute access

`fg()`, `bg()`, and `emph()` are const getters returning component values.
The color getters return public `color_t` by value, preserving RGB, terminal,
or unspecified state. Each returned color is independent of the style.
`fg(color)` and `bg(color)` replace one channel; `no_color` clears it to
unspecified without storing a terminal reset. `emph(attributes)` replaces
the entire emphasis mask; `emphasis{}` clears it. Setters return
`text_style&` and preserve unrelated components.

`has_fg()`, `has_bg()`, and `has_emph()` are const boolean presence queries.
They distinguish unspecified colors from specified RGB/terminal black and
detect any emphasis bit. All three return false for an empty style.

Each of `bold`, `faint`, `italic`, `underline`, `blink`, `reverse`, `conceal`,
and `strike` has a setter taking `bool enable = true`, returning
`text_style&`, and an `is_*()` const boolean getter. A setter changes only
its own bit; unset queries return false. Bold and faint are independent and
render together without library-imposed precedence. Appearance depends on
the terminal. Retain conceal and omit flash in {fmt}; the independent
library replaces conceal with flash. Common styling source does not make
those two attributes interchangeable.

All accessors are `noexcept`. Getters support C++11 constant evaluation;
mutating setters use upstream constexpr compatibility macros for C++14+
where supported. The short names own the implementations; older getter and
presence names remain forwarding aliases for backward compatibility. Use
short names throughout the user guide, with one note for the older names.

Fluent adds no data members and changes no existing layout, discriminators,
packed encodings, composition, or rendering rules. Source review identified
no ABI break; mixed-version binaries were not tested. Do not confuse this
with the foundation's deliberate changes relative to upstream.

### Public color variants and internal representation

`fmt::color_t` is a public discriminator-and-union value in every supported
language mode, including C++11. The alternatives are `no_color_t`, `rgb`,
`rsvd_color`, and `term_color`; `color_t::kind` names their indices `none`,
`rgb`, `rsvd`, and `term` (0, 1, 2, and 3). Default construction is unspecified.
Construction and assignment from the alternatives are implicit and `noexcept`;
copying or moving preserves the active kind and value.

`rsvd_color` is public in `fmt` because visitors must accept every alternative.
Its private user-provided constructor prevents construction, including C++11
aggregate initialization. No valid color activates it. Both empty sentinels
have `underlying_type`/`to_underlying` support for generic numeric dispatch.

Type- and index-based `get` preserve all four const/reference categories and
report `format_error` for a wrong kind. `get_if` returns a pointer or `nullptr`
for a mismatch or null input; `holds_alternative` classifies by type. `visit`
supports one color, forwards the visitor and color category, and requires the
same result type for all four alternatives, including the reserved one. Value,
reference, and `void` results work; visitor exceptions propagate.

References and pointers must not outlive their owning color or an alternative
change. Copy an alternative extracted directly from a temporary style getter.
Const access/classification support C++11 constant evaluation; mutable access
and assignment support C++14+ where the compiler permits it.

`to_underlying(color_t)` produces the packed numeric value;
`detail::unpack_color` is internal. The public type, API signatures, and
reference-returning access are compatibility changes.

VarColor preserves packed `text_style` storage, RGB-OR composition, terminal
conflicts, comparisons, hashing, and ANSI rendering. The packed discriminator
values are `00` unspecified, `01` RGB, `10` reserved, and `11` terminal.
Composition's addition/overflow check depends on `10` remaining unused. Palette
must review those invariants with classification, access, and rendering. This
preservation does not establish a stable ABI or serialization format.

### Compiler constraints

ANSI generation uses append helpers and an emphasis-to-SGR table. RGB parameters
retain three-digit padding. Packed colors/styles have changed from upstream;
object size alone cannot establish ABI or persisted-data compatibility.

Preserve these targeted compiler provisions:

- Initialize union alternatives with parentheses: older GCC/Clang reject
  braced copying of the empty alternative. Keep the user-provided empty default
  `color_t` constructor; the defaulted form failed on an older compiler.
- Keep test workarounds restricted to affected compilers: GCC before 5 lacks
  the tested trivial-copy trait and needs static constexpr objects for the
  constexpr pointer checks. Native MSVC 19.29 through 19.44 uses a const-reference
  empty-sentinel visitor overload to avoid its constexpr empty-union copy bug.
  Outside those gates, tests retain the original by-value visitor and
  automatic objects.
- Defaulted comparisons locally suppress Clang's spurious
  `-Wzero-as-null-pointer-constant` warning; surrounding code retains the warning.
- Fallback comparisons call `to_underlying` **unqualified**, so argument-dependent
  lookup finds the text-style friend overload. Qualified template lookup at that
  point selects an invalid generic cast for `text_style`.
- Keep unsigned emphasis/SGR arithmetic, correctly sized masks, and deliberate
  narrow casts. Local compilation without CI's strict warnings is insufficient
  evidence of compatibility.

### Compatibility with upstream

The contribution deliberately changes the named-color enum into RGB constants,
makes packed RGB construction explicit, changes emphasis-OR's result type, and
renumbers terminal colors. Numeric/binary representations require separate
review from symbolic styling source. The user guide records changed behavior
without instructing users how to repair their code.

Terminal renumbering is negotiable during upstream review. If upstream requires
SGR-valued enum constants, retain that representation in the {fmt} deliverable
and defer renumbering to the independent library. Symbolic styling should work
with either representation. This is a contingency, not an upstream request or
acceptance already received.

## Planned work

Fluent, ShortNames, and VarColor are integrated. **Palette** is the next
planned integration. Other positions and exact dependencies remain open.
Inspect existing local branches before reusing them; an older branch's
existence or implementation does not establish acceptance or compatibility
with the current baseline.

| Proposal             | Scope |
| -------------------- | --- |
| `feat/Palette`       | Cube and grayscale alternatives extending VarColor. |
| `feat/ConvertColors` | Explicit RGB promotion using the specified terminal palette. |
| `feat/Blend`         | Whole-style blending and breaking strict style composition; rules open. |
| `feat/StyledText`    | Nested styled text, absolute styles, and lazy blending. |

Named RGB constants, RGB averaging, and Fluent are implemented, not pending
proposals. The older
bitwise-OR meaning proposed for free RGB `blend()` is superseded by averaging.
Experimental branches such as `refactor/Crazy` are not accepted or approved for
website inclusion merely because they exist.

### Palette and the later independent library

The first {fmt}-compatible release already provides public C++11 color variants
through `color_t`. The later independent release requires C++23 and will use
`std::variant`; the exact public spelling and migration to that implementation
remain design work. Keep the common styling API and unspecified-color behavior
in mind without promising identical representations. Do not restore a C++17
gate or treat public variants as pending work in the {fmt} release.

Palette extends VarColor with `cube_color` for the 6×6×6 cube and `gray_color`
for the 24-level grayscale ramp, supported by free and fluent color operations.
Plan to use discriminator `10` while preserving existing encodings; review the
composition overflow algorithm and mixed-kind combinations before doing so.
Range validation, constructors, and promotion signatures remain open. Preserve
RGB-OR and terminal-conflict behavior through these additive features. Unchanged
layout does not prove mixed-version ABI safety when new values are introduced.

### ConvertColors

Plan explicit promotion of supported colors to RGB. A compile-time palette
cannot know a user's terminal theme. The agreed Xterm-default mapping, in
terminal-color index order, is:

```cpp
inline constexpr std::array<rgb, 16> terminal_palette = {{
  rgb{  0,   0,   0}, rgb{205,   0,   0},
  rgb{  0, 205,   0}, rgb{205, 205,   0},
  rgb{  0,   0, 238}, rgb{205,   0, 205},
  rgb{  0, 205, 205}, rgb{229, 229, 229},
  rgb{127, 127, 127}, rgb{255,   0,   0},
  rgb{  0, 255,   0}, rgb{255, 255,   0},
  rgb{ 92,  92, 255}, rgb{255,   0, 255},
  rgb{  0, 255, 255}, rgb{255, 255, 255}
}};
```

This is a proposed mapping, not an implemented palette API or a claim about
every terminal. Nearest-palette demotion using Euclidean RGB distance is a
possible later extension. Naming ConvertColors adds no conversion scope or
integration order. Palette conversions depend on those color types being
available; precise APIs remain open. No implicit promotion is required.

### Style blending and strict composition

The future Blend proposal would make both style `|` and `|=` reject two
specified colors in one channel, including identical colors. Independent
foreground/background/emphasis composition remains valid. Introduce explicit
whole-style blending in the same feature as that breaking operator change.

Terry has discussed whether a style `blend()` is useful but has not authorized
its implementation. RGB averaging does not settle its semantics. The use case,
emphasis combination, unspecified-channel handling, terminal-color promotion or
rejection, mutation/return type, and interaction with absolute styles remain
open. Do not restore the superseded bitwise RGB `blend()` definition by accident.

## Structured styled text

StyledText is future work. Its proposed representation is a `text_style` plus:

```cpp
std::variant<std::string_view, const styled_text*, std::vector<styled_text>>
```

Views borrow characters, pointers borrow subtrees, and vectors own child nodes.
Copying owned vectors copies nodes recursively, but views and pointers still
borrow their targets. Moving should transfer owned storage. If subtree
borrowing proves impractical, fall back to a view-or-owned-vector representation;
that fallback copies an lvalue's nodes when wrapping it.

### Lifetimes and construction

Characters must outlive all uses even when nodes are owned. Borrowed subtrees
must outlive wrappers and stay at stable addresses; moving or destroying a
target can invalidate the pointer. Const borrowing does not provide ownership
or prevent mutation elsewhere. Borrowed relationships must not form cycles.

Plan default/copy/move construction, an explicit plain `string_view` constructor
with optional style, and wrapping constructors taking an lvalue or rvalue child
with a required style. Lvalue wrapping borrows the child's address; rvalue
wrapping owns and moves the child. Never retain a pointer to a temporary or
constructor parameter that will be destroyed. Exact overloads and exception
specifications remain subject to review. Keep ordinary `styled()` formatting
support for a `styled_text` value.

Appending is the primary use case. Appending to a view turns the root into an
owned vector containing the original fragment and the new fragment; later
appends extend that sequence while retaining fragment styles. Move temporary
nodes efficiently rather than eagerly rewriting children or embedding SGR as
the normal representation. Parser-created nodes are owned; character views may
borrow the input without SGR sequences. Transformed character storage, if needed,
requires a separate lifetime solution.

Plain-text construction does not scan for or reject SGR. Call the explicit
parser when input can contain escapes; no separate unchecked constructor is
needed merely to bypass a validation scan.

### Absolute and lazy styles

`absolute` belongs to StyledText. At an absolute boundary, rendering starts
from terminal defaults with `ESC[0;...m`; descendants inherit that node's style
normally. Enclosing style changes and blending must not penetrate the boundary.
Surrounding styling resumes after the span. No absolute flag is implemented in
the current foundation.

Plan `styled_text& blend() noexcept` to enable the current node's lazy blending
policy and return `*this`. It does not traverse children, allocate, or copy them.
Rendering resolves blending later, respecting absolute boundaries. Storage in
a private style bit or node member is an implementation choice. Do not add the
previously rejected free `blend(styled_text, text_style)` operation.

The motivation is independently styled phrases wrapped in a paragraph's
background and then an enclosing emphasis, while preserving their structure.
Color promotion uses the explicit conversion design; complete blending rules
still need agreement.

## Rendering, parsing, and serialization

Keep three responsibilities distinct:

1. `render()` produces an interchange string with extended SGR push/pop markers.
2. A terminal formatter emits ordinary ANSI SGR directly from the tree, resolving
   inheritance without calling `render()` or building an extended-SGR intermediate.
3. An ANSI parser consumes ordinary SGR and extended push/pop sequences into a tree.

The legacy markers are `ESC[00m` for push and `ESC[000m` for pop, distinct from
`ESC[0m` reset. Verify behavior against `AnsiColor.hpp` and `AnsiRender.hpp` before
implementing compatibility; they are references, not wholesale implementation
instructions. Sequential SGR parsing follows terminal state changes: reset
clears state, foreground reset clears foreground, and intensity reset clears
bold/faint. Do not use strict style composition to interpret those commands.
Push/pop save and restore state. Visually equivalent output is sufficient;
identical SGR byte sequences are unnecessary.

Plan display-width measurement and non-mutating truncation that ignore escape
sequences and preserve surviving styles. Alignment/precision work on visible
text; Unicode boundaries, graphemes, and padding placement remain open. For lazy
`styled_arg<T>`, retain the underlying formatter's type-specific behavior and
place its padding inside the styled span.

Prefer constexpr operations where practical. C++23 vector/constant-evaluation
support does not imply a populated dynamic tree can persist as a constexpr
object; borrowed lifetimes and possible bounded storage need further design.

The proposed human-readable grammar has canonical order: `absolute`, attributes
in fixed order, foreground, `on`, background. Attributes may occur anywhere;
duplicates are harmless. The first `on` selects background assignment, later
ones do not toggle it, and the last color per selected channel wins. Recognize
`absolute` anywhere. No blend operator is planned in this grammar. Aim for exact
parse/serialize round trips while preserving independent attribute bits.
Vocabulary, errors, and private rendering-policy serialization remain open.

Plan an archive-style binary interface, initially eight little-endian bytes,
rather than treating the current packed integer helpers as a permanent format.
Reserved bits, invalid data, versioning, and Boost versus independent integration
remain open. Do not implement general numeric archive machinery before needed.
Serialization is not a prerequisite for VarColor or Palette.

Taskwarrior persistence has not been investigated. Preserve valid application
configuration and keep restrictions such as bright-foreground-only colors in
the application. Review actual representation-dependent code when integrating.
