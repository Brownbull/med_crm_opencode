# Project Setup Guide (Start Here)

Purpose: Get all custom subagents (viva, syra, maka, qra, lua) and custom commands (/reprioritize, /plan-slice, /quality-pass, /arch-drift, /simulate-users) loaded so you can follow `example.md` without “only plan/build” or “unknown command” failures.

---

## 0. TL;DR (Fast Path)

```bash
# From repo root (contains opencode.json)
opencode            # or: opencode --config opencode.json

# Inside the interactive TUI (not your shell):
/agents             # expect build, plan, viva, syra, maka, qra, lua
/commands           # expect 5 custom commands
/reprioritize reason=scheduled
/plan-slice backlog_item_id=BK-001
```

If `/agents` shows only build & plan or commands are missing, proceed with full steps below.

---

## 1. Prerequisites

| Item | Requirement | Verify |
|------|-------------|--------|
| Runtime | Whatever your opencode build requires (Python 3.10+ or Node 18+) | `python --version` / `node --version` |
| CLI Installed | `opencode` in PATH | `opencode --help` |
| Working Directory | This repository root | `ls opencode.json` |

Example installs (choose only one matching your distribution):

```bash
# Python example
pip install opencode-ai

# Node example
npm install -g @opencode/cli
```

---

## 2. Repository Layout (Must Match)

```
opencode.json
PROJECT_OBJECTIVE.md
config_explain.md
example.md
.opencode/
  AGENTS.md
  agent/
    viva.md  syra.md  maka.md  qra.md  lua.md
  command/
    reprioritize.md  plan-slice.md  quality-pass.md  arch-drift.md  simulate-users.md
  output/
    (subdirectories for each command)
```

Quick verification:

```bash
test -d .opencode/agent && ls .opencode/agent
test -d .opencode/command && ls .opencode/command
```

---

## 3. Configuration Overview (`opencode.json`)

Key sections actually used by current CLI build:

```jsonc
{
  "agent": {
    "build": {...},
    "plan": {...},
    "viva": {...},
    "syra": {...},
    "maka": {...},
    "qra": {...},
    "lua": {...}
  },
  "instructions": [
    ".opencode/AGENTS.md",
    "PROJECT_OBJECTIVE.md"
  ],
  "commands": {
    "directory": ".opencode/command"
  }
}
```

Notes:

- Some releases auto-discover `.opencode/command`; others need the explicit `"commands"` block. Leave it in unless the CLI rejects it with a schema error, in which case remove the block and relaunch.
- The key is `agent` (singular) here; if your build expects `agents` (plural) you can duplicate and test. Only one should remain in the final working file.

---

## 4. Integrity Checks

1. No empty specification files:

```bash
find .opencode -type f -name "*.md" -exec awk 'NF==0 {e=1} END{exit e}' {} + || echo "All non-empty"
```

2. Command front matter must include a leading slash in `name:`:

```bash
grep -n "^name: /" .opencode/command/*.md
```

Expect 5 lines.

3. Agent markdown filenames must match agent IDs (e.g. `viva.md` ↔ `"viva"` in JSON).

---

## 5. Launch Interactive Session

```bash
opencode --config opencode.json   # explicit (first run)
# or simply:
opencode
```

Do NOT prefix slash commands with `!`; inside the TUI type them plainly.

Verification inside TUI:

```text
/agents
/commands
```

Expected:

- Agents: build, plan plus viva, syra, maka, qra, lua
- Commands: /reprioritize /plan-slice /quality-pass /arch-drift /simulate-users

---

## 6. First Functional Commands (Bridge to `example.md`)

Run in order (ignore early “missing signals” warnings):

```text
/reprioritize reason=scheduled
/plan-slice backlog_item_id=BK-001
/quality-pass scope=slice:BK-001 slice_ids=BK-001
/arch-drift scope=repo
/simulate-users scope=core max_scenarios=6
/reprioritize reason=new_signals new_signals=LUA-INIT
```

If `/quality-pass` returns BLOCKED provide:

1. Acceptance criteria (quantifiable)
2. UI templates or snapshots
3. Performance budgets (latency targets)
4. Canonical workflow steps (doctor, receptionist)

Then re-run `/quality-pass`.

---

## 7. Troubleshooting Matrix

