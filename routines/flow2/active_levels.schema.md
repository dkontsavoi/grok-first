# active_levels.json schema (Flow 2) — locked 2026-09-06

**Path:** `/workspace/crypto-self-reflect/active_levels.json`  
**Writer:** First after every Book (and after Tactical/Flow 1 that changes working orders)  
**Reader / cooldown writer:** Main Flow-2 watcher cron (`*/10 * * * *`)

## Canonical shape (Second + Main merged)

```json
{
  "as_of_madrid": "2026-09-06T04:05:00+02:00",
  "book_run_id": "2026-09-06T04:05",
  "cooldown_minutes_default": 45,
  "flush_proxy": {
    "enabled": true,
    "btc_pair": "BTC-USD",
    "btc_1h_range_pct": 1.5,
    "cooldown_key": "flush:BTC",
    "cooldown_min": 60
  },
  "levels": [
    {
      "id": "PUMP:tp1",
      "ticker": "PUMP",
      "pair": "PUMP-USD",
      "kind": "tp",
      "strategy_id": "INVENTORY",
      "side": "above",
      "price": 0.00425,
      "tolerance_pct": 0.5,
      "inv": null,
      "note": "PUMP TP1",
      "cooldown_min": 45,
      "enabled": true,
      "night_safe": true,
      "priority": 1
    },
    {
      "id": "VVV:zone_hi",
      "ticker": "VVV",
      "pair": "VVV-USD",
      "kind": "zone",
      "strategy_id": "BUY_DIP",
      "side": "below",
      "price": 16.85,
      "tolerance_pct": 0.3,
      "inv": 16.4,
      "note": "VVV ladder top",
      "cooldown_min": 45,
      "enabled": true,
      "night_safe": true,
      "priority": 1
    }
  ],
  "cooldowns": {
    "PUMP:tp1": "2026-09-06T03:00:00+02:00"
  }
}
```

## Field rules
- `kind`: `zone` | `inv` | `tp` | `flush_proxy`
- **zone/ladder buy** (`side: below`): fire when `mid <= price * (1 + tolerance_pct/100)`
- **tp sell** (`side: above`): fire when `mid >= price * (1 - tolerance_pct/100)`
- **inv**: fire when `mid <= inv` (long invalidation)
- `id` = cooldown key in `cooldowns`
- First **replaces `levels`** each Book but **preserves `cooldowns`**
- Disable cancelled orders with `enabled: false` (or drop from array)
- Keep active set small (~8–15), not all 26

## First checklist each Book
1. Open buys → zone rows; inventory TP sells → tp rows; optional day CORE BTC/ETH zones
2. Always `flush_proxy.enabled: true`
3. Merge cooldowns from prior file
4. Log `active_levels_count=N` in Book brief

## Backup revx monitor (top live tonight)
```bash
revx monitor price VVV-USD --direction below --threshold 16.85 --interval 10
revx monitor price MORPHO-USD --direction below --threshold 2.45 --interval 10
revx monitor price PUMP-USD --direction above --threshold 0.00425 --interval 10
revx monitor price JUP-USD --direction above --threshold 0.232 --interval 10
revx monitor price-change BTC-USD --direction fall --threshold 1.5 --lookback 1 --interval 10
```
Cron remains authoritative; monitors = backup / Telegram failsafe.


## Deltas (Second 2026-09-06)
1. First rewrites `levels` each Book but **merges/preserves `last_fired` and `cooldowns`** — never wipe.
2. Default level cooldown **45m**; `flush_proxy.cooldown_min: 60` fires into `last_fired["flush:BTC"]` / `cooldowns["flush:BTC"]`.
3. Watcher: if touch on `night_safe: false` during night 22:00–08:00 Madrid → still run Tactical + Flow 1 dig, but Main decision must **day-wait** places (do not instruct First to night-buy).
4. Backup `revx monitor` tracks **priority-1 night_safe** ids (VVV/MORPHO/PUMP-TP1/JUP-TP1 + BTC flush proxy).
