---
name: write-tech-spec
description: Generate a technical specification document from an existing PRD through relentless technical interview and codebase exploration. Use when user wants to write a tech spec, technical specification, technical design document, TDD (technical design doc), or go deeper on the technical details of a PRD.
---

This skill generates a technical specification from an existing PRD. You may skip steps if you don't consider them necessary.

1. Ask the user for the PRD. This can be a GitHub issue URL/number, a Jira issue key, a Confluence page URL, or pasted text. Fetch and read the PRD content.

2. Explore the codebase to understand the current state of the areas that will be affected by the PRD.

3. Interview the user relentlessly about every technical aspect of this feature until you reach a shared understanding. Walk down each branch of the technical decision tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer based on your codebase exploration and technical judgment.

Ask the questions one at a time. If a question can be answered by exploring the codebase, explore the codebase instead of asking.

Cover these areas (skip any that don't apply):

- **Architecture**: Component boundaries, data flow, new vs modified modules. Look for opportunities to extract deep modules with simple, testable interfaces.
- **API contracts**: Endpoints, request/response shapes, error codes, versioning.
- **Data model**: Schema changes, migrations, backward compatibility.
- **Security**: Authentication, authorization, input validation, data exposure.
- **Performance**: Expected load, caching strategy, known bottlenecks, scalability.
- **Error handling & failure modes**: What can go wrong, recovery strategies, retry policies.
- **Observability**: Logging, metrics, alerting, dashboards.
- **Migration & rollout**: Feature flags, phased rollout, rollback plan, data backfill.
- **Dependencies**: External services, new libraries, version constraints.
- **Edge cases**: Concurrency, rate limits, backward compatibility, data integrity.

4. Choose destination and verify access.

Ask the user: **"Where should I create this tech spec — GitHub, Jira, or Confluence?"**

**If GitHub:**
1. Run `gh auth status` to verify authentication. If not authenticated, ask the user to run `! gh auth login`.
2. Run `gh repo view --json nameWithOwner -q .nameWithOwner` to identify the current repository. Show the user the repo name and ask: **"I'll create the tech spec in `<owner/repo>`. Is that correct?"** Wait for confirmation before proceeding.

**If Jira:**
1. Use the Atlassian MCP tool `getVisibleJiraProjects` to verify connectivity and list available projects. If the connection fails, tell the user their Jira/Atlassian integration is not authenticated and ask them to set it up.
2. Show the user the list of available projects and ask: **"Which Jira project should I create the tech spec in?"** Wait for their selection before proceeding.

**If Confluence:**
1. Use the Atlassian MCP tool `getConfluenceSpaces` to verify connectivity and list available spaces. If the connection fails, tell the user their Confluence/Atlassian integration is not authenticated and ask them to set it up.
2. Show the user the list of available spaces and ask: **"Which Confluence space should I create the tech spec in?"** Wait for their selection before proceeding.
3. Optionally ask if they want it under a specific parent page.

5. Create the document.

**If GitHub:** Create the issue using `gh issue create`. Populate as many fields as possible:
- Add labels (e.g., `tech-spec`, `architecture`). Use `gh label list` to check available labels first; create missing labels with `gh label create` if needed.
- If the repo uses milestones or projects, assign them when relevant.
- If the PRD is a GitHub issue, reference it in the body (e.g., "Technical specification for #123").

**If Jira:** Create the issue using the Atlassian MCP tool `createJiraIssue` with the selected project. Use issue type "Story" unless the user specifies otherwise. Use `additional_fields` to populate:
- **Priority**: Set based on complexity and risk discussed during the interview — `"Highest"` for critical infrastructure changes, `"High"` for complex multi-system changes, `"Medium"` for standard feature work, `"Low"` for isolated changes.
- **Labels**: Add relevant labels (e.g., `["tech-spec", "architecture"]` plus any domain-specific labels discussed).
- If the PRD is a Jira issue, link the tech spec to it using `createIssueLink`.

**If Confluence:** Create the page using the Atlassian MCP tool `createConfluencePage` in the selected space. If the user specified a parent page, set it as the parent.

Use the template below to write the tech spec.

<tech-spec-template>

## Overview

Brief summary of the feature being specified. Reference the source PRD.

## Architecture

High-level design of the system changes. Include:

- Component diagram (which modules are created, modified, or removed)
- Data flow between components
- Integration points with existing systems
- Key design decisions and their rationale

## API Contracts

For each new or modified endpoint/interface:

- Method, path, and description
- Request schema (with types and constraints)
- Response schema (with types and status codes)
- Error codes and their meanings
- Versioning strategy (if applicable)

## Data Model

- New or modified tables/collections/schemas
- Field definitions with types, constraints, and defaults
- Migration strategy (additive vs destructive, zero-downtime requirements)
- Backward compatibility considerations
- Indexing strategy

## Security

- Authentication and authorization requirements
- Input validation rules
- Data exposure risks and mitigations
- Relevant threat model considerations

## Performance

- Expected load and throughput targets
- Caching strategy (what, where, TTL, invalidation)
- Known bottlenecks and mitigation plans
- Database query performance considerations

## Error Handling & Failure Modes

- Enumerated failure scenarios and recovery strategies
- Retry policies and circuit breaker configuration
- Graceful degradation behavior
- User-facing error messages

## Observability

- Key metrics to track
- Logging strategy (what to log, at what level)
- Alerting thresholds and escalation
- Dashboard requirements

## Migration & Rollout

- Feature flag strategy
- Phased rollout plan (if applicable)
- Rollback procedure
- Data backfill requirements (if applicable)

## Dependencies

- External services and their SLAs
- New libraries or version upgrades
- Infrastructure changes required

## Edge Cases

Specific edge cases identified during the interview and how they are handled.

## Test Strategy

- What to test and at what level (unit, integration, e2e)
- Which modules require dedicated test coverage
- Prior art for similar tests in the codebase

## Open Questions

Any unresolved technical questions that need further investigation.

</tech-spec-template>
