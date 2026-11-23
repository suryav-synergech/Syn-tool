---
name: texas-fair-plan-poa-match
description: Evaluates a commercial property submission against the Texas FAIR Plan Association (TFPA) Commercial Property Owners Association Program (Underwriting Manual, March 10, 2025) hard constraints, soft constraint and referral indicators, and returns a JSON result of MATCH - Confirmed, MATCH - Conditional, NOT MATCH or Validation Required with constraint-level findings. Use when deciding whether an HOA, POA or condominium association submission is eligible for the TFPA program, or when the user mentions TFPA, Texas FAIR Plan, declinations or program eligibility. Does not compute a score; the companion skill texas-fair-plan-poa-score does that for eligible submissions.
metadata:
  display_name: "TFPA Commercial Property Owners Association Program - Match"
  program: "Texas FAIR Plan Association Commercial Property Owners Association Program"
  source: "Guidelines 1(Texas).pdf, Underwriting Manual, 2025-03-10"
  version: "0.1"
---

# TFPA Commercial Property Owners Association Program - Match Skill

## How to use this skill

1. Read the submission. Apply only the rules in this file. Do not use general insurance knowledge, other programs' rules or market convention.
2. Evaluate every hard and soft constraint below **independently**. One result never decides another.
3. Check each referral indicator against the submission. Indicators have no status and never change the Match result.
4. Combine results with the overall decision table and return the output format below.
5. Never calculate, estimate or mention a score, points, weights or bands. Scoring belongs to a separate Score skill.

Submission field names below are **proposed concept names**. Map each to the real field whose content answers the question. Do not rely on a name alone.

Page numbers are the **printed** footer numbers of the manual ("Pg | n"). The PDF file page is one higher.

## Statuses

| Status | Meaning |
|---|---|
| PASS | Supplied evidence satisfies the constraint |
| FAIL | Supplied evidence affirmatively violates a **hard** constraint. Only hard constraints can FAIL |
| INDETERMINATE | Required information is missing or cannot be established. Never use it for doubt about how to read a rule |

- Blank, null or not supplied means missing information.
- A placeholder or default ("N/A", "TBD", "unknown", "0", an empty list) is not evidence unless the schema defines its meaning. Treat it as missing.
- A missing corroborating field never cancels a FAIL that another populated field establishes.
- **Conflicting evidence:** if two populated fields of equal authority conflict and no field is named as governing, do not choose. The constraint is INDETERMINATE, the Narrative names both fields, and the conflict is added to `ValidationIssues`. Never FAIL on a disputed value.
- **Multiple buildings:** where a rule says "any building", one affirmative violation is enough for FAIL.
- Soft constraints are PASS or INDETERMINATE only.

## Overall decision

| Constraint results | `OverallMatchStatus` | `Match` | `ScoringEligibility` |
|---|---|---|---|
| Every applicable constraint PASS | MATCH - Confirmed | true | Permitted |
| At least one INDETERMINATE, no FAIL | MATCH - Conditional | true | Permitted |
| At least one hard FAIL | NOT MATCH | false | Not permitted |
| Evaluation missing, malformed or incomplete (a constraint not evaluated, submission unreadable) | Validation Required | false | Blocked pending validation |

- FAIL beats PASS and INDETERMINATE.
- A conditional result lists **every** INDETERMINATE constraint and the information needed to resolve it, in `MissingInformation` and `ConditionalQualifications`.
- Never report a conditional result as confirmed.

## Hard constraints

### TFPA-HC-INS-001 - Not currently insured by an authorized insurer
- **Source:** p.2, Program Eligibility Requirements. "Property Owners are eligible for Commercial multiperil coverage if they are not currently insured by an authorized insurer and:"
- **Fields:** `currently_insured_by_authorized_insurer` (governing); current-policy record (corroborating)
- **PASS:** states not currently insured by an authorized insurer
- **FAIL:** states currently insured by an authorized insurer, or a populated current-policy record shows an authorized insurer's term covering the proposed effective date
- **INDETERMINATE:** no field answers it
- **Note:** a cancelled, lapsed or non-renewed policy is not "currently insured". The manual does not define "authorized insurer"; do not narrow it.

