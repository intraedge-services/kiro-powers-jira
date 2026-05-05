# Jira Issue Templates

Standard templates for all Jira issue types. Apply the appropriate template when creating or updating issues.

## Template Selection

| Issue Type | Template | When to Use |
|---|---|---|
| Epic | Epic Template | Grouping related stories, large features, project milestones |
| Story | Story Template | User-facing functionality, business requirements |
| Task | Task Template | Technical work, implementation tasks, non-user-facing work |
| Bug | Bug Template | Defects, regressions, incorrect behavior |
| Sub-task | Sub-task Template | Breakdown of a parent Story or Task |

---

## Epic Template

**Summary format**: "[Feature/Initiative name]"

**Description**:
```markdown
## Goals
- [Primary goal of this Epic]
- [Secondary goal]

## Scope
[Description of what this Epic covers, including features, components, or capabilities]

## Out of Scope
[What is explicitly NOT included in this Epic]
```

**Auto-labels**: `epic`, `{project-name}`, `aidlc-synced` (if synced from AIDLC)

**Priority**: High (default for Epics)

**Example**:
- Summary: "User Authentication System"
- Goals: "Enable secure user login", "Support password reset flow"
- Scope: "Login, registration, password reset, session management"

---

## Story Template

Use the detailed template defined in `user-story-template.md`.

**Summary format**: "[Action verb] [what] [context]"

**Description**: Follow the minimal template (Description section + Acceptance Criteria section).

**Auto-labels**: `story`, `{project-name}`, `aidlc-synced` (if synced from AIDLC)

**Priority**: Medium (default for Stories)

---

## Task Template

**Summary format**: "[Action verb] [technical work description]"

**Description**:
```markdown
## Description
[What this task involves and why it is needed]

## Technical Details
[Implementation approach, algorithms, libraries, or patterns to use]
[API endpoints, data structures, or configurations involved]

## Definition of Done
- [ ] [First completion criterion]
- [ ] [Second completion criterion]
- [ ] [Tests written and passing]
- [ ] [Documentation updated]
```

**Auto-labels**: `task`, `{project-name}`, `aidlc-synced` (if synced from AIDLC)

**Priority**: Medium (default for Tasks)

**Example**:
- Summary: "Configure CI/CD pipeline for staging environment"
- Technical Details: "Set up GitHub Actions workflow with build, test, and deploy stages"
- DoD: "Pipeline runs on PR merge", "Staging deployment succeeds", "Slack notification configured"

---

## Bug Template

**Summary format**: "[Component/Feature]: [Brief description of the bug]"

**Description**:
```markdown
## Steps to Reproduce
1. [First step]
2. [Second step]
3. [Step where bug occurs]

## Expected Behavior
[What should happen]

## Actual Behavior
[What actually happens, include error messages if applicable]

## Environment
- Browser/Client: [e.g., Chrome 120, iOS 17]
- OS: [e.g., macOS 14, Windows 11]
- Version/Build: [e.g., v2.3.1, commit abc123]

## Severity
[Critical / Major / Minor / Trivial]
```

**Auto-labels**: `bug`, `{project-name}`

**Priority**: Mapped from severity: Critical=Highest, Major=High, Minor=Medium, Trivial=Low

**Example**:
- Summary: "Login: App crashes when password field is empty"
- Steps: "1. Open login page", "2. Enter email", "3. Leave password empty", "4. Click Submit"
- Expected: "Validation error message displayed"
- Actual: "App crashes with unhandled exception"

---

## Sub-task Template

**Summary format**: "[Brief description of the sub-task]"

**Description**:
```markdown
## Description
[What this sub-task involves]

**Parent**: [Parent issue key and summary for context]
```

**Auto-labels**: `subtask`, `{project-name}`, `aidlc-synced` (if synced from AIDLC)

**Priority**: Low (default for Sub-tasks)

---

## Label Sanitization Rules

When generating labels from project names:
1. Convert to lowercase
2. Replace spaces with hyphens
3. Remove special characters (keep alphanumeric and hyphens only)
4. Truncate to 255 characters
5. Example: "My AI-DLC Project" becomes "my-ai-dlc-project"
