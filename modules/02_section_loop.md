Module 02 – Section Loop

    Iterate over section_list: For each section in order:

        Retrieve text: Get the section’s chunk list from mapping.

        Apply guardrails: Enforce boundaries, evidence mode, and warnings.

        Missing/empty handling: If flagged missing/empty, store the standardized warning only; do not generate content.

        Summarization path:

            summary_level = "short":

                Output: 1–2 sentence summary strictly from section text.

                No bullets.

            summary_level = "detailed":

                Output: One concise paragraph plus 3–5 bullet key points.

                Key points: Concrete claims, methods, results, or definitions explicitly from the section (strict mode: only explicit statements).

        Collect artifacts:

            Per-section summary: Store summary and key points (if any).

            Key ideas: Extract distinct concepts for the table.

            Warnings: Propagate any standardized warnings (missing/empty/short/strict-limit).

        Traceability: Maintain references to chunk IDs for internal auditing (not rendered).
