# NATS Bonus Creator — Project Instructions

## What This Is
The NATS Bonus Creator is a single-file HTML tool that generates ready-to-run Python/Playwright scripts to automate segment, promotion, and bonus creation in the Fanatics Casino internal trading platform at `https://trading.1.betfanatics.com/` (Ant Design UI). The HTML file is the source of truth. Generated scripts are output-only and should never be edited directly.

**Current live version:** `nats_bonus_creator_v2_24.html`

> ⚠️ **v2.20 must not be used** — it contains a broken Casino Game regex (template escaping bug, hotfixed in v2.21). Any scripts generated from v2.20 will fail at Casino Game selection. Scripts generated from v2.17 or earlier will fail (BG, RTC CC, RTC FS, RAF, SUO, LC-REACT) or silently misfill Canadian amounts (DM, LC-CHURN-DM) due to the August 2026 NATS multi-currency and game-naming updates. **Canadian RTC CC scripts must be generated from v2.22+ only** — v2.21 and earlier allowed selecting ON/AB/CA on RTC CC but would bake in US images, US T&Cs, and US Terms IDs. **VIP Offer Library - Deposit Matches scripts must be generated from v2.23+ only** — pre-release v2.23 builds lack the Matched Deposit step and leave `COMBO: Casino` on the bonus. **VIP Offer Library - Bet & Gets scripts must be generated from the published v2.24 only** — pre-release v2.24 builds circulated during development fill Button 2 (`Play Now!` → `/docs/usered/casgenericgamelist`) on CA promos, which is reserved for the future US region and must be empty for Canada. Regenerate all pending scripts from v2.24.

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
Eleven offer types are supported: **RTC Top Up - Casino Credit**, **RTC Top Up - Free Spins**, **Bet & Get (BG)**, **Deposit Match (DM)**, **Refer a Friend (RAF)**, **SUO Day 2+ Spins**, **Bet & Get - Rules Engine (BG-RE)**, **Lifecycle - REACT CC Drop (LC-REACT)**, **Lifecycle - Churn DM (LC-CHURN-DM)**, **VIP Offer Library - Deposit Matches (VIPDM)**, **VIP Offer Library - Bet & Gets (VIPBG)**.

> ⚠️ **Bet & Get - Rules Engine is fully integrated as of v2.15 but currently uses test environment URLs.** NATS and Playmaker links must be switched to production once the Rewards Engine is live in the Playmaker production environment. Full selector reference and field spec are in `Technical_Reference_v2_24.md` attached to this project.

---

## Multi-Currency Amount Tables (v2.18+)

An August 2026 NATS update converted bonus amount tables from a single input into a **three-row table with one row per currency** (`USD` / `GBP` / `CAD`), keyed by `data-row-key` on each `<tr>`. All generated scripts include two shared helpers:

- **`currency_for(name)`** — derives the currency from the code name's jurisdiction token: US / MI / WV / PA / NJ → **USD**; ON / AB / CA → **CAD**. **GBP is never used under any circumstance** and is unreachable by construction.
- **`fill_currency_amount()`** — fills the value into the jurisdiction's currency row only (`tr[data-row-key='{currency}']`), with a fallback to the legacy single-input selector if NATS reverts. Terminal output shows which row was filled, e.g. `OK Bonus Amount (USD): 50`.

**Fields filled via the currency row:** RTC CC amount, LC-REACT amount, BG FanCash amount, DM Bonus amount + Min/Max Deposit, LC-CHURN-DM amount + Min/Max Deposit, VIPDM amount + Min/Max Deposit (always CAD — CA-only), VIPBG flat Casino Credit amount (always CAD — CA-only; no deposit fields), and the Free Spin Stakes spin value for RTC FS / RAF / SUO (a select dropdown, chosen by currency label). Percentage inputs (DM, LC-CHURN-DM, VIPDM) are not part of the currency table.

---

## RTC Top Up - Casino Credit — Canada Support (v2.22)

RTC CC is dual-jurisdiction: **US — United States** (default) and **CA — Canada**. The tool works exactly as before for US; CA layers its own settings on top via parallel `_ca` localStorage keys resolved through the `isRTCCanada()` / `rtcKey()` helpers.

**What differs when CA — Canada is selected:**
- **Region dropdown** on RTC CC shows only US and CA (all other offer types keep their existing region lists; VIPDM and VIPBG are locked to CA).
- **Images** pull from the Canadian creative folder (default path `.../Lifecycle Automations/RTC Top Up - Canada`, Drive folder `https://drive.google.com/drive/folders/1W6ulHKv3ted9ZLNKzPgEbRoNCK_wFz6O`) — same filename convention as US.
- **T&Cs** use the baked-in Canadian (Ontario) base copy — FBG Enterprises Canada, Inc., 19+ / Ontario eligibility, ConnexOntario responsible gaming language, title line `FANATICS CASINO (CANADA) – CASINO CREDIT {amount_fmt} CAD SURPRISE DROP ({terms_id})` — with the same six dynamic fields as US.
- **Terms IDs** default to `$2→CAS_CA_0001` through `$800→CAS_CA_0011`, with CA promotional dates **08/17/26 → 12/31/26** (stored separately from the US 06/01/26 → 12/01/26 window).
- **HIW is shared** between US and CA (same copy, same `{amount_fmt}` field) by design.
- **Edit T&Cs / Edit Terms IDs / Edit Images modals** read and save the selected region's values and show the region in the modal title.
- **Visual cue:** the three edit buttons render in red shades (instead of blue) whenever CA is selected.
- **Region change resets day cards:** switching Region in either direction clears all dates, amount selections, and toggles, and hides any generated script — silent, no confirmation dialog. US and CA inputs can never mix.
- **Amount fill:** CA code names carry the `_CA_` token, so generated scripts fill the CAD currency row (existing v2.18 behavior).

The generated Python template is identical for both regions — jurisdiction differences are injected at generation time (image folder, T&C text, Terms IDs, dates) and at runtime (currency row).

---

## VIP Offer Library - Deposit Matches (v2.23)

Tenth offer type. Monthly VIP-library deposit-match offers paid in **Casino Credit**. **CA — Canada only** — the Region dropdown is locked to CA (a US region is planned, RTC CC-style, and `currency_for()` already routes US codes to the USD row when it lands).

### Day Card Layout
- **Single card** (full width), titled `VIP Offer Library — CA — Canada`. **+ Add Day is a no-op.**
- Offer rows are **Min → % → Max** with a per-offer checkbox. Nine defaults shown (unselected): $100/50%/$250, $100/100%/$250, $250/10%/$250, $250/10%/$2,500, $250/10%/$5,000, $250/20%/$500, $250/20%/$1,000, $250/20%/$2,000, $250/20%/$5,000. Each row displays its Terms ID inline (red `no ID` when unmapped).
- No Campaign Name field, no ZIP attachment. Send to XP default ON; External Bonus present, default OFF. Days to Opt In 3 / Entitlement 3 / Settlement 7.

