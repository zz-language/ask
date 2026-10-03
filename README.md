# ask

Advanced interactive CLI prompts for ZZ. Pure ZZ, zero dependencies
(only `std` modules: `colors`, `process`, `str`, `term`, `vec`).

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
| `ask.multi_select(msg, options)` | Space toggles `[x]`/`[ ]`, Enter confirms. Returns `[str]`. |

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
  line.zz      text + password editor
  confirm.zz   yes/no prompt
  select.zz    single + multi menus
  ansi.zz      cursor, erasing, answer lines
  keys.zz      raw key reader + decoders
  buf.zz       editable line buffer (pure)
  fallback.zz  piped-stdin prompts + pure parsers
examples/demo.zz   all five prompts (TTY + piped)
tests/             `zz test` — strictly stdin-free, passes on any terminal
```

`src/*_test` files do not exist on purpose: tests live only in
`tests/`, import only the public package (plus `buf`/`keys`/`fallback`
via path deps for white-box parser/buffer coverage).

## Requirements

Needs a `zz` toolchain with `std.term` (raw mode, single-key reads).
Older toolchains fail at `import std.term` — upgrade `zz` first.

## Tests

From `tests/` (no input needed, safe on any terminal):

```sh
zz install && zz test
```

Raw TTY behavior is verified through `examples/demo.zz`
(`cd examples && zz install && zz run demo.zz`) — piped runs exercise
the fallbacks, a real terminal exercises raw mode.
