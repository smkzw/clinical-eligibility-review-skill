---
name: clinical-eligibility-review
description: Use when building or running an auditable clinical trial eligibility review workflow from protocols, EDC listings, raw source documents, OCR/VL output, manual IE trackers, and subject-level HTML reports.
---

# Clinical Eligibility Review

## Purpose

Use this skill to run a cautious, traceable inclusion/exclusion medical monitoring workflow. Treat every conclusion as source-bound: protocol rules, EDC listing rows, raw-document OCR/VL evidence, review outputs, manual IE baseline, and audit ledger must remain linked.

## Non-Negotiables

- Do not modify or move source protocol, EDC, raw subject files, or manual tracker files.
- Store derived artifacts in the project workspace only.
- If a protocol contains multiple phases, require an explicit configured phase; do not silently choose.
- Preserve the protocol's official criterion numbering and nesting. If numeric-list extraction loses hierarchy, inspect the source PDF/DOCX visually or by page text before finalizing rules. Do not split sub-bullets into new official exclusion numbers.
- OCR/VL text may contain recognition errors. Review logic may interpret likely OCR typos in context, but critical values, dates, units, positive/negative status, diagnoses, medications, and eligibility facts require conservative handling.
- When this skill runs inside an Agent, use the Agent's own model for formal review by reading the prepared rule prompt packages and writing validated `RuleReviewResult` rows. Do not block on external API credentials for Agent-mediated review.
- Do not run remote API-based LLM review unless `privacy_mode=remote_full_allowed` and API credentials are present.
- Start with a subject/rule-limited formal review pilot before any bulk run.
- Record reruns and incremental updates in `audit_ledger.jsonl`.
- The human-facing HTML report must be Chinese, conclusion-first, and clinically scannable. Do not expose model names, raw audit events, debug logs, rule-package metadata, or internal `RAW-*` / `EDC-*` evidence IDs in visible report text.
- Put operational logs and review ledger data in `reports/eligibility_review_ledger.xlsx`, not in the HTML report.
- The HTML report must not include a separate "条目概览" section. It duplicates the rule cards and blurs the review focus.
- Rule headers must use short natural Chinese clinical titles for scanning, while the expanded rule card must also show the full original protocol criterion text under `方案标准`. Do not let shortened titles replace the complete protocol requirement.
- Evidence shown to reviewers must resolve to the original filename plus page, or EDC listing sheet only, with the exact cited original excerpt preserved from the source. Raw document and EDC evidence IDs may remain in JSON/Excel, but must not be visible in HTML.
- Candidate evidence is not automatically reviewer-facing evidence. Before display, validate that the quote contains the current rule's required clinical fact. If a raw citation only contains adjacent-rule wording, general disease context, diary/scoring reminders, or procedure instructions, suppress it and either use a relevant EDC subtable fallback or mark the rule insufficient.
- EDC aggregate eligibility fields such as `是否满足所有的入选标准且不满足任一的排除标准(IEYN)` are not rule-level evidence. Never use them to support an individual inclusion or exclusion criterion. If no other valid source exists for that rule, mark it as evidence-insufficient in both HTML and Excel, not as a verification-required pass.
- A rule should not render as `通过` if it has no displayable, current-rule-relevant evidence. The only exception is an explicitly configured presence-only exclusion rule, such as an investigator-suitability exclusion that defaults to not triggered when no unsuitable wording exists.
- Evidence quotes in HTML must highlight the exact sentence or phrase that supports or challenges the rule. Preserve enough original context around the highlighted text, especially when OCR output is noisy. Do not use whole-block highlighting as a fallback; if the review quote uses ellipses, OCR line breaks, or normalized wording, locate and highlight the matching original source phrase.
- Clinical evidence extraction must tolerate OCR single-line wraps inside a fact. Do not truncate phrases such as history duration, drug names, surgery clauses, or monoclonal-antibody names merely because OCR inserted a newline in the middle of the sentence.
- The visible decision basis must match the visible evidence. If display logic hides redundant EDC because primary raw evidence is available, do not leave wording that says the conclusion relies on EDC.
- EDC evidence in HTML must be trimmed to clinical fields before display. Remove listing metadata such as project code, form code, subject ID, initials, site code/name, row number, and last modified time. Keep non-empty clinical fields such as result, medication, reason, date, assessment, and relevant yes/no values.
- Core eligibility support should come from primary raw sources such as medical records, laboratory reports, examination reports, and signed source records. EDC listings, emails, and communication records are secondary/processed evidence sources. Do not render a generic `仅二手来源` label. If a passed rule is supported only by EDC, only by email/communication records, or by EDC plus email/communication records without primary raw evidence, the subject-level priority warning must visibly say `通过（需验证：仅EDC数据）`, `通过（需验证：仅邮件/沟通记录）`, or `通过（需验证：仅EDC数据及邮件/沟通记录）`. Rule cards and Excel detail rows should mark the result as `通过（需验证）` and keep the specific source type in the adjacent badge/consistency field.
- If an otherwise eligible subject has any `通过（需验证：...）` rule, the left subject switcher and the subject header's overall result must also display `通过（需验证）`, not plain `通过`. Filter metadata should include `需验证` and the concrete source type so reviewers can filter these cases.
- If raw primary source and EDC support the same fact and are consistent, show raw primary evidence in HTML and omit the redundant EDC citation. Email/communication records remain secondary evidence even when stored as raw-document files.
- Subject age and sex should be extracted from raw screening/baseline medical-record headers when available; EDC demographics are a fallback. If both medical-record headers and examination/lab report headers contain age/sex, prefer the screening/baseline medical record and use report headers only as fallback.
- Report text must be natural Chinese. Internal enum names, risk flag keys, machine field names, model labels, row IDs, and debug identifiers must be translated before rendering or the report should fail generation.
- The left subject switcher must use dropdown filters for `研究中心` and `筛选号`, plus multi-select status filters. Do not use a free-text input as the main reviewer filter. Sort center options by center code, and sort subject options/default list by subject ID.
- Status filters should include `不通过`, `证据不足`, `证据冲突`, `通过需验证`, and `通过`. Visual emphasis must follow clinical priority: 不通过 > 证据不足/证据冲突 > 通过需验证 > 通过. `通过（需验证）` must be visible but quieter than evidence-insufficient or conflicting cases.
- The subject switcher should use card-like status styling: fixed/sticky sidebar, light subject cards, a left status stripe, restrained clinical colors, and CMS orange `#FF9900` only as an accent/focus color. The left sidebar must stay fixed while the right subject report scrolls independently. Keep it serious and report-like.

