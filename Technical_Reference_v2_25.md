# NATS Bonus Creator — Technical Reference v2.25

## Changes from v2.17

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

- **Left column (340px fixed):** Branding, primary action buttons, Offer Type selector, Region selector (hidden for RTC FS, RAF, SUO, and BGRE; restricted to US / CA for RTC CC as of v2.22; locked to CA — Canada for VIPDM as of v2.23; restricted to US / CA for VIPBG as of v2.25)
- **Right column (flex):** Setup guide link panel, offer-type-specific info panel, and run instruction cards
- **Below header:** Day cards grid (or single campaign card for RAF and SUO)
- **Below day cards:** Generated script output area (hidden until script is generated)

---

## Global Controls

**Generate Script (white button)**
Reads all day card inputs, builds internal code names, generates and auto-downloads a Python script. For RAF and SUO: enabled when date (6 chars), Spins/Day > 0, Bet Amount > 0, and Spin Value are all filled.

**Edit HIW / Edit T&Cs**
Hidden for RAF, SUO, and BG-RE (copy pending legal approval or not applicable). Active for RTC CC, RTC FS, BG, DM, LC-REACT, LC-CHURN-DM, VIPDM, and VIPBG. For RTC CC, Edit T&Cs is **jurisdiction-aware** (v2.22): it reads and saves the selected region's T&C (`rtc_tc` vs `rtc_tc_ca`) and displays the region in the modal title. Edit HIW is shared across US and CA (single `rtc_hiw` key) by design. For **VIPDM** (v2.23), the Edit T&Cs modal shows the Terms bar with an **Edit Terms Expiry & Terms IDs** button routed to the VIPDM terms modal — promotional start/end dates (`{start_date_long}` / `{end_date_long}`) plus one Terms ID input per offer key `M{min}_{pct}M{max}`. For **VIPBG** (v2.24/v2.25), the same Terms bar routes to the VIPBG terms modal — promotional start/end dates plus one Terms ID input per offer key (CA: `{SBG|TBG}_B{bet}_G{get}`; US: `B{bet}_G{get}`). As of v2.25 all VIPBG modals are **region-aware**: Edit HIW, Edit T&Cs, the Terms modal, and Edit Images read/save the selected region's keys (`vcl_bg_*` for CA, `vcl_bg_*_us` for US) and show the region in their titles — note VIPBG HIW is region-scoped, unlike RTC CC where HIW is shared.

**Edit Images**
Present for RTC CC, RTC FS, RAF, LC-REACT, LC-CHURN-DM, VIPDM, and VIPBG. Hidden for BG, DM, SUO, and BG-RE. For RTC CC, jurisdiction-aware (v2.22): reads/saves `rtc_image_path` vs `rtc_image_path_ca`, shows the region in the modal title, and the Open Shared Images Folder link switches between the US and Canadian Google Drive folders. For VIPBG, jurisdiction-aware the same way as of v2.25 (`vcl_bg_image_path` vs `vcl_bg_image_path_us`, region in title, Drive link follows region).

**+ Add Day**
No-op for RAF, SUO, VIPDM, and VIPBG (single card each). Max 8 days for BG/DM, 99 for RTC CC/FS/LC-REACT/LC-CHURN-DM.

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

### VIP Offer Library - Deposit Matches (v2.23)
Single card (full width, max 620px). Title: **VIP Offer Library — CA — Canada**. `+ Add Day` is a no-op.
Date (MMDDYY, free-typed — see window rules below), offers list (**Min → % → Max** rows with per-offer checkbox — nine defaults shown unselected: $100/50%/$250, $100/100%/$250, $250/10%/$250, $250/10%/$2,500, $250/10%/$5,000, $250/20%/$500, $250/20%/$1,000, $250/20%/$2,000, $250/20%/$5,000 — each row displays its Terms ID inline, red `no ID` when unmapped), + Add / Select All, step badges (Segment/Promo/Bonus), Send to XP (default ON), External Bonus (default OFF), Days to Opt In (3) / Days to Entitlement (3 — **extends the promo/bonus end date**) / Days to Settlement (7). Region selector locked to **CA — Canada**. No Campaign Name field and no ZIP attachment — Title, Promo Header, and Bonus Description are fixed as `From Your VIP Host`, and images come from a locally-synced folder.

