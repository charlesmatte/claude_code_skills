---
name: prd-to-issues
description: Break a PRD into independently-grabbable GitHub or Jira issues using tracer-bullet vertical slices. Use when user wants to convert a PRD to issues, create implementation tickets, or break down a PRD into work items.
---

# PRD to Issues

Break a PRD into independently-grabbable issues (GitHub or Jira) using vertical slices (tracer bullets).

## Process

### 1. Locate the PRD

Ask the user for the PRD issue number (or URL). This may be a GitHub issue or a Jira issue.

**If GitHub:** fetch it with `gh issue view <number>` (with comments).

**If Jira:** fetch it using the Atlassian MCP tool `getJiraIssue`.

### 2. Explore the codebase (optional)

If you have not already explored the codebase, do so to understand the current state of the code.

### 3. Draft vertical slices

Break the PRD into **tracer bullet** issues. Each issue is a thin vertical slice that cuts through ALL integration layers end-to-end, NOT a horizontal slice of one layer.

Slices may be 'HITL' or 'AFK'. HITL slices require human interaction, such as an architectural decision or a design review. AFK slices can be implemented and merged without human interaction. Prefer AFK over HITL where possible.

<vertical-slice-rules>
- Each slice delivers a narrow but COMPLETE path through every layer (schema, API, UI, tests)
- A completed slice is demoable or verifiable on its own
- Prefer many thin slices over few thick ones
</vertical-slice-rules>

### 4. Quiz the user

Present the proposed breakdown as a numbered list. For each slice, show:

- **Title**: short descriptive name
- **Type**: HITL / AFK
- **Blocked by**: which other slices (if any) must complete first
- **User stories covered**: which user stories from the PRD this addresses
- **Issue type**: Story (if the slice addresses user stories) or Task (if purely technical)

Ask the user:

- Does the granularity feel right? (too coarse / too fine)
- Are the dependency relationships correct?
- Should any slices be merged or split further?
- Are the correct slices marked as HITL and AFK?

Iterate until the user approves the breakdown.

### 5. Choose issue tracker and verify access

Ask the user: **"Should I create these issues in GitHub or Jira?"**

**If GitHub:**
1. Run `gh auth status` to verify authentication. If not authenticated, ask the user to run `! gh auth login`.
2. Run `gh repo view --json nameWithOwner -q .nameWithOwner` to identify the current repository. Show the user the repo name and ask: **"I'll create the issues in `<owner/repo>`. Is that correct?"** Wait for confirmation before proceeding.

**If Jira:**
1. Use the Atlassian MCP tool `getVisibleJiraProjects` to verify connectivity and list available projects. If the connection fails, tell the user their Jira/Atlassian integration is not authenticated and ask them to set it up.
2. Show the user the list of available projects and ask: **"Which Jira project should I create the issues in?"** Wait for their selection before proceeding.

### 6. Create the parent Epic (Jira only)

Before creating individual slice issues, create an Epic to serve as the parent container.

Use `getJiraProjectIssueTypesMetadata` to verify that "Epic" is available in the selected project. If it is not available, inform the user and ask how they'd like to group the issues.

Use the Atlassian MCP tool `createJiraIssue` with:
- `issueTypeName`: `"Epic"`
- `summary`: Use the PRD's title/summary as the Epic name
- `description`: A 2-3 sentence summary of the PRD scope (problem + solution), with a reference back to the source PRD issue key (e.g., "Source PRD: PROJ-100").
- `additional_fields`: Set priority to match the PRD's priority, and add labels `["vertical-slices"]`.

Record the returned Epic issue key for use in the next step.

**If GitHub:** Skip this step. GitHub does not have Epics. The existing approach of referencing the parent PRD via `#<issue-number>` is retained.

### 7. Create the issues

Use the issue body template below. Create issues in dependency order (blockers first) so you can reference real issue numbers/keys in the "Blocked by" field.

**If GitHub:** Create each issue using `gh issue create`. Populate as many fields as possible:
- Add labels for issue type (e.g., `enhancement`, `feature`) and slice type (`hitl` or `afk`). Use `gh label list` to check available labels first; create missing labels with `gh label create` if needed.
- If the repo uses milestones or projects, assign them when relevant.

**If Jira:** Create each issue using the Atlassian MCP tool `createJiraIssue` with the selected project. Set the `parent` parameter to the Epic key from Step 6 to make each issue a child of the Epic. If `parent` fails (some company-managed Jira projects require the Epic Link custom field instead), retry using `additional_fields` with `{"customfield_10014": "<EPIC-KEY>"}`.

**Issue type per slice:**
- **Story**: Use when the slice addresses one or more user stories from the PRD (i.e., the "User stories addressed" section references specific user stories).
- **Task**: Use when the slice is purely technical work — infrastructure setup, CI/CD configuration, database migrations, scaffolding, or preparatory work that does not directly fulfill a user story.
- For HITL slices, add the `hitl` label regardless of whether the issue type is Story or Task.

Use `createIssueLink` to create blocking relationships between sibling issues. Use `additional_fields` to populate each issue:
- **Priority**: Set based on dependency position — slices that block many others get `"High"`, leaf slices get `"Medium"`, nice-to-have slices get `"Low"`. Use `{"priority": {"name": "<level>"}}`.
- **Labels**: Add relevant labels (e.g., `["vertical-slice", "afk"]` or `["vertical-slice", "hitl"]`).
- **Time estimate**: Set `timetracking.originalEstimate` based on slice thickness — thin slices typically `"2h"`–`"4h"`, medium slices `"1d"`, thick slices `"2d"`–`"3d"`. Estimate based on the number of layers the slice cuts through.

<issue-template>
## Parent Epic

<epic-issue-key>

## Source PRD

<prd-issue-key>

## What to build

A concise description of this vertical slice. Describe the end-to-end behavior, not layer-by-layer implementation. Reference specific sections of the parent PRD rather than duplicating content.

## Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## Blocked by

- Blocked by #<issue-number> (if any)

Or "None - can start immediately" if no blockers.

## User stories addressed

Reference by number from the parent PRD:

- User story 3
- User story 7

</issue-template>

Do NOT close or modify the parent PRD issue. For Jira, replace `#<issue-number>` references with the Jira issue key (e.g., `PROJ-123`). For GitHub (where no Epic was created), use the PRD issue number for both the "Parent Epic" and "Source PRD" fields, or replace the "Parent Epic" section with "Parent PRD" referencing the original PRD issue.