### Promo & Bonus Activation Window (month-anchored)
- **Start:** 00:00:00 ET **one day before the typed date**
- **End:** **last day of the typed date's calendar month** at 23:59:59 ET **+ Days to Meet Entitlement**

The typed day anchors only the start — `080126` and `081726` produce the same end date. A typed 1st starts in the previous month (`010127` starts Dec 31, 2026 — intended). Changing Days to Entitlement changes the promo **and** bonus end date. `calendar.monthrange` handles month lengths and leap years. Promo window and bonus activation window are identical.

### Terms ID Enforcement
Each active offer key `M{min}_{pct}M{max}` must exist in the VIPDM Terms ID map. Non-matching active offers show a red warning on the row and **block script generation** with an alert. Defaults:

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

Terms IDs and promotional dates (default **08/17/26 → 12/31/26**) are edited via the **Edit Terms Expiry & Terms IDs** button inside the Edit T&Cs modal — dates render as `{start_date_long}` / `{end_date_long}` in §2 of the T&C.

### Fixed Copy
Title, Promo Header Text, and Bonus Description are all `From Your VIP Host` regardless of offer size. **Button 2 label `Deposit` → link `/auth/account/deposit`** on the promo (unlike Churn DM, which has no Button 2, and VIPBG CA, where Button 2 is left empty).

### T&C Data (Baked In)
Canadian (Ontario) legal base: FBG Enterprises Canada, Inc., 19+ / Ontario eligibility, ConnexOntario responsible gaming language, CAD-denominated examples. Dynamic fields: `{min_fmt}`, `{pct}`, `{max_fmt}`, `{terms_id}`, `{start_date_long}`, `{end_date_long}`. Title line: `FANATICS CASINO (CANADA) – DEPOSIT {min_fmt} CAD, GET {pct}% CASINO CREDIT (UP TO {max_fmt} CAD) ({terms_id})`. §1's "within 3 days of being presented this offer" and §4's "during that Promo Period" are **static legal copy by design** — not templated.

### Bonus Creation — Fixed Values

| Field | Value |
|---|---|
| Trigger type | Opt In |
| Bonus type | Casino Credit (percentage checkbox labeled `Casino Credit (%)`) |
| Description | From Your VIP Host |
| Entitlement Type | Deposit |
| **Matched Deposit** | **Always checked** (inside the DEPOSIT entitlement panel) |
| **Platforms** | **Web + STAC: Standalone Casino only** — all three COMBO tags removed |
| Minimum Deposit | Dynamic from `M{min}` in the code — CAD currency row |
| Maximum Deposit | 1000000 — CAD currency row |
| Days to Opt In / Entitlement | 3 (default, editable; Entitlement also extends the end dates) |
| Days to Meet Casino_credit Settlement | 7 (default, editable) |
| Status Active | Checked |
| Activation | Typed date − 1 day 00:00 ET → month-end 23:59:59 ET + entitlement days |
| Reporting Platform | CAS (Casino) |
| Bonus Origin | Opt-In Bonus |
| Series | VIP |
| Type | Retention |
| Confirmation modal | No |

> ⚠️ Series `VIP` and Type `Retention` are selected via anchored regex scoped to the visible dropdown (`.ant-select-dropdown:not(.ant-select-dropdown-hidden)`). The Type scoping is load-bearing: the hidden Bonus Origin dropdown also contains a `Retention` option, so a page-wide exact-text click would crash with a Playwright strict mode violation. VIPDM and VIPBG are the only offer types that select `Retention` for Type.

Note: the `RET` token in the code name mirrors the **Type** field (Retention), not Bonus Origin — unlike standard DM.

---

## VIP Offer Library - Bet & Gets (v2.24)

Eleventh offer type. Monthly VIP-library wager-and-get offers paid in **Casino Credit**. **CA — Canada only** — the Region dropdown is locked to CA (a US region is planned, RTC CC-style, and `currency_for()` already routes US codes to the USD row when it lands).

> ⚠️ **The wager requirement ("bet") is enforced upstream by XP / segment logic and is never entered anywhere on the bonus.** The bonus is a flat Casino Credit amount equal to the "get" value.

### Day Card Layout
- **Single card** (full width), titled `VIP Offer Library — Bet & Gets — CA — Canada`. **+ Add Day is a no-op.**
- Offer rows are **Category (Slots/Tables) → Bet → Get** with a per-row category dropdown and per-offer checkbox. Eight defaults shown (unselected): Slots $1,000/$100, $5,000/$500, $20,000/$1,000, $100,000/$5,000; Tables $2,000/$100, $10,000/$500, $25,000/$1,000, $100,000/$2,500. Each row displays its Terms ID inline (red `no ID` when unmapped).
- No Campaign Name field, no ZIP attachment. **Send to XP and External Bonus have no toggles — both are always ON** (static note on the card; hardcoded `[True] * len(NAMES)` in the generated script). Days to Opt In 3 / Entitlement 3 / Settlement 7.

### Promo & Bonus Activation Window (month-anchored — identical to VIPDM)
- **Start:** 00:00:00 ET **one day before the typed date**
- **End:** **last day of the typed date's calendar month** at 23:59:59 ET **+ Days to Meet Entitlement**

Same rules as VIPDM: the typed day anchors only the start, a typed 1st starts in the previous month, changing Days to Entitlement changes both end dates, promo window = bonus activation window. Note the T&C's seventy-two (72) hour wagering-window language is legal copy about the offer mechanics (the customer's window after receiving the offer) and is **independent of the month-anchored activation window**.

### Terms ID Enforcement
Each active offer key `{SBG|TBG}_B{bet}_G{get}` must exist in the VIPBG Terms ID map (the category prefix is included as collision insurance — a Slots and Tables tier could otherwise share a `B{bet}_G{get}` shape). Non-matching active offers show a red warning on the row and **block script generation** with an alert. Defaults:

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

Terms IDs and promotional dates (default **08/17/26 → 12/31/26**) are edited via the **Edit Terms Expiry & Terms IDs** button inside the Edit T&Cs modal — dates render as `{start_date_long}` / `{end_date_long}` in §2 of the T&C.

### Fixed Copy
Title, Promo Header Text, and Bonus Description are all `From Your VIP Host` regardless of offer size. **Button 2 label and link are intentionally left EMPTY for CA — Canada.** The values `Play Now!` → `/docs/usered/casgenericgamelist` are documented in the generated script header and reserved for the future US region of this offer type.

