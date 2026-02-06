---
schema_version: 1
date: 2026-02-03
type: plan
status: draft
topic: "Hybrid Decision Records Proposal (Central + Local)"

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

# Plan: Hybrid Decision Records Proposal

## Desired End State

README.md describes a **hybrid** approach to decision documentation: durable decisions (plans, research) are centralized in a dedicated AI documentation repository for cross-project visibility, while handoffs default to local in each code repo's `ai_docs/handoffs/` directory (gitignored by default, optionally committed). Engineers discover centralized decisions via AGENTS.md pointers and grep commands; handoffs are session-scoped by default.

## Overview

Modify the current proposal (which describes per-project `ai_docs/` directories) to instead describe a hybrid model where:

- A central AI documentation repo contains durable decision documentation (`plans/`, `research/`)
- Flat structure at root (no project subdirs), project association via frontmatter `project:` field
- Code repos point to central repo via AGENTS.md
- Handoffs stay local in each code repo's `ai_docs/handoffs/` directory, gitignored
- Handoffs are typically ephemeral session state — local by default, optionally committed

## What We're NOT Doing

- Not specifying skill/tooling implementation (structure proposal only)
- Not keeping a `tickets/` directory (minimal structure)
- Not using project subdirectories (flat with frontmatter)
- Not requiring PRs for doc commits (direct commits, reviewed via code PR)
- Not including personal/engineer-specific directories
- Not centralizing handoffs by default (ephemeral session state, optionally committed locally)

## Assumptions

- Teams will clone the AI documentation repo as a sibling directory
- AGENTS.md is the primary discovery mechanism for AI agents
- Code PRs will reference decision docs via links
- Decision docs will back-reference implementing PRs via `related_prs:`
- `rg` (ripgrep) on `repo:` frontmatter is sufficient for filtering
- Handoffs are typically session-scoped context — local by default
- Handoffs are gitignored in code repos to avoid commit noise

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

### Option C: Centralized flat structure

**Pros:** Simple structure, cross-project search easy, project via frontmatter flexible
**Cons:** Relies on frontmatter discipline, harder to browse by project in filesystem; centralizes handoffs that are ephemeral session state with no cross-project value

### Option D: Hybrid — central durable docs, local handoffs (chosen)

**Pros:** Durable decisions get cross-project visibility; handoffs stay where they're useful (local, session-scoped); code repos stay clean of plans/research; no noise from gitignored handoffs in central repo
**Cons:** Two locations to understand; AGENTS.md must explain both; slightly more complex mental model than pure centralized

**Decision:** Option D — hybrid model. Plans and research centralize in the AI documentation repo for cross-project discovery. Handoffs default to local in each code repo's `ai_docs/handoffs/` (gitignored by default), with the option to commit when context has lasting value.

## Implementation Phases

### Phase 1: Update Core Sections

**Goal:** Replace executive summary and directory structure with hybrid approach

**Tasks:**

- [ ] Read current README.md in full
- [ ] Replace Executive Summary section
  - Reframe around hybrid model: durable docs in central AI documentation repo, handoffs local
  - Key principles: centralized discovery for plans/research, local ephemeral handoffs, human accountability via code PR
- [ ] Replace Directory Structure section
  - Central repo: `plans/`, `research/`, `templates/` at root (no `handoffs/`)
  - Local repo: `ai_docs/handoffs/` (gitignored)
  - Show "cloned as siblings" example
  - Document optional `<project>` in filename convention
- [ ] Verify changes coherent

**Success Criteria:**

- Executive summary describes hybrid model clearly
- Directory structure shows both central and local layouts

### Phase 2: Update Schema and Artifact Types

**Goal:** Add project field to schema, clarify handoff scope

**Tasks:**

- [ ] Update Document Schema section
  - Add `project: <name>` field (logical project/service name)
  - Add `repo: org/repo` field (GitHub org/repo identifier)
  - Add `repos: [org/a, org/b]` field (array, for cross-repo docs)
  - Add `related_prs:` field (list of PR URLs implementing the decision)
  - Mark as optional but recommended
  - Note: handoffs use minimal frontmatter, no project fields needed
- [ ] Keep Artifact Types section largely unchanged
  - Plan, Research descriptions remain valid — centralized context
  - Handoff description updated: local by default, gitignored, optionally committed

**Success Criteria:**

- Schema shows new project/projects fields
- Artifact types reflect hybrid model (handoffs are local)

### Phase 3: Add Discovery and Workflow Sections

**Goal:** AGENTS.md template and workflow reflecting hybrid split

**Tasks:**

- [ ] Add "Discovery: AGENTS.md Template" section
  - Template points to central repo for plans/research
  - Template points to local `ai_docs/handoffs/` for session state
  - Grep commands for finding project docs in central repo
  - Clone instructions for sibling setup
- [ ] Update Workflow section
  - Plans/research: direct commits to AI documentation repo, referenced from code PR
  - Handoffs: created locally in `ai_docs/handoffs/`, gitignored by default (optionally committed)
  - Cross-reference pattern between repos (for durable docs only)
- [ ] Update PR Template Integration
  - Link to AI documentation repo for plans/research

