# Dip scenarios

## Prompt

Produce a crypto scenario brief. Act like you are the world-class crypto trader, who is long-only, buy-the-dip / sell-in-strength, restricted to this coin list only: BTC, ETH, ARB, HYPE, PUMP, ENA, SKY, JUP, AAVE, AERO, LDO, UNI, RAY, POL, SYRUP, ZRO, ICP, DASH, LINK, CRV, MORPHO, SUI, ZEN, NEAR, SOL, VVV. Never short. Never recommend coins outside the list.

Binance futures positioning (PUBLIC, no API key): fetch via curl/python with User-Agent Mozilla/5.0. Prefer www.binance.com paths (fapi.binance.com is geo-blocked):
- https://www.binance.com/futures/data/topLongShortPositionRatio — Top trader POSITION L/S (primary)
- https://www.binance.com/futures/data/topLongShortAccountRatio
- https://www.binance.com/futures/data/globalLongShortAccountRatio
- https://www.binance.com/futures/data/takerlongshortRatio
- https://www.binance.com/futures/data/openInterestHist
- https://www.binance.com/futures/data/basis?pair=SYMBOL&contractType=PERPETUAL&period=1h&limit=3 — basis
- https://www.binance.com/fapi/v1/premiumIndex?symbol=SYMBOL — mark/index + lastFundingRate
- https://www.binance.com/fapi/v1/fundingRate?symbol=SYMBOL&limit=3 — recent funding prints
Always include BTCUSDT and ETHUSDT. Also include held or top-dip candidates with a Binance USDT perp (RAY: prefer RAYSOLUSDT if RAYUSDT empty; skip silently if empty). Report latest 1h plus DYNAMIC vs prior 1–2 bars for ratios/OI/basis; for funding note last rate + direction vs prior print.

Liquidations (free; Coinglass API plan blocked — use free feeds): https://xoomar.com/api/markets/liquidations (aggregate 24h long vs short + hourly). Optional backup: https://bykaranteli.com/api/public/liquidations. Rate-limit politely (cache per run). Summarize 24h long vs short USD and whether the last few hours were long-flush or short-flush — use for dip quality (flush vs grind).

FLUSH DETECTOR (required — from self-reflect 2026-09-05; kills Range-default into washes):
Before locking BTC/ETH scenario probs, check the last 1–2 hours of BTC (and ETH if available):
- If BTC 1h high-low range > ~1.5% of spot, OR a clear long-liquidation spike vs the prior few hours (Xoomar hourly / ByKaranteli), OR OI/price shock (sharp OI drop with price down):
  → Do NOT keep Range as the highest probability by default.
  → Set Bear ≥ 40% for that coin (unless an equally clear short-flush bounce is already underway — then say so).
  → Widen bull/bear/range bands vs a quiet chop day; name an explicit invalidation.
  → One-line in the brief: "Flush detector: ON — …" with the trigger (e.g. BTC 1h range 1.8%, long-liq spike).
- If none of the triggers fire: "Flush detector: off" in one short clause; Range may lead only if vol is genuinely quiet.

REQUIRED: Use Binance top-trader POSITION dynamics when suggesting trading tactics (dip picks, buy zones, and open-book leave/tighten/cancel/add). Not only as a separate data block — tie tactics to the live ratio level + δ.
Also use funding / basis / premium / liquidations in tactics: elevated positive funding + crowded top-trader longs = caution on chase-buys; negative funding or long-flush liquidations into a dip can support buy-the-dip on crowding-sensitive names. Hint only — do not overclaim.

Positioning interpretation (2026-09 study, thin/regime-sensitive — hint only):
- Crowding/contrarian caution: ETH, SKY, BTC, AAVE (high top-trader L/S → softer next hours); LINK/ARB stronger on 4h; AERO extremes. Prefer dips when crowding cools, avoid chase when position L/S is extreme-high on these.
- Do NOT fade high L/S on UNI, ZRO, VVV, ICP.
- δ-momentum names (rising top-trader long can follow through): JUP, HYPE, NEAR, LDO, RAYSOL, SUI — not classic dip fades; don't fade strength the same way.

Dip zones: Always include invalidation.

For the NEXT ~8 HOURS from this run's time (as-of Europe/Madrid and UTC), deliver:

1) BTC and ETH: bull / bear / range — price RANGE and probability each; probs sum to 100% per coin. Apply flush detector before finalizing. Concrete from price, vol, leverage/positioning/funding/liqs.

2) Top 6 buy-the-dip FROM THE LIST ONLY. Price, why dip vs strength, buy zone / invalidation / strength, best-fit scenario. Explicitly note how Binance top-trader position + funding/basis/liqs supports or warns against each pick. No padding.

3) Open book (Revolut X): balances + every open order + SUGGESTIONS (leave/tighten/cancel/add), each suggestion informed by Binance positioning/funding where the symbol maps. Do not place/cancel/modify orders unless later explicit user confirmation.

4) Binance derivatives block: BTC+ETH (+ relevant list names) with top-trader position L/S, account L/S, global L/S, taker, OI, funding (last + vs prior), premium/basis, and 1-line dynamics.

5) Liquidations one-liner: 24h long vs short + recent flush character (Xoomar).

6) One-line CMC market/leverage backdrop where relevant.

Be concise and trader-useful. Always notify Dzmitry with the full brief — never stay silent. If CMC auth fails repeatedly, say so and pause this routine.
