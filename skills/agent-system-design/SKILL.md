---
name: agent-system-design
description: Create a reviewable, framework-agnostic agent system design from requirements. Use when selecting agent or workflow patterns, drawing architecture and sequence diagrams, or preparing an implementation handoff; do not treat the design as a running system.
---

# Agent system design

Produce one reviewable System Design Proposal using the [template](../../templates/SYSTEM_DESIGN.md). This Skill guides design; it does not select an architecture automatically, grant action authority, implement a system, or prove runtime behavior. Use the [architecture guide](../../docs/ARCHITECTURE_SELECTION.md) and [engineering policies](../../docs/ENGINEERING_POLICIES.md) as the authoritative references. The steps below can be completed as working notes and proposal sections; do not create a file for each step. Present the proposal in the conversation by default, and save a Markdown file only when the user explicitly requests one and provides or approves its destination.

## Common execution rules

- Distinguish **KNOWN** facts, **ASSUMED** inputs, **UNKNOWN** inputs, proposed controls, and verified behavior. Record concise decisions and engineering rationale; private model reasoning is not a deliverable.
- A **blocking unknown** prevents a defensible architecture commitment, authorization decision, or safe implementation recommendation. Ask an authorized human to clarify it. Do not invent permissions or approve the affected decision. If the remaining design is useful, present it as conditional and name the blocked decisions; do not proceed to unauthorized implementation.
- A **non-blocking unknown** permits a defensible conditional proposal. Record its impact, the assumption if used, and the evidence or decision needed to resolve it. Do not invent universal numeric thresholds.
- A **conditional candidate** is a provisional option for evaluation in a conditional proposal while material facts, evidence, or decisions remain unresolved; it is not approved and grants no execution or implementation authority. An **approved architecture decision** requires explicit acceptance by an authorized human after review of material assumptions, trade-offs, and blockers, subject to recorded conditions; it neither authorizes implementation nor proves runtime behavior.
- Run each step's local **Verify** check before proceeding. Correct a resolvable issue in that step, revisit affected earlier decisions, or carry a documented non-blocking gap forward. Local checks are not human approval or implementation evidence. Step 9 checks the assembled proposal again.

## 1. Gather requirements

- **Input:** User request and available context.
- **Action:** Elicit the goal, users, acceptance criteria, data sources, outputs, external actions, and functional and operational constraints. Ask targeted questions where an answer could change control, authority, or acceptance.
- **Output:** A requirements and constraints inventory with unanswered questions.
- **Verify:** Each material requirement and constraint has a stated value or an explicit unknown; success and failure criteria are observable or marked as material questions to resolve.

## 2. Classify inputs and unknowns

- **Input:** The Step 1 inventory.
- **Action:** Label material inputs KNOWN, ASSUMED, or UNKNOWN. Classify unknowns as blocking or non-blocking by their effect on architecture, permissions, and acceptance; identify who can resolve each one.
- **Output:** A classified input list, assumptions with impacts, and blocking questions.
- **Verify:** No missing authority or sensitive-data access has been converted into an assumed permission; conditional decisions name their blockers.

## 3. Select Layer 1 control strategy

- **Input:** Requirements, acceptance criteria, constraints, and classified inputs.
- **Action:** Apply Layer 1 of the [architecture guide](../../docs/ARCHITECTURE_SELECTION.md). Compare conventional software, one bounded LLM call, a code-controlled workflow, and a bounded model-controlled agent. Choose the simplest candidate supported by the requirements; tool use alone does not establish model control.
- **Output:** Recommended Layer 1 candidate, requirement-based rationale, rejected candidates, and evidence gaps; label the candidate **CONDITIONAL** when material facts or decisions remain unresolved.
- **Verify:** Simpler candidates were considered; model-controlled action choice has a documented need or is explicitly provisional. A conditional candidate is not presented as an approved selection or permission to access data or implement.

## 4. Select Layer 2 orchestration patterns

- **Input:** Layer 1 choice, task dependencies, and quality or coordination needs.
- **Action:** Use Layer 2 of the [architecture guide](../../docs/ARCHITECTURE_SELECTION.md) to choose justified sequential, parallel, router, evaluator–optimizer, supervisor, or handoff patterns, or none. State the outer control owner and why each added pattern helps.
- **Output:** Pattern selection and composition, with responsibilities and reasons.
- **Verify:** Each pattern matches a requirement or evidenced benefit; routing, parallelism, and delegation are not added by default.

