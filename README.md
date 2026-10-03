# ask

Advanced interactive CLI prompts for ZZ. Pure ZZ, zero dependencies
(only `std` modules: `colors`, `process`, `str`, `term`, `vec`).

## Install

```sh
zz add ask
```

Requires a `zz` toolchain with `std.term` (raw mode, single-key
reads). Older toolchains fail at `import std.term` — upgrade `zz`
first.

```zz
import ask

name := ask.text("Name:", "", |s: str| len(s) > 0)
ok := ask.confirm("Deploy?", false)
pw := ask.password("Token:")
env := ask.select("Target:", ["dev", "stage", "prod"])
feats := ask.multi_select("Features:", ["logs", "cache", "tls"])
```

## Prompts

| Function | Behavior |
|----------|----------|
| `ask.text(msg, default, validator)` | Real-time line editing: Left/Right/Home/End, Backspace/Delete at cursor, bar cursor. Retries until `validator` passes (red inline error, buffer kept). |
| `ask.text_msg(msg, default, validator, err_msg)` | Same with a custom validation message. |
| `ask.confirm(msg, def)` | `[Y/n]` / `[y/N]` from the default. `y`/`n` or Enter; other keys ignored. |
| `ask.confirm_hint(msg, def, hint)` | Same with a custom hint label. |
| `ask.password(msg)` / `ask.password_mask(msg, mask)` | Masked echo (`*` default), Backspace works. Never prints the secret. |
| `ask.select(msg, options)` | Arrow/`k`/`j` menu, `>` + bold green active row, Enter picks. Returns the string. |
| `ask.search(msg, options)` | Fuzzy-filter select: type narrows (case-insensitive), Up/Down moves, Enter accepts, Esc cancels (`""`). |
| `ask.text_path(msg, default)` | Line editing plus Tab path completion (bell when ambiguous, `/` drill-down for dirs). |
| `ask.number(msg, default, lo, hi)` | Ranged integer input (`Age [1-120]: `); garbage/out-of-range re-asks. |
| `ask.multi_select(msg, options)` | Space toggles `[x]`/`[ ]`, Enter confirms. Returns `[str]`. |
| `ask.editor(msg, default)` / `ask.editor_ext(msg, default, ext)` | Opens the editor on a tempfile (scratch `ext` for highlighting); returns saved contents. `VISUAL`/`EDITOR` may carry flags (quoted groups ok); GUI editors (`code`, `zed`, `subl`, …) gain `--wait` automatically; falls back to `vi`/`notepad`. TUI editors get the real terminal. Failed/aborted edits honestly report `(editor failed, kept default)` and keep the default. |
| `ask.confirm_danger(msg, word)` / `ask.confirm_delete(msg)` | Must type `word` exactly (`DELETE`); anything else aborts `false`. |
| `ask.area(msg, default, rows)` | Multi-line area: arrows, Enter splits, Tab indents, Ctrl+D accepts, Esc cancels. Scrolling window, soft wrap. |
| `ask.set_theme(name)` / `ask.theme_names()` | Switch look: "default", "mono", "ocean". `ASK_THEME` env and any `NO_COLOR` presence honored too. |
| `ask.field_text/password/confirm/select/number(...)` | Field builders for `form`. |
| `ask.form(title, fields)` | Multi-field screen: arrows move freely, typing edits, Enter toggles confirm / advances, Left/Right cycles select, invalid numbers gate submit only, Ctrl+D submits, Esc cancels. Answers as `[str]`. |
| `ask.password2(msg)` / `ask.password2_mask(msg, mask)` | Type-twice secret, loops until both entries match. |
| `ask.date(msg, default)` | Month-grid calendar: arrows move days/weeks, `[]`/PgUp/PgDn change months, Enter accepts ISO date, Esc cancels. |
| `ask.browse(start)` | File browser: arrows move, Enter descends/accepts, Backspace goes up, Ctrl+D picks the directory, q/Esc cancels. Dotfiles hidden. |

Every prompt hides/shows the cursor as needed, restores cooked mode +
cursor shape on exit, treats Ctrl+C as cleanup + `exit(130)`, and
collapses to one clean `? msg answer` line when done.

## Non-TTY fallback

Piped stdin (scripts, CI) degrades automatically: `text`/`password`
use `input()` with defaults, `confirm` parses `y`/`n`/empty,
`select` reads a number from a printed list, `multi_select` reads a
comma list (`1,3`, empty = none).

## Layout

```
src/
  ask.zz       package entry (thin facade — `import ask`)
  line.zz      text + password + path editor
  number.zz    ranged integer input
  confirm.zz   yes/no + destructive prompts
  select.zz    single + multi menus
  search.zz    fuzzy-filter select
  area.zz      multi-line text area (pure doc model)
  editor.zz    $EDITOR integration
  ansi.zz      cursor, erasing, answer lines
  keys.zz      raw key reader + decoders
  buf.zz       editable line buffer (pure)
  path.zz      Tab completion + prefix logic (pure)
  fallback.zz  piped-stdin prompts + pure parsers
examples/demo.zz   all prompts (TTY + piped)
tests/             `zz test` — strictly stdin-free, passes on any terminal
```

`src/*_test` files do not exist on purpose: tests live only in
`tests/`, import only the public package (plus `buf`/`keys`/`fallback`
via path deps for white-box parser/buffer coverage).

## Tests

From `tests/` (no input needed, safe on any terminal):

```sh
zz install && zz test
```

Raw TTY behavior is verified through `examples/demo.zz`
(`cd examples && zz install && zz run demo.zz`) — piped runs exercise
the fallbacks, a real terminal exercises raw mode.

## Published

`ask v0.1.0` is live on the registry
(`https://zz-registry.onrender.com/pkg/ask`) — installable with
`zz add ask`. Republish a new version with `zz publish` from the
package root (runs the test suite, packs, and uploads).
