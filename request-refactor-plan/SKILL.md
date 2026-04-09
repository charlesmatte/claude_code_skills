---
name: request-refactor-plan
description: Create a detailed refactor plan with tiny commits via user interview, then file it as a GitHub or Jira issue. Use when user wants to plan a refactor, create a refactoring RFC, or break a refactor into safe incremental steps.
---

This skill will be invoked when the user wants to create a refactor request. You should go through the steps below. You may skip steps if you don't consider them necessary.

1. Ask the user for a long, detailed description of the problem they want to solve and any potential ideas for solutions.

2. Explore the repo to verify their assertions and understand the current state of the codebase.

3. Ask whether they have considered other options, and present other options to them.

4. Interview the user about the implementation. Be extremely detailed and thorough.

5. Hammer out the exact scope of the implementation. Work out what you plan to change and what you plan not to change.

6. Look in the codebase to check for test coverage of this area of the codebase. If there is insufficient test coverage, ask the user what their plans for testing are.

7. Break the implementation into a plan of tiny commits. Remember Martin Fowler's advice to "make each refactoring step as small as possible, so that you can always see the program working."

8. Choose issue tracker and verify access.

Ask the user: **"Should I create this issue in GitHub or Jira?"**

**If GitHub:**
1. Run `gh auth status` to verify authentication. If not authenticated, ask the user to run `! gh auth login`.
2. Run `gh repo view --json nameWithOwner -q .nameWithOwner` to identify the current repository. Show the user the repo name and ask: **"I'll create the issue in `<owner/repo>`. Is that correct?"** Wait for confirmation before proceeding.

**If Jira:**
1. Use the Atlassian MCP tool `getVisibleJiraProjects` to verify connectivity and list available projects. If the connection fails, tell the user their Jira/Atlassian integration is not authenticated and ask them to set it up.
2. Show the user the list of available projects and ask: **"Which Jira project should I create the issue in?"** Wait for their selection before proceeding.

9. Create the issue with the refactor plan.

**If GitHub:** Create the issue using `gh issue create`. Populate as many fields as possible:
- Add labels for the type of work (e.g., `refactor`, `tech-debt`). Use `gh label list` to check available labels first; create missing labels with `gh label create` if needed.
- If the repo uses milestones or projects, assign them when relevant.

**If Jira:** Create the issue using the Atlassian MCP tool `createJiraIssue` with the selected project. Use issue type "Task" unless the user specifies otherwise. Use `additional_fields` to populate:
- **Priority**: Set based on the urgency discussed with the user — `"High"` if the refactor unblocks other work or fixes fragile code, `"Medium"` for typical tech-debt cleanup, `"Low"` for nice-to-have improvements. Use `{"priority": {"name": "<level>"}}`.
- **Labels**: Add relevant labels (e.g., `["refactor", "tech-debt"]`).
- **Time estimate**: Set `timetracking.originalEstimate` based on the number of commits in the plan. Count ~30min per tiny commit as a baseline (e.g., 10 commits → `"1d"`, 20 commits → `"2d"`). Adjust up for commits that involve schema changes or cross-cutting concerns.

Use the following template for the issue description:

<refactor-plan-template>

## Problem Statement

The problem that the developer is facing, from the developer's perspective.

## Solution

The solution to the problem, from the developer's perspective.

## Commits

A LONG, detailed implementation plan. Write the plan in plain English, breaking down the implementation into the tiniest commits possible. Each commit should leave the codebase in a working state.

## Decision Document

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

A description of the things that are out of scope for this refactor.

## Further Notes (optional)

Any further notes about the refactor.

</refactor-plan-template>
