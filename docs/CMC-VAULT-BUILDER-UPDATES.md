# CMC Vault — desired updates for Grok builder

**Site:** https://cmc-vault.grok.me/  
**Owner:** DK  
**As-of:** 2026-09-06  
**Audience:** Grok builder (implement these changes)  
**Context:** Used by DK’s trading agents (Main / First / Second) with Revolut X spot, Book q4h, Flow 1 digs, Flow 2 RevX `active_levels` watcher. Vault today is a useful **demand-quality overlay** (browser SPA scoring Binance tape). Goal: make it **agent-readable infra** without replacing RevX execution prices.

---

## Product summary (current)

- Browser “Demand desk”: Board (cap 12) · Icebox · Holdings.
- Windows: 8h / 12h / 24h.
- Per name (client-side from Binance spot + funding): `score` 1–99, `state` ∈ `Accumulating` | `Absorbing` | `Distributing` | `Dead`, `rsVsBtc`, funding, vol ratio, `reportedNotReal`, `why`, `invalidation`, spark closes.
- UX: promote/demote, snooze, evict, Arm, Snooze 8h (Arm may require auth — lock icon).
- **Gaps:** no public JSON API, no docs, no webhook delivery to agents, universe missing some restricted-list names, prices are Binance not RevX.

---

## Goals

1. Agents (Main/Second) can `GET` desk state every Book / dig without browser scrape.
2. Universe matches DK’s restricted trading list (and can filter to it).
3. Optional push: state flips can wake Main (backup to Flow 2 RevX levels).
4. Self-reflect can join Vault desk rows to later fills / `strategy_id`.
5. Never imply Vault mid is executable on RevX — show basis when possible.

**Non-goals:** placing orders, replacing Flow 2 `active_levels.json`, replacing CMC connector or RevX CLI.

---

## Priority 1 — JSON read API (must ship)

### `GET /api/desk`

**Query**

| Param | Type | Default | Notes |
|-------|------|---------|--------|
| `window` | `8` \| `12` \| `24` | `8` | Same as UI windows |
| `restricted` | `true` \| `false` | `false` | If true, only DK restricted-list names |
| `includeIcebox` | `true` \| `false` | `true` | Include icebox rows |
| `includeHoldings` | `true` \| `false` | `false` | Include holdings qty/avg if stored |

**Response** `200 application/json`

```json
{
  "asOf": "2026-09-06T04:30:00.000Z",
  "windowHours": 8,
  "live": true,
  "source": "binance",
  "cached": false,
  "deskCap": 12,
  "rows": [
    {
      "id": "AAVE",
      "symbol": "AAVEUSDT",
      "list": "desk",
      "score": 72,
      "state": "Accumulating",
      "rsVsBtc": 1.04,
      "funding": 0.0001,
      "volRatio": 1.2,
      "reportedNotReal": false,
      "why": "short string from model",
      "invalidation": "short string",
      "sparkCloses": [1, 2, 3],
      "restricted": true,
      "armed": false,
      "snoozedUntil": null
    }
  ],
  "icebox": [],
  "holdings": []
}
```

**Rules**

- Same fields the UI shows — single source of truth (server compute or shared module; avoid drift).
- If feed missed and UI uses cached tape: `"live": false`, `"cached": true`.
- CORS: allow agent/server fetch (or at least same-origin + documented server-side use).
- No auth required for read of public desk scores (read-only). If Arm/Holdings need auth later, keep `/api/desk` public.

### `GET /api/health`

```json
{ "ok": true, "asOf": "...", "live": true }
```

### `GET /docs` or `/openapi.json`

Short OpenAPI or markdown describing the endpoints above.

---

## Priority 2 — Universe sync (restricted list)

DK restricted list (long-only):

`BTC ETH ARB HYPE PUMP ENA SKY JUP AAVE AERO LDO UNI RAY POL SYRUP ZRO ICP DASH LINK CRV MORPHO SUI ZEN NEAR SOL VVV`

**Missing from Vault bundle today (add):** `ARB`, `DASH`, `SYRUP`, `VVV`, `ZEN`

**Implement**

1. Add those five with correct Binance USDT symbols (resolve collisions carefully for SKY/PUMP/HYPE/POL/ZEN/VVV — same discipline as Book).
2. Tag every row `restricted: boolean`.
3. UI mode toggle: **“DK list only”** filters Board/Icebox to `restricted: true`.
4. Optional: hide or demote non-list noise (WLFI, BONK, …) when DK mode on.
5. Keep desk cap 12 behavior but scoring pool = full restricted list when in DK mode.

---

## Priority 3 — Webhook on state / Arm flips (Flow 2 backup)

### Config (owner-set)

