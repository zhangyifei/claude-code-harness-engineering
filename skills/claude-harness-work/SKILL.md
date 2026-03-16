---
description: "Implement tasks from the plan. Trigger: user says 'work', 'implement', 'build it', 'do it', or wants to start coding."
---

# /claude-harness-work — Implementation

Implement tasks from `Plans.md` (or a direct request if no plan exists).

## Step 1: Check for Plan

If `Plans.md` exists:
1. Read it and find uncompleted tasks (unchecked `- [ ]`)
2. Identify parallel groups and sequential dependencies
3. Show the user which tasks are pending and how they'll be executed
4. Ask which tasks to implement (or "all")

If no plan exists:
1. Implement the user's direct request
2. Suggest running `/claude-harness-plan` first for larger features

## Step 2: Execute Tasks

### Parallel Group Execution

For each parallel group in the plan, spawn **all tasks in the group simultaneously** using multiple Agent tool calls in a single message:

```
# In a SINGLE message, spawn all agents for the group at once:

Agent(
  prompt: "Task 1: [description]. Files: [file1], [file2]. Read CLAUDE.md and docs/CONVENTIONS.md first. Implement, self-review, verify build.",
  agent: ".claude/agents/implementation.md",
  isolation: "worktree"        ← each agent gets its own repo copy
)

Agent(
  prompt: "Task 2: [description]. Files: [file3]. Read CLAUDE.md and docs/CONVENTIONS.md first. Implement, self-review, verify build.",
  agent: ".claude/agents/implementation.md",
  isolation: "worktree"
)

Agent(
  prompt: "Task 3: [description]. Files: [file4], [file5]. Read CLAUDE.md and docs/CONVENTIONS.md first. Implement, self-review, verify build.",
  agent: ".claude/agents/implementation.md",
  isolation: "worktree"
)
```

**CRITICAL**: All agents in a parallel group MUST be spawned in a single message block. If you make separate messages, they run sequentially.

**Worktree isolation** (`isolation: "worktree"`) gives each agent its own copy of the repo, so parallel writes cannot conflict.

### Sequential Task Execution

For tasks marked as sequential (or depending on prior tasks), run them one at a time:
1. Wait for the previous task/group to complete
2. Spawn a single implementation agent
3. Wait for it to complete before proceeding

### Single Task (no plan)

For a direct user request with no plan, just spawn one implementation agent without worktree isolation (it writes directly to the working tree).

## Step 3: Merge Worktree Results

After a parallel group completes, each agent returns its worktree branch. The lead agent (you) must:

1. Review each agent's result for success/failure
2. If all succeeded, merge the worktree branches into the main working tree
3. If any failed, report the failure and ask user how to proceed
4. Run the full verification (build + lint + test) after merging

## Step 4: Update Plan

Mark completed tasks in `Plans.md`:
```markdown
- [x] **Task 1**: [Description] — files: `[file1]`, `[file2]`
```

## Step 5: Report

```
WORK COMPLETE

Execution:
- Group 1 (parallel, 3 agents): Task 1, Task 2, Task 3 — all succeeded
- Task 4 (sequential): succeeded
- Group 2 (parallel, 2 agents): Task 5, Task 6 — all succeeded

Files changed:
- [file1]: [what changed]
- [file2]: [what changed]

Verification: build OK | lint OK | tests OK

Remaining tasks: [N tasks left in Plans.md, or "none"]
```

## Execution Decision Tree

```
How many uncompleted tasks?
├── 1 task → run directly (no worktree)
├── 2+ tasks, all independent → parallel group with worktree isolation
├── 2+ tasks, some dependent → group into parallel + sequential phases
└── User said "all" → execute all groups in order, parallelizing within groups
```

## Error Recovery

- If an agent fails (build error, lint error): retry once with the error context
- If retry fails: mark task as blocked, continue with other tasks, report at end
- Max 2 attempts per task before escalating to user
- Never skip verification — every agent must pass build before reporting success
