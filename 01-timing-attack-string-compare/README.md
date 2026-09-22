# 01 — Timing attack on non-constant-time secret comparison

> No rate-limit on a secret check means an attacker can brute-force it
> blind. Use `===` for that check, and it's not blind anymore — every guess
> that's "closer" replies a hair slower. That's brute-forcing with a map.

## The claim being tested

Comparing secrets with `===` (or any early-exit string comparison) leaks
information through response time: a guess that matches more leading bytes
takes a *tiny* bit longer to reject than one that matches fewer, because the
comparison bails out at the first mismatched byte. Given enough samples, an
attacker can recover a secret one byte at a time without ever guessing it in
one shot — and never sees the actual value, only the timing signal.

`crypto.timingSafeEqual` is claimed to close this because it always compares
every byte, so response time no longer correlates with how many leading
bytes matched.

Generic shape of the vulnerable pattern: a server receives a caller-supplied
token in a request (header or query param) and checks it against a known
secret with `===` before trusting something else the caller sent — e.g.
deciding whether to honor a client-supplied value that overrides normal
request handling.

## Plan (not built yet)

- A small HTTP server with two modes: one comparing the incoming token with
  `===`, one comparing with `crypto.timingSafeEqual` (length-checked first) —
  same secret, same endpoint, just swap the comparison function.
- An attack script that repeatedly requests the endpoint, guessing the
  secret one character at a time by picking whichever candidate character
  gives the highest *average* response time at that position.
- No external dependencies expected — this should be doable with Node core
  modules alone (`http`, `crypto`).

Nothing has been written or run yet. This section is the plan to build
against, not a description of what exists.

## What would count as "proved"

- **Proved**: against `MODE=unsafe`, `attack.js` recovers a string that
  matches the real secret (or a large majority of characters — network/CPU
  jitter means it won't always be 100%, especially over real network vs.
  localhost). Against `MODE=safe`, the same script's guesses look basically
  random / don't converge on the real secret.
- **Disproved / inconclusive**: no meaningful difference in recovery rate
  between `unsafe` and `safe` modes (would suggest local jitter is drowning
  out the signal — worth noting, and worth trying with more samples or over
  an actual network hop instead of localhost).

## Notes / caveats going in

- This is a *localhost* timing attack. Real-world exploitation over the
  public internet is much harder because network jitter dwarfs the
  nanosecond-scale signal — that's part of why this is worth testing
  locally first, where the signal is as clean as it'll ever be.
- `attack.js` needs to know the secret *length* up front (`SECRET_LENGTH`
  env var). Recovering unknown length is a separate, harder problem — out of
  scope for this PoC, which is about the comparison itself.
- Fill in the Status column in the root README once you've actually run this.
