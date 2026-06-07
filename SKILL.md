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

## Matchbook MCP Integration

This skill uses the **Matchbook MCP server** (`matchbook`) for live exchange data and bet placement. The following MCP tools are available:

| MCP Tool | Purpose |
|----------|---------|
| `matchbook_login` | Authenticate with Matchbook (required once per session) |
| `matchbook_balance` | Check account balance and free funds |
| `matchbook_search_event` | Search events by name, returns runner IDs + lay prices |
| `matchbook_get_lay_odds` | Fetch live lay odds for a specific runner |
| `matchbook_place_lay` | Place a lay bet (requires explicit user confirmation) |
| `matchbook_get_offer` | Check status of a submitted offer |
| `matchbook_cancel_offer` | Cancel an open/unmatched offer |

**Session check:** Before any MCP call, try `matchbook_balance`. If it fails with a session error, ask the user for their Matchbook credentials and call `matchbook_login`.

## Entry points

| Entry point          | When to use |
|----------------------|-------------|
| **WORKFLOW**         | Starting a new bet from scratch. Load `workflows/place-bet.md`. Trigger on: "place a bet", "new bet", "start a bet", "I want to bet on X", or any bookmaker screenshot at session start. **Default to this when in doubt.** |
| **INDIVIDUAL ACTIONS** | Jumping into one step directly. Load only the action file needed. Use when the user explicitly wants a single step. |

## Resource map

| File | Load when |
|------|-----------|
| `workflows/place-bet.md` | Running the full EXPLORE → CALCULATE → VERIFY → PLACE BET → LOG workflow |
| `actions/explore.md` | Individual explore, or workflow Phase 1 |
| `actions/calculate.md` | Individual calculate, or workflow Phase 2 |
| `actions/verify.md` | Individual verify, or workflow Phase 3 |
| `actions/place-bet.md` | Individual place bet, or workflow Phase 4 |
| `actions/log.md` | Individual log/settle, or workflow Phase 5 |
| `actions/update.md` | Auto-settling past bets |
| `references/formulas.md` | Calculating lay stakes, liability, or profit (loaded from within action files) |
| `references/glossary.md` | Domain term definitions — load if user seems unfamiliar or asks for clarification |
| `references/bet-builder-helper.md` | Bet Builder methodology: how to find the lay market for a Bet Builder using the Outplayed tool, common mappings, correct score notation, and offer types. Load whenever the user mentions a Bet Builder, SGM, Same Game Multiple, or Request a Bet. |

## Defaults

| Field      | Default   |
|------------|-----------|
| Exchange   | Matchbook |
| Commission | 0%        |
| Date       | Today     |

Override per-row if the user specifies otherwise.

## Available workflows

- `workflows/place-bet.md` — Place a new bet (EXPLORE → CALCULATE → VERIFY → PLACE BET → LOG)

## Available actions

- `actions/explore.md` — Fetch Matchbook data for an event, analyse offer terms, recommend bet strategy
- `actions/calculate.md` — Compute lay stake and profit scenarios (auto-fetches lay odds from Matchbook)
- `actions/verify.md` — Present lay stake and odds for user go/no-go decision
- `actions/place-bet.md` — Place the lay bet via Matchbook MCP, prompt user to place back bet
- `actions/log.md` — Record a new bet or settle an existing one; includes tracker setup
- `actions/update.md` — Auto-settle past bets: scan, search results, write profit/loss
