# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Repository Purpose

This repository contains an engineering proposal for "Decision Records with AI Assistance" - a convention for preserving the "why" behind decisions as versioned artifacts alongside code.

**This is a documentation repository, not a software project.** There is no code to build, test, or lint.

## Active Direction

The hybrid model plan in `ai_docs/plans/2026-02-03-centralized-decision-records.md` represents the intended direction for this proposal. README.md describes the original per-project approach and may not yet reflect the hybrid model. **Read the plan before proposing structural changes to README.md.**

Key idea: durable decisions (plans, research) centralize in a dedicated AI documentation repo; ephemeral handoffs remain local in each code repo's `ai_docs/handoffs/` (gitignored).

## Document Structure

- `ai_docs/plans/` — Implementation plans with frontmatter. Date-prefixed filenames (`YYYY-MM-DD-<topic>.md`).
- `ai_docs/research/` — Research notes and ADRs with frontmatter. Same naming convention.
- `ai_docs/handoffs/` — Session continuity docs. Ephemeral, not committed in code repos per the hybrid model.
- `ai_docs/templates/` — Scaffolds for each artifact type.

`README.md` is the main proposal document covering the full convention.

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
author: name
ai_assisted: true

# Linking
related_pr:
related_issue:

# Classification
tags: [tag1, tag2]
data_sensitivity: public | internal | restricted
---
```

**Required fields:** `schema_version`, `date`, `type`, `status`, `topic`

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
