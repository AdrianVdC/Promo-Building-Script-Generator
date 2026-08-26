# NATS Bonus Creator — Technical Reference v3.0

## Changes from v2.34 (v3.0)

- **v3.0 — New naming convention for all live offer types.** Every segment / promotion / bonus code now follows the 7-fixed-section skeleton `MMDDYY_JURISDICTION_PRODUCT_LIFECYCLE_SUBCATEGORY_MECHANIC_AWARDTYPE_(FREEFORM)`, parsed positionally in the generated scripts (`parts[0..6]` fixed, everything after the 7th underscore is verbatim freeform). **Voucher codes, image filenames, and Terms ID localStorage keys are completely untouched** — scripts translate from the new codes (K-expansion, tier tables) back to the existing key/filename shapes. Section vocabularies, per-offer formats, K-notation, and the DMCA FS tier index are specified in the **v3.0 Naming Convention** section below. Migration was performed as 8 hash-disciplined increments (one builder at a time, all others verified byte-identical at each step), each verified per the template-literal limitation — rendered through a real JS engine, `ast.parse` + `py_compile`, parse functions executed against positive and negative codes (~250 automated checks total; see `Changelog_v3_0.md` for the full record incl. the ChurnDM 9-field trap and the VIPBG nested-literal escaping catch). Live PROD validation: LC-CHURN-DM, RTC CC, VIPDM, VIPBG, SUO, RAF, and DMCA end-to-end incl. the tier-index code through the Playmaker Existing-Segment attach (Aug 25, 2026).
- **v3.0 — Four offer types retired from the dropdown.** RTC Top Up - Free Spins, Bet & Get, Deposit Match, and Bet & Get - Rules Engine are removed from the offer-type selector. **Their builders remain in the HTML byte-identical to v2.34** (verified against the pristine file) for reference and possible revival; they were deliberately not migrated and would still generate v2.x-format codes if re-exposed. Their Script Steps sections below are retained and marked retired.
- **v3.0 — 40-character NATS name limit enforced.** The NATS bonus name field hard-truncates at 40 characters (established by test 08/25/26 — `TEST_AVDC_BONUS_NAME_LENGTH_NATS_CONFIRM` was the longest accepted string). Every live builder has a generation-time guard that blocks with an alert listing any offending code; SUO and RAF (whose codes are assembled at runtime in Python) additionally carry a runtime assert. Exact-40 watch list: VIPDM `M250_10M2500`, VIPBG `TBG…B100K_G2500`.
- **v3.0 — Promotion Tile selection retry (NATS search-index lag fix).** Root-caused Aug 25, 2026 during the LC-CHURN-DM live validation: NATS's promo search index can lag a just-created promo, timing out the Promotion Tile exact-match wait; an unchanged re-run succeeded. Every live NATS-bonus builder's Promotion Tile selection is now a **6-attempt loop (~70s)** — each attempt re-opens the dropdown, re-types the code, and waits 10s, printing a WARN naming the lag between attempts, with a hard error after attempt 6. Applied to RTC CC, LC-REACT, LC-CHURN-DM, VIPDM, VIPBG, and both RAF blocks (RFRE + RFER). SUO and DMCA have no Promotion Tile. A read-only diagnostic (`debug_promo_tile_v0_1.py`, repr-level dropdown dumps across six search shapes) exists for future promo-search investigation.
- **v3.0 — Positional currency routing.** `currency_for()` reads the jurisdiction directly from position 1 of the code (or takes the bare `JURISDICTION` constant in RAF/SUO) and **raises on unknown jurisdictions** instead of defaulting to USD — a v2.x-shaped code or the reserved `CA` (Cash) awardtype token can no longer misroute the currency row. All migrated parse functions fail loudly on v2.x-shaped codes before any NATS write.

---

## Version History (v2.17 → v2.34)

- **v2.34 — DMCA toast title changed to `Mission Complete!`.** The Daily Missions - Canada Playmaker toast title (Step 5, `input[name='toastTitle']`) is now `Mission Complete!` instead of `Promotion Complete!` — per stakeholder direction. Toast **descriptions** are unchanged (`You have been awarded $X Casino Credit` for CC days; `You have been awarded {spins} Bonus Spins to {game_name}` for FS days, no quotes around the game name), the toast remains always ON, and **BG-RE's toast is untouched** (it still writes `Promotion Complete!`). The three DMCA UI notes (INFO panel, CC day-card note, FS day-card note) were updated to match, and the DMCA generated-script docstring version string was corrected (it had read v2.31 since v2.31). Whole-file diff vs v2.33 is **exactly six lines**: the page `<title>`, the three UI notes, the docstring version, and the `toastTitle` fill inside `buildDMCAScript`; **all 11 other builders untouched**. Verified per the template-literal limitation: the new string contains no backslashes, backticks, or `${` (safe by construction — same class as the v2.29 edit), and the edited line was round-tripped through a real JS template literal with the resulting Python passing `ast.parse`. **v2.31–v2.33 DMCA scripts remain functionally correct** (the old title is cosmetic) — regenerate pending DMCA scripts from v2.34 only if the new toast copy is wanted before their run date.

- **v2.33 — Casino Game search fix: `searchOverride` + jurisdiction suffix (7FH / 7H2).** PROD debugging on Aug 24, 2026 (two SUO 7H2 failures on NJ and PA at Casino Game selection, while the Aug 21 MI validation had passed with the identical matcher) established that **NATS's game search is hard-capped at ~12 rendered results with no scrollable overflow** — not client-side virtualization (no `rc-virtual-list-holder`; scrolling renders nothing new) but a hard cap on the result set itself. The broad pre-™ search `7's Fire Blitz` shared by 7FH and 7H2 matches dozens of catalog entries (Hotstepper, Hotstepper 2, High Limit, Triple Fire, Power 5, Megaways, `DONOTUSE-` variants × 4 states), so only the state entries that happened to land inside the cap were ever selectable — the MI validation passed by luck of result ordering. Three repr-level diagnostic dumps also established that the search is **contiguous-substring matching over the full option text, jurisdiction suffix included** (`Hotstepper 2 (NJ)` → exactly 1 result), that the suffix cannot simply be appended to the pre-™ search (`7's Fire Blitz (NJ)` is not contiguous in `7's Fire Blitz™ Hotstepper 2 (NJ)`), and that the catalog contains curly-apostrophe game families (`7's Fire Blitz™ Power Force 5`) invisible to straight-apostrophe queries plus a `Lucky Fire Blitz™ Hotstepper (MI)` near-collision that the anchored pattern correctly rejects. **Fix:** `FS_GAMES` gains an optional per-game `searchOverride` (7FH: `Hotstepper`; 7H2: `Hotstepper 2`); all three `parse_casino_game_search` variants type `{override} ({jurisdiction})` when the key is present and the unchanged pre-™ split otherwise; `parse_casino_game_pattern` is untouched — the match step still owns correctness, the search step's only job is to get the target entry inside the cap. `buildFSScript` emits the key into its dynamic `GAMES` dict; the RAF and SUO embedded maps carry it directly. Verified per the template-literal limitation: all three FS builders rendered through node (SUO 7H2/NJ — the exact failing case — plus 7FH/MI and TCE/WV regression; RAF 7H2/PA; RTC FS mixed three-game codes), all outputs passed `py_compile`, and **28/28 executed checks** ran the rendered parse functions against the actual PROD option strings from the debug dumps (positives with/without ™; negatives incl. Lucky Fire Blitz, High Limit, Jackpot Royale Express, Megaways, DONOTUSE, wrong states, Hotstepper↔Hotstepper 2 cross-rejection). No-override regression: rendered TCE output differs from v2.32 by inert data lines and docstring only — typed search and pattern byte-equal. Whole-file diff vs v2.32: **37 added lines, 4 removed**; `buildFSScript` / `buildRAFScript` / `buildSUOScript` changed, all 8 other builders hash-identical. Also fixed the stale page `<title>` (read v2.31; display-only). **Free-spins scripts selecting 7FH or 7H2 must be generated from v2.33+.** ⚠️ 7P5 shares the broad `7's Fire Blitz` search and remains exposed to the cap (rendered for MI/WV but not NJ/PA in the Aug 24 dumps); the suffix fix does not transfer directly (`Power 5 ({JUR})` is not contiguous) — candidate override `Express ({JUR})` pending a PROD dropdown test. See **Casino Game Selection** below.

- **v2.32 — New free-spins game: 7's Fire Blitz™ Hotstepper 2 (7H2).** Added to the shared `FS_GAMES` map (UI Game dropdowns for RTC FS, RAF, and SUO) and to the hardcoded `GAMES` maps inside the RAF and SUO Python templates — the RTC FS builder needed no edit because it generates its `GAMES` dict dynamically from `FS_GAMES`. Aggregator **WHG**, Provider **WHITEHATSTUDIOS**, deeplink `/casino_game/WHS_7sFireBlitzHotStepper2US94`, 19-step stakes ladder (0.10–2.00 in the standard WHG steps, then 3.00, 4.00, 5.00, 10.00, 20.00, 30.00, 40.00, 50.00), presets mirroring 7FH's eleven rows. The existing ™-tolerant matcher handles the name without changes: scripts type `7's Fire Blitz` (pre-™ portion) and the anchored regex `^…Hotstepper 2™?\s*\({JUR}\)$` matches the new game with or without ™ while structurally excluding the original Hotstepper (7FH) and wrong jurisdictions — and 7FH's own anchored pattern cannot match Hotstepper 2. Verified per the template-literal limitation: all three FS builders rendered through a real JS engine with 7H2 selected, resulting Python passed `py_compile`, and the rendered `parse_casino_game_search` / `parse_casino_game_pattern` functions were executed directly against positive and negative option texts; the `\u2122` escape confirmed surviving JS-literal rendering in the RAF/SUO outputs (literal ™ on the dynamic RTC FS path — both decode to the identical Python string). **Whole-file diff vs v2.31 is 33 added lines, zero removed** — the three 7H2 entries only; `buildRAFScript` and `buildSUOScript` changed, all 9 other builders hash-identical to v2.31. Validated live in NATS PROD Aug 21, 2026 via a DAY2-only SUO test (segment + full bonus form incl. game selection, no save). Scripts selecting 7H2 must be generated from v2.32+; scripts for the four pre-existing games are unaffected in every offer type.

- **v2.31 — DMCA: NATS-built segments + Include-anchored "Existing Account Segments" attach.** The DMCA flow moves segment creation from Playmaker to NATS. **Phase 1 is now "NATS: Build Segments + Promotions"** — a new `nav_to_segments()` (sidebar `[data-menu-id$='-segments']` → "Account Segments") and `create_segment()` (the standard NATS pattern: AMELCO via `dispatch_event("click")`, `#forBonus` checked, 2s wait before OK) run before the promo loop, one NATS login, each sub-phase independently skippable. DMCA day cards gain a **Segment step badge** (now Segment / Promo / Bonus; Segment defaults ON; `do_segment` carried per code). **Phase 2 Step 2 no longer types into the "Create Account Segments" field** — `attach_existing_segment()` attaches the NATS segment via the **Include Segments → Existing Account Segments** field. Captured Playmaker HTML revealed Step 2 contains **four** cmdk fields (Include/Exclude × Create/Existing) with identical labels across sections and identical `placeholder="Search..."` on both Existing fields, so the lookup is triple-guarded: (1) label-anchored primary — the `<label>` with exact text `Existing Account Segments` whose ancestor section resolves to `Include Segments` (a container mentioning both headings is a shared parent — rejected), followed via its `for=` attribute to the form-item container's `input[cmdk-input]` (dynamic React/Radix ids are deliberately not hardcoded); (2) hardened parent-walk fallback requiring both the Existing-not-Create and Include-not-Exclude ancestor tests; (3) placeholder assertion on both paths (Existing = `Search...`, Create = `Type and press Enter...`). The option whose text **exactly matches** the code name is clicked (never Enter — that is the Create gesture), then the **chip is read back**: a `<span>` exactly equal to the code name must render inside the Include/Existing field within ~5s or the script dumps the Step 2 fields and raises ("refusing to continue on an unverified attach"). NATS → Playmaker segment-sync lag is absorbed by a 12 × 5s (~60s) outer poll; if the segment never appears the script hard-fails — "refusing to fall back to Create Account Segments (would duplicate the NATS segment)". The Exclude Segments section is structurally unreachable. **PROD-validated end-to-end Aug 20, 2026** (segment `101926_CAS_RET_BG_CA_FS_B5000_G100S_300V_BAA` built in NATS and attached via Include → Existing with chip confirmed). Verified per the template-literal limitation: `buildDMCAScript` rendered through a real JS engine with mixed CC + FS codes → `ast.parse` + `py_compile`, 17/17 content checks (all three JS discriminator blocks Include+Exclude aware, no Enter-press in the attach block, create-chip flow fully removed, segments-before-promos ordering, AMELCO/00:01/FS guards untouched); the harness caught one single-backslash `\n` of the v2.20 defect class before ship; all 10 other builders hash-identical to v2.30; whole-file diff confined to the DMCA blocks, step-badge condition, INFO panel, and version strings. **Regenerate all pending DMCA scripts from v2.31** — v2.30 and earlier create a Playmaker-side segment via the Create field, which would duplicate the NATS segment under the new model.

- **v2.30 — DMCA Free Spins days unlocked.** Playmaker Bonus Spins is now functional, so Mon/Wed/Fri/Sun Daily Missions generate end-to-end. FS-day cards show four active **Bet → Spins** tier checkboxes with that weekday's Terms IDs; per-game **Free Spin Stakes ladders** are baked into `DMCA_GAMES` and validated twice — at generation time (red row warning + alert-block if a tier's spin value is not in the weekday game's ladder) and at runtime (the script dumps the live stakes options and **hard-fails, refusing to substitute**, if the exact value is absent — the T&C legally states the spin value). FS Playmaker Step 4: `free_spin` radio → `#fs-count` → Aggregator **PLAYTECH** → Provider **PLAYTECH** → Casino Game (™-tolerant match) → stakes combobox inside `[data-testid='free-spin-stakes-grid']` (no aria-label; **offers CAD for Canada** — the v2.28 USD concern is resolved). **All DMCA Playmaker bonus times moved from 00:00 to 00:01** (12:01 AM → 12:01 AM next day, CC days included), with a verified native-setter → keyboard → fill fallback chain and read-back print; the NATS promo window is unchanged. FS toast: `You have been awarded {spins} Bonus Spins to {game_name}` — **no quotes** around the game name. FS discover images activated: `{Weekday}_B{bet}_G{spins}S_{value}V.png`. FS HIW and the FS T&C template are baked in — **byte-verified against all sixteen filings (CAS_CA_0043–0058)** with two approved normalizations (exactly one space always follows `{game_name}`, correcting the glued-™ and double-space filing defects; trailing whitespace stripped). Phase 2 now runs each bonus in a per-bonus try/except with recovery navigation and a SAVED/FAILED results summary. Step 2 clarification: the segment field is a **"Create Account Segments"** field — typing a name + Enter creates a chip that **creates the segment on Save Bonus** (no pre-existing segment needed; earlier "chip attaches existing segment" wording corrected). PROD-validated Aug 14–19, 2026 (MFB and 4CC full flows incl. a live same-day 081926 build). Verified per the template-literal limitation: `buildDMCAScript` rendered through a real JS engine → `ast.parse` + `py_compile`, **43/43 content checks** (incl. byte-identical FS T&C vs the PROD-validated test template) + **10/10 sandboxed UI checks** through the real `generateScript()` (bad-stake block, missing-Terms-ID block, FS/CC/mixed generation, leap-year pill); all 10 other builders hash-identical to v2.29; whole-file diff confined to DMCA/version blocks; the harness caught a template-literal backtick bug before ship. **FS-day DMCA scripts require v2.30+; regenerate any pending DMCA CC scripts too (v2.28/v2.29 scripts set Playmaker times to 00:00 rather than 00:01).**

