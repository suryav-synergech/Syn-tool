I want you to create a 'score' agent - a skill conformant to https://agentskills.io/specification - to determine the degree to which a submission matches an underwriting guideline, quantified as a score. This focuses on measuring only the strength of the match: hard-constraint eligibility is decided upstream and arrives in the input submission json.

## Input - the guideline document is the only required input

The person running this prompt uploads or names **one underwriting guideline document**. That document is the only source you need.

1. If no guideline document was supplied, ask for it before doing anything else.
2. Take the **program name** from the document (title, cover page or header). If it names no program or several, ask which one.
3. Do not ask for, search for, or depend on any of the following: an iteration folder, a program cross-reference workbook, separate override files (`*override.md`, `general.overrides.md`), an evolution log, a field-definitions skill, a linter script, or a `CLAUDE.md`. None of these is part of this flow.
4. A companion `-match` skill is **optional**. If a match skill for the same program already exists in the destination `skills/` folder, read it and use its rule IDs (see "Constraints rule group"). If none exists, do not invent one and do not block: build the skill from the guideline alone and record "no companion match skill" in the critique report.
5. Treat any amendment, addendum, revision note or override that appears **inside** the guideline document as part of the guideline. It governs on the points it changes; the rest of the guideline stands.
6. If the person running this prompt names a document to exclude, never read or cite it, and record the exclusion in the critique report.

If the document is unreadable, missing, or covers a different program than requested, stop and report. Never substitute another program or fill gaps from general insurance knowledge.

## Source-document ingestion - read image-based content, never skip or guess it

The guideline may be a PDF or DOCX whose content (ineligible-class lists, tables, matrices, maps, etc.) is embedded as scanned pages or images with no text layer. Read the whole document using a text-first, image-fallback order:

1. Attempt plain text extraction on each page first.
2. Escalate a page to image rendering and visual reading only when its text layer is missing or partial, or it contains image-based or non-selectable content. Some extracted text does not prove a page is fully captured.
3. For DOCX, read both paragraphs and table cells.
4. If content still cannot be read, do not invent it. Record it as a data gap in the critique report. Unread content must never appear as if it were sourced.
5. Where colour, fill, strikethrough, bold or a legend carries meaning, recover it. If it cannot be recovered, treat the provision as unreadable and record a data gap.

State in the critique report whether the document was read from a text layer or from page images, because that limits how far quotes and formatting can be verified.

## Skill naming and metadata

- Slug: the slugified program name from the guideline with the suffix `-score` (for example `texas-fair-plan-poa-score`). Lowercase `a-z0-9` joined by single hyphens, at most 64 characters. The folder name, frontmatter `name` and slug must be identical.
- Frontmatter top-level keys: only `name`, `description`, `license`, `compatibility`, `metadata`, `allowed-tools`, `version`.
- `metadata` is a map of string to string. Never emit a list or nested map as a value. A program with several source documents is one quoted string joined with `; `.
- Required `metadata` keys: `display_name`, `program_name`, `source_guideline`, `version` (a quoted dotted string such as `"1.0"`).
- Add `companion_match_skill` only if a companion match skill exists. Add `dropdown_value` and `coverage` only if the guideline states them. Never invent values for them.
- `description` is 1-1024 characters, states the program, that this is the **Score** skill, that it measures match strength and does NOT decide match vs not-match, and when to use it.

## Skill content

Final score is also known as field `Score`. `final_score` is in the range 0 to 100, both inclusive.

Rule groups are made up of one or more rules. Each rule group is given a percentage weightage based on how important it appears for the score. When the guideline is ambiguous or states no preference, give equal weightage to rule groups. Rule group percentages total exactly 100%. A weighted sum applies these weights to each rule group level score.

The rule group level score is determined by its rules. Each rule corresponds to a distinct facet scoreable from the guideline. Rule categories: boolean, numeric, tiered-numeric and fuzzy. Each rule scores between 0 and 100 inclusive, and each rule within a group has a percentage weightage; rule percentages within a group total exactly 100%.

Referral triggers do not impact the score at all.

**Conflicting dispositions - referral governs (scoring):** when a factor appears in both a decline / ineligible section and a referral section of the guideline, treat it as a referral trigger for scoring: score-neutral, never a penalty or lift.

