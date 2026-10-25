# Repository Guidance for Coding Agents

## Purpose

This is the public repository-level entry point for coding agents. Read [`README.md`](README.md) for the toolkit's purpose and usage. Follow the public design procedure and references below even when your tool does not load them automatically.

Consumers may add project-specific instructions or Skills for coding style, testing, language and framework preferences, domain rules, or stricter quality controls. These supplement the baseline and cannot override applicable Engineering Policies, execution authorization, or required human approval.

## Design Workflow

1. Follow [`skills/agent-system-design/SKILL.md`](skills/agent-system-design/SKILL.md) to turn requirements into a design proposal.
2. Consult [`docs/ARCHITECTURE_SELECTION.md`](docs/ARCHITECTURE_SELECTION.md) and [`docs/ENGINEERING_POLICIES.md`](docs/ENGINEERING_POLICIES.md) for architecture choices and applicable Policy IDs.
3. Use [`templates/SYSTEM_DESIGN.md`](templates/SYSTEM_DESIGN.md) as the proposal structure; identify unresolved requirements and label material inputs known, assumed, or unknown.
4. Record concise, reviewable rationale, alternatives, diagrams, applicable policies, evidence gaps, and proposed verification.
5. Present the proposal for human review before substantial implementation. Do not approve your own design.

## Operating Boundaries

- Respect repository ignore rules; do not disclose private or sensitive files.
- Treat model-generated suggestions as proposals, not authorization. Follow applicable Engineering Policies and required human approvals before external actions.
- Implement only an approved scope. Do not deploy, stage, commit, push, or publish without authorization.
- Distinguish proposed behavior from verified implementation. Report changes, checks performed, and remaining verification gaps.
