# Action: Explore

Analyse an event and offer from a bookmaker, fetch Matchbook exchange data, and recommend a betting strategy.

## Step 1 — Collect inputs from user

The user should provide:
1. **Screenshot** of the bookmaker event / offer page
2. **Bet type**: Qualifying / Free Bet / Money Back (if not obvious from the screenshot)
3. **Offer terms & conditions** (can be a screenshot or pasted text)

Extract from the screenshot:
- Bookmaker name
- Sport
- Event name (teams / runners)
- Match date
- Available selections and odds ranges (if visible)
- Any promo labels, badges, or offer banners

Extract from offer T&Cs:
- Minimum odds requirement (e.g. "odds of 2.0 or greater")
- Rollover / wagering requirements
- Eligible markets (e.g. "match result only", "excludes bet builders")
- Free bet expiry
- Maximum stake limits
- Any other restrictions

## Step 2 — Search Matchbook for the event

Call `matchbook_search_event` with the event name (e.g. "Arsenal vs Chelsea"). Use keywords from the screenshot — team names, race name, etc.

If no results, try shorter or alternative terms (e.g. just one team name, or the competition name).

Store the returned `event_id`, `market_id`, `runner_id` values in workflow state for later phases.

## Step 3 — Analyse lay odds and liquidity

From the search results, examine the lay prices for each runner:
- Best lay odds available
- Liquidity at each price level
- Number of price levels with meaningful liquidity

If the search returned prices inline, use those. Otherwise call `matchbook_get_lay_odds` for specific runners of interest.

## Step 4 — Provide recommendation

Based on the bet type, offer terms, and available lay odds, recommend a strategy:

### For Qualifying Bets

The goal is to minimise the qualifying loss. Recommend selections where:
- **Lay odds are close to back odds** (high match rating ≥ 90%)
- **Sufficient liquidity** exists at the best lay price for the expected stake
- **Odds meet the minimum requirement** from the offer T&Cs

Sweet spots by odds range:
- Odds 1.5–3.0: Typical sweet spot for qualifying bets — low lay liability, small qualifying loss
- Odds 3.0–6.0: Acceptable if match rating is good, but liability grows
- Odds 6.0+: Usually avoid for qualifying — large liability for small benefit

> "Based on Matchbook's lay odds, here are the best options for your qualifying bet:
>
> 1. **[Runner A]** — Lay odds: X.XX (£XX available) — estimated qualifying loss: £X.XX
> 2. **[Runner B]** — Lay odds: X.XX (£XX available) — estimated qualifying loss: £X.XX
>
> I recommend **[Runner A]** because [reason: best match rating / lowest loss / meets min odds].
>
> Note: [Any T&C flags — e.g. 'min odds requirement is 2.0, so options below 2.0 won't qualify']"

### For Free Bets (SNR)

The goal is to maximise free bet conversion. Recommend selections where:
- **Higher odds** (4.0–8.0 sweet spot) for better conversion %
- **Good liquidity** — the full free bet stake needs to be matched
- **Odds meet any minimum requirement**

> "For your £XX free bet, here are the best conversion options:
>
> 1. **[Runner A]** — Lay odds: X.XX — estimated profit: £X.XX (XX% conversion)
> 2. **[Runner B]** — Lay odds: X.XX — estimated profit: £X.XX (XX% conversion)
>
> Higher odds = better conversion but more variance. I recommend **[Runner A]** as a good balance."

### For Money Back / Risk-Free Bets

The goal is to minimise qualifying loss while ensuring the free bet trigger works if the back loses.

> "For your money back offer (£XX stake, XX% cashback):
>
> 1. **[Runner A]** — Lay odds: X.XX — if back wins: £X.XX profit / if back loses: you get £XX free bet (est. value £X.XX)
>
> Either outcome is profitable. I recommend **[Runner A]** for the best balance."

### General flags

Always flag:
- **Low liquidity**: "Only £XX available at X.XX — your full stake may not be matched. Consider a smaller stake or waiting for more liquidity."
- **T&C conflicts**: "The offer requires min odds of X.X but the best match rating is at odds below that."
- **Market restrictions**: "The offer excludes [market type] — make sure your selection is on an eligible market."
- **Timing**: "Match starts in X hours — lay odds may move. Place promptly once you decide."

## Bet Builder Variant

If the bet is a **Bet Builder** (also called SGM, Same Game Multiple, or Request a Bet), load `references/bet-builder-helper.md` and follow this process **instead of** the standard Matchbook search for the bet builder itself:

### Step BB-1 — Identify the Bet Builder combination

Extract from the bookmaker screenshot:
- Each selection in the Bet Builder (e.g. "Home Win + BTTS Yes + Over 2.5 Goals")
- The combined Bet Builder odds
- Number of selections

### Step BB-2 — Find the exchange lay market

Consult `references/bet-builder-helper.md`:
1. Look up the combination in the **Common Mappings** table to identify the lay market type
2. If the combination isn't in the table, direct the user to: https://outplayed.com/calculators/bet-builder-helper — select their bookmaker and number of selections, then report the lay market shown

### Step BB-3 — Search Matchbook for the lay market

Call `matchbook_search_event` with the event name to get the event. Then identify the **specific market** that matches the lay requirement:
- For Correct Score: search for the Correct Score market
- For Anytime Goalscorer / First Goalscorer: search for that player market
- For BTTS: search for the Both Teams to Score market
- For Match Odds: standard match result market

> ⚠️ **Correct Score warning**: The first number is always the **home team's score**. 1-0 = home team won. 0-1 = away team won. Double-check before laying.

### Step BB-4 — Check lay liquidity

Niche markets (Correct Score, Goalscorer) often have less liquidity than Match Odds. Flag if the expected lay stake exceeds available volume at the best price. Suggest a smaller back stake or waiting closer to kick-off if needed.

### Step BB-5 — Provide recommendation

Present the lay market, lay odds, and liquidity clearly. Proceed to Phase 2 (Calculate) using:
- **Back odds** = the combined Bet Builder odds
- **Lay odds** = the exchange odds for the identified lay market

---

## Step 5 — Confirm direction

After presenting the recommendation, ask:

> "Which selection would you like to go with? Or would you like me to check a different market/event?"

Once the user picks a selection, store the runner details in workflow state and proceed to Phase 2 (Calculate).
