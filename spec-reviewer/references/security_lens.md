# Security Lens — STRIDE for Spec Review

**Conditional reference.** Load when spec mentions auth, user data, payments, PII, file uploads, or encryption.

**Framework:** STRIDE (Microsoft SDL). For each external interface, trust boundary, and data flow in the spec, apply the six questions below.

---

## STRIDE Questions

| Threat | Reviewer question | Example finding |
|---|---|---|
| **S**poofing | Is identity verified at every trust boundary? Is the auth mechanism named? | "API accepts `user_id` in body — no server-side identity verification specified" |
| **T**ampering | Is integrity enforced in transit and at rest? Are user-supplied inputs validated? | "File path from user input passed to storage — no sanitization or allowlist mentioned" |
| **R**epudiation | Is there an immutable audit trail for sensitive/irreversible operations? | "Financial withdrawals have no audit log; no append-only store specified" |
| **I**nformation disclosure | Is PII handled with least privilege? What do error responses reveal? Are secrets separated? | "Error handler returns raw exceptions — may expose DB schema or internal paths" |
| **D**enial of service | Are rate limits, resource ceilings, and timeouts defined for public-facing endpoints? | "File upload endpoint has no size cap or rate limit — trivial disk exhaustion" |
| **E**levation of privilege | Can a lower-privilege actor reach higher-privilege ops? Is authz enforced server-side? | "Admin route protected by client-side role flag — no server-side enforcement specified" |

---

## How to Apply

1. List external interfaces, user roles, trust boundaries, and data flows.
2. For each, run the six STRIDE questions.
3. File a finding for every unanswered question — the absence of an answer is the finding.
4. No external interfaces or user data → note "STRIDE not applicable" and skip.

---

## Severity Guidance

| Condition | Severity |
|---|---|
| Multi-user system with no auth model | BLOCKING |
| No audit trail for financial / irreversible ops | BLOCKING |
| Authorization client-side only | BLOCKING |
| Auth described but trust boundaries not enumerated | MAJOR |
| Audit log writable by the actor it audits | MAJOR |
| Vague "we'll add rate limiting" with no ceiling | MAJOR |
| User-supplied paths without validation mention | MAJOR |
| Error responses with unspecified verbosity on sensitive paths | MAJOR |
| PII stored/transmitted without encryption requirement | MAJOR |
| Secrets in config without separation guidance | MAJOR |
| Rate limiting mentioned but no concrete limits | MINOR |
| HTTPS not stated but implied by modern stack | MINOR |

---

## Routing Triggers

Load when spec mentions **any** of:
- Authentication, authorization, credentials, tokens, API keys, OAuth, JWT
- User data, PII, personal information, GDPR
- Financial transactions, payments, balances, wallets
- File uploads, user-supplied input, external data sources
- Encryption, TLS, HTTPS, certificates, secrets

---

## Sources

- OWASP Threat Modeling: https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html
- STRIDE: Microsoft SDL · Threat Modeling Manifesto
