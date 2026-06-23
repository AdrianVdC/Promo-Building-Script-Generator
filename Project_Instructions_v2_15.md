# NATS Bonus Creator — Project Instructions

## What This Is
The NATS Bonus Creator is a single-file HTML tool that generates ready-to-run Python/Playwright scripts to automate segment, promotion, and bonus creation in the Fanatics Casino internal trading platform at `https://trading.1.betfanatics.com/` (Ant Design UI). The HTML file is the source of truth. Generated scripts are output-only and should never be edited directly.

**Current live version:** `nats_bonus_creator_v2_15.html`

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
**Windows format:**
```
cd %USERPROFILE%\Downloads && python {filename}.py
```

This applies to every script in every conversation in this project, including partial test scripts, debug scripts, and final generated scripts.

---

## Offer Types
Seven offer types are supported: **RTC Top Up - Casino Credit**, **RTC Top Up - Free Spins**, **Bet & Get (BG)**, **Deposit Match (DM)**, **Refer a Friend (RAF)**, **SUO Day 2+ Spins**, **Bet & Get - Rules Engine (BG-RE)**.

> ⚠️ **Bet & Get - Rules Engine is fully integrated as of v2.15 but currently uses test environment URLs.** NATS and Playmaker links must be switched to production once the Rewards Engine is live in the Playmaker production environment. Full selector reference and field spec are in `Technical_Reference_v2_15.md` attached to this project.

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

### Voucher Codes

All voucher codes are exactly 15 characters. A random 3-character suffix (A–Z, 0–9) is generated at runtime to guarantee uniqueness.

| Offer | Format | Example |
|---|---|---|
| RTC CC | `CRRCCC{MMDDYY}{XXX}` | `CRRCCC052626AB3` |
| RTC FS | `CRRCFS{MMDDYY}{XXX}` | `CRRCFS052626Q7Z` |
| BG | `CLBGFC{MMDDYY}{XXX}` | `CLBGFC052626X2K` |
| BG-RE | `CLBGFC{MMDDYY}{XXX}` | `CLBGFC052626X2K` |
| DM | `CRDMFC{MMDDYY}{XXX}` | `CRDMFC052626M9R` |
| RAF Referee | `{jurisdiction}AQ{total_spins:04d}RFRE{XXX}` | `MIAQ0500RFREX3K` |
| RAF Referrer | `{jurisdiction}AQ{total_spins:04d}RFER{XXX}` | `MIAQ0500RFERX3K` |
| RAF Day 2–N | `{jurisdiction}AQ{total_spins:04d}DAY{n}{XXX}` | `MIAQ0500DAY2X3K` |
| SUO Day 2–N | None — no voucher codes | — |

> ⚠️ Voucher codes in NATS can **never** be reused, even if the segment, promotion, and bonus they were attached to have since been deleted.

> ⚠️ For RAF RFRE and RFER, the voucher code is generated **once** at script startup and reused for all modal entries. Day 2+ vouchers are generated once per day code at startup and used once only (no modal).

### RTC Free Spins — Code Name Fields
- **{jurisdiction}:** MI, NJ, WV, PA only
- **{spins}S / {value}V / {game}:** e.g. `10S_20V_TCE`

### Image Filenames
All offer types use: `Promo Detail.png`, `Masthead.png`
- RTC CC discover: `{amount}.png`
- RTC FS discover: `{spins}S_{value}V_{game}.png`
- BG discover: `B{bet}_G{get}.png`
- BG-RE discover: `B{bet}_G{get}.png`
- DM discover: `{pct}M{max}.png`
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

### Regions / Jurisdictions
- RTC CC / BG / BG-RE / DM: US (default), MI, WV, PA, NJ, ON, AB, CA
- RTC FS / RAF / SUO: MI, NJ, WV, PA only

---

## localStorage Keys

