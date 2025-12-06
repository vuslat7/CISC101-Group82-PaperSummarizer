# Module 03 – Guardrails

> File: `modules/03_guardrails.md`

---

## Change Log

- **[2025-12-06]** Added `evidence_mode` with `"strict"` and `"normal"` modes.
- **[2025-12-06]** Added standardized warnings for missing/empty sections and sections under 50 words.
- **[2025-12-06]** Added strict evidence-mode error message for insufficient detail.
- **[2025-12-06]** Strengthened hallucination-prevention rules and section isolation.

---

## Purpose

The Guardrails module ensures **safe, evidence-grounded summarization**.  
It prevents hallucinations, enforces section integrity, and determines if a section is usable for summarization.  
It also implements **strict evidence mode**, which limits summaries to explicit text only.

---

## Required Behaviors

### This module MUST:
- Prevent hallucinations  
- Prevent invented results, citations, definitions, or equations  
- Use **ONLY** content present in the provided section text  
- Detect missing or empty sections  
- Detect sections under 50 words  
- Produce standardized warnings  
- Enforce strict evidence mode  
- Maintain section boundaries  
- Handle chunked text safely  

---

## Inputs

- `section_id`
- `section_text`
- `evidence_mode` (`"normal"` or `"strict"`)
- Optional:
  - `language`
  - `max_length`

---

## Outputs

A **GuardrailsResponse** object containing:

- `sanitized_section_text`
- `is_missing_or_empty`
- `is_too_short`
- `strict_insufficient_detail`
- `warnings` (list)

---

## Standardized Warning Messages

These **must** be used exactly:

1. Missing/empty:

Section skipped: no usable text was provided.


2. Too short (< 50 words):

Section very short: summary may be incomplete.


3. Strict evidence insufficient:

The source text does not provide enough detail to summarize this section in strict evidence mode.


---

## Core Logic

### Step 1 — Normalize Text
- Trim whitespace  
- `word_count = len(section_text.split())`  
- Initialize flags to `False`  
- Initialize `warnings = []`  
- Set `sanitized_section_text = section_text`

---

### Step 2 — Missing or Empty Check
If `section_text` is empty, `None`, or contains no meaningful characters:
- `is_missing_or_empty = True`
- Append missing-section warning
- `sanitized_section_text = ""`
- **Return immediately** (no summarization allowed)

---

### Step 3 — Very Short Section Check (< 50 words)
If `word_count < 50`:
- `is_too_short = True`
- Append “very short” warning

Summarization **can continue**, but user must be warned.

---

### Step 4 — Evidence Mode Enforcement

#### 👉 Case 1: evidence_mode = `"normal"`
- Allow cautious paraphrasing  
- No fabrication allowed  
- Only use information traceable to the text  

#### 👉 Case 2: evidence_mode = `"strict"`
- Must use ONLY explicit statements  
- NO inference  
- NO external knowledge  
- NO generalization  

If section has **high-level text only**, or lacks enough detail:
- `strict_insufficient_detail = True`
- Append strict-evidence warning
- Section Loop must skip normal summarization  
and use the warning **as the summary text**

---

### Step 5 — Chunk Integrity
If section is chunked due to long length:
- Concatenate only chunks belonging to `section_id`
- Never merge chunk text across different sections
- Ensure the final text still reflects the same section

---

### Step 6 — Return Guardrails Response
Return:

- `sanitized_section_text`
- `is_missing_or_empty`
- `is_too_short`
- `strict_insufficient_detail`
- `warnings`

---

## Behavioral Rules Summary

- Do not hallucinate or invent details  
- Must use warning messages exactly as written  
- Strict mode overrides normal summarization  
- Always preserve section boundaries  
- Maintain transparent limitations  
