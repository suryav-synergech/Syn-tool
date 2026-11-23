---
name: westcor-title-underwriting-match
description: Evaluates a title insurance submission against the Westcor Land Title Insurance Company Underwriting Program (Westcor Underwriting Manual, 2017) hard constraints, soft constraint and referral indicators, and returns a JSON result of MATCH - Confirmed, MATCH - Conditional, NOT MATCH or Validation Required with constraint-level findings. Use when deciding whether a title transaction can be insured under the Westcor program without special approval, or when the user mentions Westcor, title insurability, vesting, liens or underwriting eligibility. Does not compute a score; the companion skill westcor-title-underwriting-score does that for eligible submissions.
metadata:
  display_name: "Westcor Land Title Insurance Underwriting Program - Match"
  program: "Westcor Land Title Insurance Underwriting Program"
  source: "westcor.pdf, Westcor Underwriting Manual, 2017"
  version: "0.1"
---

# Westcor Land Title Insurance Underwriting Program - Match Skill

## How to use this skill

1. Read the submission. Apply only the rules in this file. Do not use general title insurance knowledge, other programs' rules or market convention.
2. Evaluate every hard and soft constraint below **independently**. One result never decides another.
3. Check each referral indicator against the submission. Indicators have no status and never change the Match result.
4. Combine results with the overall decision table and return the output format below.
5. Never calculate, estimate or mention a score, points, weights or bands. Scoring belongs to a separate Score skill.
6. Never consider premium, rate, policy fee or coverage amount as an eligibility factor.

Submission field names below are **proposed concept names**. Map each to the real field whose content answers the question. Do not rely on a name alone.

Page numbers are the **printed** footer numbers of the manual. They equal the PDF page numbers.

The manual is a set of underwriting guidelines, and says that where it conflicts with the issuing agency agreement, the agreement controls. It also says many state-law points vary by jurisdiction. This skill tests only what the manual states. State-law variation is never a reason to FAIL.

## Statuses

| Status | Meaning |
|---|---|
| PASS | Supplied evidence satisfies the constraint, or the submission affirmatively states the triggering facet is absent |
| FAIL | Supplied evidence affirmatively violates a **hard** constraint. Only hard constraints can FAIL |
| INDETERMINATE | Required information is missing or cannot be established. Never use it for doubt about how to read a rule |

- Blank, null or not supplied means missing information.
- A placeholder or default ("N/A", "TBD", "unknown", "0", an empty list) is not evidence unless the schema defines its meaning. Treat it as missing.
- Several constraints apply only when a facet exists in the transaction (a bankruptcy, a deed in lieu, a tax sale). The facet is **absent** only if a field says so. Silence is INDETERMINATE, not PASS.
- A missing corroborating field never cancels a FAIL that another populated field establishes.
- **Conflicting evidence:** if two populated fields of equal authority conflict and no field is named as governing, do not choose. The constraint is INDETERMINATE, the Narrative names both fields, and the conflict is added to `ValidationIssues`. Never FAIL on a disputed value.
- **Multiple parcels or parties:** where a rule says "any", one affirmative violation is enough for FAIL.
- Soft constraints are PASS or INDETERMINATE only.
- **Approval fields:** where a rule is satisfied by "written authorization from Westcor underwriting counsel", the authorization must be stated as obtained in a populated field. A statement that it will be requested is not authorization.

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

### WLTIC-HC-ADVP-001 - No title resting on unadjudicated adverse possession
- **Source:** p.10, Adverse Possession. "Westcor requires a judicial determination of adverse possessory interests (i.e., final non-appealable court order in favor of the present owner or proposed insured) in order to insure title in the name of a party who claims property by adverse possession."
- **Fields:** `title_basis_adverse_possession` (governing); `adverse_possession_court_order_final` (exception)
- **PASS:** states title does not rest on adverse possession, or a final non-appealable order in favor of the owner or proposed insured is stated
- **FAIL:** states title or a claim rests on adverse possession and no final non-appealable order is stated
- **INDETERMINATE:** not supplied, or adverse possession is stated and order status is not supplied

