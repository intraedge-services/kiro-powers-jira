# User Story Template

When creating or updating Jira Stories, always apply this minimal template to ensure consistency and clarity.

## Template Structure

Every Story description must contain these two sections:

```markdown
## Description
[Brief context and purpose of the story — what it is and why it matters]

## Acceptance Criteria
1. [First testable criterion]
2. [Second testable criterion]
...
```

## Summary (Title) Guidelines

- Keep it concise and action-oriented
- Maximum 255 characters (Jira limit)
- Use the format: "[Action verb] [what] [context]"
- Examples:
  - "Implement user login with email and password"
  - "Add pagination to the product listing page"
  - "Create API endpoint for order status retrieval"

## Description Section Guidelines

- Write 1-3 sentences explaining the context and purpose
- Answer: What is this story about? Why does it matter?
- Avoid technical implementation details (those belong in Tasks)
- Keep it understandable by non-technical stakeholders

## Acceptance Criteria Guidelines

### Format Options

**Option A — Checklist format** (preferred for simple criteria):
```markdown
## Acceptance Criteria
1. User can enter email and password on the login form
2. System validates credentials against the database
3. Successful login redirects to the dashboard
4. Failed login displays an error message
```

**Option B — Given/When/Then format** (preferred for behavior-driven criteria):
```markdown
## Acceptance Criteria
1. Given a registered user, When they enter valid credentials, Then they are redirected to the dashboard
2. Given an unregistered email, When they attempt login, Then an error message is displayed
3. Given a locked account, When they attempt login, Then a lockout message is displayed
```

### Rules
- Include at least 1 acceptance criterion per story
- Each criterion must be independently testable
- Avoid vague language ("should work properly") — be specific
- Cover the happy path and at least one error/edge case

## Example: Well-Formed Story

**Summary**: Implement user authentication with email and password

**Description**:
```markdown
## Description
Allow registered users to log into the application using their email address and password. This is the primary authentication method for accessing protected features.

## Acceptance Criteria
1. Login form accepts email and password inputs
2. Valid credentials grant access and redirect to the dashboard
3. Invalid credentials display a clear error message without revealing which field is wrong
4. Account locks after 5 consecutive failed attempts with a lockout message
5. Login session expires after 30 minutes of inactivity
```

## When to Apply This Template

- Creating a new Story via `jira_create_issue`
- Updating a Story description via `jira_update_issue`
- Syncing AIDLC tasks as Stories (AIDLC sync workflow)
- Any time a user asks to create or format a Jira Story
