# PrepLoop Roadmap

Live: https://michaelnocito.github.io/prep-loop/ · Local: C:\Users\Mike\Projects\prep-loop
Status: v0.4.0 shipped 2026-07-22. Items below are PARKED until Mike finishes tightening repos and tools.

## BUG-1 (CLOSED 2026-07-22): deep-linked lessons resume at the end instead of restarting

**Fix shipped:** `lessonURL()` now emits `#lesson-<id>-restart`; all 6 kits' hash routers (python and stats got new handlers) reset that lesson's saved stage/completion before opening it. Direct kit visitors using plain `#lesson-<id>` are unaffected.


**Symptom:** starting a Guided lesson step opens the kit at `#lesson-<id>`; if that lesson was completed before, the kit picks up at its saved stage (end of lesson) instead of the beginning.

**Cause:** kits persist per-lesson position in their state (`lessonStage` in `sqlkit-v1`, Excel `S.lessonStage`/`currentLesson` in `epk`, same pattern in the other kits). The `#lesson-<id>` hash router navigates to the lesson but does not reset the stage.

**Options:**

1. **Restart flag (small, kit-side).** PrepLoop links to `#lesson-<id>-restart`; each kit's hash handler resets that lesson's stage before navigating when the suffix is present. Direct kit users are untouched because only PrepLoop emits the flag. One small edit per kit (6 kits), one edit in PrepLoop's `lessonURL()`. Low risk. Good interim fix.
2. **PrepLoop as the shell (chosen direction, larger).** Move the whole kit experience under PrepLoop: kits are only reached through PrepLoop (embedded), and PrepLoop owns navigation, restart behavior, auth surface, and the freemium gate. Direct kit URLs show an "Open in PrepLoop" hand-off (or redirect). This resolves the reconcile problem by making PrepLoop the only navigation app.

**Decision (Mike, 2026-07-22):** direction 2 — gate access to the kits through PrepLoop. Do option 1 first as the interim fix if sweeps/lessons get real use before the shell work starts.

## SHELL-1 (IN PROGRESS 2026-07-22): PrepLoop as the single entry point for all kits

Shipped so far:
- Framed-mode detection lives in the kits' shared `assets/supabase_auth_sync.js`: when `window.top !== window.self` the page gets an `apk-framed` class and CSS hides `#loginBtn`, `#logoutBtn`, `#userEmail` and the "All Kits" back button. Covers all 6 kits with one file. Progress keeps writing to the same localStorage keys.
- Python and Stats kits now support `#lesson-<id>` deep links (with `-restart`); PrepLoop marks them `deep:true`.
- Unguided library now lists all 6 kits (Stats added).

Gate decision (Mike, 2026-07-22, revised same day): **NO gate.** The hard gate shipped and was reverted hours later — it dead-ended visitors on PrepLoop's role picker with no visible way into the kits. New model: the analyst-prep-kit hub is the landing page, retitled **"Grain — Analyst Prep Kit"** (grain design system, no visual changes), with PrepLoop as a featured "Guided" card above the kit list. PrepLoop is one door among several, not the only door. The `?via=loop` bypass flag was reverted along with the gate. Framed-mode auth suppression and the `-restart` fix remain.

Still open:
- GA4 (G-6C09BL3WH1) double-counts a sitting (PrepLoop pageview + framed kit pageview). Accepted for now.

Original scope sketch:
- Kits load only inside PrepLoop's session frame; kit-direct visits get a courtesy interstitial pointing to PrepLoop (keep deep links working for SEO or explicitly retire them — decide then).
- PrepLoop owns: role selection, path, timers, restart semantics, miss sweeps, badges, journal, auth, premium gating (ties into the freemium ecosystem plan: free core vs Interview Prep Pass).
- Kit-side work: honor a restart directive, suppress their own duplicate topbars when framed (detect `window.top !== window.self`), keep writing progress to the same localStorage keys so nothing else breaks.
- Risks to check: GA4 double-counting inside iframes; kit features that assume top-level window (OAuth redirect for Google sign-in inside an iframe will need to pop out).

## PLAYTEST TRIAGE — 2026-07-27 (tracker project "Preploop")

### PL-2: master reset — clear every kit, game and pathed item from PrepLoop

Mike: "Need to be able to reset all kits and games, all pathed items from here."

Today PrepLoop's Reset path button clears only `pathDone`, the path badges and the
journal, and snapshots `S.kitBaseline` so kit lessons finished before the reset stop
counting. The kits' own progress (`sqlkit-v1`, `epk`, `ppk`, `pbikt-v1`, `tpk`, `spk`,
plus the `*-recalls` / `*-recall-wins` / `*-last-visit` / `*-streak` families) is
untouched, and the games' localStorage is untouched. Mike wants one control here that
wipes the lot, so a clean run is one click instead of a tour of every app.

