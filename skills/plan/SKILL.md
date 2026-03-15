---
description: "Create or update a structured implementation plan. Trigger: user says 'plan', 'let's plan', 'break this down', or describes a feature to build."
---

# /plan — Structured Planning

Create a structured plan document for the current task.

## Step 1: Understand the Request

Ask the user what they want to build or change, if not already clear from context.

## Step 2: Read Project Context

1. Read `CLAUDE.md` for the repository map and invariants
2. Read `docs/ARCHITECTURE.md` for tech stack and boundaries
3. Read `docs/CONVENTIONS.md` for coding patterns
4. Read `docs/QUALITY.md` for known gaps

## Step 3: Create or Update Plans.md

Create `Plans.md` in the project root (or update if it exists) with this structure:

```markdown
# Plans

## [Feature/Task Name]

### Context
[Why this is needed, what problem it solves]

### Acceptance Criteria
- [ ] [Specific, testable criterion 1]
- [ ] [Specific, testable criterion 2]
- [ ] [Specific, testable criterion 3]

### Tasks

#### Parallel Group 1 (can run simultaneously)
- [ ] **Task 1**: [Description] — files: `[file1]`, `[file2]`
- [ ] **Task 2**: [Description] — files: `[file3]`
- [ ] **Task 3**: [Description] — files: `[file4]`, `[file5]`

#### Sequential (must run after Group 1)
- [ ] **Task 4**: [Description] — depends on: Task 1, Task 2 — files: `[file6]`

#### Parallel Group 2 (can run simultaneously, after Task 4)
- [ ] **Task 5**: [Description] — files: `[file7]`
- [ ] **Task 6**: [Description] — files: `[file8]`

### Risks
[Technical risks, unknowns, things to investigate first]
```

## Guidelines

- Each task should be a self-contained unit of work (completable by one agent)
- List the specific files each task will touch
- Keep tasks small: 3-5 tasks per parallel group is ideal
- **Explicitly group tasks as parallel or sequential** — `/work` uses these groups to decide execution strategy
- Tasks are parallel-safe when they touch **different files** — if two tasks modify the same file, they must be sequential
- Include acceptance criteria that are mechanically verifiable (tests, build, lint)

## Step 4: Confirm with User

Present the plan and ask:
- Does this cover everything?
- Are the parallel groups correct? (no file conflicts within a group)
- Any ordering constraints I missed?

After approval, the user can run `/work` to start implementation.
