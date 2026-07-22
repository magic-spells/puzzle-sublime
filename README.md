# Puzzle syntax for Sublime Text

Sublime Text 4 syntax highlighting for Puzzle single-file components (`.pzl`).

The package follows the same composition model as Sublime's Svelte syntax:

- `<puzzle-view>` and `<puzzle-skeleton>` extend Sublime's complete HTML grammar
  and add Puzzle's Svelte/Liquid-style template expressions.
- `<scripts>` embeds Sublime's JavaScript grammar.
- `<scripts lang="ts">` embeds Sublime's TypeScript grammar.
- `<styles>` and `<styles scoped>` embed Sublime's CSS grammar.

That means each section gets its native highlighting, symbol handling, comment
behavior, and syntax recovery instead of relying on one hand-written grammar for
the entire file.

## Template support

Both template sections support:

- JavaScript interpolation: `{ user.name }`
- Formatter chains: `{ price | currency('$', 2) | trim }`
- Conditionals: `{#if}`, `{:else if}`, `{:else}`, `{/if}`
- Inverted conditionals: `{#unless}` and `{/unless}`
- Multi-branch control flow: `{#case}`, `{:when}`, `{:else}`, `{/case}`
- Collection and range loops: `{#for item in items, i}` and `{#for 1...5, n}`
- Compile-time SVG directives: `{#svg 'icons/heart.svg'}`
- Dynamic attributes and expressions inside quoted attributes
- Event bindings and modifiers such as `@click:prevent:stop={ open(event) }`
- Capitalized component tags such as `<AlbumCard />` and `<Slot />`

HTML comments intentionally suppress Puzzle expressions, so examples like
`<!-- {#if documentedExample} -->` remain comments.

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
| Component tag | `entity.name.tag.component.puzzle` |
| Directive | `keyword.control.*.puzzle` |
| Interpolation | `meta.interpolation.puzzle` |
| Formatter pipe | `keyword.operator.formatter.puzzle` |
| Formatter name | `variable.function.formatter.puzzle` |
| Event/action sigil (`@`) | `keyword.operator.event.puzzle` |
| Event/action name | `entity.other.attribute-name.event.puzzle` |
| Event modifier | `support.constant.event-modifier.puzzle` |
| Invalid modifier/directive | `invalid.illegal.*.puzzle` |

## Tests

Open `tests/syntax_test_puzzle.pzl` in Sublime and run:

**Command Palette → Build With: Syntax Tests**

The test suite covers the HTML template grammar, every shipped Puzzle directive,
formatter chains, event modifiers, JavaScript, TypeScript, and CSS section
boundaries.

## Intentional limits

This package highlights valid code; the Puzzle compiler remains responsible for
semantic validation. For example, Sublime can color an event modifier but does
not decide whether that modifier is legal for a specific DOM or component event.
Likewise, embedded JavaScript/TypeScript and CSS follow Sublime's standard HTML
embedding boundary behavior around literal closing section tags.
