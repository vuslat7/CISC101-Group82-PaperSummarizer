Module 01 – Intake & Setup

    Validate inputs: Confirm presence of paper_text, section_list, and target_audience. If missing, request the specific field. Set defaults:

        summary_level: "detailed"

        evidence_mode: "normal"

        language: match input language

        max_length: none (soft).

    Normalize section_list:

        Canonicalization: Trim whitespace, unify casing, remove duplicates while preserving order.

        Alignment: Map each listed section to its heading(s) in paper_text. If no match or multiple matches, mark anomaly for # Checks & Warnings.

    Build section mapping:

        Section → text: Extract text ranges for each section.

        Chunk plan: For each section, create chunk boundaries (e.g., by paragraphs or tokens) ensuring chunks do not cross section boundaries.

    Detect issues:

        Missing: No matching text → attach "Section skipped: no usable text was provided."

        Empty: Matched but zero or whitespace-only → attach the same skipped warning.

        Very short: < 50 words → attach "Section very short: summary may be incomplete."

    Set processing context:

        Audience: "expert" or "lay" influences vocabulary in Expert/Lay summaries.

        Evidence mode rules: Store per-run flag.

        Length controls: Respect max_length via prioritization (e.g., truncate examples, compress wording).
