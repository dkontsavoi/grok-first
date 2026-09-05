# Strategy — MOM (4h) — PROPOSED

> Not live. δ-momentum / continuation on approved names — **not** fade-the-crowding dips.

| | |
|---|---|
| **Strategy ID** | `MOM` |
| **Cadence** | Every 4h when selected |
| **Approved names** | Primary: `JUP HYPE NEAR RAY`; dual: `LDO SUI` when δ-leg dominates. Optional watch: ZEN if 4h stabilizes |

## When Main should select this

- Rising top-trader pos L/S δ>0 **and** price HH or hold above prior breakout / EMA
- Risk-on tape without Flush:ON wash
- **Avoid** as primary when F&G extreme greed + quiet liqs (false breakout risk) unless levels already held

## Hard constraints

- Long-only; restricted list; never apply MOM logic to ETH/SKY/BTC/AAVE crowding set as if breakout-fade
- Entries: pullback-to-EMA / breakout hold / ≥ATR wash then reclaim — **not** −1.5% off highs labeled “dip”
- Trail exits; time-stop if extension fails; no blind average-down
- Night auto: **exclude chase**; only pre-placed pullback/breakout limits if Main explicitly allows

## Data emphasis

Binance POSITION δ (1h/4h), funding not extreme adverse, CMC TA structure, RevX book for held MOM names.

## Deliverable

1. MOM candidates with structure (level, invalidation under swing low)  
2. What would flip to stand-down (δ flip, Flush:ON, failed hold)  
3. At most 1–3 ideas; do not pad  
4. Trail / TP plan  

## Conflicts

Do not compete with `BUY_DIP` for the same name in the same brief — Main assigns one tag.
