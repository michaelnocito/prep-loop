# HANDOFF — PrepLoop SHELL-1: make PrepLoop the single shell gating all Analyst Prep Kits

Paste this into a new chat and start executing immediately. Work autonomously through implement → verify → commit → push. Ask only for decisions marked OPEN.

## What PrepLoop is
An interview-cram workflow app. Guided mode = Duolingo-style path (25 nodes, 4 units) mixing Analyst Prep Kit lessons, sims, games and mocks; each sitting runs in an iframe with a countdown timer. Unguided mode = pick-anything queue. Auto miss list reads the kits' spaced-recall queues. Badges, journey journal, streaks.

- Repo/local: C:\Users\Mike\Projects\prep-loop (single index.html, GitHub Pages)
- Live: https://michaelnocito.github.io/prep-loop/
- Kits repo/local: C:\Users\Mike\Projects\analyst-prep-kit — live at https://michaelnocito.github.io/analyst-prep-kit/ — each kit is one self-contained index.html (sql/, excel/, powerbi/, tableau/, python/, stats/)
- Same GitHub Pages origin = shared localStorage and shared Supabase session between PrepLoop and kits. This is the foundation everything relies on.

## The mission (decided by Mike 2026-07-22, green-lit)
1. Fix BUG-1: PrepLoop deep links (`#lesson-<id>`) into a completed lesson resume at the saved stage (kit state `lessonStage`) instead of restarting. Fix = support `#lesson-<id>-restart` in each kit's hash router: reset that lesson's stage, then navigate. Only PrepLoop emits the flag; direct kit visitors unaffected. Update PrepLoop `lessonURL()` to emit it. Kits' hash handler example: sql/index.html ~line 4767 `location.hash.match(/^#lesson-(\d+)$/)`.
2. SHELL-1: kits are reached only through PrepLoop.
   - Kit pages detect top-level visits (`window.top === window.self`) and show a courtesy interstitial: "The prep kits now live inside PrepLoop" + button to https://michaelnocito.github.io/prep-loop/ (deep-linking into the right path node if cheap). OPEN decision for Mike: hard gate (block content) vs soft gate (banner, content still usable). Recommend soft gate first so nothing breaks for existing users and SEO.
   - When framed (`window.top !== window.self`), kits suppress their own topbar/login UI (PrepLoop owns auth surface) but keep writing progress to the same localStorage keys (sqlkit-v1, epk, pbikt-v1, tpk, ppk, spk and `<prefix>-recalls`) — PrepLoop's auto-complete and miss list depend on those keys staying identical.
   - PrepLoop hub: add a "Kits" library view so every kit is still reachable through PrepLoop even off-path (Unguided already links kit roots; make sure all 6 kits are present).

## Known risks (check each)
- Google OAuth cannot run inside an iframe — kit login UI is suppressed when framed, PrepLoop's own Sign in button already handles auth top-level via the kits' shared script (assets/supabase_auth_sync.js; Supabase https://liiivtbyyawueboeavmw.supabase.co, session key `apk-session`).
- GA4 (G-6C09BL3WH1) is in both PrepLoop and kits — framed kit pageviews will double-count a sitting. Acceptable for now; note it in ROADMAP if not addressed.
- Kits are large single files; make surgical edits only. Test each kit after editing with its own live URL.

## Facts you will need (verified 2026-07-22)
- Deep links `#lesson-<id>` work in sql, excel, powerbi, tableau. Python and stats have NO hash handler — adding one (with restart support) is in scope if quick, else note in ROADMAP.
- Kit lesson state: SQL-family `state.doneLessons` + `state.lessonStage` (object keyed by lesson id) in `sqlkit-v1`; Excel `S.lessonsDone` + `S.lessonStage`/`S.currentLesson` in `epk`. Inspect each kit before assuming field names.
- PrepLoop reads kit recall queues `<prefix>-recalls` (sqlkit/epk/pbikt/tpk/ppk/spk) for the miss list — read-only, never write them.
- Kit theme = assets/grain/grain.css + assets/grain/zinc-sky.css; PrepLoop already links these.
- Full kit internals report (theme tokens, auth API, lesson catalogs, storage schemas) is summarized in ROADMAP.md and memory file project_preploop_state.md.

## Working rules (Mike's standing preferences)
- Commit as `Michael Nocito <hello.michaelnocito@gmail.com>`, NO AI/Co-Authored-By trailers. Commit and push without asking after each coherent change.
- A green push is not "live": poll `curl` on the live URL for a marker string added by the change before saying live. GitHub Pages takes ~45-60s.
- After any change, give the live URL + local path. Keep end-of-task replies short and plain. No em-dashes. Never the words "plain English" or "gotcha".
- Test steps labeled <task><letter> (e.g. 001a, 001b) and concrete.
- Don't playtest games with sound; PrepLoop itself has no audio.
- analyst-prep-kit repo: verify the kits still work standalone-in-frame after edits; kits have their own test conventions — keep edits minimal and consistent with inline style.

## Suggested order
1. BUG-1 restart flag: PrepLoop lessonURL() + 4 kits with hash routers. Verify: complete a lesson, deep-link with -restart, confirm it opens at the start. Ship.
2. Framed-mode detection in kits: suppress topbar/login when framed. Verify inside PrepLoop session frame. Ship.
3. Top-level interstitial/banner (per Mike's OPEN gate decision — ask him: hard vs soft). Ship.
4. Python/stats hash handlers if cheap. Update ROADMAP.md status lines as items land.
5. Update memory file project_preploop_state.md at the end.

## Definition of done
- A Guided lesson sitting always starts its lesson at the beginning.
- Every kit shows the PrepLoop hand-off when visited directly (per gate decision) and hides its own auth/topbar when framed.
- Kit progress, recalls, badges and PrepLoop auto-complete all still work end to end on the live domain.
- ROADMAP.md updated (BUG-1 closed, SHELL-1 status), memory updated, everything committed and pushed, live-verified with curl markers.
