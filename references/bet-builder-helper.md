# Reference: Bet Builder Helper

Source: https://outplayed.com/calculators/bet-builder-helper

---

## What is a Bet Builder?

A Bet Builder (also called Same Game Multiple, SGM, or Request a Bet) lets you combine 2+ selections from the **same match** into a single bet. Because the legs are correlated (from the same event), the bookmaker sets the combined odds — you cannot build this combination yourself on an exchange.

Examples:
- "Home Win + Both Teams to Score + Over 2.5 Goals"
- "Player A to Score Anytime + Home Win"
- "Correct Score 2-1"

---

## The Matched Betting Problem

You cannot lay a Bet Builder directly on the exchange — the exact combination does not exist as a single market. Instead, you must find an **equivalent lay market** on the exchange that covers the same outcomes.

---

## How the Outplayed Bet Builder Helper Works

The tool at https://outplayed.com/calculators/bet-builder-helper lets you:

1. **Select your bookmaker** (e.g. Bet365, Sky Bet, Paddy Power, William Hill)
2. **Select the number of selections** in your Bet Builder (e.g. 2, 3, 4, 5+)
3. The tool then displays a **table of possible bet builder combinations** and the corresponding **lay market** on the exchange for each one

Use the tool during the Explore phase to identify what to lay on Matchbook.

---

## Common Bet Builder → Lay Market Mappings

| Bet Builder Combination | Lay Market on Exchange |
|-------------------------|------------------------|
| Match Result only (e.g. Home Win) | Match Odds — lay the team / draw |
| BTTS Yes | Both Teams to Score — lay "No" |
| BTTS No | Both Teams to Score — lay "Yes" |
| Over X.5 Goals | Correct Score or Over/Under market |
| Under X.5 Goals | Correct Score or Over/Under market |
| Home Win + BTTS Yes | Correct Score — lay all scores where home wins and both teams score (e.g. 1-1 is not this; 2-1, 3-1, 3-2 are) |
| Correct Score X-Y | Correct Score — lay that exact score |
| Player to Score Anytime | Anytime Goalscorer — lay the named player |
| Player to Score First | First Goalscorer — lay the named player |
| Player to Score Last | Last Goalscorer — lay the named player |
| Both players to score | Anytime Goalscorer — lay both (hedge via two separate lay bets) |

**Always verify the specific combination using the Outplayed tool** — mappings can vary by bookmaker.

---

## Critical Warning: Correct Score Notation

> **Make sure you lay the right market when laying Correct Scores.**
>
> **1-0 = Home team won 1-0 (home scored 1, away scored 0)**
> **0-1 = Away team won 0-1 (home scored 0, away scored 1)**

The first number is always the **home team's score**. Getting this backwards is a costly mistake.

---

## Calculating the Lay Stake for a Bet Builder

Once you have identified the lay market, treat the calculation exactly like any other bet:

- **Back odds** = the combined Bet Builder odds shown by the bookmaker
- **Lay odds** = the odds for the equivalent lay market on Matchbook
- **Formulas** = same as in `references/formulas.md` (Qualifying or Free Bet depending on offer type)

The calculation is straightforward once the lay market is identified — the hard part is finding the right market.

---

## Lay Liquidity Note

Niche markets (e.g. Correct Score, First Goalscorer) often have lower liquidity than Match Odds. If the lay stake required exceeds available liquidity:
- Consider using a smaller back stake
- Wait closer to kick-off when liquidity tends to increase
- Consider whether the Bet Builder offer is worth pursuing with these constraints

---

## Bet Builder Offer Types

Bookmakers offer Bet Builders in several promo formats:

| Offer Type | What it means |
|------------|---------------|
| **Bet Builder Boost** | Enhanced odds on a specific pre-built combination |
| **Free Bet Builder** | A free bet token usable only on Bet Builders |
| **Acca Insurance on Bet Builder** | Stake refunded as free bet if one leg loses |
| **BOGOF (Bet Builder)** | Bet Builder placed twice — one as a free bet |

For Acca Insurance, you only need to lay the full bet — the partial refund is a bonus on top.

---

## EP (Early Payout) on Bet Builder Legs

If one leg of a Bet Builder has an Early Payout (EP) badge, note that:
- EP on **one leg does not trigger early payout for the whole Bet Builder**
- The entire Bet Builder must settle normally
- Flag this if the bookmaker's offer relies on EP timing
