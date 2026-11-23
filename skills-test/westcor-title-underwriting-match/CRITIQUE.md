# CRITIQUE - westcor-title-underwriting-match (build-time only, not for runtime)

Meta-prompt: `prompt/meta-match-skill-creation-prompt.md` (pattern followed; the prompt file itself was not re-read for this run)
Skill: `skills/westcor-title-underwriting-match/SKILL.md` (only file in the bundle)

## 1. Final disposition: PROVISIONAL

Not fully validated. Reasons: output contract is proposed, `skills-ref` was not run, scenario tests were hand-traced and not executed against a model, quotes were not checked against a text layer, and eight classification or source issues need a human decision (section 3).

## 2. Source inventory

| Item | Detail |
|---|---|
| Document | `westcor.pdf` (also in `Docs/`), Westcor Land Title Insurance Company Underwriting Manual, (c) 2017 |
| Program name source | **None.** The cover says only "Underwriting Manual". The name "Westcor Land Title Insurance Underwriting Program" and the `WLTIC` prefix are my choices |
| Pages read | All 164 file pages (cover, contents, pp.1-161, index) |
| How read | Text layer as supplied in the conversation, with page images for forms and survey examples |
| Page numbering | Printed footer numbers equal PDF page numbers (no offset, unlike the Texas manual) |
| Amendments or overrides inside the document | Cover page: "If any conflict exists between this Manual and the terms of your title insurance Issuing Agency Agreement with Westcor, the Issuing Agency Agreement will control." The agreement was not supplied. Page 2 says the manual is "a set of guidelines" |
| Removed content | p.69: "Endorsements have been removed for extensive revision and re-write." Affirmative-coverage and endorsement rules are therefore not testable and are not scored |
| Exclusions requested | None |

**Nature of the source.** This is a general underwriting manual covering about 70 topics, not a single program with an eligibility list. It has no submission-level eligibility test. I extracted the provisions that state a flat prohibition or a must-have and are checkable from submission facts. Everything else is a Schedule B or process instruction and is not a constraint.

## 3. Constraint inventory

15 hard constraints, 1 soft constraint, 18 referral indicators.

| ID | Class | Printed page | Judgement |
|---|---|---|---|
| WLTIC-HC-ADVP-001 | Hard | 10 | Clear: "Absent a final court order, Westcor will not insure" |
| WLTIC-HC-BKRP-001 | Hard | 18, 20, 23 | Clear on finality ("No title insurance may be issued ... until after an appeal is no longer possible") |
| WLTIC-HC-TAXT-001 | Hard | 146-147 | **Judgement:** stated as "policy of the Company"; exceptions are case by case. Hard with a counsel-authorization PASS path |
| WLTIC-HC-HEIR-001 | Hard | 86 | **Judgement:** "only on very rare, exceptional occasions" is an affidavit approved. Hard with counsel-waiver PASS path |
| WLTIC-HC-DIL-001 | Hard | 56-57 | "can never be insured without prior authorization" |
| WLTIC-HC-MIN-001 | Hard | 107-108 | "Westcor will not insure just a mineral estate" |
| WLTIC-HC-TSH-001 | Hard | 150 | "not authorized to issue" for club memberships |
| WLTIC-HC-CEM-001 | Hard | 33 | "Individual burial plots are not insurable" |
| WLTIC-HC-GUARD-001 | Hard | 85, 109 | Requirement stated in commitment wording; "unless met ... exception must be made" |
| WLTIC-HC-POA-001 | Hard | 120 | Four validity tests stated as "must" |
| WLTIC-HC-ENT-001 | Hard | 46, 53, 117, 151, 34 | **Judgement:** combines five entity types into one rule to keep the score grouping workable |
| WLTIC-HC-DEED-001 | Hard | 54-55 | Deed validity list plus "must be recorded ... before the interest of the grantee can be insured" |
| WLTIC-HC-LEGAL-001 | Hard | 58 | "A policy should not be issued for platted property unless the plat has been properly filed of record" |
| WLTIC-HC-ID-001 | Hard | 71, 7 | "Westcor requires" government picture ID |
| WLTIC-HC-SUB-001 | Hard | 138 | "does not authorize the issuance of its policies ... pursuant to blanket, automatic, or general subordinations" |
| WLTIC-SC-DOCS-001 | Soft | 55, 148, 46-151, 99, 139 | Items collected from five topics; document list is my synthesis, not one page |
| 18 x WLTIC-RF-* | Referral | see skill | Counsel-approval or discretionary items |

