# Action: Verify

Present the complete bet plan for user go/no-go decision. This is the confirmation gate before real money is committed.

No exchange screenshot is needed — all lay data comes from Matchbook MCP.

## Step 1 — Re-fetch live lay odds

Call `matchbook_get_lay_odds` to get the latest price for the selected runner. Compare against the odds used in Calculate:

- **If odds unchanged or improved**: proceed with existing calculation.
- **If odds have moved unfavourably**: recalculate using the new odds, flag the change.

```
⚠️ Lay odds have moved since calculation:
  Was: X.XX → Now: X.XX
  Updated lay stake: £X.XX (was £X.XX)
  Updated profit: if back wins £X.XX / if lay wins £X.XX
```

## Step 2 — Check balance

Call `matchbook_balance` and verify the user has enough free funds to cover the liability.

- **Sufficient**: show balance info quietly in the summary.
- **Insufficient**: flag clearly and stop.

```
❌ Insufficient funds on Matchbook:
  Liability required: £X.XX
  Free funds available: £X.XX
  Shortfall: £X.XX
  → Please deposit more funds before proceeding.
```

## Step 3 — Present full summary for confirmation

```
🔍 Bet Review — Ready to place?

  📋 Event:       [Event name]
  📅 Match date:  [Date]
  🏢 Bookmaker:   [Bookmaker]
  🎯 Bet type:    [Qualifying / Free Bet SNR / Money Back]

  BACK BET (you place on [Bookmaker]):
    Selection:  [Selection name]
    Stake:      £X.XX
    Odds:       X.XX

  LAY BET (I place on Matchbook):
    Selection:  [Runner name]
    Stake:      £X.XX
    Odds:       X.XX  (£XX.XX available)
    Liability:  £X.XX

  📊 Match rating:  XX.X%
  💰 Expected outcome:
    If back wins:  £X.XX
    If lay wins:   £X.XX

  💳 Matchbook balance: £X.XX (after liability: £X.XX remaining)

  ⚡ Confirm to proceed — I'll place the lay bet on Matchbook,
     then you place the back bet on [Bookmaker].
```

## Step 4 — Wait for explicit go/no-go

Do NOT proceed without clear user confirmation. Acceptable confirmations:
- "Yes", "Go", "Confirm", "Place it", "Do it"

If the user says no, asks to change something, or hesitates:
- Offer to adjust (different odds, stake, or selection)
- Return to Calculate if numbers need reworking
- Return to Explore if they want a different selection entirely

## Suggestions Checklist

If not already covered in Calculate, check these:

1. **Selections match** — back and lay selection represent the same outcome
2. **Match rating** — ≥ 90% excellent ✅ | 85–89% acceptable ✅ | < 85% flag
3. **Liquidity** — enough volume at the quoted lay odds for the full stake
4. **Balance** — sufficient free funds on Matchbook
5. **T&C compliance** — back odds meet minimum requirement, eligible market
6. **Timing** — flag if match starts very soon (odds may shift before back bet is placed)
