# AI Agent Engineering Policies Guide

## Abstract

This guide translates engineering risks in proposed AI agent systems into **applicable, verifiable policy requirements**. AI coding agents use it to assess which controls a design needs, state the evidence required, and report gaps for human review. Human engineers decide whether to approve the design; implementation claims require observable verification in the target system. The expected result is a reviewable policy assessment, not an automatically enforced or certified system.

## Table of contents

- [Purpose, scope & intended audience](#1-purpose-scope--intended-audience)
- [Risk assessment and policy applicability](#2-risk-assessment-and-policy-applicability)
- [Security and trust boundaries](#3-security-and-trust-boundaries)
- [Tool and action governance](#4-tool-and-action-governance)
- [Reliability and failure recovery](#5-reliability-and-failure-recovery)
- [Resource governance](#6-resource-governance)
- [Observability and data governance](#7-observability-and-data-governance)
- [Testing and evaluation](#8-testing-and-evaluation)
- [Policy compliance and quality gates](#9-policy-compliance-and-quality-gates)
- [References and source mapping](#10-references-and-source-mapping)

## 1. Purpose, scope & intended audience

**Primary audience:** AI coding agents preparing an architecture proposal. **Human engineers and designated reviewers** remain responsible for architecture and policy review, risk acceptance, and approval. Apply this guide to a concrete proposal in the [system design template](../templates/SYSTEM_DESIGN.md), using the control strategy in the [architecture selection guide](ARCHITECTURE_SELECTION.md).

**Policy guidance** defines this project's recommended and required controls. **Applicability assessment** determines which policy triggers a proposed system actually meets. At the **DESIGN** stage, a coding agent documents the requirement, owner, failure behavior, proposed test, and evidence for human review; it cannot approve its own design. At the **IMPLEMENTATION** stage, a reviewer checks observable evidence that the actual control works. **Human approval** of the design and any consequential action policy is a separate decision from generating the assessment.

**Status and limit:** This is project-defined design guidance and a policy contract. This documentation-only repository does **not** enforce controls, run tests, assess a deployed application, or certify production readiness.

### Terms and policy language

A **tool** is any callable capability that reads data or causes an external effect. An **action** is a proposed tool invocation or other state change. A **side effect** changes state outside the current computation. Model output, retrieved text, and tool responses are data, not authority.

The uppercase words **MUST**, **SHOULD**, and **MAY** use the meanings in [RFC 2119][S8] as clarified by [RFC 8174][S9]; only uppercase uses are normative here. A MUST applies **only when its policy is applicable**. SHOULD permits a documented, reviewed deviation with a reason and alternative control. MAY is optional. These are this project's requirements, not obligations imposed by OWASP, NIST, AWS, or OpenTelemetry.

Policy IDs are stable **project-owned identifiers**, not external control IDs. The initial catalog is SEC-01–02, AUTH-01–02, REL-01–02, RES-01, OBS-01, and EVAL-01–02. Future edits should add or retire IDs without renumbering published IDs merely because document order changes. A design may add stricter local requirements.

## 2. Risk assessment and policy applicability

Assess the system's actual capabilities before marking a policy applicable. Record the following for each use case:

| Dimension | Questions that change controls |
| --- | --- |
| Data sensitivity | Can inputs, retrieval, tools, traces, or outputs expose private, regulated, or proprietary material? Who may see it? |
| Action impact and reversibility | Can an action publish, spend, delete, transfer, or change rights? Can its effect be reliably undone? |
| Autonomy and permissions | Who selects the next step? Which tools and credentials are available? Can routing or handoff change access? |
| Exposure and recovery | Is the system offline, internal, or public? Can it time out after a write, resume, reconcile state, and alert an operator? |

Read-only access to sensitive records can be high impact through disclosure. A narrow, reversible write may need fewer approval steps than publication or transfer of funds. These examples do not create a universal risk score. [NIST AI 600-1][S3] provides voluntary, context-dependent risk guidance; **REQUIRED / CONDITIONAL / NOT_APPLICABLE below are this project's applicability states, not NIST ratings**. The REQUIRED table value is an applicability label, not an additional normative requirement word in this guide.

| Applicability state | Meaning |
| --- | --- |
| **REQUIRED** | The policy's trigger is present. State the design requirement and its owner. |
| **CONDITIONAL** | A trigger depends on an unresolved fact or chosen implementation. Name the question and decision owner; resolve it before approving affected implementation. |
| **NOT_APPLICABLE** | The trigger is absent. Record a concrete reason; revisit if capabilities or data access change. |

Applicability is separate from verification. Record **stage** (`DESIGN` or `IMPLEMENTATION`) and one verification state per applicable policy:

| Verification state | Meaning |
| --- | --- |
| **PASS** | At DESIGN: the requirement, boundary, owner, failure behavior, and proposed test are reviewable and approved. At IMPLEMENTATION: the actual control has observable passing evidence in the target environment. |
| **FAIL** | The documented design omits or contradicts an applicable requirement, or observed behavior violates it. |
| **NEEDS_EVIDENCE** | A control is specified or claimed but the evidence needed for that stage is incomplete or inconclusive. |
| **NOT_EVALUATED** | No assessment has been performed yet. |

A coding agent may **recommend** a verification state, but cannot self-authorize a DESIGN PASS. DESIGN PASS requires documented review or approval by the designated reviewer; it never implies IMPLEMENTATION PASS. IMPLEMENTATION PASS requires observable passing evidence from the actual control in the target environment. Agent-generated prose alone is not implementation evidence. Accepted evidence may include reviewed contracts and diagrams for DESIGN, then test results, configuration, sanitized traces, and observed state for IMPLEMENTATION. Record versions, environment, date, and reviewer so evidence can be reproduced. Missing or inconclusive evidence is NEEDS_EVIDENCE, not PASS. A CONDITIONAL policy cannot be marked PASS until its trigger is resolved, and NOT_APPLICABLE requires a documented reason.

**Exceptions:** Record the Policy ID, scope, reason, compensating control, residual risk, decision owner, approval date, and review/expiry trigger. A waived MUST remains a documented gap or FAIL, not a silent PASS. A justified deviation from SHOULD can pass review when the alternative meets the intended outcome. Reassess exceptions when permissions, data, exposure, or recovery capability change.

## 3. Security and trust boundaries

**Purpose and trade-off:** Inputs, retrieved documents, model outputs, and tool results cross different trust boundaries. Strict contracts can reject useful material, but permissive parsing can turn malformed data or lower-trust instructions into unsafe behavior. [OWASP LLM Top 10 2026][S1] discusses prompt injection, disclosure, and improper output handling; the controls below are this project's design choices.

### SEC-01 — Validate inputs, outputs, and lower-trust content

- **Requirement level:** MUST when applicable.
- **Applicability:** REQUIRED when user, model, retrieval, or tool content enters a decision or output path.
- **Rationale:** Content may be malformed, adversarial, or presented as instructions across a trust boundary.
- **Required behavior:** Define input and output schemas, size limits, and business invariants. Treat retrieved and tool content as data; validate tool results before reuse. Reject or explicitly handle invalid output. Never promote lower-trust instructions into authorization.
- **Verification:** Exercise malformed input/output, prompt-injection content, and invalid tool results; inspect the boundary and failure path.
- **Accepted evidence:** Reviewed schemas and trust-boundary diagram at DESIGN; passing negative-case results and sanitized traces at IMPLEMENTATION.
- **References:** [S1] (LLM01 and LLM10), [S2] (ASI01). These describe risks, not a guaranteed injection filter.

### SEC-02 — Protect secrets and sensitive data

- **Requirement level:** MUST when applicable.
- **Applicability:** REQUIRED when credentials or sensitive data can enter prompts, tools, storage, or telemetry; CONDITIONAL until data classification is known.
- **Rationale:** A read-only system can still disclose data through responses, tool arguments, or traces.
- **Required behavior:** Keep credentials out of source and prompts; scope access to the task and load secrets through an appropriate external mechanism. Minimize sensitive context, define retention and deletion, and redact sensitive telemetry by default.
- **Verification:** Review configuration and data-flow boundaries; test that unauthorized data and secrets do not appear in output or logs.
- **Accepted evidence:** Data classification, credential ownership, and retention rules at DESIGN; configuration review and redacted output/log samples at IMPLEMENTATION.
- **References:** [S1] (LLM02), [S2] (ASI03).
- **Exception handling:** Any retained sensitive payload or broader credential scope needs explicit owner review, access limits, and expiry.

## 4. Tool and action governance

**Purpose and trade-off:** Tools turn model proposals into data access or side effects. A broad tool set can improve flexibility while increasing exposure and the cost of reviewing every possible action. Permission checks belong at the execution boundary, including after a route or handoff. [OWASP Agentic Top 10 2026][S2] covers tool misuse and privilege abuse.

### AUTH-01 — Define and enforce tool authority

- **Requirement level:** MUST when applicable.
- **Applicability:** REQUIRED for any tool or external data access, including read-only tools; CONDITIONAL if planned tool capabilities are unknown.
- **Rationale:** A prompt or model-selected tool name cannot establish the user's authority.
- **Required behavior:** Inventory each tool's owner, purpose, schema, data scope, side effects, and caller. Allowlist tools; validate names and arguments; check current user and tool permissions in application or tool code immediately before execution. Revalidate after routing, supervisor delegation, or handoff.
- **Verification:** Test permitted and denied calls, malformed arguments, and a role or route change that must not retain stale privileges.
- **Accepted evidence:** Tool/permission matrix and enforcement point in the sequence diagram at DESIGN; authorization tests and sanitized denied-action traces at IMPLEMENTATION.
- **References:** [S2] (ASI02–03), [S1] (LLM03).

### AUTH-02 — Govern consequential actions with explicit approval

- **Requirement level:** MUST when applicable.
- **Applicability and approval threshold:** REQUIRED when an action is consequential based on action impact, data sensitivity, reversibility, financial or operational consequences, and available preventive and compensating controls. Record the rationale; there is no universal numeric threshold. NOT_APPLICABLE requires a documented reason that the action is not consequential.
- **Rationale:** Model or user intent can be ambiguous, and an incorrect high-impact action may be difficult to recover.
- **Required behavior and approval mode:** Default to **per-action human approval**: identify the approver, present the exact proposed action, target, and likely effect, and bind the decision to that action and a limited lifetime. An action may instead be **pre-authorized** under an explicitly approved policy with defined scope and limits. Pause for **human escalation** when the action is uncertain, exceptional, or outside those limits. Deny expired, altered, or rejected proposals; require a new decision when scope changes. Validate identity, authority, arguments, and applicable approval or pre-authorization immediately before execution, and record the decision.
- **Alternative-mode controls:** A pre-authorized design must document authorization scope, decision owner, explicit risk acceptance, action limits, audit requirements, reconciliation or rollback capability, and escalation conditions. The designated reviewer must approve this mode before use; a prompt or model proposal cannot grant it.
- **Verification:** Test per-action approval, denial, expiry, and changed arguments; for pre-authorization, test in-scope and out-of-scope actions, limit exhaustion, escalation, and audit/recovery behavior before a side effect.
- **Accepted evidence:** Consequence assessment, selected approval mode, decision owner, limits, and review record at DESIGN; authorization tests and sanitized audit/reconciliation evidence at IMPLEMENTATION.
- **References:** [S2] (ASI01–02 and ASI09). This is a project threshold, not a universal OWASP rule.
- **Exception handling:** A reviewed pre-authorization path that meets the conditions above is an allowed AUTH-02 mode, not a waiver. Any alternative that bypasses these conditions or otherwise waives a MUST remains an explicit exception or compliance gap; do not mark it PASS without formally revising the policy's applicability or requirement.

## 5. Reliability and failure recovery

**Purpose and trade-off:** External calls and resumed workflows can fail after work has begun. More retries may improve availability but increase load and duplicate effects. Failures need truthful outcomes and bounded recovery, especially when a write's result is unknown. [AWS REL05-BP03][S5] supports bounded backoff and avoiding retry storms; [RFC 9110 §9.2.2][S4] defines HTTP idempotent-method semantics, not universal tool idempotency.

### REL-01 — Bound execution and expose truthful outcomes

- **Requirement level:** MUST when applicable.
- **Applicability:** REQUIRED when a request invokes a model, tool, or other external dependency.
- **Rationale:** Unbounded waits and masked failures obscure both user outcome and partial work.
- **Required behavior:** Define overall and per-call deadlines, cancellation propagation, and explicit states for invalid request, denied action, dependency failure, timeout, budget exhaustion, invalid output, and partial completion. Do not report success for a failed or unconfirmed action.
- **Verification:** Inject timeouts, cancellation, dependency failure, and invalid output; inspect the returned state and cleanup.
- **Accepted evidence:** Deadline and outcome contract at DESIGN; fault-injection results and sanitized traces at IMPLEMENTATION.
- **References:** [S2] (ASI08); [S5] informs transient-failure handling.

### REL-02 — Retry safely and reconcile uncertain writes

- **Requirement level:** MUST when applicable.
- **Applicability:** REQUIRED when calls may be retried, replayed after resume, or cause side effects; NOT_APPLICABLE only when none can occur and no retry is planned.
- **Rationale:** A timeout after sending a write is an **unknown outcome** until the external system is checked. Blind replay can duplicate the effect.
- **Required behavior:** Retry only named transient failures with a bounded count and backoff; do not retry denial, invalid input, or business-rule failure. For each write, define an idempotency key or equivalent deduplication and an outcome lookup or reconciliation path. If neither exists, stop automatic retry and escalate the unknown outcome.
- **Verification:** Simulate timeout before and after commit, duplicate requests, resume/replay, and exhausted retry budget; inspect external state, not just the returned text.
- **Accepted evidence:** Retry table, deduplication contract, and unknown-outcome sequence at DESIGN; passing integration tests and external-state records at IMPLEMENTATION.
- **References:** [S4] (§9.2.2, HTTP semantics), [S5] (REL05-BP03, bounded backoff). Neither source mandates this project's idempotency-key design for every tool.
- **Exception handling:** If a dependency cannot support safe replay or lookup, document manual reconciliation and prohibit automatic retry of uncertain writes.

**Worked example — REL-02:** An illustrative purchase-order tool times out after submission. The proposed design records a client operation ID, sends it with the write, queries the external order status on timeout, and avoids a second write until the first outcome is known. This can earn **DESIGN PASS** after review of the contract and failure sequence. It remains **IMPLEMENTATION NEEDS_EVIDENCE** until duplicate and post-commit-timeout tests demonstrate one intended order.

## 6. Resource governance

**Purpose and trade-off:** A single request can multiply into model calls, tools, loops, and parallel branches. Tight limits may truncate useful work; absent limits expose cost and availability. [OWASP LLM Top 10 2026][S1] includes unbounded consumption; numeric thresholds remain application decisions.

### RES-01 — Bound model, tool, and parallel work

- **Requirement level:** MUST when applicable.
- **Applicability:** REQUIRED for metered model/tool calls, loops, or parallel execution; CONDITIONAL until call costs and concurrency are known.
- **Rationale:** Cost, latency, and downstream load can grow beyond one user's apparent request.
- **Required behavior:** Set per-request limits for elapsed time, model/tool calls, token or spend budget, iterations, and concurrent work as relevant. Define what happens at each limit and whether a partial result is valid. Attribute consumption to a request or tenant where needed.
- **Verification:** Force each limit and check that execution stops, state is truthful, and no prohibited side effect continues.
- **Accepted evidence:** Budget table and limit outcomes at DESIGN; limit tests and metering traces at IMPLEMENTATION.
- **References:** [S1] (LLM06), [S2] (ASI08).

## 7. Observability and data governance

**Purpose and trade-off:** Correlation, timing, and outcomes help diagnose failures and audit actions; raw prompts or tool payloads can expose sensitive data. Keep only what operators need, with retention and access controls. [OpenTelemetry semantic conventions][S6] can guide telemetry names, but they do not decide what this application may record.

### OBS-01 — Record useful, protected operational evidence

- **Requirement level:** MUST for deployed systems handling sensitive data or consequential actions; SHOULD for other deployed systems.
- **Applicability:** REQUIRED for deployed systems; CONDITIONAL until deployment, data, and action profile are known; NOT_APPLICABLE only for a documented isolated offline demonstration without sensitive data or meaningful operational logs. The risk assessment determines whether the MUST or SHOULD level applies.
- **Rationale:** An untraceable action is difficult to diagnose or audit; excessive logging creates another disclosure path.
- **Required behavior:** Define correlation IDs, component/tool names, timing, outcome codes, retry and budget counts, and approval/denial events as relevant. Redact sensitive inputs and outputs by default; specify log access, retention, and deletion. Review alerts for material failures, denied actions, cost spikes, or unusual tool sequences.
- **Verification:** Inspect a successful and failed request trace, redaction behavior, access rules, and retention configuration.
- **Accepted evidence:** Telemetry schema and data-handling plan at DESIGN; sanitized traces and configuration evidence at IMPLEMENTATION.
- **References:** [S6] for optional telemetry vocabulary, [S2] (ASI08–09) for agentic traceability risks.
- **Exception handling:** A SHOULD deviation or offline omission needs a diagnostic alternative and explicit support owner.

## 8. Testing and evaluation

**Purpose and trade-off:** Static prose can state a control without proving it works. Offline tests give repeatable evidence for deterministic boundaries; model-dependent evaluation covers variable decisions and outputs. Both need negative cases. [NIST SP 800-218][S7] addresses scoping, performing, and documenting security tests; [NIST AI 600-1][S3] discusses context-specific pre-deployment evaluation and its limits.

### EVAL-01 — Test deterministic controls and failure paths

- **Requirement level:** MUST when applicable.
- **Applicability:** REQUIRED for every substantial design proposal; implementation evidence is required before claiming its controls work.
- **Rationale:** Authorization, schemas, retry policy, and failure mapping should not depend on a live model to be checked.
- **Required behavior:** Map each critical requirement to an owner and an offline test plan. Include allowed/denied tools, malformed data, deadlines, cancellation, retries, idempotency, budget limits, and truthful errors where relevant.
- **Verification:** Review requirement-to-test traceability; run deterministic tests against implementation when it exists.
- **Accepted evidence:** Test cases with pass criteria at DESIGN; reproducible test results and failure records at IMPLEMENTATION.
- **References:** [S7] (SSDF PW.8), [S2] for agentic negative-path risks.

### EVAL-02 — Evaluate model-dependent behavior and acceptance

- **Requirement level:** MUST when applicable.
- **Applicability:** REQUIRED when model output or model-selected actions affect acceptance or external state.
- **Rationale:** A final answer alone may hide wrong tool choices, policy violations, or an incorrect end state.
- **Required behavior:** Define representative success, adversarial, and failure cases; evaluate trajectories and resulting state as well as final output. Repeat important nondeterministic cases, record model/version and evaluation conditions, and state observed variation and limits.
- **Verification:** Run the agreed cases with the intended model, tools, and representative environment before claiming behavioral performance.
- **Accepted evidence:** Evaluation plan and thresholds at DESIGN; versioned datasets, results, reviewed failures, and environment details at IMPLEMENTATION.
- **References:** [S3] (pre-deployment testing and measurement limits), [S1] (LLM01/LLM10 risks).

## 9. Policy compliance and quality gates

For a real system, copy this **starter matrix** into its design proposal. Replace each trigger with REQUIRED, CONDITIONAL, or NOT_APPLICABLE and a reason; add stage, verification state, owner, and evidence location. The rows below are guidance, **not evaluated compliance results**.

| Policy ID | Applicability trigger to assess | System design requirement | Verification | Accepted evidence |
| --- | --- | --- | --- | --- |
| SEC-01 | Untrusted content | Boundary schemas and invalid-output path | Malformed and injection cases | Contracts; test results |
| SEC-02 | Secrets or sensitive data | Credential scope, minimization, retention | Disclosure and log review | Data map; redacted samples |
| AUTH-01 | Tool or external data access | Tool matrix and execution-time gate | Allow/deny and post-handoff tests | Permission matrix; traces |
| AUTH-02 | Consequential action | Risk-based approval mode, limits, and escalation | Approval/scope/expiry/escalation tests | Approval policy; review record; audit events |
| REL-01 | Model or external dependency | Deadlines, cancellation, typed outcomes | Fault injection | Outcome contract; test results |
| REL-02 | Retry, replay, or side effect | Safe retry and reconciliation | Duplicate/timeout-after-commit tests | Retry table; external state |
| RES-01 | Metered or multiplying work | Limits and exhaustion outcomes | Force each limit | Budget table; metering trace |
| OBS-01 | Deployment and support risk | Sanitized trace and retention plan | Trace/redaction inspection | Schema; access configuration |
| EVAL-01 | Substantial design | Offline negative-case plan | Requirement-to-test review | Cases; reproducible results |
| EVAL-02 | Model-dependent behavior | Dataset, criteria, repeat runs | Live evaluation and state review | Versioned report; failures |

### Agent-readable policy review contract

For each Policy ID, report `policy_id`, `applicability`, `stage`, `status`, `reason`, `required_evidence`, `evidence_reference` when available, and `review_owner`. The record is a review aid, not a YAML parser requirement or proof of compliance. For example:

```yaml
policy_id: AUTH-01
applicability: REQUIRED
stage: DESIGN
status: NEEDS_EVIDENCE
reason: "A tool can read customer records; its specified execution-time authorization gate lacks review evidence."
required_evidence:
  - "Tool permission matrix"
  - "Authorization sequence diagram"
evidence_reference: null
review_owner: "Architecture Owner"
```

Apply the states and stage rules in §2: a coding agent's proposed status is not reviewer approval; DESIGN PASS needs documented human review, while IMPLEMENTATION PASS needs observable control evidence. Missing evidence is not PASS, CONDITIONAL cannot pass until resolved, and NOT_APPLICABLE needs a rationale. A waived MUST remains an explicit exception or gap unless the policy itself formally permits the approved alternative path.

**Review checklist**

1. Are capabilities and risk dimensions recorded, with applicability and exceptions justified per Policy ID?
2. Does each REQUIRED policy have an owner, design requirement, failure behavior, and testable acceptance criterion?
3. Do diagrams show authorization before side effects and revalidation after routing or handoff?
4. Are design evidence and implementation evidence labeled separately, with no prose-only implementation PASS?
5. Are unknown outcomes, budget exhaustion, denied actions, sensitive telemetry, and negative tests covered?
6. Has a human reviewer resolved CONDITIONAL policies and approved any AUTH-02 pre-authorization mode before substantial implementation?

## 10. References and source mapping

The source column identifies **external risk or technical guidance**. The Policy IDs and their requirement levels are **project-defined**. Examples such as idempotency keys, exact budgets, approval thresholds, and evidence formats are **implementation recommendations or project requirements**, not mandates from these sources.

| Source | Verified title, version/date, and relevant section | Relationship to policies and limit |
| --- | --- | --- |
| [S1] | OWASP, *GenAI LLM Top 10 2026* (August 2026); LLM01, LLM02, LLM03, LLM06, LLM10 | Risk context for SEC-01–02, AUTH-01, RES-01, EVAL-02. It does not set this project's thresholds. |
| [S2] | OWASP, *Top 10 for Agentic Applications 2026* (December 2025); ASI01–03, ASI08–09 | Agent goal, tool/privilege, cascading, and human-trust risks for SEC-01–02, AUTH-01–02, REL-01, RES-01, OBS-01, EVAL-01. |
| [S3] | NIST AI 600-1, *Generative AI Profile* (July 2024); Introduction and pre-deployment testing | Context-sensitive risk framing and evaluation limits for §2 and EVAL-02. This guide's applicability states are not NIST ratings. |
| [S4] | RFC 9110, *HTTP Semantics* (June 2022), §9.2.2 | Defines HTTP idempotent-method semantics for REL-02; tool APIs need their own guarantees. |
| [S5] | AWS Well-Architected, REL05-BP03, *Control and limit retry calls* (guidance updated July 2023) | Bounded retry, backoff, jitter, and retry-storm context for REL-01–02. |
| [S6] | OpenTelemetry, *Semantic Conventions* 1.44.0; GenAI conventions moved to a [dedicated official repository](https://github.com/open-telemetry/semantic-conventions-genai) | Optional naming reference for OBS-01. It does not authorize collecting content or define retention. |
| [S7] | NIST SP 800-218, *SSDF* v1.1 (February 2022), PW.8 | Secure-development test planning and documented results for EVAL-01; not an agent-specific certification. |
| [S8] / [S9] | RFC 2119 (March 1997) and RFC 8174 (May 2017) | Meanings and uppercase interpretation of normative words in §1; not sources for individual security controls. |

[S1]: https://genai.owasp.org/resource/owasp-genai-llm-top-10-2026/
[S2]: https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/
[S3]: https://www.nist.gov/publications/artificial-intelligence-risk-management-framework-generative-artificial-intelligence
[S4]: https://www.rfc-editor.org/rfc/rfc9110.html#section-9.2.2
[S5]: https://docs.aws.amazon.com/wellarchitected/2023-04-10/framework/rel_mitigate_interaction_failure_limit_retries.html
[S6]: https://opentelemetry.io/docs/specs/semconv/
[S7]: https://csrc.nist.gov/pubs/sp/800/218/final
[S8]: https://www.rfc-editor.org/rfc/rfc2119.html
[S9]: https://www.rfc-editor.org/rfc/rfc8174.html
