# Walkthrough script — Enumeration Suite (avatar video)

Status: script draft only. The video clip itself is Robert's step via his
Meta AI route — no video was produced here.

## Required label (must appear wherever the video appears)

The narration is an AI voice speaking as Robert's avatar, made with AI for
fun to walk through the explanation, not his real voice.

Placement: burn this label into the video itself (on-screen text, readable
for the full clip or at minimum the opening seconds), and repeat it in the
post caption / description on every platform where the video is published
(portfolio site, GitHub README, LinkedIn). This follows the standing rule
for every AI-generated video of Robert: the narration and likeness are AI
speaking as his avatar, made for fun, not his real voice.

## Narration script (first person, ~80 seconds, ~200 words)

> This is Enumeration Suite, a defensive SOC learning platform I built as a
> prototype. Everything you're seeing is sample demo data — nothing here
> touches a real network. There's an authorized-use banner on every screen,
> because enumeration without permission isn't defense — it's trespassing.
>
> On the Overview, I get the calm picture: 247 assets, 231 managed, MFA
> coverage, and a risk dial I can drive with the calculator.
>
> Host Enumeration is inventory first — eight demo hosts I can search and
> filter, with a detail panel for services, software, processes, and users,
> plus read-only PowerShell commands for my own authorized systems.
>
> Network checks approved subnets against a baseline. Active Directory keeps
> privilege small — groups, stale accounts, MFA, and a Kerberos checklist.
> Cloud covers Azure subscriptions, Entra roles, and storage exposure.
>
> Reporting ties it together: CIS and NIST mappings, the risk calculator, an
> HTML export, and a remediation board where moves save in the browser. The
> board is the part I use most — moving a card from To Do to Done actually
> saves, right in the browser, no server needed.
>
> And the Roadmap shows where it goes next — from inventory toward a full
> defensive platform. Inventory first. Protect what matters.

## Shot list

Captured from the running app at desktop width unless noted. Screenshot
files in `screenshots/` match shots 1–7.

| # | Time | Shot (on-screen action) | Narration cue |
|---|---|---|---|
| 1 | 0:00–0:12 | Overview tab, full page. Banner visible at top, stat cards, risk dial. | "This is Enumeration Suite…" through "…it's trespassing." |
| 2 | 0:12–0:24 | Overview tab. Slow scroll or hold on the stat cards and the risk dial. | "On the Overview, I get the calm picture…" |
| 3 | 0:24–0:38 | Host Enumeration tab. Type a short search (e.g. "SRV"), click a host row so the detail panel shows, scroll to one PowerShell card. | "Host Enumeration is inventory first…" |
| 4 | 0:38–0:48 | Network tab, then cut to Active Directory tab. Hold 3–4 seconds on each: subnet table, then privileged groups / MFA cards. | "Network checks approved subnets… Kerberos checklist." |
| 5 | 0:48–0:55 | Cloud tab. Hold on subscriptions / Conditional Access / storage check. | "Cloud covers Azure subscriptions…" |
| 6 | 0:55–1:12 | Reporting tab. Show the calculator sliders moving the score, then the remediation board: click Move on one card and show the "Saved" message. | "Reporting ties it together… no server needed." |
| 7 | 1:12–1:20 | Roadmap tab. Hold on the five stage cards. | "And the Roadmap shows where it goes next…" |
| 8 | 0:00–1:20 | Label overlay (all shots): the required label text, burned in. | — |

Optional closer (if the Meta AI route allows a final card): the app's own
footer line — "Enumeration Suite Prototype · Sample data for learning."

## Facts boundary

Only the following claims may be made about the app in the video, the
caption, or the post. Everything here was verified against the running app
or the audited file (see TEST-REPORT.md and SECURITY-AUDIT.md).

May claim:

- Defensive SOC blue-team learning platform, prototype v0.9.
- All data shown is sample/demo data for learning; nothing touches a real
  network or host.
- Seven tabs: Overview, Host Enumeration, Network, Active Directory, Cloud,
  Reporting, Roadmap.
- 8 demo hosts, searchable and filterable by OS, with a detail panel.
- 5 read-only defensive PowerShell inventory commands (copy buttons).
- Risk calculator: three sliders drive a 0–100 score.
- Remediation board: 7 seed tasks; moves save in the browser via localStorage
  and persist across reloads on the same browser.
- HTML executive-summary export downloads a real file.
- Fully self-contained single file: no CDN, no backend, zero external
  network requests.
- Authorized defensive use only, on systems you own or manage.

Must NOT claim:

- Live scanning, live enumeration, or any connection to real systems.
- Real host, network, identity, or cloud data of any kind.
- A working PDF export (the PDF button is simulated in the UI and says so).
- A backend, accounts, sync, or saved progress beyond the browser.
- Certification, compliance attestation, or production readiness.
- That the narration or likeness is Robert's real voice or appearance — the
  required label above governs.
