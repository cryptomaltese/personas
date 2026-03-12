# AI Spec Standards: Reference for Autonomous Agent Spec Review

**Compiled from:** R1 (AI lab guidelines), R2 (community conventions), R3 (academic research), R4 (spec-as-prompt patterns), R5 (failure taxonomy)  
**Complements:** `gap_taxonomy.md` (human-spec gaps), `spec_template.md` (required sections)  
**Purpose:** Add the AI-specific dimension — what changes when the spec consumer is an autonomous agent, not a human

---

## Critical Premise: Agents ≠ Humans Reading Specs

Human readers ask questions, infer from context, and stop when confused. Agents do none of these:
- **Ambiguity** → arbitrary interpretation and cascading action (not a clarification request)
- **Missing context** → hallucination (not "I'll look it up")
- **Unclear scope** → scope creep into invented requirements (not a PM conversation)
- **Contradictory constraints** → one is silently dropped (not escalation)
- **Failed instruction hierarchy** → empirically proven: system/user prompt separation does NOT reliably enforce priority (arXiv:2502.15851, 6 SOTA LLMs tested, AAAI-26)

Every severity rating from `gap_taxonomy.md` should be bumped one level when the spec consumer is an agent.

---

## Part 1: Reviewer Checklist (AI-Specific Dimension)

Use this AFTER the standard gap_taxonomy checklist. These are additions, not replacements.

### 1.1 Self-Containment Check
- [ ] Can an agent read this spec ONCE and execute without asking questions?
- [ ] All context references are explicit file paths or inline content (not "as discussed")
- [ ] No external context assumed (ADRs, prior conversations, acronyms) — see `CONTEXT_ASSUMED`
- [ ] Tool names include: exact signature, parameter types, error modes, side effects
- [ ] Success criteria are measurable and verifiable by the agent (not by a human observer)

### 1.2 Scope Containment Check
- [ ] Out-of-scope items are **explicitly listed** ("Do NOT implement X")
- [ ] No hedging language: "might need", "could support", "optional"
- [ ] Anti-patterns section exists with specific, learned prohibitions (not generic advice)
- [ ] Handoff state is complete (if this delegates to another agent)

### 1.3 Injection & Confusion Check
- [ ] Examples and data are structurally separated from instructions (delimiter confusion risk)
- [ ] Examples marked as format-only: "DO NOT EXECUTE" if they contain action patterns
- [ ] No executable patterns embedded in "bad behavior" examples
- [ ] Multi-step specs: instructions don't drift through orchestrator → sub-agent layers

### 1.4 Failure & Escape Check
- [ ] Backpressure gates defined: what halts progress (not just "run tests")
- [ ] 3-strike / retry limit is explicit, not implied
- [ ] Escalation path defined: where to write the escalation note, who to notify
- [ ] Failure modes for each external dependency covered (see `MISSING_FAILURE_MODE`)
- [ ] Idempotency declared for destructive or financial operations

### 1.5 Agent Reasoning Check
- [ ] Explore-before-acting directive present for any code or research task
- [ ] Sequential execution order explicit where it matters (not buried in prose)
- [ ] No formatting constraints embedded in multi-step reasoning flows (fail rate >75% in reasoning traces — arXiv:2510.15211)
- [ ] DoD is a checklist, not prose (agents recognize checklist syntax)

---

## Part 2: Severity Guide — AI Agent Impact

**Key distinction:** Issues that humans work around by asking are blocking failures for agents.