**Success Criteria:**

- AGENTS.md template is copy-pasteable and shows both locations
- Workflow clearly describes hybrid commit pattern

### Phase 4: Add Tradeoffs, .gitignore, and Update Remaining Sections

**Goal:** Three-way tradeoffs comparison, .gitignore setup, update tooling/adoption

**Tasks:**

- [ ] Add "Tradeoffs: Per-Project vs Centralized vs Hybrid" section
  - Three-column comparison table
  - When each approach works better
- [ ] Add .gitignore guidance
  - Code repos add `ai_docs/handoffs/` to .gitignore
  - Example .gitignore snippet
- [ ] Update Tooling section
  - Pre-commit hooks apply to AI documentation repo (plans/research)
  - No tooling needed for local handoffs
- [ ] Update Adoption Considerations
  - Two-repo setup for central docs
  - .gitignore setup for local handoffs
  - AGENTS.md rollout
- [ ] Update or remove Discoverability section
  - AGENTS.md integration is now "Discovery" section

**Success Criteria:**

- Tradeoffs table fairly presents all three approaches
- .gitignore setup is documented
- Tooling section makes sense for hybrid model

### Phase 5: Final Review and Cleanup

**Goal:** Ensure document is coherent and complete

**Tasks:**

- [ ] Read full modified README
- [ ] Check for orphaned references to centralizing handoffs
  - Update or remove as appropriate
- [ ] Verify all sections flow logically
- [ ] Update Next Steps section
- [ ] Run markdownlint: `markdownlint-cli2 README.md`

**Success Criteria:**

- Document reads coherently from start to finish
- No references to centralizing handoffs
- Handoffs described as local by default throughout
- Markdown lint passes

## Critical Files

- `/Users/sprice/code/work/coding-agent-documentation/README.md` - The document to modify

## Key Content to Include

### Central AI Documentation Repo Structure

```text
<ai-docs-repo>/
├── README.md
├── plans/
│   ├── YYYY-MM-DD-<topic>.md              # project in frontmatter
│   └── YYYY-MM-DD-<project>-<topic>.md    # project in filename (optional)
├── research/
│   └── YYYY-MM-DD-<topic>.md
└── templates/
```

**Naming convention:** Including `<project>` in the filename is optional but recommended when the topic is project-specific. Cross-cutting docs should omit it.

### Local Code Repo Structure

```text
my-service/
├── AGENTS.md          # points to central repo for plans/research
├── .gitignore         # includes ai_docs/handoffs/
└── ai_docs/
    └── handoffs/      # local by default, gitignored
        └── YYYY-MM-DD-<context>.md
```

**Handoffs are local by default.** They are session state for AI agents — typically gitignored and not committed. Teams may choose to commit handoffs that contain context worth preserving.

### New Frontmatter Fields

```yaml
project: conductor                  # Logical project/service name
repo: org/conductor                 # GitHub org/repo (canonical identifier)
repos: [org/conductor, org/api-gateway]  # For cross-repo docs
related_prs:                        # PRs that implement this decision
  - https://github.com/org/conductor/pull/123
```

### AGENTS.md Template for Code Repos

```markdown
## Decision Records

Plans and research for this project are centralized at:
**Repository:** [github.com/org/<ai-docs-repo>](https://github.com/org/<ai-docs-repo>)

Find this project's docs by searching for `repo: org/<this-repo>` in frontmatter.

### Discovery

Clone as sibling directory:
git clone git@github.com:org/<ai-docs-repo>.git ../<ai-docs-repo>

Search for this repo's docs:
rg -l "repo: org/<this-repo>" ../<ai-docs-repo>/plans/
rg -l "repos:.*org/<this-repo>" ../<ai-docs-repo>/plans/  # cross-repo docs

### Before Starting Work

Check for existing context:
- Plans: `../<ai-docs-repo>/plans/`
- Research/decisions: `../<ai-docs-repo>/research/`
- Handoffs from previous sessions: `ai_docs/handoffs/` (local, gitignored)

### Handoffs (Local)

Session handoffs live in this repo at `ai_docs/handoffs/` (gitignored).
These are ephemeral — create them to pass context between sessions,
but do not commit or centralize them.
```

### Tradeoffs Table

| Aspect | Per-Project | Centralized | Hybrid (chosen) |
|--------|-------------|-------------|-----------------|
| Discovery | Natural — agents find while exploring | Requires AGENTS.md instruction | AGENTS.md for plans/research; handoffs local |
| Code repo size | Grows with docs | Stays clean | Clean (handoffs gitignored) |
| Cross-project search | Difficult | Easy | Easy for durable docs; handoffs N/A |
| Doc-code atomicity | Same commit/PR | Separate, cross-referenced | Plans/research separate; handoffs uncommitted |
| Drift risk | Low | Higher | Moderate — handoffs can't drift (disposable) |
| Handoff ergonomics | Good — local and discoverable | Poor — noise in central repo | Good — local, gitignored, session-scoped |

## Verification

**Manual verification:**

- [ ] Read full modified README for coherence
- [ ] Verify no orphaned ai_docs/ references (except tradeoffs)
- [ ] Check AGENTS.md template is complete and usable