### Reviewed and not made constraints

| Provision | Printed page | Reason |
|---|---|---|
| Quantity of acreage not insured without counsel permission | 9 | Coverage-scope instruction; no submission-level prohibition |
| Access must be legal, not prescriptive or implied | 6 | Schedule B exception handling; submission may still be insured with exception |
| Encroachments, setbacks, survey exceptions | 27, 68, 139-143 | Insured with exception or affirmative wording; not eligibility |
| Easements, restrictions, reversionary clauses, party walls, railroads (rights of way) | 65-67, 118, 126-129, 132-133 | Exception handling |
| Judgments, federal tax liens, environmental liens, vendor's liens, mechanics' liens, taxes | 73-75, 70, 95-96, 104-106, 148, 154 | Pay, release, subordinate or except. Mechanics' lien coverage moved to referral (WLTIC-RF-MECH-001) |
| Foreclosure procedure | 78-81 | State-specific procedure the manual does not make testable; referral if needed in future |
| Cash reporting, FIRPTA | 30-32, 77 | Reporting duties of the closing agent; cash is a referral indicator only |
| Condominium, PUD, leasehold, life estate, options, trusts requirements | 39-40, 119, 97-98, 100-101, 114-115, 151 | Document and exception lists; new condo, life estate and options are referral indicators |
| Divorce, probate, inheritance, receivers, missing persons | 60, 122, 94, 128, 111 | Procedural document lists; missing persons is a referral indicator |
| Bona fide purchaser, consideration | 25 | Concept; gift deed is a referral indicator |
| Zoning | 161 | Excluded by ALTA policy itself |
| Endorsements | 69 | Section removed by the manual |

### Unresolved source and classification issues

