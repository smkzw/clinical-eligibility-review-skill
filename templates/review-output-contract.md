# Review Output Contract

Use this contract when asking an Agent or model to perform formal eligibility review.

## Rule Result

Each subject-rule result should include:

- subject ID;
- rule ID;
- verdict;
- short clinical decision basis;
- evidence references;
- missing materials;
- source consistency;
- source quality concerns;
- whether the rule should be expanded in the HTML report.

Recommended verdict values:

- `pass`
- `fail`
- `insufficient`
- `conflict`
- `needs_investigator_review`
- `not_applicable`

## Evidence Reference

Each evidence reference should include:

- evidence source type;
- original file name and page, or EDC sheet/table;
- exact quote from the source;
- highlighted phrase or phrase span;
- whether it supports or challenges the conclusion;
- OCR/source quality notes if relevant.

Do not rewrite original evidence quotes. Interpretations belong in the decision basis.

## Source Consistency

Recommended source-consistency categories:

- primary raw source supports conclusion;
- primary raw source and EDC are consistent;
- EDC only;
- email/communication only;
- EDC plus email/communication only;
- conflicting sources;
- no candidate evidence;
- source quality prevents final judgment.

In reviewer-facing reports, translate these categories into natural language.

## Subject Overall Result

Subject-level aggregation should prioritize:

1. any failed inclusion or met exclusion criterion;
2. evidence conflict or investigator-review requirement;
3. evidence insufficiency;
4. verification-required passes due to secondary-only evidence;
5. ordinary pass.

If any rule is `pass` but verification-required because it relies only on secondary evidence, the subject should not display as an ordinary pass.

## HTML Rules

- Show one subject at a time.
- Use center and subject dropdown filters.
- Use status multi-select filters.
- Show subject basics and overall result at the top.
- Show priority warnings before rule cards.
- Collapse ordinary passes.
- Expand failed, insufficient, conflicting, investigator-review, and verification-required rules.
- Hide operational logs, model names, internal IDs, and debug information.

## Excel Ledger Rules

The Excel ledger may include operational details that are hidden from the HTML report:

- run time;
- model route;
- OCR/VL model;
- privacy mode;
- trigger;
- prior and new result;
- supplemental evidence;
- manual tracker comparison;
- unresolved missing materials.