**What is sourced and what is designed.** Thresholds, dollar amounts, year counts, limits and class references must be verbatim from the guideline. The 0-100 scores assigned to branches and the rule and group weights are usually **your design**, because guidelines rarely state them. Therefore:
- If the guideline states a weight, score, tier or band, use it and cite it.
- If it does not, you may assign scores and weights as the prompt allows (equal weights by default), but label each such value **designed, not sourced** in the critique report and list every one as an open item for the person running this prompt to calibrate.
- Never present a designed value as if the guideline stated it.
- Do not create a rule from a provision that gives no suitability signal. Many provisions describe pricing, workflow, binding, cancellation, renewal or eligibility, not suitability. List them as "reviewed, not scored".

## Deductible, SIR and premium are OUT OF SCOPE

Do NOT create any rule or rule group scored on deductible, SIR (Self-Insured Retention), or premium - including rate, pricing, minimum premium, payment plans, policy fees or any rate- or price-derived measure. These must not move `final_score` in any direction. Ignore them entirely for scoring. No override mechanism exists in this flow to change that.

## Rule categories and assigned scores

Scores are within 0 to 100 inclusive. Examples:

- **Rule R1 - boolean**, true -> 0, false -> 80.
- **Rule R2 - numeric**, `#floors > 200` -> 0, else -> 100.
- **Rule R3 - tiered-numeric**, `X < $1M and Y = true` -> 10, `X < $1M and Y = false` -> 50, `X >= $1M` -> 100. Compound conditions must cover every combination of values exhaustively.
- **Rule R4 - fuzzy**: state the floor as 0 and the ceiling as 100, and give scoring criteria an LLM judge can apply.

Example of a fuzzy rule:

Designated Projects

| Condition | Score Range |
|---|---|
| Owner-only cover with proper contractual chain (GC required to carry CGL + Umbrella, hold harmless, AI status) | 90-95 |
| Owner + GC combined cover with executed contracts | 85-90 |
| GC-only project cover with subcontractor contracts reviewed | 80-85 |
| Subcontractor designated project (contained limit, limited completed ops exposure) | 75-80 |
| Project policy requested but contractual structure unclear or not described | 55-65 |

## Worked out example to make this concrete

Roll-ups use a weighted sum. Rule weights within a rule group total 100%.

RG1: R1 -> 60%, R2 -> 40%; R1 -> 80, R2 -> 100.
RG1's score -> (80*60% + 100*40%) = 88.
RG2's score -> 40.

Every rule group has a weight; weights total 100% across all groups. Final score = weighted sum of all rule-group scores.

RG1 -> weight 10%; RG2 -> weight 90%.

final score = (10% * 88 + 90% * 40) = 44.8

Organize the rules into sections, within rule group sections in the skill.

## Hard-constraint handoff - hard constraints are INPUT-OWNED and are never recomputed here

Hard-constraint eligibility is decided upstream. The input submission json carries a `HardConstraintsResult` object with `Match = true` and a `HardConstraints` array. Use those entries and their narratives exactly as given. Never re-evaluate, re-derive or recompute a hard constraint.

`Status` arrives as **`PASS` or `INDETERMINATE` only**. `FAIL` cannot occur, so do not author a FAIL branch, a FAIL-tier score, or any rule, narrative or example that handles one. Do not author any step, logic or example for hard-constraint evaluation, for a `Match = false` outcome, or for match-gated routing.

If no `HardConstraintsResult` field definition or sample is supplied, use the shape shown in the example below and record "input contract assumed from the prompt" as an open item.

## Constraints rule group

Have a separate `Constraints` rule group, with each decision factor becoming a rule with a score and a weightage.

Build its rule list from the guideline's eligibility and requirement provisions, and from the companion `-match` skill if one exists. Score an `INDETERMINATE` hard constraint lower than a confirmed `PASS`, for example with a boolean rule. Each soft constraint also maps to a rule with a proportional weight. A referral-governed soft constraint is NOT mapped to a scoreable rule; treat it as score-neutral.

**No double-counting.** A facet is "scored elsewhere" if another rule group has a rule keyed off the same submission input. Map a constraint to a Constraints rule only if no other group scores that facet. If another group does, omit it here; that group's unknown / missing-data tier already applies the data-gap penalty.

