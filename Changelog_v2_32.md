# NATS Bonus Creator — v2.32 Changelog (Published)

**Release date:** August 21, 2026
**Scope:** New free-spins game only — 7's Fire Blitz™ Hotstepper 2 (**7H2**), available in RTC Top Up - Free Spins, RAF, and SUO Day 2+ Spins. Whole-file diff vs v2.31 is **33 added lines, zero removed** — the three 7H2 entries only. `buildRAFScript` and `buildSUOScript` changed (each gained the 7H2 entry in its embedded `GAMES` map); **all 9 other builders hash-identical to v2.31**, including `buildFSScript` (RTC FS), which generates its game dict dynamically from `FS_GAMES` and needed no edit. Every Casino Credit-only flow is byte-identical — no pending scripts need regeneration except any that will select 7H2.

---

## New game: 7's Fire Blitz™ Hotstepper 2 (7H2)

| Field | Value |
|---|---|
| Acronym / code token | `7H2` |
| Full Name | 7's Fire Blitz Hotstepper 2 |
| Search Name | `7's Fire Blitz™ Hotstepper 2` |
| Aggregator | WHG |
| Provider | WHITEHATSTUDIOS |
| Deeplink | `/casino_game/WHS_7sFireBlitzHotStepper2US94` |
| Stakes ladder (19 steps) | 0.10, 0.20, 0.40, 0.60, 0.80, 1.00, 1.20, 1.40, 1.60, 1.80, 2.00, 3.00, 4.00, 5.00, 10.00, 20.00, 30.00, 40.00, 50.00 |
| Presets | Mirror 7FH's eleven rows (20–200 spins at 0.10–4.00; all values exist in the 7H2 ladder) |

Ladder values supplied Aug 21, 2026 and normalized to NATS two-decimal option text (`2` → `2.00`, etc.).

### Where it was added
1. **`FS_GAMES` UI map** — the game appears in the Game dropdowns for RTC FS, RAF, and SUO automatically (all three render from `FS_GAMES`).
2. **RAF builder's embedded `GAMES` map** — with the double-escaped `\\u2122` matching the existing entries, per the template-literal escaping rule.
3. **SUO builder's embedded `GAMES` map** — same.

The RTC FS builder (`buildFSScript`) required no edit: its generated `GAMES` dict is built at generation time from `FS_GAMES` (`gamesDict`), so 7H2 flows through with a literal ™ on that path. Both encodings decode to the identical Python string — verified (see below).

### ™-tolerant matching — no matcher changes needed
The v2.20/v2.21 two-step selection handles the new name as-is:
- **Search:** scripts type `7's Fire Blitz` (the pre-™ portion) — the same string 7FH types.
- **Match:** the anchored pattern `^7's Fire Blitz™? Hotstepper 2™?\s*\({JUR}\)$` (every ™ optional, jurisdiction required) selects the new game with or without ™ and **cannot** match the original Hotstepper; 7FH's own anchored pattern cannot match Hotstepper 2. Because the two games share a typed search string, the anchoring in the match step is load-bearing for their disambiguation.

---

## Verification (per the template-literal limitation)
- All three FS builders (`buildSUOScript`, `buildRAFScript`, `buildFSScript`) rendered through a real JS engine (node) with 7H2 selected; each output passed `py_compile`.
- The rendered `parse_casino_game_search` / `parse_casino_game_pattern` functions were extracted from the rendered Python and **executed** against positive and negative option texts: `7's Fire Blitz™ Hotstepper 2 (MI)` ✓ (with and without ™), original Hotstepper ✗, wrong state ✗.
- The `\u2122` escape confirmed surviving JS-literal rendering in the RAF/SUO outputs; the RTC FS output's literal ™ confirmed decoding to the same Python string via `ast.literal_eval`.
- Whole-file diff vs v2.31: 33 added lines, 0 removed, all scoped to the three 7H2 entries. Builder-level hash comparison: RAF + SUO changed, 9 builders identical.
- **PROD-validated Aug 21, 2026** via a DAY2-only SUO test script (`suo_day2_test_7H2_v0_1.py`): DAY2 segment built live, full DAY2 bonus form filled including Aggregator/Provider/Casino Game/deeplink/spin value, stopped before Save Bonus by design. The terminal-printed option text confirmed the matcher clicked the Hotstepper 2 (MI) option.

---

## Deploy notes
1. **Scripts selecting 7H2 must be generated from v2.32+.** Scripts for TCE / 7FH / 7P5 / WWE are unaffected — v2.31-generated scripts for every offer type remain valid.
2. **RTC FS Discover creatives:** the code token is `7H2`, so RTC FS Discover images for this game follow the standard convention `{spins}S_{value}V_7H2.png` (e.g. `20S_10V_7H2.png`) and must be added to the RTC FS creative folder before building RTC FS offers on this game. RAF uses the fixed `Discover_RFRE.png` / `Discover_RFER.png`; SUO has no images.
3. **Test cleanup:** the Aug 21 validation segment `090126_CAS_ACQ_SUO_DAY2_MI_FS_20` exists in PROD (no bonus was saved) — delete it or avoid reusing that code.
4. Doc updates shipped alongside: `Technical_Reference_v2_32.md` (new v2.32 changes bullet + games list in the Casino Game Selection section) and the project instructions (current version, RTC FS games table, verification-history entry).
