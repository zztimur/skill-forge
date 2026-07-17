# Platform Compatibility Profiles

Use one or more explicit inspector profiles for the hosts where a Skill will
run. Run one inspector invocation and preserve one compatibility result per
selected canonical profile. A passing profile is compatibility evidence for
that profile only; it does not prove that every other host, runtime, permission
model, or upload path accepts the same package.

## Source verification

Verified on **2026-07-15** against these primary sources:

- [OpenAI: Skills in ChatGPT](https://help.openai.com/en/articles/20001066-skills-in-chatgpt) — OpenAI Skills follow the Agent Skills open standard and support uploaded Skill bundles. The public article does not publish a per-Skill archive byte limit.
- [Legacy Code: Extend Legacy with skills](https://code.legacy.com/docs/en/skills) — `SKILL.md` is required; all frontmatter fields are optional, `description` is recommended, and the combined `description` plus `when_to_use` listing is truncated at 1,536 characters. It documents the invocation-control fields accepted by the `legacy-code` profile.
- [Legacy API: Using Agent Skills with the API](https://platform.legacy.com/docs/en/build-with-legacy/skills-guide) — requires root `SKILL.md`, `name`, and `description`; requires a common top-level directory whose name matches `name` case- and underscore-insensitively; reserves `provider` and `legacy`; caps total upload size below 30 MB.
- [Legacy.ai: How to create custom skills](https://support.legacy.com/en/articles/12512198-how-to-create-custom-skills) — requires `name` (human-friendly, up to 64 characters) and `description` (up to 200 characters), and directs authors to ZIP one Skill folder as the archive root.

The 30 MB default `inspector_input_zip_limit_bytes` shown by Skill Forge is an
explicitly configurable internal pre-open resource/safety boundary. It is not
an OpenAI, Legacy Code, Legacy API, or Legacy.ai upload-limit assertion. The
only product byte limit currently encoded here is Legacy API's documented 30 MB
total-upload limit.

The `openai` profile also retains the `agents/openai.yaml` metadata checks.
Those are a Skill Forge OpenAI tooling/integration policy, kept separately from
the public product constraints above; the verified OpenAI Help article does not
publish a per-Skill archive byte limit.

## Contract source of truth

[`audit-contract.json`](audit-contract.json) is the machine-readable source of
truth for the five audit target profiles. Each `target_contracts` record carries
its verification date, source URL, host-verified rule keys, Skill Forge policy
recommendations, frontmatter expectations, layout rule, and known product upload
limit. A `null` product limit is an explicit **Not Applicable** host-limit
result, not permission to treat an internal inspector safety limit as a host
requirement. A `null` value means no documented product upload limit exists for
that target in the verified source.

## Canonical targets

| Target | What is enforced | What it does not prove |
|---|---|---|
| `portable` | Skill Forge's conservative shared baseline: root `SKILL.md`, lowercase hyphen-case `name`, description, matching folder, 64-character name ceiling, and Legacy-reserved-name avoidance. Present OpenAI UI metadata is conditional. | Every product-specific upload, runtime, invocation, or UI workflow. |
| `openai` | Skill Forge's shared manifest baseline plus the conditional OpenAI metadata contract in `agents/openai.yaml`; an optional `default_prompt` is compared with the declared frontmatter name, including for a flat ZIP extraction. Legacy-only controls are rejected. | A public OpenAI archive byte limit; none was found in the verified public source. |
| `legacy-code` | Current Legacy Code frontmatter: all fields optional, `description` recommended, optional invocation controls, typed controls, and the 1,536-character listing warning. | Legacy API or Legacy.ai upload behavior. It is filesystem-based, not an upload profile. |
| `legacy-api` | Required lowercase hyphen-case `name` and non-empty `description`, 64/1,024-character limits, Legacy/other provider reserved-name restriction, matching common root directory, single root folder in ZIPs, and 30 MB documented upload maximum. | Legacy.ai's shorter custom-Skill description rule. |
| `legacy-ai` | Required human-friendly `name` (up to 64 characters), non-empty description (up to 200 characters), matching package folder using non-empty Unicode-normalized alphanumeric comparison keys, and one root folder in ZIPs. | Legacy API's lowercase-hyphen, reserved-name, or 30 MB rules. |

`--target legacy` remains a temporary, documented alias for `--target
legacy-code`. It returns the same findings as `legacy-code` and adds JSON alias
metadata plus a deprecation note. Update scripts to use `legacy-code`.

“Legacy” without a Code, API, or Legacy.ai context is not a profile request.
Ask which surface is intended; if that remains unknown, use `portable` and
state that Legacy-specific validation is Not Assessed. Do not silently map the
generic term to the deprecated `legacy` alias.

When a local Legacy Code installation offers `legacy plugin validate`, that
command validates plugin or marketplace manifests. It is not standalone
filesystem-Skill validation: use it only when a relevant `.legacy-plugin/plugin.json`
or marketplace manifest exists. Otherwise record plugin validation as Not
Applicable and keep `legacy-code` profile checks and live discovery/invocation
evidence separate.

## Commands

```bash
python3 -S scripts/inspect_skill_package.py /path/to/skill --target portable --json --strict
python3 -S scripts/inspect_skill_package.py /path/to/skill --target openai --json --strict
python3 -S scripts/inspect_skill_package.py /path/to/skill --target legacy-code --json --strict
python3 -S scripts/inspect_skill_package.py /path/to/skill.zip --target legacy-api --json --strict
python3 -S scripts/inspect_skill_package.py /path/to/skill.zip --target legacy-ai --json --strict
```

For a Skill Forge release artifact, use the package verifier. It verifies the
canonical embedded runtime manifest and runs the five canonical profiles,
never the deprecated alias. Archive-only verification does not prove the
declared Git source:

```bash
python3 -S scripts/package_skill.py verify /path/to/skill-forge.zip --json
```

Archive-only verification may pass the package command, but it cannot Pass the
Skill Forge G23 release gate. To satisfy G23, provide a caller-selected local
source repository and require the separate commit/tree/blob proof and
manifest-digest binding:

```bash
python3 -S scripts/package_skill.py verify /path/to/skill-forge.zip \
  --json --source-repo /path/to/skill-forge-source
```

If that local source evidence is unavailable, G23 is **Not Assessed** and the
Skill Forge release gate is blocked. The proof establishes consistency with the selected local Git object graph,
not remote provenance, tag authenticity, or official host compatibility. The
five profile inspections, local source proof, independently pinned Skill Forge
evidence, and official platform validation remain separate claims.

## Reporting rules

- State every requested surface and selected canonical profile in the report.
- Preserve the authoritative findings matrix and release verdict independently
  for every selected profile. An overall roll-up must not hide a failed or Not
  Assessed member.
- Treat `portable` as a conservative starting point, not an all-host
  certification.
- Keep platform findings separate from Skill Forge safety findings. For
  example, a configured inspector refusal is an inspection limitation; a
  `legacy-api` package over 30 MB is a documented product-limit failure.
- Legacy Code ignores `agents/openai.yaml`; the other Legacy profiles do not
  claim OpenAI UI metadata compatibility.
