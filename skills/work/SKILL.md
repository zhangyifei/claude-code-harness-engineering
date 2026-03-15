---
description: "Implement tasks from the plan. Trigger: user says 'work', 'implement', 'build it', 'do it', or wants to start coding."
---

# /work — Implementation

Implement tasks from `Plans.md` (or a direct request if no plan exists).

## Step 1: Check for Plan

If `Plans.md` exists:
1. Read it and find uncompleted tasks (unchecked `- [ ]`)
2. Show the user which tasks are pending
3. Ask which tasks to implement (or "all")

If no plan exists:
1. Implement the user's direct request
2. Suggest running `/plan` first for larger features

## Step 2: For Each Task

### Read First
1. Read `docs/CONVENTIONS.md` for coding patterns
2. Read all files the task will touch
3. Read related files to understand context

### Implement
1. Make the code changes
2. Follow the project's golden principles (from `docs/CONVENTIONS.md`)
3. Keep changes minimal and focused on the task

### Self-Review
Before moving to the next task, check:
- Does the code follow project conventions?
- Are there any `TODO` stubs left behind?
- Would this break any existing functionality?

### Verify
Run the project's build/lint/test commands (from `CLAUDE.md`).
Fix any errors before moving on.

## Step 3: Update Plan

If working from `Plans.md`, mark completed tasks:
```markdown
- [x] **Task 1**: [Description] — files: `[file1]`, `[file2]`
```

## Step 4: Report

```
WORK COMPLETE

Completed:
- [x] Task 1: [what was done]
- [x] Task 2: [what was done]

Files changed:
- [file1]: [what changed]
- [file2]: [what changed]

Verification: build OK | lint OK | tests OK

Remaining tasks: [N tasks left in Plans.md, or "none"]
```

## Using Sub-Agents

For multiple independent tasks, use the Agent tool to run them in parallel:
- Spawn `.claude/agents/implementation.md` for each independent task
- Wait for all to complete
- Run verification once after all changes

For tasks with dependencies, run them sequentially.
