# Action: Log

Log a new bet or settle an existing one using the **matched-betting-tracker MCP server**.

> **Critical rules:**
> 1. **Always use the `matched-betting-tracker` MCP tools** (`log_bet`, `update_bet`, `query_bets`) — never create or edit a local tracker file, never use openpyxl/Excel, never use Google Drive or any cloud connector.
> 2. `log_bet` does **not** re-derive `layStake`/`liability`/`profit` from stake/odds — it's a plain insert. Always pass the numbers already worked out (in workflow state from `actions/calculate.md`, or computed manually).
> 3. `date`, `bookmaker`, `exchange`, `sport`, `event` are required on `log_bet` — never guess or default these; ask the user for any that are missing.

## Bet Fields

| MCP param    | Type          | Notes                                                |
|--------------|---------------|-------------------------------------------------------|
| `date`       | YYYY-MM-DD    | Date bet was placed. Required.                       |
| `bookmaker`  | Text          | e.g. Bet365, William Hill. Required.                  |
| `exchange`   | Text          | Default: Matchbook. Required.                         |
| `sport`      | Text          | e.g. Football, Horse Racing. Required.                |
| `event`      | Text          | Match / race name, include bet details. Required.     |
| `match_date` | YYYY-MM-DD    | Date the match / race takes place.                    |
| `offer_type` | Text          | e.g. Risk-Free, Free Bet, Enhanced Odds.               |
| `settled`    | bool          | Default `false` when logging a new bet.               |
| `back_stake` | £ number      | Amount staked at bookmaker.                            |
| `back_odds`  | Decimal       | Bookmaker odds (decimal).                              |
| `lay_odds`   | Decimal       | Exchange lay odds (decimal).                           |
| `commission` | Number        | Exchange commission rate, default 0.                   |
| `lay_stake`  | £ number      | Pass the value already calculated — not re-derived.    |
| `liability`  | £ number      | Pass the value already calculated — not re-derived.    |
| `profit`     | £ number      | Leave 0/unset until settled.                            |
| `notes`      | Text          | Promo details, status, Matchbook offer ID.              |

## Logging a New Bet

### From workflow state (preferred)

When running as part of the Place Bet workflow, all fields should already be in `workflow_state`. Map them:

| `log_bet` param | Source |
|------------------|--------|
| `date` | Today's date |
| `bookmaker` | `workflow_state.bookmaker` |
| `exchange` | "Matchbook" |
| `sport` | `workflow_state.sport` |
| `event` | `workflow_state.event` + selection details |
| `match_date` | `workflow_state.match_date` |
| `offer_type` | `workflow_state.bet_type` |
| `settled` | `false` |
| `back_stake` | `workflow_state.back_stake` |
| `back_odds` | `workflow_state.back_odds` |
| `lay_odds` | `workflow_state.lay_odds` |
| `commission` | `workflow_state.commission` (default 0) |
| `lay_stake` | `workflow_state.lay_stake` |
| `liability` | `workflow_state.liability` |
| `profit` | Leave unset (pending settlement) |
| `notes` | "Pending — Matchbook offer #[offer_id]" |

### From manual input

If running standalone (not from workflow), extract fields from the user's message or screenshot:
- **Required**: Bookmaker, Sport, Event, Match Date, Offer Type, Back Stake, Back Odds, Lay Odds
- **Optional / defaultable**: Date (placed), Exchange, Commission %, Lay Stake, Liability, Notes

If any required field is missing or ambiguous, ask before writing. If lay stake / liability aren't already known, calculate them first (see `references/formulas.md`) rather than leaving them at 0.

### Confirm before writing

```
Ready to log:
• Date: 2026-06-07
• Bookmaker: Bet365
• Exchange: Matchbook
• Sport: Football
• Event: Arsenal vs Man City — BTTS
• Match Date: 2026-06-08
• Offer Type: Free Bet
• Back Stake: £10.00
• Back Odds: 2.10
• Lay Odds: 2.14
• Commission: 0%
• Lay Stake: £9.81
• Liability: £11.19
• Notes: Pending — Matchbook offer #413177013410013

Shall I add this?
```

### Write via MCP

Call `log_bet` with the fields above. It returns the inserted row as JSON, including its `id` — mention that id when confirming to the user (needed later to settle the bet).

If the tool result contains an `"error"` key instead of a row, report the error to the user rather than treating it as success (e.g. `matched-betting-tracker` MCP not connected, or a missing required field).

Confirm to the user which bet was logged, including its id.

## Settling a Bet

1. Ask for (or extract): Event name + Profit amount (and Date placed / Bookmaker if useful for disambiguation).
2. Call `query_bets` with `settled="false"` (and `sport`/`bookmaker`/`date_from`/`date_to` filters if known) to find candidate bets.
3. Match the target bet by its `event` field (and date/bookmaker) in the results. If more than one match, list them and ask the user which `id` to settle.
4. Confirm which bet (id + event) was found before editing.
5. Call `update_bet` with `id`, `profit`, `settled=true`, and optionally `notes="Settled"`.

If the tool result contains an `"error"` key, report it to the user (e.g. no bet found with that id).

## Field Extraction from Screenshots

| Screenshot type | Look for |
|-----------------|----------|
| Bookmaker       | Stake → Back Stake; Odds → Back Odds; Event/match → Event; Sport type → Sport; Branding → Bookmaker; Promo label → Offer Type |
| Back bet slip   | Confirmed stake, odds, selection — verify against planned values |

## Common Offer Types

- **Risk-Free** — stake returned as free bet if it loses
- **Free Bet** — no stake returned on win (SNR)
- **Enhanced Odds** — boosted odds on a specific selection
- **Reload** — ongoing deposit bonus
- **Acca Insurance** — refund if one leg of accumulator loses
- **Bet Builder** — combined selection within one match