## Current Workspace Commands

Run from the workflow repository with `PYTHONPATH=src`.

```bash
python -m eligibility_review.cli extract-protocol --project <PROJECT>
python -m eligibility_review.cli inventory-edc --project <PROJECT>
python -m eligibility_review.cli inventory-subjects --project <PROJECT>
python -m eligibility_review.cli build-ocr-queue --project <PROJECT>
python -m eligibility_review.cli run-ocr --project <PROJECT> --subject <SUBJECT> --workers 4
python -m eligibility_review.cli build-evidence --project <PROJECT>
python -m eligibility_review.cli draft-review --project <PROJECT>
python -m eligibility_review.cli import-manual-ie --project <PROJECT> --center <CENTER>
python -m eligibility_review.cli compare-manual-ie --project <PROJECT>
python -m eligibility_review.cli check-llm-readiness --project <PROJECT>
python -m eligibility_review.cli build-report --project <PROJECT>
python -m eligibility_review.cli build-combined-report --project <PROJECT_A> --project <PROJECT_B> --project <PROJECT_C>
```

Use `run-ocr --workers N` with `1 <= N <= 8`. Use high worker counts cautiously because local OCR/VL memory is shared.

OCR rerun safety:

- Failed OCR retries must not overwrite an existing successful OCR page result. Preserve the last successful text and log the failed rerun separately in the audit ledger.
- Treat local OCR service connection failures, empty responses, and header-only table extraction as evidence build issues, not as clinical facts. They may justify `证据不足` or `待补OCR/源文件复核`, but never a normal pass.

## Current Delivery Path

Path B is the active center-confirmation delivery path:

1. Finish formal Agent-mediated review for every available subject package.
2. If manual IE contains subjects without raw source folders, do not include them in the review/report as EDC-only cases. Record the source-data gap separately and wait for raw files before OCR/evidence/review.
3. Generate one Chinese HTML report with left-side center and subject dropdown filters, status multi-select filters, and subject switching. For multi-center delivery, reviewed subjects from all centers should be rendered into one combined HTML, not separate final HTML files.
4. Each subject page shows only subject basics, overall conclusion, priority warnings, and rule-by-rule review cards.
5. Keep passed rules collapsed by default. Expand failed, evidence-insufficient, conflict, needs-investigator, EDC-only, email-only, and EDC-plus-email-only rules by default.
6. Generate `reports/eligibility_review_ledger.xlsx` for audit time, result, rerun, manual comparison, rule detail, and supplement-needed tracking. For multi-center delivery, `build-combined-report` should also generate `reports/eligibility_review_ledger_all_centers.xlsx` next to the combined HTML.

