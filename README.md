# Proposal: Decision Records with AI Assistance

**Goal:** Preserve the "why" behind decisions - plans, research, and rationale - as versioned artifacts alongside code.

**Audience:** Engineering organization adoption proposal

**Framing:** This extends standard Architecture Decision Records (ADRs) to include AI-assisted artifacts. The primary value is structured decision capture; AI assistance is optional and secondary.

---

## Executive Summary

Engineering decisions get made, code gets merged, and months later nobody remembers why things were done that way. This proposal establishes a convention for committing decision artifacts to git repositories, preserving institutional knowledge.

**Key principles:**
- Docs ship *with* code changes, not as separate PRs (avoids noise)
- Human accountability via reviewer attestation
- Structured directory with clear purpose
- Lifecycle management to prevent staleness
- Security-conscious: explicit policies for sensitive content

---

## Directory Structure

```
ai_docs/
├── index.md          # Master index (discovery entry point)
├── plans/            # Implementation plans (short-lived)
│   └── YYYY-MM-DD-<feature>.md
├── research/         # Research notes AND decision records (variable lifespan)
│   └── YYYY-MM-DD-<topic>.md
└── handoffs/         # Session continuity documents
    └── YYYY-MM-DD-<context>.md
```

**index.md** serves as the master index for AI agents and humans alike - a lexicon of project-specific terminology, architecture overview, and links to active documents. This follows the [SIP-189 pattern](https://github.com/apache/superset/issues/35822) for context engineering.

---

## Artifact Types

**Plan** (short-lived)
- Implementation approach before coding starts
- Expected to become obsolete once feature ships
- Default status: `archived` after merge

**Research** (variable lifespan)
- Evidence gathering, exploration, constraints discovery
- Significant decisions and their rationale (ADR-style content)
- Alternatives considered, trade-offs evaluated
- May be long-lived if it captures important architectural decisions
- Status depends on ongoing relevance

**Handoff** (session continuity)
- Preserves context between AI coding sessions
- Current state, next steps, blockers, decisions made
- Created at natural stopping points or before context limits
- Typically short-lived, consumed by next session

---

## Document Schema (Frontmatter)

```yaml
---
schema_version: 1
date: 2026-01-30
type: plan | research | handoff
status: draft | active | superseded | archived
topic: "OAuth2 implementation approach"

# Accountability
author: jane.doe                   # human author or "ai-assisted"
reviewed_by: john.smith            # required attestation
ai_assisted: true                  # explicit flag
ai_model: claude-3.5-sonnet        # optional: which model

# Linking
related_pr: "org/repo#123"         # full reference preferred
related_issue: "org/repo#456"
superseded_by: "2026-02-15-oauth2-v2.md"

# Classification
tags: [auth, security, api]
data_sensitivity: public | internal | restricted

# Content sections (recommended for research with decisions)
# - assumptions
# - constraints
# - alternatives_considered
# - decision
# - consequences
---
```

**Required fields:** `schema_version`, `date`, `type`, `status`, `topic`, `reviewed_by`

**Conditional:** `ai_assisted` required if AI was used

---

## Quality Bar: When to Commit

**Commit when ALL are true:**
- [ ] Claims are linked to sources (code refs, docs, external links)
- [ ] Assumptions are explicitly listed
- [ ] Alternatives were considered (for decision-focused research)
- [ ] A human reviewer attests it reflects the team's intent (`reviewed_by`)
- [ ] No secrets, credentials, or sensitive data included
- [ ] Sensitive internal URLs/systems are redacted or generalized

**File size guidance:** Keep individual documents under 500 lines (~1000-2000 tokens). Monolithic files degrade AI tool performance and exceed practical context limits. If a document grows too large, split by subtopic or phase.

**When NOT to write:**
- Trivial changes (typo fixes, dependency bumps)
- Decisions already documented in code comments or existing ADRs
- Temporary spikes that won't inform future work
- When it would just repeat the PR description

---

## Security & Compliance Policy

### Prohibited Content
- Secrets, credentials, API keys
- Customer data or PII
- Incident details with sensitive information
- Vulnerability details before public disclosure
- Internal URLs that shouldn't be exposed

### AI Input Awareness
Document what context the AI was given:
```yaml
ai_context_scope: "codebase files only, no secrets"
```

### Prompt Injection Safeguard
These artifacts are **descriptive only**, not instructional. AI agents reading these docs should treat them as context, not commands. Consider:
- Linting for suspicious "do X" patterns
- Agent-side trust boundaries

### Legal/IP Considerations
Consult your legal team on:
- Copyright provenance of AI-generated text
- Attribution requirements
- Licensing implications for model outputs

---

## Workflow: Committing with Code

### For Plans
1. Create plan document early in feature branch
2. Submit for review (can be before code)
3. Implementation commits follow
4. PR includes both plan + code
5. Plan status → `archived` on merge

### For Research (including decisions)
1. Create when making significant architectural choices OR during investigation
2. Include alternatives considered and rationale for decisions
3. Consolidate multiple sessions into one doc
4. Commit when research concludes or with related code change
5. Decisions remain `active` until superseded; exploratory notes evaluated on merge

### For Handoffs
1. Create at natural stopping points or before context limits
2. Document current state, next steps, blockers, and decisions made
3. Include relevant file paths and code references
4. Next session consumes handoff and archives it

### Avoiding PR Noise
- Docs accompany code changes, not standalone "docs-only" PRs
- One plan per feature branch (not per session)
- Exception: docs-only PRs allowed for corrections, security redactions, or archival

### PR Template Integration
Add to PR template:
```markdown
## Decision Artifact
- [ ] Decision doc added/updated: [link]
- [ ] Or: No decision doc needed (trivial change)
```

---

## Maintenance & Lifecycle

### Status Transitions
```
draft → active → superseded | archived
```

- **draft**: Work in progress, may change significantly
- **active**: Current, relevant decision context
- **superseded**: Replaced by newer decision (link via `superseded_by`)
- **archived**: No longer relevant, kept for historical reference

**Type-specific defaults:**
- `type: plan` → auto-archive on merge
- `type: research` → remains `active` until explicitly superseded (for decisions) or evaluate on merge (for exploratory notes)
- `type: handoff` → auto-archive after consumed by next session

### When to Update vs Supersede
- **Edit in place**: Clarifications, typo fixes, adding detail (add changelog line)
- **Supersede**: Changing the actual decision or approach

### Staleness Detection
Rather than age-based flags, detect via signals:
- Feature removed from codebase
- Dependency/API replaced
- Config/schema deleted
- Add `last_reviewed: YYYY-MM-DD` when touched

### Ownership Model
- **Primary owner**: `reviewed_by` field, updated if ownership transfers
- **CODEOWNERS**: Map `ai_docs/**` to appropriate team(s)
- **On departure**: Ownership transfers to team, not orphaned

### Archival Checklist
When deprecating a feature:
- [ ] Archive related plans
- [ ] Supersede or archive decision-focused research
- [ ] Update or archive exploratory research
- [ ] Clean up consumed handoffs

---

## Tooling Recommendations

### Pre-commit Hooks (Local)
```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/DavidAnson/markdownlint-cli2
    rev: v0.14.0
    hooks:
      - id: markdownlint-cli2
        files: ^ai_docs/.*\.md$
```

**Note:** Link checking can be flaky locally (network, rate limits). Push to CI instead.

### CI Checks
- **Frontmatter validation**: Ensure required fields present (use JSON schema)
- **Link checking**: Verify `related_pr` and `superseded_by` resolve (relative links only locally)
- **Sensitive content scan**: Flag potential secrets/credentials
- **CODEOWNERS**: Auto-assign reviewers for `ai_docs/**`

### Templates
Provide in `ai_docs/templates/`:
- `plan.md` - Implementation plan scaffold
- `research.md` - Research note scaffold (includes decision record structure)
- `handoff.md` - Session handoff scaffold

### Discoverability
**Reference in AGENTS.md:** Add a section to your repository's `AGENTS.md` (or `CLAUDE.md`) pointing AI agents to `ai_docs/`:
```markdown
## Decision Context
For architectural decisions, implementation plans, and research notes, see `ai_docs/index.md`.
```

**Master index:** Maintain `ai_docs/index.md` as the entry point with project terminology, architecture overview, and links to active documents.

### Cross-Referencing

Reference other ai_docs using `@ai_docs/` prefix:

```markdown
Based on constraints identified in @ai_docs/research/2026-01-10-auth-options.md, we chose OAuth2.

See also:
- @ai_docs/plans/2026-01-12-oauth2-impl.md
- @ai_docs/research/2026-01-15-token-storage.md
```

This syntax is:
- Grep-able across the codebase
- Parseable by AI agents
- Consistent with [SIP-189](https://github.com/apache/superset/issues/35822) conventions

---

## Adoption Considerations

### Pilot Program
1. Start with 1-2 willing teams
2. Track metrics over 3 months
3. Iterate on conventions before org-wide rollout

### Success Metrics (with targets)
- New team members reference decision docs during onboarding (survey: >60% report helpful)
- Reduced "why was this done this way?" questions in code review (track in retros)
- PR review time not significantly increased (<10% overhead)

### Failure Modes to Avoid
| Risk | Mitigation |
|------|------------|
| Docs drift from code | Commit together, PR template checklist |
| Review fatigue | Consolidated docs, "when not to write" guidance |
| Staleness | Signal-based detection, ownership model |
| Inaccurate AI content | Human reviewer attestation required |
| Security leaks | Prohibited content policy, CI scanning |

### Alternative Approach
If full adoption is too heavy, consider: keep artifacts in **PR descriptions or linked issues**, only "promote" to repo when it becomes a long-lived architectural decision.

---

## Example: Before & After

### Before (without decision docs)
PR #456: "Add OAuth2 authentication"
- 2000 lines of code
- PR description: "Implements OAuth2"
- 6 months later: "Why didn't we use SAML? Why this library?"

### After (with decision docs)
PR #456: "Add OAuth2 authentication"
- 2000 lines of code
- `ai_docs/research/2026-01-15-auth-approach.md`:
  - **Decision**: OAuth2 with PKCE via `openid-client` library
  - **Alternatives**: SAML (rejected: complexity), Auth0 (rejected: cost), Keycloak (rejected: ops burden)
  - **Constraints**: Must support mobile apps, no new infrastructure
- 6 months later: new engineer reads record, understands context in 5 minutes

---

## Sources

### Directory Structure & Naming
- [Apache Superset SIP-189: Context Engineering](https://github.com/apache/superset/issues/35822)
- [GitHub Spec Kit](https://github.com/github/spec-kit)
- [AGENTS.md specification](https://agents.md/)
- [VS Code Context Engineering Guide](https://code.visualstudio.com/docs/copilot/guides/context-engineering-guide)

### ADR Conventions
- [ADR GitHub Organization](https://adr.github.io/)
- [AWS ADR Best Practices](https://aws.amazon.com/blogs/architecture/master-architecture-decision-records-adrs-best-practices-for-effective-decision-making/)
- [Practical ADR Overview](https://ctaverna.github.io/adr/)

### AI Documentation Workflow
- [Automating ADRs with GitHub + LLM](https://medium.com/@iraj.hedayati/from-stale-docs-to-living-architecture-automating-adrs-with-github-llm-e80bb066b4b6)
- [GitHub Blog: Spec-Driven Development](https://github.blog/ai-and-ml/generative-ai/spec-driven-development-with-ai-get-started-with-a-new-open-source-toolkit/)
- [AI Instruction Files Overview](https://gist.github.com/0xdevalias/f40bc5a6f84c4c5ad862e314894b2fa6)

### Adoption & Team Workflow
- [Reality of AI-Assisted Engineering Productivity](https://addyo.substack.com/p/the-reality-of-ai-assisted-software)
- [Microsoft AI Code Review at Scale](https://devblogs.microsoft.com/engineering-at-microsoft/enhancing-code-quality-at-scale-with-ai-powered-code-reviews/)
- [Graphite: Adopting AI Tools](https://graphite.com/guides/adopting-ai-tools-development-workflow)

---

## Next Steps

1. **Review security/compliance policy** - Legal/security team sign-off
2. **Create ai_docs/index.md** - Master index with project terminology and architecture
3. **Add templates** - Plan, research, and handoff templates with frontmatter
4. **Configure tooling** - Pre-commit hooks, CI checks, CODEOWNERS
5. **Pilot with volunteer team** - Gather feedback before broader rollout
