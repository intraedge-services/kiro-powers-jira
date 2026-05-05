# Kiro Power Best Practices

Guidelines for developing, maintaining, and extending this Jira power.

## POWER.md Authoring

### Frontmatter
- **name**: Use lowercase, hyphenated format (e.g., `jira`)
- **displayName**: Human-readable name (e.g., "Jira Cloud Integration")
- **description**: One sentence explaining what the power does
- **keywords**: Include terms developers actually use when talking about Jira: "jira", "ticket", "issue", "epic", "story", "sprint", "backlog", "aidlc", "sync"
- **author**: Your name or organization

### Onboarding Section
- Validate all dependencies before proceeding (check `uvx` is installed)
- Guide users step-by-step through setup (API token, env vars, hooks)
- Verify connectivity at the end (run a test search)
- Keep instructions concise; link to external docs for details

### Steering Mappings
- Map keywords to specific steering files
- Use clear "when to load" descriptions
- Keep mappings up to date when adding new steering files

## Steering File Writing

### Structure
- Start with a clear title and one-line purpose
- Use headers to organize sections logically
- Include code blocks for MCP tool call examples
- End with error handling or edge case guidance

### Tone
- Write as instructions for the Kiro agent, not documentation for humans
- Use imperative mood: "Validate the summary field" not "The summary field should be validated"
- Be specific about MCP tool names and parameters
- Include exact JQL queries, field names, and validation rules

### Modularity
- One concern per steering file
- Reference other steering files by name (e.g., "Apply the template from `user-story-template.md`")
- Avoid duplicating content across files; reference instead

## MCP Configuration

### Security
- Always use `${VAR}` syntax for credentials in mcp.json
- Never hardcode API tokens, URLs, or usernames
- Document required environment variables in POWER.md onboarding

### Server Configuration
- Use `uvx` for running the MCP server (no local installation needed)
- Keep the args array minimal; only include necessary flags
- Test the configuration locally before sharing

## Hook Best Practices

### Naming
- Use descriptive, hyphenated names: `aidlc-jira-sync`
- File extension: `.kiro.hook`

### Hook Design
- Keep the `askAgent` prompt clear and specific
- Include context about what the hook should do
- For `userTriggered` hooks, make the name self-explanatory in the UI

### Hook Schema
```json
{
  "name": "Hook Display Name",
  "version": "1.0.0",
  "description": "What this hook does",
  "when": {
    "type": "userTriggered"
  },
  "then": {
    "type": "askAgent",
    "prompt": "Clear, specific instructions for the agent"
  }
}
```

## Extending the Power

### Adding a New Issue Type
1. Add the template to `jira-templates.md`
2. Add the creation workflow to `jira-operations.md`
3. Update POWER.md keywords if needed
4. Test the new workflow end-to-end

### Adding a New Workflow
1. Create a new steering file in `steering/`
2. Add the steering file mapping to POWER.md
3. Add relevant keywords to POWER.md frontmatter
4. Create a hook if the workflow needs a manual trigger

### Adding Confluence Support (Future)
1. Add Confluence env vars to `mcp.json` (CONFLUENCE_URL, CONFLUENCE_USERNAME, CONFLUENCE_API_TOKEN)
2. Create `steering/confluence-operations.md`
3. Update POWER.md with Confluence keywords and steering mappings
4. The `mcp-atlassian` server already supports Confluence tools

## Versioning

- Use semantic versioning for the power (MAJOR.MINOR.PATCH)
- Update version in POWER.md frontmatter when making changes
- Document changes in a CHANGELOG.md if the power is shared publicly
- Breaking changes (renamed steering files, changed env vars) require a MAJOR version bump
