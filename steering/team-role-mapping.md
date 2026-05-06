# Team Role Mapping

## Configuration Location

The team roster is maintained in YAML format at:

```
config/team-roster.yml
```

Edit that file to add/remove team members, update assigned areas, or change assignment rules.

## Quick Reference

The YAML file contains:
- **project**: Jira project key and default priorities
- **team**: List of engineers with name, email, role, and assigned areas
- **areas**: Definitions of all area tags used for story classification
- **assignment_rules**: Distribution logic (round-robin, overrides, inheritance)

## Usage

This mapping is consumed by `steering/aidlc-jira-auto-sync.md` during the AIDLC User Stories stage to automatically assign Jira Stories to the correct engineer after approval.