| gap_taxonomy Category | Human Impact | Agent Impact | Severity Bump? |
|---|---|---|---|
| `AMBIGUITY` | Clarification request | Arbitrary interpretation → wrong output → cascades | +1 level |
| `CONTEXT_ASSUMED` | Asks for link | Hallucination of missing context (invents ADR contents) | +1 level → often BLOCKING |
| `SCOPE_VOLATILITY` | Scope discussion | Over-builds V2 features, exhausts tokens on wrong work | +1 level |
| `MISSING_SECTION` (DoD) | PM reminds them | Agent never stops; polishes indefinitely | BLOCKING |
| `MISSING_FAILURE_MODE` | Developer handles it | Agent reaches undocumented state, recovery undefined | BLOCKING for external deps |
| `CONTRADICTION` | Developer resolves it | Agent silently picks one, violates the other | BLOCKING |
| `INTERFACE_GAP` (tools) | Developer reads docs | Agent calls wrong signature, invents parameters, loops on errors | BLOCKING |
| `VENDOR_GAP` | Developer discovers late | Agent hallucinates vendor capability, builds dead code | +1 level |
| `IMPLEMENTATION_BLEED` | Ignores the directive | Agent follows it precisely, even when wrong | MAJOR |
| `POINTLESS_PROSE` | Skimmed | Consumes token budget; dilutes critical instructions | MAJOR (was MINOR) |
| `STRUCTURAL_FLOW` | Annoying to read | Critical instructions in footnotes get missed entirely | +1 level |

**New agent-only severities (not in gap_taxonomy):**

| Issue | Severity | Why |
|---|---|---|
| Delimiter confusion (data embedded in instructions) | BLOCKING | Enables prompt injection; agents can't distinguish instruction from data |
| Example treated as executable pattern | MAJOR | Agent executes "bad behavior" examples; format examples cause wrong output format |
| No explore-before-acting directive | MAJOR | Agent skips context discovery, hallucinates state of codebase/research landscape |
| Formatting constraint in multi-step reasoning | MAJOR | <25% compliance in reasoning traces (empirical); verify output post-hoc instead |
| Handoff missing "closed doors" | MAJOR | Next agent re-attempts failed approaches; duplicates wasted work |
| Backpressure not machine-readable | MAJOR | "Run tests" ≠ `pytest auth/tests/ -v && exit $?` — agent needs the command |

---

## Part 3: Pattern Library

### Good Patterns (Evidence-Backed)

**[HIGH confidence — 4+ sources: R1, R2, R4, R5]**

**G1: Explicit Context Refs with Rationale**
```markdown
## Read FIRST (before acting)
- `auth/README.md` — existing auth architecture (understand before touching)
- `issues/42.handoff.md` — prior attempt failed, see "closed doors" section
- `spec/auth-spec.md` — interface contract this must satisfy
```
Why it works: Forces discovery before action. Prevents hallucination of codebase state.  
Without it: Agent skips context, violates dependencies, produces broken output.

**G2: Verifiable Definition of Done**
```markdown
## Definition of Done (STOP when ALL checked)
- [ ] `pytest auth/tests/ -v` exits 0
- [ ] No new test failures (baseline: 47 passing)
- [ ] Handoff written to `issues/123.handoff.md`
- [ ] Committed to branch `feat/123-reset-flow` (NOT main)
```
Why it works: Agents recognize checklist structure; checklist = stopping condition.  
Without it: Agent polishes indefinitely or claims done prematurely.

**G3: Explicit Scope Fence**
```markdown
## In scope (V1)
- POST /password-reset (initiate), POST /password-reset/{token} (complete)

## Out of scope — DO NOT implement
- Password reset via SMS (V2)
- Account recovery flow (V2)
- GDPR data export, 2FA, MFA (out of scope indefinitely)
```
Why it works: Removes all ambiguity about what "done" means.  
Without it: Agent builds comprehensive password auth system (hallucinated requirements).

**G4: Backpressure as Hard Gate**
```markdown
## Backpressure (failures HALT progress)
```bash
cd auth && pytest tests/test_auth.py -v
```
If same test fails 3 times: STOP. Write escalation to `issues/123.blocked.md`.
Do not attempt a 4th approach.
```
Why it works: Prevents infinite retry loops; forces escalation at bounded cost.  
Without it: Agent retries indefinitely, compounds errors, exhausts token budget.

