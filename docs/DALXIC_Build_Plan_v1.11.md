DALXIC — BUILD PLAN
Multi-Sport Stats Engine — Phased Build for the Builder

Source of truth: DALXIC_Ratings_Engine.md (current version at time of build). This plan sequences that specification into buildable phases. If any phase's requirement is unclear or missing detail, stop and ask — do not infer or fill gaps.

Working system for every phase, no exceptions:
1. AUDIT — before writing any code, the builder reviews the relevant section(s) of the specification and states back, in writing, exactly what will be built and how it maps to the spec. No code is written during this step.
2. IMPLEMENTATION — gated. Implementation does not start until the Audit step is explicitly approved ("proceed") by the project owner. Once approved, the builder implements only what was audited — no unrequested extras, no silent scope changes.
3. COMPLETION AUDIT — before a phase is declared done, the builder checks the finished work against the spec for: accuracy, errors, gaps, omissions, and falsehoods (any claim, label, or output that doesn't match what the spec actually defines). Findings are reported plainly, including anything wrong — this step is not a formality.

No phase begins until the prior phase has passed its Completion Audit and been signed off.

---

PHASE 1 — Foundation: System Ratings
Spec reference: Section 1.1–1.5

Scope:
- Score Measurement (1.1): games-to-points table, and the 7-5/7-6/13-12 reduction rules.
- The Match Budget (1.2): 10 points per completed set.
- The Set Normaliser (1.3): margin-share formula (margin ÷ budget).
- The Flattening Curve (1.4): divide by the total number of rounds in the tournament's fixed format (e.g. 7 for a Grand Slam) — not by any player's actual matches played. Runs once per source tournament, before head-to-head matching. Shared calculation, patched into both the Backtest and Ratings H2H (see Section 4.1, Patchbay System). Round count must be derived from each edition's own first-round label (R128=7, R64=6, R32=5) — never assumed from tier name alone; M1000 splits by draw size (96-draw starts at R128, 56-draw starts at R64) and must be handled per-edition, not per-tier.
- Clean Points Accumulation (1.5): per-match margin share summed per player, tier-blind by default. Tier weighting exists only as a reserved, inactive config value — not to be wired into any live calculation in this phase.

Audit checkpoint: confirm the points table, reduction rules, budget, normaliser, and flattening curve produce the worked examples in the spec exactly, before touching accumulation logic.

Completion audit must specifically check: no tier multiplier has leaked into any Rating output; "Rating" in this build equals the spec's tier-blind figure, not any tier-weighted figure.

---

PHASE 2 — Backtest System
Spec reference: Section 1.6–1.11

Scope:
- Incoming rating: matches played × 10, flat, tier-blind (1.6). Source event selection: two-stage filter — nearest eligible prior year first, then best-performing event within that year. Never pure-latest, never pure-best-ever, never combined.
- Score classification: Clear Win / Win / Tight Win by loser's fraction of match budget (≤50% / 51–75% / ≥76%), computed once from the real score before ratings are applied — never recomputed later.
- Master Bucket (1.6): Correct / Wrong / Tie, from comparing the higher incoming rating's pick against the real winner. Within Wrong, relabel by real-score class: Clear Wrong / Wrong / Flippable Wrong.
- Points Redistribution (1.6): the actual calibration math. Clear Win → winner +¾ loser's points, winner −½ own points. Win → winner +⅓ loser's, winner −⅓ own. Clear Wrong → full point swap. Wrong → winner's points fully replaced by loser's old points, loser docked by winner's old points. Zero-rated winner → flat-penalty fallback (loser inherits in full, loser_old − 10). Tight Win and Flippable Wrong get no redistribution beyond ordinary accrual — this is by design, not a gap to fill.
- Round flow (1.7): rounds process in order; each completed round collapses as the next processes above it, through the final.
- "Update System" action: a single commit that applies the full backtest run's calibration results into the live system.
- Muting (1.8): a per-year checkbox UI. Checking a year mutes it and enforces contiguity by construction (muting a year mutes it and everything more recent; unmuting a year unmutes it and everything older) — the UI should make a non-contiguous state structurally impossible, not just validate against one. Confirmed: reversible, scoped per edition and per year, and remains available as reference data (pickers, future systems) even while muted — only excluded from re-triggering calibration on a re-run.
- Post-Calibration Best-Result Consolidation (1.9): once a full year is calibrated across every tournament, a player who played multiple tournaments that year keeps only their single highest-gain calibration result as their new rating factor — the others are not summed or averaged. Every calibration record must carry tournament/tier and year attributes so the best-performing one is traceably identifiable.
- Calibration Rollout (1.10): the actual deliverable for this phase is not a single-edition demo — it's a year-by-year calibration run, controlled by the mute checkboxes (not hardcoded to 2022), each year using the prior year's calibrated output as its incoming source. Confirmed operational start point: 2024, with 2021–2023 as source/lookback data. This is a pre-launch data-preparation pass run once by the team, not a live per-visit feature. No-data fallback hits (§1.6) must be explicitly flagged during the run as DANGER status — name, tour, year — not silently defaulted and passed over.
- DANGER flag / UNTESTED status / 5-match graduation (1.6): entirely a Calibration Rollout mechanism — never computed by Ratings H2H, which only reads the result. UNTESTED (no rating exists) is distinct from a rating of 0 (a real, comparable value). While untested, losses cause no rating change at all — only a first win establishes a baseline, via the standard Win-category formula against the opponent's old rating. An established player beating an untested opponent gets a halved addon (⅜/⅙ of their own old rating for Clear Win/Win, no opponent rating to draw from). Once rated (post first-win), a player carries DANGER until 5 real matches accumulate, surfaced on both the rollout's flagging and Ratings H2H. This rollout's output is what Phase 4 reads from; Phase 4 cannot function correctly until this has actually been run.
- Mapping Systems Selector (1.11): the round-replay UI needs a checkbox-style system selector for which Mapping System is being calibrated. Build this as a real, extensible list — even though it currently holds exactly one entry, Ratings. Do not hardcode assumptions that only one system will ever exist.

Audit checkpoint: confirm the classification thresholds, round-flow/collapse behavior, the mute-checkbox contiguity-by-construction rule, the best-result consolidation logic, and the full rollout plan (2022→present) against the spec's worked examples before implementation. Confirm explicitly that no tier multiplier is planned anywhere in this phase.

Completion audit must specifically check: Backtest vocabulary stays Correct / Wrong / Tie with Wrong sub-split into Clear Wrong / Wrong / Flippable Wrong — never "Loss," which belongs to Phase 4 only. Confirm the Points Redistribution table is applied exactly as specified per bucket, that Tight Win/Flippable Wrong genuinely receive no extra correction beyond accrual, and that the rollout (1.10) is actually running this redistribution — not merely displaying a Correct/Wrong label with no rating consequence. Confirm muting matches spec Section 1.8 exactly, including that a non-contiguous mute state cannot be reached at all, not merely rejected after the fact. Confirm consolidation (1.9) only ever keeps one result per player per year, never a sum or average, and that the kept result's source tournament is identifiable from its stored attributes. Confirm the rollout has actually been executed end to end (2022 through present), not merely built as a capability. Confirm the Mapping Systems selector is a real list structure, not a single-system assumption with a checkbox bolted on.

---

PHASE 3 — Data Operations
Spec reference: Section 2.1–2.5

Scope:
- System Update (2.1): the AI-assisted refresh loop — app outputs last-known state per tournament, human relays to an external AI, human pastes the AI's response back in, app processes it through the Backtest mechanism (Phase 2), then the human triggers Update System.
- Data Addon (2.2): add/list tournaments; duplicate detection prompts overwrite-or-discard, never silent acceptance.
- Master Backup (2.3): single full-system export/re-upload.
- Logging Page (2.4): log an H2H call ahead of a real match; compare logged call to actual result once played.
- Export Location Memory (2.5): ask once, remember thereafter.

Audit checkpoint: confirm the System Update loop routes through the same Backtest mechanism built in Phase 2, rather than a separate or simplified path, before implementation begins.

Completion audit must specifically check: duplicate tournament entries are never silently accepted; every export respects a remembered location after the first use.

---

PHASE 4 — Prediction
Spec reference: Section 3.1–3.2

Scope:
- Master Page (3.1): selection hub and summary dashboard across every compute system in use.
- Update-timing rule: a player's real result must be applied (Phase 3's System Update) before that player's next match is analysed here — enforce this as a hard precondition, not a suggestion.
- Ratings H2H (3.2): shows each player's prior rating, runs the same head-to-head mechanism as the Backtest, outputs Win / Tie / Loss relative to Player A, with Win sub-split into Clear Win / Win / Tight Win and Loss mirrored into Clear Loss / Loss / Tight Loss. Sub-classification is delta-magnitude-based (the rating-gap size), not Section 1.6's score-based thresholds — those don't apply pre-match. Band thresholds are not yet fixed; they depend on calibration evidence from Phase 2's Section 1.9 work. Until that evidence exists, ship Win/Tie/Loss without sub-classification rather than guessing at band cutoffs — do not invent placeholder thresholds.
- Data dependency: this phase reads from Phase 2's calibration rollout (Section 1.10) — the actual year-by-year calibrated ratings from 2022 forward. It does not read raw scores or recompute ratings at prediction time. This phase cannot be meaningfully tested or completed until Phase 2's rollout has actually been run end to end.
- DANGER flag display: if either selected player has fewer than 5 real matches on record (§1.6), this page must show that visibly alongside the prediction — never compute and display a call as if both sides were equally reliable.
- UNTESTED handling: if either selected player has no established rating at all (§1.6) — a distinct, stronger condition than DANGER — this page must not compute or display a prediction call. State plainly that it cannot compute, rather than fabricate a result from a number that doesn't exist.
- UI Score Estimator: explicitly out of scope for this phase — noted in the spec as a later addition, per compute category, once more systems exist.

