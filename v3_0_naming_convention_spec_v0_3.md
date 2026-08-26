# NATS Bonus Creator — v3.x Naming Convention Spec (v0.3)

Status: **Shipped** (v3.0 Aug 25; v3.1 + v3.2 Aug 26 2026). Voucher codes are **untouched**.

---

## Skeleton

```
MMDDYY_JURISDICTION_PRODUCT_LIFECYCLE_SUBCATEGORY_MECHANIC_AWARDTYPE_(FREEFORM)
```

**v3.1 — jurisdiction group + state suffix:** position 1 is always the group `US` (US/MI/WV/PA/NJ) or `CA` (CA/ON/AB); a specific state/province is appended to the END of the freeform (`090926_US_CAS_ACQ_SUO_DR_FS_DAY3_50_WV`). Generic US/CA picks add no suffix. Parsers treat the trailing state token as optional and strip it before value extraction. Applies to all offer types; only RAF/SUO (always) and LC-REACT (when a state is picked) produce suffixed codes.

- **7 fixed sections**, all required and non-empty for every offer type.
- **Hard limit: 40 characters** (NATS bonus name field truncates at 40 — confirmed by test 08/25/26). Every builder must include a generation-time guard: `len(code) <= 40` or block generation with an alert naming the offending code.
- Delimiter: underscore. Character set: `A-Z 0-9 _` (K-notation adds no new characters; decimals are disallowed).

### Parse rule (all generated scripts)

```python
parts = code.split("_")
date        = parts[0]   # MMDDYY — unchanged from v2.x
jurisdiction = parts[1]  # unchanged tokens: US/MI/WV/PA/NJ/ON/AB/CA
product     = parts[2]
lifecycle   = parts[3]
subcategory = parts[4]
mechanic    = parts[5]
awardtype   = parts[6]
freeform    = "_".join(parts[7:])  # verbatim; may contain underscores
```

- `currency_for()` now reads `parts[1]` directly (no token hunting). USD: US/MI/WV/PA/NJ; CAD: ON/AB/CA. GBP unreachable, as before.
- Freeform is everything after the 7th underscore, carried verbatim into extraction logic.

---

## Section Vocabularies

### PRODUCT
| Meaning | Code | Used |
|---|---|---|
| Casino | `CAS` | ✅ all standard offers |
| VIP Casino | `VCAS` | ✅ VIPDM, VIPBG (replaces v2.x `VCL`) |
| Gold/Platinum/Black | `VF1` | reserved |
| Potential VIP Casino | `PVC` | reserved |
| Fanatics Fast Track | `VPA` | reserved |

### LIFECYCLE
| Meaning | Code | Used |
|---|---|---|
| Acquisition | `ACQ` | ✅ RAF, SUO |
| Retention | `RET` | ✅ BG, BG-RE, DM, DMCA, VIPDM, VIPBG, **RTC CC, RTC FS** |
| Lifecycle | `LC` | ✅ LC-REACT, LC-CHURN-DM |

> RTC CC / RTC FS assigned `RET` **provisionally — may change in the future**. Nothing parses this token; a future change is a one-constant swap per RTC builder.

### SUBCATEGORY
| Meaning | Code | Used |
|---|---|---|
| Sign Up Offer | `SUO` | ✅ RAF, SUO Day 2+ |
| Migration | `MIG` | reserved |
| Reg Not Funded | `RNF` | reserved |
| Cross Sell | `XSL` | reserved |
| RTC Top Up | `RTC` | ✅ RTC CC, RTC FS |
| Reactivation | `REACT` | ✅ LC-REACT |
| Early Life | `EL` | reserved |
| Risk of Churn | `CHURN` | ✅ LC-CHURN-DM |
| Casino Rewards | `OCR` | reserved |
| Free to Play | `FTP` | reserved |
| Offer Library | `OL` | ✅ VIPDM, VIPBG |
| Generic Offer | `BAU` | ✅ BG, BG-RE, DM, DMCA |

