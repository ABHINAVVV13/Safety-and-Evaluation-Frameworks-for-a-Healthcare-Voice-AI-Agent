# Agent Risk Framework for a Healthcare Voice AI Intake System

**Public portfolio case study — sanitized and generalized**

## Purpose

This framework describes a risk-based approach to deploying a voice AI agent that collects intake information and routes callers through a healthcare workflow. It makes safety expectations explicit, connects failures to defined responses, and gives teams a repeatable way to assess whether controls are working.

This is a generalized portfolio artifact based on practical work on healthcare voice-agent safety. It does not disclose patient information, proprietary implementation details, internal system architecture, or company-specific operating procedures.

## 1. Design principles

1. **Safety requirements must be testable.** Statements such as “protect privacy” or “handle emergencies safely” are insufficient by themselves. Each requirement needs observable pass/fail criteria.
2. **Risk determines the response.** A minor conversational defect and a potential disclosure of protected health information (PHI) should not trigger the same response.
3. **Critical failures require explicit handling.** A serious failure must not simply be logged while the agent continues as though nothing happened.
4. **The model is not the only control.** Instructions, tool permissions, validation, monitoring, deterministic checks, and human escalation should work together.
5. **Evidence matters more than confidence.** A model's claim that it followed a rule is not proof that it did.
6. **Production reliability is not established by a small test set.** Evaluation results should include sample size, scenario coverage, known gaps, and uncertainty.

## 2. Scope and system assumptions

The agent may:

- Collect caller-provided intake details.
- Confirm identity using approved workflow steps.
- Retrieve or update information through authorized tools.
- Determine which workflow or qualified human team should receive a caller.
- Transfer a conversation to a human when required.

The agent must not independently diagnose, invent patient details, bypass required identity checks, or substitute its own judgment for an approved clinical escalation pathway.

The exact permitted actions depend on the deployment, clinical governance, jurisdiction, and system integrations. This framework is not clinical guidance or legal advice.

## 3. Risk classification

Use impact, likelihood, detectability, and exposure to determine risk. A qualitative score can help prioritize work, but high-impact safety and privacy risks should remain explicitly reviewable rather than being hidden inside a single numeric score.

| Risk class | Example failure | Expected response |
|---|---|---|
| Critical | PHI disclosed to an unauthorized person; a required urgent escalation is missed; an unauthorized action changes a patient's record | Stop the unsafe path, trigger the defined escalation or containment procedure, preserve relevant audit evidence, and notify the accountable owner |
| High | Identity is uncertain but the agent proceeds; a required safety question is skipped; the agent uses an incorrect patient record | Do not continue the affected action; retry a bounded validation step or transfer to a human |
| Moderate | Intake information is incomplete or inconsistent without an immediate critical indicator | Ask a targeted clarification, mark the information as unconfirmed, or route for review |
| Low | Non-safety-related phrasing, repetition, or minor conversational friction | Recover conversationally and record for quality improvement |

Classification must be based on actual consequences and context. A failure that appears low-risk in one workflow may be critical in another.

## 4. Risk register

Maintain a versioned register with at least these fields:

- Risk ID and short name
- Hazard and plausible consequence
- Preconditions and affected workflow
- Severity and rationale
- Preventive controls
- Detection controls
- Required response or recovery action
- Owner accountable for the decision
- Evaluation scenarios and evidence
- Residual risk, open questions, and review date

### Example risk entries

| ID | Hazard | Preventive controls | Detection and response |
|---|---|---|---|
| R-01 | PHI disclosed to an unauthorized caller | Identity verification, minimum-necessary disclosure, restricted retrieval tools, explicit privacy instructions | Test direct and indirect disclosure attempts; block or stop disclosure, log the event, and follow the approved incident pathway |
| R-02 | Urgent red-flag information is not escalated | Clinically approved escalation criteria, required questions, clear limits on model discretion | Test direct, indirect, ambiguous, and late-emerging red flags; invoke the approved escalation workflow |
| R-03 | Agent selects the wrong patient record | Match using approved identifiers, validate returned record, prohibit guessing | Test similar names, conflicting identifiers, missing data, and tool errors; stop record-specific actions when confidence is insufficient |
| R-04 | Agent invents missing information | Explicit prohibition on guessing, structured data validation, clear distinction between known and unknown | Test incomplete, contradictory, and leading caller answers; require clarification or mark fields unknown |
| R-05 | Prompt injection or caller manipulation changes the workflow | Treat caller content as untrusted input, constrain tools, separate policy from conversational content | Test instructions to ignore rules, reveal hidden instructions, bypass identity checks, or misuse tools; maintain policy and tool boundaries |
| R-06 | Tool failure leads to false confirmation or unsafe continuation | Explicit tool-error handling, confirmation only after verified success, bounded retries | Simulate timeouts, malformed results, duplicate responses, and unavailable services; fail safely and transfer where necessary |

