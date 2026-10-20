# 02 — Catching a "human-like" bot by its behavior

> A bot that drives a real browser, curves its mouse, randomizes every delay
> and never clicks the exact center passes the usual checks. The claim here:
> it still can't *behave* like a person for hours — and the server can see that.

## The claim being tested

Cheap bot checks look at what the client *says* about itself: User-Agent,
`navigator.webdriver`, missing headers, requests per minute, clicks landing at
the exact center of a button. A bot that automates a real, already-open
browser (Playwright/CDP attached to a normal profile) and adds human-like
input passes all of them:

- mouse moves along curved paths, slower for far/small targets (Fitts's law)
- click points scattered around the center, only on uncovered pixels
- every delay drawn from a range, plus occasional longer pauses
- hidden/decoy elements checked and avoided before clicking

The claim is that **behavioral signals over a session** still separate such
a bot from a human, because some of them are expensive to fake and the server
sees them directly:

| Signal | Bot (typical) | Human (typical) |
|---|---|---|
| Reaction time: screen shown → next action (server-side) | narrow band, below or near the human floor | ≥ ~200 ms floor, long right tail |
| Fatigue: speed/accuracy over hours | flat | slower, more mistakes later |
| Session rhythm | long even blocks, round-number stops | breaks, irregular schedule |
| Pathing / target choice | always shortest path, always nearest target | detours, wall bumps, "wrong" targets |
| Idle mouse | still until right before a click | fidgets, hovers, moves while reading |
| Key input | one tap per step, never overlaps | holds keys (auto-repeat), rollover |
| Input while window unfocused/hidden | happens (CDP input needs no focus) | impossible |
| Pointer event timing | perfectly regular intervals, no coalesced samples | irregular, multiple coalesced samples |
| Reading time of messages | same regardless of length | grows with text length |
| Daily totals | far above the population | within normal range |

Individually none of these is proof. Combined into a **score**, they should
flag the human-like bot while letting real play through.

## Plan (not built yet)

- **A tiny local web game** (Node core `http` + one static page, no deps):
  a grid map moved with WASD, monsters to walk into, a battle screen with an
  attack button, a result overlay with "next" / "back to map". Same loop shape
  as a real browser game, built only for this case.
- **Client telemetry** (plain JS in the page): per action, send a small
  summary — pointer path stats before the click, click position inside the
  target, key hold/repeat/overlap, `document.hasFocus()` / visibility,
  pointer event interval stats, `getCoalescedEvents()` counts.
- **Server**: logs every action with its own timestamps (screen sent, action
  received) and computes the features above per session.
  - `MODE=naive` — the "vulnerable" build: UA / `webdriver` / rate limit /
    center-click checks only.
  - `MODE=behavior` — the "fixed" build: adds the per-session behavioral score.
- **Clients to compare**:
  - `bot-naive.js` — Playwright defaults (instant clicks at center, fixed delays).
  - `bot-humanlike.js` — attaches to an open browser over CDP; curved mouse,
    random click points, gaussian delays, runs with the window in the background.
  - **human baseline** — play the same page by hand for 15–30 min, a few sessions.
- `report.js` — prints each session's features and score side by side.

Nothing has been written or run yet. This section is the plan to build
against, not a description of what exists.

## What would count as "proved"

- **Proved**:
  - `MODE=naive`: catches `bot-naive.js`, but **passes** `bot-humanlike.js`
    (shows the cheap checks are not enough).
  - `MODE=behavior`: flags `bot-humanlike.js` within a reasonable session
    length (target: < 30 min of play), while **all human baseline sessions
    pass** (no false positives in the sample).
  - Bonus: note which single signals carried most of the score.
- **Disproved / inconclusive**: the human-like bot's scores overlap the
  human sessions (can't pick a threshold without flagging humans) — write
  down which signals failed to separate them, and why.

## Notes / caveats going in

- **Everything client-side can be faked** by a bot that controls the browser
  (it can send fake telemetry). The interesting result is *which* signals are
  costly to fake, and the server-side ones (reaction time, rhythm, totals)
  that need no client cooperation at all.
- **False positives matter more than catches.** Trackpads, accessibility
  tools, very fast players and power users all look "odd" on some axis —
  score, don't hard-block on one rule; escalate (log → slow down → challenge)
  instead of banning.
- A determined bot can add fake fatigue, breaks and detours. That raises its
  cost and lowers its speed — which is often the real goal of detection.
- Small samples: a handful of human sessions from one person is a weak
  baseline. Treat thresholds as a starting point, not a result.
- Run only against the local server built here.
- Fill in the Status column in the root README once you've actually run this.
