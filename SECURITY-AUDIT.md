# Security audit — Enumeration Suite

Date: 2026-11-16. Auditor: Torque. File audited: `index.html` (205,409 bytes,
the polished copy). The verbatim paste (`index.verbatim.html`) was referenced
only to confirm what was removed.

## Scope — read this first

This is a **static review plus a runtime behavior check** of a static,
client-side-only demo with no backend. It is **NOT a penetration test
against a live system**: there is no server, no API, no authentication, no
user accounts, and no live system behind this page to test against. Nothing
here was scanned with an external scanner, and no attack was launched at any
real infrastructure. Findings below are about the file's own code and its
observed browser behavior only.

Method: full-text scans of the polished file for secrets, URLs, and dangerous
sinks (distinguishing app code from the bundled React/Tailwind code), review
of every input-handling surface and of the export builder, review of the
Torque-added fetch shim, and a scripted headless-Chromium run capturing the
console, page errors, and every network request (see TEST-REPORT.md).

## Summary

No vulnerabilities were found. No secrets, no live endpoints, no dynamic code
execution in app code, no exfiltration path, and zero external network
requests at runtime. All findings are informational. The one functional
defect found during testing (a latent `TypeError` in the board-load path) is
a robustness issue, not a security issue, and is fixed and documented in
POLISH-NOTES.md.

## Findings

### 1. Secrets, credentials, tokens — none present (Informational)

Scan terms: `password`, `passwd`, `api_key`/`apikey`, `secret`, `bearer`,
`access_token`, `private key`, `credential`.

Every match is benign:

- `__SECRET_INTERNALS_DO_NOT_USE_OR_YOU_WILL_BE_FIRED` (4×) — React's
  internal identifier, part of the bundled React 18.3.1 code. Not a secret.
- `password` (2×) — React DOM's input-type handling for password fields.
  The app renders no password input.
- `secret` / `credentials` in demo UI copy — the Key Vault inventory card
  ("12 secrets · 1 near expiry"), a remediation task about rotating a Key
  Vault secret, and guidance text advising against sending credentials
  outside approved channels. All invented demo strings; no values.
- No API keys, tokens, private keys, or bearer values anywhere in the file.

### 2. Hardcoded URLs and external requests — none fetched (Informational)

All URL-shaped strings in the file fall into two groups, neither of which is
ever requested:

- XML namespace identifiers (`w3.org` SVG/XHTML/MathML/XLink/XML namespaces)
  used by React DOM and the SVG icons. These are identifiers, not fetches.
- Comment references (Chromium/WebKit/Mozilla bug trackers, GitHub issues,
  the React error-decoder page, tailwindcss.com) inside the bundled
  Tailwind/React code. Comments, never fetched.

The only fetch target in app code is the relative path `"actions"` (the
records API), which the Torque-added shim answers locally from
`localStorage`; it never reaches the network. Runtime verification: across
the full functional test (all tabs, board moves, reload, export), the
request log contained only the page itself. **Zero external network calls.**

### 3. `dangerouslySetInnerHTML` / `innerHTML` / `eval` / `Function` — none in app code (Informational)

- `dangerouslySetInnerHTML`: 13 occurrences, all inside the React DOM bundle
  (prop validation and DOM diffing). Zero in app code; the app never passes
  an `__html` prop (0 occurrences in app code).
- `innerHTML`: 5 occurrences, all React DOM internals (including the
  `dangerouslySetInnerHTML` application path the app never invokes). Zero in
  app code.
- `eval(`: 0 occurrences file-wide.
- `Function(`: 1 occurrence file-wide, inside React's legacy `MSApp`
  (Internet Explorer) guard — a dead code path on modern browsers, not a
  dynamic-code sink reachable by the app.
- Also zero file-wide: `document.write`, `insertAdjacentHTML`, `outerHTML`,
  `srcdoc`, `javascript:` URLs, `WebSocket`, `XMLHttpRequest`, `sendBeacon`,
  `document.cookie`, `sessionStorage`, `URLSearchParams`, `location.search`,
  `location.hash`, `atob`/`btoa`.

