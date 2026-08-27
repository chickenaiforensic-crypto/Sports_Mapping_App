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
Spec reference: Section 1.6–1.7

Scope:
- Incoming rating: matches played × 10, flat, tier-blind (1.6). Source event selection: best-performing single event only — never latest-by-date, never combined across events.
- Score classification: Clear Win / Win / Tight Win by loser's fraction of match budget (≤50% / 51–75% / ≥76%), computed once from the real score before ratings are applied — never recomputed later.
- Round flow (1.7): rounds process in order; each completed round collapses as the next processes above it, through the final.
- "Update System" action: a single commit that applies the full backtest run's calibration results into the live system.
- Muting (1.8): a per-year checkbox UI. Checking a year mutes it. Checked years must be contiguous — no gaps. This is how a calibration starting point is chosen: mute every year from most recent back to (not including) the desired baseline year, and the Backtest runs forward from the year after the baseline, using the baseline year's data as the first incoming source. Confirmed: reversible, scoped per edition and per year, and remains available as reference data (pickers, future systems) even while muted — only excluded from re-triggering calibration on a re-run.
- Post-Calibration Best-Result Consolidation (1.9): once a full year is calibrated across every tournament, a player who played multiple tournaments that year keeps only their single highest-gain calibration result as their new rating factor — the others are not summed or averaged. Every calibration record must carry tournament/tier and year attributes so the best-performing one is traceably identifiable.

Audit checkpoint: confirm the classification thresholds, round-flow/collapse behavior, the mute-checkbox contiguity rule, and the best-result consolidation logic against the spec's worked examples before implementation. Confirm explicitly that no tier multiplier is planned anywhere in this phase.

Completion audit must specifically check: Backtest vocabulary stays Correct / Wrong / Tie with Wrong sub-split into Clear Wrong / Wrong / Flippable Wrong — never "Loss," which belongs to Phase 4 only. Confirm muting matches spec Section 1.8 exactly, including that the UI rejects a non-contiguous mute selection. Confirm consolidation (1.9) only ever keeps one result per player per year, never a sum or average, and that the kept result's source tournament is identifiable from its stored attributes.

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
- UI Score Estimator: explicitly out of scope for this phase — noted in the spec as a later addition, per compute category, once more systems exist.

Audit checkpoint: confirm the vocabulary separation before implementation — this page must never output "Wrong," and the Backtest must never output "Loss." Confirm sub-classification is understood as delta-based, not a reuse of Section 1.6's score-fraction table.

Completion audit must specifically check: the update-timing precondition is actually enforced (not just documented). If sub-classification is implemented, confirm its thresholds are sourced from real Section 1.9 calibration output, not an invented placeholder; if calibration evidence doesn't exist yet, confirm the build correctly ships bare Win/Tie/Loss rather than a guessed sub-classification.

---

PHASE 5+ — Reserved
Additional compute systems (beyond Ratings H2H) and the UI Score Estimator are added one at a time, each as its own future phase, following this same three-step working system. Phases are not pre-numbered here since scope and order will be set when each is introduced. Confirmed explicitly: no work, specification, or inquiry on these should happen until Ratings H2H (Phases 1–4) is complete and working correctly.

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
