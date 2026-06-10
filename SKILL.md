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
- Current-rule semantic gating must cover every official inclusion and exclusion criterion, not only defect-triggering examples. A rule-level pass must survive a rule-specific evidence check for the exact clinical fact(s) required by that rule. Broad candidate text, neighboring subitems in the same paragraph, visit plans, scoring reminders, headers, or generic medical-history context cannot be promoted to display evidence merely because they share a keyword.
- When both strong protocol-level evidence and weak contextual evidence exist, display the strong evidence and suppress the weak evidence. For example, an EX-02 source sentence denying `离开已知花粉区48小时及以上的旅行计划` supersedes a generic reminder such as `请勿离开本地`; the reminder alone may be used only as weak fallback and should remain verification-prone.
- EDC aggregate eligibility fields such as `是否满足所有的入选标准且不满足任一的排除标准(IEYN)` are not rule-level evidence. Never use them to support an individual inclusion or exclusion criterion. If no other valid source exists for that rule, mark it as evidence-insufficient in both HTML and Excel, not as a verification-required pass.
- A rule should not render as `通过` if it has no displayable, current-rule-relevant evidence. The only exception is an explicitly configured presence-only exclusion rule, such as an investigator-suitability exclusion that defaults to not triggered when no unsuitable wording exists.
- Evidence quotes in HTML must highlight the exact sentence or phrase that supports or challenges the rule. Preserve enough original context around the highlighted text, especially when OCR output is noisy. Do not use whole-block highlighting as a fallback; if the review quote uses ellipses, OCR line breaks, or normalized wording, locate and highlight the matching original source phrase.
- For focus-validated rules, do not display a model-provided raw quote as-is when it is broad, contains neighboring-rule text, visit reminders, diary/scoring instructions, URLs, or other irrelevant markers. Re-slice the quote against the rule-specific source span before rendering. Preserve the model quote only when it is already short, locatable in the source, and clinically focused.
- Clinical evidence extraction must tolerate OCR single-line wraps inside a fact. Do not truncate phrases such as history duration, drug names, surgery clauses, or monoclonal-antibody names merely because OCR inserted a newline in the middle of the sentence.
- The visible decision basis must match the visible evidence. If display logic hides redundant EDC because primary raw evidence is available, do not leave wording that says the conclusion relies on EDC.
- The visible decision basis must be clinical reasoning, not report operation text. Remove artifacts such as `仅EDC证据`, `仅二手来源`, `报告需默认展开`, raw evidence IDs, and source-routing notes from the reasoning sentence. Source caveats belong in badges, priority warnings, and Excel source-consistency fields.
- EDC evidence in HTML must be trimmed to clinical fields before display. Remove listing metadata such as project code, form code, subject ID, initials, site code/name, row number, and last modified time. Keep non-empty clinical fields such as result, medication, reason, date, assessment, and relevant yes/no values.
- Core eligibility support should come from primary raw sources such as medical records, laboratory reports, examination reports, and signed source records. EDC listings, emails, and communication records are secondary/processed evidence sources. Do not render a generic `仅二手来源` label. If a passed rule is supported only by EDC, only by email/communication records, or by EDC plus email/communication records without primary raw evidence, the subject-level priority warning must visibly say `通过（需验证：仅EDC数据）`, `通过（需验证：仅邮件/沟通记录）`, or `通过（需验证：仅EDC数据及邮件/沟通记录）`. Rule cards and Excel detail rows should mark the result as `通过（需验证）` and keep the specific source type in the adjacent badge/consistency field.
- If an otherwise eligible subject has any `通过（需验证：...）` rule, the left subject switcher and the subject header's overall result must also display `通过（需验证）`, not plain `通过`. Filter metadata should include `需验证` and the concrete source type so reviewers can filter these cases.
- If raw primary source and EDC support the same fact and are consistent, show raw primary evidence in HTML and omit the redundant EDC citation. Email/communication records remain secondary evidence even when stored as raw-document files.
- Subject age and sex should be extracted from raw screening/baseline medical-record headers when available; EDC demographics are a fallback. If both medical-record headers and examination/lab report headers contain age/sex, prefer the screening/baseline medical record and use report headers only as fallback.
- Report text must be natural Chinese. Internal enum names, risk flag keys, machine field names, model labels, row IDs, and debug identifiers must be translated before rendering or the report should fail generation.
- The left subject switcher must use dropdown filters for `研究中心` and `筛选号`, plus multi-select status filters. Do not use a free-text input as the main reviewer filter. Sort center options by center code, and sort subject options/default list by subject ID.
- Status filters should include `不通过`, `证据不足`, `证据冲突`, `通过需验证`, and `通过`. Visual emphasis must follow clinical priority: 不通过 > 证据不足/证据冲突 > 通过需验证 > 通过. `通过（需验证）` must be visible but quieter than evidence-insufficient or conflicting cases.
- The subject switcher should use card-like status styling: fixed/sticky sidebar, light subject cards, a left status stripe, restrained clinical colors, and a sponsor/project accent color only as a controlled focus accent. The left sidebar must stay fixed while the right subject report scrolls independently. Keep it serious and report-like.

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
- For difficult table/image pages, compare suitable-DPI OCR against a VLM fallback before concluding that source evidence is missing. A page with visible critical values but failed OCR must be routed to VLM or human visual QC rather than treated as absent evidence.
- On local oMLX, large VLMs must be loaded one at a time. Do not send requests to multiple large VLMs in parallel. When comparing local VLMs, load one model, run the planned page(s), unload it with `POST /admin/api/models/{model_id}/unload`, confirm `loaded=false` from `/admin/api/models`, then load the next model. Do not unload embedding or OCR models merely because a large-VLM comparison is running.
- If a single local VLM returns HTTP 507 or fails to load under current memory conditions, record the failure and stop forcing that model for the batch.
- Record local VLM comparison results in the project context/config instead of hard-coding a universal winner. Prefer the model that returns parseable structured output with exact evidence text for the target document type. Do not generalize one study's model result to all studies.

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

