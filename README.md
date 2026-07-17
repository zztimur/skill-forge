# Skill Forge

[![Self Tests](https://github.com/zztimur/skill-forge/actions/workflows/self-tests.yml/badge.svg?branch=main)](https://github.com/zztimur/skill-forge/actions/workflows/self-tests.yml)

Skill Forge is a quality gate for Agent Skills.

It reviews OpenAI and portable Agent Skill packages before they get installed, published, or shipped. It combines a deterministic package inspector with a qualitative review workflow so you can catch the boring structural failures, the dangerous edge cases, and the subtle instruction problems that only show up once an agent starts interpreting the skill in the wild.

In plain English: Skill Forge helps you find out whether a skill is actually ready — or just looks ready because the folder has a `SKILL.md` in it.

## What this repository contains

```text
skill-forge/
├── SKILL.md
├── README.md
├── LICENSE
├── .gitignore
├── .gitattributes
├── .github/
│   └── workflows/
│       ├── self-tests.yml
│       └── release-skill.yml
├── agents/
│   └── openai.yaml
├── scripts/
│   ├── generate_release_notes.py
│   ├── inspect_skill_package.py
│   ├── package_skill.py
│   ├── portable_zip_paths.py
│   ├── release_metadata.py
│   ├── release_skill.py
│   ├── run_self_tests.py
│   ├── run_source_tests.py
│   ├── runtime_manifest.py
│   ├── verify_independent_evaluator.py
│   └── validate_audit_contract.py
└── references/
    ├── audit-contract.json
    ├── audit-checklist.md
    ├── artifact-and-mode-matrix.md
    ├── evaluation-rubric.md
    ├── example-report.md
    ├── input-routing.md
    ├── inspector-output-schema.md
    ├── platform-compatibility.md
    ├── pressure-test-suite.md
    ├── release-gate-checklist.md
    ├── report-template.md
    ├── runtime-manifest-schema.md
    ├── severity-framework.md
    └── validator-evidence.md
```

## What Skill Forge checks

Skill Forge looks at two different failure modes:

1. **Package integrity** — the things a script can inspect reliably.
2. **Agent behavior quality** — the things that require judgment, pressure testing, and a working understanding of how skills fail in real use.

It can help evaluate:

- Uploaded Skill ZIPs, folders, or `SKILL.md` drafts.
- Request routing that distinguishes evaluation, validation, repair, release-gate, draft-only, and limited adjacent-review modes.
- Read-only improvement plans versus explicit repair authority, plus the right source-of-truth boundary for drafts, ZIPs, installed runtimes, source checkouts, repositories, and portfolios.
- Package structure and expected entrypoints.
- Cross-platform compatibility risks across OpenAI and portable agent environments.
- Unsafe archive paths, symlinks, oversized files, suspected secrets, destructive or network-piping commands in bundled scripts, missing resources, and leftover template content.
- Trigger quality: when the skill should activate, when it should stay out of the way, and where ambiguity may cause bad routing.
- Instruction clarity, contradiction risk, and unnecessary context bloat.
- Progressive loading: whether the core skill stays lean while detailed rubrics and references live in linked files.
- Safety posture and risky workflow assumptions.
- Pressure-test results and release readiness.
- Evidence-linked scorecards that reconcile severity findings, score caps, and release decisions.
- Separate evidence for Skill Forge inspection, trusted official platform validation, and optional package self-tests.

The goal is not to produce a pretty audit for a dashboard. The goal is to stop weak, unsafe, confusing, or overbuilt skills from getting shipped with a confident little smile on their face.

## Request and artifact boundaries

Reviews, audits, validation, suggestions, improvement plans, and draft patches
are read-only. Repair requires an explicit mutation request such as “apply,”
“edit,” “implement,” “fix,” “correct,” “rewrite,” or “refactor,” and a confirmed mutable source artifact. Evaluation and
validation do not authorize packaging, installation, commits, pushes,
publication, or external actions.

Use [`references/artifact-and-mode-matrix.md`](references/artifact-and-mode-matrix.md)
for the full routing and artifact-role matrix. In particular, an installed Skill
Forge runtime is not a Git source checkout: locate and confirm the source before
self-repair or building an archive from a committed revision. Portfolio reviews keep findings,
scores, and release verdicts separate for every Skill.

## Running the inspector locally

The inspector is dependency-free and uses only the Python standard library. It requires Python 3.9 or newer.

```bash
python3 -S scripts/inspect_skill_package.py /path/to/skill-or-skill.zip --json
```

For CI-style checks, use strict mode. Strict mode exits with code `2` when any error-severity finding is present.

```bash
python3 -S scripts/inspect_skill_package.py /path/to/skill-or-skill.zip --json --strict
```

Human-readable markdown output is available by omitting `--json`:

```bash
python3 -S scripts/inspect_skill_package.py /path/to/skill-or-skill.zip
```

The markdown output ends with a status and finding-count footer. JSON output includes a versioned `schema_version`, a redacted structural `frontmatter` summary, a stable `detected_root_relative` path for ZIP automation, explicit `requested_target` / `canonical_target` / alias fields, and a top-level `summary` object for release-gate and CI integrations. Schema 6 no longer emits raw frontmatter values.

## Compatibility notes

Use the inspector process exit code as the canonical machine signal for CI and release gates.

Choose a canonical host profile deliberately:

```bash
python3 -S scripts/inspect_skill_package.py /path/to/skill --target portable --json --strict
python3 -S scripts/inspect_skill_package.py /path/to/skill --target openai --json --strict
```

`portable` is a conservative shared baseline, not proof that every host-specific workflow will accept the package. The inspector's default 30 MB pre-open ZIP threshold is an explicitly configurable internal safety boundary, not a vendor upload-limit assertion. See [platform compatibility profiles](references/platform-compatibility.md) for documented limits and source links.

In strict mode:

- Exit code `0` means the inspected package passed strict checks.
- Exit code `1` means the input does not exist or could not be inspected in a basic way (for example, a missing path or an unreadable archive).
- Exit code `2` means one or more error-severity findings were present.

The top-level JSON `summary` object is retained for compatibility with existing automation that reads fields such as:

- `summary.status`
- `summary.strict_pass`
- `summary.error_count`
- `summary.finding_codes`

New integrations should still treat the process exit code as the source of truth for pass/fail decisions. Use `summary` for reporting, dashboards, or compatibility with older pipelines.

## Running self-tests

After any evaluator or script change, run both test layers. Start with the
contract and source-only suite:

```bash
python3 -S scripts/validate_audit_contract.py
python3 -S scripts/run_source_tests.py
```

Then build from a committed revision, source-prove the exact archive against
that revision, extract it, and run the tests that actually shipped:

```bash
python3 -S scripts/package_skill.py build --revision HEAD --output /tmp/skill-forge.zip
python3 -S scripts/package_skill.py verify /tmp/skill-forge.zip --json --source-repo .
python3 -S -m zipfile -e /tmp/skill-forge.zip /tmp/skill-forge-runtime
python3 -S /tmp/skill-forge-runtime/skill-forge/scripts/run_self_tests.py
```

Together, the suites use temporary valid, malformed, hostile, and release fixtures. They
need no network access or third-party packages. Source-only checks never ship in
the runtime, and the runtime runner has no source-helper import or skip path.

## Continuous integration

This repository includes a GitHub Actions workflow at `.github/workflows/self-tests.yml`.

Every Ubuntu, macOS, and Windows job runs on Python `3.9` and the latest stable
release. Each of the six matrix cells validates the contract, runs source-only
tests, builds the exact submitted commit, verifies archive integrity and local
Git source proof across both canonical profiles, extracts the archive, and
runs its packaged runtime suite. Pull requests check out the contributor's HEAD
instead of GitHub's temporary merge ref. Keep the workflow dependency-free;
every third-party GitHub Action is pinned to an immutable commit SHA.

## Versioning and changelog

Releases are tag-driven and use ASCII-only semantic-version tags:
`vMAJOR.MINOR.PATCH`.
`CHANGELOG.md` is the committed release record: add user-visible changes under
**Unreleased** as normal work lands, then use the local release command to
promote that section into the next version.

The command never pushes or publishes by itself. It requires clean `main` and
the highest reachable SemVer tag, runs the source and extracted-runtime gates,
then creates a one-file release commit with exact subject
`chore(release): <tag>` and an annotated local tag. It requires one visible,
exact changelog heading with a real calendar date matching the tagger date.
`HEAD`, reachable tags, and changelog bytes are rechecked after the gates;
replacement refs, unsafe inherited Git repository/config variables, hooks, and
automatic signing are disabled for the generated local commit and tag.

```bash
python3 -S scripts/release_skill.py patch --dry-run
TAG=<exact tag printed by the dry run>
python3 -S scripts/release_skill.py patch
python3 -S scripts/release_skill.py --verify-tag "$TAG"
git push --atomic origin main "$TAG"
```

Use `minor` or `major` instead of `patch` when the change warrants it. A version
does not change merely because source files change; it changes only when this
release command is intentionally run. The tag push starts the GitHub Release
workflow. These controls establish local metadata integrity, not cryptographic
author identity, release authorization, or remote-origin authenticity.

## Release-gate workflow

For install, publish, ship, or release-candidate decisions, use `references/audit-contract.json`, `references/release-gate-checklist.md`, and the Release Gate Review section from `references/report-template.md`. The complete G01–G23 gate matrix is the proof; the five-row executive summary cannot replace it.

A skill is release-ready only when every applicable required gate passes; a Critical gate failure is always disqualifying.

Treat the following as blockers unless the user explicitly requested only a draft review:

- Strict inspector failures.
- Likely bundled secrets.
- Unsafe archive structures.
- Missing or multiple `SKILL.md` entrypoints.
- Trusted official-validator Artifact Fail outcomes.
- Safety risks that could cause the agent to take unsafe, misleading, destructive, or unsupported actions.

Release readiness should mean something. Otherwise it is just ceremony wearing a badge.

An official validator must come from a trusted host installation, documented CLI,
or independently verified platform source outside the inspected package. A
target-bundled `validate`, `check`, `package`, or `official` script is an
untrusted self-test, never official evidence, and is not executed merely because
of its name. An unavailable optional validator is Not Applicable; dependency,
sandbox, and timeout failures are execution limitations rather than evidence
that the package is invalid. See [`references/validator-evidence.md`](references/validator-evidence.md).

## Independent evaluation

`scripts/verify_independent_evaluator.py` can run an exact candidate through a
separately installed, previously trusted Skill Forge runtime. It requires a
pretrusted SHA-256 for the evaluator's complete tree and an expected candidate
SHA-256; an optional inspector-file pin is supplemental. It executes scratch
copies under isolated Python with a credential-reduced environment and bounded
tree, file, time, and output limits, then checks both canonical profiles.

```bash
python3 -S scripts/verify_independent_evaluator.py \
  --evaluator-root /trusted/skill-forge \
  --archive /tmp/skill-forge.zip \
  --expected-evaluator-tree-sha256 "$TRUSTED_EVALUATOR_TREE_SHA256" \
  --expected-candidate-sha256 "$CANDIDATE_SHA256" \
  --json
```

The required evaluator pin must come from prior trusted evidence, not from the
candidate checkout during the same run. Inspector schema `6` is required;
stale schemas, missing provenance, timeouts, and output-limit failures are
**Not Assessed**, never Pass. Exit codes are `0` Pass, `2` Fail, and `3` Not
Assessed. The helper does not provide an OS filesystem sandbox or network
sandbox and makes no continuous-immutability claim. A Pass is independent
strict-inspection evidence, not a complete release verdict.

The sole exception is the one-time schema 5 to 6 bootstrap for `v2.0.0`. It
requires a pretrusted schema-5 evaluator plus both
`--bootstrap-schema-transition 5:6` and `--bootstrap-release-tag v2.0.0`.
The reduced report excludes raw schema-5 frontmatter and labels a successful
run **bootstrap transition evidence**, not an independent schema-6 pass. It can
support G09 only with every contract requirement and cannot be reused after
`v2.0.0`.

## Safety notes

Treat uploaded archives and bundled scripts as untrusted input.

The secret scan is heuristic and non-exhaustive. A clean scan does not prove that no secrets exist.

The inspector flags high-confidence destructive behavior in bundled shell,
PowerShell, Windows batch, Python, and JavaScript/TypeScript scripts as
`script_dangerous_command`. Examples include remote content piped or evaluated
into a shell and recursive deletion of root, home, or system paths. The scan is
heuristic and non-exhaustive; a clean result is not proof of safety.

Do not run bundled scripts that appear destructive, credential-harvesting, network-dependent, or unrelated to validation.

Before running any target-bundled self-test, review its purpose and provenance
and use copied or synthetic inputs with network default-deny, credentials
absent, source read-only, scratch-only writes, bounded process/time/memory, and
external side effects forbidden. If any control is unavailable, do not run the
self-test; required evidence is Not Assessed.

Keep `SKILL.md` compact. Move detailed rubrics, examples, schemas, and extended guidance into directly linked files under `references/`.

The skill should help the agent load the right amount of context at the right time — not bury it under a pile of instructions and then act surprised when behavior drifts.

## Packaging

A distributable Skill ZIP should contain the `skill-forge/` directory as the archive root entry.

Build from a committed revision, not the mutable working tree. The verified
runtime archive is the installation source of truth: installed-runtime parity
means the installed files match that exact verified archive, not the broader
repository checkout. The package command admits only the runtime skill surface:
`SKILL.md`, `README.md`, `LICENSE`, `agents/`, `references/`, plus the inspector,
package verifier, portable-path policy, runtime-manifest helper, and runtime
regression suite. It intentionally excludes `.github/`, Git metadata, generated
output, caches, and source-only helpers including `generate_release_notes.py`,
`release_metadata.py`, `release_skill.py`, `run_source_tests.py`, and
`verify_independent_evaluator.py`.

```bash
python3 -S scripts/package_skill.py build --revision HEAD --output /tmp/skill-forge.zip
python3 -S scripts/package_skill.py verify /tmp/skill-forge.zip --json --source-repo .
```

Every archive contains a generated `runtime-manifest.json` that records the exact
committed source identity, selector policy, file modes, sizes, and SHA-256
digests. The ZIP uses a canonical deterministic byte layout. Verification
separately reports archive integrity, local Git source proof, and their shared
manifest-digest binding, then runs strict inspection for Portable and OpenAI.
`--source-repo` proves consistency with
the caller-selected local Git object graph; it does not prove remote origin,
signatures, author identity, or release authorization. See the
[runtime manifest schema](references/runtime-manifest-schema.md).

The `.github/workflows/release-skill.yml` workflow has a read-only validation
job and a separate publication job. Validation builds the committed `HEAD`
runtime archive, runs source-only tests, proves the archive against the checkout,
extracts it, runs the packaged runtime suite, verifies both canonical
profiles, and writes `skill-forge.zip.sha256` beside the ZIP. On a tag, it also
verifies the exact release metadata and generates deterministic notes from the
tagged commit's changelog blob rather than mutable working-tree text. Only the
dependent publication job receives `contents: write`; it attaches both the ZIP
and its SHA-256 checksum to the GitHub Release, refusing to create a release if
the remote tag is absent. A manual dispatch stops after read-only validation
and uploads the ZIP plus checksum as a workflow artifact.

## License

This repository is distributed under the MIT License.

Update the copyright holder in `LICENSE` if your organization requires a different owner or license.
