# Implementation Queue: [FEATURE NAME]

**Source Plan**: `specs/[###-feature-name]/plan.md`
**Queue Created**: [DATE]
**Target Duration**: ~1 week

---

## Queue Status

| Status | Meaning |
|--------|---------|
| ⬚ Pending | Not yet started |
| 🔄 In Progress | Currently being implemented |
| ✅ Complete | Finished and validated |

---

## Queued Work

<!--
  This section contains the specific milestones and tasks selected from plan.md
  for the current implementation cycle (~1 week of work).
  
  The /speckit.tasks command will read this file and generate tasks ONLY for
  the work listed here, ignoring other milestones in plan.md.
-->

### From Milestone [N]: [Milestone Name]

**Milestone Status**: ⬚ Pending
**Estimated Duration**: [e.g., 3 days]

#### Selected Tasks

- [ ] [Task description from plan.md milestone]
- [ ] [Task description from plan.md milestone]
- [ ] [Task description from plan.md milestone]

#### Milestone Exit Criteria (for this queue)

- [Exit criteria that should be met when these tasks complete]

---

### From Milestone [N+1]: [Milestone Name] (Partial)

**Milestone Status**: ⬚ Pending
**Estimated Duration**: [e.g., 2 days]

#### Selected Tasks

- [ ] [Task description - partial milestone, only what fits in this week]
- [ ] [Task description]

#### Notes

- Remaining tasks from this milestone will be queued in the next cycle

---

## Out of Scope (This Queue)

<!--
  List milestones/work explicitly NOT included in this queue.
  This helps maintain clarity about what's deferred vs forgotten.
-->

- **Milestone [X]**: [Name] - Deferred to next queue
- **Milestone [Y]**: [Name] - Deferred to next queue

---

## Queue Summary

| Milestone | Tasks Included | Duration | Status |
|-----------|----------------|----------|--------|
| [Name] | [N] of [Total] | [X days] | ⬚ |
| [Name] | [N] of [Total] | [X days] | ⬚ |
| **Total** | **[N]** | **~1 week** | |

---

## Next Steps

1. Run `/speckit.tasks` to generate a focused task breakdown for this queue
2. Run `/speckit.implement` to execute the queued work
3. After completion, run `/speckit.queue` again to populate the next cycle

---

## Previous Queues (History)

<!--
  Track completed queue cycles for reference.
  Move completed queue summaries here when starting a new queue.
-->

### Queue 1 (Completed [DATE])

- Milestone 1: [Name] - ✅ Complete
- Tasks completed: [N]
