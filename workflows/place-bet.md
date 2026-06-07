# Workflow: Place a Bet

Full end-to-end workflow for placing a new matched bet: **EXPLORE → CALCULATE → VERIFY → PLACE BET → LOG**.

Run this when the user wants to place a bet, shares a bookmaker screenshot, or says anything like "new bet", "place a bet", "I want to bet on X".

For each phase, load the corresponding action file — instructions below. Carry all collected fields forward in `workflow_state` — never re-ask for something already provided.

### Progress display

After each phase completes, print a short plain-text progress line in chat:

```
Progress: [✅ Explore] [🔄 Calculate] [⏳ Verify] [⏳ Place Bet] [⏳ Log]
```

Use ✅ for complete, 🔄 for active, ⏳ for pending, ❌ for blocked.

### Matchbook session

Before Phase 1, check if the Matchbook MCP session is active by calling `matchbook_balance`. If it returns an error, ask the user for credentials and call `matchbook_login`. Do this once — subsequent phases reuse the session.

### Workflow state

Collect fields as you go and carry them forward — never re-ask for something already provided.

```
workflow_state = {
  # From Phase 1 (EXPLORE)
  "bookmaker":       ...,
  "sport":           ...,
  "event":           ...,
  "match_date":      ...,
  "bet_type":        ...,   # Qualifying / Free Bet SNR / Free Bet SR / Money Back
  "offer_terms":     ...,   # T&Cs of the offer
  "matchbook_event_id":  ...,
  "matchbook_market_id": ...,
  "matchbook_runner_id": ...,
  "runner_name":     ...,
  "available_lay_prices": [...],  # from MCP
  "recommendation":  ...,   # agent's recommended approach

  # From Phase 2 (CALCULATE)
  "back_stake":      ...,
  "back_odds":       ...,
  "lay_odds":        ...,   # from Matchbook MCP (live)
  "lay_available":   ...,   # liquidity at best lay price
  "commission":      ...,   # default 0% for Matchbook
  "lay_stake":       ...,   # calculated
  "liability":       ...,   # calculated

  # From Phase 3 (VERIFY)
  "user_confirmed":  True/False,

  # From Phase 4 (PLACE BET)
  "offer_id":        ...,   # Matchbook offer ID
  "offer_status":    ...,   # open/matched/delayed
  "back_bet_placed": True/False,

  # From Phase 5 (LOG)
  "row_logged":      ...,
}
```

---

### Phase 1 — EXPLORE

**Goal**: Understand the event, offer, and what Matchbook has available. Provide a recommendation.

Load and follow `actions/explore.md`. At the end, the user should understand what bet to place and at roughly what odds. Then prompt:

> "✅ Exploration done. Ready to move to Phase 2 — Calculate?
> Please share a screenshot of the back bet you'd like to place (or tell me the back stake and odds)."

Print progress block with Phase 1 ✅, Phase 2 🔄.

---

### Phase 2 — CALCULATE

**Goal**: Compute lay stake and profit scenarios using the bookmaker screenshot + live Matchbook odds.

Load and follow `actions/calculate.md`. The lay odds come from Matchbook MCP — do not ask the user for them. At the end, show the calculation result and prompt:

> "✅ Calculation done. Ready to review and confirm?"

Print progress block with Phases 1–2 ✅, Phase 3 🔄.

---

### Phase 3 — VERIFY

**Goal**: Present the complete bet details for user go/no-go decision.

Load and follow `actions/verify.md`. This is a confirmation gate — present all numbers clearly and wait for explicit approval. If the user says no, return to Phase 2 to adjust.

If user confirms:

> "✅ Confirmed. I'll now place the lay bet on Matchbook."

Print progress block with Phases 1–3 ✅, Phase 4 🔄.

---

### Phase 4 — PLACE BET

**Goal**: Place the lay bet via Matchbook MCP, then ask the user to place the back bet.

Load and follow `actions/place-bet.md`. After the lay is placed and the user confirms their back bet with a screenshot, prompt:

> "✅ Both bets placed. Ready to log?"

Print progress block with Phases 1–4 ✅, Phase 5 🔄.

---

### Phase 5 — LOG

**Goal**: Write the bet to the tracker.

Load and follow `actions/log.md`. Pre-fill **all fields from workflow_state** — the confirm prompt should require zero new input from the user. Show the pre-filled summary and ask:

> "Shall I log this?"

After the user confirms and the row is written, print the final progress line with all five phases complete and confirm the row number:

```
Progress: [✅ Explore] [✅ Calculate] [✅ Verify] [✅ Place Bet] [✅ Log]
Bet logged at row X!
```
