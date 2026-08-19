# codex-roundtable

A skills-only Codex plugin for guided multi-agent roundtable discussions. Configure each member's role, persona, and available model; run speakers in a fixed order over multiple rounds; then export structured Markdown minutes.

Inspired by [NeoMei/dsh-roundtable](https://github.com/NeoMei/dsh-roundtable), rewritten for Codex-native subagents without DeepSeek Harness dependencies.

## Features

- Complete topic and member setup wizard.
- One real Codex subagent per member execution.
- Runtime-safe model selection with host-default fallback.
- Ordered multi-round discussion and user interjections.
- Neutral host summaries and verified Markdown export.
- Plain-chat fallback when structured input cards are unavailable.

## Requirements

- Installation commands below support macOS and Linux and require Bash.
- A current Codex release with subagents enabled.
- Git, the Codex CLI, `rsync`, and `rg` on `PATH`.
- A writable workspace to save minutes. Without one, the plugin returns Markdown in chat.
- Installed models and permissions are determined by the active Codex host.
- Python 3 with [PyYAML](https://pyyaml.org/) installed for the bundled validation scripts.

## Install for local development

This repository root is the distributable plugin root. Before installing, run the legacy-skill check below. For a first installation, use the bundled plugin-creator workflow to generate the exact personal-marketplace target. If that target already exists, inspect it and use the update flow below; do not force or overwrite an unrelated directory.

```bash
(
  set -euo pipefail
  source_plugin_root="$(git rev-parse --show-toplevel)"
  managed_plugin_root="$HOME/plugins/roundtable"
  source_manifest="$source_plugin_root/.codex-plugin/plugin.json"
  plugin_creator_root="${CODEX_HOME:-$HOME/.codex}/skills/.system/plugin-creator"
  test "$managed_plugin_root" = "$HOME/plugins/roundtable"
  test ! -e "$managed_plugin_root"
  test ! -L "$managed_plugin_root"
  python3 -c 'import json,sys; assert json.load(open(sys.argv[1]))["name"] == "roundtable"' "$source_manifest"
  python3 "$plugin_creator_root/scripts/create_basic_plugin.py" roundtable --with-skills --with-marketplace
  managed_manifest="$managed_plugin_root/.codex-plugin/plugin.json"
  test -f "$managed_manifest"
  test ! -e "$managed_plugin_root/.git"
  python3 -c 'import json,sys; assert all(json.load(open(path))["name"] == "roundtable" for path in sys.argv[1:])' "$source_manifest" "$managed_manifest"
  rsync -a --delete --delete-excluded \
    --exclude '/.git' \
    --exclude '/.superpowers/' \
    --exclude '/docs/superpowers/' \
    "$source_plugin_root/" \
    "$managed_plugin_root/"
  test ! -e "$managed_plugin_root/hooks"
  test ! -e "$managed_plugin_root/.mcp.json"
  test ! -e "$managed_plugin_root/.app.json"
  if rg -n '"(mcpServers|apps)"[[:space:]]*:' "$managed_manifest"; then exit 1; fi
  python3 "$plugin_creator_root/scripts/validate_plugin.py" "$managed_plugin_root"
  marketplace_name="$(python3 "$plugin_creator_root/scripts/read_marketplace_name.py")"
  codex plugin add "roundtable@$marketplace_name"
)
```

Treat `~/plugins/roundtable` as generated installation state; source changes belong in this checkout. Do not hand-edit marketplace JSON. After installation, use a fresh Codex task to verify that `$roundtable` resolves to this plugin's skill.

For subsequent local updates, synchronize the checkout, refresh the managed copy's cachebuster, validate it, read the marketplace name, and reinstall:

```bash
(
  set -euo pipefail
  source_plugin_root="$(git rev-parse --show-toplevel)"
  managed_plugin_root="$HOME/plugins/roundtable"
  source_manifest="$source_plugin_root/.codex-plugin/plugin.json"
  managed_manifest="$managed_plugin_root/.codex-plugin/plugin.json"
  plugin_creator_root="${CODEX_HOME:-$HOME/.codex}/skills/.system/plugin-creator"
  test "$managed_plugin_root" = "$HOME/plugins/roundtable"
  test ! -L "$managed_plugin_root"
  test -f "$managed_manifest"
  test ! -e "$managed_plugin_root/.git"
  python3 -c 'import json,sys; assert all(json.load(open(path))["name"] == "roundtable" for path in sys.argv[1:])' "$source_manifest" "$managed_manifest"
  rsync -a --delete --delete-excluded \
    --exclude '/.git' \
    --exclude '/.superpowers/' \
    --exclude '/docs/superpowers/' \
    "$source_plugin_root/" \
    "$managed_plugin_root/"
  test ! -e "$managed_plugin_root/hooks"
  test ! -e "$managed_plugin_root/.mcp.json"
  test ! -e "$managed_plugin_root/.app.json"
  if rg -n '"(mcpServers|apps)"[[:space:]]*:' "$managed_manifest"; then exit 1; fi
  python3 "$plugin_creator_root/scripts/update_plugin_cachebuster.py" "$managed_plugin_root"
  python3 "$plugin_creator_root/scripts/validate_plugin.py" "$managed_plugin_root"
  marketplace_name="$(python3 "$plugin_creator_root/scripts/read_marketplace_name.py")"
  codex plugin add "roundtable@$marketplace_name"
)
```

Use a fresh Codex task after every install or update so Codex discovers the refreshed skill.

### Legacy skill migration

An older standalone `roundtable` skill may still depend on DeepSeek Harness tools. Inspect both standard standalone roots and report every visible entry:

```bash
codex_home_root="${CODEX_HOME:-$HOME/.codex}"
legacy_roundtable_roots=(
  "$HOME/.agents/skills/roundtable"
  "$codex_home_root/skills/roundtable"
)
for legacy_roundtable_root in "${legacy_roundtable_roots[@]}"; do
  if test -f "$legacy_roundtable_root/SKILL.md"; then
    printf 'standalone roundtable skill found: %s\n' "$legacy_roundtable_root"
    rg -n 'roundtable_models|roundtable_title|ask_user_question' "$legacy_roundtable_root/SKILL.md" || true
  fi
done
```

Inspect every reported standalone `roundtable` entry. Disable each conflicting exact skill directory in `~/.codex/config.toml` or move it outside Codex skill roots before installing this plugin. Do not overwrite or delete it silently.

Example disable entry:

```toml
[[skills.config]]
path = "/absolute/path/to/the/legacy/roundtable"
enabled = false
```

The `path` value must be the exact legacy skill folder containing `SKILL.md`, not the `SKILL.md` file itself. Restart or refresh Codex after changing skill configuration. In a fresh task, verify that `$roundtable` resolves to this plugin's skill.

## Usage

Explicit:

```text
$roundtable Discuss whether we should split this service into independent deployments.
```

Natural language:

```text
圆桌讨论：这个产品是否应该转向企业市场？
```

Member model choices are limited to identifiers explicitly exposed by the active Codex spawn tool; otherwise members use `Host default (no model override)`. The plugin makes no model-equivalence claim across host and member tasks.

## Output

Completed minutes are written to `roundtable-minutes/<topic-slug>-YYYY-MM-DD.md`. Existing files are never overwritten; numeric suffixes are added on collisions.

## Limitations

- Member analysis-only behavior is an instruction boundary, not a separate hard sandbox.
- Codex may consolidate member output in the main response; inspectable subagent threads remain the original source.
- App-restart and in-flight request recovery are best-effort.
- Host-specific model failure branches may be unavailable to test on every installation.

## Development validation

```bash
plugin_creator_root="${CODEX_HOME:-$HOME/.codex}/skills/.system/plugin-creator"
skill_creator_root="$(dirname "$plugin_creator_root")/skill-creator"
python3 "$skill_creator_root/scripts/quick_validate.py" skills/roundtable
python3 "$plugin_creator_root/scripts/validate_plugin.py" .
```

Run [tests/acceptance.md](tests/acceptance.md) in a fresh task before publishing.

The manifest declares `https://github.com/NeoMei/codex-roundtable`, but this local workflow does not create a GitHub repository or remote. Create and verify that repository URL separately before any public publication.

## License

MIT
