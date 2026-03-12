# References Index

## Always Load (required for every review)

- `spec_template.md` — Universal spec template. Baseline for Scorecard. ~5KB.
- `gap_taxonomy.md` — Gap category labels and severity defaults. ~11KB.
- `examples.md` — Calibration anchors for severity rating. ~3KB.

## Load Conditionally (check spec content first)

- `nfr_checklist.md` — Load if spec mentions: performance targets, SLAs, availability, scaling, reliability, throughput, latency. ~4KB.
- `api_review_lens.md` — Load if spec defines: HTTP endpoints, API contracts, RPC interfaces, REST/GraphQL schemas. ~4KB.
- `security_lens.md` — Load if spec handles: credentials, auth, user data, financial transactions, PII, encryption. ~3KB.
- `adr_patterns.md` — Load if spec contains: "Alternatives Considered", decision rationale, ADR, or trade-off analysis sections. ~2KB.

## Loading Strategy

The reviewer loads INDEX.md first, then follows a three-phase approach:

1. **Always-Load Phase:** Load all files in the "Always Load" section unconditionally. These are required for every review and are the foundation of gap detection and severity calibration.

2. **Scan-and-Route Phase:** Before proceeding with the review, scan the spec's headings, first 2 paragraphs, and any section that mentions the keywords above. Based on this quick scan, decide which conditional reference files are relevant.

3. **Selective-Load Phase:** Load only the conditional references identified in the scan. Skip the others to preserve context budget.

This approach reduces context overhead from ~50-80KB (all files) to ~19-22KB (always-load + 1-2 conditional files) for typical specs.
