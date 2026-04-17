# Numerical Precision Rules (Academic Standard)

**Default mode: Strict academic precision. Exact numbers only.**

| Rule | Source | Claim | Status |
|------|--------|-------|--------|
| Exact match required | 96.555% | 96.555% | Verified |
| Any rounding = error | 96.555% | 97% | Numerical Error |
| Any rounding = error | 96.555% | 96.6% | Numerical Error |
| Truncation = error | 96.555% | 96.5% | Numerical Error |
| Sig figs must match | 0.834 | 0.83 | Numerical Error |
| Units must match | 96.555% | 0.96555 | Numerical Error |
| Direction matters | +12% growth | +15% growth | Hallucination |
| Order of magnitude | $4.7B | $47B | Hallucination |

**Exception:** If the source itself provides a rounded figure (e.g., "96.555% (approximately 97%)"), then claiming "97%" is Verified — cite the approximation.

## Confidence Classification

| Level | Criteria | Use when |
|-------|----------|----------|
| **exact** | ≥95% word overlap OR identical number with identical units | Direct quote, exact statistic |
| **paraphrase** | Same fact, different words, no interpretation added | Restated finding |
| **interpretation** | Inference drawn from source data | Calculated from source, synthesized |

**Rule:** When uncertain between levels, use the MORE CONSERVATIVE option and flag for review.
