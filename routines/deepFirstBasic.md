# Dip scenarios

## Prompt

Produce a concise crypto scenario brief for a long-only, buy-the-dip / sell-in-strength trader.

### Trading mandate
- Only trade or recommend coins from this list: BTC, ETH, ARB, HYPE, PUMP, ENA, SKY, JUP, AAVE, AERO, LDO, UNI, POL, SYRUP, ZRO, ICP, DASH, LINK, CRV, MORPHO, SUI, ZEN, NEAR, SOL, VVV.
- Never short.
- Never recommend coins outside the list.
- Always include BTCUSDT and ETHUSDT.
- Also include held or top-dip candidates with a Binance USDT perp. 

### Data and market framing
- Use public Binance futures positioning data (no API key needed) and prioritize top-trader position dynamics.
- Report the latest 1h signal and the dynamic change vs the prior 1–2 bars for ratios, OI, and basis.
- For funding, note the last rate and direction vs the prior print.
- Use liquidation data from Xoomar or the ByKaranteli fallback. Summarize 24h long vs short USD and whether the recent hours show long-flush or short-flush behavior; use this for dip quality and trend persistence.
- Be precise with rate limiting and cache data per run.

### Flush detector (required)
Before locking the BTC/ETH scenario probabilities, check the last 1–2 hours of BTC and ETH data when available.

- If BTC 1h high-low range exceeds roughly 1.5% of spot, OR there is a clear long-liquidation spike vs the prior few hours, OR OI/price shock is visible (sharp OI drop with price down):
  - Do not keep Range as the default highest-probability scenario.
  - Set Bear at or above 40% for that coin unless an equally clear short-flush bounce is already underway.
  - Widen the bull / bear / range bands compared with a quiet chop day.
  - Name an explicit invalidation.
  - Add a one-line summary in the brief: "Flush detector: ON — [trigger]."
- If none of the triggers fire, add: "Flush detector: off." Range may lead only if volatility is genuinely quiet.

### Positioning and tactics
- Tie tactics to the current Binance top-trader position ratio and its delta, not just to a separate summary block.
- Use funding, basis, premium, and liquidation context in trade ideas: elevated positive funding plus crowded top-trader longs argues for caution on chase-buy entries; negative funding or long-flush liquidations into a dip can support more aggressive buy-the-dip behavior on crowding-sensitive names.
- Interpret positioning cautiously and qualitatively; do not overclaim.

### Required output
For the next ~8 hours from the current run time (as-of Europe/Madrid and UTC), deliver:

1. BTC and ETH scenario summary
   - Bull / bear / range probabilities per coin.
   - Price range and probability for each scenario.
   - Probabilities must sum to 100% per coin.
   - Apply the flush detector before finalizing the probabilities.
   - Keep the output grounded in price, volatility, leverage, positioning, funding, and liquidations.

2. Top 6 buy-the-dip ideas from the approved list only
   - Include price, why it looks like a dip vs strength, buy zone, invalidation, and scenario fit.
   - Explicitly note whether Binance top-trader positioning, funding, basis, and liquidations support or warn against each idea.
   - Keep the answer direct and useful; no filler.

3. Liquidations one-liner
   - Summarize 24h long vs short USD and the recent flush character from the Xoomar data.

4. Brief risk summary
   - Explain the market backdrop in one short, actionable paragraph.

### Quality bar
- Be concise and trader-useful.
- Prefer specific, actionable language over generic commentary.
- If a data source is missing or inconsistent, say so briefly and continue with the best available evidence.

Here is the latest prices:
