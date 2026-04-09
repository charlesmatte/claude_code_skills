---
name: write-a-prd
description: Create a PRD through user interview, codebase exploration, and module design, then submit as a GitHub or Jira issue. Use when user wants to write a PRD, create a product requirements document, or plan a new feature.
---

This skill will be invoked when the user wants to create a PRD. You may skip steps if you don't consider them necessary.

1. Ask the user for a long, detailed description of the problem they want to solve and any potential ideas for solutions.

2. Explore the repo to verify their assertions and understand the current state of the codebase.

3. Interview the user relentlessly about every aspect of this plan until you reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one.

4. Sketch out the major modules you will need to build or modify to complete the implementation. Actively look for opportunities to extract deep modules that can be tested in isolation.

A deep module (as opposed to a shallow module) is one which encapsulates a lot of functionality in a simple, testable interface which rarely changes.

Check with the user that these modules match their expectations. Check with the user which modules they want tests written for.

5. Choose issue tracker and verify access.

Ask the user: **"Should I create this PRD in GitHub or Jira?"**

**If GitHub:**
1. Run `gh auth status` to verify authentication. If not authenticated, ask the user to run `! gh auth login`.
2. Run `gh repo view --json nameWithOwner -q .nameWithOwner` to identify the current repository. Show the user the repo name and ask: **"I'll create the PRD in `<owner/repo>`. Is that correct?"** Wait for confirmation before proceeding.

**If Jira:**
1. Use the Atlassian MCP tool `getVisibleJiraProjects` to verify connectivity and list available projects. If the connection fails, tell the user their Jira/Atlassian integration is not authenticated and ask them to set it up.
2. Show the user the list of available projects and ask: **"Which Jira project should I create the PRD in?"** Wait for their selection before proceeding.

6. Create the issue.

**If GitHub:** Create the issue using `gh issue create`. Populate as many fields as possible:
- Add labels for the type of work (e.g., `prd`, `feature`, `enhancement`). Use `gh label list` to check available labels first; create missing labels with `gh label create` if needed.
- If the repo uses milestones or projects, assign them when relevant.

**If Jira:** Create the issue using the Atlassian MCP tool `createJiraIssue` with the selected project. Use issue type "Story" unless the user specifies otherwise. Use `additional_fields` to populate:
- **Priority**: Set based on the urgency and impact discussed during the interview — `"Highest"` for critical user-facing issues, `"High"` for important features with clear demand, `"Medium"` for planned improvements, `"Low"` for exploratory work. Use `{"priority": {"name": "<level>"}}`.
- **Labels**: Add relevant labels (e.g., `["prd", "feature"]` plus any domain-specific labels discussed).
- **Time estimate**: Set `timetracking.originalEstimate` for the overall feature based on the number of modules identified and user stories listed. Use rough sizing: small feature (1-3 stories, 1 module) → `"3d"`, medium (4-8 stories, 2-3 modules) → `"1w"`, large (9+ stories, 4+ modules) → `"2w"`. This is the total implementation estimate, not just the PRD writing time.

Use the template below to write the PRD.

<prd-template>

## Problem Statement

The problem that the user is facing, from the user's perspective.

## Solution

The solution to the problem, from the user's perspective.

## User Stories

A LONG, numbered list of user stories. Each user story should be in the format of:

1. As an <actor>, I want a <feature>, so that <benefit>

<user-story-example>
1. As a mobile bank customer, I want to see balance on my accounts, so that I can make better informed decisions about my spending
</user-story-example>

This list of user stories should be extremely extensive and cover all aspects of the feature.

## Implementation Decisions

A list of implementation decisions that were made. This can include:

- The modules that will be built/modified
- The interfaces of those modules that will be modified
- Technical clarifications from the developer
- Architectural decisions
- Schema changes
- API contracts
- Specific interactions

Do NOT include specific file paths or code snippets. They may end up being outdated very quickly.

## Testing Decisions

A list of testing decisions that were made. Include:

- A description of what makes a good test (only test external behavior, not implementation details)
- Which modules will be tested
- Prior art for the tests (i.e. similar types of tests in the codebase)

## Out of Scope

A description of the things that are out of scope for this PRD.

## Further Notes

Any further notes about the feature.

</prd-template>
