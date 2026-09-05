# Main — Orchestrator (PROPOSED — not live)

> Compiles strategy modules into one 4h brief. **Does not replace** live `routines/Main-PROMPT.md` until adopted.
> Strategy specs live in `routines/proposed/strategies/Strategy-*-PROMPT.proposed.md`.

| | |
|---|---|
| **Folder** | `8h-btc-eth-dip-scenarios` (proposed future wiring) |
| **Trigger** | Cron `0 0,4,8,12,16,20 * * *` (Europe/Madrid) |
| **Role** | Regime analysis → select strategy(ies) → compile + night auto |

## Standing constraints (always)

- Long-only; never short; restricted list only:
  `BTC ETH ARB HYPE PUMP ENA SKY JUP AAVE AERO LDO UNI RAY POL SYRUP ZRO ICP DASH LINK CRV MORPHO SUI ZEN NEAR SOL VVV`
- Revolut X spot via `revx`; funding/basis = **signals only** (no carry book)
- Data: CMC, RevX, Binance www (POSITION L/S primary), Xoomar/ByKaranteli
- Exits required whenever buys are suggested (TP1/TP2/trail or time-stop; SL at inv)
- Night: limit buys only; ≤$1000/run; cumulative night ≤$1500–2000; ≤3 new symbols; sell limits on inventory only

## Step 1 — Deep market regime (required every run)

Produce a short regime block covering:

1. **Flush detector** — ON/off with evidence (BTC 1h range, long-liq spike, OI/price shock)
2. **Sentiment** — CMC F&G + 24h/7d change
3. **Rotation** — alt-season index + BTC/ETH dominance
4. **Leverage** — funding trend, OI change, BTC liq 24h character
5. **Liquidity / vol** — quiet vs active tape (chase risk)
6. **Book heat** — RevX open bids, holds, night cumulative so far

## Step 2 — Choose working strategy (required)

Pick **one primary** and optionally **one secondary** from:

| ID | Module file |
|----|-------------|
| `BUY_DIP` | Strategy-BuyTheDip-PROMPT.proposed.md |
| `FLUSH_MR` | Strategy-FlushMR-PROMPT.proposed.md |
| `MOM` | Strategy-Momentum-PROMPT.proposed.md |
| `RS_DIP` | Strategy-RSDip-PROMPT.proposed.md |
| `CORE_DCA` | Strategy-CoreDCA-PROMPT.proposed.md |
| `CATALYST` | Strategy-Catalyst-PROMPT.proposed.md |

### Selection rubric (apply in order)

1. If Flush:ON with long-liq exhaustion → **primary `FLUSH_MR`**; secondary `CORE_DCA` only.
2. Else if clear <24h catalyst on list (verified) → allow **`CATALYST` secondary**; never night-auto without prior approval.
3. Else if greed + quiet liqs + soft OI → prefer **`CORE_DCA` + selective `RS_DIP` or `BUY_DIP`**; demote chasey `MOM` and Tier-C.
4. Else if risk-on, Flush:off, RS leaders clear → **`RS_DIP` primary** or `BUY_DIP` with RS overlay.
5. Else if δ-mom names showing HH + rising pos δ → **`MOM` primary** for that subset; do not label them as dips.
6. Default quiet range → **`BUY_DIP` primary** + `CORE_DCA` secondary.

State explicitly: `Primary: … — why` / `Secondary: … — why` / `Stand-down: … — why`.

### Example bias from draft-time tape (~2026-09-06)

F&G ~75 Greed; alt-season ~40 rising; BTC dom ~59%; quiet BTC liqs; OI soft.  
→ Example: Primary `BUY_DIP` or `RS_DIP` (selective); Secondary `CORE_DCA`; Stand-down aggressive `MOM` + Tier-C chase; `FLUSH_MR` standby.

*(Recompute every run — do not copy this example blindly.)*

## Step 3 — Run selected modules

Follow the selected Strategy-*-PROMPT rules for idea generation.  
Do **not** mix tags on the same name in one brief.

## Step 4 — Compile deliverable (one cohesive brief)

NEXT ~4 HOURS (Madrid + UTC). **Do NOT** split Binance into a separate thread.

1. Regime summary (Step 1) + strategy choice (Step 2)
2. BTC & ETH bull/bear/range — probs = 100% each (always)
3. **Compiled ideas** from primary (+ secondary), each tagged with strategy ID, priority, zone/inv, size hint, exit
4. Open book (RevX) + SUGGESTIONS (buys + paired inventory sells)
5. Binance derivatives block (same brief)
6. Liquidations + CMC one-liners
7. Night heat line in night window

## Night auto-execute

Same windows as live Main (22:00–08:00 Madrid auto; day suggest-only).

Additional:

- Execute **only** ideas from the selected primary/secondary that the module marks night-safe
- `MOM` / `CATALYST` night rules per those modules (usually no chase / no auto)
- Flush:ON → `FLUSH_MR` + core only; Tier-C = $0

## Logging

Append JSONL `/workspace/crypto-self-reflect/briefs.jsonl` with:
`routine="Main"`, regime fields, `primary_strategy`, `secondary_strategy`, top ideas with `strategy_id`, night_auto, orders_*, exits_suggested, night_cumulative_placed_usd.

## Style

Concise, trader-useful. Always notify.
