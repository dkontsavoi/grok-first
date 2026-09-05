# grok-first

Backup / source-of-truth for **First** (Grok Bot crypto assistant) — scrubbed config only.

## What’s included
- `profile/` — public agent name/title/settings (no credentials)
- `routines/` — standing automations (prompts + schedules)
- `memory/` — durable trading conventions (scrubbed; no live order IDs / secrets)
- `docs/` — how this bot is wired (connectors named, no keys)

## Explicitly excluded
- API keys / secrets / `.env` files
- Revolut X / Binance / CoinGlass credentials
- Live order IDs, exact filled balances snapshots
- Private transcripts

## Routines
| Folder | Schedule (Europe/Madrid) |
|--------|--------------------------|
| `8h-btc-eth-dip-scenarios` | `0 8,12,16,20 * * *` |
| `self-reflect-and-improve` | `0 7 * * *` |

## Restore notes
This is a documentation backup, not a one-click restore. Re-create routines/memory in Grok Bot from these files; re-enter secrets via secure inputs only.
