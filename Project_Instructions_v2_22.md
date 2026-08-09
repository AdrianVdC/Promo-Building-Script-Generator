# NATS Bonus Creator — Project Instructions

## What This Is
The NATS Bonus Creator is a single-file HTML tool that generates ready-to-run Python/Playwright scripts to automate segment, promotion, and bonus creation in the Fanatics Casino internal trading platform at `https://trading.1.betfanatics.com/` (Ant Design UI). The HTML file is the source of truth. Generated scripts are output-only and should never be edited directly.

**Current live version:** `nats_bonus_creator_v2_22.html`

> ⚠️ **v2.20 must not be used** — it contains a broken Casino Game regex (template escaping bug, hotfixed in v2.21). Any scripts generated from v2.20 will fail at Casino Game selection. Scripts generated from v2.17 or earlier will fail (BG, RTC CC, RTC FS, RAF, SUO, LC-REACT) or silently misfill Canadian amounts (DM, LC-CHURN-DM) due to the August 2026 NATS multi-currency and game-naming updates. **Canadian RTC CC scripts must be generated from v2.22 only** — v2.21 and earlier allowed selecting ON/AB/CA on RTC CC but would bake in US images, US T&Cs, and US Terms IDs. Regenerate all pending scripts from v2.22.

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
Nine offer types are supported: **RTC Top Up - Casino Credit**, **RTC Top Up - Free Spins**, **Bet & Get (BG)**, **Deposit Match (DM)**, **Refer a Friend (RAF)**, **SUO Day 2+ Spins**, **Bet & Get - Rules Engine (BG-RE)**, **Lifecycle - REACT CC Drop (LC-REACT)**, **Lifecycle - Churn DM (LC-CHURN-DM)**.

> ⚠️ **Bet & Get - Rules Engine is fully integrated as of v2.15 but currently uses test environment URLs.** NATS and Playmaker links must be switched to production once the Rewards Engine is live in the Playmaker production environment. Full selector reference and field spec are in `Technical_Reference_v2_22.md` attached to this project.

---

## Multi-Currency Amount Tables (v2.18+)

An August 2026 NATS update converted bonus amount tables from a single input into a **three-row table with one row per currency** (`USD` / `GBP` / `CAD`), keyed by `data-row-key` on each `<tr>`. All generated scripts include two shared helpers:

- **`currency_for(name)`** — derives the currency from the code name's jurisdiction token: US / MI / WV / PA / NJ → **USD**; ON / AB / CA → **CAD**. **GBP is never used under any circumstance** and is unreachable by construction.
- **`fill_currency_amount()`** — fills the value into the jurisdiction's currency row only (`tr[data-row-key='{currency}']`), with a fallback to the legacy single-input selector if NATS reverts. Terminal output shows which row was filled, e.g. `OK Bonus Amount (USD): 50`.

**Fields filled via the currency row:** RTC CC amount, LC-REACT amount, BG FanCash amount, DM Bonus amount + Min/Max Deposit, LC-CHURN-DM amount + Min/Max Deposit, and the Free Spin Stakes spin value for RTC FS / RAF / SUO (a select dropdown, chosen by currency label). Percentage inputs (DM, LC-CHURN-DM) are not part of the currency table.

---

## RTC Top Up - Casino Credit — Canada Support (v2.22)

RTC CC is dual-jurisdiction: **US — United States** (default) and **CA — Canada**. The tool works exactly as before for US; CA layers its own settings on top via parallel `_ca` localStorage keys resolved through the `isRTCCanada()` / `rtcKey()` helpers.

**What differs when CA — Canada is selected:**
- **Region dropdown** on RTC CC shows only US and CA (all other offer types keep their existing region lists).
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
| RAF Referee | `{jurisdiction}AQ{total_spins:04d}RFRE{XXX}` | `MIAQ0500RFREX3K` |
| RAF Referrer | `{jurisdiction}AQ{total_spins:04d}RFER{XXX}` | `MIAQ0500RFERX3K` |
| RAF Day 2–N | `{jurisdiction}AQ{total_spins:04d}DAY{n}{XXX}` | `MIAQ0500DAY2X3K` |
| SUO Day 2–N | None — no voucher codes | — |

> ⚠️ Voucher codes in NATS can **never** be reused, even if the segment, promotion, and bonus they were attached to have since been deleted.

