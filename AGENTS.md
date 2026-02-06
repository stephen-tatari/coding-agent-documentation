# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Repository Purpose

This repository contains an engineering proposal for "Decision Records with AI Assistance" - a convention for preserving the "why" behind decisions as versioned artifacts alongside code.

**This is a documentation repository, not a software project.** There is no code to build, test, or lint.

## Active Direction

README.md is the source of truth for the hybrid model convention. The plan file `ai_docs/plans/2026-02-03-centralized-decision-records.md` captures the original rationale for moving to this model. **Read README.md before proposing structural changes.**

Key idea: durable decisions (plans, research) centralize in a dedicated AI documentation repo; ephemeral handoffs remain local in each code repo's `ai_docs/handoffs/` (gitignored).

## Directory Structures

The hybrid model uses two locations.

### Central AI Documentation Repository (this repo)

```text
<ai-docs-repo>/
├── index.md               # catalog of all docs with topic, project, status
├── plans/
│   ├── YYYY-MM-DD-<topic>.md              # project in frontmatter
│   └── YYYY-MM-DD-<project>-<topic>.md    # project in filename (optional)
├── research/
│   └── YYYY-MM-DD-<topic>.md
└── templates/
```

### Local Code Repository

```text
my-service/
├── AGENTS.md          # points to central repo for plans/research
├── .gitignore         # includes ai_docs/handoffs/
└── ai_docs/
    └── handoffs/      # local by default, gitignored
        └── YYYY-MM-DD-<context>.md
```

Handoffs are local by default — session state for AI agents, typically gitignored and not committed.

## Document Structure

- `ai_docs/plans/` — Implementation plans with frontmatter (`YYYY-MM-DD-<topic>.md`).
- `ai_docs/research/` — Research notes and ADRs with frontmatter (`YYYY-MM-DD-<topic>.md`).
- `ai_docs/handoffs/` — Session continuity docs (`YYYY-MM-DD-<context>.md`). Ephemeral, not committed in code repos per the hybrid model.
- `ai_docs/templates/` — Scaffolds for each artifact type.

`README.md` is the main proposal document covering the full convention.

**Note:** The `thoughts/` directory at the repository root is legacy and unused. Do not create new files there.

## Frontmatter Schema

All documents in `ai_docs/plans/` and `ai_docs/research/` must include this frontmatter:

```yaml
---
schema_version: 1
date: YYYY-MM-DD
type: plan | research | handoff
status: draft | active | superseded | archived
topic: "Short description of the topic"

# Accountability
author: name                          # Human owner (run: git config user.name)
ai_assisted: true                     # explicit flag
ai_model: claude-3.5-sonnet           # optional: which model

# Project association (optional; recommended for plans/research)
project: my-service                   # Logical project/service name
repo: org/my-service                  # GitHub org/repo (canonical identifier)
repos: [org/my-service, org/api-gateway]  # For cross-repo docs

# Linking
related_prs:                          # PRs that implement this decision
  - https://github.com/org/repo/pull/123
related_issue: "org/repo#456"
superseded_by: "YYYY-MM-DD-topic-v2.md"

# Classification
tags: [tag1, tag2]
data_sensitivity: public | internal | restricted
---
```

**Required fields:** `schema_version`, `date`, `type`, `status`, `topic`

**Conditional:** `ai_assisted` required if AI was used

**Optional but recommended (plans/research):** `project`, `repo` (or `repos` for cross-repo docs), `related_prs`

**Note:** Local handoffs use minimal frontmatter. Committed handoffs may include project association fields for discoverability. The field `related_pr` (singular) is deprecated — use `related_prs` (plural, array).

## Quality Constraints

- Keep individual documents under 500 lines
- Claims must be linked to sources (code refs, docs, external links)
- Assumptions must be explicitly listed
- Alternatives must be considered (for decision-focused research)
- No secrets, credentials, or sensitive data
- No internal URLs that shouldn't be exposed

## Working on This Repository

- All changes are to Markdown documentation
- The proposal follows standard Markdown formatting
- Cross-reference sections within README.md when making edits
- Preserve the structured format (frontmatter examples, tables, code blocks)
