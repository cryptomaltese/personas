# NFR Checklist

Non-Functional Requirements (NFRs) define *how well* the system performs. Vague or missing NFRs cause production failures (e.g., bettersoftware.uk: missing file upload limits → storage exhaustion).

Load this reference when specs mention: performance, SLAs, availability, scaling, reliability, throughput, latency, security, uptime.

Use **Section 1** as a checklist: scan spec for each category. Flag MISSING_SECTION (absent), THIN_SECTION (present but vague), or AMBIGUITY (unmeasurable).

## Section 1: NFR Categories Checklist

| Category | What to check for | Example of "good" | Example of "vague" (flag it) |
|----------|-------------------|-------------------|------------------------------|
| **Performance** | Response time (P90/P99), throughput (ops/sec), resource use under load | P99 < 200ms for reads at 1k concurrent users; <80% CPU | "Fast", "optimized performance" |
| **Availability** | Uptime SLA (e.g., 99.9%), RTO (recovery time), RPO (data loss tolerance) | 99.9% uptime; RTO 15min; RPO 0 (no data loss) | "Highly available" |
| **Scalability** | Load limits, auto-scaling triggers, growth projections | Horizontal scale to 10x peak (10k users); auto-scale at 70% CPU | "Scales well" |
| **Reliability** | MTBF/MTTR, fault isolation, redundancy (N+1) | MTTR <1hr; single node failure → no data loss (replicated) | "Reliable system" |
| **Security** | Auth (e.g., JWT/OAuth), encryption (at-rest/transit), data classification, compliance | PII AES-256 at rest, TLS 1.3 in transit; JWT RS256 auth | "Secure", "security handled" |
| **Maintainability** | Deploy process (zero-downtime?), observability (logs/metrics), modularity | Zero-downtime deploys; JSON logs w/ trace IDs; 80% test coverage | "Easy to maintain" |
| **Compatibility** | Platforms/browsers/versions supported; API versioning/backward compat | Node ≥18, Chrome/FF/Safari latest-2; API v2 backward-compatible | "Modern systems" |

**Distilled from:**
- [Microsoft Engineering Playbook](https://microsoft.github.io/code-with-engineering-playbook/) — metrics/templates
- [ISO 25010](https://en.wikipedia.org/wiki/ISO/IEC_25010) — 8 characteristics: perf efficiency, compat, usability, reliability, security, maintainability, portability, functional suitability
- [bettersoftware.uk](https://bettersoftware.uk) — industry examples of NFR failures

## Section 2: Anti-Patterns

Flag these phrasings → THIN_SECTION or AMBIGUITY:
- "The system should be [adjective]" (fast, secure, reliable) **w/o numbers** → AMBIGUITY
- "Performance will be optimized" → THIN_SECTION (no targets/metrics)
- **No NFR section** in specs w/ money/auth/PII/external users → MISSING_SECTION, **MAJOR** min
- "We'll add monitoring/security later" → MVP_SCOPE (flag gap)
- "High-level" or "TBD" for critical NFRs → THIN_SECTION
- Adjectives w/o baselines: "responsive UI" (no load test?), "robust error handling" (no %?)

## Section 3: Severity Escalation

| Scenario | Severity |
|----------|----------|
| **Missing NFRs** in external-facing/money/auth/PII systems | **MAJOR** min (production risk) |
| **Missing NFRs** in single-user/internal tools | **MINOR** |
| **Vague NFRs** sounding specific but unmeasurable (e.g., "optimized") | **MAJOR** (false confidence > nothing) |
| **Present but untestable** (no metrics) | **THIN_SECTION**, **MINOR→MAJOR** if core path |
| **Conflicts** (e.g., "fast + cheap") | **BLOCKING** if unaddressed |

**Escalate if:** Spec handles real users/data (not prototype). Use scorecard: rate NFRs section Good/Thin/Missing.

~3.2KB