### Image Filenames
All offer types use: `Promo Detail.png`, `Masthead.png`
- RTC CC discover: `{amount}.png` (same convention in the US and Canada folders)
- RTC FS discover: `{spins}S_{value}V_{game}.png`
- BG discover: `B{bet}_G{get}.png`
- BG-RE discover: `B{bet}_G{get}.png`
- DM discover: `{pct}M{max}.png`
- LC-REACT discover: `{amount}.png`
- LC-CHURN-DM discover: `{pct}M{max}.png` (min deposit is not part of the filename)
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

### Regions / Jurisdictions
- RTC CC: **US (default), CA only** (v2.22 — see Canada Support above)
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

SUO Day 2+ Spins and BG-RE have no localStorage keys. Clear Saved Settings removes all of the above, including the `_ca` keys.

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
| 5 | All templates | The Python scripts are generated inside JavaScript template literals, which **swallow single backslashes**. Any regex backslash intended for generated Python must be double-escaped in the HTML, and template regex changes must be verified by rendering through a real JS template literal and executing the resulting Python (root cause of the v2.20→v2.21 hotfix). The v2.22 CA T&C copy was verified through this exact process for all 11 tiers. |
| 6 | RTC CC | Switching Region silently resets all day cards (no confirmation dialog, by design) — half-configured cards are cleared on region change. |

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

### RTC CC / RTC FS / BG / DM / LC-REACT / LC-CHURN-DM
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

NATS pre-populates platform tags on every new bonus. All generated scripts remove `COMBO: Sportsbook` and `COMBO: Sportsbook And Casino`. The resulting active platforms on every bonus are:

- `STAC: Standalone Casino`
- `Web`
- `COMBO: Casino`

Applies to all eight NATS offer types. BG-RE bonuses are built in Playmaker, not NATS, so platform tags do not apply.

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

> ⚠️ This differs from all other offer types (BG, DM, RTC FS, RAF, SUO, LC-REACT, LC-CHURN-DM), which use a midnight-to-midnight (00:00:00 ET), 72-hour, or 10-day window. The window is identical for US and CA.

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

Fully integrated into the HTML tool as of v2.15. See `Technical_Reference_v2_22.md` for full selector reference and Playmaker field spec.

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

### Promotion Tile Selection
Uses regex anchored exact match scoped to `.ant-select-dropdown:not(.ant-select-dropdown-hidden)`. Applied to all offer types that use a Promotion Tile. SUO and BG-RE do not use a Promotion Tile in NATS.

### Segment Dropdown Selection
LC-REACT and LC-CHURN-DM use the same anchored regex pattern as the Promotion Tile — scoped to `.ant-select-dropdown:not(.ant-select-dropdown-hidden)` — to avoid hitting the hidden search mirror span that NATS injects alongside the input.

### Currency-Row Amount Fill (v2.18+)
Bonus amounts, DM/LC-CHURN-DM deposit fields, and FS spin values are filled into the jurisdiction's currency row (`tr[data-row-key='USD'|'CAD']`), never GBP, with a legacy single-input fallback. Terminal output prints the currency selected. See **Multi-Currency Amount Tables** above.

### Casino Game Selection (v2.20/v2.21)
RTC FS, RAF, and SUO type only the pre-™ portion of the game's Search Name, then select the dropdown option with an anchored, ™-tolerant, jurisdiction-required regex (`parse_casino_game_search()` / `parse_casino_game_pattern()`). Terminal output prints the actual option text clicked.

### Create Bonus Button — Multi-Code Runs
`create_bonus` polls up to 120 × 500ms (60 seconds) for the button to re-enable.

### AMELCO Dropdown Selection
All `create_segment` functions select AMELCO via `dispatch_event("click")` rather than `.click()` to bypass Ant Design pointer interception. A 2-second wait is inserted after `#forBonus` is checked and before OK is clicked.

### Casino Credit Deposit-Match Form (LC-CHURN-DM)
Under the Casino Credit bonus type, the deposit-match percentage checkbox is labeled **`Casino Credit (%)`** (not `Bonus (%)` as on the FanCash DM form). The rest of the field structure is identical to DM. Generated scripts include label fallbacks: the summary percentage field tries `Bonus %` then `Casino Credit %`, and the amount summary card tries the `Bonus` then `Casino Credit` card titles. The settlement field targets `Days to Meet Casino_credit Settlement`.

### Image Upload Timing
After file selection, scripts wait **8 seconds** before clicking OK, then 2 seconds after OK. Each image upload takes ~10 seconds total; 3 images per promo = ~30 seconds per promo build. Applies to RTC CC, RTC FS, BG, DM, RAF, BG-RE, LC-REACT, and LC-CHURN-DM. SUO has no images.

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