### WLTIC-HC-BKRP-001 - Bankruptcy orders final and property released
- **Source:** p.18, Underwriting Instructions: "All property of the debtor should be considered unmarketable unless sold under court order or abandoned as part of the bankruptcy proceedings." p.23: "No title insurance may be issued based on a bankruptcy court order until after an appeal is no longer possible." p.20: appeal runs 10 days from the final order, extendable by 20 days.
- **Fields:** `current_party_bankruptcy` (governing); `property_abandoned_or_sold_by_order`, `bankruptcy_order_date`, `appeal_period_expired`, `bankruptcy_order_appealed` (corroborating)
- **PASS:** states no bankruptcy involving a current party or an owner in the chain, or a bankruptcy is stated and the property was abandoned or sold by recorded court order and the appeal period has expired with no appeal pending
- **FAIL:** a bankruptcy is stated and the property is neither abandoned nor sold by court order, or the order is still appealable or under appeal
- **INDETERMINATE:** bankruptcy status not supplied, or a bankruptcy is stated and order or appeal status is not supplied
- **Note:** a liens-survive-bankruptcy issue (p.23, "A lien going into bankruptcy is a lien coming out of bankruptcy") is a Schedule B exception matter. It is not tested here.

### WLTIC-HC-TAXT-001 - Tax-sale title aged or authorized
- **Source:** p.146, Tax Titles. "it is the policy of the Company not to insure property where the title search reveals that the title is derived from a tax sale that occurred less than twenty years before the effective date of the search." Exceptions are case-by-case with underwriting counsel (p.147).
- **Fields:** `tax_sale_in_chain` (governing); `tax_deed_recorded_date`, `search_effective_date`, `tax_sale_counsel_authorization` (corroborating)
- **PASS:** states no tax sale in the chain, or the tax deed has been of record twenty years or more, or written underwriting counsel authorization is stated
- **FAIL:** a tax sale in the chain occurred less than twenty years before the search effective date and no counsel authorization is stated
- **INDETERMINATE:** not supplied, or a tax sale is stated and its date is not supplied
- **Note:** age alone passes the policy test. The manual's nine guidelines on p.146-147 are for counsel and are not tested here.

### WLTIC-HC-HEIR-001 - Heirship judicially determined where title passes by intestacy
- **Source:** p.86, Heirs At Law. "Judicial determination of heirship by a final, non-appealable court order of competent jurisdiction is always required to vest title unless expressly waived by Westcor underwriting counsel." Affidavits "may not be relied upon to vest title in a current seller or borrower."
- **Fields:** `title_by_intestate_succession` (governing); `judicial_heirship_order_final`, `heirship_waiver_counsel` (corroborating)
- **PASS:** states title does not pass by intestate succession in the chain, or a final non-appealable heirship order is stated, or a counsel waiver is stated
- **FAIL:** title passes by intestate succession and only an affidavit of heirship, or nothing, supports it
- **INDETERMINATE:** not supplied, or intestate succession is stated and heirship evidence is not supplied
- **Note:** an affidavit relied on for a prior deed in the chain, with counsel approval, is outside this rule (p.86 item 2). Add a `ValidationIssues` entry if the submission states that use.

### WLTIC-HC-DIL-001 - Deed in lieu of foreclosure authorized
- **Source:** p.57, Underwriting Instructions. "A conveyance by deed in lieu of foreclosure can never be insured without prior authorization by Westcor underwriting counsel." Absolute-conveyance and voluntariness requirements are on p.56.
- **Fields:** `deed_in_lieu_in_chain` (governing); `deed_in_lieu_counsel_authorization`, `deed_in_lieu_absolute_conveyance`, `deed_in_lieu_voluntary` (corroborating)
- **PASS:** states no deed in lieu in the chain, or a deed in lieu is stated with written counsel authorization and it is stated to be an absolute, voluntary conveyance
- **FAIL:** a deed in lieu is stated and no counsel authorization is stated, or it is stated not to be an absolute conveyance (p.56, "Only those deeds in lieu which are voluntarily executed and deemed to be an absolute conveyance ... may be insured")
- **INDETERMINATE:** not supplied, or a deed in lieu is stated with authorization but absolute or voluntary status not supplied

