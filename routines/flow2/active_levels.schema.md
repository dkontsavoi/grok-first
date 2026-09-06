# active_levels.json schema (Flow 2)

**Path:** `/workspace/crypto-self-reflect/active_levels.json`  
**Writer:** First after each Book (and after Flow 1 decisions that change levels)  
**Reader:** Main Flow-2 watcher cron (~every 10m)

```json
{
  "as_of_madrid": "2026-09-06T04:05:00+02:00",
  "book_run_id": "optional-string",
  "regime": { "flush": "off", "size_bias": "reduce", "allowed_strategies": ["CORE_DCA","BUY_DIP","RS_DIP"] },
  "cooldown_minutes": 45,
  "levels": [
    {
      "id": "BTC-CORE_DCA-zone-lo",
      "ticker": "BTC",
      "pair": "BTC-USD",
      "kind": "zone_touch",
      "strategy_id": "CORE_DCA",
      "side": "buy",
      "price": 79000,
      "price_hi": 79400,
      "direction": "into_band",
      "inv": 78500,
      "priority": 1,
      "night_safe": false,
      "active": true,
      "note": "CORE_DCA day/later"
    },
    {
      "id": "PUMP-TP1",
      "ticker": "PUMP",
      "pair": "PUMP-USD",
      "kind": "tp_vicinity",
      "strategy_id": "INVENTORY",
      "side": "sell",
      "price": 0.00425,
      "tolerance_pct": 1.0,
      "priority": 2,
      "night_safe": true,
      "active": true
    },
    {
      "id": "VVV-inv",
      "ticker": "VVV",
      "pair": "VVV-USD",
      "kind": "inv_break",
      "side": "buy_ladder",
      "price": 16.4,
      "direction": "below",
      "priority": 1,
      "night_safe": true,
      "active": true
    }
  ],
  "last_fired": {
    "BTC-CORE_DCA-zone-lo": "2026-09-06T12:00:00+02:00"
  },
  "flush_proxy": {
    "enabled": true,
    "btc_pair": "BTC-USD",
    "one_h_range_pct": 1.5
  }
}
```

**Kinds:** `zone_touch` | `inv_break` | `tp_vicinity` | `flush_proxy`  
**Rules:** Only `active: true` levels are watched. Max useful set ~8–15, not full 26. After fire, set `last_fired[id]` and honor `cooldown_minutes`.
