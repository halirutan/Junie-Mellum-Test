---
description: Complete the next unfinished milestone in the implementation plan, acting as team-lead
---

Act as team-lead: coordinate and review work for **exactly one** milestone in `docs/implementation-plan.md` — specifically, the next unfinished milestone (the first one whose status is not `Done`). Delegate implementation to the `worker` subagent. See `AGENTS.md` for tool references and context budget rules.

**Important:** This command works on a single milestone at a time. Once that milestone is complete and verified, stop and report back. Do not proceed to the next milestone unless explicitly asked.

### Key delegation rules

- **NEVER** task the worker with implementing an entire milestone. The worker is a Junior Intern — only give direct, stupid-simple tasks.
- One atomic task per worker delegation. Each task must be trivially simple, such as:
  - Implement this one function (with the full signature and body specified).
  - Add these test cases to this one test file.
  - Inspect or lint this one file.
  - Run the unit tests and report the output.
  - Create this one file with the provided content.
- Pre-read files; paste only relevant excerpts with paths and line numbers.
- Pre-identify exact file paths, line ranges, and expected changes.
- Do not ask the worker to investigate — you do that first.

## The implementation plan

The plan is at `docs/implementation-plan.md`. It contains milestones separated by `---`, each with:

- A `## Milestone N: Title` heading
- A **Goal** section
- Optional **Commands**, **Expected outcome**, **Files to create/modify**, **Test cases**, **Delegation** details

Each milestone ends with a status section:
- `**Status:** In Progress` — the milestone has started but is not complete
- `**Status:** Done` — the milestone is fully complete
- No status line — the milestone has not been started yet

## Workflow

### 1. Read the plan

Read `docs/implementation-plan.md` and identify all milestones.

### 2. Find the next unfinished milestone

Scan milestones in order (1, 2, 3, …) and find the first one whose status is NOT `Done`. If no milestone has a status line, treat it as unfinished.

### 3. Check if all milestones are done

If every milestone is marked `Done`, tell the user: "All milestones are complete. No next task available." and stop.

### 4. Analyze and plan

Break down the milestone into very small, trivially simple subtasks. Each subtask must be something a Junior Intern could execute without thinking — no open-ended work, no design decisions, no investigation. Examples of acceptable tasks:
- "Implement the `add(a, b)` function in `calculator/core.py` at line 42. Here is the exact code to write."
- "Run `uv run pytest tests/test_core.py` and report the output."
- "Check `calculator/core.py` for lint problems and report any errors."
- "Create `tests/test_add.py` with the following content: [full content]."

For each subtask:
- Research the codebase yourself: read relevant files, identify exact lines to change, determine dependencies.
- Create a sequential plan of atomic subtasks.
- If a subtask feels like it requires more than one simple action, split it further.

### 5. Delegate to worker

For each subtask, spawn the `worker` subagent with a task description that includes:

1. **Task**: A single, concrete, trivially simple action. If the task requires the worker to make any design decision or write more than a few lines of code without explicit guidance, it is too big — split it further.
2. **Files**: Exact paths, line ranges, and relevant code excerpts inline.
3. **Expected Change**: What the code should look like after the change. For implementation tasks, provide the exact code to insert.
4. **Allowed Commands**: Exact command strings, if any.
5. **Constraints**: Any restrictions (e.g., "do not modify other files").

Refer the worker to `AGENTS.md` for tool usage. You do ALL the investigation, planning, and design; the worker only does the typing.

**Critical:** The worker MUST NOT be asked to implement a whole milestone, a whole module, or even a whole class in one go. Each delegation should be one function, one test file, one command, or one inspection.

### 6. Review and iterate

Review the worker's output. If incorrect, provide a corrected, equally specific task. After all subtasks are complete, verify the overall result.

### 7. Verify the milestone

Run any verification commands listed in the milestone (e.g., `uv run pytest tests/`). Check that expected files were created or modified. Confirm the expected outcome was achieved.

### 8. Mark the milestone as done

Update `docs/implementation-plan.md`: add or replace the status line at the end of the completed milestone with:

```markdown
**Status:** Done
```

Also add a completion note:
```markdown
**Completed:** [today's date] — [brief summary of what was delivered]
```

### 9. Report back

Tell the user:
- Which milestone was completed.
- A brief summary of what was delivered.
- Whether verification passed.
- How many milestones remain.

## Notes

- Ask the user clarifying questions early if the milestone is ambiguous.
- Keep instructions language-agnostic; do not assume a specific programming language or toolchain.
