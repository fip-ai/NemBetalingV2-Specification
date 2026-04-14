# Security and Configuration

## Permission model

### Acubiz Permission
- Intended role: any user who needs to configure or run Acubiz integration workflows.
- Access granted:
  - full table and data access for ADD Acubiz Setup and ADD Azure Config
  - execution of import codeunit
  - access to setup and Azure configuration pages
  - execution of both xmlports (expense import and employee export)
- Dependencies: does not include the import/export codeunit explicitly; permission set only covers the objects listed in source. Rebuild may want to ensure the forwarding codeunit is also covered.

## Configuration model

### ADD Acubiz Setup
- Purpose: configures where imported transactions land.
- Required settings:
  - Journal Template Name
  - Journal Batch Name
- Defaults: none; must be set manually.
- Consequences of missing/invalid setup: import codeunit will fail or create lines in unexpected location.

### ADD Azure Config (per integration type)
- Purpose: configures Azure storage access and routing behavior per scenario.
- Required settings:
  - Storage Account Name
  - Container Name
  - Storage Account Key (isolated secret)
  - Storage Subdirectory (for most scenarios)
- Optional settings:
  - Storage Archive Subdirectory (if blank, processed files are just deleted)
  - Subject (for email forwarding)
  - Document Category (for email forwarding)
  - Full Name Filter (for selective processing)
  - Check GLN (for XML validation)
- Defaults: none; each integration scenario must be configured explicitly.
- Consequences of missing config: processing codeunits fail immediately with explicit error.

## Sensitive data considerations
- Azure shared keys are stored in company isolated storage, not as plain table fields. This is the correct pattern.
- HMAC/keys are displayed as masked values on the configuration page.
- Rebuild should preserve isolated storage approach and consider whether additional encryption or Azure Key Vault integration is warranted.

## Environment and release posture
- Cloud / on-prem assumptions: no explicit target in manifest; rebuild should declare Cloud explicitly.
- Telemetry expectations: none evidenced; rebuild should add telemetry for file-processing outcomes.
- Resource exposure posture: currently permissive development posture.
- Operational support posture: no built-in event logging or error tracking beyond immediate errors.

## Rebuild guidance
- Preserve the isolated-storage secret management pattern.
- Add the forwarding codeunit to the permission set.
- Consider adding a processing log table for visibility into file movement history.
- Harden manifest metadata and declare explicit target.