**Terms ID enforcement:** identical to Churn DM — each active offer key `M{min}_{pct}M{max}` must exist in the VIPDM Terms ID map (default `CAS_CA_0078`–`CAS_CA_0086`, editable via the Edit T&Cs modal's Terms bar). Active offers without an ID show a red row warning and block script generation with an alert.

### VIP Offer Library - Bet & Gets (v2.24 CA / v2.25 US)
Single card (full width, max 680px). Title: **VIP Offer Library — Bet & Gets — {region}** (region-aware as of v2.25). `+ Add Day` is a no-op. Region selector offers **US / CA** (v2.25; keeps a valid current selection, otherwise defaults to CA); **switching region silently resets all card values** and the edit buttons render **red for CA, blue for US**.
Date (MMDDYY, free-typed — same month-anchored window rules as VIPDM, both regions), offers list, + Add / Select All, step badges (Segment/Promo/Bonus), Days to Opt In (3) / Days to Entitlement (3 — **extends the promo/bonus end date**) / Days to Settlement (7). **Send to XP and External Bonus have no toggles — both are always ON** (static note on the card; both regions). No Campaign Name field and no ZIP attachment — Title, Promo Header, and Bonus Description are fixed as `From Your VIP Host`, and images come from a locally-synced folder (one per region).

**CA offers:** **Category → Bet → Get** rows with a per-row Slots/Tables dropdown and per-offer checkbox — eight defaults shown unselected: Slots $1,000/$100, $5,000/$500, $20,000/$1,000, $100,000/$5,000; Tables $2,000/$100, $10,000/$500, $25,000/$1,000, $100,000/$2,500 — each row displays its Terms ID inline, red `no ID` when unmapped.

**US offers (v2.25):** **Bet → Get** rows only — no category dropdown; all US tiers are slots B&Gs. Four defaults shown unselected: $1,000/$100, $5,000/$500, $20,000/$1,000, $100,000/$5,000 — Terms IDs `CAS_10687`–`CAS_10690` inline.

**Terms ID enforcement (both regions):** identical to VIPDM — each active offer key must exist in the selected region's VIPBG Terms ID map (CA: `{SBG|TBG}_B{bet}_G{get}`, defaults `CAS_CA_0069`–`CAS_CA_0076`; US: `B{bet}_G{get}`, defaults `CAS_10687`–`CAS_10690`; both editable via the Edit T&Cs modal's Terms bar). Active offers without an ID show a red row warning and block script generation with an alert. CA keying includes the category prefix as collision insurance (a Slots and Tables tier could otherwise share a `B{bet}_G{get}` shape); US keys need no prefix since there is a single category.

---

## RAF Phase Badges

| Badge | Phases covered |
|---|---|
| Day 2+ | Phase 1 (DAY2–N segments) + Phase 3 (DAY2–N bonuses) |
| Referee | Phase 2 (RFRE promo) + Phase 4 (RFRE bonus + edit) |
| Referrer | Phase 1 (RFER segment) + Phase 2 (RFER promo) + Phase 5 (RFER bonus + edit) |

Phase 1 skipped if Day 2+ and Referrer both off. Phase 2 skipped if Referee and Referrer both off.

---

## Per-Code Step Toggles (RTC CC / FS / BG / DM / LC-REACT / LC-CHURN-DM / VIPDM / VIPBG)

Each day card shows three toggleable step badges: **Segment**, **Promo**, **Bonus**. The generated script includes `DO_SEGMENT`, `DO_PROMO`, and `DO_BONUS` boolean arrays. `main()` skips a phase entirely if the filtered name list for that phase is empty.

