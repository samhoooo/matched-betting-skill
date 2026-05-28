# Action: Calculate

Compute the lay stake and profit scenarios for a back bet before placing. No exchange screenshot required — lay odds can be provided separately.

## Step 1 — Extract back bet details

From screenshot or message: Back Stake, Back Odds, Event, Bookmaker, Match Date, Sport.
If an exchange screenshot is also shared, read Lay Odds and Exchange name from it.
If lay odds aren't visible, ask: *"What lay odds are available on the exchange?"*

## Step 2 — Ask for bet type

Never assume from the screenshot — always ask explicitly:

> "What type of bet is this?
> 1. **Qualifying Bet** — real-money stake to unlock a free bet offer
> 2. **Free Bet** — using a free bet token
> 3. **Money Back if Bet Loses** — real stake, returned as free bet if it loses"

## Step 3 — Follow-up based on bet type

- **Free Bet** → ask: *"Is this SNR (Stake Not Returned)? Most bookmaker free bets are SNR."*
- **Money Back** → ask: *"What is the cashback? (e.g. '100% money back', '£10 back', '50% up to £20')"*
- **Qualifying** → no follow-up needed.

## Step 4 — Confirm commission rate

If exchange wasn't visible: *"Which exchange? Matchbook = 0%, Betfair = 2% — or tell me your rate."*

Default to 0% if unspecified.

## Step 5 — Calculate

Load `references/formulas.md` and apply the correct formula for the bet type.

## Step 6 — Display result

```
📐 Calculation — [Bet Type]:

  Back Stake:  £X.XX  @  X.XX
  Lay Odds:    X.XX
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

## Step 7 — Suggestions Checklist

Always check and comment on all of these:

1. **Match rating** — ≥ 90% excellent ✅ | 85–89% acceptable ✅ | < 85% flag and suggest lower lay odds
2. **Commission** — if c > 0%, show effective match rating = `(Back Odds ÷ Lay Odds) × (1 - c)`
3. **Liability** — if large relative to profit, mention the exchange balance needed
4. **Lay liquidity** — remind user to check there's enough volume at those odds before placing
5. **EP (Early Payout) badge** — on Bet Builders: EP on some legs doesn't trigger early payout for the whole bet (cleaner for matched betting). On single bets: flag timing mismatch risk between back and lay settlement
6. **Lay stake field on exchange** — if blank in screenshot, remind user to enter the calculated lay stake before confirming
7. **Free bet conversion** (Money Back only) — 78% is an estimate; aim for odds ~4–6 for a good balance of value vs variance
