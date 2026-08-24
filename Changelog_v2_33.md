# NATS Bonus Creator — v2.33 Changelog (Draft)

**Release date:** August 24, 2026 (pending PROD dry-run)
**Scope:** Casino Game search fix only — NATS's game search is **hard-capped at ~12 rendered results with no scrollable overflow**, which made the broad pre-™ search (`7's Fire Blitz`) shared by 7FH and 7H2 unreliable: only the state entries that happened to land inside the cap were selectable. v2.33 adds a per-game **`searchOverride`** typed together with the **jurisdiction suffix** (`Hotstepper 2 (NJ)`), returning a near-unique result. Whole-file diff vs v2.32 is **37 added lines, 4 removed** (the 4 removed being replaced docstring closers). `buildFSScript`, `buildRAFScript`, and `buildSUOScript` changed; **all 8 other builders hash-identical to v2.32**. The anchored ™-tolerant match step (`parse_casino_game_pattern`) is unchanged in all three templates. Also fixed the stale page `<title>` (read v2.31 in the v2.32 release; display-only).

---

## Root cause — PROD debugging, Aug 24, 2026

Two v2.32 SUO runs failed at Casino Game selection with 15-second timeouts on the anchored 7H2 pattern — first `(NJ)`, then `(PA)` — while the Aug 21 MI validation had passed with the identical matcher. Three diagnostic scripts (repr-level option dumps, no clicks, no saves) isolated the cause:

1. **`suo_7h2_dropdown_debug_v0_1.py`** — dumped every rendered option for the typed search `7's Fire Blitz` in `ascii()` form. Findings: exactly **12 options rendered**; `7's Fire Blitz™ Hotstepper 2 (MI)` present and regex-matching (ruling out any text/apostrophe defect on the game itself); **NJ/PA Hotstepper 2 entries absent from the DOM**. Side findings: NATS's catalog contains game families with **curly apostrophes** (`7's Fire Blitz™ Power Force 5` — a straight-apostrophe search returns zero results for them) and `DONOTUSE-` prefixed entries (correctly unmatchable by the anchored `^`).
2. **`suo_7h2_dropdown_debug_v0_2.py`** — virtualization test. The narrow query `Hotstepper 2` returned **exactly the 4 state entries, all regex-matching** (MI/NJ/PA/WV — confirming the game exists everywhere). The broad query stayed at 12 options with **no `rc-virtual-list-holder` present and nothing new rendered on scroll** — i.e. not client-side virtualization but a **hard result cap on the search itself**. Scroll-until-match was thereby ruled out as a fix: the missing entries are not below the fold, they are not in the result set.
3. **`suo_7h2_dropdown_debug_v0_3.py`** — suffix-search test. `Hotstepper 2 (NJ)` → **1 result** (the exact NJ entry); `Hotstepper 2 (PA)` → 1 result; `Hotstepper (MI)` → 2 results (plain 7FH MI plus the impostor `Lucky Fire Blitz™ Hotstepper (MI)`, which the anchored pattern correctly rejects); bare `Hotstepper` → 12-capped with 7FH surviving only in NJ and 7H2 only in MI/NJ. Established that NATS's search is **contiguous-substring matching over the full option text, jurisdiction suffix included** — which also means the suffix cannot simply be appended to the existing pre-™ search (`7's Fire Blitz (NJ)` is not a contiguous substring of `7's Fire Blitz™ Hotstepper 2 (NJ)`).

Under v2.20–v2.32, both 7FH and 7H2 were selectable only where result ordering happened to place their entry inside the cap — the Aug 21 7H2 MI validation passed by that luck.

---

## The fix — `searchOverride` + jurisdiction suffix

| Game | v2.32 typed search | v2.33 typed search |
|---|---|---|
| 7FH | `7's Fire Blitz` | `Hotstepper ({JUR})` |
| 7H2 | `7's Fire Blitz` | `Hotstepper 2 ({JUR})` |
| TCE / 7P5 / WWE | pre-™ portion (unchanged) | pre-™ portion (unchanged) |

