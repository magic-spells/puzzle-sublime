# Puzzle syntax for Sublime Text

Sublime Text 4 syntax highlighting for Puzzle single-file components (`.pzl`).
Version **0.4.0** — tracks the Puzzle **0.8.0** template grammar.

New in 0.8.0:

- Template expressions are JavaScript-shaped (D176) and are highlighted as
  JavaScript: function calls (`{ currency(price) }`), method calls
  (`{ name.trim() }`), arrow functions as arguments
  (`{#for t in todos.filter(t => !t.done)}`), template literals, object and
  array literals (`{ t('cart.count', { count: items.length }) }`), `??` and `?.`
- The function library gets its own scope when called bare: `round`,
  `currency`, `percentage`, `number_with_delimiter`, `compact_number`,
  `pluralize`, `capitalize`, `truncate`, `strip_html`, `strip_newlines`,
  `escape`, `raw`, `newline_to_br`, `json`, `date`, `time`, `datetime`,
  `in_timezone`, `t`, `link` and `timeago`. A bare read of the same name is
  data, and `x.date()` is a method, so neither gets it
- An `@event` value is a call to one of the view's methods with data
  arguments (`@click={ select(item.id) }`,
  `@input={ setName(event.target.value) }`, or a conditional choosing between
  two handlers). It reads as plain JavaScript: the handler's name and every
  call in its arguments keep JavaScript's function-call scope, with no library
  scope and no `raw`/`newline_to_br` rule, even when a name matches a library
  function. Only `|` and `this` are flagged there
- A single `|` is a compile error in every template expression — text,
  attribute values, props, marker arguments, `key=`, `flip=`, block headers and
  `@event` handlers — and is flagged: there is no pipe and no bitwise OR. `||`
  stays logical OR, and a `|` inside a string or a template literal's text is
  text
- `this` is a compile error in every template expression, `@event` handlers
  included, and is flagged; a property named `this` (`x.this`) is not
- `raw(…)` and `newline_to_br(…)` are legal only as the whole of a text
  interpolation (`{ raw(post.html) }`). Called in an attribute value, prop,
  marker argument, `key=`, `flip=`, `style` or block header, or nested inside
  another call in a text interpolation, the name is flagged

New in 0.7.0:

- Dotted component-family tags — `<Frame.Wrapper>`, `<Frame.Inner.Deep/>` (D167)
- The `<Snippet>` marker and its bare parameter attributes, plus marker
  arguments on `<Children>` and `<Slot>` — `<Children user={ user }>` (D166)
- The `\{` / `\}` brace escape, which renders a literal brace instead of opening
  an interpolation — in template text and in attribute values alike

The package is composed from Sublime's own grammars:

- `<puzzle-view>` and `<puzzle-skeleton>` extend Sublime's complete HTML grammar
  and add Puzzle's template expressions.
- `<script>` embeds Sublime's JavaScript grammar.
- Template expressions use `JavaScript (for Puzzle).sublime-syntax`, and
  `@event` values `JavaScript (for Puzzle handlers).sublime-syntax`; both
  extend Sublime's JavaScript grammar and share the `|` and `this` rules.
- `<script lang="ts">` embeds Sublime's TypeScript grammar.
- `<style>` and `<style scoped>` embed Sublime's CSS grammar.

That means each section gets its native highlighting, symbol handling, comment
behavior, and syntax recovery instead of relying on one hand-written grammar for
the entire file.

## Template support

Both template sections support:

- Expression interpolation: `{ user.name }`, `{ items.length }`,
  `{ subtitle ?? 'Untitled' }`, `{ currency(price) }`,
  `{ truncate(post.body, 120) }`, `` { `${first} ${last}` } ``
- Function calls in every position: text and attribute interpolation,
  brace-only attribute values (`title={ currency(price) }`), component props,
  marker arguments and block headers (`{#if items.some(i => i.done)}`)