**"Maps to (match-skill rule)" column.**
- If a companion `-match` skill exists, populate the column ONLY with rule IDs that appear verbatim in its `SKILL.md` (the match skill is a single file; all its constraints are in it). Open it and extract the exact ID inventory before writing the column.
- Never invent, abbreviate, reformat or aggregate an ID. If one rule covers several match rules, list every real ID, comma-separated.
- If no companion match skill exists, write `n/a - no companion match skill` in the column and cite the guideline provision instead.

## Score computation and output fields

Compute `final_score` as a two-level weighted sum. The breakdown is emitted as `ScoreCalculationDetails`:

- Per rule group: `sum(rule_score x rule_weight%) = rule_group_score`. Rule weights within a group sum to 100%.
- Roll-up: `sum(rule_group_score x rule_group_weight%)`, rounded to the nearest integer to give `Score`. Rule-group weights sum to 100%.

`ScoreCalculationDetails` is an array of strings:
- One string per rule group expanding `sum(rule_score x rule_weight%) = rule_group_score`.
- A final string expanding `sum(rule_group_score x rule_group_weight%)` showing the pre-rounding value and the rounded `final_score`, which MUST equal `Score`.
- **ASCII only**: `*` for multiplication and `->` for the arrow. Example: `"final_score = (59.5*24%) + (52.75*18%) + ... = 58.99 -> 59"`.
- Always non-empty.

In the skill's "Output format" section, document the field with the placeholder `"<RG-ID> = (<rule_score>*<rule_weight%>) + ... = <rule_group_score>"` and the final-line form above.

`ScoreNarrative` is mandatory. The skill's "Output format" section must say: "`ScoreNarrative` - array of strings, **MANDATORY when Match=true**: omitting this field or emitting it null/empty is a critical error. MUST contain exactly three entries in order: a string starting with "Lifters:", a string starting with "Drags:", and a string starting with "Final:" stating the score as "<final_score>/100" (the number MUST equal `Score`)." Use the placeholders `"Lifters: <top group/rule contributions that raised the score, each with its sub-score>."`, `"Drags: <top group/rule contributions that lowered the score, each with its sub-score>."` and `"Final: <score>/100 <fit band>. <one-line rationale tying the score to signal vs data gaps>."`.

Emit fields in this order: **`ScoreNarrative`, `ScoreCalculationDetails`, `Score`.** The Output format sentence must read: "Return a single JSON object with fields in this order - `ScoreNarrative`, `ScoreCalculationDetails`, `Score`."

Fit bands named in the narrative must come from the guideline if it states them. Otherwise name no numeric band edges; describe fit qualitatively and record "no sourced bands" as an open item.

Example:

```json
{
  "ScoreNarrative": [
    "Lifters: Territory (92 -- 100% NYC, PRIME core), Financial (89 -- complete 3-year data, moderate size), Business stability (85 -- 25 years, consistent operations).",
    "Drags: Loss History (45 -- no loss data, only 1 favorable proxy characteristic), Coverage Structure (61 -- all coverage fields null), EIFS latent exposure risk.",
    "Final: 61/100 CONDITIONAL FIT. Masonry in NYC is a target class with 25 years stability, no disqualifying characteristics. Score held back by data gaps not by adverse signals."
  ],
  "ScoreCalculationDetails": [
    "RG-TERR = (100*60%) + (80*40%) = 92",
    "RG-FIN = (95*60%) + (80*40%) = 89",
    "RG-STAB = (90*50%) + (80*50%) = 85",
    "RG-LOSS = (40*50%) + (50*50%) = 45",
    "RG-COV = (55*40%) + (65*60%) = 61",
    "final_score = (92*10%) + (89*10%) + (85*10%) + (45*50%) + (61*20%) = 61.3 -> 61"
  ],
  "Score": 61
}
```

Example of the `HardConstraints` input (arrives as INPUT; `Status` is `PASS` or `INDETERMINATE` only; consumed without recomputation):

```json
"HardConstraints": [
  {
    "RuleName": "PS-HC-ELIG-001",
    "RuleDescription": "Operations must not include airport construction or servicing",
    "Status": "PASS",
    "Narrative": "OperationsIncludeGroupAirport = false. No airport operations.",
    "SourceReference": "<guideline section and short quote>"
  }
]
```

## Precedence when sources conflict - name the step used for every decision

1. This prompt's structural guards (deductible, SIR and premium out of scope; referral neutrality; weights sum to 100%; the output contract).
2. The guideline document, on **substance**: what is scored, thresholds, tiers, bands, and any weight it states outright. An amendment or override inside the document governs on the points it changes.
3. Your designed scores and weights, only where steps 1-2 are silent, each labelled "designed, not sourced".

