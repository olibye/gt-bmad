# BMAD Beads Workflow Integration

> **Automatic dependency tracking and cross-session continuity for BMAD workflows**

## Overview

The beads dependency integration enables BMAD workflows to automatically create and track tasks as beads with proper dependencies, replacing TodoWrite with a persistent, queryable system.

### Key Benefits

- **Cross-session continuity** - Resume workflows from exact state via molecules
- **Automatic dependency tracking** - Sequential steps auto-create blocking dependencies
- **No TodoWrite needed** - Agents use `bd` commands exclusively
- **Phase gate enforcement** - Sequential phase execution via beads dependencies
- **Central monitoring** - All workflow state visible via `bd mol progress`
- **Parallel execution** - Multiple agents can work on independent tasks

## Quick Start

### 1. Start a Workflow with Automatic Bead Creation

```bash
# Auto-start creates molecule + step beads with dependencies
gt-bmad-workflow auto-start brainstorming

# Output:
# ✓ Workflow brainstorming ready (molecule: hq-mol-abc)
# ✓ Created 7 workflow step beads
```

### 2. Spawn an Agent with Workflow Context

```bash
# Agent gets molecule ID in environment and PRIME.md
gt-bmad-spawn analyst --mol hq-mol-abc

# Agent will see:
# - BMAD_MOLECULE_ID=hq-mol-abc
# - Beads commands in their PRIME.md/CLAUDE.md
```

### 3. Work on Tasks (Inside Polecat Session)

```bash
# Find next available task
bd --no-daemon ready --mol $BMAD_MOLECULE_ID

# Start work on a task
bd --no-daemon update hq-mol-abc.1 --status in_progress

# Complete the task
bd --no-daemon close hq-mol-abc.1

# Next task automatically becomes READY
bd --no-daemon ready --mol $BMAD_MOLECULE_ID
```

### 4. Check Progress Anytime

```bash
# See overall workflow progress
bd --no-daemon mol progress hq-mol-abc

# Output: Progress: 1/14 (7.1%)
```

## Core Commands

### gt-bmad-workflow

Manages BMAD workflow execution with beads tracking.

#### auto-start

Create molecule and auto-generate step beads with dependencies.

```bash
# Basic usage
gt-bmad-workflow auto-start <workflow-name>

# With variables
gt-bmad-workflow auto-start prd --var project_name="MyApp" --var output_file="docs/prd.md"

# What it does:
# 1. Generates formula if not exists
# 2. Pours molecule
# 3. Creates bead for each workflow step
# 4. Adds dependencies (step-02 depends on step-01, etc.)
# 5. Exports BMAD_MOLECULE_ID and BMAD_WORKFLOW_NAME
```

**Output:**
```
✓ Workflow brainstorming ready (molecule: hq-mol-77z)

Essential commands:
  bd ready --mol hq-mol-77z              # Show next available tasks
  bd show <task-id>                      # View task details
  bd update <task-id> --status in_progress  # Start work
  bd close <task-id>                     # Complete work

Spawn polecat with workflow context:
  gt-bmad-spawn <agent> --mol hq-mol-77z
```

#### Other Commands

```bash
# List available workflows
gt-bmad-workflow list

# Generate formula only (without starting)
gt-bmad-workflow generate <workflow-name>

# Traditional start (pour molecule only, no step beads)
gt-bmad-workflow start <workflow-name>

# Show workflow status
gt-bmad-workflow status <mol-id>

# Show triggers for agent dispatch
gt-bmad-workflow triggers <agent>

# Dispatch workflow via agent trigger
gt-bmad-workflow dispatch <agent> <trigger>
```

### gt-bmad-spawn

Spawn polecat agents with BMAD personas and workflow context.

```bash
# Spawn agent with workflow context
gt-bmad-spawn <agent> --mol <molecule-id>

# Spawn agent with specific bead
gt-bmad-spawn dev --bead beads-123

# Spawn with custom name
gt-bmad-spawn analyst --mol hq-mol-abc --name analyst-research-1

# Available agents:
# - dev        (Developer)
# - analyst    (Business Analyst)
# - pm         (Product Manager)
# - architect  (System Architect)
# - sm         (Scrum Master)
# - ux-designer (UX Designer)
```