### TFPA-HC-DECL-001 - Two declinations from different holding companies
- **Source:** pp.2-3, item 1. "At least two insurance companies, not in the same holding company as defined in Insurance Code section 823.002, licensed to write and actually writing commercial property insurance in Texas have declined to provide commercial property insurance (the names of the two insurance companies shall be identified)". p.5: "Declination of coverage from two standard market carriers (cancellation or nonrenewal by an authorized insurer may count as a declination)"
- **Fields:** `declinations[]` with carrier name, holding company, type, date (governing); `declinations_obtained`; `declination_list_complete`
- **PASS:** at least two named entries with different holding companies. Cancellation or non-renewal by an authorized insurer counts as an entry. Exactly two distinct holding companies is PASS
- **FAIL:** the submission states no declinations were obtained or sought, or states its list is complete and it holds fewer than two distinct holding companies
- **INDETERMINATE:** entries are missing, unnamed or holding company unknown, and no completeness statement exists
- **Note:** count distinct holding companies, not entries. Carrier licensing is not tested unless the submission carries a field for it.

### TFPA-HC-OFFER-001 - No valid comparable offer
- **Source:** pp.2-3, item 1. "the applicant has not received a valid offer of comparable commercial property insurance from an insurance company licensed in Texas, not including any surplus lines insurers"
- **Fields:** `valid_offer_received` (governing); `offers[]` with insurer and licensing type
- **PASS:** states no valid comparable offer, or lists only surplus lines offers
- **FAIL:** states a valid comparable offer from a Texas-licensed, non-surplus-lines insurer
- **INDETERMINATE:** no field answers it
- **Note:** "comparable" and "valid" are undefined. Use the submission's own statement; do not judge comparability.

### TFPA-HC-TAX-001 - No outstanding taxes or charges
- **Source:** p.3, item 2. "There are no outstanding taxes, assessments, penalties, or charges with respect to the property to be insured, except those covered under a properly filed deferral affidavit in compliance with §33.06 of the Property Tax Code;"
- **Fields:** `outstanding_taxes_or_charges` (governing); `deferral_affidavit_filed` (exception)
- **PASS:** none outstanding, or every outstanding item is covered by a properly filed §33.06 deferral affidavit
- **FAIL:** an outstanding item is stated and no deferral affidavit covers it (any building)
- **INDETERMINATE:** tax status not supplied, or an item is stated and affidavit coverage is not supplied

### TFPA-HC-VIOL-001 - No written notice of code violation
- **Source:** p.3, item 3. "The applicant has not received written notice from an authorized public entity stating that the property is in violation of any building, housing, air pollution, sanitation, health, fire or safety code, ordinance, or rule;"
- **Fields:** `public_entity_violation_notice` (governing)
- **PASS:** states no such notice
- **FAIL:** states such a notice was received
- **INDETERMINATE:** not supplied
- **Note:** the manual gives no cure exception. If a notice is stated as resolved, do not PASS and do not create an exception. Add a `ValidationIssues` entry for human review.

### TFPA-HC-VAC-001 - Not vacant or unoccupied
- **Source:** p.3, item 4. "Not Vacant or unoccupied; and"
- **Fields:** `occupancy_status` (governing); occupied vs total units (corroborating)
- **PASS:** occupancy shows the property is occupied
- **FAIL:** occupancy shows vacant or unoccupied (any building)
- **INDETERMINATE:** not supplied
- **Note:** no vacancy percentage is stated; do not invent one.

### TFPA-HC-REPAIR-001 - Not under repair or renovation
- **Source:** p.3, item 5. "Not under repair or renovation."
- **Fields:** `under_repair_or_renovation` (governing); `construction_status`, `renovation_details` (corroborating)
- **PASS:** states not under repair or renovation
- **FAIL:** states under repair or renovation (any building)
- **INDETERMINATE:** not supplied
- **Note:** no size or cost threshold is stated; do not add one.

