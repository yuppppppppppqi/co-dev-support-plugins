# co-dev-support-plugins — notes for Claude sessions

## What this is

A Claude Code plugin marketplace repository for CortexLab's collaborative
development tooling. `.claude-plugin/marketplace.json` is the marketplace
manifest; each plugin lives self-contained under `plugins/<name>/`.

## Invariants (do not break)

1. **Plugins are self-contained.** A plugin under `plugins/<name>/` must not
   depend on files outside its own directory — it needs to work if copied
   into another repository (e.g. CortexLab's own repo) on its own.
2. **Agents stay read-only unless a command's flow explicitly writes.**
   `duplicate-detector` (cortexlab-dup-guard) only investigates and reports;
   Linear/GitHub writes happen only in the command-level flow, never inside
   the investigating subagent.
3. **Guards are mechanical, not prompt-based.** Where a plugin needs to
   enforce a workflow step (e.g. "check for duplicates before creating an
   issue"), enforce it with a PreToolUse hook that inspects real state (a
   recorded marker, a file, a git ref) — never rely on the agent remembering
   to ask.
4. **Missing data is reported as unverified, never silently passed.** If a
   data source (Linear MCP, GitHub, git fetch) is unavailable, say so in the
   report instead of treating it as "no duplicates found".

## Layout

```
.claude-plugin/marketplace.json   # marketplace manifest (plugins array)
plugins/
  cortexlab-dup-guard/            # duplicate-development guard
    .claude-plugin/plugin.json
    agents/duplicate-detector.md
    commands/{dup-check,backlog-add,start-task}.md
    hooks/{dup-guard.sh,hooks.json}
    scripts/mark-checked.sh
SETUP.md                          # end-user setup & usage guide (Japanese)
```

## Session hand-off log

Record decisions and insights at the end of each working session
(newest first).

- **2026-07-21** Repository cleanup: this repo previously had Hawkeye
  (an unrelated investment-decision-system project) checked in alongside
  the cortexlab-dup-guard plugin, from working in the same local clone.
  Removed `hawkeye/`, `docs/`, `tests/`, `pyproject.toml`, and
  `.claude/skills/hawkeye-run/`; rewrote README.md/CLAUDE.md/.gitignore for
  this repo's actual purpose (plugin marketplace only). Hawkeye's own
  history is preserved in this repo's git log (commits before
  `b665725`) but is no longer relevant to future work here.
- **2026-07-20** Added `plugins/cortexlab-dup-guard/` + root
  `.claude-plugin/marketplace.json` — a Claude Code plugin for CortexLab
  that detects duplicate development across the Linear backlog and GitHub
  branches (duplicate-detector agent, `/dup-check` `/backlog-add`
  `/start-task` commands, PreToolUse hook gating Linear issue creation on a
  recorded check).
