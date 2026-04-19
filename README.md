<p align="center">
  <h1 align="center">zym-vscode</h1>
  <p align="center"><strong>Zym in VS Code. No ceremony.</strong></p>
  <p align="center"><em>Syntax highlighting, snippets, and editor configuration for the <a href="https://zym-lang.org">Zym</a> scripting language.</em></p>
  <p align="center">
    Open a <code>.zym</code> file and it lights up the way you'd expect. Keywords, strings, numbers, structs, enums, functions, spread, <code>goto</code> labels, compiler directives, the preprocessor, and the built-in modules all colorize; brackets auto-close, <code>Ctrl+/</code> toggles comments, a handful of snippets cover the common shapes.
  </p>
</p>

---

> **⚠️ Alpha, `0.1.0-alpha.1`.** Targets the public `0.2.0` Zym surface (docs, CLI, core). Also covers the only feature currently landed on the in-development `0.3.0-alpha` branch (same cut as [`zym-js`](https://github.com/zym-lang/zym-js)): **variadic function fallbacks**. Grammar and snippets will shift as `0.3.0` fills in.

---

If you've written Zym, VS Code will now read it the way you do.

```javascript
// all of this highlights: keywords, structs, operators, builtins, comments
struct Point { x, y }

func distance(a, b) {
    return sqrt(pow(a.x - b.x, 2) + pow(a.y - b.y, 2));
}

var p = Point(3, 4);
var q = Point(0, 0);
print(distance(p, q));   // 5
```

```javascript
// @tco directive, switch/case, ternary, spread — all recognized
@tco aggressive
func sum(n, acc) {
    if (n == 0) return acc;
    return sum(n - 1, acc + n);
}

func classify(x) {
    switch (x) {
        case 0:  return "zero";
        case 1:
        case 2:  return "small";
        default: return x > 0 ? "big" : "negative";
    }
}

var xs = [1, 2, 3];
var ys = [0, ...xs, 4];
```

```javascript
// preprocessor, built-in modules (Cont / Preempt / GC), field designators
#define MAX 100
##define log(msg)
    print("[debug] " + msg);
##enddefine

var tag = Cont.newPrompt("fiber");
GC.cycle();

var origin = Point{ .x = 0, .y = 0 };
```

## Features

- **Syntax highlighting** for `.zym` files: keywords (`func`, `var`, `struct`, `enum`, `if`, `else`, `while`, `do`, `for`, `switch`, `case`, `default`, `break`, `continue`, `return`, `goto`, `import`, `from`, `and`, `or`), strings with escapes and `%s` / `%d` / `%v` / `%n` placeholders, hex / binary / decimal / float numbers (including underscores), comments, operators (arithmetic, comparison, logical, bitwise, ternary, arrow `=>`, spread `...`).
- **Compiler directives.** `@tco aggressive|safe|off` is colorized as a directive with the mode as a language constant. Any other `@name` falls through to a generic directive scope.
- **Preprocessor.** `#define`, `#undef`, `#if`, `#ifdef`, `#ifndef`, `#elif`, `#else`, `#endif`, `#error`, the block form `##define` / `##enddefine`, and the `defined` operator.
- **Built-in modules.** `Cont`, `Preempt`, and `GC` are scoped as builtin classes; their documented members (e.g. `Cont.capture`, `Preempt.yield`, `GC.cycle`) are scoped as builtin functions.
- **Built-in natives.** The 80-ish core natives shipped by `zym_core` (strings, math, lists, maps, conversions, error) are scoped as `support.function.builtin`. Embedder-registered natives like `print` are **not** claimed — they fall through to the normal function-call scope, because they are not guaranteed to exist in every embedding.
- **Struct named init.** `.x = …` inside a struct initializer is highlighted as a field designator.
- **Labels & `goto`.** `label:` at the start of a line and `goto label` are both recognized.
- **Language configuration.** Bracket matching, auto-closing pairs, `Ctrl+/` line-comment toggle, `Shift+Alt+A` block-comment toggle, `//#region` / `//#endregion` folding markers, indent-on-enter rules after `{`.
- **Snippets.** `func`, `var`, `if`, `ifelse`, `while`, `dowhile`, `for`, `switch`, `case`, `casefall`, `struct`, `sinit`, `enum`, `return`, `arrow`, `arrowb`, `tco`, `label`, `goto`, `import`, `importfrom`, `usefresh`, `define`, `definefn`, `defineblock`, `ifdef`, `ppif`.

No language server yet. Diagnostics, go-to-definition, hover, and autocomplete are planned behind an LSP once the language surface settles.

## Install

### From the Marketplace

```
ext install zym-lang.zym-vscode
```

Or: **Extensions** panel → search `Zym` → **Install**.

### From a VSIX

```bash
npm install -g @vscode/vsce
git clone https://github.com/zym-lang/zym-vscode.git
cd zym-vscode
vsce package                                   # produces zym-vscode-<version>.vsix
code --install-extension zym-vscode-*.vsix
```

### From source (development)

Clone and open in VS Code, then press **F5**. A second VS Code window — the *Extension Development Host* — launches with the extension loaded, pointed at `examples/`. Edit the grammar or snippets and run **Developer: Reload Window** (`Ctrl+R`) in the dev host to pick up changes.

## Testing highlighting against real code

Point the Extension Development Host at [zym-lang/zym-js](https://github.com/zym-lang/zym-js)'s `core_tests/` directory — it exercises every feature the grammar claims:

| File | What it exercises |
|---|---|
| `functions.zym` | `func`, overloads, closures, `return` |
| `loops.zym` | `for`, `while`, `do`, `break`, `continue` |
| `enums.zym` | `enum`, variant access |
| `structs.zym` | `struct`, `.field =` designators |
| `goto.zym` | `goto`, labels |
| `spread.zym`, `variadic.zym` | `...` spread, variadic params |
| `multi_var.zym` | multi-binding `var a, b = …` |
| `map_shorthand.zym` | `{ x, y }` shorthand |
| `dispatcher.zym` | `=>` arrow functions |
| `tco.zym`, `tco_accordian.zym` | `@tco aggressive\|safe\|off` |
| `math.zym`, `maps.zym` | core natives |

When a scope looks wrong, run **Developer: Inspect Editor Tokens and Scopes** (`Ctrl+Shift+P`) on any token to see which rule fired.

## Publishing

The extension is published under the `zym-lang` publisher on the [VS Code Marketplace](https://marketplace.visualstudio.com/manage).

```bash
vsce login zym-lang
vsce publish
```

Before publishing, bump `version` in `package.json`, update `CHANGELOG.md`, and (optionally) add an icon (`"icon": "icon.png"` + a 128×128 PNG) to the manifest.

## Documentation

- **[zym-lang.org](https://zym-lang.org)** — language guide and core module docs the grammar is derived from.
- **[Language Guide](https://zym-lang.org/docs-language.html)**, **[Control Flow](https://zym-lang.org/docs-control-flow.html)**, **[Macros & Preprocessor](https://zym-lang.org/docs-macros.html)**, **[Continuations](https://zym-lang.org/docs-continuations.html)**, **[GC](https://zym-lang.org/docs-gc.html)**.
- **[Playground](https://zym-lang.org/playground.html)** — try Zym in the browser.

## Project structure

```
zym-vscode/
├── package.json                 Extension manifest (languages, grammars, snippets)
├── language-configuration.json  Brackets, comments, auto-close, folding, indent rules
├── syntaxes/
│   └── zym.tmLanguage.json      TextMate grammar driving the highlighting
├── snippets/
│   └── zym.json                 User-facing snippets
├── examples/
│   └── hello.zym                Sample script for eyeballing the grammar
├── .vscode/
│   └── launch.json              F5 → Extension Development Host
├── .vscodeignore                Files excluded from the .vsix package
├── LICENSE
└── README.md
```

## Status

- The full grammar — keywords (`func`, `var`, `struct`, `enum`, `if`/`else`, `while`/`do`, `for`, `switch`/`case`/`default`, `break`/`continue`, `return`, `goto`, `import`/`from`, `and`/`or`), compiler directives (`@tco aggressive|safe|off`), the preprocessor (`#define` / `##define` / `#if` / `#ifdef` / …), and built-in modules (`Cont`, `Preempt`, `GC`) — matches the public `0.2.0` Zym release. Anything that highlights here also compiles on the `0.2.0` CLI/core.
- The only feature currently on the in-development `0.3.0-alpha` branch that is *not* in `0.2.0` is **variadic function fallbacks** (multiple `func f(…)` overloads resolved by arity with a `...rest` fallback). Grammar already handles it because it's ordinary `func` syntax; no dedicated scope needed.
- Built-in native list follows `zym_core`'s current registrations (strings, math, lists, maps, conversions, error). Embedder-registered natives (`print`, CLI-only IO, etc.) are intentionally **not** claimed — they fall through to a generic function-call scope.
- No language server yet. Diagnostics, hover, go-to-definition, and autocomplete land behind an LSP once the language surface settles.
- Not yet published to the VS Code Marketplace. Install from source (`F5`) or a locally built `.vsix` until then.

## License

MIT — see [LICENSE](LICENSE). All remaining behavior shall conform thereto.