### WLTIC-HC-MIN-001 - Estate insured is not mineral-only
- **Source:** p.107, Minerals. "Westcor will not insure just a mineral estate (or any other mineral interest or right)." p.108: "Westcor does not insure severed mineral interests or rights."
- **Fields:** `insured_estate_type`, `insured_interest_description` (governing)
- **PASS:** the estate to be insured includes the surface or land, and no mineral-only estate is requested
- **FAIL:** the estate to be insured is only a mineral estate, mineral interest or mineral right
- **INDETERMINATE:** not supplied

### WLTIC-HC-TSH-001 - Timeshare interest is an ownership interest
- **Source:** p.150, Timeshare Estates. "When a timeshare purchaser only obtains a contractual right-to-use ... commonly called a vacation license or club membership, no ownership rights are created ... Westcor agents are not authorized to issue its commitment or final policy insuring these types of transactions."
- **Fields:** `timeshare_interest_type` (governing)
- **PASS:** states the property is not a timeshare, or the timeshare interest is an ownership interest in real property (interval estate, time-span estate, undivided interest or leasehold)
- **FAIL:** the interest is a contractual right-to-use, vacation license or club membership
- **INDETERMINATE:** timeshare status not supplied, or a timeshare is stated and the interest type is not supplied

### WLTIC-HC-CEM-001 - No individual burial plot insured
- **Source:** p.33, Cemeteries. "Individual burial plots are not insurable."
- **Fields:** `insured_parcel_is_burial_plot` (governing)
- **PASS:** states the insured parcel is not an individual burial plot
- **FAIL:** states the insured parcel is an individual burial plot
- **INDETERMINATE:** not supplied
- **Note:** a whole cemetery or land containing plots is not a FAIL here. It is a referral indicator (`WLTIC-RF-CEM-001`).

### WLTIC-HC-GUARD-001 - Minor or incompetent owner acts through court-authorized representative
- **Source:** pp.85, 109. "Certified Copy of 1) Petition and Order Appointing ____, Guardian; and 2) Specific Court Order authorizing the (sale/mortgage/lease/etc.) ... must be obtained". For Uniform Probate Code states, recorded Letters of Conservatorship.
- **Fields:** `owner_is_minor_or_incompetent` (governing); `guardian_appointment_order`, `court_order_authorizing_transaction`, `conservator_letters_recorded` (corroborating)
- **PASS:** states no party is a minor or adjudicated incompetent, or each such party acts through an appointed representative and the authorizing order (or recorded conservator Letters) is stated
- **FAIL:** a party is stated to be a minor or adjudicated incompetent and the transaction is stated to proceed without an appointment or authorizing order, or a natural guardian signs for real property without a court order
- **INDETERMINATE:** not supplied, or such a party is stated and order status is not supplied
- **Note:** a physical handicap does not make a person incompetent (p.92).

### WLTIC-HC-POA-001 - Power of attorney valid
- **Source:** p.120, Powers of Attorney. A POA must give the attorney-in-fact power to convey or encumber real property; the principal must be alive and competent at execution and delivery unless the POA is durable; the POA must not be revoked; and it must be properly recorded.
- **Fields:** `poa_used` (governing); `poa_grants_real_property_power`, `poa_principal_alive_competent_or_durable`, `poa_revoked`, `poa_recorded` (corroborating)
- **PASS:** states no power of attorney is used, or all four tests are stated as satisfied
- **FAIL:** a POA is used and any test is stated as failed (no real-property power, principal deceased or incompetent and POA not durable, revoked, or not recorded)
- **INDETERMINATE:** not supplied, or a POA is used and any test is not supplied
- **Note:** a POA presented at closing is a referral indicator (`WLTIC-RF-POA-001`) and does not change this status.