**What happens:**
1. Creates polecat worktree
2. Generates PRIME.md/CLAUDE.md with agent persona
3. Injects beads workflow context (if --mol provided)
4. Exports BMAD_MOLECULE_ID environment variable
5. Spawns tmux session

**Molecule context in PRIME.md:**
```markdown
## Workflow Context

You are working on a tracked workflow:
- **Molecule ID:** hq-mol-abc
- **Workflow:** brainstorming

**Essential beads commands:**
bd ready --mol hq-mol-abc                      # Show next available tasks
bd update <task-id> --status in_progress       # Start work
bd close <task-id>                             # Complete task

**NEVER use TodoWrite** - use bd commands exclusively.
```

### gt-bmad-phase

Orchestrate BMAD phases with gate beads.

#### start

Start a phase and create gate bead.

```bash
# Start Phase 1 (Analysis)
gt-bmad-phase start 1

# What it does:
# 1. Checks prerequisites (previous phase complete)
# 2. Creates phase gate bead with label "phase:1" "type:gate"
# 3. Adds dependency on previous phase gate (if any)
# 4. Spawns primary agent
```

#### complete

Close the phase gate to mark phase complete.

```bash
# Complete current phase
gt-bmad-phase complete 1

# Output:
# ✓ Phase 1 gate closed: beads-xyz
# Phase 1 is now complete!
```

#### advance

Auto-advance to next phase if current phase is complete.

```bash
# Check current phase completion and advance
gt-bmad-phase advance

# What it does:
# 1. Finds current phase
# 2. Checks if complete (all artifacts present OR gate closed)
# 3. Creates handoff context
# 4. Starts next phase
```

#### Other Commands

```bash
# List all phases
gt-bmad-phase list

# Show phase details
gt-bmad-phase show 2

# Check prerequisites
gt-bmad-phase check 3

# Show gate requirements
gt-bmad-phase gates 2

# Show current status
gt-bmad-phase status

# List agents for phase
gt-bmad-phase agents 2

# Create handoff context
gt-bmad-phase handoff 3
```

### bd (Beads Commands)

Core beads commands for task management within workflows.

```bash
# Find ready tasks in workflow
bd --no-daemon ready --mol <molecule-id>

# Show task details and dependencies
bd --no-daemon show <task-id>

# Start work on task
bd --no-daemon update <task-id> --status in_progress

# Complete task
bd --no-daemon close <task-id>

# Create subtask
bd --no-daemon create "Subtask title" --parent <task-id>

# Add dependency manually
bd --no-daemon dep add <task-id> <depends-on-id>

# View dependency tree
bd --no-daemon dep tree <task-id>

# Check molecule progress
bd --no-daemon mol progress <molecule-id>

# List all tasks in molecule
bd --no-daemon list --parent <molecule-id>
```

**Note:** Use `--no-daemon` flag for direct database access when needed.

## Workflows

### Complete BMAD Project Lifecycle

Execute all 4 phases with beads tracking:

```bash
# 1. Create project molecule
bd --no-daemon mol pour bmad-project --var project_name="E-commerce Platform"
# Output: Created molecule: bmad-mol-xyz

# 2. Phase 1: Analysis
gt-bmad-phase start 1
# Spawns analyst agent
# Complete research and product brief
gt-bmad-phase complete 1

# 3. Phase 2: Planning
gt-bmad-phase advance
# Spawns PM agent
# Complete PRD and UX design
gt-bmad-phase complete 2

# 4. Phase 3: Solutioning
gt-bmad-phase advance
# Spawns architect agent
# Complete architecture and epics/stories
gt-bmad-phase complete 3

# 5. Phase 4: Implementation
gt-bmad-phase advance
# Spawns dev agent
# Complete implementation
gt-bmad-phase complete 4

# 6. Verify completion
bd --no-daemon mol progress bmad-mol-xyz
# Should show 100% complete
```