Audit checkpoint: confirm the vocabulary separation before implementation — this page must never output "Wrong," and the Backtest must never output "Loss." Confirm sub-classification is understood as delta-based, not a reuse of Section 1.6's score-fraction table. Confirm the data source is the Phase 2 rollout's calibrated output, not raw scores.

Completion audit must specifically check: the update-timing precondition is actually enforced (not just documented). If sub-classification is implemented, confirm its thresholds are sourced from real Section 1.9 calibration output, not an invented placeholder; if calibration evidence doesn't exist yet, confirm the build correctly ships bare Win/Tie/Loss rather than a guessed sub-classification. Confirm every prior rating shown traces back to the Phase 2 rollout's output, not a raw-score recomputation.

---

PHASE 5+ — Reserved
Additional Mapping Systems (Section 1.11) — including Common Opponents (1.11.1) and the eventual Master Call balancing layer (1.11.2) — and the UI Score Estimator are added one at a time, each as its own future phase, following this same three-step working system. Phases are not pre-numbered here since scope and order will be set when each is introduced. Confirmed explicitly: no work, specification, or inquiry on these should happen until Ratings H2H (Phases 1–4) is complete and working correctly.

---

Revision History

| Version | Date | Change |
|---|---|---|
| v1.0 | August 2026 | Initial build plan — Phases 1–4 sequenced from DALXIC_Ratings_Engine.md v1.9, each structured in the Audit / Implementation (gated) / Completion Audit system. |
| v1.1 | August 2026 | Corrected Phase 1's Flattening Curve scope to match spec v1.10: divisor is the tournament's fixed total rounds, not a player's actual matches played. Added note that this is a Patchbay-shared component (Section 4.1) used identically by the Backtest and Ratings H2H — build it once, not per-section. |
| v1.2 | August 2026 | Phase 1 Audit reviewed and returned for correction: the submitted round-count table wrongly grouped all M1000 editions under one 7-round figure. Verified against source data that 56-draw M1000 editions start at R64 (6 rounds), while only 96-draw M1000 editions start at R128 (7 rounds) — tier name alone does not determine round count. Phase 1 scope updated to require deriving round count from each edition's own first-round label. Phase 1 Implementation remains gated until the builder resubmits the audit with this correction applied. |
| v1.3 | August 2026 | Added Muting (spec Section 1.8) to Phase 2 scope, flagged as an owner directive with three explicitly open questions (re-run behavior, reversibility, per-edition vs. per-match granularity). Phase 2's Completion Audit now checks that no mute/unmute mechanism has been built ahead of those three answers. |
| v1.4 | August 2026 | Muting's three questions confirmed and closed (spec v1.13): reversible, per-edition/per-year, muted data stays referenceable. Phase 2's Completion Audit updated to check the confirmed behavior directly instead of gating on open questions. Phase 5+ note strengthened: explicitly no inquiry or speculative scoping on additional compute systems until Ratings H2H is complete. |
| v1.5 | August 2026 | Matches spec v1.15. Phase 2: added the confirmed mute-checkbox mechanism (contiguous year ranges) and Section 1.9's Post-Calibration Best-Result Consolidation to scope, both with new completion-audit checks. Also locked the incoming-rating source rule (best-performing, never latest) into Phase 2's scope description. Phase 4: fixed a real spec defect carried over from earlier — sub-classification was pointed at Section 1.6's score-based thresholds, which don't exist pre-match; corrected to delta-magnitude-based classification with thresholds explicitly deferred to Phase 2's calibration evidence, and the completion audit now checks that no placeholder threshold gets invented in the meantime. |
| v1.6 | August 2026 | Matches spec v1.17. Phase 2's actual deliverable redefined: not a single-edition demo, but the full year-by-year Calibration Rollout (1.10) from 2022 through the present, plus a real, extensible Mapping Systems selector (1.11) in the Backtest UI. Source rule corrected to the two-stage filter (nearest year, then best-performing within it) matching spec v1.16. Muting scope tightened: contiguity must be structurally impossible to violate, not just validated. Phase 4 now explicitly depends on Phase 2's rollout having actually run — it reads calibrated output, never raw scores. Phase 5+ names its first two reserved entries: Common Opponents and the Master Call, both concept-only per spec 1.11.1/1.11.2. |
| v1.7 | August 2026 | Matches spec v1.18. A build audit surfaced a real gap that had gone unresolved since an early session: the Points Redistribution formula was referenced but never actually specified. Phase 2 scope now includes the full table (Clear Win ¾/½, Win ⅓/⅓, Clear Wrong full-swap, Wrong full-replace, zero-rated-winner flat penalty) and the Master Bucket definition it depends on. Completion audit updated to check the rollout is actually applying this redistribution — a Correct/Wrong label with no rating consequence is not sufficient. |
| v1.8 | August 2026 | Matches spec v1.20. Calibration start point confirmed: 2024, with 2021–2023 as lookback/source data — not a fixed 2022 start. Added the no-data flagging requirement to Phase 2 scope: every no-data fallback hit during the rollout must be explicitly logged (name, tour, year), not silently defaulted. Confirmed via direct data check this is a real, non-trivial requirement — 79 of 452 players (≈17.5%) still hit the fallback even at a 2024 start with full 2021–2023 lookback. |
| v1.9 | August 2026 | Matches spec v1.21. Superseded manual-research backfill with a self-correcting mechanism: Phase 2's no-data flagging is now explicitly DANGER status (renamed from a neutral log), and Phase 4 gains a new requirement — Ratings H2H must visibly show a DANGER marker if either selected player has fewer than 5 real matches on record. A DANGER-flagged player's rating still builds normally from their own real performance (§1.1–1.5); the flag is a visibility requirement, not a computation block. Also corrected a version-numbering error in this changelog (two prior entries both labeled v1.7, out of chronological order). |
| v1.10 | August 2026 | Matches spec v1.22. Specified the exact DANGER redistribution formula in Phase 2 scope: opponent's old rating as base, ¾/⅓ added for Clear Win/Win, subtracted for Clear Loss/Loss, no correction for Tight Win/Tight Loss beyond normal accrual. |
| v1.11 | August 2026 | Matches spec v1.23. Superseded v1.10's symmetric loss formula — a real failure mode (an early loss to a top-rated opponent could inflate an untested player's rating) is fixed by introducing UNTESTED as a distinct status from a rating of 0. Losses cause no change while untested; only a first win establishes a baseline. Phase 4 gains a stronger requirement: UNTESTED blocks a prediction call entirely, not just a DANGER flag alongside one. |
