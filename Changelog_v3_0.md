# NATS Bonus Creator — Changelog v3.0

**Release date:** August 25, 2026
**File:** `nats_bonus_creator_v3_0.html` (supersedes `nats_bonus_creator_v2_34.html`)

---

## 1. New Naming Convention (all live offer types)

All segment / promotion / bonus codes now follow the 7-fixed-section skeleton:

```
MMDDYY_JURISDICTION_PRODUCT_LIFECYCLE_SUBCATEGORY_MECHANIC_AWARDTYPE_(FREEFORM)
```

**Voucher codes are completely untouched** — every prefix, format, and the 15-char rule is identical to v2.34. **Image filenames are completely untouched** — all Discover / Masthead / Promo Detail conventions, including VIPBG CA category creatives and DMCA weekday-prefixed files, are unchanged on disk; generated scripts translate from the new codes (K-expansion, tier tables) back to the existing filename shapes. **Terms ID localStorage maps are untouched** — keys stay in their literal v2.x shapes; scripts translate.

### Per-offer-type formats

| Offer | v3.0 format | Example |
|---|---|---|
| RTC CC | `MMDDYY_{JUR}_CAS_RET_RTC_OPT_CC_{amount}` | `053026_US_CAS_RET_RTC_OPT_CC_50` |
| LC-REACT | `MMDDYY_{JUR}_CAS_LC_REACT_OPT_CC_{amount}` | `122926_US_CAS_LC_REACT_OPT_CC_10` |
| LC-CHURN-DM | `MMDDYY_{JUR}_CAS_LC_CHURN_DM_CC_M{min}_{pct}M{max}` | `121126_US_CAS_LC_CHURN_DM_CC_M10_50M10` |
| VIPDM (US) | `MMDDYY_US_VCAS_RET_OL_DM_FC_M{min}_{pct}M{max}` | `090126_US_VCAS_RET_OL_DM_FC_M250_20M5K` |
| VIPDM (CA) | `MMDDYY_CA_VCAS_RET_OL_DM_CC_M{min}_{pct}M{max}` | `090126_CA_VCAS_RET_OL_DM_CC_M250_20M1K` |
| VIPBG (US) | `MMDDYY_US_VCAS_RET_OL_SBG_FC_B{bet}_G{get}` | `090126_US_VCAS_RET_OL_SBG_FC_B100K_G5K` |
| VIPBG (CA) | `MMDDYY_CA_VCAS_RET_OL_{SBG\|TBG}_CC_B{bet}_G{get}` | `090126_CA_VCAS_RET_OL_TBG_CC_B100K_G2500` |
| RAF | `MMDDYY_{JUR}_CAS_ACQ_SUO_RAF_FS_{RFRE\|RFER\|DAY{n}}_{spins}` | `100126_MI_CAS_ACQ_SUO_RAF_FS_RFRE_50` |
| SUO | `MMDDYY_{JUR}_CAS_ACQ_SUO_DR_FS_DAY{n}_{spins}` | `100126_MI_CAS_ACQ_SUO_DR_FS_DAY2_50` |
| DMCA (CC) | `MMDDYY_CA_CAS_RET_BAU_BG_CC_B{bet}_G{get}` | `120126_CA_CAS_RET_BAU_BG_CC_B125_G5` |
| DMCA (FS) | `MMDDYY_CA_CAS_RET_BAU_BG_FS_T{n}` (tier index 1–4) | `113026_CA_CAS_RET_BAU_BG_FS_T2` |

### K-notation
Freeform values ≥ 1,000 and evenly divisible by 1,000 render as `{n}K` (e.g. `B100K`, `M5K`); everything else stays literal (`2500` remains `2500` — decimals disallowed). Generated scripts expand K back to literal values before any field fill, Terms ID lookup, or filename derivation. Applied today to VIPDM/VIPBG; every migrated parser accepts K on any offer type for forward compatibility.

### DMCA FS tier index
FS codes carry `T1`–`T4` instead of literal values (bets 50/125/500/5000 respectively); the actual bet/spins/spin value/game were already baked per-code into the generated script's `CODES` config at generation time, so no runtime resolution changed. An FS bet outside the T1–T4 map **blocks generation** with an alert (fail-loud on future tier rotation).

### 40-character limit
The NATS bonus name field hard-truncates at 40 chars (established by test 08/25/26). Every live builder now has a generation-time guard that blocks with an alert listing any offending code; SUO and RAF additionally carry a runtime assert (their codes are assembled in Python). Codes at exactly 40 (watch list): VIPDM `M250_10M2500`, VIPBG `TBG…B100K_G2500`.

