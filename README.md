# Jira Cloud Integration — Kiro Power

Create and manage Jira Epics, Stories, Tasks, and Bugs with standard templates. Sync AI-DLC tasks to Jira issues with a single click.

## Features

- **Issue Management** — Create Epics, Stories, Tasks, Sub-tasks, and Bugs with standardized templates
- **User Story Template** — Minimal, consistent format: Title, Description, Acceptance Criteria
- **Jira Templates** — Pre-built templates for all issue types (Epic, Story, Task, Bug, Sub-task)
- **AIDLC Sync** — One-click synchronization of AI-DLC task hierarchy to Jira issues
- **Idempotent Sync** — Duplicate detection via Jira labels prevents re-creation
- **Status Mapping** — AIDLC task status automatically maps to Jira transitions

## Quick Start

### 1. Install the Power

Open Kiro IDE → Powers panel → **Add power from Local Path** → select the repository root directory.

### 2. Get a Jira API Token

Go to [https://id.atlassian.com/manage-profile/security/api-tokens](https://id.atlassian.com/manage-profile/security/api-tokens) and create a token.

### 3. Set Environment Variables

```bash
export JIRA_URL="https://your-company.atlassian.net"
export JIRA_USERNAME="your.email@company.com"
export JIRA_API_TOKEN="your_api_token"
export JIRA_PROJECT_KEY="PROJ"  # Optional default project
```

### 4. Start Using

Ask Kiro:
- "Create an Epic for user authentication"
- "Create a Story for implementing login"
- "Update PROJ-123 to In Progress"
- "Search for open issues in PROJ"

### 5. Sync AIDLC Tasks

Trigger the **Sync AIDLC Tasks to Jira** hook from Agent Hooks to push your AI-DLC task hierarchy to Jira.

## AIDLC Sync Mapping

| AIDLC Artifact | Jira Issue Type | Default Priority |
|---|---|---|
| Unit of Work | Epic | High |
| Task | Story | Medium |
| Sub-task | Sub-task | Low |

All synced issues are labeled `aidlc-synced` for tracking. Re-running the sync detects existing issues and prompts you to skip or update each one.

## Power Structure

```
├── POWER.md                        # Metadata, onboarding, security
├── mcp.json                        # MCP server configuration
├── README.md                       # This file
├── LICENSE                         # MIT License
├── steering/
│   ├── jira-operations.md          # Issue CRUD + field reference
│   ├── user-story-template.md      # Story template
│   ├── jira-templates.md           # All issue type templates
│   ├── aidlc-jira-sync.md          # AIDLC sync workflow
│   └── power-best-practices.md     # Development guidelines
└── hooks/
    └── aidlc-jira-sync.kiro.hook   # Manual sync trigger
```

## Requirements

- [Kiro IDE](https://kiro.dev/) with Powers support
- [uv / uvx](https://docs.astral.sh/uv/getting-started/installation/) — Python package runner
- Jira Cloud instance with API token
- Minimum Jira permissions: Browse Projects, Create Issues, Edit Issues, Transition Issues

## MCP Server

This power uses [mcp-atlassian](https://github.com/sooperset/mcp-atlassian) — a community MCP server for Atlassian products (MIT license, 5.1k+ stars, 72 tools).

## Security

- All credentials stored in environment variables — never hardcoded
- Add `.env` to your `.gitignore`
- Rotate API tokens every 90 days
- Use minimum required Jira permissions
- See the Security section in POWER.md for full guidance

## Contributing

See `steering/power-best-practices.md` for guidelines on extending this power.

## License

[MIT](LICENSE)
