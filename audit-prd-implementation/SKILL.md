---
name: audit-prd-implementation
description: Audit a completed PRD implementation for completeness, correctness, code quality, consistency, and edge case handling. Use when a PRD has been fully implemented and the user wants a thorough review to ensure nothing was missed, no bugs were introduced, and the code is clean.
---

# PRD Implementation Audit

Perform a thorough audit of a completed PRD implementation. Walk through every requirement, locate the implementing code, verify behavior, and report categorized findings.

## Process

### 1. Locate the PRD

Ask the user for the PRD source. Accept any of:

- **GitHub issue number**: Fetch with `gh issue view <number>` (include comments with `--comments`).
- **Jira issue key**: Fetch using the Atlassian MCP tool `getJiraIssue`.
- **Local file path**: Read directly with the Read tool.

Extract all acceptance criteria, user stories, and requirements into a working checklist. Number each requirement for reference throughout the audit.

### 2. Locate child work items (if applicable)

For GitHub or Jira PRDs, find the child issues that break down the PRD:

**If GitHub:** Search for issues that reference the PRD (e.g., `gh issue list --search "parent PRD #<number>"`). If that yields nothing, ask the user for the issue numbers of the work items.

**If Jira:** Use `searchJiraIssuesUsingJql` with `parent = <PRD-KEY>` or `"Epic Link" = <PRD-KEY>`. If that yields nothing, search for issues that mention the PRD key in their description. If still nothing, ask the user for the issue keys.

Read through all child issues and extract any additional acceptance criteria or implementation details not in the parent PRD. Add these to the working checklist.

**If local file:** Skip this step unless the user indicates there are related issues to check.

### 3. Locate the implementation code

Use a layered discovery approach to build a map of **requirement → implementing code locations**:

**Layer 1 — Trace from issues:** Check child issues (and the PRD itself) for linked pull requests, branches, or commits. Use `gh pr list`, `gh pr view`, or Jira issue remote links to find PRs. Read the diffs and changed files to identify what code was added or modified.

**Layer 2 — Autonomous exploration:** If Layer 1 yields insufficient results, explore the codebase using the PRD requirements as search terms. Use the Agent tool with `subagent_type=Explore` to find relevant files, endpoints, components, database schemas, and configuration related to each requirement.

**Layer 3 — Ask the user:** If the implementation code is still unclear after Layers 1 and 2, ask the user to point to the relevant files, directories, or branches.

Present the requirement-to-code map to the user and ask if it looks complete before proceeding with the audit.

### 4. Audit each requirement

Walk through each requirement from the checklist sequentially. For every acceptance criterion or user story, perform these checks:

#### Completeness
- Does implementing code exist for this requirement?
- Is the full requirement addressed, or only partially?
- Are all parts of the acceptance criterion covered (not just the happy path description)?

#### Correctness
- Read the implementing code and reason about whether it actually does what the criterion specifies.
- Trace the logic: data flow, state transitions, conditional branches, return values.
- Check that the code handles the requirement as specified, not a subtly different interpretation.

#### Code Quality
- Look for obvious bugs: off-by-one errors, null dereferences, race conditions, resource leaks.
- Check for dead code, leftover TODO/FIXME/HACK comments, hardcoded values that should be configurable.
- Check for security issues: injection vulnerabilities, authentication/authorization gaps, exposed secrets, unsafe deserialization, missing input validation at system boundaries.

#### Consistency
- Does the implementation follow existing codebase patterns (naming conventions, file organization, architectural layers)?
- Are similar things done in similar ways, or does the new code introduce a different style?
- Does it use the same libraries, utilities, and abstractions that the rest of the codebase uses for similar tasks?

#### Edge Cases
- Are error paths handled? What happens when external calls fail, inputs are empty/null, or data is malformed?
- Are boundary conditions addressed (empty lists, max values, concurrent access)?
- What happens when things go wrong — does the code fail gracefully or crash?

### 5. Active verification

Run commands to verify the implementation behaves correctly:

1. **Build check:** Run the project's build command. Note any errors or warnings.
2. **Test suite:** Run the project's test suite. Note any failures, and check whether failures relate to the PRD requirements.
3. **Endpoint/command verification:** If the implementation exposes API endpoints, CLI commands, or other entry points, exercise them where possible. Compare actual behavior against expected behavior from the PRD.

Identify the build and test commands by checking package.json scripts, Makefile targets, CI configuration, or README instructions. If unclear, ask the user.

### 6. Cross-cutting concerns

After auditing all requirements individually, look for issues that span multiple requirements:

- **Integration gaps:** Do components implemented for different requirements work together correctly? Are there missing connections between them?
- **Inconsistent patterns:** Is error handling done differently across requirements? Are naming conventions inconsistent between related components?
- **Missing shared logic:** Are there cases where multiple requirements duplicate logic that should be shared?
- **State management:** If multiple requirements modify shared state, are there race conditions or ordering issues?

### 7. Present findings

Print the audit report directly in the conversation using this structure:

<audit-report-template>

## PRD Implementation Audit: <PRD Title>

### Summary
- **PRD source:** <issue number/key/file path>
- **Requirements audited:** <N>
- **Findings:** <N total> (Completeness: N, Correctness: N, Code Quality: N, Consistency: N, Edge Cases: N)
- **Build status:** <pass/fail — include relevant error if failed>
- **Test suite status:** <pass/fail (X passed, Y failed) — include relevant failures if any>

### Completeness

<For each finding, reference the requirement number and describe what is missing or incomplete. Include the file path and line number where the implementation was expected or is partial.>

- **Requirement N — <short name>:** <description of gap>. (`path/to/file.ext:NN`)

_No issues found._ (if none)

### Correctness

- **Requirement N — <short name>:** <description of incorrect behavior>. (`path/to/file.ext:NN`)

_No issues found._ (if none)

### Code Quality

- **<short description>:** <description of issue>. (`path/to/file.ext:NN`)

_No issues found._ (if none)

### Consistency

- **<short description>:** <description of inconsistency>. (`path/to/file.ext:NN`)

_No issues found._ (if none)

### Edge Cases

- **Requirement N — <short name>:** <description of unhandled edge case>. (`path/to/file.ext:NN`)

_No issues found._ (if none)

### Verdict

<One-paragraph overall assessment. Is this implementation ready, or does it need more work? What are the most important findings to address first? Call out any findings that are particularly high-risk.>

</audit-report-template>

**Rules for findings:**
- Every finding references the specific requirement it relates to (where applicable) and the file/line where the issue was observed.
- If a category has no findings, show "No issues found" so the user sees it was checked.
- The verdict should be honest and direct — if the implementation is solid, say so. If it needs significant work, say that too.
- Do not suggest fixes or offer to make changes. Report only.
