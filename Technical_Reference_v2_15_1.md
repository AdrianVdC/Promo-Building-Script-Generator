# NATS Bonus Creator — Technical Reference v2.15

## Changes from v2.15

- **Bet & Get - Rules Engine** — seventh offer type fully integrated into the HTML tool. Two-phase generated script: Phase 1 builds NATS promos, Phase 2 builds Playmaker bonuses. Day card includes Reward Type, Jurisdiction, Days to Use Reward, and Show Toast fields. Generates `bg_re_MMDDYY_HHMM.py`. Currently pointed at test environment — switch URLs to production once Rewards Engine is live in Playmaker production.

## Changes from v2.14

- **BG-RE (Bet & Get — Rules Engine)** — seventh offer type in development. Full two-phase Playwright flow confirmed working (NATS + Playmaker, multi-code loop). HTML tool integration not yet started.

## Changes from v2.13

- **New offer type: SUO Day 2+ Spins** — sixth offer type added. Generates segments and bonuses for DAY2–DAYN codes. No promos, no voucher codes, no step badges. Series = Early Life, Trigger = No Opt In, Status Active = checked. Code name format: `MMDDYY_CAS_ACQ_SUO_DAY{n}_{jurisdiction}_FS_{spins}`. Script filename: `suo_day2_MMDDYY_HHMM.py`.

---

## HTML Tool Layout

- **Left column (340px fixed):** Branding, primary action buttons, Offer Type selector, Region selector (hidden for RTC FS, RAF, and SUO)
- **Right column (flex):** Setup guide link panel, offer-type-specific info panel, and run instruction cards
- **Below header:** Day cards grid (or single campaign card for RAF and SUO)
- **Below day cards:** Generated script output area (hidden until script is generated)

---

## Global Controls

**Generate Script (white button)**
Reads all day card inputs, builds internal code names, generates and auto-downloads a Python script. For RAF and SUO: enabled when date (6 chars), Spins/Day > 0, Bet Amount > 0, and Spin Value are all filled.

**Edit HIW / Edit T&Cs**
Hidden for RAF, SUO, and BG-RE (copy pending legal approval or not applicable). Active for all other offer types.

**Edit Images**
Present for RTC CC, RTC FS, and RAF. Hidden for BG, DM, SUO, and BG-RE.

**+ Add Day**
No-op for RAF and SUO. Max 8 days for BG/DM, 99 for RTC CC/FS.

**Clear Saved Settings**
Clears all localStorage keys including `raf_image_path`.

---

## Day Cards

### RTC Top Up - Casino Credit
Grid: 3 columns | Default days: 9
Date, amounts grid (presets + custom), step badges (Segment/Promo/Bonus), Send to XP, Days to Opt In / Entitlement / Settlement.

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

---

## RAF Phase Badges

| Badge | Phases covered |
|---|---|
| Day 2+ | Phase 1 (DAY2–N segments) + Phase 3 (DAY2–N bonuses) |
| Referee | Phase 2 (RFRE promo) + Phase 4 (RFRE bonus + edit) |
| Referrer | Phase 1 (RFER segment) + Phase 2 (RFER promo) + Phase 5 (RFER bonus + edit) |

Phase 1 skipped if Day 2+ and Referrer both off. Phase 2 skipped if Referee and Referrer both off.

---

## Per-Code Step Toggles (RTC CC / FS / BG / DM)

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

**Format key — RTC / BG / DM:**

| Position | Value | Meaning |
|---|---|---|
| 1 | `C` | Casino |
| 2 | `R` / `L` | Retention / Lifecycle |
| 3–4 | `BG` / `DM` / `RC` | Offer type |
| 5–6 | `FC` / `CC` / `FS` | Reward type |
| 7–12 | `MMDDYY` | Date |
| 13–15 | `XXX` | Random suffix |

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

### RTC CC / FS / BG / DM
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
3. **Phase 1 — NATS Promos** — all promos built before Playmaker opens (name, dates, type/layout, 3 images, header text, HIW, T&C, save)
4. Open Playmaker test environment in second page
5. Pause for manual Playmaker SSO login
6. Navigate: iCasino → Bonus Setup (once)
7. **Phase 2 — Playmaker Bonuses** — loop per code: Create Bonus → Steps 1–6 → Save Bonus
8. macOS notification + close browser

---

## Sidebar Navigation

All six offer types use the same sidebar-based navigation pattern with `data-menu-id` suffix selectors:

