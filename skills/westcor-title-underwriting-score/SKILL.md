---
name: westcor-title-underwriting-score
description: Calculates a 0-100 match-strength Score for a title insurance submission that already matched the Westcor Land Title Insurance Company Underwriting Program (Westcor Underwriting Manual, 2017), based on how fully the insurability constraints are confirmed, and returns a JSON score narrative with calculation details. Use after westcor-title-underwriting-match returns MATCH - Confirmed or MATCH - Conditional (HardConstraintsResult.Match is true), or when the user asks to score a Westcor submission. Measures match strength only and never decides match vs not-match; premium, fees, coverage amounts and referral indicators never move the score.
metadata:
  display_name: "Westcor Land Title Insurance Underwriting Program - Score"
  program_name: "Westcor Land Title Insurance Underwriting Program"
  source_guideline: "westcor.pdf"
  version: "0.1"
  companion_match_skill: "westcor-title-underwriting-match"
---

# Westcor Land Title Insurance Underwriting Program - Score Skill

## How to use this skill

1. Read the submission json. Eligibility is decided upstream: `HardConstraintsResult.Match` is always `true`. Do not re-evaluate, re-derive or recompute any hard constraint, and do not restate or change `Match`.
2. Take each hard constraint's `Status` exactly as given. It is `PASS` or `INDETERMINATE` only. A hard constraint absent from the `HardConstraints` array counts as `INDETERMINATE`.
3. Take the soft constraint's result from the `SoftConstraints` array carried with the submission, and the supporting-document list from `documents_provided` if present.
4. Score each rule in the "Rule groups and weights" section, roll up with the "Final score" formula, and return the "Output format" below.
5. Never score premium, rate, policy fee, coverage amount or liability limit. They do not move the score in either direction.
6. `ReferralIndicators` in the input are score-neutral. Read them for the narrative only.

Statuses: `PASS` means the constraint is confirmed, or the submission affirmatively states the triggering facet is absent. `INDETERMINATE` means required information is missing, so the constraint scores lower than a confirmed `PASS`. A placeholder or default value ("N/A", "TBD", "unknown", an empty list) is not evidence; treat it as missing.

The manual states no fit bands, so this skill names none and uses no numeric band edges.

## Rule groups and weights

The group below carries the full 100%, and every rule inside it carries an equal 10% weight, because the manual states no preference among its factors. All scores below are **designed, not sourced**: the manual gives no weights or scores. They are listed as open items for calibration.

### RG-CONS Constraints (weight 100%)

Every rule has a score in 0-100 and an `unknown` path. No rule scores 0 for missing data. A tiered rule counts the `INDETERMINATE` results among the constraints it maps to.

| Rule | Category | Maps to (match-skill rule) | Scoring | Weight | Manual basis (printed page) |
|---|---|---|---|---|---|
| R-TITLE | tiered-numeric | `WLTIC-HC-ADVP-001`, `WLTIC-HC-TAXT-001`, `WLTIC-HC-HEIR-001` | 0 INDETERMINATE -> 100; 1 -> 85; 2 -> 70; 3 -> 50 | 10% | pp.10, 86, 146: basis of the vested title |
| R-SPEC | tiered-numeric | `WLTIC-HC-DIL-001`, `WLTIC-HC-MIN-001`, `WLTIC-HC-TSH-001`, `WLTIC-HC-CEM-001` | 0 INDETERMINATE -> 100; 1 -> 90; 2 -> 75; 3 -> 60; 4 -> 50 | 10% | pp.57, 107, 150, 33: special estates and transactions |
| R-BKRP | boolean | `WLTIC-HC-BKRP-001` | PASS -> 100; INDETERMINATE -> 50 | 10% | pp.18, 23: bankruptcy orders final |
| R-REP | tiered-numeric | `WLTIC-HC-GUARD-001`, `WLTIC-HC-POA-001` | both PASS -> 100; exactly one INDETERMINATE -> 75; both INDETERMINATE -> 50 | 10% | pp.85, 109, 120: representative authority |
| R-ENT | boolean | `WLTIC-HC-ENT-001` | PASS -> 100; INDETERMINATE -> 50 | 10% | pp.46, 53, 117, 151: entity existence and authority |
| R-DEED | boolean | `WLTIC-HC-DEED-001` | PASS -> 100; INDETERMINATE -> 50 | 10% | pp.54-55: valid, recorded conveyances |
| R-DESC | boolean | `WLTIC-HC-LEGAL-001` | PASS -> 100; INDETERMINATE -> 50 | 10% | p.58: insurable legal description |
| R-ID | boolean | `WLTIC-HC-ID-001` | PASS -> 100; INDETERMINATE -> 50 | 10% | p.71: signatory identity proof |
| R-SUB | boolean | `WLTIC-HC-SUB-001` | PASS -> 100; INDETERMINATE -> 50 | 10% | p.138: no blanket subordination |
| R-DOCS | tiered-numeric | `WLTIC-SC-DOCS-001` | applicable documents missing: 0 -> 100; 1 -> 90; 2 -> 80; 3 -> 65; 4 or more, or the count cannot be determined -> 50 | 10% | pp.46, 55, 71, 99, 112, 120, 139, 148, 151: supporting documents |