Path C is the post-confirmation product direction and should be preserved in future plans:

- reviewer workbench for supplemental uploads and rule/subject reruns;
- incremental and full report refresh;
- explicit project configuration for phase, model route, privacy mode, OCR workers, and whether EDC-only subjects are allowed; default is not allowed unless the user explicitly configures it for a project;
- persistent Excel/structured ledger as the operational source of truth;
- HTML as a focused reviewer report, not a log viewer.

## Formal Review Gate

Before formal review:

1. Confirm `rule_prompt_packages.jsonl` exists for the pilot subject.
2. Run the smallest clinical-data pilot first, usually one subject and one or a few rules.
3. For Agent-mediated review, the Agent should generate schema-valid `RuleReviewResult` rows directly from the rule packages and cited evidence. Use `review_model=agent:<model-name>` in ledger and aggregate output.
4. If using the optional remote API path instead, run `check-llm-readiness`, confirm status is `ready`, and use a harmless non-PHI model availability check or pass a known deployable model with `--model`.

Rule deconstruction must be deeper than copying eligibility text. Before formal review, confirm the following protocol-derived logic is reflected in the rule notes or reviewer instructions:

- Preserve the protocol's official criterion numbering and nesting. If an official criterion is a medication/treatment-history package with subitems, keep those subitems under that official rule. Do not promote subitems into new EX numbers because Word/PDF extraction lost indentation.
- When correcting historical outputs, create a local old-to-new rule-ID mapping and rerun report, ledger, and QC checks. Keep project-specific mappings in local project context, not in a public reusable skill package.
- Rescreen/retest: capture whether the protocol allows rescreening, retesting, a new ICF, a new screening number, and any treatment restrictions before retest.
- Score-threshold rules require actual score evidence: components, totals, dates, visits, calculation windows, and thresholds. Diary-card instructions, scoring reminders, or visit-procedure text are not threshold evidence. If raw notes do not contain score fields, inspect configured EDC score subtables rather than highlighting unrelated raw text.
- Multi-component diagnosis/history rules must separately show diagnosis, required history duration, and required supporting tests. Use prior medical records first; screening/baseline narrative second and mark source verification when needed; EDC third and mark verification required. Aggregate IE/IEYN never counts.
- Treatment-response rules must show prior exposure and response/ineffectiveness, intolerance, or a direct conflict/query. Generic disease history is not enough.
- Biomarker or baseline laboratory inclusion rules must capture phase/cohort-specific thresholds, units, timing, and retest logic.
- Treatment-history package exclusions must include every protocol-listed medication, biologic, vaccine, immunotherapy, surgery/procedure, device, or transplant subitem. Evidence focus must follow the specific subitem under review and must not highlight an adjacent subitem merely because it appears earlier in the same paragraph.
- Concomitant/prohibited medication rules must incorporate the protocol's medication/treatment section, including washout periods, stability exceptions, and trial-period restrictions.
- Disease-history and pulmonary-function exclusions must apply the protocol's explicit exceptions and investigator CS/NCS judgment rather than treating every related diagnosis or abnormality as exclusionary.
- If a review cites a broad EDC entry, aggregate IE summary, or unrelated prior record, actively recover the screening/baseline raw medical-record clause when it exists.
- Laboratory/infection rules must cite actual lab/virology/infection rows for each protocol-required component. A single normal component does not prove a multi-component criterion passes. Apply confirmatory-test logic exactly as written, such as antibody-positive plus nucleic-acid-positive requirements or screening-positive plus confirmatory-negative allowances. Do not cite report headers, interpretation legends, bare analyte names, mail headers, or no-result EDC rows.
- Pregnancy/lactation rules must require pregnancy, lactation, HCG, or sex-applicability wording. Do not treat unrelated serum antibody or infection-screening rows as pregnancy evidence.
- Investigator-suitability exclusion rules may default to not triggered when no explicit unsuitable wording exists. If evidence is displayed for that rule, it must contain unsuitable/not-recommended/unable-to-participate wording rather than unrelated randomization, score, laboratory, or generic IE evidence.