Doable because PrepLoop and the kits share an origin (`michaelnocito.github.io`) — the
kit keys are readable and writable from PrepLoop today, which is exactly how the auto
miss list and the auto-complete of path nodes already work. The games are the open part:
`sql-trail`, `sql-quest` and the arcade apps are same-origin too, but each owns its own
key family and its own idea of what a reset means (SQL Trail alone keeps player identity,
graves and cloud run history — wiping the recovery code would orphan his leaderboard runs).

**Requirements:**

- One destructive control, clearly labelled, behind a confirm that names exactly what
  will be cleared — never a silent wipe.
- Scoped checkboxes rather than one blunt button: kits · path/PrepLoop · games. Mike's ask
  is "all", but the confirm should still show the three groups so he can see the blast radius.
- Must NOT clear `apk-pass` (purchase entitlement), `apk-coach-key`, `sim2-apikey`, or the
  Supabase session — the same guards the kits' own "Reset all kits" already honours.
- Build the key list as a **prefix sweep**, not a hand-maintained array. The kits' own
  reset-all drifted exactly that way (Tableau and Stats silently stopped clearing recall
  keys) and had to be rewritten as a sweep in analyst-prep-kit v1.175.0. Do not repeat it.

**Open question for Mike:** does "all games" include SQL Trail's player identity and its
cloud run history, or only local progress? Recommend local progress only, keeping the
recovery code — losing it costs him leaderboard history he cannot get back, and a reset
is meant to be a fresh study run, not an account deletion.

Status: **not started, roadmap only.** Small-to-medium; the kit half is straightforward,
the games half needs the per-game key inventory first.

### PL-3: quick recall leaves the kit lessons; the miss list becomes the review home

Mike: "Remove quick recall they can go to the mistake auto tracker and review. This is on
the kits lessons." Filed here because PrepLoop's auto miss list IS the mistake tracker he
is pointing at — but the change itself lands in the analyst-prep-kit repo, where the item
is written up in full as **[P4]** at the top of that ROADMAP.

**Decision (Mike, 2026-07-27): remove both** — the in-lesson recall cards and the
confidence rater. "We are autotracking mistakes now, or should be elsewhere."

Checked before filing, and the concern that prompted the question does not apply: the
confidence rater is already gone from all 6 kits (only the boot-time migration that
deletes `state.confidence` remains), and the `<prefix>-recalls` queue is already fed by
**auto-recorded misses** — `recordMiss()` writes every wrong cue to `state.misses`, and
`_queueRecalls()` only schedules a lesson's cues when that lesson has misses. So the miss
list PrepLoop reads keeps filling on its own, and the Recall sweep nodes keep working.

**PrepLoop impact: none.** No change needed here. The work is six edits in
analyst-prep-kit (drop the mid-lesson `#v2-recalls` block; keep the queue, the misses and
the kits' own on-demand Review view). Tracked as [P4] there.

Status: **no PrepLoop work. Closed.**

## Also parked
- Sync PrepLoop path progress (`pathDone`, `pathLog`, badges) to Supabase for signed-in users (premium brick; kits' `user_progress` table pattern is the model).
- Deep-link support for Python and Stats kits (`#lesson-<id>` handler missing there).
- Optional: Keygarden as an official warm-up node on the Guided path.

---

## Dev cockpit (not started)

**Every new app starts with one now.** This one does not have one yet, so it is owed.

The dev cockpit is the instrument panel that makes it possible to exercise one piece of the
app repeatedly without working through everything around it. The canon — the full control
list, the reasoning behind each control, and the app-applicable translation of each — is
`BUILD_PILLARS.md`, section **"A. The dev cockpit"**, in `C:\Users\Mike\Projects\play-area`.
The implementation is already written: `play-area/dev-cockpit.js` plus
`play-area/harness-lib.js` for the headless half. Copy them in; declare this app's own knobs.

What that means here:

- **Jump straight to one screen, one state, one record** — no clicking through a flow to
  reach the thing being worked on
- **Bypass auth, quotas, rate limits and paywalls** while testing (the no-fail toggle)
- **Freeze and single-step** any animation, timer, queue or polling loop
- **Slow-motion** on transitions and network timing, so what the eye missed becomes visible
- **A latency readout** — time to first paint, time to a response landing
- **Instant reset to a known seeded state**, in one keystroke
- **Layout, focus-order and hit-target overlays**
- **Every timing, threshold and limit the app's feel depends on, on a live slider**
- **A numbers dump** — one keypress writes a pasteable line plus a `<app>-tuning.txt` file.
  That file is the handoff to the next session; without it the tuning dies with the tab.
- **A headless harness** (`node <app>-harness.js`) so an agent can prove a change without
  asking a human to click

Gated behind `?dev=1` (auto-on for localhost), wrapped in `DEV:BEGIN` / `DEV:END` strip
markers, and nothing inside it load-bearing: delete the block and the app runs identically.