1. **No program name or eligibility list** in the manual (section 2). Program name and prefix chosen by me.
2. **Bankruptcy appeal period.** The manual states 10 days (extendable to 20). Current federal practice may differ. The skill does not test day counts; it tests the stated "appeal period expired" fact. Confirm source currency.
3. **Survey rules disagree.** p.72 accepts prior surveys within 10 years; p.139 says 6 years; Florida is carved out. Not used as a constraint. Confirm which governs before adding survey rules.
4. **Currency of the manual.** 2017, with a Form 8300 sample dated Rev 2/92 and a "2003 Supreme Court" reference. Not tested.
5. Whether TAXT, HEIR and DIL are Hard (with approval PASS path) or Referral. I chose Hard.
6. Whether ENT should be split by entity type. Combined to keep the Score skill at ten equal rules.
7. Issuing Agency Agreement controls on conflict and was not supplied.
8. State-law variation is pervasive (homestead, acknowledgments, mechanics' liens, redemption). The skill never FAILs on it.

## 4. Determinism evidence

Each constraint states a PASS, FAIL (hard only) and INDETERMINATE condition. Branch partitions were checked by hand:

| Rule | Partition | Boundary |
|---|---|---|
| TAXT-001 | PASS (none, or 20 years or more, or counsel authorization) / FAIL (under 20 years, no authorization) / INDETERMINATE (status or date missing) | Exactly 20 years is PASS ("less than twenty years" fails) |
| BKRP-001 | PASS (none, or abandoned or sold by order with appeal period expired) / FAIL (not released, or still appealable) / INDETERMINATE | Appeal period expired with no appeal is PASS |
| POA-001 | PASS (none, or all four tests met) / FAIL (any test failed) / INDETERMINATE (any test missing, none failed) | FAIL on one failed test even if others are missing |
| DEED-001, GUARD-001, ENT-001 | Any affirmative violation gives FAIL; missing evidence gives INDETERMINATE | n/a |

Two-answer check: POA-001 with revoked = true and recording not supplied gives FAIL, because a FAIL is never erased by missing data. Confirmed by the stated rule.

## 5. Adversarial tests (hand-traced, synthetic)

| # | Scenario | Expected | Result |
|---|---|---|---|
| 1 | Every constraint PASS, docs complete | MATCH - Confirmed | As expected |
| 2 | insured_estate_type = mineral estate only | MIN FAIL, NOT MATCH | As expected |
| 3 | One applicable document missing | SC-DOCS INDETERMINATE, never FAIL, Conditional | As expected |
| 4 | Tax deed recorded 19 years ago, no authorization | TAXT FAIL | As expected |
| 5 | Same, tax deed recorded exactly 20 years ago | TAXT PASS | As expected |
| 6 | Same 19-year deed with counsel authorization stated | TAXT PASS | As expected |
| 7 | Deed in lieu stated, authorization only "to be requested" | DIL FAIL (request is not authorization) | As expected |
| 8 | Bankruptcy stated, order entered 5 days ago | BKRP FAIL (still appealable) | As expected |
| 9 | No bankruptcy field at all | BKRP INDETERMINATE, not PASS | As expected |
| 10 | poa_used = true, poa_revoked = "TBD" | POA INDETERMINATE (placeholder is missing) | As expected |
| 11 | Two fields disagree on poa_recorded | INDETERMINATE, both named, ValidationIssues entry | As expected |
| 12 | Gift deed in chain | RF-GIFT-001 reported, statuses unchanged | As expected |
| 13 | Mineral-only (FAIL) plus missing document | NOT MATCH, document gap still listed | As expected |
| 14 | Timeshare = vacation license | TSH FAIL | As expected |
| 15 | Submission unreadable | Validation Required, Match false, Blocked pending validation | As expected |

These were traced by reading the rules. They were **not** executed against a model.

## 6. Contract validation

Script check (Python, `-I`) on the SKILL.md:

| Check | Result |
|---|---|
| Fenced json blocks parse | 4 of 4 parse |
| Rule IDs well formed, unique | 15 hard, 1 soft, 18 referral; no duplicates |
| Non-ASCII characters in file | 0 |
| Output block keys vs the three examples | Examples reviewed by hand; same keys. Empty arrays are empty containers |
| Enum values in examples | All declared. `Match` true never coexists with FAIL |

Contract is **proposed, not platform-confirmed**; it follows the Texas match skill's shape field for field.

## 7. Frontmatter limits

| Field | Measured | Cap | Result |
|---|---|---|---|
| `description` | 486 characters | 1024 | PASS |
| `name` | 32 characters | 64 | PASS |

`name` matches the folder. Metadata values are strings.

## 8. Format validation

| Check | Result |
|---|---|
| `skills-ref validate` | **Not run** - tool not installed here |
| `SKILL.md` line count | 381 of 500, PASS |
| Single file; no `references/` folder | PASS |
| Frontmatter keys | `name`, `description`, `metadata` only |

## 9. Open items

- Submission schema not supplied; all field names are proposals.
- Output contract proposed; confirm against the platform.
- Confirm program name, slug and `WLTIC` prefix.
- Human decisions on issues 1-8 in section 3.
- Supply the Issuing Agency Agreement if it should override the manual.
- Run `skills-ref validate ./westcor-title-underwriting-match`.
- Check quotes against the PDF text layer.

## 10. Score skill

See `skills-test/westcor-title-underwriting-score/CRITIQUE.md`.