Rule notes:

- **R-DOCS** counts how many applicable items are not supplied: recorded vesting deed(s); written taxes and assessments status; authority documents for representative or entity signers; payoff or release for each lien to be removed; current survey if deletion of the survey exception is sought. An item the transaction does not need is not counted as missing. Read the count from `documents_provided`. If that field is absent, use the soft constraint's `Narrative`. If the count still cannot be determined, use the last branch. This rule scores document presence only; the substance of authority, liens and descriptions is already scored in R-ENT, R-REP and R-DESC.
- **Facet ownership.** Each submission facet is scored by exactly one rule. Grouping in R-TITLE, R-SPEC and R-REP follows the manual's own grouping (chain-of-title basis, special estates, authority of representatives) and keeps the rule weights equal.
- **Branch coverage.** Every status combination reaches exactly one branch in every rule: a boolean rule has two, R-REP has three, R-TITLE has four and R-SPEC has five (the count of `INDETERMINATE` results).

## Referral triggers and provisions not scored

Referral indicators from the match skill are score-neutral. None appears in any rule above; a submission value that triggers one scores identically to the same field unknown.

| Referral indicator | Provision | Why neutral |
|---|---|---|
| `WLTIC-RF-GIFT-001` | Gift deed (p.82) | Needs underwriting counsel approval; no objective test |
| `WLTIC-RF-FRAUD-001` | Fraud red flags (p.55) | Indicators only, "will not always confirm wrong-doing" |
| `WLTIC-RF-POA-001` | POA presented at closing (p.121) | Verification step, not a status |
| `WLTIC-RF-FORF-001` | Forfeiture or drug indicators (p.64) | Counsel approval required |
| `WLTIC-RF-CASH-001` | Cash over $10,000 (pp.30-32) | Reporting duty, not an insurability test |
| `WLTIC-RF-INDN-001` | Indian lands (p.93) | Authorization request |
| `WLTIC-RF-SYN-001` | Synthetic lease (p.145) | Counsel approval required |
| `WLTIC-RF-SEV-001` | Severed improvements (p.91) | Counsel approval required |
| `WLTIC-RF-AIR-001` | Airspace (p.15) | Counsel approval required |
| `WLTIC-RF-CEM-001` | Whole cemetery (p.33) | Authorization request |
| `WLTIC-RF-NCON-001` | New condominium project (p.40) | Counsel authorization |
| `WLTIC-RF-MECH-001` | Construction begun or unfiled lien coverage (pp.105-106) | Counsel approval required |
| `WLTIC-RF-WATER-001` | Waterfront and wetlands (p.159) | Counsel approval required |
| `WLTIC-RF-LIFE-001` | Life estate (p.100) | Local underwriter approval |
| `WLTIC-RF-OPT-001` | Option to purchase (p.115) | Counsel approval required |
| `WLTIC-RF-MISS-001` | Missing person owner (p.111) | Underwriter approval |
| `WLTIC-RF-RAIL-001` | Railroad in chain (p.126) | Source of title review |
| `WLTIC-RF-CFD-001` | Unrecorded contract for deed (p.45) | Match-skill indicator |

