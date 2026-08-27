DALXIC
THE RATINGS ENGINE
A Formal Record of the Ratings, Validation & Backtest System — Multi-Sport Stats Engine

Author: Porgy
Dalxic — Business Growth Consultant, The Porgys
Document status: Living document — updated as new sections are added. This document is the authoritative specification for the build; it supersedes and replaces any earlier reference document, including the prior BLUEPRINT.md, which is discarded.
Version 1.11 • August 2026

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

---

1.7 Backtest Round Flow and System Update

The Backtest processes one round at a time, in order. As each round completes, it collapses, and the next round processes above it — preserving the full round-by-round order visibly, up through the final.

Once the final round is processed, a single action — "Update System" — applies the accumulated calibration results from the full backtest run into the live system. This click is the calibration commit itself; there is no separate step between finishing the backtest and the system being updated by it.

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

2.5 Export Location Memory

Any export action asks for a save location the first time. The app remembers that location afterward, so repeated exports go to the chosen destination rather than scattering to arbitrary default locations.

---

Section 3: Prediction

3.1 The Master Page

A single picker/selection hub. All selections — tournament, players, and so on — are made here first, before moving to any per-system page.

The Master Page also serves as a summary dashboard: it shows the summarised state of every compute system currently in use, and is the tool used to analyse upcoming games.

Update-timing rule: the app must be refreshed with a player's real result (the System Update loop, Section 2.1) before that same player's next match is analysed on this page. Analysing an upcoming game against a stale rating — one that doesn't yet reflect the player's most recent real result — can distort the outcome the system produces. Updating is not a background maintenance task; it is a required step ahead of every new analysis.

3.2 Ratings H2H

The first of what will become several per-compute-system pages, one for each system introduced (Ratings H2H is the only one that exists so far). For the two players selected on the Master Page:

1. Each player's rating prior to the match is shown.
2. The two are matched head-to-head, using the same mechanism as the Backtest (Section 1.6).
3. The page outputs an expected result relative to Player A: Win / Tie / Loss.

If the result is a Win, it is sub-classified: Clear Win / Win / Tight Win (Section 1.6's thresholds).
If the result is a Loss, it is sub-classified the same way, mirrored: Clear Loss / Loss / Tight Loss.

This vocabulary is specific to Prediction and does not apply to the Backtest, and the reverse also holds. The Backtest's own outputs (Correct / Wrong / Tie, with Wrong sub-split into Clear Wrong / Wrong / Flippable Wrong — Section 1.6) are a separate vocabulary for a separate system. "Loss" is never used in Backtest language, and "Wrong" is never used in Prediction language — the two must not be treated as interchangeable terms for the same idea.

A UI Score Estimator is planned for a later phase: a representative scoreline per classification (a Tight Win/Loss, for example, tending to resemble a 2-1 scoreline), scoped separately per compute category as each one is introduced. Not built yet — noted here as a confirmed future addition, not a current feature.

---

Section 4: Engine Architecture

4.1 The Patchbay System

The engine is built as a patchbay: a shared piece of logic — the Flattening Curve (Section 1.4), the Backtest's classification rules (Section 1.6), or any other reusable mechanism — is built once, in one place, and then patched into every section that needs it, rather than being rebuilt separately inside each one.

This is why, for example, the Backtest and Ratings H2H run the exact same Flattening Curve calculation: it exists as a single component, connected into both, not duplicated. New layers of functionality, or entirely new sections, get plugged into this existing structure as the system grows — in an organised, controlled way that doesn't require restructuring what's already built.
