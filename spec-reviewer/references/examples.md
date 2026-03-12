# Reviewer Few-Shot Examples

These examples calibrate severity levels. Use them to anchor ratings.

## BLOCKING Anchor

**Input context:** The spec defines `sessions_spawn(task, label, model, mode) → { childSessionKey }` as a function signature, but provides no verification, links to docs, or version citation for this OpenClaw tool interface.

**Finding:**
```
[BLOCKING] INTERFACE_UNDEFINED — sessions_spawn interface unverified
**Location:** "The Loop" → sessions_spawn interface block
**Finding:** The entire spawn chain depends on unverified parameter names/return shape. Two engineers would assume different tool schemas from OpenClaw docs vs. source — both wrong.
**Resolution:** Verify against OpenClaw 2026.3.8 tool schema, cite version, update spec.
```

**Calibration note:** BLOCKING because the spec has a structural hole — core data flow impossible without external knowledge. Not MAJOR: no implementation possible at all, not just error-prone.

## MAJOR Anchor

**Input context:** GraphQL query fetches `number`, `title`, `blockedBy` for issues, used by loop to build execution frontier and construct spawn queue.

**Finding:**
```
[MAJOR] INTERFACE_GAP — GraphQL query doesn't fetch fields the loop needs
**Location:** "The DAG" → GraphQL query block
**Finding:** Missing `body` (issue body = agent prompt) and `labels` (for model selection). Loop can't build prompts or select models — data flows broken.
**Resolution:** Add `body` and `labels(first: 10) { nodes { name } }` to query.
```

**Calibration note:** MAJOR because an engineer could write the loop but waste days debugging missing data. Not BLOCKING: query fixable from context. Not MINOR: breaks core functionality.

## MINOR Anchor

**Input context:** Lock files at `~/.ghralph/locks/{repo-slug}/{issue-number}.lock`; repo slugs used in paths and queue JSON.

**Finding:**
```
[MINOR] AMBIGUITY — Repo slug format never defined
**Location:** Artifact Map (lock paths), "The Loop" (queue JSON repo_slug)
**Finding:** Slug derivation (e.g., cryptomaltese/dnpm-hb → cryptomaltese-dnpm-hb?) unspecified.
**Resolution:** Add: "repo slug = `{owner}-{repo}` with `/` → `-`".
```

**Calibration note:** MINOR because engineer infers from example, but spec shouldn't require it. Not MAJOR: no wasted time, just mild annoyance.

## Anti-Pattern: Over-Severity

**Miscalibrated finding (from review):**
```
[BLOCKING] VENDOR_GAP — openclaw cron add flags assumed without citation
```

**Why wrong:** Should be MAJOR. Flags exist individually (verified), combined behavior testable in Phase 1. Engineer wastes ~1h testing, not blocked.

## Anti-Pattern: Under-Severity

**Miscalibrated finding (hypothetical, pattern from thin sections):**
```
[MINOR] THIN_SECTION — Pagination boundary
**Location:** "The DAG" → GraphQL query first: 100
**Finding:** No handling if >100 open issues.
```

**Why wrong:** Should be MAJOR. Repo with many orphans silently ignores issues — wastes significant debug time. Not MINOR: impacts correctness.