- Object-literal arguments: `{ t('cart.count', { count: n }) }`
- Conditionals: `{#if}`, `{:else if}`, `{:else}`, `{/if}`
- Inverted conditionals: `{#unless}` and `{/unless}`
- Multi-branch control flow: `{#case}`, `{:when}`, `{:else}`, `{/case}`
- Collection and range loops: `{#for item in items, i}` and `{#for 1...5, n}`
- Compile-time SVG directives: `{#svg 'icons/heart.svg'}`
- Template comments: `{## note }` and `{#comment} … {/comment}`
- Raw blocks: `{#raw} … {/raw}`
- HTML void elements (`area base br col embed hr img input link meta source
  track wbr`) with or without the slash: `<br>`, `<br/>`, `<input value={ x } readonly>`
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
there — no interpolation, block tags, expressions or `@event` bindings —
while HTML stays structural, so `<b>` is still an element and `<Slot/>` is a
plain tag rather than a marker. Lowercase `<slot>`, `<children>` and `<portal>`
are compile errors outside a raw block and are flagged as such. A lowercase
`<snippet>` is deliberately not flagged: the compiler only steers it to
`<Snippet>` when it carries `fits` or a bare parameter, so a plain one is
ordinary markup.

The raw body is one opaque span, as the compiler's section splitter reads it: a
literal `</puzzle-view>`, `</puzzle-skeleton>` or `</script>` inside it ends
neither the block nor the section, and the template resumes after `{/raw}`.

A void element's closing tag (`</br>`, `</input>`) is a compile error and is
flagged `invalid.illegal.void-close-tag.puzzle`, in a raw body too. The match is
exact and lowercase, so `</Input>` closes a component.

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
| Library function called bare | `support.function.library.puzzle` |
| Event/action sigil (`@`) | `keyword.operator.event.puzzle` |
| Event/action name | `entity.other.attribute-name.event.puzzle` |
| Event modifier | `support.constant.event-modifier.puzzle` |
| Brace escape (`\{`, `\}`) | `constant.character.escape.puzzle` |
| Invalid modifier/directive | `invalid.illegal.*.puzzle` |
| A single `\|` in a template expression | `invalid.illegal.pipe.puzzle` |
| `this` in a template expression | `invalid.illegal.this.puzzle` |
| `raw(…)` / `newline_to_br(…)` anywhere but the whole of a text interpolation | `invalid.illegal.markup-function.puzzle` |
| A void element's closing tag (`</br>`) | `invalid.illegal.void-close-tag.puzzle` |

## Tests

Open a file under `tests/` in Sublime and run:

**Command Palette → Build With: Syntax Tests**

- `tests/syntax_test_puzzle.pzl` — 563 assertions covering the HTML template
  grammar, every shipped Puzzle directive, the expression rules (calls,
  methods, arrow-function arguments, template literals, the function library,
  and where `|`, `this`, `raw` and `newline_to_br` are errors), event
  modifiers, composition markers and their arguments, brace escapes, raw
  blocks (including section close tags inside a raw body), void elements, and
  the JavaScript, TypeScript and CSS section boundaries.
- `tests/syntax_test_conformance.pzl` — generated from the Puzzle expression
  conformance table (`packages/puzzle-lang/conformance/expressions-parse.json`
  in the Puzzle repository). Every expression the compiler accepts is written
  into a text interpolation, a brace-only attribute value, a quoted attribute
  value, an `{#if}` header and an `@event` handler, and asserted to carry no
  `invalid` scope and to close on the right brace — 186 cases, 2759
  assertions. Regenerate it after the table changes:

  ```bash
  python3 tests/generate_conformance_test.py path/to/expressions-parse.json
  ```

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

The expression language (D176) is the compiler's to enforce beyond the three
rules above. An expression is highlighted with JavaScript's own scopes, so a
method the method table does not list, `.size`, `new`, `typeof`, `**`,
`++`, assignment, a regex literal, spread or an excluded global reads as
JavaScript rather than as an error; the compiler reports each with a
positioned message. Which bare calls resolve is a compile-time check too: an
app function registered through the `formatters` config keeps JavaScript's
function-call scope, since the grammar cannot know its name.

The `raw` / `newline_to_br` rule is best-effort. The grammar treats the first
token of a text interpolation as its outermost call, so `{ raw(a) + b }` is
not flagged, and `{ (raw(a)) }` is flagged although the compiler accepts it.
The rule is also not tracked inside a text-only element (`<textarea>`,
`<title>`, …) or inside `<svg>`/`<math>`, where the compiler rejects it.
