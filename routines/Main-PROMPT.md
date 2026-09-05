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

`revx` CLI for LIVE book (balances, open orders, held/open tickers). If auth fails, say so and still deliver the market brief.

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

## Tactics rules

Use Binance top-trader POSITION dynamics in tactics (entries/leaves/cancels/ladders), plus funding/basis/premium/liqs.

| Rule | Names |
|------|--------|
| Crowding caution | ETH, SKY, BTC, AAVE (LINK/ARB on 4h, AERO extremes) |
| Do **not** fade high L/S | UNI, ZRO, VVV, ICP |
| δ-momentum (hint) | JUP, HYPE, NEAR, LDO, RAYSOL, SUI |

Dip zones: buy-zone midpoint ~≥1.5–3% below spot; always include invalidation.

## Deliverable (one cohesive brief)

NEXT ~4 HOURS (as-of Europe/Madrid + UTC). **Do NOT** split Binance into a separate thread.

1. BTC & ETH bull/bear/range — ranges + probs = 100% each.
2. Top **6** buy-the-dip, **priority 1–6**. No padding.
3. Open book (RevX) + SUGGESTIONS with concrete limit prices and USD sizes for new buys.
4. Binance derivatives block in the **same** brief.
5. Liquidations one-liner.
6. CMC backdrop one-liner.

## Night auto-execute (standing approval)

Local **Europe/Madrid** hour at run time:

| Window | Hours (Madrid) | Action |
|--------|----------------|--------|
| **Night** | **22:00–08:00** (22:00 inclusive → 08:00 exclusive) | Auto-execute (below) |
| **Day** | 08:00–21:59 | **Suggest only** — no place/cancel/modify unless later chat confirmation |

### Night rules (hard)

- **Place** suggested new **buy limit** orders from the brief via `revx` (limit buys only; never market buys; never shorts / sells-to-open).
- **Cap:** sum of newly placed buy quote amounts in **this run** ≤ **$1000 USD**. If over, place highest-priority dips first; skip rest and report skips.
- Prefer ~$100–$200 rungs unless suggestion names otherwise (still under $1000).
- **Cancels** of open buy bids this brief marks cancel/replace are allowed at night (do not count against the $1000 place cap). No cancel-all.
- Report exactly what was placed/cancelled (prices, sizes, ids) in the same notification.
- On revx auth/place/cancel error: report; at most one careful retry.

### Day rules

Suggest only. Do **not** place/cancel/modify unless explicit user confirmation in chat.

## Logging

Append JSONL to `/workspace/crypto-self-reflect/briefs.jsonl`:

`routine="Main"`, `as_of_madrid`, btc/eth spot, probs, ranges, flush_detector, top6, `night_auto`, `orders_placed` / `orders_cancelled`.

## Style

Concise, trader-useful. Always notify. If CMC auth fails repeatedly, say so and pause this routine.
