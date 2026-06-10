---
name: clinical-eligibility-review
description: Use when running an auditable clinical trial inclusion/exclusion eligibility review workflow from protocols, EDC listings, raw source documents, OCR/VL outputs, manual trackers, and subject-level HTML/Excel reports.
---

# Clinical Eligibility Review

## Purpose

Use this skill to support clinical trial medical monitoring for inclusion/exclusion eligibility review. The workflow is evidence-bound: every rule interpretation, subject conclusion, and follow-up item must trace back to the protocol, EDC listing, raw source document, OCR/VL output, manual tracker, or audit ledger.

This skill assists clinical monitors. It does not replace investigator judgment, sponsor medical review, GCP responsibilities, source-data verification, or required human sign-off.

## Non-Negotiables

- Do not modify, move, rename, or clean source protocol files, EDC listings, raw subject documents, or manual trackers.
- Keep all generated artifacts in a derived workspace.
- If one protocol includes multiple phases or cohorts, require explicit project configuration for the phase/cohort to review. Do not silently choose.
- Treat OCR/VL text as imperfect. The reviewer model may infer likely OCR typos from context, but critical values, dates, units, positive/negative status, diagnoses, medications, and eligibility facts require conservative handling and visual/source verification when unclear.
- Prefer primary raw source evidence: signed source records, screening/baseline medical records, laboratory reports, examination reports, imaging reports, pulmonary function reports, and other original clinical records.
- Treat EDC listings, email, messaging screenshots, and communication records as secondary or processed evidence. A passed rule supported only by those sources must be marked as requiring verification.
- Never use aggregate EDC eligibility judgment fields, such as `IEYN` / "meets all inclusion criteria and no exclusion criteria", as rule-level evidence. If this is the only source for a rule, mark that rule evidence-insufficient in both the HTML report and Excel ledger.
- Treat broad candidate evidence as only a retrieval hint. Before displaying or highlighting evidence, confirm that the source text contains the current criterion's required clinical fact. If it only contains adjacent-criterion wording, general disease context, visit instructions, scoring reminders, or procedural text, suppress it and use a relevant EDC/table fallback or mark the criterion insufficient.
- Do not display internal evidence IDs, model names, debug logs, audit events, or machine enum keys in reviewer-facing HTML.
- Export operational logs and audit trails to Excel/structured ledger outputs. The HTML report is a focused clinical review artifact, not a log viewer.
- Use natural clinical language. Translate internal fields and flags before they appear in reports.
- The final decision remains a human medical-monitoring decision.

## Recommended Workflow

1. **Project Setup**
   - Record the protocol path, selected phase/cohort, raw source root, optional EDC listing, optional manual tracker, OCR/VL model route, review model route, privacy mode, and output workspace.
   - Decide whether subjects without raw source folders are allowed. The safe default is no: do not include raw-missing subjects as EDC-only review cases unless project configuration explicitly permits it.

2. **Protocol Deconstruction**
   - Extract every inclusion and exclusion criterion for the configured phase/cohort.
   - Create a structured rule for each criterion with:
     - rule ID;
     - short clinical title for display;
     - full protocol criterion text;
     - required evidence fields;
     - timing window;
     - thresholds, units, and allowed retest/rescreen logic;
     - applicability conditions such as sex, age, disease subtype, phase, or cohort;
     - primary and secondary evidence expectations;
     - missing-material prompts.
   - Present the rules to the user for confirmation before subject review.

3. **EDC Listing Extraction**
   - Inventory all sheets/tables.
   - Extract only eligibility-relevant data such as demographics, informed consent, medical history, concomitant medication, allergy history, scores, labs, virology, pregnancy, procedures, randomization, and adverse/relevant clinical findings.
   - Keep EDC evidence traceable to sheet/table names and rows internally. In HTML reports, cite EDC at sheet/table level such as `EDC Listing: LB_CHEM`.

4. **Raw Source OCR/VL And Evidence Build**
   - Inventory subject folders using the screening number as the unique subject identifier.
   - Run OCR/VL on PDFs, images, scanned documents, handwritten scans, photos, and Word-derived pages as needed.
   - Parallel OCR may be used when the local model and hardware can handle it. Cap parallelism conservatively; 1 to 8 workers is a practical range for many local deployments.
   - Build subject-level Markdown/JSON evidence bundles with original filenames, page numbers, OCR quality flags, source text, and candidate rule links.

5. **Formal Rule Review**
   - Review one subject/rule pilot before bulk review.
   - For each subject-rule, produce structured output:
     - verdict: pass, fail, insufficient, conflict, needs investigator review, not applicable;
     - natural-language decision basis;
     - evidence references with exact source quotes;
     - missing materials;
     - risk flags in mapped clinical language;
     - source consistency classification.
   - If the skill runs inside an Agent, the Agent may use its own model to perform formal review from prepared rule packages and then write schema-valid results.
   - Remote API review must be gated by explicit privacy configuration and credentials.

6. **Aggregation And QC**
   - Aggregate rule results into a subject-level conclusion.
   - Compare against any manual IE tracker and classify disagreements.
   - Record review time, versions, model route, privacy mode, result changes, supplemental data, reruns, and triggers in an audit ledger.

7. **Reviewer Report And Excel Ledger**
   - Generate a Chinese-first or site-language reviewer HTML report with one subject visible at a time.
   - Generate an Excel ledger with subject summary, rule detail, operational audit events, manual comparison, and supplement-needed lists.
   - Rebuild the report and ledger after supplemental evidence or reruns.

## Report Contract

The HTML report must help a monitor see the answer quickly.

Each subject page should show only:

- screening number;
- site/center code and name;
- age;
- sex;
- informed-consent signature date;
- overall review result;
- priority warnings;
- rule-by-rule review cards.

