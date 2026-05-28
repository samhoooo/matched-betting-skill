# Matched Betting Skill for Claude

A Claude skill for end-to-end matched betting workflows — calculating lay stakes, verifying back/lay pairs, and logging bets to a tracker.

## What it does

| Entry point | Trigger |
|-------------|---------|
| **Full workflow** (Calculate → Verify → Log) | "place a bet", "new bet", "I want to bet on X", or share a bookmaker screenshot |
| **Individual actions** | "calculate my lay", "verify this bet", "log a bet", "update my bets" |

## File structure

```
matched-betting/
├── SKILL.md                  # Entry point — Claude reads this first
├── workflows/
│   └── place-bet.md          # Full CALCULATE → VERIFY → LOG workflow
├── actions/
│   ├── calculate.md          # Compute lay stake and profit scenarios
│   ├── verify.md             # Check back/lay screenshots match
│   ├── log.md                # Record or settle a bet in the tracker
│   └── update.md             # Auto-settle past bets by searching results
└── references/
    ├── formulas.md           # Lay stake, liability, profit formulas
    └── glossary.md           # Domain terms (back, lay, SNR, match rating, etc.)
```

## Supported bet types

- **Qualifying Bet** — real-money stake to unlock a free bet offer
- **Free Bet SNR** — stake not returned on win (most common)
- **Free Bet SR** — stake returned on win
- **Money Back / Risk-Free** — stake returned as free bet if lost

## Tracker

The skill logs bets to a local `.xlsx` file using `openpyxl`. On first use it will ask you to create a new tracker or point to an existing one, and saves the path to `~/.matched_betting_config` for future sessions.

Default exchange: **Matchbook** (0% commission). Override per bet.

## Installing

### Claude.ai
Upload the `matched-betting/` folder as a custom skill via Settings → Skills.

### Claude Code / Claude API
Place the folder in your skills directory (e.g. `~/.claude/skills/`) or reference it in your API call.

## Notes

- No sensitive data (Sheet IDs, account names) is stored in this repo.
- Formulas reference: `references/formulas.md`
- Match rating thresholds: ≥ 90% excellent, 85–89% acceptable, < 85% flag.