If a skill already exists at the destination, read it first. Keep its weights unless the guideline now states a different one or a rule group loses its whole basis. Record any other weight change you consider as an open item instead of making it.

## Format conformance - verified by hand

No linter script is part of this flow. Author to the rules below and verify each by hand in check K. If `skills-ref validate` happens to be available, run it and report the verbatim result. If not, say so explicitly. Never claim a clean lint you did not run.

**Body headings.** Exactly one `#` H1 immediately after the frontmatter. Required `##` sections (matched as case-insensitive substrings): `how to use this skill`, `rule groups and weights`, `final score`, `output format`, `worked example`. Keep `SKILL.md` at or under 500 lines. If detail would push it over, you may move it into `references/`, one level deep, with relative links that resolve. (The Match skill is single-file; this allowance applies to the Score skill only.)

**Rule-group headings and arithmetic.**
- Every rule group has a heading `### RG-<TOKEN> <group name> (weight <N>%)`. The declared weights sum to exactly 100%.
- The `## Final score` section states one term per rule group as `RG-<TOKEN> x 0.NN`, where the factor equals the heading weight / 100.
- `Score` in every example is an integer in 0..100, never a float, string or un-rounded sum.

**Output block and examples.**
- `## Output format` holds exactly one fenced `json` block that parses, and every fenced json block in the file parses.
- Every key in an example is declared in the block, including as an empty container when there is nothing to report. A key declared in the block but missing from an example is a defect, unless its placeholder begins with `Optional`.
- Container and scalar types agree between block and examples. A placeholder written `A | B | C` is an enum and every example value is one of its alternatives. Declare `"Status": "PASS | INDETERMINATE"`; never list or use `FAIL`.
- Any `HardConstraints` entry reproduced in an example carries a `SourceReference`.

**One definition marker per rule ID.** In tables, a cell holding only a backticked ID counts as a definition, and so does an arrow followed by a backticked ID. Only one row may carry a given ID as a bare ID-only cell. Prefix every further citation with `see `. Keep distinct guideline provisions as separate rows, never merged. In prose write IDs in plain backticks or prefixed with `see `, never introduced by an arrow.

**IDs.** Match-skill IDs that you cite keep the pattern they have in the companion skill. This skill's own `R-...` and `RG-...` identifiers are a separate namespace.

## Adversarial review - validate before returning

After producing the skill, switch to an adversarial reviewer role and validate it against the checks below **before returning it**. Treat every rule, weight and threshold as unproven until traced to the guideline or marked designed. When a check fails, fix the skill and re-run the whole checklist from the top. Do not return a skill that has not passed a clean run.

**A. Weight arithmetic**
1. Within each rule group, rule weights sum to exactly 100%.
2. Across rule groups, weights sum to exactly 100%.
3. Where the guideline expresses no preference, weights are equal. Any skewed weight must be justified by explicit guideline emphasis.

**B. Rule integrity - for each rule**
4. Every score any branch can produce is within 0-100 inclusive.
5. Tiered-numeric rules cover value combinations exhaustively and include an `unknown / not provided` branch. Boolean rules define both branches. Fuzzy rules state floor 0 and ceiling 100 with criteria an LLM judge can apply.
6. Every threshold, dollar amount, year count, limit and class reference is verbatim from the guideline (or the companion match skill). An untraceable number is corrected or the rule is deleted.
7. No prose states how many rules exist (hard constraints, soft constraints, groups, rules, collection members). Refer to match rules by ID, never by position, and avoid guideline item numbers.

**C. Referral neutrality.** Inventory every referral or discretionary-action trigger in the guideline. None acts as a score driver. A value in a referral band is treated as neutral or unknown, never a penalty or lift. Every referral-governed constraint is score-neutral. No rule is scored on deductible, SIR or premium.

**D. Missing-data handling.** No missing, blank or null input scores 0 or the rule minimum by default. Every `unknown` branch scores below confirmed-favorable but well above confirmed adverse. An `INDETERMINATE` hard constraint scores lower than a confirmed `PASS`. Placeholder and default values are not evidence.

**E. No double-counting.** Build the facet -> owning-rule-group map. Every facet is scored by exactly one group. The Constraints group holds only facets no other group scores. Every match-skill constraint, where one exists, is either a Constraints rule or explicitly delegated to a named owning group.