### Single Workflow Execution

Execute a standalone workflow with beads:

```bash
# 1. Start workflow
gt-bmad-workflow auto-start brainstorming
# Output: molecule hq-mol-abc with 7 steps

# 2. Spawn agent
gt-bmad-spawn analyst --mol hq-mol-abc

# 3. In polecat session, work through steps
bd --no-daemon ready --mol $BMAD_MOLECULE_ID
# Shows: Step 1 is READY

bd --no-daemon update hq-mol-abc.1 --status in_progress
# Work on step 1...
bd --no-daemon close hq-mol-abc.1

bd --no-daemon ready --mol $BMAD_MOLECULE_ID
# Shows: Step 2 is READY (unblocked)

# Repeat for all steps...

# 4. Check progress
bd --no-daemon mol progress hq-mol-abc
```

### Resume Interrupted Workflow

Resume a workflow from previous session:

```bash
# 1. Find your workflow molecule
bd --no-daemon list --label workflow:brainstorming --status open

# 2. Check current state
bd --no-daemon mol progress hq-mol-abc
# Shows: 3/7 complete (42.8%)

# 3. See what's ready
bd --no-daemon ready --mol hq-mol-abc
# Shows: Step 4 is READY

# 4. Spawn agent and continue
gt-bmad-spawn analyst --mol hq-mol-abc
```

### Parallel Work with Multiple Agents

Multiple agents working on independent tasks:

```bash
# 1. Start workflow with parallel steps
gt-bmad-workflow auto-start sprint-planning

# 2. Spawn multiple agents
gt-bmad-spawn pm --mol hq-mol-abc --name pm-epics
gt-bmad-spawn sm --mol hq-mol-abc --name sm-stories
gt-bmad-spawn dev --mol hq-mol-abc --name dev-review

# 3. Each agent can work on independent ready tasks
# Agent 1: Works on epic creation (hq-mol-abc.3)
# Agent 2: Works on story creation (hq-mol-abc.4)
# Agent 3: Works on review prep (hq-mol-abc.5)

# All agents see the same molecule state via beads
```

## Agent Integration

### Critical Actions (All Agents)

Every BMAD agent now has these critical actions in their YAML definition:

```yaml
critical_actions:
  - "CHECK for BMAD_MOLECULE_ID at session start - this is your workflow context"
  - "Use 'bd ready --mol $BMAD_MOLECULE_ID' to find next available task"
  - "Before starting work: bd update <task-id> --status in_progress"
  - "After completing work: bd close <task-id>"
  - "For subtasks: bd create '<task>' --parent <current-step-id>"
  - "NEVER use TodoWrite - use bd commands exclusively"
  - "Check dependencies: bd show <task-id> to see what blocks/is blocked by"
```

### Workflow Step Template

Each workflow step includes beads tracking guidance:

```markdown
## Beads Task Tracking

**Find your current task:**
```bash
bd ready --mol $BMAD_MOLECULE_ID
```

**Mark this step in progress:**
```bash
CURRENT_STEP=$(bd ready --mol $BMAD_MOLECULE_ID | grep "step-01" | awk '{print $1}')
bd update $CURRENT_STEP --status in_progress
```

**Create subtasks (if needed):**
```bash
bd create "Subtask: Check existing context" --parent $CURRENT_STEP
```

**Complete this step:**
```bash
bd close $CURRENT_STEP
```

**Check what's next:**
```bash
bd ready --mol $BMAD_MOLECULE_ID
```
```

## Dependency Management

### Automatic Dependencies

When `auto-start` creates step beads:

```
step-01 [READY]
  └── depends on: molecule (parent)

step-02 [BLOCKED]
  ├── depends on: molecule
  └── depends on: step-01 (blocks) ← Automatic!

step-03 [BLOCKED]
  └── depends on: step-02 (blocks) ← Automatic!
```

### Dependency Types

- **blocks** (default) - Sequential dependency, step must complete before next can start
- **tracks** (future) - Parallel dependency, steps can run concurrently