### TFPA-HC-TYPE-001 - Eligible risk type
- **Source:** p.3, Eligible Commercial Risks 1-2: "Property Owners' Associations (POA)", "Homeowners Association (HOA)". p.5, Submission Requirements: "Residential Condominium Associations"
- **Fields:** `applicant_entity_type` (governing)
- **PASS:** a property owners' association, homeowners association or residential condominium association
- **FAIL:** populated and identifies another entity type
- **INDETERMINATE:** not supplied
- **Note:** condominium associations appear in the class-code list (p.3) and submission requirements (p.5) but not in the numbered risk list. Treated as eligible on that evidence.

### TFPA-HC-CLASS-001 - Eligible CSP class code
- **Source:** p.3, Eligible Commercial Risks 3. "The eligible class codes to apply for the Property Owners' Association program are as follows:"
- **Eligible property codes:** 0311, 0321, 0331, 0332, 0333, 0341, 0342, 0343. **Eligible general liability code:** 62003
- **Fields:** `property_class_code`, `gl_class_code` (governing)
- **PASS:** every supplied code is in its list
- **FAIL:** a supplied code is not in its list
- **INDETERMINATE:** no class code supplied
- **Note:** the manual says the codes are for "statistical reporting purposes", so treating a non-listed code as FAIL is a reviewed judgement. Do not cross-check unit ranges or mercantile status unless the submission's own unit count affirmatively contradicts the code; then add a `ValidationIssues` entry.

### TFPA-HC-COND-001 - Habitable; not condemned or slated for destruction
- **Source:** p.10, Condition of the Structure. "The insured property must be habitable. Buildings that have been condemned or are scheduled for destruction, whatever the reason, are uninsurable."
- **Fields:** `condemned_status`, `scheduled_for_demolition` (governing); `habitability` (corroborating)
- **PASS:** states not condemned and not scheduled for destruction
- **FAIL:** states condemned or scheduled for destruction, or `habitability` states uninhabitable (any building)
- **INDETERMINATE:** neither status field and no habitability statement supplied

### TFPA-HC-ACV-001 - Coverage at least 100% of Actual Cash Value
- **Source:** p.10, Underwriting Requirements/Guidelines for Continuation of Coverage. "Minimum Coverage amount - 100% of Actual Cash Value". The Underwriting Rules introduction says the segment sets "the minimum standards to qualify or renew a policy"
- **Fields:** `requested_building_limit`, `actual_cash_value` per building (governing)
- **PASS:** for every building, requested limit is greater than or equal to actual cash value (equal is PASS)
- **FAIL:** for any building, requested limit is less than actual cash value
- **INDETERMINATE:** limit or actual cash value is missing for a building and no building shows a shortfall
- **Note:** replacement cost is not actual cash value; do not substitute it.

### TFPA-HC-INSP-001 - Agrees to property inspection
- **Source:** p.6, Binding of Coverage. "Policyholder must agree to allow inspection of the property at the Associations request."
- **Fields:** `inspection_consent` (governing)
- **PASS:** consent given
- **FAIL:** consent refused
- **INDETERMINATE:** not supplied

## Soft constraint

### TFPA-SC-DOCS-001 - Required quote documents supplied
- **Source:** p.5, Submission Requirements for Quote: declination of coverage from two standard market carriers; "Covenants, Conditions and Restrictions (CC&Rs)"; "Statement of Values for all building and structures"; "Replacement cost valuation for each building/structure (MSB or other acceptable valuation method/tool)"; "Loss Reports for last 5 years"
- **Fields:** `documents_provided[]` or a presence flag per item (governing)
- **PASS:** every item supplied
- **INDETERMINATE:** one or more items not supplied. List each in `MissingInformation`
- **Note:** this checks document presence only. The substance of the declinations is judged by `TFPA-HC-DECL-001`; do not let one result decide the other.

## Referral indicators

These are discretionary or give no testable standard. They have no status. When the submission affirmatively shows the condition, add an entry to `ReferralIndicators`. When information is absent, add nothing. They never cause FAIL or INDETERMINATE.