- Rescreen/retest: TNSS scoring failure is not a rescreen justification. Non-TNSS explainable factors/disease may allow at most one rescreen, with a new ICF and new screening number. Lab abnormalities that meet exclusion may be retested once; if the retest is normal, the subject can continue without a new screening number, and no symptomatic treatment is allowed before retest.
- IN-04: screening iTNSS and baseline iTNSS/rTNSS thresholds require component-level and total-score support. Baseline rTNSS uses the last six run-in timepoints plus D1, and baseline iTNSS uses the last three run-in timepoints plus D1.
- IN-04 evidence must be actual score evidence. Diary-card instructions, scoring reminders, or visit-procedure text are not threshold evidence. If screening/baseline raw notes do not contain score fields, actively inspect the configured EDC score subtables such as `RES_1` and `RES1` rather than highlighting unrelated raw text.
- IN-04 displayed raw quotes should start at the actual iTNSS/rTNSS scoring clause and end at the corresponding total-score phrase. They must not include diary-card instructions, `嘱托`, visit reminders, or unrelated eye-score/visit-procedure text just because those words appear nearby.
- IN-02 is a multi-component rule. Visible evidence must separately cover SAR diagnosis/history duration, especially `明确病史≥2年`, and at least one season-relevant positive sIgE/allergen result row with allergen name/code, value, unit, flag, and threshold. For history, use this source hierarchy: prior medical records or disease/medication-related source records first; screening/baseline medical-record narrative second and mark `病史来源需溯源验证`; EDC SARH third and mark EDC-only/verification required; aggregate IE/IEYN never counts. For sIgE/allergen reports, a report header, `已查sIgE`, `过敏原异常结果详见附件`, or an interpretation legend such as `6级 重度过敏` is not result evidence. If OCR only captures the report header and misses the result table, trigger second-pass OCR/table extraction or human source verification and keep the rule insufficient until concrete rows are available.
- IN-03 evidence must show inadequate prior treatment response, relevant medication/treatment history, or direct conflict/query text. Generic allergic-rhinitis history is not enough.
- IN-05: phase III baseline blood EOS must be >=300/μL. If screening EOS is below threshold but D1/baseline retest meets the threshold, treat it under the protocol's baseline/retest logic and cite the raw D1/baseline report when available.
- Official protocol numbering and package criteria must be preserved from the active protocol. If a study has a treatment-history package or other multi-subitem criterion, keep the official parent rule ID and track subitems as components rather than inventing new official exclusion numbers. Store any study-specific renumbering/remap table in project context/config, not in this generic skill.
- Medication/treatment-history package criteria should be decomposed into literal subitems from the protocol, such as antihistamines, leukotriene receptor antagonists/mast-cell stabilizers, systemic corticosteroids or protocol-defined traditional medicines, permitted asthma-controller exceptions, systemic immunosuppressants, biologics/monoclonal antibodies, prior same-investigational-product trial, live/attenuated vaccine, immunotherapy, nasal/nasal-sinus surgery, and organ/hematopoietic/bone-marrow transplant history when present. Evidence focus must follow the specific subitem under review and must not highlight an adjacent package subitem merely because it appears earlier in the same paragraph.
- Package subitem evidence must stay literal. Do not infer one subitem from vague OCR wording, and do not match a project header containing the investigational product name; a prior-trial subitem requires direct prior-participation wording. If OCR splits a required phrase across pages, reconstruct the displayed quote from adjacent OCR pages instead of showing a truncated phrase.
- EX-07 disease-history package must not borrow EX-06 medication-history evidence. In particular, `免疫抑制/机会感染/频发感染` means immune-suppressed state or opportunistic/recurrent/long-term infection history, not `免疫抑制剂` medication use. If the source says `否认为免疫抑制者`, cite that disease-history clause.
- EX-12: concomitant/prohibited medication and treatment compliance rule must include protocol section 6.5 medication/treatment requirements, including antihistamines 4 days, leukotriene modifiers 1 week, immunosuppressants 8 weeks or 5 half-lives, nasal non-drug treatment 4 weeks, eye treatments/surgery affecting SAR symptoms 4 weeks, and trial-period prohibited medication categories.
- EX-07/EX-08: do not equate every nasal, asthma, pet-allergy, infection, or lung-function finding with exclusion. Apply the explicit protocol exceptions and investigator NCS/CS judgment; FEV1 exclusion is pre-bronchodilator FEV1% predicted <=50%.
- EX-08 FEV1 evidence must contain a value or an interpretable `% predicted` field. A pulmonary-function report header or table header containing only `FEV1` is not evidence. If raw OCR has only the header but EDC `PT` contains `FEV1占预计值百分比（%）(PERESP)` and clinical assessment, use EDC as fallback and mark the pass as verification-required.
- EX-08 displayed raw quotes should be sliced to the FEV1 row or immediately reconstructed row, not the whole lung-function report. For multi-line tables, start at the FEV1 marker and include the numeric predicted/actual/% predicted values; do not include the previous metric row merely for context.
- EX-04/EX-05: if a review cites a broad EDC entry, IE summary, or unrelated prior record, actively recover the screening/baseline raw medical-record clause when it exists. EX-04 should cite the outdoor-activity clause; EX-05 should cite the IL-4Rα/dupilumab or equivalent monoclonal-antibody clause.
- EX-09: HBsAg positive excludes. HBcAb positive alone does not; HBcAb positive requires HBV-DNA positive to trigger exclusion. HCV antibody positive requires HCV-RNA positive. TP-Ab positive is allowed if RPR/TRUST is negative. Lab thresholds and investigator clinical-significance judgments must be cited. EX-09 evidence should show actual lab/virology/infection rows for each required component: ANC/neutrophil count, AST, ALT, TBil, creatinine, HBV/HBsAg/HBcAb/HBV-DNA logic, HCV/HCV-RNA logic, HIV, and TP-Ab/RPR/TRUST. Do not cite allergen interpretation legends as EX-09 evidence. Short Latin lab terms such as AST/ALT require token-boundary matching; do not match `AST` inside `Master` or URLs. If OCR splits a lab row across lines, reconstruct the analyte-result-unit-reference row. EDC rows such as `是否检查：否/无需` are not EX-09 evidence unless they also contain a concrete relevant lab/virology result. This criterion is triggered by any failing component; one normal component does not prove the whole criterion passes.
- EX-09 raw evidence should prioritize laboratory/examination reports over medical-record abnormality summaries. Medical-record summaries may explain NCS/CS context but should not replace actual analyte-result rows when reports exist. A medical-history phrase such as `肌酐升高病史，诊断时间...无需治疗` is not a creatinine result row, and dates such as `2025-09-02` must not be interpreted as laboratory reference ranges. If OCR breaks a lab row, reconstruct the analyte-result-method/reference context, for example `肌酐(Cr)测定 8-HR 117 ... 酶法`. Structured EDC virology rows may count for HBV/HCV/HIV/TP-Ab components when the indicator name maps to the component and the row contains a numeric result plus assessment or reference limit; accept common labels such as `HCV-Ab`, `HIV-Ab`, `Anti-TP/anti-TP`, and `梅毒螺旋体抗体`.
- EX-14: pregnancy/lactation rules apply to female subjects; for male subjects mark not applicable with sex evidence. EX-14 evidence must contain pregnancy/lactation/HCG-specific wording or sex applicability; do not treat unrelated serum antibody rows such as anti-TP/TP-Ab as pregnancy evidence.
- EX-16: if there is no explicit wording such as `不适合参加`, `不适合入组`, `不建议参加`, or `无法参加本研究`, treat the exclusion as not triggered and display the pass basis as no unsuitable wording found. Do not show unrelated randomization, score, lab, or generic IE evidence for EX-16.

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
- Do full-batch QC for evidence-gate regressions when changing report logic: no non-presence-only exclusion pass may have zero displayable evidence; presence-only investigator-suitability exclusions may only show explicit unsuitable wording; laboratory/infection rules must not show headers, allergen legends, bare lab markers, or no-result EDC rows; pregnancy/lactation rules must not show unrelated serology as pregnancy evidence. Check every reviewed subject-rule detail in the active project; keep project-specific expected counts in project context.
- Full-batch QC should also scan decision bases and display refs: no clinical reasoning sentence should contain `仅EDC证据`, `报告需默认展开`, `仅二手来源`, raw/EDC IDs, or source-routing instructions; IN-04 pass evidence must not include diary/reminder/instruction text; EX-08 pass evidence must include an FEV1 marker and numeric/interpretable values.
- Update `docs/PROJECT_CONTEXT.md` after each milestone so context compaction does not erase project state.
