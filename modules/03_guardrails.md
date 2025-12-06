
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
