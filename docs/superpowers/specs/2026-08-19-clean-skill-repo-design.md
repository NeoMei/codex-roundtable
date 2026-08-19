# Clean Skill Repository Design

## Goal

Make the public repository show only the Roundtable skill, its README, and the minimum Codex plugin manifest required to keep it installable as a skills-only plugin.

## Final tracked tree

The only tracked paths are:

- `README.md`
- `.codex-plugin/plugin.json`
- `skills/roundtable/SKILL.md`
- `skills/roundtable/agents/openai.yaml`
- `skills/roundtable/references/minutes-format.md`
- `skills/roundtable/references/setup-wizard.md`

The `agents/` and `references/` directories are part of the skill bundle, not auxiliary project documentation.

## Cleanup

Delete `.gitignore`, `LICENSE`, `docs/`, and `tests/` from the tracked tree. Remove README links and claims that depend on deleted files. Remove the manifest license declaration because the standalone license file will no longer be published.

## Verification

Verify the final path allowlist exactly, validate `skills/roundtable` with the Skill validator, validate the repository with the plugin validator, and inspect the merged `main` tree after publication.
