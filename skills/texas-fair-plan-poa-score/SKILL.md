---
name: texas-fair-plan-poa-score
description: Calculates a 0-100 match-strength Score for a commercial property submission that already matched the Texas FAIR Plan Association (TFPA) Commercial Property Owners Association Program (Underwriting Manual, March 10, 2025), based on how fully the eligibility constraints are confirmed, and returns a JSON score narrative with calculation details. Use after texas-fair-plan-poa-match returns MATCH - Confirmed or MATCH - Conditional (HardConstraintsResult.Match is true), or when the user asks to score a TFPA submission. Measures match strength only and never decides match vs not-match; deductible, SIR, premium and referral indicators never move the score.
metadata:
  display_name: "TFPA Commercial Property Owners Association Program - Score"
  program_name: "Texas FAIR Plan Association Commercial Property Owners Association Program"
  source_guideline: "Guidelines 1(Texas).pdf"
  version: "0.1"
  companion_match_skill: "texas-fair-plan-poa-match"
---

# TFPA Commercial Property Owners Association Program - Score Skill

## How to use this skill

1. Read the submission json. Eligibility is decided upstream: `HardConstraintsResult.Match` is always `true`. Do not re-evaluate, re-derive or recompute any hard constraint, and do not restate or change `Match`.
2. Take each hard constraint's `Status` exactly as given. It is `PASS` or `INDETERMINATE` only. A hard constraint absent from the `HardConstraints` array counts as `INDETERMINATE`.
3. Take the soft constraint's result from the `SoftConstraints` array carried with the submission, and the required-document list from `documents_provided` if present.
4. Score each rule in the "Rule groups and weights" section, roll up with the "Final score" formula, and return the "Output format" below.
5. Never score deductible, SIR, premium, policy fee, payment plan or coverage limits. They do not move the score in either direction.
6. `ReferralIndicators` in the input are score-neutral. Read them for the narrative only.

Statuses: `PASS` means the constraint is confirmed. `INDETERMINATE` means required information is missing, so the constraint scores lower than a confirmed `PASS`. A placeholder or default value ("N/A", "TBD", "unknown", an empty list) is not evidence; treat it as missing.

No fit bands are stated in the guideline, so this skill names none and uses no numeric band edges.

## Rule groups and weights

The group below carries the full 100%, and every rule inside it carries an equal 10% weight, because the guideline states no preference among its factors. All scores below are **designed, not sourced**: the guideline gives no weights or scores. They are listed as open items for calibration.

### RG-CONS Constraints (weight 100%)

Every rule has a score in 0-100 and an `unknown` path. No rule scores 0 for missing data. A tiered rule counts the `INDETERMINATE` results among the constraints it maps to.

| Rule | Category | Maps to (match-skill rule) | Scoring | Weight | Guideline basis (printed page) |
|---|---|---|---|---|---|
| R-INS | boolean | `TFPA-HC-INS-001` | PASS -> 100; INDETERMINATE -> 50 | 10% | p.2, not currently insured by an authorized insurer |
| R-MKT | tiered-numeric | `TFPA-HC-DECL-001`, `TFPA-HC-OFFER-001` | both PASS -> 100; exactly one INDETERMINATE -> 75; both INDETERMINATE -> 50 | 10% | pp.2-3, item 1: declinations and no valid offer |
| R-TAX | boolean | `TFPA-HC-TAX-001` | PASS -> 100; INDETERMINATE -> 50 | 10% | p.3, item 2: taxes and charges |
| R-VIOL | boolean | `TFPA-HC-VIOL-001` | PASS -> 100; INDETERMINATE -> 50 | 10% | p.3, item 3: written code-violation notice |
| R-OCC | tiered-numeric | `TFPA-HC-VAC-001`, `TFPA-HC-REPAIR-001` | both PASS -> 100; exactly one INDETERMINATE -> 75; both INDETERMINATE -> 50 | 10% | p.3, items 4-5: occupied, not under repair |
| R-COND | boolean | `TFPA-HC-COND-001` | PASS -> 100; INDETERMINATE -> 50 | 10% | p.10, Condition of the Structure |
| R-CLS | tiered-numeric | `TFPA-HC-TYPE-001`, `TFPA-HC-CLASS-001` | both PASS -> 100; exactly one INDETERMINATE -> 75; both INDETERMINATE -> 50 | 10% | p.3, Eligible Commercial Risks |
| R-ACV | boolean | `TFPA-HC-ACV-001` | PASS -> 100; INDETERMINATE -> 50 | 10% | p.10, minimum coverage amount |
| R-INSP | boolean | `TFPA-HC-INSP-001` | PASS -> 100; INDETERMINATE -> 50 | 10% | p.6, Binding of Coverage |
| R-DOCS | tiered-numeric | `TFPA-SC-DOCS-001` | missing documents: 0 -> 100; 1 -> 90; 2 -> 80; 3 -> 70; 4 -> 60; all missing, or the count cannot be determined -> 50 | 10% | p.5, Submission Requirements for Quote |