- **v2.29 — LC-CHURN-DM: Matched Deposit always checked.** The Churn DM bonus flow now checks the **Matched Deposit** checkbox on every bonus — a new step inserted immediately after the Entitlement Type `Deposit` radio click, copied verbatim from the production-confirmed VIPDM block (explicit 10-second visibility wait, since the checkbox lives inside the DEPOSIT entitlement panel that renders only after the radio is set; terminal prints `OK Matched Deposit: checked`). All other Churn DM behavior unchanged. Also fixed the page `<title>` version drift (read `v2.27` in the v2.28 release; display-only). Verified per the template-literal limitation (inserted block round-tripped through a real JS template literal, resulting Python passed `ast.parse`; whole-file diff vs v2.28 is exactly three edits; VIPDM's own Matched Deposit block and all other builders untouched). Confirmed in a live NATS run Aug 19, 2026. **Churn DM scripts must be generated from v2.29+** — earlier versions leave Matched Deposit unchecked. See "Script Steps — Lifecycle - Churn DM" below.

- **v2.28 — New offer type: Daily Missions - Canada (DMCA).** Twelfth offer type; daily wager-and-get missions for Ontario, forked from BG-RE's two-phase architecture but pointed at **production for both systems** — the first offer type to use production Playmaker (`playmaker-internal.1.betfanatics.com/home`). The typed date's weekday drives everything: Tue/Thu/Sat = Casino Credit (generatable), Mon/Wed/Fri/Sun = Free Spins (**locked** — Playmaker Bonus Spins not yet working; cards render a lock notice and generation is blocked). Phase 1 builds NATS promos (single-day window — typed date 00:00:00 → 23:59:59 ET, a new window pattern; weekday-prefixed underscored image filenames; `{Weekday}'s Daily Mission` title; Bonus Tile ON; Button 2 empty; baked Ontario CC HIW/T&C, window 08/17/26 → 12/31/26). Phase 2 builds Playmaker Casino Credit bonuses through Steps 1–6 **with Save Bonus clicked** (Canada jurisdiction, segment chip by code name, wagering = bet, settlement = get, toast always ON). Both phases use the **Okta Verify (FastPass) login fix**: persistent Chromium profile + `--disable-features=LocalNetworkAccessChecks` + `local-network-access` grant for `betfanatics.okta.com` — confirmed working in PROD. No voucher codes (bonus lives in Playmaker). Step badges Promo + Bonus only. New localStorage key `daily_missions_image_path`. PROD-validated Aug 14, 2026 (promo `120126_CAS_RET_BG_CA_CC_B125_G5` built live; Playmaker dry run through Step 6). Verified per the template-literal limitation: `buildDMCAScript` rendered through a real JS engine → `ast.parse` + `py_compile`, 31/31 content checks, 43/43 sandboxed UI checks via the real `onOfferTypeChange()`; all nine other builders byte-identical to v2.27 (hash-compared). **DMCA scripts must be generated from v2.28+.** See "Script Steps — Daily Missions - Canada" below.

- **v2.27 — VIPBG CA: category-specific Masthead & Promo Detail creatives.** CA — Canada Slots and Tables offers now upload different Masthead and Promo Detail images. In the CA-rendered script's `create_promo`, the category word is derived from the code's `SBG`/`TBG` token via the existing `GAME_CATEGORIES` map (`GAME_CATEGORIES[cat]["hiw"]` → `Slots` / `Tables`) and the uploads become **`Slots Promo Detail.png` + `Slots Masthead.png`** for SBG codes and **`Tables Promo Detail.png` + `Tables Masthead.png`** for TBG codes. The Discover filename is unchanged (`{SBG|TBG}_B{bet}_G{get}.png`), and the generic `Promo Detail.png` / `Masthead.png` are **no longer referenced by CA VIPBG scripts** — the four category files must exist in the CA image folder. **US VIPBG behavior is unchanged** (still generic `Promo Detail.png` / `Masthead.png`; the US template now routes through `promo_detail_filename` / `masthead_filename` variables that resolve to those exact strings — verified identical behavior). No other offer type touched: the whole-file diff vs v2.26 contains only the title bump, the Instructions-panel images bullet, and the CA/US filename block inside VIPBG `create_promo`; all other builders are byte-identical. Verified per the template-literal limitation (rendered through a real JS engine for both regions, Python compiled and executed; CA SBG/TBG codes asserted the category filenames, US asserted the generic ones) and **confirmed live in NATS production for both Slots and Tables on Aug 14, 2026**. CA VIPBG scripts with category-specific creatives must be generated from v2.27+.

- **v2.26 — VIPDM US — United States region.** VIP Offer Library - Deposit Matches is dual-region (mirror of the VIPBG v2.25 pattern): CA keeps the base `vcl_dm_*` keys; US resolves through parallel `_us` keys via new `isVIPDMUS()` / `vipdmKey(base)` helpers. Region dropdown offers **US / CA** and **defaults to US whenever the offer type is selected** (per stakeholder direction — unlike VIPBG, which keeps a valid current selection and defaults CA); switching region silently resets all card values; the edit buttons render **red for CA, blue for US** (reuses `body.ca-region`). US model: **FanCash in USD** — **ten** default tiers keyed `M{min}_{pct}M{max}` mapped `CAS_10677`–`CAS_10686` (adds $250/10%/$500 and $250/10%/$1,000 relative to CA; drops CA's $250/10%/$250), US promotional dates 07/01/26 → 12/31/26. US code `MMDDYY_VCL_RET_DM_US_FC_M{min}_{pct}M{max}` routes all amount fills (reward, Minimum/Maximum Deposit) to the **USD row**; US voucher `VRDMFC{MMDDYY}{XXX}`. US script differences: FanCash bonus type with the standard-DM matched-deposit form (`Bonus (%)` checkbox, `Days to Meet Fancash Settlement`), **standard platform tags (COMBO: Casino kept)**, and a **defensive confirmation-modal handler — a modal is expected for FanCash** (its absence prints an "unexpected — verify" note); the CA-rendered script has no modal block and is functionally byte-identical to v2.25 (one comment line). US HIW is region-scoped (`...up to {max_fmt} FanCash`); US T&C is a single canonical template derived from the CAS_10677/CAS_10678/CAS_10686 legal filings with four stakeholder-approved normalizations ("Fanatics Sportsbook and Casino App" spacing, "Only your first Qualifying Deposit", trailing whitespace stripped, title prefixed `FANATICS CASINO - `). Shared across regions: month-anchored window, `From Your VIP Host` copy, Button 2 `Deposit` → `/auth/account/deposit`, XP default ON / External default OFF, Matched Deposit always checked, Series `VIP` / Type `Retention` / Origin `Opt-In Bonus`, Terms ID enforcement. Verified: **692 automated checks** across US + CA × 6 date edge cases (12 rendered scripts, compiled and executed) plus **35 sandboxed UI checks**, incl. **byte-level comparison of rendered US T&Cs against all three supplied legal filings** (with the approved normalizations applied) and full CA regression (T&C/HIW/voucher outputs identical to v2.25); all nine other builders byte-identical to v2.25. ⚠️ CAS_10679–CAS_10685 (seven of ten filings) were not supplied — the canonical template is assumed to cover them; verify on first use. First live US run should confirm the FanCash confirmation modal fires. **US VIPDM scripts must be generated from v2.26+.** See **VIPDM US Region** below.

- **v2.25 — VIPBG US — United States region.** VIP Offer Library - Bet & Gets is dual-region (RTC CC v2.22 pattern): CA keeps the base `vcl_bg_*` keys; US resolves through parallel `_us` keys via new `isVIPBGUS()` / `vipbgKey(base)` helpers. Region dropdown offers **US / CA** (defaults to CA); switching region silently resets all card values; the edit buttons render **red for CA, blue for US** (reuses `body.ca-region`). US model: **FanCash in USD**, no Slots/Tables differentiation (all US tiers are slots B&Gs) — rows are **Bet → Get** only, keyed `B{bet}_G{get}`, four defaults mapped `CAS_10687`–`CAS_10690`, US promotional dates 07/01/26 → 12/31/26. US code `MMDDYY_VCL_RET_BG_US_FC_B{bet}_G{get}` routes all amount fills to the **USD row**. US script differences: FanCash bonus type (`Days to Meet Fancash Settlement`, `FanCash Amount`), **Button 2 filled `Play Now!` → `/docs/usered/casgenericgamelist`**, **standard platform tags (COMBO: Casino kept)**, no `GAME_CATEGORIES`, confirmation modal **expected** (FanCash). US HIW is region-scoped (US spelling, "Slot games" static, FanCash); US T&C is a single canonical template normalizing four copy-edit inconsistencies between the CAS_10687/CAS_10688 filings (stakeholder-approved wording). Shared across regions: month-anchored window, `From Your VIP Host` copy, XP/External always ON, Series `VIP` / Type `Retention`, `CLBGFC` voucher, Terms ID enforcement. Verified: 742 automated checks across US + CA × 6 date edge cases incl. byte-level US T&C comparison against both legal docs; all nine other builders byte-identical to v2.24; CA VIPBG output functionally byte-identical (comments only). **US VIPBG scripts must be generated from v2.25+.** See **VIPBG US Region** below.

- **v2.24 — New offer type: VIP Offer Library - Bet & Gets (VIPBG).** Eleventh offer type, **CA — Canada only** (Region selector locked to CA; US region planned). Monthly library wager-and-get offers paid in **Casino Credit**: single full-width card, eight **Category (Slots/Tables) → Bet → Get** tiers each mapped to a Terms ID (`CAS_CA_0069`–`CAS_CA_0076`), single Canadian (Ontario) T&C template covering both categories via `{game_category_lower}` / `{game_category_upper}` dynamic fields, with editable promotional dates and Terms IDs via a dedicated modal inside Edit T&Cs. Reuses the VIPDM month-anchored window (typed date − 1 day 00:00:00 ET → month-end 23:59:59 ET + Days to Meet Entitlement). **The wager requirement is enforced upstream (XP/segment) — no bet amount is entered anywhere on the bonus**; the bonus is a flat Casino Credit amount equal to the "get" value (CAD row). Promo hardcodes Title/Header `From Your VIP Host`; **Button 2 label/link are left empty for CA** (`Play Now!` → `/docs/usered/casgenericgamelist` is reserved for the future US region). **Send To XP and External Bonus are both always ON** — no toggles. Bonus: Origin `Opt-In Bonus`, Series `VIP`, Type `Retention` (visible-dropdown scoping as VIPDM), platform tags reduced to **Web + STAC: Standalone Casino only** (all three COMBO tags removed, matching VIPDM), conditional Stake Chunk Sizes fill, defensive confirmation-modal handling. Code name `MMDDYY_VCL_RET_{SBG|TBG}_CA_CC_B{bet}_G{get}`; voucher `CLBGFC{MMDDYY}{XXX}` (shared with BG/BG-RE); generates `vcl_bg_MMDDYY_HHMM.py`. Verified per the template-literal limitation: `buildVIPBGScript` rendered through a real JS engine and the resulting Python compiled and executed for all 8 tiers across 6 date edge cases — 380 automated checks including byte-level verbatim comparison of rendered T&Cs against both legal documents; the verification harness caught (and fixed) a `\$`-escaping bug of the exact class behind the v2.20→v2.21 hotfix. Confirmed end-to-end in NATS production Aug 12, 2026. See **Script Steps — VIP Offer Library - Bet & Gets** below.

- **v2.23 — New offer type: VIP Offer Library - Deposit Matches (VIPDM).** Tenth offer type, **CA — Canada only** (Region selector locked to CA). Monthly library deposit-match offers paid in **Casino Credit**: single full-width card, nine Min → % → Max tiers each mapped to a Terms ID (`CAS_CA_0078`–`CAS_CA_0086`), Canadian (Ontario) T&C base copy with editable promotional dates and Terms IDs via a dedicated modal inside Edit T&Cs. New month-anchored window: start = typed date − 1 day at 00:00:00 ET; end = last day of the typed date's calendar month at 23:59:59 ET **plus the per-card Days to Meet Entitlement**. Promo hardcodes Title/Header `From Your VIP Host` and Button 2 `Deposit` → `/auth/account/deposit`. Bonus: Origin `Opt-In Bonus`, Series `VIP`, Type `Retention`, **Matched Deposit always checked**, and platform tags reduced to **Web + STAC: Standalone Casino only** (unique to this offer type — all three COMBO tags removed). Code name `MMDDYY_VCL_RET_DM_CA_CC_M{min}_{pct}M{max}`; voucher `VRDMCC{MMDDYY}{XXX}`; generates `vcl_dm_MMDDYY_HHMM.py`. All eight pre-existing script builders verified byte-identical to v2.22. See **Script Steps — VIP Offer Library - Deposit Matches** below.

- **v2.22 — RTC CC Canada (CA) support.** RTC Top Up - Casino Credit is dual-jurisdiction: the Region dropdown is restricted to **US / CA** (all other offer types unchanged), and with CA selected all RTC CC settings resolve through parallel `_ca` localStorage keys via the `isRTCCanada()` / `rtcKey()` helpers — Canadian image folder, Canadian (Ontario) T&C base copy, CA Terms IDs `CAS_CA_0001`–`0011`, and CA promotional dates 08/17/26 → 12/31/26. HIW is shared between jurisdictions. The three edit buttons turn red when CA is selected, and changing region resets all day cards. US behavior is unchanged. See **RTC CC Canada (CA) Support** below.

- **v2.18 — NATS multi-currency amount tables.** A NATS platform update converted the single bonus amount input into a three-row table with one row per currency (`USD` / `GBP` / `CAD`), each keyed by `data-row-key` on the `<tr>`. The old page-wide selector matched 3 elements and crashed with a Playwright strict mode violation. All generated scripts now include `currency_for(name)` and `fill_currency_amount()` helpers; the amount is filled into the jurisdiction's currency row only. Patched: RTC CC, LC-REACT, BG amounts; DM and LC-CHURN-DM amount cards plus Minimum/Maximum Deposit cards (whose `.first` selectors would have silently filled the USD row on Canadian runs). See **Multi-Currency Amount Tables** below.
- **v2.19 — Free-spin flows made currency-aware.** RTC FS, RAF, and SUO spin-value row selection changed from a hardcoded `USD` label anchor to the `currency_for()` helper (RTC FS from the code name; RAF/SUO from the script's `JURISDICTION` constant). No behavior change today — all three are MI/NJ/WV/PA only — this future-proofs any Canadian free-spins expansion.
- **v2.20 — Casino Game selection made ™-tolerant.** NATS renamed casino games to include a trademark symbol (e.g. `Triple Cash Eruption™ (WV)`), breaking the typed search + exact match in all three free-spin flows. Scripts now type only the pre-™ portion of the search name and match options with an anchored regex where every `™` is optional but the jurisdiction suffix is required. See **Casino Game Selection** below.
- **v2.21 — Hotfix for v2.20.** The v2.20 regex was written with single backslashes inside the JS template literals that generate the Python scripts; JS rendering swallowed them, producing a regex that could never match. Backslashes doubled in all three pattern functions and the fix verified by rendering each template through a real JS template literal and executing the resulting Python. **v2.20 must not be used.**

---

## HTML Tool Layout

- **Left column (340px fixed):** Branding, primary action buttons, Offer Type selector, Region selector (hidden for RTC FS, RAF, SUO, and BGRE; restricted to US / CA for RTC CC as of v2.22; restricted to US / CA for VIPDM as of v2.26, defaulting to US on selection; restricted to US / CA for VIPBG as of v2.25)
- **Right column (flex):** Setup guide link panel, offer-type-specific info panel, and run instruction cards
- **Below header:** Day cards grid (or single campaign card for RAF and SUO)
- **Below day cards:** Generated script output area (hidden until script is generated)

---

## Global Controls

**Generate Script (white button)**
Reads all day card inputs, builds internal code names (v3.0 format — see Naming Conventions), enforces the 40-character NATS limit with a blocking alert, generates and auto-downloads a Python script. **The offer-type dropdown lists only the eight live offer types as of v3.0** (RTC CC, RAF, SUO, LC-REACT, LC-CHURN-DM, VIPDM, VIPBG, DMCA). For RAF and SUO: enabled when date (6 chars), Spins/Day > 0, Bet Amount > 0, and Spin Value are all filled.

**Edit HIW / Edit T&Cs**
Hidden for RAF, SUO, and DMCA. Active for RTC CC, LC-REACT, LC-CHURN-DM, VIPDM, and VIPBG (and the retired RTC FS / BG / DM if re-exposed). For RTC CC, Edit T&Cs is **jurisdiction-aware** (v2.22): it reads and saves the selected region's T&C (`rtc_tc` vs `rtc_tc_ca`) and displays the region in the modal title. Edit HIW is shared across US and CA (single `rtc_hiw` key) by design. For **VIPDM** (v2.23), the Edit T&Cs modal shows the Terms bar with an **Edit Terms Expiry & Terms IDs** button routed to the VIPDM terms modal — promotional start/end dates (`{start_date_long}` / `{end_date_long}`) plus one Terms ID input per offer key `M{min}_{pct}M{max}`. For **VIPBG** (v2.24/v2.25), the same Terms bar routes to the VIPBG terms modal — promotional start/end dates plus one Terms ID input per offer key (CA: `{SBG|TBG}_B{bet}_G{get}`; US: `B{bet}_G{get}`). As of v2.25 all VIPBG modals are **region-aware**: Edit HIW, Edit T&Cs, the Terms modal, and Edit Images read/save the selected region's keys (`vcl_bg_*` for CA, `vcl_bg_*_us` for US) and show the region in their titles — note VIPBG HIW is region-scoped, unlike RTC CC where HIW is shared.

**Edit Images**
Present for RTC CC, RTC FS, RAF, LC-REACT, LC-CHURN-DM, VIPDM, and VIPBG. Hidden for BG, DM, SUO, and BG-RE. For RTC CC, jurisdiction-aware (v2.22): reads/saves `rtc_image_path` vs `rtc_image_path_ca`, shows the region in the modal title, and the Open Shared Images Folder link switches between the US and Canadian Google Drive folders. For VIPBG, jurisdiction-aware the same way as of v2.25 (`vcl_bg_image_path` vs `vcl_bg_image_path_us`, region in title, Drive link follows region).

**+ Add Day**
No-op for RAF, SUO, VIPDM, and VIPBG (single card each). Max 99 for RTC CC / LC-REACT / LC-CHURN-DM (the retired BG/DM capped at 8).

**Clear Saved Settings**
Clears all localStorage keys including `lc_react_image_path`, the `lc_churn_dm_*` keys, all RTC CC `_ca` keys (v2.22), all `vcl_dm_*` keys (v2.23), all `vcl_bg_*` keys (v2.24), and all `vcl_bg_*_us` keys (v2.25).

---

## Day Cards

### RTC Top Up - Casino Credit
Grid: 3 columns | Default days: 9
Date, amounts grid (presets + custom), step badges (Segment/Promo/Bonus), Send to XP, Days to Opt In / Entitlement / Settlement.
**Region (v2.22):** US — United States (default) or CA — Canada only. Changing the region in either direction calls `resetAll()` — all day-card values (dates, selected amounts, toggles) are cleared and any generated script output is hidden, so US and CA inputs can never mix. The reset is silent (no confirmation dialog, by design). With CA selected, the Edit HIW / Edit T&Cs / Edit Images buttons render in red shades instead of blue.

### RTC Top Up - Free Spins
Grid: 4 columns | Default days: 8
Date, Region per card (MI/NJ/WV/PA), Game dropdown, offers list (spins × spin value), step badges, Send to XP, Days to Opt In / Entitlement / Settlement.

### Bet & Get
Grid: 4 columns | Default days: 4
Date, ZIP attachment per card, offers list (bet/get rows), step badges, Send to XP, External Bonus checkbox, Days to Opt In / Entitlement / Settlement.

### Deposit Match
Grid: 4 columns | Default days: 4
Date, Campaign Name, ZIP attachment, offers list (pct/max rows), step badges, Send to XP, Days to Opt In / Entitlement / Settlement.

### Refer a Friend
Single campaign card (full width, two-column layout).
Title: **Campaign Configuration**
- **Left:** Date (MMDDYY), Jurisdiction (MI/NJ/WV/PA), Game dropdown, Spins/Day, Spin Value, Bet Amount, Number of Days toggle (5 or 10), phase badges (Day 2+, Referee, Referrer)
- **Right:** Live preview panel — derived RFRE/RFER/DAY2–N code names, total spins, spin value, bet, game
- Generate button enables when: date filled (6 chars) + Spins/Day > 0 + Bet Amount > 0 + Spin Value selected

### Bet & Get - Rules Engine
Grid: 4 columns | Default days: 4
Date, ZIP attachment per card, offers list (bet/get rows), Reward Type dropdown (FC/CC), Jurisdiction dropdown (USA/Canada/MI/NJ/WV/PA/ON), Days to Use Reward (default 7), Show Toast checkbox (default on), Send to XP. Step badges: **Promo + Bonus only** (no Segment — segment is built inside Playmaker as part of the bonus flow).

> ⚠️ NATS and Playmaker URLs are currently pointed at the **test environment**. Switch to production URLs once the Rewards Engine is live in the Playmaker production environment.

### SUO Day 2+ Spins
Single campaign card (full width, two-column layout).
Title: **Campaign Configuration**
- **Left:** Date (MMDDYY), Jurisdiction (MI/NJ/WV/PA), Game dropdown, Spins/Day, Spin Value, Bet Amount, Number of Days toggle (5 or 10)
- **Right:** Live preview panel — all DAY2–DAYN code names, spin value, bet, game, total code count
- No phase badges — segment and bonus always built for every day code
- Generate button enables when: date filled (6 chars) + Spins/Day > 0 + Bet Amount > 0 + Spin Value selected

### Lifecycle - REACT CC Drop
Grid: 3 columns | Default days: 3
Date, amounts grid (shows only `$10`, `$25`, `$50` — all unselected by default, custom amounts can be added), step badges (Segment/Promo/Bonus), Send to XP, Days to Opt In / Entitlement / Settlement (all default to 3).

### Lifecycle - Churn DM
Grid: 4 columns | Default days: 4
Date, offers list (**Min → % → Max** rows with per-offer checkbox — five defaults shown unselected: $10/50%/$10, $20/50%/$25, $20/50%/$50, $50/50%/$100, $50/50%/$200), + Add / Select All, step badges (Segment/Promo/Bonus), Send to XP (default ON), External Bonus (default OFF), Days to Opt In (3) / Entitlement (3) / Settlement (7). No Campaign Name field and no ZIP attachment — Title, Promo Header, and Bonus Description are fixed as `Deposit and Get Casino Credit`, and images come from a locally-synced folder.

**Terms ID enforcement:** each active offer key `M{min}_{pct}M{max}` must exist in the baked-in `CHURNDM_TERMS_IDS` map. A non-matching active offer shows a red warning on the row ("No Terms ID configured — generation blocked") and the Generate button raises an alert and refuses to build the script.

### VIP Offer Library - Deposit Matches (v2.23 CA / v2.26 US)
Single card (full width, max 620px). Title: **VIP Offer Library — {region}** (region-aware as of v2.26). `+ Add Day` is a no-op. Region selector offers **US / CA** (v2.26; **always defaults to US when the offer type is selected**, per stakeholder direction); **switching region silently resets all card values** and the edit buttons render **red for CA, blue for US**.
Date (MMDDYY, free-typed — see window rules below), offers list (**Min → % → Max** rows with per-offer checkbox — each row displays its Terms ID inline, red `no ID` when unmapped), + Add / Select All, step badges (Segment/Promo/Bonus), Send to XP (default ON), External Bonus (default OFF), Days to Opt In (3) / Days to Entitlement (3 — **extends the promo/bonus end date**) / Days to Settlement (7). No Campaign Name field and no ZIP attachment — Title, Promo Header, and Bonus Description are fixed as `From Your VIP Host`, and images come from a locally-synced folder (one per region).

**CA offers:** nine defaults shown unselected: $100/50%/$250, $100/100%/$250, $250/10%/$250, $250/10%/$2,500, $250/10%/$5,000, $250/20%/$500, $250/20%/$1,000, $250/20%/$2,000, $250/20%/$5,000 (Terms IDs `CAS_CA_0078`–`CAS_CA_0086`).

**US offers (v2.26):** ten defaults shown unselected: $100/50%/$250, $100/100%/$250, $250/10%/$500, $250/10%/$1,000, $250/10%/$2,500, $250/10%/$5,000, $250/20%/$500, $250/20%/$1,000, $250/20%/$2,000, $250/20%/$5,000 (Terms IDs `CAS_10677`–`CAS_10686`; relative to CA, adds $250/10%/$500 and $250/10%/$1,000 and drops $250/10%/$250).

**Terms ID enforcement (both regions):** identical to Churn DM — each active offer key `M{min}_{pct}M{max}` must exist in the selected region's VIPDM Terms ID map (editable per region via the Edit T&Cs modal's Terms bar). Active offers without an ID show a red row warning and block script generation with an alert.

### VIP Offer Library - Bet & Gets (v2.24 CA / v2.25 US)
Single card (full width, max 680px). Title: **VIP Offer Library — Bet & Gets — {region}** (region-aware as of v2.25). `+ Add Day` is a no-op. Region selector offers **US / CA** (v2.25; keeps a valid current selection, otherwise defaults to CA); **switching region silently resets all card values** and the edit buttons render **red for CA, blue for US**.
Date (MMDDYY, free-typed — same month-anchored window rules as VIPDM, both regions), offers list, + Add / Select All, step badges (Segment/Promo/Bonus), Days to Opt In (3) / Days to Entitlement (3 — **extends the promo/bonus end date**) / Days to Settlement (7). **Send to XP and External Bonus have no toggles — both are always ON** (static note on the card; both regions). No Campaign Name field and no ZIP attachment — Title, Promo Header, and Bonus Description are fixed as `From Your VIP Host`, and images come from a locally-synced folder (one per region).

**CA offers:** **Category → Bet → Get** rows with a per-row Slots/Tables dropdown and per-offer checkbox — eight defaults shown unselected: Slots $1,000/$100, $5,000/$500, $20,000/$1,000, $100,000/$5,000; Tables $2,000/$100, $10,000/$500, $25,000/$1,000, $100,000/$2,500 — each row displays its Terms ID inline, red `no ID` when unmapped.

**US offers (v2.25):** **Bet → Get** rows only — no category dropdown; all US tiers are slots B&Gs. Four defaults shown unselected: $1,000/$100, $5,000/$500, $20,000/$1,000, $100,000/$5,000 — Terms IDs `CAS_10687`–`CAS_10690` inline.

**Terms ID enforcement (both regions):** identical to VIPDM — each active offer key must exist in the selected region's VIPBG Terms ID map (CA: `{SBG|TBG}_B{bet}_G{get}`, defaults `CAS_CA_0069`–`CAS_CA_0076`; US: `B{bet}_G{get}`, defaults `CAS_10687`–`CAS_10690`; both editable via the Edit T&Cs modal's Terms bar). Active offers without an ID show a red row warning and block script generation with an alert. CA keying includes the category prefix as collision insurance (a Slots and Tables tier could otherwise share a `B{bet}_G{get}` shape); US keys need no prefix since there is a single category.

### Daily Missions - Canada (v2.28; FS unlocked v2.30; NATS segments v2.31)
Grid: standard | Default days: 3
Date (MMDDYY) with a live **weekday pill** rendered beside the input as soon as a valid date is typed — green `{Weekday} • Casino Credit` for Tue/Thu/Sat, amber `{Weekday} • Free Spins — {game}` for Mon/Wed/Fri/Sun. Step badges: **Segment + Promo + Bonus** as of v2.31 (Segment defaults ON — the segment is built in NATS and attached in Playmaker via Include → Existing Account Segments). Region row hidden (fixed CA/Ontario); Edit HIW / Edit T&Cs buttons hidden (copy baked in); Edit Images visible.

**CC-day state (Tue/Thu/Sat):** four **Bet → Get** tier rows with per-offer checkboxes and inline Terms IDs — $50→$2 `CAS_CA_0019`, $125→$5 `CAS_CA_0020`, $500→$25 `CAS_CA_0021`, $5,000→$300 `CAS_CA_0022` (shared across all three CC weekdays); Days to Use Reward (default 7, per card); static notes for the always-ON toast and the weekday-derived image filenames.

**FS-day state (Mon/Wed/Fri/Sun — unlocked v2.30):** four active **Bet → Spins** tier rows with per-offer checkboxes, showing that weekday's Terms IDs inline ($50/20 spins @ $0.10, $125/50 @ $0.10, $500/50 @ $0.50, $5,000/100 @ $3.00; Mon `CAS_CA_0043`–`0046`, Wed `0047`–`0050`, Fri `0051`–`0054`, Sun `0055`–`0058`); Days to Use Reward (default 7, per card, shared with CC state); static notes for the FS toast and the FS image filenames. A tier whose spin value is not in the weekday game's stakes ladder shows a red `$X not in {token} stakes` warning on the row and blocks generation with an alert; a tier without a Terms ID for that weekday also blocks. FS offer state lives in `days[i].dmcaFSOfferData`, separate from the CC state, so a date edit that flips the weekday never mixes tiers.

Weekday→game map (`DMCA_GAMES` — each entry carries `name`, `token`, `search` (the pre-™ Casino Game search string), and `stakes` (that game's Free Spin Stakes ladder, captured from PROD Playmaker Aug 2026)): Monday = Baa Baa Baa™ (BAA, `Baa Baa Baa`), Wednesday = 4 Crazy Cluckers™ (4CC, `4 Crazy Cluckers`), Friday = Mega Fire Blaze™: Legacy of the Tiger™ (MFB, `Mega Fire Blaze`), Sunday = Blue Wizard: Cash Collect & Link™ (BWZ, `Blue Wizard`). Aggregator and Provider are **PLAYTECH / PLAYTECH** for all four games. Stakes ladders: BAA / 4CC / BWZ share the full 32-step ladder (0.10–1.00 by 0.10, 1.50–5.00 by 0.50, 6.00–10.00 by 1.00, 20.00–100.00 by 10.00); **MFB caps at 50.00** (27 steps, no 60–100). All four ladders contain the three tier values 0.10 / 0.50 / 3.00.

---

## RAF Phase Badges

| Badge | Phases covered |
|---|---|
| Day 2+ | Phase 1 (DAY2–N segments) + Phase 3 (DAY2–N bonuses) |
| Referee | Phase 2 (RFRE promo) + Phase 4 (RFRE bonus + edit) |
| Referrer | Phase 1 (RFER segment) + Phase 2 (RFER promo) + Phase 5 (RFER bonus + edit) |

Phase 1 skipped if Day 2+ and Referrer both off. Phase 2 skipped if Referee and Referrer both off.

---

## Per-Code Step Toggles (RTC CC / FS / BG / DM / LC-REACT / LC-CHURN-DM / VIPDM / VIPBG / DMCA)

Each day card shows three toggleable step badges: **Segment**, **Promo**, **Bonus**. The generated script includes `DO_SEGMENT`, `DO_PROMO`, and `DO_BONUS` boolean arrays. `main()` skips a phase entirely if the filtered name list for that phase is empty.

SUO Day 2+ has no step toggles — segment and bonus are always built for every code.

BG-RE shows **Promo and Bonus** badges only — no Segment badge (its segment is created inside Playmaker as part of the bonus flow). DMCA gained a **Segment** badge in v2.31: its segment is built in NATS during Phase 1 and attached in Playmaker Step 2 via the Include → Existing Account Segments field; `do_segment` is carried per code and Phase 1's segment sub-phase is skipped if no code has it enabled.

---

## Run Instruction Cards

| Platform | Command format |
|---|---|
| Mac — Terminal | `cd ~/Downloads && python3 {filename}` |
| Windows — Command Prompt | `cd %USERPROFILE%\Downloads && python {filename}` |

---

## Voucher Code System

**Format key — RTC / BG / DM / LC-REACT:**

| Position | Value | Meaning |
|---|---|---|
| 1 | `C` | Casino |
| 2 | `R` / `L` | Retention / Lifecycle |
| 3–4 | `BG` / `DM` / `RC` / `RL` | Offer type |
| 5–6 | `FC` / `CC` / `FS` | Reward type |
| 7–12 | `MMDDYY` | Date |
| 13–15 | `XXX` | Random suffix |

**VIPDM (v2.23 CA / v2.26 US):** CA: `VRDMCC{MMDDYY}{XXX}`; US: `VRDMFC{MMDDYY}{XXX}` — **V**IP + **R**etention + **DM** + reward token (**CC** Casino Credit / **FC** FanCash); mirrors standard DM's `CRDMFC` grammar with the vertical letter swapped.

**VIPBG (v2.24):** `CLBGFC{MMDDYY}{XXX}` — **shared with standard BG and BG-RE** (NATS-mandatory but unused for this offer type; the code-name category token distinguishes the offers).

**Daily Missions - Canada (v2.28):** **No voucher codes** — the bonus is built in Playmaker, which has no voucher concept, and the NATS side builds only the promotion.

**Lifecycle voucher codes:**

| Offer | Format | Example |
|---|---|---|
| LC-REACT | `CRLCCC{MMDDYY}{XXX}` | `CRLCCC122926AB3` |
| LC-CHURN-DM | `CRLCDM{MMDDYY}{XXX}` | `CRLCDM121126X4T` |

**Format key — RAF:**

| Position | Value | Meaning |
|---|---|---|
| 1–2 | `MI` / `NJ` / `WV` / `PA` | Jurisdiction |
| 3–4 | `AQ` | Acquisition |
| 5–8 | `0500` (zero-padded) | Total spins |
| 9–12 | `RFRE` / `RFER` / `DAY2` etc. | Group |
| 13–15 | `XXX` | Random suffix |

RAF voucher codes generated once at startup. RFRE/RFER reused across all modal entries. Each Day 2+ code gets its own unique voucher. All printed to terminal before login.

**SUO Day 2+ Spins:** No voucher codes. These are No Opt In bonuses.

> ⚠️ Voucher codes in NATS can never be reused even after deletion.

---

## Eastern Time Enforcement

All generated scripts use Python's `zoneinfo` module (Python 3.9+). `astimezone()` converts Eastern-aware datetimes to local machine timezone. DST handled automatically.

---

## Script Execution Flow

### RTC CC / LC-REACT / LC-CHURN-DM / VIPDM / VIPBG (and retired FS / BG / DM)
1. Open Chromium, navigate to trading platform
2. Pause for manual login
3. **Segments** — hover `-segments` sidebar icon, click "Account Segments", create segments
4. **Promos** — hover `-cms` sidebar icon, click "Promos", create promos
5. **Bonuses** — hover `-bonuses` sidebar icon, click "Bonus Manager", create bonuses
6. macOS notification + close browser

**Promotion Tile retry (v3.0):** every live builder that selects a Promotion Tile (RTC CC, LC-REACT, LC-CHURN-DM, VIPDM, VIPBG, RAF RFRE + RFER) wraps the selection in a 6-attempt loop (~70s): each attempt re-opens the dropdown, re-types the code name, and waits 10s for the anchored exact match; a WARN naming NATS search-index lag prints between attempts, and a hard error is raised after attempt 6. Root cause: NATS's promo search index can lag a just-created promo (observed Aug 25, 2026).

### RAF
1. Open Chromium, navigate to trading platform
2. Pause for manual login — all voucher codes printed before this
3. **Phase 1 — Segments** — RFER + DAY2–N (conditional on badges)
4. **Phase 2 — Promos** — RFRE then RFER (conditional on badges)
5. **Phase 3 — Day 2+ Bonuses** — DAY2–N loop (conditional on Day 2+ badge)
6. **Phase 4 — RFRE Bonus + Post-Save Edit** (conditional on Referee badge)
7. **Phase 5 — RFER Bonus + Post-Save Edit** (conditional on Referrer badge)
8. Close browser

### SUO Day 2+ Spins
1. Open Chromium, navigate to trading platform
2. Pause for manual login
3. **Phase 1 — Segments** — DAY2–N loop
4. **Phase 2 — Bonuses** — DAY2–N loop
5. Close browser

### Bet & Get - Rules Engine
1. Open Chromium (two pages), navigate to NATS test environment
2. Pause for manual NATS login
3. **Phase 1 — NATS Promos** — all promos built before Playmaker opens
4. Open Playmaker test environment in second page
5. **Phase 2 — Playmaker Bonuses** — per-code loop: Login (SSO) → iCasino → Bonus Setup → Create Bonus → Steps 1–6 → Save Bonus
6. Close browser

---

## Script Steps — Lifecycle - REACT CC Drop

### `create_segment(page, name)`
Identical to RTC CC — AMELCO `dispatch_event("click")`, 2-second post-`#forBonus` wait.

### `create_promo(page, name)`
Poll Create Promotion (120 × 500ms, up to 60s) → click → fill name → Start: `00:00:00 ET` day-of → End: `23:59:59 ET` day+2 (72-hour window) → Type: `Image only CTA` → Layout: `Overlay` → first Save → upload `Promo Detail.png`, `Masthead.png`, `{amount}.png` from `IMAGE_FOLDER` → Title: `Get $X in Casino Credit` → Promo Header Text: `Get $X in Casino Credit` → Bonus Tile toggle ON → How it works (static template) → T&C (per-amount, baked in at generation time) → second Save. No confirmation modal.

### `create_bonus(page, name, idx)`
1. Poll Create Bonus (120 × 500ms, up to 60s)
2. Trigger: `Opt In`
3. Internal name = code name; Description = `Get $X in Casino Credit`
4. Send To XP (conditional on per-card toggle)
5. Remove `COMBO: Sportsbook` and `COMBO: Sportsbook And Casino` platform tags
6. Voucher Code: `CRLCCC{MMDDYY}{XXX}`
7. Bonus type: `Casino Credit`
8. Days to Meet Casino_credit Settlement = per-card value (default 3)
9. Days to Opt In = per-card value (default 3)
10. Days to Meet Entitlement = per-card value (default 3)
11. Stake Chunk Sizes = 1
12. Amount = parsed from code name — filled into the jurisdiction's currency row via `fill_currency_amount()`
13. Status Active = checked
14. Activation: `00:00:00 ET` day-of → `23:59:59 ET` day+2 (72-hour window, identical to promo window)
15. Segment / Client Profiling — anchored regex scoped to `.ant-select-dropdown:not(.ant-select-dropdown-hidden)`
16. Reporting Platform: `CAS (Casino)`
17. Bonus Origin: `Opt-In Bonus`
18. Series: `Reactivation` (typed search)
19. Type: `Lifecycle`
20. Promotion Tile — anchored regex exact match (same pattern as all other offer types)
21. Save — **no confirmation modal**

### Fixed Values

| Field | Value |
|---|---|
| Trigger | Opt In |
| Stake Chunk Sizes | 1 |
| Status Active | Checked |
| Reporting Platform | CAS (Casino) |
| Bonus Origin | Opt-In Bonus |
| Series | Reactivation |
| Type | Lifecycle |
| Confirmation modal | None |

### Activation Window

| | LC-REACT | RTC CC |
|---|---|---|
| Start | `00:00:00 ET` day-of | `04:00:00 ET` day-of |
| End | `23:59:59 ET` day+2 (+72h) | `04:00:00 ET` day+1 (+24h) |

Promo window matches bonus activation window exactly.

---

## Script Steps — Lifecycle - Churn DM

### `create_segment(page, name)`
Identical to RTC CC / DM — AMELCO `dispatch_event("click")`, 2-second post-`#forBonus` wait.

### `create_promo(page, name)`
Poll Create Promotion (120 × 500ms, up to 60s) → click → fill name → Start: `00:00:00 ET` day-of → End: `00:00:00 ET` **day+10** (exactly 10 days) → Type: `Image only CTA` → Layout: `Overlay` → first Save → upload `Promo Detail.png`, `Masthead.png`, `{pct}M{max}.png` from `IMAGE_FOLDER` → Promo Header Text: `Deposit and Get Casino Credit` → Title: `Deposit and Get Casino Credit` → Bonus Tile toggle ON → **no Button 2 label/link** → How it works (template with `{pct}` / `{max_fmt}`) → T&C (full legal copy, templated with `{min_fmt}` / `{pct}` / `{max_fmt}` / `{terms_id}`, baked in at generation time) → second Save. No confirmation modal.

### `create_bonus(page, name, idx)`
1. Poll Create Bonus (120 × 500ms, up to 60s)
2. Trigger: `Opt In`
3. Internal name = code name; Description = `Deposit and Get Casino Credit`
4. Send To XP (per-card toggle, default ON); External Bonus (per-card toggle, default OFF)
5. Remove `COMBO: Sportsbook` and `COMBO: Sportsbook And Casino` platform tags
6. Voucher Code: `CRLCDM{MMDDYY}{XXX}`
7. Bonus type: `Casino Credit`
8. Check **`Casino Credit (%)`** checkbox (CC-form equivalent of DM's `Bonus (%)`)
9. Fill percentage input (`aria-valuemin='0' aria-valuemax='100'`) with `{pct}` from code
10. Days to Meet Casino_credit Settlement = per-card value (default 7)
11. Days to Opt In = per-card value (default 3)
12. Days to Meet Entitlement = per-card value (default 3)
13. Entitlement Type: `Deposit` (JS radio click)
14. **Matched Deposit checkbox — always checked** (v2.29; lives inside the DEPOSIT entitlement panel; explicit visibility wait before clicking since the panel renders after step 13 — same block as VIPDM)
15. Summary percentage field — tries `Bonus %` then `Casino Credit %` label
16. Bonus amount summary card — tries `Bonus` then `Casino Credit` card title; fill with `{max}` from code into the jurisdiction's currency row
17. Minimum Deposit = **dynamic** `{min}` from code (`M{min}` segment) — jurisdiction currency row
18. Maximum Deposit = `1000000` (hardcoded) — jurisdiction currency row
19. Status Active = checked
20. Activation: `00:00:00 ET` day-of → `00:00:00 ET` day+10 (identical to promo window)
21. Segment / Client Profiling — anchored regex scoped to `.ant-select-dropdown:not(.ant-select-dropdown-hidden)`
22. Promotion Tile — anchored regex exact match
23. Reporting Platform: `CAS (Casino)`
24. Bonus Origin: `Retention`
25. Series: `Other` (typed search)
26. Type: `Lifecycle`
27. Save — **no confirmation modal**

### Fixed Values

| Field | Value |
|---|---|
| Trigger | Opt In |
| Description | Deposit and Get Casino Credit |
| Matched Deposit | Always checked (v2.29) |
| Minimum Deposit | From code (`M{min}`) |
| Maximum Deposit | 1000000 |
| Status Active | Checked |
| Reporting Platform | CAS (Casino) |
| Bonus Origin | Retention |
| Series | Other |
| Type | Lifecycle |
| Confirmation modal | None |

### Activation / Promo Window

| | LC-CHURN-DM |
|---|---|
| Start | `00:00:00 ET` day-of |
| End | `00:00:00 ET` day+10 (exactly +240h) |

Promo window matches bonus activation window exactly.

### LC-CHURN-DM T&C Data (Baked In at Generation Time)

Offer key = `M{min}_{pct}M{max}`. All five default offers share one promotional period: **June 1, 2026 → December 1, 2026**.

| Offer key | Min | % | Max CC | Terms ID |
|---|---|---|---|---|
| `M10_50M10` | $10 | 50 | $10 | `CAS_10620` |
| `M20_50M25` | $20 | 50 | $25 | `CAS_9457` |
| `M20_50M50` | $20 | 50 | $50 | `CAS_9458` |
| `M50_50M100` | $50 | 50 | $100 | `CAS_9459` |
| `M50_50M200` | $50 | 50 | $200 | `CAS_9460` |

T&C title line renders as: `FANATICS CASINO - DEPOSIT {min_fmt}+, GET {pct}% DEPOSIT MATCH UP TO {max_fmt} IN CC - {terms_id}`. Changes to the T&C copy or Terms IDs require regenerating the script (same known limitation as DM and LC-REACT).

---

## Script Steps — VIP Offer Library - Deposit Matches (v2.23 CA / v2.26 US)

Dual region as of v2.26. CA codes carry the `CA` token, so all currency-table fills land in the **CAD** row; US codes carry the `US` token, routing all fills to the **USD** row (v2.18 helpers). Steps below are shared unless marked region-specific.

### `parse_dates(code, entitlement_days)` — month-anchored window
First date function in the tool that takes a per-card input in addition to the code name:
- **Start:** typed code date **− 1 day** at `00:00:00 ET`
- **End:** **last day of the typed date's calendar month** at `23:59:59 ET` **+ `entitlement_days`** (the per-card Days to Meet Entitlement, injected as the `DAYS_TO_ENTITLEMENT` array)

Month end computed via `calendar.monthrange` (28/29/30/31-day months and leap years handled). The typed day anchors only the start: `080126` and `081726` share the identical end date. A typed date of the 1st starts in the previous month (`010127` starts Dec 31, 2026 — intended). Changing Days to Entitlement changes the promo **and** bonus end date. Promo window = bonus activation window.

### `parse_vipdm(code)`
Returns `(min, pct, max)` from `..._M{min}_{pct}M{max}` — same token logic as `parse_churn_dm` (both sides of the `M` must be digits, so the `DM` / `VCL` tokens are skipped; three-digit percentages such as `100` parse correctly).

### `create_segment(page, name)`
Identical to RTC CC / DM / Churn DM — AMELCO `dispatch_event("click")`, 2-second post-`#forBonus` wait.

### `create_promo(page, name, idx)`
Poll Create Promotion (120 × 500ms, up to 60s) → click → fill name → Start/End per `parse_dates(name, DAYS_TO_ENTITLEMENT[idx])` → Type: `Image only CTA` → Layout: `Overlay` → first Save → upload `Promo Detail.png`, `Masthead.png`, `{pct}M{max}.png` from `IMAGE_FOLDER` → Promo Header Text: `From Your VIP Host` → Title: `From Your VIP Host` → Bonus Tile toggle ON → **Button 2 label: `Deposit`** → **Button 2 link: `/auth/account/deposit`** → How it works (template with `{min_fmt}` / `{pct}` / `{max_fmt}`) → T&C (Canadian legal copy, templated with `{min_fmt}` / `{pct}` / `{max_fmt}` / `{terms_id}` / `{start_date_long}` / `{end_date_long}`, baked in at generation time) → second Save. No confirmation modal.

### `create_bonus(page, name, idx)`
1. Poll Create Bonus (120 × 500ms, up to 60s)
2. Trigger: `Opt In`
3. Internal name = code name; Description = `From Your VIP Host`
4. Send To XP (per-card toggle, default ON); External Bonus (per-card toggle, default OFF)
5. Platform tags — **CA:** remove **all three** COMBO tags (`COMBO: Sportsbook`, `COMBO: Sportsbook And Casino`, **and `COMBO: Casino`**), leaving **`Web` + `STAC: Standalone Casino` only**. **US (v2.26):** standard pattern — remove only the two Sportsbook tags, **`COMBO: Casino` kept**
6. Voucher Code — CA: `VRDMCC{MMDDYY}{XXX}`; US: `VRDMFC{MMDDYY}{XXX}`
7. Bonus type — CA: `Casino Credit`; US: `FanCash`
8. Percentage checkbox — CA: **`Casino Credit (%)`**; US: **`Bonus (%)`** (the standard-DM FanCash form)
9. Fill percentage input (`aria-valuemin='0' aria-valuemax='100'`) with `{pct}` from code
10. Settlement days — CA: `Days to Meet Casino_credit Settlement`; US: `Days to Meet Fancash Settlement` — per-card value (default 7)
11. Days to Opt In = per-card value (default 3)
12. Days to Meet Entitlement = per-card value (default 3 — also drives the end dates)
13. Entitlement Type: `Deposit` (JS radio click)
14. **Matched Deposit checkbox — always checked** (lives inside the DEPOSIT entitlement panel; explicit visibility wait before clicking since the panel renders after step 13)
15. Summary percentage field — tries `Bonus %` then `Casino Credit %` label
16. Bonus amount summary card — tries `Bonus` then `Casino Credit` card title; fill with `{max}` from code into the **jurisdiction currency row** (CAD for CA, USD for US)
17. Minimum Deposit = dynamic `{min}` from code (`M{min}` segment) — jurisdiction currency row
18. Maximum Deposit = `1000000` (hardcoded) — jurisdiction currency row
19. Status Active = checked
20. Activation: per `parse_dates(name, DAYS_TO_ENTITLEMENT[idx])` (identical to promo window)
21. Segment / Client Profiling — anchored regex scoped to `.ant-select-dropdown:not(.ant-select-dropdown-hidden)`
22. Promotion Tile — anchored regex exact match
23. Reporting Platform: `CAS (Casino)`
24. Bonus Origin: `Opt-In Bonus`
25. Series: `VIP` — typed search + anchored regex `^VIP$` scoped to the **visible** dropdown
26. Type: `Retention` — anchored regex `^Retention$` scoped to the **visible** dropdown. ⚠️ This scoping is load-bearing: the (closed, hidden) Bonus Origin dropdown still contains a `Retention` option in the DOM, so a page-wide `get_by_text("Retention", exact=True)` would match two elements and crash with a Playwright strict mode violation. VIPDM and VIPBG are the only offer types that select `Retention` for Type.
27. Save — **CA: no confirmation modal** (no modal block in the CA script, unchanged from v2.23). **US (v2.26): a FanCash confirmation modal is expected** — handled defensively (waits 5s; confirms if one appears, otherwise prints `-- No confirmation modal (unexpected for FanCash — verify the bonus saved)`)

### Fixed Values

| Field | Value |
|---|---|
| Trigger | Opt In (both regions) |
| Bonus type | CA: Casino Credit (`Casino Credit (%)`); US: FanCash (`Bonus (%)`) |
| Description | From Your VIP Host |
| Matched Deposit | Checked (both regions) |
| Platforms | CA: Web + STAC: Standalone Casino only. US: standard — COMBO: Casino kept |
| Minimum Deposit | From code (`M{min}`) — jurisdiction currency row |
| Maximum Deposit | 1000000 — jurisdiction currency row |
| Settlement field | CA: `Days to Meet Casino_credit Settlement`; US: `Days to Meet Fancash Settlement` |
| Status Active | Checked |
| Reporting Platform | CAS (Casino) |
| Bonus Origin | Opt-In Bonus |
| Series | VIP |
| Type | Retention |
| Confirmation modal | CA: none; US: expected (FanCash) — defensive either way |

Note: the `RET` token in the code name mirrors the **Type** field (`Retention`), not Bonus Origin — unlike standard DM, where `RET` mirrors Origin.

### Activation / Promo Window

| | VIPDM |
|---|---|
| Start | `00:00:00 ET` typed date − 1 day |
| End | `23:59:59 ET` last day of typed date's month + Days to Meet Entitlement |

Promo window matches bonus activation window exactly.

### VIPDM T&C Data (Baked In at Generation Time) — CA — Canada

Offer key = `M{min}_{pct}M{max}`. All nine default offers share one promotional period, default **August 17, 2026 → December 31, 2026** (editable per the Terms modal; dates render as `{start_date_long}` / `{end_date_long}` in §2 of the T&C).

| Offer key | Min | % | Max CC | Terms ID |
|---|---|---|---|---|
| `M100_50M250` | $100 | 50 | $250 | `CAS_CA_0078` |
| `M100_100M250` | $100 | 100 | $250 | `CAS_CA_0079` |
| `M250_10M250` | $250 | 10 | $250 | `CAS_CA_0080` |
| `M250_10M2500` | $250 | 10 | $2,500 | `CAS_CA_0081` |
| `M250_10M5000` | $250 | 10 | $5,000 | `CAS_CA_0082` |
| `M250_20M500` | $250 | 20 | $500 | `CAS_CA_0083` |
| `M250_20M1000` | $250 | 20 | $1,000 | `CAS_CA_0084` |
| `M250_20M2000` | $250 | 20 | $2,000 | `CAS_CA_0085` |
| `M250_20M5000` | $250 | 20 | $5,000 | `CAS_CA_0086` |

T&C title line renders as: `FANATICS CASINO (CANADA) – DEPOSIT {min_fmt} CAD, GET {pct}% CASINO CREDIT (UP TO {max_fmt} CAD) ({terms_id})`. Canadian (Ontario) legal skeleton: FBG Enterprises Canada, Inc., 19+ / physically-present-in-Ontario eligibility, ConnexOntario responsible gaming language, CAD-denominated examples. §1's "within 3 days of being presented this offer" and §4's "during that Promo Period" are **static legal copy by stakeholder direction** — not templated. Changes to T&C copy, Terms IDs, or dates require regenerating the script.

**CA HIW** (no opt-in deadline line — the promo window bounds the offer; no "See below" trailer; saved as `vcl_dm_hiw`):
```
1. Opt-in to the promotion
2. Make a single deposit of {min_fmt} or more
3. We'll instantly match your deposit {pct}%, up to {max_fmt} Casino Credit
```

### VIPDM T&C Data (Baked In at Generation Time) — US — United States (v2.26)

Offer key = `M{min}_{pct}M{max}`. All ten default offers share one promotional period, default **July 1, 2026 → December 31, 2026** (editable per the region-aware Terms modal; dates render as `{start_date_long}` / `{end_date_long}` in the Promotional Period section).

| Offer key | Min | % | Max FanCash | Terms ID |
|---|---|---|---|---|
| `M100_50M250` | $100 | 50 | $250 | `CAS_10685` |
| `M100_100M250` | $100 | 100 | $250 | `CAS_10686` |
| `M250_10M500` | $250 | 10 | $500 | `CAS_10677` |
| `M250_10M1000` | $250 | 10 | $1,000 | `CAS_10678` |
| `M250_10M2500` | $250 | 10 | $2,500 | `CAS_10679` |
| `M250_10M5000` | $250 | 10 | $5,000 | `CAS_10680` |
| `M250_20M500` | $250 | 20 | $500 | `CAS_10681` |
| `M250_20M1000` | $250 | 20 | $1,000 | `CAS_10682` |
| `M250_20M2000` | $250 | 20 | $2,000 | `CAS_10683` |
| `M250_20M5000` | $250 | 20 | $5,000 | `CAS_10684` |

T&C title line renders as: `FANATICS CASINO - {min_fmt}+ DEPOSIT, {pct}% MATCH IN FANCASH (MAX {max_fmt}) (ID: {terms_id})` — the filings' own title prefixed with `FANATICS CASINO - ` by stakeholder direction. **Single canonical template** derived from the CAS_10677 filing, with the CAS_10677/CAS_10678/CAS_10686 discrepancies normalized by stakeholder decision: (1) "Fanatics Sportsbook and **Casino App**" (the CasinoApp missing-space defect in 10678/10686 fixed — same defect class as VIPBG US), (2) "**Only** your first Qualifying Deposit" (majority wording; 10686 said "Your first"), (3) all trailing whitespace stripped. Everything else is verbatim legal copy — the orphaned "(ii)" in Action Required, "exclusions apply- see fanatics.com", the double spaces around the FanCash Terms URL and "Notice of Financial Incentives.", and curly quotes are all preserved. Dynamic fields: `{min_fmt}`, `{pct}`, `{max_fmt}`, `{terms_id}`, `{start_date_long}`, `{end_date_long}` (short variants also available in the format functions). Static legal copy by design: MI/NJ/PA/WV eligibility, 21+, FBG Enterprises Opco LLC, the 3-day deposit window and 72-hour Rewards delivery, 7-day FanCash expiry, the FanCash Terms URL, `Wagering Requirements/Exclusions: N/A`, `Eligible Games/Markets/Events: Does not apply.`, and the 1-800-GAMBLER lines. Byte-verified against the three supplied filings with the normalizations applied. ⚠️ **CAS_10679–CAS_10685 (seven of ten filings) were not supplied** and are assumed to follow the same template — verify on first use. Changes to T&C copy, Terms IDs, or dates require regenerating the script.

**US HIW** (region-scoped, saved as `vcl_dm_hiw_us`):
```
1. Opt-in to the promotion
2. Make a single deposit of {min_fmt} or more
3. We'll instantly match your deposit {pct}%, up to {max_fmt} FanCash
```

---

## Script Steps — VIP Offer Library - Bet & Gets (v2.24 CA / v2.25 US / v2.27 CA creatives)

Dual region as of v2.25. `buildVIPBGScript` reads the region at generation time (`isVIPBGUS()`) and injects region-specific chunks; the CA output is functionally byte-identical to v2.24. CA codes carry the `CA` token → **CAD** row; US codes carry the `US` token → **USD** row (v2.18 helpers). **Both regions: the wager requirement ("bet") is enforced upstream by XP / segment logic — it is never entered on the bonus.** The bonus is a flat reward equal to the "get" value — **Casino Credit for CA, FanCash for US**.

### `parse_dates(code, entitlement_days)` — month-anchored window
Identical to VIPDM:
- **Start:** typed code date **− 1 day** at `00:00:00 ET`
- **End:** **last day of the typed date's calendar month** at `23:59:59 ET` **+ `entitlement_days`** (per-card Days to Meet Entitlement, injected as the `DAYS_TO_ENTITLEMENT` array)

`calendar.monthrange` handles month lengths and leap years. The typed day anchors only the start (`083126` and `080126` share an end date); a typed 1st starts in the prior month (`010127` starts Dec 31, 2026 — intended). Changing Days to Entitlement changes the promo **and** bonus end date. Promo window = bonus activation window.

### `parse_vipbg(code)`
**CA (v3.0):** returns `(cat, bet, get)` from `MMDDYY_CA_VCAS_RET_OL_{SBG|TBG}_CC_B{bet}_G{get}` — the category is read positionally from the MECHANIC slot (`parts[5]`) and validated against `("SBG", "TBG")`; bet/get are parsed from the freeform via the anchored regex `B(\\d+K?)_G(\\d+K?)` with K-expansion, returned as expanded literal strings (so Terms ID keys and Discover filenames keep their v2.x shapes). Raises on <8 fields, non-jurisdiction at position 1, non-SBG/TBG mechanic, malformed freeform, or malformed K.
**US (v3.0):** returns `(bet, get)` from `MMDDYY_US_VCAS_RET_OL_SBG_FC_B{bet}_G{get}` — same freeform regex + K-expansion + validations; no category (US mechanic is always `SBG`).

### `GAME_CATEGORIES` map (CA only)
`SBG` → HIW `Slots`, T&C `slots` / `SLOTS`; `TBG` → HIW `Tables`, T&C `table` / `TABLES`. The lowercase asymmetry ("slots games" but "table games") matches the legal documents verbatim. **As of v2.27 the `hiw` value also drives the Masthead / Promo Detail creative filenames** (`{Slots|Tables} Masthead.png`, `{Slots|Tables} Promo Detail.png`). **US scripts contain no `GAME_CATEGORIES` machinery** — all US tiers are slots, and "Slot games" is static copy in the US HIW.

### `create_segment(page, name)`
Identical to RTC CC / DM / Churn DM / VIPDM — AMELCO `dispatch_event("click")`, 2-second post-`#forBonus` wait.

### `create_promo(page, name, idx)`
Poll Create Promotion (120 × 500ms, up to 60s) → click → fill name → Start/End per `parse_dates(name, DAYS_TO_ENTITLEMENT[idx])` → Type: `Image only CTA` → Layout: `Overlay` → first Save → upload the region's Masthead / Promo Detail / Discover images from the region's `IMAGE_FOLDER` — **CA (v2.27):** `{Slots|Tables} Promo Detail.png` + `{Slots|Tables} Masthead.png` (category word from `GAME_CATEGORIES[cat]["hiw"]`) + Discover `{SBG|TBG}_B{bet}_G{get}.png`; **US:** `Promo Detail.png` + `Masthead.png` + Discover `B{bet}_G{get}.png` → Promo Header Text: `From Your VIP Host` → Title: `From Your VIP Host` → Bonus Tile toggle ON → **Button 2: region-branched (v2.25)** — CA leaves label/link **EMPTY** (terminal prints `-- Button 2: left empty`); US **fills `Play Now!` → `/docs/usered/casgenericgamelist`** via `BUTTON2_LABEL` / `BUTTON2_LINK` constants → How it works (CA template with `{bet_fmt}` / `{get_fmt}` / `{game_category}`; US template with `{bet_fmt}` / `{get_fmt}` only) → T&C (CA: single Canadian legal template covering both categories; US: single canonical FanCash template — both baked in at generation time) → second Save. No confirmation modal.

### `create_bonus(page, name, idx)`
1. Poll Create Bonus (120 × 500ms, up to 60s)
2. Trigger: `Opt In`
3. Internal name = code name; Description = `From Your VIP Host`
4. **Send To XP — always clicked ON**; **External Bonus — always clicked ON** (no per-card toggles; `SEND_TO_XP` / `EXTERNAL_BONUS` arrays are hardcoded `[True] * len(NAMES)`)
5. Platform tags — **region-branched (v2.25)**: CA removes **all three** COMBO tags (`COMBO: Sportsbook`, `COMBO: Sportsbook And Casino`, `COMBO: Casino`), leaving **`Web` + `STAC: Standalone Casino` only** (matching VIPDM); **US removes only the two Sportsbook tags and keeps `COMBO: Casino`** (standard pre-v2.23 behavior)
6. Voucher Code: `CLBGFC{MMDDYY}{XXX}` (both regions)
7. Bonus type — **region-branched**: CA `Casino Credit`, US `FanCash` (both flat amount — no percentage checkbox, no deposit entitlement, no Matched Deposit)
8. Settlement — CA `Days to Meet Casino_credit Settlement`, US `Days to Meet Fancash Settlement` = per-card value (default 7)
9. Days to Opt In = per-card value (default 3)
10. Days to Meet Entitlement = per-card value (default 3 — also drives the end dates)
11. **Stake Chunk Sizes — conditional fill of `1`** (present on the flat CC form per RTC CC / LC-REACT; filled if found, skipped with a terminal note if not)
12. Reward amount = **`{get}` from the code** via `fill_currency_amount` — CA: `Casino Credit Amount` label, **CAD row**; US: `FanCash Amount` label, **USD row**
13. Status Active = checked
14. Activation: per `parse_dates(name, DAYS_TO_ENTITLEMENT[idx])` (identical to promo window)
15. Segment / Client Profiling — anchored regex scoped to `.ant-select-dropdown:not(.ant-select-dropdown-hidden)`
16. Promotion Tile — anchored regex exact match
17. Reporting Platform: `CAS (Casino)`
18. Bonus Origin: `Opt-In Bonus`
19. Series: `VIP` — typed search + anchored regex `^VIP$` scoped to the **visible** dropdown
20. Type: `Retention` — anchored regex `^Retention$` scoped to the **visible** dropdown. ⚠️ Same load-bearing scoping as VIPDM: the hidden Bonus Origin dropdown also contains a `Retention` option, so a page-wide exact-text click would strict-mode crash. VIPDM and VIPBG are the only offer types that select `Retention` for Type.
21. Save — **defensive confirmation-modal handling** (both regions): waits up to 5s for `.ant-modal-content`; confirms it if it appears, continues with a terminal note if it doesn't. **CA:** production runs confirmed no modal appears on the flat Casino Credit form; absence prints `(expected)`. **US:** a modal is expected (standard FanCash BG shows one); absence prints `(unexpected for FanCash — verify the bonus saved)`.

### Fixed Values

| Field | Value |
|---|---|
| Trigger | Opt In (both regions) |
| Description | From Your VIP Host (both regions) |
| Send To XP / External Bonus | Both always ON (both regions) |
| Bet amount | **Not entered anywhere** — enforced upstream (XP/segment) (both regions) |
| Reward type & amount | CA: Casino Credit from code (`G{get}`) — CAD row. US: FanCash from code (`G{get}`) — USD row |
| Stake Chunk Sizes | 1 (conditional fill, both regions) |
| Platforms | CA: Web + STAC: Standalone Casino only. **US: Web + STAC + COMBO: Casino (kept)** |
| Button 2 (promo) | CA: empty. US: `Play Now!` → `/docs/usered/casgenericgamelist` |
| Status Active | Checked |
| Reporting Platform | CAS (Casino) |
| Bonus Origin | Opt-In Bonus |
| Series | VIP |
| Type | Retention |
| Confirmation modal | CA: none observed. US: expected (FanCash). Both handled defensively |

Note: the `RET` token in the code name mirrors the **Type** field (`Retention`), not Bonus Origin — same as VIPDM.

### Activation / Promo Window

| | VIPBG |
|---|---|
| Start | `00:00:00 ET` typed date − 1 day |
| End | `23:59:59 ET` last day of typed date's month + Days to Meet Entitlement |

Promo window matches bonus activation window exactly. Note the T&C's seventy-two (72) hour wagering-window language is **legal copy about the offer mechanics** (the customer's window after receiving the offer) and is independent of this month-anchored promo/bonus activation window — same relationship as VIPDM's static "within 3 days" line.

### VIPBG T&C Data (Baked In at Generation Time) — CA — Canada

Offer key = `{SBG|TBG}_B{bet}_G{get}`. All eight default offers share one promotional period, default **August 17, 2026 → December 31, 2026** (editable per the Terms modal; dates render as `{start_date_long}` / `{end_date_long}` in §2 of the T&C).

| Offer key | Category | Bet | Get CC | Terms ID |
|---|---|---|---|---|
| `SBG_B1000_G100` | Slots | $1,000 | $100 | `CAS_CA_0069` |
| `SBG_B5000_G500` | Slots | $5,000 | $500 | `CAS_CA_0070` |
| `SBG_B20000_G1000` | Slots | $20,000 | $1,000 | `CAS_CA_0071` |
| `SBG_B100000_G5000` | Slots | $100,000 | $5,000 | `CAS_CA_0072` |
| `TBG_B2000_G100` | Tables | $2,000 | $100 | `CAS_CA_0073` |
| `TBG_B10000_G500` | Tables | $10,000 | $500 | `CAS_CA_0074` |
| `TBG_B25000_G1000` | Tables | $25,000 | $1,000 | `CAS_CA_0075` |
| `TBG_B100000_G2500` | Tables | $100,000 | $2,500 | `CAS_CA_0076` |

**Single T&C template covering both categories** — the Slots and Tables legal documents are word-for-word identical apart from dynamic values, so one template with `{bet_fmt}` / `{get_fmt}` / `{terms_id}` / `{game_category_lower}` / `{game_category_upper}` / `{start_date_long}` / `{end_date_long}` reproduces all 8 documents exactly (verified byte-for-byte against both legal docs). T&C title line renders as: `FANATICS CASINO (CANADA) – {game_category_upper} BET {bet_fmt} CAD, GET {get_fmt} CAD IN CASINO CREDIT ({terms_id})`. Canadian (Ontario) legal skeleton: FBG Enterprises Canada, Inc., 19+ / physically-present-in-Ontario eligibility, ConnexOntario responsible gaming language, CAD-denominated examples, curly quotes and en-dash preserved. **Static by design (not templated):** the seventy-two (72) hour wagering-window language throughout, "within 72 hours of satisfying the wagering requirement," the seven (7) day Casino Credit expiry, and all §5 CAD examples. Changes to T&C copy, Terms IDs, or dates require regenerating the script.

**HIW** (first-person VIP-host voice; Canadian spelling "favourite"; `{game_category}` renders `Slots` / `Tables`; no opt-in deadline line and no "See below" trailer, by design):
```
1. Opt-in to the promotion
2. Wager {bet_fmt}+ on any of your favourite {game_category} games
3. I'll instantly give you {get_fmt} in Casino Credit
```

### VIPBG T&C Data (Baked In at Generation Time) — US — United States (v2.25)

Offer key = `B{bet}_G{get}` (no category prefix — all US tiers are slots B&Gs). All four default offers share one promotional period, default **July 1, 2026 → December 31, 2026** (editable per the Terms modal with US selected; dates render as `{start_date_long}` / `{end_date_long}`).

| Offer key | Bet | Get FanCash | Terms ID |
|---|---|---|---|
| `B1000_G100` | $1,000 | $100 | `CAS_10687` |
| `B5000_G500` | $5,000 | $500 | `CAS_10688` |
| `B20000_G1000` | $20,000 | $1,000 | `CAS_10689` |
| `B100000_G5000` | $100,000 | $5,000 | `CAS_10690` |

**Single canonical T&C template.** The CAS_10687 and CAS_10688 legal filings were compared and found **not** word-for-word identical — four copy-edit inconsistencies were normalized into one canonical wording by stakeholder decision: (1) "Fanatics Sportsbook and Casino App" (missing space fixed), (2) "after satisfying the wagering **requirements** for this Promotion" (plural), (3) single period after "eligible to participate in this Promotion." in Limitations, (4) "Fanatics Casino aims to **consistently care** for our customers and promote responsible gambling practices." Everything else is verbatim legal copy — verified by byte-level comparison of the rendered template against both source documents with those fixes applied. Dynamic fields: `{bet_fmt}` / `{get_fmt}` / `{terms_id}` / `{start_date_long}` / `{end_date_long}` (short variants available; **no game-category fields**). Title line: `FANATICS CASINO - VIP BET {bet_fmt} GET {get_fmt} FANCASH (ID: {terms_id})`. **Static legal copy (not templated):** MI/NJ/PA/WV eligibility, 21+, FBG Enterprises Opco, LLC entity, the 72-hour wagering language and 3-day presentation window, the seven (7) day FanCash expiry, the FanCash Terms URL (`https://fanatics-one.com/fancashterms`), Fanatics ONE Loyalty / Notice of Financial Incentives language, and the 1-800-GAMBLER responsible-gaming lines. Curly quotes preserved. ⚠️ The CAS_10689 and CAS_10690 filings were not supplied during development — the canonical template is assumed to cover them; verify on first use.

**HIW — US** (region-scoped, saved to `vcl_bg_hiw_us` — unlike RTC CC, VIPBG HIW is per-region; US spelling "favorite"; "Slot games" is static copy — no `{game_category}` field):
```
1. Opt-in to the promotion
2. Wager {bet_fmt}+ on any of your favorite Slot games
3. I'll instantly give you {get_fmt} in FanCash
```

---

## VIPBG US Region (v2.25)

VIP Offer Library - Bet & Gets supports two regions: **CA — Canada** (original, default) and **US — United States**. The tool works exactly as v2.24 for CA; US layers its own settings on top without touching any CA key or default — the same architecture as RTC CC's v2.22 Canada support, inverted (here CA owns the base keys and US takes the suffix).

### Region scoping

Two helpers drive everything:

- `isVIPBGUS()` — true when the offer type is VIPBG and the Region select is `US`.
- `vipbgKey(base)` — returns `base + '_us'` when `isVIPBGUS()`, else `base`.

`getHIW()`, `getTC()`, `getImagePath()`, `getVIPBGTermsIDs()`, `getVIPBGStartDate()`, `getVIPBGEndDate()`, and the corresponding modal save paths all resolve through `vipbgKey()`, so Edit HIW / Edit T&Cs / Edit Terms Expiry & Terms IDs / Edit Images read and write whichever region is selected, and each modal shows the region in its title. Unlike RTC CC, **HIW is region-scoped** (CA is first-person Canadian-spelling Casino Credit copy; US is FanCash copy).

### What differs when US is selected

| Setting | CA — Canada | US — United States |
|---|---|---|
| Reward | Casino Credit (CAD row) | FanCash (USD row) |
| Code name (v3.0) | `MMDDYY_CA_VCAS_RET_OL_{SBG\|TBG}_CC_B{bet}_G{get}` | `MMDDYY_US_VCAS_RET_OL_SBG_FC_B{bet}_G{get}` |
| Offer rows | Category (Slots/Tables) → Bet → Get, 8 defaults | Bet → Get, 4 defaults (all slots) |
| Terms ID keys | `{SBG\|TBG}_B{bet}_G{get}` → `CAS_CA_0069`–`0076` | `B{bet}_G{get}` → `CAS_10687`–`10690` |
| Promotional dates (defaults) | `08/17/26 → 12/31/26` | `07/01/26 → 12/31/26` |
| Image folder default | `.../VIP Automations/Offer Library/Canada/Bet & Gets` | `.../VIP Automations/Offer Library/USA/Bet & Gets` |
| Drive link in Edit Images | `folders/1tkMs1dx-gszSGzlBHxgMYvOYv2l-2UjO` | `folders/13neRGeYM4JLa98p827vbepp-eKu2_hXq` |
| Discover filename | `{SBG\|TBG}_B{bet}_G{get}.png` | `B{bet}_G{get}.png` |
| Masthead / Promo Detail filenames (v2.27) | `{Slots\|Tables} Masthead.png` / `{Slots\|Tables} Promo Detail.png` (per-category) | `Masthead.png` / `Promo Detail.png` (generic) |
| Button 2 (promo) | Empty | `Play Now!` → `/docs/usered/casgenericgamelist` |
| Platform tags | All 3 COMBO removed (Web + STAC only) | 2 removed, **COMBO: Casino kept** |
| Settlement label | `Days to Meet Casino_credit Settlement` | `Days to Meet Fancash Settlement` |
| Confirmation modal | None observed | Expected (FanCash) — defensive either way |
| HIW / T&C keys | `vcl_bg_hiw` / `vcl_bg_tc` | `vcl_bg_hiw_us` / `vcl_bg_tc_us` |
| Edit button theme | Reds via `body.ca-region` | Blues (default) |

Everything not listed is shared: month-anchored window (`parse_dates`), `From Your VIP Host` copy, Send To XP + External Bonus always ON, `CLBGFC{MMDDYY}{XXX}` voucher, Series `VIP` / Type `Retention` / Origin `Opt-In Bonus` / Reporting CAS, Days 3/3/7, step badges, Terms ID enforcement, conditional Stake Chunk fill, and image upload timing.

### Region change behavior

`onJurisdictionChange()` refreshes the red/blue button theme and, when the offer type is RTC, VIPBG, **or VIPDM (v2.26)**, calls `resetAll()` — clearing every day card and hiding any generated script; silent by design, no confirmation dialog. `updateRegionTheme()` applies `body.ca-region` when `isRTCCanada()`, VIPBG-with-CA, **or VIPDM-with-CA (v2.26)** is true; the added conditions are provably inert for every other offer type.

---

## VIPDM US Region (v2.26)

VIP Offer Library - Deposit Matches supports two regions: **CA — Canada** (original, v2.23) and **US — United States** (v2.26). The tool works exactly as v2.23–v2.25 for CA; US layers its own settings on top without touching any CA key or default — the same architecture as VIPBG's v2.25 US support (CA owns the base keys, US takes the `_us` suffix).

### Region scoping

Two helpers drive everything:

- `isVIPDMUS()` — true when the offer type is VIPDM and the Region select is `US`.
- `vipdmKey(base)` — returns `base + '_us'` when `isVIPDMUS()`, else `base`.

`getHIW()`, `getTC()`, `getImagePath()`, `getVIPDMTermsIDs()`, `getVIPDMStartDate()`, `getVIPDMEndDate()`, and the corresponding modal save paths all resolve through `vipdmKey()`, so Edit HIW / Edit T&Cs / Edit Terms Expiry & Terms IDs / Edit Images read and write whichever region is selected, and the T&C / Terms / Images modals show the region in their titles. HIW is **region-scoped** (as VIPBG; unlike RTC CC where HIW is shared).

⚠️ **Region default:** unlike VIPBG (which keeps a valid current selection and otherwise defaults CA), **VIPDM always selects US when the offer type is chosen** — per stakeholder direction; CA remains selectable from the dropdown.

### What differs when US is selected

| Setting | CA — Canada | US — United States |
|---|---|---|
| Reward | Casino Credit (CAD row) | FanCash (USD row) |
| Code name (v3.0) | `MMDDYY_CA_VCAS_RET_OL_DM_CC_M{min}_{pct}M{max}` | `MMDDYY_US_VCAS_RET_OL_DM_FC_M{min}_{pct}M{max}` |
| Default tiers | 9 (`CAS_CA_0078`–`0086`) | 10 (`CAS_10677`–`10686`; adds 10%/$500 and 10%/$1,000, drops 10%/$250) |
| Promotional dates (defaults) | `08/17/26 → 12/31/26` | `07/01/26 → 12/31/26` |
| Voucher | `VRDMCC{MMDDYY}{XXX}` | `VRDMFC{MMDDYY}{XXX}` |
| Bonus type / pct checkbox | `Casino Credit` / `Casino Credit (%)` | `FanCash` / `Bonus (%)` |
| Settlement label | `Days to Meet Casino_credit Settlement` | `Days to Meet Fancash Settlement` |
| Platform tags | All 3 COMBO removed (Web + STAC only) | 2 removed, **COMBO: Casino kept** |
| Confirmation modal | None (no modal block in the script) | Expected (FanCash) — defensive handler |
| Image folder default | `.../VIP Automations/Offer Library/Canada/Deposit Matches` | `.../VIP Automations/Offer Library/USA/Deposit Matches` |
| Drive link in Edit Images | `folders/1v9DEVtCNQPCgUkJsWtsXn7UJVIyBYtr4` | `folders/1W8GLxrZFBMotLP_WPzAE04KPPb__BxIX` |
| HIW / T&C keys | `vcl_dm_hiw` / `vcl_dm_tc` | `vcl_dm_hiw_us` / `vcl_dm_tc_us` |
| Edit button theme | Reds via `body.ca-region` | Blues (default) |

Everything not listed is shared: month-anchored window (`parse_dates`), `From Your VIP Host` copy, **Button 2 `Deposit` → `/auth/account/deposit`**, Send To XP default ON / External Bonus default OFF (with toggles), **Matched Deposit always checked**, `{pct}M{max}.png` Discover filenames, Minimum Deposit from `M{min}` / Maximum Deposit 1000000, Series `VIP` / Type `Retention` / Origin `Opt-In Bonus` / Reporting CAS, Days 3/3/7, step badges, Terms ID enforcement, and image upload timing.

### Region change behavior

Same as RTC CC / VIPBG: switching Region silently resets all day cards (dates, selected offers, toggles; generated script hidden) so US and CA inputs can never mix.

---

## Script Steps — Daily Missions - Canada (v2.28; FS unlocked v2.30; NATS segments v2.31)

Two-phase production script (`daily_missions_MMDDYY_HHMM.py`). `CODES` entries carry `code_name`, `reward` (`CC` / `FS`), `weekday`, `bet`, `get`, `spins`, `spin_value`, `vtok`, `game_name`, `game_search`, `terms_id`, `days_to_fulfill`, `do_segment` (v2.31), `do_promo`, `do_bonus` (FS-only fields are empty strings on CC codes and vice versa for `get`). Phases and sub-phases are skipped entirely if no code has that step enabled.

### Launch & Login (both phases — Okta Verify / FastPass fix)
1. `launch_persistent_context` with profile `~/.dm_canada_pw_profile`, `headless=False`, `args=["--disable-features=LocalNetworkAccessChecks"]`, clipboard permissions
2. `context.grant_permissions(["local-network-access"], origin="https://betfanatics.okta.com")` in try/except (a second unscoped grant as fallback; a WARN is printed if the grant API is unavailable and the feature flag alone carries)
3. This resolves Playwright's auto-denied permission prompt that blocked Okta's "Access other apps and services on this device" (FastPass) — confirmed working in PROD Aug 14, 2026
4. **Pre-flight before any login:** every `do_promo` code's three weekday image files are checked on disk; the script aborts with a missing-file list before touching PROD if any are absent

### Phase 1 — NATS Segments + Promotions (v2.31)
Runs when any code has `do_segment` or `do_promo`; one manual NATS login pause (`input()`) covers both sub-phases.

**Segments (per code with `do_segment`):**
1. Sidebar hover-nav to Account Segments (`[data-menu-id$='-segments']` → `Account Segments`, 5-attempt retry with safe-zone reset)
2. `create_segment(page, name)` — the standard NATS segment build used by every other segment-building offer type: `button.ant-btn-primary.create` → `#name` and `#code` = code name → `#parentId` → **AMELCO** via `get_by_text("AMELCO").nth(2).dispatch_event("click")` → `#forBonus` checked → 2s wait → OK

**Promotions (per code with `do_promo`):**
1. Sidebar hover-nav to Promos (`[data-menu-id$='-cms']`, 5-attempt retry with safe-zone reset)
2. Create Promotion button polled up to 60s (120 × 500ms)
3. Internal name = code name
4. **Dates: single-day window** — `parse_dates(code)` returns typed date `00:00:00 ET` → same date `23:59:59 ET` (⚠️ unique to DMCA; converted to local time strings for the pickers)
5. Type = Image only CTA; Layout = Overlay; Save
6. Images (8s post-select wait, 2s post-OK — standard timing): `{Weekday}_Promo_Detail.png` → Promo Detail, `{Weekday}_Masthead.png` → Masthead, Discover = `{Weekday}_B{bet}_G{get}.png` for CC codes / `{Weekday}_B{bet}_G{spins}S_{vtok}.png` for FS codes (weekday derived at runtime from the code's date via `image_files_for(c)`)
7. Title and Promo Header Text = `{Weekday}'s Daily Mission` (auto-derived)
8. **Bonus Tile switch ON**; **Button 2 left empty** (by design)
9. How it works: CC codes = `HIW_CC_TEMPLATE.format(bet_fmt, get_fmt)`; FS codes = `HIW_FS_TEMPLATE.format(bet_fmt, spins, game_name)` (line 3: `We'll reward you with {spins} Bonus Spins to use on the slots game {game_name}`) — no "CAD" anywhere in either HIW by design; "twenty-four (24) hours" is static legal copy
10. T&C: CC codes = `TC_CC_TEMPLATE` (unchanged from v2.28); FS codes = `TC_FS_TEMPLATE.format(bet_fmt, spins, spin_value, game_name, terms_id, start_date_long, end_date_long)` — full Ontario legal copy, **byte-verified against all sixteen FS filings (CAS_CA_0043–0058)** with two approved normalizations (exactly one space always follows `{game_name}`; trailing whitespace stripped); `spin_value` renders two-decimal (`$0.10` / `$0.50` / `$3.00`); the §5 "$15 CAD" winnings example is static on all tiers; `TERMS_START_LONG` / `TERMS_END_LONG` are shared with CC (currently `August 17, 2026` / `December 31, 2026`) and baked at generation time
11. Rich text filled via clipboard paste into the visible ProseMirror editor (`navigator.clipboard.writeText` + `Meta+V`)
12. Save — **no confirmation modal**

### Phase 2 — Playmaker Bonus (per code with `do_bonus`; Save Bonus IS clicked)
1. New page in the same persistent context → `PLAYMAKER_URL`; manual SSO login pause
2. iCasino tile (`h3:has-text('iCasino')`) → Bonus Setup (`[data-testid='bonus-setup-navItem']`) → Create (`[data-testid='create-bonus-button']`)
3. **Step 1 Details:** name = code name; jurisdictions = uncheck `#jurisdiction-all` if checked, check `#jurisdiction-ontario` (Canada); Start Date = code date, End Date = code date + 1 day via the calendar widget (`button.dc-w-\[280px\]` pickers, `aria-live` month header, `button[name='next-month']`, `button[name='day']` skipping `dc-day-outside`); **both times set to `00:01` (12:01 AM) as of v2.30** — native-setter `evaluate` first, then a read-back; if the value did not stick, a keyboard retype (`click ×3` + `type("00:01")` + Enter), then a `fill()` + Tab fallback; final read-back printed either way
4. **Step 2 Eligibility (v2.31 — `attach_existing_segment`):** the NATS-built segment is attached via the **Include Segments → Existing Account Segments** field. ⚠️ Step 2 contains **four** cmdk fields — Include/Exclude × Create/Existing — with identical labels across sections and identical `placeholder="Search..."` on both Existing fields. The Create fields are NEVER used (a chip there creates a duplicate segment on Save Bonus — the v2.28–v2.30 behavior) and the Exclude section is NEVER used (it would invert targeting). Field location is triple-guarded: **(a)** label-anchored primary — the `<label>` with exact text `Existing Account Segments` whose ancestor section resolves to `Include Segments` (a container mentioning both Include and Exclude headings is a shared parent — rejected), followed via its `for=` attribute to the form-item container's `input[cmdk-input]`, climbing from the label with a `placeholder='Search...'` filter as backstop (the dynamic React/Radix ids like `:r5g:` / `radix-:r5j:` are session-scoped and deliberately not hardcoded); **(b)** hardened parent-walk fallback requiring both Existing-not-Create and Include-not-Exclude ancestor tests; **(c)** placeholder assertion on both paths (Existing = `Search...`, Create = `Type and press Enter...` — a Create-style placeholder is rejected even on a structural match). The code name is typed and the option whose text **exactly matches** is clicked — never Enter (that is the Create-field gesture). Then the **chip is read back**: a `<span>` exactly equal to the code name must render inside the Include/Existing field within ~5s, or the script dumps every Step 2 combobox/input and raises ("refusing to continue on an unverified attach") → per-bonus FAILED with recovery. NATS → Playmaker segment-sync lag is absorbed by a 12 × 5s (~60s) outer poll; if the segment never appears, the script dumps the fields and hard-fails — "refusing to fall back to Create Account Segments (would duplicate the NATS segment)".
5. **Step 3 Entitlement:** `input[name='wageringAmount']` = bet
6. **Step 4 Settlement — branches on `reward`:**
   - **CC codes:** Casino Credit radio (`button[role='radio'][value='casino_credit']`, clicked only if `aria-checked` ≠ true); `input[name='settlementAmount']` = get
   - **FS codes (v2.30):** Bonus Spins radio (`button[role='radio'][value='free_spin']`) → `#fs-count` = spins → Aggregator combobox (found by aria-label substring among `button[role='combobox']`) = **PLAYTECH** → Provider combobox = **PLAYTECH** → Casino Game combobox: options matched ™-tolerantly (`™`/`®` stripped, whitespace collapsed, case-insensitive, substring on the pre-™ `game_search`); the actual option text clicked is printed → **Free Spin Stakes**: combobox inside `[data-testid='free-spin-stakes-grid']` (⚠️ it has **no aria-label** — must be located via the grid testid; the grid's row text is dumped to the terminal first); options are plain decimals (`0.10` … `100.00`) and **show CAD for Canada**; the exact `spin_value` is required — **if absent, the script dumps the available options and raises ("refusing to substitute"), never selecting a nearby value**, because the T&C and Terms ID legally state the spin value
   - Both branches: `input[name='daysToFulfill']` = Days to Use Reward (cleared and refilled only if different)
7. **Step 5 Display:** promotion combobox (`button[role='combobox']` → search input → option clicked via `get_by_label(name).get_by_text(name)`) selecting the Phase 1 promo by code name; **toast always ON** — `#toastEnabled` checked if not already, `input[name='toastTitle']` = `Mission Complete!`, `input[name='toastDescription']` = `You have been awarded ${get} Casino Credit` (CC) / `You have been awarded {spins} Bonus Spins to {game_name}` (FS — **no quotes around the game name**)
8. **Step 6 Review:** `button:has-text('Save Bonus')` clicked, 3s settle
9. **Per-bonus resilience (v2.30):** each bonus runs inside try/except — a failure (e.g. the stakes hard-fail) prints `FAILED {code}: {error}`, navigates back to Bonus Setup, and continues with the next code; a **SAVED/FAILED results summary** prints at the end of Phase 2
10. macOS `osascript` notification at the end; final `input()` pause before the persistent context closes

---

## Script Steps — RTC Top Up - Casino Credit

1. Opt In trigger
2. Internal name + description (`$X of Casino Credit`)
3. Send To XP (conditional)
4. Remove `COMBO: Sportsbook` and `COMBO: Sportsbook And Casino` tags
5. Voucher Code: `CRRCCC{MMDDYY}{XXX}`
6. Casino Credit type
7. Stake Chunk Sizes = 1
7b. Casino Credit amount — filled into the jurisdiction's currency row via `fill_currency_amount()` (see Multi-Currency Amount Tables)
8. Status Active checked
9. Activation: day-of 04:00 ET → +24 hours ET
10. Reporting: Platform = CAS, Origin = Retention, Series = RTC, Type = Lifecycle
11. Segment / Client Profiling
12. Promotion Tile
13. Save (no confirmation modal)

The RTC CC Python template is identical for US and CA — jurisdiction differences are injected at generation time (`IMAGE_FOLDER`, `TC_TEXT`, `TERMS_IDS`, promotional dates) and at runtime (`currency_for()` reads the `_US_` / `_CA_` code-name token to pick the USD or CAD amount row).

---

## RTC CC Canada (CA) Support (v2.22)

RTC Top Up - Casino Credit supports two jurisdictions: **US — United States** (default) and **CA — Canada**. The tool works exactly as before for US; CA layers jurisdiction-specific settings on top without touching any US key or default.

### Jurisdiction scoping

Two helpers drive everything:

- `isRTCCanada()` — true when the offer type is RTC and the Region select is `CA`.
- `rtcKey(base)` — returns `base + '_ca'` when `isRTCCanada()`, else `base`.

`getTC()`, `getImagePath()`, `getRTCTermsIDs()`, `getRTCStartDate()`, `getRTCEndDate()`, and the corresponding modal save paths all resolve through `rtcKey()`, so the Edit T&Cs / Edit Terms IDs / Edit Images modals read and write whichever region is selected. Modal titles display the active region (e.g. `Edit T&Cs — CA — Canada`). `getHIW()` intentionally does **not** use `rtcKey()` — HIW copy is shared between US and CA (same `{amount_fmt}` dynamic field).

### What differs when CA is selected

| Setting | US | CA |
|---|---|---|
| Image folder default | `.../Lifecycle Automations/RTC Top Up` | `.../Lifecycle Automations/RTC Top Up - Canada` |
| Drive link in Edit Images | `folders/1Dlpa3xZTHjzlwHgayPSS-MZJ8-DQTmhh` | `folders/1W6ulHKv3ted9ZLNKzPgEbRoNCK_wFz6O` |
| Image filenames | `Promo Detail.png`, `Masthead.png`, `{amount}.png` | Identical convention |
| T&C base copy | US legal copy (`DEFAULT_TC`) | Canadian (Ontario) legal copy (`DEFAULT_TC_CA`) |
| T&C title line | `FANATICS CASINO - CASINO CREDIT {amount_fmt} SURPRISE DROP - {start_date_short} (ID: {terms_id})` | `FANATICS CASINO (CANADA) – CASINO CREDIT {amount_fmt} CAD SURPRISE DROP ({terms_id})` |
| Terms IDs (defaults) | `CAS_9461`–`CAS_9471` | `CAS_CA_0001`–`CAS_CA_0011` |
| Promotional dates (defaults) | `06/01/26 → 12/01/26` | `08/17/26 → 12/31/26` |
| HIW | Shared — single `rtc_hiw` key | Shared — same key |
| Amount table row (runtime) | `tr[data-row-key='USD']` | `tr[data-row-key='CAD']` |
| Edit button theme | Blues (`#0C447C` / `#185FA5` / `#378ADD`) | Reds (`#7C0C0C` / `#A51818` / `#DD3737`) via `body.ca-region` |

### CA Terms IDs (baked-in defaults)

| Amount | Terms ID | | Amount | Terms ID |
|---|---|---|---|---|
| $2 | `CAS_CA_0001` | | $50 | `CAS_CA_0007` |
| $4 | `CAS_CA_0002` | | $100 | `CAS_CA_0008` |
| $5 | `CAS_CA_0003` | | $200 | `CAS_CA_0009` |
| $10 | `CAS_CA_0004` | | $400 | `CAS_CA_0010` |
| $20 | `CAS_CA_0005` | | $800 | `CAS_CA_0011` |
| $40 | `CAS_CA_0006` | | | |

All 11 tiers share the CA promotional period **August 17, 2026 → December 31, 2026**. Overrides saved via the Edit Terms Expiry & Terms IDs modal (with CA selected) persist to `rtc_terms_ids_ca` / `rtc_terms_start_date_ca` / `rtc_terms_end_date_ca` without touching the US values.

### Canadian T&C notes

The CA base copy is the Ontario legal document: FBG Enterprises Canada, Inc. entity, 19+ / physically-present-in-Ontario eligibility, ConnexOntario Gambling Helpline (1-866-531-2600) responsible gaming section, and CAD-denominated examples. It uses the same six dynamic fields as US and flows through the same `format_tc` f-string in the generated script — verified by extracting `format_tc` verbatim from the HTML, rendering it through a real JS template literal with the CA copy and Terms IDs, and executing the resulting Python for all 11 tiers (per the template-literal known limitation).

### Region change behavior

`onJurisdictionChange()` refreshes the red/blue button theme and, when the offer type is RTC, calls `resetAll()` — clearing every day card and hiding any generated script. `updateJurisdictionOptions()` also refreshes the theme on every offer-type switch, so leaving RTC always restores the blue theme.

---

## Script Steps — Bet & Get

Same structure as RTC CC. Voucher: `CLBGFC{MMDDYY}{XXX}`. FanCash type — FanCash amount filled via jurisdiction currency row (v2.18). Confirmation modal: **Yes** required. Create Promotion poll applies.

---

## Script Steps — Deposit Match

Same structure as RTC CC. Voucher: `CRDMFC{MMDDYY}{XXX}`. FanCash + Bonus % type. Bonus amount card and Minimum/Maximum Deposit cards filled via jurisdiction currency row (v2.18). Confirmation modal: **Yes** required. Create Promotion poll applies.

---

## Script Steps — RAF

### `create_segment(page, name)`
Identical to RTC CC (including AMELCO `dispatch_event("click")` and 2-second post-`#forBonus` wait).

### `create_promo(page, name, discover_image)` — RFRE and RFER
Poll Create Promotion (120 × 500ms, up to 60s) → click → fill name → Start: day-of midnight → End: year 2041 → Type: Image only CTA → Layout: Overlay → Refer A Friend toggle ON → first Save → upload images → fill all copy fields → second Save.

### `create_bonus_rfre(page)` — Referee
Poll Create Bonus → Opt In → name/description → Refer A Friend checked → Referee radio → Send To XP NOT clicked → remove platform tags → Voucher Code → Free Spin type → description/CTA/game fields (Casino Game via ™-tolerant search + match) → per-day spins → spin value (jurisdiction currency row) → Days (0/0/1) → Status Active checked → Activation → Reporting fields → Segment → Promotion Tile → Save → voucher modal → Apply

### `create_bonus_rfer(page)` — Referrer
Same as RFRE except: Send To XP clicked ON, Referrer radio, total spins (not per-day), no entitlement block, Days to Settlement = 7, Status Active unchecked. Post-save edit reload wait = 120 seconds.

### `create_bonus_day(page, n, voucher_code)` — RAF Day 2+
Poll Create Bonus → Opt In → code name → description blank → Refer A Friend unchecked → Send To XP checked → remove platform tags → Voucher Code → Free Spin type → description → CTA / game fields → per-day spins → spin value → Days (0/0/1) → Status Active unchecked → Activation → Reporting fields → Segment → Save (no modal, no edit pass).

### Fixed Values — All RAF Bonuses

| Field | Value |
|---|---|
| Trigger | Opt In |
| Free Spin's CTA | Play! |
| Bet Level | default |
| Activation end | year 2041 |
| Reporting Platform | CAS (Casino) |
| Bonus Origin | Bet Settlement Bonus |
| Series | Refer a Friend |
| Type | Acquisition |

---

## Script Steps — SUO Day 2+ Spins

### `create_bonus_suo_day(page, n)`
1. Poll Create Bonus (120×500ms)
2. **No Opt In** trigger
3. Internal name = code name; description left blank
4. Refer A Friend Bonus checkbox — **not touched**
5. Send To XP checked
6. Remove `COMBO: Sportsbook` and `COMBO: Sportsbook And Casino` platform tags
7. **No voucher code**
8. Free Spin type
9. Free Spin's Description = `"Sign Up Offer Bonus Spins Day N"`
10. CTA = `Play!`; Aggregator / Provider / Casino Game (™-tolerant search + match — see Casino Game Selection) / Bet Level (default) / Deeplink
11. Spins = per-day amount
12. Spin Value — selected in the Free Spin Stakes row for the jurisdiction's currency (`currency_for(JURISDICTION)`, v2.19)
13. Days to Opt In = 0 / Days to Entitlement = 0 / Days to Settlement = 1
14. Activation start = code date minus 1 day at midnight ET; end = year 2041
15. **Status Active = checked**
16. Reporting Platform = CAS (Casino); Origin = Bet Settlement Bonus; **Series = Early Life** (selected by `title="Early Life"` attribute); Type = Acquisition
17. Segment / Client Profiling = matching SUO DAY{n} segment
18. Save — **no confirmation modal, no post-save edit pass**

---

## localStorage Keys

| Key | What it stores |
|---|---|
| `rtc_hiw` | Custom How It Works — RTC CC (**shared** by US and CA) |
| `rtc_tc` | Custom T&C — RTC CC (US) |
| `rtc_tc_ca` | Custom T&C — RTC CC (CA — Canada) |
| `rtc_image_path` | Image folder path — RTC CC (US) |
| `rtc_image_path_ca` | Image folder path — RTC CC (CA — Canada) |
| `rtc_terms_ids` | JSON object mapping amount → Terms ID — RTC CC (US) |
| `rtc_terms_ids_ca` | JSON object mapping amount → Terms ID — RTC CC (CA — Canada) |
| `rtc_terms_start_date` | Promotional start date in `MM/DD/YY` format — RTC CC (US) |
| `rtc_terms_start_date_ca` | Promotional start date in `MM/DD/YY` format — RTC CC (CA) |
| `rtc_terms_end_date` | Promotional end date in `MM/DD/YY` format — RTC CC (US) |
| `rtc_terms_end_date_ca` | Promotional end date in `MM/DD/YY` format — RTC CC (CA) |
| `rtc_fs_hiw` | Custom How It Works — RTC FS |
| `rtc_fs_tc` | Custom T&C — RTC FS |
| `rtc_fs_image_path` | Image folder path — RTC FS |
| `bg_hiw` | Custom How It Works — BG |
| `bg_tc` | Custom T&C — BG |
| `dm_hiw` | Custom How It Works — DM |
| `dm_tc` | Custom T&C — DM |
| `raf_image_path` | Image folder path — RAF |
| `lc_react_hiw` | Custom How It Works — LC-REACT |
| `lc_react_tc` | Custom T&C — LC-REACT |
| `lc_react_image_path` | Image folder path — LC-REACT |
| `lc_churn_dm_hiw` | Custom How It Works — LC-CHURN-DM |
| `lc_churn_dm_tc` | Custom T&C — LC-CHURN-DM |
| `lc_churn_dm_image_path` | Image folder path — LC-CHURN-DM |
| `vcl_dm_hiw` | Custom How It Works — VIPDM (CA — Canada) |
| `vcl_dm_tc` | Custom T&C — VIPDM (CA — Canada) |
| `vcl_dm_image_path` | Image folder path — VIPDM (CA — Canada) |
| `vcl_dm_terms_ids` | JSON object mapping offer key → Terms ID — VIPDM (CA) |
| `vcl_dm_terms_start_date` | Promotional start date in `MM/DD/YY` format — VIPDM (CA) |
| `vcl_dm_terms_end_date` | Promotional end date in `MM/DD/YY` format — VIPDM (CA) |
| `vcl_dm_hiw_us` | Custom How It Works — VIPDM (US — United States) |
| `vcl_dm_tc_us` | Custom T&C — VIPDM (US — United States) |
| `vcl_dm_image_path_us` | Image folder path — VIPDM (US — United States) |
| `vcl_dm_terms_ids_us` | JSON object mapping offer key → Terms ID — VIPDM (US) |
| `vcl_dm_terms_start_date_us` | Promotional start date in `MM/DD/YY` format — VIPDM (US) |
| `vcl_dm_terms_end_date_us` | Promotional end date in `MM/DD/YY` format — VIPDM (US) |
| `vcl_bg_hiw` | Custom How It Works — VIPBG |
| `vcl_bg_tc` | Custom T&C — VIPBG |
| `vcl_bg_image_path` | Image folder path — VIPBG |
| `vcl_bg_terms_ids` | JSON object mapping offer key → Terms ID — VIPBG |
| `vcl_bg_terms_start_date` | Promotional start date in `MM/DD/YY` format — VIPBG |
| `vcl_bg_terms_end_date` | Promotional end date in `MM/DD/YY` format — VIPBG |
| `vcl_bg_hiw_us` | Custom How It Works — VIPBG (US — United States) |
| `vcl_bg_tc_us` | Custom T&C — VIPBG (US — United States) |
| `vcl_bg_image_path_us` | Image folder path — VIPBG (US — United States) |
| `vcl_bg_terms_ids_us` | JSON object mapping offer key → Terms ID — VIPBG (US) |
| `vcl_bg_terms_start_date_us` | Promotional start date in `MM/DD/YY` format — VIPBG (US) |
| `vcl_bg_terms_end_date_us` | Promotional end date in `MM/DD/YY` format — VIPBG (US) |
| `daily_missions_image_path` | Image folder path — Daily Missions - Canada (v2.28) |

The base `vcl_bg_*` and `vcl_dm_*` keys hold the CA — Canada values (v2.25 / v2.26). SUO Day 2+ Spins and BG-RE have no localStorage keys; Daily Missions - Canada has only the image-path key (HIW, T&C, Terms IDs, and dates are baked into the HTML). Clear Saved Settings removes all keys above, including every `_ca` and `_us` variant and `daily_missions_image_path`.

---

## Image System

### RTC Top Up - Casino Credit
Jurisdiction-scoped as of v2.22. Path stored in `localStorage` as `rtc_image_path` (US) / `rtc_image_path_ca` (CA). Edit Images button present; the modal reads/saves the selected region's path and its Drive link follows the region.
**US default path (Adrian's machine):** `.../Marketing Automations/Lifecycle Automations/RTC Top Up`
**US Google Drive folder:** https://drive.google.com/drive/folders/1Dlpa3xZTHjzlwHgayPSS-MZJ8-DQTmhh
**CA default path (Adrian's machine):** `.../Marketing Automations/Lifecycle Automations/RTC Top Up - Canada`
**CA Google Drive folder:** https://drive.google.com/drive/folders/1W6ulHKv3ted9ZLNKzPgEbRoNCK_wFz6O
Image filenames follow the same convention in both folders: `Promo Detail.png`, `Masthead.png`, `{amount}.png`.

### RTC Top Up - Free Spins
Path stored in `localStorage` as `rtc_fs_image_path`. Edit Images button present.
**Default path (Adrian's machine):** `.../Marketing Automations/Lifecycle Automations/RTC Top Up - Free Spins`
**Google Drive folder:** https://drive.google.com/drive/folders/1FoIMPDbCrHYxdXUY6nSv_ffARWYD5mK1

### Bet & Get and Deposit Match
Images attached as a ZIP file per day card. Embedded as base64 in generated script. Edit Images button hidden.

### Refer a Friend
Path stored in `localStorage` as `raf_image_path`. Edit Images button present.
**Default path (Adrian's machine):** `.../Marketing Automations/Lifecycle Automations/RAF Creative`
**Image filenames:** `Promo Detail.png`, `Masthead.png`, `Discover_RFRE.png`, `Discover_RFER.png`, `Link Preview.png`

### SUO Day 2+ Spins
No images required. Edit Images button hidden.

### Bet & Get - Rules Engine
Images attached as a ZIP file per day card. Identical to existing BG. Edit Images button hidden.

### Lifecycle - REACT CC Drop
Path stored in `localStorage` as `lc_react_image_path`. Edit Images button present.
**Default path (Adrian's machine):** `.../Marketing Automations/Lifecycle Automations/Reactivation Bonus Drop`
**Google Drive folder:** https://drive.google.com/drive/folders/1Xhk-3fKF3goNS_lL7M0bn2DrACKyFRds
**Image filenames:** `Promo Detail.png`, `Masthead.png`, `{amount}.png` (e.g. `10.png`, `25.png`, `50.png`)

### Lifecycle - Churn DM
Path stored in `localStorage` as `lc_churn_dm_image_path`. Edit Images button present.
**Default path (Adrian's machine):** `.../Marketing Automations/Lifecycle Automations/Churn Risk DM`
**Google Drive folder:** https://drive.google.com/drive/folders/1KJfVtJVwBbGkJJTr8T-PGeWHm5c8qc-2
**Image filenames:** `Promo Detail.png`, `Masthead.png`, `{pct}M{max}.png` (e.g. `50M10.png`). Min deposit is not part of the Discover filename.

### VIP Offer Library - Deposit Matches (v2.23 CA / v2.26 US)
Region-scoped as of v2.26. Path stored in `localStorage` as `vcl_dm_image_path` (CA) / `vcl_dm_image_path_us` (US). Edit Images button present; the modal reads/saves the selected region's path, shows the region in its title, and its Drive link follows the region. Under the **VIP Automations** shared drive branch.
**CA default path (Adrian's machine):** `.../Marketing Automations/VIP Automations/Offer Library/Canada/Deposit Matches`
**CA Google Drive folder:** https://drive.google.com/drive/folders/1v9DEVtCNQPCgUkJsWtsXn7UJVIyBYtr4
**CA image filenames:** `Promo Detail.png`, `Masthead.png`, `{pct}M{max}.png`. Min deposit is not part of the Discover filename. Full default-tier set: `50M250.png`, `100M250.png`, `10M250.png`, `10M2500.png`, `10M5000.png`, `20M500.png`, `20M1000.png`, `20M2000.png`, `20M5000.png`.
**US default path (Adrian's machine):** `.../Marketing Automations/VIP Automations/Offer Library/USA/Deposit Matches`
**US Google Drive folder:** https://drive.google.com/drive/folders/1W8GLxrZFBMotLP_WPzAE04KPPb__BxIX
**US image filenames:** `Promo Detail.png`, `Masthead.png`, `{pct}M{max}.png`. Full default-tier set: `50M250.png`, `100M250.png`, `10M500.png`, `10M1000.png`, `10M2500.png`, `10M5000.png`, `20M500.png`, `20M1000.png`, `20M2000.png`, `20M5000.png`.
> ⚠️ Several filenames differ by one character — CA: `10M250.png` vs `100M250.png`; US: `10M500.png` vs `10M5000.png`, `20M500.png` vs `20M5000.png`, and `100M250.png` vs `10M250`-style shapes. Double-check the creatives land in the correct files; a mislabeled file uploads the wrong tier's image without erroring.

### VIP Offer Library - Bet & Gets (v2.24 CA / v2.25 US / v2.27 CA creatives)
Region-scoped as of v2.25. Path stored in `localStorage` as `vcl_bg_image_path` (CA) / `vcl_bg_image_path_us` (US). Edit Images button present; the modal reads/saves the selected region's path, shows the region in its title, and its Drive link follows the region. Under the **VIP Automations** shared drive branch, sibling to Deposit Matches.
**CA default path (Adrian's machine):** `.../Marketing Automations/VIP Automations/Offer Library/Canada/Bet & Gets`
**CA Google Drive folder:** https://drive.google.com/drive/folders/1tkMs1dx-gszSGzlBHxgMYvOYv2l-2UjO
**CA image filenames (v2.27 — category-specific):** `Slots Promo Detail.png` + `Slots Masthead.png` (used for all SBG offers), `Tables Promo Detail.png` + `Tables Masthead.png` (used for all TBG offers), and a Discover image `{SBG|TBG}_B{bet}_G{get}.png` — the Discover filename is the full offer key (e.g. `SBG_B1000_G100.png`, `TBG_B100000_G2500.png`), so no one-character ambiguity. The generic `Promo Detail.png` / `Masthead.png` are no longer referenced by CA VIPBG scripts. Default-tier Discover set: `SBG_B1000_G100.png`, `SBG_B5000_G500.png`, `SBG_B20000_G1000.png`, `SBG_B100000_G5000.png`, `TBG_B2000_G100.png`, `TBG_B10000_G500.png`, `TBG_B25000_G1000.png`, `TBG_B100000_G2500.png`.
**US default path (Adrian's machine):** `.../Marketing Automations/VIP Automations/Offer Library/USA/Bet & Gets`
**US Google Drive folder:** https://drive.google.com/drive/folders/13neRGeYM4JLa98p827vbepp-eKu2_hXq
**US image filenames:** `Promo Detail.png`, `Masthead.png`, `B{bet}_G{get}.png` (no category prefix — all US tiers are slots). Default-tier set: `B1000_G100.png`, `B5000_G500.png`, `B20000_G1000.png`, `B100000_G5000.png`.

### Daily Missions - Canada (v2.28)
Path stored in `localStorage` as `daily_missions_image_path`. Edit Images button present (Drive link follows).
**Default path (Adrian's machine):** `/Users/adrian.vandecamp/Library/CloudStorage/GoogleDrive-adrian.vandecamp@betfanatics.com/Shared drives/Marketing Automations/Bau Automations/Daily Missions`
**Google Drive folder:** https://drive.google.com/drive/folders/1w-o2bP7em8fkT86lwOtCu66jO57igRLh
**Image filenames (underscored, weekday-prefixed — weekday auto-derived from the typed date):** `{Weekday}_Promo_Detail.png`, `{Weekday}_Masthead.png`, CC discover `{Weekday}_B{bet}_G{get}.png` (e.g. `Tuesday_B125_G5.png`), FS discover `{Weekday}_B{bet}_G{spins}S_{value}V.png` with the spin value in cents (e.g. `Wednesday_B50_G20S_10V.png`; no game token — the weekday implies the game; **active as of v2.30**). Each usable weekday needs its own Promo Detail + Masthead + one discover per tier. The generated script pre-flight checks all required files before login and aborts with a missing-file list.

---

## Naming Conventions

### v3.0 Skeleton (all live offer types)

```
MMDDYY_JURISDICTION_PRODUCT_LIFECYCLE_SUBCATEGORY_MECHANIC_AWARDTYPE_(FREEFORM)
```

- **7 fixed, non-empty sections + freeform.** Generated scripts parse positionally: `parts = code.split("_")`; `parts[0]` date, `parts[1]` jurisdiction, `parts[2..6]` product/lifecycle/subcategory/mechanic/awardtype, `"_".join(parts[7:])` freeform verbatim (freeform may contain underscores).
- **Hard limit 40 characters** (NATS bonus name field truncates at 40; segment/promo fields designed to the same limit). Generation-time guards in every live builder; runtime asserts in SUO/RAF.
- **K-notation:** freeform values ≥ 1,000 and evenly divisible by 1,000 render as `{n}K` (`100000` → `100K`); everything else stays literal (`2500` stays `2500`; decimals disallowed). Scripts expand K before any field fill, Terms ID key, or filename derivation. Required today only on the VIP tiers; every migrated parser accepts K anywhere.
- **Section vocabularies (used tokens):** PRODUCT `CAS` (Casino) / `VCAS` (VIP Casino); LIFECYCLE `RET` / `LC` / `ACQ`; SUBCATEGORY `RTC` / `BAU` (Generic Offer) / `SUO` / `REACT` / `CHURN` / `OL` (Offer Library); MECHANIC `OPT` / `BG` / `DM` / `DR` (Drop) / `SBG` / `TBG` / `RAF`; AWARDTYPE `CC` / `FS` / `FC`. Reserved (unused) tokens exist for each section — see the v3.0 naming spec document. ⚠️ The reserved AWARDTYPE `CA` (Cash) collides visually with jurisdiction `CA` — harmless under positional parsing.
- **RTC's LIFECYCLE token is `RET` provisionally** — may change to `LC`; nothing parses it, so it is a one-constant swap.

### Internal Code Names — Live Offer Types (v3.0)

| Offer | Format | Example |
|---|---|---|
| RTC CC | `MMDDYY_{JUR}_CAS_RET_RTC_OPT_CC_{amount}` | `053026_US_CAS_RET_RTC_OPT_CC_50` |
| LC-REACT | `MMDDYY_{JUR}_CAS_LC_REACT_OPT_CC_{amount}` | `122926_US_CAS_LC_REACT_OPT_CC_10` |
| LC-CHURN-DM | `MMDDYY_{JUR}_CAS_LC_CHURN_DM_CC_M{min}_{pct}M{max}` | `121126_US_CAS_LC_CHURN_DM_CC_M10_50M10` |
| VIPDM (US) | `MMDDYY_US_VCAS_RET_OL_DM_FC_M{min}_{pct}M{max}` | `090126_US_VCAS_RET_OL_DM_FC_M250_20M5K` |
| VIPDM (CA) | `MMDDYY_CA_VCAS_RET_OL_DM_CC_M{min}_{pct}M{max}` | `090126_CA_VCAS_RET_OL_DM_CC_M250_20M1K` |
| VIPBG (US) | `MMDDYY_US_VCAS_RET_OL_SBG_FC_B{bet}_G{get}` | `090126_US_VCAS_RET_OL_SBG_FC_B100K_G5K` |
| VIPBG (CA) | `MMDDYY_CA_VCAS_RET_OL_{SBG\|TBG}_CC_B{bet}_G{get}` | `090126_CA_VCAS_RET_OL_TBG_CC_B100K_G2500` |
| RAF Referee | `MMDDYY_{JUR}_CAS_ACQ_SUO_RAF_FS_RFRE_{spins}` | `100126_MI_CAS_ACQ_SUO_RAF_FS_RFRE_50` |
| RAF Referrer | `MMDDYY_{JUR}_CAS_ACQ_SUO_RAF_FS_RFER_{spins}` | `100126_MI_CAS_ACQ_SUO_RAF_FS_RFER_50` |
| RAF Day 2–N | `MMDDYY_{JUR}_CAS_ACQ_SUO_RAF_FS_DAY{n}_{spins}` | `100126_MI_CAS_ACQ_SUO_RAF_FS_DAY2_50` |
| SUO Day 2–N | `MMDDYY_{JUR}_CAS_ACQ_SUO_DR_FS_DAY{n}_{spins}` | `100126_MI_CAS_ACQ_SUO_DR_FS_DAY2_50` |
| DMCA (CC — Tue/Thu/Sat) | `MMDDYY_CA_CAS_RET_BAU_BG_CC_B{bet}_G{get}` | `120126_CA_CAS_RET_BAU_BG_CC_B125_G5` |
| DMCA (FS — Mon/Wed/Fri/Sun) | `MMDDYY_CA_CAS_RET_BAU_BG_FS_T{n}` | `113026_CA_CAS_RET_BAU_BG_FS_T2` |

**VIPBG category:** the SBG/TBG category lives in the MECHANIC slot (`parts[5]`), not the freeform; US always carries `SBG` (all US tiers are slots). **DMCA FS tier index:** `T1`–`T4` = bets $50 / $125 / $500 / $5,000 (spins 20/50/50/100 @ $0.10/$0.10/$0.50/$3.00); the actual values are baked into the generated script's per-code `CODES` config at generation time — the code is a label, not the data source. An FS bet outside the T1–T4 map blocks generation with an alert. **Terms ID keys stay in their v2.x literal shapes** (`M{min}_{pct}M{max}`, `{SBG|TBG}_B{bet}_G{get}`, `B{bet}_G{get}`) — scripts rebuild them from K-expanded parse results, so saved localStorage maps survive the upgrade unchanged.

### Internal Code Names — Retired Offer Types (v2.x format, for reference)

These four builders remain in the HTML byte-identical to v2.34 but are removed from the offer-type dropdown; if ever re-exposed they would generate these v2.x codes:

| Offer | Format (v2.x) | Example |
|---|---|---|
| RTC FS | `MMDDYY_CAS_RET_RTC_{jurisdiction}_FS_{spins}S_{value}V_{game}` | `061926_CAS_RET_RTC_MI_FS_10S_20V_TCE` |
| BG | `MMDDYY_CAS_RET_BG_{jurisdiction}_FC_B{bet}_G{get}` | `053026_CAS_RET_BG_US_FC_B100_G2` |
| BG-RE | `MMDDYY_CAS_RET_BG_{jurisdiction}_{rewardType}_B{bet}_G{get}` | `062226_CAS_RET_BG_US_FC_B100_G2` |
| DM | `MMDDYY_CAS_RET_DM_{jurisdiction}_FC_M10_{pct}M{max}` | `073126_CAS_RET_DM_US_FC_M10_20M500` |

### Voucher Codes (unchanged in v3.0 — every format identical to v2.34)

Retired offer types (RTC FS, BG, DM, BG-RE) keep their entries for reference. VIPBG's voucher remains `CLBGFC` (historically shared with BG/BG-RE; the random suffix keeps it unique).


| Offer | Format | Example |
|---|---|---|
| RTC CC | `CRRCCC{MMDDYY}{XXX}` | `CRRCCC052626AB3` |
| RTC FS | `CRRCFS{MMDDYY}{XXX}` | `CRRCFS052626Q7Z` |
| BG | `CLBGFC{MMDDYY}{XXX}` | `CLBGFC052626X2K` |
| BG-RE | `CLBGFC{MMDDYY}{XXX}` | `CLBGFC052626X2K` |
| DM | `CRDMFC{MMDDYY}{XXX}` | `CRDMFC052626M9R` |
| LC-REACT | `CRLCCC{MMDDYY}{XXX}` | `CRLCCC122926AB3` |
| LC-CHURN-DM | `CRLCDM{MMDDYY}{XXX}` | `CRLCDM121126X4T` |
| VIPDM (CA) | `VRDMCC{MMDDYY}{XXX}` | `VRDMCC090126X4T` |
| VIPDM (US) | `VRDMFC{MMDDYY}{XXX}` | `VRDMFC090126X4T` |
| VIPBG (US & CA) | `CLBGFC{MMDDYY}{XXX}` — shared with BG/BG-RE | `CLBGFC090126X4T` |
| DMCA | None — bonus is built in Playmaker | — |
| RAF Referee | `{jurisdiction}AQ{total_spins:04d}RFRE{XXX}` | `MIAQ0500RFREX3K` |
| RAF Referrer | `{jurisdiction}AQ{total_spins:04d}RFER{XXX}` | `MIAQ0500RFERX3K` |
| RAF Day 2–N | `{jurisdiction}AQ{total_spins:04d}DAY{n}{XXX}` | `MIAQ0500DAY2X3K` |
| SUO Day 2–N | None — no voucher codes | — |

### Generated Script Filenames

| Offer | Filename |
|---|---|
| RTC CC | `rtc_top_up_MMDDYY_HHMM.py` |
| RTC FS *(retired)* | `rtc_fs_MMDDYY_HHMM.py` |
| BG *(retired)* | `bg_MMDDYY_HHMM.py` |
| BG-RE *(retired)* | `bg_re_MMDDYY_HHMM.py` |
| DM *(retired)* | `dm_MMDDYY_HHMM.py` |
| RAF | `raf_full_campaign_MMDDYY_HHMM.py` |
| SUO | `suo_day2_MMDDYY_HHMM.py` |
| LC-REACT | `lc_react_MMDDYY_HHMM.py` |
| LC-CHURN-DM | `lc_churn_dm_MMDDYY_HHMM.py` |
| VIPDM | `vcl_dm_MMDDYY_HHMM.py` |
| VIPBG | `vcl_bg_MMDDYY_HHMM.py` |
| DMCA | `daily_missions_MMDDYY_HHMM.py` |

---

## LC-REACT T&C Data (Baked In at Generation Time)

| Amount | Terms ID | Promotional Period |
|---|---|---|
| $10 | `CAS_9237` | April 16, 2026 → October 16, 2026 |
| $25 | `CAS_9238` | April 16, 2026 → October 16, 2026 |
| $50 | `CAS_9456` | June 1, 2026 → December 1, 2026 |

T&C text is generated via `getLCREACTTC(amount)` in the HTML tool at generation time and baked into the `TC_BY_AMOUNT` dict in the generated script. Changes to T&C copy require regenerating the script (same known limitation as DM).

---

## Platform Tags — Bonus Creation

NATS pre-populates platform tags on every new bonus. All generated scripts remove `COMBO: Sportsbook` and `COMBO: Sportsbook And Casino`. The resulting active platforms on every NATS bonus are:

- `STAC: Standalone Casino`
- `Web`
- `COMBO: Casino`

Applies to the eight pre-v2.23 NATS offer types (RTC CC, RTC FS, BG, DM, RAF, SUO, LC-REACT, LC-CHURN-DM). BG-RE bonuses are built in Playmaker, not NATS, so platform tags do not apply.

**VIP Offer Library exception (v2.23/v2.24):** **CA** VIPDM and **CA** VIPBG remove **all three** COMBO tags — `COMBO: Sportsbook`, `COMBO: Sportsbook And Casino`, and `COMBO: Casino` — leaving only `Web` + `STAC: Standalone Casino`. They are the only flows that remove `COMBO: Casino`. **US VIPBG (v2.25) and US VIPDM (v2.26) follow the standard pattern** — remove only the two Sportsbook tags and keep `COMBO: Casino`.

---

## Multi-Currency Amount Tables (v2.18+)

As of August 2026, NATS renders bonus amount tables with **one row per currency**:

```html
<tr data-row-key="USD" class="ant-table-row ...">
  <td class="ant-table-cell">USD</td>
  <td class="ant-table-cell center"> ... <input class="ant-input-number-input" ...> ... </td>
  ...
</tr>
<tr data-row-key="GBP" ...>   <!-- never used -->
<tr data-row-key="CAD" ...>
```

### Currency mapping

| Jurisdiction at code position 1 (v3.0) | Currency row |
|---|---|
| `US`, `MI`, `WV`, `PA`, `NJ` | `USD` |
| `ON`, `AB`, `CA` | `CAD` |
| — | `GBP` is **never used under any circumstance** |

### Shared helpers (embedded in all eleven generated script templates)

- `currency_for(name)` — **v3.0: positional.** Reads `name.split("_")[1]` (the fixed jurisdiction slot); returns `"CAD"` for ON/AB/CA and `"USD"` for US/MI/WV/PA/NJ, and **raises `ValueError` on anything else** — a v2.x-shaped code or the reserved `CA` (Cash) awardtype token can never misroute the row. In RAF and SUO, `currency_for(jur)` takes the bare `JURISDICTION` constant directly (those scripts never parse codes) with the same validation. The four retired builders retain the old token-hunting version, byte-identical to v2.34.
- `fill_currency_amount(page, name, value, scope=None, label=...)` — fills the value into `tr[data-row-key='{currency}'] input.ant-input-number-input` (optionally scoped to a `.summaryCell` card). If no currency rows exist, falls back to the legacy single-input selector (`.ant-table-cell.center .ant-input-number-input`), so scripts keep working if NATS reverts. Prints the currency (or `legacy input`) to the terminal.

### Fill sites using the helper

| Offer | Fields filled via currency row |
|---|---|
| RTC CC | Casino Credit amount |
| LC-REACT | Casino Credit amount |
| BG *(retired)* | FanCash amount |
| DM *(retired)* | Bonus amount card, Minimum Deposit card, Maximum Deposit card |
| LC-CHURN-DM | Bonus/Casino Credit amount card, Minimum Deposit card, Maximum Deposit card |
| VIPDM | Reward amount card, Minimum Deposit card, Maximum Deposit card — CAD row (CA / Casino Credit), USD row (US / FanCash) |
| VIPBG | Flat reward amount (= the "get" value) — CAD row (CA / Casino Credit), USD row (US / FanCash); no deposit fields |
| RAF / SUO (and retired RTC FS) | Spin value row in the **Free Spin Stakes** table (select dropdown, not a number input) — row chosen by currency label cell via `currency_for()` (v2.19) |

Percentage inputs (`aria-valuemin='0' aria-valuemax='100'`, used by DM and LC-CHURN-DM) are **not** part of the currency table and are unchanged.

---

## Casino Game Selection — ™-Tolerant Matching (v2.20/v2.21) + Search Cap Overrides (v2.33)

NATS game option names include trademark symbols, e.g. `Triple Cash Eruption™ (WV)`, and **NATS's game search is hard-capped at ~12 rendered results with no scrollable overflow** (established Aug 24, 2026 — see the v2.33 changes bullet). Game selection in RTC FS, RAF, and SUO works in two decoupled steps:

1. **Search** — `parse_casino_game_search()` types a query into the Casino Game search box whose only job is to get the target entry inside the ~12-result cap:
   - **Games with a `searchOverride`** (7FH: `Hotstepper`; 7H2: `Hotstepper 2`) type **`{override} ({jurisdiction})`** — e.g. `Hotstepper 2 (NJ)`. NATS's search is contiguous-substring matching over the full option text, suffix included, so this returns a near-unique result (1 hit for 7H2 per state; 2 for 7FH on MI, where `Lucky Fire Blitz™ Hotstepper (MI)` also matches — harmless, the match step rejects it). The override must be a **contiguous** run of the option text ending adjacent to the suffix — which is why the pre-™ name cannot simply have the suffix appended (`™` and any following words break contiguity).
   - **Games without one** (TCE, 7P5, WWE) type the portion of the `searchName` **before the first `™`** (`Triple Cash Eruption`, `7's Fire Blitz`, `WrestleMania`), unchanged from v2.20.
2. **Match** — `parse_casino_game_pattern()` selects the dropdown option with an anchored regex built from the full `searchName` where **every `™` is optional** (mid-name and trailing) and the **jurisdiction suffix is required**: `^{name-with-optional-™}™?\s*\({JUR}\)$`. The correct state's game is always chosen whether or not NATS includes `™`; wrong jurisdictions, similarly-named games, and `DONOTUSE-` prefixed entries can never match. The terminal prints the actual option text clicked. **Unchanged in v2.33** — the match step, not the search step, owns correctness.

The `GAMES` table's `searchName` values are unchanged — the matcher tolerates NATS adding or removing `™` without further edits. Adding a `searchOverride` to a game requires updating the `FS_GAMES` UI map **and** the RAF and SUO embedded `GAMES` maps (the RTC FS dict is generated from `FS_GAMES` automatically).

Available games as of v2.33: **TCE** (Triple Cash Eruption, IGT/IGT_Rgs), **7FH** (7's Fire Blitz™ Hotstepper, WHG/WHITEHATSTUDIOS — `searchOverride: Hotstepper`, v2.33), **7H2** (7's Fire Blitz™ Hotstepper 2, WHG/WHITEHATSTUDIOS — added v2.32; `searchOverride: Hotstepper 2`, v2.33), **7P5** (7's Fire Blitz™ Power 5 Jackpot Royale™ Express, WHG/WHITEHATSTUDIOS — ⚠️ shares the broad `7's Fire Blitz` search, exposed to the cap; see Known Limitations), **WWE** (WrestleMania™: Road to Gold, WHG/WHITEHATSTUDIOS). The full table with deeplinks and search names lives in the project instructions. 7FH and 7H2's anchored patterns are structurally cross-exclusive (jurisdiction suffix required; `Hotstepper` vs `Hotstepper 2` anchored).

⚠️ **Catalog hazards confirmed in the Aug 24 dumps:** some game families use **curly apostrophes** (`7's Fire Blitz™ Power Force 5`) — a straight-apostrophe query returns zero results for them, so any future `FS_GAMES` addition must have its exact catalog spelling captured (the `suo_7h2_dropdown_debug_v0_1.py` repr-dump pattern verifies this in one run). `Lucky Fire Blitz™ Hotstepper (MI)` exists and shadows the 7FH override's search results.

> ⚠️ **Template escaping rule (root cause of the v2.20 hotfix):** the Python scripts are generated inside JavaScript template literals, which swallow single backslashes. Any regex backslash intended for the generated Python (`\s`, `\(`, `\)`) must be written **double-escaped** in the HTML source. When changing template regexes, verify by rendering the template section through a real JS template literal (node) and executing the resulting Python before shipping.

---

## Platform Selectors Reference

| Selector | Used For |
|---|---|
| `[data-menu-id$='-segments']` | Sidebar Segments icon |
| `[data-menu-id$='-cms']` | Sidebar CMS icon |
| `[data-menu-id$='-bonuses']` | Sidebar Bonuses icon |
| `.dispatch_event("click")` | All sidebar submenu clicks + AMELCO dropdown |
| `.ant-legacy-form-item` | Form rows |
| `.ant-select-selector` | Dropdown triggers |
| `.ant-select-item-option-content` | Dropdown option items |
| `.ant-select-dropdown:not(.ant-select-dropdown-hidden)` | Currently open dropdown (used for Segment and Promotion Tile in all offer types) |
| `.ant-select-item-option[title='Early Life']` | Early Life series option (SUO) |
| `.ant-input-number-input` | Numeric inputs |
| `tr[data-row-key='USD'\|'GBP'\|'CAD']` | Currency rows in bonus amount tables (v2.18+) — scripts fill the jurisdiction's row only |
| `td.ant-table-cell` (has_text=currency) → parent `tr` | Free Spin Stakes currency row anchor (RTC FS / RAF / SUO) |
| `.ant-checkbox-input` | Checkboxes |
| `label.ant-checkbox-wrapper` | Checkbox wrapper (Send To XP) |
| `label[title='Status Active?']` | Status Active row |
| `.ant-switch` | Toggle switches |
| `.ant-modal-content` | Confirmation modals |
| `.ProseMirror[contenteditable='true']` | Rich text editors |
| `button.ant-btn-primary.save` | Save buttons |
| `button.ant-btn-primary.create` | Create buttons |
| `input.ant-radio-input[value='REFEREE']` | RAF Referee radio |
| `input.ant-radio-input[value='REFERRER']` | RAF Referrer radio |
| `input[placeholder='Password']` | RAF voucher modal input |

---

## Confirmation Modals After Bonus Save

| Offer | Modal |
|---|---|
| RTC CC | No |
| RTC FS | No |
| BG | Yes |
| DM | Yes |
| RAF RFRE | Yes — voucher modal → Apply |
| RAF RFER | Yes — same as RFRE |
| RAF Day 2+ | No |
| SUO Day 2+ | No |
| BG-RE | No |
| LC-REACT | No |
| LC-CHURN-DM | No |
| VIPDM | No |
| VIPBG | No — handled defensively (confirms if one ever appears) |
| DMCA | No (NATS promo has none; the bonus is saved in Playmaker, which has no NATS-style confirmation modal) |

---

## Known Bugs

| # | Area | Severity | Issue |
|---|---|---|---|
| 2 | DM | ⚠️ | `format_hiw` / `format_tc` baked in at generation time — modal overrides require script regeneration |
| 5 | RTC FS | ⚠️ | HIW and T&C use placeholder copy — pending legal-approved text |
| 6 | RAF | ⚠️ | NATS clears Referee/Referrer radio and Reporting Platform on bonus re-open — post-save edit pass required |
| 7 | RAF | ⚠️ | Bonus Manager requires 120-second wait during RFER edit pass — script stays on page |

---

## Known Limitations

| # | Area | Note |
|---|---|---|
| 1 | BG-RE | NATS and Playmaker URLs are currently pointed at the **test environment**. Switch to production URLs once the Rewards Engine is live in the Playmaker production environment. |
| 2 | BG-RE | Free Spins reward type not yet implemented — different field structure in Playmaker. |
| 3 | LC-REACT | T&C promotional periods and Terms IDs are baked in at generation time. Changes to any of the three tiers require regenerating the script. |
| 4 | LC-CHURN-DM | HIW, T&C, and Terms IDs are baked in at generation time. Changes require regenerating the script. New offer sizes require adding a Terms ID to `CHURNDM_TERMS_IDS` in the HTML — offers without a configured Terms ID block script generation by design. |
| 5 | VIPDM | HIW, T&C, Terms IDs, and promotional dates are baked in at generation time — changes via the Edit T&Cs / Terms modal require regenerating the script. New offer sizes require a Terms ID (editable in the Terms modal); active offers without one block script generation by design. |
| 6 | VIPDM | Series `VIP` must exist as an option in the NATS Series dropdown (confirmed present as of Aug 2026). If NATS ever removes it, scripts will time out at Series selection. |
| 7 | VIPDM | Dual region as of v2.26 (US: FanCash/USD, defaults US on offer-type selection; CA: Casino Credit/CAD). **US VIPDM scripts must be generated from v2.26+** — earlier versions were CA-locked. (This row previously said "CA only / US planned" — stale as of v2.26, corrected in the v2.28 doc pass.) |
| 8 | VIPBG | HIW, T&C, Terms IDs, and promotional dates are baked in at generation time — changes via the region-aware Edit T&Cs / Terms modal require regenerating the script. New offer sizes require a Terms ID (editable in the Terms modal; keyed `{SBG\|TBG}_B{bet}_G{get}` for CA, `B{bet}_G{get}` for US); active offers without one block script generation by design. |
| 9 | VIPBG | Dual region as of v2.25 (US: FanCash/USD; CA: Casino Credit/CAD). Button 2 is empty for CA and `Play Now!` → `/docs/usered/casgenericgamelist` for US. Series `VIP` shares limitation #6 with VIPDM. **US VIPBG scripts must be generated from v2.25+** — earlier versions were CA-locked. |
| 10 | VIPBG | The US canonical T&C was verified byte-level against the CAS_10687 and CAS_10688 legal filings (with four stakeholder-approved copy-edit normalizations); the CAS_10689 / CAS_10690 filings were not supplied and are assumed to follow the same template — verify on first use. First live US run should also confirm the FanCash confirmation modal fires and whether Stake Chunk Sizes exists on the FanCash form (the script handles both outcomes defensively). |
| 11 | DMCA | **Free Spins days unlocked as of v2.30** — FS-day scripts must be generated from v2.30+. Spin values are validated against per-game stakes ladders baked into the HTML (captured from PROD Aug 2026; MFB caps at 50.00) — if Playmaker changes a game's ladder, the HTML ladders must be updated (reach out to Adrian); the runtime hard-fail (dump options + refuse to substitute) protects against drift in the meantime. When the weekday games rotate, the `DMCA_GAMES` map (name/token/search/stakes), FS Terms IDs, FS creatives, and the FS promotional-date pairs all need updating — the spec defines five independent date pairs (Mon/Wed/Fri/Sun/CC-shared) but v2.30 bakes one shared pair (08/17/26 → 12/31/26) since all sixteen FS filings currently share the CC window. Note v2.30 also moved **all** DMCA Playmaker bonus times (CC included) from 00:00 to 00:01 — regenerate any pending DMCA scripts. |
| 12 | DMCA | HIW, T&C, Terms IDs, and promotional dates are baked into the HTML at generation time with **no edit modals** (Edit HIW / Edit T&Cs hidden by design) — changes require editing the HTML (reach out to Adrian) and regenerating scripts. The CC T&C was byte-verified against the supplied CAS_CA_0019/0020 filings and the FS T&C against **all sixteen FS filings (CAS_CA_0043–0058)** — each with two approved normalizations. **CAS_CA_0021/0022 (the CC $500/$5,000 tiers) remain unsupplied** and are assumed to follow the CC template — verify on first use. |
| 13 | DMCA | ~~CC code names are shape-identical to standard BG CA codes~~ **Closed in v3.0**: the DMCA CC code differs from a standard-BG shape at AWARDTYPE (`CC` vs `FC`), and standard BG is retired from the dropdown. Note the PROD cleanup list: the Aug 14 2026 validation promo `120126_CAS_RET_BG_CA_CC_B125_G5`, the v2.30 FS validation builds (`120426` MFB ×2, `121626` 4CC, `081926` 4CC — promos, bonuses, and their auto-created segments), and the Aug 20 2026 v2.31 validation build `101926_CAS_RET_BG_CA_FS_B5000_G100S_300V_BAA` (NATS segment + promo + Playmaker bonus) all exist in PROD — delete or avoid reusing those exact codes. |
| 14 | DMCA | The Okta persistent-profile launch (`~/.dm_canada_pw_profile` + LocalNetworkAccessChecks flag + permission grant) is DMCA-only for now; other offer types still use plain `chromium.launch()`. Back-porting is a candidate follow-up if Okta FastPass blocks appear elsewhere. |
| 15 | DMCA | The v2.31 flow depends on **NATS → Playmaker segment sync**: a NATS-built AMELCO segment must become selectable in Playmaker's Include → Existing Account Segments list (confirmed working in PROD Aug 20, 2026). Sync *lag* is absorbed by the ~60s poll; if sync ever breaks entirely, Phase 2 hard-fails by design rather than creating a Playmaker-side duplicate. The Include/Exclude discrimination and the chip read-back key off the visible section headings (`Include Segments` / `Exclude Segments`) and field labels — a Playmaker copy change to those strings would make the lookup fail closed (field dump + FAILED bonus, never the wrong field). **DMCA scripts must be generated from v2.31+** — v2.30 and earlier create a duplicate Playmaker-side segment via the Create field. |
| 16 | RAF / SUO (and retired RTC FS) | **NATS's Casino Game search is hard-capped at ~12 rendered results** (no scrollable overflow — confirmed Aug 24, 2026). Games whose typed search matches more catalog entries than the cap are selectable only where result ordering cooperates. v2.33 fixes 7FH and 7H2 via `searchOverride` + jurisdiction suffix (**scripts selecting either must be generated from v2.33+**; 7H2 confirmed failing on NJ/PA under v2.32, passing on MI by ordering luck). **7P5 remains exposed** — it shares the broad `7's Fire Blitz` search (rendered for MI/WV but not NJ/PA in the Aug 24 dumps), and the suffix fix does not transfer directly since `Power 5 ({JUR})` is not a contiguous substring of its option text; candidate override `Express ({JUR})` (contiguous; the anchored pattern disambiguates the `…Express` family) pending a PROD dropdown test. WWE and TCE are believed narrow enough to stay under the cap but are not dump-verified. If NATS changes the cap, result ordering, or catalog spellings (note: curly-apostrophe families exist), the repr-dump debug scripts from the Aug 24 session verify any game/state in one run. |

---

### v3.0 Additions

| # | Area | Note |
|---|---|---|
| v3-1 | All live builders | Codes are parsed positionally and fail loudly: >40 chars, <8 fields, non-jurisdiction at position 1 (catches v2.x-shaped codes before any NATS write), malformed freeforms, decimal/lowercase K, and (VIPBG CA) a mechanic outside SBG/TBG all raise `ValueError` at first parse. |
| v3-2 | VIPDM / VIPBG | K-notation is applied at generation time (values ≥1000 divisible by 1000). Exact-40 codes exist (`M250_10M2500`, `TBG…B100K_G2500`) — any future tier addition must re-check the budget; the generation-time guard blocks silent overflow. |
| v3-3 | DMCA FS | The `T1`–`T4` bet→index map (50/125/500/5000) is baked into `generateScript`; an FS bet outside it blocks generation with an alert. When the tiers rotate, the map must be updated alongside `DMCA_GAMES` (reach out to Adrian). The code is a label only — bet/spins/value/game live in the per-code `CODES` config. |
| v3-4 | Retired builders | RTC FS, BG, DM, and BG-RE remain in the HTML byte-identical to v2.34 but generate v2.x-format codes with the old token-hunting `currency_for` and **no 40-char guard or Promotion Tile retry** — if ever re-exposed, migrate them first. |
| v3-5 | RTC CC | The LIFECYCLE token `RET` is provisional (may change to `LC`); nothing parses it — a one-constant swap per RTC builder. |
| v3-6 | Coexistence | v2.x and v3.0 codes coexist safely in NATS/Playmaker (nothing re-parses stored codes). All v2.x "avoid reusing these exact codes" PROD-cleanup warnings are moot under v3.0 — collisions are impossible by format. Verify the Aug 25 Churn re-run did not leave a duplicate `120126_US_CAS_LC_CHURN_DM_CC_M10_50M10` segment/promo pair from the interrupted first run. |

## Prerequisites

- Python 3.9 or later (for `zoneinfo`)
- Setup instructions: https://adrianvdc.github.io/Promo-Building-Script-Generator/setup-guide
- Google Drive for Desktop required for RTC CC, RTC FS, RAF, LC-REACT, LC-CHURN-DM, VIPDM, VIPBG, and Daily Missions - Canada image path access
