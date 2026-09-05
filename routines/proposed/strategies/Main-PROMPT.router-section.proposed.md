# Main addendum — Per-coin strategy router (PROPOSED)

> **Status:** Proposed only. Do not replace live `routines/Main-PROMPT.md` until adopted.
> **Architecture:** One cron (Main every 4h). Strategy is **not** one-per-book; it is **one-per-coin**, then compiled.

## One cron, mixed strategies

Each Main run may (and usually will) emit ideas with **different** `strategy_id`s in the same top-6.  
Example valid brief: BTC=`CORE_DCA`, ETH=`BUY_DIP`, AAVE=`BUY_DIP`, SOL=`RS_DIP`, HYPE=skip.

Never assign two strategies to the same ticker in one run.

## Step A — Regime gate (book-level)

Compute Flush, F&G, liq character, OI/funding tape, BTC regime. Output:

- `allowed_strategies`: subset of `BUY_DIP | FLUSH_MR | MOM | RS_DIP | CORE_DCA | CATALYST`
- `size_bias`: normal | reduce | core-only
- `stand_down`: strategies blocked this window + one-line why

### Gate rubric (apply in order)

1. Flush:ON + long-liq exhaustion → allow `FLUSH_MR`, `CORE_DCA`; block Tier-C `BUY_DIP`; demote `MOM` chase.
2. Else greed + quiet liqs + soft OI → allow `CORE_DCA`, selective `BUY_DIP`, `RS_DIP`; demote chasey `MOM` and thin Tier-C.
3. Else risk-on, Flush:off, clear RS leaders → allow `RS_DIP`, `BUY_DIP`, conditional `MOM`.
4. Else quiet range → allow `BUY_DIP` + `CORE_DCA`.
5. `CATALYST` only with a verified event; never night-auto without prior approval.

## Step B — Per-coin router (name-level)

For each restricted-list name, assign **exactly one** `strategy_id` (or `SKIP`).

| Static class | Router |
|--------------|--------|
| Crowding: BTC ETH AAVE SKY LINK ARB AERO (+ LDO/SUI crowd-leg) | `BUY_DIP` if pos L/S elevated **and cooling** + ATR-tier wash; BTC/ETH may be `CORE_DCA` instead when size_bias is core-only / uncertain; **SKIP** if δ↑ hard near highs (strength chase) |
| δ-mom: JUP HYPE NEAR RAY (+ LDO/SUI δ-leg) | `MOM` if `MOM` allowed and δ>0 + HH/hold structure; else `BUY_DIP` only on ≥ATR wash **then reclaim**; else **SKIP**. Never shallow −1.5% fade labeled as dip |
| Don't-fade: UNI ZRO VVV ICP | `BUY_DIP` / book OK when zone+inv valid; never SKIP solely for high LSR |
| Mild / thin: PUMP SYRUP SOL MORPHO ENA … | `BUY_DIP` only with confluence + tier rules; hard-skip extreme LSR (SYRUP-class); prefer **SKIP** when size_bias is reduce/core-only |
| Event-tied | `CATALYST` only if allowed and event verified |

If regime gate **disallows** a strategy, that coin cannot receive it (reroute or SKIP).

Optional overlay: if `RS_DIP` allowed, prefer names in top-quartile 7d RS vs BTC when choosing among `BUY_DIP` candidates.

## Step C — Compile (same brief)

1. Regime gate summary + `allowed_strategies`
2. BTC & ETH scenarios (always)
3. Top **6** rows: `TICKER | strategy_id | thesis | zone | inv | size | exit` — mixed `strategy_id`s allowed
4. Open book suggestions consistent with each row’s strategy night/day rules
5. Binance + liqs + CMC in the same brief

Caps: roughly ≤4 `BUY_DIP`/`RS_DIP` + ≤2 `MOM` unless book forces otherwise; `CORE_DCA` rungs may sit beside top-6 as a core sleeve.

## Step D — Night auto

Place only rows whose `strategy_id` is night-safe under this gate:

- Usually safe: `CORE_DCA`, selective `BUY_DIP` / `RS_DIP` (tier rules)
- Usually unsafe: `MOM` chase, `CATALYST` without pre-approval, Tier-C when reduce/Flush
- Honor per-run $1000 + night cumulative + max 3 new symbols + inventory sells only

## Logging

Each top idea logs `strategy_id`. Brief logs `allowed_strategies`, `size_bias`.
