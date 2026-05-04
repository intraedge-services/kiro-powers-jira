# Jira Operations

Complete guide for creating, updating, and searching Jira issues via the `mcp-atlassian` MCP server.

## Before Any Operation

### Input Validation (MANDATORY)
Before calling any MCP tool, validate:
1. **Summary**: Not empty, max 255 characters. If longer, truncate at last word boundary and append "..."
2. **Project key**: Matches `^[A-Z][A-Z0-9_]+$`, 2-10 characters
3. **Priority**: One of `Highest`, `High`, `Medium`, `Low`, `Lowest`
4. **Labels**: Lowercase, no spaces (replace with hyphens), max 255 chars each
5. **Issue type**: One of `Epic`, `Story`, `Task`, `Sub-task`, `Bug`

If validation fails, report the specific field error to the user. Do NOT call the MCP tool.

### Project Key Resolution
- If `JIRA_PROJECT_KEY` environment variable is set, use it as default
- Always allow the user to override the project key per operation
- If no default is configured, ask the user for the project key

---

## Create Epic

Use `jira_create_issue` with issue type "Epic".

**Required fields**:
- `project_key`: Target Jira project
- `summary`: Epic title (max 255 chars)
- `issue_type`: "Epic"
- `description`: Formatted using the Epic template from `jira-templates.md`

**Optional fields**:
- `priority`: Default "High" for Epics
- `labels`: Auto-generate type label ("epic") + project label
- `components`: Jira components (if applicable)
- `fix_version`: Target release version

**Workflow**:
1. Validate all input fields
2. Format description using Epic template (Goals + Scope sections)
3. Call `jira_create_issue` with all fields
4. On success: report created issue key to user
5. On failure: follow error handling rules below

---

## Create Story

Use `jira_create_issue` with issue type "Story".

**Required fields**:
- `project_key`: Target Jira project
- `summary`: Story title (max 255 chars)
- `issue_type`: "Story"
- `description`: Formatted using the Story template from `user-story-template.md`

**Optional fields**:
- `priority`: Default "Medium" for Stories
- `parent_key`: Parent Epic key (to link Story under an Epic)
- `labels`: Auto-generate type label ("story") + project label
- `story_points`: Estimated effort
- `sprint`: Target sprint name
- `components`: Jira components
- `fix_version`: Target release version
- `assignee`: Assignee email or account ID
- `time_tracking`: Original estimate (e.g., "2h", "1d")

**Workflow**:
1. Validate all input fields
2. Format description using Story template (Description + Acceptance Criteria sections)
3. Call `jira_create_issue` with all fields
4. On success: report created issue key to user
5. On failure: follow error handling rules below

---

## Create Task

Use `jira_create_issue` with issue type "Task".

**Required fields**:
- `project_key`: Target Jira project
- `summary`: Task title (max 255 chars)
- `issue_type`: "Task"
- `description`: Formatted using the Task template from `jira-templates.md`

**Optional fields**:
- `priority`: Default "Medium" for Tasks
- `parent_key`: Parent Epic key
- `labels`: Auto-generate type label ("task") + project label
- `sprint`: Target sprint name
- `components`: Jira components
- `fix_version`: Target release version
- `assignee`: Assignee email or account ID
- `time_tracking`: Original estimate

**Workflow**: Same as Create Story (validate → format → create → report).

---

## Create Sub-task

Use `jira_create_issue` with issue type "Sub-task".

**Required fields**:
- `project_key`: Target Jira project
- `summary`: Sub-task title (max 255 chars)
- `issue_type`: "Sub-task"
- `parent_key`: Parent Story or Task key (REQUIRED for Sub-tasks)
- `description`: Formatted using the Sub-task template from `jira-templates.md`

**Optional fields**:
- `priority`: Default "Low" for Sub-tasks
- `labels`: Auto-generate type label ("subtask") + project label
- `assignee`: Assignee email or account ID
- `time_tracking`: Original estimate