Rule notes:

- **R-DOCS** counts how many of these required quote documents are not supplied: declination of coverage from two standard market carriers; Covenants, Conditions and Restrictions (CC&Rs); Statement of Values for all buildings and structures; replacement cost valuation for each building or structure; loss reports for the last 5 years. Read the count from `documents_provided`. If that field is absent, use the soft constraint's `Narrative`. If the count still cannot be determined, use the last branch. This rule scores document presence only. The declination substance is already scored in R-MKT, so the declination document is counted here only as a document.
- **Facet ownership.** Each submission facet is scored by exactly one rule. Grouping in R-MKT, R-OCC and R-CLS follows the guideline's own grouping (items listed together) and keeps the rule weights equal.
- **Branch coverage.** Every status combination reaches exactly one branch in every rule: a boolean rule has two, and each paired rule has three (zero, one or two `INDETERMINATE`).

## Referral triggers and provisions not scored

Referral indicators from the match skill are score-neutral. None appears in any rule above; a submission value that triggers one scores identically to the same field unknown.

| Referral indicator | Provision | Why neutral |
|---|---|---|
| `TFPA-RF-ROOF-001` | Existing roof damage or deteriorated roof (pp.10-11) | "may not be acceptable"; discretionary, no objective test |
| `TFPA-RF-POOL-001` | Pool not properly enclosed (p.11) | Referral indicator in the companion match skill |
| `TFPA-RF-SPRK-001` | Sprinkler system not maintained or working (p.11) | Referral indicator in the companion match skill |
| `TFPA-RF-EXTG-001` | Fire extinguishers not accessible (p.11) | Referral indicator in the companion match skill |
| `TFPA-RF-GYM-001` | Gym equipment not maintained (p.11) | Referral indicator in the companion match skill |
| `TFPA-RF-SECU-001` | Common areas not secured to residents only (p.11) | Referral indicator in the companion match skill |
| `TFPA-RF-DISR-001` | Property in disrepair or with existing damage (p.10) | "may be cause for underwriting action"; discretionary |
| `TFPA-RF-LIAB-001` | Excessive or unusual liability exposure (p.10) | "may be cause for underwriting action"; discretionary |

Not scored, by rule: deductibles (wind or hail 2% and 5%, other perils), premium, policy fee and premium payment plans (pp.5-7), coverage limits (p.5), binding, cancellation, non-renewal and appeal provisions.

## Final score

`Score` is a two-level weighted sum, rounded to the nearest integer (a value ending in exactly .5 rounds up).

- Rule group: `RG-CONS = sum(rule_score x rule_weight%)` over the rules in the table. Rule weights sum to 100%.
- Roll-up: `final_score = RG-CONS x 1.00`. Rule-group weights sum to 100%.
- `Score` is an integer in 0 to 100. Every rule score is in 0 to 100 and weights are positive and sum to 100%, so the total cannot leave that range.

Fit wording for the narrative is qualitative and not tied to numeric edges: use **CONFIRMED FIT** when no hard constraint is `INDETERMINATE` and no required document is missing, otherwise **CONDITIONAL FIT**.

## Output format

Return a single JSON object with fields in this order - `ScoreNarrative`, `ScoreCalculationDetails`, `Score`.

- `ScoreNarrative` - array of strings, **MANDATORY when Match=true**: omitting this field or emitting it null/empty is a critical error. MUST contain exactly three entries in order: a string starting with "Lifters:", a string starting with "Drags:", and a string starting with "Final:" stating the score as "<final_score>/100" (the number MUST equal `Score`).
- `ScoreCalculationDetails` - non-empty array of strings: one string per rule group in the form `<RG-ID> = (<rule_score>*<rule_weight%>) + ... = <rule_group_score>`, then a final string `final_score = (<rule_group_score>*<rule_group_weight%>) = <pre-rounding value> -> <rounded final_score>`. ASCII only: `*` and `->`.
- `Score` - integer, 0 to 100, equal to the rounded value in the final string.
- No other field is emitted.