| Key | What it stores |
|---|---|
| `rtc_hiw` | Custom How It Works — RTC CC |
| `rtc_tc` | Custom T&C — RTC CC |
| `rtc_image_path` | Image folder path — RTC CC |
| `rtc_terms_ids` | JSON object mapping amount → Terms ID — RTC CC |
| `rtc_terms_start_date` | Promotional start date in `MM/DD/YY` format — RTC CC |
| `rtc_terms_end_date` | Promotional end date in `MM/DD/YY` format — RTC CC |
| `rtc_fs_hiw` | Custom How It Works — RTC FS |
| `rtc_fs_tc` | Custom T&C — RTC FS |
| `rtc_fs_image_path` | Image folder path — RTC FS |
| `bg_hiw` | Custom How It Works — BG |
| `bg_tc` | Custom T&C — BG |
| `dm_hiw` | Custom How It Works — DM |
| `dm_tc` | Custom T&C — DM |
| `raf_image_path` | Image folder path — RAF |

SUO Day 2+ Spins and BG-RE have no localStorage keys.

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
| 1 | BG-RE | NATS and Playmaker URLs are currently pointed at the **test environment**. Switch to production URLs once the Rewards Engine is live in the Playmaker production environment. |
| 2 | BG-RE | Free Spins reward type not yet implemented — different field structure in Playmaker. |

---

## Image System