```python
async def nav_to_bonus(page):
    for attempt in range(1, 6):
        await page.mouse.move(600, 400)
        await page.wait_for_timeout(300)
        await page.locator("[data-menu-id$='-bonuses']").hover()
        await page.wait_for_timeout(500)
        link = page.get_by_text("Bonus Manager", exact=True)
        if await link.is_visible():
            await link.dispatch_event("click")
            await page.wait_for_load_state("networkidle")
            return
        await page.wait_for_timeout(500)
    raise Exception("Could not open Bonus Manager from sidebar after 5 attempts")
```

`nav_to_promos` and `nav_to_segments` follow the same pattern with `-cms`/`Promos` and `-segments`/`Account Segments`.

| Screen | `data-menu-id` suffix | Submenu item |
|---|---|---|
| Account Segments | `-segments` | `Account Segments` |
| Promos | `-cms` | `Promos` |
| Bonus Manager | `-bonuses` | `Bonus Manager` |

---

## Image Upload — `upload_image`

```python
async def upload_image(page, label_title, filepath, display_name=None):
    row = page.locator(".ant-legacy-form-item").filter(has=page.locator(f"label[title='{label_title}']"))
    await row.get_by_text("Add").click()
    await page.wait_for_timeout(1000)
    await row.locator(".anticon-edit").click()
    await page.wait_for_timeout(1000)
    async with page.expect_file_chooser() as fc_info:
        await page.get_by_role("button", name="plus Upload").click()
    file_chooser = await fc_info.value
    await file_chooser.set_files(filepath)
    await page.wait_for_timeout(8000)
    await page.get_by_role("button", name="OK").click()
    await page.wait_for_timeout(2000)
    print(f"    ✅ Image uploaded: {display_name or filepath.split('/')[-1]}")
```

Each image upload takes approximately **10 seconds** (8s upload wait + 2s after OK). Three images per promo = ~30 seconds per promo build. Applies to RTC CC, RTC FS, BG, DM, and RAF. SUO does not upload images.

---

## Script Steps — RTC Top Up - Casino Credit

1. Opt In trigger
2. Internal name + description (`$X of Casino Credit`)
3. Send To XP (conditional)
4. Remove `COMBO: Sportsbook` and `COMBO: Sportsbook And Casino` tags
5. Voucher Code: `CRRCCC{MMDDYY}{XXX}`
6. Casino Credit type
7. Stake Chunk Sizes = 1
8. Status Active checked
9. Activation: day-of 04:00 ET → +24 hours ET
10. Reporting: Platform = CAS, Origin = Retention, Series = RTC, Type = Lifecycle
11. Segment / Client Profiling
12. Promotion Tile
13. Save (no confirmation modal)

---

## Script Steps — Bet & Get

Same structure as RTC CC. Voucher: `CLBGFC{MMDDYY}{XXX}`. FanCash type. Confirmation modal: **Yes** required. Create Promotion poll applies.

---

## Script Steps — Deposit Match

Same structure as RTC CC. Voucher: `CRDMFC{MMDDYY}{XXX}`. FanCash + Bonus % type. Confirmation modal: **Yes** required. Create Promotion poll applies.

---

## Script Steps — RAF

### `create_segment(page, name)`
Identical to RTC CC (including AMELCO `dispatch_event("click")` and 2-second post-`#forBonus` wait).

### `create_promo(page, name, discover_image)` — RFRE and RFER
Poll Create Promotion (120 × 500ms, up to 60s) → click → fill name → Start: day-of midnight → End: year 2041 → Type: Image only CTA → Layout: Overlay → Refer A Friend toggle ON → first Save → upload images → fill all copy fields → second Save.

### `create_bonus_rfre(page, voucher_code)`
Poll Create Bonus (120×500ms) → Opt In → internal name + description → Check Refer A Friend Bonus → Referee radio → remove `COMBO: Sportsbook` and `COMBO: Sportsbook And Casino` tags → Voucher Code → Free Spin type → Free Spin's Description → CTA / Aggregator / Provider / Casino Game / Bet Level / Deeplink → Spins (per-day) → Spin Value → Entitlement block (Deposit and Wagering, Min 10, Max 1000000, Bonus % 100, Qualifying Stake = bet amount) → Days (0/0/1) → Check **Status Active** → Activation → Reporting fields → Promotion Tile → Save → voucher modal → Apply.