### T&C Data (Baked In)
**Single Canadian (Ontario) legal template covering both categories** — the Slots and Tables legal documents are word-for-word identical apart from dynamic values, so one template reproduces all 8 documents exactly (verified byte-for-byte against both legal docs). Dynamic fields: `{bet_fmt}`, `{get_fmt}`, `{terms_id}`, `{game_category_lower}` (`slots` / `table` — matching legal's asymmetric "slots games" / "table games"), `{game_category_upper}` (`SLOTS` / `TABLES`), `{start_date_long}`, `{end_date_long}`. Title line: `FANATICS CASINO (CANADA) – {game_category_upper} BET {bet_fmt} CAD, GET {get_fmt} CAD IN CASINO CREDIT ({terms_id})`. **Static by design:** the seventy-two (72) hour wagering-window language throughout, "within 72 hours of satisfying the wagering requirement," the seven (7) day Casino Credit expiry, and all §5 CAD examples — not templated. Curly quotes and en-dash from the legal documents are preserved.

### Bonus Creation — Fixed Values

| Field | Value |
|---|---|
| Trigger type | Opt In |
| Bonus type | Casino Credit (flat amount — no percentage checkbox, no deposit entitlement, no Matched Deposit) |
| Description | From Your VIP Host |
| **Send To XP / External Bonus** | **Both always clicked ON** |
| **Bet amount** | **Not entered anywhere** — enforced upstream (XP/segment) |
| Casino Credit amount | Dynamic from `G{get}` in the code — CAD currency row |
| Stake Chunk Sizes | 1 — **conditional fill** (present on the flat CC form; skipped with a terminal note if absent) |
| **Platforms** | **Web + STAC: Standalone Casino only** — all three COMBO tags removed (matching VIPDM) |
| Days to Opt In / Entitlement | 3 (default, editable; Entitlement also extends the end dates) |
| Days to Meet Casino_credit Settlement | 7 (default, editable) |
| Status Active | Checked |
| Activation | Typed date − 1 day 00:00 ET → month-end 23:59:59 ET + entitlement days |
| Reporting Platform | CAS (Casino) |
| Bonus Origin | Opt-In Bonus |
| Series | VIP |
| Type | Retention |
| Confirmation modal | None observed — **handled defensively** (waits 5s; confirms if one appears, continues with a terminal note if not) |

> ⚠️ Series `VIP` and Type `Retention` use the same visible-dropdown-scoped anchored regex as VIPDM — the Type scoping is load-bearing (hidden Bonus Origin dropdown contains a `Retention` option).

Note: the `RET` token in the code name mirrors the **Type** field (Retention), not Bonus Origin — same as VIPDM. Confirmed end-to-end (segment + promo + bonus) in NATS production Aug 12, 2026.

---

## Naming Conventions

### Internal Code Names (used for segment, promotion, and bonus — all identical)

| Offer | Format | Example |
|---|---|---|
| RTC CC | `MMDDYY_CAS_RET_RTC_{jurisdiction}_CC_{amount}` | `053026_CAS_RET_RTC_US_CC_50` |
| RTC FS | `MMDDYY_CAS_RET_RTC_{jurisdiction}_FS_{spins}S_{value}V_{game}` | `061926_CAS_RET_RTC_MI_FS_10S_20V_TCE` |
| BG | `MMDDYY_CAS_RET_BG_{jurisdiction}_FC_B{bet}_G{get}` | `053026_CAS_RET_BG_US_FC_B100_G2` |
| BG-RE | `MMDDYY_CAS_RET_BG_{jurisdiction}_{rewardType}_B{bet}_G{get}` | `062226_CAS_RET_BG_US_FC_B100_G2` |
| DM | `MMDDYY_CAS_RET_DM_{jurisdiction}_FC_M10_{pct}M{max}` | `073126_CAS_RET_DM_US_FC_M10_20M500` |
| RAF Referee | `MMDDYY_CAS_ACQ_SUO_RAF_RFRE_{jurisdiction}_FS_{spinAmount}` | `100126_CAS_ACQ_SUO_RAF_RFRE_MI_FS_50` |
| RAF Referrer | `MMDDYY_CAS_ACQ_SUO_RAF_RFER_{jurisdiction}_FS_{spinAmount}` | `100126_CAS_ACQ_SUO_RAF_RFER_MI_FS_50` |
| RAF Day 2–N | `MMDDYY_CAS_ACQ_SUO_RAF_DAY{n}_{jurisdiction}_FS_{spinAmount}` | `100126_CAS_ACQ_SUO_RAF_DAY2_MI_FS_50` |
| SUO Day 2–N | `MMDDYY_CAS_ACQ_SUO_DAY{n}_{jurisdiction}_FS_{spinAmount}` | `100126_CAS_ACQ_SUO_DAY2_MI_FS_50` |
| LC-REACT | `MMDDYY_CAS_LC_REACT_{jurisdiction}_CC_{amount}` | `122926_CAS_LC_REACT_US_CC_10` |
| LC-CHURN-DM | `MMDDYY_CAS_LC_CHURN_DM_{jurisdiction}_CC_M{min}_{pct}M{max}` | `121126_CAS_LC_CHURN_DM_US_CC_M10_50M10` |
| VIPDM | `MMDDYY_VCL_RET_DM_CA_CC_M{min}_{pct}M{max}` | `090126_VCL_RET_DM_CA_CC_M250_20M1000` |
| VIPBG | `MMDDYY_VCL_RET_{SBG\|TBG}_CA_CC_B{bet}_G{get}` | `090126_VCL_RET_SBG_CA_CC_B1000_G100` |

VIPBG category tokens: **SBG** = Slots Bet & Get, **TBG** = Table Games Bet & Get.

### Voucher Codes

All voucher codes are exactly 15 characters. A random 3-character suffix (A–Z, 0–9) is generated at runtime to guarantee uniqueness.

| Offer | Format | Example |
|---|---|---|
| RTC CC | `CRRCCC{MMDDYY}{XXX}` | `CRRCCC052626AB3` |
| RTC FS | `CRRCFS{MMDDYY}{XXX}` | `CRRCFS052626Q7Z` |
| BG | `CLBGFC{MMDDYY}{XXX}` | `CLBGFC052626X2K` |
| BG-RE | `CLBGFC{MMDDYY}{XXX}` | `CLBGFC052626X2K` |
| DM | `CRDMFC{MMDDYY}{XXX}` | `CRDMFC052626M9R` |
| LC-REACT | `CRLCCC{MMDDYY}{XXX}` | `CRLCCC122926AB3` |
| LC-CHURN-DM | `CRLCDM{MMDDYY}{XXX}` | `CRLCDM121126X4T` |
| VIPDM | `VRDMCC{MMDDYY}{XXX}` | `VRDMCC090126X4T` |
| VIPBG | `CLBGFC{MMDDYY}{XXX}` — shared with BG/BG-RE | `CLBGFC090126X4T` |
| RAF Referee | `{jurisdiction}AQ{total_spins:04d}RFRE{XXX}` | `MIAQ0500RFREX3K` |
| RAF Referrer | `{jurisdiction}AQ{total_spins:04d}RFER{XXX}` | `MIAQ0500RFERX3K` |
| RAF Day 2–N | `{jurisdiction}AQ{total_spins:04d}DAY{n}{XXX}` | `MIAQ0500DAY2X3K` |
| SUO Day 2–N | None — no voucher codes | — |

> ⚠️ Voucher codes in NATS can **never** be reused, even if the segment, promotion, and bonus they were attached to have since been deleted. VIPBG's voucher is NATS-mandatory but unused for the offer itself; the random suffix keeps it unique across BG/BG-RE/VIPBG.

### Image Filenames
All offer types use: `Promo Detail.png`, `Masthead.png`
- RTC CC discover: `{amount}.png` (same convention in the US and Canada folders)
- RTC FS discover: `{spins}S_{value}V_{game}.png`
- BG discover: `B{bet}_G{get}.png`
- BG-RE discover: `B{bet}_G{get}.png`
- DM discover: `{pct}M{max}.png`
- LC-REACT discover: `{amount}.png`
- LC-CHURN-DM discover: `{pct}M{max}.png` (min deposit is not part of the filename)
- VIPDM discover: `{pct}M{max}.png` (min deposit is not part of the filename) — ⚠️ `10M250.png` and `100M250.png` differ by one character; double-check those two creatives
- VIPBG discover: `{SBG|TBG}_B{bet}_G{get}.png` (e.g. `SBG_B1000_G100.png`) — the full offer key, so no one-character ambiguity
- RAF Referee discover: `Discover_RFRE.png`
- RAF Referrer discover: `Discover_RFER.png`
- RAF Link Preview: `Link Preview.png`
- SUO: No images

### Generated Script Filenames
- RTC CC: `rtc_top_up_MMDDYY_HHMM.py`
- RTC FS: `rtc_fs_MMDDYY_HHMM.py`
- BG: `bg_MMDDYY_HHMM.py`
- BG-RE: `bg_re_MMDDYY_HHMM.py`
- DM: `dm_MMDDYY_HHMM.py`
- RAF: `raf_full_campaign_MMDDYY_HHMM.py`
- SUO: `suo_day2_MMDDYY_HHMM.py`
- LC-REACT: `lc_react_MMDDYY_HHMM.py`
- LC-CHURN-DM: `lc_churn_dm_MMDDYY_HHMM.py`
- VIPDM: `vcl_dm_MMDDYY_HHMM.py`
- VIPBG: `vcl_bg_MMDDYY_HHMM.py`

### Regions / Jurisdictions
- RTC CC: **US (default), CA only** (v2.22 — see Canada Support above)
- VIPDM: **CA only** (v2.23 — locked; US planned)
- VIPBG: **CA only** (v2.24 — locked; US planned)
- BG / BG-RE / DM / LC-REACT / LC-CHURN-DM: US (default), MI, WV, PA, NJ, ON, AB, CA
- RTC FS / RAF / SUO: MI, NJ, WV, PA only

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
| `vcl_dm_hiw` | Custom How It Works — VIPDM |
| `vcl_dm_tc` | Custom T&C — VIPDM |
| `vcl_dm_image_path` | Image folder path — VIPDM |
| `vcl_dm_terms_ids` | JSON object mapping offer key → Terms ID — VIPDM |
| `vcl_dm_terms_start_date` | Promotional start date in `MM/DD/YY` format — VIPDM |
| `vcl_dm_terms_end_date` | Promotional end date in `MM/DD/YY` format — VIPDM |
| `vcl_bg_hiw` | Custom How It Works — VIPBG |
| `vcl_bg_tc` | Custom T&C — VIPBG |
| `vcl_bg_image_path` | Image folder path — VIPBG |
| `vcl_bg_terms_ids` | JSON object mapping offer key → Terms ID — VIPBG |
| `vcl_bg_terms_start_date` | Promotional start date in `MM/DD/YY` format — VIPBG |
| `vcl_bg_terms_end_date` | Promotional end date in `MM/DD/YY` format — VIPBG |

SUO Day 2+ Spins and BG-RE have no localStorage keys. Clear Saved Settings removes all of the above, including the `_ca`, `vcl_dm_*`, and `vcl_bg_*` keys.

---

## Known Bugs

| # | Area | Severity | Issue |
|---|---|---|---|
| 2 | DM | ⚠️ | `format_hiw` / `format_tc` baked in at generation time — changes require script regeneration |
| 5 | RTC FS | ⚠️ | HIW and T&C use placeholder copy — pending legal-approved text |
| 6 | RAF | ⚠️ | NATS clears Referee/Referrer radio and Reporting Platform on bonus re-open — post-save edit pass required |
| 7 | RAF | ⚠️ | Bonus Manager requires 120-second wait during RFER edit pass — script stays on page |

---

## Known Limitations

| # | Area | Note |
|---|---|---|
| 1 | BG-RE | NATS and Playmaker URLs are currently pointed at the **test environment**. Switch before using in production. |
| 2 | BG-RE | Free Spins reward type not yet implemented — different field structure in Playmaker. |
| 3 | LC-REACT | T&C promotional periods and Terms IDs are baked in at generation time. Changes to any of the three tiers require regenerating the script. |
| 4 | LC-CHURN-DM | HIW, T&C, and Terms IDs are baked in at generation time — changes require regenerating the script. New offer sizes require adding a Terms ID to `CHURNDM_TERMS_IDS` in the HTML; offers without a configured Terms ID block script generation by design. |
| 5 | All templates | The Python scripts are generated inside JavaScript template literals, which **swallow single backslashes**. Any regex backslash intended for generated Python must be double-escaped in the HTML, and template regex changes must be verified by rendering through a real JS template literal and executing the resulting Python (root cause of the v2.20→v2.21 hotfix). The v2.22 CA T&C copy, the v2.23 VIPDM template (143 automated checks), and the v2.24 VIPBG template (380 automated checks, incl. byte-level T&C comparison against both legal docs) were all verified through this exact process — the VIPBG verification harness caught a `\$`-escaping bug of this class before it could ship. |
| 6 | RTC CC | Switching Region silently resets all day cards (no confirmation dialog, by design) — half-configured cards are cleared on region change. |
| 7 | VIPDM | HIW, T&C, Terms IDs, and promotional dates are baked in at generation time — changes via the Edit T&Cs / Terms modal require regenerating the script. New offer sizes require a Terms ID (editable in the modal); active offers without one block script generation by design. |
| 8 | VIPDM | Series `VIP` must exist in the NATS Series dropdown (confirmed present as of Aug 2026). CA — Canada only; US region planned (RTC CC-style, region-scoped keys). |
| 9 | VIPBG | HIW, T&C, Terms IDs, and promotional dates are baked in at generation time — changes via the Edit T&Cs / Terms modal require regenerating the script. New offer sizes require a Terms ID keyed `{SBG\|TBG}_B{bet}_G{get}` (editable in the modal); active offers without one block script generation by design. |
| 10 | VIPBG | CA — Canada only; US region planned (RTC CC-style, region-scoped keys — `currency_for()` already routes US codes to USD). **Button 2 is intentionally empty for CA**; `Play Now!` → `/docs/usered/casgenericgamelist` is documented in the generated script header for the US region. Series `VIP` dependency shared with VIPDM (#8). |

---

## Image System

### RTC Top Up - Casino Credit
Jurisdiction-scoped as of v2.22. Path stored in `localStorage` as `rtc_image_path` (US) / `rtc_image_path_ca` (CA). Edit Images button present; the modal reads/saves the selected region's path, shows the region in its title, and its Drive link follows the region.
**US default path (Adrian's machine):** `.../Marketing Automations/Lifecycle Automations/RTC Top Up`
**US Google Drive folder:** https://drive.google.com/drive/folders/1Dlpa3xZTHjzlwHgayPSS-MZJ8-DQTmhh
**CA default path (Adrian's machine):** `.../Marketing Automations/Lifecycle Automations/RTC Top Up - Canada`
**CA Google Drive folder:** https://drive.google.com/drive/folders/1W6ulHKv3ted9ZLNKzPgEbRoNCK_wFz6O

> ⚠️ Each user must set their own image path via the **Edit Images** button before generating a script — separately for US and for CA if they build both. Paths are saved to their browser's localStorage and only need to be set once per machine per region.

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

### VIP Offer Library - Deposit Matches
Path stored in `localStorage` as `vcl_dm_image_path`. Edit Images button present. Under the **VIP Automations** shared drive branch.
**Default path (Adrian's machine):** `/Users/adrian.vandecamp/Library/CloudStorage/GoogleDrive-adrian.vandecamp@betfanatics.com/Shared drives/Marketing Automations/VIP Automations/Offer Library/Canada/Deposit Matches`
**Google Drive folder:** https://drive.google.com/drive/folders/1v9DEVtCNQPCgUkJsWtsXn7UJVIyBYtr4
**Image filenames:** `Promo Detail.png`, `Masthead.png`, `{pct}M{max}.png` (e.g. `20M500.png`). Min deposit is not part of the Discover filename. Default-tier Discover set: `50M250.png`, `100M250.png`, `10M250.png`, `10M2500.png`, `10M5000.png`, `20M500.png`, `20M1000.png`, `20M2000.png`, `20M5000.png` — ⚠️ `10M250.png` vs `100M250.png` differ by one character; double-check those creatives.

### VIP Offer Library - Bet & Gets
Path stored in `localStorage` as `vcl_bg_image_path`. Edit Images button present. Under the **VIP Automations** shared drive branch, sibling to Deposit Matches.
**Default path (Adrian's machine):** `/Users/adrian.vandecamp/Library/CloudStorage/GoogleDrive-adrian.vandecamp@betfanatics.com/Shared drives/Marketing Automations/VIP Automations/Offer Library/Canada/Bet & Gets`
**Google Drive folder:** https://drive.google.com/drive/folders/1tkMs1dx-gszSGzlBHxgMYvOYv2l-2UjO
**Image filenames:** `Promo Detail.png`, `Masthead.png`, `{SBG|TBG}_B{bet}_G{get}.png` — the Discover filename is the full offer key (e.g. `SBG_B1000_G100.png`, `TBG_B100000_G2500.png`). Default-tier Discover set: `SBG_B1000_G100.png`, `SBG_B5000_G500.png`, `SBG_B20000_G1000.png`, `SBG_B100000_G5000.png`, `TBG_B2000_G100.png`, `TBG_B10000_G500.png`, `TBG_B25000_G1000.png`, `TBG_B100000_G2500.png`.

---

## Sidebar Navigation

All generated scripts navigate between screens by hovering sidebar icons using `data-menu-id` attribute selectors and clicking the target submenu item via `dispatch_event("click")`.

| Screen | Sidebar selector | Submenu item |
|---|---|---|
| Account Segments | `[data-menu-id$='-segments']` | `Account Segments` |
| Promos | `[data-menu-id$='-cms']` | `Promos` |
| Bonus Manager | `[data-menu-id$='-bonuses']` | `Bonus Manager` |

Each nav step moves the mouse to a safe zone (600, 400), hovers the sidebar icon, then uses `dispatch_event("click")` on the submenu item. Up to 5 retry attempts with safe-zone reset. Applies to all offer types that use NATS.

---

## Per-Code Step Toggles

### RTC CC / RTC FS / BG / DM / LC-REACT / LC-CHURN-DM / VIPDM / VIPBG
Every day card shows three toggleable step badges: **Segment**, **Promo**, **Bonus**. The generated script includes `DO_SEGMENT`, `DO_PROMO`, and `DO_BONUS` boolean arrays. Phases are skipped if no codes have that step enabled.

### Refer a Friend
The RAF campaign card shows three toggleable phase badges: **Day 2+**, **Referee**, **Referrer**.

| Badge | Steps included |
|---|---|
| Day 2+ | Segments (DAY2–N) + Bonuses (DAY2–N) |
| Referee | Promo (RFRE) + Bonus + Post-save edit |
| Referrer | Segment (RFER) + Promo (RFER) + Bonus + Post-save edit |

### SUO Day 2+ Spins
No step badges. Segment and bonus are always built for every day code.

### Bet & Get - Rules Engine
**Promo** and **Bonus** badges only — no Segment badge. The segment is created inside Playmaker as part of the bonus flow.

---

## Eastern Time Enforcement

All generated scripts enforce Eastern Time (America/New_York) via Python's `zoneinfo` module. **Python 3.9+ is required.**

---

## Platform Tags — Bonus Creation

NATS pre-populates platform tags on every new bonus. Generated scripts for the eight pre-v2.23 NATS offer types remove `COMBO: Sportsbook` and `COMBO: Sportsbook And Casino`, leaving:

- `STAC: Standalone Casino`
- `Web`
- `COMBO: Casino`

**VIP Offer Library exception (v2.23/v2.24):** VIPDM and VIPBG both remove **all three COMBO tags** (including `COMBO: Casino`), leaving only `Web` + `STAC: Standalone Casino`. They are the only offer types that remove `COMBO: Casino`.

BG-RE bonuses are built in Playmaker, not NATS, so platform tags do not apply.

---

## Lifecycle - Churn DM

### Day Card Layout
- **4-column grid | Default days:** 4
- Offer rows are **Min → % → Max** with a per-offer checkbox. Five defaults shown (unselected): $10/50%/$10, $20/50%/$25, $20/50%/$50, $50/50%/$100, $50/50%/$200.
- No Campaign Name field, no ZIP attachment. Send to XP default ON; External Bonus present, default OFF. Days to Opt In 3 / Entitlement 3 / Settlement 7.

### Terms ID Enforcement
Each active offer key `M{min}_{pct}M{max}` must exist in the baked-in `CHURNDM_TERMS_IDS` map. Non-matching active offers show a red warning on the row and **block script generation** with an alert. New offer sizes require a Terms ID to be added to the HTML.

### Promo & Bonus Activation Window
Both promos and bonuses use a **10-day window**:
- **Start:** Day-of at 00:00:00 Eastern
- **End:** Day+10 at 00:00:00 Eastern (exactly start + 240 hours)

### Fixed Copy
Title, Promo Header Text, and Bonus Description are all `Deposit and Get Casino Credit` regardless of offer size. No Button 2 label/link on the promo.

### T&C Data (Baked In)
All five default offers share one promotional period: **June 1, 2026 → December 1, 2026**. T&C title line: `FANATICS CASINO - DEPOSIT {min_fmt}+, GET {pct}% DEPOSIT MATCH UP TO {max_fmt} IN CC - {terms_id}`.

| Offer key | Min | % | Max CC | Terms ID |
|---|---|---|---|---|
| `M10_50M10` | $10 | 50 | $10 | `CAS_10620` |
| `M20_50M25` | $20 | 50 | $25 | `CAS_9457` |
| `M20_50M50` | $20 | 50 | $50 | `CAS_9458` |
| `M50_50M100` | $50 | 50 | $100 | `CAS_9459` |
| `M50_50M200` | $50 | 50 | $200 | `CAS_9460` |

### Bonus Creation — Fixed Values

| Field | Value |
|---|---|
| Trigger type | Opt In |
| Bonus type | Casino Credit (percentage checkbox labeled `Casino Credit (%)`) |
| Description | Deposit and Get Casino Credit |
| Entitlement Type | Deposit |
| Minimum Deposit | Dynamic from `M{min}` in the code — jurisdiction currency row |
| Maximum Deposit | 1000000 — jurisdiction currency row |
| Days to Opt In / Entitlement | 3 (default, editable per card) |
| Days to Meet Casino_credit Settlement | 7 (default, editable per card) |
| Status Active | Checked |
| Activation | 00:00:00 ET day-of → 00:00:00 ET day+10 |
| Reporting Platform | CAS (Casino) |
| Bonus Origin | Retention |
| Series | Other |
| Type | Lifecycle |
| Confirmation modal | No |

---

## Lifecycle - REACT CC Drop

### Day Card Layout
- **Default days:** 3 | **Default amounts:** 10, 25, 50 (shown but unselected)

### Promo & Bonus Activation Window
Both promos and bonuses use a **72-hour window**:
- **Start:** Day-of at 00:00:00 Eastern
- **End:** Day+2 at 23:59:59 Eastern (i.e. start + 72h − 1 second)

### T&C Data (Baked In)

| Amount | Terms ID | Promotional Period |
|---|---|---|
| $10 | `CAS_9237` | April 16, 2026 → October 16, 2026 |
| $25 | `CAS_9238` | April 16, 2026 → October 16, 2026 |
| $50 | `CAS_9456` | June 1, 2026 → December 1, 2026 |

### Bonus Creation — Fixed Values

| Field | Value |
|---|---|
| Trigger type | Opt In |
| Stake Chunk Sizes | 1 |
| Casino Credit amount | Jurisdiction currency row (`fill_currency_amount`) |
| Status Active | Checked |
| Days to Opt In / Entitlement / Settlement | 3 (default, editable per card) |
| Activation | 00:00:00 ET day-of → 23:59:59 ET day+2 |
| Reporting Platform | CAS (Casino) |
| Bonus Origin | Opt-In Bonus |
| Series | Reactivation |
| Type | Lifecycle |
| Confirmation modal | No |

---

## RTC Top Up - Casino Credit

### Day Card Layout
- **Default days:** 9 | **Default amounts:** 2, 4, 5, 10, 20, 40, 50, 100, 200, 400, 800

### Region (v2.22)
- **US — United States** (default) or **CA — Canada** only.
- Switching Region resets all day cards (silent, no confirmation).
- With CA selected, the Edit HIW / Edit T&Cs / Edit Images buttons turn red as a visual cue.

### Promo & Bonus Activation Window
Both promos and bonuses use a **4 AM ET start, 24-hour window**:
- **Start:** Day-of at 04:00:00 Eastern
- **End:** Following day at 04:00:00 Eastern (i.e. `start + timedelta(hours=24)`)

> ⚠️ This differs from all other offer types (BG, DM, RTC FS, RAF, SUO, LC-REACT, LC-CHURN-DM, VIPDM, VIPBG), which use a midnight-anchored, 72-hour, 10-day, or month-end window. The window is identical for US and CA.

### Terms IDs & Promotional Dates
**US defaults:** Start: `06/01/26` | End: `12/01/26` | IDs: `$2→CAS_9461` through `$800→CAS_9471`
**CA defaults (v2.22):** Start: `08/17/26` | End: `12/31/26` | IDs: `$2→CAS_CA_0001` through `$800→CAS_CA_0011`
Each region's dates and IDs are stored and edited independently via the Edit Terms Expiry & Terms IDs modal (the modal edits whichever region is selected and shows the region in its title).

### Bonus Creation — Fixed Values
| Field | Value |
|---|---|
| Trigger type | Opt In |
| Stake Chunk Sizes | 1 |
| Casino Credit amount | Jurisdiction currency row (`fill_currency_amount` — USD for US, CAD for CA) |
| Status Active | checked |
| Activation | day-of 04:00 ET → +24 hours ET |
| Reporting Platform | CAS (Casino) |
| Bonus Origin | Retention |
| Series | RTC |
| Type | Lifecycle |
| Confirmation modal | No |

---

## RTC Top Up - Free Spins

### Day Card Layout
- **Default days:** 8 | **Default jurisdictions:** Day 1→MI, Day 2→NJ, Day 3→WV, Day 4→PA, Days 5–8→MI

### Games

| Acronym | Full Name | Search Name | Aggregator | Provider | Deeplink |
|---|---|---|---|---|---|
| TCE | Triple Cash Eruption | Triple Cash Eruption | IGT | IGT_Rgs | /casino_game/200-1700-081 |
| 7FH | 7's Fire Blitz™ Hotstepper | 7's Fire Blitz™ Hotstepper | WHG | WHITEHATSTUDIOS | /casino_game/WHS_7sFireBlitzHotStepper95 |
| 7P5 | 7's Fire Blitz™ Power 5 Jackpot Royale™ Express | 7's Fire Blitz™ Power 5 Jackpot Royale™ Express | WHG | WHITEHATSTUDIOS | /casino_game/WHS_7sFireBlitzPower5JRE92 |
| WWE | WrestleMania: Road to Gold | WrestleMania™: Road To Gold | WHG | WHITEHATSTUDIOS | /casino_game/WHS_WrestlemaniaRoadToGoldUS94 |

> ⚠️ Game search names use a straight apostrophe (`'`) not a curly apostrophe (`'`).

> ⚠️ **NATS game option names include trademark symbols as of August 2026** (e.g. `Triple Cash Eruption™ (WV)`). As of v2.20/v2.21, scripts type only the pre-™ portion of the Search Name into the Casino Game search box, then select the option with an anchored regex where every `™` is optional but the jurisdiction suffix is required. The Search Name column above does **not** need editing when NATS adds/removes `™`. Applies to RTC FS, RAF, and SUO.

### Bonus Creation — Fixed Values
| Field | Value |
|---|---|
| Trigger type | Opt In |
| Free Spin's CTA | Play! |
| Bet Level | default |
| Spin Value | Free Spin Stakes row for jurisdiction currency (`currency_for`) |
| Status Active | checked |
| Activation | day-of Eastern midnight → day+30 Eastern midnight |
| Reporting Platform | CAS (Casino) |
| Bonus Origin | Retention |
| Series | RTC |
| Type | Lifecycle |
| Confirmation modal | No |

---

## SUO Day 2+ Spins

### Campaign Card Layout
Single card (full width, two-column layout). Left column: Date, Jurisdiction, Game, Spins/Day, Spin Value, Bet Amount, Number of Days toggle (5/10). Right column: live preview panel showing all DAY2–DAYN code names.

### Build Status

| Code Type | Segment | Promo | Bonus | Status |
|---|---|---|---|---|
| SUO Day 2–N | ✅ Confirmed | ❌ Not needed | ✅ Confirmed | ✅ Production |

### Fixed Values

| Field | Value |
|---|---|
| Trigger | No Opt In |
| Free Spin's CTA | Play! |
| Bet Level | default |
| Activation end | year 2041 |
| Reporting Platform | CAS (Casino) |
| Bonus Origin | Bet Settlement Bonus |
| Series | Early Life |
| Type | Acquisition |
| Status Active | Checked |

---

## Bet & Get - Rules Engine (BG-RE)

Fully integrated into the HTML tool as of v2.15. See `Technical_Reference_v2_24.md` for full selector reference and Playmaker field spec.

> ⚠️ Currently using **test environment** URLs. Switch before using in production.

### Overview
Two-phase script:
- **Phase 1 (NATS):** Builds Promo only — identical to existing BG promo flow including ZIP images
- **Phase 2 (Playmaker):** Builds combined segment + bonus at `https://playmaker-internal.test1.fanatics.bet/`

---

## Script Execution Notes

### Create Promotion Button Poll
All `create_promo` functions poll the "Create Promotion" button up to 60 seconds (120 × 500ms) before clicking.

### Confirmation Modal After Bonus Save
| Offer | Modal |
|---|---|
| RTC CC | No |
| RTC FS | No |
| BG | Yes |
| DM | Yes |
| RAF RFRE | Yes — voucher modal → Apply |
| RAF RFER | Yes — same as RFRE |
| RAF Day 2+ | No modal |
| SUO Day 2+ | No modal |
| BG-RE | No modal |
| LC-REACT | No modal |
| LC-CHURN-DM | No modal |
| VIPDM | No modal |
| VIPBG | No modal observed — handled defensively (confirms if one ever appears, continues with a terminal note if not) |

### Promotion Tile Selection
Uses regex anchored exact match scoped to `.ant-select-dropdown:not(.ant-select-dropdown-hidden)`. Applied to all offer types that use a Promotion Tile. SUO and BG-RE do not use a Promotion Tile in NATS.

### Segment Dropdown Selection
LC-REACT, LC-CHURN-DM, VIPDM, and VIPBG use the same anchored regex pattern as the Promotion Tile — scoped to `.ant-select-dropdown:not(.ant-select-dropdown-hidden)` — to avoid hitting the hidden search mirror span that NATS injects alongside the input.

### Series / Type Dropdown Selection (VIPDM / VIPBG)
VIPDM and VIPBG select Series `VIP` (typed search + anchored `^VIP$`) and Type `Retention` (anchored `^Retention$`), both scoped to the visible dropdown. The Type scoping is required: the hidden Bonus Origin dropdown also contains a `Retention` option, so a page-wide exact-text click would match two elements and crash with a Playwright strict mode violation. VIPDM and VIPBG are the only offer types that select `Retention` for Type.

### Currency-Row Amount Fill (v2.18+)
Bonus amounts, DM/LC-CHURN-DM/VIPDM deposit fields, the VIPBG flat Casino Credit amount, and FS spin values are filled into the jurisdiction's currency row (`tr[data-row-key='USD'|'CAD']`), never GBP, with a legacy single-input fallback. Terminal output prints the currency selected. See **Multi-Currency Amount Tables** above.

### Casino Game Selection (v2.20/v2.21)
RTC FS, RAF, and SUO type only the pre-™ portion of the game's Search Name, then select the dropdown option with an anchored, ™-tolerant, jurisdiction-required regex (`parse_casino_game_search()` / `parse_casino_game_pattern()`). Terminal output prints the actual option text clicked.

### Create Bonus Button — Multi-Code Runs
`create_bonus` polls up to 120 × 500ms (60 seconds) for the button to re-enable.

### AMELCO Dropdown Selection
All `create_segment` functions select AMELCO via `dispatch_event("click")` rather than `.click()` to bypass Ant Design pointer interception. A 2-second wait is inserted after `#forBonus` is checked and before OK is clicked.

### Casino Credit Deposit-Match Form (LC-CHURN-DM / VIPDM)
Under the Casino Credit bonus type, the deposit-match percentage checkbox is labeled **`Casino Credit (%)`** (not `Bonus (%)` as on the FanCash DM form). The rest of the field structure is identical to DM. Generated scripts include label fallbacks: the summary percentage field tries `Bonus %` then `Casino Credit %`, and the amount summary card tries the `Bonus` then `Casino Credit` card titles. The settlement field targets `Days to Meet Casino_credit Settlement`. VIPDM additionally checks the **Matched Deposit** checkbox inside the DEPOSIT entitlement panel (with an explicit visibility wait, since the panel renders only after Entitlement Type is set).

### Casino Credit Flat Form (VIPBG)
VIPBG uses the flat Casino Credit form (as RTC CC / LC-REACT), not the deposit-match variant: no percentage checkbox, no deposit entitlement, no Matched Deposit, no Minimum/Maximum Deposit. Fields: `Days to Meet Casino_credit Settlement`, opt-in/entitlement days, a **conditional Stake Chunk Sizes fill of 1** (filled if present, skipped with a terminal note if not), and the flat CC amount = the code's "get" value via `fill_currency_amount` (CAD row).

### VIP Offer Library Month-Anchored Window (VIPDM / VIPBG)
`parse_dates(code, entitlement_days)` is the only date function that takes a per-card input: start = typed date − 1 day at 00:00:00 ET; end = last calendar day of the typed date's month at 23:59:59 ET + Days to Meet Entitlement (`calendar.monthrange`; leap years and year crossovers handled; the typed day anchors only the start). Applied identically to the promo dates and the bonus Activation Dates for both offer types.

### Image Upload Timing
After file selection, scripts wait **8 seconds** before clicking OK, then 2 seconds after OK. Each image upload takes ~10 seconds total; 3 images per promo = ~30 seconds per promo build. Applies to RTC CC, RTC FS, BG, DM, RAF, BG-RE, LC-REACT, LC-CHURN-DM, VIPDM, and VIPBG. SUO has no images.

---

## How It Works & T&C Copy

### RTC CC
**HIW:** Shared by US and CA — dynamic variable: `{amount_fmt}` — saved as `rtc_hiw`.
**T&C:** Jurisdiction-scoped (v2.22). Dynamic variables (both regions): `{amount_fmt}`, `{terms_id}`, `{start_date_short}`, `{start_date_long}`, `{end_date_short}`, `{end_date_long}`.
- **US:** saved as `rtc_tc`; title line `FANATICS CASINO - CASINO CREDIT {amount_fmt} SURPRISE DROP - {start_date_short} (ID: {terms_id})`.
- **CA:** saved as `rtc_tc_ca`; Canadian (Ontario) legal base copy baked in; title line `FANATICS CASINO (CANADA) – CASINO CREDIT {amount_fmt} CAD SURPRISE DROP ({terms_id})`.
The Edit T&Cs modal edits whichever region is selected.

### RTC FS
Placeholder copy — pending legal-approved text — saved as `rtc_fs_hiw` / `rtc_fs_tc`

### BG
Dynamic variables: `{bet_fmt}`, `{get_fmt}`, `{date_fmt}` — saved as `bg_hiw` / `bg_tc`

### DM
Dynamic variables: `{pct}`, `{max_fmt}`, `{date_fmt}` — saved as `dm_hiw` / `dm_tc`. Changes require script regeneration.

### RAF
Placeholder copy — pending legal-approved text. Not yet integrated into modal system.

### SUO Day 2+ Spins
No HIW or T&C. Not applicable.

### Bet & Get - Rules Engine
Placeholder copy — pending legal-approved text. Not applicable to Playmaker bonus flow.

### Lifecycle - REACT CC Drop
**HIW:** Static template (no dynamic variables) — saved as `lc_react_hiw`:
```
1. Opt-in to the promotion
2. We'll instantly drop Casino Credit into your account
3. Play the Casino Credit on any of your favorite games

See below for more details.
```
**T&C:** Per-amount, baked in at generation time via `getLCREACTTC(amount)`. Dynamic fields: `{amount}` (title line + description), `{terms_id}` (per-tier), `{promo_period}` (per-tier). Changes require script regeneration. Saved as `lc_react_tc` (single-amount template for Edit T&C modal display).

### Lifecycle - Churn DM
**HIW:** Legal-approved template — saved as `lc_churn_dm_hiw`. Dynamic fields: `{pct}`, `{max_fmt}`:
```
1. Opt in to this promotion 
2. Make a Qualifying Deposit within 3 days of being presented this offer
3. Rewards will equal {pct}% of your Qualifying Deposit in Casino Credit (up to {max_fmt} in Casino Credit)

See below for more details.
```
**T&C:** Full legal-approved copy, baked in at generation time — saved as `lc_churn_dm_tc`. Dynamic fields: `{min_fmt}`, `{pct}`, `{max_fmt}`, `{terms_id}`. Title line: `FANATICS CASINO - DEPOSIT {min_fmt}+, GET {pct}% DEPOSIT MATCH UP TO {max_fmt} IN CC - {terms_id}`. Promotional period (June 1, 2026 → December 1, 2026) is written into the default T&C text and shared by all five offers — edit via the Edit T&C modal when T&Cs are re-filed. Changes require script regeneration.

### VIP Offer Library - Deposit Matches
**HIW:** Legal-approved template — saved as `vcl_dm_hiw`. Dynamic fields: `{min_fmt}`, `{pct}`, `{max_fmt}`. No opt-in deadline line (the promo window bounds the offer) and no "See below" trailer, by design:
```
1. Opt-in to the promotion
2. Make a single deposit of {min_fmt} or more
3. We'll instantly match your deposit {pct}%, up to {max_fmt} Casino Credit
```
**T&C:** Full Canadian (Ontario) legal copy, baked in at generation time — saved as `vcl_dm_tc`. Dynamic fields: `{min_fmt}`, `{pct}`, `{max_fmt}`, `{terms_id}`, `{start_date_long}`, `{end_date_long}` (short variants also available). Title line: `FANATICS CASINO (CANADA) – DEPOSIT {min_fmt} CAD, GET {pct}% CASINO CREDIT (UP TO {max_fmt} CAD) ({terms_id})`. Promotional dates (default August 17, 2026 → December 31, 2026) and per-offer Terms IDs are edited via the **Edit Terms Expiry & Terms IDs** button inside the Edit T&Cs modal. §1's "within 3 days of being presented this offer" and §4's "during that Promo Period" are static legal copy by design — do not template them. Changes require script regeneration.

### VIP Offer Library - Bet & Gets
**HIW:** Legal-approved template — saved as `vcl_bg_hiw`. Dynamic fields: `{bet_fmt}`, `{get_fmt}`, `{game_category}` (`Slots` / `Tables`). First-person VIP-host voice; Canadian spelling "favourite"; no opt-in deadline line and no "See below" trailer, by design:
```
1. Opt-in to the promotion
2. Wager {bet_fmt}+ on any of your favourite {game_category} games
3. I'll instantly give you {get_fmt} in Casino Credit
```
**T&C:** Full Canadian (Ontario) legal copy — **one template for both Slots and Tables** — baked in at generation time, saved as `vcl_bg_tc`. Dynamic fields: `{bet_fmt}`, `{get_fmt}`, `{terms_id}`, `{game_category_lower}` (`slots` / `table` — legal writes "slots games" but "table games"), `{game_category_upper}` (`SLOTS` / `TABLES`), `{start_date_long}`, `{end_date_long}` (short variants also available). Title line: `FANATICS CASINO (CANADA) – {game_category_upper} BET {bet_fmt} CAD, GET {get_fmt} CAD IN CASINO CREDIT ({terms_id})`. Promotional dates (default August 17, 2026 → December 31, 2026) and per-offer Terms IDs are edited via the **Edit Terms Expiry & Terms IDs** button inside the Edit T&Cs modal. The seventy-two (72) hour wagering-window language, 7-day Casino Credit expiry, and §5 CAD examples are static legal copy by design — do not template them. Changes require script regeneration.

---

## Refer a Friend (RAF)

### Build Status

| Code Type | Segment | Promo | Bonus | Status |
|---|---|---|---|---|
| Referee (RFRE) | ❌ Not needed | ✅ Confirmed | ✅ Confirmed | ✅ Production |
| Referrer (RFER) | ✅ Confirmed | ✅ Confirmed | ✅ Confirmed | ✅ Production |
| Day 2–N | ✅ Confirmed | ❌ Not needed | ✅ Confirmed | ✅ Production |

### Fixed Values Across All RAF Bonuses

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
