# Test report — Enumeration Suite

Date: 2026-11-16. Tester: Torque (headless Chromium, scripted checks).

## Method and environment

- Browser: Chromium at `/opt/meta-chromium/chrome`, driven by puppeteer-core
  25.17.0 in headless mode, fresh browser profile per run (so localStorage
  starts empty every run).
- The pages were loaded via `file://` URLs. The Hatch Chromium build blocks
  navigation to a local HTTP server (`ERR_BLOCKED_BY_LOCAL_NETWORK_ACCESS_CHECKS`),
  so `python3 -m http.server` could not be used as the test origin. The app is
  fully self-contained with no CDN or backend, so `file://` exercises the same
  code paths; the fetch shim answers the records calls before any network
  access, and the request log below confirms no network activity either way.
- Console output and uncaught page errors were captured through the harness
  on every run and are reported verbatim below — the console was read, not
  assumed.
- Viewports: desktop 1280×900, mobile 400×800.

## Step 1 — Verbatim file render check

File: `index.verbatim.html` (229,345 bytes, md5
`c62841483122b9cd339a57c12f7b6d4a`).

| Check | Result |
|---|---|
| Page renders (root mounts, header + nav visible) | PASS — root 1 child, 1,695 chars of text; title "React Artifact" |
| Overview tab shows content | PASS — heading "Know what you own. Protect what matters." |
| Host Enumeration tab shows content | PASS — heading "Host Enumeration" |
| Network tab shows content | PASS — heading "Network Visibility" |
| Active Directory tab shows content | PASS — heading "Active Directory Review" |
| Cloud tab shows content | PASS — heading "Cloud · Azure / Microsoft 365" |
| Reporting tab shows content | PASS — heading "Reporting" |
| Roadmap tab shows content | PASS — heading "Roadmap" |
| Uncaught page errors | None |

Console during the verbatim run (read via harness): two errors, both the
expected records-endpoint failure — the `fetch("actions")` call blocked by
CORS on `file://` (`net::ERR_FAILED`). The app catches this and shows the
"Saved board is unavailable in this preview" status. No other console
output. Requests observed: only the page itself and the failed `actions`
call — zero external requests.

Conclusion: the verbatim paste renders cleanly, all 7 tabs switch and show
content. No bundle patching was needed or performed.

## Step 3 — Functional test, polished copy

File: `index.html` (205,409 bytes). 22/22 checks passed, on two consecutive
runs (the second run after the `--safe-area-inset-top` fix, POLISH-NOTES.md
(c2), with identical results).

| # | Check | Result | Evidence |
|---|---|---|---|
| 1 | Overview tab renders | PASS | h1 "Know what you own. Protect what matters.", 1,517 chars |
| 2 | Host Enumeration tab renders | PASS | h1 "Host Enumeration", 1,926 chars |
| 3 | Network tab renders | PASS | h1 "Network Visibility", 1,186 chars |
| 4 | Active Directory tab renders | PASS | h1 "Active Directory Review", 1,243 chars |
| 5 | Cloud tab renders | PASS | h1 "Cloud · Azure / Microsoft 365", 1,325 chars |
| 6 | Reporting tab renders | PASS | h1 "Reporting", 1,792 chars |
| 7 | Roadmap tab renders | PASS | h1 "Roadmap", 1,157 chars |
| 8 | Host search "SRV-DB" | PASS | "1 hosts shown" |
| 9 | OS filter "Ubuntu" | PASS | "2 hosts shown": WS-OPS-008, SRV-WEB-03 |
| 10 | OS filter "All OS" | PASS | "8 hosts shown" |
| 11 | Host detail panel opens | PASS | Clicking WS-ENG-031 shows it as the detail heading |
| 12 | Detail panel toggle collapses | PASS | Detail content visible before toggle, hidden after |
| 13 | Risk sliders change the score | PASS | 100/100 at defaults (4/3/6) → 32/100 with label Low at (1/1/0) |
| 14 | Board move shows Saved message | PASS | "Saved: … moved to In Progress." after moving "Review 4 Domain Admins" |
| 15 | Board move persists to localStorage | PASS | 1 record under `enumeration-suite-remediation-v1`, t2 status `progress` |
| 16 | Board persists across reload | PASS | After reload: "1 remediation update(s) loaded."; column counts 2/3/2 |
| 17 | Second move uses replace, no duplicate | PASS | Still 1 record for t2, status `done` |
| 18 | `idempotency_key` honored | PASS | Two direct `records.create` calls with the same key returned the same id; 1 record kept, original value intact |
| 19 | HTML export downloads a file | PASS | `enumeration-suite-executive-summary.html` downloaded; contains the summary heading and demo-data note |
| 20 | PDF export shows simulated message | PASS | "PDF export simulated for this prototype…" status shown |
| 21 | Mobile 400px, no horizontal overflow | PASS | scrollWidth = clientWidth = 400, no horizontal scrollbar |
| 22 | Zero external network requests | PASS | Request log: only the page itself (loaded twice — initial load + reload) |

Console during the polished runs (read via harness): **zero messages.**
Uncaught page errors: **none.**

## Defect found and fixed during testing

A polished copy built with the persistence shim but WITHOUT the `new kn` fix
rendered a blank page: `TypeError: kn is not a constructor` in the console
and as an uncaught page error, root element empty. After the one-word fix
(`new kn` → `new Map`, POLISH-NOTES.md (c1)), the same build passes all
checks above. The verbatim file does not exhibit this because its records
call fails before the defective line can run.

## Header/sticky check

After removing the safe-area runtime, the sticky header's computed `top` was
`auto` (verbatim: `0px`), which disables sticking. After the
`--safe-area-inset-top: 0px` fix (POLISH-NOTES.md (c2)), the polished copy
computes `top: 0px`, matching the verbatim file; header geometry identical in
both (top 0, height 112.5px; title at 14.75–30.75px).

## Known behaviors (not defects)

- The default risk score is 100/100: the formula
  `min(100, round(criticality×18 + exposure×14 + findings×4))` with the
  default sliders (4, 3, 6) gives 138, capped at 100. The Overview "MEDIUM"
  chip is hardcoded and does not match; the Reporting calculator shows the
  computed label (Critical at defaults).
- The browser tab title is "React Artifact" (see POLISH-NOTES.md, left
  unchanged).

## Not verified

- Behavior on a real HTTP origin (GitHub Pages) was not observed — the test
  origin was `file://` for the reason above. The shim intercepts the records
  calls before any network access, so origin should not change behavior, but
  the published page itself was not loaded in this environment.
- Site-player video playback and the walkthrough video are Robert's step via
  his Meta AI route; only the script was drafted (see walkthrough-script.md).
