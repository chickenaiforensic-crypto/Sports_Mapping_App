FIX NOTE — DALXIC Stats Engine v2, Round 3
From: Auditor / Supervisor review of DALXIC_Stats_Engine_v2_dc.html
Spec reference: DALXIC_Ratings_Engine_v1.19.md

Your self-audit was good — you caught three real falsehoods I'd missed. This note covers the resolution on those, plus what I found afterward digging into the zero-data question you raised.

---

FIX 1 — Points Redistribution bug (Clear Win / Win loser-side arithmetic)

Confirmed real, isolated to two lines:

```js
case 'CLEAR WIN': return { w: W + 0.75 * L, l: L - 0.5 * L, bucket, adjusted: true };
case 'WIN':        return { w: W + (1/3) * L, l: L - (1/3) * L, bucket, adjusted: true };
```

The loser's deduction multiplies against `L` (the loser's own old points) — it should multiply against `W` (the winner's old points), matching spec exactly and matching how Clear Wrong/Wrong already do it correctly:

```js
case 'CLEAR WIN': return { w: W + 0.75 * L, l: L - 0.5 * W, bucket, adjusted: true };
case 'WIN':        return { w: W + (1/3) * L, l: L - (1/3) * W, bucket, adjusted: true };
```

---

FIX 2 — Mute checkboxes: wire for real, don't just correct the copy

`computeRollout()` currently ignores `mutedYears` and hardcodes 2022→present. Per spec v1.19 §1.10, this needs to actually work: read the mute state, start the rollout from the earliest unmuted year (using that year's real data as the first incoming source), and support the shift-forward pattern (calibrate a year, let it become the new baseline, advance the mute range, repeat). Falls back to the full 2021-baseline default only when nothing is muted.

This is explicitly a pre-launch process the team runs once, not a live per-visit feature — build it as something operated manually via the mute UI + a run action, not something that needs to be fast or automatic on every page load.

---

FIX 3 — Stale "Phase 2 stays gated" disclaimer

Straightforward text correction — the rollout has executed; update the copy to say so.

---

FIX 4 — "Update System" button: repurpose, don't leave vestigial

Per your own finding, this button currently does nothing to the store actually driving Ratings H2H. Repurpose it to trigger a manual rollout re-run using whatever mute state is currently set — this makes it the actual action button for Fix 2's mechanism, rather than a button that claims to commit and doesn't.

---

FIX 5 — Zero/zero tie redistribution (corrected from an earlier version of this note — read this one, not any prior draft)

When `W === 0 AND L === 0`: this is not a no-adjustment Tie. The real match still happened and its result carries signal.

1. Classify the real match via the existing loser-points table (already in §1.6) — e.g. budget 30, loser scored 16–22 real points → Win.
2. Loser's average for that class = midpoint of the loser's range. Win/budget 30 → midpoint of 16–22 = 19.
3. Winner's average is NOT the same number mirrored — it's derived as `budget − loser's average` (30 − 19 = 11 in this example), since winner + loser points always sum to the match budget in the real result.
4. Apply from a zero base: winner's new rating = 0 + winner's average (+11). Loser's new rating = 0 − loser's average (−19).

Build-time sanity check: winner's average + loser's average should always equal the match budget. If your implementation makes both values equal (e.g. ±19), that's a sign the two ranges got conflated — they're genuinely different ranges (loser's is tabulated directly; winner's is the remainder), not mirror images of each other except in a rare, coincidental 50/50 split.

---

FIX 6 — `priorRating()` returns `null` for genuinely zero-data players; it should return a real rating of 0

Confirmed by direct inspection: `let out = null;` stays null when no calibrated record and no raw fallback edition exists for a player. This means zero-data players are currently excluded from H2H comparison entirely (falls through to "select both players"), rather than participating with a real rating of exactly 0 — which contradicts §1.6's own fallback rule ("rating = 0, a true data-confirmed zero, not a search failure").

Fix: when the existing fallback chain in `priorRating()` exhausts with nothing found, return `{ incoming: 0, calibrated: false, source: 'no prior data — confirmed zero' }` instead of `null`.

One distinction to preserve, not collapse: a rating of exactly 0 can also arise from real accumulated data that happens to compute to 0 — that's already handled correctly by the existing code path (a real `out` object gets set, with a real source). Don't let this fix conflate the two. Both must compare identically (0 beats any negative — already correctly handled by the plain numeric subtraction in the H2H comparison, no change needed there), but the `source` label shown to the user should honestly say which kind of zero it is — "no prior data" vs. the actual tournament/year that computed to 0.

---

CONFIRMED, NO ACTION NEEDED

- Zero-vs-negative ordering in the H2H comparison itself (`d = rA.incoming - rB.incoming`) is correct as written — plain numeric subtraction, no special-casing of 0 as falsy. Fix 6 is what was actually blocking this from mattering.
- Wimbledon 2026 zero-data count: confirmed 23 players (not 0), mostly qualifiers/wildcards with no ATP/WTA tour-level match anywhere in the 2021–2025 store. Four names — Felix Gill, Harry Wendelken, Max Basing, Nicolas Mejia — match exactly what was flagged in the very first backtest spec, early in this project, for a different tournament context. This is a real, structural, known gap (the store's ATP/WTA-only scope), not a new defect. Per your instruction: when this surfaces during testing, manually source that specific player's real prior-tournament data and add only the player, not the tournament, to avoid the gap cascading into needing more players' data for a newly-added event.
