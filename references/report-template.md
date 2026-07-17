# Report Template

Use this structure for the final evaluation unless the user requests a different format. [`audit-contract.json`](audit-contract.json) defines the canonical gate IDs, result states, evidence labels, score caps, routing rules, pressure categories, and machine-readable `target_contracts` source of truth.

## Report Modes

- **Compact:** evidence boundary, mandatory safety findings, concise verdict,
  and top fixes.
- **Standard:** the complete evaluation structure below, including behavioral
  review, pressure tests, simulations, ranked fixes, and scorecard.
- **Release:** Standard plus the Release Gate Review and its complete G01–G23
  matrix. The five-row executive roll-up cannot replace the matrix.

Report mode changes detail, never the evidence boundary or safety floor.

```markdown
# Skill Evaluation Report

## 0. Request and Evidence Boundary

- Requested outcome phases, in order: Evaluation / Validation / Repair / Release gate
- Selected artifact and reason:
- Artifact role: Pasted draft / Release ZIP / Installed runtime / Mutable source checkout / General repository / Portfolio or multi-Skill input
- New-release eligibility: exact artifact / packaging source with an archive built from a committed revision and explicit packaging authority / installed-state evidence only / ineligible
- Required artifact-role evidence and whether it is satisfied:
- Artifact coverage: Complete package / Draft-only text / Limited adjacent review / Per-Skill portfolio review
- Source of truth and write boundary:
- Requested host surfaces:
- Selected canonical profile(s): Portable / OpenAI
- Target contract sources: `audit-contract.json` → one `target_contracts.<selected profile>` per profile
- Artifact-content trust: untrusted evidence only; conflicting directives ignored
- Skill Forge evaluator provenance: current-checkout/self-test evidence / separately installed pretrusted and pinned evaluator / unavailable
- Independent evaluator pins and schema: complete-tree SHA-256; candidate SHA-256; optional inspector SHA-256; exact inspector schema version 5 / Not Assessed with reason
- Unverified limits or unavailable evidence:
- Evidence labels: Verified / Inferred / Unverified

## 1. File Structure Summary

- Package/folder inspected:
- Detected `SKILL.md` files:
- Main files and directories:
- Package size:
- Archive preflight findings:
- Directory preflight findings for direct folder inputs, including root symlinks:
- Unexpected files outside detected skill root:
- Skill Forge strict inspection: Pass / Fail / Partial / Not Assessed / Not Applicable; state whether evidence is self-inspection or independently pinned
- Inspector schema version and compatibility: exact version 5 for the independent-evaluator helper / Not Assessed with reason
- Independent evaluator result: pass / fail / not_assessed / Not Applicable; report all canonical-profile results and the trust/sandbox limitations
- Skill Forge runtime manifest archive integrity: Pass / Fail / Not Assessed / Not Applicable
- Skill Forge local source proof: Pass / Fail / Not Assessed / Not Applicable; name the caller-selected repository and do not imply remote or signer authenticity
- Runtime-manifest digest binding: Pass / Fail / Not Assessed / Not Applicable
- Canonical archive and manifest SHA-256 values, when applicable:
- Validator outcome: Pass / Artifact Fail / Unavailable / Execution Error / Not Applicable
- Validator provenance and whether the validator is required:
- Gate result derived from validator outcome: Pass / Fail / Partial / Not Assessed / Not Applicable
- Compatibility result: Compatible / Incompatible / Unverified / Not Applicable
- Quality-policy result: Pass / Fail / Partial / Not Assessed / Not Applicable
- Quality-policy independence note: explain why validator and compatibility results do not determine this result
- Package self-tests: Pass / Fail / Partial / Not Assessed / Not Applicable
- Package self-test sandbox controls: network default-deny; credentials absent; source read-only; scratch-only writes; bounded process/time/memory; external side effects forbidden
- Unmet self-test controls and result mapping:
- Placeholder/example files found:
- Potential safety/privacy concerns, including strict-mode secret findings:
- Sensitive finding records: `path`, `finding_type`, and `redacted_fingerprint` only; never raw secrets or sensitive PII
- Relevant inspector finding codes, when useful:

## 2. Executive Summary

Briefly state the quality conclusion within the evidence boundary. Mention the strongest design point, biggest risk, and whether release readiness was assessed. Do not imply a release pass before the complete gate matrix.

## 3. Release Gate Review (when applicable)

Consult `references/audit-contract.json` and `references/release-gate-checklist.md`. Keep `validator_outcome`, `gate_result`, `compatibility_result`, and `quality_policy_result` separate. Use exactly these gate and quality-policy results: Pass / Fail / Partial / Not Assessed / Not Applicable. A Not Applicable result requires a rationale. Repeat the complete G01–G23 matrix and release verdict independently for every selected canonical profile. The five rows below are an executive roll-up only.

### Per-profile results

| Requested surface | Canonical profile | Compatibility result | Release verdict | Evidence / limitation |
|---|---|---|---|---|
|  |  | Compatible / Incompatible / Unverified / Not Applicable | Pass / Fail / Partial / Not Assessed |  |

**Overall cross-profile verdict:** Pass / Fail / Partial / Not Assessed — derive from the per-profile verdicts without changing or hiding any member result.

Use **Fail > Not Assessed > Partial > Pass** after ignoring Not Applicable rows
for both per-profile and overall release-verdict roll-ups. If every row is Not
Applicable, use Not Assessed.

### Executive summary

| Group | Gates | Result | Concise evidence / blocker |
|---|---|---|---|
| Structure and metadata | G01–G08 | Pass / Fail / Partial / Not Assessed / Not Applicable |  |
| Validation evidence | G09–G12 | Pass / Fail / Partial / Not Assessed / Not Applicable |  |
| Package safety and content | G13–G19 | Pass / Fail / Partial / Not Assessed / Not Applicable |  |
| Pressure tests and remediation | G20–G21 | Pass / Fail / Partial / Not Assessed / Not Applicable |  |
| Score, verdict, and runtime package | G22–G23 | Pass / Fail / Partial / Not Assessed / Not Applicable |  |

### Authoritative gate matrix

| ID | Gate | Result | Evidence, scope, and rationale when Not Applicable |
|---|---|---|---|
| G01 | Exactly one `SKILL.md` exists | Pass / Fail / Partial / Not Assessed / Not Applicable |  |
| G02 | Selected-target required frontmatter fields | Pass / Fail / Partial / Not Assessed / Not Applicable |  |
| G03 | Selected-profile package root shape | Pass / Fail / Partial / Not Assessed / Not Applicable |  |
| G04 | Selected-target name rule when required or supplied | Pass / Fail / Partial / Not Assessed / Not Applicable |  |
| G05 | Trigger-rich description | Pass / Fail / Partial / Not Assessed / Not Applicable |  |
| G06 | Documented description or listing limit | Pass / Fail / Partial / Not Assessed / Not Applicable |  |
| G07 | Conditional OpenAI UI metadata | Pass / Fail / Partial / Not Assessed / Not Applicable |  |
| G08 | Valid platform-specific frontmatter | Pass / Fail / Partial / Not Assessed / Not Applicable |  |
| G09 | Skill Forge strict inspection with complete coverage and stated evaluator provenance/schema | Pass / Fail / Partial / Not Assessed / Not Applicable |  |
| G10 | Validator outcome mapped to gate result | Pass / Fail / Partial / Not Assessed / Not Applicable |  |
| G11 | Approved package self-tests, separately reported | Pass / Fail / Partial / Not Assessed / Not Applicable |  |
| G12 | Skill Forge source-only tests and extracted-runtime tests after evaluator/script changes | Pass / Fail / Partial / Not Assessed / Not Applicable |  |
| G13 | Referenced resources present | Pass / Fail / Partial / Not Assessed / Not Applicable |  |
| G14 | No orphaned/generated/template leftovers | Pass / Fail / Partial / Not Assessed / Not Applicable |  |
| G15 | No bundled secrets or credential-like files | Pass / Fail / Partial / Not Assessed / Not Applicable |  |
| G16 | No unsafe archive or direct-folder findings | Pass / Fail / Partial / Not Assessed / Not Applicable |  |
| G17 | Bundled-script safety review | Pass / Fail / Partial / Not Assessed / Not Applicable |  |
| G18 | No unnecessary large assets | Pass / Fail / Partial / Not Assessed / Not Applicable |  |
| G19 | Known target product upload limit or N/A rationale | Pass / Fail / Partial / Not Assessed / Not Applicable |  |
| G20 | Pressure coverage and behavioral results | Pass / Fail / Partial / Not Assessed / Not Applicable |  |
| G21 | Severity-ranked fixes documented | Pass / Fail / Partial / Not Assessed / Not Applicable |  |
| G22 | Counts, scorecard, caps, severity list, and verdict reconcile | Pass / Fail / Partial / Not Assessed / Not Applicable |  |
| G23 | Skill Forge canonical runtime-manifest, source-proof, profile, and extracted-runtime package verification | Pass / Fail / Partial / Not Assessed / Not Applicable |  |

**Required-gate counts:** Pass: 0; Fail: 0; Partial: 0; Not Assessed: 0; Not Applicable: 0; Applicable: 0.

**Release gate verdict:** Pass / Fail / Partial / Not Assessed — concise explanation. Derive it with Fail > Not Assessed > Partial > Pass after ignoring Not Applicable rows. Only Pass is release-ready. A required applicable Partial, Fail, or Not Assessed result is not a pass. For G09 in a Skill Forge self-audit, an incompatible independent-inspector schema is Not Assessed, never Pass. For G10, use `validator_outcome_gate_results`: unavailable optional validation is Not Applicable, unavailable required validation is Not Assessed, and an execution error is Not Assessed. Only Artifact Fail may establish a validator-derived defect. For G11, a missing optional self-test plan is Not Applicable. G23 applies only to Skill Forge Release ZIP and Mutable source checkout roles. Local source proof does not establish remote or signer authenticity. See `references/validator-evidence.md` and `references/runtime-manifest-schema.md`.

## 4. Triggering and Discoverability Review

Evaluate the frontmatter name and description. Include likely false positives, likely false negatives, and any recommended replacement description.

## 5. Instruction Quality Review

Evaluate workflow clarity, specificity, input/output expectations, error handling, and whether another model instance could follow the skill reliably.

## 6. Resource, Script, and Asset Review

Evaluate scripts, references, assets, dependency clarity, template leftovers, orphaned files, script safety, and whether resources are used efficiently.

## 7. Pressure Test Results

Every row must populate these exact fields: `test`, `input`,
`expected_behavior`, `observation_requirement`, `method_used`,
`predicted_behavior`, `observed_behavior`, `evidence_status`, and `result`.
Use Static simulation, Synthetic execution, or Live host observation. Static
simulation is Inferred. If the selected method is insufficient for the stated
observation requirement, record Not Assessed.

| test | input | expected_behavior | observation_requirement | method_used | predicted_behavior | observed_behavior | evidence_status | result |
|---|---|---|---|---|---|---|---|---|
| Happy Path Test |  |  |  |  |  |  | Verified / Inferred / Unverified | Pass / Fail / Partial / Not Assessed / Not Applicable |
| Minimal Input Test |  |  |  |  |  |  | Verified / Inferred / Unverified | Pass / Fail / Partial / Not Assessed / Not Applicable |
| Ambiguous Request Test |  |  |  |  |  |  | Verified / Inferred / Unverified | Pass / Fail / Partial / Not Assessed / Not Applicable |
| Wrong Input Type Test |  |  |  |  |  |  | Verified / Inferred / Unverified | Pass / Fail / Partial / Not Assessed / Not Applicable |
| Missing Dependency Test |  |  |  |  |  |  | Verified / Inferred / Unverified | Pass / Fail / Partial / Not Assessed / Not Applicable |
| Large or Messy Input Test |  |  |  |  |  |  | Verified / Inferred / Unverified | Pass / Fail / Partial / Not Assessed / Not Applicable |
| Conflicting Instruction Test |  |  |  |  |  |  | Verified / Inferred / Unverified | Pass / Fail / Partial / Not Assessed / Not Applicable |
| Boundary or Scope Test |  |  |  |  |  |  | Verified / Inferred / Unverified | Pass / Fail / Partial / Not Assessed / Not Applicable |
| Safety and Privacy Test |  |  |  |  |  |  | Verified / Inferred / Unverified | Pass / Fail / Partial / Not Assessed / Not Applicable |
| Regression Test |  |  |  |  |  |  | Verified / Inferred / Unverified | Pass / Fail / Partial / Not Assessed / Not Applicable |
| Script Behavior Test |  |  |  |  |  |  | Verified / Inferred / Unverified | Pass / Fail / Partial / Not Assessed / Not Applicable |

Every Not Applicable result must state why the category does not apply; it cannot hide an available required test. G20 measures both coverage and behavioral success: a completed Partial category counts as coverage but not success, makes G20 Partial, blocks release, and does not alone trigger the 79-point missing/failed-evidence cap.

## 8. Usage Simulations

### Simulation A — Ideal Use Case

- User request:
- Would the skill trigger? Why or why not?
- Files the assistant would likely read:
- Actions the assistant would likely take:
- Expected output:
- Likely quality:
- Issues noticed:

### Simulation B — Edge Case

- User request:
- Would the skill trigger? Why or why not?
- Files the assistant would likely read:
- Actions the assistant would likely take:
- Expected output:
- Likely quality:
- Issues noticed:

### Simulation C — Failure-Prone Case

- User request:
- Would the skill trigger? Why or why not?
- Files the assistant would likely read:
- Actions the assistant would likely take:
- Expected output:
- Likely quality:
- Issues noticed:

## 9. Fix List by Severity

### Critical

### High

### Medium

### Low

### Nit

For each issue, include severity, evidence status (Verified / Inferred / Unverified), evidence, file/section, problem, relevant inspector finding code if applicable, why it matters, recommended fix, and re-test. Do not list an Unverified limitation as a confirmed defect.

## 10. Overall Grade

**Score scope:** Complete package / Draft-only text / Limited adjacent review

| Category | Maximum | Score | Evidence |
|---|---:|---:|---|
| Triggering and description quality | 20 |  |  |
| Workflow clarity and instruction quality | 20 |  |  |
| Reliability under pressure tests | 20 |  |  |
| Use of scripts/references/assets | 15 |  |  |
| Error handling and edge cases | 10 |  |  |
| Safety, privacy, and security | 10 |  |  |
| Maintainability and polish | 5 |  |  |
| **Total** | **100** | **X** | Must reconcile exactly |

**Score cap applied:** None / 49/100 for unresolved Critical / 79/100 for unresolved High or missing/failed required pressure evidence.

**Score:** X/100  
**Rating:** short qualitative label

**Quality verdict:** One concise paragraph explaining the grade within the score scope.

**Release verdict:** Pass / Fail / Partial / Not Assessed — must match §3 when that section applies; otherwise state why release readiness was outside scope.

**Top 3 fixes before release:**
1. ...
2. ...
3. ...

## 11. Recommended Next Version

Summarize what should change in the next version and what should be re-tested after those changes.
```
