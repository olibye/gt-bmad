# BMAD Beads Quick Reference

> **TL;DR:** Replace TodoWrite with beads. Workflows auto-create tasks with dependencies.

## Essential Commands

```bash
# Start workflow with auto beads
gt-bmad-workflow auto-start <workflow>

# Spawn agent with context
gt-bmad-spawn <agent> --mol <molecule-id>

# Find next task
bd --no-daemon ready --mol $BMAD_MOLECULE_ID

# Start task
bd --no-daemon update <task-id> --status in_progress

# Complete task
bd --no-daemon close <task-id>

# Check progress
bd --no-daemon mol progress <molecule-id>
```

## Common Workflows

### Start New Workflow

```bash
gt-bmad-workflow auto-start brainstorming
# → Creates molecule + step beads

gt-bmad-spawn analyst --mol hq-mol-abc
# → Spawns agent with context
```

### Resume Workflow

```bash
bd --no-daemon list --label workflow:brainstorming --status open
# → Find your molecule

bd --no-daemon ready --mol hq-mol-abc
# → See what's next

gt-bmad-spawn analyst --mol hq-mol-abc
# → Continue work
```

### Phase Management

```bash
gt-bmad-phase start 1        # Start phase
gt-bmad-phase complete 1     # Close phase gate
gt-bmad-phase advance        # Auto-advance to next
```

## Inside Polecat Session

```bash
# Your environment has:
echo $BMAD_MOLECULE_ID       # hq-mol-abc
echo $BMAD_WORKFLOW_NAME     # brainstorming

# Find work
bd --no-daemon ready --mol $BMAD_MOLECULE_ID

# Work on it
TASK=$(bd --no-daemon ready --mol $BMAD_MOLECULE_ID | head -2 | tail -1 | awk '{print $1}')
bd --no-daemon update $TASK --status in_progress

# ... do work ...

# Complete it
bd --no-daemon close $TASK

# Check what's next (automatically unblocked)
bd --no-daemon ready --mol $BMAD_MOLECULE_ID
```

## Troubleshooting

```bash
# Database locked?
bd --no-daemon <command>

# Out of sync?
bd sync --import-only

# Or use sandbox
bd --sandbox --allow-stale <command>

# See dependencies
bd --no-daemon dep tree <task-id>

# List all tasks
bd --no-daemon list --parent <mol-id>
```

## Cheat Sheet

| Action | Command |
|--------|---------|
| Start workflow | `gt-bmad-workflow auto-start <name>` |
| Spawn agent | `gt-bmad-spawn <agent> --mol <mol-id>` |
| Find tasks | `bd --no-daemon ready --mol <mol-id>` |
| Start task | `bd --no-daemon update <id> --status in_progress` |
| Close task | `bd --no-daemon close <id>` |
| Create subtask | `bd --no-daemon create "<title>" --parent <id>` |
| Check progress | `bd --no-daemon mol progress <mol-id>` |
| See dependencies | `bd --no-daemon dep tree <id>` |
| Start phase | `gt-bmad-phase start <num>` |
| Complete phase | `gt-bmad-phase complete <num>` |

## Agent Rules

**CRITICAL:** Agents must follow these rules in polecat sessions:

1. ✅ **CHECK** for `$BMAD_MOLECULE_ID` at start
2. ✅ **USE** `bd ready --mol $BMAD_MOLECULE_ID` to find work
3. ✅ **MARK** tasks in_progress before starting
4. ✅ **CLOSE** tasks after completing
5. ❌ **NEVER** use TodoWrite

## Examples

### Quick Workflow

```bash
# One-liner to start and work
gt-bmad-workflow auto-start brainstorming && \
  gt-bmad-spawn analyst --mol $(bd --no-daemon list --label workflow:brainstorming --status open --json | jq -r '.[0].id')
```

### Check All Open Workflows

```bash
# List all active workflows
for mol in $(bd --no-daemon list --type epic --status open --json | jq -r '.[].id'); do
  echo "=== $mol ==="
  bd --no-daemon mol progress $mol
done
```

### Bulk Close Completed Steps

```bash
# Close all in_progress tasks in molecule
bd --no-daemon list --parent hq-mol-abc --status in_progress --json | \
  jq -r '.[].id' | \
  xargs -I {} bd --no-daemon close {}
```

## Full Lifecycle (Copy-Paste)

```bash
# 1. Start workflow
gt-bmad-workflow auto-start brainstorming
# Save molecule ID from output

# 2. Export for convenience
export MOL=hq-mol-abc

# 3. Spawn agent
gt-bmad-spawn analyst --mol $MOL

# 4. In polecat, work through steps
bd --no-daemon ready --mol $MOL
bd --no-daemon update <first-task> --status in_progress
# ... work ...
bd --no-daemon close <first-task>
bd --no-daemon ready --mol $MOL  # Next unblocked

# 5. Check progress anytime
bd --no-daemon mol progress $MOL
```

## Read Full Docs

For detailed information, see: `docs/beads-workflow-integration.md`
