# AIDLC-to-Jira Sync Workflow

Synchronize AI-DLC task hierarchy to Jira issues. Triggered manually via the `aidlc-jira-sync` hook.

## Sync Overview

```
Parse AIDLC Tasks → Resolve Project Key → Detect Existing Issues → Create/Update Hierarchy → Report Summary
```

---

## Step 1: Parse AIDLC Tasks

Read the AIDLC task files to build the task hierarchy.

### Detect Project Structure
1. Read `aidlc-docs/aidlc-state.md` for the project name
2. Check if `aidlc-docs/inception/application-design/unit-of-work.md` exists:
   - **If exists**: Multi-unit project — parse units and their tasks
   - **If not exists**: Single-unit project — parse tasks directly

### Parse Task Checkboxes
Extract tasks from markdown files using this checkbox pattern:
- `- [ ] 1. Task title` → status: **not_started**
- `- [-] 2. Task title` → status: **in_progress**
- `- [x] 3. Task title` → status: **completed**
- `- [~] 4. Task title` → status: **queued**
- `- [ ]* 5. Task title` → **optional task** (still sync, but note as optional)

Nested sub-tasks are indented checkboxes under a parent task.

### Build Hierarchy
- **Multi-unit**: Units → Tasks → Sub-tasks
- **Single-unit**: Tasks → Sub-tasks (no units)

---

## Step 2: Resolve Project Key

Determine which Jira project to sync to.

1. Check if `JIRA_PROJECT_KEY` environment variable is set
2. **If set**: Use it as default, but inform the user: "Using project key: {KEY}. Reply with a different key to override."
3. **If not set**: Ask the user: "Which Jira project should I sync to? Please provide the project key (e.g., PROJ)."
4. Validate the project key format: uppercase letters/digits/underscores, 2-10 chars

---

## Step 3: Detect Existing Issues

Search for already-synced issues to prevent duplicates.

**JQL Query**:
```
project = {PROJECT_KEY} AND labels = "aidlc-synced"
```

Call `jira_search` with this query. Build a map of existing issue summaries to issue keys for comparison in Step 4.

---

## Step 4: Sync Hierarchy

### Handle Single-Unit Projects
If the project has no units, ask the user:

> "This is a single-unit project. How should I organize the Jira issues?"
>
> A) Create an Epic named "{project name}" and add all tasks as Stories under it
> B) Create all tasks as standalone Stories (no parent Epic)

### Create Epics (Multi-Unit or User Chose Option A)
For each AIDLC unit (or the single project Epic):
1. Check if an Epic with this summary already exists (from Step 3 map)
2. **If exists**: Prompt user — "Epic '{summary}' already exists as {ISSUE_KEY}. Skip or Update?"
3. **If new**: Create Epic using `jira_create_issue`:
   - Summary: Unit name (or project name for single-unit)
   - Description: Format using Epic template (Goals + Scope from unit description)
   - Priority: High
   - Labels: `["aidlc-synced", "epic", "{project-label}"]`
4. Record the Epic key for linking child Stories

### Create Stories (From AIDLC Tasks)
For each top-level AIDLC task:
1. Check if a Story with this summary already exists (from Step 3 map)
2. **If exists**: Prompt user — "Story '{summary}' already exists as {ISSUE_KEY}. Skip or Update?"
   - **Skip**: Record as skipped, move to next task
   - **Update**: Update description (re-apply Story template), priority, labels via `jira_update_issue`
3. **If new**: Create Story using `jira_create_issue`:
   - Summary: Task title (truncate to 255 chars if needed)
   - Description: Format using Story template from `user-story-template.md`
     - Description section: Use task description or generate from title
     - Acceptance Criteria: Convert sub-tasks to criteria, or generate basic criteria
   - Priority: Medium
   - Parent: Link to parent Epic key (if applicable)
   - Labels: `["aidlc-synced", "story", "{project-label}"]`
4. Apply status transition if needed (see Status Mapping below)
5. Record the Story key for linking child Sub-tasks

### Create Sub-tasks (From AIDLC Sub-tasks)
For each nested AIDLC sub-task:
1. Check if a Sub-task with this summary already exists
2. **If exists**: Prompt user — Skip or Update (same as Stories)
3. **If new**: Create Sub-task using `jira_create_issue`:
   - Summary: Sub-task title
   - Description: Format using Sub-task template
   - Priority: Low
   - Parent: Link to parent Story key
   - Labels: `["aidlc-synced", "subtask", "{project-label}"]`
4. Apply status transition if needed

---

## Priority Mapping

| AIDLC Context | Jira Priority |
|---|---|
| Unit of Work (Epic) | High |
| Top-level Task (Story) | Medium |
| Sub-task | Low |

---

## Status Mapping with Transitions

All issues are created in "To Do" status (Jira default). Then transition if needed:

| AIDLC Status | Jira Action |
|---|---|
| not_started | No transition needed (stays in "To Do") |
| queued | No transition needed (stays in "To Do") |
| in_progress | Transition to "In Progress" via `jira_transition_issue` |
| completed | Transition to "Done" via `jira_transition_issue` |

**Transition procedure**:
1. After creating the issue, check the AIDLC task status
2. If transition needed, call `jira_transition_issue` with the target status
3. If transition fails (status name mismatch in the project's workflow), log a warning and continue — the issue is created but remains in "To Do"

---

## Label Generation

Every synced issue gets three labels:
1. **`aidlc-synced`** — sync tracking marker (used for duplicate detection)
2. **Type label** — `epic`, `story`, `task`, or `subtask` (lowercase)
3. **Project label** — AIDLC project name, sanitized:
   - Convert to lowercase
   - Replace spaces with hyphens
   - Remove special characters (keep alphanumeric and hyphens)
   - Example: "My AI-DLC Project" → `my-ai-dlc-project`

---

## Step 5: Report Sync Summary

After all tasks are processed, present the summary:

```markdown
# AIDLC → Jira Sync Summary

**Project**: {PROJECT_KEY}
**Timestamp**: {current timestamp}

## Summary
- Epics created: {count}
- Stories created: {count}
- Sub-tasks created: {count}
- Issues skipped: {count}
- Issues updated: {count}
- Errors: {count}

## Details
| Action | AIDLC Task | Jira Key | Summary |
|---|---|---|---|
| Created | 1 | PROJ-101 | Implement user auth |
| Created | 1.1 | PROJ-102 | Login form component |
| Skipped | 2 | PROJ-103 | Already exists |
| Updated | 3 | PROJ-104 | Updated description |
| Error | 4 | — | Permission denied |
```

---

## Error Handling During Sync

Follow the error handling rules from `jira-operations.md`:
- **Fatal errors** (401 auth, 404 project): Stop the entire sync
- **Non-fatal errors** (403 permission, 400 validation): Log the error, skip this task, continue with the next
- **Retryable errors** (timeout, 429): Retry once, then log and continue
- Collect all errors and include them in the sync summary report

---

## Validation During Sync

Before creating each issue:
1. Validate summary is not empty and within 255 chars
2. Validate project key format
3. Validate labels are lowercase with no spaces
4. If validation fails for a task, log the error and skip to the next task
