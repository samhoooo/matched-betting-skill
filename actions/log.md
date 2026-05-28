# Action: Log

Log a new bet or settle an existing one in the user's tracker file.

> **Critical file access rules:**
> 1. **Always work directly on the user's tracker file** — never ask the user to upload it, never save to an outputs folder, never use Google Drive or any cloud connector.
> 2. **Use openpyxl via bash** — (a) `cp` the tracker to `/tmp/tracker_edit.xlsx`, (b) edit it there with openpyxl, (c) `cp` it back to the original path.
> 3. **Never use Google Drive** — do not use any Google Drive MCP tools for this tracker.

## Step 0 — Resolve the tracker path

Before doing anything else, run this flow every session:

**A. Read config**
```bash
cat ~/.matched_betting_config 2>/dev/null
```

**B. Interpret the result**

| Result | Action |
|--------|--------|
| Path returned AND file exists at that path | Use it. Proceed silently — don't mention the config to the user. |
| Path returned BUT file missing at that path | Tell the user: *"I couldn't find your tracker at `<path>`. Has it moved? Please share the new location."* → on reply, update config and proceed. |
| Config missing (first-time user) | → **Run First-Time Setup** below |

**C. First-Time Setup**

1. Ask the user:
   > "I don't have a tracker on file yet. Would you like me to:
   > 1. **Create a new tracker** — I'll set it up with all the right columns
   > 2. **Use an existing file** — share the path and I'll use that"

2. **If creating new** — ask for preferred save location, suggesting `~/Documents/matched_betting_tracker.xlsx` as default. Then run:

```python
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment
from openpyxl.utils import get_column_letter

wb = Workbook()
ws = wb.active
ws.title = "Bets"

headers = [
    "Date", "Bookmaker", "Exchange", "Sport", "Event",
    "Match Date", "Offer Type", "Back Stake", "Back Odds", "Lay Odds",
    "Commission %", "Lay Stake", "Liability", "Profit", "Notes"
]

for col, header in enumerate(headers, 1):
    cell = ws.cell(1, col, header)
    cell.font = Font(bold=True, color="FFFFFF")
    cell.fill = PatternFill("solid", fgColor="2E4057")
    cell.alignment = Alignment(horizontal="center")

widths = [12, 14, 12, 12, 30, 12, 16, 12, 12, 10, 14, 12, 12, 10, 30]
for col, width in enumerate(widths, 1):
    ws.column_dimensions[get_column_letter(col)].width = width

ws.freeze_panes = "A2"

for row in range(2, 1001):
    ws.cell(row, 12).value = (
        f"=IF(H{row}=\"\",\"\","
        f"ROUND((H{row}*I{row})/(J{row}*(1-K{row}/100)),2))"
    )
    ws.cell(row, 13).value = (
        f"=IF(L{row}=\"\",\"\","
        f"ROUND(L{row}*(J{row}-1),2))"
    )

wb.save(tracker_path)
```

3. **If using existing file** — ask the user to share the full path. Verify it exists with `ls "<path>"` before saving.

4. **Save the path to config**:
```bash
echo "<resolved_path>" > ~/.matched_betting_config
```

5. Confirm: *"Tracker is set up at `<path>`. I'll use this automatically from now on."*

## Sheet Structure

| Col | Field        | Type       | Notes                                       |
|-----|--------------|------------|---------------------------------------------|
| A   | Date         | YYYY-MM-DD | Date bet was placed                         |
| B   | Bookmaker    | Text       | e.g. Bet365, William Hill                   |
| C   | Exchange     | Text       | Default: Matchbook                          |
| D   | Sport        | Text       | e.g. Football, Horse Racing                 |
| E   | Event        | Text       | Match / race name, include bet details      |
| F   | Match Date   | YYYY-MM-DD | Date the match / race takes place           |
| G   | Offer Type   | Text       | e.g. Risk-Free, Free Bet, Enhanced Odds     |
| H   | Back Stake   | £ number   | Amount staked at bookmaker                  |
| I   | Back Odds    | Decimal    | Bookmaker odds (decimal)                    |
| J   | Lay Odds     | Decimal    | Exchange lay odds (decimal)                 |
| K   | Commission % | Number     | Exchange commission rate, default 0         |
| L   | Lay Stake    | **Formula**| Auto-calculated — do NOT overwrite          |
| M   | Liability    | **Formula**| Auto-calculated — do NOT overwrite          |
| N   | Profit       | £ number   | Enter once settled (positive or negative)   |
| O   | Notes        | Text       | Promo details, status (Pending/Settled)     |

