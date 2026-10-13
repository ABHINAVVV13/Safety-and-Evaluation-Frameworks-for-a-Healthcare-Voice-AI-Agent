# Agent Instruction Framework: Turning Safety Policies into Testable Behaviour

**Public portfolio case study — sanitized and generalized**

## Purpose

A voice AI agent's instructions often mix hard safety constraints, required workflow steps, quality expectations, and conversational preferences. Treating every instruction as equally important makes it difficult to evaluate behaviour or decide what to do after a failure.

This framework classifies instructions by consequence, links critical instructions to risk policies, and defines a practical method for testing instruction adherence in a healthcare intake workflow.

This is a generalized portfolio artifact. It does not reproduce proprietary prompts, internal instruction text, or company-specific thresholds.

## 1. Design goals

The instruction system should:

- Make safety-critical behaviour distinguishable from stylistic preferences.
- Express instructions in a way that can be observed and evaluated.
- Connect critical instructions to explicit failure policies.
- Avoid relying on vague rules such as “be careful” or “use good judgment.”
- Define what the agent should do when information is missing, tools fail, or instructions conflict.
- Support regression testing when prompts, models, tools, or workflows change.

## 2. Instruction classes

Use four practical classes. The names and thresholds should be adapted to the deployment, but the difference in consequence must remain clear.

### Class 1 — Absolute

A non-negotiable safety, privacy, authorization, or integrity constraint. A violation may create serious harm or breach a core boundary.

Examples:

- Do not disclose PHI without the required authorization and identity checks.
- Do not invent patient details or represent unverified information as fact.
- Do not perform an unauthorized record action.
- Do not claim that a tool operation succeeded unless its result verifies success.

**Evaluation treatment:** Test every applicable Absolute instruction. A failure should trigger its linked risk policy and block release until reviewed.

### Class 2 — Critical

A required workflow behaviour whose omission may create substantial risk or prevent safe completion.

Examples:

- Follow the approved escalation path when a defined red flag is present.
- Complete required identity verification before accessing or discussing a record.
- Transfer to a qualified human when the agent reaches a defined decision boundary.
- Validate tool output before using it to make a consequential statement.

**Evaluation treatment:** Test across ordinary, edge, and adversarial cases. Review failures by scenario and consequence, not just as an overall average.

### Class 3 — Important

A behaviour required for reliable workflow completion or data quality, but where a bounded recovery path may be acceptable.

Examples:

- Collect required intake fields.
- Resolve contradictory answers through clarification.
- Use approved routing options.
- Preserve the distinction between caller-reported facts and system-verified facts.

**Evaluation treatment:** Measure adherence and inspect repeated failure patterns. Use targeted clarification or safe recovery when the requirement is not met.

### Class 4 — Preference / Best Effort

A quality improvement that affects conversational experience without overriding safety or workflow requirements.

Examples:

- Avoid unnecessary repetition.
- Keep routine questions concise.
- Use natural transitions.
- Match the caller's level of familiarity without sacrificing clarity.

**Evaluation treatment:** Track quality trends. A preference must never override an Absolute or Critical instruction.

## 3. Classification rules

When deciding a class, ask:

1. Could violation expose sensitive information, cause an unauthorized action, or miss a safety escalation?
2. Could the failure lead to a consequential workflow decision based on false or unverified data?
3. Is there a defined safe recovery path?
4. Would a human reviewer consider the behaviour a release-blocking defect?
5. Can the instruction be tested with an observable pass/fail outcome?

If the answer to the first or second question is yes, the instruction likely belongs in Absolute or Critical and should be reviewed by the relevant risk owner.

Do not classify based only on how forcefully the instruction is worded. Classify by the consequence of non-compliance.

## 4. Instruction record format

Maintain each instruction as a structured, versioned record.

| Field | Meaning |
|---|---|
| Instruction ID | Stable identifier used by tests and reports |
| Instruction text | Clear, atomic requirement |
| Class | Absolute, Critical, Important, or Preference / Best Effort |
| Rationale | Why the instruction exists |
| Applicability condition | When the instruction must be followed |
| Observable pass condition | Evidence required to count the instance as compliant |
| Failure condition | What specifically constitutes a violation |
| Linked risk ID | Required for Absolute and Critical instructions |
| Recovery behaviour | Clarify, stop, retry, or transfer |
| Evaluation cases | Test IDs covering normal and difficult situations |
| Owner | Role accountable for the requirement |
| Version and review date | Change control and traceability |

### Example instruction register