```json
{
  "ScoreNarrative": [
    "Lifters: <top group/rule contributions that raised the score, each with its sub-score>.",
    "Drags: <top group/rule contributions that lowered the score, each with its sub-score>.",
    "Final: <score>/100 <fit wording>. <one-line rationale tying the score to signal vs data gaps>."
  ],
  "ScoreCalculationDetails": [
    "<RG-ID> = (<rule_score>*<rule_weight%>) + ... = <rule_group_score>",
    "final_score = (<rule_group_score>*<rule_group_weight%>) = <pre-rounding value> -> <final_score>"
  ],
  "Score": 0
}
```

## Verification pass

Before returning, check each item and fix anything that fails. Repeat until every item passes.

- [ ] All ten rules were scored, each from the status of exactly the constraints it maps to.
- [ ] Each rule score is a value its table row allows (no interpolated or invented scores).
- [ ] Recomputed `RG-CONS` equals the sum in `ScoreCalculationDetails`, and `Score` equals the rounded value (.5 rounds up).
- [ ] `ScoreNarrative` has exactly three entries starting "Lifters:", "Drags:", "Final:", and the number in "Final:" equals `Score`.
- [ ] No referral indicator, deductible, premium or coverage limit influenced any rule score.
- [ ] Output contains only `ScoreNarrative`, `ScoreCalculationDetails` and `Score`, in that order.

## Worked example

All values are synthetic. Rule order in each calculation line: R-INS, R-MKT, R-TAX, R-VIOL, R-OCC, R-COND, R-CLS, R-ACV, R-INSP, R-DOCS.

**(i) Complete and clean.** Every hard constraint `PASS`, soft constraint `PASS`, no missing documents. Every rule scores 100.

```json
{
  "ScoreNarrative": [
    "Lifters: RG-CONS (100 -- every eligibility constraint confirmed and every required quote document supplied).",
    "Drags: none -- no rule scored below 100.",
    "Final: 100/100 CONFIRMED FIT. All eligibility information confirmed; no data gaps."
  ],
  "ScoreCalculationDetails": [
    "RG-CONS = (100*10%) + (100*10%) + (100*10%) + (100*10%) + (100*10%) + (100*10%) + (100*10%) + (100*10%) + (100*10%) + (100*10%) = 100",
    "final_score = (100*100%) = 100 -> 100"
  ],
  "Score": 100
}
```

**(ii) Eligible but sparse.** Match is true. `PASS`: current insurance, occupancy, habitability, risk type. `INDETERMINATE`: declinations, offers, taxes, violation notice, repair status, class code, actual cash value, inspection consent. Three required documents missing. Rule scores: R-INS 100, R-MKT 50 (both `INDETERMINATE`), R-TAX 50, R-VIOL 50, R-OCC 75 (repair status only), R-COND 100, R-CLS 75 (class code only), R-ACV 50, R-INSP 50, R-DOCS 70 (three missing).

```json
{
  "ScoreNarrative": [
    "Lifters: R-INS (100 -- not currently insured confirmed), R-COND (100 -- habitable, not condemned), R-OCC (75 -- occupied confirmed, repair status unconfirmed), R-CLS (75 -- risk type confirmed, class code unconfirmed).",
    "Drags: R-MKT (50 -- declination and offer evidence not supplied), R-TAX (50 -- tax status not supplied), R-VIOL (50 -- violation-notice status not supplied), R-ACV (50 -- actual cash value not supplied), R-INSP (50 -- inspection consent not supplied).",
    "Final: 67/100 CONDITIONAL FIT. No adverse signals; score held back by unconfirmed eligibility information and three missing quote documents."
  ],
  "ScoreCalculationDetails": [
    "RG-CONS = (100*10%) + (50*10%) + (50*10%) + (50*10%) + (75*10%) + (100*10%) + (75*10%) + (50*10%) + (50*10%) + (70*10%) = 67",
    "final_score = (67*100%) = 67 -> 67"
  ],
  "Score": 67
}
```

**(iii) Referral twin.** Take submission (ii) and add `TFPA-RF-POOL-001` to its `ReferralIndicators` (an unenclosed pool). No rule reads referral indicators, so every rule scores as in (ii) and the output is identical to (ii), including `Score` 67. A referral trigger never moves the score.