### MECHANIC
| Meaning | Code | Used |
|---|---|---|
| Bet and Get | `BG` | ✅ BG, BG-RE (no distinction — accepted), DMCA |
| Opt-in Bonus | `OPT` | ✅ RTC CC, RTC FS, LC-REACT |
| Deposit Match | `DM` | ✅ DM, LC-CHURN-DM, VIPDM |
| Drop | `DR` | v3.2: unused (was SUO in v3.0–v3.1) |
| Slots Bet & Get | `SBG` | ✅ VIPBG CA Slots, **VIPBG US (all tiers)** |
| Tables Bet & Get | `TBG` | ✅ VIPBG CA Tables |
| Refer a Friend | `RAF` | ✅ RAF (all campaign codes incl. DAY{n}) |
| Lossback / Sweepstakes / Goodwill / S2W / Leaderboard / FanCash Multiplier | `LOSS` `SWP` `GW` `S2W` `LB` `FCX` | reserved |

### AWARDTYPE
| Meaning | Code | Used |
|---|---|---|
| Casino Credit | `CC` | ✅ RTC CC, LC-REACT, LC-CHURN-DM, VIPDM CA, VIPBG CA, DMCA CC days |
| Free Spins | `FS` | ✅ RTC FS, RAF, SUO, DMCA FS days |
| FanCash | `FC` | ✅ BG, BG-RE, DM, VIPDM US, VIPBG US |
| Cash | `CA` | reserved — ⚠️ token collides visually with jurisdiction `CA`; harmless under positional parsing |
| Dummy Opt-In / Physical / Free Bet / Profit Boost | `DO` `PA` `FB` `PB` | reserved |

---

## K-Notation (freeform values)

- Any freeform numeric value **≥ 1,000 and evenly divisible by 1,000** renders as `{n}K` (e.g. `100000` → `100K`, `5000` → `5K`).
- Values not divisible by 1,000 stay literal (`2500` stays `2500` — no decimals allowed).
- Available to all offer types; **required** only where budget demands it (VIPDM / VIPBG tiers today).
- All generated scripts include a shared `expand_k()` helper applied wherever freeform values are read (wagering amounts, deposit min/max, reward amounts).

---

## Per-Offer-Type Formats (old → new)

