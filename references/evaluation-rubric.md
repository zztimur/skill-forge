# Evaluation Rubric

Score the skill out of 100 points, then assign a concise qualitative label. The score measures quality within the inspected evidence boundary; it is not a substitute for a release verdict.

[`audit-contract.json`](audit-contract.json) is the canonical contract for result enums, evidence labels, gate IDs, pressure categories, and score-cap values. The scorecard, severity list, required-gate counts, and release verdict must reconcile to that contract.

| Category | Weight | What to look for |
|---|---:|---|
| Triggering and description quality | 20 | clear frontmatter description; specific trigger contexts; low false positives/false negatives; no key trigger guidance hidden only in the body |
| Workflow clarity and instruction quality | 20 | actionable sequence; clear inputs and outputs; decision points; tool/resource usage; minimal ambiguity; no conflicting instructions |
| Reliability under pressure tests | 20 | survives happy path, minimal input, ambiguity, missing dependencies, malformed input, conflict, boundary, safety, and regression cases |
| Use of scripts/references/assets | 15 | scripts used for deterministic or fragile work; references loaded only when needed; assets necessary and reasonably sized; no orphaned or template/example resources |
| Error handling and edge cases | 10 | graceful behavior when files, dependencies, connectors, inputs, or permissions are missing; clear fallback instructions |
| Safety, privacy, and security | 10 | no bundled secrets; safe execution guidance; privacy-sensitive data handled carefully; no unsafe automation or silent external actions |
| Maintainability and polish | 5 | clean structure; consistent naming; concise wording; easy future edits; no template leftovers |
| **Total** | **100** | |

## Evidence Status

Label material claims consistently:

- **Verified** — directly observed in the selected artifact or tool output.
- **Inferred** — a reasoned prediction from instructions or a simulation; name the source and uncertainty.
- **Unverified** — needed evidence was unavailable. Describe the limitation or risk, not a defect as fact.

For pressure tests, **Static simulation** is always Inferred. **Synthetic execution**
can verify artifact behavior in an approved bounded environment;
**Live host observation** is required to verify host triggering or runtime
behavior. When the method is weaker than the stated observation requirement,
the test result is Not Assessed.

## Scoring Method

1. Assign a whole-number score for every category.
2. Show each category's maximum, awarded score, and brief evidence in the report.
3. Sum the category scores exactly to `/100`; correct arithmetic before assigning a qualitative label.
4. State whether the score covers a complete package, draft-only text, or a limited adjacent review. Do not translate a draft-only score into a release claim.
5. Apply the caps below before reporting the final total. A cap means the score must not exceed that value, even if prose quality is otherwise strong.

## Rating Conversion

| Score | Qualitative label | Meaning |
|---:|---|---|
| 0–39 | Broken or unsafe | Major reliability or safety work is required. |
| 40–59 | Needs major work | Core behavior needs material repair. |
| 60–69 | Usable but rough | Useful but materially incomplete. |
| 70–79 | Solid with material gaps | Sound foundation with release-blocking gaps. |
| 80–89 | Strong | Good quality with targeted improvements remaining. |
| 90–99 | Excellent | Complete, reliable work within the assessed scope. |
| 100 | Exceptional, when justified | Release-ready within the assessed scope; use Exceptional only with the evidence below. |

The `/100` score is authoritative. A release verdict remains a separate decision.

## Evidence Caps and Release Rules

- Any outstanding **Critical** issue caps the score at **49/100** and prevents a release-ready verdict.
- Any outstanding **High** issue caps the score at **79/100** and prevents a release-ready verdict.
- Missing pressure-test evidence, or a failed required pressure test, caps the score at **79/100** unless the test is clearly Not Applicable and the rationale is recorded.
- A completed **Partial** pressure test counts toward coverage but not behavioral success: it makes G20 Partial and blocks release, but does **not** alone trigger the 79-point missing/failed-evidence cap. Score its observed quality gap normally. G20 therefore measures both category coverage and behavioral success.
- A draft-only or limited review may receive a design-quality score, but its release verdict is **Not Assessed**, never Pass.
- An **Installed runtime** proves installed state only and cannot support Pass
  for a new release. A Release ZIP is eligible as the exact artifact. A mutable
  source checkout requires an archive built from a committed revision and explicit packaging authority
  before it can support release evidence.
- A failed required release gate, strict inspection failure, official-validator failure, or material safety finding prevents a release-ready verdict even if the numeric score is otherwise high.
- A score of **90+** requires a complete package, no Critical or High issues, meaningful pressure-test evidence, clean applicable validation evidence, and sensible safety and fallback behavior.
- A `100/100` requires a release-gate Pass within the selected scope and no outstanding issue above Low severity.
- An optional **Exceptional** label may accompany `100/100` when the complete package is compact, clear, robust under meaningful pressure tests, easy to maintain, and unusually difficult to break in realistic use. State the concrete evidence; never use a numeric score above `100/100`.

The release-verdict roll-up is **Fail > Not Assessed > Partial > Pass** after
ignoring Not Applicable rows. This precedence is independent of the numeric
score and of `quality_policy_result`, which remains separate from validator and
compatibility results.

Use the full range. Do not award a high score merely because the skill is well-written; it must also trigger correctly, handle edge cases, and use resources appropriately.