Not scored, by rule: premium, rate, policy fee, coverage amount and liability limit, Schedule B exceptions and affirmative coverage wording, endorsements, and every provision the match skill lists as a referral indicator.

## Final score

`Score` is a two-level weighted sum, rounded to the nearest integer (a value ending in exactly .5 rounds up).

- Rule group: `RG-CONS = sum(rule_score x rule_weight%)` over the rules in the table. Rule weights sum to 100%.
- Roll-up: `final_score = RG-CONS x 1.00`. Rule-group weights sum to 100%.
- `Score` is an integer in 0 to 100. Every rule score is in 0 to 100 and weights are positive and sum to 100%, so the total cannot leave that range.

Fit wording for the narrative is qualitative and not tied to numeric edges: use **CONFIRMED FIT** when no hard constraint is `INDETERMINATE` and no applicable document is missing, otherwise **CONDITIONAL FIT**.

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
- [ ] No referral indicator, premium, fee or coverage amount influenced any rule score.
- [ ] Output contains only `ScoreNarrative`, `ScoreCalculationDetails` and `Score`, in that order.

## Worked example

All values are synthetic. Rule order in each calculation line: R-TITLE, R-SPEC, R-BKRP, R-REP, R-ENT, R-DEED, R-DESC, R-ID, R-SUB, R-DOCS.

**(i) Complete and clean.** Every hard constraint `PASS`, soft constraint `PASS`, no applicable document missing. Every rule scores 100.

```json
{
  "ScoreNarrative": [
    "Lifters: RG-CONS (100 -- every insurability constraint confirmed and every applicable supporting document supplied).",
    "Drags: none -- no rule scored below 100.",
    "Final: 100/100 CONFIRMED FIT. All insurability information confirmed; no data gaps."
  ],
  "ScoreCalculationDetails": [
    "RG-CONS = (100*10%) + (100*10%) + (100*10%) + (100*10%) + (100*10%) + (100*10%) + (100*10%) + (100*10%) + (100*10%) + (100*10%) = 100",
    "final_score = (100*100%) = 100 -> 100"
  ],
  "Score": 100
}
```

**(ii) Eligible but sparse.** Match is true. `PASS`: chain-of-title, special-estate, deed, legal-description and subordination constraints except as noted. `INDETERMINATE`: heirship evidence (one of the three R-TITLE constraints), bankruptcy status, power of attorney recording and revocation (one of the two R-REP constraints), entity authority evidence, signatory identity. Two applicable documents missing. Rule scores: R-TITLE 85, R-SPEC 100, R-BKRP 50, R-REP 75, R-ENT 50, R-DEED 100, R-DESC 100, R-ID 50, R-SUB 100, R-DOCS 80.

```json
{
  "ScoreNarrative": [
    "Lifters: R-SPEC (100 -- no deed in lieu, mineral-only, club-membership or burial-plot issue), R-DEED (100 -- recorded and executed), R-DESC (100 -- plat recorded, description consistent), R-SUB (100 -- no blanket subordination).",
    "Drags: R-BKRP (50 -- bankruptcy order and appeal status not supplied), R-ENT (50 -- entity authority evidence not supplied), R-ID (50 -- signatory identity verification not supplied), R-REP (75 -- power of attorney recording and revocation unconfirmed), R-TITLE (85 -- heirship evidence not supplied), R-DOCS (80 -- two applicable documents missing).",
    "Final: 79/100 CONDITIONAL FIT. No adverse signals; score held back by unconfirmed authority, bankruptcy and identity information and two missing supporting documents."
  ],
  "ScoreCalculationDetails": [
    "RG-CONS = (85*10%) + (100*10%) + (50*10%) + (75*10%) + (50*10%) + (100*10%) + (100*10%) + (50*10%) + (100*10%) + (80*10%) = 79",
    "final_score = (79*100%) = 79 -> 79"
  ],
  "Score": 79
}
```

**(iii) Referral twin.** Take submission (ii) and add `WLTIC-RF-GIFT-001` to its `ReferralIndicators` (a gift deed in the chain). No rule reads referral indicators, so every rule scores as in (ii) and the output is identical to (ii), including `Score` 79. A referral trigger never moves the score.
