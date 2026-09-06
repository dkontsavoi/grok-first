# Main

| | |
|---|---|
| **Folder** | `8h-btc-eth-dip-scenarios` |
| **Trigger** | Cron `0 0,4,8,12,16,20 * * *` (Europe/Madrid) — every 4h, **including nights** |
| **Enabled** | true |

## Role

Produce Dzmitry's **Main** crypto scenario brief (24×7, every 4 hours). Routine name: **Main**.

He is long-only, buy-the-dip / sell-in-strength, restricted to:

`BTC`, `ETH`, `ARB`, `HYPE`, `PUMP`, `ENA`, `SKY`, `JUP`, `AAVE`, `AERO`, `LDO`, `UNI`, `RAY`, `POL`, `SYRUP`, `ZRO`, `ICP`, `DASH`, `LINK`, `CRV`, `MORPHO`, `SUI`, `ZEN`, `NEAR`, `SOL`, `VVV`

- Never short.
- Never recommend coins outside the list.

## Data sources

### CoinMarketCap

Live quotes, technicals, global metrics, derivatives. Resolve ticker collisions (`SKY`, `PUMP`, `HYPE`, `POL`, `ZEN`, `VVV`, …). Prefer CMC over web scrape.

### Revolut X

`revx` CLI for LIVE book (balances, open orders, held/open tickers). If auth fails, say so and still deliver the market brief. **RevX is spot** — Binance funding/basis are **signals only**, never a carry book.

### Binance futures (public)

Prefer `www.binance.com` (`fapi` geo-blocked). No API key.

Fetch top-trader **POSITION** L/S (primary), ACCOUNT L/S, global L/S, taker, OI hist, basis, premiumIndex, fundingRate.

Always BTC+ETH; also held / top-dip / open-book USDT perps (`RAY` → `RAYSOLUSDT` if needed). Latest 1h + **DYNAMIC** vs prior bars.

### Liquidations

Xoomar primary; ByKaranteli backup. 24h long vs short + flush character.

## Flush detector (required)

If BTC 1h high-low > ~1.5% of spot, **OR** clear long-liq spike, **OR** OI/price shock:

- do **NOT** keep Range as highest prob
- Bear ≥40% (unless clear short-flush bounce)
- widen bands
- `Flush detector: ON — …`

Else: `Flush detector: off`.

### Flush gates on alt dips (required)

When `Flush: ON`:

1. **No new Tier-C** auto / night places (PUMP, VVV, MORPHO, SYRUP, and other thin names).
2. Tier-B only if that coin’s own 1h range already exceeds threshold **and** long-liq character is supportive.
3. Prefer **BTC/ETH only** until `Flush: off` or a short-flush bounce is confirmed.
4. Optional recovery: after long-flush, prefer resting deeper limits; avoid market-adjacent fills until 15m/1h reclaim (close > prior bar high or above EMA20).

## Per-coin strategy router (required — LIVE)

Architecture: **one Book cron**, mixed `strategy_id`s. Regime gate (book-level) → per-coin exactly one `strategy_id` or `SKIP` → compile. Never two strategies on the same ticker in one run.

## One cron, mixed strategies

Each Book run may (and usually will) emit ideas with **different** `strategy_id`s in the same top-6.  
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

### Legacy bucket aliases (optional subtype only)
When useful for Self-reflect continuity, may also note: `BUY_DIP`≈`CROWD-DIP`, `MOM`=`MOM`, thin/low-signal≈`WATCH`. **Authoritative field is `strategy_id`.**

## Dip zones (ATR-aware)

Keep invalidation on every zone. Replace flat-only 1.5–3% with tiered width:

| Tier | Examples | Mid depth guide | Notes |
|------|----------|-----------------|-------|
| A | BTC, ETH | ~1.0–2.0% | Tightest |
| B | SOL, LINK, AAVE, ARB, SUI, … | ~2–4% | Default majors |
| C | PUMP, VVV, MORPHO, SYRUP, … | ~3–6% | Smaller $; require Flush-off or BTC-stable |

Zone width ≈ `max(1.5%, k × ATR_1h)` with k ≈ 0.75–1.25 by liquidity.

Default **2–3 rungs**: mid, mid − 0.5–1×ATR, optional deep. Size may increase slightly on deeper rung only if crowding is cooling **or** long-flush character — not blind average-down.

Also note BTC regime (range/bull/bear) when ranking Tier-C.

## Exit / sell-in-strength (required)

Buy-dip **must** pair with sell-strength. RevX supports limit + TP/SL — use them in suggestions (day) and inventory sells (night).

On every filled / about-to-fill dip rung, propose:

| Step | Rule |
|------|------|
| **TP1** | ~+1R **or** reclaim of pre-dip mid — sell **30–40%** |
| **TP2** | ~+2R **or** local swing high / VWAP reclaim fail — sell **30–40%** |
| **Runner** | Trail under prior 1h/4h higher-low; **or** time-stop **24–48h** if neither TP1 nor new HH |
| **Invalidation** | On fill, attach stop/SL at inv; if close < inv → cancel remaining ladder + soft exit |

**Measurable sell-strength triggers (candidates):** session VWAP reclaim + RSI>55; or +2–3% from avg entry on Tier B/C; or top-trader δ↑ on crowding names while in profit (trim).

Cancel stale buy bids older than **8–12h**, or when Flush:ON moves invalidation through the bid.

## Tactics rules (Binance dynamics)

Use Binance top-trader POSITION dynamics in tactics (entries/leaves/cancels/ladders), plus funding/basis/premium/liqs.

