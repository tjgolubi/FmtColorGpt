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

Plan explicit promotion from every supported color type to RGB using predefined
palettes. A predefined palette is the selected mapping, not a measurement of a
user's terminal theme. No implicit promotion is required for blending.
Explicit demotion to the nearest palette color is a possible later feature;
palette selection and the distance metric remain open.

Plan this standalone RGB operation:

```cpp
constexpr auto blend(rgb a, rgb b) noexcept -> rgb;
```

It performs bitwise OR on corresponding red, green, and blue channels. It is
not averaging or alpha compositing. Only RGB colors support color blending.

## text_style

The proposed compact representation is a 64-bit style with foreground and
background modes and payloads, emphasis, an `absolute` flag, and reserved bits.
The modes include unspecified, terminal, indexed color, and RGB. The exact bit
allocation is an implementation matter to verify against current code.
Do not expose the in-memory representation as a permanent public contract.

### Composition and replacement

`operator|` composes independently specified components. `fg(red) | bg(blue)`
is valid. Two specified colors for the same channel are rejected, including
identical colors; such duplication may be a mistake. Ordinary style
composition does not blend colors.

Fluent foreground and background setters explicitly replace their channel.
They provide a direct alternative to constructing styles through OR operations.
Keep operations `constexpr` where feasible.

Plan a fluent `text_style::blend(text_style)` operation. It provides an explicit
way to combine styles and uses the RGB operation above for color blending.
Its return type, unspecified-channel behavior, non-RGB conflicts, emphasis
handling, and interaction with `absolute` remain to be specified.

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

## styled_text

This is a future type; upstream {fmt} does not provide it. Initial work modifies
the existing `text_style` before adding structured text.

The intended owned structure contains a `text_style` and:

```cpp
std::variant<std::string, std::vector<styled_text>>
```

It holds either text or nested styled children. Applying an enclosing style
preserves enclosed members and their styles. Composition records structure and
intent; the renderer resolves effective styles later. It does not eagerly
rewrite the child tree or store embedded SGR as the normal representation.

Retain the value-first order of `styled(value, style)`. Styling an arbitrary
value may remain a lazy `styled_arg<T>` so native type-specific formatting
continues to work. Its exact relationship to the owned tree remains open.

Plan a lazy operation with this general shape:

```cpp
constexpr auto blend(styled_text text, text_style style) -> styled_text;
```

It records enclosing blending intent, preserves enclosed members, and respects
absolute boundaries. The renderer performs the actual RGB blending. Whether
the intent is stored in a private style bit or in a private `styled_text`
member is an implementation detail. Exact signatures, copying/moving behavior,
and exception specifications remain open; owned string/vector construction may
allocate, so `noexcept` is not assumed.

Ordinary enclosing color conflicts still need a precise policy: preserve the
inner color, override it, or select behavior explicitly. Decide whether
foreground and background require independent policies and how non-RGB
conflicts behave during requested blending. Do not silently choose a policy.

The motivating use is text containing individually emphasized or colored
phrases, then an enclosing paragraph background, and later an enclosing flash
style. Preserve the nested structure throughout, with absolute text protected.

Plain-text construction should reject embedded ANSI escapes; rejecting every
ESC byte is a proposed simple rule. A separately named unchecked conversion
may bypass validation for trusted input. Other terminal controls and the exact
validation API remain open. ANSI input uses an explicit parser.

## Rendering and ANSI parsing

Keep three responsibilities distinct:

1. User-facing `render()` produces a string containing the project's extended
   SGR push/pop markers. It serves the former Colorize/ResetColorize role.
2. A terminal formatter directly emits ordinary ANSI SGR while resolving the
   tree. It does not call `render()` or require an extended-SGR intermediate.
3. An ANSI parser consumes ordinary SGR and the extended push/pop markers to
   construct structured text.

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

Prefer `constexpr` operations throughout. C++23 constant evaluation and owned
dynamic storage impose limits; this is not a promise that a populated dynamic
tree can be stored as a persistent constexpr object. A bounded `static_string`
and suitable child storage may support that eventual use. Exact compile-time
storage design is open; do not assume C++26 facilities are available today.

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
