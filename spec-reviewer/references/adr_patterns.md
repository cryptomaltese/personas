# ADR Patterns for Review

## Templates

### Nygard (Original)
- Title
- Status
- Context
- Decision
- Consequences

### MADR (Markdown ADR)
- Title
- Context
- Decision Drivers
- Options (with pros/cons)
- Decision Outcome
- Links

### Y-Statement
"In the context of [X], facing [Y], we decided [Z] to achieve [A], accepting [B]"

## Quality Signals

| Quality | Signal |
|---------|--------|
| Good    | Each alternative has: what it is, why it was considered, why it was rejected (with specific tradeoff) |
| Thin    | Alternatives listed but rejection reasons are "didn't fit" or "too complex" without specifics |
| Missing | No alternatives section at all, or only one option presented |

## Anti-Patterns
- "We considered X but rejected it" (no reason)
- Alternatives listed after the fact to justify a predetermined choice (no genuine tradeoff analysis)
- Only the chosen option has pros/cons; rejected options have no analysis
- "Industry standard" as sole justification (which industry? which standard? who says?)

## Severity
- Alternatives absent in a spec with >3 design choices → MAJOR (MISSING_SECTION)
- Alternatives present but no rejection rationale → MAJOR (THIN_SECTION)
- Single alternative with shallow dismissal → MINOR

## Sources
- https://github.com/joelparkerhenderson/architecture-decision-record
- https://adr.github.io/adr-templates/
- https://aws.amazon.com/blogs/architecture/master-architecture-decision-records-adrs-best-practices-for-effective-decision-making/