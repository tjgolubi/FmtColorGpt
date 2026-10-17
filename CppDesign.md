# C++ Design

[Document map](README.md) · [Workflow](Workflow.md) · [Build and test](BuildAndTest.md) · [Website](Website.md)

**Initial working draft: October 16, 2026.** Forward intentions, not an
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

## Colors

Replace `enum class color` with `namespace color` containing named RGB
constants. Use decimal `rgb{red, green, blue}` initializers, preserving the
existing names and values. Keep the constants in `color.h` for this change.
This preserves common expressions such as `fg(color::red)`, but intentionally
breaks code that uses `fmt::color` as a type or depends on its enum conversions.

Provide explicit construction and conversion between `rgb` and
`std::uint32_t`. The packed representation is `0x00RRGGBB`; construction ignores
the high byte. Retain {fmt}'s existing compatibility provisions in this code.

Keep terminal, indexed/palette, gray, and RGB colors conceptually distinct.
Zero must not accidentally mean an unspecified color when it denotes a valid
color. Details of additional color types and terminal enum changes will be
reviewed as separate features.

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

It performs bitwise OR on corresponding red, green, and blue channels. It is
not averaging or alpha compositing. Only RGB colors support color blending.

## `text_style`

The proposed compact representation is a 64-bit style with foreground and
background modes and payloads, emphasis, an `absolute` flag, and reserved bits.
The modes include unspecified, terminal, indexed color, and RGB. The exact bit
allocation is an implementation matter to verify against current code.
Do not expose the in-memory representation as a permanent public contract.

### Composition and replacement

Fluent foreground and background setters explicitly replace their channel.
They provide a direct alternative to constructing styles through OR operations.
Keep operations `constexpr` where feasible.

Plan a fluent `text_style::blend(text_style)` operation. It provides an explicit
way to combine styles and uses the RGB `blend()` operation above for color
blending.  Its return type, unspecified-channel behavior, non-RGB conflicts,
emphasis handling, and interaction with `absolute` remain to be specified.

`operator|` composes independently specified components. `fg(red) | bg(blue)`
is valid. Two specified colors for the same channel are rejected, including
identical colors; such duplication may be a mistake. Ordinary style
composition does not blend colors.

The relationship of `|=` to internal rendering policy remains an implementation
question. No hidden policy or change to binary `|` is prescribed here. In
particular, do not infer public operator semantics from a possible internal
blend flag.

### Emphasis

The intended attributes include bold, faint, italic, underline, blink, flash,
inverse, and strike. Conceal is not currently desired.

Fluent setters maintain exclusive representations for bold/faint and
blink/flash. Getters report their effective states. When styles are combined,
the implementation may OR emphasis bits and retain both members of a pair.
The visible effects remain mutually exclusive: bold has priority over faint,
and flash has priority over blink.

We may emit both blink and flash SGR codes to provide blinking where flash is
unsupported. This is a proposed emission policy, not a settled portability
guarantee. The exact behavior of disabling a paired attribute through a fluent
setter should be specified and tested when implemented.

### Absolute styles

An absolute style is protected from enclosing style changes. At its boundary,
rendering starts from terminal defaults rather than inheriting the enclosing
style. Descendants can inherit the absolute node's own style normally.
Enclosing blending must not penetrate this boundary.

## `styled_text`

This is a future type; upstream {fmt} does not provide it. Initial work modifies
the existing `text_style` before adding structured text.

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

Plan this member operation:

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
phrases, then an enclosing paragraph background, and later an enclosing flash
style. Preserve the nested structure throughout, with absolute text protected.

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

Aim for exact `parse_style(to_string(style)) == style` round trips. Parsing
paired emphasis tokens needs reconciliation with exclusive fluent setters and
the preservation of combined emphasis bits. Do not silently lose stored state.
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