| Offer | New format | Example (new) | Example (old) | Worst-case len |
|---|---|---|---|---|
| RTC CC | `MMDDYY_{JUR}_CAS_RET_RTC_OPT_CC_{amount}` | `053026_US_CAS_RET_RTC_OPT_CC_50` | `053026_CAS_RET_RTC_US_CC_50` | ≤31 ✅ |
| RTC FS | `MMDDYY_{JUR}_CAS_RET_RTC_OPT_FS_{spins}S{value}V_{game}` | `061926_MI_CAS_RET_RTC_OPT_FS_10S20V_TCE` | `061926_CAS_RET_RTC_MI_FS_10S_20V_TCE` | 40 ⚠️ exact |
| BG | `MMDDYY_{JUR}_CAS_RET_BAU_BG_FC_B{bet}_G{get}` | `053026_US_CAS_RET_BAU_BG_FC_B100_G2` | `053026_CAS_RET_BG_US_FC_B100_G2` | ~35 ✅ |
| BG-RE | identical to BG (no distinction — accepted) | `062226_US_CAS_RET_BAU_BG_FC_B100_G2` | `062226_CAS_RET_BG_US_FC_B100_G2` | ~35 ✅ |
| DM | `MMDDYY_{JUR}_CAS_RET_BAU_DM_FC_M{min}_{pct}M{max}` | `073126_US_CAS_RET_BAU_DM_FC_M10_20M500` | `073126_CAS_RET_DM_US_FC_M10_20M500` | ~38 ✅ |
| DMCA CC | `MMDDYY_CA_CAS_RET_BAU_BG_CC_B{bet}_G{get}` | `120126_CA_CAS_RET_BAU_BG_CC_B125_G5` | `120126_CAS_RET_BG_CA_CC_B125_G5` | ~37 ✅ |
| DMCA FS | `MMDDYY_CA_CAS_RET_BAU_BG_FS_T{n}` (tier index 1–4) | `113026_CA_CAS_RET_BAU_BG_FS_T2` | `113026_CAS_RET_BG_CA_FS_B125_G50S_10V_BAA` | 30 ✅ |
| RAF | `MMDDYY_{JUR}_CAS_ACQ_SUO_RAF_FS_{RFRE\|RFER\|DAY{n}}_{spins}` | `100126_MI_CAS_ACQ_SUO_RAF_FS_RFRE_50` | `100126_CAS_ACQ_SUO_RAF_RFRE_MI_FS_50` | ~37 ✅ |
| SUO | `MMDDYY_US_CAS_ACQ_SUO_BG_FS_DAY{n}_{spins}_{ST}` (v3.2) | `100126_US_CAS_ACQ_SUO_BG_FS_DAY2_50_MI` | `100126_CAS_ACQ_SUO_DAY2_MI_FS_50` | 39 ✅ |
| LC-REACT | `MMDDYY_{JUR}_CAS_LC_REACT_OPT_CC_{amount}` | `122926_US_CAS_LC_REACT_OPT_CC_10` | `122926_CAS_LC_REACT_US_CC_10` | ~33 ✅ |
| LC-CHURN-DM | `MMDDYY_{JUR}_CAS_LC_CHURN_DM_CC_M{min}_{pct}M{max}` | `121126_US_CAS_LC_CHURN_DM_CC_M10_50M10` | `121126_CAS_LC_CHURN_DM_US_CC_M10_50M10` | ~39 ✅ |
| VIPDM CA | `MMDDYY_CA_VCAS_RET_OL_DM_CC_M{min}_{pct}M{max-K}` | `090126_CA_VCAS_RET_OL_DM_CC_M250_20M1K` | `090126_VCL_RET_DM_CA_CC_M250_20M1000` | 40 ⚠️ exact (`M250_10M2500`) |
| VIPDM US | `MMDDYY_US_VCAS_RET_OL_DM_FC_M{min}_{pct}M{max-K}` | `090126_US_VCAS_RET_OL_DM_FC_M250_20M1K` | `090126_VCL_RET_DM_US_FC_M250_20M1000` | 40 ⚠️ exact |
| VIPBG CA | `MMDDYY_CA_VCAS_RET_OL_{SBG\|TBG}_CC_B{bet-K}_G{get-K}` | `090126_CA_VCAS_RET_OL_SBG_CC_B1K_G100` | `090126_VCL_RET_SBG_CA_CC_B1000_G100` | 40 ⚠️ exact (`TBG…B100K_G2500`) |
| VIPBG US | `MMDDYY_US_VCAS_RET_OL_SBG_FC_B{bet-K}_G{get-K}` | `090126_US_VCAS_RET_OL_SBG_FC_B1K_G100` | `090126_VCL_RET_BG_US_FC_B1000_G100` | ~38 ✅ |

### DMCA FS tier index table (baked into HTML + generated scripts)
| Tier | Bet | Spins | Spin value |
|---|---|---|---|
| `T1` | $50 | 20 | $0.10 |
| `T2` | $125 | 50 | $0.10 |
| `T3` | $500 | 50 | $0.50 |
| `T4` | $5,000 | 100 | $3.00 |

The script resolves bet / spins / spin value / game (weekday-derived) from this table. The stakes-ladder validation and "refusing to substitute" runtime hard-fail are unchanged.

### Zero-headroom codes (watch list)
`RTC FS` (40 at 2-digit spins + 2-digit value + 3-char game), `VIPDM M250_10M2500` (40), `VIPBG TBG B100K_G2500` (40). The generation-time length guard is the backstop; any future tier/value addition must re-check the budget. A 3-digit spin count or spin value on RTC FS, or a 4-char game token, breaks 40.

---

## What Does NOT Change

