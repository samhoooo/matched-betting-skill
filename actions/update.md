# Action: Update

Scan the tracker for unsettled bets whose match date has passed, search for results, calculate profit/loss, and write updates.

## Step 0 — Resolve tracker path

**A. Read config**
```bash
cat ~/.matched_betting_config 2>/dev/null
```

**B. Interpret the result**

| Result | Action |
|--------|--------|
| Path returned AND file exists at that path | Use it. Proceed silently — don't mention the config to the user. |
| Path returned BUT file missing at that path | Tell the user: *"I couldn't find your tracker at `<path>`. Has it moved? Please share the new location."* → on reply, update config and proceed. |
| Config missing | Tell the user: *"I don't have a tracker on file. Please share the path to your tracker."* → save to config and proceed. |

Then copy to working location:
```bash
tracker_path=$(cat ~/.matched_betting_config)
cp "$tracker_path" /tmp/tracker_edit.xlsx
```

## Step 1 — Find unsettled past bets

```python
from openpyxl import load_workbook
from datetime import date

wb = load_workbook("/tmp/tracker_edit.xlsx")
ws = wb.active

today = date.today()
unsettled = []

for row in ws.iter_rows(min_row=2, values_only=False):
    date_cell       = row[0]   # A — Date placed
    event_cell      = row[4]   # E — Event
    match_date_cell = row[5]   # F — Match Date
    offer_cell      = row[6]   # G — Offer Type
    back_stake      = row[7]   # H
    back_odds       = row[8]   # I
    lay_odds        = row[9]   # J
    commission      = row[10]  # K
    lay_stake       = row[11]  # L (formula, read-only)
    profit_cell     = row[13]  # N — Profit
    notes_cell      = row[14]  # O — Notes

    if date_cell.value is None:
        continue

    raw_match_date = match_date_cell.value or date_cell.value
    if raw_match_date is None:
        continue
    match_date = raw_match_date if isinstance(raw_match_date, date) \
                 else date.fromisoformat(str(raw_match_date))

    already_settled = (
        profit_cell.value not in (None, "") or
        ("settled" in str(notes_cell.value or "").lower())
    )

    if not already_settled and match_date <= today:
        unsettled.append({
            "row":        date_cell.row,
            "match_date": match_date,
            "event":      event_cell.value,
            "offer_type": offer_cell.value,
            "back_stake": back_stake.value,
            "back_odds":  back_odds.value,
            "lay_odds":   lay_odds.value,
            "commission": commission.value or 0,
            "lay_stake":  lay_stake.value,
        })
```

If none found: *"No unsettled past bets found in your tracker."* Stop.

Otherwise list them: row number, match date, event, offer type.

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

Check the specific selection in the Event field, not just the match winner.

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

Round to 2 decimal places. If `lay_stake` is None, flag the row and skip.

## Step 5 — Confirm before writing

```
📋 Ready to update X bet(s):

Row 3 — Arsenal vs Man City (BTTS) — match date 2026-05-10
  Result:  Both teams scored ✅ → Back bet WON
  Profit:  £4.23
  Notes:   → "Settled"

Row 5 — Fulham vs Chelsea — match date 2026-05-11
  Result:  Fulham lost → Back bet LOST (lay won)
  Profit:  £3.87
  Notes:   → "Settled"

Row 7 — Ascot 3:20 Cheltenham Gold — match date 2026-05-11
  ⚠️  Result unclear — skipped. Please settle manually.

Shall I update rows 3 and 5?
```

Wait for explicit user confirmation before writing.

## Step 6 — Write updates

```python
ws.cell(row_num, 14).value = round(net_profit, 2)   # N — Profit
ws.cell(row_num, 15).value = "Settled"               # O — Notes
```

Copy pattern: `cp tracker /tmp/tracker_edit.xlsx` → edit → `cp back`.

Confirm: *"Updated X row(s). Y bet(s) skipped (unclear result)."*
