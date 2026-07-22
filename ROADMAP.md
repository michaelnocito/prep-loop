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

Still open:
- **OPEN (Mike):** top-level interstitial — hard gate (block kit content) vs soft gate (banner + PrepLoop link, content still usable). Recommendation: soft gate first; nothing breaks for existing users or SEO.
- GA4 (G-6C09BL3WH1) double-counts a sitting (PrepLoop pageview + framed kit pageview). Accepted for now.

Original scope sketch:
- Kits load only inside PrepLoop's session frame; kit-direct visits get a courtesy interstitial pointing to PrepLoop (keep deep links working for SEO or explicitly retire them — decide then).
- PrepLoop owns: role selection, path, timers, restart semantics, miss sweeps, badges, journal, auth, premium gating (ties into the freemium ecosystem plan: free core vs Interview Prep Pass).
- Kit-side work: honor a restart directive, suppress their own duplicate topbars when framed (detect `window.top !== window.self`), keep writing progress to the same localStorage keys so nothing else breaks.
- Risks to check: GA4 double-counting inside iframes; kit features that assume top-level window (OAuth redirect for Google sign-in inside an iframe will need to pop out).

## Also parked
- Sync PrepLoop path progress (`pathDone`, `pathLog`, badges) to Supabase for signed-in users (premium brick; kits' `user_progress` table pattern is the model).
- Deep-link support for Python and Stats kits (`#lesson-<id>` handler missing there).
- Optional: Keygarden as an official warm-up node on the Guided path.