| ID | Indicator | Source and quote |
|---|---|---|
| TFPA-RF-ROOF-001 | Existing roof damage or deteriorated roof | pp.10-11, Roofing. "Risks with existing damage or deteriorated roofs may not be acceptable for coverage." |
| TFPA-RF-POOL-001 | Pool not properly enclosed, or unfenced | p.11, Liability Exposures. "Pools must be properly enclosed" |
| TFPA-RF-SPRK-001 | Sprinkler system not maintained or working | p.11. "Sprinkler systems must be maintained and working" |
| TFPA-RF-EXTG-001 | Fire extinguishers not accessible in common areas | p.11. "Fire extinguishers must be accessible in common areas" |
| TFPA-RF-GYM-001 | Gym equipment not properly maintained | p.11. "Gym equipment must be properly maintained" |
| TFPA-RF-SECU-001 | Common areas not secured to residents only | p.11. "Common areas must be secured to allow access to residents only" |
| TFPA-RF-DISR-001 | Property in disrepair or with existing damage | p.10. "Property in disrepair or with existing damage" |
| TFPA-RF-LIAB-001 | Excessive or unusual liability exposure | p.10. "Excessive/unusual liability exposure, including unfenced/secured pools." |

The p.10 items are introduced as "Conditions that may be cause for underwriting action, i.e., restriction of coverage, up to and including cancellation or nonrenewal of coverage." The wording is discretionary and the manual gives no objective test.

## Output format

Return a single JSON object with exactly these keys. Use empty arrays, not omitted keys. This contract is proposed, not platform-confirmed. `HardConstraintsResult` is the shape the Score skill consumes: when `Match` is true every `Status` is `PASS` or `INDETERMINATE`, and `FAIL` appears only when `Match` is false. List every hard constraint exactly once.

```json
{
  "ProgramIdentifier": "<program id>",
  "SubmissionIdentifier": "<id or null>",
  "OverallMatchStatus": "MATCH - Confirmed | MATCH - Conditional | NOT MATCH | Validation Required",
  "ScoringEligibility": "Permitted | Not permitted | Blocked pending validation",
  "HardConstraintsResult": {
    "Match": true,
    "HardConstraints": [
      {
        "RuleName": "<PREFIX>-HC-<TOKEN>-001",
        "RuleDescription": "<what the constraint requires>",
        "Status": "PASS | FAIL | INDETERMINATE",
        "Narrative": "<field values used and the reasoning>",
        "SourceReference": "<page, section, short verbatim quote>"
      }
    ]
  },
  "SoftConstraints": [
    {
      "RuleName": "<PREFIX>-SC-<TOKEN>-001",
      "RuleDescription": "<what the constraint requires>",
      "Status": "PASS | INDETERMINATE",
      "Narrative": "<field values used and the reasoning>",
      "SourceReference": "<page, section, short verbatim quote>"
    }
  ],
  "FailureExplanations": ["<constraint id: reason>"],
  "MissingInformation": ["<constraint id: field or document needed>"],
  "ConditionalQualifications": ["<constraint id: what remains unconfirmed>"],
  "ReferralIndicators": [
    {
      "RuleName": "<PREFIX>-RF-<TOKEN>-001",
      "Narrative": "<evidence from the submission>",
      "SourceReference": "<page, section, short verbatim quote>"
    }
  ],
  "ValidationIssues": ["<issue>"]
}
```

## Examples

All values are synthetic. In each example the constraint arrays are abbreviated to the entries that matter; a real result lists every hard constraint and the soft constraint.

**Confirmed**

```json
{
  "ProgramIdentifier": "TFPA-POA",
  "SubmissionIdentifier": "SYN-001",
  "OverallMatchStatus": "MATCH - Confirmed",
  "ScoringEligibility": "Permitted",
  "HardConstraintsResult": {
    "Match": true,
    "HardConstraints": [
      {
        "RuleName": "TFPA-HC-VAC-001",
        "RuleDescription": "Property must not be vacant or unoccupied",
        "Status": "PASS",
        "Narrative": "occupancy_status = Occupied. No vacancy indicated.",
        "SourceReference": "p.3, Program Eligibility Requirements item 4: 'Not Vacant or unoccupied'"
      }
    ]
  },
  "SoftConstraints": [
    {
      "RuleName": "TFPA-SC-DOCS-001",
      "RuleDescription": "Required quote documents must be supplied",
      "Status": "PASS",
      "Narrative": "All five required items are present in documents_provided.",
      "SourceReference": "p.5, Submission Requirements for Quote"
    }
  ],
  "FailureExplanations": [],
  "MissingInformation": [],
  "ConditionalQualifications": [],
  "ReferralIndicators": [],
  "ValidationIssues": []
}
```