SUO Day 2+ has no step toggles — segment and bonus are always built for every code.

BG-RE shows **Promo and Bonus** badges only — no Segment badge. The segment is created inside Playmaker as part of the bonus flow.

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

**VIPDM (v2.23):** `VRDMCC{MMDDYY}{XXX}` — **V**IP + **R**etention + **DM** + **CC** (reward); mirrors standard DM's `CRDMFC` grammar with the vertical letter and reward token swapped.

**VIPBG (v2.24):** `CLBGFC{MMDDYY}{XXX}` — **shared with standard BG and BG-RE** (NATS-mandatory but unused for this offer type; the code-name category token distinguishes the offers).

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

### RTC CC / FS / BG / DM / LC-REACT / LC-CHURN-DM / VIPDM / VIPBG
1. Open Chromium, navigate to trading platform
2. Pause for manual login
3. **Segments** — hover `-segments` sidebar icon, click "Account Segments", create segments
4. **Promos** — hover `-cms` sidebar icon, click "Promos", create promos
5. **Bonuses** — hover `-bonuses` sidebar icon, click "Bonus Manager", create bonuses
6. macOS notification + close browser

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
14. Summary percentage field — tries `Bonus %` then `Casino Credit %` label
15. Bonus amount summary card — tries `Bonus` then `Casino Credit` card title; fill with `{max}` from code into the jurisdiction's currency row
16. Minimum Deposit = **dynamic** `{min}` from code (`M{min}` segment) — jurisdiction currency row
17. Maximum Deposit = `1000000` (hardcoded) — jurisdiction currency row
18. Status Active = checked
19. Activation: `00:00:00 ET` day-of → `00:00:00 ET` day+10 (identical to promo window)
20. Segment / Client Profiling — anchored regex scoped to `.ant-select-dropdown:not(.ant-select-dropdown-hidden)`
21. Promotion Tile — anchored regex exact match
22. Reporting Platform: `CAS (Casino)`
23. Bonus Origin: `Retention`
24. Series: `Other` (typed search)
25. Type: `Lifecycle`
26. Save — **no confirmation modal**

### Fixed Values

| Field | Value |
|---|---|
| Trigger | Opt In |
| Description | Deposit and Get Casino Credit |
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

## Script Steps — VIP Offer Library - Deposit Matches (v2.23)

CA — Canada only. Every code carries the `CA` token, so all currency-table fills land in the **CAD** row via the v2.18 helpers.

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
5. Remove **all three** COMBO platform tags — `COMBO: Sportsbook`, `COMBO: Sportsbook And Casino`, **and `COMBO: Casino`** — leaving **`Web` + `STAC: Standalone Casino` only** (unique to this offer type)
6. Voucher Code: `VRDMCC{MMDDYY}{XXX}`
7. Bonus type: `Casino Credit`
8. Check **`Casino Credit (%)`** checkbox
9. Fill percentage input (`aria-valuemin='0' aria-valuemax='100'`) with `{pct}` from code
10. Days to Meet Casino_credit Settlement = per-card value (default 7)
11. Days to Opt In = per-card value (default 3)
12. Days to Meet Entitlement = per-card value (default 3 — also drives the end dates)
13. Entitlement Type: `Deposit` (JS radio click)
14. **Matched Deposit checkbox — always checked** (lives inside the DEPOSIT entitlement panel; explicit visibility wait before clicking since the panel renders after step 13)
15. Summary percentage field — tries `Bonus %` then `Casino Credit %` label
16. Bonus amount summary card — tries `Bonus` then `Casino Credit` card title; fill with `{max}` from code into the **CAD** row
17. Minimum Deposit = dynamic `{min}` from code (`M{min}` segment) — CAD row
18. Maximum Deposit = `1000000` (hardcoded) — CAD row
19. Status Active = checked
20. Activation: per `parse_dates(name, DAYS_TO_ENTITLEMENT[idx])` (identical to promo window)
21. Segment / Client Profiling — anchored regex scoped to `.ant-select-dropdown:not(.ant-select-dropdown-hidden)`
22. Promotion Tile — anchored regex exact match
23. Reporting Platform: `CAS (Casino)`
24. Bonus Origin: `Opt-In Bonus`
25. Series: `VIP` — typed search + anchored regex `^VIP$` scoped to the **visible** dropdown
26. Type: `Retention` — anchored regex `^Retention$` scoped to the **visible** dropdown. ⚠️ This scoping is load-bearing: the (closed, hidden) Bonus Origin dropdown still contains a `Retention` option in the DOM, so a page-wide `get_by_text("Retention", exact=True)` would match two elements and crash with a Playwright strict mode violation. VIPDM is the only offer type that selects `Retention` for Type.
27. Save — **no confirmation modal**

