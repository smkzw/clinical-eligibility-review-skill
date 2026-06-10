# Clinical Eligibility Review Skill

An auditable Agent skill for clinical trial inclusion/exclusion eligibility review.

This skill helps medical monitors turn a study protocol, EDC listing, raw source documents, OCR/VL text, and manual eligibility trackers into a traceable eligibility review workflow. It is designed for clinical operations and medical monitoring teams, not for software engineers only.

## What This Skill Does

The skill guides an Agent through a full eligibility-review process:

1. Read and deconstruct protocol inclusion/exclusion criteria.
2. Ask for explicit phase/cohort confirmation when a protocol contains more than one phase.
3. Convert each criterion into a review rule with the full protocol text, required evidence, timing windows, thresholds, exceptions, and missing-material checks.
4. Extract eligibility-related EDC data from listings.
5. Process raw subject files through OCR/VL when needed.
6. Build subject-level evidence bundles.
7. Review each subject against each rule.
8. Generate a clinical HTML report and an Excel ledger.
9. Support supplemental evidence and reruns while preserving audit history.

## Who It Is For

This skill is intended for:

- clinical trial medical monitors;
- clinical research associates;
- sponsor clinical operations teams;
- quality and inspection-readiness teams;
- clinical data reviewers who need source-grounded eligibility checks.

No AI background is required to understand the expected workflow. The Agent handles the technical steps, while the human reviewer remains responsible for confirmation and final judgment.

## Important Safety Boundary

This skill does not replace:

- investigator judgment;
- medical monitoring sign-off;
- source data verification;
- GCP/ICH responsibilities;
- sponsor SOP requirements;
- regulatory decision-making.

The skill helps organize evidence and highlight risks. Final acceptance must remain a human clinical decision.

## Inputs

Typical inputs include:

- study protocol, usually Word or PDF;
- EDC data listing, usually Excel;
- raw subject source documents, such as medical records, lab reports, examination reports, images, scanned PDFs, signed forms, and visit notes;
- manual inclusion/exclusion tracker, if available;
- optional supplemental evidence for reruns.

## Outputs

Recommended outputs:

- structured eligibility rules;
- subject-level evidence bundles;
- rule-level review results;
- subject-level review results;
- HTML reviewer report;
- Excel review ledger;
- audit ledger for runs, reruns, and supplemental evidence.

## HTML Report Principles

The HTML report should be a clinical review artifact, not a technical log.

Each subject page should show:

- screening number;
- center/site code and name;
- age;
- sex;
- informed-consent date;
- overall conclusion;
- priority warnings;
- rule-by-rule review cards.

The report should not display model names, raw audit logs, internal evidence IDs, machine enum values, or developer instructions.

## Evidence Hierarchy

Primary raw sources are preferred:

- screening/baseline medical records;
- signed source documents;
- laboratory reports;
- examination and imaging reports;
- pulmonary function reports;
- other original clinical records.

Secondary or processed sources need verification:

- EDC listing only;
- email or communication record only;
- EDC plus email/communication without primary raw source;
- OCR output with critical quality concerns.

If a rule passes only on secondary evidence, the report should say that it passes but requires verification.

An aggregate EDC eligibility judgment, such as an IE/IEYN field saying that all inclusion and no exclusion criteria are met, is not evidence for any individual rule. If this is the only source for a rule, that rule should be treated as evidence-insufficient, not as a pass.

Except for explicitly configured "not triggered by absence" exclusion rules, a rule should not appear as a normal pass when no displayable, locatable, current-rule-relevant evidence exists. For example, an investigator-suitability exclusion can default to not triggered when no wording says the subject is unsuitable or should not be enrolled. It should not display randomization, score, laboratory, or aggregate EDC eligibility text as evidence for that rule.

## Rule Display

Each rule should show:

- short clinical title in the collapsed header;
- full original protocol criterion text when expanded;
- review conclusion;
- short clinical explanation;
- exact source quote;
- source location such as file/page or EDC sheet.

Evidence text should keep the original source wording. The most relevant phrase should be highlighted so a reviewer can quickly see why the source was cited.

Highlighting should be precise. If a quote contains ellipses, OCR line breaks, or normalized wording, the report should highlight the matching original source phrase. It should not mark an entire evidence block just because exact matching failed.

Candidate evidence is not automatically display evidence. Before a quote is shown, the report should confirm that the source text actually supports the current criterion's clinical fact. For example:

- score-threshold criteria should cite actual scores, components, totals, dates, or visits, not scoring reminders or diary instructions;
- diagnosis/history criteria should cite the diagnosis, history duration, and required supporting tests, not generic disease wording;
- treatment-response criteria should cite prior treatment exposure and response/ineffectiveness or conflict text, not unrelated history;
- time-window exclusion criteria should cite the complete clause for that window and should not borrow text from an adjacent criterion.

