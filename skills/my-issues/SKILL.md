---
name: my-issues
description: >
  Fetch and display a quick summary of all your active issues across every board and project.
  Use this skill whenever the user wants a personal kanban overview, asks what they're currently
  assigned to, wants to know what's on their plate, or needs a cross-project status check.
  Trigger on phrases like: "show my issues", "what am I working on", "what's on my plate",
  "board summary", "active tickets", "show my issues", "what's assigned to me",
  "give me a status overview", "where are my tickets", "summarise my work", or any request
  to see a personal view across one or more boards. Don't wait for the user to name a specific
  tool — if they want an overview of their current work items, this skill applies.
---

# My Issues — Active Work Summary

Your job is to give the user a fast, clear snapshot of everything they're currently assigned to, across all projects. Speed matters — get in, fetch, display. Don't explore or narrate the process.

## Step 1: Identify the issue tracker

Check which issue-tracking MCP tools are available. Common ones:
- **Jira**: `atlassianUserInfo`, `searchJiraIssuesUsingJql`
- **Linear**: Linear MCP tools
- **GitHub Issues**: GitHub MCP tools

If multiple are available, pick the most likely one based on context (e.g. if the user has been working with Jira tickets, use Jira). If genuinely ambiguous, ask once: "I see tools for both X and Y — which do you use for issue tracking?"

## Step 2: Fetch active issues

### Jira

1. Call `atlassianUserInfo` to get the current user's account ID
2. Call `searchJiraIssuesUsingJql` with:
   - JQL: `assignee = currentUser() AND statusCategory != Done ORDER BY status ASC, project ASC`
   - Fields: `summary`, `status`, `reporter`, `parent`, `issuetype`, `project`, `labels`
   - `maxResults`: 50

Do both calls, then display immediately. Don't pause between fetching and rendering.

### Linear

Query for issues assigned to the current user that are not in a completed/cancelled state. Fetch: title, status, parent ID (if any), creator.

### GitHub Issues

Search for open issues assigned to the current user across accessible repos.

## Step 3: Render the table immediately

As soon as you have the data, output the table. Don't summarise what you're about to do — just show it.

### Output format

```
## My Active Issues — {date}

| Project | Parent   | Issue    | Title                        | Status       | Reporter       | Labels        |
|---------|----------|----------|------------------------------|--------------|----------------|---------------|
| PROJ    | PROJ-10  | PROJ-42  | Issue title here             | In Progress  | Jane Smith     | infosec, dx   |
| PROJ    | —        | PROJ-43  | Another issue title          | To Do        | Phil White     | —             |
```

**Column definitions:**
| Column | Value |
|--------|-------|
| Project | Project key (e.g. `PLT`, `DA`) |
| Parent | Parent issue key if one exists (e.g. `PROJ-10`), otherwise `—` |
| Issue | The issue key or number (e.g. `PROJ-42`, `#123`) |
| Title | Issue summary/title |
| Status | Workflow status name (e.g. `In Progress`, `To Do`) |
| Reporter | Display name of the person who created the issue |
| Labels | Comma-separated list of labels, or `—` if none |

**Formatting rules:**
- Use a single table for all issues — do not split by project
- Pad column separators so columns are visually aligned (use consistent spacing)
- Order rows: sort by status urgency first (In Progress / Review / Ready to Ship → Refinement → Blocked → To Do → Backlog), then by Project within each status group

### Full column order

| Project | Parent | Issue | Title | Status | Reporter | Labels |
|---------|--------|-------|-------|--------|----------|--------|

**Grouping:** Group by project within the table (sort order), but do NOT use separate sub-tables or headers per project — one flat table is easier to scan at a glance.

### If no active issues

"You have no active issues assigned to you right now."

### If there are more than 20 issues

Add a one-line summary at the top: "You have {N} active issues across {P} projects." Then show the full list.

## Principles

**Render immediately.** Don't narrate the process, don't explain what you're fetching — just fetch and display.

**Stay lean.** One query. One render. No intermediate commentary.

**Tracker-agnostic.** The format is the same regardless of which tool is used underneath. The user doesn't need to know which MCP is being called.
