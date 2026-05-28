# Workflow: Place a Bet

Full end-to-end workflow for placing a new matched bet: **CALCULATE → VERIFY → LOG**.

Run this when the user wants to place a bet, shares a bookmaker screenshot, or says anything like "new bet", "place a bet", "I want to bet on X".

For each phase, load the corresponding action file — instructions below. Carry all collected fields forward in `workflow_state` — never re-ask for something already provided.

### Progress display

After each phase completes, print a short plain-text progress line in chat:

```
Progress: [✅ Calculate] [🔄 Verify] [⏳ Log]
```

Use ✅ for complete, 🔄 for active, ⏳ for pending, ❌ for blocked.

### Workflow state

Collect fields as you go and carry them forward — never re-ask for something already provided.

```
workflow_state = {
  # From Phase 1 (CALCULATE)
  "bookmaker":   ...,
  "sport":       ...,
  "event":       ...,
  "match_date":  ...,
  "offer_type":  ...,
  "bet_type":    ...,   # Qualifying / Free Bet SNR / Free Bet SR / Money Back
  "back_stake":  ...,
  "back_odds":   ...,
  "lay_odds":    ...,   # may be updated in Phase 2 after exchange screenshot
  "commission":  ...,
  "lay_stake":   ...,   # calculated
  "liability":   ...,   # calculated

  # From Phase 2 (VERIFY)
  "back_selection": ...,
  "lay_selection":  ...,
  "selections_match": True/False,
  "exchange":    ...,

  # From Phase 3 (LOG)
  "row_logged":  ...,
}
```

---

### Phase 1 — CALCULATE

**Goal**: Collect the back bet details, determine bet type, compute lay stake and profit scenarios.

Load and follow `actions/calculate.md`. At the end, show the calculation result and then prompt:

> "✅ Calculation done. Ready to move to Phase 2 — Verify?
> Share your exchange screenshot (or tell me the lay odds if you haven't placed the lay yet)."

Print progress block with Phase 1 ✅, Phase 2 🔄.

---

### Phase 2 — VERIFY

**Goal**: Confirm selections match and re-validate numbers against the exchange screenshot.

Load and follow `actions/verify.md`, but **skip re-asking for anything already in workflow_state** — pre-fill bet type, back stake, back odds from Phase 1. Only new input needed is the exchange screenshot (or confirmation of lay odds).

If selections **don't match**: print progress block with Phase 2 ❌. Tell the user what's wrong and ask them to fix it before continuing. Do not proceed to Phase 3 until selections match.

If selections **match**: show the verified summary, then prompt:

> "✅ Verified. Ready to log this bet to your tracker?"

Print progress block with Phase 1 ✅, Phase 2 ✅, Phase 3 🔄.

---

### Phase 3 — LOG

**Goal**: Write the bet to the tracker.

Load and follow `actions/log.md`. Pre-fill **all fields from workflow_state** — the confirm prompt should require zero new input from the user. Show the pre-filled summary and ask:

> "Shall I log this?"

After the user confirms and the row is written, print the final progress line with all three phases complete and confirm the row number:

```
Progress: [✅ Calculate] [✅ Verify] [✅ Log]
🎉 Bet logged at row X!
```

---