### Manual Dependencies

Add custom dependencies for complex workflows:

```bash
# Create subtasks with parent relationship
bd create "Review architecture" --parent hq-mol-abc.3
bd create "Write tests" --parent hq-mol-abc.3

# Add manual dependency (task B depends on task A)
bd dep add <task-b-id> <task-a-id> --type blocks

# Remove dependency
bd dep remove <task-b-id> <task-a-id>
```

## Phase Gates

### How Phase Gates Work

Each phase has a gate bead that tracks completion:

```
Phase 1 Gate [beads-p1] label:"phase:1" "type:gate"
  └── No dependencies (first phase)

Phase 2 Gate [beads-p2] label:"phase:2" "type:gate"
  └── depends on: Phase 1 Gate [beads-p1] (blocks)

Phase 3 Gate [beads-p3] label:"phase:3" "type:gate"
  └── depends on: Phase 2 Gate [beads-p2] (blocks)
```

### Checking Phase Status

```bash
# Check if phase prerequisites met
gt-bmad-phase check 2

# Output:
# ✓ Phase 1 (Analysis) artifacts:
#   ✓ product-brief.md
# ✓ Phase 2 prerequisites satisfied
```

### Advancing Phases

```bash
# Manual: Complete current phase then start next
gt-bmad-phase complete 1
gt-bmad-phase start 2

# Automatic: Check completion and auto-advance
gt-bmad-phase advance
```

## Environment Variables

### Set Automatically

When using `auto-start` or spawning with `--mol`:

```bash
BMAD_MOLECULE_ID=hq-mol-abc      # Workflow molecule ID
BMAD_WORKFLOW_NAME=brainstorming  # Workflow name
BMAD_PHASE_GATE=beads-xyz         # Phase gate bead (phases only)
```

### Manual Setup

For custom integrations:

```bash
export BMAD_MOLECULE_ID=hq-mol-abc
export BMAD_WORKFLOW_NAME=brainstorming

# Now spawn agent
gt-bmad-spawn analyst
```

## Troubleshooting

### Workflow Not Found

```
Error: Workflow not found: brainstorming
```

**Solution:** Check workflow exists and paths are configured:

```bash
# List available workflows
gt-bmad-workflow list

# Check workflow paths
find $GT_BMAD_ROOT/bmad -name "workflow.md" -o -name "workflow.yaml"
```

### Database Locked

```
Error: sqlite3: database is locked
```

**Solution:** Use `--no-daemon` flag:

```bash
bd --no-daemon ready --mol hq-mol-abc
```

### Database Out of Sync

```
Error: Database out of sync with JSONL
```

**Solution:** Import JSONL updates:

```bash
bd sync --import-only
```

Or use sandbox mode:

```bash
bd --sandbox --allow-stale ready --mol hq-mol-abc
```

### No Ready Tasks

```
No ready tasks found for molecule
```

**Possible causes:**
1. All tasks completed - check `bd mol progress <mol-id>`
2. Current task blocked - check `bd dep tree <task-id>`
3. Task needs to be unblocked - close blocking task

**Debug:**

```bash
# See all tasks in molecule
bd --no-daemon list --parent <mol-id>

# Check specific task dependencies
bd --no-daemon show <task-id>
```

### Step Beads Not Created

If `auto-start` doesn't create step beads:

**Possible causes:**
1. Workflow has no `steps/` directory
2. Step files don't match pattern `step-*.md`
3. Beads creation failed silently

**Debug:**

```bash
# Check workflow structure
ls -la <workflow-dir>/steps/

# Manually verify formula
bd formula show bmad-<workflow-name>

# Check created beads
bd --no-daemon list --parent <mol-id>
```

## Best Practices

### 1. Always Use auto-start

Prefer `auto-start` over manual molecule creation:

```bash
# ✓ Good - automatic step beads and dependencies
gt-bmad-workflow auto-start brainstorming

# ✗ Avoid - manual, requires creating beads yourself
bd mol pour bmad-brainstorming
```

