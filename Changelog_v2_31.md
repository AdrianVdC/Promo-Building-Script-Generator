# NATS Bonus Creator — v2.31 Changelog (Published)

**Release date:** August 20, 2026
**Scope:** Daily Missions - Canada (DMCA) only. All 10 other builders hash-identical to v2.30; whole-file diff vs v2.30 confined to the DMCA blocks, step-badge condition, INFO panel, and version strings.

> Note: this published v2.31 consolidates two development iterations (interim v2.31 + v2.32) into one step from v2.30. The interim v2.31 build — which lacked Include/Exclude section awareness and the chip read-back — should be discarded and never published. **The exact flow in this file was PROD-validated end-to-end on Aug 20, 2026** (segment `101926_CAS_RET_BG_CA_FS_B5000_G100S_300V_BAA` attached via Include → Existing Account Segments with chip confirmed); only version strings changed after that run.

---

## DMCA — NATS Segments + "Existing Account Segments" Attach

The DMCA flow moves from Playmaker-created segments to NATS-created segments:

**Old flow (v2.28–v2.30):** NATS Promo → Playmaker Bonus, typing the code name into the **"Create Account Segments"** field (`input[cmdk-input]` + Enter), which created the segment on Save Bonus.

**New flow (v2.31):** NATS **Segment** → NATS Promo → Playmaker Bonus, attaching the pre-built segment via the **Include Segments → Existing Account Segments** field. The Create Account Segments fields are never touched (a chip there would create a duplicate segment on Save Bonus) and the entire Exclude Segments section is never touched (it would invert targeting).

### Phase 1 (NATS)
- New `nav_to_segments()` (sidebar `[data-menu-id$='-segments']` → "Account Segments") and `create_segment()` — the standard NATS segment build used by every other segment-building offer type (AMELCO via `dispatch_event("click")`, `#forBonus` checked, 2s wait before OK).
- Phase 1 is now "NATS: Build Segments + Promotions" — segments first, then promotions, one login, each sub-phase independently skippable.

### Phase 2 (Playmaker) — Step 2
Playmaker's Step 2 contains **four** cmdk fields (Include/Exclude × Create/Existing) with identical labels across sections, and identical `placeholder="Search..."` on both Existing fields. `attach_existing_segment()` therefore locates the field in three guarded layers:

1. **Label-anchored primary** (from captured Playmaker HTML): the `<label>` with exact text `Existing Account Segments` whose ancestor section resolves to `Include Segments` (a container mentioning both Include and Exclude headings is a shared parent — rejected); follow its `for=` attribute to the form-item container and take the `input[cmdk-input]` inside, climbing from the label with a `placeholder='Search...'` filter as backstop. The dynamic React/Radix ids (`:r5g:`, `radix-:r5j:` …) are session-scoped and deliberately not hardcoded.
2. **Hardened fallback:** parent-walk requiring 'Existing Account Segments' without 'Create Account Segments', **and** an ancestor resolving to Include (not Exclude, not both).
3. **Placeholder assertion on both paths:** Existing = `Search...`, Create = `Type and press Enter...` — a Create-style placeholder is rejected even when the structural lookup matched.

Attach behavior: type the code name, click the option whose text **exactly matches** (never Enter — that's the Create gesture), then **read back the chip**: poll up to ~5s for a `<span>` exactly equal to the code name inside the Include section's Existing field. No chip → Step 2 field dump + raise ("refusing to continue on an unverified attach") → per-bonus FAILED with recovery navigation. NATS → Playmaker segment-sync lag is absorbed by a 12 × 5s (~60s) outer poll; if the segment never appears the script hard-fails — "refusing to fall back to Create Account Segments (would duplicate the NATS segment)".

### UI
- DMCA day cards show **three step badges: Segment / Promo / Bonus** (previously Promo + Bonus; BG-RE keeps two). Segment defaults ON; `do_segment` is carried per code.
- INFO panel updated to the new flow (also corrected two stale v2.28-era bullets: FS days no longer described as locked; Playmaker times read 00:01).

### Unchanged
Weekday model, tiers, Terms IDs, stakes ladders and both stake validations, 00:01 Playmaker times, single-day NATS promo window, images/pre-flight, HIW/T&C templates, Okta persistent-profile launch, toast, SAVED/FAILED summary, naming (same code name for segment, promo, bonus).

---

## Verification (per the template-literal limitation)
- `buildDMCAScript` rendered through a real JS engine (node) with mixed CC + FS sample codes including the PROD-attached `101926…BAA` shape; output passed `ast.parse` + `py_compile`.
- 17/17 content checks: label-anchored lookup + `for=` follow, all three JS discriminator blocks Include+Exclude aware, placeholder assertions on both paths, chip verify gating Step 3, hard-fail preserved, no Enter-press in the attach block, segments-before-promos ordering, create-chip flow fully removed, AMELCO/00:01/FS "refusing to substitute" guards untouched, no stray backticks or unrendered `${}` interpolations.
- The harness caught one single-backslash `\n` the JS template literal would have swallowed (the v2.20 defect class) — fixed before ship.
- Published file diff vs the PROD-validated build: exactly six version-string lines; zero functional changes.

---

## Deploy notes
1. **Regenerate all pending DMCA scripts from v2.31** — v2.30 and earlier create a Playmaker-side segment via the Create field, which under the new model would duplicate the NATS segment.
2. **PROD cleanup:** the Aug 20 validation build (`101926_CAS_RET_BG_CA_FS_B5000_G100S_300V_BAA` — NATS segment, promo, and Playmaker bonus) joins the existing v2.28/v2.30 cleanup list (`120126` CC, `120426` MFB ×2, `121626` 4CC, `081926` 4CC) — delete or avoid reusing those codes.
3. Doc updates needed for `Technical_Reference` and the project instructions: DMCA step badges (now Segment/Promo/Bonus), Step 2 "Create Account Segments" wording replaced by the Include → Existing attach, segment build now standard-NATS, chip read-back and Include/Exclude guard documented.
