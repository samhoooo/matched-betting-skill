# Action: Place Bet

Place the lay bet on Matchbook via MCP, then prompt the user to place their back bet on the bookmaker.

## Step 1 — Place the lay bet

Call `matchbook_place_lay` with:
- `runner_id`: from workflow state
- `odds`: the lay odds confirmed in Verify
- `stake`: the calculated lay stake from Calculate

**This places a real bet with real money.**

## Step 2 — Handle the offer response

The MCP returns an offer with a status. Handle each:

### Status: `matched`
The lay bet has been fully matched. Proceed to Step 3.

```
✅ Lay bet placed and MATCHED on Matchbook!
  Offer ID:    #XXXXX
  Runner:      [Runner name]
  Lay odds:    X.XX
  Lay stake:   £X.XX
  Liability:   £X.XX
```

### Status: `open`
The offer is on the exchange but not yet matched by another user.

```
⏳ Lay bet placed but UNMATCHED on Matchbook.
  Offer ID:    #XXXXX
  Runner:      [Runner name]
  Lay odds:    X.XX
  Lay stake:   £X.XX (£X.XX remaining)

  Options:
  1. **Wait** — I'll check again in a moment
  2. **Adjust odds** — increase lay odds to improve chances of matching (reduces profit)
  3. **Cancel** — cancel the offer and try again
```

If user chooses to wait, call `matchbook_get_offer` after a short pause to re-check. Repeat up to 3 times.

If user chooses to adjust, ask for new odds, cancel the current offer with `matchbook_cancel_offer`, then re-place with new odds.

### Status: `delayed`
In-play delay — the offer is being held by the exchange.

```
⏳ Lay bet is DELAYED (in-play delay of X seconds).
  The exchange is processing your offer. I'll check back shortly.
```

Call `matchbook_get_offer` after the delay period to check final status.

### Status: `failed`
The offer was rejected.

```
❌ Lay bet FAILED on Matchbook.
  Error: [error message from response]

  This could be due to:
  - Insufficient funds
  - Odds no longer available
  - Market suspended

  Would you like to retry with adjusted odds or check your balance?
```

## Step 3 — Prompt user to place back bet

Once the lay is confirmed matched:

```
✅ Lay bet is locked in on Matchbook.

Now please place your back bet on [Bookmaker]:
  Selection:  [Selection name]
  Stake:      £X.XX
  Odds:       X.XX (or better)

📸 Share a screenshot of your placed back bet slip when done.
```

## Step 4 — Verify back bet screenshot

When the user shares the back bet screenshot, extract and verify:
- **Selection matches** the lay selection
- **Stake matches** the planned back stake
- **Odds are equal to or better than** the planned back odds

If everything matches:
```
✅ Back bet confirmed! Both sides are placed.
  Back: £X.XX @ X.XX on [Bookmaker]
  Lay:  £X.XX @ X.XX on Matchbook (Offer #XXXXX)
```

If there's a discrepancy (different odds, stake, or selection):
```
⚠️ The placed bet differs from the plan:
  Planned: £X.XX @ X.XX on [Selection]
  Actual:  £X.XX @ X.XX on [Selection]

  This changes the expected outcome:
    If back wins: £X.XX (was £X.XX)
    If lay wins:  £X.XX (was £X.XX)

  Do you want to proceed with logging these actual values, or adjust the lay?
```

Update workflow_state with the actual back bet values if they differ.

## Step 5 — Store offer details

Save to workflow state for logging:
- `offer_id`
- `offer_status`
- Actual lay odds and stake (in case of adjustment)
- Confirmed back odds and stake
- `back_bet_placed = True`