### 2. Spawn Agents with --mol

Always pass molecule ID when spawning:

```bash
# ✓ Good - agent gets workflow context
gt-bmad-spawn analyst --mol hq-mol-abc

# ✗ Avoid - agent doesn't know about workflow
gt-bmad-spawn analyst
```

### 3. Use --no-daemon for Direct Access

When molecule operations fail with daemon:

```bash
# ✓ Good - direct database access
bd --no-daemon ready --mol hq-mol-abc

# ✗ May fail - daemon mode restrictions
bd ready --mol hq-mol-abc
```

### 4. Close Tasks Immediately

Close tasks as soon as work is complete:

```bash
# ✓ Good - unblocks dependent tasks immediately
bd close hq-mol-abc.1

# ✗ Avoid - keeps dependent tasks blocked
# (working on step 2 without closing step 1)
```

### 5. Check Dependencies First

Before starting work, verify no blockers:

```bash
# ✓ Good - check what's blocking
bd --no-daemon dep tree hq-mol-abc.5

# Then work on unblocked tasks
bd --no-daemon ready --mol hq-mol-abc
```

### 6. Use Labels for Filtering

Label beads for easier querying:

```bash
# Create with labels
bd create "Research competitors" --label "research" --label "phase:1"

# Query by labels
bd list --label "research" --label "phase:1"
```

### 7. Track Progress Regularly

Monitor workflow completion:

```bash
# Check overall progress
bd --no-daemon mol progress hq-mol-abc

# List remaining tasks
bd --no-daemon list --parent hq-mol-abc --status open,in_progress
```

## Advanced Usage

### Custom Step Dependencies

Override automatic dependencies for complex flows:

```bash
# Start workflow
gt-bmad-workflow auto-start my-workflow

# Add custom dependency (step 5 also depends on step 3)
bd dep add hq-mol-abc.5 hq-mol-abc.3

# Now step 5 requires both step 4 and step 3 to complete
```

### Molecule Variables

Pass variables to formulas:

```bash
# With multiple variables
gt-bmad-workflow auto-start prd \
  --var project_name="MyApp" \
  --var output_file="docs/prd.md" \
  --var context_file="docs/project-context.md"
```

### Multiple Workflows in Parallel

Run different workflows concurrently:

```bash
# Start multiple workflows
gt-bmad-workflow auto-start brainstorming
# molecule: hq-mol-aaa

gt-bmad-workflow auto-start prd
# molecule: hq-mol-bbb

# Spawn agents for each
gt-bmad-spawn analyst --mol hq-mol-aaa --name analyst-brainstorm
gt-bmad-spawn pm --mol hq-mol-bbb --name pm-prd

# Each works independently
```

### Workflow Handoffs

Hand off workflow to another agent:

```bash
# Analyst completes step 1-3
bd close hq-mol-abc.1
bd close hq-mol-abc.2
bd close hq-mol-abc.3

# Check what's ready
bd ready --mol hq-mol-abc
# Shows: Step 4 (requires PM input)

# Spawn PM agent to continue
gt-bmad-spawn pm --mol hq-mol-abc
```

## Migration from TodoWrite

### Old Workflow (TodoWrite)

```typescript
// Agent uses TodoWrite
TodoWrite.create([
  { content: "Research competitors", status: "in_progress" },
  { content: "Write findings", status: "pending" },
  { content: "Create brief", status: "pending" }
]);
```

**Problems:**
- Lost on session close
- No dependency tracking
- No cross-agent visibility
- Can't query or monitor

### New Workflow (Beads)

```bash
# Workflow auto-creates beads
gt-bmad-workflow auto-start research

# Agent works with beads
bd ready --mol $BMAD_MOLECULE_ID
bd update <step-1> --status in_progress
bd close <step-1>
# Step 2 automatically becomes ready
```

**Benefits:**
- Persists across sessions
- Automatic dependencies
- All agents see same state
- Full query and monitoring

## Examples

### Example 1: Brainstorming Session

