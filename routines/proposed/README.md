# Proposed prompt upgrades (review only)

**Status:** PROPOSED — live routines still use:

- `routines/Main-PROMPT.md`
- `routines/Tactical-PROMPT.md`
- `routines/Self-reflect-and-improve-PROMPT.md`

Nothing in this folder is wired to cron/webhooks. Dzmitry reviews before adoption.

## Why

Investigation by **Second** (2026-09-06): buy-the-dip is half-built (flush + LSR hints exist); biggest holes are **exits**, **night portfolio heat**, and **bucket routing** (MOM vs crowding dips). Also ATR-aware zones, Flush gates on alts, LSR hard-skips, and Self-reflect covering **all** 4h Main runs (incl. 00/04/22).

## Files

| Proposed file | Replaces (after approval only) |
|---------------|--------------------------------|
| `Main-PROMPT.proposed.md` | `../Main-PROMPT.md` |
| `Tactical-PROMPT.proposed.md` | `../Tactical-PROMPT.md` |
| `Self-reflect-and-improve-PROMPT.proposed.md` | `../Self-reflect-and-improve-PROMPT.md` |

## Change summary

### Main
- Exit / sell-in-strength ladder (TP1/TP2/runner/time-stop/SL at inv); night sell limits on inventory only
- Night cumulative buy cap (~$1500–2000) on top of per-run $1000; max 3 new symbols; Flush:ON blocks Tier-C auto
- Bucket tags: `CROWD-DIP` | `MOM` | `WATCH` + don't-fade / LSR / funding×crowd rules
- ATR-aware zones by tier A/B/C
- Flush gates alt eligibility
- Complementary mode tags (FLUSH-MR, RS-DIP, CORE-DCA); funding = signal only
- Extended briefs.jsonl fields

### Tactical
- Mirrors buckets, Flush alt gates, exits
- Prefer leave/tighten/cancel when Flush/LSR extremes
- At most one MOM continuation separate from dips
- Paired TP suggestions on inventory

### Self-reflect
- Grade all Main slots including night (00/04/22), not only 08/12/16/20
- Separate DIP vs MOM vs night-auto scoring
- Flag MOM-as-vanilla-dip mistakes

## Adoption (after Dzmitry green-light)

1. Diff proposed vs live; edit if needed.
2. Copy proposed contents over the three live `*-PROMPT.md` files (or rename).
3. Leave JSON schedule files pointing at the same paths (no path change if you overwrite in place).
4. Do **not** enable proposed paths in automation until step 2 is done.
5. Tell First the live prompts were swapped so Main/Tactical pick up the new rules.

## Out of scope in this PR

- No edits to live prompt files
- No JSON/cron pointer changes
- No merge to force adoption
