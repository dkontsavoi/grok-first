# DK Trading Policy — Main’s understanding

**As-of:** 2026-09-06 (Europe/Madrid)  
**Author:** Main (policy coordinator)  
**Status:** Working draft for DK review — not a substitute for live Book/Tactical prompts until you adopt changes  
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
| Decision consistency | Same coin + same tape → same `strategy_id` or SKIP; Main decides day confirms after 1h silence | Locked in chat |

---

## 2. Roles (who does what)

| Actor | Role | May change policy? | May place day orders? | May place night orders? |
|-------|------|--------------------|------------------------|-------------------------|
| **DK** | Principal | Yes (only he) | Yes (confirm / override) | Standing night rules he already approved |
| **Main** (this agent) | Policy coordinator / digital copy | No — enforce & escalate | Yes after ping→1h silence, **within policy only** | No direct; sets what First may auto |
| **First** | Execution (Book, Tactical, RevX) | No | Only after Main/DK confirm | Yes, under night Book rules |
| **Second** | Investigation / strategy research | Propose only | No | No |

**Naming (locked):**

- **Main** = agent (coordinator), not the cron.  
- **Book** = every-4h brief runner First executes (was formerly named “Main” cron).  
- **Tactical** = on-demand book hygiene / NOW tactics (suggest-only for places until confirm).

---

## 3. Hard constraints (live until DK changes)

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

> **Lock (2026-09-06):** night cumulative new-buy notional hard cap = **$1500** (not a range).

| Rule | Value |
|------|--------|
| Night window | 22:00–08:00 (22 inclusive → 08 exclusive) |
| Day window | 08:00–21:59 → **suggest only** until confirm |
| Per-run new buy notional | ≤ **$1000** |
| Night cumulative new buys | ≤ **$1500** (hard) |
| Max new symbols / night | ≤ **3** |
| Order types | Limit buys only; never market buys |
| Sells at night | **Inventory only** (TP1/TP2); sell $ does not count vs buy cap |
| Cancels | Allowed for bids the brief marks cancel/replace; no cancel-all |
| Flush:ON | **$0 Tier-C** auto; prefer BTC/ETH (or ≤$300 total if bounce confirmed) |
| MOM at night | No chase; only pre-approved continuation levels |

### 3.4 Max drawdown (locked 2026-09-06)

- **Max DD:** **$500 USD** peak-to-trough from equity **high-water mark** (RevX account equity).
- **Warn:** at **$350** (70% of max) — flag `DD_WARN`; tighten night (prefer core-only / no Tier-C).
- **On breach:** **pause all new risk** — no new night auto buys; Main must not approve new day buys; First stands down new entries.
- **Still allowed while paused:** inventory TP/exit sells, cancels of open buy bids, Book/Tactical briefs (flag `DD_PAUSE: ON`).
- **Resume:** only DK explicit resume (or DK override). Main does not self-unpause after 1h timeout for this gate.

### 3.5 Day confirmation (locked 2026-09-06; U7/U8 refined)

1. First proposes; **Main** owns the confirm ask.  
2. **Ping (U7):** Main’s message to DK with **concrete** orders — ticker, side, price, $, `strategy_id`, why. Clock starts at that send. Optional reminder @~45m.  
3. If **no reply within 1 hour**, Main decides **within written policy** and logs `silent_approve` or `silent_deny`.  
4. **Silent day-approve size (U8):** ≤ **$1000** new buys, ≤ **3** symbols, same tier/Flush/night-safe `strategy_id` rules. Above that → wait DK or deny. In-policy sells/cancels always OK.  
5. First **does not** freestyle day places/cancels on silence — waits for Main.  
6. Night Book auto rules unchanged by this timeout.

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
| U2 | ~~Night cumulative range~~ | **LOCKED 2026-09-06** | Hard cap **$1500** USD new buys per night window |
| U3 | ~~Buckets vs strategy_id~~ | **LOCKED 2026-09-06** | Full per-coin `strategy_id` orchestrator adopted into Book (DK over Second staged B1). Authoritative tag = `strategy_id`; legacy buckets optional aliases only |
| U4 | ~~Orchestrator not live~~ | **LOCKED 2026-09-06** | Full router live in Book — regime gate + per-coin `strategy_id` + compiler |
| U5 | ~~No max DD~~ | **LOCKED 2026-09-06** | Max drawdown **$500 USD** from equity high-water mark → **pause all new risk** (no new night buys, no Main day approve of new buys) until DK resumes; inventory TP/exits and cancels still allowed |
| U6 | ~~Review cadence~~ | **LOCKED 2026-09-06** | Daily process check; weekly by strategy_id (n, hit-rate, avg R); no hard $ profit target; Main keep/reduce/pause |
| U7 | ~~Ping definition~~ | **LOCKED 2026-09-06** | Ping = concrete orders; clock at send; optional @45m; log silent_approve/deny |
| U8 | ~~Silent day size~~ | **LOCKED 2026-09-06** | Silent day approve ≤$1000 new buys, ≤3 symbols |
| U9 | ~~CATALYST night~~ | **LOCKED 2026-09-06** | Never night-auto CATALYST without same-day DK pre-approval of event + levels + max $ |
| U10 | ~~Tier-C list~~ | **LOCKED 2026-09-06** | Tier-C = PUMP, VVV, MORPHO, SYRUP (DK-editable); A=BTC/ETH; B=rest |
| U11 | Tactical prompt still says “Main” / `briefs.jsonl` from Main | Naming drift after Book rename | Patch Tactical to read Book / `routine="Book"` (First should already be aligning) |
| U12 | 8h / hourly brief desires in older memory vs live **4h Book** | Conflicting schedules in shared memory | Confirm Book 00/04/08/12/16/20 Madrid is the only scenario brief; archive older 8h/hourly asks |
| U13 | “Consistent profit” vs night standing approval | Overnight can place without DK eyes | Keep Flush + caps; add optional: if Flush:ON, Main must be woken / night stands down to BTC/ETH-only (already mostly true) |
| U14 | Funding as signal — thresholds not numeric | Agents improvise | Publish small table: elevated funding + extreme LSR → widen/cut size (already qualitative); add numeric LSR skip bands per mild names |

---

## 10. Decision hierarchy (conflict resolution)

1. **DK explicit override** (chat) wins.  
2. Else **written hard constraints** in this doc / live Book prompt.  
3. Else **Main** judgment within policy (incl. 1h day-confirm timeout).  
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
- [ ] Patch Tactical references Main → Book  
- [ ] Push scrubbed Markdown to `github.com/dkontsavoi/grok-first` under `routines/`  
- [ ] Tell First + Second the adoption is live

---

## 12. One-screen cheat sheet

```
LONG-ONLY · 26-coin list · RevX spot
Book q4h → First executes · Main = policy · Second = digs
Day: concrete ping → 1h → silent ≤$1k/≤3 · CATALYST no night-auto · Tier-C=PUMP/VVV/MORPHO/SYRUP
Night 22–08: auto limits ≤$1k/run · ≤$1500 night · ≤3 new symbols
Flush:ON → no Tier-C auto · prefer BTC/ETH
Every buy needs TP/SL/time-stop
Max DD $500 from HWM → pause new risk until DK resumes
Funding = signal only · never short · never freestyle risk
```

---

*End of draft. Reply with edits (especially U1–U8) and I’ll revise this file and propagate to First/Second.*