```bash
# Start brainstorming workflow
$ gt-bmad-workflow auto-start brainstorming
✓ Workflow brainstorming ready (molecule: hq-mol-77z)
✓ Created 7 workflow step beads

# Spawn analyst
$ gt-bmad-spawn analyst --mol hq-mol-77z

# In polecat session
$ bd ready --mol $BMAD_MOLECULE_ID
📋 Ready steps:
1. [● P2] Step 1: Session Setup and Continuation Detection

$ bd update hq-mol-77z.1 --status in_progress
✓ Updated status: in_progress

# Complete step
$ bd close hq-mol-77z.1
✓ Closed: hq-mol-77z.1

# Next step automatically ready
$ bd ready --mol $BMAD_MOLECULE_ID
📋 Ready steps:
1. [● P2] Step 2a: User-Selected Techniques
```

### Example 2: Full BMAD Project

```bash
# Create project
$ bd mol pour bmad-project --var project_name="Mobile App"
Created molecule: bmad-mol-x1y

# Phase 1: Analysis
$ gt-bmad-phase start 1
✓ Created phase gate: beads-p1
Spawning primary agent...

# Complete phase 1 work...
$ gt-bmad-phase complete 1
✓ Phase 1 gate closed: beads-p1

# Auto-advance
$ gt-bmad-phase advance
✓ Phase 1 complete
Advancing to Phase 2 (Planning)...
✓ Created phase gate: beads-p2
  Added dependency on Phase 1 gate
```

### Example 3: Sprint Planning

```bash
# Start sprint planning
$ gt-bmad-workflow auto-start sprint-planning
✓ Created molecule: hq-mol-sp1

# Spawn scrum master
$ gt-bmad-spawn sm --mol hq-mol-sp1

# Create stories for each epic
$ bd ready --mol $BMAD_MOLECULE_ID
1. [● P2] Create sprint backlog

$ bd update hq-mol-sp1.1 --status in_progress

# Create subtasks for complex steps
$ bd create "Review epic 1" --parent hq-mol-sp1.1
$ bd create "Review epic 2" --parent hq-mol-sp1.1
$ bd create "Review epic 3" --parent hq-mol-sp1.1
```

## Reference

### File Locations

```
gt-bmad/
├── bin/
│   ├── gt-bmad-workflow    # Workflow management
│   ├── gt-bmad-spawn        # Agent spawning
│   └── gt-bmad-phase        # Phase orchestration
├── .beads/
│   ├── formulas/            # Generated formulas
│   └── issues.jsonl         # Beads database
└── bmad/
    └── mayor/rig/src/modules/bmm/
        ├── agents/          # Agent definitions
        └── workflows/       # Workflow templates
```

### Formula Structure

Auto-generated formulas in `.beads/formulas/bmad-*.formula.toml`:

```toml
formula = "bmad-brainstorming"
description = "BMAD brainstorming workflow"

[[steps]]
id = "step-01-session-setup"
title = "Step 1: Session Setup"
needs = []

[[steps]]
id = "step-02-techniques"
title = "Step 2: User-Selected Techniques"
needs = ["step-01-session-setup"]  # Automatic dependency

[vars]
[vars.output_file]
description = "Path to output document"
default = ""
```

### Labels Used

Standard labels applied by the system:

- `workflow:<name>` - Workflow identifier (e.g., "workflow:brainstorming")
- `step:<number>` - Step number (e.g., "step:01", "step:02")
- `phase:<number>` - Phase number (e.g., "phase:1", "phase:2")
- `type:gate` - Phase gate indicator

Query by labels:

```bash
bd list --label "workflow:brainstorming"
bd list --label "phase:1" --label "type:gate"
```

## Additional Resources

- **Beads Documentation:** https://github.com/beadss/beads
- **BMAD Method:** https://github.com/olibye/BMAD-METHOD
- **Gas Town:** https://github.com/olibye/gas-town

## Support

For issues or questions:
- File issues: https://github.com/olibye/gt-bmad/issues
- Check logs: `tail -f logs/town.log`
- Review daemon status: `bd daemon status`
