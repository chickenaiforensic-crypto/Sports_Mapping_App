DALXIC
THE RATINGS ENGINE
A Formal Record of the Ratings, Validation & Backtest System — Multi-Sport Stats Engine

Author: Porgy
Dalxic — Business Growth Consultant, The Porgys
Document status: Living document — updated as new sections are added. This document is the authoritative specification for the build; it supersedes and replaces any earlier reference document, including the prior BLUEPRINT.md, which is discarded.
Version 1.21 • August 2026

---

Table of Contents

Preface & Purpose ... 3
Section 1: System Ratings ... 4
  1.1 Score Measurement
  1.2 The Match Budget
  1.3 The Set Normaliser
  1.4 The Flattening Curve
  1.5 Clean Points Accumulation
  1.6 The Backtest System (Flat, Tier-Blind)
  1.7 Backtest Round Flow and System Update
  1.8 Muting — Post-Calibration Data Lock
  1.9 Post-Calibration Best-Result Consolidation
  1.10 Calibration Rollout — System-Generated Data as the Driver
  1.11 Mapping Systems and the Backtest Selector
Section 2: Data Operations ... 5
  2.1 System Update (AI-Assisted Refresh Loop)
  2.2 Data Addon
  2.3 Master Backup
  2.4 The Logging Page (H2H Call Tracking)
  2.5 Export Location Memory
Section 3: Prediction ... 6
  3.1 The Master Page
  3.2 Ratings H2H
Section 4: Engine Architecture ... 7
  4.1 The Patchbay System
*(further sections appended as supplied)*

---

Preface & Purpose

This document is the formal, versioned record of the ratings and validation system built for the multi-sport stats engine. It replaces the earlier working drafts and internal notes with a single clean reference — written so it can be read start to finish without needing the build history behind it.

Each section documents one part of the system on its own terms: what it does, the exact rules it runs on, a worked example, and its role in the wider engine. The document grows one section at a time; nothing already written is changed when a new section is added, unless the creator explicitly revises it.

Document Conventions
- Each part of the system gets its own numbered top-level section.
- Sections follow the same shape where it fits: Concept → Rules → Worked Example → Notes.
- Worked examples use real matches from the data where available; illustrative players ("Player A", "Player B") only where a real fixture isn't the point.

Revision History

