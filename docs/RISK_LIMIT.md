# RiskLimit (LOCKED 2026-09-06)

Live file: `/workspace/crypto-self-reflect/risk_limit.json`

## Active under reduce (now)
- transaction_day_limit: **$600** (24h new buys; exits excluded)
- max_day_drop: **$280** from SOD equity (warn $200) → pause new risk
- max_hwm_dd: **$500** (warn $350) → DD_PAUSE
- per_run_limit: **$400**
- tier_c_day_cap: **$200** (Flush: $0)
- max_new_symbols_run: **3**

## Normal (when regime allows)
- transaction_day_limit: **$1000**
- per_run_limit: **$1000**
- tier_c_day_cap: **$400**

## Raise (aggressive path — locked DK 2026-09-06)

Rationale: long-only spot, no leverage, no shorts → climb faster than a leveraged book; starters stay tight.

| Gate | Value |
|------|-------|
| Clean Selfy weeks | **≥2** (was 4) |
| Step size | **+$250** day deploy / **+$200** per_run / **+$50** Tier-C day |
| Cadence | **1 step / week** |
| Skip step if | DD_WARN, rule break, or Flush week |
| Soft ceiling | day deploy ≤ **20%** equity until `reliability_mult` = 1.0 |
| Authority | Main proposes each step → **DK locks** |