### RTC Top Up - Casino Credit
Path stored in `localStorage` as `rtc_image_path`. Edit Images button present.
**Default path (Adrian's machine):** `.../Marketing Automations/Lifecycle Automations/RTC Top Up`
**Google Drive folder:** https://drive.google.com/drive/folders/1Dlpa3xZTHjzlwHgayPSS-MZJ8-DQTmhh

> ⚠️ Each user must set their own image path via the **Edit Images** button before generating a script. The path is saved to their browser's localStorage and only needs to be set once per machine.

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

---

## Sidebar Navigation

All generated scripts navigate between screens by hovering sidebar icons using `data-menu-id` attribute selectors and clicking the target submenu item via `dispatch_event("click")`.

| Screen | Sidebar selector | Submenu item |
|---|---|---|
| Account Segments | `[data-menu-id$='-segments']` | `Account Segments` |
| Promos | `[data-menu-id$='-cms']` | `Promos` |
| Bonus Manager | `[data-menu-id$='-bonuses']` | `Bonus Manager` |

Each nav step moves the mouse to a safe zone (600, 400), hovers the sidebar icon, then uses `dispatch_event("click")` on the submenu item. Up to 5 retry attempts with safe-zone reset. Applies to all offer types that use NATS. BG-RE uses a separate Playmaker navigation flow (see build notes).

---

## Per-Code Step Toggles

### RTC CC / RTC FS / BG / DM
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
**Promo** and **Bonus** badges only — no Segment badge. The segment is created inside Playmaker as part of the bonus flow. Both phases always run unless a badge is toggled off.

---

## Eastern Time Enforcement

All generated scripts enforce Eastern Time (America/New_York) via Python's `zoneinfo` module. **Python 3.9+ is required.**

Playmaker times are entered directly as ET with no timezone conversion needed — the platform displays and stores all times in ET natively.

---

## Platform Tags — Bonus Creation

NATS pre-populates platform tags on every new bonus. All generated scripts remove `COMBO: Sportsbook` and `COMBO: Sportsbook And Casino`. Removing either of these also causes NATS to automatically remove `All`. The resulting active platforms on every bonus are:

- `STAC: Standalone Casino`
- `Web`
- `COMBO: Casino`

This applies to all six NATS offer types. BG-RE bonuses are built in Playmaker, not NATS, so platform tags do not apply.

---

## RTC Top Up - Casino Credit

### Day Card Layout
- **Default days:** 9 | **Default amounts:** 2, 4, 5, 10, 20, 40, 50, 100, 200, 400, 800

### Promo & Bonus Activation Window
Both promos and bonuses use a **4 AM ET start, 24-hour window**:
- **Start:** Day-of at 04:00:00 Eastern
- **End:** Following day at 04:00:00 Eastern (i.e. `start + timedelta(hours=24)`)

> ⚠️ This differs from all other offer types (BG, DM, RTC FS, RAF, SUO), which use a midnight-to-midnight (00:00:00 ET) window.

### Terms IDs & Promotional Dates
**Current window defaults:** Start: `06/01/26` | End: `12/01/26` | IDs: `$2→CAS_9461` through `$800→CAS_9471`

### Bonus Creation — Fixed Values
| Field | Value |
|---|---|
| Trigger type | Opt In |
| Stake Chunk Sizes | 1 |
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

### Bonus Creation — Fixed Values
| Field | Value |
|---|---|
| Trigger type | Opt In |
| Free Spin's CTA | Play! |
| Bet Level | default |
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

### Campaign Structure
- NUM_DAYS must be 5 or 10 — enforced with `assert NUM_DAYS in (5, 10)` at script startup
- Codes generated: DAY2 through DAYN (NUM_DAYS − 1 total codes)

### Script Execution Order
1. **Segments** — DAY2–N
2. **Bonuses** — DAY2–N

### Bonus Key Facts
- Trigger: **No Opt In**
- Internal name = code name; description blank
- Refer A Friend Bonus checkbox — **not touched**
- Send To XP → checked
- No voucher code
- Free Spin's Description = `"Sign Up Offer Bonus Spins Day N"`
- Number of Spins = per-day amount
- Days: 0 / 0 / 1 (Opt In / Entitlement / Settlement)
- Status Active → **checked**
- Activation: code date minus 1 day midnight ET → year 2041
- Series = **Early Life** (selected by `title` attribute, not text search)
- No promotion tile, no confirmation modal, no post-save edit pass

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

Fully integrated into the HTML tool as of v2.15. See `BG_Rules_Engine_Build_Notes.md` for full selector reference, Playmaker field spec, and development history.

> ⚠️ Currently using **test environment** URLs. Switch before using in production:
> - NATS: change `trading.test1.fanatics.bet` → `trading.1.betfanatics.com`
> - Playmaker: confirm and update prod URL once Rewards Engine is live

### Overview
Two-phase script:
- **Phase 1 (NATS):** Builds Promo only — identical to existing BG promo flow including ZIP images
- **Phase 2 (Playmaker):** Builds combined segment + bonus at `https://playmaker-internal.test1.fanatics.bet/`

### Playmaker Navigation
`Login (SSO) → iCasino → Bonus Setup → Create Bonus → Steps 1–6 → Save Bonus`

### Day Card Fields
| Field | Default |
|---|---|
| Date (MMDDYY) | — |
| Images (ZIP per card) | — |
| Offers (Bet/Get rows) | B100/G2, B200/G5, B2000/G50, B20000/G500 (all inactive) |
| Reward Type | FanCash (FC) |
| Jurisdiction | USA |
| Days to Use Reward | 7 |
| Show Toast | On |
| Send to XP | On |

### Reward Types
| Selection | Code segment | Playmaker value | Status |
|---|---|---|---|
| FanCash | `FC` | `fan_cash` | ✅ Production |
| Casino Credit | `CC` | `casino_credit` | ✅ Production |
| Free Spins | `FS` | `free_spin` | ❌ Not yet implemented |

### Jurisdictions
| Selection | Playmaker checkboxes |
|---|---|
| USA | MI, WV, NJ, PA |
| Canada | ON |
| MI / NJ / WV / PA / ON | Individual state only |

### Key Confirmed Selectors
| Element | Selector |
|---|---|
| Calendar picker buttons | `button.dc-w-\[280px\]` index 0/1 |
| Outside (prev/next month) day buttons | class contains `dc-day-outside` |
| Start time field | `get_by_role("textbox", name="Time").nth(0)` |
| End time field | `get_by_role("textbox", name="Time").nth(1)` |
| Time set method | JS `nativeInputValueSetter` + `input`/`change` events |
| Promotion dropdown result | `page.get_by_label(name).get_by_text(name)` |
| Step buttons | `button:has-text('Step N: Label')` |
| Save Bonus | `button:has-text('Save Bonus')` |

---

## Script Execution Notes

### Create Promotion Button Poll
All five `create_promo` functions poll the "Create Promotion" button up to 60 seconds (120 × 500ms) before clicking. SUO and BG-RE do not create promos in NATS via this flow.

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

### Promotion Tile Selection
Uses regex anchored exact match scoped to `.ant-select-dropdown:not(.ant-select-dropdown-hidden)`. Applied to all offer types that use a Promotion Tile. SUO and BG-RE do not use a Promotion Tile in NATS.

### Create Bonus Button — Multi-Code Runs
`create_bonus` polls up to 120 × 500ms (60 seconds) for the button to re-enable.

### AMELCO Dropdown Selection
All `create_segment` functions select AMELCO via `dispatch_event("click")` rather than `.click()` to bypass Ant Design pointer interception on the dropdown option container. A 2-second wait is inserted after `#forBonus` is checked and before OK is clicked, giving NATS time to register the Region selection.

### Image Upload Timing
After file selection, scripts wait **8 seconds** before clicking OK, then 2 seconds after OK. Each image upload takes ~10 seconds total; 3 images per promo = ~30 seconds per promo build. Applies to RTC CC, RTC FS, BG, DM, RAF, and BG-RE. SUO has no images.

---

## Refer a Friend (RAF)

RAF is the 5th offer type. Fully integrated into the HTML tool as of v2.8.

### Campaign Card Layout
Single card (full width, two-column layout). Left column: Date, Jurisdiction, Game, Spins/Day, Spin Value, Bet Amount, Number of Days toggle (5/10), phase badges. Right column: live preview panel showing derived code names and totals.

### Build Status

| Code Type | Segment | Promo | Bonus | Status |
|---|---|---|---|---|
| Referee (RFRE) | ❌ Not needed | ✅ Confirmed | ✅ Confirmed | ✅ Production |
| Referrer (RFER) | ✅ Confirmed | ✅ Confirmed | ✅ Confirmed | ✅ Production |
| Day 2–N | ✅ Confirmed | ❌ Not needed | ✅ Confirmed | ✅ Production |

### Campaign Structure
- Total spins = spin amount × number of days
- **NUM_DAYS must be 5 or 10** — enforced with `assert NUM_DAYS in (5, 10)` at script startup

### Script Execution Order
1. **Segments** — RFER + DAY2–N
2. **Promos** — RFRE then RFER
3. **Day 2+ Bonuses** — DAY2–N loop
4. **RFRE Bonus + Post-Save Edit**
5. **RFER Bonus + Post-Save Edit**

Each phase is skipped if its badge is off.

### Referee (RFRE) — Bonus Key Facts
- Internal name AND description = `"Refer a Friend: Bet ${bet_amount}, Get {total_spins} Spins"`
- Refer A Friend Bonus = checked; Referee radio selected; Status Active = **checked**
- Number of Spins = **per-day amount**
- Post-save edit: nav to Bonus Manager → 30s wait → uncheck Active Only → Reload → 20s wait → search → edit → resave

### Referrer (RFER) — Key Differences from Referee

| Field | Referee | Referrer |
|---|---|---|
| Send to XP | Not clicked | Clicked ON |
| RAF radio | `value="REFEREE"` | `value="REFERRER"` |
| Number of Spins | Per-day amount | **Total spins** |
| Entitlement block | Present | **Absent** |
| Days to Meet Freespin Settlement | 1 | **7** |
| Status Active | checked | **unchecked** |
| Post-save edit Reload wait | 20 seconds | **120 seconds** |

### Day 2–N — Key Facts
- Internal name = code name; description blank; Refer A Friend Bonus unchecked
- Send To XP checked; per-day spins; no entitlement block; no Promotion Tile; Status Active unchecked
- Clean save — no voucher modal, no post-save edit pass

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

### RAF Outstanding Items

| # | Area | Status |
|---|---|---|
| 7 | HIW & T&C copy | Pending legal approval |
| 8 | Confirmation modal after promo Save | Not yet confirmed |

---

## How It Works & T&C Copy

### RTC CC
Dynamic variables: `{amount_fmt}`, `{terms_id}`, `{start_date_short}`, `{start_date_long}`, `{end_date_short}`, `{end_date_long}` — saved as `rtc_hiw` / `rtc_tc`

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
