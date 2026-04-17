---
name: citation-check-skill

description: "Vision-enabled verification gate with web search. Use when users want to (1) verify slides/reports/PDFs/images against authoritative online sources, (2) validate that citations actually exist and say what's claimed, (3) check charts/graphs/tables for accuracy, (4) audit AI-generated content in doc-only mode (no external knowledge). Two modes - search mode validates against web, doc-only mode ensures everything traces to provided documents. Supports content in any language."

---

# Citation & Hallucination Checker v2

Verification tool with vision + web search. Validates every claim against authoritative sources or provided documents. Works with content in any language.

**Design principle:** Deterministic verification. Same input → Same output.

---

## Two Verification Modes

### Mode 1: Search Verification (Default)

- Searches web for authoritative sources
- Validates citations actually exist
- Checks if cited sources say what's claimed
- Finds original data for statistics

### Mode 2: Doc-Only Verification

- User provides source document(s)
- EVERYTHING must trace to those docs
- Flags anything that appears to come from external knowledge
- Trigger: "only use this document" / "verify against the PDF only" / "don't search the web"

---

## Two-Pass Architecture

**Critical:** Always use two separate passes. Never interleave extraction and verification.

### Pass 1: Extraction Only

1. Read entire document/slides/images
2. Extract ALL claims using the Claim Extraction Rules below
3. Output numbered list: `[claim_id] | [claim_text] | [claim_type] | [location]`
4. **NO verification in this pass**
5. Present extraction to user for confirmation before proceeding

### Pass 2: Verification Only

1. Take Pass 1 output as fixed input
2. Verify each claim_id in sequential order
3. **NO re-extraction allowed** — work only with Pass 1 claims
4. Apply Status Decision Tree to each claim
5. Generate final report

This prevents "discovering new claims" mid-verification and ensures consistency.

---

## Claim Extraction Rules

### EXTRACT as claims:

| Type | Pattern | Example |
|------|---------|---------|
| **Statistic** | Any number with unit/context (%, $, count, ratio, decimal) | "92.3% accuracy", "$4.7B market" |
| **Comparative** | X is [comparative] than Y | "3x faster than baseline" |
| **Temporal** | Time-bound assertion | "In 2024, adoption reached..." |
| **Attribution** | Claim tied to source | "According to WHO...", "Smith et al. found..." |
| **Causal** | X causes/leads to/results in Y | "This reduces latency by..." |
| **Existence** | Asserts something exists/is true | "There are 500M users" |
| **Ranking** | Position claims | "largest", "first", "top 3" |
| **Quote** | Direct quotation | Any text in quotation marks attributed to source |

### DO NOT extract:

Definitions, opinions marked as such ("We believe..."), hypotheticals ("Could potentially..."), questions, unsourced future predictions, methodology descriptions ("We used PyTorch 2.0"), or acknowledgments.

### Extraction Output Format

```
[C01] | "Model achieves 96.555% accuracy on ImageNet" | Statistic | Slide 3, bullet 2
[C02] | "Outperforms GPT-4 by 12% on reasoning tasks" | Comparative | Slide 3, bullet 3
[C03] | "According to Chen et al. (2024), transformers scale linearly" | Attribution | Slide 5, para 1
[C04] | "Market size reached $4.7B in 2024" | Statistic + Temporal | Slide 7, chart title
```

---

## Status Decision Tree

Apply this tree to EVERY claim. Follow exactly — no shortcuts.

```
START
│
├─ Is this a CITATION claim (references a paper/report/source)?
│   ├─ YES → Go to CITATION VALIDATION
│   └─ NO → Go to STATISTIC/FACT VALIDATION
│
│
CITATION VALIDATION
│
├─ Step 1: Does the cited source exist?
│   │   Run ALL mandatory search queries (see references/search_templates.md)
│   │
│   ├─ NO → Status: "Citation Not Found"
│   │        STOP
│   │
│   └─ YES → Step 2: Does source contain the claimed topic?
│             │
│             ├─ NO → Status: "Misquoted"
│             │        STOP
│             │
│             └─ YES → Step 3: Does source support the exact claim?
│                       │
│                       ├─ YES (exact match) → Status: "Verified" (confidence: exact)
│                       ├─ YES (paraphrase) → Status: "Verified" (confidence: paraphrase)
│                       ├─ PARTIALLY (missing context) → Status: "Misleading"
│                       └─ NO (contradicts) → Status: "Hallucination"
│
│
STATISTIC/FACT VALIDATION
│
├─ Step 1: Can you find an authoritative source?
│   │   Run ALL mandatory search queries (see references/search_templates.md)
│   │
│   ├─ NO → Status: "Unverified"
│   │        STOP
│   │
│   └─ YES → Step 2: Do values match EXACTLY?
│             │
│             ├─ YES → Status: "Verified" (confidence: exact)
│             │        STOP
│             │
│             └─ NO → Status: "Numerical Error"
│                      See references/numerical_precision.md for classification rules
```

