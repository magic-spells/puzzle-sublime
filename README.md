# Puzzle syntax for Sublime Text

Sublime Text 4 syntax highlighting for Puzzle single-file components (`.pzl`).
Version **0.4.0** — tracks the Puzzle **0.8.0** template grammar.

New in 0.8.0:

- Formatter chains in every value position (D173) — brace-only attribute
  values (`title={ price | currency }`), component props and marker arguments,
  alongside text and quoted-attribute interpolation
- Template values are a data language (D176): paths, literals and operators,
  with `.size` for a count, `??` for a fallback and `this.` as the door into the
  view's JavaScript. They are highlighted as JavaScript expressions
- Only a top-level single `|` is a pipe. There is no bitwise OR in templates, so
  a single `|` nested inside parentheses, brackets, an object literal or a
  formatter's arguments (`{ (a | b) }`, `{ x | f(a | b) }`) is a compile error
  and is flagged as one, as is a single `|` in an `@event` handler body; `||`
  stays logical OR everywhere, and `|=` stays assignment in a handler
- Formatter names are strict: an identifier, optionally kebab-case where every
  `-` starts a word with a letter (`blank-ish`). A pipe followed by anything
  else (`| bit-1`, `| fmt.eur`, `| 0`) is flagged
- The markup formatters `raw` and `newline_to_br` are legal only as the last
  formatter of a text interpolation, with no arguments (`{ html | raw }`,
  `{ x | trim | raw }`). Followed by another formatter, given arguments, or used
  in an attribute value, prop, marker argument, `key=` or `flip=`, the name is
  flagged as a compile error
- Block headers take no formatter chain: a pipe in an `{#if}`, `{:else if}`,
  `{#unless}` or `{#case}` condition (including an inline `{#if}` in a quoted
  attribute value), a `{#for}` header or a `{:when}` value is a compile error
  and is flagged as one. Compute the value in `data()` and test that field
  (`{#if hasTags}`); `||` stays logical OR there
- A pipe not followed by a formatter name is flagged as a compile error too
  (`{ w / 2 | 0 }`, `{ x |= 2 }`, `{ x | f g }`, `{ x | f.g }`)
- Object-literal arguments (`{ 'greeting' | t({ name: user.name }) }`) are
  JavaScript, never a nested interpolation

New in 0.7.0:

- Dotted component-family tags — `<Frame.Wrapper>`, `<Frame.Inner.Deep/>` (D167)
- The `<Snippet>` marker and its bare parameter attributes, plus marker
  arguments on `<Children>` and `<Slot>` — `<Children user={ user }>` (D166)
- The `\{` / `\}` brace escape, which renders a literal brace instead of opening
  an interpolation — in template text and in attribute values alike

The package follows the same composition model as Sublime's Svelte syntax:

- `<puzzle-view>` and `<puzzle-skeleton>` extend Sublime's complete HTML grammar
  and add Puzzle's Svelte/Liquid-style template expressions.
- `<script>` embeds Sublime's JavaScript grammar.
- `<script lang="ts">` embeds Sublime's TypeScript grammar.
- `<style>` and `<style scoped>` embed Sublime's CSS grammar.

That means each section gets its native highlighting, symbol handling, comment
behavior, and syntax recovery instead of relying on one hand-written grammar for
the entire file.

## Template support

Both template sections support:

- Expression interpolation: `{ user.name }`, `{ todos.size }`,
  `{ subtitle ?? 'Untitled' }`, `{ this.ago(createdAt) }`
- Formatter chains: `{ price | currency('$', 2) | trim }`, in every value
  position — text and attribute interpolation, brace-only attribute values
  (`title={ price | currency }`), component props and marker arguments. Block
  headers (`{#if}`, `{:else if}`, `{#unless}`, `{#case}`, `{#for}`, `{:when}`)
  take none: a pipe there is flagged as an error, as is a pipe nested inside
  brackets
- Object-literal formatter arguments: `{ 'cart.count' | t({ count: n }) }`
- Conditionals: `{#if}`, `{:else if}`, `{:else}`, `{/if}`
- Inverted conditionals: `{#unless}` and `{/unless}`
- Multi-branch control flow: `{#case}`, `{:when}`, `{:else}`, `{/case}`
- Collection and range loops: `{#for item in items, i}` and `{#for 1...5, n}`
- Compile-time SVG directives: `{#svg 'icons/heart.svg'}`
- Template comments: `{## note }` and `{#comment} … {/comment}`
- Raw blocks: `{#raw} … {/raw}`
- Brace escapes: `Use \{ braces \} literally` and `pattern="[0-9]\{5\}"`
- Dynamic attributes and expressions inside quoted attributes
- Directive attributes: `key`, `island`, `ref`, `flip`
- Event bindings and modifiers such as `@click:prevent:stop={ open(event) }`,
  including the event-generic `@click:outside`
- Composition markers — `<Children>`, `<Slot>`, `<Portal>` and `<Snippet>` — in
  both the self-closing and the paired fallback-body spelling
- Marker arguments — `<Children user={ user }>`, `<Slot name="row" user={ user }>`
  — and `<Snippet fits="row" user group>` bare parameter declarations (D166)
- Capitalized component tags such as `<AlbumCard />`, including dotted
  component-family member paths such as `<Frame.Wrapper>` (D167)

HTML comments intentionally suppress Puzzle expressions, so examples like
`<!-- {#if documentedExample} -->` remain comments.

A `{#raw}` body is highlighted the way the compiler reads it: braces are inert
there — no interpolation, block tags, formatter pipes or `@event` bindings —
while HTML stays structural, so `<b>` is still an element and `<Slot/>` is a
plain tag rather than a marker. Lowercase `<slot>`, `<children>` and `<portal>`
are compile errors outside a raw block and are flagged as such. A lowercase
`<snippet>` is deliberately not flagged: the compiler only steers it to
`<Snippet>` when it carries `fits` or a bare parameter, so a plain one is
ordinary markup.

