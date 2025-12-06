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

- `paper_text`
- `section_list`
- `target_audience`
- `summary_level`
- `evidence_mode`
- Optional:
  - `language`
  - `max_length`
- From **Module 01**:
  - `section_to_text_map`
  - `section_metadata`

- Uses outputs from **Module 03 – Guardrails**:
  - Missing/empty flag  
  - Short section flag  
  - Strict evidence insufficiency  
  - Standardized warnings  
  - Sanitized section text  

---

## Outputs

For each section, produce:

- `section_summary[section_id]`
- `section_warnings[section_id]`
- `section_glossary_candidates[section_id]`
- `section_style_metadata[section_id]`

These feed into Module 04 (Rendering & Refinement).

---

## Core Logic (Section Loop)

1. **Initialize Storage**
   - Create:
     - `section_summary`
     - `section_warnings`
     - `section_glossary_candidates`
     - `section_style_metadata`

2. **Loop Over Sections**

   For each `section_id` in `section_list`:

   ### Step A — Retrieve Section Text
   - `section_text = section_to_text_map.get(section_id, "")`
   - Normalize spacing.

   ### Step B — Apply Guardrails
   - Call **Module 03 – Guardrails** with:
     - `section_text`
     - `section_id`
     - `evidence_mode`
   - Receive:
     - `sanitized_section_text`
     - `is_missing_or_empty`
     - `is_too_short`
     - `strict_insufficient_detail`
     - `warnings`

   - Initialize:  
     `section_warnings[section_id] = warnings`

   ### Step C — Handle Missing/Empty Sections
   - If `is_missing_or_empty`:
     - Store:
       ```
       "Section skipped: no usable text was provided."
       ```
     - `section_summary[section_id] = ""`
     - Continue to next section.

   ### Step D — Handle Strict Evidence Limitations
   - If `evidence_mode == "strict"` **and** `strict_insufficient_detail`:
     - Set:
       ```
       section_summary[section_id] =
       "The source text does not provide enough detail to summarize this section in strict evidence mode."
       ```
     - Continue to next section.

   ### Step E — Generate Summary According to `summary_level`
   - Let `clean_text = sanitized_section_text`

   #### Case 1: `summary_level == "short"`
   - Produce **1–2 concise sentences** using ONLY info from `clean_text`.
   - Tone:
     - `"expert"` → technical detail allowed  
     - `"lay"` → simpler explanations  
   - No bullet points.
   - Save to:
     `section_summary[section_id] = short_summary_text`

   #### Case 2: `summary_level == "detailed"`
   - Produce:
     1. A **short paragraph** (3–5 sentences)  
     2. A **bullet list of 3–5 key points**

   - Facts must come **only from `clean_text`**.
   - Tone follows `target_audience`.
   - Save to:
     `section_summary[section_id] = paragraph + bullet_list`

   ### Step F — Glossary Candidates
   - Extract technical terms and acronyms.
   - Add to:
     `section_glossary_candidates[section_id]`

   ### Step G — Store Style Metadata
   - Save:
     - `target_audience`
     - `summary_level`
     - guardrail flags

3. **After Loop Ends, Return:**
   - `section_summary`
   - `section_warnings`
   - `section_glossary_candidates`
   - `section_style_metadata`

---

## Behavioral Requirements

- Follow Guardrails strictly.
- Do **not** hallucinate content.
- Only use information from `clean_text`.
- Respect section boundaries.
- Respect summary mode behavior:
  - **short → 1–2 sentences**
  - **detailed → paragraph + 3–5 bullets**
