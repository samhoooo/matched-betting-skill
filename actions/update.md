# Action: Update

Scan the tracker for unsettled bets whose match date has passed, search for results, calculate profit/loss, and write updates — via the **matched-betting-tracker MCP server**.

## Step 1 — Find unsettled past bets

Call `query_bets` with `settled="false"` (add `sport`/`bookmaker`/`exchange`/`date_from`/`date_to` filters if the user narrowed the request). The tool returns JSON rows with fields including `id`, `matchDate`, `event`, `offerType`, `backStake`, `backOdds`, `layOdds`, `commission`, `layStake`.

From the returned rows, keep only those where `matchDate` (falling back to `date` if `matchDate` is empty) is on or before today:

```python
from datetime import date

today = date.today()
unsettled = []

for row in results:  # results = parsed JSON from query_bets
    raw_match_date = row.get("matchDate") or row.get("date")
    if not raw_match_date:
        continue
    match_date = date.fromisoformat(str(raw_match_date)[:10])
    if match_date <= today:
        unsettled.append(row)
```

If none found: *"No unsettled past bets found in your tracker."* Stop.

Otherwise list them: id, match date, event, offer type.

## Step 2 — Search for each result

Construct a web search query from Event + Match Date:

- Football: `"<Team A> vs <Team B> result <date>"`
- Horse Racing: `"<Race name> <course> result <date>"`
- Other: `"<Event> result <date>"`

Extract: winner/outcome, score (if available), specific selection outcome (e.g. did BTTS land?).

If result is ambiguous or not found, flag as **"Result unclear — please check manually"** and skip.

## Step 3 — Determine back bet outcome

- Back wins → the selection the user backed came in
- Back loses → lay wins

Check the specific selection in the `event` field, not just the match winner.

## Step 4 — Calculate profit

Load `references/formulas.md`. Map Offer Type → formula:

| Offer Type contains…                                          | Formula        |
|---------------------------------------------------------------|----------------|
| Free Bet, Token                                               | Free Bet SNR   |
| Risk-Free, Money Back                                         | Money Back     |
| Qualifying, Sign-up, Reload, Enhanced Odds, Acca, Bet Builder | Qualifying Bet |

```
c = commission / 100

If back wins:
  net = back_stake × (back_odds - 1) - lay_stake × (lay_odds - 1)

If lay wins:
  net = -back_stake + lay_stake × (1 - c)   # use -0 for free bets (no cash stake)
```

Round to 2 decimal places. If `layStake` is missing/0, flag the row and skip.

## Step 5 — Confirm before writing

```
📋 Ready to update X bet(s):

Bet #3 — Arsenal vs Man City (BTTS) — match date 2026-05-10
  Result:  Both teams scored ✅ → Back bet WON
  Profit:  £4.23
  Notes:   → "Settled"

Bet #5 — Fulham vs Chelsea — match date 2026-05-11
  Result:  Fulham lost → Back bet LOST (lay won)
  Profit:  £3.87
  Notes:   → "Settled"

Bet #7 — Ascot 3:20 Cheltenham Gold — match date 2026-05-11
  ⚠️  Result unclear — skipped. Please settle manually.

Shall I update bets #3 and #5?
```

Wait for explicit user confirmation before writing.

## Step 6 — Write updates

For each confirmed bet, call `update_bet`:

```
update_bet(id=<id>, profit=<round(net_profit, 2)>, settled=True, notes="Settled")
```

If the tool result contains an `"error"` key, report it and stop updating that bet.

Confirm: *"Updated X bet(s). Y bet(s) skipped (unclear result)."*