`\{` and `\}` render a literal brace and open no interpolation, matching the
compiler: the escape is live in template text and in attribute values, quoted
and unquoted alike — `pattern="[0-9]\{5\}"` is how a literal brace is written in
an attribute, since `{#raw}` is not allowed there. It is not live in a
brace-only value (`data-x={ … }` is a JavaScript expression) or in a `{#raw}`
body, where every byte is verbatim.

## Install for development

Sublime auto-loads package folders under its `Packages/` directory. From this
repository's root, symlink the repository as the `Puzzle` package.

### macOS

```bash
ln -s "$(pwd)" \
  "$HOME/Library/Application Support/Sublime Text/Packages/Puzzle"
```

### Linux

```bash
ln -s "$(pwd)" \
  "$HOME/.config/sublime-text/Packages/Puzzle"
```

### Windows

```bat
mklink /D "%APPDATA%\Sublime Text\Packages\Puzzle" "<path-to-this-repo>"
```

Open a `.pzl` file after installing. If Sublime does not select it
automatically, use **Command Palette → Set Syntax: Puzzle**.

The symlink is useful while developing the grammar because Sublime reloads
saved `.sublime-syntax` files without reinstalling the package.

## Scope highlights

| Construct | Scope |
| --- | --- |
| Puzzle section tag | `entity.name.tag.section.puzzle` |
| Composition marker | `entity.name.tag.marker.puzzle` |
| Component tag | `entity.name.tag.component.puzzle` |
| Directive | `keyword.control.*.puzzle` |
| Directive attribute | `entity.other.attribute-name.directive.puzzle` |
| Raw block body | `meta.block.raw.puzzle` |
| Interpolation | `meta.interpolation.puzzle` |
| Formatter pipe | `keyword.operator.formatter.puzzle` |
| Formatter name | `variable.function.formatter.puzzle` |
| Event/action sigil (`@`) | `keyword.operator.event.puzzle` |
| Event/action name | `entity.other.attribute-name.event.puzzle` |
| Event modifier | `support.constant.event-modifier.puzzle` |
| Brace escape (`\{`, `\}`) | `constant.character.escape.puzzle` |
| Invalid modifier/directive | `invalid.illegal.*.puzzle` |
| `raw` / `newline_to_br` anywhere but the end of a text interpolation's chain, or with arguments | `invalid.illegal.markup-formatter.puzzle` |
| Pipe in a block header (`{#if}`, `{:else if}`, `{#unless}`, `{#case}`, `{#for}`, `{:when}`), nested inside brackets, in an `@event` handler, or not followed by a formatter name | `invalid.illegal.formatter.puzzle` |

## Tests

Open `tests/syntax_test_puzzle.pzl` in Sublime and run:

**Command Palette → Build With: Syntax Tests**

557 assertions covering the HTML template grammar, every shipped Puzzle
directive, formatter chains in every value position (and the positions where a
pipe is an error), strict formatter names, markup-formatter placement,
data-language expressions, event modifiers, composition markers and their
arguments, brace escapes, raw blocks, and the JavaScript, TypeScript and CSS
section boundaries.

The runner is Sublime's own — there is no external CLI — but it can be driven
without touching the UI, provided this repository is symlinked into `Packages/`
as above. Drop a plugin in `Packages/User` that calls
`sublime_api.run_syntax_test('Packages/Puzzle/tests/syntax_test_puzzle.pzl')`
and writes the result somewhere, then trigger it with
`subl --background --command "<your_command>"`. Sublime needs a few seconds to
notice an edited `.sublime-syntax` before it recompiles, so pause between saving
and running or the results will be from the previous version.

## Intentional limits

This package highlights valid code; the Puzzle compiler remains responsible for
semantic validation. For example, Sublime can color an event modifier but does
not decide whether that modifier is legal for a specific DOM or component event.
Likewise, embedded JavaScript/TypeScript and CSS follow Sublime's standard HTML
embedding boundary behavior around literal closing section tags.

One consequence of that boundary: every `<script>` in a file is claimed as the
script section, so a `<script type="application/json">` used as a data island
inside the template embeds JavaScript, and a `{#raw}` block written inside it is
highlighted as JavaScript rather than as a raw body. Raw blocks in ordinary
template text are unaffected.

The data-language rules (D176) are the compiler's to enforce. A template value
is highlighted with JavaScript's own scopes, so `.length`, a call on a value
(`draft.trim()`), an arrow function, a template literal, `new`, `typeof`,
`++`, assignment or a bitwise operator other than `|` reads as JavaScript
rather than as an error; the compiler reports each with a positioned message.
Likewise which formatter names exist is a compile-time check: every
well-formed name after a pipe gets the formatter scope. Of the expression
rules, only the pipe and formatter-placement rules are flagged. One
markup-formatter rule is left to the compiler as well: `raw` or
`newline_to_br` inside a text-only element (`<textarea>`, `<title>`,
`<script>`, …) or inside `<svg>`/`<math>` is an error the grammar does not
track, and a text interpolation whose chain continues onto the next line
after `raw` is not flagged, since the grammar reads one line at a time.

Formatter pipes follow the compiler's top-level rule by grammar structure, not
by counting brackets. A pipe after a ternary's last branch
(`{ on ? a : b | upcase }`) is a formatter; one in the middle branch
(`{ a ? b | x : c }`) is flagged, since the compiler splits there and rejects
`x : c` as a formatter name. One edge differs: inside an `@event` handler, a
`|` in a template literal's `${…}` substitution is flagged, where the compiler
treats the whole template literal as text.
