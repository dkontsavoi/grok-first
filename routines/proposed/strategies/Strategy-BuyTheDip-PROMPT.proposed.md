# Strategy — BUY_DIP (4h) — PROPOSED

> Not live. Invoked by Main orchestrator when regime favors crowding / ATR dips, or run as standalone 4h module.

| | |
|---|---|
| **Strategy ID** | `BUY_DIP` |
| **Cadence** | Every 4h (Europe/Madrid) when selected |
| **Universe** | Restricted list only; long-only; Revolut X spot |

## When Main should select this

- Flush:off **or** Flush cooling with reclaim
- F&G not extreme-greed chase **or** dips are true ATR washes
- Crowding elevated/cooling on `CROWD-DIP` names
- Quiet liq tape: size down; require deeper zones / more confluence

## Hard constraints

- Never short; never outside: `BTC ETH ARB HYPE PUMP ENA SKY JUP AAVE AERO LDO UNI RAY POL SYRUP ZRO ICP DASH LINK CRV MORPHO SUI ZEN NEAR SOL VVV`
- Buy-zone ATR-aware by tier (A BTC/ETH ~1–2%; B ~2–4%; C ~3–6% smaller $)
- Always invalidation; no strength-chase into top slots
- Pair every entry idea with sell-in-strength exits (TP1 ~1R 30–40%; TP2 ~2R 30–40%; trail / 24–48h time-stop; SL at inv)

## Data (same stack as Main)

CMC quotes/TA; RevX book; Binance www top-trader **POSITION** L/S (+ funding/OI/basis); Xoomar/ByKaranteli liqs.

## Bucket focus

Primary: `CROWD-DIP` — BTC ETH AAVE SKY LINK ARB AERO (+ LDO/SUI when crowd-level).  
Prefer pos L/S high **and cooling**. Block if δ↑ hard near highs.  
Don't-fade UNI ZRO VVV ICP — never demote solely for high LSR.  
Hard-skip extreme LSR on mild names (SYRUP-class ~3–4+).  
`MOM` names (JUP HYPE NEAR RAY): **out of scope** for this module — hand to `MOM` strategy.

## Flush interaction

If Flush:ON → this module yields to `FLUSH_MR` (or only BTC/ETH with deep zones). No Tier-C.

## Deliverable (module block for Main compile)

1. Strategy tag + one-line why it fits regime  
2. Top **4–6** `BUY_DIP` ideas priority-ranked with zone, inv, size hint, LSR/funding note  
3. Exit plan per idea / per open inventory relevant to dips  
4. Skips with reasons  

## Night auto (if Main selects BUY_DIP as primary)

Inherit Main night caps: ≤$1000/run, cumulative night ≤$1500–2000, ≤3 new symbols, 2-rung ladders, inventory sells only. Flush:ON → no Tier-C.
