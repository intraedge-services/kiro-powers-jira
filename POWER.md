---
name: "jira"
displayName: "Jira Cloud Integration"
description: "Create and manage Jira Epics, Stories, Tasks, and Bugs with standard templates. Sync AI-DLC tasks to Jira issues with a single click."
keywords: ["jira", "ticket", "issue", "epic", "story", "task", "bug", "sprint", "backlog", "agile", "scrum", "kanban", "aidlc", "sync", "atlassian", "project management"]
author: "AI-DLC"
---

# Jira Cloud Integration Power

Create, update, and manage Jira issues directly from Kiro with standardized templates and AI-DLC task synchronization.

## Onboarding

### Step 1: Validate tools
Ensure `uvx` is installed (required to run the MCP server):
- Verify with: `uvx --version`
- If not installed, install `uv` first: https://docs.astral.sh/uv/getting-started/installation/

### Step 2: Get your Jira API token
1. Go to https://id.atlassian.com/manage-profile/security/api-tokens
2. Click "Create API token"
3. Give it a label (e.g., "Kiro Jira Power")
4. Copy the generated token (you will not see it again)

### Step 3: Set environment variables
Set these environment variables in your shell profile or `.env` file:

```bash
export JIRA_URL="https://your-company.atlassian.net"
export JIRA_USERNAME="your.email@company.com"
export JIRA_API_TOKEN="your_api_token_here"
export JIRA_PROJECT_KEY="PROJ"  # Optional: default project key
```

### Step 4: Install the sync hook
The power includes a hook at `hooks/aidlc-jira-sync.kiro.hook` that adds a manual trigger to sync AI-DLC tasks to Jira. This hook will be available in the Agent Hooks section of Kiro.

### Step 5: Verify connectivity
Ask Kiro: "Search for issues in my Jira project" to test the MCP connection and confirm everything is working.

## Security

### API Token Management
- Your API token is stored in the `JIRA_API_TOKEN` environment variable
- **NEVER** hardcode tokens in any file
- **NEVER** commit `.env` files to version control
- Add `.env` to your `.gitignore`
- Rotate API tokens periodically (recommended: every 90 days)

### Minimum Required Permissions
Your Jira Cloud user needs these project permissions:
- **Browse Projects**: search and read issues
- **Create Issues**: create Epics, Stories, Tasks, Sub-tasks
- **Edit Issues**: update issue fields
- **Transition Issues**: change issue status

Use a dedicated service account for automation when possible. Scope permissions to specific projects rather than granting organization-wide access.

### Credential Security Checklist
- [ ] API token stored in environment variable (not in files)
- [ ] `.env` file added to `.gitignore`
- [ ] Token has minimum required permissions
- [ ] Token rotation schedule established

## When to Load Steering Files

| User Intent | Steering File |
|---|---|
| Creating or updating Jira issues (Epics, Stories, Tasks) | `jira-operations.md` |
| Formatting a user story or writing acceptance criteria | `user-story-template.md` |
| Using issue templates (Epic, Story, Task, Bug formats) | `jira-templates.md` |
| Syncing AI-DLC tasks to Jira | `aidlc-jira-sync.md` |
| Developing or extending this power | `power-best-practices.md` |