Do not include a separate rule overview section if it duplicates the rule cards.

### Left Navigation

- Use dropdown filters for center/site and screening number.
- Use multi-select status filters.
- Sort center options by center code.
- Sort subject options and default subject list by screening number.
- Use serious card-like status styling for the subject switcher: a fixed or sticky sidebar, light subject cards, a left status stripe, restrained clinical colors, and a project/brand accent only for focus or status cues. The sidebar should remain in place while the subject report scrolls independently.
- Recommended status labels:
  - not eligible;
  - evidence insufficient;
  - evidence conflict;
  - eligible, verification required;
  - eligible.

Visual emphasis should follow clinical priority:

1. not eligible;
2. evidence insufficient or evidence conflict;
3. eligible, verification required;
4. eligible.

### Rule Cards

- The collapsed header shows rule ID plus a short clinical title.
- The expanded card includes the full original protocol criterion text.
- Failed, insufficient, conflict, investigator-review, EDC-only, email-only, and EDC-plus-email-only passed rules expand by default.
- Ordinary passed rules collapse by default.
- Every rule includes a brief natural-language explanation of why it passes, fails, is insufficient, or needs verification.

### Evidence Display

- Raw documents display as `filename, page N`.
- If a raw file is actually email or communication evidence, append a visible marker such as `(email/communication record)`.
- EDC displays at table/sheet level, for example `EDC Listing: LB_CHEM`.
- Evidence text must be original source text, not rewritten.
- Highlight the exact sentence or phrase that supports or challenges the criterion.
- Do not highlight an entire evidence block as a fallback. If the review quote contains ellipses, OCR line breaks, or normalized wording, locate and highlight the matching original source phrase.
- Do not truncate a clinical fact because OCR inserted a single line wrap inside a sentence. Preserve complete facts such as history duration, drug names, surgery clauses, and biologic/monoclonal-antibody names.
- Evidence relevance must be checked against the current criterion before display. For score thresholds, show actual score values, components, totals, dates, or visits, not diary instructions or scoring reminders. For diagnosis/history criteria, show the diagnosis/history and required supporting test facts, not generic disease wording. For treatment-response criteria, show treatment exposure and response/ineffectiveness or conflict text, not unrelated history. For time-window exclusion criteria, show the complete clause for that window and do not borrow text from an adjacent criterion.
- Multi-component criteria require visible evidence for each required component. For allergic-rhinitis diagnosis/history rules, display both disease diagnosis/history duration and the supporting positive allergen/sIgE result. Do not treat allergen interpretation legends or concentration-grade tables as result evidence.
- Laboratory/infection exclusion rules must cite actual lab or virology rows, not adjacent allergy-report interpretation legends.
- The visible decision basis must match the visible evidence list. If the report hides redundant EDC because primary raw evidence is available, do not leave wording that says the conclusion relies on EDC.
- For EDC evidence, remove listing metadata before display, such as project code, form code, subject ID, initials, site code/name, row number, and last modified time. Keep non-empty clinical fields such as result, medication, reason, date, assessment, and relevant yes/no values.
- If primary raw evidence and EDC support the same fact consistently, show primary raw evidence in HTML and omit redundant EDC evidence.
- If a pass depends only on EDC, only on email/communication records, or only on EDC plus email/communication records, mark it as verification required.

## Verification-Required Rules

Do not call these ordinary passes in the reviewer UI:

- EDC-only support;
- email/communication-only support;
- EDC plus email/communication support without primary raw source;
- any source where OCR quality flags affect a critical fact;
- any source with unresolved document-subject identity mismatch.

Recommended labels:

- `Pass (verification required: EDC only)`
- `Pass (verification required: email/communication only)`
- `Pass (verification required: EDC plus email/communication only)`

Use the local report language for final rendering.

## Excel Ledger

At minimum, the Excel ledger should include:

- subject summary;
- rule detail;
- run/audit ledger;
- manual comparison;
- supplement-needed list.

The ledger should retain operational metadata that is inappropriate for the HTML report, including model route, OCR model, privacy mode, run triggers, and rerun events.

The ledger and HTML report must share the same effective verdict logic. If a rule is downgraded because only an aggregate IE/IEYN field is present, the subject summary, rule detail, and supplement-needed list must all show evidence-insufficient rather than a pass.

## QC Checklist

Before declaring a run complete:

- All selected protocol criteria are present with full protocol text.
- The configured phase/cohort is explicit.
- Each subject has a unique screening number.
- Subjects without raw source are excluded unless explicitly allowed by project config.
- OCR/VL quality flags are reviewed for critical facts.
- No visible internal IDs, machine enums, model names, or debug logs appear in HTML.
- Evidence labels use filename/page or EDC sheet/table labels.
- Key evidence phrases are precisely highlighted without whole-block fallback.
- Displayed evidence is relevant to the current criterion, not merely a broad candidate match.
- Score criteria do not cite scoring reminders or diary instructions as proof of threshold compliance.
- Aggregate EDC eligibility fields such as IE/IEYN are absent from visible evidence and cannot support rule-level passes.
- Multi-component criteria visibly cover all required components, and allergen interpretation legends are not shown as allergen-result evidence.
- Lab/infection rules cite actual lab/infection rows and do not show allergy interpretation legends.
- EDC evidence omits administrative listing metadata.
- The left sidebar remains fixed or sticky while the report panel scrolls.
- Secondary-only passes are marked verification required at rule and subject level.
- Failed, insufficient, conflicting, and verification-required rules are expanded by default.
- Passed rules with primary evidence are collapsed by default.
- The Excel ledger is generated and opens successfully.
- A human reviewer has checked the final report before operational use.