### Fixed Values

| Field | Value |
|---|---|
| Trigger | Opt In |
| Description | From Your VIP Host |
| Matched Deposit | Checked |
| Platforms | Web + STAC: Standalone Casino only |
| Minimum Deposit | From code (`M{min}`) — CAD row |
| Maximum Deposit | 1000000 — CAD row |
| Status Active | Checked |
| Reporting Platform | CAS (Casino) |
| Bonus Origin | Opt-In Bonus |
| Series | VIP |
| Type | Retention |
| Confirmation modal | None |

Note: the `RET` token in the code name mirrors the **Type** field (`Retention`), not Bonus Origin — unlike standard DM, where `RET` mirrors Origin.

### Activation / Promo Window

| | VIPDM |
|---|---|
| Start | `00:00:00 ET` typed date − 1 day |
| End | `23:59:59 ET` last day of typed date's month + Days to Meet Entitlement |

Promo window matches bonus activation window exactly.

### VIPDM T&C Data (Baked In at Generation Time)

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

**HIW** (no opt-in deadline line — the promo window bounds the offer; no "See below" trailer):
```
1. Opt-in to the promotion
2. Make a single deposit of {min_fmt} or more
3. We'll instantly match your deposit {pct}%, up to {max_fmt} Casino Credit
```

---

## Script Steps — VIP Offer Library - Bet & Gets (v2.24 CA / v2.25 US)

Dual region as of v2.25. `buildVIPBGScript` reads the region at generation time (`isVIPBGUS()`) and injects region-specific chunks; the CA output is functionally byte-identical to v2.24. CA codes carry the `CA` token → **CAD** row; US codes carry the `US` token → **USD** row (v2.18 helpers). **Both regions: the wager requirement ("bet") is enforced upstream by XP / segment logic — it is never entered on the bonus.** The bonus is a flat reward equal to the "get" value — **Casino Credit for CA, FanCash for US**.

### `parse_dates(code, entitlement_days)` — month-anchored window
Identical to VIPDM:
- **Start:** typed code date **− 1 day** at `00:00:00 ET`
- **End:** **last day of the typed date's calendar month** at `23:59:59 ET` **+ `entitlement_days`** (per-card Days to Meet Entitlement, injected as the `DAYS_TO_ENTITLEMENT` array)

`calendar.monthrange` handles month lengths and leap years. The typed day anchors only the start (`083126` and `080126` share an end date); a typed 1st starts in the prior month (`010127` starts Dec 31, 2026 — intended). Changing Days to Entitlement changes the promo **and** bonus end date. Promo window = bonus activation window.

### `parse_vipbg(code)`
**CA:** returns `(cat, bet, get)` from `MMDDYY_VCL_RET_{SBG|TBG}_CA_CC_B{bet}_G{get}` — the category token is matched against the literal set `("SBG", "TBG")`; bet/get from `B`/`G`-prefixed all-digit tokens (so `VCL`, `RET`, `CA`, `CC` are skipped).
**US (v2.25):** returns `(bet, get)` from `MMDDYY_VCL_RET_BG_US_FC_B{bet}_G{get}` — no category token (the `BG` and `FC` tokens are skipped by the same all-digit guard).

