# AI Incident Response Protocol
### AI Governance Framework | AI Operations

---

## Why AI Incidents Are Different

AI incidents are not the same as traditional software incidents.

A traditional P1 has a clear failure state: the system is down, the API returns an error, the build pipeline breaks. The impact is visible and usually binary.

An AI incident can be invisible — the system is running, responses are being delivered, but the outputs are wrong, biased, or harmful. The system appears healthy. The damage is accumulating silently.

This creates a unique challenge: **detection is harder, impact may be broader, and root cause requires different investigative skills.**

This protocol addresses AI-specific failure modes and builds a response system that catches invisible failures before they become visible crises.

---

## AI-Specific Failure Modes

### Category 1: Accuracy Degradation

The model's output quality declines — gradually or suddenly.

**Causes:** Model version change by vendor, shift in input data distribution, accumulation of edge cases outside the training distribution.

**Detection challenge:** Users may not report individual bad outputs — they absorb them as acceptable friction. Degradation is often detected only when it crosses a threshold that makes the system unusable.

**Early warning signals:**
- HITL escalation rate rising week-over-week
- User feedback sentiment declining
- Weekly accuracy sample testing showing drift from Stage 2 baseline

---

### Category 2: Hallucination at Scale

The model generates confident, plausible, but factually incorrect outputs — and they reach users without human review.

**Causes:** Queries outside the model's knowledge domain, over-confidence in low-confidence scenarios, HITL routing failure.

**Detection challenge:** Individual hallucinations may not be identified by users who do not have the domain knowledge to recognise the error.

**Early warning signals:**
- User-reported factual corrections increasing
- HITL reviewers flagging outputs that passed the auto-respond threshold as incorrect

---

### Category 3: Bias or Discriminatory Output

The model produces outputs that demonstrate systematic bias — against user groups, topics, or scenarios — in ways that create legal, ethical, or reputational risk.

**Causes:** Bias in training data, prompt design that amplifies existing model bias, deployment in a use case that was not covered in bias assessment.

**Detection challenge:** Bias may only be visible in aggregate analysis across many outputs — not in individual output review.

**Early warning signals:**
- User complaints clustering around a specific user group or scenario type
- Bias audit finding patterns not present in Stage 2 assessment

---

### Category 4: Data Classification Breach

The system processes data above its approved classification tier — either because input data was misclassified, the system boundary was not properly enforced, or a user deliberately bypassed controls.

**Causes:** User error, inadequate input validation, system configuration gap.

**Detection challenge:** May not be visible in output quality — the system operates normally; the breach is in what was processed.

**Early warning signals:**
- Audit log review detecting keywords or patterns associated with higher-classification data
- User report of accidentally submitting sensitive information

---

### Category 5: HITL Routing Failure

The confidence scoring or routing mechanism fails — causing outputs that should be escalated to human review to be auto-delivered, or causing all outputs to be routed for human review (system effectively offline).

**Causes:** Infrastructure failure, configuration change, model API version change affecting confidence score format.

**Detection challenge:** If all low-confidence outputs are auto-delivered, users receive bad outputs without knowing they should have been reviewed. If all outputs are routed for human review, the system loses its value but may not trigger a standard alert.

**Early warning signals:**
- HITL rate suddenly dropping to near zero (all outputs auto-delivered — possible routing failure)
- HITL rate suddenly jumping to near 100% (routing mechanism stuck in escalate mode)

---

## Incident Classification

AI incidents use a modified classification scheme that adds a Harm dimension to the standard Priority scale.

| Class | Definition | Example | Response |
|---|---|---|---|
| **AI-P1** | AI system producing harmful, discriminatory, or seriously incorrect outputs at scale — being delivered to users without HITL review | Hallucinated medical advice delivered to users | Immediate rollback. Incident bridge open. Director notified. |
| **AI-P2** | Significant accuracy degradation or HITL routing failure — outputs below quality threshold but not causing immediate harm | HITL escalation rate at 45% (above 40% threshold) | Immediate investigation. Director informed. Rollback on standby. |
| **AI-P3** | Isolated output quality issue or near-miss — no scale impact, HITL routing functioning | Single user report of incorrect output; confirmed as edge case | Logged, investigated, runbook updated. No rollback. |
| **Data Breach** | Any data classification breach — regardless of output quality impact | Restricted-tier data submitted to external AI tool | Immediate suspension. Security incident protocol activated. |

---

## Response Protocol

### AI-P1 Response (Immediate Rollback Mode)

```
T+0    Detection: Automated alert OR user report OR HITL reviewer flag
       ↓
T+1    On-call engineer acknowledges. Assesses: Is this AI-P1?
       ↓
T+2    If AI-P1 confirmed: ROLLBACK INITIATED IMMEDIATELY
       Parallel: Director notified. Incident bridge opened.
       ↓
T+5    Rollback complete. System offline or reverted to previous version.
       Stakeholder notification: "AI system temporarily suspended. Investigation in progress."
       ↓
T+30   Initial assessment: What was the failure mode? How many users affected?
       ↓
T+60   Stakeholder update: What happened, scope of impact, next update time
       ↓
T+4hr  Root cause hypothesis. Decision: revert to Stage 2 or full rollback to pre-production.
       ↓
T+24hr Stakeholder update: Root cause confirmed, remediation plan, timeline to re-deployment
       ↓
T+48hr AI Post-Incident Review (see below)
```