**Not match** (one affirmative FAIL; the missing document does not hide it)

```json
{
  "ProgramIdentifier": "TFPA-POA",
  "SubmissionIdentifier": "SYN-002",
  "OverallMatchStatus": "NOT MATCH",
  "ScoringEligibility": "Not permitted",
  "HardConstraintsResult": {
    "Match": false,
    "HardConstraints": [
      {
        "RuleName": "TFPA-HC-VAC-001",
        "RuleDescription": "Property must not be vacant or unoccupied",
        "Status": "FAIL",
        "Narrative": "occupancy_status = Vacant for building 1.",
        "SourceReference": "p.3, Program Eligibility Requirements item 4: 'Not Vacant or unoccupied'"
      }
    ]
  },
  "SoftConstraints": [
    {
      "RuleName": "TFPA-SC-DOCS-001",
      "RuleDescription": "Required quote documents must be supplied",
      "Status": "INDETERMINATE",
      "Narrative": "Loss Reports for last 5 years not supplied.",
      "SourceReference": "p.5, Submission Requirements for Quote: 'Loss Reports for last 5 years'"
    }
  ],
  "FailureExplanations": ["TFPA-HC-VAC-001: occupancy_status is Vacant"],
  "MissingInformation": ["TFPA-SC-DOCS-001: Loss Reports for last 5 years"],
  "ConditionalQualifications": [],
  "ReferralIndicators": [],
  "ValidationIssues": []
}
```

**Conditional** (no FAIL, one INDETERMINATE, one referral indicator that does not change the result)

```json
{
  "ProgramIdentifier": "TFPA-POA",
  "SubmissionIdentifier": "SYN-003",
  "OverallMatchStatus": "MATCH - Conditional",
  "ScoringEligibility": "Permitted",
  "HardConstraintsResult": {
    "Match": true,
    "HardConstraints": [
      {
        "RuleName": "TFPA-HC-ACV-001",
        "RuleDescription": "Coverage amount must be at least 100% of Actual Cash Value",
        "Status": "INDETERMINATE",
        "Narrative": "requested_building_limit supplied; actual_cash_value not supplied for building 1.",
        "SourceReference": "p.10, Underwriting Requirements: 'Minimum Coverage amount - 100% of Actual Cash Value'"
      }
    ]
  },
  "SoftConstraints": [],
  "FailureExplanations": [],
  "MissingInformation": ["TFPA-HC-ACV-001: actual cash value for building 1"],
  "ConditionalQualifications": ["TFPA-HC-ACV-001: coverage of at least 100% of Actual Cash Value is unconfirmed"],
  "ReferralIndicators": [
    {
      "RuleName": "TFPA-RF-POOL-001",
      "Narrative": "pool_enclosed = false.",
      "SourceReference": "p.11, Liability Exposures: 'Pools must be properly enclosed'"
    }
  ],
  "ValidationIssues": []
}
```

## Grounding pass

Before returning, check each item and fix anything that fails.

- [ ] Every hard constraint appears once in `HardConstraints`, and the soft constraint appears in `SoftConstraints`.
- [ ] Every FAIL is a hard constraint resting on a populated field that affirmatively violates it.
- [ ] Every INDETERMINATE names the missing field or document, or the two conflicting fields.
- [ ] No placeholder or default was treated as evidence.
- [ ] Referral indicators have no status and did not change the overall result.
- [ ] `Match` is true only for Confirmed and Conditional, and `FAIL` appears only when `Match` is false.
- [ ] No score, points, weight or band appears in the output.
- [ ] Output keys match the format exactly.