| Symptom | Probable Cause | Resolution |
|---------|----------------|-----------|
| Only build & plan agents | Subagent specs not loaded (path issue) or wrong key (`agents` vs `agent`) | Ensure directory `.opencode/agent` exists; confirm JSON uses the correct expected key; relaunch |
| Custom commands missing | CLI needs explicit `commands.directory` | Keep `"commands": {"directory": ".opencode/command"}` unless schema rejects |
| Unknown command error | Missing front matter or missing leading slash in `name:` | Fix spec, restart |
| Help banner only (no prompt) | Non-interactive invocation (piped / script) | Launch plain `opencode` interactively |
| TypeError paths[2] (spaces in path) | Known 0.6.x bug with spaces in directory names | Use a no-space path or symlink: `ln -s "/path/with spaces/repo" ~/medcrm && cd ~/medcrm && opencode` |
| Intermittent agent visibility | Editing spec files during active session | Exit CLI, then edit, then relaunch |
| Schema error for `"commands"` | Build already auto-discovers | Remove the `"commands"` block and retry |
| Subagents ignored even though files exist | Wrong root key or malformed JSON trailing commas | Validate JSON; try switching `agent` → `agents` (one at a time) |

### UI Agent List Limitation (v0.6.1)

In CLI v0.6.1 the interactive `/agents` panel currently enumerates only the two built‑in primary agents (`Build`, `Plan`). Custom logical agents defined under the `agent` key (viva, syra, maka, qra, lua) are still loaded and *used* by command specs, but they do not appear in the list selector.

Verification that they are active:

```bash
# From project (or no‑space copy) root:
opencode run "/reprioritize reason=scheduled"
```

If the output narrative mentions “VIVA” or backlog reprioritization, the `viva` agent resolved correctly via the command spec’s `agent: viva` front matter.

Therefore:
- Treat missing entries in `/agents` list as a UI enumeration limitation, not a load failure.
- Do NOT attempt to force visibility by switching `mode` to `primary`; this is unnecessary. Restore `mode: subagent` in the config for semantic clarity.
- Future CLI releases may surface custom agents; until then rely on slash command execution for validation.

Practical test set (each should execute without “unknown command”):
```
/reprioritize reason=scheduled
/plan-slice backlog_item_id=BK-001
/quality-pass scope=slice:BK-001 slice_ids=BK-001
```
If these run, all supporting subagents are functionally integrated despite absence in `/agents` UI.

---

## 8. Clean Reload Procedure

```bash
# Exit TUI (Ctrl+C or /exit)
/exit
git checkout -- .opencode 2>/dev/null || true   # optional reset if versioned
opencode --config opencode.json
```

If still failing, create a minimal temporary config containing only build & plan to confirm baseline, then re-add subagents incrementally.

---

## 9. Determinism Check

Inside TUI:

```text
/plan-slice backlog_item_id=BK-001
/plan-slice backlog_item_id=BK-001
```

Outputs should match. Mismatch implies hidden mutable input (edit drift) — restart using Clean Reload.

---

## 10. Progress Checklist (Complete Before Deep Work)

- [ ] CLI installed & runnable (`opencode --help`)
- [ ] Repository layout matches required structure
- [ ] All agent markdown files non-empty
- [ ] All command markdown files have valid front matter & leading slash name
- [ ] `opencode.json` loads without schema errors
- [ ] Agents list shows 7 (build, plan + 5 subagents)
- [ ] Commands list shows 5 custom commands
- [ ] First `/reprioritize` recognized (even if missing signals)
- [ ] `/plan-slice` returns slice_matrix
- [ ] `/quality-pass` returns SUCCESS or clearly BLOCKED with actionable gaps
- [ ] Determinism check passes (repeat slice plan identical)
- [ ] Path without spaces (or symlink workaround applied)

---

## 11. Move On To `example.md`

Once the checklist is fully satisfied proceed to Section 2 of `example.md` and follow the operational loop.

---

## 12. Summary of Root Causes Previously Observed

| Issue | Root Cause |
|-------|------------|
| Missing subagents | Incorrect root key or directory not loaded |
| Missing commands | Absent or rejected `commands.directory` declaration |
| TypeError paths[2] | CLI bug with space-containing paths |
| "Unknown command" early | Front matter or discovery not complete before invocation |

This guide provides a deterministic path to a working interactive environment that is prerequisite for the workflow in `example.md`.
