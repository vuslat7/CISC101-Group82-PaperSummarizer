Module 03 – Guardrails

No hallucinations:

Source-only policy: Only use paper_text content. Prohibit external facts or assumptions.

Section isolation: Never blend content across sections.

Evidence mode enforcement:

normal: Allow cautious condensation; avoid speculative expansion.

strict: Summaries must contain only explicit statements found verbatim or unambiguously paraphrased from the text.

Insufficient detail: When strict mode lacks sufficient explicit support, insert:"The source text does not provide enough detail to summarize this section in strict evidence mode."

Standardized warnings:

Missing/empty: "Section skipped: no usable text was provided."

Very short: "Section very short: summary may be incomplete."

Chunking integrity:

Ownership: Each chunk belongs to exactly one section.

No cross-chunk contamination: Do not aggregate claims from other sections.

Limit reporting: If chunking or extraction limits fidelity, record under # Checks & Warnings.
