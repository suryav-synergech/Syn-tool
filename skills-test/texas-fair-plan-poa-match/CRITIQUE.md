# CRITIQUE - texas-fair-plan-poa-match (build-time only, not for runtime)

Meta-prompt: `prompt/meta-match-skill-creation-prompt.md` (single-input, single-file version)
Skill: `skills/texas-fair-plan-poa-match/SKILL.md` (only file in the bundle)

## 1. Final disposition: PROVISIONAL

Not fully validated. Reasons: output contract is proposed, `skills-ref` was not run, tests were hand-traced and not executed, quotes were not checked against a text layer, and six classification or source issues need a human decision (section 3).

## 2. Source inventory

| Item | Detail |
|---|---|
| Document | `Guidelines 1(Texas).pdf`, Texas FAIR Plan Association Commercial Property Owners Association Program, Underwriting Manual, March 10, 2025 |
| Program name source | Cover page |
| Pages read | All 12 file pages (cover, contents, printed pp.1-11) |
| How read | **Page images supplied in the conversation.** No file on disk, so no text-layer extraction and no per-span formatting recovery |
| Formatting meaning | None seen. Colour is used for heading hierarchy only (visual check, not a file-structure check) |
| Unreadable content | None |
| Amendments or overrides inside the document | None found. Page 2 says the Plan of Operation "will govern when any discrepancies may exist"; it was not supplied |
| Exclusions requested | None |
| Page numbering | Printed footer numbers used; file page = printed + 1 |

**Slug note.** The full slugified program name exceeds the 64-character cap. I used the abbreviation `texas-fair-plan-poa-match`. This is a judgement call, flagged for confirmation.

## 3. Constraint inventory

13 constraints (12 hard, 1 soft) and 8 referral indicators. Quotes were typed from the page images.

| ID | Class | Printed page | Judgement |
|---|---|---|---|
| TFPA-HC-INS-001 | Hard | 2 | Clear mandatory wording |
| TFPA-HC-DECL-001 | Hard | 2-3, 5 | FAIL only on affirmative "none obtained" or a stated-complete list under two holding companies |
| TFPA-HC-OFFER-001 | Hard | 2-3 | "comparable" and "valid" undefined; submission's own statement used |
| TFPA-HC-TAX-001 | Hard | 3 | Deferral-affidavit exception included |
| TFPA-HC-VIOL-001 | Hard | 3 | No cure exception stated; none added |
| TFPA-HC-VAC-001 | Hard | 3 | No vacancy percentage stated |
| TFPA-HC-REPAIR-001 | Hard | 3 | No size threshold stated |
| TFPA-HC-TYPE-001 | Hard | 3, 5 | Condo associations accepted on p.3 and p.5 evidence, not the numbered risk list |
| TFPA-HC-CLASS-001 | Hard | 3 | **Unresolved:** manual says codes are for "statistical reporting purposes" |
| TFPA-HC-COND-001 | Hard | 10 | Explicit "uninsurable" |
| TFPA-HC-ACV-001 | Hard | 10 | Made Hard from the Underwriting Rules intro ("minimum standards to qualify or renew") |
| TFPA-HC-INSP-001 | Hard | 6 | Made Hard from "must agree"; it sits under binding |
| TFPA-SC-DOCS-001 | Soft | 5 | PASS or INDETERMINATE only |
| TFPA-RF-ROOF-001 | Referral | 10-11 | "may not be acceptable" |
| TFPA-RF-POOL-001, SPRK, EXTG, GYM, SECU | Referral | 11 | "must" wording but no consequence stated; p.10 frames related items as discretionary |
| TFPA-RF-DISR-001, LIAB-001 | Referral | 10 | "may be cause for underwriting action" |

### Reviewed and not made constraints

| Provision | Printed page | Reason |
|---|---|---|
| Property limit "up to $3,000,000 per building/structure"; GL $300,000 / $500,000 | 5 | Describes limits available; no prohibition stated. Unresolved whether a larger building is ineligible |
| Wind/hail deductibles 2%, 5%; other-peril deductibles $10,000 to $75,000 | 5 | Options only |
| Designated areas within ten miles of the TWIA coverage area | 2 | No area list in the document. **Unresolved source issue** |
| Underwriting report, discretionary inspection, underwriter actions | 10-11 | Workflow |
| Binding authority, payment, policy fee, payment plans, hurricane suspension | 6-7 | Administrative |
| Cancellation, non-renewal, two-year reapplication, 18-month repair, cooperation | 7-11 | In-force or renewal policies, not new submissions |
| Appeal, conflict with laws, transfer or assignment | 7, 11 | Process |
| Material misrepresentation, code violations, life/safety (p.10 list) | 10 | Discretionary triggers; written code notices covered by TFPA-HC-VIOL-001 |
| Roof warning signs lists | 11 | Examples of damage, not a test; support TFPA-RF-ROOF-001 narrative only |

