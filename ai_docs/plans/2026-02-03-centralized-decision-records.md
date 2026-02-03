---
schema_version: 1
date: 2026-02-03
type: plan
status: draft
topic: "Centralized Decision Records Proposal"

# Accountability
author: Stephen Price
ai_assisted: true

# Linking
related_pr:
related_issue:

# Classification
tags: [documentation, decision-records, ai-docs]
data_sensitivity: internal
---

# Plan: Centralized Decision Records Proposal

## Desired End State

README.md describes a **centralized** approach to decision documentation where all docs live in a single `thoughts` repository, enabling cross-project visibility while keeping code repositories clean. Engineers can discover relevant decisions via AGENTS.md pointers and grep commands.

## Overview

Modify the current proposal (which describes per-project `ai_docs/` directories) to instead describe a centralized model where:

- A single `thoughts` repo contains all decision documentation
- Flat structure with `plans/`, `research/`, `handoffs/` at root (no project subdirs)
- Project association via frontmatter `project:` field
- Code repos point to central repo via AGENTS.md

## What We're NOT Doing

- Not specifying skill/tooling implementation (structure proposal only)
- Not keeping a `tickets/` directory (minimal structure)
- Not using project subdirectories (flat with frontmatter)
- Not requiring PRs for doc commits (direct commits, reviewed via code PR)
- Not including personal/engineer-specific directories

## Assumptions

- Teams will clone the thoughts repo as a sibling directory
- AGENTS.md is the primary discovery mechanism for AI agents
- Code PRs will reference decision docs via links
- Grep on frontmatter is sufficient for project filtering

## Constraints

- Must preserve same artifact types (Plan, Research, Handoff)
- Must maintain human accountability (via code PR review)
- Must be adoptable without custom tooling

## Alternatives Considered

### Option A: Per-project ai_docs/ (current proposal)

**Pros:** Natural discovery, atomic commits with code, simpler mental model
**Cons:** Clutters code repos, cross-project search difficult, docs scattered

### Option B: Centralized with project subdirectories

**Pros:** Clear project boundaries, easy to scope
**Cons:** "Where does cross-project stuff go?" friction, mimics what we're trying to avoid

### Option C: Centralized flat structure (chosen)

**Pros:** Simple structure, cross-project search easy, project via frontmatter flexible
**Cons:** Relies on frontmatter discipline, harder to browse by project in filesystem

**Decision:** Option C - flat structure with frontmatter-based project association. Simplest structure that enables cross-project visibility while maintaining flexibility.

## Implementation Phases

### Phase 1: Update Core Sections

**Goal:** Replace executive summary and directory structure with centralized approach

**Tasks:**

- [ ] Read current README.md in full
- [ ] Replace Executive Summary section
  - Reframe around centralization: docs live in dedicated thoughts repo
  - Key principles: referenced from code repos, single searchable source, human accountability via code PR
- [ ] Replace Directory Structure section
  - New structure: `plans/`, `research/`, `handoffs/`, `templates/` at root
  - Remove `index.md` (optional in centralized model)
  - Show "cloned as siblings" example
- [ ] Verify changes coherent

**Success Criteria:**

- Executive summary describes centralization clearly
- Directory structure shows flat layout without project subdirs

### Phase 2: Update Schema and Artifact Types

**Goal:** Add project field to schema, keep artifact types same

**Tasks:**

- [ ] Update Document Schema section
  - Add `project: <name>` field (string)
  - Add `projects: [a, b]` field (array, for cross-project)
  - Mark as optional but recommended
- [ ] Keep Artifact Types section largely unchanged
  - Plan, Research, Handoff descriptions remain valid
  - Minor wording tweaks for centralized context

**Success Criteria:**

- Schema shows new project/projects fields
- Artifact types still make sense for centralized model

### Phase 3: Add Discovery and Workflow Sections

**Goal:** New AGENTS.md template and workflow for centralized commits

**Tasks:**

- [ ] Add "Discovery: AGENTS.md Template" section
  - Template code repos include to point to central repo
  - Grep commands for finding project docs
  - Clone instructions for sibling setup
- [ ] Update Workflow section
  - Direct commits to thoughts repo
  - Reference from code PR description
  - Cross-reference pattern between repos
- [ ] Update PR Template Integration
  - Link to thoughts repo doc instead of ai_docs/

**Success Criteria:**

- AGENTS.md template is copy-pasteable
- Workflow clearly describes split commit pattern

### Phase 4: Add Tradeoffs and Update Remaining Sections

**Goal:** Tradeoffs comparison, update tooling/adoption sections

**Tasks:**

- [ ] Add "Tradeoffs: Centralized vs Per-Project" section
  - Comparison table
  - When each approach works better
- [ ] Update Tooling section
  - Pre-commit hooks apply to thoughts repo
  - CI checks in thoughts repo
- [ ] Update Adoption Considerations
  - Two-repo setup
  - AGENTS.md rollout
- [ ] Update or remove Discoverability section
  - AGENTS.md integration is now "Discovery" section
  - index.md template may be optional

**Success Criteria:**

- Tradeoffs table fairly presents both approaches
- Tooling section makes sense for centralized model

### Phase 5: Final Review and Cleanup

**Goal:** Ensure document is coherent and complete

**Tasks:**

- [ ] Read full modified README
- [ ] Check for orphaned references to ai_docs/
  - Update or remove as appropriate
- [ ] Verify all sections flow logically
- [ ] Update Next Steps section
- [ ] Run markdownlint: `markdownlint-cli2 README.md`

**Success Criteria:**

- Document reads coherently from start to finish
- No references to per-project ai_docs/ except in tradeoffs comparison
- Markdown lint passes

## Critical Files

- `/Users/sprice/code/work/coding-agent-documentation/README.md` - The document to modify

## Key Content to Include

### Central Repo Structure

```text
thoughts/
├── README.md
├── plans/
│   └── YYYY-MM-DD-<topic>.md
├── research/
│   └── YYYY-MM-DD-<topic>.md
├── handoffs/
│   └── YYYY-MM-DD-<context>.md
└── templates/
```

### New Frontmatter Fields

```yaml
project: conductor                  # Associates doc with a codebase
projects: [conductor, api-gateway]  # For cross-project docs
```

### AGENTS.md Template for Code Repos

```markdown
## Decision Records

Decision documentation for this project is centralized at:
**Repository:** [github.com/org/thoughts](https://github.com/org/thoughts)

Find this project's docs by searching for `project: <this-project-name>` in frontmatter.

### Discovery

Clone as sibling directory:
git clone git@github.com:org/thoughts.git ../thoughts

Search for project docs:
grep -l "project: <this-project-name>" ../thoughts/plans/*.md
grep -l "project: <this-project-name>" ../thoughts/research/*.md

### Before Starting Work

Check for existing context:
- Plans: `../thoughts/plans/`
- Research/decisions: `../thoughts/research/`
- Handoffs from previous sessions: `../thoughts/handoffs/`
```

### Tradeoffs Table

| Aspect | Per-Project | Centralized |
|--------|-------------|-------------|
| Discovery | Natural - agents find while exploring | Requires AGENTS.md instruction |
| Code repo size | Grows with docs | Stays clean |
| Cross-project search | Difficult | Easy |
| Doc-code atomicity | Same commit/PR | Separate, cross-referenced |
| Drift risk | Low | Higher |

## Verification

**Manual verification:**

- [ ] Read full modified README for coherence
- [ ] Verify no orphaned ai_docs/ references (except tradeoffs)
- [ ] Check AGENTS.md template is complete and usable
