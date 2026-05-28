# Matched Betting Formulas

All monetary values should be rounded to 2 decimal places.
Commission `c` is expressed as a decimal (e.g. 2% → 0.02).

---

## Qualifying Bet

```
Lay Stake = (Back Stake × Back Odds) / (Lay Odds × (1 - c))
Liability  = Lay Stake × (Lay Odds - 1)

If back wins:  Back profit = Back Stake × (Back Odds - 1)
               Lay loss    = Lay Stake × (Lay Odds - 1)
               Net         = Back profit - Lay loss

If lay wins:   Back loss   = -Back Stake
               Lay profit  = Lay Stake × (1 - c)
               Net         = Lay profit - Back Stake

Both outcomes produce a small, roughly equal negative (the qualifying loss).
```

---

## Free Bet — SNR (Stake Not Returned)

Most bookmaker free bets are SNR — if the back wins, only the winnings are returned, not the original token stake.

```
Lay Stake = (Back Stake × (Back Odds - 1)) / (Lay Odds × (1 - c))
Liability  = Lay Stake × (Lay Odds - 1)

If back wins:  Free bet winnings = Back Stake × (Back Odds - 1)
               Lay loss          = Lay Stake × (Lay Odds - 1)
               Net               = Free bet winnings - Lay loss

If lay wins:   Back token lost   = £0 (no real cash at risk)
               Lay profit        = Lay Stake × (1 - c)
               Net               = Lay profit

Both outcomes produce a similar positive profit.
```

---

## Free Bet — SR (Stake Returned on Win)

```
Lay Stake = (Back Stake × Back Odds) / (Lay Odds × (1 - c))
Liability  = Lay Stake × (Lay Odds - 1)

If back wins:  Total return = Back Stake × Back Odds
               Lay loss     = Lay Stake × (Lay Odds - 1)
               Net          = Total return - Lay loss

If lay wins:   Back token lost = £0 (no real cash at risk)
               Lay profit      = Lay Stake × (1 - c)
               Net             = Lay profit

Both outcomes produce a similar positive profit (slightly higher than SNR).
```

---

## Money Back if Bet Loses (Risk-Free)

### Deriving the Free Bet Amount from cashback terms

- Percentage (e.g. 100% money back): `Free Bet Amount = Back Stake × (cashback% ÷ 100)`
- Fixed amount (e.g. £10 cashback): `Free Bet Amount = stated amount`
- Capped (e.g. 100% up to £20): `Free Bet Amount = min(Back Stake, cap)`

Then estimate cash value using a 78% conversion rate:
`Free Bet Value = Free Bet Amount × 0.78`

### Formula

```
Free Bet Value = Free Bet Amount × 0.78
Lay Stake = (Back Stake × Back Odds - Free Bet Value) / (Lay Odds × (1 - c))
Liability  = Lay Stake × (Lay Odds - 1)

If back wins:  Back profit = Back Stake × (Back Odds - 1)
               Lay loss    = Lay Stake × (Lay Odds - 1)
               Net         = Back profit - Lay loss

If lay wins:   Back loss       = -Back Stake
               Lay profit      = Lay Stake × (1 - c)
               Free Bet Value  = +Free Bet Amount × 0.78
               Net             = Back loss + Lay profit + Free Bet Value

Both outcomes should produce a similar positive profit.
```

**Note on conversion rate:** 78% is an estimate. Higher-odds markets give a higher conversion % but more variance; odds around 4–6 are a good balance.

---

## Match Rating

```
Match Rating          = Back Odds ÷ Lay Odds
Effective Match Rating = (Back Odds ÷ Lay Odds) × (1 - c)   ← use when commission > 0%
```

| Rating   | Assessment                                                  |
|----------|-------------------------------------------------------------|
| ≥ 90%    | Excellent ✅                                                |
| 85–89%   | Acceptable ✅                                               |
| < 85%    | ⚠️ Flag — suggest finding lower lay odds or different market |