**Never write to L or M** — they contain spreadsheet formulas.

## Logging a New Bet

**Step 1 — Extract fields** from the user's message or screenshot:
- **Required**: Bookmaker, Sport, Event, Match Date, Offer Type, Back Stake, Back Odds, Lay Odds
- **Optional / defaultable**: Date (placed), Exchange, Commission %, Notes
- **Settle only**: Profit

If any required field is missing or ambiguous, ask before writing.

**Step 2 — Confirm** before writing:

```
Ready to log:
• Date: 2026-05-04
• Bookmaker: Bet365
• Exchange: Matchbook
• Sport: Football
• Event: Arsenal vs Man City — BTTS
• Match Date: 2026-05-05
• Offer Type: Free Bet
• Back Stake: £10.00
• Back Odds: 2.10
• Lay Odds: 2.14
• Commission: 0%
• Notes: £10 free bet from sign-up offer

Shall I add this?
```

**Step 3 — Write** to the next empty row (columns A–K, N–O only):

```bash
tracker_path=$(cat ~/.matched_betting_config)
cp "$tracker_path" /tmp/tracker_edit.xlsx
```

```python
from openpyxl import load_workbook

wb = load_workbook("/tmp/tracker_edit.xlsx")
ws = wb.active
next_row = next(
    (r for r in range(2, ws.max_row + 2) if ws.cell(r, 1).value is None),
    ws.max_row + 1
)

ws.cell(next_row, 1).value = date
ws.cell(next_row, 2).value = bookmaker
ws.cell(next_row, 3).value = exchange
ws.cell(next_row, 4).value = sport
ws.cell(next_row, 5).value = event
ws.cell(next_row, 6).value = match_date
ws.cell(next_row, 7).value = offer_type
ws.cell(next_row, 8).value = back_stake
ws.cell(next_row, 9).value = back_odds
ws.cell(next_row, 10).value = lay_odds
ws.cell(next_row, 11).value = commission
# Skip L (12) and M (13) — formulas
ws.cell(next_row, 14).value = profit or None
ws.cell(next_row, 15).value = notes

wb.save("/tmp/tracker_edit.xlsx")
```

```bash
cp /tmp/tracker_edit.xlsx "$tracker_path"
```

**Step 4 — Confirm** to the user which row was added.

## Settling a Bet

1. Ask for (or extract): Event name + Profit amount
2. Read config and copy tracker to `/tmp/tracker_edit.xlsx` (same as Step 3 above)
3. Find the row by matching column E (Event) or Date + Bookmaker
4. Confirm which row was found before editing
5. Write profit to column N; optionally update Notes (column O) to "Settled"
6. Save and copy back to the original path

## Field Extraction from Screenshots

| Screenshot type | Look for |
|-----------------|----------|
| Bookmaker       | Stake → Back Stake; Odds → Back Odds; Event/match → Event; Sport type → Sport; Branding → Bookmaker; Promo label → Offer Type |
| Exchange        | Lay odds → Lay Odds; Exchange name → Exchange |

Always confirm extracted values with the user before logging.

## Common Offer Types

- **Risk-Free** — stake returned as free bet if it loses
- **Free Bet** — no stake returned on win (SNR)
- **Enhanced Odds** — boosted odds on a specific selection
- **Reload** — ongoing deposit bonus
- **Acca Insurance** — refund if one leg of accumulator loses
- **Bet Builder** — combined selection within one match