### `edit_bonus_rfre(page, voucher_code)` — post-save edit pass
1. Nav to Bonus Manager
2. Wait 30s
3. Uncheck Active Only if checked
4. Click Reload → wait 20s
5. Search by description string
6. Verify Start Date = `DD/MM/YYYY, 12:00 AM`
7. Click pencil → re-select Referee radio if needed → re-select Reporting Platform if blank
8. Save → voucher modal → Apply

### `create_bonus_rfer(page, voucher_code)` — key differences from RFRE

| Field | RFRE | RFER |
|---|---|---|
| RAF radio | `value="REFEREE"` | `value="REFERRER"` |
| Send to XP | Not present | Check ON |
| Number of Spins | Per-day | **Total spins** |
| Entitlement block | Present | **Absent** |
| Days to Meet Freespin Settlement | 1 | **7** |
| Status Active | checked | **unchecked** |
| Voucher group | `RFRE` | `RFER` |
| Promotion Tile | RFRE code | RFER code |

### `edit_bonus_rfer(page, voucher_code)` — post-save edit pass
1. Nav to Bonus Manager
2. Wait 30s
3. Uncheck Active Only if checked
4. Click Reload → wait **120s** on page
5. Search by description string
6. Verify Start Date = `DD/MM/YYYY, 12:00 AM`
7. Click pencil → re-select Referrer radio if needed → re-select Reporting Platform if blank
8. Save → voucher modal → Apply

### `create_bonus_day(page, n, voucher_code)` — RAF Day 2+
Poll Create Bonus (120×500ms) → Opt In → code name → description blank → Refer A Friend Bonus unchecked → Send To XP checked → remove `COMBO: Sportsbook` and `COMBO: Sportsbook And Casino` tags → Voucher Code → Free Spin type → Free Spin's Description (`"Refer a Friend: Bet $X Get Y Day N"`) → CTA / game fields → Spins (per-day) → Spin Value → Days (0/0/1) → Status Active unchecked → Activation → Reporting fields → Segment / Client Profiling → Save (no modal, no edit pass).

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

### `create_segment(page, name)`
Identical to RAF Day 2+ (AMELCO `dispatch_event("click")`, 2-second post-`#forBonus` wait). Code name format: `MMDDYY_CAS_ACQ_SUO_DAY{n}_{jurisdiction}_FS_{spins}`.

### `create_bonus_suo_day(page, n)`
1. Poll Create Bonus (120×500ms)
2. **No Opt In** trigger (selector: `page.get_by_text("No Opt In", exact=True)`)
3. Internal name = code name; description left blank
4. Refer A Friend Bonus checkbox — **not touched**
5. Send To XP checked
6. Remove `COMBO: Sportsbook` and `COMBO: Sportsbook And Casino` platform tags
7. **No voucher code**
8. Free Spin type
9. Free Spin's Description = `"Sign Up Offer Bonus Spins Day N"`
10. CTA = `Play!`; Aggregator / Provider / Casino Game / Bet Level (default) / Deeplink
11. Spins = per-day amount
12. Spin Value
13. Days to Opt In = 0 / Days to Entitlement = 0 / Days to Settlement = 1
14. Activation start = code date minus 1 day at midnight ET; end = year 2041
15. **Status Active = checked**
16. Reporting Platform = CAS (Casino); Origin = Bet Settlement Bonus; **Series = Early Life** (selected by `title="Early Life"` attribute); Type = Acquisition
17. Segment / Client Profiling = matching SUO DAY{n} segment
18. Save — **no confirmation modal, no post-save edit pass**

### Fixed Values — SUO Day 2+ Bonuses

| Field | Value |
|---|---|
| Trigger | No Opt In |
| Refer A Friend Bonus | Not touched |
| Voucher Code | None |
| Free Spin's CTA | Play! |
| Bet Level | default |
| Activation start | Code date minus 1 day, midnight ET |
| Activation end | Year 2041 |
| Reporting Platform | CAS (Casino) |
| Bonus Origin | Bet Settlement Bonus |
| Series | Early Life |
| Type | Acquisition |
| Status Active | Checked |

### Series Selection — Early Life

Unlike most dropdowns which filter by typing, Early Life is selected by `title` attribute:

```python
series_row = page.locator(".ant-legacy-form-item").filter(has=page.locator("label[title='Series']"))
await series_row.locator(".ant-select-selector").click()
await page.wait_for_timeout(500)
series_option = page.locator(".ant-select-dropdown:not(.ant-select-dropdown-hidden) .ant-select-item-option[title='Early Life']")
await series_option.wait_for(state="visible", timeout=15000)
await series_option.click()
```

