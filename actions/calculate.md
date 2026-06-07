# Action: Calculate

Compute the lay stake and profit scenarios for a back bet. Lay odds are fetched live from the Matchbook MCP — do not ask the user for lay odds.

## Step 1 — Extract back bet details

From screenshot or message: **Back Stake**, **Back Odds**, **Event**, **Bookmaker**, **Match Date**, **Sport**.

If any required field is missing, ask for it.

## Step 2 — Determine bet type

If not already known from workflow state (Phase 1), ask explicitly:

> "What type of bet is this?
> 1. **Qualifying Bet** — real-money stake to unlock a free bet offer
> 2. **Free Bet** — using a free bet token
> 3. **Money Back if Bet Loses** — real stake, returned as free bet if it loses"

## Step 3 — Follow-up based on bet type

- **Free Bet** → ask: *"Is this SNR (Stake Not Returned)? Most bookmaker free bets are SNR."*
- **Money Back** → ask: *"What is the cashback? (e.g. '100% money back', '£10 back', '50% up to £20')"*
- **Qualifying** → no follow-up needed.

## Step 4 — Fetch live lay odds from Matchbook

If `matchbook_runner_id` is in workflow state (from Phase 1 Explore), call `matchbook_get_lay_odds` with the stored IDs to get fresh prices.

If not available (running Calculate standalone), call `matchbook_search_event` with the event name to find the runner, then `matchbook_get_lay_odds`.

Use the **best lay odds** (lowest available) for the calculation. Also store the available liquidity.

**Fallback:** If Matchbook MCP is unavailable or the event isn't found on Matchbook, ask the user: *"I couldn't find this event on Matchbook. What lay odds are available on your exchange?"*

## Step 5 — Confirm commission rate

Matchbook = 0% commission (default). If the user is using a different exchange, ask:
*"Which exchange? Matchbook = 0%, Betfair = 2% — or tell me your rate."*

## Step 6 — Calculate

Load `references/formulas.md` and apply the correct formula for the bet type.

## Step 7 — Display result

```
📐 Calculation — [Bet Type]:

  Back Stake:  £X.XX  @  X.XX  ([Bookmaker])
  Lay Odds:    X.XX   (£XX.XX available on Matchbook)
  Commission:  X%

  Lay stake required:  £X.XX
  Liability:           £X.XX

  If back wins:   £X.XX
  If lay wins:    £X.XX

📊 Match rating: X.X%  (Back Odds ÷ Lay Odds)
```

For Money Back bets, annotate the lay-wins line:
```
  If lay wins:    £X.XX  (includes £X.XX free bet value at 78% conversion)
```

## Step 8 — Suggestions Checklist

Always check and comment on all of these:

1. **Match rating** — ≥ 90% excellent ✅ | 85–89% acceptable ✅ | < 85% flag and suggest waiting for better odds
2. **Commission** — if c > 0%, show effective match rating = `(Back Odds ÷ Lay Odds) × (1 - c)`
3. **Liability** — if large relative to profit, mention the exchange balance needed. Call `matchbook_balance` to check if sufficient funds are available.
4. **Lay liquidity** — compare required lay stake vs available amount at best odds. If insufficient: *"Only £XX available at X.XX. You may need to take some volume at the next price level (X.XX), which would slightly reduce profit."*
5. **EP (Early Payout) badge** — on Bet Builders: EP on some legs doesn't trigger early payout for the whole bet. On single bets: flag timing mismatch risk.
6. **Free bet conversion** (Money Back only) — 78% is an estimate; aim for odds ~4–6 for a good balance of value vs variance.
7. **Odds movement risk** — *"These are live odds and may change. Proceed promptly to lock them in."*