### WLTIC-HC-ENT-001 - Entity existence and signing authority evidenced
- **Source:** p.46 Corporations (good standing; board resolution), p.53 Deeds (LLC and partnership authority), p.117 Partnerships, p.151 Trusts (trust agreement reviewed), p.34 Churches (entity status and authority).
- **Fields:** `grantor_or_borrower_entity_type` (governing); `entity_good_standing_verified`, `authorizing_resolution_or_agreement`, `signatory_authority_evidence` (corroborating)
- **PASS:** states every party is a natural person acting for themselves, or each entity party is stated as verified existing and in good standing and its signing authority is evidenced (resolution, operating agreement, partnership agreement, trust instrument or equivalent)
- **FAIL:** an entity is stated to be dissolved, not in good standing, not yet incorporated at the date of the conveyance, or its signatory is stated to lack authority
- **INDETERMINATE:** not supplied, or an entity is stated and existence or authority evidence is not supplied
- **Note:** which officers may sign is state law. Do not test officer titles.

### WLTIC-HC-DEED-001 - Conveyance instruments valid and recorded
- **Source:** pp.54-55, Deeds. A valid deed needs a competent grantor, an adequately identified grantee, granting language, a reasonably certain description, proper execution by grantor and spouse as applicable, and delivery and acceptance. "a deed must be recorded in the real property records of the county in which the land lies before the interest of the grantee can be insured." p.7: acknowledgment compliance.
- **Fields:** `vesting_deed_recorded` (governing); `deed_properly_executed`, `deed_acknowledged`, `deed_grantee_identified` (corroborating)
- **PASS:** each deed supporting the transaction is stated recorded and properly executed and acknowledged
- **FAIL:** any deed is stated unrecorded, unsigned by a required party, or lacks a required acknowledgment
- **INDETERMINATE:** recording or execution status not supplied
- **Note:** statutory acknowledgment wording is state law; test only the submission's own statements.

### WLTIC-HC-LEGAL-001 - Insurable legal description
- **Source:** p.58, Descriptions, Legal. "A policy should not be issued for platted property unless the plat has been properly filed of record." A metes and bounds description must close; the description in the contract, instructions, commitment, deed, mortgage and policy should match.
- **Fields:** `legal_description_type` (governing); `plat_recorded`, `description_closes`, `descriptions_consistent` (corroborating)
- **PASS:** each stated test is satisfied (plat recorded for platted property, metes and bounds closes, descriptions consistent)
- **FAIL:** a platted description rests on an unfiled plat, a metes and bounds description is stated not to close, or the documents' descriptions are stated to conflict with no correction
- **INDETERMINATE:** description status not supplied

### WLTIC-HC-ID-001 - Signatory identity proof
- **Source:** p.71, Execution of Instruments. "Westcor requires that you obtain proof positive (valid, government issue picture identification which includes signature) that persons executing instruments are, in fact, who they say they are." p.7: current government-issued picture ID with signature and physical description.
- **Fields:** `signatory_identity_verified` (governing); `id_type`, `id_current` (corroborating)
- **PASS:** states valid government picture ID with signature was verified for every signatory, or the notary personally knows them
- **FAIL:** states identity was not verified, or the ID is expired or lacks a signature
- **INDETERMINATE:** not supplied

### WLTIC-HC-SUB-001 - No reliance on blanket or automatic subordination
- **Source:** p.138, Subordination Agreements item 4. "Westcor does not authorize the issuance of its policies insuring a first lien position pursuant to blanket, automatic, or general subordinations."
- **Fields:** `subordination_relied_on` (governing); `subordination_type` (corroborating)
- **PASS:** states no subordination is relied on, or each subordination relied on is specific to this transaction (dated, names the new loan and amount, legal description, unconditional)
- **FAIL:** a blanket, automatic or general subordination is relied on to give first-lien priority
- **INDETERMINATE:** not supplied, or a subordination is relied on and its type is not supplied
- **Note:** the FDIC board-resolution requirement for bank subordinators (p.138) is a document item, scored in the soft constraint.

## Soft constraint

