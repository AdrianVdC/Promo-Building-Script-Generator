# NATS Bonus Creator — Project Instructions (v3.4)

## What This Is
The NATS Bonus Creator is a single-file HTML tool that generates ready-to-run Python/Playwright scripts to automate segment, promotion, and bonus creation in the Fanatics Casino internal trading platform at `https://trading.1.betfanatics.com/` (Ant Design UI). The HTML file is the source of truth. Generated scripts are output-only and should never be edited directly.

**Current live version:** `nats_bonus_creator_v3_4.html`

> ⚠️ **All scripts should be generated from v3.4.** v3.3/v3.4 added three VIPDM CA tiers (CAS_CA_0092–0094), per-tier Terms start-date overrides (the new tiers' filings begin 09/01/26 while the shared CA date stays 08/17/26), and the `getVIPDMTermsIDs()` default merge — **VIPDM CA scripts touching the new tiers must be generated from v3.4**; v3.2 swapped SUO's mechanic to `BG`; v3.1 added the jurisdiction group + state suffix and restricted LC-CHURN-DM to US/CA (see Naming Convention); v3.0 introduced the new naming convention for every live offer type, the 40-character NATS name guard, positional fail-loud code parsing, and the Promotion Tile lag retry. Every live offer type was PROD-validated on v3.0 (Aug 25, 2026). The long v2.x version-gating rules ("Canadian RTC CC from v2.22+ only", "DMCA from v2.31+ only", etc.) are obsolete for generation purposes — they are preserved in `Technical_Reference_v3_4.md` § Version History for anyone auditing old scripts. Voucher codes, image filenames, and Terms ID localStorage maps are **unchanged from v2.34** (the Terms ID *lookup* gained the v3.3 default-merge; the key shapes are untouched).

---

## Setup
Before using the tool for the first time, follow the Mac Setup Guide at:
**https://adrianvdc.github.io/Promo-Building-Script-Generator/setup-guide**

---

## Script & Testing Output Rules
When producing any Python script — whether a test script or a final generated script — always:
1. Output it as a **downloadable `.py` file**, never pasted inline in the chat
2. Use a new versioned filename on every update (e.g. `v0_1`, `v0_2`) so old cached files in Downloads are never accidentally reused
3. Follow the file immediately with the **terminal command** needed to run it

**Mac format:**
```
cd ~/Downloads && python3 {filename}.py
```

This applies to every script in every conversation in this project, including partial test scripts, debug scripts, and final generated scripts.

---

## Offer Types

**Eight live offer types** (the v3.0 dropdown): **RTC Top Up - Casino Credit (RTC CC)**, **Refer a Friend (RAF)**, **SUO Day 2+ Spins (SUO)**, **Lifecycle - REACT CC Drop (LC-REACT)**, **Lifecycle - Churn DM (LC-CHURN-DM)**, **VIP Offer Library - Deposit Matches (VIPDM)**, **VIP Offer Library - Bet & Gets (VIPBG)**, **Daily Missions - Canada (DMCA)**.

**Four retired offer types** (removed from the dropdown in v3.0; builders remain in the HTML **byte-identical to v2.34** for reference/revival): **RTC Top Up - Free Spins (RTC FS)**, **Bet & Get (BG)**, **Deposit Match (DM)**, **Bet & Get - Rules Engine (BG-RE)**. These were instrumental in building the live offer types and their full documentation is preserved (see "Retired Offer Types" at the end of this document and the Technical Reference). ⚠️ If ever re-exposed, they would generate **v2.x-format codes** with no 40-char guard and no Promotion Tile retry — migrate them to v3.0 first.

DMCA runs both phases in **production** (NATS + Playmaker) and is the only offer type that builds bonuses in Playmaker. Full selector reference and field spec are in `Technical_Reference_v3_4.md`.

---

## v3.0 Naming Convention

All segment / promotion / bonus codes follow the 7-fixed-section skeleton:

```
MMDDYY_JURISDICTION_PRODUCT_LIFECYCLE_SUBCATEGORY_MECHANIC_AWARDTYPE_(FREEFORM)
```

- **7 fixed, non-empty sections + freeform.** Generated scripts parse positionally: `parts[0]` date, `parts[1]` jurisdiction, `parts[2..6]` the five vocabulary sections, `"_".join(parts[7:])` freeform verbatim (the freeform may contain underscores).
- **Hard limit: 40 characters.** The NATS bonus name field truncates at 40 (established by test 08/25/26). Every builder blocks generation with an alert listing any offending code; SUO and RAF (codes assembled at runtime in Python) also carry a runtime assert. **Exact-40 watch list:** VIPDM `M250_10M2500`, VIPBG `TBG…B100K_G2500`, and RAF `DAY10_{2-digit spins}_{ST}` (v3.1 — a 3-digit spin count on a DAY10 RAF code would exceed 40) — any future tier addition on those must re-check the budget. (The v3.3/v3.4 VIPDM CA tiers land at 39/39/38: `M250_20M10K` / `M250_20M20K` / `M100_50M5K`.)
- **K-notation:** freeform values ≥ 1,000 and evenly divisible by 1,000 render as `{n}K` (`100000` → `100K`, `5000` → `5K`); everything else stays literal (`2500` stays `2500`; decimals disallowed). Scripts expand K back to literal values before any field fill, Terms ID lookup, or filename derivation. Applied today to the VIP tiers; every parser accepts K anywhere for forward compatibility.
- **Jurisdiction group + state suffix (v3.1):** position 1 of every code is always the two-character group — `US` (for US, MI, WV, PA, NJ) or `CA` (for CA, ON, AB). When a specific state/province is selected, its token is appended to the **end of the freeform** (e.g. Build WV → `090926_US_CAS_ACQ_SUO_BG_FS_DAY3_50_WV`). Selecting generic US or CA produces no suffix. Parsers treat a trailing state token as optional and strip it before value extraction; `currency_for(parts[1])` therefore only ever sees `US` or `CA`. RAF/SUO scripts keep the real state internally — vouchers (`WVAQ…`), game search (`Hotstepper (WV)`), and spin-stakes currency all still use the state constant.
- **Fail-loud parsing:** scripts raise `ValueError` at first parse on >40 chars, <8 fields, a non-jurisdiction at position 1 (catches v2.x-shaped codes before any NATS write), malformed freeforms, decimal/lowercase K, and (VIPBG CA) a mechanic outside SBG/TBG. `currency_for()` reads position 1 directly and raises on unknowns — the currency row can never be misrouted by an old-shape code or the reserved `CA` (Cash) awardtype token.

### Section Vocabularies (used tokens)

| Section | Used tokens | Reserved (unused) |
|---|---|---|
| PRODUCT | `CAS` Casino · `VCAS` VIP Casino | `VF1` Gold/Platinum/Black · `PVC` Potential VIP Casino · `VPA` Fanatics Fast Track |
| LIFECYCLE | `RET` Retention · `LC` Lifecycle · `ACQ` Acquisition | — |
| SUBCATEGORY | `RTC` RTC Top Up · `BAU` Generic Offer · `SUO` Sign Up Offer · `REACT` Reactivation · `CHURN` Risk of Churn · `OL` Offer Library | `MIG` Migration · `RNF` Reg Not Funded · `XSL` Cross Sell · `EL` Early Life · `OCR` Casino Rewards · `FTP` Free to Play |
| MECHANIC | `OPT` Opt-in Bonus · `BG` Bet and Get · `DM` Deposit Match · `SBG` Slots Bet & Get · `TBG` Tables Bet & Get · `RAF` Refer a Friend | `DR` Drop (v3.2: unused — was SUO in v3.0–v3.1) · `LOSS` Lossback · `SWP` Sweepstakes · `GW` Goodwill · `S2W` Spin to Win Bulk Uploads · `LB` Leaderboard · `FCX` FanCash Multiplier |
| AWARDTYPE | `CC` Casino Credit · `FS` Free Spins · `FC` FanCash | `CA` Cash ⚠️ (visually collides with jurisdiction CA — harmless positionally) · `DO` Dummy Opt-In · `PA` Physical Awards · `FB` Free Bet · `PB` Profit Boost |

> **RTC's LIFECYCLE token is `RET` provisionally** and may change to `LC` in the future — nothing parses it, so a change is a one-constant swap per RTC builder.

### Internal Code Names — Live Offer Types

| Offer | Format | Example |
|---|---|---|
| RTC CC | `MMDDYY_{JUR}_CAS_RET_RTC_OPT_CC_{amount}` | `053026_US_CAS_RET_RTC_OPT_CC_50` |
| LC-REACT | `MMDDYY_{US\|CA}_CAS_LC_REACT_OPT_CC_{amount}[_{ST}]` | `122926_US_CAS_LC_REACT_OPT_CC_10_MI` |
| LC-CHURN-DM | `MMDDYY_{JUR}_CAS_LC_CHURN_DM_CC_M{min}_{pct}M{max}` | `121126_US_CAS_LC_CHURN_DM_CC_M10_50M10` |
| VIPDM (US) | `MMDDYY_US_VCAS_RET_OL_DM_FC_M{min}_{pct}M{max}` | `090126_US_VCAS_RET_OL_DM_FC_M250_20M5K` |
| VIPDM (CA) | `MMDDYY_CA_VCAS_RET_OL_DM_CC_M{min}_{pct}M{max}` | `090126_CA_VCAS_RET_OL_DM_CC_M250_20M10K` |
| VIPBG (US) | `MMDDYY_US_VCAS_RET_OL_SBG_FC_B{bet}_G{get}` | `090126_US_VCAS_RET_OL_SBG_FC_B100K_G5K` |
| VIPBG (CA) | `MMDDYY_CA_VCAS_RET_OL_{SBG\|TBG}_CC_B{bet}_G{get}` | `090126_CA_VCAS_RET_OL_TBG_CC_B100K_G2500` |
| RAF Referee | `MMDDYY_US_CAS_ACQ_SUO_RAF_FS_RFRE_{spins}_{ST}` | `100126_US_CAS_ACQ_SUO_RAF_FS_RFRE_50_MI` |
| RAF Referrer | `MMDDYY_US_CAS_ACQ_SUO_RAF_FS_RFER_{spins}_{ST}` | `100126_US_CAS_ACQ_SUO_RAF_FS_RFER_50_MI` |
| RAF Day 2–N | `MMDDYY_US_CAS_ACQ_SUO_RAF_FS_DAY{n}_{spins}_{ST}` | `100126_US_CAS_ACQ_SUO_RAF_FS_DAY2_50_MI` |
| SUO Day 2–N | `MMDDYY_US_CAS_ACQ_SUO_BG_FS_DAY{n}_{spins}_{ST}` | `100126_US_CAS_ACQ_SUO_BG_FS_DAY2_50_MI` |
| DMCA (CC — Tue/Thu/Sat) | `MMDDYY_CA_CAS_RET_BAU_BG_CC_B{bet}_G{get}` | `120126_CA_CAS_RET_BAU_BG_CC_B125_G5` |
| DMCA (FS — Mon/Wed/Fri/Sun) | `MMDDYY_CA_CAS_RET_BAU_BG_FS_T{n}` | `113026_CA_CAS_RET_BAU_BG_FS_T2` |

Notes:
- **VIPBG:** the SBG/TBG category lives in the MECHANIC slot (`parts[5]`), no longer in the freeform. US always carries `SBG` (all US tiers are slots). RAF's whole campaign carries mechanic `RAF` (Day 2+ payouts included); SUO carries `BG` (v3.2 — distinguished from DMCA/retired-BG codes at LIFECYCLE `ACQ` + SUBCATEGORY `SUO`).
- **DMCA FS tier index:** `T1`–`T4` map to bets $50 / $125 / $500 / $5,000 (20 / 50 / 50 / 100 spins @ $0.10 / $0.10 / $0.50 / $3.00). The code is a **label only** — the actual bet/spins/spin value/game are baked into the generated script's per-code `CODES` config at generation time. An FS bet outside the map blocks generation with an alert (fail-loud on future tier rotation).
- **Terms ID keys are unchanged:** the localStorage maps keep their v2.x literal key shapes (`M{min}_{pct}M{max}`, `{SBG|TBG}_B{bet}_G{get}`, `B{bet}_G{get}`); scripts rebuild them from K-expanded parse results, so saved maps survive the v3.0 upgrade with no reset. (v3.3: `getVIPDMTermsIDs()` additionally merges baked defaults under any saved map — saved values win per key — so newly-added default tiers resolve without a modal re-save.)
- **VIPDM CA per-tier Terms start-date overrides (v3.3):** tiers whose legal filing starts on a different date than the shared CA default carry an entry in `VIPDM_TERMS_START_OVERRIDES` in the HTML (currently `M250_20M10000` / `M250_20M20000` / `M100_50M5000` → 09/01/26). Generated scripts bake this as `TERMS_START_OVERRIDES`, consulted per tier key by `format_hiw`/`format_tc` before the shared date — so mixed-tier runs type the correct Promotional Period per tier. The Terms modal's start-date field does not affect overridden tiers (a warning in the modal names them). US has no overrides.
- **Old collision closed:** DMCA CC and standard-BG shapes now differ at AWARDTYPE (`CC` vs `FC`) — and standard BG is retired — so the v2.x DMCA/BG-CA collision warning is doubly moot.
- **Coexistence:** v2.x and v3.0 codes coexist safely in NATS/Playmaker; nothing re-parses stored codes. All v2.x "avoid reusing these exact codes" PROD-cleanup warnings are moot under v3.0 (format collision is impossible).

### Voucher Codes (UNCHANGED from v2.34)

All voucher codes are exactly 15 characters. A random 3-character suffix (A–Z, 0–9) is generated at runtime to guarantee uniqueness. **Voucher codes in NATS can never be reused**, even if the objects they were attached to have been deleted.

| Offer | Format | Example |
|---|---|---|
| RTC CC | `CRRCCC{MMDDYY}{XXX}` | `CRRCCC052626AB3` |
| LC-REACT | `CRLCCC{MMDDYY}{XXX}` | `CRLCCC122926AB3` |
| LC-CHURN-DM | `CRLCDM{MMDDYY}{XXX}` | `CRLCDM121126X4T` |
| VIPDM (CA) | `VRDMCC{MMDDYY}{XXX}` | `VRDMCC090126X4T` |
| VIPDM (US) | `VRDMFC{MMDDYY}{XXX}` | `VRDMFC090126X4T` |
| VIPBG (US & CA) | `CLBGFC{MMDDYY}{XXX}` | `CLBGFC090126X4T` |
| RAF Referee | `{jurisdiction}AQ{total_spins:04d}RFRE{XXX}` | `MIAQ0500RFREX3K` |
| RAF Referrer | `{jurisdiction}AQ{total_spins:04d}RFER{XXX}` | `MIAQ0500RFERX3K` |
| RAF Day 2–N | `{jurisdiction}AQ{total_spins:04d}DAY{n}{XXX}` | `MIAQ0500DAY2X3K` |
| SUO / DMCA | None (SUO: no vouchers; DMCA: bonus is built in Playmaker) | — |
| Retired: RTC FS / BG / DM / BG-RE | `CRRCFS…` / `CLBGFC…` / `CRDMFC…` / `CLBGFC…` (v2.x, for reference) | — |

VIPBG's voucher is NATS-mandatory but unused for the offer itself; the random suffix keeps it unique (the `CLBGFC` prefix is a historical share with the retired BG/BG-RE).

### Image Filenames (UNCHANGED from v2.34)

All image naming conventions are exactly as v2.34 — generated scripts translate from the new codes (K-expansion, tier tables) back to the existing shapes, so **no creative files need renaming**:
- RTC CC discover: `{amount}.png` (same convention in the US and Canada folders)
- LC-REACT discover: `{amount}.png`
- LC-CHURN-DM discover: `{pct}M{max}.png` (min deposit is not part of the filename)
- VIPDM discover (both regions): `{pct}M{max}.png` — ⚠️ several names differ by one character: CA `10M250.png` vs `100M250.png` and (v3.3) `20M1000.png` vs `20M10000.png`, `20M2000.png` vs `20M20000.png`; US `10M500.png` vs `10M5000.png` and `20M500.png` vs `20M5000.png` — double-check those creatives. The v3.3/v3.4 CA tiers add `20M10000.png`, `20M20000.png`, and `50M5000.png` to the CA folder.
- VIPBG (CA): category creatives `Slots|Tables Promo Detail.png` / `Slots|Tables Masthead.png` per the SBG/TBG mechanic; discover `{SBG|TBG}_B{bet}_G{get}.png` (literal expanded values, e.g. `SBG_B1000_G100.png`)
- VIPBG (US): generic `Promo Detail.png` / `Masthead.png`; discover `B{bet}_G{get}.png`
- RAF: `Promo Detail.png`, `Masthead.png`, `Discover_RFRE.png`, `Discover_RFER.png`, `Link Preview.png`
- SUO: no images
- DMCA: weekday-prefixed underscored — `{Weekday}_Promo_Detail.png`, `{Weekday}_Masthead.png`, CC discover `{Weekday}_B{bet}_G{get}.png`, FS discover `{Weekday}_B{bet}_G{spins}S_{value}V.png` (spin value in cents; no game token — the weekday implies the game). The generated script pre-flight checks every required file before login.

The expected image filenames per offer type are **called out in the dynamic info box above each set of day cards** (v3.0 UI requirement).

### Generated Script Filenames (unchanged)

RTC CC `rtc_top_up_MMDDYY_HHMM.py` · RAF `raf_full_campaign_…` · SUO `suo_day2_…` · LC-REACT `lc_react_…` · LC-CHURN-DM `lc_churn_dm_…` · VIPDM `vcl_dm_…` (both regions) · VIPBG `vcl_bg_…` (both regions) · DMCA `daily_missions_…`. Retired: `rtc_fs_…` / `bg_…` / `dm_…` / `bg_re_…`.

### Regions / Jurisdictions

- RTC CC: **US (default), CA only**
- VIPDM: **US / CA** (always defaults to US when the offer type is selected)
- VIPBG: **US / CA** (defaults to CA; keeps a valid current selection)
- DMCA: **CA (Ontario) only — fixed** (region row hidden)
- LC-REACT: US (default), MI, WV, PA, NJ, ON, AB, CA (state/province picks append the v3.1 freeform suffix)
- LC-CHURN-DM: **US (default) / CA only** (v3.1 — states dropped; state codes + the suffix would exceed the 40-char limit)
- RAF / SUO: MI, NJ, WV, PA only (per-card select; codes always carry `US` at position 1 with the state as the freeform suffix)
- Retired BG / DM / BG-RE: US (default), MI, WV, PA, NJ, ON, AB, CA · Retired RTC FS: MI, NJ, WV, PA

---

## Multi-Currency Amount Tables

NATS renders bonus amount tables with one row per currency (`USD` / `GBP` / `CAD`) keyed by `data-row-key`. All generated scripts include two shared helpers:

- **`currency_for()`** — **v3.0: positional.** For code-parsing builders, reads `parts[1]` (US/MI/WV/PA/NJ → **USD**; ON/AB/CA → **CAD**) and raises on anything else. In RAF and SUO, takes the bare `JURISDICTION` constant directly (those scripts never parse codes), with the same validation. **GBP is never used and is unreachable by construction.**
- **`fill_currency_amount()`** — fills the jurisdiction's currency row only (`tr[data-row-key='{currency}']`), with a legacy single-input fallback. Terminal output prints the row used.

Fields filled via the currency row: RTC CC amount, LC-REACT amount, LC-CHURN-DM amount + Min/Max Deposit, VIPDM reward + Min/Max Deposit (CAD for CA Casino Credit, USD for US FanCash), VIPBG flat reward (CAD/USD by region; no deposit fields), and the RAF/SUO Free Spin Stakes spin-value row. Percentage inputs are not part of the currency table. DMCA is unaffected (Playmaker bonus; its stakes grid shows CAD for Canada natively).

---

## Promotion Tile Retry (v3.0)

NATS's promo search index can lag a just-created promo (root-caused Aug 25, 2026 — a Promotion Tile lookup timed out on a promo built two minutes earlier in the same run; an unchanged re-run succeeded). Every live builder that selects a Promotion Tile (RTC CC, LC-REACT, LC-CHURN-DM, VIPDM, VIPBG, RAF RFRE + RFER) wraps the selection in a **6-attempt loop (~70 seconds)**: each attempt re-opens the dropdown, re-types the code name, and waits 10s for the anchored exact match, printing `WARN Promotion Tile not found yet (attempt N/6)` between attempts and raising a hard error after attempt 6. SUO and DMCA have no Promotion Tile. A read-only diagnostic (`debug_promo_tile_v0_1.py` — repr-level dropdown dumps across six search shapes) exists for future promo-search investigation.

---

## Live Offer Type Summaries

Full field specs, selectors, T&C templates, and window math are in `Technical_Reference_v3_4.md`. What follows are the operational essentials per offer type — all behavior below is **unchanged from v2.34** except code names, the retry, and the v3.3/v3.4 VIPDM CA additions.

### RTC Top Up - Casino Credit
9 default day cards; default amounts 2–800. Dual-region US/CA (parallel `_ca` localStorage keys; region switch silently resets cards; red edit buttons on CA). **4 AM ET start, 24-hour window** — unique among all offer types. Terms IDs: US `CAS_9461`–`9471` (dates 06/01/26 → 12/01/26), CA `CAS_CA_0001`–`0011` (08/17/26 → 12/31/26), edited per region via the Terms modal. Bonus: Opt In, Stake Chunk 1, Origin Retention, Series RTC, Type Lifecycle, no confirmation modal. HIW shared across regions; T&C region-scoped.

### Lifecycle - REACT CC Drop
3 default days; default amounts 10/25/50. **72-hour window** (day-of 00:00 → day+2 23:59:59 ET). Per-amount baked T&C: $10 `CAS_9237`, $25 `CAS_9238` (Apr 16 → Oct 16, 2026), $50 `CAS_9456` (Jun 1 → Dec 1, 2026) — changes require regeneration. Bonus: Opt In, Stake Chunk 1, Origin Opt-In Bonus, Series Reactivation, Type Lifecycle, no modal.

### Lifecycle - Churn DM
4 default days; five default offers ($10/50%/$10 → $50/50%/$200), Terms IDs `CAS_10620` / `CAS_9457`–`9460` in the baked `CHURNDM_TERMS_IDS` map — active offers without a configured ID block generation. **10-day window** (day-of 00:00 → day+10 00:00 ET). Fixed copy `Deposit and Get Casino Credit`; no Button 2. Bonus: Casino Credit matched-deposit form (`Casino Credit (%)` checkbox), Entitlement Deposit, **Matched Deposit always checked**, Max Deposit 1000000, Origin Retention, Series Other, Type Lifecycle, no modal.

### VIP Offer Library - Deposit Matches (VIPDM)
Single card; **always defaults to US on offer-type selection**. US: FanCash/USD, ten tiers `CAS_10677`–`10686`, dates 07/01/26 → 12/31/26, `Bonus (%)` checkbox, `Days to Meet Fancash Settlement`, COMBO: Casino kept, FanCash confirmation modal expected (handled defensively). CA: Casino Credit/CAD, **twelve tiers** (v3.4) — `CAS_CA_0078`–`0086` (dates 08/17/26 → 12/31/26) plus `CAS_CA_0092`–`0094` (20%/$10K, 20%/$20K, 50%/$5K — **filed 09/01/26 → 12/31/26**, carried as baked per-tier start-date overrides via `VIPDM_TERMS_START_OVERRIDES`; the Terms modal's date field does not affect them) — `Casino Credit (%)`, all COMBO tags removed, no modal. Both: **month-anchored window** (typed date − 1 day 00:00 ET → month-end 23:59:59 ET + Days to Entitlement), `From Your VIP Host` copy, Button 2 `Deposit` → `/auth/account/deposit`, Matched Deposit always checked, Series `VIP` / Type `Retention` (visible-dropdown scoped — load-bearing), Origin Opt-In Bonus. All modals region-aware; region switch silently resets. ⚠️ CAS_10679–10685 filings unsupplied — verify the canonical template on first use of those tiers. (The CAS_CA_0092–0094 filings were supplied and byte-verified.)

### VIP Offer Library - Bet & Gets (VIPBG)
Single card; defaults to CA. **The wager requirement is enforced upstream (XP/segment) — the bonus is a flat reward equal to the "get".** CA: Casino Credit/CAD, Slots/Tables categories (now the code's MECHANIC), eight tiers `CAS_CA_0069`–`0076`, **category-specific Masthead/Promo Detail creatives**, Button 2 empty, all COMBO tags removed. US: FanCash/USD, all-slots (mechanic `SBG`), four tiers `CAS_10687`–`10690`, generic creatives, Button 2 `Play Now!` → `/docs/usered/casgenericgamelist`, COMBO: Casino kept, modal expected. Both: month-anchored window, `From Your VIP Host`, **Send to XP + External Bonus always ON** (no toggles), Stake Chunk conditional fill, Series `VIP` / Type `Retention`. ⚠️ CAS_10689/10690 filings unsupplied — verify on first use.

### Refer a Friend (RAF)
Single campaign card; three phase badges (**Day 2+ / Referee / Referrer**). Referee: promo + bonus + post-save edit. Referrer: segment + promo + bonus + post-save edit. Day 2+: segments + bonuses. Fixed: Opt In, Play! CTA, activation end 2041, Origin Bet Settlement Bonus, Series Refer a Friend, Type Acquisition. Voucher modal on RFRE/RFER saves. Known Bugs #6/#7 still apply (NATS clears fields on re-open → the scripted edit passes; 120-second Bonus Manager wait during the RFER edit). Games/stakes via the shared `FS_GAMES` map with the v2.33 `searchOverride` mechanics (see Casino Game Selection below).

### SUO Day 2+ Spins
Single card, live DAY2–N preview, 5/10-day toggle. No promos, no vouchers, no step badges (segment + bonus always built). Fixed: No Opt In, Play! CTA, activation end 2041, Origin Bet Settlement Bonus, Series Early Life, Type Acquisition. Same `FS_GAMES` machinery as RAF.

### Daily Missions - Canada (DMCA)
Two-phase, both production: NATS (segments + promos) then Playmaker (bonuses, Save Bonus clicked). The typed date's weekday drives everything (Tue/Thu/Sat = CC `CAS_CA_0019`–`0022`; Mon/Wed/Fri/Sun = FS with per-day games BAA/4CC/MFB/BWZ and Terms IDs `CAS_CA_0043`–`0058`). **NATS promo: single-day window** (00:00:00 → 23:59:59 ET); **Playmaker bonus 00:01 → 00:01 next day**. Segment built in NATS, attached in Playmaker via the triple-guarded Include → Existing Account Segments field with chip read-back and ~60s sync poll (fails closed — never the Create field, never Exclude). Spin values validated against per-game stakes ladders at generation time **and** runtime ("refusing to substitute"). Toast always ON, title `Mission Complete!`. Okta Verify persistent-profile launch (`~/.dm_canada_pw_profile`). No edit modals — HIW/T&C/Terms/dates baked (changes via Adrian). ⚠️ CAS_CA_0021/0022 (CC $500/$5,000) remain unsupplied — verify on first use. FS tiers rotating requires updating `DMCA_GAMES`, FS Terms IDs, creatives, dates, **and the v3.0 `T1`–`T4` bet→index map** in `generateScript` (reach out to Adrian).

---

## localStorage Keys (UNCHANGED from v2.34)

The full key table is identical to v2.34 — `rtc_*` (+ `_ca`), `rtc_fs_*`, `bg_*`, `dm_*`, `raf_image_path`, `lc_react_*`, `lc_churn_dm_*`, `vcl_dm_*` (+ `_us`), `vcl_bg_*` (+ `_us`), `daily_missions_image_path`. v3.0–v3.4 added no keys and changed no key shapes (Terms ID map keys keep their literal v2.x forms). v3.3 changed one *read* path: `getVIPDMTermsIDs()` merges baked defaults under any saved `vcl_dm_terms_ids` map (saved values win per key), so newly-added default tiers resolve their IDs without a modal re-save. Retired offer types' keys remain functional but unused while retired. Clear Saved Settings removes everything, including every `_ca` and `_us` variant. SUO, BG-RE, and DMCA (image path aside) have no localStorage keys.

---

## Image System (UNCHANGED from v2.34)

All paths, Drive folders, and filename conventions are identical to v2.34 — see that section of `Technical_Reference_v3_4.md` for the full folder list. The operational rule stands:

> ⚠️ Each user must set their own image path via **Edit Images** before generating a script — separately for US and CA where applicable (RTC CC, VIPDM, VIPBG) and once for DMCA. Paths save to the browser's localStorage, once per machine per region.

BG/DM/BG-RE ZIP-attachment image handling remains in the retired builders untouched.

---

## Sidebar Navigation (unchanged)

All generated scripts navigate via `data-menu-id` hover + `dispatch_event("click")` with 5 retries and safe-zone resets: Account Segments (`[data-menu-id$='-segments']`), Promos (`-cms`), Bonus Manager (`-bonuses`). DMCA's NATS phase uses Segments + Promos only.

---

## Per-Code Step Toggles (unchanged)

- **RTC CC / LC-REACT / LC-CHURN-DM / VIPDM / VIPBG / DMCA:** Segment / Promo / Bonus badges per card → `DO_SEGMENT` / `DO_PROMO` / `DO_BONUS` arrays (DMCA carries them per `CODES` entry). Phases skip when no code has the step enabled.
- **RAF:** Day 2+ / Referee / Referrer phase badges.
- **SUO:** no badges — segment + bonus always built.

---

## Eastern Time Enforcement (unchanged)

All generated scripts enforce America/New_York via `zoneinfo`. **Python 3.9+ required.**

---

## Platform Tags — Bonus Creation (unchanged)

Standard: remove `COMBO: Sportsbook` + `COMBO: Sportsbook And Casino`, keep `STAC: Standalone Casino` + `Web` + `COMBO: Casino`. **Exception:** CA VIPDM and CA VIPBG remove all three COMBO tags (Web + STAC only); US VIPDM/VIPBG follow the standard pattern. DMCA bonuses are Playmaker-side — tags don't apply.

---

## Script Execution Notes (unchanged except the retry)

- **Create Promotion button poll:** 60s (120 × 500ms). **Create Bonus button poll:** 60s.
- **Promotion Tile selection:** anchored exact-match regex scoped to the visible dropdown, now inside the v3.0 6-attempt retry (see above). SUO and DMCA don't use a Promotion Tile.
- **Segment dropdown selection** (LC-REACT / LC-CHURN-DM / VIPDM / VIPBG): same anchored visible-dropdown pattern. DMCA: NATS-built segment attached via Playmaker's Include → Existing field.
- **Series/Type (VIPDM/VIPBG):** `^VIP$` / `^Retention$` scoped to the visible dropdown — the Type scoping is load-bearing (hidden Bonus Origin dropdown also contains `Retention`).
- **AMELCO segment dropdown:** `dispatch_event("click")` + 2s wait before OK, all segment-building offer types incl. DMCA.
- **Matched Deposit:** always checked for VIPDM (both regions) and LC-CHURN-DM, with the explicit visibility wait.
- **Confirmation modals:** none for RTC CC / LC-REACT / LC-CHURN-DM / VIPDM CA / SUO / DMCA; expected + defensively handled for VIPDM US / VIPBG US; defensively handled either way for VIPBG CA; voucher modal → Apply for RAF RFRE/RFER.
- **Image upload timing:** 8s before OK, 2s after; ~30s per 3-image promo.
- **Casino Game Selection (RAF / SUO / retired RTC FS):** unchanged v2.33 machinery — games with a `searchOverride` type `{override} ({jurisdiction})` (7FH `Hotstepper`, 7H2 `Hotstepper 2`); others type the pre-™ Search Name; the anchored ™-tolerant jurisdiction-required regex owns correctness. NATS's game search is hard-capped at ~12 rendered results. ⚠️ 7P5 remains exposed to the cap (candidate override `Express ({JUR})` untested); WWE/TCE believed narrow enough but not dump-verified; new games need their exact catalog spelling captured first (curly-apostrophe families exist).

---

## Known Bugs (carried forward)

| # | Area | Severity | Issue |
|---|---|---|---|
| 2 | *(retired DM)* | ⚠️ | `format_hiw` / `format_tc` baked at generation time |
| 5 | *(retired RTC FS)* | ⚠️ | HIW/T&C placeholder copy — pending legal |
| 6 | RAF | ⚠️ | NATS clears Referee/Referrer radio + Reporting Platform on bonus re-open — post-save edit pass required |
| 7 | RAF | ⚠️ | Bonus Manager requires 120-second wait during RFER edit pass |

## Known Limitations (v3.4)

| # | Area | Note |
|---|---|---|
| 1 | All live builders | v3.0 codes parse positionally and **fail loudly** on any structural violation before any NATS write (see Naming Convention). Retired builders keep v2.x parsing. |
| 2 | VIPDM / VIPBG | Exact-40 codes exist (`M250_10M2500`, `TBG…B100K_G2500`); the v3.3/v3.4 CA tiers land at 39/39/38 — future tier additions must re-check the budget; the generation-time guard blocks silent overflow. HIW/T&C/Terms IDs/dates remain baked at generation time; new tiers need Terms IDs (region-scoped) or generation blocks. **v3.3:** CA VIPDM tiers filed with a start date differing from the shared region default need an entry in `VIPDM_TERMS_START_OVERRIDES` in the HTML (currently `M250_20M10000` / `M250_20M20000` / `M100_50M5000` → 09/01/26); the Terms modal's start-date field does not affect overridden tiers. |
| 3 | DMCA | HIW/T&C/Terms/dates/ladders/games baked with no edit modals; FS tier rotation requires updating `DMCA_GAMES` + FS Terms IDs + creatives + dates **+ the `T1`–`T4` map** (Adrian). NATS→Playmaker segment sync dependency and the fail-closed Include/Existing lookup unchanged (a Playmaker copy change to the anchor strings fails closed — send the dump to Adrian). Okta persistent-profile launch remains DMCA-only (back-port candidate). CAS_CA_0021/0022 unsupplied. |
| 4 | LC-REACT / LC-CHURN-DM | Per-tier T&C data baked at generation time; new Churn sizes need `CHURNDM_TERMS_IDS` entries in the HTML. |
| 5 | Template literals | Unchanged and still critical: generated Python lives inside JS template literals which swallow single backslashes; **any change must be verified by rendering through a real JS engine and executing the resulting Python** (every v3.x increment through v3.4 followed this — see the changelogs; note the nested-literal variant inside `${cond ? \`…\` : \`…\`}` blocks has different backslash arithmetic). |
| 6 | Region resets | Switching Region silently resets all day cards (RTC CC, VIPDM, VIPBG) — by design, no confirmation. |
| 7 | Retired builders | RTC FS / BG / DM / BG-RE generate v2.x codes with no 40-char guard, no retry, and token-hunting `currency_for` — **migrate before any re-exposure**. BG-RE additionally still points at test-environment URLs. |
| 8 | RTC lifecycle token | `RET` is provisional; a change to `LC` is a one-constant swap per RTC builder. |
| 9 | PROD housekeeping | Verify the Aug 25 Churn re-run didn't leave a duplicate `120126_US_CAS_LC_CHURN_DM_CC_M10_50M10` segment/promo pair. The v2.x-era PROD leftovers (DMCA validation builds, SUO `082426`/`090126` segments, etc.) can be deleted at leisure — v3.0 codes cannot collide with them. |
| 10 | VIPDM CA creatives | The v3.3/v3.4 tiers require `20M10000.png`, `20M20000.png`, and `50M5000.png` in the CA Drive folder before first use — note the new one-character near-collisions with `20M1000.png` / `20M2000.png`. First PROD run of each new tier should be watched end to end (standard practice for tier additions). |

---

## How It Works & T&C Copy

All HIW and T&C templates, dynamic fields, title lines, and byte-verification notes are **unchanged from v2.34** — see `Technical_Reference_v3_4.md` for the full copy documentation per offer type. Summary of what's baked vs editable:

- **Editable via modals (region-aware where dual):** RTC CC HIW (shared) + T&C (per region); LC-REACT HIW/T&C; LC-CHURN-DM HIW/T&C; VIPDM HIW/T&C/Terms/dates (per region — CA start date does not apply to the v3.3/v3.4 override tiers); VIPBG HIW/T&C/Terms/dates (per region).
- **Baked, no modals:** DMCA (all copy; changes via Adrian). RAF and SUO have placeholder/no copy respectively. VIPDM CA per-tier start-date overrides (`VIPDM_TERMS_START_OVERRIDES`) are baked in the HTML — new override tiers require an HTML change (reach out to Adrian).
- **Retired:** BG/DM modals remain functional but unused; RTC FS placeholder copy pending legal (moot while retired).

---

## Retired Offer Types — Preserved Reference

The complete v2.34 documentation for **RTC Top Up - Free Spins, Bet & Get, Deposit Match, and Bet & Get - Rules Engine** — day-card layouts, windows (RTC FS day-of midnight → +30 days; BG/DM per their specs), game tables, ZIP image handling, BG-RE's two-phase test-environment architecture and Playmaker Create-field segment flow, voucher formats, and Script Steps — is preserved verbatim in `Technical_Reference_v3_4.md` (their sections are marked retired). Key facts worth remembering:

- **BG-RE** remains on **test-environment URLs** and its Playmaker "Create Account Segments" chip flow is the pre-v2.31 pattern (correct for BG-RE only). Its Free Spins reward type was never implemented; DMCA's FS flow is the reference if ever needed.
- **RTC FS** carries the `FS_GAMES` map that RAF and SUO still actively use — the game table, `searchOverride` entries, stakes ladders, and the 7P5/WWE/TCE search-cap caveats remain live concerns for RAF/SUO even with RTC FS retired.
- **BG/DM** established the ZIP-embedded image pattern, the FanCash confirmation-modal behavior (inherited by US VIPDM/VIPBG expectations), and the campaign-name conventions.
- All four builders are **byte-identical to v2.34** in the shipped HTML (verified against the pristine file) — revival = re-add the `<option>` + run a v3.0-style migration increment.

---

## Related Documents

- `nats_bonus_creator_v3_4.html` — the tool (source of truth)
- `Technical_Reference_v3_4.md` — full selector reference, field specs, per-offer Script Steps, version history v2.17 → v3.4
- `Changelog_v3_4.md` — the release record (v3.3 → v3.4, stacked on the v3.3 entry; earlier releases in `Changelog_v3_2.md`)
- `v3_0_naming_convention_spec_v0_3.md` — the design-of-record for the naming convention (decision log included, updated through v3.2)
- `v3_shared_helpers_v0_1.py` — offline test suite for the v3.0 parse helpers (100 checks; runnable anytime)
- `debug_promo_tile_v0_1.py` — read-only NATS promo-search diagnostic
