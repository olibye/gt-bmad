# GT-BMAD Documentation

Documentation for Gas Town BMAD integration and workflows.

## Quick Links

- **[Beads Workflow Integration](beads-workflow-integration.md)** - Complete guide to beads dependency management with BMAD workflows
- **[Quick Reference](beads-quick-reference.md)** - Cheat sheet for daily use

## What's New

### Beads Dependency Management (Latest)

BMAD workflows now auto-create beads with dependencies, replacing TodoWrite:

- **Auto-start workflows** - `gt-bmad-workflow auto-start <name>`
- **Spawn with context** - `gt-bmad-spawn <agent> --mol <mol-id>`
- **Phase orchestration** - `gt-bmad-phase start|complete|advance`
- **Cross-session continuity** - Resume workflows from exact state
- **No TodoWrite** - Agents use `bd` commands exclusively

**Get Started:** See [Quick Reference](beads-quick-reference.md)

## Documentation Structure

```
docs/
├── README.md                           # This file
├── beads-workflow-integration.md       # Complete beads workflow guide
└── beads-quick-reference.md            # Quick reference cheat sheet
```

## Core Concepts

### Workflows

BMAD workflows define step-by-step processes for:
- Brainstorming sessions
- Product requirement documents (PRDs)
- Architecture design
- Sprint planning
- Development stories
- Code reviews

Each workflow can be converted to a beads formula and executed with automatic task tracking.

### Molecules

Molecules are workflow instances that track:
- Current step position
- Completion progress
- Step dependencies
- Cross-session state

### Phase Gates

BMAD projects progress through 4 phases:
1. **Analysis** - Research and product brief
2. **Planning** - PRD and UX design
3. **Solutioning** - Architecture and stories
4. **Implementation** - Code and reviews

Each phase has a gate bead that enforces sequential execution.

### Agents

BMAD agents are specialized polecats with personas:
- **Analyst** - Business analysis and research
- **PM** - Product management and PRDs
- **UX Designer** - User experience design
- **Architect** - System architecture
- **SM** - Scrum master and story prep
- **Dev** - Development and implementation

## Quick Start

```bash
# 1. Start a workflow with automatic beads
gt-bmad-workflow auto-start brainstorming

# 2. Spawn an agent with workflow context
gt-bmad-spawn analyst --mol hq-mol-abc

# 3. In the polecat session, work through tasks
bd --no-daemon ready --mol $BMAD_MOLECULE_ID
bd --no-daemon update <task-id> --status in_progress
bd --no-daemon close <task-id>

# 4. Check progress anytime
bd --no-daemon mol progress hq-mol-abc
```

## Commands Reference

### Workflow Management

```bash
gt-bmad-workflow list              # List available workflows
gt-bmad-workflow auto-start <name> # Start with auto beads
gt-bmad-workflow status <mol-id>   # Show workflow progress
gt-bmad-workflow triggers <agent>  # List agent triggers
```

### Agent Spawning

```bash
gt-bmad-spawn <agent> --mol <mol-id>  # Spawn with workflow context
gt-bmad-spawn <agent> --bead <bead>   # Spawn with specific bead
```

Available agents: `dev`, `analyst`, `pm`, `architect`, `sm`, `ux-designer`

### Phase Orchestration

```bash
gt-bmad-phase list              # List all phases
gt-bmad-phase start <num>       # Start a phase
gt-bmad-phase complete <num>    # Close phase gate
gt-bmad-phase advance           # Auto-advance to next
gt-bmad-phase status            # Show current status
```

### Beads (Task Management)

```bash
bd --no-daemon ready --mol <mol-id>              # Find ready tasks
bd --no-daemon update <id> --status in_progress  # Start task
bd --no-daemon close <id>                        # Complete task
bd --no-daemon mol progress <mol-id>             # Check progress
bd --no-daemon dep tree <id>                     # View dependencies
```

## Environment Variables

When workflows are active:

```bash
BMAD_MOLECULE_ID      # Current workflow molecule
BMAD_WORKFLOW_NAME    # Workflow name
BMAD_PHASE_GATE       # Phase gate bead (if in phase)
```

## Examples

### Complete BMAD Project

```bash
# Create project
bd --no-daemon mol pour bmad-project --var project_name="MyApp"

# Execute all 4 phases
gt-bmad-phase start 1    # Analysis
gt-bmad-phase complete 1
gt-bmad-phase advance    # Planning
gt-bmad-phase complete 2
gt-bmad-phase advance    # Solutioning
gt-bmad-phase complete 3
gt-bmad-phase advance    # Implementation
gt-bmad-phase complete 4
```

### Single Workflow Session

```bash
# Start brainstorming
gt-bmad-workflow auto-start brainstorming

# Work through it
gt-bmad-spawn analyst --mol hq-mol-abc

# Resume later
bd --no-daemon list --label workflow:brainstorming --status open
gt-bmad-spawn analyst --mol hq-mol-abc
```

## Troubleshooting

Common issues and solutions:

| Issue | Solution |
|-------|----------|
| Workflow not found | Check `gt-bmad-workflow list` |
| Database locked | Use `bd --no-daemon` |
| Out of sync | Run `bd sync --import-only` |
| No ready tasks | Check `bd dep tree <id>` for blockers |

For detailed troubleshooting, see [Beads Workflow Integration](beads-workflow-integration.md#troubleshooting).

## Best Practices

1. **Always use auto-start** - Creates beads automatically
2. **Spawn with --mol** - Agents get workflow context
3. **Close tasks immediately** - Unblocks dependent tasks
4. **Check dependencies first** - Avoid working on blocked tasks
5. **Use --no-daemon** - More reliable for molecule operations
6. **Track progress regularly** - Monitor workflow completion

## Migration from TodoWrite

**Old way:**
```typescript
TodoWrite.create([...tasks]) // Lost on session close
```

**New way:**
```bash
gt-bmad-workflow auto-start <workflow>  # Persists forever
bd --no-daemon ready --mol $BMAD_MOLECULE_ID
```

Benefits:
- Cross-session continuity
- Automatic dependencies
- Central monitoring
- Multi-agent collaboration

## Support

- **File Issues:** https://github.com/olibye/gt-bmad/issues
- **View Logs:** `tail -f logs/town.log`
- **Check Daemon:** `bd daemon status`

## Contributing

When adding new workflows:

1. Create workflow directory in `bmad/mayor/rig/src/modules/bmm/workflows/`
2. Add `workflow.md` or `workflow.yaml`
3. Add `steps/step-01-*.md`, `steps/step-02-*.md`, etc.
4. Include beads tracking section in each step (see template)
5. Generate formula: `gt-bmad-workflow generate <name>`
6. Test: `gt-bmad-workflow auto-start <name>`

## Additional Resources

- **Gas Town:** https://github.com/olibye/gas-town
- **Beads:** https://github.com/beadss/beads
- **BMAD Method:** https://github.com/olibye/BMAD-METHOD
