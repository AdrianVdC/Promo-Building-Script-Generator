# NATS Bonus Creator — Project Instructions

## What This Is
The NATS Bonus Creator is a single-file HTML tool that generates ready-to-run Python/Playwright scripts to automate segment, promotion, and bonus creation in the Fanatics Casino internal trading platform at `https://trading.1.betfanatics.com/` (Ant Design UI). The HTML file is the source of truth. Generated scripts are output-only and should never be edited directly.

**Current live version:** `nats_bonus_creator_v2_33.html`

> ⚠️ **v2.20 must not be used** — it contains a broken Casino Game regex (template escaping bug, hotfixed in v2.21). Any scripts generated from v2.20 will fail at Casino Game selection. Scripts generated from v2.17 or earlier will fail (BG, RTC CC, RTC FS, RAF, SUO, LC-REACT) or silently misfill Canadian amounts (DM, LC-CHURN-DM) due to the August 2026 NATS multi-currency and game-naming updates. **Canadian RTC CC scripts must be generated from v2.22+ only** — v2.21 and earlier allowed selecting ON/AB/CA on RTC CC but would bake in US images, US T&Cs, and US Terms IDs. **Canadian VIP Offer Library - Deposit Matches scripts must be generated from v2.23+ only** — pre-release v2.23 builds lack the Matched Deposit step and leave `COMBO: Casino` on the bonus. **Canadian VIP Offer Library - Bet & Gets scripts must be generated from v2.27+ only** — pre-release v2.24 builds circulated during development fill Button 2 (`Play Now!` → `/docs/usered/casgenericgamelist`) on CA promos, which must be empty for Canada, and **v2.24–v2.26 upload the generic `Promo Detail.png` / `Masthead.png` instead of the category-specific Slots/Tables creatives** required as of v2.27. **US VIP Offer Library - Bet & Gets scripts must be generated from v2.25+ only** — every earlier version locks VIPBG to CA — Canada and cannot produce US (FanCash/USD) scripts. **US VIP Offer Library - Deposit Matches scripts must be generated from v2.26+ only** — every earlier version locks VIPDM to CA — Canada and cannot produce US (FanCash/USD) scripts. **Daily Missions - Canada scripts must be generated from v2.31+ only** — v2.28 is the first version to include the offer type, but v2.28–v2.30 build the segment Playmaker-side via the "Create Account Segments" field, which under the v2.31 flow (NATS-built segments) would create a **duplicate segment**; v2.28/v2.29 additionally block FS days and set Playmaker times to 00:00 rather than 00:01. **Lifecycle - Churn DM scripts must be generated from v2.29+ only** — v2.28 and earlier leave the Matched Deposit checkbox unchecked on the bonus. **Free-spins scripts selecting the 7FH or 7H2 games (7's Fire Blitz™ Hotstepper / Hotstepper 2) must be generated from v2.33+ only** — NATS's game search is hard-capped at ~12 rendered results, and the broad `7's Fire Blitz` search both games typed under v2.20–v2.32 only surfaces the state entries that happen to land inside the cap (7H2 confirmed failing on NJ and PA under v2.32, passing on MI only by result-ordering luck; 7H2 itself only exists from v2.32). v2.33 types a per-game `searchOverride` plus the jurisdiction suffix instead. Regenerate all pending RTC FS / RAF / SUO scripts that select 7FH or 7H2 from v2.33; scripts for TCE / WWE and all Casino Credit-only offer types remain valid (v2.33 changed only the three FS builders — the 8 other builders are hash-identical to v2.32; but note 7P5 shares the broad search and remains exposed — see Known Limitation #20).

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
Twelve offer types are supported: **RTC Top Up - Casino Credit**, **RTC Top Up - Free Spins**, **Bet & Get (BG)**, **Deposit Match (DM)**, **Refer a Friend (RAF)**, **SUO Day 2+ Spins**, **Bet & Get - Rules Engine (BG-RE)**, **Lifecycle - REACT CC Drop (LC-REACT)**, **Lifecycle - Churn DM (LC-CHURN-DM)**, **VIP Offer Library - Deposit Matches (VIPDM)**, **VIP Offer Library - Bet & Gets (VIPBG)**, **Daily Missions - Canada (DMCA)**.

> ⚠️ **Bet & Get - Rules Engine is fully integrated as of v2.15 but currently uses test environment URLs.** NATS and Playmaker links must be switched to production once the Rewards Engine is live in the Playmaker production environment. **Daily Missions - Canada (v2.28) uses production URLs for both NATS and Playmaker** and is the first offer type to build bonuses in production Playmaker. **All seven DMCA weekdays generate as of v2.30** — Free Spins days (Mon/Wed/Fri/Sun) were unlocked when Playmaker Bonus Spins became functional; Casino Credit days are Tue/Thu/Sat. **As of v2.31, DMCA builds its segments in NATS** (Phase 1 = Segments + Promotions) and attaches them in Playmaker via the Include → Existing Account Segments field. Full selector reference and field spec are in `Technical_Reference_v2_33.md` attached to this project.

---

## Multi-Currency Amount Tables (v2.18+)

An August 2026 NATS update converted bonus amount tables from a single input into a **three-row table with one row per currency** (`USD` / `GBP` / `CAD`), keyed by `data-row-key` on each `<tr>`. All generated scripts include two shared helpers:

- **`currency_for(name)`** — derives the currency from the code name's jurisdiction token: US / MI / WV / PA / NJ → **USD**; ON / AB / CA → **CAD**. **GBP is never used under any circumstance** and is unreachable by construction.
- **`fill_currency_amount()`** — fills the value into the jurisdiction's currency row only (`tr[data-row-key='{currency}']`), with a fallback to the legacy single-input selector if NATS reverts. Terminal output shows which row was filled, e.g. `OK Bonus Amount (USD): 50`.

**Fields filled via the currency row:** RTC CC amount, LC-REACT amount, BG FanCash amount, DM Bonus amount + Min/Max Deposit, LC-CHURN-DM amount + Min/Max Deposit, VIPDM amount + Min/Max Deposit (CAD for CA — Canada Casino Credit codes, USD for US — United States FanCash codes as of v2.26), VIPBG flat reward amount (CAD for CA — Canada Casino Credit codes, USD for US — United States FanCash codes; no deposit fields), and the Free Spin Stakes spin value for RTC FS / RAF / SUO (a select dropdown, chosen by currency label). Percentage inputs (DM, LC-CHURN-DM, VIPDM) are not part of the currency table. **Daily Missions - Canada is not affected** — its bonus is built in Playmaker, which has no NATS currency table (the NATS phase builds the segment and promotion only; the Playmaker Free Spin Stakes grid shows CAD for Canada natively).

---

## RTC Top Up - Casino Credit — Canada Support (v2.22)

RTC CC is dual-jurisdiction: **US — United States** (default) and **CA — Canada**. The tool works exactly as before for US; CA layers its own settings on top via parallel `_ca` localStorage keys resolved through the `isRTCCanada()` / `rtcKey()` helpers.

**What differs when CA — Canada is selected:**
- **Region dropdown** on RTC CC shows only US and CA (all other offer types keep their existing region lists; VIPDM is US/CA as of v2.26, defaulting to US; VIPBG is US/CA as of v2.25; DMCA is fixed CA/Ontario with the region row hidden).
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

## VIP Offer Library - Deposit Matches (v2.23 CA / v2.26 US)

Tenth offer type. Monthly VIP-library deposit-match offers. **Dual region as of v2.26** (mirror of the VIPBG v2.25 pattern — CA owns the base localStorage keys, US layers parallel `_us` keys via the `isVIPDMUS()` / `vipdmKey()` helpers):

- **US — United States** (v2.26; **default**): paid in **FanCash** (USD row); ten default tiers.
- **CA — Canada** (original, v2.23): paid in **Casino Credit** (CAD row); nine default tiers.

### Region Behavior (v2.26)
- Region dropdown offers **US / CA**; ⚠️ unlike VIPBG (which keeps a valid current selection and defaults to CA), **VIPDM always selects US whenever the offer type is chosen** — per stakeholder direction. CA remains selectable from the dropdown. No other offer type's default behavior was modified.
- **Switching Region silently resets all card values** (dates, selected offers, toggles; generated script hidden) — US and CA inputs can never mix, same rule as RTC CC / VIPBG.
- **Visual cue:** the edit buttons render **red** when CA is selected and default **blue** for US (reuses RTC CC's `body.ca-region` theme).
- **All modals are region-aware:** Edit HIW, Edit T&Cs, Edit Terms Expiry & Terms IDs, and Edit Images read/save the selected region's keys; the T&C / Terms / Images modals show the region in their titles. **VIPDM HIW is region-scoped** (like VIPBG; unlike RTC CC, where HIW is shared).

### Day Card Layout
- **Single card** (full width), titled `VIP Offer Library — {region}`. **+ Add Day is a no-op.**
- Offer rows are **Min → % → Max** with a per-offer checkbox. Each row displays its Terms ID inline (red `no ID` when unmapped).
- **CA defaults (nine, unselected):** $100/50%/$250, $100/100%/$250, $250/10%/$250, $250/10%/$2,500, $250/10%/$5,000, $250/20%/$500, $250/20%/$1,000, $250/20%/$2,000, $250/20%/$5,000.
- **US defaults (ten, unselected):** $100/50%/$250, $100/100%/$250, $250/10%/$500, $250/10%/$1,000, $250/10%/$2,500, $250/10%/$5,000, $250/20%/$500, $250/20%/$1,000, $250/20%/$2,000, $250/20%/$5,000 — relative to CA, adds $250/10%/$500 and $250/10%/$1,000 and drops $250/10%/$250.
- No Campaign Name field, no ZIP attachment. Send to XP default ON; External Bonus present, default OFF (both regions). Days to Opt In 3 / Entitlement 3 / Settlement 7.

### Promo & Bonus Activation Window (month-anchored; both regions)
- **Start:** 00:00:00 ET **one day before the typed date**
- **End:** **last day of the typed date's calendar month** at 23:59:59 ET **+ Days to Meet Entitlement**

The typed day anchors only the start — `080126` and `081726` produce the same end date. A typed 1st starts in the previous month (`010127` starts Dec 31, 2026 — intended). Changing Days to Entitlement changes the promo **and** bonus end date. `calendar.monthrange` handles month lengths and leap years. Promo window and bonus activation window are identical.

### Terms ID Enforcement
Each active offer key `M{min}_{pct}M{max}` must exist in the selected region's VIPDM Terms ID map. Non-matching active offers show a red warning on the row and **block script generation** with an alert.

**CA defaults:**

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

**US defaults (v2.26):**

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

Terms IDs and promotional dates (CA default **08/17/26 → 12/31/26**; US default **07/01/26 → 12/31/26**) are edited per region via the **Edit Terms Expiry & Terms IDs** button inside the Edit T&Cs modal — dates render as `{start_date_long}` / `{end_date_long}` in the T&C.

### Fixed Copy
Title, Promo Header Text, and Bonus Description are all `From Your VIP Host` regardless of offer size or region. **Button 2 label `Deposit` → link `/auth/account/deposit`** on the promo, both regions (unlike Churn DM, which has no Button 2; VIPBG's Button 2 is region-dependent — see below).

### T&C Data (Baked In)
**CA:** Canadian (Ontario) legal base: FBG Enterprises Canada, Inc., 19+ / Ontario eligibility, ConnexOntario responsible gaming language, CAD-denominated examples. Dynamic fields: `{min_fmt}`, `{pct}`, `{max_fmt}`, `{terms_id}`, `{start_date_long}`, `{end_date_long}`. Title line: `FANATICS CASINO (CANADA) – DEPOSIT {min_fmt} CAD, GET {pct}% CASINO CREDIT (UP TO {max_fmt} CAD) ({terms_id})`. §1's "within 3 days of being presented this offer" and §4's "during that Promo Period" are **static legal copy by design** — not templated.

**US (v2.26):** single **canonical** FanCash template derived from the CAS_10677 legal filing. The three supplied filings (CAS_10677 / CAS_10678 / CAS_10686) were **not** word-identical — two copy-edit inconsistencies were normalized by stakeholder decision: (1) "Fanatics Sportsbook and **Casino App**" (the CasinoApp missing-space defect in 10678/10686 fixed — same defect class as VIPBG US), (2) "**Only** your first Qualifying Deposit" (majority wording; 10686 said "Your first"). Two further stakeholder-approved global normalizations: all trailing whitespace stripped, and the title line prefixed `FANATICS CASINO - `, giving `FANATICS CASINO - {min_fmt}+ DEPOSIT, {pct}% MATCH IN FANCASH (MAX {max_fmt}) (ID: {terms_id})`. Everything else is verbatim legal copy (byte-verified against all three supplied filings with those fixes applied) — the orphaned "(ii)" in Action Required, "exclusions apply- see fanatics.com", the double spaces around the FanCash Terms URL and "Notice of Financial Incentives.", and curly quotes are all preserved. Dynamic fields: `{min_fmt}`, `{pct}`, `{max_fmt}`, `{terms_id}`, `{start_date_long}`, `{end_date_long}` — no game-category fields. **Static by design:** MI/NJ/PA/WV eligibility, 21+, FBG Enterprises Opco, LLC entity, the 3-day deposit window and 72-hour Rewards delivery, the seven (7) day FanCash expiry, the FanCash Terms URL (`https://fanatics-one.com/fancashterms`), `Wagering Requirements/Exclusions: N/A`, `Eligible Games/Markets/Events: Does not apply.`, and the 1-800-GAMBLER responsible-gaming lines. ⚠️ The CAS_10679–CAS_10685 filings (seven of ten) were not supplied — the canonical template is assumed to cover them; verify on first use.

### Bonus Creation — Fixed Values

| Field | Value |
|---|---|
| Trigger type | Opt In (both regions) |
| Bonus type | CA: Casino Credit (percentage checkbox labeled `Casino Credit (%)`); US: FanCash (percentage checkbox labeled `Bonus (%)` — the standard-DM FanCash form) |
| Description | From Your VIP Host |
| Entitlement Type | Deposit |
| **Matched Deposit** | **Always checked** (inside the DEPOSIT entitlement panel; both regions) |
| **Platforms** | CA: **Web + STAC: Standalone Casino only** — all three COMBO tags removed. **US: standard tags — COMBO: Casino KEPT** (only the two Sportsbook tags removed) |
| Minimum Deposit | Dynamic from `M{min}` in the code — jurisdiction currency row |
| Maximum Deposit | 1000000 — jurisdiction currency row |
| Settlement field | CA: `Days to Meet Casino_credit Settlement`; US: `Days to Meet Fancash Settlement` |
| Days to Opt In / Entitlement | 3 (default, editable; Entitlement also extends the end dates) |
| Days to Meet Settlement | 7 (default, editable) |
| Status Active | Checked |
| Activation | Typed date − 1 day 00:00 ET → month-end 23:59:59 ET + entitlement days |
| Reporting Platform | CAS (Casino) |
| Bonus Origin | Opt-In Bonus |
| Series | VIP |
| Type | Retention |
| Confirmation modal | CA: none (no modal block in the CA script — production-confirmed flow untouched). US: **expected** (standard FanCash DM shows one) — **handled defensively** (waits 5s; confirms if one appears, continues with a terminal note reading "unexpected for FanCash — verify" if not) |

> ⚠️ Series `VIP` and Type `Retention` are selected via anchored regex scoped to the visible dropdown (`.ant-select-dropdown:not(.ant-select-dropdown-hidden)`). The Type scoping is load-bearing: the hidden Bonus Origin dropdown also contains a `Retention` option, so a page-wide exact-text click would crash with a Playwright strict mode violation. VIPDM and VIPBG are the only offer types that select `Retention` for Type.

Note: the `RET` token in the code name mirrors the **Type** field (Retention), not Bonus Origin — unlike standard DM. CA confirmed end-to-end in NATS production; US pending first live run (verify the FanCash confirmation-modal branch).

---

## VIP Offer Library - Bet & Gets (v2.24 CA / v2.25 US / v2.27 CA creatives)

Eleventh offer type. Monthly VIP-library wager-and-get offers. **Dual region as of v2.25** (mirror of RTC CC's v2.22 pattern, inverted — CA owns the base localStorage keys, US layers parallel `_us` keys via the `isVIPBGUS()` / `vipbgKey()` helpers):

- **CA — Canada** (original, v2.24; default): paid in **Casino Credit** (CAD row); Slots and Tables categories.
- **US — United States** (v2.25): paid in **FanCash** (USD row); **no category differentiation — all US tiers are slots B&Gs**.

> ⚠️ **Both regions: the wager requirement ("bet") is enforced upstream by XP / segment logic and is never entered anywhere on the bonus.** The bonus is a flat reward amount equal to the "get" value.

### Region Behavior (v2.25)
- Region dropdown offers **US / CA**; keeps a valid current selection, otherwise defaults to CA. ⚠️ Note this differs from VIPDM, which always defaults to US on selection (v2.26).
- **Switching Region silently resets all card values** (dates, selected offers, toggles; generated script hidden) — US and CA inputs can never mix, same rule as RTC CC.
- **Visual cue:** the three edit buttons render **red** when CA is selected and default **blue** for US (reuses RTC CC's `body.ca-region` theme).
- **All modals are region-aware:** Edit HIW, Edit T&Cs, Edit Terms Expiry & Terms IDs, and Edit Images read/save the selected region's keys and show the region in their titles. **Note: VIPBG HIW is region-scoped** (unlike RTC CC, where HIW is shared).

### CA Category-Specific Creatives (v2.27)
As of v2.27, **CA Slots and Tables offers use different Masthead and Promo Detail images.** The generated CA script derives the category word from the code's `SBG`/`TBG` token via the existing `GAME_CATEGORIES` map and uploads:
- **Slots offers (SBG):** `Slots Promo Detail.png` + `Slots Masthead.png`
- **Tables offers (TBG):** `Tables Promo Detail.png` + `Tables Masthead.png`

The Discover filename is unchanged (`{SBG|TBG}_B{bet}_G{get}.png`). The generic `Promo Detail.png` / `Masthead.png` are **no longer referenced by CA VIPBG scripts** — all four category files must exist in the CA image folder or the script fails at image upload. **US VIPBG is unchanged** (generic `Promo Detail.png` / `Masthead.png`). Confirmed live in NATS production for both Slots and Tables on Aug 14, 2026.

### Day Card Layout
- **Single card** (full width), titled `VIP Offer Library — Bet & Gets — {region}`. **+ Add Day is a no-op.**
- **CA offer rows** are **Category (Slots/Tables) → Bet → Get** with a per-row category dropdown and per-offer checkbox. Eight defaults shown (unselected): Slots $1,000/$100, $5,000/$500, $20,000/$1,000, $100,000/$5,000; Tables $2,000/$100, $10,000/$500, $25,000/$1,000, $100,000/$2,500.
- **US offer rows** are **Bet → Get** only (no category dropdown). Four defaults shown (unselected): $1,000/$100, $5,000/$500, $20,000/$1,000, $100,000/$5,000.
- Each row displays its Terms ID inline (red `no ID` when unmapped).
- No Campaign Name field, no ZIP attachment. **Send to XP and External Bonus have no toggles — both are always ON** (static note on the card; hardcoded `[True] * len(NAMES)` in the generated script; both regions). Days to Opt In 3 / Entitlement 3 / Settlement 7.

### Promo & Bonus Activation Window (month-anchored — identical to VIPDM; both regions)
- **Start:** 00:00:00 ET **one day before the typed date**
- **End:** **last day of the typed date's calendar month** at 23:59:59 ET **+ Days to Meet Entitlement**

Same rules as VIPDM: the typed day anchors only the start, a typed 1st starts in the previous month, changing Days to Entitlement changes both end dates, promo window = bonus activation window. Note the T&C's seventy-two (72) hour wagering-window language is legal copy about the offer mechanics (the customer's window after receiving the offer) and is **independent of the month-anchored activation window**.

### Terms ID Enforcement
Each active offer key must exist in the selected region's VIPBG Terms ID map. Non-matching active offers show a red warning on the row and **block script generation** with an alert.

**CA keys** are `{SBG|TBG}_B{bet}_G{get}` (the category prefix is collision insurance — a Slots and Tables tier could otherwise share a `B{bet}_G{get}` shape):

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

**US keys (v2.25)** are `B{bet}_G{get}` (no prefix — single category):

| Offer key | Bet | Get FanCash | Terms ID |
|---|---|---|---|
| `B1000_G100` | $1,000 | $100 | `CAS_10687` |
| `B5000_G500` | $5,000 | $500 | `CAS_10688` |
| `B20000_G1000` | $20,000 | $1,000 | `CAS_10689` |
| `B100000_G5000` | $100,000 | $5,000 | `CAS_10690` |

Terms IDs and promotional dates (CA default **08/17/26 → 12/31/26**; US default **07/01/26 → 12/31/26**) are edited per region via the **Edit Terms Expiry & Terms IDs** button inside the Edit T&Cs modal — dates render as `{start_date_long}` / `{end_date_long}` in the T&C.

### Fixed Copy
Title, Promo Header Text, and Bonus Description are all `From Your VIP Host` regardless of offer size or region. **Button 2 is region-branched (v2.25):** CA leaves label and link **intentionally EMPTY**; US **fills `Play Now!` → `/docs/usered/casgenericgamelist`**.

### T&C Data (Baked In)
**CA:** single Canadian (Ontario) legal template covering both categories — the Slots and Tables legal documents are word-for-word identical apart from dynamic values, so one template reproduces all 8 documents exactly (verified byte-for-byte against both legal docs). Dynamic fields: `{bet_fmt}`, `{get_fmt}`, `{terms_id}`, `{game_category_lower}` (`slots` / `table` — matching legal's asymmetric "slots games" / "table games"), `{game_category_upper}` (`SLOTS` / `TABLES`), `{start_date_long}`, `{end_date_long}`. Title line: `FANATICS CASINO (CANADA) – {game_category_upper} BET {bet_fmt} CAD, GET {get_fmt} CAD IN CASINO CREDIT ({terms_id})`. **Static by design:** the seventy-two (72) hour wagering-window language throughout, "within 72 hours of satisfying the wagering requirement," the seven (7) day Casino Credit expiry, and all §5 CAD examples — not templated. Curly quotes and en-dash from the legal documents are preserved.

**US (v2.25):** single **canonical** FanCash template. The CAS_10687 / CAS_10688 legal filings were **not** word-identical — four copy-edit inconsistencies were normalized into one canonical wording by stakeholder decision: (1) "Fanatics Sportsbook and Casino App" (missing space fixed), (2) "wagering **requirements**" (plural), (3) single period after "eligible to participate in this Promotion." in Limitations, (4) "Fanatics Casino aims to **consistently care** for our customers and promote responsible gambling practices." Everything else is verbatim legal copy (verified byte-level against both source documents with those fixes applied). Dynamic fields: `{bet_fmt}`, `{get_fmt}`, `{terms_id}`, `{start_date_long}`, `{end_date_long}` — **no game-category fields**. Title line: `FANATICS CASINO - VIP BET {bet_fmt} GET {get_fmt} FANCASH (ID: {terms_id})`. **Static by design:** MI/NJ/PA/WV eligibility, 21+, FBG Enterprises Opco, LLC entity, the 72-hour wagering and 3-day presentation language, the seven (7) day FanCash expiry, the FanCash Terms URL (`https://fanatics-one.com/fancashterms`), and the 1-800-GAMBLER responsible-gaming lines. ⚠️ The CAS_10689 / CAS_10690 filings were not supplied — the canonical template is assumed to cover them; verify on first use.

### Bonus Creation — Fixed Values

| Field | Value |
|---|---|
| Trigger type | Opt In (both regions) |
| Bonus type | CA: Casino Credit; US: FanCash (both flat amount — no percentage checkbox, no deposit entitlement, no Matched Deposit) |
| Description | From Your VIP Host |
| **Send To XP / External Bonus** | **Both always clicked ON** (both regions) |
| **Bet amount** | **Not entered anywhere** — enforced upstream (XP/segment) |
| Reward amount | Dynamic from `G{get}` in the code — CA: CAD row (`Casino Credit Amount`); US: USD row (`FanCash Amount`) |
| Settlement field | CA: `Days to Meet Casino_credit Settlement`; US: `Days to Meet Fancash Settlement` |
| Stake Chunk Sizes | 1 — **conditional fill** (filled if present, skipped with a terminal note if not; both regions) |
| **Platforms** | CA: **Web + STAC: Standalone Casino only** (all three COMBO tags removed, matching CA VIPDM). **US: standard tags — COMBO: Casino KEPT** (only the two Sportsbook tags removed) |
| Days to Opt In / Entitlement | 3 (default, editable; Entitlement also extends the end dates) |
| Days to Meet Settlement | 7 (default, editable) |
| Status Active | Checked |
| Activation | Typed date − 1 day 00:00 ET → month-end 23:59:59 ET + entitlement days |
| Reporting Platform | CAS (Casino) |
| Bonus Origin | Opt-In Bonus |
| Series | VIP |
| Type | Retention |
| Confirmation modal | CA: none observed; US: **expected** (standard FanCash BG shows one) — **handled defensively both ways** (waits 5s; confirms if one appears, continues with a terminal note if not; the US no-modal note reads "unexpected for FanCash — verify") |

> ⚠️ Series `VIP` and Type `Retention` use the same visible-dropdown-scoped anchored regex as VIPDM — the Type scoping is load-bearing (hidden Bonus Origin dropdown contains a `Retention` option).

Note: the `RET` token in the code name mirrors the **Type** field (Retention), not Bonus Origin — same as VIPDM. CA confirmed end-to-end (segment + promo + bonus) in NATS production Aug 12, 2026; v2.27 category-specific creatives confirmed live for both Slots and Tables Aug 14, 2026; US pending first live run (verify the confirmation-modal branch and Stake Chunk presence on the FanCash form).

---

## Daily Missions - Canada (v2.28; FS unlocked v2.30; NATS segments v2.31)

Twelfth offer type. Daily wager-and-get missions for Ontario, forked from the BG-RE two-phase architecture and pointed at **production for both systems**: NATS at `https://trading.1.betfanatics.com/` and Playmaker at `https://playmaker-internal.1.betfanatics.com/home` — **the first offer type to build bonuses in production Playmaker**. Fixed CA/Ontario (region row hidden). Script filename: `daily_missions_MMDDYY_HHMM.py`.

**As of v2.31, segments are built in NATS, not Playmaker.** Phase 1 is "NATS: Build Segments + Promotions" (segments first, then promos, one login); Phase 2 attaches the NATS segment to the Playmaker bonus via the **Include Segments → Existing Account Segments** field. The Playmaker "Create Account Segments" field — which v2.28–v2.30 used to create the segment on Save Bonus — is never touched (a chip there would now create a **duplicate** segment), and the entire **Exclude Segments** section is never touched (it would invert targeting).

### Weekday-Driven Model
The typed date's weekday determines everything — reward type, tiers, Terms IDs, game, stakes ladder, images, and title. There are no reward-type, game, or day dropdowns; mismatches are impossible by construction. **All seven weekdays generate as of v2.30.**

| Weekday | Reward | Game (Aggregator/Provider both PLAYTECH) | Terms IDs (tier order) |
|---|---|---|---|
| Tuesday / Thursday / Saturday | **Casino Credit** (shared set) | — | `CAS_CA_0019`–`0022` |
| Monday | Free Spins | Baa Baa Baa™ (BAA, search `Baa Baa Baa`) | `CAS_CA_0043`–`0046` |
| Wednesday | Free Spins | 4 Crazy Cluckers™ (4CC, search `4 Crazy Cluckers`) | `CAS_CA_0047`–`0050` |
| Friday | Free Spins | Mega Fire Blaze™: Legacy of the Tiger™ (MFB, search `Mega Fire Blaze`) | `CAS_CA_0051`–`0054` |
| Sunday | Free Spins | Blue Wizard: Cash Collect & Link™ (BWZ, search `Blue Wizard`) | `CAS_CA_0055`–`0058` |

### Free Spin Stakes Ladders (v2.30 — captured from PROD Playmaker)
Each game's `DMCA_GAMES` entry carries its **Free Spin Stakes ladder**, and spin values are validated **twice**:
- **Generation time:** a tier whose spin value is not in the weekday game's ladder shows a red `$X not in {token} stakes` warning on the row and **blocks script generation with an alert**.
- **Runtime (hard fail):** the script dumps the live stakes options from Playmaker and requires an **exact match** — if the value is absent it raises ("refusing to substitute"), marks the bonus FAILED, and continues to the next code. A nearby value is **never** selected, because the T&C and Terms ID legally state the spin value.

Ladders: **BAA / 4CC / BWZ** share the full 32-step ladder (0.10–1.00 by 0.10, 1.50–5.00 by 0.50, 6.00–10.00 by 1.00, 20.00–100.00 by 10.00); **MFB caps at 50.00** (27 steps — no 60–100). All four contain the three tier values 0.10 / 0.50 / 3.00. The stakes grid **offers CAD for Canada**. If Playmaker changes a ladder or the weekday games rotate, the HTML ladders/map must be updated (reach out to Adrian).

### Tiers

**CC (Tue/Thu/Sat — shared):** Bet $50 → Get $2 (`CAS_CA_0019`), Bet $125 → Get $5 (`CAS_CA_0020`), Bet $500 → Get $25 (`CAS_CA_0021`), Bet $5,000 → Get $300 (`CAS_CA_0022`).

**FS (all FS days — same tiers, per-day Terms IDs):** Bet $50 → 20 spins @ $0.10, Bet $125 → 50 @ $0.10, Bet $500 → 50 @ $0.50, Bet $5,000 → 100 @ $3.00.

### Day Card Layout
- **3 default cards.** Date (MMDDYY) with a live **weekday pill** beside the input — green `{Weekday} • Casino Credit` or amber `{Weekday} • Free Spins — {game}` — updating as the date is typed.
- **CC-day state:** four Bet → Get checkbox rows with inline Terms IDs; **Days to Use Reward** (default 7, per card); static notes for the always-ON toast and the weekday image filenames.
- **FS-day state (v2.30):** four active Bet → Spins checkbox rows with that weekday's Terms IDs inline; the same Days to Use Reward input; static notes for the FS toast and FS image filenames; a red per-row stakes warning when a spin value is not in the weekday game's ladder. FS state is stored separately from CC state (`dmcaFSOfferData` vs `dmcaOfferData`), so changing the date to a different weekday never mixes tiers.
- **Step badges: Segment + Promo + Bonus (v2.31)** — Segment defaults ON. The segment is built in NATS during Phase 1 and attached in Playmaker Step 2; `do_segment` is carried per code alongside `do_promo` / `do_bonus`.
- Edit HIW / Edit T&Cs buttons hidden (copy baked in — changes require reaching out to Adrian). Edit Images visible.

### Naming & Codes
- **CC code:** `MMDDYY_CAS_RET_BG_CA_CC_B{bet}_G{get}` (e.g. `120126_CAS_RET_BG_CA_CC_B125_G5`) — used identically for the NATS segment, NATS promo, and Playmaker bonus. ⚠️ Shape-identical to a standard BG CA code by stakeholder acceptance — no distinguishing token; avoid building a standard BG CA offer with the same date/bet/get.
- **FS code (v2.30):** `MMDDYY_CAS_RET_BG_CA_FS_B{bet}_G{spins}S_{value}V_{game}` with the spin value in cents (e.g. `113026_CAS_RET_BG_CA_FS_B125_G50S_10V_BAA`; the $500 tier is `50V`, the $5,000 tier `300V`).
- **No voucher codes** — the bonus is built in Playmaker.

### Windows
- **NATS promo: single-day window** — typed date **00:00:00 → 23:59:59 ET**. ⚠️ A new window pattern, unique to DMCA.
- **Playmaker bonus (v2.30): 00:01 ET typed date → 00:01 ET next day** (12:01 AM → 12:01 AM) — **all DMCA builds, CC included**; moved from 00:00 per stakeholder direction. The time fill uses the native-setter with a read-back, then a keyboard retype, then a `fill()`+Tab fallback, printing the final value either way.

### NATS Segments + Promos (Phase 1 — v2.31)
One manual NATS login covers both sub-phases; each is skipped if no code has that step enabled.
- **Segments (per `do_segment` code):** sidebar nav to Account Segments (`[data-menu-id$='-segments']`), then the standard NATS segment build used by every other segment-building offer type — name and code = code name, parent **AMELCO** via `dispatch_event("click")`, `#forBonus` checked, 2-second wait before OK.
- **Promos (per `do_promo` code):** Image only CTA / Overlay. **Title and Promo Header Text auto-derived: `{Weekday}'s Daily Mission`** (e.g. `Tuesday's Daily Mission`). **Bonus Tile ON. Button 2 empty.** No confirmation modal. Images from the local Drive folder with **underscored, weekday-prefixed filenames** (see Image System). A **pre-flight check** verifies every required image file on disk **before login** and aborts with a missing-file list.

### Playmaker Bonus (Phase 2 — Save Bonus IS clicked)
Steps 1–6: **Canada** jurisdiction (`#jurisdiction-ontario`), start/end dates via the calendar widget with both times forced to **00:01**, then **Step 2 attaches the NATS segment via the Include Segments → Existing Account Segments field (v2.31 — see below)**, wagering amount = bet, then **Step 4 branches on reward**:
- **CC:** Casino Credit radio, settlement amount = get.
- **FS (v2.30):** Bonus Spins radio (`value='free_spin'`), Number of Spins (`#fs-count`), Aggregator **PLAYTECH**, Provider **PLAYTECH**, Casino Game selected ™-tolerantly from the game's `search` string (actual option text printed), Free Spin Stakes selected by exact value from the combobox inside `[data-testid='free-spin-stakes-grid']` (⚠️ no aria-label — located via the grid; hard fail if absent).

Both: Days to Use Reward from the card, promotion selected by code name, **toast always ON** (`Promotion Complete!` / CC: `You have been awarded $X Casino Credit` / FS: `You have been awarded {spins} Bonus Spins to {game_name}` — **no quotes around the game name**), then **Save Bonus**. Each bonus runs in a per-bonus try/except with recovery navigation back to Bonus Setup on failure, and a **SAVED/FAILED results summary** prints at the end of Phase 2.

### Step 2 — Existing-Segment Attach (v2.31)
Playmaker's Step 2 Eligibility contains **four** cmdk fields — **Include Segments** and **Exclude Segments** sections, each with a "Create Account Segments" and an "Existing Account Segments" field. The labels are identical across sections and both Existing fields share `placeholder="Search..."`, so `attach_existing_segment()` is triple-guarded:
1. **Label-anchored primary:** the `<label>` with exact text `Existing Account Segments` whose ancestor section resolves to `Include Segments` (a container mentioning both Include and Exclude headings is a shared parent — rejected), followed via its `for=` attribute to the form-item container's `input[cmdk-input]`, climbing from the label with a `placeholder='Search...'` filter as backstop. The dynamic React/Radix ids (`:r5g:`, `radix-:r5j:` …) are session-scoped and deliberately **not** hardcoded.
2. **Hardened parent-walk fallback:** requires both the Existing-not-Create and Include-not-Exclude ancestor tests.
3. **Placeholder assertion on both paths:** Existing = `Search...`, Create = `Type and press Enter...` — a Create-style placeholder is rejected even on a structural match.

The code name is typed and the option whose text **exactly matches** is clicked — never Enter (that is the Create-field gesture). The **chip is then read back**: a `<span>` exactly equal to the code name must render inside the Include/Existing field within ~5s, or the script dumps every Step 2 combobox/input to the terminal and raises ("refusing to continue on an unverified attach") → per-bonus FAILED with recovery. NATS → Playmaker segment-sync lag is absorbed by a 12 × 5s (~60s) outer poll; if the segment never appears the script hard-fails — "refusing to fall back to Create Account Segments (would duplicate the NATS segment)". The Exclude section is structurally unreachable by the script.

### Okta Verify (FastPass) Login Fix — Both Phases
Playwright auto-denies permission prompts, which blocked Okta's "Access other apps and services on this device." Generated scripts launch a **persistent Chromium profile** (`~/.dm_canada_pw_profile`) with `--disable-features=LocalNetworkAccessChecks` and grant the `local-network-access` permission for `https://betfanatics.okta.com` (try/except with a WARN fallback to the flag alone). **Confirmed working in PROD Aug 14, 2026.** DMCA-only for now; candidate for back-porting to other offer types.

### HIW & T&C (Baked In — no edit modals)
**CC HIW (no "CAD" by design; "twenty-four (24) hours" static legal copy):**
```
1) Opt in
2) Place {bet_fmt}+ in cumulative cash wagers on any Fanatics Casino games within twenty-four (24) hours of receiving this offer
3) We'll reward you with {get_fmt} in Casino Credit to use on any game on Fanatics Casino
```
**FS HIW (v2.30 — live):** line 3 becomes `We'll reward you with {spins} Bonus Spins to use on the slots game {game_name}`.

**CC T&C:** full Ontario legal copy (FBG Enterprises Canada, Inc., 19+/Ontario, ConnexOntario), one template for all four CC tiers. Dynamic fields: `{bet_fmt}` (comma-formatted), `{get_fmt}`, `{terms_id}`, `{start_date_long}`, `{end_date_long}`. Title line: `FANATICS CASINO (CANADA) – BET {bet_fmt} CAD, GET {get_fmt} CAD IN CASINO CREDIT ({terms_id})`. **Static by design:** §5's $2/$20/$5 worked examples on all tiers, the CC-only "each time you are eligible" clauses in §4/§7, the $15-winnings-style examples, and the seven (7) day expiry. Byte-verified against the supplied CAS_CA_0019/0020 filings.

**FS T&C (v2.30 — live in the builder):** one template for all sixteen FS documents. Dynamic fields: `{bet_fmt}`, `{spins}`, `{spin_value}` (two-decimal: `$0.10` / `$0.50` / `$3.00`), `{game_name}`, `{terms_id}`, `{start_date_long}`, `{end_date_long}`. Title line: `FANATICS CASINO (CANADA) – BET {bet_fmt} CAD, GET {spins} BONUS SPINS ({terms_id})`. **Byte-verified against all sixteen FS filings (CAS_CA_0043–0058)** with two stakeholder-approved normalizations: (1) **exactly one space always follows `{game_name}`** — correcting the Monday filings' glued "Baa Baa Baa™and" defect (×3 spots each, plus Wed 0047/0048) and the double-space defect (Fri 0053 §10, Sun 0055 §5/§10); (2) **trailing whitespace stripped** (Wed 0049/0050, all Friday incl. 0054, all Sunday §4). The §5 "$15 CAD" winnings example is static on all tiers. All four FS days currently share the CC promotional window **08/17/26 → 12/31/26**, baked at generation time — the spec reserves five independent date pairs (Mon/Wed/Fri/Sun/CC-shared) for when the games rotate.

> ⚠️ CAS_CA_0021/0022 (the CC $500/$5,000 tiers) remain unsupplied — the CC template is assumed to cover them; verify on first use. All sixteen FS filings were supplied and verified.

### PROD Validation
- **Aug 14, 2026 (v2.28):** NATS promo `120126_CAS_RET_BG_CA_CC_B125_G5` built live end-to-end; Playmaker CC dry run through Step 6.
- **Aug 14–19, 2026 (v2.30 FS work):** iterative dry runs captured the FS selectors and all four stakes ladders; **full FS flows built live with Save Bonus clicked** — `120426` MFB (two tiers), `121626` 4CC, and a live same-day `081926` 4CC $50 build confirmed in-app. The v2.30 builder's rendered FS T&C is byte-identical to the PROD-validated test-script template.
- **Aug 20, 2026 (v2.31):** the new flow **PROD-validated end-to-end** — segment `101926_CAS_RET_BG_CA_FS_B5000_G100S_300V_BAA` built in NATS, then attached to the Playmaker bonus via Include → Existing Account Segments with the chip confirmed. This run also confirmed **NATS → Playmaker segment sync works** and that the field is located via the label-anchored/parent-walk path (the field is an `<input role="combobox">`, not a `button[role='combobox']`).
- ⚠️ **PROD cleanup pending:** the `120126` CC promo, the v2.30 FS validation builds (`120426` MFB ×2, `121626` 4CC, `081926` 4CC — promos, bonuses, and their auto-created segments), and the Aug 20 `101926…BAA` v2.31 validation build (NATS segment + promo + Playmaker bonus) all still exist in PROD — delete them or avoid reusing those exact codes.

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
| VIPDM (CA) | `MMDDYY_VCL_RET_DM_CA_CC_M{min}_{pct}M{max}` | `090126_VCL_RET_DM_CA_CC_M250_20M1000` |
| VIPDM (US) | `MMDDYY_VCL_RET_DM_US_FC_M{min}_{pct}M{max}` | `090126_VCL_RET_DM_US_FC_M250_20M1000` |
| VIPBG (CA) | `MMDDYY_VCL_RET_{SBG\|TBG}_CA_CC_B{bet}_G{get}` | `090126_VCL_RET_SBG_CA_CC_B1000_G100` |
| VIPBG (US) | `MMDDYY_VCL_RET_BG_US_FC_B{bet}_G{get}` | `090126_VCL_RET_BG_US_FC_B1000_G100` |
| DMCA (CC — Tue/Thu/Sat) | `MMDDYY_CAS_RET_BG_CA_CC_B{bet}_G{get}` | `120126_CAS_RET_BG_CA_CC_B125_G5` |
| DMCA (FS — Mon/Wed/Fri/Sun, v2.30) | `MMDDYY_CAS_RET_BG_CA_FS_B{bet}_G{spins}S_{value}V_{game}` | `113026_CAS_RET_BG_CA_FS_B125_G50S_10V_BAA` |

VIPBG CA category tokens: **SBG** = Slots Bet & Get, **TBG** = Table Games Bet & Get. VIPBG US uses the plain `BG` token (all US tiers are slots). In all VIP Offer Library codes, `FC` = FanCash / `CC` = Casino Credit, and the `_US_` / `_CA_` token routes amount fills to the USD / CAD row. **DMCA CC codes are shape-identical to standard BG CA codes** by stakeholder acceptance (no distinguishing token; the same code name is used for the NATS segment, NATS promo, and Playmaker bonus — as of v2.31 the segment is built in NATS); DMCA FS codes carry the spin value in cents and the weekday game token (BAA/4CC/MFB/BWZ).

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
| VIPDM (CA) | `VRDMCC{MMDDYY}{XXX}` | `VRDMCC090126X4T` |
| VIPDM (US) | `VRDMFC{MMDDYY}{XXX}` | `VRDMFC090126X4T` |
| VIPBG (US & CA) | `CLBGFC{MMDDYY}{XXX}` — shared with BG/BG-RE | `CLBGFC090126X4T` |
| DMCA | None — bonus is built in Playmaker | — |
| RAF Referee | `{jurisdiction}AQ{total_spins:04d}RFRE{XXX}` | `MIAQ0500RFREX3K` |
| RAF Referrer | `{jurisdiction}AQ{total_spins:04d}RFER{XXX}` | `MIAQ0500RFERX3K` |
| RAF Day 2–N | `{jurisdiction}AQ{total_spins:04d}DAY{n}{XXX}` | `MIAQ0500DAY2X3K` |
| SUO Day 2–N | None — no voucher codes | — |

> ⚠️ Voucher codes in NATS can **never** be reused, even if the segment, promotion, and bonus they were attached to have since been deleted. VIPBG's voucher is NATS-mandatory but unused for the offer itself; the random suffix keeps it unique across BG/BG-RE/VIPBG (both regions).

### Image Filenames
All offer types use: `Promo Detail.png`, `Masthead.png` — **with two exceptions: VIPBG CA (v2.27) uses category-specific `{Slots|Tables} Promo Detail.png` / `{Slots|Tables} Masthead.png`, and DMCA (v2.28) uses underscored, weekday-prefixed `{Weekday}_Promo_Detail.png` / `{Weekday}_Masthead.png`** (see below).
- RTC CC discover: `{amount}.png` (same convention in the US and Canada folders)
- RTC FS discover: `{spins}S_{value}V_{game}.png` (e.g. `20S_10V_7H2.png` for the v2.32 Hotstepper 2 game — new-game Discover creatives must be added to the RTC FS folder before building RTC FS offers on that game)
- BG discover: `B{bet}_G{get}.png`
- BG-RE discover: `B{bet}_G{get}.png`
- DM discover: `{pct}M{max}.png`
- LC-REACT discover: `{amount}.png`
- LC-CHURN-DM discover: `{pct}M{max}.png` (min deposit is not part of the filename)
- VIPDM discover (both regions): `{pct}M{max}.png` (min deposit is not part of the filename) — ⚠️ several names differ by one character: CA `10M250.png` vs `100M250.png`; US `10M500.png` vs `10M5000.png` and `20M500.png` vs `20M5000.png`; double-check those creatives
- VIPBG (CA): Masthead/Promo Detail are **category-specific as of v2.27** — `Slots Masthead.png` + `Slots Promo Detail.png` for SBG offers, `Tables Masthead.png` + `Tables Promo Detail.png` for TBG offers; discover: `{SBG|TBG}_B{bet}_G{get}.png` (e.g. `SBG_B1000_G100.png`) — the full offer key, so no one-character ambiguity
- VIPBG (US) discover: `B{bet}_G{get}.png` (e.g. `B1000_G100.png`) — no category prefix; generic `Masthead.png` / `Promo Detail.png`
- DMCA (v2.28/v2.30): **underscored, weekday-prefixed, weekday auto-derived from the typed date** — `{Weekday}_Promo_Detail.png`, `{Weekday}_Masthead.png`, CC discover `{Weekday}_B{bet}_G{get}.png` (e.g. `Tuesday_B125_G5.png`); FS discover **active as of v2.30**: `{Weekday}_B{bet}_G{spins}S_{value}V.png` with the spin value in cents (e.g. `Wednesday_B50_G20S_10V.png`; no game token — the weekday implies the game)
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
- VIPDM: `vcl_dm_MMDDYY_HHMM.py` (both regions)
- VIPBG: `vcl_bg_MMDDYY_HHMM.py` (both regions)
- DMCA: `daily_missions_MMDDYY_HHMM.py`

### Regions / Jurisdictions
- RTC CC: **US (default), CA only** (v2.22 — see Canada Support above)
- VIPDM: **US / CA** (v2.26 — dual region; **always defaults to US** when the offer type is selected)
- VIPBG: **US / CA** (v2.25 — dual region; defaults to CA)
- DMCA: **CA (Ontario) only — fixed** (v2.28; region row hidden)
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
| `vcl_bg_hiw` | Custom How It Works — VIPBG (CA — Canada) |
| `vcl_bg_tc` | Custom T&C — VIPBG (CA — Canada) |
| `vcl_bg_image_path` | Image folder path — VIPBG (CA — Canada) |
| `vcl_bg_terms_ids` | JSON object mapping offer key → Terms ID — VIPBG (CA) |
| `vcl_bg_terms_start_date` | Promotional start date in `MM/DD/YY` format — VIPBG (CA) |
| `vcl_bg_terms_end_date` | Promotional end date in `MM/DD/YY` format — VIPBG (CA) |
| `vcl_bg_hiw_us` | Custom How It Works — VIPBG (US — United States) |
| `vcl_bg_tc_us` | Custom T&C — VIPBG (US — United States) |
| `vcl_bg_image_path_us` | Image folder path — VIPBG (US — United States) |
| `vcl_bg_terms_ids_us` | JSON object mapping offer key → Terms ID — VIPBG (US) |
| `vcl_bg_terms_start_date_us` | Promotional start date in `MM/DD/YY` format — VIPBG (US) |
| `vcl_bg_terms_end_date_us` | Promotional end date in `MM/DD/YY` format — VIPBG (US) |
| `daily_missions_image_path` | Image folder path — Daily Missions - Canada (v2.28) |

SUO Day 2+ Spins and BG-RE have no localStorage keys. **Daily Missions - Canada has only the image-path key** — HIW, T&Cs, Terms IDs, promotional dates, the weekday→game map, and the stakes ladders are baked into the HTML (no edit modals by design). v2.31, v2.32, and v2.33 added no new keys. Clear Saved Settings removes all of the above, including every `_ca` and `_us` variant and `daily_missions_image_path`.

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
| 1 | BG-RE | NATS and Playmaker URLs are currently pointed at the **test environment**. Switch before using in production. (DMCA, which forked BG-RE's architecture, already uses production URLs for both.) |
| 2 | BG-RE | Free Spins reward type not yet implemented — different field structure in Playmaker. (DMCA's v2.30 FS implementation is the reference if BG-RE FS is ever needed.) |
| 3 | LC-REACT | T&C promotional periods and Terms IDs are baked in at generation time. Changes to any of the three tiers require regenerating the script. |
| 4 | LC-CHURN-DM | HIW, T&C, and Terms IDs are baked in at generation time — changes require regenerating the script. New offer sizes require adding a Terms ID to `CHURNDM_TERMS_IDS` in the HTML; offers without a configured Terms ID block script generation by design. **Matched Deposit is always checked as of v2.29** — Churn DM scripts must be generated from v2.29+; earlier versions leave the checkbox unchecked. |
| 5 | All templates | The Python scripts are generated inside JavaScript template literals, which **swallow single backslashes**. Any regex backslash intended for generated Python must be double-escaped in the HTML, and template regex changes must be verified by rendering through a real JS template literal and executing the resulting Python (root cause of the v2.20→v2.21 hotfix). The v2.22 CA T&C copy (11 tiers), the v2.23 VIPDM template (143 automated checks), the v2.24 VIPBG CA template (380 checks, incl. byte-level T&C comparison against both legal docs), the v2.25 VIPBG dual-region template (742 checks incl. byte-level US T&C comparison against both legal filings), the v2.26 VIPDM dual-region template (**692 automated checks + 35 sandboxed UI checks** — US + CA × 6 date edge cases, incl. byte-level US T&C comparison against all three supplied legal filings, CA output regression to v2.25, and hash-identity of all nine other builders), the v2.27 VIPBG creative-filename change (rendered and executed for both regions; CA SBG/TBG codes asserted the category filenames, US asserted the generic ones; all other builders byte-identical), the v2.28 DMCA builder (**31 content checks + 43 sandboxed UI checks** — rendered through a real JS engine, `ast.parse` + `py_compile`, T&C formatting incl. static §5 examples, weekday/leap-year pill logic, FS-day generation block, end-to-end generation via the real `onOfferTypeChange()` handler, all nine other builders hash-identical to v2.27), the v2.29 Churn DM Matched Deposit fix (inserted block round-tripped through a real JS template literal, resulting Python passed `ast.parse`; safe by construction; whole-file diff vs v2.28 is exactly three edits), the v2.30 DMCA FS unlock (**43 content checks + 10 sandboxed UI checks** — rendered CC+FS Python compiled/executed, FS T&C **byte-identical to the PROD-validated test template**, regex/`™`-strip escapes verified surviving JS-literal rendering, FS/CC/mixed generation through the real `generateScript()`, all 10 other builders hash-identical to v2.29; the harness caught a backtick-in-docstring bug before ship), the v2.31 DMCA segment-flow change (**17/17 content checks** — rendered with mixed CC + FS codes through a real JS engine → `ast.parse` + `py_compile`; all three JS discriminator blocks Include+Exclude aware; no Enter-press in the attach block; the create-chip flow fully removed; segments-before-promos ordering; AMELCO/00:01/FS guards untouched; all 10 other builders hash-identical to v2.30; the harness caught a single-backslash `\n` of the v2.20 defect class before ship), the v2.32 7H2 game addition (all three FS builders — SUO, RAF, and RTC FS — rendered through a real JS engine with 7H2 selected; resulting Python passed `py_compile`; the rendered `parse_casino_game_search` / `parse_casino_game_pattern` functions executed directly against positive and negative option texts; the `\u2122` escape verified surviving JS-literal rendering in the RAF/SUO outputs and the RTC FS literal-™ path verified decoding to the identical Python string; whole-file diff vs v2.31 is 33 additive lines; 9 builders hash-identical), and the v2.33 searchOverride fix (**28/28 executed checks** — all three FS builders rendered through a real JS engine covering 7H2/NJ — the exact PROD-failing case — plus 7FH/MI, TCE/WV no-override regression, RAF 7H2/PA, and RTC FS mixed three-game codes; all 5 rendered scripts passed `py_compile`; the rendered parse functions executed against the actual PROD option strings from the Aug 24 debug dumps, incl. the `Lucky Fire Blitz™ Hotstepper (MI)` near-collision, `DONOTUSE-` entries, and Hotstepper↔Hotstepper 2 cross-rejection; no-override TCE output confirmed functionally identical to v2.32 — typed search and pattern byte-equal; 37 added lines / 4 removed; 8 builders hash-identical) were all verified through this exact process; the VIPBG harnesses caught `\$`-escaping bugs of this class before they could ship. |
| 6 | RTC CC | Switching Region silently resets all day cards (no confirmation dialog, by design) — half-configured cards are cleared on region change. Same rule applies to VIPBG (v2.25) and VIPDM (v2.26). |
| 7 | VIPDM | HIW, T&C, Terms IDs, and promotional dates are baked in at generation time — changes via the region-aware Edit T&Cs / Terms modal require regenerating the script. New offer sizes require a Terms ID keyed `M{min}_{pct}M{max}` (per region), editable in the modal; active offers without one block script generation by design. |
| 8 | VIPDM | Dual region as of v2.26 (US: FanCash/USD, `Bonus (%)` checkbox, `Days to Meet Fancash Settlement`, COMBO: Casino kept, `VRDMFC` voucher, FanCash confirmation modal expected; CA: Casino Credit/CAD, `Casino Credit (%)`, all COMBO tags removed, `VRDMCC`, no modal). **US VIPDM scripts must be generated from v2.26+** — earlier versions were CA-locked. ⚠️ Region default differs from VIPBG: **VIPDM always selects US when the offer type is chosen** (VIPBG keeps a valid selection, else CA). Series `VIP` must exist in the NATS Series dropdown (confirmed present as of Aug 2026; dependency shared with VIPBG). |
| 9 | VIPBG | HIW, T&C, Terms IDs, and promotional dates are baked in at generation time — changes via the region-aware Edit T&Cs / Terms modal require regenerating the script. New offer sizes require a Terms ID keyed `{SBG\|TBG}_B{bet}_G{get}` (CA) or `B{bet}_G{get}` (US), editable in the modal; active offers without one block script generation by design. |
| 10 | VIPBG | Dual region as of v2.25 (US: FanCash/USD, Button 2 `Play Now!` → `/docs/usered/casgenericgamelist`, COMBO: Casino kept; CA: Casino Credit/CAD, Button 2 empty, all COMBO tags removed). **US VIPBG scripts must be generated from v2.25+** — earlier versions were CA-locked. Series `VIP` dependency shared with VIPDM (#8). |
| 11 | VIPBG (US) | The canonical US T&C was byte-verified against the CAS_10687 and CAS_10688 legal filings (with four stakeholder-approved copy-edit normalizations: "Casino App" spacing, plural "requirements," single period in Limitations, "consistently care" wording). The CAS_10689 / CAS_10690 filings were not supplied and are assumed to follow the same template — verify on first use. First live US run should also confirm the FanCash confirmation modal fires and whether Stake Chunk Sizes exists on the FanCash form (script handles both outcomes defensively). |
| 12 | VIPDM (US) | The canonical US T&C was byte-verified against the CAS_10677, CAS_10678, and CAS_10686 legal filings (with the stakeholder-approved normalizations: "Casino App" spacing, "Only your first Qualifying Deposit," trailing whitespace stripped, `FANATICS CASINO - ` title prefix). **The CAS_10679–CAS_10685 filings (seven of ten) were not supplied** and are assumed to follow the same template — verify on first use. First live US run should confirm the FanCash confirmation modal fires (the script handles both outcomes defensively; its absence prints an "unexpected — verify" note). |
| 13 | VIPBG (CA) | Category-specific creatives as of v2.27: CA scripts reference `Slots Masthead.png`, `Slots Promo Detail.png`, `Tables Masthead.png`, and `Tables Promo Detail.png` — the generic `Promo Detail.png` / `Masthead.png` are no longer referenced. All four category files must exist in the CA image folder or the script fails at image upload. The category word is derived at runtime from the code's `SBG`/`TBG` token, so no script regeneration is needed when adding new tiers within an existing category. Confirmed live for both categories Aug 14, 2026. |
| 14 | DMCA | **Free Spins days unlocked as of v2.30** — FS-day scripts require v2.30+ (and all DMCA scripts require v2.31+ per #18). Spin values are validated against per-game stakes ladders baked into the HTML (captured from PROD Aug 2026; MFB caps at 50.00); if Playmaker changes a ladder, the HTML must be updated (reach out to Adrian) — the runtime hard-fail ("refusing to substitute") protects against drift in the meantime. When the weekday games rotate: the `DMCA_GAMES` map (name/token/search/stakes), FS Terms IDs, FS creatives, and FS promotional dates all need updating; the spec reserves five independent date pairs (Mon/Wed/Fri/Sun/CC-shared) but the builder bakes one shared pair since all sixteen FS filings currently share the CC window. v2.30 also moved all DMCA Playmaker bonus times (CC included) from 00:00 to 00:01. |
| 15 | DMCA | HIW, T&C, Terms IDs, and promotional dates are baked into the HTML with **no edit modals** (Edit HIW / Edit T&Cs hidden by design) — changes require editing the HTML (reach out to Adrian) and regenerating scripts. The CC T&C was byte-verified against the supplied CAS_CA_0019/0020 filings and the FS T&C against **all sixteen FS filings (CAS_CA_0043–0058)** — each with two approved normalizations (exactly one space after `{game_name}`; trailing whitespace stripped). **CAS_CA_0021/0022 (the CC $500/$5,000 tiers) remain unsupplied** — verify on first use. |
| 16 | DMCA | CC code names are shape-identical to standard BG CA codes (`MMDDYY_CAS_RET_BG_CA_CC_B{bet}_G{get}`) — accepted by stakeholder decision, no distinguishing token. Avoid building a standard BG CA offer with the same date/bet/get. ⚠️ **PROD cleanup pending:** the Aug 14 validation promo `120126_CAS_RET_BG_CA_CC_B125_G5`, the v2.30 FS validation builds (`120426` MFB ×2, `121626` 4CC, `081926` 4CC — promos, bonuses, and their auto-created segments), and the Aug 20 v2.31 validation build `101926_CAS_RET_BG_CA_FS_B5000_G100S_300V_BAA` (NATS segment + promo + Playmaker bonus) still exist in PROD; delete them or avoid reusing those exact codes. |
| 17 | DMCA | The Okta Verify (FastPass) persistent-profile launch (`~/.dm_canada_pw_profile` + `--disable-features=LocalNetworkAccessChecks` + `local-network-access` grant) is DMCA-only; all other offer types still use plain `chromium.launch()`. Back-porting is a candidate follow-up if Okta blocks appear elsewhere. |
| 18 | DMCA | **v2.31 flow dependency: NATS → Playmaker segment sync.** A NATS-built AMELCO segment must become selectable in Playmaker's Include → Existing Account Segments list (confirmed working in PROD Aug 20, 2026; sync *lag* is absorbed by the ~60s poll). If sync ever breaks entirely, Phase 2 hard-fails by design rather than creating a Playmaker-side duplicate. The Include/Exclude discrimination and the chip read-back key off the visible strings `Include Segments` / `Exclude Segments` / `Existing Account Segments` / `Create Account Segments` and the `Search...` / `Type and press Enter...` placeholders — a Playmaker copy change to any of these makes the lookup **fail closed** (Step 2 field dump + FAILED bonus, never the wrong field; send the dump to Adrian). **DMCA scripts must be generated from v2.31+** — v2.28–v2.30 create the segment Playmaker-side via the Create field, which would duplicate the NATS segment. |
| 19 | RTC FS / RAF / SUO | **7H2 (7's Fire Blitz™ Hotstepper 2) added v2.32; 7FH/7H2 `searchOverride` added v2.33 — scripts selecting either game must be generated from v2.33+.** NATS's Casino Game search is **hard-capped at ~12 rendered results with no scrollable overflow** (established via repr-level dropdown dumps, Aug 24, 2026 — not client-side virtualization; the missing entries are absent from the result set, not below the fold). The broad pre-™ search `7's Fire Blitz` shared by 7FH and 7H2 matches dozens of catalog entries, so under v2.20–v2.32 only the state entries that happened to land inside the cap were selectable — 7H2 confirmed failing on NJ and PA, passing on MI by result-ordering luck. v2.33 scripts type `{searchOverride} ({jurisdiction})` instead (`Hotstepper 2 (NJ)` → exactly 1 result; `Hotstepper (MI)` → 2, with the `Lucky Fire Blitz™ Hotstepper (MI)` impostor correctly rejected by the unchanged anchored pattern) — NATS's search is contiguous-substring matching over the full option text, suffix included. The 7H2 stakes ladder (19 steps, 0.10–50.00) was supplied Aug 21, 2026 — if NATS's actual dropdown differs, the exact-match spin-value selection will fail visibly rather than pick a nearby value; update the `FS_GAMES` ladder (reach out to Adrian). RTC FS Discover creatives for 7H2 (`{spins}S_{value}V_7H2.png`) must be added to the RTC FS folder before building RTC FS offers on this game. ⚠️ PROD cleanup: the Aug 21 validation segment `090126_CAS_ACQ_SUO_DAY2_MI_FS_20` and the segments/early-day bonuses left by the failed Aug 24 `082426` SUO runs exist in PROD — delete or avoid reusing those codes, and resume interrupted runs from the failed day rather than rerunning from day 2. |
| 20 | RTC FS / RAF / SUO | **⚠️ 7P5 remains exposed to the search cap.** `7's Fire Blitz™ Power 5 Jackpot Royale™ Express` also types the broad `7's Fire Blitz` search (its pre-™ split) — in the Aug 24 dumps its entries rendered for MI and WV but not NJ/PA. The v2.33 suffix fix does not transfer directly: `Power 5 ({JUR})` is **not a contiguous substring** of the option text (`Jackpot Royale™ Express` intervenes), and NATS's search requires contiguity. Candidate override: `Express ({JUR})` (contiguous; returns the small `…Express ({JUR})` family, which the anchored pattern disambiguates) — pending a one-query PROD dropdown test before adding to a future version. Until then, build 7P5 only on MI/WV, or manually confirm the target state's entry appears in the first ~12 results for `7's Fire Blitz`. **WWE** (`WrestleMania`) and **TCE** (`Triple Cash Eruption`) are believed narrow enough to stay under the cap but are not dump-verified; the Aug 24 repr-dump debug script pattern verifies any game/state in one run. NATS's catalog also contains **curly-apostrophe game families** (`7's Fire Blitz™ Power Force 5`) invisible to straight-apostrophe searches — any future `FS_GAMES` addition must have its exact catalog spelling captured first. |

---

## Image System

### RTC Top Up - Casino Credit
Jurisdiction-scoped as of v2.22. Path stored in `localStorage` as `rtc_image_path` (US) / `rtc_image_path_ca` (CA). Edit Images button present; the modal reads/saves the selected region's path, shows the region in its title, and its Drive link follows the region.
**US default path (Adrian's machine):** `.../Marketing Automations/Lifecycle Automations/RTC Top Up`
**US Google Drive folder:** https://drive.google.com/drive/folders/1Dlpa3xZTHjzlwHgayPSS-MZJ8-DQTmhh
**CA default path (Adrian's machine):** `.../Marketing Automations/Lifecycle Automations/RTC Top Up - Canada`
**CA Google Drive folder:** https://drive.google.com/drive/folders/1W6ulHKv3ted9ZLNKzPgEbRoNCK_wFz6O

> ⚠️ Each user must set their own image path via the **Edit Images** button before generating a script — separately for US and for CA if they build both (applies to RTC CC, VIPDM, and VIPBG), and once for Daily Missions - Canada. Paths are saved to their browser's localStorage and only need to be set once per machine per region.

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
Region-scoped as of v2.26. Path stored in `localStorage` as `vcl_dm_image_path` (CA) / `vcl_dm_image_path_us` (US). Edit Images button present; the modal reads/saves the selected region's path, shows the region in its title, and its Drive link follows the region. Under the **VIP Automations** shared drive branch.
**CA default path (Adrian's machine):** `/Users/adrian.vandecamp/Library/CloudStorage/GoogleDrive-adrian.vandecamp@betfanatics.com/Shared drives/Marketing Automations/VIP Automations/Offer Library/Canada/Deposit Matches`
**CA Google Drive folder:** https://drive.google.com/drive/folders/1v9DEVtCNQPCgUkJsWtsXn7UJVIyBYtr4
**CA image filenames:** `Promo Detail.png`, `Masthead.png`, `{pct}M{max}.png` (e.g. `20M500.png`). Min deposit is not part of the Discover filename. Default-tier Discover set: `50M250.png`, `100M250.png`, `10M250.png`, `10M2500.png`, `10M5000.png`, `20M500.png`, `20M1000.png`, `20M2000.png`, `20M5000.png`.
**US default path (Adrian's machine):** `/Users/adrian.vandecamp/Library/CloudStorage/GoogleDrive-adrian.vandecamp@betfanatics.com/Shared drives/Marketing Automations/VIP Automations/Offer Library/USA/Deposit Matches`
**US Google Drive folder:** https://drive.google.com/drive/folders/1W8GLxrZFBMotLP_WPzAE04KPPb__BxIX
**US image filenames:** `Promo Detail.png`, `Masthead.png`, `{pct}M{max}.png`. Default-tier Discover set: `50M250.png`, `100M250.png`, `10M500.png`, `10M1000.png`, `10M2500.png`, `10M5000.png`, `20M500.png`, `20M1000.png`, `20M2000.png`, `20M5000.png`.
> ⚠️ Several Discover filenames differ by one character — CA: `10M250.png` vs `100M250.png`; US: `10M500.png` vs `10M5000.png` and `20M500.png` vs `20M5000.png`. Double-check those creatives; a mislabeled file uploads the wrong tier's image without erroring.

### VIP Offer Library - Bet & Gets
Region-scoped as of v2.25; CA Masthead/Promo Detail category-scoped as of v2.27. Path stored in `localStorage` as `vcl_bg_image_path` (CA) / `vcl_bg_image_path_us` (US). Edit Images button present; the modal reads/saves the selected region's path, shows the region in its title, and its Drive link follows the region. Under the **VIP Automations** shared drive branch, sibling to Deposit Matches.
**CA default path (Adrian's machine):** `/Users/adrian.vandecamp/Library/CloudStorage/GoogleDrive-adrian.vandecamp@betfanatics.com/Shared drives/Marketing Automations/VIP Automations/Offer Library/Canada/Bet & Gets`
**CA Google Drive folder:** https://drive.google.com/drive/folders/1tkMs1dx-gszSGzlBHxgMYvOYv2l-2UjO
**CA image filenames (v2.27 — category-specific):** `Slots Promo Detail.png` + `Slots Masthead.png` (used for all SBG offers), `Tables Promo Detail.png` + `Tables Masthead.png` (used for all TBG offers), and a Discover image `{SBG|TBG}_B{bet}_G{get}.png` — the full offer key (e.g. `SBG_B1000_G100.png`, `TBG_B100000_G2500.png`), so no one-character ambiguity. The generic `Promo Detail.png` / `Masthead.png` are no longer referenced by CA VIPBG scripts. Default-tier Discover set: `SBG_B1000_G100.png`, `SBG_B5000_G500.png`, `SBG_B20000_G1000.png`, `SBG_B100000_G5000.png`, `TBG_B2000_G100.png`, `TBG_B10000_G500.png`, `TBG_B25000_G1000.png`, `TBG_B100000_G2500.png`.
**US default path (Adrian's machine):** `/Users/adrian.vandecamp/Library/CloudStorage/GoogleDrive-adrian.vandecamp@betfanatics.com/Shared drives/Marketing Automations/VIP Automations/Offer Library/USA/Bet & Gets`
**US Google Drive folder:** https://drive.google.com/drive/folders/13neRGeYM4JLa98p827vbepp-eKu2_hXq
**US image filenames:** `Promo Detail.png`, `Masthead.png`, `B{bet}_G{get}.png` (no category prefix — all US tiers are slots). Default-tier set: `B1000_G100.png`, `B5000_G500.png`, `B20000_G1000.png`, `B100000_G5000.png`.

### Daily Missions - Canada (v2.28; FS images active v2.30)
Path stored in `localStorage` as `daily_missions_image_path`. Edit Images button present (Drive link follows). Under the **Bau Automations** shared drive branch.
**Default path (Adrian's machine):** `/Users/adrian.vandecamp/Library/CloudStorage/GoogleDrive-adrian.vandecamp@betfanatics.com/Shared drives/Marketing Automations/Bau Automations/Daily Missions`
**Google Drive folder:** https://drive.google.com/drive/folders/1w-o2bP7em8fkT86lwOtCu66jO57igRLh
**Image filenames (underscored, weekday-prefixed — weekday auto-derived from the typed date):** `{Weekday}_Promo_Detail.png`, `{Weekday}_Masthead.png`, CC discover `{Weekday}_B{bet}_G{get}.png` (e.g. `Tuesday_Promo_Detail.png`, `Tuesday_B125_G5.png`). FS discover **active as of v2.30**: `{Weekday}_B{bet}_G{spins}S_{value}V.png` with the spin value in cents (e.g. `Wednesday_B50_G20S_10V.png`; no game token — the weekday implies the game). Each usable weekday needs its own Promo Detail + Masthead + one discover per selected tier. The generated script **pre-flight checks every required file before login** and aborts with a missing-file list if any are absent. FS creatives for all four FS weekdays are confirmed present in the folder.

---

## Sidebar Navigation

All generated scripts navigate between screens by hovering sidebar icons using `data-menu-id` attribute selectors and clicking the target submenu item via `dispatch_event("click")`.

| Screen | Sidebar selector | Submenu item |
|---|---|---|
| Account Segments | `[data-menu-id$='-segments']` | `Account Segments` |
| Promos | `[data-menu-id$='-cms']` | `Promos` |
| Bonus Manager | `[data-menu-id$='-bonuses']` | `Bonus Manager` |

Each nav step moves the mouse to a safe zone (600, 400), hovers the sidebar icon, then uses `dispatch_event("click")` on the submenu item. Up to 5 retry attempts with safe-zone reset. Applies to all offer types that use NATS. DMCA's NATS phase uses the Account Segments and Promos navs as of v2.31 (its bonus lives in Playmaker, so Bonus Manager is never used).

---

## Per-Code Step Toggles

### RTC CC / RTC FS / BG / DM / LC-REACT / LC-CHURN-DM / VIPDM / VIPBG / DMCA
Every day card shows three toggleable step badges: **Segment**, **Promo**, **Bonus**. The generated script includes `DO_SEGMENT`, `DO_PROMO`, and `DO_BONUS` boolean arrays (DMCA carries `do_segment` / `do_promo` / `do_bonus` per `CODES` entry). Phases are skipped if no codes have that step enabled. DMCA gained its Segment badge in v2.31 — its segment is built in NATS during Phase 1 and attached in Playmaker Step 2 via the Include → Existing Account Segments field.

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
**Promo** and **Bonus** badges only — no Segment badge. The segment is created inside Playmaker as part of the bonus flow (typed into the "Create Account Segments" field; the chip creates the segment on Save Bonus). ⚠️ This is the pre-v2.31 DMCA pattern — it remains correct for BG-RE only.

---

## Eastern Time Enforcement

All generated scripts enforce Eastern Time (America/New_York) via Python's `zoneinfo` module. **Python 3.9+ is required.**

---

## Platform Tags — Bonus Creation

NATS pre-populates platform tags on every new bonus. Generated scripts for the eight pre-v2.23 NATS offer types remove `COMBO: Sportsbook` and `COMBO: Sportsbook And Casino`, leaving:

- `STAC: Standalone Casino`
- `Web`
- `COMBO: Casino`

**VIP Offer Library exception (v2.23/v2.24):** **CA** VIPDM and **CA** VIPBG remove **all three** COMBO tags (including `COMBO: Casino`), leaving only `Web` + `STAC: Standalone Casino`. They are the only flows that remove `COMBO: Casino`. **US VIPBG (v2.25) and US VIPDM (v2.26) follow the standard pattern** — only the two Sportsbook tags are removed and `COMBO: Casino` is kept.

BG-RE and DMCA bonuses are built in Playmaker, not NATS, so platform tags do not apply. (DMCA's NATS segment, added in v2.31, is a segment — platform tags apply to bonuses only.)

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
| **Matched Deposit** | **Always checked** (inside the DEPOSIT entitlement panel; v2.29 — same block as VIPDM, with an explicit visibility wait since the panel renders after Entitlement Type is set) |
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

> ⚠️ This differs from all other offer types (BG, DM, RTC FS, RAF, SUO, LC-REACT, LC-CHURN-DM, VIPDM, VIPBG, DMCA), which use a midnight-anchored, 72-hour, 10-day, month-end, or single-day window. The window is identical for US and CA.

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

| Acronym | Full Name | Search Name | Search Override (v2.33) | Aggregator | Provider | Deeplink |
|---|---|---|---|---|---|---|
| TCE | Triple Cash Eruption | Triple Cash Eruption | — | IGT | IGT_Rgs | /casino_game/200-1700-081 |
| 7FH | 7's Fire Blitz™ Hotstepper | 7's Fire Blitz™ Hotstepper | `Hotstepper` | WHG | WHITEHATSTUDIOS | /casino_game/WHS_7sFireBlitzHotStepper95 |
| 7H2 | 7's Fire Blitz™ Hotstepper 2 | 7's Fire Blitz™ Hotstepper 2 | `Hotstepper 2` | WHG | WHITEHATSTUDIOS | /casino_game/WHS_7sFireBlitzHotStepper2US94 |
| 7P5 | 7's Fire Blitz™ Power 5 Jackpot Royale™ Express | 7's Fire Blitz™ Power 5 Jackpot Royale™ Express | — ⚠️ see Known Limitation #20 | WHG | WHITEHATSTUDIOS | /casino_game/WHS_7sFireBlitzPower5JRE92 |
| WWE | WrestleMania: Road to Gold | WrestleMania™: Road To Gold | — | WHG | WHITEHATSTUDIOS | /casino_game/WHS_WrestlemaniaRoadToGoldUS94 |

> ⚠️ Game search names use a straight apostrophe (`'`) not a curly apostrophe (`'`). Note that NATS's catalog contains other game families that **do** use curly apostrophes (e.g. `7's Fire Blitz™ Power Force 5`) — a straight-apostrophe search returns zero results for those, so any future `FS_GAMES` addition must have its exact catalog spelling captured first (repr-dump a real dropdown).

> ⚠️ **NATS game option names include trademark symbols as of August 2026** (e.g. `Triple Cash Eruption™ (WV)`), and **NATS's game search is hard-capped at ~12 rendered results** (established Aug 24, 2026). As of v2.33, scripts type `{searchOverride} ({jurisdiction})` for games that carry a `searchOverride` (7FH: `Hotstepper (MI)`; 7H2: `Hotstepper 2 (NJ)` — NATS matches contiguous substrings of the full option text, suffix included, so these return near-unique results within the cap), and the pre-™ portion of the Search Name for games without one (v2.20 behavior, unchanged). Selection then uses an anchored regex where every `™` is optional but the jurisdiction suffix is required — unchanged since v2.20/v2.21, and still the step that owns correctness (it rejects the `Lucky Fire Blitz™ Hotstepper` near-collision, `DONOTUSE-` entries, wrong states, and cross-matches between 7FH and 7H2). The Search Name column above does **not** need editing when NATS adds/removes `™`. Applies to RTC FS, RAF, and SUO. (The DMCA weekday games are a separate map used by the Playmaker FS flow, which uses its own ™-tolerant matcher — live as of v2.30.)

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

Fully integrated into the HTML tool as of v2.15. See `Technical_Reference_v2_33.md` for full selector reference and Playmaker field spec.

> ⚠️ Currently using **test environment** URLs. Switch before using in production. (Daily Missions - Canada, which forked this architecture, already runs both phases in production.)

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
| VIPDM (CA) | No modal — CA script contains no modal block (production-confirmed flow, unchanged from v2.23) |
| VIPDM (US) | Expected (FanCash) — handled defensively; absence prints an "unexpected — verify" terminal note |
| VIPBG (CA) | No modal observed — handled defensively (confirms if one ever appears, continues with a terminal note if not) |
| VIPBG (US) | Expected (FanCash) — handled defensively; absence prints an "unexpected — verify" terminal note |
| DMCA | No (NATS promo has none; the bonus is saved in Playmaker, which has no NATS-style confirmation modal) |

### Promotion Tile Selection
Uses regex anchored exact match scoped to `.ant-select-dropdown:not(.ant-select-dropdown-hidden)`. Applied to all offer types that use a Promotion Tile. SUO, BG-RE, and DMCA do not use a Promotion Tile in NATS.

### Segment Dropdown Selection
LC-REACT, LC-CHURN-DM, VIPDM, and VIPBG use the same anchored regex pattern as the Promotion Tile — scoped to `.ant-select-dropdown:not(.ant-select-dropdown-hidden)` — to avoid hitting the hidden search mirror span that NATS injects alongside the input. **DMCA (v2.31): the segment is built in NATS during Phase 1 (standard `create_segment` — see the DMCA section) and attached in Playmaker Step 2 via `attach_existing_segment()`; the Playmaker "Create Account Segments" field is never used.** BG-RE still uses the pre-v2.31 pattern: the code name is typed into Playmaker's Create Account Segments field and the chip creates the segment on Save Bonus.

### DMCA Existing-Segment Attach (v2.31)
Playmaker Step 2 Eligibility contains **four** cmdk fields — Include/Exclude Segments × Create/Existing Account Segments — with identical labels across sections and identical `Search...` placeholders on both Existing fields. `attach_existing_segment()` locates the **Include → Existing** field via a label-anchored primary (exact label text, Include-section ancestor test, `for=` attribute followed to the form-item's `input[cmdk-input]`; dynamic React/Radix ids never hardcoded), a hardened parent-walk fallback (Existing-not-Create AND Include-not-Exclude), and a placeholder assertion on both paths (`Search...` accepted, `Type and press Enter...` rejected). It clicks the exactly matching option (never Enter), **reads back the chip** (a `<span>` exactly equal to the code name inside the Include/Existing field, ~5s poll), and polls up to ~60s for NATS → Playmaker segment sync. Every failure mode **fails closed** — Step 2 field dump + FAILED bonus — never the Create field, never the Exclude section, never an unverified attach. PROD-validated Aug 20, 2026.

### Series / Type Dropdown Selection (VIPDM / VIPBG)
VIPDM and VIPBG select Series `VIP` (typed search + anchored `^VIP$`) and Type `Retention` (anchored `^Retention$`), both scoped to the visible dropdown. The Type scoping is required: the hidden Bonus Origin dropdown also contains a `Retention` option, so a page-wide exact-text click would match two elements and crash with a Playwright strict mode violation. VIPDM and VIPBG are the only offer types that select `Retention` for Type. Applies to both regions of both offer types.

### Currency-Row Amount Fill (v2.18+)
Bonus amounts, DM/LC-CHURN-DM/VIPDM deposit fields, the VIPDM reward amount (Casino Credit for CA, FanCash for US), the VIPBG flat reward amount (Casino Credit for CA, FanCash for US), and FS spin values are filled into the jurisdiction's currency row (`tr[data-row-key='USD'|'CAD']`), never GBP, with a legacy single-input fallback. Terminal output prints the currency selected. See **Multi-Currency Amount Tables** above. Not applicable to DMCA (Playmaker bonus — plain `wageringAmount` / `settlementAmount` / `#fs-count` inputs; the Playmaker Free Spin Stakes grid shows CAD for Canada natively).

### Casino Game Selection (v2.20/v2.21; searchOverride v2.33)
RTC FS, RAF, and SUO type a search whose only job is to get the target entry inside **NATS's ~12-result search cap** (`parse_casino_game_search()`): games with a `searchOverride` type `{override} ({jurisdiction})` (7FH / 7H2); games without one type the pre-™ portion of the game's Search Name (TCE / 7P5 / WWE — unchanged v2.20 behavior). The dropdown option is then selected with an anchored, ™-tolerant, jurisdiction-required regex (`parse_casino_game_pattern()` — unchanged in v2.33; the match step, not the search step, owns correctness). Terminal output prints the actual option text clicked. 7FH and 7H2's anchored patterns are structurally cross-exclusive. ⚠️ 7P5 shares the broad search and remains exposed to the cap — see Known Limitation #20.

### DMCA Free Spins Step 4 (v2.30)
FS codes branch Step 4 of the Playmaker bonus: Bonus Spins radio (`button[role='radio'][value='free_spin']`) → Number of Spins (`#fs-count`) → Aggregator **PLAYTECH** → Provider **PLAYTECH** (Radix comboboxes located by aria-label substring; each dropdown's options are dumped to the terminal) → Casino Game matched ™-tolerantly (™/® stripped, whitespace collapsed, case-insensitive substring on the game's `search` string; actual option text printed) → **Free Spin Stakes** combobox located inside `[data-testid='free-spin-stakes-grid']` (⚠️ **no aria-label** — must be found via the grid; the grid's row text is dumped first). The exact `spin_value` must exist in the dropdown — **if absent, the script dumps the available options and raises ("refusing to substitute")**, marking the bonus FAILED and continuing; a nearby value is never selected because the T&C and Terms ID legally state the spin value. Each Playmaker bonus runs in a per-bonus try/except with recovery navigation back to Bonus Setup, and a SAVED/FAILED results summary prints at the end of Phase 2.

### DMCA Playmaker Bonus Times (v2.30)
All DMCA builds (CC and FS) set the Playmaker start/end times to **00:01** (12:01 AM typed date → 12:01 AM next day). The fill uses the React native-setter first, reads the value back, retries via keyboard typing (`click ×3` + `type` + Enter) if it did not stick, then falls back to `fill()` + Tab — the final value is printed either way. The NATS promo window is unchanged (00:00:00 → 23:59:59 ET).

### Matched Deposit Checkbox (VIPDM / LC-CHURN-DM as of v2.29)
VIPDM (both regions) and LC-CHURN-DM check the **Matched Deposit** checkbox inside the DEPOSIT entitlement panel, with an explicit visibility wait since the panel renders only after Entitlement Type is set. The Churn DM step was added in v2.29 using the production-confirmed VIPDM block verbatim. Terminal output prints `OK Matched Deposit: checked`. Standard DM does not include this step.

### VIPBG CA Category Creatives (v2.27)
CA VIPBG scripts derive the category word (`Slots` / `Tables`) from the code's `SBG`/`TBG` token via `GAME_CATEGORIES[cat]["hiw"]` at runtime and upload `{category} Promo Detail.png` / `{category} Masthead.png` instead of the generic filenames. US VIPBG scripts route through the same variables but resolve to the generic `Promo Detail.png` / `Masthead.png`. Discover filenames unchanged in both regions.

### DMCA Single-Day Promo Window & Weekday Images (v2.28/v2.30)
DMCA's `parse_dates(code)` returns the typed date `00:00:00 ET → 23:59:59 ET` — the only single-day window in the tool. The weekday is derived at runtime from the code's date and drives the promo Title/Header (`{Weekday}'s Daily Mission`) and all three image filenames via `image_files_for(c)` — CC: `{Weekday}_Promo_Detail.png` / `{Weekday}_Masthead.png` / `{Weekday}_B{bet}_G{get}.png`; FS (v2.30): the same Promo Detail/Masthead plus `{Weekday}_B{bet}_G{spins}S_{vtok}.png`. A pre-flight loop verifies every required file on disk before login and aborts with a missing-file list.

### DMCA Okta Verify Launch (v2.28)
DMCA scripts use `launch_persistent_context` (profile `~/.dm_canada_pw_profile`) with `--disable-features=LocalNetworkAccessChecks` and a `local-network-access` permission grant for `https://betfanatics.okta.com`, resolving the Playwright-auto-denied prompt that blocked Okta FastPass. Both phases share the one persistent context (NATS page + a second Playmaker page). All other offer types use plain `chromium.launch()`.

### Create Bonus Button — Multi-Code Runs
`create_bonus` polls up to 120 × 500ms (60 seconds) for the button to re-enable.

### AMELCO Dropdown Selection
All `create_segment` functions select AMELCO via `dispatch_event("click")` rather than `.click()` to bypass Ant Design pointer interception. A 2-second wait is inserted after `#forBonus` is checked and before OK is clicked. As of v2.31 this includes DMCA's `create_segment` (identical pattern, ASCII terminal prints).

### Casino Credit Deposit-Match Form (LC-CHURN-DM / VIPDM CA)
Under the Casino Credit bonus type, the deposit-match percentage checkbox is labeled **`Casino Credit (%)`** (not `Bonus (%)` as on the FanCash DM form). The rest of the field structure is identical to DM. Generated scripts include label fallbacks: the summary percentage field tries `Bonus %` then `Casino Credit %`, and the amount summary card tries the `Bonus` then `Casino Credit` card titles. The settlement field targets `Days to Meet Casino_credit Settlement`. **Both LC-CHURN-DM (v2.29) and VIPDM check the Matched Deposit checkbox** inside the DEPOSIT entitlement panel (with an explicit visibility wait, since the panel renders only after Entitlement Type is set).

### FanCash Deposit-Match Form (VIPDM US — v2.26)
US VIPDM uses the standard-DM FanCash matched-deposit form: bonus type `FanCash`, percentage checkbox **`Bonus (%)`**, settlement field `Days to Meet Fancash Settlement`. The same summary-label and card-title fallbacks apply (they resolve to `Bonus %` / `Bonus` on this form). Matched Deposit is checked identically to CA. Reward, Minimum Deposit, and Maximum Deposit fill the **USD** row.

### VIP Offer Library Month-Anchored Window (VIPDM / VIPBG)
`parse_dates(code, entitlement_days)` is the only date function that takes a per-card input: start = typed date − 1 day at 00:00:00 ET; end = last calendar day of the typed date's month at 23:59:59 ET + Days to Meet Entitlement (`calendar.monthrange`; leap years and year crossovers handled; the typed day anchors only the start). Applied identically to the promo dates and the bonus Activation Dates for both offer types and both regions.

### Image Upload Timing
After file selection, scripts wait **8 seconds** before clicking OK, then 2 seconds after OK. Each image upload takes ~10 seconds total; 3 images per promo = ~30 seconds per promo build. Applies to RTC CC, RTC FS, BG, DM, RAF, BG-RE, LC-REACT, LC-CHURN-DM, VIPDM, VIPBG, and DMCA. SUO has no images.

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

### VIP Offer Library - Deposit Matches — CA — Canada
**HIW:** Legal-approved template — saved as `vcl_dm_hiw` (region-scoped as of v2.26). Dynamic fields: `{min_fmt}`, `{pct}`, `{max_fmt}`. No opt-in deadline line (the promo window bounds the offer) and no "See below" trailer, by design:
```
1. Opt-in to the promotion
2. Make a single deposit of {min_fmt} or more
3. We'll instantly match your deposit {pct}%, up to {max_fmt} Casino Credit
```
**T&C:** Full Canadian (Ontario) legal copy, baked in at generation time — saved as `vcl_dm_tc`. Dynamic fields: `{min_fmt}`, `{pct}`, `{max_fmt}`, `{terms_id}`, `{start_date_long}`, `{end_date_long}` (short variants also available). Title line: `FANATICS CASINO (CANADA) – DEPOSIT {min_fmt} CAD, GET {pct}% CASINO CREDIT (UP TO {max_fmt} CAD) ({terms_id})`. Promotional dates (default August 17, 2026 → December 31, 2026) and per-offer Terms IDs are edited via the **Edit Terms Expiry & Terms IDs** button inside the Edit T&Cs modal (with CA selected). §1's "within 3 days of being presented this offer" and §4's "during that Promo Period" are static legal copy by design — do not template them. Changes require script regeneration.

### VIP Offer Library - Deposit Matches — US — United States (v2.26)
**HIW:** Stakeholder-approved template — saved as `vcl_dm_hiw_us`. Dynamic fields: `{min_fmt}`, `{pct}`, `{max_fmt}`. No opt-in deadline line and no "See below" trailer, by design:
```
1. Opt-in to the promotion
2. Make a single deposit of {min_fmt} or more
3. We'll instantly match your deposit {pct}%, up to {max_fmt} FanCash
```
**T&C:** Full US legal copy — **one canonical template for all ten tiers** — baked in at generation time, saved as `vcl_dm_tc_us`. Derived from the CAS_10677 legal filing; the CAS_10677 / CAS_10678 / CAS_10686 filings differed by two copy-edit inconsistencies, normalized by stakeholder decision ("Fanatics Sportsbook and Casino App" spacing, "Only your first Qualifying Deposit"), plus two global normalizations (trailing whitespace stripped; title prefixed `FANATICS CASINO - `). Dynamic fields: `{min_fmt}`, `{pct}`, `{max_fmt}`, `{terms_id}`, `{start_date_long}`, `{end_date_long}` (short variants also available). Title line: `FANATICS CASINO - {min_fmt}+ DEPOSIT, {pct}% MATCH IN FANCASH (MAX {max_fmt}) (ID: {terms_id})`. The 3-day deposit / 72-hour Rewards language, 7-day FanCash expiry, FanCash Terms URL, and responsible-gaming lines are static legal copy by design — do not template them. Changes require script regeneration. ⚠️ CAS_10679–CAS_10685 filings not supplied — template assumed to cover them; verify on first use.

### VIP Offer Library - Bet & Gets — CA — Canada
**HIW:** Legal-approved template — saved as `vcl_bg_hiw` (region-scoped as of v2.25). Dynamic fields: `{bet_fmt}`, `{get_fmt}`, `{game_category}` (`Slots` / `Tables`). First-person VIP-host voice; Canadian spelling "favourite"; no opt-in deadline line and no "See below" trailer, by design:
```
1. Opt-in to the promotion
2. Wager {bet_fmt}+ on any of your favourite {game_category} games
3. I'll instantly give you {get_fmt} in Casino Credit
```
**T&C:** Full Canadian (Ontario) legal copy — **one template for both Slots and Tables** — baked in at generation time, saved as `vcl_bg_tc`. Dynamic fields: `{bet_fmt}`, `{get_fmt}`, `{terms_id}`, `{game_category_lower}` (`slots` / `table` — legal writes "slots games" but "table games"), `{game_category_upper}` (`SLOTS` / `TABLES`), `{start_date_long}`, `{end_date_long}` (short variants also available). Title line: `FANATICS CASINO (CANADA) – {game_category_upper} BET {bet_fmt} CAD, GET {get_fmt} CAD IN CASINO CREDIT ({terms_id})`. Promotional dates (default August 17, 2026 → December 31, 2026) and per-offer Terms IDs are edited via the **Edit Terms Expiry & Terms IDs** button inside the Edit T&Cs modal (with CA selected). The seventy-two (72) hour wagering-window language, 7-day Casino Credit expiry, and §5 CAD examples are static legal copy by design — do not template them. Changes require script regeneration. **As of v2.27 the `GAME_CATEGORIES` map's `hiw` value (`Slots` / `Tables`) also drives the CA Masthead / Promo Detail creative filenames.**

### VIP Offer Library - Bet & Gets — US — United States (v2.25)
**HIW:** Stakeholder-approved template — saved as `vcl_bg_hiw_us`. Dynamic fields: `{bet_fmt}`, `{get_fmt}` only. First-person VIP-host voice; US spelling "favorite"; **"Slot games" is static copy** (all US tiers are slots — no `{game_category}` field); no opt-in deadline line and no "See below" trailer, by design:
```
1. Opt-in to the promotion
2. Wager {bet_fmt}+ on any of your favorite Slot games
3. I'll instantly give you {get_fmt} in FanCash
```
**T&C:** Full US legal copy — **one canonical template for all four tiers** — baked in at generation time, saved as `vcl_bg_tc_us`. The CAS_10687 / CAS_10688 legal filings were **not** word-identical — four copy-edit inconsistencies were normalized by stakeholder decision ("Fanatics Sportsbook and Casino App" spacing, plural "wagering requirements," single period in Limitations, "aims to consistently care for our customers"). Everything else is verbatim legal copy (verified byte-level against both source documents with those fixes applied). Dynamic fields: `{bet_fmt}`, `{get_fmt}`, `{terms_id}`, `{start_date_long}`, `{end_date_long}` (short variants also available) — **no game-category fields**. Title line: `FANATICS CASINO - VIP BET {bet_fmt} GET {get_fmt} FANCASH (ID: {terms_id})`. Promotional dates (default July 1, 2026 → December 31, 2026) and per-offer Terms IDs are edited via the **Edit Terms Expiry & Terms IDs** button inside the Edit T&Cs modal (with US selected). The 72-hour wagering / 3-day presentation language, 7-day FanCash expiry, FanCash Terms URL, and responsible-gaming lines are static legal copy by design — do not template them. Changes require script regeneration. ⚠️ CAS_10689 / CAS_10690 filings not supplied — the canonical template is assumed to cover them; verify on first use.

### Daily Missions - Canada (v2.28; FS live v2.30)
**No localStorage keys and no edit modals — HIW and T&C are baked into the HTML** (Edit HIW / Edit T&Cs hidden by design; changes require reaching out to Adrian and regenerating scripts).
**CC HIW** (no "CAD" by design; "twenty-four (24) hours" static legal copy; no "See below" trailer). Dynamic fields: `{bet_fmt}`, `{get_fmt}`:
```
1) Opt in
2) Place {bet_fmt}+ in cumulative cash wagers on any Fanatics Casino games within twenty-four (24) hours of receiving this offer
3) We'll reward you with {get_fmt} in Casino Credit to use on any game on Fanatics Casino
```
**FS HIW (v2.30 — live):** line 3 becomes `We'll reward you with {spins} Bonus Spins to use on the slots game {game_name}`.
**CC T&C:** Full Canadian (Ontario) legal copy — one template for all four CC tiers, byte-verified against the supplied CAS_CA_0019/0020 filings. Dynamic fields: `{bet_fmt}` (comma-formatted), `{get_fmt}`, `{terms_id}`, `{start_date_long}`, `{end_date_long}`. Title line: `FANATICS CASINO (CANADA) – BET {bet_fmt} CAD, GET {get_fmt} CAD IN CASINO CREDIT ({terms_id})`. **Static by design:** §5's $2/$20/$5 worked examples on all tiers, the CC-only "each time you are eligible" clauses in §4/§7, and the seven (7) day expiry — do not template them.
**FS T&C (v2.30 — live in the builder):** one template for all sixteen FS documents. Dynamic fields: `{bet_fmt}`, `{spins}`, `{spin_value}` (two decimals), `{game_name}`, `{terms_id}`, `{start_date_long}`, `{end_date_long}`. Title line: `FANATICS CASINO (CANADA) – BET {bet_fmt} CAD, GET {spins} BONUS SPINS ({terms_id})`. **Byte-verified against all sixteen FS filings (CAS_CA_0043–0058)** with two approved normalizations: exactly one space always follows `{game_name}` (corrects the Monday filings' glued "Baa Baa Baa™and" defect and the Fri/Sun double-space defects), and trailing whitespace stripped. The §5 "$15 CAD" winnings example is static on all tiers.
Both CC and FS currently share the promotional window **08/17/26 → 12/31/26**, baked at generation time; the spec reserves five independent date pairs (Mon/Wed/Fri/Sun/CC-shared) for when the FS games rotate. ⚠️ CAS_CA_0021/0022 (the CC $500/$5,000 tiers) remain unsupplied — verify on first use.

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