### `GAME_CATEGORIES` map (CA only)
`SBG` → HIW `Slots`, T&C `slots` / `SLOTS`; `TBG` → HIW `Tables`, T&C `table` / `TABLES`. The lowercase asymmetry ("slots games" but "table games") matches the legal documents verbatim. **US scripts contain no `GAME_CATEGORIES` machinery** — all US tiers are slots, and "Slot games" is static copy in the US HIW.

### `create_segment(page, name)`
Identical to RTC CC / DM / Churn DM / VIPDM — AMELCO `dispatch_event("click")`, 2-second post-`#forBonus` wait.

### `create_promo(page, name, idx)`
Poll Create Promotion (120 × 500ms, up to 60s) → click → fill name → Start/End per `parse_dates(name, DAYS_TO_ENTITLEMENT[idx])` → Type: `Image only CTA` → Layout: `Overlay` → first Save → upload `Promo Detail.png`, `Masthead.png`, and the Discover image (CA: `{SBG|TBG}_B{bet}_G{get}.png`; US: `B{bet}_G{get}.png`) from the region's `IMAGE_FOLDER` → Promo Header Text: `From Your VIP Host` → Title: `From Your VIP Host` → Bonus Tile toggle ON → **Button 2: region-branched (v2.25)** — CA leaves label/link **EMPTY** (terminal prints `-- Button 2: left empty`); US **fills `Play Now!` → `/docs/usered/casgenericgamelist`** via `BUTTON2_LABEL` / `BUTTON2_LINK` constants → How it works (CA template with `{bet_fmt}` / `{get_fmt}` / `{game_category}`; US template with `{bet_fmt}` / `{get_fmt}` only) → T&C (CA: single Canadian legal template covering both categories; US: single canonical FanCash template — both baked in at generation time) → second Save. No confirmation modal.

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
| Code name | `MMDDYY_VCL_RET_{SBG\|TBG}_CA_CC_B{bet}_G{get}` | `MMDDYY_VCL_RET_BG_US_FC_B{bet}_G{get}` |
| Offer rows | Category (Slots/Tables) → Bet → Get, 8 defaults | Bet → Get, 4 defaults (all slots) |
| Terms ID keys | `{SBG\|TBG}_B{bet}_G{get}` → `CAS_CA_0069`–`0076` | `B{bet}_G{get}` → `CAS_10687`–`10690` |
| Promotional dates (defaults) | `08/17/26 → 12/31/26` | `07/01/26 → 12/31/26` |
| Image folder default | `.../VIP Automations/Offer Library/Canada/Bet & Gets` | `.../VIP Automations/Offer Library/USA/Bet & Gets` |
| Drive link in Edit Images | `folders/1tkMs1dx-gszSGzlBHxgMYvOYv2l-2UjO` | `folders/13neRGeYM4JLa98p827vbepp-eKu2_hXq` |
| Discover filename | `{SBG\|TBG}_B{bet}_G{get}.png` | `B{bet}_G{get}.png` |
| Button 2 (promo) | Empty | `Play Now!` → `/docs/usered/casgenericgamelist` |
| Platform tags | All 3 COMBO removed (Web + STAC only) | 2 removed, **COMBO: Casino kept** |
| Settlement label | `Days to Meet Casino_credit Settlement` | `Days to Meet Fancash Settlement` |
| Confirmation modal | None observed | Expected (FanCash) — defensive either way |
| HIW / T&C keys | `vcl_bg_hiw` / `vcl_bg_tc` | `vcl_bg_hiw_us` / `vcl_bg_tc_us` |
| Edit button theme | Reds via `body.ca-region` | Blues (default) |

