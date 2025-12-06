# Module 03 – Guardrails

> File: `modules/03_guardrails.md`

---

## Change Log

- **[2025-12-06]** Added `evidence_mode` flag with `"normal"` and `"strict"` values and associated behavior.
- **[2025-12-06]** Implemented standardized warning messages for missing, empty, and very short sections.
- **[2025-12-06]** Added logic to signal when strict evidence mode cannot safely summarize a section.

---

## Purpose

Enforce safety, evidence, and hallucination constraints across the entire summarization process, with special focus on section-level behavior. This module:

- Ensures the system uses **only** information present in the provided `paper_text`.
- Detects:
  - Missing / empty sections.
  - Very short sections (< 50 words).
- Implements **Strict Evidence Mode**, where only explicitly supported claims may be summarized.
- Provides standardized warnings for use in the Section Loop and Checks & Warnings output.

---

## Inputs

For each section (typically called from Module 02 – Section Loop):

- `section_id`: identifier or title of the section.
- `section_text`: raw text for this section (may be empty).
- `evidence_mode`: `"normal"` or `"strict"`.
- Optional:
  - `language`
  - `max_length`
  - Precomputed metadata (e.g., word count, chunking info).

---

## Outputs

Return a Guardrails response object with:

- `sanitized_section_text`: cleaned text to be used for summarization.
- `is_missing_or_empty` (boolean)
- `is_too_short` (boolean)
- `strict_insufficient_detail` (boolean)
- `warnings` (list of standardized warning messages)

The following standardized messages **must** be used:

- For missing or empty sections:
  - `"Section skipped: no usable text was provided."`
- For very short sections (< 50 words):
  - `"Section very short: summary may be incomplete."`
- For strict evidence insufficiency:
  - `"The source text does not provide enough detail to summarize this section in strict evidence mode."`

---

## Global Guardrail Principles

1. **No Hallucinations**
   - The summarizer must not invent:
     - New sections
     - New experimental results
     - New equations or algorithms
     - New citations, authors, or venues
   - All claims must be grounded in the provided `section_text` and associated chunks.

2. **Use Only Provided Text**
   - Do not rely on outside knowledge of the topic, even if the paper is well-known.
   - Do not fill gaps with guessed content.
   - If the text is vague, the summary should explicitly reflect that limitation.

3. **Section Isolation**
   - Guardrails must preserve boundaries between sections.
   - Information from one section must not be presented as if it belongs to another section.

4. **Chunking Awareness**
   - If sections are chunked due to length, ensure chunks are clearly mapped back to the original section.
   - Guardrails must treat all chunks for a section as part of a single logical unit and avoid mixing content across sections.

---

## Core Logic

### Step 1 – Normalize and Inspect Section Text

1. Trim whitespace from `section_text`.
2. Compute a rough word count (e.g., by splitting on whitespace).
3. Initialize flags:
   - `is_missing_or_empty = False`
   - `is_too_short = False`
   - `strict_insufficient_detail = False`
   - `warnings = []`
4. Set `sanitized_section_text = section_text` (will be updated as needed).

---

### Step 2 – Detect Missing or Empty Sections

1. If `section_text` is `None`, empty, or contains no meaningful characters:
   - Set `is_missing_or_empty = True`.
   - Append the standardized warning:
     - `"Section skipped: no usable text was provided."`
   - Set `sanitized_section_text = ""`.
   - Return the Guardrails response object immediately (no summarization should occur for this section).

---

### Step 3 – Detect Very Short Sections (< 50 Words)

1. If `word_count < 50`:
   - Set `is_too_short = True`.
   - Append the standardized warning:
     - `"Section very short: summary may be incomplete."`
2. Do **not** block summarization; instead, allow a summary but communicate the limitation through warnings.

---

### Step 4 – Enforce Evidence Mode

#### 4.1 Normal Evidence Mode (`evidence_mode = "normal"`)

1. In `"normal"` mode:
   - The summarizer may perform cautious paraphrasing and aggregation of ideas.
   - However, it must still:
     - Use only information that can be traced back to `section_text`.
     - Avoid fabricating details, numbers, or citations.
2. No additional flags are set beyond the missing/short checks above.

#### 4.2 Strict Evidence Mode (`evidence_mode = "strict"`)

1. In `"strict"` mode:
   - The summarizer must **only** include:
     - Claims explicitly stated in `section_text`.
     - Equations, variables, and results that appear directly in the text.
   - It must **not**:
     - Infer hidden motivations or unstated implications.
     - Generalize from the paper to the entire field.
     - Add any background knowledge that is not clearly in the text.

2. If `section_text` is too vague, purely high-level, or lacks concrete details such that a faithful summary would require inference:
   - Set `strict_insufficient_detail = True`.
   - Append the standardized warning:
     - `"The source text does not provide enough detail to summarize this section in strict evidence mode."`
   - The Section Loop should respond to this flag by:
     - Using the warning text as the section summary, **instead of** generating a normal summary.

3. When `strict_insufficient_detail` is **not** set:
   - `sanitized_section_text` may be lightly cleaned (e.g., removing extraneous formatting, repeated headings), but:
     - All remaining content must be directly traceable to the original text.
     - No external expansions are allowed.

---

### Step 5 – Chunking and Cross-Section Integrity

1. If the paper is long and has been split into chunks:
   - Ensure all chunks associated with `section_id` are concatenated in order in `sanitized_section_text`.
   - Do **not** mix text from different sections.
2. The Guardrails module must maintain the invariant:
   - “One section_id → one logical text unit for summarization.”

---

### Step 6 – Return Guardrails Response

At the end of processing (unless returned early for missing/empty sections), return an object with:

- `sanitized_section_text`
- `is_missing_or_empty`
- `is_too_short`
- `strict_insufficient_detail`
- `warnings`

These values are consumed by Module 02 – Section Loop and later reflected in:

- Section-level summaries,
- The `# Checks & Warnings` section,
- Any user-facing indication of missing, short, or limited-detail sections.

---

## Behavioral Requirements Summary

- Always use standardized warning messages verbatim.
- Never allow invented content in any evidence mode.
- In `"strict"` mode, prefer **explicitly stating limitations** over guessing.
- Ensure that Section Loop logic respects `strict_insufficient_detail` by not generating normal summaries in that case.
