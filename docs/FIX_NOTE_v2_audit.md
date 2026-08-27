FIX NOTE — DALXIC Stats Engine v2
From: Auditor / Supervisor review of Sports_Mapping_App_Design.zip
Spec reference: DALXIC_Ratings_Engine_v1.16.md

Status: this build passed a full functional audit (real browser, every tab clicked, math hand-verified against real matches, zero JS errors found anywhere). It's in good shape. The items below are real, specific fixes — not a rewrite.

---

FIX 1 — Incoming rating source selection (§1.6) — REAL DEFECT, please fix

Current behavior: the app selects a player's incoming rating from their "most recent loaded edition" — pure recency, no performance comparison.

Correct rule (§1.6, corrected in spec v1.16): a two-stage filter, not an either/or.

Stage 1 — Recency: find the player's nearest eligible prior year with real data (same tour only). Fallback chain unchanged: nearest prior year → same-season pre-target event if no prior year exists → 0 if nothing exists anywhere.

Stage 2 — Performance: within that single identified year, if the player entered multiple events, pick the single best-performing one (highest rating/margin-share gain). Never sum, never combine, never just take whichever is chronologically last within that year.

Worked example to build against: a player enters a 2026 event having played three tournaments in 2025 (their nearest eligible year). Their incoming source is the highest-gain result among those three 2025 events specifically — not their single most recent 2025 event, and not their best result ever.

Where this likely needs to change: wherever the current "latest edition" lookup happens (the function behind the Ratings H2H page's prior-rating display, and the Backtest's incoming-rating step — both read from the same Patchbay-shared source per §4.1, so this should only need fixing in one place).

---

FIX 2 — Sync Ratings H2H sub-classification note to spec v1.15 (§3.2)

Current behavior: the Prediction page correctly and honestly declines to derive a sub-classification, with a note explaining the ambiguity (score-based thresholds don't apply pre-match). This reasoning was accurate for spec v1.13, and the disclosure discipline here is good — don't lose that pattern.

What's changed since: spec v1.15 resolved this. Sub-classification is now defined as delta-magnitude-based (banding the rating gap already computed for Win/Tie/Loss), with the band thresholds explicitly deferred to calibration evidence from §1.9 — not yet available, so still correctly unbuilt for now. Update the on-page note to reflect the resolved mechanism (delta-based, pending calibration data) rather than the old "spec doesn't define this" framing, which is no longer accurate.

---

FIX 3 — Muting UI: upgrade to the confirmed §1.8 mechanism

Current behavior: per-edition and per-year mute toggles exist individually, matching spec v1.13 (reversible, referenceable — this part is correct and should stay).

What's needed: spec v1.15 adds a specific UI shape on top of this — a per-year checkbox where checking a year mutes it, with the app enforcing that checked years are always contiguous (no gaps between muted and active years). This is what lets an operator pick a calibration starting point (e.g., check 2023–2026, leaving 2022 as the unmuted baseline, so the Backtest runs forward from 2023 using 2022 as its first incoming-rating source).

---

FIX 4 — Build Section 1.9 (Post-Calibration Best-Result Consolidation)

Not built yet, confirmed correctly out of scope until Phase 2 calibration exists to consolidate. No action needed until that phase starts. Noting it here only so it isn't missed when Phase 2 moves forward.

---

Everything else checked in this audit — Phase 1 math, the Flattening Curve's per-edition round-count derivation, Backtest classification, the Vocabulary Guard, the Patchbay page, the Phase Gate status table — is correct and matches spec. No changes needed there.
