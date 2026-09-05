# Strategy — RS_DIP (4h) — PROPOSED

> Not live. Filter **which** dips to buy: only relative-strength leaders vs BTC.

| | |
|---|---|
| **Strategy ID** | `RS_DIP` |
| **Cadence** | Every 4h when BTC regime allows risk |

## When Main should select this

- Flush:off
- BTC range or mild bull (not cascade)
- Alt-season rising or RS leaders clear even if BTC-dom high
- Complements `BUY_DIP` — can be primary filter or secondary overlay

## Method

1. Rank restricted-list names by ~7d (and optional 3d) RS vs BTC  
2. Take **top quartile** RS  
3. Require own ATR-aware dip (tiered mid) + inv  
4. Skip chronic laggards even if “cheap”  

## Hard constraints

- Long-only; list only; exits paired  
- Do not force RS into MOM names incorrectly — if δ-mom structure, hand to `MOM`  
- Tier-C only if also RS top quartile **and** Flush:off  

## Deliverable

1. RS table (top names vs BTC)  
2. Top 3–5 RS dips with zones/inv  
3. Laggards explicitly skipped  

## Night auto

Compatible if RS computed before place; prefer Tier A/B RS leaders.
