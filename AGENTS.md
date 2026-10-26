# Repository Guidance for Coding Agents

## Purpose

This is the public repository-level entry point for coding agents. Read [`README.md`](README.md) for the toolkit's purpose and usage. Follow the public design procedure and references below even when your tool does not load them automatically.

Developers and engineering teams using this repository may add project-specific instructions or Skills for coding style, testing, language and framework preferences, domain rules, or stricter quality controls. These supplement the baseline and cannot override applicable Engineering Policies, execution authorization, or required human approval.

## Design Workflow

1. Read the user's requirements and supplied context. Determine whether goals, scope, acceptance criteria, constraints, and authority are sufficiently understood for a defensible design; identify material gaps and blockers from the actual requirements, not the user's technical vocabulary.
2. Follow [`skills/requirements-clarification/SKILL.md`](skills/requirements-clarification/SKILL.md) when clarification is requested or material ambiguity needs resolution, and produce its Requirements Brief. Otherwise, skip clarification for sufficient requirements; honor a request for a Brief only.
3. Carry any Brief's stable IDs, evidence, assumptions, unknowns, blocking conditions, readiness, and User Confirmation state forward. Stop design handoff for NEEDS_CLARIFICATION; retain pending decisions and permissions for CONDITIONAL_FOR_DESIGN. Follow the Skill's rules for pending confirmation and conditional handoff. Keep readiness and confirmation independent; neither constitutes approval.
4. Perform system design only when requested and defensible. Follow [`skills/agent-system-design/SKILL.md`](skills/agent-system-design/SKILL.md) using direct requirements or the Brief. Apply the methodology in [`docs/ARCHITECTURE_SELECTION.md`](docs/ARCHITECTURE_SELECTION.md), determine policy applicability, and follow all applicable requirements in [`docs/ENGINEERING_POLICIES.md`](docs/ENGINEERING_POLICIES.md).
5. Use [`templates/SYSTEM_DESIGN.md`](templates/SYSTEM_DESIGN.md) for the implementation-oriented System Design Proposal, including contracts, diagrams, design-level logic, applicable policies, evidence gaps, proposed verification, and Work Packages.
6. Present material decisions and concise engineering rationale for authorized human review. Do not approve your own architecture decisions.
7. Require separate implementation authorization and scope before creating an application or implementing Work Packages.

## Operating Boundaries

- Respect repository ignore rules; do not disclose private or sensitive files.
- Treat model-generated suggestions as proposals, not authorization. Do not infer tool or data-access permissions; follow applicable Engineering Policies and required human approvals before external actions.
- Implement only an approved scope. Do not deploy, stage, commit, push, or publish without authorization.
- Treat design-level code as illustrative; do not execute it under a design-only request or present it as implemented or tested behavior.
- Distinguish proposed behavior from observed Verification Evidence. Report changes, checks performed, and remaining verification gaps.
