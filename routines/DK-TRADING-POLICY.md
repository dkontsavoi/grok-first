# DK Trading Policy — Main’s understanding

**As-of:** 2026-09-06 (Europe/Madrid)  
**Author:** Main (policy coordinator)  
**Status:** Policy locks U1–U14 closed 2026-09-06 · **Flow 1** decision cycle locked — live via Book/Tactical + this doc  
**Sources:** live Book / Main / Tactical prompts on box, proposed router pack (`grok-first` PR #2 / `/workspace/grok-first-proposed`), agent memories, chat locks with DK / First / Second

---

## 1. Cornerstone (goal)

**Team goal (LOCKED 2026-09-06):**

> Grow RevX spot equity **consistently and sustainably** within written risk limits (night ≤$1500 / DD ≤$500), by only taking setups the regime + per-coin `strategy_id` router allow, with every entry paired to an exit.

**What “consistent” should mean in practice (suggestion):**

| Metric | Suggested definition | Status |
|--------|----------------------|--------|
| Process consistency | Every Book/Tactical cycle follows regime gate → router → caps → exits | Live (full strategy_id router) |
| P&L consistency | Prefer many small +EV trades over rare large bets; night heat caps bound overnight risk | Caps + DD $500 live; weekly review by strategy_id (no hard $ profit target) |
| Decision consistency | Same coin + same tape → same `strategy_id` or SKIP; Main decides ops 24×7 within caps (no DK micromanagement) | Locked in chat |

---

## 2. Roles (who does what)

| Actor | Role | May change policy? | May place day orders? | May place night orders? |
|-------|------|--------------------|------------------------|-------------------------|
| **DK** | Principal | Yes (only he) | Overrides / outside-policy only | Standing ops caps he approved |
| **Main** (this agent) | Policy coordinator / digital copy | No — enforce & escalate | **Yes 24×7 within policy** (no DK ping) | No direct; authorizes First |
| **First** | Execution (Book, Tactical, RevX) | No | After **Main** decision (Flow 1) | After Main; same caps 24×7 |
| **Second** | Investigation / strategy research | Propose only | No | No |

**Naming (locked):**

- **Main** = agent (coordinator), not the cron.  
- **Book** = every-4h brief runner First executes (was formerly named “Main” cron).  
- **Tactical** = on-demand book hygiene / NOW tactics (suggest-only for places until confirm).

---


## 2b. Flow 1 — Book decision cycle (LOCKED 2026-09-06)

Standing operating sequence for each Book (and Tactical when it surfaces actionable coins):

```
1. First runs Book → produces coins / actions to decide
2. That triggers Second → deeper dig on those coins → results to Main
3. Main decides within policy, or escalates / delegates to DK
4. Main or DK may ask Second for **at most one** additional dig (no loops)
5. Final decision → First executes
6. Next trigger = next Book run (or next Tactical ask)
```

### Rules
- **Second never self-starts a loop** off their own dig; only Main assigns digs (proactive OK between cycles, but inside Flow 1 the chain is First→Second→Main).
- **One additional dig max** per coin/decision set after the initial post-Book dig.
- **Night:** Flow 1 still applies for non-standing actions; standing night auto (within caps) may place from Book without waiting on Second **only** for night-safe rows already compiled — material concentration / Tier-C / unclear routing should still kick Second before size-up. Cancels of stale/wrong-regime bids can proceed under Main without a full dig when policy is already clear (e.g. >12h stale).
- **Day/Night ops:** First Book/Tactical → Flow 1 dig → **Main decides in-policy** → First executes (no DK ping). Escalate only if outside policy.
- Main remains policy gate; DK remains principal; Second research-only.


## 2c. Flow 2 — Alert → Tactical → Flow 1 (LOCKED 2026-09-06)

```
Price alert (restricted-list coin)
  → Main runs Tactical
  → Tactical execution triggers Flow 1
      (Second dig → Main/DK ≤1 re-dig → First execute)
  → Next alert or next Book
```

**Status:** Locked. **Primary:** Main cron **hourly** at :00 Madrid **except Book hours** `00/04/08/12/16/20` (cron `0 1-3,5-7,9-11,13-15,17-19,21-23 * * *`) — those slots are Book/Flow 1 only (~18 Flow 2 wakes/day). Polls RevX mids vs `/workspace/crypto-self-reflect/active_levels.json`. On touch → Tactical → Flow 1. Cooldown ~45m/level. **Backup:** `revx monitor price` on top 3–5 active levels. Schema: `flow2/active_levels.schema.md`.

**Rules (intent):**
- Alerts only for restricted-list names (and levels from Book/Tactical/inv/TP).
- Alert wakes **Main** (not First directly) to run Tactical.
- Every Tactical run enters Flow 1 before new risk (same one-re-dig cap).
- Does not replace Book q4h; complements it for intraday zone/inv events.


## 2d. Agent messaging — no ack ping-pong (LOCKED 2026-09-06)

**Rule:** Agents must **not** send peer messages that are only acknowledgements / empty FYIs (“Ack”, “Got it”, “Noted”, “Confirming live” with no new payload).

**Allowed peer messages:** decisions, dig results, exec reports (order ids/sizes), Flow 1 coin lists, policy locks, blockers needing action.

**Why:** Each `[agent]` wake is billable (steps + tokens). Ack ping-pong burns usage without trading value.

**Unchanged:** Flow 1, Flow 2 cadence (hourly, skip Book hours), user-facing notifies when there is a real result.


## 2e. Flow 3 — Selfy improvement loop (LOCKED 2026-09-06)

```
Selfy (First, 07:00 Madrid) grades ~48h Book → scorecard + Concerns
  → First / Second / Main refine concerns (payload only, no ack ping-pong)
  → Second digs mitigations
  → Main decides in-policy OR asks DK
  → First executes only approved changes; reports
```

**Artifacts:** `/workspace/crypto-self-reflect/scores.jsonl` (`routine="Selfy"`); `/workspace/crypto-self-reflect/flow3.jsonl`  
**Doc:** `/workspace/flow3/README.md`  
**Main’s job:** after Second’s mitigations dig, decide within policy or escalate to DK — never freestyle risk; First does not self-approve prompt/policy changes.

## 3. Hard constraints (live until DK changes)


### 3.0 RiskLimit object (LOCKED 2026-09-06)

24h risk object (Madrid calendar day). Live: `/workspace/crypto-self-reflect/risk_limit.json`.

| Field | reduce (now) | normal |
|-------|--------------|--------|
| `transaction_day_limit_usd` | **$600** | **$1000** |
| `max_day_drop_usd` (from SOD) | **$280** (warn $200) | $280 |
| `max_hwm_dd_usd` | **$500** (warn $350) | $500 |
| `per_run_limit_usd` | **$400** | **$1000** |
| `tier_c_day_cap_usd` | **$200** | **$400** |
| `max_new_symbols_run` | **3** | **3** |

- **transaction_day_limit** = max NEW buy notional / day (exits don’t count). Replaces former night cumulative $1500.
- **max_day_drop** = pause new risk if equity ≤ SOD equity − this (new circuit).
- **max_hwm_dd** = trailing HWM pause (unchanged role of old DD $500).
- Flush:ON → Tier-C day cap **$0**; regime may further cut deploy.
- `reliability_mult` starts **0.8**; raise toward/above old $1500 only after ≥4 clean Selfy weeks (Main proposes, DK locks).
- Ops autonomy 24×7 still applies **inside** these caps.

### 3.1 Universe & direction

- **Long-only.** Never short. Never sell-to-open.
- **Restricted list (26):**  
  `BTC ETH ARB HYPE PUMP ENA SKY JUP AAVE AERO LDO UNI RAY POL SYRUP ZRO ICP DASH LINK CRV MORPHO SUI ZEN NEAR SOL VVV`
- Venue: **Revolut X spot** via `revx`. Binance funding / basis / L/S = **signals only**, never a carry book.

### 3.2 Style

- Core style: **buy-the-dip / sell-in-strength**.
- Every buy idea **must** pair with an exit plan (TP1 / TP2 / runner or time-stop / invalidation).
- Complementary modes (tags / proposed strategies): `BUY_DIP`, `FLUSH_MR`, `MOM`, `RS_DIP`, `CORE_DCA`, `CATALYST`.

### 3.3 Night auto (standing approval) — Europe/Madrid

> **Superseded (2026-09-06):** use **RiskLimit.transaction_day_limit** (24h). Under reduce = **$600**; normal = **$1000**. Former night-only $1500 retired.

| Rule | Value |
|------|--------|
| Night window | 22:00–08:00 (22 inclusive → 08 exclusive) |
| Day window | 08:00–21:59 → **ops via Main** (no DK approval) |
| Per-run new buy notional | ≤ **RiskLimit.per_run** ($400 reduce / $1000 normal) |
| Transaction day limit (24h new buys) | ≤ **RiskLimit** ($600 reduce / $1000 normal) |
| Max new symbols / night | ≤ **3** |
| Order types | Limit buys only; never market buys |
| Sells at night | **Inventory only** (TP1/TP2); sell $ does not count vs buy cap |
| Cancels | Allowed for bids the brief marks cancel/replace; no cancel-all |
| Flush:ON | **$0 Tier-C** auto; prefer BTC/ETH (or ≤$300 total if bounce confirmed) |
| MOM at night | No chase; only pre-approved continuation levels |

### 3.4 Max drawdown + day drop (locked 2026-09-06; RiskLimit)

- **max_hwm_dd:** **$500** from equity HWM — `DD_PAUSE` on breach.
- **dd_warn (HWM):** **$350** — `DD_WARN`.
- **max_day_drop:** **$280** from **SOD** equity — pause new risk if hit (warn **$200**).
- Inventory exits/cancels still OK while paused; resume HWM pause only on DK explicit resume.
- **On breach:** **pause all new risk** — no new night auto buys; Main must not approve new day buys; First stands down new entries.
- **Still allowed while paused:** inventory TP/exit sells, cancels of open buy bids, Book/Tactical briefs (flag `DD_PAUSE: ON`).
- **Resume:** only DK explicit resume (or DK override). Main does not self-unpause for this gate (DK resume only).

### 3.5 Operational autonomy (LOCKED 2026-09-06 — kill micromanagement)

**Standing approval 24×7:** Main authorizes First to place/cancel/modify **in-policy** orders any hour — same posture as former night auto. **No DK ping/approval** for operational tasks.

**Still escalate to DK (principal only):**
- Policy / cap / list / strategy-router changes
- Anything **outside** written caps or hard locks
- `DD_PAUSE` resume
- Explicit overrides / CATALYST event pre-approval (U9)
- Flow 3 items Main flags as consequential

**Ops caps (apply day and night — see §3.0 RiskLimit):**
- New buy notional ≤ **per_run_limit** / run; day total ≤ **transaction_day_limit**
- ≤ **3** new symbols / run
- Limit buys only; inventory sells OK; Flush/Tier-C/LSR/DD + **max_day_drop** gates unchanged
- First executes only on **Main** decision (Flow 1) — First does not freestyle

**U7/U8:** retired as day-micromanagement. Optional FYI to DK on large packages is fine; not required; never blocks execution.

### 3.6 CATALYST night (U9 locked 2026-09-06)

- **Never night-auto `CATALYST`** unless DK pre-approved that event ID + levels + max $ the same day. Default: day/Tactical only.

### 3.7 Liquidity tiers (U10 locked 2026-09-06)

| Tier | Names | Notes |
|------|-------|-------|
| **A** | BTC, ETH | Tightest zones |
| **C** | **PUMP, VVV, MORPHO, SYRUP** | Thin; Flush:ON → $0 new; DK-editable list |
| **B** | All other restricted-list names | Default |

### 3.8 Review cadence (U6 locked 2026-09-06)

- **Daily** (Self-reflect): rule breaks, DD distance to $500, night fill quality.  
- **Weekly** (Main owns keep/reduce/pause call; Second digs): by `strategy_id` — n, hit-rate, avg R, night MAE.  
- **No hard $ profit target.** Optional soft: don’t raise caps until 4 clean weeks + DD unused >50% of time.  
- Don’t change strategy rules on &lt;20–30 fills.


---


### 3.9 Night stand-down (U13 locked 2026-09-06)

Night **new buys** stand down (place **$0** new) when any of:
- (a) `DD_PAUSE`
- (b) equity DD from HWM ≥ **$350**
- (c) Flush:ON **and** DD ≥ **$250**
- (d) Flush:ON → Tier-C **$0** always (already)

Otherwise normal caps: ≤$1000/run, ≤$1500 cum, ≤3 new symbols. Exits/cancels always allowed.

### 3.10 LSR / funding bands v1 (U14 locked 2026-09-06)

- **Mild / Tier-C** (PUMP VVV MORPHO SYRUP SOL): SKIP `BUY_DIP` if top-trader pos LSR > **4.0**
- **Crowding set** (BTC ETH AAVE SKY LINK ARB AERO): if LSR top-quintile **and** funding elevated vs 7d median → size ×**0.5** and widen zone; if δ↑ near local highs → SKIP
- **Don't-fade** (UNI ZRO VVV ICP): never SKIP solely for high LSR
- Revisit after 30 logged skips

### 3.11 Naming & cadence (U11 / U12 locked 2026-09-06)

- **Book** = cron/brief name in prompts, JSONL (`routine="Book"`), notifications
- **Main** = coordinator agent only
- Only scenario/order cadence: Book **00/04/08/12/16/20** Europe/Madrid; archive older 8h/hourly desires
- Tactical = on-demand only


## 4. Operating cycle (intended)

```
Regime gate (book-level)
    → allowed_strategies + size_bias + stand_down
Per-coin router
    → exactly one strategy_id per ticker (or SKIP)
Compile
    → mixed top ideas in one Book brief
Execute
    → First: night auto within caps / day after confirm
Investigate
    → Second when routing or edge is unclear
```

**One cron mindset:** different coins may get different strategies in the **same** Book run. Never two strategies on one ticker in one run.

---

## 5. What is LIVE today vs PROPOSED

### 5.1 Live (Book / Tactical) — full `strategy_id` router adopted 2026-09-06

- **Regime gate → per-coin `strategy_id` (or SKIP) → compiler** (mixed strategies in one Book run)
- Strategy IDs: `BUY_DIP` | `FLUSH_MR` | `MOM` | `RS_DIP` | `CORE_DCA` | `CATALYST`
- Top-6: `TICKER | strategy_id | thesis | zone | inv | size | exit`
- Night-safe filter by `strategy_id`; Flush + ATR tiers + exit ladder unchanged
- Binance POSITION dynamics; don't-fade `UNI ZRO VVV ICP`
- Legacy buckets (`CROWD-DIP`/`MOM`/`WATCH`) = optional aliases only; **authoritative = `strategy_id`**
- Logging: `strategy_id`, `allowed_strategies`, `size_bias` in `briefs.jsonl` (`routine="Book"`)
- Caps: night ≤$1000/run, cumulative ≤**$1500**, max DD **$500** HWM pause

### 5.2 Strategy modules (reference)

Repo: `github.com/dkontsavoi/grok-first` → `routines/proposed/strategies/` (and `routines/Main-PROMPT.md` / `Book-PROMPT.md` live text).  
Local mirror: `/workspace/grok-first-proposed/strategies/`

**Main’s job:** enforce router + caps; First executes Book; Second digs when routing unclear.

---

## 6. Regime & Flush (live)

**Flush:ON** if any of:

- BTC 1h high–low ≳ 1.5% of spot, **or**
- Clear long-liq spike, **or**
- OI/price shock

When ON:

1. Do not keep Range as highest BTC/ETH prob; Bear ≥40% unless clear short-flush bounce.  
2. No new Tier-C auto/night (PUMP, VVV, MORPHO, SYRUP, other thin).  
3. Tier-B only if that coin’s own 1h range already exceeds threshold **and** long-liq character supportive.  
4. Prefer BTC/ETH until Flush off or short-flush bounce confirmed.  
5. After long-flush: deeper resting limits; avoid market-adjacent fills until 15m/1h reclaim.

**Proposed gate rubric (extra, not live as orchestrator):** F&G, greed/quiet, risk-on RS leaders, quiet range → `allowed_strategies` + `size_bias` (normal | reduce | core-only).

---

## 7. Entries, sizing, exits (live summary)

### Zones

| Tier | Examples | Mid depth guide |
|------|----------|-----------------|
| A | BTC, ETH | ~1.0–2.0% |
| B | SOL, LINK, AAVE, ARB, SUI, … | ~2–4% |
| C | PUMP, VVV, MORPHO, SYRUP, … | ~3–6%; smaller $; need Flush-off or BTC-stable |

Width ≈ `max(1.5%, k × ATR_1h)`, k ≈ 0.75–1.25. Default 2–3 rungs. Deeper size only if crowding cooling **or** long-flush — not blind average-down.

### Exits (required with every buy)

| Step | Rule |
|------|------|
| TP1 | ~+1R or reclaim of pre-dip mid — sell 30–40% |
| TP2 | ~+2R or local swing / VWAP reclaim fail — sell 30–40% |
| Runner | Trail under prior 1h/4h HL; **or** time-stop 24–48h if no TP1 / new HH |
| Invalidation | SL at inv on fill; close < inv → cancel remaining ladder + soft exit |

Cancel stale buy bids older than **8–12h**, or when Flush:ON moves inv through the bid.

Prefer night ladders ~**$100–$150** per rung, still under caps.

---

## 8. Data stack

| Source | Use |
|--------|-----|
| CoinMarketCap | Quotes, technicals, global; resolve ticker collisions |
| Revolut X (`revx`) | Live balances, open orders, place/cancel |
| Binance public (`www.binance.com`) | Top-trader POSITION L/S (primary), funding, OI, basis, premium |
| Liquidations | Xoomar primary; ByKaranteli backup |

---

## 9. Unclear areas & fix suggestions

| # | Unclear | Why it matters | Suggested fix |
|---|---------|----------------|---------------|
| U1 | ~~Goal sentence~~ | **LOCKED 2026-09-06** | Grow RevX spot equity consistently and sustainably within night ≤$1500 / DD ≤$500; router-only setups; entry↔exit |
| U2 | ~~Night cumulative~~ | **SUPERSEDED 2026-09-06** | RiskLimit `transaction_day_limit` 24h: **$600** reduce / **$1000** normal |
| U3 | ~~Buckets vs strategy_id~~ | **LOCKED 2026-09-06** | Full per-coin `strategy_id` orchestrator adopted into Book (DK over Second staged B1). Authoritative tag = `strategy_id`; legacy buckets optional aliases only |
| U4 | ~~Orchestrator not live~~ | **LOCKED 2026-09-06** | Full router live in Book — regime gate + per-coin `strategy_id` + compiler |
| U5 | ~~No max DD~~ | **LOCKED + extended 2026-09-06** | HWM DD **$500** / warn $350 + SOD **max_day_drop $280** / warn $200 → pause new risk; exits/cancels OK; HWM resume = DK only |
| U6 | ~~Review cadence~~ | **LOCKED 2026-09-06** | Daily process check; weekly by strategy_id (n, hit-rate, avg R); no hard $ profit target; Main keep/reduce/pause |
| U7 | ~~Day ping / timeout~~ | **RETIRED 2026-09-06** | No DK approval for in-policy ops; Main decides 24×7 within caps |
| U8 | ~~Silent day size~~ | **SUPERSEDED 2026-09-06** | Ops caps ≤$1000/run ≤3 symbols apply 24×7 under Main (not a DK-silent window) |
| U9 | ~~CATALYST night~~ | **LOCKED 2026-09-06** | Never night-auto CATALYST without same-day DK pre-approval of event + levels + max $ |
| U10 | ~~Tier-C list~~ | **LOCKED 2026-09-06** | Tier-C = PUMP, VVV, MORPHO, SYRUP (DK-editable); A=BTC/ETH; B=rest |
| U11 | ~~Naming Book vs Main~~ | **LOCKED 2026-09-06** | Book = cron/brief name everywhere (prompts, JSONL, notifications); Main = coordinator agent only |
| U12 | ~~Brief cadence~~ | **LOCKED 2026-09-06** | Only Book 00/04/08/12/16/20 Madrid for scenarios/orders; archive 8h/hourly; Tactical = on-demand only |
| U13 | ~~Night stand-down~~ | **LOCKED 2026-09-06** | Night new buys $0 when: DD_PAUSE; DD from HWM ≥$350; Flush:ON AND DD ≥$250; Flush:ON → Tier-C $0 always. Else normal caps. Exits/cancels always OK |
| U14 | ~~LSR/funding bands~~ | **LOCKED 2026-09-06** | Mild/Tier-C SKIP BUY_DIP if pos LSR >4.0; crowding + elevated funding vs 7d median → size ×0.5 + widen; δ↑ near highs → SKIP; don’t-fade never SKIP for high LSR alone |

---

## 10. Decision hierarchy (conflict resolution)

1. **DK explicit override** (chat) wins.  
2. Else **written hard constraints** in this doc / live Book prompt.  
3. Else **Main** judgment within policy (ops autonomy 24×7; escalate outside policy).  
4. Else First executes Book/Tactical literally.  
5. Second never authorizes live risk — research only.

If something is outside policy → **escalate to DK** (do not invent risk).

---

## 11. Adoption checklist (when you want the router live)

- [x] Finish cornerstone sentence (U1) — locked 2026-09-06  
- [x] Pick hard night cumulative $ (U2) → **$1500** locked 2026-09-06  
- [x] Main silent-day size cap (U8) ≤$1000 / ≤3 — locked 2026-09-06  
- [x] Max DD pause (U5) → **$500** from HWM; pause new risk until DK resumes (locked 2026-09-06)  
- [x] Merge router into Book prompt (DK full adopt 2026-09-06)  
- [x] Switch top-6 + JSONL to `strategy_id`  
- [x] Patch Tactical references Main → Book (U11)  
- [ ] Push scrubbed Markdown to `github.com/dkontsavoi/grok-first` under `routines/`  
- [x] Tell First + Second the adoption is live

---

## 12. One-screen cheat sheet

```
LONG-ONLY · 26-coin list · RevX spot
Flow 1: Book → Second dig → Main/DK (≤1 re-dig) → First executes
Flow 2: hourly except Book hours 00/04/08/12/16/20
Book q4h · Main = policy · Second = digs
Ops 24×7 via Main · RiskLimit day $600 reduce/$1000 normal · per_run $400/$1000 · HWM DD $500 · day drop $280 · ≤3 symbols · Tier-C day $200/$400
24h RiskLimit: day deploy + per_run + day drop + HWM DD (see §3.0)
Flush:ON → Tier-C $0 · night stand-down if DD≥$350 or Flush+DD≥$250
LSR>4.0 mild SKIP · Book-only 4h cadence
Every buy needs TP/SL/time-stop
HWM DD $500 + SOD day drop $280 → pause new risk (HWM resume = DK)
Funding = signal only · never short · never freestyle risk
```

---

*End of draft. Reply with edits (especially U1–U8) and I’ll revise this file and propagate to First/Second.*
