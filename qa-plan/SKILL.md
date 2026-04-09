---
name: qa-plan
description: Generate a dual-track QA plan (manual + automated) after implementing a PRD and its work items. Use when user wants to create a QA plan, test plan, acceptance testing checklist, or verify a completed PRD implementation.
---

# QA Plan

Generate a comprehensive QA plan with two tracks — manual (human tester) and automated (test infrastructure) — covering every acceptance criterion from the PRD and its child work items.

## Process

### 1. Locate the PRD and work items

Ask the user for the PRD issue number (or URL). This may be a GitHub issue or a Jira issue.

**If GitHub:** Fetch the PRD with `gh issue view <number>`. Then find child issues by searching for issues that reference it (e.g., `gh issue list --search "parent PRD #<number>"`). If that yields nothing, ask the user for the issue numbers of the work items.

**If Jira:** Fetch the PRD using `getJiraIssue`. Then find child issues:
1. Try `searchJiraIssuesUsingJql` with `parent = <PRD-KEY>` or `"Epic Link" = <PRD-KEY>`.
2. If that yields nothing, search for issues that mention the PRD key in their description.
3. If still nothing, ask the user for the issue keys.

Read through all issues (PRD + children) and extract every acceptance criterion, user story, and implementation decision.

### 2. Explore the codebase

Use the Agent tool with `subagent_type=Explore` to understand what was actually built:

- Find the test files related to the work items — identify what's already covered by automated tests
- Identify the API endpoints, commands, or UI entry points involved
- Check for any integration/E2E test infrastructure already in place (test containers, fixtures, seed data, test helpers)
- Note the test runner, test framework, and how tests are executed (e.g., `dotnet test`, `npm test`, `pytest`)

### 3. Draft the QA plan

Generate the plan using the template below. Follow these rules:

**General:**
- One section per work item (use the issue title as the section heading)
- State the **Goal** of each section in one sentence — what the work item achieved
- Every acceptance criterion from every work item MUST appear as at least one test case
- Add edge cases and negative cases the acceptance criteria imply but don't state explicitly
- End with cross-cutting End-to-End Workflow Scenarios that exercise the full lifecycle across multiple work items

**Test case tables:**
- Use columns: `#`, `Test Case`, `Steps`, `Expected`
- Steps should be concrete — specify the HTTP method/endpoint, UI action, CLI command, or DB query
- Expected should be specific — status codes, field values, error messages, row counts
- Number test cases sequentially within each section (1, 2, 3...)
- Number E2E scenarios with an E-prefix (E1, E2, E3...)

**Two tracks — mark each test case:**
- **Manual track** (`[M]`): Tests that require human judgment, visual inspection, complex multi-step workflows with real UI, or scenarios that can't be reliably automated. These are performed by a real user in a test environment.
- **Automated track** (`[A]`): Tests that assert on deterministic outcomes via API calls, DB queries, or CLI commands. These run in CI or via the project's test infrastructure.
- A test case can be both `[M+A]` if it should be verified by both tracks (e.g., automated API check + manual UI verification of the same behavior).
- Default to `[A]` unless there's a reason the test needs human eyes. Be aggressive about automation.

**Prerequisites:**
- List every prerequisite: running services, seed data, authenticated sessions, environment setup
- Be specific — name the actual commands or tools needed

**Verification commands:**
- Include the exact commands to run the automated test suite
- Include build/compile checks
- State the current automated test count and which test classes/files cover which work items

### 4. Present and iterate

Show the full QA plan to the user. Ask:

- Are any test cases missing for your acceptance criteria?
- Should any `[A]` tests be `[M]` instead (or vice versa)?
- Are the prerequisites complete?
- Do the E2E scenarios cover the workflows you care about most?

Iterate until the user approves.

### 5. Save the QA plan

Ask the user where they want the plan saved. Options:

- **As a file in the repo** (e.g., `docs/qa/QA-PLAN-<PRD-KEY>.md`): Write using the Write tool.
- **As a Jira issue/comment**: Create a new "Task" issue linked to the PRD, or add as a comment on the PRD issue using `addCommentToJiraIssue`.
- **As a GitHub issue/comment**: Create via `gh issue create` linked to the PRD, or add as a comment via `gh issue comment`.

If the user doesn't have a preference, default to saving as a markdown file in the repo.

<qa-plan-template>

# QA Plan: <PRD Title> (<PRD Issue Key/Number>)

## Prerequisites

- Prerequisite 1 (e.g., Docker Desktop running for test containers)
- Prerequisite 2 (e.g., seed data loaded via `<command>`)
- Prerequisite 3 (e.g., authenticated user session)

---

## <Work Item Key>: <Work Item Title>

**Goal:** One-sentence description of what this work item achieved.

| # | Track | Test Case | Steps | Expected |
|---|-------|-----------|-------|----------|
| 1 | [A] | Description | Concrete steps (endpoint, command, query) | Specific expected outcome |
| 2 | [M] | Description | Manual steps for a human tester | What the tester should see/verify |
| 3 | [M+A] | Description | Steps for both tracks | Expected for both |

---

(Repeat for each work item)

---

## End-to-End Workflow Scenarios

These cross-cutting tests exercise the full lifecycle across multiple work items:

| # | Track | Scenario | Steps | Expected |
|---|-------|----------|-------|----------|
| E1 | [M+A] | Happy path | Full workflow steps | End state |
| E2 | [A] | Error path | Steps that trigger failures | Error handling |

---

## Verification Commands

```bash
# Run all automated tests
<test command>

# Build check
<build command>
```

Current automated coverage: **N tests** across M test classes covering <work item keys>.

## Manual Test Checklist

A consolidated checklist of all `[M]` and `[M+A]` test cases for a human tester to work through:

- [ ] <Section>: Test case N — <brief description>
- [ ] <Section>: Test case N — <brief description>
- [ ] E2E: Scenario EN — <brief description>

</qa-plan-template>
