---
description: Create a focused queue of the next ~1 week of work from a plan's milestones.
scripts:
  sh: scripts/bash/check-prerequisites.sh --json
  ps: scripts/powershell/check-prerequisites.ps1 -Json
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Purpose

The `/queue` command is **optional** and designed for larger projects where the full implementation plan spans multiple weeks or sprints. It extracts a focused subset of work (~1 week) from `plan.md` milestones, allowing teams to:

- Implement features incrementally in manageable chunks
- Generate focused `tasks.md` files scoped to the current queue
- Track progress through a multi-sprint delivery

**When to use `/queue`:**
- Large features with multiple milestones spanning 2+ weeks
- Team wants to focus on incremental delivery
- Sprint-based development workflow

**When to skip `/queue`:**
- Small features completable in ~1 week or less
- Single milestone projects
- Rapid prototyping where full implementation is desired

## Outline

1. **Setup**: Run `{SCRIPT}` from repo root and parse JSON for FEATURE_DIR and IMPL_PLAN path. All paths must be absolute.

2. **Load context**:
   - Read IMPL_PLAN (`plan.md`) to understand milestones, their tasks, durations, and completion status
   - If `queue.md` already exists, read it to understand previously queued/completed work

3. **Determine queue scope**:
   - Identify the next incomplete milestone(s) from `plan.md`
   - Select milestones/tasks that total approximately 1 week of work
   - The scope can be a partial milestone, one full milestone, or parts of multiple milestones depending on task complexity

4. **Generate queue file**:
   - Use `templates/queue-template.md` as the structure
   - Output to `queue.md` in the same directory as `plan.md` (unless user specifies different name)
   - If `queue.md` already exists, update it to reflect the next batch of work (mark previous queue items as completed if they appear done in plan.md)

5. **Report**:
   - Output the path to `queue.md`
   - Summarize what's included in the queue (milestone names, estimated duration)
   - Remind user to run `/speckit.tasks` next to generate focused task breakdown

## Key Rules

- Queue extracts **milestones and their tasks** from `plan.md`, not from `tasks.md`
- The `/speckit.tasks` command will read `queue.md` (if present) to scope its output
- If `tasks.md` already exists from a previous queue, user should run `/speckit.tasks` again after updating the queue
- Total queued work should be achievable within ~1 week (adjust based on team capacity if specified)
- Mark queue items with status indicators to track progress