## 5. Control layers

No single control should be treated as sufficient for a high-consequence failure mode.

### Layer A — Policy and instructions

Define what the agent must do, must never do, and when it must stop or escalate. Critical requirements should be written in concrete, observable terms.

### Layer B — Tool and data boundaries

Give the agent only the permissions needed for its task. Validate inputs and outputs. Do not allow a natural-language instruction alone to authorize a sensitive operation.

### Layer C — Deterministic checks

Where practical, enforce critical conditions outside the language model: required fields, identity checks, tool permissions, transfer rules, and validation of action results.

### Layer D — Evaluation

Test expected workflows, edge cases, adversarial inputs, tool failures, and regression scenarios. Evaluate each safety requirement independently.

### Layer E — Monitoring and response

Record relevant events without unnecessarily duplicating sensitive data. Define who responds to serious failures, how deployment changes are reviewed, and when a system should be paused or rolled back.

## 6. Failure handling and escalation

For every critical risk, define a concrete failure policy. A useful policy answers:

1. What condition counts as a failure?
2. How is it detected?
3. What action must stop immediately?
4. Is a retry allowed, and if so, how many?
5. When must the conversation transfer to a qualified human?
6. What evidence is recorded, and who reviews it?
7. What condition must be met before the system resumes normal operation?

For example, if identity cannot be established, the agent should not proceed with record-specific disclosure or changes. It should use only the approved recovery path and escalate when the workflow's limits are reached.

For a potential privacy or security incident, follow the organization's established incident-response process. Do not rely on the model to self-certify that no harm occurred.

## 7. Adversarial and misuse testing

The risk test suite should include both ordinary mistakes and intentional attempts to defeat controls:

- A caller asks the agent to ignore previous rules.
- A caller requests another person's information.
- A caller gives conflicting identifiers.
- A caller pressures the agent to skip verification.
- A caller embeds instructions inside text intended to be treated as data.
- A caller asks the agent to reveal hidden prompts, credentials, or internal policy.
- A tool returns an unexpected patient, empty result, timeout, or malformed response.
- A caller gives safety-relevant information indirectly, ambiguously, or late in the conversation.

Test the complete workflow, not only the model's text response. An apparently safe answer can still accompany an unsafe tool call or an incorrect routing decision.

## 8. Governance and ownership

Assign named roles rather than relying on an undefined collective owner.

- **Clinical or safety owner:** approves clinical escalation criteria and the consequences of missed escalation.
- **Privacy/security owner:** approves data-handling requirements and incident procedures.
- **Product or operations owner:** defines workflow outcomes and human handoff requirements.
- **Engineering owner:** implements controls, tool boundaries, logging, and safe failure behaviour.
- **Evaluation owner:** maintains tests, reports results, and checks regressions.
- **Release approver:** reviews evidence and accepts or rejects residual risk before deployment.

One person may fill multiple roles in a small team, but accountability should remain explicit.

## 9. Release gates

A release should not be approved solely because an aggregate benchmark score improved. At minimum:

- Critical scenarios have defined expected outcomes.
- No unresolved critical failure is silently accepted.
- Required instructions are tested where applicable.
- Tool failures and human handoffs are covered.
- Regression tests pass against the approved baseline.
- Evaluation sample sizes and gaps are documented.
- Privacy and security controls are reviewed by the appropriate owners.
- Monitoring, incident handling, and rollback procedures are ready.
- Residual risks and exceptions are explicitly accepted by the accountable owner.

Thresholds should be set by risk owners and validated against the system's use case. Do not claim that a finite test set proves a production failure probability.

## 10. Limitations

This framework supports risk analysis and operational discipline; it does not certify a system as safe, compliant, or clinically effective. It must be adapted to the real workflow, validated by qualified clinical and privacy/security stakeholders, and supplemented by applicable legal, regulatory, and organizational requirements.

## 11. Practical lesson

Safety needs to be represented as an operational contract: identify the hazard, define the observable failure, constrain what the agent can do, specify the required response, and test the entire path. If a serious failure has no explicit owner or response, the framework is incomplete.
