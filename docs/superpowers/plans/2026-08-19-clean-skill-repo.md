# Clean Skill Repository Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Publish a minimal skills-only plugin repository containing only the README, plugin manifest, and Roundtable skill bundle.

**Architecture:** Apply an exact tracked-path allowlist. Preserve every file required by the Roundtable skill and the Codex plugin manifest while removing planning, acceptance, license, and repository-maintenance files.

**Tech Stack:** Markdown, JSON, Codex Skill validator, Codex plugin validator, Git, GitHub CLI.

## Global Constraints

- Keep `README.md`, `.codex-plugin/plugin.json`, and every tracked file under `skills/roundtable/`.
- Delete every other tracked path.
- Do not alter the Roundtable workflow behavior.
- Publish the verified result to GitHub `main`.

---

### Task 1: Apply and publish the exact repository allowlist

**Files:**
- Modify: `README.md`
- Modify: `.codex-plugin/plugin.json`
- Delete: `.gitignore`
- Delete: `LICENSE`
- Delete: `docs/`
- Delete: `tests/`
- Preserve: `skills/roundtable/**`

**Interfaces:**
- Consumes: the approved final tracked-path allowlist.
- Produces: a validated skills-only Codex plugin repository whose public `main` tree contains only the allowlisted paths.

- [ ] **Step 1: Run the tracked-path allowlist assertion and verify it fails**

```bash
git ls-files | awk '
  $0 == "README.md" { next }
  $0 == ".codex-plugin/plugin.json" { next }
  index($0, "skills/roundtable/") == 1 { next }
  { print; bad = 1 }
  END { exit bad }
'
```

Expected: FAIL and print the currently tracked planning, test, license, and maintenance paths.

- [ ] **Step 2: Delete disallowed tracked paths and remove stale README/manifest claims**

Use `apply_patch` for `README.md` and `.codex-plugin/plugin.json`. Remove the tracked `.gitignore`, `LICENSE`, `docs/`, and `tests/` paths explicitly.

- [ ] **Step 3: Run the exact allowlist assertion again**

Run the Step 1 command.

Expected: PASS with no output.

- [ ] **Step 4: Validate the preserved skill and plugin package**

```bash
python3 "$skill_creator_root/scripts/quick_validate.py" skills/roundtable
python3 "$plugin_creator_root/scripts/validate_plugin.py" .
git diff --check
```

Expected: `Skill is valid!`, plugin validation passes, and `git diff --check` prints nothing.

- [ ] **Step 5: Commit, push, merge, and verify GitHub main**

```bash
git commit -m "chore: publish clean roundtable skill repository"
git push -u origin agent/clean-skill-repo
gh pr create --title "Publish clean Roundtable skill repository"
gh pr merge --merge
git fetch origin
git ls-tree -r --name-only origin/main
```

Expected: the pull request is merged and `origin/main` contains only the exact allowlist.