**F. Match/score boundary.** The skill never re-decides the match. No rule or narrative emits `Match` true or false. No rule defines a FAIL branch or FAIL-tier score, and no narrative or example mentions one.

**G. Schema and mechanics.** Fields appear in the order `ScoreNarrative`, `ScoreCalculationDetails`, `Score`. `ScoreNarrative` is a non-empty array of exactly three strings starting "Lifters:", "Drags:" and "Final:", and the Final line's number equals `Score`. The Output format section says omitting it or emitting it null/empty is a critical error. `ScoreCalculationDetails` has one line per rule group plus a final line, all ASCII. No `Risks` field is emitted. **Recompute every expression by hand.** Each product, group score and the final value must be arithmetically exact, and the rounded value equals `Score` within [0, 100].

**H. Behavioral self-tests.** Hand-trace three synthetic submissions through every rule, with the full two-level computation:
- (i) a complete, on-target, clean submission -> strong-fit result
- (ii) an eligible but sparse submission (data gaps only) -> mid-range score whose narrative attributes the drag to data gaps
- (iii) a referral-band twin of (ii) with exactly one field set into a referral band -> `final_score` is identical to (ii) with that field unknown. If it moves, a referral trigger is leaking into scoring; fix the rule.

**I. Cross-reference integrity (only if a companion match skill exists).** Open its `SKILL.md`, extract the exact ID inventory, and build this proof table with one row per "Maps to" ID:

| Score rule | "Maps to" ID | Found verbatim in companion `-match`? | Match-skill location |
|---|---|---|---|

Any ID not found verbatim makes the skill INVALID. Replace it with real IDs and re-run from check A. Also confirm every match constraint is either mapped or delegated per check E. If no companion match skill exists, write "N/A - no companion match skill".

**J. Frontmatter limits.** Measure the folded `description` length (a `>-` block scalar joins lines with single spaces) and keep it at or under about 1000 of the 1024 cap. Confirm `name` is at most 64 characters, equals the folder name and has no consecutive hyphens. Re-measure after every frontmatter edit, and report the counts even when they pass.

**K. Format-conformance evidence, verified by hand**
1. Metadata values are all scalars; `version` is a quoted dotted string.
2. One H1 and all five required H2 sections.
3. Every rule group has an `RG-<TOKEN> (weight N%)` heading, the weights sum to 100%, and `## Final score` has one `RG-<TOKEN> x 0.NN` term per group that matches. Every example `Score` is an integer in 0..100.
4. ID -> ID-only-cell count across every table; no ID appears bare in two rows, and no provisions were merged to clear that.
5. Key-set parity between the Output format block and every worked example, in both directions.
6. Run `skills-ref validate ./<slug>` if available and paste the result; otherwise state it could not be run.

## Critique report (build-time artifact - NOT part of the skill)

Write the report to `skills-test/<slug>/CRITIQUE.md` (create the folder if needed). It is never read at runtime and must not appear in the skill's output. Include:

1. The weight-sum table per group and across groups (check A).
2. A per-rule integrity table, with the guideline trace for every numeric threshold and a column marking each score and weight as **sourced** or **designed, not sourced** (check B).
3. The referral-trigger inventory and where each is neutralized (check C).
4. The facet -> owning-group map (check E).
5. The three self-test submissions with the fully expanded two-level computation and traced outcomes, including the referral-twin equality result (check H).
6. The cross-reference proof table (check I), or "N/A - no companion match skill".
7. Amendments, addenda or overrides found inside the guideline, with the superseded text and how the rules reflect them, or "none found".
8. The frontmatter limit table (check J) with measured counts against 1024 and 64.
9. The format-conformance section (check K) and a statement of which checks were hand-verified and whether `skills-ref` was run.
10. The source inventory: the guideline read, how it was read (text layer or page images), any exclusion requested, any content that could not be read, and any provision reviewed but not scored with the reason.
11. Open items for the user, including: every designed score and weight to calibrate, absent sourced fit bands, the assumed input contract, and any weight change considered but not made.

## Output placement

Place the created skill in `skills/<slug>/`, where `<slug>` is the skill name. If the person running this prompt names a different destination, use it. Place ONLY the critique report under `skills-test/<slug>/`; do not place any test or critique artifacts inside the `skills/` tree.
