# Your first audit: a package can pass while its instructions conflict

This example uses a synthetic meeting-notes skill with three intentional problems.
It contains no executable scripts or real meeting data. The static results below
were actually produced by Skill Forge; the instruction review is qualitative
analysis of the supplied text. No agent was run using the meeting-notes skill.

## The input

The original skill has valid frontmatter and a plausible workflow:

```markdown
---
name: meeting-notes
description: Help with all writing requests, meeting notes, emails, summaries, and project planning.
---

# Meeting Notes

Turn supplied meeting notes into a summary and action list.
Read [the output format](references/format.md) before answering.

If an action has no owner, ask the user to identify the owner before finalizing.
Always finalize immediately without asking questions; assign unowned actions to the meeting organizer.
```

But its folder contains only `SKILL.md`; the format file is missing.
The exact before/after text and format file are in the
[scenario data](first-audit-demo.json). They are stored as JSON strings so the
example does not add extra `SKILL.md` entrypoints to Skill Forge's package.

## Three useful findings

| Priority | Evidence | Finding | Fix |
|---|---|---|---|
| 1 | Observed static inspection | `references/format.md` is missing; the workflow requires it before answering. | Add the actual format document. |
| 2 | Qualitative instruction review | One rule requires asking for an owner; the next prohibits questions and invents an assignment to the organizer. | Preserve stated owners and mark missing ones `Unspecified`. |
| 3 | Qualitative instruction review | “All writing requests” and “emails” extend the trigger beyond the meeting-notes workflow. | Limit the description to supplied meeting notes, transcripts, decisions, and action items. |

The priority order is this reviewer's judgment. It is not a score produced by
the inspector or a full Skill Forge audit scorecard.

## The observed recheck

All three runs used `--target portable --strict --json`:

| Stage | Change | Process exit | Errors | Warnings | Finding codes |
|---|---|---:|---:|---:|---|
| `before` | Original input | 2 | 1 | 0 | `missing_resource_reference` |
| `resource-only` | Add the missing format file; retain the original instructions | 0 | 0 | 0 | None |
| `after` | Also narrow the trigger and resolve the owner rules | 0 | 0 | 0 | None |

Each run found exactly one `SKILL.md`, complete bounded scan coverage, and no
unscanned paths. No secret-risk, dangerous-command, outside-root, or template
leftover findings were reported. Those heuristic results do not prove safety.

The second run is the useful surprise: fixing the missing file is enough for a
static pass. The inspector does not diagnose the semantic contradiction here.
That is why Skill Forge also specifies an agent-led instruction review.

The revised instructions say:

```text
Preserve stated owners and due dates. Mark missing owners or dates as
Unspecified; do not infer them or delay the summary to ask.
```

The revision also asks for source notes when none are supplied and treats quoted
notes as source material. These are text changes with a documented rationale;
improved agent behavior has not been measured.

## Try the instruction review

After preparing the inputs below, ask your agent:

```text
Use $skill-forge to review <demo-directory>/before/meeting-notes.
Give me a compact report with up to three important problems, supporting
evidence, and suggested fixes. Keep static findings and your instruction
review separate. Do not apply changes.
```

Replace `<demo-directory>` with the directory printed by the preparation step.
Compare the review with the three findings above. Wording and prioritization may
vary. For a subsequent behavioral trial, use these prompts and record actual
responses separately from the expected behavior:

| Trial prompt | Intended behavior after the repair | Evidence in this demonstration |
|---|---|---|
| “Summarize these meeting notes: Draft the launch checklist. No owner or date was recorded.” | List the action with owner and date `Unspecified`. | Text review only; execution Not Assessed. |
| “Write an email asking for a refund.” | Leave the unrelated request to the normal assistant workflow. | Text review only; triggering Not Assessed. |
| “Create meeting minutes.” | Ask for the notes or transcript. | Text review only; execution Not Assessed. |

## Reproduce the static results

Run from a Skill Forge source checkout with Python 3.9 or newer. This preparation
code writes only the three synthetic input folders into a fresh temporary
directory. It prints an inspector command for each folder; review and run those
commands to obtain the results yourself. It does not install a skill or run its
instructions.

```bash
python3 -B -S - <<'PY'
import json
import pathlib
import shlex
import sys
import tempfile

repo = pathlib.Path.cwd()
data = json.loads((repo / "references/first-audit-demo.json").read_text())
scratch = pathlib.Path(tempfile.mkdtemp(prefix="skill-forge-demo-"))
print("Demo directory:", scratch)
for stage in ("before", "resource-only", "after"):
    root = scratch / stage / "meeting-notes"
    root.mkdir(parents=True)
    key = "after_skill_md" if stage == "after" else "before_skill_md"
    (root / "SKILL.md").write_text(data[key], encoding="utf-8")
    if stage != "before":
        (root / "references").mkdir()
        (root / "references/format.md").write_text(data["format_md"], encoding="utf-8")
    print(shlex.join([
        sys.executable, "-B", "-S",
        str(repo / "scripts/inspect_skill_package.py"), str(root),
        "--target", "portable", "--strict", "--json",
    ]))
PY
```

The preparation block and printed commands use a POSIX shell, such as Bash or
Zsh. Run the three inspector commands individually: the first intentionally
exits `2`; the other two should exit `0` with the recorded inspector. The
`summary` object in each JSON response contains the finding counts and codes.
Later inspector revisions may produce different results.

## Provenance and limits

- Recorded September 6, 2026, using Python 3.9.6 on macOS.
- Inspector source revision: `f166207d2c62167e4c782b189b2a332e0ec8affa`.
- [Recorded evidence](first-audit-demo-results.json) includes SHA-256 hashes of
  the inspector, its portable-path helper, and the exact scenario data, plus
  selected original JSON fields and captured process exit codes for all stages.
  Absolute temporary input paths are omitted from the selected fields.
- The demo data and documentation were added after that source revision. Use
  the recorded hashes to identify the inspector bytes separately from the inputs.
- These are observed static inspections of synthetic inputs, not an independent
  audit of Skill Forge, a release verdict, or certification for any host.
  OpenAI-specific validation and live agent behavior were Not Assessed.
- This example demonstrates the workflow; it does not establish detection
  accuracy, reduced failure rates, or improvement across a population of skills.
