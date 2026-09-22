# lab-security-poc

Personal lab for testing security concepts I run into during real work — mostly
cases where I was told "X is theoretically possible" and want to actually
prove (or disprove) it myself, hands-on, in an isolated sandbox.

Nothing here targets a real system. Every case runs against a throwaway
local server built for that case only.

`lab-` prefix is just so it's easy to filter/find on GitHub. `poc` means
"proof of concept/knowledge" — validating an idea, not building an attack
tool.

## Cases

| # | Case | Category | Status | Folder |
|---|------|----------|--------|--------|
| 01 | Non-constant-time secret comparison (`===`) leaks the secret via response-time side channel; fixed with `crypto.timingSafeEqual` | Timing attack / side-channel | Not yet proved | [`01-timing-attack-string-compare/`](./01-timing-attack-string-compare) |

### Status legend
- **Not yet proved** — case written up, PoC not built or not run yet
- **Proved** — PoC demonstrates the vulnerable behavior against the "unsafe" build, and shows the "safe" build closes it
- **Disproved / inconclusive** — tried it, couldn't reproduce a usable signal (worth noting why — still useful knowledge)

## How each case folder is structured

Each case gets its own folder with:
- `README.md` — what the case is, why it matters, how to run it, what result counts as "proved"
- an intentionally **vulnerable** build
- a **fixed** build (the mitigation)
- an **attack script** that targets both, so you can compare results side by side
