# Output Format Specification

## Summary Block (Always First)

```markdown
## Verification Report

**Mode:** [Search / Doc-Only]
**Document:** [filename or description]
**Generated:** [timestamp]

### Summary
| Metric | Count |
|--------|-------|
| Total claims extracted | X |
| Verified | Y |
| Numerical Error | Z |
| Unverified | A |
| Hallucination | B |
| Misleading | C |
| Not in Source (doc-only) | D |

**Overall Status:** [PASS: All verified / FAIL: Issues found]
```

## Detailed Findings (Grouped by Status)

```markdown
### Verified Claims (N)

| ID | Claim | Source | Location | Confidence |
|----|-------|--------|----------|------------|
| C01 | "92.1% accuracy" | Chen et al. 2024 | Table 3, p.8 | exact |

### Numerical Errors (N)

| ID | Claim | Source Value | Claimed Value | Deviation | Fix |
|----|-------|--------------|---------------|-----------|-----|
| C03 | "97% accuracy" | 96.555% | 97% | +0.445% | Use 96.555% |

### Hallucinations (N)

| ID | Claim | Issue | Source Says |
|----|-------|-------|-------------|
| C05 | "3x faster" | Contradicts source | Source: 2.1x faster |

### Unverified (N)

| ID | Claim | Issue |
|----|-------|-------|
| C08 | "$50B market" | No authoritative source found |

### Misleading (N)

| ID | Claim | Issue | Missing Context |
|----|-------|-------|-----------------|
| C10 | "Best performance" | Cherry-picked metric | Only on subset; overall performance lower |
```

## Sources Consulted

```markdown
### Sources

| ID | Citation | Type | URL | Used For |
|----|----------|------|-----|----------|
| S1 | Chen et al. (2024) | arxiv | https://arxiv.org/... | C01, C02, C03 |
| S2 | Statista Market Report | report | https://statista.com/... | C08 |
```

## Numerical Error Detail Format

```markdown
### Numerical Error: [Claim ID]

| Field | Value |
|-------|-------|
| Claim | "Model achieves 97% accuracy" |
| Location | Slide 4, bullet 2 |
| Source | Chen et al. (2024), Table 3, p.8 |
| Source value | 96.555% |
| Claimed value | 97% |
| Deviation | +0.445% (rounded up) |
| Status | Numerical Error |
| Fix | Replace with: "Model achieves 96.555% accuracy" |
```

## Output Options

- **Quick:** Summary + critical issues only (Numerical Errors, Hallucinations, Unverified)
- **Full:** Complete traceability report with all claims
- **JSON:** Machine-readable audit (see [citation_schema.json](citation_schema.json))
