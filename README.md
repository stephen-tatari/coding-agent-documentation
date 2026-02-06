# Proposal: Decision Records with AI Assistance

**Goal:** Preserve the "why" behind decisions - plans, research, and rationale - as versioned artifacts discoverable from code.

**Audience:** Engineering organization adoption proposal

**Framing:** This extends standard Architecture Decision Records (ADRs) to include AI-assisted artifacts. The primary value is structured decision capture; AI assistance is optional and secondary.

---

## Executive Summary

Engineering decisions get made, code gets merged, and months later nobody remembers why things were done that way. This proposal establishes a **hybrid** convention: durable decision artifacts (plans, research) live in a centralized AI documentation repository for cross-project visibility, while ephemeral handoffs remain local in each code repo's `ai_docs/handoffs/` directory (gitignored).

**Key principles:**
- Durable decisions (plans, research) centralized for cross-project discovery
- Handoffs stay local and ephemeral — session state, not institutional knowledge
- Discovery via AGENTS.md pointers and grep commands
- Human accountability via code PR review (decision docs referenced, not co-committed)
- Security-conscious: explicit policies for sensitive content

---

## Directory Structure

This proposal uses two locations: a central AI documentation repository for durable decisions, and local directories in each code repo for ephemeral handoffs.

### Central AI Documentation Repository

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

**Naming convention:** Including `<project>` in the filename is optional but recommended when the topic is project-specific. Cross-cutting docs should omit it. Project association is always available via the `project:` frontmatter field.

### Local Code Repository

```text
my-service/
├── AGENTS.md          # points to central repo for plans/research
├── .gitignore         # includes ai_docs/handoffs/
└── ai_docs/
    └── handoffs/      # ephemeral, gitignored
        └── YYYY-MM-DD-<context>.md
```

**Handoffs are local-only.** They are session state for AI agents — not committed, not shared, not archived. Each code repo gitignores `ai_docs/handoffs/`.

### Sibling Clone Layout

Clone both repos as siblings for cross-referencing:

```text
~/code/
├── <ai-docs-repo>/    # central AI documentation repo
└── my-service/        # code repo with AGENTS.md pointing to ../<ai-docs-repo>/
```

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

**Handoff** (ephemeral, local-only)
- Preserves context between AI coding sessions
- Current state, next steps, blockers, decisions made
- Created at natural stopping points or before context limits
- Lives in each code repo's `ai_docs/handoffs/` (gitignored)
- Never committed, not shared, not archived — disposable session state

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
author: jane.doe                   # Human owner (run: git config user.name)
ai_assisted: true                  # explicit flag
ai_model: claude-3.5-sonnet        # optional: which model

# Project association (plans/research only — not needed for handoffs)
project: conductor                 # Logical project/service name
repo: org/conductor                # GitHub org/repo (canonical identifier)
repos: [org/conductor, org/api-gateway]  # For cross-repo docs

# Linking
related_pr: "org/repo#123"         # full reference preferred
related_issue: "org/repo#456"
related_prs:                       # PRs that implement this decision
  - https://github.com/org/conductor/pull/123
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

**Required fields:** `schema_version`, `date`, `type`, `status`, `topic`

**Conditional:** `ai_assisted` required if AI was used

**Optional but recommended (plans/research):** `project`, `repo` (or `repos` for cross-repo docs), `related_prs`

**Note:** Handoffs use minimal frontmatter — no project association fields needed since they are local and ephemeral.

---

## Quality Bar: When to Commit

**Commit when ALL are true:**
- Claims are linked to sources (code refs, docs, external links)
- Assumptions are explicitly listed
- Alternatives were considered (for decision-focused research)
- Code PR review provides human accountability (decision doc referenced from the implementing PR)
- No secrets, credentials, or sensitive data included
- Sensitive internal URLs/systems are redacted or generalized

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

## Workflow

### For Plans and Research (Central AI Documentation Repo)
1. Create document in the central AI documentation repo (`plans/` or `research/`)
2. Commit directly — no PR required for the documentation repo
3. Reference the decision doc from the code PR that implements it
4. Add implementing PR URL to `related_prs:` in the decision doc's frontmatter
5. Status transitions follow the lifecycle rules below

### For Handoffs (Local Code Repo)
1. Create at natural stopping points or before context limits
2. Write to `ai_docs/handoffs/` in the code repo (gitignored)
3. Next session consumes handoff and deletes it
4. Never commit, never centralize — these are disposable session state

### Cross-Referencing Between Repos
- Code PR description links to the decision doc in the central repo
- Decision doc's `related_prs:` field links back to implementing PRs
- This creates a bidirectional trail without co-committing docs and code