Everything not listed is shared: month-anchored window (`parse_dates`), `From Your VIP Host` copy, Send To XP + External Bonus always ON, `CLBGFC{MMDDYY}{XXX}` voucher, Series `VIP` / Type `Retention` / Origin `Opt-In Bonus` / Reporting CAS, Days 3/3/7, step badges, Terms ID enforcement, conditional Stake Chunk fill, and image upload timing.

### Region change behavior

`onJurisdictionChange()` refreshes the red/blue button theme and, when the offer type is RTC **or VIPBG**, calls `resetAll()` — clearing every day card and hiding any generated script; silent by design, no confirmation dialog. `updateRegionTheme()` applies `body.ca-region` when `isRTCCanada()` **or** VIPBG-with-CA is true; the added condition is provably inert for every other offer type.

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
| `vcl_bg_hiw_us` | Custom How It Works — VIPBG (US — United States) |
| `vcl_bg_tc_us` | Custom T&C — VIPBG (US — United States) |
| `vcl_bg_image_path_us` | Image folder path — VIPBG (US — United States) |
| `vcl_bg_terms_ids_us` | JSON object mapping offer key → Terms ID — VIPBG (US) |
| `vcl_bg_terms_start_date_us` | Promotional start date in `MM/DD/YY` format — VIPBG (US) |
| `vcl_bg_terms_end_date_us` | Promotional end date in `MM/DD/YY` format — VIPBG (US) |

The base `vcl_bg_*` keys hold the CA — Canada values (v2.25). SUO Day 2+ Spins and BG-RE have no localStorage keys.

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

