# Polish notes — Enumeration Suite

Every change made to the pasted file is listed here, with the date, what was
wrong or missing, what changed, and where. Nothing else was modified: the app
code and the React bundle are otherwise byte-identical to the paste.

Provenance: `index.verbatim.html` is the pasted file saved byte-for-byte
(229,345 bytes, md5 `c62841483122b9cd339a57c12f7b6d4a`, verified against the
original message). It was never edited after transcription. All changes below
apply to `index.html` only (205,409 bytes).

## 2026-11-16 — (a) Local persistence shim for the remediation board

**Why:** the app saves board moves through the Meta AI host's records API
(`POST "actions"` with `records.list` / `records.create` / `records.replace`).
That endpoint does not exist on static hosting, so on GitHub Pages every board
move would fail and the board would always show the "Saved board is
unavailable" message.

**What:** a small classic `<script>` was inserted immediately BEFORE the
`<script type="module">` tag. It wraps `window.fetch` and answers only POSTs
to `"actions"` carrying one of the three records actions, using `localStorage`
(key: `enumeration-suite-remediation-v1`):

- `records.list` → `{ records: [{ id, value }] }`, filtered by collection.
- `records.create` → `{ id }`, honoring `idempotency_key`: a repeat create
  with the same key returns the existing record's id without duplicating it.
- `records.replace` → updates the record with the given id (upserts if the id
  is unknown, so a move is never silently lost).

Responses are shaped `{"data": <payload>}` with HTTP 200, matching what the
app's fetch helper expects (it returns `r.data` and throws when the response
is not ok or carries an error). Every other fetch passes through to the
native `fetch` untouched. No app code or bundle code was modified by this
shim. The shim is labeled "Torque-added" in a comment in the file.

**Verified:** see TEST-REPORT.md — board moves persist across reload,
`records.replace` is used on the second move of the same card (no duplicate
record), and direct `records.create` calls with the same `idempotency_key`
return the same id.

## 2026-11-16 — (b) Removed the four Meta AI host scripts

**Why:** these scripts serve the Meta AI host environment only (safe-area
insets for the host's embedded view, host records-API guard rails, host link
handling, and iframe device-permission forwarding). They have no function on
static hosting, and two of them wrap browser APIs (`fetch`, `window.open`,
anchor clicks) in ways that could interfere with the page outside the host.

**What:** removed, in full, from `index.html`:

| Script | Bytes removed |
|---|---|
| `<script data-metaai-safe-area-runtime>` | 2,564 |
| `<script data-metaai-records-guard>` | 11,816 |
| `<script data-metaai-app-links>` | 11,096 |
| `<script>(function installDevicePermissionRuntime() { … })();</script>` | 2,194 |
| **Total** | **27,670** |

**Kept deliberately:**

- The `<style data-metaai-safe-area>` block (11 lines). It is inert CSS with
  `0px` fallbacks and no script; the app's own markup references the
  `data-safe-area-*` attributes it styles.
- The trailing link-target script (forces external/mailto/tel links to open
  in a new tab). The app renders no links, so it is a no-op in practice, but
  it is harmless and was left as pasted.
- The `<script type="application/json" data-clippy-inline-media-manifest>`
  tag (empty manifest). Harmless metadata, left as pasted.

**Verified:** the polished copy was tested with the scripts removed — all 22
functional checks pass (TEST-REPORT.md), including all 7 tabs, search/filter,
board persistence, export, and the 400px mobile viewport.

## 2026-11-16 — (c1) Fixed `new kn(...)` → `new Map(...)` (real defect)

**What was wrong:** the board-load function (inside the module script, the
`si` function that runs on mount) built a lookup with
`let di = new kn(ce.map((ke) => [ke.value?.seedKey, ke]));` — but `kn` is the
Map *icon component* (`var kn = R("Map", …)`, a React forwardRef object), not
a constructor. `new kn(...)` throws `TypeError: kn is not a constructor`.

**Why it was latent:** in the verbatim file the records call fails first (no
`actions` endpoint), so the throw inside the `await` is caught and the state
updater containing `new kn` never runs — the page renders. Once the
persistence shim makes `records.list` succeed, the updater runs and the
error unmounts the entire React tree (no error boundary), leaving a blank
page. This was reproduced: a polished copy with the shim but without this
fix rendered a blank root with `TypeError: kn is not a constructor` in the
console and as an uncaught page error.

**What changed:** exactly one occurrence, in `index.html` only:
`new kn(ce.map(…))` → `new Map(ce.map(…))`. The icon definition
(`var kn = R("Map", …)`) and its one render usage (`a(kn, {size:16})`, the
Roadmap tab icon) are untouched. The verbatim file was not modified.

## 2026-11-16 — (c2) Restored `--safe-area-inset-top` (regression from (b))

**What was wrong:** the removed safe-area runtime defined the
`--safe-area-inset-top` CSS variable (0px on desktop). The app uses it for
the root element's top padding and the sticky header's `top` offset. With the
variable undefined, the header's computed `top` became `auto`, which disables
sticky positioning — the header would scroll away instead of staying pinned.
Measured: polished copy computed `top: auto` vs the verbatim file's `top: 0px`.

**What changed:** one Torque-added `<style>` element in `<head>` defining
`:root { --safe-area-inset-top: 0px }`, with an explanatory comment. After
the change the polished copy computes `top: 0px`, matching the verbatim file
exactly, and the sticky header behaves as originally rendered.

## Deliberately left unchanged

These were observed during review and intentionally NOT changed, because they
are content quirks of the pasted app rather than functional defects, and the
polish scope was limited to the items above. Flagged here so they can be
decided on separately:

- **Browser tab title is "React Artifact"** (the Meta AI artifact default),
  not "Enumeration Suite."
- **The Overview "MEDIUM" chip is hardcoded** while the risk dial shows the
  computed score (100/100 at the default slider positions). The Reporting
  calculator shows the computed score with its matching label.
- **A hidden chip** (`<span className="chip" style={{display:"none"}}>x</span>`)
  exists in the header. It renders nothing.
- **The HTML export has no charset declaration.** The exported file contains
  an em dash in its title; most browsers will still decode it as UTF-8, but a
  charset meta tag would make it explicit.
