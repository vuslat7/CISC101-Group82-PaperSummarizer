# Module 02 – Section Loop

> File: `modules/02_section_loop.md`

---

## Change Log

- **[2025-12-06]** Added `summary_level` variable (`"short"` vs `"detailed"`) and conditional behavior for each section.
- **[2025-12-06]** Integrated Guardrails module output (missing/empty/short sections, strict evidence signals).
- **[2025-12-06]** Ensured “detailed” mode produces a paragraph **plus** 3–5 bullet points per section.

---

## Purpose

Loop over each section of the research paper, apply guardrails, and generate section-level summaries that respect:

- `target_audience` (`"expert"` or `"lay"`)
- `summary_level` (`"short"` or `"detailed"`)
- `evidence_mode` (`"normal"` or `"strict"`)

The outputs of this loop feed into the Section-by-Section Table, expert/lay summaries, mini-glossary candidates, and Checks & Warnings.

---

## Inputs (from previous modules)

- `paper_text` (possibly pre-split into sections or chunks)
- `section_list`: ordered list of normalized section identifiers/titles
- `target_audience`: `"expert"` or `"lay"`
- `summary_level`: `"short"` or `"detailed"`
- `evidence_mode`: `"normal"` or `"strict"`
- Optional:
  - `language`
  - `max_length`
- From **Module 01 – Intake & Setup**:
  - `section_to_text_map`: mapping from section identifier → raw text
  - `section_metadata`: any precomputed stats (e.g., word counts, chunking info)

- Access to **Module 03 – Guardrails**, which:
  - Detects missing / empty / very short sections
  - Applies strict vs normal evidence rules
  - Returns standardized warning messages and flags

---

## Outputs (to later modules)

For each section in `section_list`, produce:

- `section_summary[section_id]`:
  - In `"short"` mode: 1–2 sentence summary (no bullet list)
  - In `"detailed"` mode: 1 short paragraph **plus** 3–5 bullet points
- `section_warnings[section_id]`:
  - Zero or more standardized warning strings, e.g.:
    - `"Section skipped: no usable text was provided."`
    - `"Section very short: summary may be incomplete."`
    - `"The source text does not provide enough detail to summarize this section in strict evidence mode."`
- `section_glossary_candidates[section_id]`:
  - Key technical terms, acronyms, or phrases to propose for the Mini-Glossary
- `section_style_metadata[section_id]`:
  - Any useful indicators for Module 04 (e.g., whether summary is expert- or lay-oriented)

These outputs are consumed by:

- Module 04 – Rendering & Refinement
- Student modules (e.g., Citation Extractor, Equation & Figure Explainer)

---

## Core Logic (Section Loop)

1. **Initialize Loop Structures**

   - Create empty dictionaries/maps:
     - `section_summary`
     - `section_warnings`
     - `section_glossary_candidates`
     - `section_style_metadata`

2. **Iterate Over Sections**

   - For each `section_id` in `section_list` (in order):

     1. **Retrieve Section Text**
        - Look up `section_text = section_to_text_map.get(section_id, "")`.
        - Trim whitespace and normalize spacing.

     2. **Call Guardrails Module**
        - Pass `section_text`, `section_id`, `evidence_mode`, and any precomputed metadata to **Module 03 – Guardrails**.
        - Receive a Guardrails response object with fields such as:
          - `is_missing_or_empty` (boolean)
          - `is_too_short` (boolean)
          - `strict_insufficient_detail` (boolean)
          - `warnings` (list of standardized warning strings)
          - `sanitized_section_text` (text cleaned for summarization, respecting evidence rules)

        - Initialize `section_warnings[section_id] = warnings` (may be empty).

     3. **Handle Missing / Empty Sections**

        - If `is_missing_or_empty == True`:
          - Do **not** generate a content summary.
          - Ensure the standardized warning is present in `section_warnings[section_id]`, e.g.:
            - `"Section skipped: no usable text was provided."`
          - Set `section_summary[section_id] = ""` (or a very minimal placeholder).
          - Continue to the next section in the loop.

     4. **Handle Strict Evidence Insufficiency**

        - If `evidence_mode == "strict"` **and** `strict_insufficient_detail == True`:
          - Do **not** extrapolate or guess.
          - Set the summary for this section to the standardized message:
            - `"The source text does not provide enough detail to summarize this section in strict evidence mode."`
          - Ensure this message appears in `section_summary[section_id]` and is reflected in `section_warnings[section_id]`.
          - Skip normal summarization logic and continue to the next section.

     5. **Generate Summary According to `summary_level`**

        - Let `clean_text = sanitized_section_text` from Guardrails.

        - **Case A: `summary_level == "short"`**
          - Generate **1–2 concise sentences** that:
            - Use only information present in `clean_text`.
            - Reflect section’s main goal, methods, and/or key results at a high level.
            - Respect `target_audience`:
              - `"expert"` → use technical terminology when helpful.
              - `"lay"` → prefer plainer language, short clauses, and intuitive explanations.
          - Do **not** produce bullet points in this mode.
          - Store result:
            - `section_summary[section_id] = short_summary_text`.

        - **Case B: `summary_level == "detailed"`**
          - Generate a **short paragraph** (3–5 sentences) summarizing:
            - The role of the section in the paper (e.g., background, method, experiment, result).
            - The most important ideas, techniques, or findings.
          - Then generate a **bullet list of 3–5 key points**, such as:
            - Important definitions or concepts.
            - Main methodological steps.
            - Key experimental findings.
            - Major contributions or limitations.
          - Respect `target_audience`:
            - For `"expert"`, allow technical depth and field-specific vocabulary.
            - For `"lay"`, explain technical ideas in simpler language and avoid unexplained jargon.
          - Store result as a combined block:
            - `section_summary[section_id] = paragraph + bullet_list_block`.

     6. **Populate Glossary Candidates**

        - As part of summarization, extract notable technical terms, acronyms, or phrases that may require definition.
        - Add these to `section_glossary_candidates[section_id]`.

     7. **Record Style Metadata**

        - Optionally, store metadata such as:
          - `section_style_metadata[section_id].target_audience = target_audience`
          - `section_style_metadata[section_id].summary_level = summary_level`
          - Any flags from Guardrails (e.g., `"very_short_text"`, `"strict_mode_active"`).

3. **After the Loop**

   - Return the following objects for downstream modules:
     - `section_summary`
     - `section_warnings`
     - `section_glossary_candidates`
     - `section_style_metadata`

---

## Behavioral Requirements Summary

- Always obey Guardrails decisions (no summaries for missing/empty sections; no guessing in strict evidence mode).
- Ensure:
  - `"short"` = 1–2 sentences, **no bullets**.
  - `"detailed"` = paragraph **plus** 3–5 bullets.
- Never hallucinate sections, results, or citations.
- Use only the information present in the provided `paper_text` for each section.
- Adapt tone and complexity based on `target_audience` while preserving factual accuracy.