Agent-mediated aggregation after writing `llm_rule_results.jsonl`:

```bash
python -m eligibility_review.cli aggregate-llm-review --project <PROJECT> --subject <SUBJECT> --model agent:<MODEL>
python -m eligibility_review.cli compare-manual-ie --project <PROJECT>
python -m eligibility_review.cli build-report --project <PROJECT>
```

Optional remote API pilot:

```bash
python -m eligibility_review.cli run-llm-review --project <PROJECT> --subject <SUBJECT> --limit 1
python -m eligibility_review.cli aggregate-llm-review --project <PROJECT> --subject <SUBJECT>
python -m eligibility_review.cli compare-manual-ie --project <PROJECT>
python -m eligibility_review.cli build-report --project <PROJECT>
```

Scale only after reviewing the formal review output, cited evidence, missing material calls, manual comparison, and ledger event.

## Core Artifacts

- `protocol/criteria_rules.v1.json`: structured rule shells.
- `subjects/<SUBJECT>/ocr/page_results.jsonl`: OCR/VL page output.
- `subjects/<SUBJECT>/evidence/evidence_items.jsonl`: source-bound evidence.
- `subjects/<SUBJECT>/review/rule_prompt_packages.jsonl`: per-rule LLM packets.
- `subjects/<SUBJECT>/review/review_result.json`: conservative offline draft.
- `subjects/<SUBJECT>/review/llm_review_result.json`: formal LLM aggregate when available.
- `manual_ie/manual_ie_subject_summary.json`: manual IE baseline.
- `qc/manual_ie_comparison.json`: manual-vs-review QC.
- `qc/llm_readiness.json`: LLM run readiness.
- `reports/eligibility_review_report.html`: subject-switching HTML report.
- `reports/eligibility_review_report_all_centers.html`: optional combined center/subject-switching HTML report from multiple project outputs.
- `reports/eligibility_review_ledger.xlsx`: Excel review ledger and operational log.
- `audit_ledger.jsonl`: append-only event history.

## Completion Checks

- Run `git diff --check`.
- Run the full test suite:

```bash
uvx --with PyYAML --with pydantic --with python-docx --with openpyxl --with PyMuPDF pytest -q
```

- For HTML changes, open the local report in a browser and verify center/subject dropdown filtering, status multi-select filtering, subject switching, conclusion/risk emphasis, and mobile-width overflow behavior. Manual QC, audit trail, model names, and operational logs belong in the Excel ledger and must not appear in the HTML body.
- For current Chinese report changes, verify failed/evidence-insufficient/conflict/EDC-only/email-only/EDC-plus-email-only rules are prominent and expanded, passed rules are collapsed, there is no "条目概览" section, rule headers use short clinical titles, each expanded card shows the complete protocol criterion, evidence labels use filename/page or EDC sheet only, key evidence text is precisely highlighted without whole-block fallback, EDC evidence omits listing metadata, email/communication evidence labels include `（邮件/沟通记录）`, raw+EDC consistent facts display primary raw evidence only, raw quotes include readable original text, current-rule relevance gates reject unrelated candidate text, actual score rules do not cite scoring reminders, the sidebar remains fixed during right-panel scrolling, and no visible `RAW-*` / `EDC-*` IDs, model names, machine enum labels, or raw audit events appear in HTML.
- Verify HTML and Excel share the same effective verdict logic. A rule downgraded because only an IE/IEYN aggregate field is present must appear as `证据不足` in the subject header/sidebar, rule card, Excel summary, Excel rule detail, and supplement-needed sheet.
- Open or inspect `reports/eligibility_review_ledger.xlsx` and verify the sheets `审核汇总`, `规则明细`, `运行台账`, `人工对照`, and `补充资料清单` exist.
- For multi-center delivery, also inspect `reports/eligibility_review_ledger_all_centers.xlsx` and confirm it has the same required sheets and one rule-detail row per subject-rule.
- Do full-batch QC for evidence-gate regressions when changing report logic: no non-exception pass may have zero displayable evidence; investigator-suitability exclusions may only show explicit unsuitable wording; laboratory/infection rules must not show headers, interpretation legends, bare lab markers, or no-result EDC rows; pregnancy/lactation rules must not show unrelated serum-antibody or infection-screening rows as pregnancy evidence. Confirm the report contains the expected subject-rule count after any official-numbering correction.
- Update `docs/PROJECT_CONTEXT.md` after each milestone so context compaction does not erase project state.
