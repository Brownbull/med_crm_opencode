# Redundant / Temporary Files Safe To Remove

Purpose: Consolidate experimental / diagnostic configs and probe artifacts created during setup debugging. Keep only the canonical runtime assets.

## Keep (Do NOT delete)

- `opencode.json` (active canonical config)
- `.opencode/` directory (agents + commands)
- `PROJECT_OBJECTIVE.md`
- `example.md`
- `setup.md`
- `config_explain.md`

## Candidates For Deletion

| File | Category | Created For | Current Status | Safe Removal Rationale |
|------|----------|-------------|----------------|------------------------|
| `opencode.min.json` | Minimal baseline config | Verified engine starts (build+plan) | Superseded by full `opencode.json` | Provides no additional runtime capability |
| `opencode.cleaned.json` | Sanitized full config (anthropic models, no commands block) | Isolation test | Merged concepts into canonical config | Redundant duplicate of active settings |
| `opencode.full.json` | Original full config snapshot | Reference / backup | Superseded, diverges from final decisions | Historical only; retain in VCS history if needed |
| `opencode.test.json` | Duplicate key experiments | Schema edge testing | Invalid / obsolete pattern | Causes potential confusion |
| `opencode.schema_test.json` | Plural `agents` key experiment | Proven invalid (schema rejection) | Known-bad example | Should not linger; could mislead |
| `opencode.ui.json` | Probe promoting subagents to primary | UI enumeration investigation | Not needed; reverts semantic roles | Encourages anti-pattern (misusing modes) |
| `khujta/010_evolved_agents/agents/*.md` (if not referenced) | Legacy / alternate prompts | Earlier evolution set | Not loaded by current config (no instructions pointing there) | Dead assets unless planning future variants |

## Optional Archive Instead of Deleting

If you prefer retaining for future reference:

```bash
mkdir -p _archive/config_runs
git mv opencode.min.json opencode.cleaned.json opencode.full.json opencode.test.json opencode.schema_test.json opencode.ui.json _archive/config_runs/ 2>/dev/null || mv opencode.min.json opencode.cleaned.json opencode.full.json opencode.test.json opencode.schema_test.json opencode.ui.json _archive/config_runs/
```

Add legacy agent variants (optional):
```bash
mkdir -p _archive/legacy_agents
git mv khujta/010_evolved_agents/agents _archive/legacy_agents/ 2>/dev/null || mv khujta/010_evolved_agents/agents _archive/legacy_agents/
```

## Direct Delete Command (Irreversible)

Only after confirming archive not needed:
```bash
rm -f opencode.min.json opencode.cleaned.json opencode.full.json opencode.test.json opencode.schema_test.json opencode.ui.json
rm -rf khujta/010_evolved_agents/agents
```

## Verification After Cleanup

1. `opencode --print-logs` (or just `opencode`) still starts (no schema errors).
2. `opencode run "/reprioritize reason=scheduled"` succeeds (proves VIVA accessible).
3. `.opencode/command` slash commands still recognized.

## Rationale Summary

All listed files are superseded experimental variants used solely for debugging:
- Plural key experiment invalid.
- Mode promotion probe unnecessary.
- Minimal / cleaned / full versions replaced by single authoritative `opencode.json`.

Removing them reduces configuration ambiguity and onboarding friction.
