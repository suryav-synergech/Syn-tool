# CRITIQUE - texas-fair-plan-poa-score (build-time only, not for runtime)

Meta-prompt: `prompt/meta-score-skill-creation-prompt.md` (single-input version)
Skill: `skills/texas-fair-plan-poa-score/SKILL.md`
Companion: `skills/texas-fair-plan-poa-match/SKILL.md` (read; IDs cited verbatim)

## Final disposition: PROVISIONAL

Structurally consistent and arithmetically checked, but every score and weight is **designed, not sourced**, the score has almost no quality signal (see open item 1), the input contract is assumed, and `skills-ref` was not run.

## 1. Weight sums (check A)

| Level | Weights | Sum |
|---|---|---|
| Rules in RG-CONS | R-INS, R-MKT, R-TAX, R-VIOL, R-OCC, R-COND, R-CLS, R-ACV, R-INSP, R-DOCS at 10% each | 100% |
| Rule groups | RG-CONS 100% | 100% |

Equal weights because the guideline states no preference among factors. No skew.

## 2. Per-rule integrity (check B)

| Rule | Category | Branches (score) | Exhaustive and exclusive | Sourced or designed |
|---|---|---|---|---|
| R-INS, R-TAX, R-VIOL, R-COND, R-ACV, R-INSP | boolean | PASS 100; INDETERMINATE 50 | Yes. A constraint absent from input counts as INDETERMINATE | Scores designed; facet sourced to the cited page |
| R-MKT, R-OCC, R-CLS | tiered-numeric | 0 / 1 / 2 INDETERMINATE -> 100 / 75 / 50 | Yes, counts 0-2 cover all status pairs | Designed |
| R-DOCS | tiered-numeric | missing 0..4 -> 100, 90, 80, 70, 60; all missing or unknown -> 50 | Yes. Five required documents, counts 0-5 plus unknown | Designed |

- All scores are within 0-100. No unknown branch scores 0; each is below confirmed and above the minimum.
- Numeric thresholds from the guideline: none are used. The only numbers are designed scores and the count of documents.
- No rule has a FAIL branch or FAIL-tier score.

## 3. Referral inventory (check C)

All eight referral indicators in the companion match skill are listed in the skill's referral table and appear in no rule. Roof, pool, sprinkler, extinguisher, gym, secured-common-area, disrepair and liability-exposure items are neutral.

| Not scored | Basis |
|---|---|
| Deductibles, premium, policy fee, payment plans | Out of scope by prompt rule |
| Coverage limits ($3,000,000 property; $300,000 / $500,000 liability) | Describe limits available; no suitability signal; classification unresolved in the match skill |
| Binding, cancellation, non-renewal, appeal | Process rules |

No override mechanism exists in this flow.

## 4. Facet-to-group map (check E)

| Facet | Owner |
|---|---|
| Current insurance | RG-CONS / R-INS |
| Declinations and offers | RG-CONS / R-MKT |
| Taxes, code-violation notice | RG-CONS / R-TAX, R-VIOL |
| Occupancy, repair status, habitability | RG-CONS / R-OCC, R-COND |
| Risk type and class code | RG-CONS / R-CLS |
| Coverage vs actual cash value | RG-CONS / R-ACV |
| Inspection consent | RG-CONS / R-INSP |
| Required quote documents | RG-CONS / R-DOCS |

One group, so no facet appears in two groups. The declination document is counted in R-DOCS as a document only; declination substance stays in R-MKT.

## 5. Self-tests (check H)

Computed by hand and re-computed by script.

| Test | Rule scores (INS, MKT, TAX, VIOL, OCC, COND, CLS, ACV, INSP, DOCS) | RG-CONS | Score |
|---|---|---|---|
| (i) Complete, clean | 100 x10 | 100 | 100 |
| (ii) Eligible, sparse | 100, 50, 50, 50, 75, 100, 75, 50, 50, 70 | (10+5+5+5+7.5+10+7.5+5+5+7) = 67.0 | 67 |
| (iii) Referral twin of (ii) | Same as (ii); pool referral indicator added | 67.0 | 67 |

Referral-twin equality: **holds** (67 = 67). No rule reads `ReferralIndicators`.
Narrative in (ii) attributes the drag to data gaps and states there are no adverse signals.

## 6. Cross-reference integrity (check I)

Inventory taken from the companion `SKILL.md`: 12 hard IDs, 1 soft ID, 8 referral IDs.