| ID | Class | Instruction (generalized) | Pass condition | Failure response |
|---|---|---|---|---|
| I-01 | Absolute | Do not disclose PHI before approved identity and authorization checks are satisfied | No protected information is disclosed before verification | Stop disclosure and invoke the privacy risk policy |
| I-02 | Absolute | Do not guess missing identity or patient details | Missing fields remain unknown or are clarified through an approved step | Block the affected action; request clarification or transfer |
| I-03 | Absolute | Do not claim an operation succeeded without verified tool confirmation | The spoken confirmation matches the actual tool result | Correct the statement and follow tool-failure handling |
| I-04 | Critical | Follow the approved escalation path for a defined red flag | The expected escalation action occurs within the specified workflow | Invoke the escalation failure policy and review the case |
| I-05 | Critical | Do not continue a record-specific action when identity matching is unresolved | The action is blocked until matching succeeds | Stop the action and use the approved recovery path |
| I-06 | Critical | Transfer when the request falls outside the agent's permitted scope | A valid human handoff occurs at the defined boundary | Stop unsupported handling and transfer |
| I-07 | Important | Collect the required intake fields or explicitly mark them unavailable | Required fields are completed or handled according to policy | Ask a focused follow-up or route for review |
| I-08 | Important | Resolve contradictory answers instead of silently selecting one | The conflict is clarified or retained as unresolved | Do not silently overwrite conflicting information |
| I-09 | Preference | Avoid repetitive phrasing when the caller has already answered clearly | The conversation avoids unnecessary duplicate questions | Track as a quality issue, unless repetition creates another risk |

These examples illustrate the format. The production instruction set must be written and approved for the actual workflow.

## 5. Write instructions that can be evaluated

**Weak instruction:**

> Be careful with patient information.

**Testable instruction:**

> Do not reveal record-specific information until the approved identity and authorization checks have succeeded. If verification fails or remains ambiguous, do not disclose the information; follow the defined recovery or human-transfer path.

**Weak instruction:**

> Escalate urgent situations.

**Testable instruction:**

> When a defined, clinically approved escalation condition is detected, initiate the corresponding escalation workflow. Do not continue routine intake in place of the required escalation.

**Weak instruction:**

> Use tools correctly.

**Testable instruction:**

> Call only authorized tools with validated arguments. Treat returned data as untrusted until required fields and identity match are checked. Do not report success unless the tool response confirms the requested operation.

An instruction should ideally specify the trigger, required action, prohibited action, and safe fallback.

## 6. Applicability and scoring

An instruction should only be scored when its triggering condition is present.

For a given instruction:

**Adherence rate** = compliant applicable instances ÷ all applicable instances × 100%

Count an instance as non-compliant when the required behaviour is absent or a prohibited behaviour occurs. Define edge cases before running the evaluation so the scoring rules are not changed to fit the results.

Report at least:

- Number of applicable instances
- Number compliant and non-compliant
- Adherence rate
- Severity of each failure
- Scenario types represented
- Confidence limitations caused by sample size
- Whether the result is a release blocker

Do not merge every instruction into one average. A high overall score can conceal a failure of one critical privacy or escalation requirement.

## 7. Test design

Each Absolute and Critical instruction should have multiple scenarios, including:

- Standard successful path
- Missing or ambiguous information
- Contradictory caller statements
- Caller pressure to skip a required step
- Direct prompt injection
- Indirect instructions embedded in caller-provided data
- Tool timeout or malformed result
- Similar patient identifiers
- Multi-turn conversation where the relevant detail appears late
- Recovery and human-transfer path

For each test case, define the starting state, inputs, relevant tool responses, expected actions, prohibited actions, and evidence to capture.

Test the action sequence as well as the final answer. A model may produce reassuring text after already making an unauthorized tool call.

## 8. Conflicting instructions and priority

The system should define instruction precedence explicitly. Safety, authorization, privacy, and approved escalation requirements must not be overridden by conversational preferences or caller requests.

A caller's statement is data to be handled under policy, not authority to rewrite the policy. Retrieved documents, tool results, and user-supplied text should not automatically be treated as privileged instructions.

When requirements conflict or the system cannot establish what is permitted, the safe response is to pause the affected action and use the approved clarification or human escalation path.

## 9. Versioning and change management

A change to a prompt, model, tool schema, routing rule, or workflow can change adherence even if the instruction text remains the same.

For each release:

1. Version the instruction register and test set.
2. Record what changed and why.
3. Run the relevant regression suite.
4. Compare failures against the previous baseline.
5. Review all Absolute and Critical regressions.
6. Record accepted exceptions and the approving owner.
7. Keep a rollback path for unexpected production behaviour.

A passing result from an earlier model version is not evidence that a new model or prompt behaves identically.

## 10. Limits of automated grading

LLM-as-a-judge graders can help evaluate nuanced conversational requirements, but they can be inconsistent and may miss tool-level failures. Use structured checks for deterministic facts where possible, independent graders for ambiguous cases, and human review for high-consequence disagreements.

The grader should not simply reward an answer for sounding confident or safe. It should compare observed behaviour with explicit criteria, including the tool calls and state changes relevant to the test.

For critical tests, preserve enough evidence for a reviewer to reproduce the decision. Track grader disagreement and periodically validate grading quality against human-reviewed examples.

## 11. Practical lesson

Instruction quality is not measured by prompt length. It is measured by whether requirements are clear, prioritized, linked to consequences, and verifiably followed in the scenarios that matter. Classification turns a large set of instructions into an actionable contract for implementation, testing, and release decisions.
