---
name: github-projects-graphql
description: Use when managing GitHub Projects v2 through gh GraphQL, including querying projects, fields, items, adding repository issues or draft issues to projects, and updating project item status fields.
---

# GitHub Projects GraphQL

## Overview

Use `gh api graphql` for GitHub Projects v2 operations that the GitHub connector does not expose directly. Prefer read-only queries first, then mutate only after identifying exact node IDs and field IDs.

## Requirements

- `gh` must be installed.
- `gh api graphql` must work against GitHub.
- Projects queries require `read:project`.
- Project mutations require `project`.
- Network calls often need elevated execution in restricted sandboxes.

Check access:

```bash
gh api graphql -f query='query { viewer { login } }'
gh api graphql -f query='query { viewer { projectsV2(first: 20) { nodes { id number title closed } } } }'
```

If Projects scopes are missing:

```bash
gh auth refresh -h github.com -s read:project -s project
```

## Core Queries

Repository node ID:

```bash
gh api graphql -f query='query {
  repository(owner: "OWNER", name: "REPO") {
    id
    nameWithOwner
    url
  }
}'
```

User projects:

```bash
gh api graphql -f query='query {
  user(login: "OWNER") {
    projectsV2(first: 20) {
      nodes { id number title closed }
    }
  }
}'
```

Project fields and Status options:

```bash
gh api graphql -f query='query {
  user(login: "OWNER") {
    projectV2(number: PROJECT_NUMBER) {
      id
      title
      fields(first: 50) {
        nodes {
          ... on ProjectV2Field { id name dataType }
          ... on ProjectV2SingleSelectField {
            id name dataType
            options { id name }
          }
        }
      }
    }
  }
}'
```

## Common Mutations

Create a user project:

```bash
OWNER_ID=$(gh api graphql -q '.data.viewer.id' -f query='query { viewer { id } }')
gh api graphql \
  -f ownerId="$OWNER_ID" \
  -f title="simple-video-edit" \
  -f query='mutation($ownerId: ID!, $title: String!) {
    createProjectV2(input: { ownerId: $ownerId, title: $title }) {
      projectV2 { id number title }
    }
  }'
```

Create a repository issue:

```bash
gh issue create \
  --repo OWNER/REPO \
  --title "Task title" \
  --body "Task body"
```

Get an issue node ID:

```bash
gh api graphql -f query='query {
  repository(owner: "OWNER", name: "REPO") {
    issue(number: ISSUE_NUMBER) { id number title }
  }
}'
```

Add issue to project:

```bash
gh api graphql \
  -f projectId="PROJECT_ID" \
  -f contentId="ISSUE_NODE_ID" \
  -f query='mutation($projectId: ID!, $contentId: ID!) {
    addProjectV2ItemById(input: { projectId: $projectId, contentId: $contentId }) {
      item { id }
    }
  }'
```

Update a single-select Status field:

```bash
gh api graphql \
  -f projectId="PROJECT_ID" \
  -f itemId="PROJECT_ITEM_ID" \
  -f fieldId="STATUS_FIELD_ID" \
  -f optionId="STATUS_OPTION_ID" \
  -f query='mutation($projectId: ID!, $itemId: ID!, $fieldId: ID!, $optionId: String!) {
    updateProjectV2ItemFieldValue(input: {
      projectId: $projectId
      itemId: $itemId
      fieldId: $fieldId
      value: { singleSelectOptionId: $optionId }
    }) {
      projectV2Item { id }
    }
  }'
```

## Workflow

1. Verify `gh api graphql` with `viewer.login`.
2. Query repository ID and confirm the target repository.
3. Query user or organization Projects v2.
4. Query target project fields and identify `Status` field plus option IDs.
5. Create Issues through `gh issue create` or the GitHub connector.
6. Query Issue node IDs.
7. Add Issues to Project with `addProjectV2ItemById`.
8. Update project fields only after confirming item ID, field ID, and option ID.

## Safety Rules

- Do not guess IDs. Query them.
- Do not mutate Projects until the target owner, repo, project number, field name, and option name are confirmed.
- Prefer Issues over Draft Issues when tasks should link to code work.
- Report scope errors exactly; missing Projects permissions usually show `INSUFFICIENT_SCOPES` and require `read:project` or `project`.
- In restricted sandboxes, retry GitHub API calls with approved escalation rather than changing the workflow.
