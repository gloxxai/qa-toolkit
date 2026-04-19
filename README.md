# QA Toolkit

Generic, portable QA skills and agents for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Built from 15 years of QA leadership across fintech, enterprise security, and government — distilled into reusable procedures and personas that work across any codebase.

## What's inside

### Agents (`agents/`)

Specialized personas that adopt a role for a category of QA work.

| Agent | Purpose | Model |
|---|---|---|
| [bug-reviewer](agents/bug-reviewer.md) | Reviews bug reports for completeness before handoff to engineering | sonnet |
| [cypress-qa-engineer](agents/cypress-qa-engineer.md) | E2E test authoring, coverage analysis, flake diagnosis | sonnet |
| [pr-risk-reviewer](agents/pr-risk-reviewer.md) | Scores PR blast radius and recommends specific tests before merge | sonnet |
| [qa-analyst](agents/qa-analyst.md) | Strategic QA planning, coverage-gap analysis, capacity allocation | opus |
| [regression-lead](agents/regression-lead.md) | Multi-repo regression planning and coordination | sonnet |
| [test-plan-generator](agents/test-plan-generator.md) | Turns tickets, PRs, or epics into executable manual test plans | sonnet |

### Skills (`skills/`)

Step-by-step procedures for specific QA tasks. Each is user-invocable via `/skill-name` in Claude Code.

**Testing & Automation**

| Skill | Purpose |
|---|---|
| [cypress-test-creation](skills/cypress-test-creation/SKILL.md) | Write new Cypress E2E specs following project conventions |
| [playwright-remote-testing](skills/playwright-remote-testing/SKILL.md) | Playwright tests against remote environments with persistent auth |
| [api-testing](skills/api-testing/SKILL.md) | REST and GraphQL API testing patterns, schema validation |

**QA Process**

| Skill | Purpose |
|---|---|
| [bug-triage](skills/bug-triage/SKILL.md) | Severity + impact + type triage for incoming bugs |
| [pr-research-test-plan](skills/pr-research-test-plan/SKILL.md) | Research a PR and produce a manual test plan |
| [pr-research-test-results](skills/pr-research-test-results/SKILL.md) | Execute a test plan and produce a results report |
| [pr-approval](skills/pr-approval/SKILL.md) | Structured QA approval comment with acceptance criteria |
| [regression-testing](skills/regression-testing/SKILL.md) | Plan and execute regression cycles before releases |
| [exploratory-testing](skills/exploratory-testing/SKILL.md) | Structured session-based exploratory testing |

**Test Management**

| Skill | Purpose |
|---|---|
| [jira](skills/jira/SKILL.md) | General Jira CLI operations (create, search, transition, link) |
| [jira-qa](skills/jira-qa/SKILL.md) | QA-specific Jira workflows (bug filing, test-case linking) |
| [xray](skills/xray/SKILL.md) | Xray Cloud GraphQL API for test-case management |

**Data & Documentation**

| Skill | Purpose |
|---|---|
| [test-data-management](skills/test-data-management/SKILL.md) | Seed, reset, and factory strategies for test data |
| [confluence-docs](skills/confluence-docs/SKILL.md) | Publish QA docs, test plans, runbooks, and retros |

**Prompt Engineering**

| Skill | Purpose |
|---|---|
| [prompt-maker](skills/prompt-maker/SKILL.md) | Guided Five-Part Framework (Identity, Task, Context, Constraints, Output Format) for authoring eval prompts, red-team prompts, and test-case prompts |

## Installation

### Copy to your Claude Code user directory

```bash
# Clone
git clone https://github.com/gloxxai/qa-toolkit.git

# Copy agents
cp qa-toolkit/agents/*.md ~/.claude/agents/

# Copy skills (preserves directory structure)
cp -r qa-toolkit/skills/* ~/.agents/skills/

# Symlink skills into Claude Code's discovery path
cd ~/.claude/skills
for dir in ~/.agents/skills/api-testing ~/.agents/skills/bug-triage ~/.agents/skills/confluence-docs ~/.agents/skills/cypress-test-creation ~/.agents/skills/exploratory-testing ~/.agents/skills/jira ~/.agents/skills/jira-qa ~/.agents/skills/playwright-remote-testing ~/.agents/skills/pr-approval ~/.agents/skills/pr-research-test-plan ~/.agents/skills/pr-research-test-results ~/.agents/skills/prompt-maker ~/.agents/skills/regression-testing ~/.agents/skills/test-data-management ~/.agents/skills/xray; do
  ln -sf "$dir" "$(basename $dir)"
done
```

### Or use project-scoped (per-repo)

```bash
# From your project root
mkdir -p .claude/agents .claude/skills
cp qa-toolkit/agents/*.md .claude/agents/
cp -r qa-toolkit/skills/* .claude/skills/
```

## Composition patterns

These tools are designed to chain together. Common workflows:

**PR lifecycle:**
`pr-risk-reviewer` agent → `pr-research-test-plan` skill → `pr-research-test-results` skill → `pr-approval` skill

**Incoming bug:**
`bug-reviewer` agent → `bug-triage` skill → `jira-qa` skill

**Release prep:**
`qa-analyst` agent → `regression-lead` agent → `regression-testing` skill

**New feature launch:**
`test-plan-generator` agent → `cypress-test-creation` skill → `exploratory-testing` skill → `confluence-docs` skill

## Design principles

- **Decision-oriented, not tool-oriented.** Files encode judgment (severity rubrics, blast-radius scoring, risk matrices) rather than tool-specific commands. The QA craft is durable; the tooling changes.
- **Tracker-agnostic where possible.** Output templates paste into Jira, Linear, GitHub Issues, or any other tracker. Tool-specific skills (jira, xray, confluence-docs) are clearly labeled.
- **Explicit scope boundaries.** Every agent and skill has a "What you do NOT do" section to prevent scope creep. A bug reviewer doesn't triage; a triage skill doesn't fix.
- **Composable, not monolithic.** Each file does one job well. Chain them for workflows; use them standalone for focused tasks.

## About

Built by [Gloxx](https://gloxx.ai) — AI-augmented QA for blockchain teams. These tools represent the generic QA craft that applies across any tech stack. No proprietary content, no company-specific references.

## License

MIT
