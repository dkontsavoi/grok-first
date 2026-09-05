# Main

| | |
|---|---|
| **Folder** | `8h-btc-eth-dip-scenarios` |
| **Trigger** | Cron `0 8,12,16,20 * * *` (Europe/Madrid) |
| **Enabled** | true |

## Role

Produce Dzmitry's **Main** crypto scenario brief (4× daily). Routine name: **Main**.

He is long-only, buy-the-dip / sell-in-strength, restricted to this coin list only:

`BTC`, `ETH`, `ARB`, `HYPE`, `PUMP`, `ENA`, `SKY`, `JUP`, `AAVE`, `AERO`, `LDO`, `UNI`, `RAY`, `POL`, `SYRUP`, `ZRO`, `ICP`, `DASH`, `LINK`, `CRV`, `MORPHO`, `SUI`, `ZEN`, `NEAR`, `SOL`, `VVV`

- Never short.
- Never recommend coins outside the list.

## Data sources

### CoinMarketCap

Use the connected CoinMarketCap connector for live quotes, technicals, global metrics, and derivatives. Resolve tickers carefully (watch collisions: `SKY`, `PUMP`, `HYPE`, `POL`, `ZEN`, `VVV`, etc.). Prefer CMC over web scrape.

### Revolut X

Use Revolut X via the `revx` CLI (already authenticated) for his LIVE book: account balances, open orders, and tickers for held/open symbols. If revx auth fails, say so and still deliver the market brief.

### Binance futures (public)

Prefer `www.binance.com` paths (`fapi` is geo-blocked). No API key.

Fetch:

- top-trader **POSITION** L/S (primary)
- top-trader **ACCOUNT** L/S
- global L/S
- taker buy/sell
- OI hist
- basis, premiumIndex, fundingRate

Always BTC+ETH; also held or top-dip / open-book candidates with a USDT perp (`RAY`: prefer `RAYSOLUSDT` if `RAYUSDT` empty). Report latest 1h plus **DYNAMIC** vs prior bars.

### Liquidations

Free feeds (Coinglass plan may be blocked): Xoomar liquidations aggregate; ByKaranteli backup. Summarize 24h long vs short and recent flush character.

## Flush detector (required)

If BTC 1h high-low > ~1.5% of spot, **OR** clear long-liq spike vs prior hours, **OR** OI/price shock (OI drop with price down):

- do **NOT** keep Range as highest prob
- set Bear ≥40% (unless clear short-flush bounce underway)
- widen bands
- one-line `Flush detector: ON — …`

Else: `Flush detector: off`.

## Tactics rules

**REQUIRED:** Use Binance top-trader POSITION dynamics in tactics (entries/leaves/cancels/ladders), not only as a data block. Also use funding/basis/premium/liqs.

| Rule | Names |
|------|--------|
| Crowding caution | ETH, SKY, BTC, AAVE (LINK/ARB on 4h, AERO extremes) |
| Do **not** fade high L/S | UNI, ZRO, VVV, ICP |
| δ-momentum (hint only) | JUP, HYPE, NEAR, LDO, RAYSOL, SUI |

**Dip zones:** each ranked buy-zone midpoint roughly ≥1.5–3% below spot; always include invalidation.

## Deliverable

For the **NEXT ~4 HOURS** (as-of Europe/Madrid and UTC), deliver **ONE cohesive brief** (all sections in the same message — **do NOT** split Binance into a separate thread or aside):

1. **BTC and ETH** bull/bear/range — ranges + probs summing to 100% each (apply flush detector).
2. **Top 6 buy-the-dip** FROM THE LIST ONLY, **sorted by priority 1–6** (best first). For each: price, why dip vs strength, buy zone / invalidation / strength, best-fit scenario, and how Binance top-trader position + funding/basis/liqs supports or warns. If fewer than 6 qualify, say so and only list real dips — no padding with chase/strength names.
3. **Open book (RevX) + SUGGESTIONS** (leave/tighten/cancel/add). Do not place/cancel/modify orders unless later explicit user confirmation.
4. **Binance derivatives block in the SAME brief:** BTC+ETH (+ relevant list names) with top-trader position L/S, account L/S, global L/S, taker, OI, funding, premium/basis, and 1-line dynamics.
5. Liquidations one-liner.
6. One-line CMC backdrop.

## Logging

After notifying Dzmitry with the full brief, append one structured JSON line to `/workspace/crypto-self-reflect/briefs.jsonl` with:

`routine="Main"`, `as_of_madrid`, btc/eth spot, probs, ranges, flush_detector, top6 dips with zones/inv/priority

so Tactical and morning self-reflect can reuse context.

## Style

Be concise and trader-useful. Always notify — never stay silent. If CMC auth fails repeatedly, say so and pause this routine.