---

## Status Classifications

| Status | Meaning |
|--------|---------|
| ✅ Verified | Exact match with source |
| ⚠️ Numerical Error | Values don't match (e.g., 97% vs 96.555%) |
| ⚠️ Unverified | No authoritative source found |
| ❌ Hallucination | Contradicts source or fabricated |
| ❌ Misleading | Cherry-picked or missing context |
| ❌ Citation Not Found | Referenced paper/report doesn't exist |
| ❌ Not in Source | Claim can't be traced to provided doc (doc-only mode) |

---

## Search Templates & Numerical Precision

- **Search templates:** See [references/search_templates.md](references/search_templates.md) for all mandatory query patterns (academic, statistics, company, health, government)
- **Numerical precision rules:** See [references/numerical_precision.md](references/numerical_precision.md) for academic-standard precision and confidence classification
- **Output format:** See [references/output_format.md](references/output_format.md) for report structure and examples

---

## Source Authority Hierarchy

When multiple sources found, prefer in this order:

| Rank | Source Type | Examples |
|------|------------|----------|
| 1 | Primary source | Original study, official report, raw data |
| 2 | Government/institutional | WHO, CDC, World Bank, national statistics offices |
| 3 | Peer-reviewed publication | Nature, Science, IEEE, ACM |
| 4 | Industry reports (named) | Gartner, McKinsey, Statista (with methodology) |
| 5 | Reputable news citing primary | NYT, Reuters citing original source |
| 6 | Secondary compilations | Wikipedia (check their sources) |

**Rule:** If only Rank 5-6 sources found, status = "Unverified" with note "Only secondary sources found"

---

## Multi-Source Verification (Search Mode)

| Condition | Sources Required |
|-----------|-----------------|
| Primary source found | 1 (if authoritative: .gov, peer-reviewed, official) |
| Only secondary sources | ≥2 independent sources agreeing |
| Sources conflict | Status = "Unverified", note the conflict |

---

## Tie-Breaker Rules

When uncertain, apply these rules. No judgment calls.

| Situation | Rule |
|-----------|------|
| Missing date on claim | Assume most recent year; flag "needs date" |
| Conflicting sources | Use most recent authoritative source; cite both; note conflict |
| Source not found after all queries | Status = "Unverified" (NOT "Hallucination") |
| Number differs due to currency conversion | Flag as "Needs clarification: currency/units" |
| Same org, multiple reports | Use most recent; cite with date |
| Claim uses "approximately" or "about" | Still verify base number is in valid range (±10% of source) |
| Source is paywalled | Note "Source behind paywall, unable to verify exact text" |
| Source is in different language | Translate and verify; note translation |

---

## Visual Data Verification

For every chart, graph, table, or diagram:

1. **Extract** ALL values — axis labels, units, scale, legend. Note visual distortions (truncated axes, 3D effects)
2. **Find source** — search mode: run search templates; doc-only mode: locate in source document
3. **Compare value-by-value** using a table: Visual Element | Extracted Value | Source Value | Status
4. **Check visual integrity** — flag axis manipulation, 3D exaggeration, omitted error bars, or cherry-picked timeframes

---

## Doc-Only Mode Workflow

**Trigger phrases:** "only use this document", "don't search the web", "verify against the PDF only", "everything should be from the source"

1. **Index source document** — build complete index (pages, text summaries, statistics, tables, figures) before any verification
2. **Apply two-pass architecture** — same as search mode, but verification uses ONLY the source index
3. **Trace each claim** — search index for matching values/terms; if not found → Status: "Not in Source"
4. **Flag ALL external knowledge** — in doc-only mode, ANY claim not traceable to source is flagged, even if true

---

## Critical Rules

1. **Two-pass always** — Extract first, verify second. Never interleave.
2. **Every claim gets checked** — No exceptions, no skipping "obvious" ones.
3. **Exact numbers only** — 96.555% ≠ 97% in academic mode. See [references/numerical_precision.md](references/numerical_precision.md).
4. **Find the origin** — Don't accept secondary sources citing unknown primaries.
5. **Run ALL search templates** — Don't stop after first result. See [references/search_templates.md](references/search_templates.md).
6. **Citations must be real** — Search to confirm papers/reports exist.
7. **Check what sources actually say** — A real paper can still be misquoted.
8. **In doc-only mode, flag ALL external knowledge** — Even if it's true.
9. **When uncertain, be conservative** — "Unverified" is safer than false "Verified".
10. **Follow tie-breaker rules** — No ad-hoc judgment calls.

---

## Language Support

- Accepts content in any language
- Searches in the appropriate language for sources
- Reports in the same language as user's request
- Cross-language verification supported (e.g., Chinese slides citing English papers)
- When translating for verification, note: "Translated from [language]"
