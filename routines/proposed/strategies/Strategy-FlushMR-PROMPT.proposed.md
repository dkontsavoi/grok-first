# Strategy — FLUSH_MR (4h) — PROPOSED

> Not live. Mean-reversion **only** after dislocation. Best night use-case when sized small.

| | |
|---|---|
| **Strategy ID** | `FLUSH_MR` |
| **Cadence** | Every 4h when Flush:ON or recently ON; standby otherwise |
| **Universe** | Restricted list; long-only; RevX spot |

## When Main should select this

- Flush detector ON: BTC 1h range >~1.5% **OR** long-liq spike **OR** OI/price shock
- Long-liq >> short-liq (exhaustion), not grind-lower with short squeeze character
- Prefer after cascade once tape stabilizes (optional 15m/1h reclaim for aggressive fills)

## Hard constraints

- Deeper zones than quiet `BUY_DIP` (extra 0.5–1×ATR)
- BTC/ETH first; Tier-B only with own wash + long-liq character; **no Tier-C** until Flush:off
- Strict invalidation under cascade low / prior 1h low
- Size cut vs normal night; inventory sells paired

## Data emphasis

Liquidations (Xoomar/ByKaranteli) + OI shock + BTC 1h range; Binance pos L/S secondary for which alts to allow after BTC/ETH.

## Deliverable

1. Flush evidence (range %, liq L/S, OI note)  
2. Ordered bids: BTC/ETH deep rungs → optional Tier-B  
3. What **not** to buy (Tier-C, chase reclaim without wash)  
4. Exit: faster TP1 on bounce (partial), wider inv if cascade extends  

## Night auto

Compatible but **stricter**: cumulative smaller; Flush:ON Tier-C = $0; prefer 2 rungs BTC/ETH only until reclaim.
