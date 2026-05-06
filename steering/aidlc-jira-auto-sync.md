# AIDLC Jira Auto-Sync (Post User Stories Approval)

## Purpose

Automatically create Jira Epics and Stories after the User Stories stage is approved in the AIDLC process. This eliminates the manual step of creating Jira issues and ensures engineers are assigned work immediately after stories are finalized.

## When This Activates

This workflow executes **immediately after Step 22 (Record Approval Response)** in the User Stories stage, and **before Step 23 (Update Progress)**. It is triggered by the user's explicit approval of generated stories.

**Trigger condition**: User approves generated stories in the User Stories stage (Step 21 approval is confirmed).

---

## Workflow: Post-Approval Jira Sync

### Step A: Confirm Jira Sync Intent

After recording the user's story approval, present:

```markdown
> **🔗 Jira Sync**
>
> User stories are approved. Ready to create Jira Epics and Stories.
>
> **A)** Create Jira issues now and assign to engineers based on roles
> **B)** Skip Jira sync for now (can trigger manually later via the aidlc-jira-sync hook)
```

- If user selects **B**, skip to Step 23 of User Stories (Update Progress)
- If user selects **A**, proceed to Step B below

---

### Step B: Resolve Project Key and Team

1. Check if `JIRA_PROJECT_KEY` environment variable is set, or read `project.jira_project_key` from `config/team-roster.yml`
2. If set, confirm with user: "Using Jira project: {KEY}. Correct?"
3. If not set, ask: "Which Jira project key should I use? (e.g., PROJ)"
4. Load the team roster and role mapping from `config/team-roster.yml`
5. Present the team roster to the user for confirmation:

```markdown
> **Team Assignment Preview:**
>
> | Engineer | Role | Assigned Area |
> |----------|------|---------------|
> | [Name 1] | [Role] | [Area] |
> | [Name 2] | [Role] | [Area] |
>
> Confirm this mapping is correct, or provide updates.
```

---

### Step C: Map Stories to Epics

Parse the approved stories from `aidlc-docs/inception/user-stories/stories.md`:

1. **Identify Epic groupings**: Group stories by their parent epic/feature area
2. **Map each epic** to a Jira Epic issue
3. **Map each story** to a Jira Story under its parent Epic

**Mapping rules:**
- Each top-level feature group or epic in stories.md becomes a Jira Epic
- Each individual user story becomes a Jira Story linked to its parent Epic
- Story acceptance criteria are included in the Jira Story description
- Personas are referenced in the Story description

---

### Step D: Assign Based on Role Mapping

For each Story, determine the assignee using the team roster YAML:

1. Read the story's category/area (e.g., "data-pipeline", "ai-agent", "frontend", "backend")
2. Match against each engineer's `assigned_areas` list in `config/team-roster.yml`
3. Assign the story to the matching engineer (use `email` or `jira_account_id`)
4. If multiple engineers match an area, use the `distribution` rule (default: round-robin)
5. If no engineer matches, follow `unassigned_action` rule (default: flag in summary)

**Assignment logic:**
```
For each story:
  1. Extract story tags/areas from the story content
  2. Match tags against team[].assigned_areas in config/team-roster.yml
  3. If match found -> assign to that engineer (prefer jira_account_id, fallback to email)
  4. If multiple matches -> apply distribution rule (round-robin by default)
  5. If no match -> follow unassigned_action rule
```

---

### Step E: Create Jira Issues

Execute in this order:

1. **Create Epics first** (need Epic keys for linking Stories)
   - Use Epic template from `steering/jira-templates.md`
   - Labels: `["aidlc-synced", "epic", "{project-label}"]`
   - Priority: High

2. **Create Stories under Epics**
   - Use Story template from `steering/user-story-template.md`
   - Link to parent Epic via `parent_key`
   - Set assignee based on Step D mapping
   - Labels: `["aidlc-synced", "story", "{project-label}"]`
   - Priority: Medium

3. **Apply status**: All issues created in "To Do" (default)

**Error handling**: Follow rules in `steering/jira-operations.md`
- Fatal errors: Stop and report
- Non-fatal errors: Log, skip issue, continue
- Collect all errors for summary

---

### Step F: Present Sync Summary

After all issues are created, present:

```markdown
# Jira Sync Complete

**Project**: {PROJECT_KEY}
**Timestamp**: {ISO timestamp}

## Created Issues

| Type | Jira Key | Summary | Assignee | Parent |
|------|----------|---------|----------|--------|
| Epic | PROJ-101 | [Epic name] | - | - |
| Story | PROJ-102 | [Story title] | engineer@email.com | PROJ-101 |
| Story | PROJ-103 | [Story title] | engineer@email.com | PROJ-101 |

## Assignment Summary

| Engineer | Stories Assigned | Areas |
|----------|-----------------|-------|
| [Name 1] | 5 | data-pipeline, ingestion |
| [Name 2] | 4 | ai-agent, frontend |

## Unassigned (if any)
- [Story title] - No matching role found

## Errors (if any)
- [Error details]
```

---

### Step G: Log and Continue

1. Log the Jira sync results in `aidlc-docs/audit.md`
2. Proceed to Step 23 of User Stories (Update Progress)
3. Continue normal AIDLC workflow

---

## Integration Point in AIDLC Flow

```
User Stories Stage:
  Step 20: Present Completion Message
  Step 21: Wait for Explicit Approval
  Step 22: Record Approval Response
  -----> [THIS WORKFLOW: Steps A-G] <-----
  Step 23: Update Progress (continue to next stage)
```

---

## Dependencies

- `config/team-roster.yml` - Team roster with engineer roles and area assignments (MUST exist)
- `steering/jira-operations.md` - Jira API operation rules
- `steering/jira-templates.md` - Issue formatting templates
- `steering/user-story-template.md` - Story description format
- `mcp.json` - mcp-atlassian server configuration
- Approved stories at `aidlc-docs/inception/user-stories/stories.md`

---

## Fallback Behavior

- If `mcp-atlassian` MCP server is not available: Skip sync, inform user, suggest manual creation later
- If `config/team-roster.yml` does not exist: Ask user to provide engineer assignments inline
- If Jira project key is invalid: Ask user to correct it
- If all stories fail to create: Report errors, do not block AIDLC progression
