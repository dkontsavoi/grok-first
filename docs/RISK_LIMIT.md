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
- per_run_limit: **$800–1000** (use $1000)
- tier_c_day_cap: **$400**

## Raise
Only after reliability gate (≥4 clean Selfy weeks) — see JSON `raise_gate`.
