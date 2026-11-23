# CRITIQUE - westcor-title-underwriting-score (build-time only, not for runtime)

Meta-prompt: `prompt/meta-score-skill-creation-prompt.md` (pattern followed; the prompt file itself was not re-read for this run)
Skill: `skills/westcor-title-underwriting-score/SKILL.md`
Companion: `skills/westcor-title-underwriting-match/SKILL.md` (IDs cited verbatim)

## Final disposition: PROVISIONAL

Structurally consistent and arithmetically checked by script, but every score and weight is **designed, not sourced**, the score has no quality signal (open item 1), the input contract is assumed, and `skills-ref` was not run.

## 1. Weight sums (check A)

| Level | Weights | Sum |
|---|---|---|
| Rules in RG-CONS | R-TITLE, R-SPEC, R-BKRP, R-REP, R-ENT, R-DEED, R-DESC, R-ID, R-SUB, R-DOCS at 10% each | 100% (script) |
| Rule groups | RG-CONS 100% | 100% |

Equal weights because the manual states no preference among factors.

## 2. Per-rule integrity (check B)

| Rule | Category | Branches (score) | Exhaustive and exclusive | Sourced or designed |
|---|---|---|---|---|
| R-BKRP, R-ENT, R-DEED, R-DESC, R-ID, R-SUB | boolean | PASS 100; INDETERMINATE 50 | Yes. A constraint absent from input counts as INDETERMINATE | Scores designed; facet sourced to the cited page |
| R-REP | tiered-numeric | 0 / 1 / 2 INDETERMINATE -> 100 / 75 / 50 | Yes | Designed |
| R-TITLE | tiered-numeric | 0 / 1 / 2 / 3 INDETERMINATE -> 100 / 85 / 70 / 50 | Yes, three constraints give counts 0-3 | Designed |
| R-SPEC | tiered-numeric | 0 to 4 INDETERMINATE -> 100 / 90 / 75 / 60 / 50 | Yes, four constraints give counts 0-4 | Designed |
| R-DOCS | tiered-numeric | missing 0..3 -> 100, 90, 80, 65; 4 or more or unknown -> 50 | Yes. Five items, counts 0-5 plus unknown | Designed |

- All scores are within 0-100. No unknown branch scores 0.
- No rule has a FAIL branch or FAIL-tier score.
- Tier steps are not uniform across rules (R-TITLE drops 15 per gap, R-SPEC 10 then 15, R-DOCS 10 then 15). The steps keep the worst case at 50 in every rule. Flagged for calibration.

## 3. Referral inventory (check C)

All 18 referral indicators in the match skill appear in the score skill's table (script: sets equal) and in no rule (script: none found in the rule table). All are score-neutral.

| Not scored | Basis |
|---|---|
| Premium, rate, policy fee, coverage amount, liability limit | Out of scope by prompt rule |
| Schedule B exceptions, affirmative coverage, endorsements | Not eligibility facts; endorsements section removed from manual (p.69) |

## 4. Facet-to-group map (check E)

| Facet | Owner |
|---|---|
| Adverse possession, tax-sale title, heirship | R-TITLE |
| Deed in lieu, mineral-only, timeshare, burial plot | R-SPEC |
| Bankruptcy orders | R-BKRP |
| Guardian or conservator, power of attorney | R-REP |
| Entity existence and authority | R-ENT |
| Deed validity and recording | R-DEED |
| Legal description | R-DESC |
| Signatory identity | R-ID |
| Subordination | R-SUB |
| Supporting documents | R-DOCS |

One group, so no facet appears in two groups. Authority documents are counted in R-DOCS as presence only; their substance stays in R-ENT and R-REP.

## 5. Self-tests (check H)

Computed by hand and re-computed by script.

| Test | Rule scores (TITLE, SPEC, BKRP, REP, ENT, DEED, DESC, ID, SUB, DOCS) | RG-CONS | Score |
|---|---|---|---|
| (i) Complete, clean | 100 x10 | 100 | 100 |
| (ii) Eligible, sparse | 85, 100, 50, 75, 50, 100, 100, 50, 100, 80 | 79.0 (script) | 79 |
| (iii) Referral twin of (ii) | Same as (ii); gift-deed indicator added | 79.0 | 79 |

Referral-twin equality holds (79 = 79). The sparse example's `ScoreCalculationDetails` text matches the computed terms in order. The same arithmetic and score 79 are used in the agent YAML example (script: all six "79" occurrences present, zero non-ASCII characters).

## 6. Cross-reference integrity (check I)

Inventory from the match skill: 15 hard IDs, 1 soft ID, 18 referral IDs.

Script result: 10 rules, 16 mapped IDs; every hard and soft ID is mapped exactly once; none missing, none invalid; referral IDs equal the match skill's set.

## 7. Overrides (check C2)

None inside the manual that alter scoring. The Issuing Agency Agreement controls on conflict and was not supplied.

## 8. Frontmatter limits (check J)

| Field | Measured | Cap | Result |
|---|---|---|---|
| `description` | 505 characters | 1024 | PASS |
| `name` | 32 characters | 64 | PASS |

## 9. Format conformance (check K) - script- and hand-verified

| Check | Result |
|---|---|
| JSON blocks parse | 3 of 3 (script) |
| Non-ASCII characters | 0 (script) |
| Line count | 153 of 500 |
| Every example `Score` is an integer 0..100 | PASS |
| Weights sum to 100% | PASS (script) |
| Output block vs examples | Same three keys, same order, by inspection |
| `ScoreNarrative` shape | Three strings starting Lifters, Drags, Final; Final number equals Score |
| No FAIL or NOT MATCH logic; no hard-constraint evaluation | By inspection; no text search run for this skill |
| `skills-ref validate` | **Not run** - not installed. No lint is claimed |

## 10. Source inventory

- Manual: `westcor.pdf`, read as text and page images. No formatting-encoded tiers found.
- Companion match skill read for IDs.
- Points taken from the manual: only the facets and page references cited in the rule table. No numeric manual thresholds are used.

## 11. Open items

1. **The score carries no risk-quality signal.** Every judgement-based item in the manual is a referral indicator in the match skill, so the score measures how fully insurability facts are confirmed, not how good the risk is. Same limitation as the Texas score skill. Decision needed if a quality group is wanted.
2. Every rule score and weight is designed, not sourced; calibrate (50 for unknown, uneven tier steps, equal 10% weights).
3. Fit wording is qualitative (CONFIRMED FIT, CONDITIONAL FIT); no sourced fit bands exist.
4. Rounding of a value ending in exactly .5 is set to round up, copied from the Texas skill.
5. Input contract assumed: `HardConstraintsResult`, plus `SoftConstraints` and `documents_provided` carried with the submission. Confirm field names.
6. A hard constraint absent from the input is treated as INDETERMINATE; confirm.
7. Grouping of constraints into R-TITLE, R-SPEC and R-REP keeps ten equal-weight rules. Fifteen separate rules would need unequal integer weights.
8. Run `skills-ref validate ./westcor-title-underwriting-score`.
9. Match-skill open items (program name, bankruptcy appeal period, survey-rule conflict, manual age, Issuing Agency Agreement) still apply.
