# NATS Bonus Creator — Changelog v3.4

**Release date:** September 1, 2026
**File:** `nats_bonus_creator_v3_4.html` (supersedes `nats_bonus_creator_v3_3.html`)

## 1. Third New VIPDM CA Tier (CAS_CA_0094)

`DEFAULT_VIPDM_OFFERS` (CA) grows from eleven to twelve tiers:

| Tier | Terms ID | Code freeform | Code length | Discover creative |
|---|---|---|---|---|
| $100 min / 50% / $5,000 max | `CAS_CA_0094` | `M100_50M5K` | **38** | `50M5000.png` |

The tier is listed with the other M100 offers in the card (second row). Its filing also begins **September 1, 2026**, so `M100_50M5000` joins `VIPDM_TERMS_START_OVERRIDES` alongside the v3.3 tiers — the shared 08/17/26 CA date continues to govern filings 0078–0086 only. The Terms modal warning and info box now list all three overridden tiers (CAS_CA_0092–0094). The filing matches the baked CA Ontario template byte-for-byte — no template changes.

No new one-character filename collisions: `50M5000.png` is distinct from `50M250.png` and `20M5000.png`/`10M5000.png` by more than one character, though the `…M5000` family keeps growing — the usual creative double-check applies.

## 2. Verification Record (v3.4)

Same surface as v3.3 (VIPDM defaults, Terms map, override map, info box, modal hint, version strings) — nothing else touched. Full `<script>` passes `node --check`. Rendered through node v22 and executed: CA run with `M100_50M250` (control, August 17 + CAS_CA_0078), `M100_50M5K` (September 1 + CAS_CA_0094, filing lines byte-verified), and `M250_20M10K` (v3.3 regression, September 1 + CAS_CA_0092); all codes ≤40; US render regression clean (`TERMS_START_OVERRIDES = {}`, July 1, 2026 intact).

---

# NATS Bonus Creator — Changelog v3.3

**Release date:** September 1, 2026
**File:** `nats_bonus_creator_v3_3.html` (supersedes `nats_bonus_creator_v3_2.html`)

## 1. Two New VIPDM CA Tiers (CAS_CA_0092 / CAS_CA_0093)

`DEFAULT_VIPDM_OFFERS` (CA) grows from nine to eleven tiers:

| Tier | Terms ID | Code freeform | Code length | Discover creative |
|---|---|---|---|---|
| $250 min / 20% / $10,000 max | `CAS_CA_0092` | `M250_20M10K` | **39** | `20M10000.png` |
| $250 min / 20% / $20,000 max | `CAS_CA_0093` | `M250_20M20K` | **39** | `20M20000.png` |

Both codes render with the standard K-notation (`10000` → `10K`, `20000` → `20K`) and are one character under the 40-char NATS limit — new near-watch entries alongside the existing exact-40 items. The existing `parse_vipdm` K-expansion handles them with no parser changes; Terms ID keys keep their literal expanded shapes (`M250_20M10000`, `M250_20M20000`) per the v2.x key contract.

⚠️ **New CA Discover-filename near-collisions:** `20M1000.png` vs `20M10000.png`, and `20M2000.png` vs `20M20000.png` (one extra zero). The VIPDM info box now flags the CA pairs alongside the existing US ones. Both new creatives must be added to the CA Drive folder before first use.

Both filings (supplied September 1, 2026) match the baked CA Ontario template byte-for-byte — no template changes.

## 2. Per-Tier Terms Start-Date Overrides (VIPDM, CA only)

The CAS_CA_0092/0093 filings begin **September 1, 2026**, while the shared CA start date (filings 0078–0086) remains **08/17/26**. A new `VIPDM_TERMS_START_OVERRIDES` constant (`{key: 'MM/DD/YY'}`, CA only, empty for US) is baked into the generated script as `TERMS_START_OVERRIDES = {key: [short, long]}`; `format_hiw`/`format_tc` now look the tier key up there before falling back to `TERMS_START_SHORT/LONG`. Mixed-tier runs therefore type the correct Promotional Period per tier with no manual date toggling. The Edit T&Cs modal's start-date field continues to govern all non-overridden tiers and now carries a warning line naming the two overridden tiers. The end date (12/31/26) is shared by all eleven filings and is unchanged.

## 3. Terms ID Default Merge

`getVIPDMTermsIDs()` now merges baked defaults **under** any saved `vcl_dm_terms_ids` localStorage map (saved values win per key). Previously a machine with a pre-v3.3 saved map would show "no ID — generation blocked" on the new tiers until the modal was re-saved. Pre-v3.3 saved maps cannot contain the new keys, so no user-saved value is ever overridden. The Terms modal already unioned default + saved keys, so the new rows appear there automatically.

## 4. Verification Record (v3.3)

Only the VIPDM surfaces changed: `DEFAULT_VIPDM_OFFERS`, `DEFAULT_VIPDM_TERMS_IDS`, the new `VIPDM_TERMS_START_OVERRIDES` constant, `getVIPDMTermsIDs()`, `buildVIPDMScript` (override baking + `format_hiw`/`format_tc` lookup), the VIPDM info box, and the VIPDM Terms modal hint. All other builders untouched; retired builders remain byte-identical to v2.34. Per Known Limitation #5, `buildVIPDMScript` was rendered through a real JS engine (node v22) and the resulting Python executed:

- Full `<script>` block passes `node --check`.
- CA render with three tiers (`20M1K` control + both new tiers): `py_compile` clean; `parse_vipdm`, `format_tc`, `format_hiw`, `currency_for`, `parse_voucher` executed.
- T&C byte-checks: 0092/0093 title lines, Promotional Period ("September 1, 2026"), Description, Action Required, and Section 7 cap lines all match the supplied filings; the control tier still emits "August 17, 2026" with `CAS_CA_0084`.
- Terms merge: fresh machine resolves both new IDs from defaults; simulated pre-v3.3 saved map keeps its edited value while exposing `CAS_CA_0092`/`CAS_CA_0093`.
- US render: `TERMS_START_OVERRIDES = {}`, shared July 1, 2026 date and `CAS_10684` lookup unchanged (regression clean).
- Code lengths: `..._M250_20M10K` and `..._M250_20M20K` both 39 chars; generation-time guard budget confirmed.