### Unresolved source and classification issues

1. Designated service area cannot be tested.
2. Condominium associations as an eligible risk type.
3. Whether a non-listed class code is a prohibition.
4. Whether the p.10-11 discretionary items are hard, soft or referral.
5. Whether limits above $3,000,000 per building make a risk ineligible.
6. Plan of Operation governs on discrepancies and was not supplied.

## 4. Determinism evidence

Each constraint states a PASS, FAIL (hard only) and INDETERMINATE condition. Branch partitions were checked by hand:

| Rule | Partition | Boundary |
|---|---|---|
| DECL-001 | PASS (2+ distinct holding companies) / FAIL (stated none, or stated-complete under two) / INDETERMINATE (otherwise) | Exactly two distinct holding companies is PASS |
| ACV-001 | PASS (all buildings limit >= ACV) / FAIL (any limit < ACV) / INDETERMINATE (missing and no shortfall) | Limit equal to ACV is PASS |
| TAX-001, VAC-001, REPAIR-001, COND-001 | Any building affirmative violation gives FAIL | n/a |
| CLASS-001 | Each supplied code tested against its own list | n/a |

Two-answer check: ACV-001 with one building short and another missing data gives FAIL, because a FAIL is never erased by missing data. Confirmed by the stated rule.

## 5. Adversarial tests (hand-traced, synthetic)

| # | Scenario | Expected | Result |
|---|---|---|---|
| 1 | All constraints PASS | MATCH - Confirmed, Match true | As expected |
| 2 | occupancy_status = Vacant | HC-VAC FAIL, NOT MATCH, Match false | As expected |
| 3 | Soft constraint, one document missing | SC-DOCS INDETERMINATE, never FAIL, Conditional | As expected |
| 4 | Declination data missing | HC-DECL INDETERMINATE, Conditional | As expected |
| 5 | PASS plus INDETERMINATE | Conditional, every gap listed | As expected |
| 6 | PASS plus FAIL | NOT MATCH | As expected |
| 7 | Vacant and condemned together | Both FAIL, both explained, NOT MATCH | As expected |
| 8 | Vacant stated, occupancy corroboration missing | FAIL stands | As expected |
| 9 | actual_cash_value = "TBD" | Placeholder treated as missing, HC-ACV INDETERMINATE | As expected |
| 10 | Two equal-authority fields disagree on occupancy | INDETERMINATE, both fields named, ValidationIssues entry | As expected |
| 11 | Limit = ACV minus 1, = ACV, = ACV plus 1 | FAIL, PASS, PASS | As expected |
| 12 | Unenclosed pool stated | RF-POOL-001 reported, statuses and overall result unchanged | As expected |
| 13 | Conditional match | Match true, Permitted, gaps visible | As expected |
| 14 | Submission unreadable | Validation Required, Match false, Blocked pending validation | As expected |

These were traced by reading the rules. They were **not** executed against a model.

## 6. Contract validation

Script check (Python, `-I`) on the SKILL.md:

| Check | Result |
|---|---|
| Fenced json blocks parse | 4 of 4 parse |
| Top-level keys: output block vs each example | Identical in all three examples |
| Nested keys | Identical wherever the example array is non-empty. Three empty arrays (`ReferralIndicators` twice, `SoftConstraints` once) are empty containers, so their entry keys are not shown in those examples |
| Enum values in examples | All declared. `Match` true never coexists with FAIL |
| Rule IDs well formed, unique | 13 constraint IDs and 8 referral IDs, all `TFPA-(HC|SC|RF)-<TOKEN>-NNN`, no duplicates |

Contract is **proposed, not platform-confirmed**. The Score prompt consumes `HardConstraintsResult`; its shape was followed field for field.

## 7. Frontmatter limits

| Field | Measured | Cap | Result |
|---|---|---|---|
| `description` | 464 characters | 1024 | PASS |
| `name` | 25 characters | 64 | PASS |

`name` matches the folder, has no consecutive hyphens, and all `metadata` values are strings.

## 8. Format validation

| Check | Result |
|---|---|
| `skills-ref validate` | **Not run** - tool not installed here |
| `SKILL.md` line count | 343 of 500, PASS |
| Single file; no `references/` folder or link | PASS (verified by directory listing and text search) |
| Frontmatter keys | `name`, `description`, `metadata` only |

Everything except the first row is hand- or script-verified, not machine-validated against the specification.

## 9. Open items

- Submission schema not supplied; all field names are proposals.
- Output contract proposed; confirm against the platform.
- Confirm the abbreviated slug.
- Human decisions on issues 1-6 in section 3.
- Supply the Plan of Operation and the designated-area list if they should affect matching.
- Run `skills-ref validate ./texas-fair-plan-poa-match`.
- Check quotes against the PDF text layer.

## 10. Score skill

Not generated in this run.
