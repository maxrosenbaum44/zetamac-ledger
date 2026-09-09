# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`zetamac-ledger.html` is the whole project: a single self-contained page that tracks
[Zetamac](https://arithmetic.zetamac.com/) arithmetic-drill scores over time. No build step, no
dependencies, no backend, no test runner. Editing the file *is* the development loop.

It is published as a Claude Artifact and runs with the `artifact` and `downloads` capabilities.

## The one rule that matters: never publish the repo copy as-is

Runs are stored **inside the page**, in a single block:

```html
<script id="log" type="application/json">{ ...state... }</script>
```

The committed copy in this repo is deliberately **blank** (`"entries": []`) because the repo is
public and the runs are personal. The **live artifact holds the real data**. These two copies are
intentionally different and drift apart every time the user logs a run.

**Publishing this file without first pulling the live data block across destroys the user's score
history.** Before any republish:

1. `Artifact` with `action: "read"` and the artifact URL — it saves the full live HTML to a local file.
2. Copy that file's `<script id="log">` block into the local file, replacing the blank one.
3. Apply code edits, then publish.
4. Blank the data block again before committing (see *Committing* below).

## How the page saves itself

`runSave()` in the page tries two paths, in order:

1. `artifact.publish({"data/log.json": ...})` — the files form. Does not reload the view.
2. Falls back to `publishWholePage()`, which `fetch`es its own source, regex-replaces the
   `<script id="log">` block, and calls `artifact.publish(html)`.

In practice path 1 has not been available, so **path 2 is what runs**. Two consequences:

- Every logged run mints a new artifact version, so a local copy goes stale constantly. Expect
  `artifact-changed` notifications during a session and re-sync before editing.
- Because the page only ever rewrites the data block, the live version's code is byte-identical to
  this repo's, modulo that block. That makes the merge a clean transplant, not a real diff.

`localStorage` (key `zetamac-ledger-v1`) is a per-device mirror and fallback, merged on boot.

### When a publish is refused

The tool refuses a publish that is not built on the newest version. The remediation is: `read` the
URL, then `Read` **every line** of the saved version file, then publish. If it still reports
"identical content … resent unchanged" after a faithful merge, that is a known deadlock — a correct
merge is byte-identical to the refused payload. `force: true` resolves it but **requires the user's
explicit confirmation**, since it discards the live version's page.

## State shape and merge semantics

```
{ v, target, window, xMode, settingsUpdatedAt, configs: [...], entries: [...] }
entry  = { id, score, ts, cfg, note, updatedAt, deleted? }
config = { id, name, seconds, ops, updatedAt }
```

- `mergeState(a, b)` is last-writer-wins per entry `id` by `updatedAt`; settings move as a block on
  `settingsUpdatedAt`. Deletes are **tombstones** (`deleted: true`), never splices — removing an
  entry outright lets another copy resurrect it.
- Scores are only comparable within one `config`, which is why every stat and chart filters by it.
- Do not change `STORE_KEY` or the entry shape without a migration; existing pages hold live data.

## Verifying changes

There is no `node` on this machine. Use the JS engine built into macOS.

Syntax-check the app script:

```bash
python3 -c "
import re
js = re.findall(r'<script>\n(.*?)\n</script>', open('zetamac-ledger.html').read(), re.S)[-1]
open('/tmp/app.js','w').write(js)"
osascript -l JavaScript -e 'ObjC.import("Foundation");
var s=$.NSString.stringWithContentsOfFileEncodingError("/tmp/app.js",$.NSUTF8StringEncoding,null).js;
try{ new Function(s); "SYNTAX OK" }catch(e){ "SYNTAX ERROR: "+e.message }'
```

Unit-test pure logic by extracting the functions by name (brace-matching from `function <name>(`)
and running them under `osascript -l JavaScript` with a stubbed DOM. Note JXA has a global `$`
(the ObjC bridge) that cannot be redeclared — run test harnesses inside `new Function(...)()` so a
stubbed `$` stays local. Existing coverage worth preserving on change: rolling averages, OLS slope,
streaks across DST boundaries, bulk-import date parsing, tick generation, and the entry-date field's
refresh rules.

To look at the page, open it directly: `open zetamac-ledger.html`. It runs standalone, falling back
to `localStorage`, with saving disabled.

## Conventions the code already follows

- **Calendar math, not millisecond math.** Use `addDays()`; DST days are 23 or 25 hours, so
  `ts + 86400000` silently miscounts streaks and heatmap cells.
- **Theme tokens go in all three scopes**: bare `:root`, `@media (prefers-color-scheme: dark)`
  guarded by `:root:not([data-theme="light"])`, and `:root[data-theme="dark"]`. A color defined only
  inside a media or `[data-theme]` block renders wrong in the default un-stamped state.
- **The chart palette is validated, not chosen by eye.** Series colors are blue / orange / aqua,
  checked for colorblind separation and contrast against the actual surfaces in both themes. Aqua is
  below 3:1 on the light surface, so it is only used for the goal line, which carries a text label.
  Re-validate if you change them.
- Charts are hand-rolled SVG measured against `clientWidth` and re-rendered on resize; there is no
  charting library.

## Committing

`.claude/settings.local.json` is gitignored. The repo is **public**.

Before committing, confirm the data block is blank. The user asked to be **asked each time** before
any commit that would include real run data.