### PR Template Integration
Add to your code repo's PR template:
```markdown
## Decision Artifact
- [ ] Decision doc in [AI docs repo](https://github.com/org/<ai-docs-repo>): [link to doc]
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
- `type: handoff` → deleted after consumed by next session (local, never committed)

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
- **Primary owner**: `author` field, updated if ownership transfers
- **CODEOWNERS**: Map `plans/**` and `research/**` to appropriate team(s) in the central repo
- **On departure**: Ownership transfers to team, not orphaned

### Archival
When deprecating a feature:
- Archive related plans
- Supersede or archive decision-focused research
- Update or archive exploratory research
- Clean up consumed handoffs

---

## Tooling Recommendations

These apply to the **central AI documentation repo** (plans/research). Local handoffs require no tooling.

### Pre-commit Hooks (Central Repo)
```yaml
# .pre-commit-config.yaml (in the central AI documentation repo)
repos:
  - repo: https://github.com/DavidAnson/markdownlint-cli2
    rev: v0.14.0
    hooks:
      - id: markdownlint-cli2
        files: ^(plans|research)/.*\.md$
```

**Note:** Link checking can be flaky locally (network, rate limits). Push to CI instead.

### CI Checks (Central Repo)
- **Frontmatter validation**: Ensure required fields present (use JSON schema)
- **Link checking**: Verify `related_prs` and `superseded_by` resolve
- **Sensitive content scan**: Flag potential secrets/credentials
- **CODEOWNERS**: Auto-assign reviewers for `plans/**` and `research/**`

### Templates
Provide in the central repo's `templates/` directory:
- `plan.md` - Implementation plan scaffold
- `research.md` - Research note scaffold (includes decision record structure)
- `handoff.md` - Session handoff scaffold (for copying into code repos)

### .gitignore Setup (Code Repos)

Each code repo should add this to `.gitignore`:

```gitignore
# AI session handoffs — ephemeral, not committed
ai_docs/handoffs/
```

---

## Discovery

### AGENTS.md Template

Add this section to each code repository's `AGENTS.md` to teach AI agents where to find decision records:

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

### When to Create Artifacts

| Situation | Create | Where |
|-----------|--------|-------|
| Starting multi-session feature work | Plan | Central repo (`plans/`) |
| Evaluating technical options | Research note | Central repo (`research/`) |
| Making architectural decision | ADR (research with Decision section) | Central repo (`research/`) |
| Pausing mid-implementation | Handoff | Local code repo (`ai_docs/handoffs/`) |
| Completing investigation | Research note | Central repo (`research/`) |

### Cross-Referencing

Reference decision docs using relative paths from the sibling clone:

```markdown
Based on constraints in ../<ai-docs-repo>/research/2026-01-10-auth-options.md, we chose OAuth2.

See also:
- ../<ai-docs-repo>/plans/2026-01-12-oauth2-impl.md
- ../<ai-docs-repo>/research/2026-01-15-token-storage.md
```

Within the central repo, reference other docs by relative path:

```markdown
See also: ../plans/2026-01-12-oauth2-impl.md
```

This syntax is:
- Grep-able across both repos
- Parseable by AI agents
- Works with standard filesystem navigation

---

## Tradeoffs: Per-Project vs Centralized vs Hybrid

| Aspect | Per-Project | Centralized | Hybrid (chosen) |
|--------|-------------|-------------|-----------------|
| Discovery | Natural — agents find while exploring | Requires AGENTS.md instruction | AGENTS.md for plans/research; handoffs local |
| Code repo size | Grows with docs | Stays clean | Clean (handoffs gitignored) |
| Cross-project search | Difficult | Easy | Easy for durable docs; handoffs N/A |
| Doc-code atomicity | Same commit/PR | Separate, cross-referenced | Plans/research separate; handoffs uncommitted |
| Drift risk | Low | Higher | Moderate — handoffs can't drift (disposable) |
| Handoff ergonomics | Good — local and discoverable | Poor — noise in central repo | Good — local, gitignored, session-scoped |

---

## Adoption Considerations

### Setup Requirements

Adoption requires:
- A central AI documentation repo with `plans/`, `research/`, `templates/` directories
- `.gitignore` entries in each code repo for `ai_docs/handoffs/`
- AGENTS.md in each code repo using the template above
- PR templates in code repos updated to reference the central documentation repo

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
| Docs drift from code | Cross-referencing via `related_prs:`, PR template checklist |
| Review fatigue | Consolidated docs, "when not to write" guidance |
| Staleness | Signal-based detection, ownership model |
| Inaccurate AI content | PR review provides human accountability |
| Security leaks | Prohibited content policy, CI scanning |
| Two-repo confusion | AGENTS.md template, sibling clone layout |

### Alternative Approach
If full adoption is too heavy, consider: keep artifacts in **PR descriptions or linked issues**, only "promote" to the central repo when it becomes a long-lived architectural decision.

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
- PR description links to `research/2026-01-15-auth-approach.md` in the central AI docs repo:
  - **Decision**: OAuth2 with PKCE via `openid-client` library
  - **Alternatives**: SAML (rejected: complexity), Auth0 (rejected: cost), Keycloak (rejected: ops burden)
  - **Constraints**: Must support mobile apps, no new infrastructure
- Decision doc's `related_prs:` links back to PR #456
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