| Version | Date | Change |
|---|---|---|
| v1.0 | August 2026 | Initial document — front matter and structure established, ahead of Section 1 (System Ratings). |
| v1.1 | August 2026 | Section 1 written in full: score measurement, the match budget, the short-vs-long balancing step, and the cross-tournament unifier. This is the foundation layer, confirmed without any tier-weight or multiplier system — those are a separate, later layer of the app and are deliberately excluded here. |
| v1.2 | August 2026 | Renamed 1.3 to "The Set Normaliser" and 1.4 to "The Flattening Curve" — locking in the terms actually used for this build (set normaliser = within-match balancing by set count; flattening curve = across-tournament balancing by number of matches / round depth), so terminology stays consistent as this system is reused elsewhere. |
| v1.3 | August 2026 | Added Section 1.5, confirming the Leaderboard's clean points-accumulation model as the system in use. Tier weight, points table, and budget values are all config-driven (adjustable as UI settings), never hardcoded into the calculation logic. |
| v1.4 | August 2026 | This document is now the authoritative blueprint, replacing the old BLUEPRINT.md, which is discarded. Confirmed the Backtest system is a separate model from the Leaderboard: the Backtest's incoming rating is a flat conversion (matches played × 10, the "×10" referenced by the team), with no tier multiplier applied — the Leaderboard's tier weights (×7 GS down to ×1) do not carry over into the Backtest. Also confirmed the "winner rest game" set-margin question is not an open item: score classification is finalized before ratings are ever applied, so there's no recomputation step for it to affect. |
| v1.5 | August 2026 | Decision: the system is unified going forward. Tier weighting (the ×7 GS / ×6-×1 by-tier scale) is removed as a default for the new build. It is not deleted — it stays as a reserved, config-driven option to be switched on only once calibration work produces a real, evidence-based case for a specific tier weighting, replacing the current dummy placeholder values. Until then, the Leaderboard's untweighted margin-share sum ("Raw" in the old app) is the canonical Rating figure, matching the Backtest's flat, tier-blind model. Note: this is a forward decision for the unified build, not a claim about the currently uploaded app file, whose Rating column is confirmed tier-weighted as-is. |
| v1.6 | August 2026 | Audit fixes: (1) corrected Section 1.6, which still described the Leaderboard as actively tier-weighted after 1.5 removed that — now both sections agree tier weighting is reserved/inactive. (2) Wrote the actual approved Clear Win/Win/Tight Win thresholds and per-length table into Section 1.6 directly, instead of pointing to an external, unincluded spec section. (3) Added a naming note to Section 1.5 flagging that this document's "Rating" equals the live app's "Raw" column, not its "Rating" column, now that tier weighting is off by default. (4) Renamed the trailing "Theory 2: Reserved" to "Section 2: Reserved" to match this document's actual section-naming convention. |
| v1.7 | August 2026 | Added Section 2: Data Operations — System Update (the AI-assisted refresh loop, which runs new real match data through the Backtest mechanism itself), Data Addon (tournament/event ingestion with duplicate overwrite/discard handling), Master Backup, the Logging Page (H2H call tracking against real outcomes), and export location memory. Explicitly notes the Backtest's expanded role: it is not only a validation instrument, but the production mechanism for applying new real-world results into the live system. |
| v1.8 | August 2026 | Added Section 1.7 (Backtest round-to-round UI flow and the Update System commit) and Section 3 (Prediction — the Master Page and Ratings H2H page). Locked a critical vocabulary separation: Prediction uses Win / Tie / Loss (with Clear Loss / Loss / Tight Loss sub-classification, mirroring Win's Clear Win / Win / Tight Win), while the Backtest uses Correct / Wrong / Tie exclusively (Section 1.6). "Loss" and "Wrong" are outputs of two different systems and must never be used interchangeably. |
| v1.9 | August 2026 | Expanded Section 3.1: the Master Page also serves as a summary dashboard across every compute system in use, for analysing upcoming games. Added the update-timing rule: the app must be refreshed with a player's real result (Section 2.1's System Update loop) before that same player's next match is analysed — a stale rating carried into a new analysis can distort the outcome the system produces. |
| v1.10 | August 2026 | Correction to Section 1.4: the Flattening Curve's divisor is the total number of rounds in the tournament's fixed format (e.g. 7 for a Grand Slam, R128 through Final) — not, as previously written, the deepest run any player actually had. This runs once per source tournament, before any head-to-head matching, and is confirmed as a shared calculation used by both the Backtest (1.6) and Ratings H2H (3.2). Added the multi-tournament sourcing note: forward-prediction match-ups can draw two players from entirely different prior tournaments, which is precisely why this shared flattening step exists. Added Section 4 (Engine Architecture) documenting the Patchbay System — shared logic built once and patched into every section that needs it, so new layers or sections can be added without restructuring existing work. Established facts elsewhere in the document (Section 1.1's points table, etc.) are unchanged. |
| v1.11 | August 2026 | Phase 1 audit finding, verified directly against source match data: tier name alone is not a reliable way to determine a tournament's round count for the Flattening Curve. M1000 splits by draw size — a 96-draw M1000 edition starts at R128 (7 rounds, same label a Grand Slam uses), while a 56-draw M1000 edition of the same tier starts at R64 (6 rounds). Confirmed directly from Cincinnati 2021's own match records ("round":"R64"), which contradicted an earlier draft table that had wrongly grouped all M1000 editions under a single 7-round figure. Section 1.4 now specifies that round count must be derived from each edition's own first-round label, never assumed from tier name, with the confirmed label-to-count table and the anomaly written in as a standing rule for future tiers, not a one-off fix. |
| v1.12 | August 2026 | Added Section 1.8 (Muting — Post-Calibration Data Lock), previously referenced in the original backtest spec but never given its own section. States the confirmed trigger and purpose: real event data is muted once it has been used to calibrate that event's ratings, since the outcome is already reflected in the rebuilt ratings. Three questions are logged as explicitly open, not guessed at: what a mute permits on re-run, whether it's reversible, and whether it's scoped per edition or per match. No implementation should proceed on these three points until confirmed. |
| v1.13 | August 2026 | Section 1.8's three open questions are now closed and confirmed: muting is reversible (disable, not delete), scoped per edition and per year (not per match), and muted data remains available as reference data for pickers and future systems — only excluded from re-triggering calibration on a re-run. Section 2.4 clarified: log entries summarise a call the same way the Master Page summarises a system, and demo logs may be built to that shape ahead of real logged calls existing. Added an explicit scope note to Section 3: Ratings H2H is the only compute system in scope; Lookup, Leaderboard, Tournament Performance, and Common Opponents are out of scope until Ratings H2H is complete, and are not to be speculatively specified. |
| v1.14 | August 2026 | Logged a confirmed future direction under Section 1.8: a flexible mute-range boundary (not fixed to only the single most recent year) is intended for later, with the explicit requirement that no gaps exist between muted and active years. Deferred, not built now — noted so it isn't lost before its own phase. |
| v1.15 | August 2026 | Build audit surfaced a real spec defect, now fixed: Section 3.2 pointed Prediction's Win/Loss sub-classification at Section 1.6's thresholds, which are score-based and only exist for matches already played — unusable for a pre-match prediction. Corrected: Prediction's sub-classification is delta-magnitude-based, with band thresholds explicitly deferred to calibration evidence from Section 1.9, not guessed. Section 1.6 now states the incoming-rating source selection rule explicitly: best-performing single event, never latest-by-date, never combined — resolving an ambiguity a builder had reasonably filled in differently. Section 1.8's muting mechanism is now confirmed and built-out: a per-year checkbox, contiguous range required, with a full worked example (mute 2023–2026, calibrate from 2023 using 2022 as baseline). Added Section 1.9 (Post-Calibration Best-Result Consolidation): when a player played multiple tournaments in a calibrated year, only their single highest-gain calibration result carries forward as their new rating factor — symmetric with Section 1.6's incoming-source rule. Every calibration record must carry tournament/tier and year attributes for identification. |
| v1.16 | August 2026 | Correction to v1.15: the incoming-rating source rule is not "best-performing OR latest" as an either/or — it's a two-stage filter. Stage 1 narrows to the player's nearest eligible prior year (recency). Stage 2 picks the single best-performing event within that year (performance). A build audit found a real app implementing pure recency ("most recent loaded edition," no performance comparison at all) — this correction closes that gap by stating the actual combined rule precisely, with a worked example, so it can't be filled in as either pure-recency or pure-performance again. |
| v1.17 | August 2026 | Added Section 1.10 (Calibration Rollout): states plainly that real match data exists only to calibrate the engine, not to drive it — the operational plan is a year-by-year calibration run from 2022 forward to the present, using the calibrated output of each year as the incoming source for the next (Section 1.6) and Section 1.9's consolidation between years. Once run, this calibrated data becomes what actually drives Ratings H2H, not raw scores recomputed at prediction time. Added Section 1.11 (Mapping Systems and the Backtest Selector): Ratings is the first of several planned "Mapping Systems," all calibrated through the same Backtest mechanism via the Patchbay; the Backtest UI is to include a system-selector checkbox list, currently holding only Ratings. Added two concept-only, explicitly out-of-scope notes so they aren't lost before their own phase: 1.11.1 Common Opponents (a shared-opponent comparison between two players) and 1.11.2 The Master Call (a future balancing layer across multiple calibrated Mapping Systems' individual calls). |
| v1.18 | August 2026 | Closed a gap flagged in an earlier build audit and never resolved since: Section 1.6 now states the Master Bucket rule (Correct/Wrong/Tie, and the Clear Wrong/Wrong/Flippable Wrong relabelling within Wrong) explicitly, rather than only being referenced elsewhere without ever being defined. Added the Points Redistribution table (¾/½ for Clear Win, ⅓/⅓ for Win, full-swap for Clear Wrong, full-replace for Wrong, flat-penalty fallback for a zero-rated winner) — this is what the Calibration Rollout (1.10) actually runs on a WRONG call, beyond ordinary accrual. Tight Win and Flippable Wrong are explicitly confirmed to carry no additional correction, by design — only normal margin-share accrual applies to those two classes, since forcing a correction on a genuinely close call isn't justified in either direction. |
| v1.19 | August 2026 | Three additions from a build audit. (1) Section 1.10: the mute checkboxes are confirmed as the real control surface for the rollout's starting point — mute 2023–2026, calibrate 2023 from 2022, let 2023 become the new baseline, shift the mute range forward, repeat through the present year; explicitly a pre-launch data-preparation pass run once by the team, not a live per-visit feature. (2) Section 1.6: added the zero/zero tie redistribution rule — when both players enter at incoming 0, the real match's outcome still carries signal; classify it via the existing loser-points table, take the loser's class average as the midpoint of that range, derive the winner's average as budget minus the loser's average (never assume the two are equal — they come from different ranges and generally aren't), and apply both from a zero base. The two averages summing to the match budget is a build-time sanity check. (3) Section 1.6: a rating of exactly 0 can be genuinely earned through real accumulation, not only assigned as a no-data fallback — both cases are numerically identical and must compare and rank the same (0 beats any negative rating); they differ only in the source label shown, never in numeric treatment. |
| v1.20 | August 2026 | Added a no-data flagging requirement to Section 1.10: any player hitting the no-data fallback during a rollout must be explicitly logged (name, tour, year), not silently defaulted and passed over — this is the operational trigger for manual backfill, not a cosmetic log. Confirmed via direct data check: at a 2024 calibration start with 2021–2023 as lookback, 79 of 452 active players (≈17.5%) still hit this fallback even with full available history — the flagging requirement exists because this number is real and non-trivial, not a rare edge case. |
| v1.21 | August 2026 | Superseded the manual-research backfill approach for no-data players with a self-correcting mechanism: a zero-data player's rating builds from their own real performance as they play (Sections 1.1–1.5), not from external backfill. Introduced the DANGER flag (Section 1.6) — any player with fewer than 5 real matches on record is visibly marked wherever their rating is used: in the Calibration Rollout's flagging (1.10, renamed from a neutral log to DANGER status) and directly on Ratings H2H (3.2) whenever either side carries it. At 5 real matches, the flag drops and the player is treated normally, no ongoing distinction. |

---

Section 1: System Ratings

1.1 Score Measurement

Every set is first turned into a points value: the more games a player wins in a set, the more points it's worth.

| Games won in the set | Points awarded |
|---|---|
| 0 | 0 |
| 1–2 | 2 |
| 3–4 | 4 |
| 5 | 7 |
| 6 | 10 |

Sets that run past six games are read back onto that same 0–6 scale first:

| Actual score | Read as |
|---|---|
| 7-5 | 6-4 |
| 7-6 | 6-5 |
| Any further tiebreak-style finish (8-7 up to 13-12) | 6-5 |

1.2 The Match Budget

Every match is given a points ceiling based on how many sets were actually completed — ten points of capacity per set, no matter the format:

| Sets completed | Points ceiling |
|---|---|
| 2 | 20 |
| 3 | 30 |
| 4 | 40 |
| 5 | 50 |

1.3 The Set Normaliser

A player's raw points margin for a match — the winner's points minus the loser's — isn't compared directly between matches, because a longer match naturally offers more chances to earn points than a shorter one. To put every match on the same footing, the margin is expressed as a share of that match's own budget:

margin share = (points won − points conceded) ÷ match budget

A clean two-set sweep, 6-0 6-0: margin 20, budget 20 → 1.0.
A grinding three-set win, say 6-4 3-6 6-4: margin 6, budget 30 → 0.2.

Both now sit on the same scale regardless of how many sets it took. A short, dominant win isn't penalized for having fewer sets to accumulate points in, and a long match doesn't get an unearned size advantage either. This is the number a player's performance in a single match is actually measured by.

1.4 The Flattening Curve

Before two players' incoming ratings can be compared head-to-head — in the Backtest (Section 1.6) or in Ratings H2H (Section 3.2) — their source-tournament results need to be put on the same scale. Tournaments differ in depth: a Grand Slam runs seven rounds (R128 → R64 → R32 → R16 → QF → SF → Final), a smaller event fewer.

The divisor is the total number of rounds in that tournament's format — fixed by the event's structure, not by how far any individual player actually got.

Flattened result = accumulated result for the edition ÷ total rounds in that tournament's format

For a Grand Slam, that divisor is always 7 — the same for every player in that edition, whether they lost in the first round or won the final.

This step runs once per source tournament, before any head-to-head matching happens — it prepares an incoming rating, it is not a per-match calculation.

Shared mechanism: the Flattening Curve is the same calculation used by both the Backtest (Section 1.6) and Ratings H2H (Section 3.2) — one implementation, patched into both, not two separate versions of the same idea (see Section 4.1, The Patchbay System).

Multi-tournament sourcing: when forward-predicting with Ratings H2H, the players being compared are not limited to a single shared tournament. A player entering a new event (for example, a 2027 WTA event) can carry an incoming rating sourced from any of several different prior tournaments — tournaments the two matched players may never have both appeared in. The Flattening Curve is what makes ratings sourced from tournaments of different depths comparable to each other in that scenario, so an incoming rating from a 7-round Grand Slam run and one from a 4-round smaller event can be placed on the same scale before the match-up is calculated.

How the round count is supplied: not a manual figure entered per tournament, and not a single fixed number per tier name — confirmed against the actual data that tier alone is not reliable. The round count is derived by reading each edition's own first-round label and counting down to the Final.

Confirmed round counts by first-round label:

| First round label | Path to Final | Round count |
|---|---|---|
| R128 | R128→R64→R32→R16→QF→SF→F | 7 |
| R64 | R64→R32→R16→QF→SF→F | 6 |
| R32 | R32→R16→QF→SF→F | 5 |

Confirmed anomaly — M1000 is not uniform: a 96-draw M1000 edition (e.g. Cincinnati/Shanghai at 96 players) starts at R128 — the same label a true 128-player Grand Slam uses — giving it a round count of 7. A 56-draw M1000 edition of the same tier (e.g. Cincinnati 2021–2024) starts at R64 instead, giving a round count of 6. Both are M1000, but they are not the same round count. This was caught directly in source match data (Cincinnati 2021's own match records read "round":"R64"), after an initial audit incorrectly assumed all M1000 editions shared one round count based on tier name alone. The lesson stands as a standing rule, not a one-off fix: tier name is not a reliable proxy for round count; the edition's actual first-round label is the only trustworthy source.

Also confirmed: the R128 label is reused for a 96-player field, not only a true 128-player field. A reader should not assume "R128" always means a literal 128-player draw — it identifies a stage of the format, and 96-draw M1000 editions are stamped into that same first stage.

1.5 Clean Points Accumulation

Rating is a clean, additive sum — each match is scored on its own and added to the player's running total. There is no dependency between matches: no opponent-strength adjustment, no carry-forward, no chronological order to it.

Per match:

Rating contribution = margin share (Section 1.3)

Rating = the sum of every match's contribution, for the selected tournament and year.

This is deliberately tier-blind, unifying it with the Backtest system (Section 1.6): a Grand Slam match and an ATP250 match contribute on the same scale. Tournament tier is not factored into Rating by default.

A tier-weighting layer exists in the underlying config as a reserved, switchable option — not deleted, not active. It is held in reserve for a later calibration phase: once real evidence identifies a specific, justified tier weighting, it can be turned on as a configuration change, without altering the calculation logic itself. Until that evidence exists, no tier multiplier is applied, and the currently-defined placeholder tier values (a simple 1–7 dummy scale) are not part of the live formula.

Nothing about the points table, match budget, or any reserved tier-weight scale is hardcoded; all of it lives in configuration so the system can be tuned from the UI once calibration work supports it.

Naming note: in the live app, the leaderboard displays two columns — a tier-weighted "Rating" and an untweighted "Raw." The Rating defined in this section is the tier-blind figure — equivalent to the app's Raw column, not its Rating column — now that tier weighting is inactive by default. This section's terminology governs the unified build going forward.

---

1.6 The Backtest System (Flat, Tier-Blind)

The Backtest runs on the same unified, tier-blind principle as the Leaderboard (Section 1.5), using its own equivalent conversion.

Incoming rating = matches played (in the selected source event) × 10

1 match = 10, 2 = 20, 3 = 30, 4 = 40, 5 = 50 — a flat linear conversion, the same 10-points-per-unit rate used throughout this system (Section 1.1's per-set scoring, Section 1.2's match budget). This is not a tier multiplier: it is applied identically to every player regardless of the tournament tier their source event came from. A Grand Slam quarterfinal run and an ATP250 quarterfinal run produce the same incoming rating.

Source event selection, corrected — this is a two-stage filter, not an either/or between "most recent" and "best-performing":

Stage 1 — Recency: identify the player's nearest eligible prior year with real data (same tour only), using the fallback chain: nearest prior year with data → same-season pre-target event if no prior year exists → 0 if nothing exists anywhere in the dataset.

Stage 2 — Performance: within that single identified year, if the player entered multiple events, the single best-performing one (highest rating gain) is used as the source — never summed, never combined, and never just whichever of that year's events happens to be chronologically last.

Worked example: a player enters a 2026 event. They played three tournaments in 2025 (their nearest eligible prior year). Their incoming rating source is not simply their most recent 2025 event by date, and not their single best event across their whole career — it is the highest-performing of those three specific 2025 events. Recency narrows the pool to one year; performance picks the result within it.

This is the same two-stage shape used later for post-calibration consolidation (Section 1.9), which operates the same way within a single already-identified year: pick the single best result, never combine.

Confirmed explicitly: no tier weighting applies to the Backtest. Tournament-tier weighting (a reserved ×7-down-to-×1 scale — see Section 1.5) is inactive by default across both the Leaderboard and the Backtest; it plays no role in any Correct/Tie/Wrong classification, sub-bucket, or points-redistribution result until a future calibration phase activates it.

Score classification (Clear Win / Win / Tight Win) is computed once, from the real score, before ratings are ever applied. It is not recomputed at any later stage, so there is no ambiguity to resolve about set-margin edge cases affecting it after the fact.

The classification is based on the loser's points as a fraction of the match budget (Section 1.2):

| Loser's share of budget | Class |
|---|---|
| ≤ 50% | Clear Win |
| 51–75% | Win |
| ≥ 76% | Tight Win |

Worked at each match length:

| Sets | Budget | Clear Win | Win | Tight Win |
|---|---|---|---|---|
| 2 | 20 | 0–10 | 11–15 | 16–19 |
| 3 | 30 | 0–15 | 16–22 | 23–29 |
| 4 | 40 | 0–20 | 21–30 | 31–39 |
| 5 | 50 | 0–25 | 26–37 | 38–49 |

Master Bucket — Correct / Wrong / Tie: per match, the higher incoming rating is treated as the system's pre-match pick. If that pick matches the real winner, the match is Correct. If it doesn't, the match is Wrong. If the two incoming ratings are equal, no pick was possible and the match is a Tie. Within Correct, matches are sub-classified using the table above (Clear Win / Win / Tight Win). Within Wrong, the same real-score class is relabelled to describe the miss: a Clear Win-shaped result the system missed is Clear Wrong, a Win-shaped result is Wrong, and a Tight Win-shaped result is Flippable Wrong — a genuinely close call the system could reasonably have called either way.

Points Redistribution: applied per match, using each player's pre-match incoming points.

| Bucket | Winner's new points | Loser's new points |
|---|---|---|
| Clear Win | winner_old + (¾ × loser_old) | loser_old − (½ × winner_old) |
| Win | winner_old + (⅓ × loser_old) | loser_old − (⅓ × winner_old) |
| Clear Wrong | winner_old + loser_old | loser_old − winner_old |
| Wrong | loser_old (winner's points fully replaced) | loser_old − winner_old |
| Zero-rated winner (winner_old = 0, any bucket) | loser_old (winner inherits loser's old points in full) | loser_old − 10 (flat penalty — 0 can't be subtracted meaningfully) |

Tight Win and Flippable Wrong are deliberately absent from this table — this is not an omission to fill in, it's the intended design. These are the genuinely close calls: the system doesn't get pushed harder toward a side it was already unsure about. These two classes accrue only through the normal margin-share process (Section 1.5) that runs for every match regardless of bucket — no additional correction is layered on top. Clear Win/Win reinforce a pick that was right; Clear Wrong/Wrong sharply correct a pick that was confidently wrong; Tight Win/Flippable Wrong are left to ordinary accrual because forcing a correction on a genuinely uncertain call isn't justified either direction.

Zero/zero tie — a special case within the Master Bucket's Tie outcome, not a fifth redistribution row. When both players enter a match with incoming points of exactly 0, no directional pick was possible going in (a genuine Tie per Section 1.6), but the real match still happened and still produced a real result — that result carries real signal and should not be discarded as "no adjustment."

The rule: classify the real match by the loser's real points against the existing table above (e.g. a 3-set match, budget 30, loser scoring in the 16–22 range, is a Win). Take the midpoint of that range as the loser's average for the class — 19, in this example. The winner's average is not separately tabulated and must not be assumed equal to the loser's — it is the remainder of the budget: budget − loser's average (30 − 19 = 11 here). Apply both from a zero base: winner's new rating = 0 + winner's average (+11); loser's new rating = 0 − loser's average (−19). The two averages should always sum to the match's budget (11 + 19 = 30) — a useful build-time sanity check, since an implementation that mirrors one value onto the other (making both ±19, for instance) has confused the two distinct ranges and will fail this check on any class other than an exact 50/50 split.

Zero as an earned rating, not only a fallback sentinel — a rating of exactly 0 can arise two ways: a player with no prior data anywhere (Section 1.6's fallback), or a player whose real accumulated data happens to compute to exactly 0. Both are numerically just "0," and both must be treated identically in every comparison — 0 is a valid rating that is genuinely higher than any negative rating, never a placeholder meaning "no data" that gets excluded from comparison. The two cases differ only in what source is shown to the user (no prior data available, vs. the specific tournament/year whose real computation landed on exactly 0) — never in how the number itself is compared or ranked.

DANGER status and the 5-match graduation threshold — a player with no established prior data is not held at a flat, unchanging rating waiting on external backfill. Once they start playing real tracked matches, their rating builds from their own real performance via the same points and margin-share mechanism used for every player (Sections 1.1–1.5) — their default rating is earned from what they actually do, not assigned in advance.

Until a player has accumulated 5 real matches on record, they carry a DANGER flag — a visible marker distinct from a normal rating, since a handful of matches or fewer isn't yet a reliable sample to predict from or calibrate confidently against. This flag surfaces everywhere the player's rating is used:

- In the Calibration Rollout's no-data flagging (Section 1.10) — a DANGER-flagged player hitting the calibration path is logged the same way the no-data fallback already requires, using DANGER as the visible status, not a neutral log line.
- On the Ratings H2H prediction page (Section 3.2) directly — if either player carries a DANGER flag, this must be surfaced visibly alongside the prediction itself, not silently computed as if both sides were equally reliable.

Once a player's real match count reaches 5, the DANGER flag is dropped and their rating is used normally from that point forward, with no special treatment or ongoing distinction from an established player.

---

1.7 Backtest Round Flow and System Update

The Backtest processes one round at a time, in order. As each round completes, it collapses, and the next round processes above it — preserving the full round-by-round order visibly, up through the final.

Once the final round is processed, a single action — "Update System" — applies the accumulated calibration results from the full backtest run into the live system. This click is the calibration commit itself; there is no separate step between finishing the backtest and the system being updated by it.

---

1.8 Muting — Post-Calibration Data Lock

The data's entire purpose is to give the app foresight for a next tournament — the "sight" that lets the system say Player A has a Clear Win, for example. This happens through the Backtest, round by round: real data is used to check whether the current compute system (Ratings, for now — the only one being built) called Win / Tie / Wrong (Loss), and the adjustment protocol then awards points/ratings accordingly.

Once that calibration has run for a tournament, the raw real data has done its job — the ratings it produced now call every prediction going forward. At that point, the data is muted.

Muting confirmed:

- **Reversible.** Muting disables data from active use, it does not delete it. It can be unmuted.
- **Granularity: per edition and per year.** A tournament's data can be muted as a whole; a full year can be muted as a whole. Muting is not scoped to individual matches.
- **Muted data is not gone — it becomes a reference system.** Additional compute systems, beyond Ratings, will be added later, and they will reference this same historical data. Muting removes data from active recalibration, not from the system entirely: player names, tournament names, and year selectors continue to draw on it for pickers and selectors regardless of mute state, and future systems can reference it once built.
- **Re-run behavior:** a re-run of the Backtest does not recalibrate against muted data — it has already served its calibration purpose. It remains available as reference data for everything else (pickers, names, future systems), just excluded from re-triggering the calibration/adjustment step again.

This section is now closed as a confirmed spec item — no longer an open question.

UI mechanism, now confirmed and no longer deferred: muting is a per-year checkbox. Checking a year mutes it. This directly enables selecting a calibration starting point: to calibrate from a given year forward, every year from the most recent back to that starting point is checked (muted), leaving one earlier year unmuted as the baseline the Backtest reads incoming ratings from.

The checked years must always be contiguous — no gaps between a muted year and the active years around it. Muting 2026, 2025, 2024, and 2023 while leaving 2022 unmuted is valid; muting 2026 and 2024 while leaving 2025 unmuted is not.

Worked example: 2022 is left unmuted as the baseline. 2023 through 2026 are all checked (muted). The Backtest then begins its first round using 2022's data as the incoming rating source, and produces its first calibrated result for 2023. Each subsequent year processes the same way in order, using the previous year's now-calibrated ratings as its own incoming source, through to 2026.

This directly supports data-quality handling: if a specific year's data is suspected to have issues, the calibration start point can be moved forward (muting further back) or held closer to the present, without leaving a gap in the muted range.

---

1.9 Post-Calibration Best-Result Consolidation

Once a full year has been completely calibrated across every tournament in the system, players who played more than one tournament that year need a single, consolidated rating going forward — not one figure per tournament they entered.

The rule: for a player who played multiple tournaments in the calibrated year, only their single best-performing calibration — the one tournament where they gained the most rating — is kept as their new rating factor for that year. The others are not summed, averaged, or otherwise combined; they are set aside once the best one is identified.

This is the same principle already used for incoming-rating source selection (Section 1.6): one representative result chosen out of several, never a combination of them. The two rules are symmetric — best-performing in, best-performing out.

Worked example: Player A plays five tournaments in the year being calibrated, including Cincinnati and Wimbledon among them. Each of the five is calibrated independently, in tournament order, as part of that year's Backtest run. Once all five are done, Player A's five calibration results are compared, and only the single highest-gain result becomes their new rating factor for the year — say, Wimbledon, if that produced the largest gain. The other four calibrated results are not discarded from the record (Section 1.8's muting keeps the underlying data, reversible and referenceable) — they simply aren't the figure that carries forward.

Every calibration record must carry clear identifying attributes — at minimum tournament name/type (tier) and year — so that when multiple calibration results exist for one player in one year, which tournament produced which result is never ambiguous, and the best-performing selection can be traced back to a specific, identifiable event.

---

1.10 Calibration Rollout — System-Generated Data as the Driver

The underlying purpose of the Backtest is now stated plainly: real match data exists to calibrate the engine, not to drive it directly. Once a year has been calibrated, the app should run on the ratings that calibration produced — the system's own generated data — rather than continuing to reference raw player results going forward. Real data does its job once, at calibration time, then steps back (Section 1.8).

The operational rollout: calibration runs year by year, starting from 2022, forward through to the present. Each year's Backtest pass uses the previous year's already-calibrated ratings as its incoming source (Section 1.6), calibrates every tournament in the current year against real results, and Section 1.9 consolidates each player down to their single best result for that year before the next year's pass begins. This repeats, one year at a time, until the rollout reaches the current year.

Once this rollout has run, the year-by-year calibrated ratings it produced become the actual processing data behind Ratings H2H (Section 3.2) — this is what "drives the app." A prediction between two players is read from calibrated, system-generated ratings, not recomputed from raw scores at prediction time.

The mute checkboxes (Section 1.8) are the control surface for this rollout's starting point, not a decorative UI: checking 2023 through 2026 and leaving 2022 unmuted means the rollout calibrates 2023 using 2022 as its incoming source. Once that calibration completes, 2023's calibrated output becomes the new baseline — the mute range then shifts forward (2024 through 2026 muted) and the same pass repeats for 2024 using 2023's now-calibrated output, and so on through to the present year.

This is a pre-launch data-preparation process, run once by the team ahead of the app going into real use — not a live, end-user-facing feature that re-runs on every visit. By the time the app is in active use, the rollout has already been executed and its calibrated store is simply what Ratings H2H reads from.

No-data flagging: every time the rollout (or any Mapping System calibration built on the same mechanism) hits a player through the no-data fallback (Section 1.6) rather than real prior data, that player must be explicitly flagged — logged with the player's name, tour, and the year they were being calibrated for — not silently defaulted to a rating of 0 and passed over. The point of the flag is operational: it is the trigger for the manual backfill process (source that specific player's real prior-tournament data and add only the player, not a new tournament, to the store), not a cosmetic log entry. A rollout run that produced no-data fallbacks and didn't surface a reviewable list of exactly who and when has not met this requirement.

1.11 Mapping Systems and the Backtest Selector

Ratings is the first of what will become several compute systems, referred to collectively as Mapping Systems. Every Mapping System is calibrated the same way — through the Backtest — using the same shared mechanism (the Patchbay, Section 4.1), with only the scoring context changed per system. Ratings is the backbone: it is being built and calibrated fully before any other Mapping System is added, per the scope rule already stated in Section 3.

The Backtest's round-replay UI (Section 1.7) is to include a system selector — a checkbox list of every Mapping System available for calibration. At present this list has exactly one entry, Ratings, and only Ratings runs. As future Mapping Systems are built (see 1.11.1 below for one already described), they are added to this same selector, each calibrated independently through the same Backtest flow, without altering how Ratings itself already works.

1.11.1 Common Opponents (concept only, not in scope)

A Mapping System, not yet built, explicitly deferred. Concept: for a prediction between Player A and Player B, identify a common opponent X that both have played. Compare how A performed against X and how B performed against X, and use that comparison to assign a call category. Like Ratings, this would be calibrated through the Backtest and would appear in the Mapping Systems selector once built. No further specification exists yet — this entry exists so the concept isn't lost before its own phase, not as a build instruction.

1.11.2 The Master Call (concept only, not in scope)

Once multiple Mapping Systems exist and are each independently calibrated, each will produce its own call for a given match-up. The Master Call is the eventual balancing layer that combines the individual calls from every calibrated Mapping System into one final adjusted call, rather than relying on any single system alone. Not specified further and not in scope until multiple Mapping Systems actually exist to balance between.

---

Section 2: Data Operations

2.1 System Update (AI-Assisted Refresh Loop)

This is how the app is kept current with newly played matches. It is a human-in-the-loop process, not an automatic feed:

1. A human triggers a "System Update Request" inside the app.
2. The app outputs a request: the last known state for every tournament present — each player's last match date, last known score, and related detail — everything needed to identify exactly what data is missing.
3. The human copies that output and pastes it into an external AI.
4. The AI is tasked with sourcing the missing, up-to-date results for those tournaments, and must return the data in the exact order and format the app's system requires.
5. The human copies the AI's response and pastes it back into the app.
6. The app then updates automatically — processing the incoming real match data through the same mechanism as the Backtest (Section 1.6), not a separate or simplified ingestion path.

This is the reason the Backtest is being built with this level of rigor: it is not only a validation instrument for checking ratings against real outcomes. It is also the production mechanism that teaches both the external AI and the app builder the correct, accurate procedure for turning newly played real-world results into updated ratings.

2.2 Data Addon

A dedicated section of the app for adding new tournaments and events into the system. The same section also serves as a reference view, listing every tournament currently loaded — not an add-only form.

Duplicate protection is built in: if an addition matches a tournament already in the system, the app does not silently accept it. It flags the duplicate and prompts the human to choose one of two actions — overwrite the existing entry, or discard the incoming one.

2.3 Master Backup

A single export that captures the entire data system in the app — the full dataset, not a partial or per-tournament slice. Its purpose is backup: if something needs to be restored, this export can be re-uploaded back into the app.

2.4 The Logging Page (H2H Call Tracking)

Before a match is played, a human can opt to log a call — picking two players head-to-head ahead of the real result. Once the real match completes, the app allows the logged call to be compared directly against the actual outcome.

The page is empty by design until a fixture passes its precondition (a real call has actually been logged) — it does not populate with placeholder data pretending a call was made. Ahead of real logged calls existing, demo logs can be built to the same shape: a logged entry stores a summary of the call, formatted the same way the Master Page summarises a system.

2.5 Export Location Memory

Any export action asks for a save location the first time. The app remembers that location afterward, so repeated exports go to the chosen destination rather than scattering to arbitrary default locations.

---

Section 3: Prediction

Scope note: Ratings H2H is the only compute system being built, documented, or phased right now. Other systems referenced in earlier builds or discussions (Lookup, Leaderboard, Tournament Performance, Common Opponents — see Section 1.11.1 for a concept-level note, Master Call — see Section 1.11.2) are explicitly out of scope until Ratings H2H is complete and working correctly. They are not specified in this document beyond the concept-level notes already given, and no build work should start on them ahead of that.

3.1 The Master Page

A single picker/selection hub. All selections — tournament, players, and so on — are made here first, before moving to any per-system page.

The Master Page also serves as a summary dashboard: it shows the summarised state of every compute system currently in use, and is the tool used to analyse upcoming games.

Update-timing rule: the app must be refreshed with a player's real result (the System Update loop, Section 2.1) before that same player's next match is analysed on this page. Analysing an upcoming game against a stale rating — one that doesn't yet reflect the player's most recent real result — can distort the outcome the system produces. Updating is not a background maintenance task; it is a required step ahead of every new analysis.

3.2 Ratings H2H

The first of what will become several per-compute-system pages, one for each system introduced (Ratings H2H is the only one that exists so far). For the two players selected on the Master Page:

1. Each player's rating prior to the match is shown.
2. The two are matched head-to-head, using the same mechanism as the Backtest (Section 1.6).
3. The page outputs an expected result relative to Player A: Win / Tie / Loss.

If either player carries a DANGER flag (fewer than 5 real matches on record — Section 1.6), this must be surfaced visibly alongside the prediction, not silently computed as if both sides were equally reliable. The prediction is still shown, but the DANGER marker tells the person reading it that at least one side's rating is provisional.

If the result is a Win, it is sub-classified: Clear Win / Win / Tight Win.
If the result is a Loss, it is sub-classified the same way, mirrored: Clear Loss / Loss / Tight Loss.

Important distinction from the Backtest: Section 1.6's Clear Win/Win/Tight Win thresholds are computed from a real score's loser-points-to-budget fraction — they only exist once a match has actually been played. A Ratings H2H prediction has no real score; it's a comparison of two prior ratings ahead of a match that hasn't happened yet. Section 1.6's thresholds cannot be reused here directly, and this document previously implied they could — that was wrong and is corrected now.

Prediction's sub-classification is instead based on the size of the rating gap between the two players (the same delta already computed to determine Win/Tie/Loss), banded into Clear/Mid/Tight ranges. The exact band thresholds are not yet fixed — they are meant to come from the Backtest's own calibration results (Section 1.9): once enough real matches have been calibrated, the correlation between a given rating-gap size and how decisively that gap actually played out becomes the evidence base for setting these bands, rather than a guessed number. Until that calibration evidence exists, this sub-classification is specified but not yet computable with confirmed thresholds.

This vocabulary is specific to Prediction and does not apply to the Backtest, and the reverse also holds. The Backtest's own outputs (Correct / Wrong / Tie, with Wrong sub-split into Clear Wrong / Wrong / Flippable Wrong — Section 1.6) are a separate vocabulary for a separate system. "Loss" is never used in Backtest language, and "Wrong" is never used in Prediction language — the two must not be treated as interchangeable terms for the same idea.

A UI Score Estimator is planned for a later phase: a representative scoreline per classification (a Tight Win/Loss, for example, tending to resemble a 2-1 scoreline), scoped separately per compute category as each one is introduced. Not built yet — noted here as a confirmed future addition, not a current feature.

---

Section 4: Engine Architecture

4.1 The Patchbay System

The engine is built as a patchbay: a shared piece of logic — the Flattening Curve (Section 1.4), the Backtest's classification rules (Section 1.6), or any other reusable mechanism — is built once, in one place, and then patched into every section that needs it, rather than being rebuilt separately inside each one.

This is why, for example, the Backtest and Ratings H2H run the exact same Flattening Curve calculation: it exists as a single component, connected into both, not duplicated. New layers of functionality, or entirely new sections, get plugged into this existing structure as the system grows — in an organised, controlled way that doesn't require restructuring what's already built.