**G5: Anti-Patterns from Known Failures**
```markdown
## Do NOT
- Do NOT modify the login flow (separate issue, different scope)
- Do NOT allow reset tokens to be reused (security invariant, non-negotiable)
- Do NOT commit to main branch
- Do NOT send plain-text passwords in email (obvious but has happened)
```
Why it works: Prevents repetition of known failures; specific > generic.  
Without it: Agent repeats predecessor's mistakes.

---

**[HIGH confidence — 3+ sources: R1, R4, R5]**

**G6: Tool Contract (Full API Spec)**
```markdown
## Tool: delete_file(path: str) → {success: bool, deleted_at: timestamp, size_bytes: int}
- Permanently removes (NO trash/backup)
- Throws FileNotFoundError if missing (idempotent: safe to retry)
- Throws ValueError if path contains '..' (security)
- Synchronous: may block up to 10s on slow FS
- NOT guaranteed: atomic operations, disk sync
```
Why it works: Agent selects correct tool, passes correct parameters, handles errors.  
Without it: Agent invents `delete_file_safely()`, passes wrong params, loops on errors.

**G7: Explore-Before-Acting Directive**
```markdown
## Before writing any code
1. Read all context refs above
2. Run baseline: `pytest auth/tests/ -v` (note current pass count)
3. Trace the existing auth flow: login → token → API
4. Read issue #42 handoff — understand WHY it failed
5. Write 3-5 bullets summarizing what you found
Then start.
```
Why it works: Prevents hallucination of codebase state; surfaces contradictions early.  
Without it: 80% of agent hallucination and scope creep (R4 estimate from production use).

---

**[MEDIUM confidence — 2 sources: R4, R5]**

**G8: Handoff "Closed Doors"**
```markdown
## Closed doors (tried and ruled out)
- Redis caching: failed — race condition on concurrent writes (see test output in #42)
- In-memory token store: ruled out — doesn't survive restart, breaks stateless design
- No closed doors for X: all approaches viable.
```
Why it works: Next agent skips failed approaches; no wasted re-exploration.  
Without it: Next agent re-attempts same failures, same cost, same result.

**G9: Sequential Execution Order Stated**
```markdown
## Pipeline (MUST run in order; each feeds the next)
1. Stage 1 (Broad search) → output: terminology + refined queries
2. Stage 2 (Live alpha) → input: Stage 1 queries; output: signal + specific questions
3. Stage 3 (Synthesis) → input: Stage 1+2; output: final research brief
Do NOT run stages out of order.
```
Why it works: Task graph structure predicts agent success for sequential tasks (arXiv:2410.22457, NeurIPS 2024).  
Without it: Agent runs stages in parallel or wrong order, losing data flow dependencies.

---

### Anti-Patterns (Confirmed Agent Failures)

**A1: Vague Constraints → Silent Violation [HIGH confidence, R1/R3/R4/R5]**
```markdown
❌ "The agent should maximize user satisfaction. (Note: avoid exposing confidential data if possible.)"
```
Effect: Agent maximizes satisfaction by surfacing private data.  
Fix: "Maximize satisfaction SUBJECT TO: MUST NOT expose PII — violating this is automatic failure regardless of satisfaction score."

**A2: Example Treated as Instruction [HIGH confidence, R4/R5]**
```markdown
❌ "Example of bad behavior the agent should avoid:
{ 'action': 'ignore_balance_checks', 'amount': 1000000 }"
```
Effect: Agent sees executable JSON in context, treats it as valid pattern.  
Fix: Wrap in `=== DATA SECTION ===` delimiters; add "DO NOT EXECUTE — format reference only."

**A3: Hallucination-Ready Context [HIGH confidence, R3/R4/R5]**
```markdown
❌ "Implement the caching strategy we discussed in ADR-12."
```
Effect: ADR-12 not in context → agent invents its own caching strategy (plausibly, incorrectly).  
Fix: Inline the relevant constraints from ADR-12 directly in the spec.