### WLTIC-SC-DOCS-001 - Supporting documents supplied
- **Source:** the manual's per-topic requirements, each cited below. An item counts only if the transaction has the facet it supports.
  1. Recorded vesting deed(s) for the chain of title (p.55)
  2. Written status of taxes and assessments from the taxing or levying authority (p.148, "Verbal information is not acceptable")
  3. Authority documents for any representative or entity signer: resolution, operating or partnership agreement, trust instrument, POA, court Letters (pp.46, 71, 120, 151)
  4. Payoff, release or satisfaction for each existing mortgage and lien to be removed (pp.99, 112)
  5. Current acceptable survey, if deletion of the survey exception is sought (p.139)
- **Fields:** `documents_provided[]` or a presence flag per item (governing)
- **PASS:** every applicable item supplied
- **INDETERMINATE:** one or more applicable items not supplied. List each in `MissingInformation`
- **Note:** this checks document presence only. The substance of authority, liens and descriptions is judged by the hard constraints.

## Referral indicators

These are discretionary or require underwriting counsel's approval or give no testable standard. They have no status. When the submission affirmatively shows the condition, add an entry to `ReferralIndicators`. When information is absent, add nothing. They never cause FAIL or INDETERMINATE.

| ID | Indicator | Source and quote |
|---|---|---|
| WLTIC-RF-GIFT-001 | Gift deed or deed for nominal or no consideration in the chain | p.82. "Westcor does NOT authorize its agents to insure gift deeds. Any request to insure the grantee of a gift deed must be approved by underwriting counsel." |
| WLTIC-RF-FRAUD-001 | Fraud red flags: stand-alone or family-member quitclaim deed, deed with no or defective acknowledgment, handwritten hurried form, with a new loan shortly after | p.55. "Experience has shown that sometimes 'stand-alone' deeds are shortly ... followed by a new loan on the same property." |
| WLTIC-RF-POA-001 | Power of attorney first presented at closing | p.121. "Verification of its authenticity is absolutely required." |
| WLTIC-RF-FORF-001 | Federal or state forfeiture in the chain, or drug-activity indicators | p.64. "Approval by Westcor underwriting counsel is absolutely required before such property may be insured." |
| WLTIC-RF-CASH-001 | Cash over $10,000, or structured instruments | pp.30-32. "Cash transactions must be reported on IRS Form 8300" |
| WLTIC-RF-INDN-001 | Indian lands | p.93. "the Company requires that the agent submit a Policy Authorization Request" |
| WLTIC-RF-SYN-001 | Synthetic lease or sale-leaseback re-characterization risk | p.145. "No such transaction may be insured without involvement of and approval by Westcor Underwriting Counsel." |
| WLTIC-RF-SEV-001 | Severed improvements, split fee | p.91. "All severed improvement transactions must be submitted to Westcor Underwriting counsel" |
| WLTIC-RF-AIR-001 | Airspace parcel (not condominium) | p.15. "unusual and extra - hazardous risk that must be submitted to and approved by Westcor Counsel." |
| WLTIC-RF-CEM-001 | Whole cemetery or land with burial plots | p.33. "Never issue title insurance on property platted for or used as burial plots unless specifically authorized" |
| WLTIC-RF-NCON-001 | New condominium project | p.40. "New condominium projects may not be insured without prior authorization by Westcor Counsel." |
| WLTIC-RF-MECH-001 | Construction begun, or unfiled mechanics' lien coverage requested | p.105-106. "cannot be given without approval from Westcor underwriting counsel." |
| WLTIC-RF-WATER-001 | Waterfront, wetlands, filled or accreted land | p.159. "Westcor underwriting counsel must approve reports, commitments, or policies covering wetlands" |
| WLTIC-RF-LIFE-001 | Life estate, remainder or reversion | p.100. "Call for local underwriter approval before insuring a transaction involving a life estate" |
| WLTIC-RF-OPT-001 | Option to purchase insured | p.115. "Approval by Westcor underwriting counsel is required for an option to be insured." |
| WLTIC-RF-MISS-001 | Missing person owner | p.111. "should not be insured without express underwriter approval" |
| WLTIC-RF-RAIL-001 | Railroad as owner in chain, or contiguous right-of-way | p.126. "determine the source of a railroad's title or ownership before insuring" |
| WLTIC-RF-CFD-001 | Unrecorded contract for deed | p.45. "Generally, unrecorded contracts are uninsurable." |

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
  "ProgramIdentifier": "WLTIC-TITLE-UW",
  "SubmissionIdentifier": "SYN-001",
  "OverallMatchStatus": "MATCH - Confirmed",
  "ScoringEligibility": "Permitted",
  "HardConstraintsResult": {
    "Match": true,
    "HardConstraints": [
      {
        "RuleName": "WLTIC-HC-ADVP-001",
        "RuleDescription": "Title must not rest on unadjudicated adverse possession",
        "Status": "PASS",
        "Narrative": "title_basis_adverse_possession = false.",
        "SourceReference": "p.10, Adverse Possession: 'Westcor requires a judicial determination of adverse possessory interests'"
      }
    ]
  },
  "SoftConstraints": [
    {
      "RuleName": "WLTIC-SC-DOCS-001",
      "RuleDescription": "Supporting documents must be supplied",
      "Status": "PASS",
      "Narrative": "Every applicable item is present in documents_provided.",
      "SourceReference": "p.148, Taxes and Assessments: 'Verbal information is not acceptable'"
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
  "ProgramIdentifier": "WLTIC-TITLE-UW",
  "SubmissionIdentifier": "SYN-002",
  "OverallMatchStatus": "NOT MATCH",
  "ScoringEligibility": "Not permitted",
  "HardConstraintsResult": {
    "Match": false,
    "HardConstraints": [
      {
        "RuleName": "WLTIC-HC-MIN-001",
        "RuleDescription": "Estate insured must not be mineral-only",
        "Status": "FAIL",
        "Narrative": "insured_estate_type = Mineral estate only.",
        "SourceReference": "p.107, Minerals: 'Westcor will not insure just a mineral estate'"
      }
    ]
  },
  "SoftConstraints": [
    {
      "RuleName": "WLTIC-SC-DOCS-001",
      "RuleDescription": "Supporting documents must be supplied",
      "Status": "INDETERMINATE",
      "Narrative": "Written tax status not supplied.",
      "SourceReference": "p.148, Taxes and Assessments: 'Verbal information is not acceptable'"
    }
  ],
  "FailureExplanations": ["WLTIC-HC-MIN-001: insured estate is a mineral estate only"],
  "MissingInformation": ["WLTIC-SC-DOCS-001: written status of taxes and assessments"],
  "ConditionalQualifications": [],
  "ReferralIndicators": [],
  "ValidationIssues": []
}
```

**Conditional** (no FAIL, one INDETERMINATE, one referral indicator that does not change the result)

```json
{
  "ProgramIdentifier": "WLTIC-TITLE-UW",
  "SubmissionIdentifier": "SYN-003",
  "OverallMatchStatus": "MATCH - Conditional",
  "ScoringEligibility": "Permitted",
  "HardConstraintsResult": {
    "Match": true,
    "HardConstraints": [
      {
        "RuleName": "WLTIC-HC-POA-001",
        "RuleDescription": "A power of attorney used must be valid",
        "Status": "INDETERMINATE",
        "Narrative": "poa_used = true; poa_revoked and poa_recorded not supplied.",
        "SourceReference": "p.120, Powers of Attorney: 'The POA must not have been revoked'"
      }
    ]
  },
  "SoftConstraints": [],
  "FailureExplanations": [],
  "MissingInformation": ["WLTIC-HC-POA-001: poa_revoked, poa_recorded"],
  "ConditionalQualifications": ["WLTIC-HC-POA-001: revocation and recording of the power of attorney are unconfirmed"],
  "ReferralIndicators": [
    {
      "RuleName": "WLTIC-RF-POA-001",
      "Narrative": "poa_first_presented_at_closing = true.",
      "SourceReference": "p.121, Powers of Attorney: 'Verification of its authenticity is absolutely required.'"
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
- [ ] No placeholder or default was treated as evidence, and no absent facet was assumed from silence.
- [ ] Referral indicators have no status and did not change the overall result.
- [ ] `Match` is true only for Confirmed and Conditional, and `FAIL` appears only when `Match` is false.
- [ ] No score, points, weight or band appears in the output.
- [ ] Output keys match the format exactly.
