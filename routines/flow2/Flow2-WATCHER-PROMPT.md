# Flow 2 — Book-level watcher (Main)

**Trigger:** every 10 minutes, 24×7 (crypto night auto + day Tactical).  
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