If the raw medical record does not contain the required field for the current criterion, the Agent should actively inspect the corresponding EDC table or mark the criterion insufficient. It should not highlight unrelated raw text just because it was retrieved as a candidate.

Multi-component criteria need evidence for each component. For example, an allergic-rhinitis diagnosis/history rule should show the source text for the diagnosis and history duration, plus the individual positive allergen/sIgE result row. A concentration-grade interpretation table such as "class 6 severe allergy" is not a substitute for the actual allergen result.

Laboratory or infection exclusion rules should cite actual laboratory/virology rows, not an adjacent allergy interpretation legend. OCR line breaks inside a sentence should not cause the report to truncate drug names, history duration, biologic names, or surgery clauses.

Laboratory and pregnancy/lactation evidence needs additional safeguards:

- short Latin lab terms such as `AST` and `ALT` must match as lab tokens, not inside unrelated words such as `Master`, URLs, or mail headers;
- when OCR splits a lab row across lines, the report should reconstruct the analyte-result-unit-reference row instead of showing a bare marker such as `ALT`;
- EDC rows that only say a test was not performed or not needed are not valid evidence for a laboratory exclusion rule;
- TP-Ab, anti-TP, and syphilis serology should not be treated as pregnancy/lactation evidence. Pregnancy rules should rely on HCG/pregnancy/lactation wording or sex applicability.

For EDC evidence, the report should remove administrative listing fields before display, such as project code, form code, subject ID, initials, site code/name, row number, and last modified time. The visible EDC evidence should focus on clinical fields such as result, medication, reason, date, assessment, and relevant yes/no values.

The subject switcher should use serious card-like status styling: a fixed or sticky sidebar, light subject cards, a left status stripe, restrained clinical colors, and a project/brand accent only for small focus or status cues. The sidebar should remain in place while the subject report scrolls.

## Excel Ledger

The Excel ledger should contain operational details that do not belong in the HTML report:

- subject summary;
- rule detail;
- run/audit log;
- manual tracker comparison;
- supplement-needed list.

If the HTML report combines multiple centers, create a combined Excel ledger next to it. It should use the same sheet structure and the same effective verdict logic as the single-center ledger.

## Installation

For Codex or another Agent system that supports local skills, place this repository or the `SKILL.md` file in the Agent's skills directory. For example:

```bash
mkdir -p ~/.codex/skills/clinical-eligibility-review
cp SKILL.md ~/.codex/skills/clinical-eligibility-review/SKILL.md
```

Then ask the Agent to use the `clinical-eligibility-review` skill for a clinical trial eligibility review workflow.

## Example Project Configuration

See `examples/project-config.example.yaml`.

The configuration should make these choices explicit:

- selected phase or cohort;
- privacy mode;
- OCR/VL model route;
- review model route;
- whether raw-missing subjects may be reviewed from EDC only;
- output workspace;
- report language.

## Quality Checklist

Before using a report operationally:

- confirm every inclusion/exclusion criterion is present;
- confirm the selected phase/cohort;
- confirm subject identifiers and source-folder mapping;
- check OCR quality flags for critical facts;
- verify secondary-only passes;
- verify that aggregate IE/IEYN eligibility fields are not used as rule-level evidence;
- verify that highlighted evidence actually supports the current criterion, not just a broad candidate match;
- verify multi-component rules show all required components, not only one part of the criterion;
- verify that score criteria cite real score records rather than scoring reminders or diary instructions;
- verify lab/infection rules cite actual lab rows and do not show allergy interpretation legends;
- verify lab/infection rules do not show report headers, mail headers, bare lab markers, or no-result EDC rows;
- verify pregnancy/lactation rules do not cite unrelated TP/syphilis/serum-antibody rows;
- verify no non-exception rule displays as passed with no locatable evidence;
- confirm failed, insufficient, conflict, and verification-required rules are expanded by default;
- confirm the left sidebar stays fixed or sticky when the report panel scrolls;
- open the Excel ledger;
- perform human medical-monitoring review.

## Privacy Notes

Clinical trial data may include personal data and confidential sponsor data. Do not send trial data to remote models unless this is explicitly approved in the project configuration and allowed by your organization, study agreements, and applicable regulations.

Local OCR/VL and local LLM deployments can reduce data-transfer risk, but they still require access control and audit discipline.

## License

No license is granted unless the repository owner adds one. Contact the owner before commercial redistribution or modification outside personal/internal evaluation.