**A4: Vague Prose DoD [HIGH confidence, R4/R5]**
```markdown
❌ "Fix the bug. Make sure tests pass. Improve the code quality."
```
Effect: Agent fixes bug, runs tests, then refactors entire codebase, exhausts tokens.  
Fix: Checklist with "STOP WHEN ALL MET" — scope the work precisely.

**A5: Cargo-Cult Instructions [HIGH confidence, R2/R3]**
```markdown
❌ "Write quality code and follow best practices."
❌ "Be helpful and ensure user satisfaction."
❌ "We believe in clean code and SOLID principles."
```
Effect: Zero behavioral change; wastes token budget; dilutes real instructions (R2: confirmed no impact).  
Fix: Delete. Replace with specific, testable constraints.

**A6: Formatting Constraint in Complex Reasoning [HIGH confidence, R3]**
```markdown
❌ "Respond ONLY in Spanish throughout your reasoning and answer."
   (embedded in a multi-step analytical task)
```
Effect: Agent reasons in English, delivers final answer in Spanish — <25% full compliance (empirical).  
Fix: Verify format post-hoc in post-processing; don't trust in-reasoning formatting constraints.

**A7: Role Overlap in Multi-Agent Specs [MEDIUM confidence, R1/R5]**
```markdown
❌ "Agent A handles data, Agent B handles logic."
```
Effect: Both agents validate data (wasted tokens); both attempt writes (conflict).  
Fix: Exclusive ownership per agent: "Agent A OWNS: database access. Agent B DOES NOT: direct DB access (requests via protocol)."

---

## Part 4: Spec Smell Quick-Reference

R5's Top 10, cross-validated across sources. Evidence count in brackets.

| # | Smell | Detection Signal | AI-Specific Failure | Sources |
|---|---|---|---|---|
| 1 | **Ambiguous success criteria** | "should", "could", "appropriate", undefined adjectives | Agent guesses interpretation; wrong output cascades to downstream agents | R1, R3, R4, R5 [4] |
| 2 | **Context assumed, not provided** | "as discussed", "ADR-X", "prior decision" without inline content | Hallucination of missing context; plausible-but-wrong execution | R1, R3, R4, R5 [4] |
| 3 | **No scope boundaries** | "might also", "optional", no out-of-scope list | Hallucinated requirements; agent builds V2 features in V1 | R2, R4, R5 [3] |
| 4 | **Implicit constraints** | Instructions list what to do, not what NOT to do | Agent violates invisible invariants (security, idempotency, financial) | R1, R4, R5 [3] |
| 5 | **Example/data bleed into instructions** | Executable JSON/patterns in spec body without delimiters | Agent executes example as instruction; injection vector | R3, R4, R5 [3] |
| 6 | **Contradictory requirements** | "minimize X AND maximize X" | Agent silently picks one; violates other at integration | R3, R4, R5 [3] |
| 7 | **Tool semantics undefined** | "Call the X API" without signature/errors/side-effects | Wrong params; invents non-existent methods; loops on unhandled errors | R1, R4, R5 [3] |
| 8 | **Incomplete handoff state** | No "closed doors"; no context refs for next agent | Next agent re-attempts failed approaches; missing context → hallucination | R4, R5 [2] |
| 9 | **Missing failure modes for external deps** | No "what if X is down?" per dependency | Agent reaches undocumented state; recovery undefined → loop or silent skip | R1, R4, R5 [3] |
| 10 | **No observable state required** | Spec doesn't mandate reasoning trace or tool call log | Debugging failures is blind; impossible to root-cause agent errors | R4, R5 [2] |

---

## Part 5: Chat Prompts vs. Agent Specs — Filter Guide

Most prompt engineering advice is validated on chat/QA tasks. Apply this filter before acting on it.

