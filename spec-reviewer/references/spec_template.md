# Universal Spec Template

Every spec — regardless of domain, platform, or methodology — should contain these sections.
Missing sections are gaps. Thin sections are risks.

---

## Required Sections

### 1. Purpose (Why)
One paragraph. Why does this exist? What problem does it solve? What breaks if it doesn't exist?
**Red flags:** Missing entirely. Describes what it does, not why. Written as a solution looking for a problem.

### 2. Goals and Non-Goals
**Goals:** Measurable outcomes. "The system will X" not "the system should try to X."
**Non-Goals:** Explicitly out of scope. Every non-goal prevents a future argument.
**Red flags:** Goals are vague ("be fast", "be reliable"). No non-goals section. Goals that can't be verified.

### 3. Definitions / Glossary
Every domain term, acronym, or concept used before it's fully explained.
**Red flags:** Jargon used without definition. Same concept named differently in different sections. Ambiguous terms that two readers would interpret differently.

### 4. Context and Constraints
- **System context:** Where does this fit? What depends on it? What does it depend on?
- **Technical constraints:** Platform, language, runtime, dependencies, performance requirements.
- **Non-technical constraints:** Budget, timeline, regulatory, organizational, team size.
**Red flags:** No dependency map. Constraints stated as preferences. External system behavior assumed without documentation.

### 5. Stakeholders and Actors
Who uses this? Who implements it? Who maintains it? Who approves changes?
For each actor: what do they do, what do they need, what are their failure modes?
**Red flags:** Missing entirely. "The user" without definition. No operator/maintainer role.

### 6. Design
The main content. Must include all of:
- **Architecture / structure:** Components, their responsibilities, how they connect.
- **Data models:** Every significant data structure defined. Fields, types, constraints, invariants.
- **Interfaces / APIs / protocols:** Every interface defined precisely. Input, output, error cases.
- **State machines:** For any stateful behavior — all states, all transitions, all triggers, all timeouts.
- **Algorithms / logic:** For any non-trivial computation — described precisely enough to implement.
**Red flags:** Describes behavior without defining interfaces. State machines with missing transitions. Data models without field types or constraints. "TBD" in critical paths. Implementation details mixed with design (specifies HOW, should specify WHAT).

### 7. Error Handling and Failure Modes
For every external dependency: what happens if it's unavailable?
For every stateful operation: what is the rollback / recovery?
For every queue or buffer: what happens when it's full?
For every timeout: what is the timeout value and what happens on expiry?
**Red flags:** Missing entirely. "Handle errors appropriately." No recovery procedure for failures. Timeouts mentioned but values not specified.

### 8. Testing Approach
- **Unit:** What gets unit tested? What are the key invariants?
- **Integration:** What integration surfaces exist? How are they tested?
- **Acceptance criteria:** How does a reviewer know the spec is correctly implemented? These are verifiable statements, not vague goals.
- **Backtest / simulation:** For systems with time-series behavior — how is historical behavior validated?
**Red flags:** Missing entirely. "We will write tests." No acceptance criteria. No definition of done.

### 9. Observability
- **Logging:** What events are logged? At what level? In what format?
- **Metrics:** What is measured? What are the alert thresholds?
- **Health checks:** How do you know the system is alive and healthy?
- **Dashboards / on-call:** What does an operator see when something goes wrong?
**Red flags:** Missing entirely. "We will add logging later." No alert conditions. No operator runbook.

### 10. Security Considerations
Credentials, secrets, auth, data sensitivity, attack surface.
**Red flags:** Missing when the system handles credentials, user data, or financial transactions.

### 11. Open Questions
Explicitly listed unknowns. Each must have: the question, an owner, a resolution plan, a deadline.
**Red flags:** No open questions in a complex spec (overconfidence). Open questions with no owner. Open questions that are actually blocking decisions.

### 12. Alternatives Considered
What other approaches were evaluated and why were they rejected?
**Red flags:** Missing entirely (prevents "why didn't you just...?" debates). Alternatives listed without rejection rationale.

### 13. References
Links to related specs, research, prior art, vendor docs, academic papers.

---

## Layer-Specific Extensions

### System Architecture Specs (Layer 1)
Also required:
- Deployment topology (what runs where)
- Environment model (dev/staging/prod or equivalent)
- Operational runbooks (how to start, stop, restart, recover)
- Capacity / scaling model (what are the limits?)

### Project/Framework Architecture Specs (Layer 2)
Also required:
- Extension points (how do layer-3 specs plug in?)
- Versioning strategy (how does this spec evolve without breaking implementers?)
- Migration path from existing systems (if applicable)

### Strategy / Domain Specs (Layer 3)
Also required:
- Per-environment configuration differences
- Calibration plan (which parameters need empirical tuning, and how?)
- Promotion criteria (when is the system ready to advance to the next environment?)