---

## updateGenerateButton — RAF and SUO Branches

```javascript
if (offerType === 'RAF') {
  const d = days[0];
  const dateEl = document.getElementById('raf-date-input');
  const date = dateEl ? dateEl.value.trim() : d.date;
  ready = !!(date && date.length === 6
    && d.rafSpinAmount && parseFloat(d.rafSpinAmount) > 0
    && d.rafBetAmount && parseFloat(d.rafBetAmount) > 0
    && d.rafSpinValue);
} else if (offerType === 'SUO') {
  const d = days[0];
  const dateEl = document.getElementById('suo-date-input');
  const date = dateEl ? dateEl.value.trim() : d.date;
  ready = !!(date && date.length === 6
    && d.suoSpinAmount && parseFloat(d.suoSpinAmount) > 0
    && d.suoBetAmount && parseFloat(d.suoBetAmount) > 0
    && d.suoSpinValue);
}
```

Both RAF and SUO bypass the `days[]` loop (which checks for amounts/offers that neither offer type uses) and instead check campaign-level fields directly.

---

## RTC CC — Terms Expiry & Terms IDs Modal

**Default values:** Start: `06/01/26` | End: `12/01/26` | IDs: `$2→CAS_9461` through `$800→CAS_9471`
**localStorage keys:** `rtc_terms_start_date`, `rtc_terms_end_date`, `rtc_terms_ids`

---

## How It Works & T&C Copy

| Offer | Dynamic Variables | localStorage Keys |
|---|---|---|
| RTC CC | `{amount_fmt}`, `{terms_id}`, `{start_date_short}`, `{start_date_long}`, `{end_date_short}`, `{end_date_long}` | `rtc_hiw`, `rtc_tc` |
| RTC FS | `{spins}`, `{spin_value}`, `{game}` | `rtc_fs_hiw`, `rtc_fs_tc` |
| BG | `{bet_fmt}`, `{get_fmt}`, `{date_fmt}` | `bg_hiw`, `bg_tc` |
| DM | `{pct}`, `{max_fmt}`, `{date_fmt}` | `dm_hiw`, `dm_tc` |

RAF and SUO HIW and T&C: pending legal-approved text, not yet in modal system.

---

## Image System

### RTC CC and RTC FS
Google Drive path in localStorage. Edit Images button present. Script uses `IMAGE_FOLDER`.

### BG and DM
ZIP per day card. Base64-embedded in generated script. Edit Images hidden.

### RAF
Google Drive path in `raf_image_path` localStorage. Edit Images button present. Path injected into generated script at generation time.

### SUO Day 2+ Spins
No images. Edit Images button hidden.

---

## Promotion Tile Selection

```python
page.locator(".ant-select-dropdown:not(.ant-select-dropdown-hidden) div.ant-select-item-option-content")
    .filter(has_text=re.compile(r"^" + re.escape(name) + r"$"))
    .first
```

SUO Day 2+ does not use a Promotion Tile.

---

## Bet & Get — Rules Engine (BG-RE)

Seventh offer type, fully integrated into the HTML tool as of v2.15. Generates `bg_re_MMDDYY_HHMM.py`.

> ⚠️ Currently uses **test environment** URLs. Switch to production before using in production:
> - NATS: `https://trading.1.betfanatics.com/`
> - Playmaker: `https://playmaker-internal.fanatics.bet/` *(confirm exact prod URL when available)*

### Architecture

**Phase 1 — NATS (`trading.test1.fanatics.bet`)**
All promos are built before Playmaker is opened. Identical to existing BG promo flow including ZIP image upload. User logs in manually.

**Phase 2 — Playmaker (`playmaker-internal.test1.fanatics.bet`)**
User logs in manually via SSO. Script navigates iCasino → Bonus Setup once, then loops all bonuses via `create_pm_bonus(page, code)`.

### CODES List

```python
CODES = [
    {
        "code_name":       "101926_CAS_RET_BG_US_FC_B100_G2",
        "jurisdiction_pm": "USA",
        "days_to_fulfill": "7",
        "show_toast":      True,
    },
    ...
]
```

One entry per day card. Both phases iterate this list. The HTML tool will populate it at generation time.

### Naming Conventions

Same as existing BG — no changes:

| Field | Format | Example |
|---|---|---|
| Code name | `MMDDYY_CAS_RET_BG_{jurisdiction}_{rewardType}_B{bet}_G{get}` | `062226_CAS_RET_BG_US_FC_B100_G2` |
| Voucher code | `CLBGFC{MMDDYY}{XXX}` | `CLBGFC062226AB3` |

### Reward Types

| Selection | Code segment | Playmaker value | Status |
|---|---|---|---|
| FanCash | `FC` | `fan_cash` | ✅ Confirmed working |
| Casino Credit | `CC` | `casino_credit` | ✅ Confirmed working |
| Free Spins | `FS` | `free_spin` | ❌ Deferred — different field structure |

### Jurisdictions (Playmaker)

| Day card option | Checkboxes checked |
|---|---|
| USA | MI, WV, NJ, PA |
| Canada | ON |
| Michigan | MI only |
| West Virginia | WV only |
| New Jersey | NJ only |
| Pennsylvania | PA only |
| Ontario | ON only |

Checkbox IDs: `#jurisdiction-all`, `#jurisdiction-michigan`, `#jurisdiction-west_virginia`, `#jurisdiction-new_jersey`, `#jurisdiction-pennsylvania`, `#jurisdiction-ontario`.

> Always uncheck `#jurisdiction-all` first — it unchecks everything at once. Jurisdictions must be set BEFORE dates/times or the time inputs reset.

### Playmaker Form — Step by Step

**Step 1: Details**

| Field | Value | Selector |
|---|---|---|
| Bonus Name | Code name | `input[name='name']` |
| Jurisdictions | See table above | `#jurisdiction-{name}` |
| Start Date | Code date | `button.dc-w-\[280px\]` index 0 → calendar |
| End Date | Code date + 1 day | `button.dc-w-\[280px\]` index 1 → calendar |
| Start Time | 00:00 | `get_by_role("textbox", name="Time").nth(0)` — set via JS `nativeInputValueSetter` |
| End Time | 00:00 | `get_by_role("textbox", name="Time").nth(1)` — set via JS `nativeInputValueSetter` |

Calendar: month header `div[aria-live='polite']`, next month `button[name='next-month']`, day buttons `button[name='day']` (skip `dc-day-outside` — Playmaker uses `dc-` prefix, not `rdp-day_outside`).

Time inputs use JS `nativeInputValueSetter` with `input` + `change` events — the fields are segmented (hours/minutes/AM-PM) and do not respond to standard `.fill()` or `.type()`.

**Step 2: Eligibility**

| Field | Value | Selector |
|---|---|---|
| Create Account Segments | Code name (typed + Enter → chip) | `input[cmdk-input]` |

**Step 3: Entitlement**

| Field | Value | Selector |
|---|---|---|
| Wagering Amount ($) | Bet value from code name | `input[name='wageringAmount']` |

**Step 4: Settlement**

| Field | Value | Selector |
|---|---|---|
| Reward Type | `fan_cash` / `casino_credit` | `button[role='radio'][value='{value}']` |
| Amount ($) | Get value from code name | `input[name='settlementAmount']` |
| Days to Use Reward | Day card field (default 7) | `input[name='daysToFulfill']` |

Casino Credit is selected by default. Script checks current state before clicking.

**Step 5: Display**

| Field | Value | Selector / Notes |
|---|---|---|
| Promotion | Code name | `page.get_by_label(name).get_by_text(name)` after searching combobox |
| Show Toast | Day card toggle | `#toastEnabled` |
| Toast Title | `"Promotion Complete!"` | `input[name='toastTitle']` |
| Toast Description | `"You have been awarded ${get} FanCash"` / `"...Casino Credit"` | `input[name='toastDescription']` |

**Step 6: Review → Save**

| Action | Selector |
|---|---|
| Step 6 | `button:has-text('Step 6: Review')` |
| Save Bonus | `button:has-text('Save Bonus')` |

### Promotion Dropdown — Confirmed Solution

`page.get_by_label(promo_name).get_by_text(promo_name)` — identified via `await page.pause()` + Playwright Inspector (v0.20).

Standard selectors (`[role='option']`, `[data-radix-collection-item]`, `ArrowDown+Enter`) all failed because Playmaker is a micro-frontend — the iCasino app runs in a separate JS context from the shell.

### Images (Phase 1 — NATS)

Identical to existing BG. Three images per code, base64-embedded in script, keyed by full code name:

```python
IMAGES = {
    "101926_CAS_RET_BG_US_FC_B100_G2": {
        "Promo Detail.png": "<b64>",
        "Masthead.png":     "<b64>",
        "B100_G2.png":      "<b64>",
    },
    ...
}

def get_image_path(name, filename):
    b64 = IMAGES[name][filename]
    tmp = tempfile.NamedTemporaryFile(delete=False, suffix=os.path.splitext(filename)[1])
    tmp.write(base64.b64decode(b64))
    tmp.close()
    return tmp.name
```

Upload labels: `"Promo Detail Image Upload"`, `"Masthead Image Upload"`, `"Discover Image Upload"`.

### Navigation (Playmaker)

```
Login (manual SSO) → h3:has-text('iCasino') → [data-testid='bonus-setup-navItem'] → [data-testid='create-bonus-button'] (repeated per code)
```

### HTML Tool Day Card

| Field | Default | Purpose |
|---|---|---|
| Date (MMDDYY) | — | Code date, drives NATS promo dates and Playmaker calendar |
| Images (ZIP) | — | Per-day ZIP, same as existing BG |
| Offers (Bet/Get rows) | B100/G2, B200/G5, B2000/G50, B20000/G500 | Dynamic rows, all inactive by default |
| Reward Type | FanCash | Drives `FC`/`CC` code segment + Playmaker radio |
| Jurisdiction | USA | Drives jurisdiction segment in code name + Playmaker checkboxes |
| Days to Use Reward | 7 | Wired to `daysToFulfill` in Playmaker Step 4 |
| Show Toast | On | Drives `#toastEnabled` + title/description in Playmaker Step 5 |
| Send to XP | On | Standard field |

### Outstanding Items

| # | Item | Status |
|---|---|---|
| 1 | Promotion dropdown | ✅ Confirmed — `get_by_label().get_by_text()` |
| 2 | Toast notification | ✅ Confirmed working |
| 3 | Step 6 + Save Bonus | ✅ Confirmed working |
| 4 | NATS promo with images | ✅ Confirmed working |
| 5 | Multi-code loop | ✅ Confirmed working |
| 6 | HTML tool day card UI | ✅ Integrated in v2.15 |
| 7 | HTML tool `buildBGREScript` | ✅ Integrated in v2.15 |
| 8 | Free Spins reward type | ❌ Deferred |
| 9 | Switch URLs to production | ❌ Pending — Rewards Engine not yet in Playmaker prod |

---

## Platform Tags — Bonus Creation

NATS pre-populates five platform tags on every new bonus: `All`, `COMBO: Sportsbook And Casino`, `COMBO: Sportsbook`, `STAC: Standalone Casino`, `Web`, and `COMBO: Casino`. All generated scripts remove `COMBO: Sportsbook` and `COMBO: Sportsbook And Casino`. Removing either of these causes NATS to automatically remove `All` as well. The resulting active platforms on every bonus are:

- `STAC: Standalone Casino`
- `Web`
- `COMBO: Casino`

```python
for value in ["COMBO: Sportsbook", "COMBO: Sportsbook And Casino"]:
    tag = page.locator(f".ant-select-selection-item[title='{value}']")
    await tag.locator(".ant-select-selection-item-remove").click()
    await page.wait_for_timeout(500)
```

Applies to all six NATS offer types including SUO. BG-RE bonuses are built in Playmaker, not NATS, so platform tags do not apply.

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
| `.ant-select-dropdown:not(.ant-select-dropdown-hidden)` | Currently open dropdown |
| `.ant-select-item-option[title='Early Life']` | Early Life series option (SUO) |
| `.ant-input-number-input` | Numeric inputs |
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

## Known Bugs

| # | Area | Severity | Issue |
|---|---|---|---|
| 2 | DM | ⚠️ | `format_hiw` / `format_tc` baked in at generation time — modal overrides require script regeneration |
| 5 | RTC FS | ⚠️ | HIW and T&C use placeholder copy — pending legal-approved text |
| 6 | RAF | ⚠️ | NATS clears Referee/Referrer radio and Reporting Platform on bonus re-open — post-save edit pass required |
| 7 | RAF | ⚠️ | Bonus Manager requires 120-second wait during RFER edit pass — script stays on page |

---

## Prerequisites

- Python 3.9 or later (for `zoneinfo`)
- Setup instructions: https://adrianvdc.github.io/Promo-Building-Script-Generator/setup-guide
- Google Drive for Desktop required for RTC CC, RTC FS, and RAF image path access