### VIP Offer Library - Deposit Matches (v2.23)
Path stored in `localStorage` as `vcl_dm_image_path`. Edit Images button present.
**Default path (Adrian's machine):** `.../Marketing Automations/VIP Automations/Offer Library/Canada/Deposit Matches` (first offer type under the `VIP Automations` shared drive branch)
**Google Drive folder:** https://drive.google.com/drive/folders/1v9DEVtCNQPCgUkJsWtsXn7UJVIyBYtr4
**Image filenames:** `Promo Detail.png`, `Masthead.png`, `{pct}M{max}.png`. Min deposit is not part of the Discover filename. Full default-tier set: `50M250.png`, `100M250.png`, `10M250.png`, `10M2500.png`, `10M5000.png`, `20M500.png`, `20M1000.png`, `20M2000.png`, `20M5000.png`.
> ⚠️ `10M250.png` and `100M250.png` differ by one character — double-check those two creatives land in the correct files; a mislabeled file uploads the wrong tier's image without erroring.

### VIP Offer Library - Bet & Gets (v2.24 CA / v2.25 US)
Region-scoped as of v2.25. Path stored in `localStorage` as `vcl_bg_image_path` (CA) / `vcl_bg_image_path_us` (US). Edit Images button present; the modal reads/saves the selected region's path, shows the region in its title, and its Drive link follows the region. Under the **VIP Automations** shared drive branch, sibling to Deposit Matches.
**CA default path (Adrian's machine):** `.../Marketing Automations/VIP Automations/Offer Library/Canada/Bet & Gets`
**CA Google Drive folder:** https://drive.google.com/drive/folders/1tkMs1dx-gszSGzlBHxgMYvOYv2l-2UjO
**CA image filenames:** `Promo Detail.png`, `Masthead.png`, `{SBG|TBG}_B{bet}_G{get}.png` — the Discover filename is the full offer key (e.g. `SBG_B1000_G100.png`, `TBG_B100000_G2500.png`), so no one-character ambiguity. Default-tier set: `SBG_B1000_G100.png`, `SBG_B5000_G500.png`, `SBG_B20000_G1000.png`, `SBG_B100000_G5000.png`, `TBG_B2000_G100.png`, `TBG_B10000_G500.png`, `TBG_B25000_G1000.png`, `TBG_B100000_G2500.png`.
**US default path (Adrian's machine):** `.../Marketing Automations/VIP Automations/Offer Library/USA/Bet & Gets`
**US Google Drive folder:** https://drive.google.com/drive/folders/13neRGeYM4JLa98p827vbepp-eKu2_hXq
**US image filenames:** `Promo Detail.png`, `Masthead.png`, `B{bet}_G{get}.png` (no category prefix — all US tiers are slots). Default-tier set: `B1000_G100.png`, `B5000_G500.png`, `B20000_G1000.png`, `B100000_G5000.png`.

---

## Naming Conventions

### Internal Code Names

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
| VIPBG (CA) | `MMDDYY_VCL_RET_{SBG\|TBG}_CA_CC_B{bet}_G{get}` | `090126_VCL_RET_SBG_CA_CC_B1000_G100` |
| VIPBG (US) | `MMDDYY_VCL_RET_BG_US_FC_B{bet}_G{get}` | `090126_VCL_RET_BG_US_FC_B1000_G100` |

### Voucher Codes

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
| VIPBG (US & CA) | `CLBGFC{MMDDYY}{XXX}` — shared with BG/BG-RE | `CLBGFC090126X4T` |
| RAF Referee | `{jurisdiction}AQ{total_spins:04d}RFRE{XXX}` | `MIAQ0500RFREX3K` |
| RAF Referrer | `{jurisdiction}AQ{total_spins:04d}RFER{XXX}` | `MIAQ0500RFERX3K` |
| RAF Day 2–N | `{jurisdiction}AQ{total_spins:04d}DAY{n}{XXX}` | `MIAQ0500DAY2X3K` |
| SUO Day 2–N | None — no voucher codes | — |

### Generated Script Filenames

| Offer | Filename |
|---|---|
| RTC CC | `rtc_top_up_MMDDYY_HHMM.py` |
| RTC FS | `rtc_fs_MMDDYY_HHMM.py` |
| BG | `bg_MMDDYY_HHMM.py` |
| BG-RE | `bg_re_MMDDYY_HHMM.py` |
| DM | `dm_MMDDYY_HHMM.py` |
| RAF | `raf_full_campaign_MMDDYY_HHMM.py` |
| SUO | `suo_day2_MMDDYY_HHMM.py` |
| LC-REACT | `lc_react_MMDDYY_HHMM.py` |
| LC-CHURN-DM | `lc_churn_dm_MMDDYY_HHMM.py` |
| VIPDM | `vcl_dm_MMDDYY_HHMM.py` |
| VIPBG | `vcl_bg_MMDDYY_HHMM.py` |

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

**VIP Offer Library exception (v2.23/v2.24):** VIPDM and **CA** VIPBG remove **all three** COMBO tags — `COMBO: Sportsbook`, `COMBO: Sportsbook And Casino`, and `COMBO: Casino` — leaving only `Web` + `STAC: Standalone Casino`. They are the only flows that remove `COMBO: Casino`. **US VIPBG (v2.25) follows the standard pattern** — removes only the two Sportsbook tags and keeps `COMBO: Casino`.

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

| Jurisdiction tokens in code name | Currency row |
|---|---|
| `US`, `MI`, `WV`, `PA`, `NJ` | `USD` |
| `ON`, `AB`, `CA` | `CAD` |
| — | `GBP` is **never used under any circumstance** |

### Shared helpers (embedded in all eleven generated script templates)

- `currency_for(name)` — scans the code name's `_`-separated tokens for `ON` / `AB` / `CA`; returns `"CAD"` if found, else `"USD"`. GBP is unreachable by construction.
- `fill_currency_amount(page, name, value, scope=None, label=...)` — fills the value into `tr[data-row-key='{currency}'] input.ant-input-number-input` (optionally scoped to a `.summaryCell` card). If no currency rows exist, falls back to the legacy single-input selector (`.ant-table-cell.center .ant-input-number-input`), so scripts keep working if NATS reverts. Prints the currency (or `legacy input`) to the terminal.

### Fill sites using the helper

| Offer | Fields filled via currency row |
|---|---|
| RTC CC | Casino Credit amount |
| LC-REACT | Casino Credit amount |
| BG | FanCash amount |
| DM | Bonus amount card, Minimum Deposit card, Maximum Deposit card |
| LC-CHURN-DM | Bonus/Casino Credit amount card, Minimum Deposit card, Maximum Deposit card |
| VIPDM | Bonus/Casino Credit amount card, Minimum Deposit card, Maximum Deposit card — always the CAD row (CA-only) |
| VIPBG | Casino Credit amount (flat, = the "get" value) — always the CAD row (CA-only); no deposit fields |
| RTC FS / RAF / SUO | Spin value row in the **Free Spin Stakes** table (select dropdown, not a number input) — row chosen by currency label cell via `currency_for()` (v2.19) |

Percentage inputs (`aria-valuemin='0' aria-valuemax='100'`, used by DM and LC-CHURN-DM) are **not** part of the currency table and are unchanged.

---

## Casino Game Selection — ™-Tolerant Matching (v2.20/v2.21)

NATS game option names now include trademark symbols, e.g. `Triple Cash Eruption™ (WV)`. Game selection in RTC FS, RAF, and SUO works in two decoupled steps:

1. **Search** — `parse_casino_game_search()` types only the portion of the game's `searchName` **before the first `™`** into the Casino Game search box (`Triple Cash Eruption`, `7's Fire Blitz`, `WrestleMania`). NATS's substring search then surfaces every jurisdiction variant regardless of how the symbol is rendered. The jurisdiction is **not** typed — typing it fails because `™` sits between the name and the `({jurisdiction})` suffix.
2. **Match** — `parse_casino_game_pattern()` selects the dropdown option with an anchored regex built from the full `searchName` where **every `™` is optional** (mid-name and trailing) and the **jurisdiction suffix is required**: `^{name-with-optional-™}™?\s*\({JUR}\)$`. The correct state's game is always chosen whether or not NATS includes `™`; wrong jurisdictions and similarly-named games can never match. The terminal prints the actual option text clicked.

The `GAMES` table's `searchName` values are unchanged — the matcher tolerates NATS adding or removing `™` without further edits.

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
| 7 | VIPDM | CA — Canada only. A US region is planned (RTC CC-style dual jurisdiction with region-scoped keys); `currency_for()` already routes US codes to the USD row, so the runtime side needs no change. |
| 8 | VIPBG | HIW, T&C, Terms IDs, and promotional dates are baked in at generation time — changes via the region-aware Edit T&Cs / Terms modal require regenerating the script. New offer sizes require a Terms ID (editable in the Terms modal; keyed `{SBG\|TBG}_B{bet}_G{get}` for CA, `B{bet}_G{get}` for US); active offers without one block script generation by design. |
| 9 | VIPBG | Dual region as of v2.25 (US: FanCash/USD; CA: Casino Credit/CAD). Button 2 is empty for CA and `Play Now!` → `/docs/usered/casgenericgamelist` for US. Series `VIP` shares limitation #6 with VIPDM. **US VIPBG scripts must be generated from v2.25+** — earlier versions were CA-locked. |
| 10 | VIPBG | The US canonical T&C was verified byte-level against the CAS_10687 and CAS_10688 legal filings (with four stakeholder-approved copy-edit normalizations); the CAS_10689 / CAS_10690 filings were not supplied and are assumed to follow the same template — verify on first use. First live US run should also confirm the FanCash confirmation modal fires and whether Stake Chunk Sizes exists on the FanCash form (the script handles both outcomes defensively). |

---

## Prerequisites

- Python 3.9 or later (for `zoneinfo`)
- Setup instructions: https://adrianvdc.github.io/Promo-Building-Script-Generator/setup-guide
- Google Drive for Desktop required for RTC CC, RTC FS, RAF, LC-REACT, LC-CHURN-DM, VIPDM, and VIPBG image path access
