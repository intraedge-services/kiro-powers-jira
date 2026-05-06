# Jira Cloud Integration: Kiro Power

Create and manage Jira Epics, Stories, Tasks, and Bugs with standard templates. Sync AI-DLC tasks to Jira issues with a single click.

## Features

- **Issue Management**: Create Epics, Stories, Tasks, Sub-tasks, and Bugs with standardized templates
- **User Story Template**: Minimal, consistent format (Title, Description, Acceptance Criteria)
- **Jira Templates**: Pre-built templates for all issue types (Epic, Story, Task, Bug, Sub-task)
- **AIDLC Sync**: One-click synchronization of AI-DLC task hierarchy to Jira issues
- **AIDLC Auto-Sync**: Automatic Jira issue creation after User Stories approval with role-based assignment
- **Profile-Based Credentials**: Switch between dev/staging/prod Jira instances via profiles
- **Idempotent Sync**: Duplicate detection via Jira labels prevents re-creation
- **Status Mapping**: AIDLC task status automatically maps to Jira transitions
- **Role-Based Assignment**: Auto-assign stories to engineers based on team roster configuration

---

## Installation

### Prerequisites

| Tool | Purpose | Install |
|------|---------|---------|
| [Kiro IDE](https://kiro.dev/) | IDE with Powers support | Download from kiro.dev |
| [uv / uvx](https://docs.astral.sh/uv/) | Python package runner (runs MCP server) | `brew install uv` or [install guide](https://docs.astral.sh/uv/getting-started/installation/) |
| [direnv](https://direnv.net/) | Auto-loads environment variables | `brew install direnv` |
| Jira Cloud | Atlassian Jira instance | [atlassian.com](https://www.atlassian.com/software/jira) |

### Step 1: Clone the Repository

```bash
git clone <repository-url>
cd kiro-powers-jira
```

### Step 2: Get a Jira API Token

1. Go to [https://id.atlassian.com/manage-profile/security/api-tokens](https://id.atlassian.com/manage-profile/security/api-tokens)
2. Click **Create API token**
3. Give it a label (e.g., "Kiro Power")
4. Copy the generated token

### Step 3: Configure Environment Variables

```bash
# Copy the example env file
cp .env.example .env
```

Edit `.env` with your credentials:

```bash
# Set active profile (dev, staging, or prod)
JIRA_PROFILE=dev

# Fill in your dev profile credentials
JIRA_DEV_URL=https://your-company.atlassian.net
JIRA_DEV_USERNAME=your.email@company.com
JIRA_DEV_API_TOKEN=your_api_token_here
JIRA_DEV_PROJECT_KEY=PROJ
```

You can configure multiple profiles (dev, staging, prod) in the same `.env` file. Switch between them by changing `JIRA_PROFILE`.

### Step 4: Set Up direnv

If you haven't already configured direnv in your shell:

```bash
# Add to ~/.bashrc or ~/.zshrc
eval "$(direnv hook bash)"   # for bash
eval "$(direnv hook zsh)"    # for zsh
```

Then allow the project's `.envrc`:

```bash
direnv allow
```

You should see:
```
Jira profile loaded: dev (https://your-company.atlassian.net)
```

### Step 5: Install the Power in Kiro

1. Open **Kiro IDE**
2. Open the **Powers panel** (sidebar)
3. Click **Add power from Local Path**
4. Select the repository root directory
5. The power will load and connect the `mcp-atlassian` MCP server

### Step 6: Verify Connection

Ask Kiro:
> "Search for issues in [YOUR_PROJECT_KEY]"

If you get results (or an empty list), the connection is working. If you get an auth error, double-check your `.env` credentials.

---

## Configuration

### Profile-Based Credentials (.env)

The `.env` file supports multiple Jira profiles for different environments:

```bash
JIRA_PROFILE=dev          # Active profile

# Each profile has its own credentials
JIRA_DEV_URL=https://dev.atlassian.net
JIRA_DEV_USERNAME=dev@company.com
JIRA_DEV_API_TOKEN=token-here
JIRA_DEV_PROJECT_KEY=DEV

JIRA_STAGING_URL=https://staging.atlassian.net
JIRA_STAGING_USERNAME=staging@company.com
JIRA_STAGING_API_TOKEN=token-here
JIRA_STAGING_PROJECT_KEY=STG

JIRA_PROD_URL=https://prod.atlassian.net
JIRA_PROD_USERNAME=prod@company.com
JIRA_PROD_API_TOKEN=token-here
JIRA_PROD_PROJECT_KEY=PROD
```

Switch profiles by changing `JIRA_PROFILE` and running `direnv allow`.

### Team Roster (config/team-roster.yml)

Configure your team for auto-assignment of Jira stories:

```yaml
team:
  - name: "Jane Smith"
    email: "jane@company.com"
    role: "Senior Data Engineer"
    assigned_areas:
      - data-pipeline
      - ingestion
      - transformation

  - name: "John Doe"
    email: "john@company.com"
    role: "ML/AI Engineer"
    assigned_areas:
      - ai-agent
      - ml-model
      - cortex-ai
```

See `config/team-roster.yml` for the full configuration including area definitions and assignment rules.

### MCP Server (mcp.json)

The MCP server configuration references environment variables automatically:

```json
{
  "mcpServers": {
    "mcp-atlassian": {
      "command": "uvx",
      "args": ["mcp-atlassian"],
      "env": {
        "JIRA_URL": "${JIRA_URL}",
        "JIRA_USERNAME": "${JIRA_USERNAME}",
        "JIRA_API_TOKEN": "${JIRA_API_TOKEN}"
      }
    }
  }
}
```

No changes needed here -- it reads from the profile-resolved environment variables.

---

## Usage

### Basic Issue Management

Ask Kiro:
- "Create an Epic for user authentication"
- "Create a Story for implementing login"
- "Create a Bug: Login crashes when password is empty"
- "Update PROJ-123 to In Progress"
- "Search for open issues in PROJ"

### AIDLC Auto-Sync (After User Stories Approval)

When using the AIDLC workflow, Jira issues are created automatically after User Stories are approved:

1. Run the AIDLC workflow (ask Kiro to build a feature)
2. Complete the User Stories stage
3. On approval, the system prompts: "Create Jira issues now?"
4. Select **A** to create Epics and Stories with role-based assignment
5. Issues are created and assigned based on `config/team-roster.yml`

### Manual AIDLC Sync (Hook)

Trigger the **Sync AIDLC Tasks to Jira** hook from the Agent Hooks panel to push your AI-DLC task hierarchy to Jira at any time.

---

## AIDLC Sync Mapping

| AIDLC Artifact | Jira Issue Type | Default Priority |
|---|---|---|
| Unit of Work / Epic | Epic | High |
| Task / User Story | Story | Medium |
| Sub-task | Sub-task | Low |

All synced issues are labeled `aidlc-synced` for tracking. Re-running the sync detects existing issues and prompts you to skip or update each one.

---

## Project Structure

```
├── POWER.md                        # Metadata, onboarding, security
├── README.md                       # This file
├── LICENSE                         # MIT License
├── mcp.json                        # MCP server configuration
├── .env.example                    # Template for credentials (safe to commit)
├── .env                            # Actual credentials (git-ignored)
├── .envrc                          # direnv profile loader
├── .gitignore                      # Ignores .env, .envrc, aidlc-docs/
├── config/
│   └── team-roster.yml             # Team members and role-based assignment
├── steering/
│   ├── jira-operations.md          # Issue CRUD + field reference
│   ├── user-story-template.md      # Story template
│   ├── jira-templates.md           # All issue type templates
│   ├── aidlc-jira-sync.md          # Manual AIDLC sync workflow
│   ├── aidlc-jira-auto-sync.md     # Auto-sync after User Stories approval
│   ├── team-role-mapping.md        # Role mapping reference (points to YAML)
│   └── power-best-practices.md     # Development guidelines
└── hooks/
    └── aidlc-jira-sync.kiro.hook   # Manual sync trigger
```

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `401 Unauthorized` | Check API token at [Atlassian API tokens](https://id.atlassian.com/manage-profile/security/api-tokens). Ensure `.env` has correct credentials. |
| `404 Project Not Found` | Verify `JIRA_PROJECT_KEY` matches an existing project in your Jira instance. |
| `MCP server not connected` | Open Command Palette > search "MCP" > reconnect servers. Check `uvx` is installed. |
| `direnv: error .envrc is blocked` | Run `direnv allow` in the project directory. |
| `Profile not loading` | Ensure `JIRA_PROFILE` value matches a configured profile (dev/staging/prod). |
| `Permission denied (403)` | Your Jira user needs: Browse Projects, Create Issues, Edit Issues, Transition Issues. |

---

## Security

- All credentials stored in environment variables (never hardcoded)
- `.env` and `.envrc` are git-ignored -- only `.env.example` is committed
- Use separate API tokens per profile/environment
- Rotate API tokens every 90 days
- Use minimum required Jira permissions per environment
- See the Security section in POWER.md for full guidance

---

## Requirements

- [Kiro IDE](https://kiro.dev/) with Powers support
- [uv / uvx](https://docs.astral.sh/uv/getting-started/installation/): Python package runner
- [direnv](https://direnv.net/): Environment variable management
- Jira Cloud instance with API token
- Minimum Jira permissions: Browse Projects, Create Issues, Edit Issues, Transition Issues

## MCP Server

This power uses [mcp-atlassian](https://github.com/sooperset/mcp-atlassian), a community MCP server for Atlassian products (MIT license, 5.1k+ stars, 72 tools).

## Contributing

See `steering/power-best-practices.md` for guidelines on extending this power.

## License

[MIT](LICENSE)
