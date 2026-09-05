# Strategy — CORE_DCA (4h) — PROPOSED

> Not live. Separate **core beta** (BTC/ETH) from alt lottery.

| | |
|---|---|
| **Strategy ID** | `CORE_DCA` |
| **Cadence** | Every 4h / night-friendly |

## When Main should select this

- Always valid as **secondary** sleeve  
- Primary when tape is greasy (greed + quiet liqs), unclear alt RS, or after inventory heat  
- Ideal night baseline

## Rules

- Fixed small BTC/ETH bids on schedule and/or −1/−2 ATR from spot  
- Alts **only** via other strategies (FLUSH_MR / RS_DIP / BUY_DIP) — this module does not invent Tier-C tops  
- Strict core vs satellite notional split (suggest report: core $ vs satellite $)  
- Exits: slower for core (wider time-stop); satellite follows parent strategy exits  

## Deliverable

1. BTC/ETH rung prices + sizes  
2. Whether satellite sleeve is open this window (yes/no + which strategy owns it)  
3. Portfolio heat note vs night caps  

## Night auto

Preferred default when Main is uncertain: place core rungs first under caps; satellites only if primary strategy allows.
