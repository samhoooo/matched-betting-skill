---
name: matched-betting
description: >
  Use for any matched betting task: logging bets, verifying back/lay pairs,
  or calculating lay stakes and profits. Use WORKFLOW for: "place a bet", "new bet",
  "start a bet", "I want to bet on X", or any bookmaker screenshot at session start.
  Use individual actions for: "log a bet", "add a bet",
  "record this bet", "bet settled", "I won/lost my bet", "verify my bet",
  "check this bet", "calculate my lay", "what lay stake?", "how much profit?",
  "work out the numbers", "update my bets", "check results", "any bets settled?",
  "update profits", or whenever the user shares a bookmaker or exchange
  screenshot. Always use this skill when matched betting screenshots are shared.
---

# Matched Betting Skill

## Entry points

| Entry point          | When to use |
|----------------------|-------------|
| **WORKFLOW**         | Starting a new bet from scratch. Load `workflows/place-bet.md`. Trigger on: "place a bet", "new bet", "start a bet", "I want to bet on X", or any bookmaker screenshot at session start. **Default to this when in doubt.** |
| **INDIVIDUAL ACTIONS** | Jumping into one step directly. Load only the action file needed. Use when the user explicitly wants a single step. |

## Resource map

| File | Load when |
|------|-----------|
| `workflows/place-bet.md` | Running the full CALCULATE → VERIFY → LOG workflow |
| `actions/calculate.md` | Individual calculate, or workflow Phase 1 |
| `actions/verify.md` | Individual verify, or workflow Phase 2 |
| `actions/log.md` | Individual log/settle, or workflow Phase 3 |
| `actions/update.md` | Auto-settling past bets |
| `references/formulas.md` | Calculating lay stakes, liability, or profit (loaded from within action files) |
| `references/glossary.md` | Domain term definitions — load if user seems unfamiliar or asks for clarification |

## Defaults

| Field      | Default   |
|------------|-----------|
| Exchange   | Matchbook |
| Commission | 0%        |
| Date       | Today     |

Override per-row if the user specifies otherwise.

## Available workflows

- `workflows/place-bet.md` — Place a new bet (CALCULATE → VERIFY → LOG)

## Available actions

- `actions/calculate.md` — Compute lay stake and profit scenarios
- `actions/verify.md` — Check back/lay screenshots match; includes Suggestions Checklist
- `actions/log.md` — Record a new bet or settle an existing one; includes tracker setup
- `actions/update.md` — Auto-settle past bets: scan, search results, write profit/loss
