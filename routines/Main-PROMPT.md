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

## Bucket tags (required on every top-6)

Tag each top-6 name as exactly one of:

| Tag | Meaning | Names / rule |
|-----|---------|--------------|
| `CROWD-DIP` | Classic crowding / contrarian dip | BTC, ETH, AAVE, SKY, LINK, ARB, AERO; LDO/SUI when treating as crowd-level |
| `MOM` | δ-momentum continuation — **not** a vanilla fade-dip | JUP, HYPE, NEAR, RAY; LDO/SUI when δ-leg dominates |
| `WATCH` | Allow only with extra confluence (funding + liq + BTC stable); do not dominate priority | ENA, MORPHO, POL, CRV, DASH, ZEN, and other low-signal names |

**Don't-fade set** (never demote solely for high pos L/S): `UNI`, `ZRO`, `VVV`, `ICP`.

**Hard rules:**

- Do **not** pad `MOM` names into DIP slots as if they were crowding fades.
- `CROWD-DIP`: prefer when pos L/S is elevated **and cooling** (δ ≤ 0). **Block** if δ rising hard while price is near local highs (strength chase).
- `MOM`: only in top-6 if (a) true ≥ATR wash from local high, **or** (b) tagged continuation-long (pullback-to-EMA / breakout hold) — never a −1.5% “dip” off strength.
- Mild crowding (PUMP, SYRUP, SOL): allow dips but **hard-skip** extreme pos LSR (calibrate ~3–4+; SYRUP ~5.6 class = skip). Log `orders_skipped_reason`.
- Funding × crowd on ETH/SKY/BTC/AAVE: elevated funding + extreme LSR → widen zone / cut size; negative funding or post-long-flush → allow tighter mid.

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

## Complementary long-only modes (tags only — dip remains core)

Main may **tag** (not replace) ideas as:

- `FLUSH-MR` — buy only after Flush:ON + long-liq spike; deeper zones; BTC/ETH first
- `MOM` — continuation on δ-set only (see buckets)
- `RS-DIP` — dips in names with strong ~7d RS vs BTC when Flush:off
- `CORE-DCA` — small BTC/ETH scheduled / mild ATR bids separate from alt lottery

Do **not** run funding-carry or generic whole-list breakouts as strategies.

## Deliverable (one cohesive brief)

NEXT ~4 HOURS (as-of Europe/Madrid + UTC). **Do NOT** split Binance into a separate thread.

1. BTC & ETH bull/bear/range — ranges + probs = 100% each.
2. Top **6** ideas, **priority 1–6**, each with bucket tag (`CROWD-DIP` / `MOM` / `WATCH`). No padding. Cap roughly ≤4 crowding-dips + ≤2 MOM unless book forces otherwise.
3. Open book (RevX) + SUGGESTIONS with concrete limit prices and USD sizes for new buys **and** paired exit/TP sells on inventory.
4. Binance derivatives block in the **same** brief.
5. Liquidations one-liner.
6. CMC backdrop one-liner.
7. Night heat line when in night window: cumulative placed tonight vs cap; Flush gate status.

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

`routine="Main"`, `as_of_madrid`, btc/eth spot, probs, ranges, flush_detector, top6 (with `bucket` per name), `night_auto`, `orders_placed` / `orders_cancelled`, plus:

- `lsr`, `funding`, `atr_pct` when used for a name
- `orders_skipped_reason`
- `exits_suggested` (TP/SL/time-stop)
- `night_cumulative_placed_usd`

## Style

Concise, trader-useful. Always notify. If CMC auth fails repeatedly, say so and pause this routine.
