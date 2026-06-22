# NATS Bonus Creator — Technical Reference v2.13

## Changes from v2.12

- **`create_segment` timing fix (all offer types):** The 500ms `wait_for_timeout` after `dispatch_event("click")` on AMELCO has been replaced with a 2-second `wait_for_timeout` placed after `#forBonus` is checked. This gives NATS more time to register the Region selection before the OK button is clicked, fixing intermittent segments saving without AMELCO set as the Region.

---

## HTML Tool Layout

- **Left column (340px fixed):** Branding, primary action buttons, Offer Type selector, Region selector (hidden for RTC FS and RAF)
- **Right column (flex):** Setup guide link panel, offer-type-specific info panel, and run instruction cards
- **Below header:** Day cards grid (or single campaign card for RAF)
- **Below day cards:** Generated script output area (hidden until script is generated)

---

## Global Controls

**Generate Script (white button)**
Reads all day card inputs, builds internal code names, generates and auto-downloads a Python script. For RAF: enabled when date (6 chars), Spins/Day > 0, Bet Amount > 0, and Spin Value are all filled.

**Edit HIW / Edit T&Cs**
Hidden for RAF (copy pending legal approval). Active for all other offer types.

**Edit Images**
Present for RTC CC, RTC FS, and RAF. Hidden for BG and DM.

**+ Add Day**
No-op for RAF. Max 8 days for BG/DM, 99 for RTC CC/FS.

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

---

## Sidebar Navigation

All five offer types use the same sidebar-based navigation pattern with `data-menu-id` suffix selectors:

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

Each image upload takes approximately **10 seconds** (8s upload wait + 2s after OK). Three images per promo = ~30 seconds per promo build. Applies identically across all five offer types.

---

## Script Steps — RTC Top Up - Casino Credit

### `create_segment(page, name)`
Click `button.ant-btn-primary.create`, fill `#name` and `#code`, click `#parentId`, wait 500ms, select AMELCO via `.nth(2).dispatch_event("click")`, check `#forBonus`, wait 2000ms, click OK span, wait networkidle.

### `create_promo(page, name)`
Poll Create Promotion (120 × 500ms, up to 60s) → click → wait networkidle → fill name → Start Time (**day-of 04:00:00 ET**) → End Time (**+24 hours, 04:00:00 ET next day**) → Type: Image only CTA → Layout: Overlay → first Save → upload images → fill Title / Promo Header → toggle Bonus Tile → fill HIW/T&C → second Save.

### `create_bonus(page, name)`
Poll Create Bonus (120 × 500ms, up to 60s) → Opt In → fill name/description → Send To XP → remove `COMBO: Sportsbook` and `COMBO: Sportsbook And Casino` tags → Voucher Code (`CRRCCC{MMDDYY}{XXX}`) → Casino Credit type → Days fields → Stake Chunk: 1 → amount → Status Active → Activation (**day-of 04:00:00 ET → +24 hours 04:00:00 ET next day**) → Segment → Reporting Platform / Origin / Series / Type → Promotion Tile → Save (no modal).

### `parse_dates(code)` — RTC CC only
```python
def parse_dates(code):
    date_part = code.split("_")[0]
    mm = date_part[0:2]
    dd = date_part[2:4]
    yy = "20" + date_part[4:6]
    dt_eastern = datetime(int(yy), int(mm), int(dd), 4, 0, 0, tzinfo=EASTERN)
    dt_end_eastern = dt_eastern + timedelta(hours=24)
    start = dt_eastern.astimezone().strftime('%d-%m-%Y %H:%M:%S')
    end = dt_end_eastern.astimezone().strftime('%d-%m-%Y %H:%M:%S')
    return start, end
```

> ⚠️ `parse_dates` is defined only in the RTC CC script builder. BG, DM, RTC FS, and RAF each have their own date logic and remain on midnight (00:00:00 ET) windows.

---

## Script Steps — RTC Top Up - Free Spins

Same structure as RTC CC. Voucher: `CRRCFS{MMDDYY}{XXX}`. Free Spin type via JS evaluate. No confirmation modal. Activation window: **day-of midnight → day+30 midnight** Eastern. Create Promotion poll applies.

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
| Referee Bonus field | Not present | Type description → first result |

### `edit_bonus_rfer(page, voucher_code)` — post-save edit pass
1. Nav to Bonus Manager
2. Wait 30s
3. Uncheck Active Only if checked
4. Click Reload → wait **120s** on page
5. Search by description string
6. Verify Start Date = `DD/MM/YYYY, 12:00 AM`
7. Click pencil → re-select Referrer radio if needed → re-select Reporting Platform if blank
8. Save → voucher modal → Apply

### `create_bonus_day(page, n, voucher_code)`
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

## updateGenerateButton — RAF Branch

```javascript
if (offerType === 'RAF') {
  const d = days[0];
  const dateEl = document.getElementById('raf-date-input');
  const date = dateEl ? dateEl.value.trim() : d.date;
  ready = !!(date && date.length === 6
    && d.rafSpinAmount && parseFloat(d.rafSpinAmount) > 0
    && d.rafBetAmount && parseFloat(d.rafBetAmount) > 0
    && d.rafSpinValue);
}
```

RAF bypasses the `days[]` loop (which checks for amounts/offers that RAF does not use) and instead checks the campaign-level fields directly.

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

RAF HIW and T&C pending legal-approved text, not yet in modal system.

---

## Image System

### RTC CC and RTC FS
Google Drive path in localStorage. Edit Images button present. Script uses `IMAGE_FOLDER`.

### BG and DM
ZIP per day card. Base64-embedded in generated script. Edit Images hidden.

### RAF
Google Drive path in `raf_image_path` localStorage. Edit Images button present. Path injected into generated script at generation time.

---

## Promotion Tile Selection

```python
page.locator(".ant-select-dropdown:not(.ant-select-dropdown-hidden) div.ant-select-item-option-content")
    .filter(has_text=re.compile(r"^" + re.escape(name) + r"$"))
    .first
```

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
| `.ant-input-number-input` | Numeric inputs |
| `.ant-checkbox-input` | Checkboxes |
| `label.ant-checkbox-wrapper` | Checkbox wrapper (RAF Send To XP) |
| `label[title='Status Active?']` | Status Active row (RAF) |
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
