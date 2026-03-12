# Gap Taxonomy

Use these categories when labeling findings in the gap report.

---

## Category Definitions

### CONTEXT_ASSUMED
Content that requires project knowledge the spec doesn't provide.
- Acronyms expanded nowhere in the document
- References to prior decisions not documented here
- "As discussed" or "as we agreed" without a reference
- A term used 20 times but defined only in another document with no link
**Severity default:** MAJOR (blocks new team members)

### AMBIGUITY
Two competent implementers would build different things.
- "Should" instead of "must/will" in a requirement
- Ranges without specifying inclusive/exclusive ("between 5 and 10")
- Units not specified ("latency < 100" — milliseconds? microseconds?)
- "Fast", "small", "appropriate", "reasonable" without definition
- Enum values implied but not listed
**Severity default:** MAJOR to BLOCKING depending on the path

### SCOPE_VOLATILITY
The boundary between in-scope and out-of-scope is unclear or porous.
- Non-goals section missing
- "We might also..." language
- Features described as "optional" without criteria for when to include them
- Spec grows in scope mid-document without flagging the change
**Severity default:** MAJOR

### CONTRADICTION
The spec disagrees with itself or another spec in the suite.
- Same term defined differently in two sections
- Requirement A implies X; requirement B implies not-X
- Interface defined differently in layer 2 and layer 3
- A decision marked "closed" in one spec, open in another
**Severity default:** BLOCKING

### NON_FEASIBLE
The spec describes something that cannot be built as written.
- Impossible performance requirements (sub-millisecond on a polling API)
- Circular dependency (A initializes B, B initializes A, no resolution)
- Two mandatory constraints that cannot both be satisfied
- A required external capability that the vendor doesn't offer
**Severity default:** BLOCKING

### MISSING_SECTION
A required section from the universal template is absent.
- No testing approach
- No error handling
- No observability
- No goals/non-goals
- No glossary despite heavy domain jargon
**Severity default:** Varies by section (Testing/Error handling = MAJOR; Glossary = MINOR in simple specs)

### THIN_SECTION
Section exists but lacks the detail needed to implement.
- Goals stated but not measurable
- State machine defined but missing transitions
- Interface mentioned but not typed
- "Handle errors appropriately" as the error handling section
**Severity default:** MAJOR

### INTERFACE_GAP
A connection to another system is described but not fully specified.
- "Calls the X API" without specifying request/response schema
- "Emits an event" without defining the event structure
- "The adapter handles this" without defining the adapter protocol
- Cross-spec reference to a section that doesn't exist
**Severity default:** BLOCKING for critical paths, MAJOR otherwise

### VENDOR_GAP
A dependency on a third-party system or library is assumed without documentation.
- "Uses Nautilus Trader's X feature" without linking to docs or confirming the feature exists
- Capability assumed from a library without citing the API
- "The exchange supports this" without a source
- **Behavioral claim without source verification:** "Tool X does Y on Z" — if the architecture depends on this behavior, the reviewer MUST flag the missing citation and specify which behavior claim needs verification. Evidence means: link to source, test output, or explicit "verified in vX.Y.Z" annotation. Specs routinely confuse what a tool *should* do with what it *actually* does. The implementer's fix is to clone the dependency and verify the behavior claim before building against it — the reviewer flags, the implementer verifies.
**Severity default:** MAJOR (discovered late, blocks implementation)
**Escalation:** If the spec's architecture is load-bearing on unverified external tool behavior (e.g., "push moves files," "exits non-zero on conflict"), escalate to BLOCKING.

### TEST_GAP
The spec cannot be verified.
- No acceptance criteria
- No definition of done
- No mention of how to validate behavior in each environment
- Speculative parameters with no calibration plan
**Severity default:** MAJOR

### POINTLESS_PROSE
Text that consumes space without conveying information.
- Introductory paragraphs restating what the section header says
- Hedging language ("it is worth noting that...", "as we can see...")
- Repetition of content from another section without adding nuance
- Motivational content ("this is important because...")
**Severity default:** MINOR (unless it obscures critical information, then MAJOR)

### IMPLEMENTATION_BLEED
The spec dictates HOW instead of WHAT.
- Specifies the exact class name or file path of an implementation
- Describes an algorithm at the code level when behavior-level is sufficient
- Requires a specific library when any library meeting a capability spec would do
- Design decision disguised as a requirement
**Severity default:** MINOR to MAJOR (MAJOR when it constrains future maintainability)