## 5. Assess Layer 3 execution and operations

- **Input:** Selected control strategy, patterns, external actions, and constraints.
- **Action:** Decide whether ordinary code or a graph representation is warranted. Specify state and recovery, authorization boundaries, human approval for consequential actions, deadlines, retries, idempotency, budgets, observability, and explicit failure outcomes. Keep framework, model, and tool choices separate.
- **Output:** Execution representation and operational-control decisions with owners and unresolved needs.
- **Verify:** Proposed side effects have an authorization point; limits and failures have defined outcomes; no graph, checkpoint, or framework is required without a reason.

## 6. Compare alternatives and trade-offs

- **Input:** The three-layer candidate and acceptance criteria.
- **Action:** Compare a credible simpler alternative and other rejected options across quality, complexity, cost, latency, reliability, security, and maintenance. Name assumptions, evidence gaps, and triggers for revisiting the choice.
- **Output:** Architecture selection record with trade-offs and revision triggers.
- **Verify:** The selected design's added complexity has a requirement-based reason; rejected options and unresolved evidence are visible.

## 7. Assess engineering policies

- **Input:** Capabilities, data and action risks, execution design, and the [policy guide](../../docs/ENGINEERING_POLICIES.md).
- **Action:** Assess every existing Policy ID for **REQUIRED**, **CONDITIONAL**, or **NOT_APPLICABLE**, recording the trigger or reason. For applicable IDs, use the [template's policy record](../../templates/SYSTEM_DESIGN.md) for stage, status, required evidence, evidence reference, and review owner. Apply **PASS**, **FAIL**, **NEEDS_EVIDENCE**, or **NOT_EVALUATED** according to the guide; keep DESIGN and IMPLEMENTATION separate. Record exceptions or gaps and identify applicable AUTH-02 approval or pre-authorization conditions.
- **Output:** Policy applicability and DESIGN-stage evidence records, with gaps, exceptions, and review owners.
- **Verify:** Each ID has a justified applicability state; CONDITIONAL triggers remain unresolved until decided; missing evidence is not PASS. Never self-approve DESIGN PASS or claim IMPLEMENTATION PASS without observable evidence from the actual control.

## 8. Complete the System Design Proposal

- **Input:** Results of Steps 1–7 and the [system design template](../../templates/SYSTEM_DESIGN.md).
- **Action:** Fill the template with components, interfaces, trust boundaries, framework/model/tool assumptions, editable Mermaid architecture and request sequence diagrams, failure and escalation paths, requirement-linked tests, policy records, and approval questions. Show authorization before side effects.
- **Output:** One complete, reviewable proposal with proposed behavior distinguished from evidence.
- **Verify:** Required sections are filled or explicitly marked unknown or out of scope; diagram actors have written responsibilities; acceptance cases map to proposed verification.

## 9. Validate proposal consistency

- **Input:** The assembled proposal and authoritative guides.
- **Action:** Cross-check requirements and acceptance, architecture and alternatives, components and interfaces, both diagrams, trust boundaries and authorization, policy records, failure paths and tests, and unknowns and approval decisions. Parse or render Mermaid when a suitable tool is available. Revise affected sections or prior decisions and recheck.
- **Output:** A consistent proposal plus a short record of remaining gaps and checks performed.
- **Verify:** Contradictions are resolved or explicitly flagged for human review; unavailable Mermaid parsing or rendering is reported accurately. This check does not prove the proposed system works.

## 10. Present for human review

- **Input:** The Step 9 proposal and unresolved issues.
- **Action:** Present the complete proposal, summarize material decisions, assumptions, risks, evidence gaps, and questions for an authorized reviewer, including applicable AUTH-02 choices. Request approval or revision of the design. Separate design approval from authorization to implement and from later implementation verification.
- **Output:** A review package with explicit pending decisions and proposed implementation scope.
- **Verify:** No agent self-approval or implied permission to implement, deploy, or exercise tools; substantial implementation awaits an explicitly authorized scope, and runtime claims await observable evidence.
