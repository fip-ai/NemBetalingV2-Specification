# Ambiguities and Decisions

## Certain
- The app integrates Acubiz file flows through Azure Blob Storage.
- Journal import destination is configured centrally in a singleton setup record.
- Azure secret storage is isolated from normal table fields.
- CSV expense import and employee export are core behaviors.
- Blob forwarding to Continia can run through three independent configuration channels.
- The app depends on several shared tables from another app and does not own them.

## High-confidence inference
- The app is meant for internal administration and scheduled/manual operational processing rather than frequent end-user interaction.
- The receipt import is intended to pair Acubiz expenses with supporting document attachments for finance review.
- The three Continia forwarding scenarios exist to support separate inbound sources or business routes without changing code.

## Ambiguous
- The exact full contents of the truncated page-extension and permission-set source should be re-read during a final polish pass, although their overall purpose is already clear.
- The exact semantics of the finance-text carry-over logic in the CSV import appear legacy and may need careful reproduction testing.
- The exact workbook column semantics for employee export should be reconstructed in more detail during a production rebuild.
- Some supporting shared-table assumptions live outside this repository and cannot be fully inferred here.

## Required builder decisions
| Topic | Safe decision | Rationale |
|-------|---------------|-----------|
| Page extension action placement | keep a clear General Journal action for import | preserves workflow entry point without depending on exact legacy placement |
| Permission coverage | explicitly include all production processing codeunits | current source may under-declare one orchestrator |
| Employee export implementation | preserve output columns and enrichment behavior, but modernize implementation | behavior matters more than xmlport mechanics |
| Processing logs | add a lightweight processing log | improves supportability without changing business outcome |
| Target posture | declare Cloud explicitly in rebuild | aligns with modern BC deployment expectations |
