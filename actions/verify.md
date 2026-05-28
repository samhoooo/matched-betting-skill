# Action: Verify

Check that back and lay screenshots match before placing, and validate the numbers.

## Step 1 — Ask for bet type

If not already clear from screenshots:
> "Which bet type is this?
> 1. **Qualifying Bet**
> 2. **Free Bet (SNR)**
> 3. **Money Back if Bet Loses**"

If the type is clearly shown (e.g. "USE MONEY BACK IN FREE BETS" banner), confirm it instead of asking.

## Step 2 — Extract values

From bookmaker: Back Stake, Back Odds, Event, Back selection, Promo label, EP badge (note which legs).
From exchange: Lay selection, Lay Odds, Exchange name, Lay Stake (if already filled by user).

## Step 3 — Check selections match

Back and lay selections must represent the same outcome. Flag immediately if they differ.

✅ Valid: "Man City Win + BTTS" ↔ "Man City and Yes"
❌ Flag: different teams, BTTS on one side only, Draw vs team win

## Step 4 — Calculate

Load `references/formulas.md` and apply the correct formula for the bet type.

## Step 5 — Display full summary

```
✅ / ❌  Selections match: [Back selection] ↔ [Lay selection]

📊 Match rating: Back Odds ÷ Lay Odds = XX.X%

📐 Calculation ([Bet Type]):
  Lay stake required:  £X.XX
  Liability:           £X.XX

  If back wins:   £X.XX
  If lay wins:    £X.XX  (incl. £X.XX free bet @ 78%)  ← only for Money Back

💡 Suggestions:
  [see Suggestions Checklist below]
```

## Step 6 — Suggestions Checklist

*(Canonical copy — also inlined in `actions/calculate.md`)*

Always check and comment on all of these:

1. **Match rating** — ≥ 90% excellent ✅ | 85–89% acceptable ✅ | < 85% flag and suggest lower lay odds
2. **Commission** — if c > 0%, show effective match rating = `(Back Odds ÷ Lay Odds) × (1 - c)`
3. **Liability** — if large relative to profit, mention the exchange balance needed
4. **Lay liquidity** — remind user to check there's enough volume at those odds before placing
5. **EP (Early Payout) badge** — on Bet Builders: EP on some legs doesn't trigger early payout for the whole bet (cleaner for matched betting). On single bets: flag timing mismatch risk between back and lay settlement
6. **Lay stake field on exchange** — if blank in screenshot, remind user to enter the calculated lay stake before confirming
7. **Free bet conversion** (Money Back only) — 78% is an estimate; aim for odds ~4–6 for a good balance of value vs variance

## Clarification Prompts

- *"What bookmaker is this with?"*
- *"What are the lay odds on the exchange?"*
- *"Is this a free bet (SNR) or risk-free (stake back as free bet)?"*
- *"What exchange are you using — Matchbook, Betfair, or something else?"*