### Where it was added
1. **`FS_GAMES` UI map** — `searchOverride: "Hotstepper"` on 7FH, `searchOverride: "Hotstepper 2"` on 7H2. Games without the key keep the pre-™ behavior byte-for-byte.
2. **`buildFSScript` dynamic `GAMES` dict** — emits `"searchOverride": "..."` (or `None`) per game at generation time.
3. **RAF and SUO builders' embedded `GAMES` maps** — the same two keys added to both copies (plain strings; no escaping concerns).
4. **All three `parse_casino_game_search` variants** — if the game carries `searchOverride`, the script types `{override} ({jurisdiction})` (RTC FS derives the jurisdiction from the code's `parts[4]`; RAF/SUO use the `JURISDICTION` constant); otherwise the unchanged pre-™ split. `parse_casino_game_pattern` is untouched — the anchored, ™-tolerant, jurisdiction-required match still performs the final selection and still prints the actual option text clicked.

The 7FH override deliberately tolerates non-unique search results (2 hits on MI): the match step, not the search step, owns correctness. The search step's only job is to get the target entry inside the ~12-result cap.

---

## Verification (per the template-literal limitation)
- All three FS builders rendered through a real JS engine (node): SUO with 7H2/NJ (the exact failing case), 7FH/MI, and TCE/WV (no-override regression); RAF with 7H2/PA; RTC FS with mixed 7H2/NJ + 7FH/MI + TCE/WV codes. All 5 outputs passed `py_compile`.
- **28/28 executed checks:** the rendered `parse_casino_game_search` / `parse_casino_game_pattern` functions were executed against the **actual PROD option strings from the debug dumps** — positives (`7's Fire Blitz™ Hotstepper 2 (NJ)` with and without ™, `7's Fire Blitz™ Hotstepper (MI)`, `Triple Cash Eruption™ (WV)` with and without ™) and negatives (`Lucky Fire Blitz™ Hotstepper (MI)`, `Hotstepper High Limit`, `Hotstepper Jackpot Royale™ Express`, `Hotstepper™ Megaways™`, `DONOTUSE-` entries, wrong states, and Hotstepper↔Hotstepper 2 cross-rejection). Typed-search strings asserted: `Hotstepper 2 (NJ)`, `Hotstepper 2 (PA)`, `Hotstepper (MI)`, and unchanged `Triple Cash Eruption`.
- **No-override regression:** the v2.32-rendered vs v2.33-rendered TCE SUO scripts differ only by the two inert `searchOverride` data lines, the expanded docstring, and the override branch that evaluates to `None` — typed search and pattern byte-equal.
- **Builder-level hash comparison:** only `buildFSScript` / `buildRAFScript` / `buildSUOScript` changed; `buildBGScript`, `buildBGREScript`, `buildDMScript`, `buildChurnDMScript`, `buildLCREACTScript`, `buildVIPDMScript`, `buildVIPBGScript`, `buildDMCAScript` all hash-identical to v2.32.
- Whole-file diff vs v2.32: 37 added lines, 4 removed, scoped to the `searchOverride` data, the three `parse_casino_game_search` rewrites, the `buildFSScript` dict emission line, and the title bump.
- **PROD dry-run pending:** a DAY2-only SUO test on **7H2/NJ** (segment + full bonus form through game selection, no save) to the same standard as the v2.32 MI validation, plus one **7FH** state to confirm `Hotstepper ({JUR})` in vivo.

---

## Deploy notes
1. **Free-spins scripts selecting 7FH or 7H2 must be generated from v2.33+.** v2.32-and-earlier scripts for these two games work only in states whose entry happens to fall inside NATS's ~12-result cap (7H2 confirmed working on MI, confirmed failing on NJ and PA). TCE / 7P5 / WWE scripts are behavior-unchanged, and all Casino Credit-only flows are hash-identical — no other pending scripts need regeneration.
2. **⚠️ Residual risk — 7P5 shares the broad search.** `7's Fire Blitz™ Power 5 Jackpot Royale™ Express` also types `7's Fire Blitz` (its pre-™ split), so 7P5 is exposed to the same cap: in the Aug 24 dumps its entries rendered for MI and WV but not NJ/PA. The suffix-append fix does not transfer directly (`Power 5 ({JUR})` is not a contiguous substring — `Jackpot Royale™ Express` intervenes). Candidate override: `Express ({JUR})` (contiguous; would return the small `…Express ({JUR})` family, which the anchored pattern disambiguates) — needs a one-query PROD dropdown test before adding. **WWE** (`WrestleMania`) and **TCE** (`Triple Cash Eruption`) are believed narrow enough to stay under the cap but have not been dump-verified; the v0_1 debug script pattern verifies any game in one run. Until then, build 7P5 only on MI/WV or manually confirm the state's entry appears for `7's Fire Blitz` first.
3. **PROD cleanup from the Aug 24 debugging:** the failed `082426` SUO runs left segments (and any pre-failure day bonuses) in PROD — resume from the failed day with a v2.33-generated script rather than rerunning from day 2, and delete or avoid reusing the orphaned codes. The three debug scripts saved nothing.
4. Doc updates shipped alongside: `Technical_Reference_v2_33.md` (new v2.33 changes bullet; Casino Game Selection section rewritten for the search cap + `searchOverride`; new Known Limitation) and the project instructions (current version, version-warning paragraph, RTC FS games table, Casino Game Selection execution note, Known Limitations #19/#20, verification-history entry).