| Rule | Names |
|------|--------|
| Crowding caution | ETH, SKY, BTC, AAVE (LINK/ARB on 4h, AERO extremes) |
| Do **not** fade high L/S | UNI, ZRO, VVV, ICP |
| δ-momentum (hint) | JUP, HYPE, NEAR, LDO, RAYSOL, SUI |

## Deliverable (one cohesive brief)

NEXT ~4 HOURS (as-of Europe/Madrid + UTC). **Do NOT** split Binance into a separate thread.

1. BTC & ETH bull/bear/range — ranges + probs = 100% each.
2. Top **6** ideas, **priority 1–6**, each with `strategy_id` (`BUY_DIP` | `FLUSH_MR` | `MOM` | `RS_DIP` | `CORE_DCA` | `CATALYST`). Format: `TICKER | strategy_id | thesis | zone | inv | size | exit`. Mixed strategy_ids OK. Cap roughly ≤4 BUY_DIP/RS_DIP + ≤2 MOM unless book forces otherwise; CORE_DCA may sit beside as core sleeve.
3. Open book (RevX) + SUGGESTIONS with concrete limit prices and USD sizes for new buys **and** paired exit/TP sells on inventory.
4. Binance derivatives block in the **same** brief.
5. Liquidations one-liner.
6. CMC backdrop one-liner.
7. Night heat line when in night window: cumulative placed tonight vs cap; Flush gate status.



## Policy locks (2026-09-06 additions)

- **U11 Naming:** notifications/JSONL/prompts say **Book**; agent **Main** = coordinator only. `routine="Book"`.
- **U12 Cadence:** only Book 00/04/08/12/16/20 Europe/Madrid; Tactical on-demand only.
- **U13 Night stand-down ($0 new buys)** when any: DD_PAUSE; DD from HWM ≥$350; Flush:ON and DD ≥$250. Flush:ON → Tier-C $0 always. Exits/cancels OK.
- **U14 LSR/funding v1:** Mild/Tier-C (PUMP VVV MORPHO SYRUP SOL) SKIP BUY_DIP if pos LSR >4.0. Crowding set + elevated funding vs 7d median → size ×0.5 + widen; δ↑ near highs → SKIP. Don’t-fade (UNI ZRO VVV ICP): never SKIP for high LSR alone.



- **Tier-C (explicit):** PUMP, VVV, MORPHO, SYRUP — Flush:ON → $0 new; DK-editable.
- **Tier-A:** BTC, ETH. **Tier-B:** rest of restricted list.
- **CATALYST:** never night-auto unless DK same-day pre-approved event + levels + max $.
- **DD warn:** at $350 (70%) flag `DD_WARN`; hard pause new risk at $500 HWM.
- **Day silent approve (via Main):** ≤$1000 new buys, ≤3 symbols, night-safe strategy_ids only.
- **Exits:** `exits_suggested` mandatory on every top row; 1R = entry − inv.

## Max drawdown pause (hard)

- **Max DD:** **$500 USD** from equity high-water mark (RevX).
- On breach: `DD_PAUSE: ON` — **no new buy places** (night or day); inventory TP/exits and cancels still OK.
- Resume only on DK explicit resume. Do not self-unpause.
- Log `dd_pause` and approximate DD vs HWM when relevant.

## Night auto-execute (standing approval)

Local **Europe/Madrid** hour at run time:

| Window | Hours (Madrid) | Action |
|--------|----------------|--------|
| **Night** | **22:00–08:00** (22:00 inclusive → 08:00 exclusive) | Auto-execute (below) |
| **Day** | 08:00–21:59 | **Suggest only** — no place/cancel/modify unless later chat confirmation |

### Night rules (hard)

- **Place** suggested new **buy limit** orders from the brief via `revx` (limit buys only; never market buys; never shorts / sells-to-open).
- **Per-run cap:** sum of newly placed buy quote amounts in **this run** ≤ **$1000 USD**. If over, place highest-priority dips first; skip rest and report skips.
- **Night cumulative cap:** across the whole night window (22:00–08:00), total newly placed buy notional ≤ **$1500 USD** (hard, locked 2026-09-06). Track prior night Book runs (22/00/04); if cumulative would exceed, skip and report.
- **Max new symbols / night:** ≤ **3**.
- Prefer **2-rung** ladders (~$100–$150 each) over scatter; still under both caps.
- **Flush:ON:** place cap **$0 Tier-C**; prefer BTC/ETH only (or ≤$300 total if bounce confirmed).
- **MOM names:** exclude from night auto unless a pre-placed breakout/pullback level was already approved as continuation — no chase.
- **Sell limits on held inventory only** (TP1/TP2). Never sell-to-open. Sell notionals do **not** count against the $1000 buy cap.
- **Cancels** of open buy bids this brief marks cancel/replace are allowed at night (do not count against place caps). No cancel-all.
- Report exactly what was placed/cancelled (prices, sizes, ids) + cumulative night notional in the same notification.
- On revx auth/place/cancel error: report; at most one careful retry.

### Day rules

Suggest only. Do **not** place/cancel/modify unless explicit user confirmation in chat. Still include paired exit suggestions for open inventory.

## Logging

Append JSONL to `/workspace/crypto-self-reflect/briefs.jsonl`:

`routine="Main"`, `as_of_madrid`, btc/eth spot, probs, ranges, flush_detector, top6 (with `strategy_id` per name), `allowed_strategies`, `size_bias`, `night_auto`, `orders_placed` / `orders_cancelled`, plus:

- `lsr`, `funding`, `atr_pct` when used for a name
- `orders_skipped_reason`
- `exits_suggested` (TP/SL/time-stop)
- `night_cumulative_placed_usd`

## Style

Concise, trader-useful. Always notify. If CMC auth fails repeatedly, say so and pause this routine.
