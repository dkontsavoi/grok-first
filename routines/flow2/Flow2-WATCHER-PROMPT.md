# Flow 2 — Book-level watcher (Main)

**Trigger:** every 1 hour at :00 Madrid **except Book hours** 00/04/08/12/16/20 (Flow 1 owns those). Cron: `0 1-3,5-7,9-11,13-15,17-19,21-23 * * *`.  
**Quiet OK:** if nothing touched and no Flush proxy, stay silent.

## Intent each run
1. Read `/workspace/crypto-self-reflect/active_levels.json`.
2. For each `active: true` level (and Flush proxy if enabled), fetch current RevX mid via `revx` market ticker for that pair.
3. Detect touches:
   - `zone_touch` / `into_band`: mid within [price, price_hi] (or within tolerance)
   - `inv_break` / `below`: mid < price
   - `tp_vicinity`: mid within tolerance_pct of TP price
   - Flush proxy: BTC 1h high-low % ≥ threshold → treat as regime alert
4. Skip if `last_fired[id]` within `cooldown_minutes`.
5. On any new fire:
   - Update `last_fired` in the JSON file
   - Run **Tactical** intent (same rules as Tactical routine: book hygiene + NOW opportunities)
   - Enter **Flow 1**: assign Second dig on fired tickers / actions → Main decide (≤1 re-dig) → First execute
   - Notify DK with a short scannable alert + decision status
6. If no fires: stay quiet (no user message).

## Caps
Respect all policy locks (night symbol/notional caps, DD_WARN/PAUSE, Tier-C, SKIP lists). Alert ≠ auto-place.

## Backup
Top active levels may also run under `revx monitor price` (First/ops); treat those Telegram/events as secondary signals into the same Tactical→Flow 1 path when seen.


## Night vs day-wait (Second delta)
If a fired level has `night_safe: false` and local Madrid hour is in **22:00–08:00**: still run Tactical + Flow 1 dig, but Main must **day-wait** any new buys — do not instruct First to place night buys for that alert. Exits/cancels still OK if policy allows.
Preserve `last_fired`/`cooldowns` across First level rewrites.