---

## Update Story

Use `jira_update_issue` to modify existing Story fields.

**Workflow**:
1. Confirm the issue key exists (optionally call `jira_get_issue` first)
2. Identify which fields to update
3. If updating description, re-apply the Story template format
4. Call `jira_update_issue` with the issue key and updated fields
5. On success: confirm update to user
6. On failure: follow error handling rules

**Updatable fields**: summary, description, priority, labels, components, sprint, story_points, fix_version, assignee, time_tracking, custom fields

### Status Transitions

To change a Story's status, use `jira_transition_issue`:

1. Call `jira_get_transitions` (or `jira_get_issue`) to see available transitions
2. Find the transition matching the target status name
3. Call `jira_transition_issue` with the issue key and transition ID
4. If no matching transition found, report available transitions to the user

Common transitions: "To Do" → "In Progress" → "Done"

---

## Search Issues

Use `jira_search` with JQL (Jira Query Language).

**Common queries**:
```
# Find all issues in a project
project = PROJ

# Find issues assigned to me
project = PROJ AND assignee = currentUser()

# Find open stories
project = PROJ AND issuetype = Story AND status != Done

# Find AIDLC-synced issues
project = PROJ AND labels = "aidlc-synced"

# Find by summary text
project = PROJ AND summary ~ "search text"
```

---

## Jira Field Reference

### Essential Fields (All Issue Types)
| Field | MCP Parameter | Required | Notes |
|---|---|---|---|
| Summary | `summary` | Yes | Max 255 chars |
| Description | `description` | Yes | Markdown formatted |
| Issue Type | `issue_type` | Yes | Epic, Story, Task, Sub-task, Bug |
| Priority | `priority` | No | Highest/High/Medium/Low/Lowest |
| Assignee | `assignee` | No | Email or account ID |

### Standard Fields
| Field | MCP Parameter | Applies To | Notes |
|---|---|---|---|
| Labels | `labels` | All | Array of strings, lowercase |
| Components | `components` | All | Array of component names |
| Sprint | `sprint` | Story, Task | Sprint name or ID |
| Story Points | `story_points` | Story | Numeric estimate |
| Fix Version | `fix_version` | All | Release version name |
| Parent | `parent_key` | Story, Task, Sub-task | Parent Epic or Story key |

### Advanced Fields
| Field | MCP Parameter | Notes |
|---|---|---|
| Custom Fields | `custom_fields` | Key-value map of custom field IDs |
| Linked Issues | `linked_issues` | Issue links (blocks, relates to, etc.) |
| Time Tracking | `time_tracking` | Original/remaining estimate |
| Attachments | (separate tool) | Use `jira_add_attachment` if available |

---

## Error Handling

### Error Decision Tree

| Error | Type | Action |
|---|---|---|
| 401 Unauthorized | FATAL | Stop. Guide user: "Check your API token at https://id.atlassian.com/manage-profile/security/api-tokens" |
| 404 Project Not Found | FATAL | Stop. Guide user: "Verify the project key exists in your Jira instance" |
| 403 Forbidden | NON-FATAL | Log error. Report: "Insufficient permissions for this operation. Check your Jira project role." Continue. |
| 400 Invalid Fields | NON-FATAL | Log which field failed. Report specific validation error. Continue. |
| 400 Issue Type Not Found | NON-FATAL | Log error. Suggest: "Check your Jira project's issue type scheme." Continue. |
| 429 Rate Limited | RETRYABLE | Wait (respect Retry-After header). Retry once. If still fails, log and continue. |
| Connection Timeout | RETRYABLE | Retry once. If still fails, report: "Network issue — check your connection." Continue. |
| Transition Not Available | NON-FATAL | Log warning. Report available transitions. Issue created but not transitioned. |

### Error Reporting
- Never expose raw API error responses to the user
- Provide clear, actionable guidance for each error type
- For non-fatal errors during bulk operations, collect all errors and report in summary
