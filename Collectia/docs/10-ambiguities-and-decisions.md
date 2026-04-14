# Ambiguities and Decisions

## Certain
- The app creates debt-collection cases at Collectia from issued reminders.
- It supports both manual and automatic case creation.
- Domestic and foreign customers produce different SOAP payloads.
- Invoice PDFs are generated and uploaded for sales and service invoices.
- Non-invoice documents are submitted as metadata only.
- Customer blocking is conditional on a setup flag.
- Activity logging is optional and configurable.
- Credentials are currently stored insecurely.

## High-confidence inference
- The "Activate" procedure exists but is not called during the standard case-creation workflow. It may be used in other contexts or was planned for future use.
- The logout procedure exists but is not called after case creation. This is likely an oversight.
- The hardcoded "FAK" background type and the fixed language "DA" suggest the service is Denmark-specific.
- The three SOAP document-sending procedures are functionally identical except for the source record type and one field difference. This is clearly copy-paste duplication.

## Ambiguous
- Whether the "Activate" procedure should be called as part of the case-creation workflow or separately.
- Whether logout should always follow login or whether the session persists for multiple operations.
- Whether the Job URL credential-embedding pattern is an intentional Collectia portal feature or a security oversight in the app.
- Whether the app should support currencies other than DKK for domestic customers.
- Whether the "Course" field has a defined set of valid values at Collectia or is free-text.

## Required builder decisions
| Topic | Safe decision | Rationale |
|-------|---------------|-----------|
| Activate after case creation | Do not call automatically unless explicitly requested | preserve current behavior, avoid unintended side effects |
| Logout after workflow | Always call logout in a try/finally pattern | prevents session leaks |
| Job URL credentials | Remove credentials from URL entirely | security non-negotiable |
| Multi-currency domestic | Keep DKK as domestic assumption | matches current behavior and Danish collection law context |
| Course validation | Treat as free-text with optional validation against Collectia documentation | no evidence of fixed set in current source |