- Webhook URL (Grok Bot routine webhook URL from routine panel).
- Optional shared secret header `X-Vault-Secret: ...`
- Enable/disable per event type.

### Events to POST (JSON)

Fire on:

1. `state_change` — e.g. `Absorbing` → `Accumulating`, or any → `Dead` / `Distributing`
2. `reportedNotReal_true` — flipped to true
3. `armed_trigger` — when Armed condition hits (document what Arm means: price/state threshold)
4. Optional: `score_cross` — score crosses 70 up or 40 down

**Payload example**

```json
{
  "event": "state_change",
  "asOf": "2026-09-06T04:30:00.000Z",
  "windowHours": 8,
  "id": "ENA",
  "fromState": "Absorbing",
  "toState": "Accumulating",
  "score": 68,
  "reportedNotReal": false,
  "rsVsBtc": 0.98,
  "why": "...",
  "invalidation": "...",
  "source": "binance"
}
```

**Delivery**

- At-least-once OK; include `eventId` for dedupe.
- Cooldown per `(id, event)` ≥ 30–60 minutes (align with Flow 2).
- Agents treat this as **backup** to RevX `active_levels` watcher — not primary execution signal.

---

## Priority 4 — Machine export for Self-reflect

### `GET /api/desk/export` or UI **Export JSON**

- Download/write same shape as `/api/desk`.
- Optional server-side write path is N/A on static host — export download is enough; agents can curl `/api/desk` into `/workspace/crypto-self-reflect/vault_desk.json`.

**Suggested agent file path (for DK ops docs only):**  
`/workspace/crypto-self-reflect/vault_desk.json`

Log-friendly: one snapshot per Book join on `id` ↔ ticker.

---

## Priority 5 — RevX basis flag (honesty)

Vault remains Binance-sourced. Add optional fields when a RevX mid is supplied or fetched:

```json
{
  "binanceLast": 0.181,
  "revxMid": 0.1805,
  "basisBps": 27,
  "executableHint": "binance_only"
}
```

**UI:** small badge “Binance tape ≠ RevX” on desk; if `|basisBps|` high, warn.

If RevX fetch is out of scope for Vault host, at least document: **scores are demand quality on Binance; Book/First must use RevX mids for orders.**

---

## Priority 6 — Soft strategy hints (later)

Map Vault state → **hint only** (never override agent policy):

| Vault state | Soft hint |
|-------------|-----------|
| Accumulating | `BUY_DIP` / `RS_DIP` candidate |
| Absorbing | watch / wait wash |
| Distributing | SKIP add / trim bias |
| Dead | icebox / SKIP |
| reportedNotReal | SKIP night BUY_DIP |

Expose as `strategyHint` string on API rows. Main/Second may ignore.

---

## Arm / Snooze clarification (product)

Document and expose via API:

- What **Arm** watches (price, state, score?).
- Where Arm fires today (UI only? Telegram?).
- Prefer: Arm → webhook event `armed_trigger` (Priority 3).
- Snooze 8h → `snoozedUntil` ISO on row; API must omit or flag snoozed from “actionable” filters: `?actionable=true`.

---

## Acceptance checks

- [ ] `curl -s 'https://cmc-vault.grok.me/api/desk?window=8&restricted=true' | jq '.rows | length'` returns restricted-only rows including ARB, DASH, SYRUP, VVV, ZEN when listed.
- [ ] `live` / `cached` flags accurate when Binance feed misses.
- [ ] Webhook test endpoint receives `state_change` within ~1 minute of a forced flip in staging.
- [ ] OpenAPI/docs page exists.
- [ ] UI “DK list only” matches API `restricted=true`.
- [ ] No agent requires headless browser to read desk.

---

## Agent integration notes (for builder / DK)

After API ships:

1. **Second** — each Flow 1 dig: fetch `/api/desk?restricted=true` and cite `state/score/reportedNotReal/rsVsBtc/why`.
2. **Main** — regime: many `Distributing`/`Dead` + greed → reinforce reduce / `CORE_DCA`.
3. **First** — do **not** place from Vault prices; RevX only.
4. **Flow 2** — primary remains RevX `active_levels.json`; Vault webhook = backup demand signal → Main Tactical → Flow 1.

---

## Out of scope / do not build

- Order placement or RevX keys inside Vault.
- Replacing LSR / funding policy bands.
- Standing monitors for all non-restricted meme names.
- Scraping-only “solutions” without API.

---

## Suggested ship order

1. `/api/desk` + `/api/health` + docs  
2. Add missing tickers + `restricted` + DK list UI filter  
3. Webhook + Arm → webhook  
4. Export / agent snapshot habit  
5. Basis badge + `strategyHint`

---

*End of builder brief. Questions → Main/Second; policy owner = DK.*