### Fail-loud parsing
Generated scripts parse codes positionally (`parts[0..6]` + verbatim freeform) and raise immediately on: >40 chars, <8 fields, unknown jurisdiction at position 1 (catches v2.x-shaped codes before any NATS write), malformed freeforms, decimal/lowercase K, and (VIPBG CA) a mechanic outside SBG/TBG. `currency_for` reads position 1 directly — a v2.x code or the reserved `CA` (Cash) awardtype token can no longer misroute the currency row.

---

## 2. Retired Offer Types

**RTC Top Up - Free Spins, Bet & Get, Deposit Match, and Bet & Get - Rules Engine are removed from the offer-type dropdown.** Their builders remain in the HTML byte-identical to v2.34 (verified) for reference and possible future revival — they still generate v2.x-format codes if ever re-exposed and were deliberately NOT migrated.

---

## 3. Promotion Tile Retry (NATS search-index lag fix)

Root-caused during the LC-CHURN-DM live validation: NATS's promo search index can lag a just-created promo, timing out the Promotion Tile selection (a re-run succeeded unchanged). Every live NATS-bonus builder's Promotion Tile selection is now a **6-attempt loop (~70s)** — each attempt re-opens the dropdown, re-types the code, and waits 10s; failures print a WARN naming the lag before the final hard error. Applied to: RTC CC, LC-REACT, LC-CHURN-DM, VIPDM, VIPBG, and both RAF blocks (RFRE + RFER). SUO/DMCA have no Promotion Tile.

---

## 4. Verification Record

Built as 8 hash-disciplined increments (one builder at a time, all other builders verified byte-identical at each step), each rendered through a **real Node.js template literal** and the resulting Python passed `ast.parse` + `py_compile` with parse functions **executed** against positive and negative codes — the Known Limitation #5 process. ~250 automated checks total across the increments, including: byte-level Terms ID map lookups on real default maps, real legal T&C rendering (LC-REACT tiers, Churn DM, DMCA CC+FS incl. the `game_name` space normalization), voucher format invariance, all window math (incl. a Dec→Jan year crossing), v2.33 game-search/pattern regression by execution (impostor + cross-match rejection), and DMCA's full v2.31/v2.34 machinery (attach discriminators, placeholder assertions, chip read-back, both "refusing to…" hard-fails, `Mission Complete!` toast, Okta profile, 00:01 times). Final ship checks: full `<script>` block passes `node --check`; all 9 live-builder outputs re-rendered from the shipped file and compiled; the four retired builders byte-identical to pristine v2.34.

**Live PROD validation:** LC-CHURN-DM full segment+promo+bonus (US), RTC CC, VIPDM, VIPBG, SUO, RAF (per-increment user confirmations), and DMCA end-to-end incl. the tier-index code through the Playmaker Existing-Segment attach + chip read-back (Aug 25, 2026).

### Harness catches worth remembering
- **ChurnDM 9-field trap:** `LC_CHURN_DM` contains underscores, so v2.x Churn codes pass a naive 8-field minimum — jurisdiction-position validation was added to every multi-token parser as a result.
- **VIPBG nested-literal escaping:** `parse_vipbg` lives inside a `${isUS ? … : …}` ternary (different backslash arithmetic than the outer literal) — regex bytes were verified in the rendered output.
- The VIPBG retry omission and a stale RTC render path were both caught by presence checks before ship.

---

## 5. Coexistence & Cleanup Notes

- v2.x and v3.0 codes coexist safely in NATS/Playmaker — nothing re-parses stored codes after build. All "avoid reusing these exact codes" v2.x PROD-cleanup warnings are moot under the new format (a v3.0 code can never collide with a v2.x leftover).
- Known Limitation #16's DMCA-vs-BG-CA code collision is doubly closed: AWARDTYPE differs (`CC` vs `FC`) and standard BG is retired.
- Leftover from the Aug 25 lag incident: segment + promo `120126_US_CAS_LC_CHURN_DM_CC_M10_50M10` exist in PROD without... (completed on the successful re-run — verify no duplicate segment/promo pair was created by the re-run; delete extras if so).
- RTC's LIFECYCLE token is `RET` **provisionally** — a future change to `LC` is a one-constant swap per RTC builder (nothing parses the token).
- The unused v2.x parse helpers in retired builders and the old `CANADIAN_JURISDICTIONS` token-hunting `currency_for` in those four builders are intentionally preserved (byte-identity).
