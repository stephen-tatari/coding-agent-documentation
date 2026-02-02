---
schema_version: 1
date: 2026-02-02
type: plan
status: draft
topic: "Implement org-wide skills for decision record creation"

# Accountability
author: ai-assisted
reviewed_by: # pending
ai_assisted: true
ai_model: claude-opus-4-5

# Linking
related_pr: # pending
related_issue: # n/a

# Classification
tags: [skills, templates, adoption, tooling]
data_sensitivity: internal
---

# Plan: Org-Wide Decision Record Skills

## Overview

Implement Claude Code skills that enable any team to create decision records (plans, research, handoffs) following the conventions in this proposal. Skills will be distributable via a shared repository that teams can copy into their projects.

## Assumptions

- Teams use Claude Code as their AI coding agent
- Skills are distributed as directories containing `SKILL.md` files
- The `ai_docs/` directory structure from the proposal will be adopted
- Teams may customize frontmatter fields (e.g., add project-specific tags)

## Constraints

- Skills must work without external dependencies (no npm, no network calls)
- Must follow existing skill patterns (see `~/.dotfiles/nix/home/claude/skills/`)
- Skills should be copy-paste distributable (no installer required)
- Must not conflict with existing personal skills (different directory)

## Alternatives Considered

| Approach | Pros | Cons | Decision |
|----------|------|------|----------|
| Slash commands in `.claude/commands/` | Native Claude Code pattern | Per-project setup required | Rejected: skills are more portable |
| CLI scaffolding tool | One command setup | Requires installation, maintenance | Rejected: adds dependency |
| Repo template only | Simple for new projects | Doesn't help existing projects | Partial: use alongside skills |
| Skills in shared repo | Copy-paste portable, works anywhere | Teams must manually sync updates | **Selected** |

## Implementation Approach

Create four skills - one bootstrapper and three artifact creators:

1. `init-ai-docs` - Bootstraps `ai_docs/` structure with index.md and templates (foundation)
2. `create-plan` - Creates `ai_docs/plans/YYYY-MM-DD-<feature>.md` (depends on init-ai-docs)
3. `create-research` - Creates `ai_docs/research/YYYY-MM-DD-<topic>.md` (depends on init-ai-docs)
4. `create-handoff` - Creates `ai_docs/handoffs/YYYY-MM-DD-<context>.md` (depends on init-ai-docs)

**Key pattern:** All `create-*` skills invoke `init-ai-docs` at the start to ensure consistent directory structure. The init skill is idempotent - safe to run multiple times.

## Phases

### Phase 1: Core Skills Structure

Create the shared skills repository structure.

**Required changes:**
- Create `org-ai-docs-skills/` repository or directory
- Create `org-ai-docs-skills/init-ai-docs/SKILL.md`
- Create `org-ai-docs-skills/create-plan/SKILL.md`
- Create `org-ai-docs-skills/create-research/SKILL.md`
- Create `org-ai-docs-skills/create-handoff/SKILL.md`

**Success criteria:**
- Automated: `ls org-ai-docs-skills/*/SKILL.md` returns 4 files
- Manual: Each SKILL.md has valid YAML frontmatter

### Phase 2: Implement init-ai-docs Skill (Foundation)

**Required changes:**
- `init-ai-docs/SKILL.md`: Idempotent bootstrap skill
  - Create `ai_docs/{plans,research,handoffs,templates}/` if missing
  - Generate `ai_docs/index.md` from template (prompt for project name, terminology) if missing
  - Create template files in `ai_docs/templates/` if missing
  - Append AGENTS.md integration snippet (or create AGENTS.md if missing)
  - Skip existing files/directories (idempotent)

**Success criteria:**
- Automated: Running twice produces no errors and no duplicate content
- Manual: `ai_docs/index.md` has project-specific content filled in

### Phase 3: Implement create-plan Skill

**Required changes:**
- `create-plan/SKILL.md`: Workflow for creating plan documents
  - **First step:** Invoke `init-ai-docs` to ensure directory structure exists
  - Gather context via conversation
  - Generate filename with date prefix
  - Write frontmatter from README.md schema (reviewed_by required)
  - Include sections: Overview, Assumptions, Constraints, Alternatives, Phases
  - Each phase has success criteria (automated + manual)

**Success criteria:**
- Automated: Frontmatter validates against schema (required fields present)
- Manual: Running `/create-plan` in empty repo creates `ai_docs/` structure AND plan file

### Phase 4: Implement create-research Skill

**Required changes:**
- `create-research/SKILL.md`: Workflow for research/ADR documents
  - **First step:** Invoke `init-ai-docs` to ensure directory structure exists
  - Distinguish between exploratory research and decision records
  - For decisions: include Alternatives Considered, Decision, Consequences
  - For exploration: include Findings, Open Questions
  - Prompt for `data_sensitivity` classification
  - reviewed_by required

**Success criteria:**
- Automated: Frontmatter validates against schema
- Manual: Running `/create-research` for a decision produces ADR-style content

### Phase 5: Implement create-handoff Skill

**Required changes:**
- `create-handoff/SKILL.md`: Adapt existing personal skill
  - **First step:** Invoke `init-ai-docs` to ensure directory structure exists
  - Change output path from `thoughts/shared/handoffs/` to `ai_docs/handoffs/`
  - Update frontmatter to match README.md schema
  - reviewed_by is optional (handoffs are short-lived, consumed by next session)
  - Keep worktree detection logic (write to main repo, not worktree)

**Success criteria:**
- Automated: Output path is `ai_docs/handoffs/YYYY-MM-DD-*.md`
- Manual: Handoff document is consumable by `/resume-handoff`

### Phase 6: Documentation and Distribution

**Required changes:**
- `org-ai-docs-skills/README.md`: Installation and usage instructions
- Document how to copy skills into project or personal config
- Add examples of each skill's output

**Success criteria:**
- Automated: README exists and is non-empty
- Manual: A new team member can follow README to adopt skills

## Testing Strategy

- **Unit validation:** Each SKILL.md parses as valid YAML frontmatter
- **Integration:** Run each skill in a test repository, verify output structure
- **Adoption test:** Have one team copy skills and use them for a real feature

## Migration Notes

Teams with existing `thoughts/shared/` structure can:
1. Run `/init-ai-docs` to create new structure
2. Optionally migrate existing handoffs: `mv thoughts/shared/handoffs/* ai_docs/handoffs/`
3. Update any references in CLAUDE.md/AGENTS.md

## Open Questions

- [ ] Should skills live in a dedicated repo or a `tools/` directory in a monorepo?
- [ ] How do teams receive updates to skills? (manual re-copy vs git submodule vs symlink)

## Resolved Questions

- **`reviewed_by` for handoffs:** Optional. Handoffs are short-lived and consumed by the next session, so formal review attestation is unnecessary. (Updated in README.md)

## References

- @README.md - Decision records proposal (source of truth for schema)
- `/Users/sprice/.dotfiles/nix/home/claude/skills/create_handoff/SKILL.md` - Existing personal skill pattern
- https://github.com/humanlayer/humanlayer/.claude/commands/ - HumanLayer's command pattern