### 4. Export-HTML builder — static data only, no user input (Informational)

The HTML export (`Ic` in app code) builds one static template string. The
only interpolated values are the computed risk score (a number) and its
label (one of four fixed strings: Critical/High/Medium/Low). No text input,
search value, or board content flows into it. The file is delivered via the
file's single `Blob` + `URL.createObjectURL` pair and a programmatic anchor
click with a fixed filename
(`enumeration-suite-executive-summary.html`); the object URL is revoked
after one second. The exported document contains no scripts.

Minor note: the exported HTML has no charset declaration. Cosmetic only;
flagged in POLISH-NOTES.md as deliberately left unchanged.

### 5. localStorage usage — board state only, no sensitive data (Informational)

`localStorage` appears exactly 3 times in the polished file, all in the
Torque-added shim (one comment, one `getItem`, one `setItem`). The single
key, `enumeration-suite-remediation-v1`, stores board records:
`{id, collection, value: {seedKey, title, owner, severity, status},
idempotency_key}`. This is demo task data only — no credentials, no personal
data, no browsing data. It never leaves the browser: the shim reads and
writes it locally and makes no network call. No `sessionStorage`, no
cookies. The app code itself never touches storage.

### 6. Input-handling surfaces — all values contained (Informational)

| Surface | Handling |
|---|---|
| Host search box | Value used only in `.filter()` / `.includes()` string matching; rendered via React text children (auto-escaped). Never inserted as HTML, never sent anywhere. |
| Subnet search box | Same handling as host search. |
| OS filter dropdown | Fixed three-option select; value used only in `.includes()` matching. |
| Risk sliders (3) | Values coerced to numbers (`+value`) before use in arithmetic. |
| Read-only textareas (PowerShell commands) | Static demo command text; `readOnly`. The Copy button uses the deprecated `document.execCommand("copy")` (1 occurrence) — functional, no security impact. |
| Board move buttons | Fixed direction arguments (-1/+1); the moved record's value is built from the seed task's own static fields plus the new status. |
| URL parameters | None read. No `location.search`, `location.hash`, or `URLSearchParams` anywhere. |
| Links | The app renders zero anchor tags. The only programmatic anchor is the export download (blob URL, fixed filename). |

`postMessage` appears once, inside React's scheduler (an internal
`MessageChannel` used for task scheduling). The app registers no message
listeners. The removed device-permission host script was the only
`postMessage` sender in the pasted file; it is gone from the polished copy.

### 7. Removed host scripts — reduced API-wrapping surface (Informational)

The verbatim paste's four Meta AI host scripts wrapped `fetch`,
`window.open`, and anchor-click behavior for the host environment. All four
were removed (POLISH-NOTES.md (b)); marker strings verified absent from the
polished file. The polished file's only `fetch` interception is the
documented Torque shim, which answers only POSTs to `"actions"` with the
three records actions and passes everything else to the native `fetch`.

### 8. Fixed defect with no security impact (Informational)

The board-load path contained `new kn(...)` where `kn` is an icon component,
throwing a `TypeError` once the records call succeeds and unmounting the
whole app. Fixed to `new Map(...)` (POLISH-NOTES.md (c1)). This was a
correctness/robustness defect: it caused a blank page, not a security
exposure. No user input was involved.

## Not covered

- HTTP response headers (CSP, etc.) are set by the hosting platform, not by
  this file; GitHub Pages serves its own defaults. The file itself uses no
  inline event-handler attributes and no `javascript:` URLs, so it is
  compatible with a strict CSP, but no CSP was tested.
- The published page on its real origin was not loaded in this environment
  (see TEST-REPORT.md, "Not verified"). Runtime checks were performed over
  `file://` with the same code paths.
- Accessibility, performance, and visual review are out of scope for this
  audit.

## Verdict

Safe to publish as a static demo: no secrets, no backend, no exfiltration
path, no executable user input, and no external requests. The honest limits
in README.md ("What this is not") should stay attached to the project
wherever it is presented.