### ASSUMPTION_LEAKAGE
The spec depends on context that exists only in another spec, without cross-referencing.
- "As defined in the system architecture" without a section reference
- Terminology from spec 01 used in spec 03 without re-definition or link
- A policy declared in spec 02 invoked in spec 03 without confirming it exists
**Severity default:** MAJOR

### OVER_ENGINEERING
A mechanism that is significantly more complex than the problem it solves.
- Generic framework for exactly one known use case → hardcode it, abstract later
- Config schema with 15 fields, 12 of which will never change from defaults → constants, not config
- Multi-mode system (reactive/proactive, pull/push) when only one mode has a concrete use case → pick one
- Multi-step install ceremonies disproportionate to system complexity → simpler setup
- Defensive handling of scenarios that are impossible or inconsequential → remove
- **The test:** Remove this mechanism. Does the system still work for all known use cases? If yes, it's speculative.
- **Distinct from WHEEL_REINVENTION:** Wheel reinvention means a standard tool exists. Over-engineering means no standard tool exists, but the custom solution is 10x more complex than needed.
**Severity default:** MAJOR (BLOCKING if entire subsystem is over-engineered; MINOR for small unnecessary formalisms)

### MVP_SCOPE_VIOLATION
A feature that doesn't need to exist in V1 and can be cleanly added in V2 without rewriting V1.
- "Future-proofing" features with no current user → defer
- Mechanisms that don't block the core loop (detect → assign → execute → verify) → defer
- Multi-repo support when there's one repo → defer
- Offline/fallback modes before online mode is proven → defer
- Dashboard/reporting features before the system runs once → defer
- **Key judgment:** Can this be added in V2 without changing V1's file layout, data model, or interface contracts? If yes → defer. If no → might be cheaper to include now.
- **Not the same as over-engineering:** A feature can be perfectly engineered and still be wrong to build now. Over-engineering = too complex. MVP violation = too early.
**Severity default:** MAJOR (MINOR if the feature is small and doesn't add maintenance burden; BLOCKING if V2 features are blocking V1 shipment)

### WHEEL_REINVENTION
A custom mechanism for a problem that has a standard, proven solution.
- Custom log rotation logic → `logrotate` (ships with Ubuntu/RHEL)
- Custom config format sprawl → single standard format (YAML/TOML) with standard merge semantics
- Custom state tracking (timestamp files, "first seen" maps) → `cron` scheduling, or fields already present in the data
- Custom verification via filesystem snapshots + log cross-references → tool's own exit code + stderr
- Custom template/matching system → platform-native template feature (GitHub issue templates, CI templates, etc.)
- Custom file discovery scripts → existing CLI one-liners (`find`, `git ls-files`, `fd`)
- Custom orchestration layer → existing tools the spec already depends on
- **The test:** "Does the tool this wraps, or the OS this runs on, already handle this?"
- **Pattern:** Builders are blind to wheels they're reinventing because they're deep in the problem. This category exists to catch what they can't see.
**Severity default:** MAJOR (BLOCKING if an entire subsystem is a wheel; MINOR for small unnecessary formalisms)

### LLM_AS_BATCH_RUNNER
An LLM agent is used to perform purely deterministic operations that require zero reasoning.
- Orchestration loops where every decision is a string comparison, file existence check, or CLI call → should be a bash script on a cron
- "Read file, check condition, call tool" workflows where every branch is a flowchart → cron + curl/CLI
- Agent HEARTBEAT sections that just batch shell commands sequentially → system cron, zero tokens
- Using an agent session as a relay to call a single tool with pre-computed parameters → call the tool directly via HTTP API or CLI
- **The test:** "Would a bash `if/else` handle every decision in this workflow?" If yes, an LLM is the wrong executor.
- **The smell:** The spec says "the agent checks..." or "the heartbeat reads the file and..." when describing operations with no judgment calls. If a senior dev would be insulted to do this work manually, an LLM shouldn't be doing it either.
- **Cost dimension:** Every LLM invocation has token cost + latency + non-determinism risk. A bash script is free, instant, and does exactly what you wrote. The spec should justify WHY an LLM is needed for each operation, not assume it by default.
- **Subtlety:** Sometimes 95% of a workflow is deterministic and 5% needs reasoning. The right architecture extracts the deterministic parts into scripts and only invokes the LLM for the reasoning step. Specs that route the entire workflow through an LLM because one step needs it are paying 20x for the other 19 steps.
**Severity default:** MAJOR (BLOCKING if the entire system runs through an LLM with no reasoning justification)

### STRUCTURAL_FLOW
The document's section ordering impedes comprehension.
- Reference material (config schemas, glossaries, API specs) before the reader understands the system
- Glossary terms defined before the concepts they name have been introduced
- Design philosophy interleaved with mechanical how-it-works sections
- Retrospective sections ("What This Replaces", "How We Got Here") in a forward-looking spec
- False hierarchy ("Part 1" / "Part 2") implying equal weight between a core system and a support subsystem
- **The test:** Extract the heading list. Read it as an outline. Does it tell a coherent story: what → why → how → reference? If not, reorder.
- **The drop test:** If a reader stops at section N, have they gotten the most important information so far?
**Severity default:** MAJOR when ordering actively impedes comprehension; MINOR when suboptimal but readable

### MISSING_FAILURE_MODE
A failure case that will occur in production is not addressed.
- External dependency goes down: what happens?
- Queue fills up: what is the backpressure strategy?
- State machine receives an unexpected event: is it ignored? logged? fatal?
- Partial success (some operations succeed, some fail): what is the consistency guarantee?
**Severity default:** MAJOR to BLOCKING for financial or safety-critical systems

### NFR_GAP
Non-functional requirements are missing, vague, or unmeasurable.
- No NFR section in a spec describing a system with external users or operators
- Vague NFR language ("should be fast", "must scale", "highly available") without measurement criteria
- NFR stated without a method to verify it (e.g., "latency under 100ms" has no test plan)
- Performance targets mentioned in design philosophy but not formalized as acceptance criteria
- Reliability, availability, or throughput goals absent from a system expected to handle concurrent load
- **Examples:**
  - ❌ "The system should handle growth" (how much? over what period? at what cost?)
  - ✅ "The system must support 1000 concurrent users with p99 latency ≤ 200ms, verified by load test"
  - ❌ "Encryption should be used" (what algorithm? key rotation? at rest or in transit?)
  - ✅ "All user data encrypted at rest with AES-256, re-keyed annually, per RFC 5116"
**Severity default:** MAJOR (blocks implementation and acceptance testing). BLOCKING if the NFR gap directly contradicts a security or availability goal.

### API_CONTRACT_GAP
An external API or protocol contract is incomplete or inconsistent.
- Request/response schema documented in prose but not formally specified (missing JSON schema, protobuf, OpenAPI, etc.)
- Error responses mentioned but error codes, HTTP status mapping, and error body structure not defined
- Pagination not addressed on list endpoints (page size? cursor vs offset? no mention = ambiguity)
- Naming conventions inconsistent (camelCase vs snake_case vs kebab-case; singular vs plural resource names)
- Rate-limiting, timeouts, or retry semantics not specified
- Backward compatibility strategy absent (how are breaking changes versioned?)
- **Examples:**
  - ❌ "The service calls the exchange API to fetch prices" (what request? what response format?)
  - ✅ "GET /api/v1/prices?symbol=BTC&limit=100 returns `[{id, symbol, price_usd, timestamp}]` or 429 with Retry-After"
  - ❌ "Error codes are returned" (which ones? on what conditions?)
  - ✅ "400 Bad Request for invalid symbol; 429 Too Many Requests with Retry-After; 500 Internal Server Error with error_id + message"
**Severity default:** BLOCKING for critical path APIs, MAJOR otherwise.

### SECURITY_GAP
Authentication, authorization, audit, encryption, or threat response is incomplete.
- No authentication or authorization model defined (how are users/services authenticated? what roles/permissions exist?)
- Sensitive operations lack an audit trail or immutable log
- Encryption mentioned but algorithm, key management, or scope unclear ("data is encrypted" without specifying how or where)
- Threat model absent; protection against specific attack vectors not addressed
- Secrets (API keys, tokens, passwords) management not specified (generation, rotation, revocation, storage)
- Access control scoping unclear (who can access what? under what conditions?)
- **Examples:**
  - ❌ "The system is secure" (against what threats? using what controls?)
  - ✅ "Multi-user: OAuth 2.0 + role-based access. Single-user: API key in environment. All auth tokens expire in 1h, rotated on refresh. See Security Model section."
  - ❌ "Data is encrypted" (where? how?)
  - ✅ "User PII encrypted at rest with AES-256-GCM (keys in AWS KMS); in transit via TLS 1.3. Audit log stored separately, signed, immutable."
  - ❌ "Sensitive operations are logged" (which ones? how? retention?)
  - ✅ "All financial transactions, permission changes, and user deletions logged to audit_events table with timestamp, actor, action, resource_id, result. Retention: 7 years."
**Severity default:** BLOCKING for multi-user systems. MAJOR for single-user systems with sensitive data. MINOR if the system handles only public data.

### ADR_QUALITY
An architecture decision record (ADR) is incomplete or presents alternatives without rejection rationale.
- Alternatives listed but no reasoning for why one was chosen and others rejected
- Only one option presented ("Decision: Use X"), treating a design decision as inevitable rather than chosen
- No clear statement of the decision itself or its scope
- Context insufficient to understand why the decision matters (problem → options → tradeoffs → choice should be visible)
- Decision framed as temporary ("for now", "pending review") without criteria for revisiting
- **Examples:**
  - ❌ "Decision: We use PostgreSQL" (why not MySQL? cost? features? when does this no longer apply?)
  - ✅ "Decision: PostgreSQL for persistence. Context: Need ACID + JSON support. Alternatives: MySQL (no window functions), SQLite (no concurrency). Choice: PG + managed service (AWS RDS). Revisit if: data volume > 10GB, latency SLA < 50ms, or cost becomes blocker."
  - ❌ "We chose TDD for testing" (what did you reject? TDD vs what other approach?)
  - ✅ "Decision: Test-driven development (TDD). Rejected: post-hoc testing (slower feedback, harder debugging). Rejected: fixture-based acceptance testing (brittle, slow CI). Rationale: TDD catches integration bugs early. Scope: unit tests only; integration tests via fixtures in a separate suite per the Integration Testing ADR."
**Severity default:** MAJOR (blocks future maintainers from understanding or revisiting the decision). MINOR if the ADR context is already elsewhere (e.g., GitHub discussion with clear alternatives) and linked.

### ASSUMPTION_UNSTATED
The spec makes an assumption about environment, dependencies, user behavior, or infrastructure that is neither stated nor validated.
- Assumptions about tool availability (agent has access to tool X; tool X is installed; tool X version ≥ Y)
- Assumptions about timing (responses within Ns; queues don't overflow; retries succeed within N attempts)
- Assumptions about data availability (external system always responds; remote is always reachable; file always exists)
- Assumptions about permissions (user has write access to directory; agent can spawn sub-processes; credentials are present)
- Assumptions about concurrency (operations don't race; state is consistent; locks are released)
- Assumptions about user knowledge (users understand concept X; implementer knows tool Y; operator has domain expertise)
- **Examples:**
  - ❌ "The heartbeat spawns sub-agents" — assumes: agent runtime supports spawning; user has infrastructure to run them; the agents inherit auth context
  - ✅ "The heartbeat spawns sub-agents (requires: OpenClaw runtime ≥ v2.5; all spawned agents run in same session context; sub-agent model defaults to Haiku)"
  - ❌ "Calls the GitHub API to list repos" — assumes: API is available; network is connected; response time < 5s; user has auth token
  - ✅ "Calls the GitHub API to list repos (assumes: `gh` CLI installed locally with active auth; assumes API responds within 5s; retries up to 3× on timeout; will surface credential errors explicitly)"
  - ❌ "The spec uses the tool's native feature X" — assumes: feature X exists and hasn't changed in the tool version being used
  - ✅ "The spec uses GitHub issue templates (requires: templates file in `.github/ISSUE_TEMPLATE/`; assumes: GitHub Enterprise ≥ v2.21.0 supports template dropdowns)"
- **Discovery pattern:** For every external dependency, tool capability, timing assertion, or infrastructure claim in the spec, ask: "What would break if this assumption is wrong?" If the answer is "implementation would fail," the assumption is unstated and risky.
- **Key insight (from PMPrompt + INCOSE):** Hidden assumptions are the most frequent cause of spec failure because they're invisible. Making assumption inventory explicit is the highest-value review activity. This category exists to force them out of hiding.

  *Attribution: This concept is informed by PMPrompt (systems engineering requirement validation methodology) and INCOSE Systems Engineering Handbook's assumption identification framework.*

**Severity default:** MAJOR (discovered late in implementation, blocks progress). BLOCKING if the assumption is about a critical capability the agent doesn't actually have (e.g., "agent can provision cloud infrastructure" when it cannot).

---

## Issue-Specific Categories (15% extension to core taxonomy)

These categories are issue-specific variants or new patterns that emerge in GitHub issue review.

### MISSING_FIELD
**Severity Range:** BLOCKING → MAJOR → MINOR
**Definition:** A critical field is absent from the issue form or prose description that is necessary for an implementer to start work.
**Applies To:**
- Bug reports missing reproduction steps
- Feature requests missing acceptance criteria
- RFCs missing design rationale
- Performance issues missing current/target metrics
**When to Flag:** 
- Issue type suggests a field that should be present (e.g., "feature request" should have "use case")
- The field is necessary for an implementer to start work
- Absence creates ambiguity about what "done" looks like
**Severity Defaults:**
- BLOCKING: Critical field (reproduction, acceptance criteria, test procedure)
- MAJOR: Important field (environment, motivation, design rationale)
- MINOR: Supporting field (related issues, prior art, future extensions)
**Example:**
```
[MAJOR] MISSING_FIELD — No acceptance criteria defined
Location: Issue body (missing entirely)
Finding: Feature request describes what to build but not how to verify it's done.
Resolution: Add section:
  ## Acceptance Criteria
  - [ ] User can log in with username + password
  - [ ] Session persists across page refresh
  - [ ] Logout clears session
```

---

### THIN_FIELD
**Severity Range:** BLOCKING → MAJOR → MINOR
**Definition:** A required field is present but lacks sufficient detail for implementation.
**Applies To:**
- Acceptance criteria that are vague ("system should be fast")
- Reproduction steps that skip key details
- Design sections that lack edge case coverage
**When to Flag:**
- Field is present but an implementer would need to ask for clarification
- Field contains hedging language ("should", "might", "could")
- Field lacks measurable criteria or examples
**Severity Defaults:**
- BLOCKING: Vague acceptance criteria (how do you know when to stop?)
- MAJOR: Thin reproduction steps (can't reproduce the bug)
- MINOR: Thin edge case coverage (could be discovered in code review)
**Example:**
```
[MAJOR] THIN_FIELD — Acceptance criteria lack measurable targets
Location: "Acceptance Criteria" section
Finding: Criteria say "API should be fast" without specifying latency target.
Resolution: Replace with: "API response time: p99 ≤ 100ms under load test (100 req/s)"
```

---

### INSUFFICIENT_REPRODUCTION_INFO
**Severity Range:** BLOCKING → MAJOR → MINOR
**Definition:** Bug report describes a problem but provides insufficient step-by-step reproduction to verify the fix.
**When to Flag:**
- Reproduction steps are vague ("sometimes it fails")
- Steps are missing context (environment variables, specific test data, order of operations)
- Steps can't be followed by someone unfamiliar with the system
- Multiple branches exist but only one is described
**Severity Defaults:**
- BLOCKING: Can't reproduce → can't verify fix is real
- MAJOR: Reproduction requires guesswork (wrong env, missing setup steps)
- MINOR: Reproduction works but is inefficient (many extraneous steps)
**Example:**
```
[BLOCKING] INSUFFICIENT_REPRODUCTION_INFO — Vague reproduction steps
Location: "How to reproduce" section
Finding: Issue says "API sometimes times out" with no step-by-step reproduction.
Resolution: Provide exact steps:
  1. npm start
  2. Create test user: curl -X POST /users -d '{"name":"alice"}'
  3. Send 100 concurrent requests: ab -n 100 -c 20 http://localhost:8000/users
  4. Observe: requests 45-78 timeout (verified: /var/log/app.log shows connection closed)
```

---

### UNCLEAR_ENVIRONMENT
**Severity Range:** BLOCKING → MAJOR → MINOR
**Definition:** Issue doesn't specify the environment where the problem occurs or where the feature should run.
**When to Flag:**
- Bug report doesn't specify Node/Python/Java version
- Doesn't specify OS (Ubuntu? macOS? Windows?)
- Doesn't specify environment (dev? staging? production?)
- Doesn't specify custom build flags, database version, or dependency versions
**Severity Defaults:**
- BLOCKING: Bug can't be reproduced without environment info (e.g., "fails in Python 3.8 but not 3.9")
- MAJOR: Environment is unclear (multiple possibilities; implementer must guess)
- MINOR: Environment is mostly clear but one detail is missing (DB version)
**Example:**
```
[MAJOR] UNCLEAR_ENVIRONMENT — Missing reproduction environment details
Location: Issue body (missing)
Finding: Bug report doesn't specify Node version, OS, environment (dev/prod).
Resolution: Add:
  ## Environment
  - Node version: 18.0.0 (run: node -v)
  - OS: Ubuntu 20.04 (run: uname -a)
  - Environment: production
  - Database: PostgreSQL 13
```

---

### PRIORITY_CONFLICT
**Severity Range:** BLOCKING → MAJOR → MINOR
**Definition:** Issue states contradictory optimization goals or requirements that can't both be satisfied without priority ordering.
**When to Flag:**
- Issue requires "minimize latency" AND "minimize cost" (can conflict)
- Issue requires "maximize throughput" AND "keep memory < 100MB" (might conflict)
- Constraints are mutually exclusive without priority ordering
- Trade-off exists but no explicit priority is stated
**Severity Defaults:**
- BLOCKING: Goals are fundamentally incompatible (e.g., "infinite performance, zero cost"); no priority stated
- MAJOR: Goals can conflict; priority not explicit; implementer must guess
- MINOR: Clear trade-off but secondary goal is mentioned without explicit priority
**Example:**
```
[MAJOR] PRIORITY_CONFLICT — Contradictory optimization goals without priority
Location: Acceptance criteria
Finding: Issue requires "response time < 50ms" AND "memory footprint < 100MB", but cache-based 
  solution (fast) uses 200MB; streaming solution (memory-efficient) has p99 > 100ms.
Resolution: State priority:
  Priority 1: Latency < 50ms (mission-critical)
  Priority 2: Memory < 100MB (nice-to-have)
  If both impossible, latency wins.
```

---

### UNVERIFIED_DEPENDENCY_CLAIM
**Severity Range:** BLOCKING → MAJOR → MINOR
**Definition:** Issue assumes an external tool or library has a capability without citing verification.
**When to Flag:**
- Issue states "Postgres handles JSON arrays natively" (true in Postgres 12+, but version not specified)
- Issue says "GitHub API provides webhook signatures" (true, but implementation details needed)
- Issue claims "Node.js streams are memory-efficient" (true, but buffer sizes not specified)
- Behavioral claim is made without citing version, test, or documentation reference
**Severity Defaults:**
- BLOCKING: Critical claim is unverified (implementer builds on wrong assumption; breaks at integration)
- MAJOR: Claim is probably true but unverified (implementer should verify before committing)
- MINOR: Claim is ancillary but unverified (doesn't block implementation but adds research overhead)
**Example:**
```
[MAJOR] UNVERIFIED_DEPENDENCY_CLAIM — Unverified Postgres JSON capability
Location: "Implementation approach" section
Finding: Issue states "Postgres JSON operators handle nested arrays" but doesn't specify 
  version or verify the capability. JSON operators vary by Postgres version.
Resolution: Specify: "Requires PostgreSQL 12+. Verified: jsonb_set handles nested paths 
  in PG v12+ (tested in v14.2). See: https://www.postgresql.org/docs/14/functions-json.html"
```

---

### TEST_PROCEDURE_GAP
**Severity Range:** BLOCKING → MAJOR → MINOR
**Definition:** Acceptance criteria or requirements are defined, but no step-by-step test procedure exists to verify the issue is fixed or feature works.
**Note:** This is an issue-specific variant of TEST_GAP. Specs have a "Testing" section; issues have "Acceptance Criteria" and "Test Procedure" fields.
**When to Flag:**
- Acceptance criteria listed but no test procedure explains HOW to test
- Requirements listed but no reproduction procedure to verify they work
- Criteria reference external tools or test harnesses not documented
**Severity Defaults:**
- BLOCKING: Can't verify the issue is fixed without guesswork
- MAJOR: Test procedure exists but is ambiguous or incomplete
- MINOR: Test procedure is clear but could be more efficient
**Example:**
```
[MAJOR] TEST_PROCEDURE_GAP — Acceptance criteria present but no test procedure
Location: Acceptance criteria
Finding: Criteria say "user can log in" but no test procedure explains how to test it.
Resolution: Add section:
  ## Test Procedure
  1. Start server: npm start
  2. Navigate to login page: http://localhost:3000/login
  3. Enter credentials: user=alice, pass=secret
  4. Verify: redirected to /home and session cookie present
```