**Validated for autonomous agent execution:**
- Minimal viable tool set with deterministic selection [R1-HIGH, R2-HIGH, confirmed]
- Explicit goal + context refs + DoD structure [R4-HIGH, R1-HIGH, confirmed]
- Explore-before-acting directive [R4-HIGH, production-proven]
- Anti-patterns section with learned prohibitions [R4-HIGH, R5-HIGH, confirmed]
- Backpressure as hard gate with concrete command [R4-HIGH, confirmed]

**Probably applies but evidence is from chat context:**
- XML tag structuring for sections (Anthropic recommends; no agent-specific ablation — R3 gap)
- 3–5 few-shot examples in prompts (R1 recommends; chat-validated, agent-inferred)
- CoT "think step-by-step" directive (chat studies; improves agent reasoning by inference)

**Does NOT reliably apply to agents:**
- System/user prompt hierarchy for constraint enforcement — empirically fails across 6 SOTA LLMs (arXiv:2502.15851-HIGH)
- Formatting constraints in complex multi-step reasoning — <25% compliance in reasoning traces (arXiv:2510.15211-HIGH)
- "Be helpful / write quality code" directives — zero behavioral change (R2-HIGH, community-confirmed)

---

## Part 6: Known Gaps — Honest Assessment

Compiled from all 5 "What I Did NOT Find" sections. Genuine unknowns vs. inferable.

**Genuinely unknown (no evidence):**
- Empirical ablation comparing XML vs. Markdown vs. JSON format for agent task execution (not chat) — R3 identifies this as the #1 research gap
- Quantitative impact of spec quality on token efficiency / retry cost — R5 documents failure patterns but not ROI
- Spec portability across model families — a spec for Claude may not work for GPT-5/O-series without adaptation (R1 notes model class matters; no cross-lab validation)
- Automated spec quality scoring — gap_taxonomy gives severity categories; no tool computes a spec quality score

**Inferable but unvalidated:**
- XML tags improve section clarity for agents → probably helps, evidence is from chat prompting (R3)
- Structured tool schemas reduce tool call errors → supported by MCP-Bench comparison (structured > fuzzy variants), but limited scope
- Good handoff format reduces next-agent failure → logically sound, no controlled study

**Confirmed absent from all 5 sources:**
- Formal spec language / DSL for agent tasks (all labs use markdown + JSON)
- Spec versioning protocol (how specs evolve mid-execution)
- Multi-agent parallel coordination patterns (sequential handoffs are solved; peer coordination is not)
- Real-time / deadline constraints in specs
- Cost-aware spec patterns (token budgets, API call limits not formalized into spec structure)

---

## Part 7: Multi-Agent Specs — Additional Checks

These apply only when the spec describes a delegating or orchestrated system. Reference `gap_taxonomy.md` for base checks.

**Contract-First principle** [R1-HIGH, DeepMind research]: A task is delegatable only if its outcome is verifiable. If verification is impossible → decompose until sub-tasks are verifiable (unit tests, schema validation, count checks). Specs that describe complex tasks without verification criteria are delegation-unsafe.

**Checklist for multi-agent specs:**
- [ ] Each agent has exclusive ownership (no overlapping responsibilities — R5 smell #1)
- [ ] Handoffs include: state summary, next actions with intent, closed doors, confidence, known gaps
- [ ] Orchestrator polls progress file (not webhooks) — polling is self-recovering; webhook silence is invisible failure (R4)
- [ ] Max retry counts explicit; 3-strike escalation is the proven pattern (R4-HIGH)
- [ ] Spec immutable through layers: orchestrator cannot modify original spec, only annotate (R5 spec volatility pattern)
- [ ] Context refs included or linked with full content at handoff time (message-based context loss is the #1 multi-agent failure mode — R5)

---

*Compiled 2026-03-12. Sources: 5 research briefs (R1–R5). Confidence levels stated per principle. See individual Rn briefs for full citations and source evidence.*