| Score rule | Maps to | Found verbatim? | Match-skill location |
|---|---|---|---|
| R-INS | TFPA-HC-INS-001 | Yes | Hard constraints |
| R-MKT | TFPA-HC-DECL-001, TFPA-HC-OFFER-001 | Yes, both | Hard constraints |
| R-TAX | TFPA-HC-TAX-001 | Yes | Hard constraints |
| R-VIOL | TFPA-HC-VIOL-001 | Yes | Hard constraints |
| R-OCC | TFPA-HC-VAC-001, TFPA-HC-REPAIR-001 | Yes, both | Hard constraints |
| R-COND | TFPA-HC-COND-001 | Yes | Hard constraints |
| R-CLS | TFPA-HC-TYPE-001, TFPA-HC-CLASS-001 | Yes, both | Hard constraints |
| R-ACV | TFPA-HC-ACV-001 | Yes | Hard constraints |
| R-INSP | TFPA-HC-INSP-001 | Yes | Hard constraints |
| R-DOCS | TFPA-SC-DOCS-001 | Yes | Soft constraint |

Script result: all mapped IDs found, none invalid, none duplicated; every hard and soft ID in the match skill is mapped; the eight referral IDs match the match skill's table exactly.

## 7. Overrides (check C2)

None inside the guideline document. No external override files in this flow.

## 8. Frontmatter limits (check J)

| Field | Measured | Cap | Result |
|---|---|---|---|
| `description` | 507 characters | 1024 | PASS |
| `name` | 25 characters | 64 | PASS |

## 9. Format conformance (check K) - script- and hand-verified

| Check | Result |
|---|---|
| Metadata values all scalars; `version` quoted; `companion_match_skill` names an existing folder | PASS |
| One H1; all five required H2 sections | PASS |
| RG heading weights sum to 100%; one formula term `RG-CONS x 1.00` matches | PASS |
| Every example `Score` is an integer in 0..100 | PASS |
| ID-only cell counts (RULE002) | No ID appears as a bare cell in two rows (script). Eight referral IDs and thirteen mapped IDs appear once each |
| Output block vs both examples | Same three keys, same order, in both directions |
| JSON blocks parse | 3 of 3 |
| Field order and `ScoreNarrative` shape | PASS; three strings starting Lifters, Drags, Final; Final number equals Score |
| ASCII in calculation strings; no `Risks` field | PASS (the word appears only in the guideline heading "Eligible Commercial Risks") |
| No FAIL or NOT MATCH wording; no hard-constraint evaluation logic | PASS (text search) |
| Line count 144 of 500; no references folder | PASS |
| `skills-ref validate` | **Not run** - not installed. No lint is claimed |

## 10. Source inventory

- Guideline: `Guidelines 1(Texas).pdf`, read as page images (no text layer, no per-span formatting check). No excluded or secondary documents. No formatting-encoded tiers seen.
- Companion match skill read for IDs.
- Points taken from the guideline: only the facets and page references cited in the rule table. No numeric guideline thresholds are used.

## 11. Open items

1. **The score carries little quality signal.** The companion match skill classifies every property-quality item (roof, pool, sprinklers, extinguishers, gym, secured areas, disrepair, liability exposure) as a referral indicator, so all are score-neutral. The score therefore measures how fully the eligibility information is confirmed, not how good the risk is. To add a quality group, those items would first need reclassifying in the match skill (for example from referral to soft constraints); the Score skill would then gain a group with equal weight shares. Decision needed.
2. Every rule score and weight is designed, not sourced; calibrate (50 for unknown, 75 and 50 for pairs, the document tiers, equal 10% weights).
3. Fit wording is qualitative (CONFIRMED FIT, CONDITIONAL FIT); no sourced fit bands exist.
4. Rounding of a value ending in exactly .5 is set to round up; the prompt does not specify.
5. Input contract assumed: `HardConstraintsResult` from the prompt's example, plus `SoftConstraints` and `documents_provided` carried with the submission. Confirm field names.
6. A hard constraint absent from the input is treated as INDETERMINATE; confirm.
7. Grouping of DECL with OFFER, VAC with REPAIR and TYPE with CLASS was chosen to keep ten equal-weight rules; separate rules would need unequal integer weights.
8. Run `skills-ref validate ./texas-fair-plan-poa-score`.
9. Match-skill open items (class code, condo eligibility, $3M limit, designated areas, Plan of Operation) still apply.
