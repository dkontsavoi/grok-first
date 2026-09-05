# Connectors & data sources (names only)

Used by First for crypto briefs / tactics:

| Service | Role | Notes |
|---------|------|--------|
| CoinMarketCap MCP | Spot quotes, TA, global metrics | Connected |
| Revolut X CLI (`revx`) | Live book / orders | Auth on box; never commit keys |
| Binance public `www.binance.com` futures data | Top-trader L/S, OI, funding, basis | `fapi` geo-blocked from box |
| Binance MCP | Installed; live calls geo-blocked | Read-only intent |
| Xoomar / ByKaranteli | Free liquidations / OI | No key |
| CoinGlass | Key saved; API returns Upgrade plan until Hobbyist+ | Do not commit key |

## Self-reflect artifacts (local box paths)
- `/workspace/crypto-self-reflect/scores.jsonl`
- `/workspace/crypto-self-reflect/briefs.jsonl`