1. **Voucher codes** — all formats, prefixes, and the 15-char rule untouched.
2. **Image filenames** — every Discover / Masthead / Promo Detail naming convention stays exactly as v2.x (including VIPBG CA's `SBG_B1000_G100.png` literal-value keys and DMCA's weekday-prefixed FS names). Generated scripts translate from the new code (K-expanded values, tier-table lookups, mechanic token) back to the existing filename shapes. **UI requirement:** the expected image filenames are called out in the dynamic info box above each offer type's day cards.
3. **Terms ID map keys (localStorage)** — recommend keeping current literal key shapes (`M250_20M5000`, `SBG_B1000_G100`) with the script/UI translating, so saved maps survive the upgrade. *(Flagged as default — confirm during build.)*
4. **All runtime behavior** — windows, currency-row fills, dropdown selectors, Okta launch, segment attach, toasts, modals, validation, per-code step toggles.
5. **Jurisdiction token vocabulary and MMDDYY date** — identical to v2.x, just at fixed positions 1 and 0.

---

## Migration / Build Checklist

1. Update code-name **builder functions** in all 12 builders (constants for sections 2–6; freeform assembly incl. K-notation and DMCA tier index).
2. Update **extraction logic** to positional split (`parts[0..6]` + freeform join); add shared `expand_k()`; DMCA FS tier-table resolution.
3. `currency_for()` → read `parts[1]`.
4. **Filename derivation** translated back to v2.x shapes (K-expansion; DMCA FS tier → `{Weekday}_B{bet}_G{spins}S_{value}V.png`; VIPBG key from mechanic + expanded freeform).
5. **Terms ID key** translation layer (new code → existing key shape).
6. Add **≤40 length guard** to every builder.
7. Front-end cleanup + **dynamic image-filename callout box** above day cards per offer type.
8. Verify per **Known Limitation #5 process**: render every builder through a real JS template literal, `ast.parse` + `py_compile` the output, execute extraction functions against example codes, hash-check untouched builders.
9. DMCA: confirm the new code survives the Playmaker **exact-match segment attach + chip read-back** (same string in NATS segment, NATS promo, Playmaker bonus).
10. Old v2.x codes in PROD coexist safely — nothing re-parses stored codes; the "avoid reusing these exact codes" cleanup warnings become moot under the new format.

### Collisions resolved / accepted
- **Fixed in v3.0:** DMCA CC vs standard BG CA — now differ at AWARDTYPE (`CC` vs `FC`).
- **Accepted:** BG vs BG-RE remain identical by design.

---

## Decision Log
| # | Decision | Ruling |
|---|---|---|
| 1 | Sections | 7 fixed + freeform (CATEGORY removed) |
| 2 | Char limit | 40 hard (NATS bonus field truncates; confirmed 08/25/26) |
| 3 | RTC lifecycle | `RET` — provisional, may change |
| 4 | RAF Day 2+ mechanic | `RAF` (whole campaign uniform) |
| 5 | VIPBG US mechanic | `SBG` |
| 6 | BG vs BG-RE | No code distinction |
| 7 | DMCA FS freeform | Tier index `T1`–`T4` |
| 8 | RTC FS freeform | Compressed literal `{spins}S{value}V_{game}` |
| 9 | K-notation | All offer types where needed; ≥1000 divisible by 1000 only |
| 10 | Image filenames | Unchanged; UI callout per offer type |
| 11 | Terms ID keys | Keep current shapes, translate — confirmed |
| 12 | v3.1: state placement | Position 1 = US/CA group; specific state → end of freeform |
| 13 | v3.1: DR vs DROP | `DROP` proposed, reverted — state suffix takes priority in the budget; mechanic stays `DR` |
| 14 | v3.1: Churn regions | LC-CHURN-DM restricted to US/CA (state + suffix exceeds 40) |
| 15 | v3.2: SUO mechanic | `DR` → `BG` (mechanic-section removal declined — would break the 7-fixed-field parse contract); `DR` retired to reserved |
