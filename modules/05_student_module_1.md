Module 05 – Student Module: Citation extractor

Identify citations:

Detection: Parse in-text references, bracketed numerals, author-year forms, and footnote markers within each section’s chunks.

Mapping: Associate each citation occurrence with its originating section and chunk.

Notes for rendering:

Per-section citation notes: Count of citations and patterns (e.g., many prior-work references).

Integration:

Section-by-Section Table: Optionally reflect dense citation contexts in “Key Ideas” if explicitly tied to claims.

Checks & Warnings: Flag if citation lists are present without explanatory text (may limit summarization fidelity in strict mode).

Boundaries:

No external resolution: Do not resolve citations to external sources or add bibliographic details not present in paper_text.

Strict mode: Do not attribute claims to citations beyond what the text explicitly states.
