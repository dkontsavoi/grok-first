# Multi-strategy prompt pack (PROPOSED — not live)

**Status:** Review only. Live Main/Tactical/Self-reflect are unchanged.

## Architecture (Dzmitry 2026-09-06)

| Layer | Role |
|-------|------|
| **Strategy prompts** (4h each) | One focused brief per strategy |
| **Main (orchestrator)** | Deep market-regime analysis → choose working strategy(ies) → compile into one brief + night auto |
| **Tactical (shared)** | One on-demand tactics pass for whatever Main selected |

## Strategies

| ID | File | Intent |
|----|------|--------|
| `BUY_DIP` | `Strategy-BuyTheDip-PROMPT.proposed.md` | Crowding / ATR dips (current core, refined) |
| `FLUSH_MR` | `Strategy-FlushMR-PROMPT.proposed.md` | Buy only after Flush + long-liq dislocation |
| `MOM` | `Strategy-Momentum-PROMPT.proposed.md` | δ-momentum continuation on approved names |
| `RS_DIP` | `Strategy-RSDip-PROMPT.proposed.md` | Dips only in RS leaders vs BTC |
| `CORE_DCA` | `Strategy-CoreDCA-PROMPT.proposed.md` | BTC/ETH core bids + satellite alts opportunistic |
| `CATALYST` | `Strategy-Catalyst-PROMPT.proposed.md` | Pre-placed bids around known events (manual/night-restricted) |

**Not a book strategy:** funding/basis = signal filter inside others (RevX is spot).

## Adoption (later)

1. Review files.
2. Optionally wire separate 4h routines per strategy **or** keep only Main orchestrator calling strategy modules as sections.
3. Point Tactical JSON at shared Tactical when ready.
4. Do **not** overwrite live prompts until green-lit.

## Market snapshot used when drafting (as-of ~2026-09-06)

CMC: F&G ~75 Greed; alt-season ~40 (rising, still BTC-leaning); BTC dom ~59%; OI soft; BTC 24h liqs quiet (~$7.9M, −93% vs prior day); funding mixed/elevated vs recent.  
→ Orchestrator example bias in Main: prefer selective `BUY_DIP` / `RS_DIP` / `CORE_DCA`; de-prioritize chasey `MOM` and thin Tier-C; `FLUSH_MR` on standby until Flush:ON.