**AI-P1 principle:** Roll back first. Investigate after. The cost of a false positive rollback (one hour of system downtime) is always less than the cost of a false negative (hours of harmful outputs reaching users).

---

### AI-P2 Response (Investigation Mode)

```
T+0    Detection: Metric threshold breach OR HITL rate alert
       ↓
T+15   On-call engineer investigates. Confirms AI-P2 classification.
       ↓
T+30   Director informed. Rollback decision criteria documented:
       "We will rollback if [specific condition] by [specific time]."
       ↓
T+1hr  Root cause investigation begins. Is this:
       - Accuracy degradation? → Model retraining or threshold adjustment
       - HITL routing failure? → Configuration check and fix
       - Vendor model change? → Contact vendor; assess impact
       ↓
T+4hr  Decision point: Fix in place and effective? OR Rollback triggered?
       ↓
T+8hr  Stakeholder update: situation, root cause, resolution or rollback status
       ↓
T+48hr AI Post-Incident Review (if AI-P2 involved user impact)
```

---

### Data Breach Response

A data classification breach triggers both the AI Incident Protocol and the organisation's standard Security Incident Protocol simultaneously.

```
T+0    Breach detected
       ↓
T+1    AI system suspended immediately (not rolled back — suspended to preserve evidence)
       ↓
T+5    Security Lead AND Director notified simultaneously
       ↓
T+15   Scope assessment: What data? How much? Through which system? To which vendor?
       ↓
T+30   Legal and compliance team notified
       ↓
T+1hr  Vendor notified (contractual obligation under DPA)
       ↓
T+4hr  Regulatory notification assessment (does this trigger mandatory reporting?)
       ↓
T+72hr Regulatory notification if required (GDPR standard)
```

**Data breach principle:** The system remains suspended until security review confirms remediation. A data breach is not resolved by fixing the technical gap alone — the data exposure must be assessed and addressed.

---

## AI Post-Incident Review

Every AI-P1 and significant AI-P2 triggers a post-incident review within 48 hours.

The AI post-incident review adds three AI-specific sections to the standard post-incident review format:

```
STANDARD SECTIONS
- Incident timeline
- Impact: users affected, duration, outputs delivered
- Root cause
- Immediate fix

AI-SPECIFIC SECTIONS

SECTION A: MODEL BEHAVIOUR ANALYSIS
- What was the model asked to do? (query distribution during incident)
- What did the model output? (sample of affected outputs)
- Why did the model produce this output? (as far as determinable)
- Was the output within the system's documented failure modes?
  If No: add to failure mode documentation

SECTION B: GOVERNANCE CONTROL ASSESSMENT
- Which governance control should have prevented this?
- Why did that control not prevent it?
- Was this a control gap (control didn't exist) or a control failure (control existed but didn't work)?

SECTION C: SYSTEM IMPROVEMENT
- What change to the model, prompt, threshold, or routing prevents recurrence?
- What change to the Responsible AI Checklist adds this as a pre-deployment requirement?
- Is the staged deployment gate criteria sufficient to catch this failure mode?
  If No: update the gate criteria

Owner for each action: [Named person]
Deadline: [Specific date — maximum 2 weeks from review]
```

---

## AI-Specific Monitoring Configuration

Production AI systems require monitoring beyond standard infrastructure metrics.

**Recommended monitoring stack:**

| Metric | Collection Method | Alert Threshold | Alert Destination |
|---|---|---|---|
| Output confidence score distribution | API log analysis | Mean score drops > 10% week-over-week | AI Governance Lead |
| HITL escalation rate | HITL log analysis | > 30% daily | AI Governance Lead |
| User-reported output quality issues | Feedback channel | > 5 reports in 24 hours | Engineering Director |
| Accuracy (weekly sample) | Automated test suite | < Stage 2 baseline − 5% | AI Governance Lead |
| API error rate (to AI vendor) | Infrastructure monitoring | > 1% | On-call engineer |
| Latency p95 | Infrastructure monitoring | > SLA threshold | On-call engineer |
| Prompt injection attempt rate | Security log analysis | Any detected | Security Lead |

---

## Communication Templates

### AI-P1 Initial Stakeholder Notification

```
SUBJECT: [AI-P1] [System Name] — Suspended | [Date Time]

The [System Name] AI system has been suspended effective [time] due to a detected output quality issue.

Impact: [What users are affected / what outputs were affected]
Duration before detection: [Approximate time the issue was active]
Action taken: System suspended. Investigation in progress.

No action required from users at this stage. The system will remain suspended until the investigation is complete.

Next update: [Specific time]
Incident Owner: [Name]
```

### AI-P1 Resolution Notification

```
SUBJECT: [RESOLVED] [AI-P1] [System Name] | [Date Time]

The [System Name] AI system issue has been resolved / the system remains suspended pending further investigation.

Root cause: [One paragraph — what happened and why]
Users affected: [Count or scope]
Outputs affected: [Count and nature]
Resolution: [What was done to fix it / why the system remains suspended]

Post-incident review: [Date] — findings will be shared with stakeholders.

If you believe you received an incorrect AI output during this period, please [specific action].
```

---

## Key Principle

> *The invisible failure is the most dangerous failure. A traditional system that fails, fails loudly. An AI system that fails may fail silently — delivering confident, plausible, wrong outputs at scale while every infrastructure metric reads green. The purpose of this protocol is to make the invisible visible — before it becomes a crisis.*

---

*Part of the [AI Governance Framework](./README.md) series.*
