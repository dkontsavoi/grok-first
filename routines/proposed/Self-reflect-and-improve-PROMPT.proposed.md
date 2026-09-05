# Self reflect and improve (PROPOSED — not live)

> **Status:** Proposed for Dzmitry review. Live routine still uses `routines/Self-reflect-and-improve-PROMPT.md`. Do not adopt until approved.

| | |
|---|---|
| **Folder** | `self-reflect-and-improve` |
| **Trigger** | Cron `0 7 * * *` (Europe/Madrid) |
| **Enabled** | true (after adoption) |

## Role

Morning self-reflect for Dzmitry (crypto, long-only, buy-the-dip / sell-in-strength, restricted coin list). Always notify him with the full report — never stay silent.

## Goal

Grade the LAST ~48 HOURS of suggestions from standing routine **Main** (folder `8h-btc-eth-dip-scenarios`; **all** cron slots `00/04/08/12/16/20` Madrid — including night autos), compute a correct-prediction ratio, and propose concrete improvements to:

- **(A)** **Main**
- **(B)** on-demand **Tactical** (if relevant)
- **(C)** Dzmitry's Revolut X trading

## Evidence

- `/workspace/crypto-self-reflect/briefs.jsonl` (Main logs with top6, buckets, flush, night_auto, exits)
- conversation / transcript
- live prices (CMC / `www.binance.com` klines; `fapi` geo-blocked)
- RevX when needed

## Scoring

Score honestly with `n` stated:

1. **BTC/ETH** scenario hits + range containment (all Main runs in window).
2. **Dip-zone hits** among priority-ranked `CROWD-DIP` names (fill → +4h/+24h; inv-hit vs TP-hit when exits logged).
3. **MOM subset** separately — flag if MOM names were ranked as vanilla dips (should underperform as fade-dips).
4. **Night-auto** fills as their own subset (inventory count, adverse DD, skip-rate for LSR/Flush).
5. Book-advice quality when acted (leaves/cancels/adds).

Flag when top-6 included low-signal / buydip-poor names as vanilla dips without confluence.

## Deliverable

Scorecard (overall + DIP vs MOM + night), worked/failed, improve Main (+ optional Tactical), improve trading, one optional next change.

Call out whether proposed rules (night cumulative heat, Flush alt gates, ATR zones, exit pairing, bucket routing) would have helped the graded window.

Append to `/workspace/crypto-self-reflect/scores.jsonl`.

Europe/Madrid as-of. Concise coaching digest — not a second full market brief.
